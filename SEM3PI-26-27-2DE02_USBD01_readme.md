# [USBD01] Project Requirements Document

* **Priority:** Must
* **Story Points:** 3
* **Module / Component:** BDDAD / Requirements Engineering
* **Status:** Done
* **Developer:** Gonçalo Abreu
* **QA:** Francisco Marques

---

## 👤 User Story Description

**As a** Product Owner  
**I want** a project requirements document to be created and maintained, including the functional and non-functional requirements included in the project requirements document, user stories and other requirements that may result from the questions and discussions with the client  
**So that** all stakeholders share a dynamic, unambiguous, and continuously updated understanding of the domain, system scope, and constraints across all project sprints.

> [!IMPORTANT]
> **Primary Deliverable Artifact:**  
> 📄 **[`doc/requirements/requirements.md`](../../requirements/requirements.md)** (Current Version: **v1.2**)  
> 
> The central and dynamic requirements document is maintained in the shared folder at [`doc/requirements/requirements.md`](../../requirements/requirements.md).  
> *Direct section shortcuts:*
> * 📑 [Version History & Evolution](../../requirements/requirements.md#1-document-control--evolution)
> * ⚙️ [Non-Functional Requirements (NFRs)](../../requirements/requirements.md#2-non-functional-requirements-nfr)
> * 📋 [Functional Requirements & User Stories](../../requirements/requirements.md#3-functional-requirements)
> * 💬 [Client Q&A & Clarification Log](../../requirements/requirements.md#4-qa--client-clarifications-log)


---

## 🚦 Definition of Ready (DoR)

* [x] **Clear User Story:** Follows standard *As a / I want / So that* pattern.
* [x] **Acceptance Criteria Defined:** All ACs defined based on Project Specification v1 (Section 7.3.1) and client session guidelines.
* [x] **Dependencies Identified:** Feeds all downstream modeling stories (USLP01/USBD03, USLP02/USBD02, USBD04, USBD05, ESINF).
* [x] **Inputs/Outputs Mapped:** Inputs from project specification and client Q&A logs; output is the dynamic requirements document.
* [x] **Estimation Complete:** 3 story points (Sprint 1 planning).
* [x] **Small & Actionable:** Dynamic repository artifact maintained iteratively in Markdown.

---

## 💬 Customer Specifications & Clarifications

* **Project Specification v1 (Section 7.3.1):**
  * Requirements document must include functional and non-functional requirements, user stories, and all rules resulting from discussions and client questions.
  * Must specify the exact source for each requirement and rule.
  * Dynamic document reflecting shared stakeholder understanding across all sprints.
* **Client Session (28/09/2026):**
  * Project involves LAPR3, ESINF, and BDDAD in Sprint 1; ArqCP in Sprints 2 & 3; Física in Sprint 3.
  * Separation of roles: developer implements tests; QA executes tests/verifications.
  * CSV input error handling: skip line, report line/field error, continue parsing.
* **Client Session (02/10/2026):**
  * Orders are final once paid; oversized products blocked at registration or logged with user notification.
  * System handles route planning with human input; robot path navigation is autonomous and outside project scope.
  * Refrigerated products handled by robots; cargo volume packing assumes human intervention for now.
  * Console UI is the baseline; alternative UI is a soft requirement requiring consensus.
  * GitHub Actions limited to single nightly build (cap on department minutes).
* **BDDAD Clarification (06/10/2026 - Prof. Angelo Martins):**
  * Data model must cover the entire specification and all project user stories. Development must be incremental in Git using non-closed formats.
* **BDDAD Clarification (09/10/2026 - Prof. Angelo Martins):**
  * Auto-generated IDs for test data are an anti-pattern. Fixed and deterministic IDs must be used.
* **Client Session (12/10/2026):**
  * Architecture ecosystem: 3 Java applications (Shopping front-office, Administrative back-office, Planning robot distribution).
  * Sprint 1 uses in-memory H2 database via JDBC; persistent DBMS (Oracle) planned for later sprints.
  * Robot operational status: adding "busy" state makes domain sense. LEDs (3 LEDs) allow combinations.
  * Robot selection criteria (US04): pick robot with highest battery; ties can be arbitrary.
  * Refrigerated items must be placed in separate refrigeration robots (never mixed with normal goods).
  * Each robot receives its own order/sub-order. Delivery scheduling (Uber-like) is for future sprints.
* **Client Session (16/10/2026):**
  * Battery measurement unit is open in Sprint 1 (currently percentage; future sprints may use meters, km, or time).
  * Feasibility check (US4): minimum battery threshold is acceptable as an input parameter or global property (e.g. 20-25%).
  * Strict robot segregation: refrigerated robots only carry refrigerated items; non-refrigerated robots carry non-refrigerated items (no mixed robots).
  * Robot routes: system proposes route; logistics operator accepts or rejects (not manual step-by-step definition).
  * POC scope: single shopping mall (Norte Shopping) for Sprint 1.
  * Data flow: GeoJSON (US6) imports levels/stores; planning algorithms (US4) consume validated data from DB.
  * Sprint 1 delivery: Git tag snapshot + collaborative peer self-evaluation spreadsheet (`management/` folder).

---

## ✅ Acceptance Criteria

* [x] **AC1 - Full Specification Coverage:** The document includes all functional and non-functional requirements from the project specification v1.
* [x] **AC2 - Traceability of Sources:** Every requirement, constraint, and user story clearly states its source (e.g. section of the specification, client meeting date, or Moodle forum announcement).
* [x] **AC3 - Dynamic Evolution & Version History:** The document has explicit versioning (`v0.1`, `v1.0`, `v1.1`, `v1.2`, ...) documenting changes as new clarifications arrive.
* [x] **AC4 - Q&A Clarification Log:** A dedicated, chronological table records every question, resolution, and reference from client sessions and teacher announcements.
* [x] **AC5 - Cross-Module Consistency:** Constraints relevant to LAPR3, ESINF, and BDDAD (such as database technologies, robot states, and architecture boundaries) are kept consistent.

---

## 🔒 Non-Functional Requirements

* **NFR01 - Business Rules:** Serves as the central repository of validated business rules and constraints.
* **NFR02 - Database:** Captures database requirements (H2 for Sprint 1 prototyping, Oracle for BDDAD relational model and future sprints).
* **NFR05 - Documentation:** Structured in Markdown inside `doc/requirements/requirements.md` in the repository, following project documentation standards.
* **NFR06 - Architecture Structure:** Documents the 3-application system structure (Shopping App, Admin App, Planning App).
* **NFR07 - CI/CD & Build Policies:** Documents the single nightly build rule and team Git workflow.

---

## 🔗 Found Out Dependencies

### Upstream Dependencies (Inputs)
* Project Specification document v1.
* Moodle Announcements & Discussion Forums (Prof. Angelo Martins, Prof. Alexandre Bragança).
* Client Sessions logs (`LAPR3-T-Online` 28/09, 02/10, 12/10).

### Downstream Dependencies (Affected Artifacts)
* **[USLP01 / USBD03] Domain Model:** Directly consumes domain concepts, states (e.g. `busy`), and relationships defined here.
* **[USLP02 / USBD02] Data Dictionary:** Uses domain rules, entity definitions, and open questions logged here.
* **[USBD04] Relational Model:** Relational schema and constraints must satisfy the requirements and deterministic ID policies recorded here.
* **[USBD05] Database Implementation:** SQL DDL and seed data must respect business rules and data models specified here.
* **[ESINF User Stories]:** Algorithmic rules (battery selection, non-mixing of refrigerated items, sub-orders) follow rules documented here.

---

## 📥 Inputs & 📤 Outputs

### Inputs
* Project specification v1 (`SEM3PI-26-27-Specification-v1.pdf`).
* Forum announcements and client Q&A logs.

### Outputs
* **Dynamic Requirements Document:** [`doc/requirements/requirements.md`](../../requirements/requirements.md)

#### 📑 Requirements Version Summary:
| Version | Date | Summary of Evolution | Key Triggers / Sources |
| :--- | :--- | :--- | :--- |
| **v0.1** | 2026-10-01 | Initial skeleton & sprint structure | Project Launch |
| **v1.0** | 2026-10-07 | Full specification requirements & global NFRs | Project Spec v1 |
| **v1.1** | 2026-10-09 | BDDAD modeling rules & deterministic ID policy | Prof. Angelo Martins announcements |
| **v1.2** | 2026-10-13 | Client session rules (3 Java apps, H2, robot `BUSY`, refrigeration separation) | Client sessions (28/09, 02/10, 12/10) |
| **v1.3** | 2026-10-16 | Client session rules (battery units, US4 threshold, strict refrigeration, route proposal, single-mall POC, submission guidelines) | Client session (16/10) |

---

## 🛠️ Work Summary & Maintenance Process

1. **Initial Baseline:** Outlined global NFRs, Sprint 1 functional stories across LAPR3, ESINF, BDDAD.
2. **Incremental Updates:**
   * Incorporated BDDAD modeling announcements from Prof. Angelo Martins (data model coverage, SVG format, deterministic IDs).
   * Incorporated Client Session logs (28/09, 02/10, 12/10, 16/10) covering application structure, in-memory H2 DB, robot states (`busy`), battery units/thresholds, refrigeration segregation, and submission rules.
3. **Traceability:** Every item added to the Q&A table in Section 4 references the exact source and date.

---

## 🚦 Definition of Done (DoD) Checklist

* [x] Requirements document created and maintained in Git (`doc/requirements/requirements.md`).
* [x] Version history reflects all changes up to Sprint 1 current date (v1.3).
* [x] Functional and Non-Functional Requirements properly organized by module and sprint.
* [x] Client session logs (28/09, 02/10, 12/10, 16/10) registered in the Q&A table.
* [ ] QA verification and sign-off (assigned to QA: Francisco Marques).
