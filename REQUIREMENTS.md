# First-release requirements

**Developer:** Daniel Mozafari  
[Project scope and current project status](README.md) · [Development plan](PROJECT_PLAN.md)

## Release objective

Daniel will develop an AI document assistant with authenticated users and a role hierarchy. Users will ask questions about selected English text-based PDFs and receive grounded answers with clickable citations highlighting the supporting passages.

The release will use one shared document collection. Roles control actions, not document visibility: every active user can read the shared collection. Department-specific and per-document access restrictions are deferred.

## Roles and permissions

The hierarchy is **Administrator → Document Manager → Reader**. Higher roles inherit lower-role permissions.

| Permission | Reader | Document Manager | Administrator |
| --- | :---: | :---: | :---: |
| View available documents | Yes | Yes | Yes |
| Ask questions | Yes | Yes | Yes |
| View answers, citations, and original source pages | Yes | Yes | Yes |
| Upload documents | No | Yes | Yes |
| Retry failed processing | No | Yes | Yes |
| Delete documents | No | Yes | Yes |
| Create and deactivate accounts | No | No | Yes |
| Assign or change roles | No | No | Yes |
| View document-management and account audit records | No | No | Yes |

A Document Manager may delete any document in the shared collection, not just their own uploads. There will be no public registration. An administrator will create accounts and assign roles.

## Functional requirements

| ID | Requirement | Completion condition |
| --- | --- | --- |
| FR-01 | Upload documents | Authorised users can upload one or more supported PDFs. Invalid, password-protected, corrupted, or oversized files receive a clear explanation. |
| FR-02 | Show processing status | Each document displays queued, processing, ready, or failed. Failed documents are not searchable. |
| FR-03 | Extract and index text | Extracted text retains document identity, PDF page numbers, and source positions. |
| FR-04 | Manage the library | Active users can view filenames, page counts, uploaders, and processing status. Documents persist after restart. |
| FR-05 | Select sources | Users select one or more ready documents. Retrieval searches only that selection. No selection prompts the user to select documents. |
| FR-06 | Ask questions | Users submit standalone questions and see a loading state. Empty and over-limit questions are rejected. |
| FR-07 | Retrieve evidence | Relevant passages can be found even when question wording differs from source wording. |
| FR-08 | Generate grounded answers | Answers use retrieved document evidence without deliberately supplementing missing information with model knowledge or web searches. |
| FR-09 | Cite claims | Document-based factual claims have references showing filename, PDF page, and supporting text. |
| FR-10 | Highlight evidence | Clicking a citation opens the correct page and highlights the supporting passage. Highlights remain aligned when zooming or resizing. |
| FR-11 | Handle insufficient evidence | Responses explain when the selected documents provide insufficient evidence. Partial answers identify unanswered parts. |
| FR-12 | Expose disagreements | Relevant conflicting retrieved passages are described and both sources cited instead of silently selecting one. |
| FR-13 | Delete documents | Only Document Managers and Administrators can delete documents. Files, metadata, and indexed content are removed; future retrieval excludes them. Existing references show source unavailable after deletion. |
| FR-14 | Handle failures | Extraction failures, model timeouts, and unavailable services produce useful messages. Technical failures are not presented as missing evidence. |
| FR-15 | Authenticate users | Login, logout, secure password hashing, authenticated sessions, and secure first-administrator setup are provided. |
| FR-16 | Administer accounts | Only Administrators can create/deactivate accounts and assign roles. Users cannot promote themselves. The last active administrator cannot be deactivated or demoted. |
| FR-17 | Enforce permissions | The backend authorises every protected operation, including source viewing and downloads. Deactivation revokes access and role changes apply on subsequent requests. The UI reflects permissions but is not the security boundary. |
| FR-18 | Record administrative actions | Uploads, deletions, account changes, and role changes produce administrator-visible audit records containing actor, action, target, time, and outcome without passwords or document contents. |

## Document and usage boundaries

