# STORIES.md

## Project Overview
**loanflow-engine** is a cloud‑native banking engine that automates loan origination, underwriting, servicing, and reporting for fintech lenders.  
The goal of the MVP is to deliver a fully‑functional, secure, and scalable core that can be integrated with external credit bureaus, payment processors, and a lightweight UI for loan officers.

---

## Epics & Story Backlog

| Epic | Story | Acceptance Criteria |
|------|-------|---------------------|
| **1. Core Loan Lifecycle** | **1.1** *As a loan officer, I want to create a new loan application, so that I can capture borrower data and initiate underwriting.* | • Loan form accepts borrower name, SSN, income, employment, and requested amount.<br>• Validation rules enforce required fields and numeric ranges.<br>• Application is persisted in the database with status “Submitted”.<br>• An event is emitted to the underwriting service. |
| | **1.2** *As a system, I want to automatically assign an underwriting score, so that we can triage applications.* | • Underwriting service receives the application event.<br>• Uses a simple rule‑based model (credit score, debt‑to‑income) to compute a score.<br>• Updates loan record with score and status “Underwriting‑In‑Progress”. |
| | **1.3** *As a loan officer, I want to approve or reject a loan, so that the borrower receives a decision.* | • UI presents score and risk flags.<br>• Approve action sets status to “Approved” and triggers disbursement.<br>• Reject action sets status to “Rejected” and records reason. |
| | **1.4** *As a system, I want to disburse funds to the borrower’s account, so that the loan is funded.* | • Integration with a mock payment gateway.<br>• Funds are transferred to a test bank account.<br>• Loan record status changes to “Funded” and a transaction record is created. |
| | **1.5** *As a borrower, I want to view my loan balance and payment schedule, so that I can manage my repayments.* | • API endpoint `/loans/{id}/status` returns principal, interest, remaining term, next due date.<br>• Data is refreshed after each payment. |
| | **1.6** *As a system, I want to process scheduled payments, so that the loan balance updates automatically.* | • Scheduler triggers daily at 00:00 UTC.<br>• Applies payment amount to principal and interest.<br>• Updates loan record and emits payment event. |
| | **1.7** *As a loan officer, I want to generate a PDF statement, so that I can provide a formal record to the borrower.* | • PDF includes loan details, payment history, and next due date.<br>• PDF is stored in the loan’s document repository. |
| | **1.8** *As a system, I want to archive closed loans, so that the database remains performant.* | • Loans with status “Closed” older than 5 years are moved to an archive table.<br>• Archived loans can be queried via a read‑only API. |

| Epic | Story | Acceptance Criteria |
|------|-------|---------------------|
| **2. Integration & Data Flow** | **2.1** *As a system, I want to fetch credit bureau data, so that underwriting can be informed by external credit scores.* | • Mock credit bureau API returns a score and history.<br>• Underwriting service consumes the API and stores the result. |
| | **2.2** *As a system, I want to publish loan events to a message bus, so that downstream services can react asynchronously.* | • Events are published to a Kafka topic `loan-events`.<br>• Each event contains loan ID, type, and payload. |
| | **2.3** *As a system, I want to consume payment events from the payment gateway, so that the loan balance updates.* | • Payment gateway posts to `/webhooks/payments`.<br>• Event is validated, stored, and triggers balance update. |
| | **2.4** *As a system, I want to expose a REST API for external partners, so that they can query loan status.* | • API is documented with OpenAPI 3.0.<br>• Supports GET `/loans/{id}` with authentication. |

| Epic | Story | Acceptance Criteria |
|------|-------|---------------------|
| **3. Security & Compliance** | **3.1** *As a system, I want to encrypt sensitive data at rest, so that borrower information is protected.* | • SSN and income are stored encrypted using AES‑256.<br>• Encryption keys are managed via AWS KMS. |
| | **3.2** *As a system, I want to enforce role‑based access control, so that only authorized users can perform actions.* | • JWT tokens contain roles `loan_officer`, `admin`.<br>• Endpoints enforce role checks. |
| | **3.3** *As a system, I want to log all audit events, so that we can meet regulatory requirements.* | • Every create/update/delete operation logs user ID, timestamp, and change set to an audit table. |
| | **3.4** *As a system, I want to perform automated vulnerability scans, so that we can detect security gaps.* | • CI pipeline runs OWASP ZAP against the API.<br>• Scan results must pass with zero critical findings. |

| Epic | Story | Acceptance Criteria |
|------|-------|---------------------|
| **4. DevOps & Observability** | **4.1** *As a DevOps engineer, I want to containerize the application, so that it can run on Kubernetes.* | • Dockerfile builds a multi‑stage image.<br>• Helm chart deploys to a test cluster. |
| | **4.2** *As a system, I want to expose metrics to Prometheus, so that we can monitor performance.* | • `/metrics` endpoint exposes latency, request count, error rate.<br>• Grafana dashboards display key metrics. |
| | **4.3** *As a system, I want to log structured JSON, so that logs can be indexed.* | • All logs are JSON with fields: timestamp, level, message, request_id. |
| | **4.4** *As a system, I want to implement CI/CD pipelines, so that changes are automatically tested and deployed.* | • GitHub Actions run unit tests, integration tests, linting, and build Docker image.<br>• Successful PR merges trigger a staging deployment. |

| Epic | Story | Acceptance Criteria |
|------|-------|---------------------|
| **5. User Experience** | **5.1** *As a loan officer, I want a responsive web UI, so that I can manage loans from any device.* | • React app built with Material‑UI.<br>• Forms validate client‑side and show inline errors. |
| | **5.2** *As a borrower, I want a self‑service portal, so that I can view my loan status and make payments.* | • Separate SPA with authentication.<br>• Payment button triggers payment flow. |
| | **5.3** *As a system, I want to provide API documentation, so that partners can integrate easily.* | • Swagger UI available at `/docs`.<br>• All endpoints are documented with example requests/responses. |

---

## MVP Release Plan

1. **Core Loan Lifecycle** (Epics 1) – 4 weeks  
2. **Integration & Data Flow** (Epics 2) – 3 weeks  
3. **Security & Compliance** (Epics 3) – 2 weeks  
4. **DevOps & Observability** (Epics 4) – 2 weeks  
5. **User Experience** (Epics 5) – 3 weeks  

Total: **14 weeks** (including buffer for testing and bug fixes).

---

## Definition of Done (DoD)

- All acceptance criteria met and unit tests ≥ 90% coverage.  
- Integration tests pass in CI.  
- Security scan reports show no critical issues.  
- Documentation (API, architecture, deployment) is complete.  
- Feature deployed to staging and manually verified by product owner.  
- Story marked as “Done” in the issue tracker.
