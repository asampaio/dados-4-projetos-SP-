# [USLP01] Conceptual Domain Model

* **Issue:** [#1](https://github.com/Departamento-de-Engenharia-Informatica/SEM3PI-26-27-2DE02/issues/1)
* **Priority:** Urgent
* **Story Points:** TBD
* **Module / Component:** LAPR3 / Domain Modeling
* **Status:** Done
* **Developer:** Tiago Marquez (group user story — all team members contributed)
* **QA:** João Ferreira (1231752)

---

## 👤 User Story Description

**As a** Product Owner  
**I want** the domain model to be created (conceptual level)  
**So that** it is an essential communication element between all stakeholders, continuously reflecting their shared understanding of the domain

---

## 🚦 Definition of Ready (DoR)

* [x] **Clear User Story:** Story follows the standard *As a / I want to / So that* format (Issue #1).
* [x] **Acceptance Criteria Defined:** ACs taken from GitHub Issue #1 and Project Specification v1 (Section 7.1.1).
* [x] **Dependencies Identified:** None upstream; it is the first artefact of Sprint 1 (see below).
* [x] **Inputs/Outputs Mapped:** Inputs and outputs documented below.
* [x] **Estimation Complete:** Estimated in the Sprint 1 planning.
* [x] **Small & Actionable:** One diagram, completed within Sprint 1 and updated incrementally.

---

## 💬 Customer Specifications & Clarifications

* **Project statement (v1, Section 7.1.1):** the model covers all business/domain aspects of the document; source in PlantUML and rendering in SVG, both in the repository.
* **Clarification (2026-09-28, LAPR3 client session):** USLP01 and USLP02 are group user stories; the domain model is the top priority of Sprint 1 (main entities, attributes, relationships, cardinalities and meaningful association names).
* **Clarification (2026-10-02, LAPR3 client session):** overlapping LAPR3/BDDAD stories (USLP01/USBD03) are documented once and cross-linked; the domain model and the relational model are distinct deliverables.
* **Clarification (2026-10-12, LAPR3 client session):** a `BUSY` robot state is allowed; refrigerated products travel in refrigerated robots, separated from normal items; each robot receives its own sub-order.
* See the [requirements Q&A log](../../requirements/requirements.md).
* **Out of Scope:** keys, data types of the database and constraints (relational model of USBD04).

---

## ✅ Acceptance Criteria

* [x] **AC1 - Coverage:** the model covers all the business/domain aspects presented and discussed in the project specification and in the client sessions.
* [x] **AC2 - Deliverables:** the source of the diagram in PlantUML and its rendering (SVG) are both in the GitHub repository, in sync.

---

## 🔒 Non-Functional Requirements

* **NFR05 - Documentation:** diagram in PlantUML with SVG export, following the project templates.

---

## 🔗 Found Out Dependencies

### Upstream Dependencies (Required before starting)
* None.

### Downstream Dependencies (Impacted by this story)
* **[USLP02 / USBD02] Glossary and data dictionary:** must stay consistent with the domain model.
* **[USBD03] Conceptual data model:** [USBD03 readme](../USBD03/readme.md).
* **[USBD04] Relational model:** [USBD04 readme](../USBD04/readme.md).
* **Sprint 1 user stories (USLP03–06, USEI01–05):** classes of the applications follow the domain concepts.

---

## 📥 Input & Output Data

### Inputs
| Input | Source |
| :--- | :--- |
| Project statement | Project Specification v1 (sections 4, 5 and 7) |
| Client clarifications | [Requirements Q&A log](../../requirements/requirements.md) |

### Outputs
| Output | Location |
| :--- | :--- |
| Domain model (PlantUML) | [doc/domain-model/V2-domain-model.puml](../../domain-model/V2-domain-model.puml) |
| Domain model (SVG) | [doc/domain-model/V2-domain-model.svg](../../domain-model/V2-domain-model.svg) |
| Previous version (V1) | [V1-domain-model.puml](../../domain-model/V1-domain-model.puml), [V1-domain-model.svg](../../domain-model/V1-domain-model.svg) |

---

## 🗺️ Domain Model

![Domain Model](../../domain-model/V2-domain-model.svg)

The business areas covered and the conventions followed are described in the [technical documentation](../../readme.md#domain-model).

### Changes from V1 to V2

* **New classes:** `Corridor` (Area), `StationMovement`, `ShoppingCart`, `CartItem`, `Route`, `Stop` (new package *Planning*).
* **New enumerations:** `MovementDirection`, `StopType`.
* **New associations:** RobotStation schedules StationMovement, StationMovement moves Robot, Robot operates in ShoppingLocation, SystemUser works for RetailerCompany / fills ShoppingCart / plans Route, ShoppingCart contains CartItem, CartItem references Product, SubOrder follows Route, Route sequence of Stop, Stop at Area.
* **Changed multiplicities:** RetailerCompany owns ShoppingLocation (`1..*`), Level contains Area (`*`), SystemUser manages Shop (`0..1`).
* **Removed:** OrderItem originates from Shop — the shop of an order item is the shop of its product (`OrderItem → Product → Shop`).

---

## Edge Cases

* **Domain model changes:** the glossary (USLP02), the conceptual data model (USBD03) and the relational model (USBD04) must be updated together.
* **Concepts not used in Sprint 1** (shopping cart, route planning, station movements, corridors): represented anyway, because the model has to cover the entire specification.

---

## 🧪 Functional Tests / Validation

| Scenario ID | Check | Expected Result |
| :--- | :--- | :--- |
| **TC-01** | Check each business concept of sections 4 and 5 of the specification | Each one is a class, an association, an attribute or a role |
| **TC-02** | Check the client clarifications (28/09, 02/10, 12/10) | All reflected in the model |
| **TC-03** | Check the concepts needed by the Sprint 1 user stories | All present |
| **TC-04** | Compare the SVG with the PlantUML source | Same classes, attributes, associations and multiplicities |

---

# 🧪 QA & Test Documentation

See Report at [QA & Test Documentation: USLP01 Conceptual Domain Model](./qa-readme.md)

---

## 🧩 Related Artifacts

* **Glossary (USLP02):** [Project Glossary](../../Glossaries/glossary/readme.md)
* **Conceptual data model (USBD03):** [V2-conceptual-data-model.svg](../../DataBase/conceptual-data-model/V2-conceptual-data-model.svg)
* **Relational model (USBD04):** [USBD04_ShopRobot_Relational_Model.svg](../../DataBase/Relational-Model/USBD04_ShopRobot_Relational_Model.svg)

---

## 🔄 Sequence Diagram

Not applicable: this user story produces a domain model, not application behaviour.

---

## 🏁 Definition of Done (DoD)

* [x] PlantUML source and SVG in the repository.
* [x] Covers the entire specification and the client clarifications.
* [x] Consistent with the glossary (USLP02).
* [x] Acceptance criteria verified by QA.