| Area | First-release decision |
| --- | --- |
| File format and language | English text-based PDFs and English questions |
| Supported content | Ordinary text paragraphs and headings; no promise of reliable complex-table, chart, or equation interpretation |
| File size | Maximum 10 MB per PDF, defined as 10,000,000 bytes |
| Page count | Maximum 100 pages per PDF |
| Collection size | Maximum 20 stored documents across the shared collection |
| Upload behaviour | Multiple files may be selected; each is separately validated and processed |
| Question length | Maximum 2,000 characters |
| Conversation | Independent questions; previous answers are not used as context |
| Document updates | Delete and upload again; no version-management interface |
| Public demo | Fixed public documents and Reader access; visitor upload and deletion disabled |

Limits will be configurable, enforced by the backend, and displayed in the interface. They are scope controls, not measured capacity claims.

Scanned or mixed-content PDFs will carry a clear explanation that image-only content is not searchable. A document with no usable extracted text will not be marked ready. Shared demonstration credentials must never grant Document Manager or Administrator permissions.

## Answer and citation rules

- References must point to evidence retrieved for that question.
- Filenames, page numbers, and source locations come from stored metadata, not model guesses.
- Displayed source quotations match extracted source text.
- Page references are labelled **PDF page** to distinguish them from printed page numbers.
- Unsupported evidence IDs never appear as valid clickable citations.
- Retrieval similarity is not presented as a probability that an answer is correct.
- Document text is evidence, including when it contains instructions attempting to redirect the assistant.

A valid citation does not prove that its passage supports the claim. Semantic support will be evaluated separately.

## Quality and engineering requirements

| ID | Requirement | Completion condition |
| --- | --- | --- |
| QR-01 | Usability | Keyboard navigation, visible focus states, and a usable smaller-screen layout support the main workflow. |
| QR-02 | Processing reliability | Retrying does not duplicate searchable chunks. Interrupted processing is recoverable or clearly marked failed. |
| QR-03 | Credentials | API keys remain server-side and outside version control; passwords are securely hashed. |
| QR-04 | Data handling | Users are told questions and retrieved passages may be sent to the model provider. Routine logs exclude document text and credentials. |
| QR-05 | Repeatable setup | Documented configuration and Docker Compose allow another developer to run the application. |
| QR-06 | Automated checks | Continuous integration runs relevant tests and code-quality checks. |
| QR-07 | Observability | Processing failures, retrieval/generation latency, and token usage where available are recorded. |
| QR-08 | Reproducible evaluation | Dataset, models, prompts, chunking configuration, and retrieval settings are recorded. |
| QR-09 | Account protection | Login rate limiting, session revocation, and server-side permission checks are verified. |

## Essential first-release checklist

Checkboxes track implementation and verification of individual requirements, not the overall project status.

### Accounts and permissions

- [ ] Login and logout with secure password hashing and authenticated sessions.
- [ ] Secure setup of the first administrator.
- [ ] Administrator, Document Manager, and Reader roles.
- [ ] Administrator-controlled account creation, deactivation, and role assignment.
- [ ] Backend permission checks on every protected operation.
- [ ] Interface hides or disables unavailable actions and explains denied access.
- [ ] Prevent self-promotion and removal or demotion of the last active administrator.
- [ ] Deactivation revokes access; role changes apply on subsequent requests.

### Documents

- [ ] Authorised English text-based PDF uploads, validation, and clear errors.
- [ ] Queued, processing, ready, and failed states.
- [ ] Authorised retry of failed processing without duplicate indexing.
- [ ] Document list with filename, page count, uploader, and processing status.
- [ ] Persistent storage and source-location metadata.
- [ ] Authorised deletion of files, metadata, and searchable content.

### Questions, answers, and evidence

