# [USLP02] Project Glossary / Data Dictionary

* **Priority:** TBD (to be defined in Sprint Planning)
* **Story Points:** 2 (size S, as estimated on the project board)
* **Module / Component:** `Documentation` / `Glossary` (`doc/glossary/readme.md`)
* **Status:** In Review (glossary drafted and updated with the client clarifications; review and sign-off pending)

---

## 👤 User Story Description

**As a** Product Owner
**I want** a data dictionary/glossary to be created
**So that** all team members and course units share a consistent understanding of the business concepts throughout the project

> Equivalent deliverable to [USBD02](../../BDDAD/USBD02/readme.md) — the requirements document states "Please refer to USLP02" for that story. Same artefact, counted for both LAPR3 and BDDAD: the glossary is shared, and this documentation describes the same file.

---

## 🚦 Definition of Ready (DoR)

* [x] **Clear User Story:** Story follows the standard *As a / I want / So that* format.
* [x] **Acceptance Criteria Defined:** Taken directly from the project requirements document.
* [x] **Dependencies Identified:** See "Found Out Dependencies" below.
* [x] **Inputs/Outputs Mapped:** Not applicable, this is a documentation deliverable, not a data-processing feature.
* [x] **Estimation Complete:** 2 story points (size S), as estimated on the project board.
* [x] **Small & Actionable:** Scoped to Sprint 1; the glossary evolves in later sprints as new requirements emerge.

---

## 💬 Customer Specifications & Clarifications

* **Requirements doc (USLP02):** the glossary should be consistent with the Conceptual Model Diagram.
* **Requirements doc (USLP02):** the focus should be on the business concepts.
* **Requirements doc (USLP02):** the clarifications/definitions provided should be relevant for the problem interpretation and avoid generic (Wikipedia-like) definitions.
* **Requirements doc (USBD02):** the BDDAD course asks for the same glossary and refers to USLP02, so there is a single glossary for both.
* **Clarification (client sessions, duplicated stories):** the glossary is required by more than one course (LAPR3 and BDDAD). It is treated as a single artefact, with one documentation entry that links to the other.
* **Clarification (client sessions, refrigeration):** refrigerated robots carry only refrigerated products, and non-refrigerated robots carry only non-refrigerated products. There are no mixed robots. Recorded in "Refrigerated Unit".
* **Clarification (client sessions, robot states):** a state for a robot that is working on an order ("busy") is allowed and makes sense, so the `READY`/`IN_USE` names are kept. Recorded in "Robot State".
* **Clarification (client sessions, battery):** the minimum battery needed for an order is not in the acceptance criteria. It can be an input parameter or a global property (an indicative value is 20–25%), and the unit is not defined in Sprint 1. Recorded in "Battery Level".
* **Clarification (client sessions, orders):** orders are final once confirmed and paid. Recorded in "Order".
* **Clarification (client sessions, routes):** route planning is done by the system with input from an operator (the operator sets the request, the system proposes the route and the operator accepts or rejects it). The navigation of the robot itself is autonomous and out of scope. Recorded in "Route".
* **Clarification (client sessions, actors):** the "Infrastructure Planner" of the ESINF stories is the System Planner of the use case diagram. The difference in naming was posted on the forum, and the answer is pending.
* **Clarification (BDDAD teacher, prices):** changing the price of a product must not affect previous transactions, and a serious retail system keeps a price history. Recorded in "Product Price" and "Product".
* **Clarification (Moodle forum, GeoJSON, as recorded in [USLP06](../USLP06/readme.md)):** the shop features have an `id` property and some `level` values are decimal numbers (for example 0.5). Recorded in "Area" and "Level".
* **Decision (scope of the terms):** business concepts only. Technical terms (database columns, endpoints, classes of the code) belong to the code documentation, not to the glossary.
* **Decision (actors):** actors that are not classes of the model (Retailer Administrator, Maintenance Operator, System Operator, System Planner and Public) are documented in the glossary, and the ones in "Other Actors" are marked as not modeled.
* **Decision (merchant of a store):** a store created by the import of the GeoJSON file has no merchant until the Retailer Administrator assigns one. Recorded in "Store" and "Merchant".
* **Assumption (terms for later sprints):** `Route` and `Stop` are documented now to avoid terminology drift, but no Sprint 1 story plans routes.
* **Assumption (order status):** the `Order Status` values are a proposal, still to be confirmed with the client.
* **Out of Scope:** the domain model diagram ([USLP01]), the relational model and data dictionary of the database tables ([USBD04]), and implementation classes.

