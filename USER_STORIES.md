# User stories and acceptance criteria

[First-release requirements](REQUIREMENTS.md) · [Project scope and current project status](README.md)

Each story links to the agreed requirements. Acceptance criteria describe observable behaviour that can later become tests. Administrator permissions include Document Manager permissions, and both include Reader permissions.

## US-01 — Sign in and sign out

*Requirements: FR-15, FR-17*

As a user, I want to sign in securely so that I can access the document assistant with my assigned permissions.

- **Given** an active account and valid credentials, **when** the user signs in, **then** access is granted with their current role.
- **Given** invalid credentials or a deactivated account, **when** sign-in is attempted, **then** access is denied without revealing whether the account exists.
- **Given** repeated failed attempts, **when** the configured login rate limit is reached, **then** further attempts are temporarily restricted.
- **Given** an authenticated session, **when** the user signs out, **then** that session can no longer access protected operations.
- **Given** no valid session, **when** a protected API or document URL is requested directly, **then** access is denied.

## US-02 — Create user accounts

*Requirements: FR-16*

As an Administrator, I want to create accounts and assign roles so that users receive appropriate access.

- An Administrator can create an account with a unique login identifier and one of the three supported roles.
- Missing required information, duplicate identifiers, and invalid roles are rejected with clear messages.
- Passwords are stored as secure hashes and are never returned through account-listing responses.
- A Reader or Document Manager cannot create accounts through either the interface or direct API requests.
- Successful account creation produces an audit record without recording the password.

## US-03 — Change roles and deactivate accounts

*Requirements: FR-16, FR-17*

As an Administrator, I want to change roles and deactivate accounts so that access remains appropriate.

- A role change takes effect on the affected user's next protected request, including requests from an existing session.
- Deactivating an account prevents new sign-ins and further protected requests from existing sessions.
- Readers and Document Managers cannot change their own or another user's role.
- Invalid role values are rejected.
- Deactivating or demoting the last active Administrator is rejected.
- Account changes record the acting Administrator, affected account, action, time, and outcome.

## US-04 — Set up the first Administrator

*Requirement: FR-15*

As the person installing the application, I want a secure way to create the first Administrator so that account management can begin.

- A documented setup procedure creates the first Administrator without public self-registration.
- Setup requires credentials supplied by the installer; no shared default password is provided.
- Passwords are not written to application logs or committed configuration files.
- Once initial setup is complete, repeating the bootstrap procedure cannot overwrite the existing Administrator or create an unauthorised privileged account.

## US-05 — Upload documents

*Requirements: FR-01, FR-17*

As a Document Manager, I want to upload PDFs so that users can ask questions about their contents.

- A Document Manager or Administrator can select and upload multiple files.
- Each file is validated separately; rejection of one file does not prevent other valid files from being processed.
- Valid text-based PDFs within the configured limits enter the processing queue.
- Unsupported, corrupted, password-protected, and oversized files receive clear rejection messages.
- With the agreed defaults, files exceeding **10,000,000 bytes** or **100 pages** are rejected.
- An upload cannot increase the shared collection beyond **20 stored documents**, including when uploads occur concurrently.
- Readers cannot upload through the interface or by sending a direct API request.
- Successful uploads record the uploader and produce an audit record.

## US-06 — Monitor processing and retry failures

*Requirements: FR-02, FR-03; QR-02*

As a Document Manager, I want to see processing results and retry failures so that documents can become searchable.

- Documents show queued, processing, ready, or failed status.
- A document becomes ready only after its usable text, source positions, and searchable chunks have been stored successfully.
- Failed documents are excluded from question answering and display a useful failure reason.
- A PDF with no usable extracted text is not marked ready.
- Where image-only content is skipped, the interface explains that this content is not searchable.
- Only a Document Manager or Administrator can retry failed processing.
- Retrying does not create duplicate searchable chunks.
- Interrupted processing is recoverable or clearly marked failed rather than remaining indefinitely in progress.

## US-07 — Browse the shared document collection

*Requirements: FR-04, FR-17*

As a Reader, I want to browse available documents so that I can choose relevant sources.

- Every active authenticated user can view the shared collection.
- Each document displays its filename, page count where available, uploader, and processing status.
- Documents remain available after an application restart.
- An empty collection displays an explanation rather than a blank or broken screen.
- Readers do not see enabled upload, retry, or deletion controls.
- Original PDFs and source-viewer endpoints require authentication, including when accessed through copied URLs.

## US-08 — Select documents and ask a question

*Requirements: FR-05, FR-06, FR-07*

As a Reader, I want to ask a question about selected documents so that the answer uses the sources I choose.

- The user can select one or more ready documents.
- Submission without any selected document prompts the user to select a source.
- Empty or whitespace-only questions are rejected.
- Questions exceeding **2,000 characters** are rejected with a clear message.
- The interface shows that a question is being processed.
- Retrieval searches only the selected, ready documents.
- Invalid, failed, or deleted document IDs are rejected rather than silently expanding the search.
- Each question is independent; previous answers are not used as evidence or conversational context.

## US-09 — Receive an evidence-based answer

*Requirements: FR-07, FR-08, FR-09*

As a Reader, I want an answer grounded in document evidence so that I can understand and verify the information.