- [ ] Select one or more ready documents.
- [ ] Ask standalone questions with clear loading and failure states.
- [ ] Retrieve only from selected documents.
- [ ] Generate grounded answers and explain missing evidence.
- [ ] Describe relevant retrieved conflicts and cite both sources.
- [ ] Claim-level citations with filename, PDF page, and source text.
- [ ] Validate evidence IDs and quotations.
- [ ] Open correct PDF pages and highlight supporting passages.

### Usability, security, and accountability

- [ ] Empty/error states, keyboard navigation, visible focus, and responsive layout.
- [ ] Server-side API keys and login rate limiting.
- [ ] Protected document downloads and source-viewer endpoints.
- [ ] Administrator-visible audit records for document and account changes.
- [ ] Records include actor, action, target, time, and outcome without sensitive contents.
- [ ] Explain model-provider data transmission.

### Testing and evaluation

- [ ] Unit and integration tests.
- [ ] Direct API tests prove Readers cannot upload, retry, or delete.
- [ ] Direct API tests prove Document Managers cannot manage accounts or roles.
- [ ] Test unauthenticated access, deactivation, changed roles, and last-administrator protection.
- [ ] End-to-end authorised upload → question → citation test.
- [ ] Verify selected-document isolation and deleted-document exclusion.
- [ ] Failure and document prompt-injection tests.
- [ ] Evaluation dataset and retrieval comparison report.
- [ ] Latency, cost, answer-quality, and citation-support measurements.

### Portfolio delivery

- [ ] Seven architecture diagrams and architecture decision records.
- [ ] UI designs and documented workflows, including role-specific actions.
- [ ] Reproducible setup and continuous integration.
- [ ] Failure-analysis report.
- [ ] Demonstration video.

## Later-additions checklist

### Advanced access control

- [ ] Permissions for individual documents or collections.
- [ ] Department/team workspaces.
- [ ] Custom roles and permissions.
- [ ] Multiple isolated organisations.

### Account convenience

- [ ] Public self-registration.
- [ ] Email invitations and password-reset emails.
- [ ] Single sign-on.
- [ ] Multi-factor authentication.

### Document capabilities

- [ ] OCR for scans.
- [ ] Word, Excel, and other formats.
- [ ] Reliable complex-table, chart, and image interpretation.
- [ ] Document version history and approval workflows.

### Assistant capabilities

- [ ] Multi-turn memory and saved conversations.
- [ ] Answer exports, feedback, and ratings.
- [ ] Multilingual support.
- [ ] Web browsing or external knowledge sources.
- [ ] Model training or fine-tuning.

### Operations

- [ ] Advanced analytics dashboards.
- [ ] Audit-log export and filtering.
- [ ] Large-scale distributed processing.
- [ ] Configurable per-user quotas.

## Release evaluation

Daniel will prepare 60 manually checked questions, separating development and held-out test sets before tuning.

| Measure | Release expectation |
| --- | --- |
| Evidence recall@5 | Target at least 85% on held-out answerable questions, measuring required evidence retrieved |
| Citation support | Target at least 90% of cited claims supported by their cited passages |
| Citation navigation | Every citation in the release test suite opens the correct available source location |
| Document filtering | No unselected or deleted documents retrieved during isolation tests |
| Answer correctness and citation coverage | Publish measured results and failure examples |
| Insufficient-evidence handling | Report correct abstentions and incorrect refusals separately |
| Performance | Publish median and p95 latency under a named configuration |
| Cost | Estimate generation cost from recorded usage and documented pricing assumptions |

Targets are goals to test, not guaranteed accuracy. Any unmet target will be disclosed. A permission failure in the release tests must be fixed before releasing authenticated functionality.

The evaluation report will compare semantic search, hybrid search, and reranking. The application will use the best justified configuration; an experimental configuration switch is not required in the user interface.

## Definition of done

The authorised upload → select → question → answer → highlighted evidence workflow works; role restrictions and essential requirements are verified; evaluation results and limitations are published; and another developer can run the application.

A hosted demo is desirable but will not block a release with a working local setup and demonstration video. Overall project status is maintained only in the README.