---

## ✅ Acceptance Criteria

* [ ] **AC1 - Consistency:** The glossary is consistent with the Conceptual Model Diagram. *(to be ticked after the reviewer confirms it)*
* [x] **AC2 - Business Focus:** The focus is on the business concepts.
* [ ] **AC3 - Relevant Definitions:** The clarifications/definitions are relevant for the problem interpretation and avoid generic definitions. *(to be ticked after the reviewer confirms it)*

---

## 🔒 Non-Functional Requirements & Security

* **NFR05 (Documentation):** the glossary follows the project documentation standards, and the figures referenced are in SVG format.
* **Process:** trunk-based development, at least one commit per day. The glossary is updated together with the domain model on every change. Review by a team member other than the author (section 3.4).
* **Not applicable:** performance, security, authorization and database NFRs (this is a static document, not a running feature).

---

## 🔗 Found Out Dependencies

### Upstream Dependencies (Required before starting)
* **Requirements document:** source of the business concepts and rules.
* **Clarifications from the teachers/client:** sessions and forum answers that change or confirm a definition.

### Parallel Dependencies
* **[USLP01]/[USBD03] Domain model:** the glossary and the model must always be consistent. Both are updated together.
* **[USBD02] Glossary (BDDAD):** same deliverable, documented in the BDDAD folder.

### Downstream Dependencies (Impacted by this story)
* **All documentation and code of all course units:** class, attribute and business names should follow the glossary, so the whole team uses the same vocabulary.
* **[USBD04] Relational model:** the names of the tables and columns are derived from the glossary and the domain model.

---

## 📥 Input & Output Data

### Input Data
| Data                    | Type            | Description                                                  |
|:------------------------|:----------------|:-------------------------------------------------------------|
| Requirements document   | PDF             | Business concepts, rules and user stories                    |
| Domain model            | PlantUML / SVG  | Classes, enumerations and value objects to be documented     |
| Clarifications          | Teachers/client | Answers that change or confirm a definition                  |

### Output Data
| Data      | Type             | Description                                                              |
|:----------|:-----------------|:-------------------------------------------------------------------------|
| Glossary  | Markdown         | `doc/glossary/readme.md`, with an index and one entry per business term |

---

## Edge Cases

* **Terms not yet confirmed with the client:** the `Order Status` values are flagged in the glossary as a proposal, pending validation.
* **Terms reserved for a future sprint:** `Route` and `Stop` are documented now but are out of scope for Sprint 1 (Planning application).
* **Actors that are not classes:** `Retailer Administrator`, `Maintenance Operator`, `System Operator`, `System Planner` and `Public` are in the glossary but not in the model.
* **Term in the model but not in the glossary:** not accepted. Every class, enumeration and value object has an entry with the same name.
* **Requirements change during the semester:** the glossary is updated together with the domain model, and the index must contain every entry.

---

## 🧪 Functional Tests / Validation

Since this is a documentation deliverable, "testing" is a review checklist (see [qa-readme.md](./qa-readme.md)):

| Scenario ID | Review Check | Expected Result |
| :--- | :--- | :--- |
| **F-01** | Open the index and follow every link | No broken link; every entry is in the index once |
| **F-02** | Compare the classes, enumerations and value objects of `domain-model.puml` with the glossary | Every one has an entry with the same name |
| **F-03** | Read the entries changed by the client and teacher clarifications (Refrigerated Unit, Robot State, Battery Level, Order, Route, Product Price, Area, Level) | They match what the client or the teacher said |
| **F-04** | Check the entry of "Customer" and "GeoJSON" | No duplicated definition; the lists are correctly indented |

---

# 🧪 QA & Test Documentation

See Report at [QA & Test Documentation: USLP02 Project Glossary](./qa-readme.md)

---

## 🧩 Related Modules & Classes

Not applicable, there is no code module. The artefact is [doc/glossary/readme.md](../../glossary/readme.md).

---

## 🔄 Sequence Diagram

Not applicable. This story has no runtime interaction/flow to diagram, it documents static business concepts.

---

## 🏁 Definition of Done (DoD)

* [x] Acceptance criteria verified against the requirements document.
* [x] Glossary updated with the client clarifications.
* [x] Index complete and without broken links.
* [ ] Glossary consistent with the domain model, confirmed by the reviewer.
* [ ] Reviewed by a team member other than the author.
* [ ] Final sign-off recorded in `qa-readme.md`.