- The answer-generation request contains the question and retrieved evidence from the selected documents.
- The application does not retrieve web content or deliberately supplement missing evidence with general model knowledge.
- Document-based factual claims include supporting citations.
- Paraphrased questions are included in retrieval evaluation to verify that matching does not depend solely on identical wording.
- Instructions inside uploaded documents are treated as document content, not as authority to change permissions or application behaviour.
- Answer correctness and evidence support are measured against the evaluation dataset; citations alone are not treated as proof of correctness.

## US-10 — Inspect citations and highlighted sources

*Requirements: FR-09, FR-10*

As a Reader, I want to open the exact supporting passage so that I can check an answer myself.

- Each citation displays the filename, **PDF page** number, and supporting source text.
- Selecting a citation opens the corresponding document page and highlights its supporting passage.
- Highlights remain aligned when the page is zoomed or resized.
- Filenames, page numbers, and source locations come from stored metadata.
- Displayed quotations match the extracted source text.
- Evidence IDs not present in the retrieved context are rejected rather than displayed as valid citations.
- A citation to a subsequently deleted document displays “source unavailable.”
- Retrieval similarity is not labelled as an answer-confidence percentage.

## US-11 — Understand missing or conflicting evidence

*Requirements: FR-11, FR-12*

As a Reader, I want the assistant to explain limitations in the evidence so that I do not mistake an unsupported answer for a documented fact.

- When retrieved evidence is insufficient, the answer explicitly says that enough information could not be found in the selected documents.
- When only part of a question is supported, the response answers that part with citations and identifies what remains unanswered.
- When relevant retrieved passages disagree, the response describes the disagreement and cites both sources.
- The response does not silently choose a conflicting statement as authoritative without supporting evidence.
- Evaluation includes unanswerable, partially answerable, and conflicting-source examples.
- Correct abstentions and incorrect refusals are reported separately.

## US-12 — Delete a document

*Requirements: FR-13, FR-17, FR-18*

As a Document Manager, I want to delete documents so that obsolete material is no longer used.

- A Document Manager or Administrator can delete any document in the shared collection.
- Readers cannot delete documents through the interface or direct API requests.
- Once deletion begins, the document is excluded from new retrieval requests.
- Successful deletion removes its stored PDF, document metadata, and indexed chunks.
- An in-progress processing job cannot restore a document that has been deleted.
- Incomplete deletion is reported as a failure rather than falsely reported as successful.
- The audit record remains available without retaining document contents.
- Previously displayed citations cannot reopen the removed source.

## US-13 — Recover from operational failures

*Requirement: FR-14*

As a user, I want clear failure messages so that I understand when an operation has failed and can try again appropriately.

- Extraction errors identify the affected document and provide a useful explanation.
- Model timeouts and unavailable services produce an error message and end the loading state.
- A service failure is not presented as “the documents contain no answer.”
- Errors do not expose credentials, internal stack traces, or private document contents.
- A failed question does not alter the selected documents or their stored content.
- Operational logs provide a request identifier and relevant timing or failure information for diagnosis.

## US-14 — Review administrative activity

*Requirement: FR-18*

As an Administrator, I want to review account and document-management activity so that changes are accountable.

- A basic audit view shows records for uploads, deletions, account creation, deactivation, and role changes.
- Each record identifies the actor, action, target, timestamp, and outcome.
- Audit records do not contain passwords, API keys, or full document contents.
- Readers and Document Managers cannot access audit records through either the interface or direct API requests.
- Audit records remain after an application restart.
- Advanced filtering and export are not required for the first release.

## US-15 — Use an accessible, understandable interface

*Requirement: QR-01*

As a user, I want clear controls and feedback so that I can complete tasks using my preferred device and input method.

- The main workflow is usable with a keyboard, with visible focus indicators.
- Inputs and controls have understandable labels.
- Loading, empty, success, failure, and permission-denied states are distinguishable.
- Smaller-screen layouts keep question submission, answers, and citations usable without overlapping controls.
- Before questions are submitted, the application makes clear that questions and retrieved passages may be sent to the configured model provider.
- Unavailable actions are hidden or disabled appropriately, while backend permissions remain enforced independently.

## US-16 — Run and assess the project

*Requirements: QR-05–QR-08*

As a developer or portfolio reviewer, I want reproducible setup and evaluation so that I can assess the implementation and its trade-offs.

- Setup instructions explain dependencies, environment variables, administrator creation, and application startup.
- Docker Compose starts the required services once documented prerequisites and configuration are supplied.
- No real credentials are included in the repository.
- Continuous integration runs the documented automated checks.
- The evaluation dataset contains **60 manually checked questions**, with development and held-out test sets separated before tuning.
- Results compare semantic retrieval, hybrid retrieval, and hybrid retrieval with reranking using documented configurations.
- Reports include evidence recall@5, answer correctness, citation support and coverage, abstention behaviour, median/p95 latency, and estimated generation cost.
- The report evaluates the agreed **85% evidence recall@5** and **90% citation-support** targets and discloses any shortfall.
- Release checks include direct API permission tests, not only checks that buttons are hidden.

The numerical limits above are the agreed defaults. Security boundaries must pass their tests before release; AI quality targets must be measured and reported honestly. These stories describe required behaviour, not features already implemented.
