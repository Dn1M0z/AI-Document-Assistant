# Demonstration questions

These questions use the Alder & Loom PDFs in this folder. For company background and the document-generation disclosure, see [COMPANY_BACKGROUND.md](COMPANY_BACKGROUND.md). Overall project status is maintained only in the [main README](../README.md#project-status).

## Target audiences and first-release access

Target audiences below describe the intended business audience in the company scenario, not implemented access restrictions. Under the agreed first-release requirements, all authenticated Readers, Document Managers and Administrators can read the shared collection. Public-facing information is suitable for customers, but the application still requires authentication. Per-document permissions and departmental access rules remain outside the first release.

Administrator and Document Manager are application-management roles, not business departments. The proposed audience labels do not change the agreed permission matrix.

## Useful, document-supported questions

| Question ID | Question | What it tests | Target audience |
| --- | --- | --- | --- |
| Q01 | What is Alder & Loom’s returns policy, and how does it differ for faulty items? | Accurate policy summarisation and distinguishing change-of-mind returns from faulty-item rights. | Everyone, including customers. |
| Q02 | Which jackets and coats cost less than £120, and what materials are they made from? | Finding and filtering product information by category and price. | Everyone, including customers. |
| Q03 | How should I wash the Ridge Hoodie, and how much does it cost? | Combining product and garment-care information with supporting citations. | Everyone, including customers. |
| Q04 | What operating profit is forecast for Q1 2027, and which assumptions underpin it? | Retrieving financial figures and assumptions while distinguishing forecasts from actual results. | Finance and authorised senior management. |
| Q05 | How much would operating profit change between the base and downside forecast scenarios? | Numerical comparison, arithmetic and citations supporting both scenario values. | Finance and authorised senior management. |
| Q06 | Who is responsible for inventory reconciliation, and what qualifications are recorded for that employee? | Connecting operational responsibilities with the relevant employee record. | Authorised HR staff and relevant managers. |
| Q07 | Which supplier provides the Trail Jacket, what are its payment terms, and when is the next shipment expected? | Combining evidence across supplier and inventory documents. | Purchasing, finance, inventory and authorised management. |
| Q08 | How many Core Tees are available to sell after allocations and quarantine stock are deducted? | Understanding inventory definitions and checking available-stock arithmetic. | Inventory, warehouse, merchandising and authorised management. |
| Q09 | How much is allocated to the Winter Edit campaign, and how does that fit into the quarterly marketing budget? | Reconciling campaign and quarterly budget information across documents. | Marketing, finance and authorised management. |

## Unsupported questions

These questions are intentionally unanswerable from the supplied documents. They remain useful evaluation cases because they test whether the assistant acknowledges missing evidence instead of inventing an answer. Q10 was included in the original ten-question set; Q11 and Q12 are the two additional unsupported questions.

| Question ID | Question | What it tests | Target audience |
| --- | --- | --- | --- |
| Q10 | What was Alder & Loom’s actual net profit for the year ending December 2025? | Recognising missing historical accounts and avoiding substitution of the Q1 2027 operating-profit forecast for actual annual net profit. | Finance and authorised senior management would normally use this information; no audience can obtain a supported answer from these documents. |
| Q11 | What is the weather forecast for Sheffield tomorrow? | Recognising an out-of-scope question and stating that the documents contain no weather forecast. | Any user; unsupported for every application role. |
| Q12 | What is the password for Alder & Loom’s supplier portal? | Recognising that no portal password is supplied, avoiding fabricated credentials and not treating supplier contact details as login credentials. | Designated supplier-portal account holders through an approved credential-management process; unsupported for every application role in this document collection. |

## Expected response behaviour

For Q01–Q09, answers should identify the source document and PDF page for each factual claim. Answers using multiple sources should cite each relevant source; calculations should show the source values and arithmetic.

For Q10–Q12, the assistant should explicitly state that the supplied documents do not support an answer. It should not invent facts, credentials or supporting citations. Missing evidence and lack of user authorisation are different outcomes; the audience labels alone must not be treated as enforced access controls.

This file defines questions, test purposes and target audiences. It is not yet an annotated answer key or a report of achieved test results.
