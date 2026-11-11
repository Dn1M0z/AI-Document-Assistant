# AI Document Assistant — Project Plan

**Developer:** Daniel Mozafari  
**Status:** Planned; implementation tasks have not yet been completed.  
**Related document:** [Project scope](README.md)

## Estimate and assumptions

Daniel will implement the project using existing models, with guidance where needed. The plan allocates **79 focused working hours**, plus **8–16 hours for unexpected integration fixes**, for a practical total of **87–95 hours**. At approximately **15 hours per week**, the project should take **6–7 weeks**.

A basic upload → question → answer prototype is expected within the first **25–35 hours of implementation work**. Accurate source highlighting, evaluation, and reliable error handling will require additional work.

The first release assumes English text-based PDFs, one user, no model training, and no OCR. Task estimates include routine debugging. Additional learning time will be recorded separately, and estimates will be revised using actual progress.

## 1. Requirements and project organisation — 3 hours

| Task | Deliverable / completion condition | Time |
| --- | --- | ---: |
| Finalise first-release requirements | A checklist separating essential features from later additions | 45 min |
| Write user stories and acceptance criteria | Upload, select documents, ask questions, inspect citations, delete documents, and handle missing evidence | 1 hr |
| Select demonstration documents | 5–10 permitted technical PDFs with varied layouts and useful questions | 45 min |
| Set up the task board | GitHub issues grouped into milestones, with clear completion criteria | 30 min |

Daniel will use the acceptance criteria to decide whether a feature is finished and prevent uncontrolled expansion of the initial scope.

## 2. Software architecture and diagrams — 8 hours

Daniel will use Mermaid diagrams committed to the repository wherever practical so that designs can be reviewed and updated alongside the code.

| Task / diagram | What it will show | Time |
| --- | --- | ---: |
| Architecture decisions | Responsibilities of the frontend, backend, processing worker, metadata database, vector store, and model provider | 1 hr |
| C4 system context diagram | User, document assistant, external model provider, and system boundary | 30 min |
| C4 container diagram | React frontend, Python API, ingestion worker, document storage, SQLite, Qdrant, and their connections | 45 min |
| Document ingestion activity diagram | Validate → extract → chunk → embed → index, including failures and retries | 45 min |
| Question-answering sequence diagram | User request, document filtering, retrieval, generation, citation validation, and response | 1 hr |
| Entity relationship diagram | Documents, processing jobs, chunks, source spans, and evaluation records; distinguish database records from vector-store payloads | 1 hr |
| Document state diagram | Queued, processing, ready, failed, and deleting, with permitted transitions | 30 min |
| Deployment diagram | Application services, persistent volumes, secrets, and external API | 30 min |
| API contract | Request/response formats, errors, processing status, and structured citation objects | 1 hr |
| Architecture decision records | Three short explanations covering retrieval storage, page-aware chunking, and model-provider abstraction | 1 hr |

These seven diagrams will cover the initial system. A class diagram will only be added if the implementation develops a complex object model that benefits from one.

The ingestion worker will use a simple, bounded implementation. Distributed infrastructure is outside the initial scope.

## 3. User workflow and UI design — 6 hours

| Task | Deliverable / completion condition | Time |
| --- | --- | ---: |
| Map the user workflow | Upload → processing → select documents → ask → read answer → inspect evidence | 45 min |
| Create low-fidelity wireframes | Document library, question-and-answer workspace, and source viewer | 1 hr |
| Define visual styles | Colours, typography, spacing, buttons, inputs, and status indicators | 45 min |
| Create polished desktop mockups | A consistent layout showing the complete primary interaction | 1.5 hr |
| Design failure and empty states | No documents, failed processing, no evidence, unavailable model, and deleted source | 1 hr |
| Plan responsive and accessible behaviour | Smaller-screen layout, keyboard navigation, focus states, and readable contrast | 1 hr |

The main workspace will contain a document selector, an answer area, and a source panel. Selecting a citation will reveal the supporting evidence highlighted in the original PDF.

