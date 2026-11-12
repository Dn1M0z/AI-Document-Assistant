# Architecture and decision records

**Developer:** Daniel Mozafari  
**Design baseline:** 12 November 2026  
[Requirements](REQUIREMENTS.md) · [User stories](USER_STORIES.md) · [Overall project status](README.md#project-status)

This document records the approved component boundaries and eight architecture decisions for the first release. These are design decisions, not evidence of implemented or tested behaviour. Named technologies remain provisional where indicated; exact versions, embedding model and generation provider will be recorded before implementation.

## Component responsibilities

| Component | Responsibilities | Boundary |
| --- | --- | --- |
| Frontend — React, TypeScript, PDF.js | Login, document selection, role-appropriate management controls, questions, answers, processing states and errors. Open cited PDF pages and transform highlights correctly during zoom and resize. | Calls the backend only. No database, vector-store or model-provider credentials. UI visibility is not an authorisation boundary. |
| Backend API — Python, FastAPI | Authenticate and authorise each protected request; validate limits; manage accounts and documents; create jobs; serve protected PDFs; record audit events; coordinate retrieval, generation and citation validation. | Owns application rules. Restricts retrieval to selected ready documents and rechecks source availability before returning evidence. |
| Processing worker — separate Python process | Validate PDF structure/page count; extract text and coordinates; create page-aware chunks; generate embeddings; index content; update processing outcomes; recover or fail interrupted work. | One document at a time initially, with configurable time/resource bounds. Does not manage sessions or generate user answers. |
| Metadata database — SQLite initially | Users, password hashes, roles, sessions, documents, jobs, canonical chunk text, page coordinates, audit records and evaluation configuration. | Authoritative source for document readiness, source identity and application permissions. Uses short transactions shared by API and worker. |
| Vector store — Qdrant provisionally | Embeddings, chunk identifiers and filterable document/processing-generation identifiers. Return ranked candidate chunks within the requested selection. | A rebuildable index; not the authority for permissions, source text or answer correctness. |
| Model provider — hosted generation through an adapter | Generate a structured answer from a standalone question and bounded retrieved evidence, including partial answers, conflicts and insufficient evidence. | No database access, browsing or actions. Returns evidence IDs; the backend resolves actual filenames, pages and highlights. |
| File storage — private persistent volume | Original PDFs under generated storage identifiers, available to the worker and protected backend source endpoints. | No public file URLs. Files are separate from canonical text and vector indexes and removed during document deletion. |

Local embedding generation is a shared application module used by the worker for chunks and by the backend for questions. Both use the same model, revision, dimensions and preprocessing configuration.

## Workflows and invariants

### Upload and processing

1. The backend checks the current role, enforces the 10,000,000-byte file limit and transactionally reserves capacity within the 20-document collection limit. Each selected upload is handled separately.
2. It stores the original PDF privately and creates a durable queued job. Failed storage/job creation is compensated so it does not leave an invisible orphan or permanently reserved capacity.
3. The worker claims one job, validates the PDF and the 100-page limit, then extracts text and source coordinates. Password-protected, corrupted and unsupported files fail with useful explanations.
4. It writes chunks and vectors for a processing generation. Image-only pages are reported as unsearchable; a document with no usable text never becomes ready.
5. The worker marks the generation ready only after its metadata and vector writes succeed. The frontend displays queued, processing, ready or failed.
6. Interrupted jobs are recovered or marked failed. Explicit retries are allowed only for Document Managers and Administrators and cannot duplicate searchable chunks.

### Question and citation

1. Validate the active session, a non-empty question of at most 2,000 characters, and a non-empty selection of ready documents.
2. Embed the question and retrieve candidates filtered to the selected document IDs and active processing generations.
3. Resolve canonical passages and source locations from metadata. Reject unavailable, stale or unselected candidates.
4. Assign evidence IDs and send the question and bounded passages to the generation adapter. Earlier questions and answers are not context.
5. Validate the response structure, evidence membership and quotations; resolve citations from stored metadata. Recheck availability before returning the response.
6. Render the answer and protected source links. Technical outages have error responses distinct from insufficient evidence.

### Deletion and recovery

Authorised deletion first makes a document unavailable to new retrieval and source requests. Cleanup removes its file, canonical chunks, vectors and document metadata; an operational cleanup job may remain until removal succeeds. Minimal audit records remain without document contents. A worker must check its job ownership, document existence and processing generation before publishing readiness, so a deleted document cannot be restored by late completion.

Cleanup failures remain retryable. Existing source references return source unavailable. This coordinates several stores without claiming a distributed transaction: metadata eligibility and generation checks prevent partially indexed or deleted sources from becoming visible.

### Access and operational boundaries

All active Readers, Document Managers and Administrators read the same collection. Managers and Administrators upload, retry and delete any shared document. Only Administrators manage accounts/roles and view audit records. Roles do not implement departmental or document-level visibility.

Users are informed that questions and retrieved passages may go to a hosted model provider. Routine logs exclude document text, prompts containing source text, passwords, session secrets and API keys. Record operational outcomes, timings and token usage where available. Audit entries record actor, action, target, time and outcome. Public demonstration access uses fixed public documents and Reader permissions.

## Architecture decision records

### ADR-001 — Application structure and component boundaries

**Context.** Interactive questions and file processing have different durations and failure modes. The first release is small enough for one deployable backend codebase.

**Decision.** Daniel will use a frontend, a modular backend API and a separate bounded worker process sharing application modules. The API owns access rules and orchestration; the worker owns ingestion. Storage services and the generation provider are accessed through explicit interfaces. Docker Compose will support reproducible local operation.

**Alternatives.** Processing entirely inside upload requests reduces process count but ties request availability to extraction. A microservice for every responsibility adds deployment and cross-service coordination beyond this release's needs.

**Consequences.** Processing can fail without occupying a web request indefinitely. A separate worker still requires durable job ownership and recovery; this is not a promise of distributed scalability.

**Verification.** Demonstrate that upload returns a queued result, the API remains usable during ingestion, worker failure produces recoverable/failed state, and only backend endpoints expose protected operations.

**Revisit conditions.** Sustained concurrency, independent deployment needs or measured resource contention justify changing process boundaries.

### ADR-002 — Data storage and source ownership

**Context.** Original pages, application records and similarity search must remain linked and persist across restarts.

**Decision.** Use private file storage for PDFs, SQLite for canonical records and source locations, and Qdrant provisionally for vectors. Stable document, generation and chunk identifiers join them. The index is rebuildable from canonical data. Metadata decides eligibility; vector payloads carry filtering identifiers rather than becoming a second source of truth.

**Alternatives.** An in-process vector index reduces services for 20 documents. PostgreSQL with vector support could consolidate records and vectors but changes the initial operational setup.

**Consequences.** Responsibilities are clear and retrieval experiments are easier to isolate. Multiple stores require readiness gates, compensating cleanup and reconciliation. Keep SQLite transactions short and configure concurrent API/worker access deliberately; do not assume it scales to many writers.

**Verification.** Restart services and retrieve existing documents. Inject failure between metadata/vector writes and confirm incomplete generations are hidden. Verify deletion, index rebuilding and collection-limit checks under simultaneous uploads.

**Revisit conditions.** Measured database contention, multi-instance deployment or simpler maintainability with consolidated storage.

### ADR-003 — PDF extraction and page-aware chunking

**Context.** Claims must link to exact PDF pages and highlight source passages. Text extraction can change reading order or lose layout.

**Decision.** Keep initial chunks within individual pages. Retain exact canonical text, PDF page number, page dimensions/rotation, source offsets and one or more bounding boxes. Preserve a mapping from any normalised retrieval text back to source spans. Tune chunk size/overlap experimentally. Distinguish PDF page indices from printed labels.

**Alternatives.** Cross-page chunks can preserve continuity but need more complex source mapping. Layout-aware segmentation may improve structure but adds extraction dependencies. PyMuPDF and pdfplumber are extraction candidates; selection is conditional on licence suitability and coordinate-quality tests.

**Consequences.** Page-bound chunks simplify navigation but may split an explanation across pages. Retrieve multiple passages where needed. Do not imply reliable complex-table, chart, equation or OCR support.

**Licence review — 12 November 2026.** Official PyMuPDF documentation lists AGPL and commercial licensing. A public repository alone is not treated as evidence that the chosen application distribution/deployment meets the licence terms. PyMuPDF remains conditional: adopt it only after recording a compatible licensing route, or choose an alternative. The official pdfplumber repository lists MIT licensing and documents character coordinates, making it a candidate for a permissively licensed implementation; preserve required notices and review pinned dependencies before adoption. No project licence is changed and no commercial licence is purchased by this decision.

Sources: [PyMuPDF licensing documentation](https://pymupdf.readthedocs.io/en/latest/about.html#license-and-copyright), [PyMuPDF licensing options](https://pymupdf.io/licensing), [pdfplumber repository, licence and coordinate documentation](https://github.com/jsvine/pdfplumber). This review records conditional suitability, not unconditional approval of an unselected dependency.

**Verification.** Compare extracted text and highlights against representative PDFs, including rotated pages, line wrapping and resized/zoomed viewers. Check image-only warnings and no-text failure. Record the chosen pinned library, licence/notice review and configuration before adding it to runtime dependencies.

**Revisit conditions.** Citation misalignment, poor extraction, unacceptable runtime cost or incompatible licensing.

### ADR-004 — Background processing and document lifecycle

**Context.** Retries, crashes and deletion must not create duplicate or stale searchable content.

**Decision.** Begin with a metadata-backed durable job queue and one worker. Claim jobs transactionally with an ownership token and expiry/heartbeat; bound job duration and resource use. Use processing-generation IDs and stable chunk IDs within each generation. Only the current job owner may publish a ready generation. Abandoned work is recovered or clearly failed; retries clean or replace incomplete work. A generation becomes visible only after all required writes succeed.

**Alternatives.** In-memory tasks are simpler but lose durable job state on restart. A dedicated broker and distributed workers improve scaling options but add infrastructure.

**Consequences.** The initial system remains small, with explicit recovery work. Internal deletion/cleanup markers are operational controls rather than extra user-facing processing states. Late workers cannot publish obsolete results.

**Verification.** Kill a worker mid-job, retry twice, fail vector writes, and delete during ingestion. Confirm no duplicate eligible chunks, no stuck ownership and no reappearance of deleted documents.

**Revisit conditions.** Queue delays, need for multiple machines, priority scheduling or distributed concurrency.

### ADR-005 — Authentication and role-based authorisation

**Context.** Everyone in the active shared collection can read, while document and account actions have different permissions.

**Decision.** Use backend-validated sessions and securely hashed passwords. Browser sessions use secure, HttpOnly cookies, appropriate SameSite policy and CSRF protection for state-changing requests. Check current account activity and role on each protected operation, including source viewing/downloads. Apply Administrator → Document Manager → Reader inheritance, revoke sessions on deactivation, and enforce the last-active-administrator rule transactionally. Bootstrap the first Administrator securely; no public registration.

**Alternatives.** Long-lived self-contained role tokens reduce lookups but complicate immediate revocation and changed roles. External identity/SSO reduces some account-management work but exceeds the agreed first-release integration scope.

**Consequences.** Central enforcement is straightforward to test and adds database/session access to requests. UI controls reflect permissions without being trusted to enforce them. Login rate limiting and audit records remain required.

**Verification.** Direct API tests cover each role, unauthenticated downloads, deactivation, role changes, CSRF, self-promotion and concurrent attempts to remove the last Administrator. Audit records contain outcomes without secrets.

**Revisit conditions.** SSO requirements, multiple organisations, finer document permissions or substantially different session-scale needs.

### ADR-006 — Retrieval strategy and evidence selection

**Context.** Answers require relevant passages, including product codes and facts spanning sources, within a limited generation context.

**Decision.** Establish semantic retrieval as the baseline using the same pinned local embedding configuration for ingestion and questions. Apply selected-ready-document filtering at retrieval time and canonical eligibility checks afterwards. Compare semantic search, hybrid keyword/vector search and reranking using a fixed evaluation dataset. Select the best justified configuration based on measured quality, latency and cost. Deduplicate passages and allocate a bounded evidence budget; preserve complementary and conflicting evidence where retrieved.

**Alternatives.** Keyword-only search handles exact terms but may miss paraphrases. Hybrid retrieval and reranking can improve particular cases but add configuration, computation and evaluation work. Sending every document ignores context limits and makes source selection harder to assess.

**Consequences.** A baseline makes improvements measurable. Similarity scores are ranking signals, not correctness probabilities. Retrieval failures can still omit needed evidence. Changing the embedding model or dimensions requires a compatible rebuilt index, not mixing vectors.

**Verification.** Split the 60-question dataset into development/held-out sets before tuning. Measure required-evidence recall@5, exact-code and paraphrase retrieval, multi-passage coverage, selection/deletion isolation and latency. Target recall@5 ≥85% on held-out answerable questions; disclose shortfalls.

**Revisit conditions.** Measured retrieval failures, corpus growth or a different quality/latency trade-off.

### ADR-007 — Model-provider integration and data handling

**Context.** Hosted providers differ in request formats, response schemas, latency, usage reporting and data handling.

**Decision.** Use one hosted generation provider behind a small backend adapter. Its input is the standalone question plus assigned evidence; its output normalises answer claims/evidence IDs, usage where available and error categories. Keep local embeddings separate. Configure timeouts, bounded retries for transient errors and output validation. Store API keys server-side outside source control. Disclose transmission of questions/passages and review provider terms, retention and model licence/access conditions before selection.

**Alternatives.** Direct provider calls reduce initial abstraction but spread provider behaviour across the application. Local generation provides more deployment control but introduces hardware and serving requirements. Automatic multi-provider failover complicates reproducibility and data-routing consent.

**Consequences.** The integration is replaceable, but model changes still need re-evaluation. Retries may add latency and cost. No provider, pricing, retention guarantee or model quality is assumed until selected.

**Verification.** Contract tests use a fake adapter for valid/malformed outputs, timeouts, rate limits and missing usage. Check that keys stay out of frontend assets/logs and that only required question/evidence content is sent. Record model, revision where available, prompt and generation configuration for evaluation.

**Revisit conditions.** Data-handling constraints, provider availability, cost or measured quality changes.

### ADR-008 — Grounded answers and citation integrity

**Context.** Plausible answers and valid-looking references can still be unsupported. Uploaded text may contain instructions attempting to redirect the assistant.

**Decision.** Treat documents as untrusted evidence. Give the model only backend-assigned evidence IDs and request claim-level references. Validate the response schema and evidence membership, resolve filenames/pages/coordinates from canonical metadata, and verify displayed quotations against stored source text. Reject invalid citations; remove or flag affected claims or return an appropriate failure/insufficient-evidence response rather than show unsupported clickable references. Describe retrieved conflicts with both sources. Partial answers identify missing parts.

**Alternatives.** Free-form model-generated filenames/pages are easier to display but permit fabricated locations. Whole-answer bibliography citations make individual claims harder to verify. Validating only citation syntax cannot establish semantic support.

**Consequences.** Citations are inspectable and traceable; semantic correctness remains a measured property, not a guarantee. Deterministic validation does not certify that a claim follows from a passage. Model errors remain distinguishable from genuine missing evidence. Deleted sources become unavailable.

**Verification.** Test unknown evidence IDs, incorrect quotations, stale/unselected sources, partial answers, contradictions and document prompt injection. Confirm page/highlight navigation at different zooms. Manually evaluate correctness, citation coverage and support; target ≥90% support and disclose failures. Use unsupported demonstration cases Q10–Q12 to check abstention without fabricated evidence.

**Revisit conditions.** Persistent unsupported claims, poor abstention, unreliable highlight mapping or measured evidence that an additional support-checking stage improves outcomes.

## Review and implementation handoff

These eight records cover the expanded scope of [issue #4](https://github.com/Dn1M0z/AI-Document-Assistant/issues/4). The seven planned architecture diagrams, detailed UI designs, implementation and automated tests remain their own tasks. Verification sections above specify future checks; no test pass or performance result is claimed here. The overall project status is maintained only in the main README.
