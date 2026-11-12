# AI Document Assistant

**Developer:** Daniel Mozafari  
## Project status

**Phase:** Planning baseline and milestone-based task tracking established; sample-answer annotations and architecture design are next.  
**Last updated:** 12 November 2026

- **Completed:** Project scope, development plan, first-release requirements, role hierarchy, essential/later feature checklists, and [16 approved user stories with acceptance criteria](USER_STORIES.md).
- **Demonstration documents:** Ten approved clothing-company demonstration PDFs added in [demonstration-documents](demonstration-documents/) (four public-facing, three internal, three mixed), covering 30 product families, 12 employee records, six suppliers, and expanded inventory. Documents range from one to six pages, within the 100-page limit. The folder includes [company background and an AI-generated document disclosure](demonstration-documents/COMPANY_BACKGROUND.md).
- **Demonstration questions:** [12 questions with IDs, test purposes and target audiences](demonstration-documents/DEMONSTRATION_QUESTIONS.md) added: nine document-supported questions and three unsupported cases. Audience labels are scenario guidance, not changes to first-release access rules.
- **Task tracking:** [48 GitHub issues](https://github.com/Dn1M0z/AI-Document-Assistant/issues) grouped into [seven milestones](https://github.com/Dn1M0z/AI-Document-Assistant/milestones), with completion checklists, estimates, dependencies and an assignee. The approved planning baseline and task-board setup are complete; implementation tasks remain open. Calendar due dates are unset pending an agreed start date.
- **Next:** [Annotate the initial 12 questions with expected answers and source pages](https://github.com/Dn1M0z/AI-Document-Assistant/issues/3), then work through the architecture and UI-design tasks in milestone M0.
- **Implementation:** Application programming has not started.

This section is the sole source of overall project status. It must be updated whenever the phase or milestone status changes, including completed work, the next step, and the last-updated date. Other documents link here rather than duplicate the status.

## First-release requirements

See [REQUIREMENTS.md](REQUIREMENTS.md) for the combined functional requirements, Administrator / Document Manager / Reader permission matrix, limits, quality targets, and essential-versus-later checklists.

## Development plan

See the [detailed project plan](PROJECT_PLAN.md) for task-by-task estimates, architecture diagrams, UI design, programming, testing, evaluation, and release milestones. The plan estimates **91–97 focused hours**, plus **8–16 hours of contingency** (**99–113 hours overall**, or approximately **7–8 weeks at 15 hours per week**).

## Project scope

### Overview

Daniel will develop an AI document assistant that allows a user to upload documents, ask questions about their contents, and receive evidence-based answers with precise, clickable source citations.

The project will use retrieval-augmented generation (RAG). The system will retrieve relevant passages from the selected documents and provide those passages to a language model to generate an answer. Uploading documents will index their contents for retrieval; it will not retrain the language model.

Daniel will use this project to demonstrate applied AI engineering, information retrieval, backend development, interface design, system architecture and systematic evaluation.

### Objectives

Daniel will:

- Build an end-to-end document ingestion and question-answering pipeline.
- Preserve source locations so that users can verify individual claims.
- Implement appropriate responses when evidence is missing or conflicting.
- Compare retrieval approaches using a reproducible evaluation dataset.
- Measure answer quality, citation support, latency, and generation cost.
- Document implementation decisions, limitations, and observed failures.

### Initial scope

The first release will support authenticated users with Administrator, Document Manager, and Reader roles, English text-based PDFs, and single-turn questions. All active users share one document collection; only Document Managers and Administrators may upload, retry processing, or delete documents. Only Administrators manage accounts and roles. A demonstration collection will use publicly available technical documentation, with robotics manuals as a potential focus. Daniel will check redistribution permissions before including documents in the repository.

| Feature | Planned behaviour |
| --- | --- |
| PDF upload | A Document Manager or Administrator will upload one or more PDFs within documented file-size and page-count limits. |
| Processing status | The system will display queued, processing, ready, or failed status, with useful error messages. |
| Document management | Active users can list documents; only Document Managers and Administrators can delete documents and indexed content. |
| Accounts and roles | Administrators manage accounts and roles; the backend enforces permissions, session revocation, and protected source access. |
| Audit records | Administrators can view records of document-management and account changes. |
| Document selection | The user will ask questions against one document or a selected collection. |
| Semantic retrieval | The system will retrieve relevant passages even when the question uses different wording. |
| Grounded answers | The system will generate answers using retrieved evidence from the selected documents. |
| Claim-level citations | Factual claims will link to supporting passages, identifying the document and PDF page. |
| Source viewer | Selecting a citation will open the relevant PDF page and highlight the supporting passage. |
| Insufficient evidence | The system will acknowledge when the available documents do not support an answer. |
| Conflicting evidence | The system will identify relevant disagreements and cite both sources. |

### Illustrative user journey

1. A Document Manager or Administrator logs in and uploads a technical manual.
2. The system extracts and indexes its contents.
3. An authenticated user, including a Reader, selects the manual and asks a question.
4. The system retrieves relevant passages and generates an answer.
5. The user selects a citation to inspect the exact supporting text in the original PDF.

For example, a question about sensor calibration could receive an answer citing a maintenance interval on PDF page 14. This is an illustrative interaction, not a claim about an existing manual or implemented feature.

## System design

### Document ingestion

The system will validate a PDF, extract its text and page coordinates, split the text into chunks, generate embeddings, and store the chunks with their source metadata.

Each chunk will retain:

- Document ID and document version or file hash.
- Unique chunk ID and extracted text.
- PDF page number.
- Source text offsets and bounding boxes for highlighting.
- Section heading, where reliably available.

Initial chunks will remain within individual pages to simplify accurate source mapping. Daniel will evaluate chunk size and overlap rather than assume a single configuration is optimal. PDF page numbers will be distinguished from printed page labels where those differ.

Documents that require OCR or cannot be extracted reliably will receive a clear unsupported-document or processing-error response.

### Question answering

The system will:

1. Receive a question and the selected document IDs.
2. Retrieve candidate passages restricted to those documents.
3. Rank the passages and assemble a bounded evidence context.
4. Ask the language model to generate an answer with structured evidence references.
5. Validate citation IDs and resolve source metadata.
6. Return the answer, citations, and supporting passages.

Selected-document filtering will apply during retrieval. Deleted documents will be excluded from subsequent retrieval and their indexed content removed.

### Citation integrity

Daniel will implement citations as structured data rather than allow the model to invent filenames or page numbers.

The system will assign evidence IDs to retrieved passages. The model will associate answer claims with those IDs, and the backend will check that each reference belongs to the retrieved evidence. Document names, page numbers, and highlight coordinates will come from stored metadata.

Any displayed quotation will be checked against the stored source text. An existing citation ID will not be treated as proof that the passage supports a claim; semantic support will be evaluated separately. Unsupported claims will be removed, flagged, or replaced with an insufficient-evidence response.

The interface will not present retrieval similarity as a probability that an answer is correct.

## Proposed technology stack

These choices are provisional and will be recorded with exact versions during implementation.

| Component | Proposed technology | Responsibility |
| --- | --- | --- |
| Frontend | React and TypeScript | Uploads, document selection, questions, answers, and source panel. |
| Backend | Python and FastAPI | Ingestion and question-answering APIs. |
| PDF extraction | PyMuPDF | Text extraction and source coordinates; licence suitability will be reviewed before adoption or distribution. |
| PDF viewer | PDF.js | Page rendering and evidence highlights. |
| Embeddings | A pretrained Sentence Transformers model | Local text embedding generation. |
| Vector search | Qdrant | Embedding storage and retrieval. |
| Metadata | SQLite initially | Users, roles, sessions, audit records, documents, processing status, and experiment records. |
| Answer generation | One hosted language-model API | Answer generation from retrieved evidence. |
| Testing | pytest | Pipeline, API, and failure-case verification. |
| Delivery | Docker Compose and GitHub Actions | Reproducible local setup and automated checks. |

Daniel will keep model access behind a small interface so providers can be changed without rewriting the pipeline. API keys will remain server-side and outside version control.

## Retrieval experiments

Daniel will first implement semantic retrieval as a baseline, then compare it against hybrid retrieval and reranking.

| Configuration | Evaluation question |
| --- | --- |
| Semantic retrieval | How effectively does meaning-based search retrieve supporting evidence? |
| Semantic and keyword retrieval | Does exact-term matching improve retrieval for identifiers, model numbers, and technical vocabulary? |
| Hybrid retrieval with reranking | Does reordering candidate passages improve quality enough to justify added latency and complexity? |

Daniel will hold the evaluation dataset constant when comparing configurations and record chunking settings, model versions, retrieval depth, prompts, and generation settings. Improvements will be reported only when supported by measured results.

Qdrant's [hybrid search and reranking documentation](https://qdrant.tech/documentation/tutorials-basics/reranking-hybrid-search/) will inform implementation.

## Evaluation plan

Daniel will create 60 manually checked questions for the first release with expected answers and supporting source passages where applicable. Development and held-out test questions will be separated before tuning.

The dataset will include:

- Direct factual lookups.
- Paraphrased questions.
- Exact technical terms and identifiers.
- Questions requiring multiple passages.
- Conflicting sources.
- Questions that cannot be answered from the selected documents.
- Documents containing instructions that attempt to redirect the assistant.

| Measure | Purpose |
| --- | --- |
| Evidence recall@5 | The proportion of required evidence units represented in the top five retrieved chunks. |
| Answer correctness | Whether answers match the expected information. |
| Citation support | Whether each cited passage supports its associated claim. |
| Citation coverage | Whether factual answer claims have supporting citations. |
| Abstention quality | Whether unanswerable questions are declined without excessive rejection of answerable questions. |
| Latency and cost | Median and slow-response latency, token usage, and generation cost under a documented configuration. |

Where evidence spans several passages, evaluation will track all required evidence units rather than count retrieval of one passage as complete success.

Automated metrics from tools such as [Ragas](https://docs.ragas.io/en/latest/concepts/metrics/available_metrics/) may supplement manual review. Model-based scores will not replace human checks of citation support.

### Provisional acceptance targets

These are proposed targets, not achieved results.

- Evidence recall@5 of at least 85% on the held-out set.
- At least 90% of cited claims supported by their cited passages.
- All returned citation links resolve to an existing source location.
- Deleted and unselected documents do not appear in subsequent retrieval results.
- Extraction failures produce useful errors rather than misleading answers.
- Latency and cost are reported with the model, hardware, dataset size, and run configuration.
- Unsupported-question handling is evaluated alongside answerable-question performance.
- Uploaded document text is treated as evidence, not as instructions that override application behaviour.

## Implementation roadmap

The planning estimate is seven to eight weeks at approximately 15 hours per week, including accounts and role-based permissions. The estimate will be revised as implementation progresses.

| Milestone | Planned deliverable |
| --- | --- |
| 1. Access foundation, dataset and ingestion | Accounts, roles, protected operations, demonstration documents, extraction, source metadata, and processing status. |
| 2. Retrieval baseline | Chunking, embeddings, vector storage, and inspectable retrieval results. |
| 3. Grounded answers | Question-answering endpoint, structured citations, and insufficient-evidence handling. |
| 4. User interface | Uploads, document selection, answer panel, and source highlights. |
| 5. Evaluation and improvements | Benchmark dataset, retrieval comparisons, reranking experiment, and failure analysis. |
| 6. Portfolio release | Reproducible setup, automated checks, demonstration video, and evaluation report. |

## Out of scope for the first release

- OCR for scanned PDFs and images.
- Reliable extraction of complex tables, charts, and equations.
- Multilingual document support.
- Multi-turn conversational memory.
- Language-model fine-tuning or training from scratch.
- Team workspaces, per-document permissions, custom roles, and multiple isolated organisations.
- Public registration, email invitations/password resets, single sign-on, and multi-factor authentication.
- Autonomous actions beyond document retrieval and answer generation.

These capabilities may be considered after the core pipeline and evaluation are reliable. Any public demo will use a fixed collection of public documents and Reader access. Visitors cannot upload or delete documents, and shared demo credentials must not grant elevated roles.

## Risks and limitations

PDF layouts may reduce extraction quality. Relevant evidence may be missed during retrieval. Generated answers may contain unsupported claims despite instructions to use evidence. Precise citations will make answers inspectable but will not, by themselves, guarantee correctness.

Daniel will document these limitations, maintain representative failure cases, and measure changes against the evaluation dataset. Private documents, API keys, and sensitive logs will not be committed to the public repository.

## Portfolio deliverables

The completed project is intended to provide:

- A working application with a short demonstration video.
- Reproducible installation and configuration instructions.
- An architecture explanation and documented engineering decisions.
- An evaluation dataset with source annotations, subject to redistribution rights.
- A results table comparing retrieval configurations.
- A failure analysis describing weaknesses and improvements.
- Automated checks for important pipeline behaviour.
- A clear statement of supported functionality and limitations.

The project will demonstrate Daniel's ability to build and evaluate a document assistant whose answers can be checked against their source evidence.