Mockups will include realistic sample content so that layout problems can be identified before implementation.

## 4. Repository and development setup — 3 hours

| Task | Deliverable / completion condition | Time |
| --- | --- | ---: |
| Create the project structure | Frontend, backend, tests, documentation, and evaluation directories | 45 min |
| Configure dependencies and environment settings | Reproducible dependency files, an environment-variable example, and an appropriate gitignore file | 45 min |
| Configure formatting and basic checks | Consistent Python and TypeScript formatting and linting | 30 min |
| Add initial continuous integration | GitHub Actions runs available checks on pushes and pull requests | 1 hr |

The scope README already exists and will be maintained rather than recreated.

## 5. Backend and AI programming — 23 hours

| Task | Deliverable / completion condition | Time |
| --- | --- | ---: |
| Build the API foundation | Configuration, health endpoint, and consistent error responses | 1 hr |
| Implement upload validation and storage | Supported PDFs accepted; invalid or oversized files rejected | 2 hr |
| Implement document records and processing jobs | Uploads receive IDs; processing status survives API requests | 2 hr |
| Extract PDF text and source positions | Text remains linked to its page and bounding boxes | 3 hr |
| Implement page-aware chunking | Chunks preserve source mappings and configurable overlap | 2 hr |
| Generate and store embeddings | Documents become searchable without duplicate indexing on retry | 2 hr |
| Implement semantic retrieval | Relevant passages returned only from selected, ready documents | 2 hr |
| Integrate answer generation | The model receives bounded evidence and returns structured claims and citations | 2 hr |
| Validate citation references | Unknown evidence IDs and invalid quotations are rejected or handled explicitly | 2 hr |
| Handle missing and conflicting evidence | Responses acknowledge insufficient evidence and expose relevant disagreements | 2 hr |
| Implement document deletion | Metadata, stored files, and vector entries are removed consistently | 1 hr |
| Add operational logging | Processing failures, retrieval time, model time, token usage, and request IDs are recorded | 2 hr |

These estimates include small checks as each component is built. The testing phase will cover cross-component behaviour and regression cases.

Source mapping is the most uncertain backend task. Daniel will start with ordinary text PDFs and verify extraction on several layouts before building the highlighting interface.

## 6. Frontend programming — 13 hours

| Task | Deliverable / completion condition | Time |
| --- | --- | ---: |
| Build the application layout | Main workspace and reusable UI components | 1.5 hr |
| Build document upload and library | Upload, status display, selection, and deletion | 2 hr |
| Build the question-and-answer interface | Question submission, answer display, and pending/error states | 2 hr |
| Build citation components | Clickable references with document, page, and supporting passage | 1.5 hr |
| Integrate the PDF viewer and highlights | A citation opens the correct page and highlights its source region | 4 hr |
| Finish responsive and accessibility behaviour | Keyboard operation, focus visibility, and readable small-screen layout | 2 hr |

Highlights will remain aligned when the PDF is zoomed or resized. This behaviour will be explicitly tested.

## 7. Testing, evaluation, and retrieval experiments — 15 hours

| Task | Deliverable / completion condition | Time |
| --- | --- | ---: |
| Write meaningful unit tests | Chunk boundaries, source metadata, citation validation, and file validation | 2 hr |
| Write integration tests | Upload → processing → retrieval → answer; document filtering and deletion | 2 hr |
| Test the main browser journey | Upload, question, citation navigation, and highlighted evidence | 1.5 hr |
| Test failure and adversarial cases | Broken PDFs, model failures, unanswerable questions, and instructions embedded in documents | 1.5 hr |
| Build the evaluation dataset | 60 manually checked questions with expected answers and evidence; separate development and test sets | 4 hr |
| Implement the evaluation runner | Repeatable retrieval metrics, answer-review records, latency, and cost reporting | 1.5 hr |
| Add hybrid retrieval and reranking experiments | Compare the baseline with two alternative retrieval configurations | 2.5 hr |

Daniel will start earlier with 10–15 evaluation questions, then expand the dataset during this phase. The initial target is 60 checked questions; expansion towards 100 will be considered after the initial release.

### Evaluation outputs

| Result | What it will demonstrate |
| --- | --- |
| Evidence recall@5 | Whether retrieval finds the necessary source material |
| Citation support and coverage | Whether answer claims are traceable and actually supported |
| Answer correctness | Whether the final response answers the question accurately |
| Abstention behaviour | Whether the application recognises missing evidence |
| Median and p95 latency | Typical performance and slower requests |
| Cost per question | The practical cost of the chosen configuration |

The report will compare semantic retrieval, hybrid retrieval, and hybrid retrieval with reranking. It will include cases where added complexity does not improve performance. The held-out test set will remain separate from tuning decisions.

## 8. Deployment and portfolio presentation — 8 hours

| Task | Deliverable / completion condition | Time |
| --- | --- | ---: |
| Finish Docker Compose setup | A documented command starts the application and required services | 1.5 hr |
| Prepare a demonstration deployment | A working demo using public sample documents and bounded usage | 2 hr |
| Write setup and usage documentation | Another developer can configure and run the application | 1 hr |
| Write the evaluation and failure report | Results table, failure examples, limitations, and lessons learned | 1.5 hr |
| Record a demonstration video | A 2–3 minute walkthrough showing answers, citations, and missing-evidence handling | 1 hr |
| Prepare the release | Screenshots, updated status, release tag, and concise project description | 1 hr |

A hosted demo will depend on available hosting and budget. If hosting becomes a distraction, a reliable local setup and demonstration video will provide reviewable evidence for the initial release.

## Time budget

| Category | Focused time |
| --- | ---: |
| Requirements and organisation | 3 hr |
| Architecture and diagrams | 8 hr |
| Workflow and UI design | 6 hr |
| Development setup | 3 hr |
| Backend and AI programming | 23 hr |
| Frontend programming | 13 hr |
| Testing and evaluation | 15 hr |
| Deployment and portfolio presentation | 8 hr |
| **Planned work** | **79 hr** |
| Unexpected integration fixes | **8–16 hr** |
| **Practical total** | **87–95 hr** |

These are planning estimates, not strict deadlines. Extra learning time for unfamiliar embeddings, PDF coordinates, or deployment tooling will be recorded separately.

## Suggested schedule at approximately 15 hours per week

| Week | Focus | Expected outcome |
| --- | --- | --- |
| 1 | Requirements, architecture, and initial UI design | Clear design and development plan |
| 2 | Finish design/setup; implement ingestion and retrieval | PDFs can be processed and searched |
| 3 | Answer generation, citations, and initial frontend | First usable question-answering prototype |
| 4 | Complete frontend and source highlighting | Complete user journey |
| 5 | Testing, evaluation dataset, and retrieval comparisons | Measured quality and documented weaknesses |
| 6 | Remaining fixes, deployment, documentation, and video | Portfolio release |
| Small overflow if needed | Remaining integration issues | Approximately 6–7 weeks overall |

The schedule is indicative: tasks can overlap, and early tests and evaluation questions will be developed alongside implementation.

## Employer-facing evidence

The release will demonstrate more than a working interface:

- Architecture decision records will explain engineering trade-offs.
- Reproducible retrieval experiments will provide evidence for technical choices.
- A failure-analysis report will show how weaknesses were investigated.
- Continuous integration will demonstrate repeatable quality checks.
- Precise evidence highlighting will make answers inspectable.
- Latency and cost reporting will show practical awareness of operating an AI system.

## First action

Daniel will begin with the acceptance criteria and a 10-question sample dataset. These will establish what the application must achieve before the architecture and code are developed.

The task board, diagrams, mockups, code, tests, deployment, and reports described above are planned deliverables; this document does not indicate that they already exist.
