# [USBD03] Conceptual Data Model (Entity-Relationship)

* **Priority:** Urgent
* **Story Points:** 3
* **Module / Component:** BDDAD / Data Modeling
* **Status:** Testing
* **Developer:** João Ferreira (1231752)
* **QA:** Francisco Marques (1212081)

---

## 👤 User Story Description

**As a** Product Owner
**I want** the domain model to be created (conceptual level)
**So that** all stakeholders share the same understanding of the ShopRobot data before the relational model (USBD04) is designed

---

## 🚦 Definition of Ready (DoR)

* [x] **Clear User Story:** Story follows the standard *As a / I want to / So that* format.
* [x] **Acceptance Criteria Defined:** ACs below, based on the project statement (USBD03 refers to USLP01) and the BDDAD clarifications.
* [x] **Dependencies Identified:** Domain model of USLP01 available in `doc/domain-model`.
* [x] **Inputs/Outputs Mapped:** Inputs and outputs documented below.
* [x] **Estimation Complete:** 3 story points (Sprint 1 planning).
* [x] **Small & Actionable:** One diagram, completed within Sprint 1.

---

## 💬 Customer Specifications & Clarifications

* **Project statement (v1, Section 7.3.1):** USBD03 refers to USLP01 — the model covers all business/domain aspects of the document; source in PlantUML and rendering in SVG, both in the repository.
* **Clarification (2026-10-02, LAPR3 client session):** overlapping LAPR3/BDDAD stories are documented once and cross-linked, but the conceptual model and the relational model are distinct deliverables with distinct BDDAD criteria.
* **Clarification (2026-10-06, Moodle BDDAD, Prof. Angelo Martins):** the data model must cover the entire specification and all project user stories, and be developed incrementally in Git in non-closed formats (SVG, PNG, PDF).
* **Clarification (2026-10-12, LAPR3 client session):** a `BUSY` robot state is allowed; refrigerated products travel in refrigerated robots, separated from normal items; each robot receives its own sub-order.
* See the [requirements Q&A log](../../requirements/requirements.md).
* **Out of Scope:** attributes, data types, keys and constraints (domain model of USLP01 and relational model of USBD04).

---

## ✅ Acceptance Criteria

* [ ] **AC1 - Coverage:** the model covers all business/domain aspects of the project specification and of the project user stories.
* [ ] **AC2 - Consistency:** entities, associations and multiplicities are consistent with the domain model (USLP01).
* [ ] **AC3 - Conceptual level:** only business entities and their relationships (name, direction, multiplicities); generalization, composition and aggregation where they apply.
* [ ] **AC4 - Deliverables:** PlantUML source and SVG rendering in the repository, in sync.

---

## 🔒 Non-Functional Requirements

* **NFR02 - Database:** the conceptual model is the basis of the relational model and of the data integrity of the database.
* **NFR05 - Documentation:** diagram in PlantUML with SVG export, following the project templates.

---

## 🔗 Found Out Dependencies

### Upstream Dependencies (Required before starting)
* **[USLP01] Domain model:** [V2-domain-model.puml](../../domain-model/V2-domain-model.puml) — entities and attributes.

### Downstream Dependencies (Impacted by this story)
* **[USBD04] Relational model:** [USBD04_ShopRobot_Relational_Model.puml](../../DataBase/Relational-Model/USBD04_ShopRobot_Relational_Model.puml) — one table per entity, FKs from the associations.
* **[USBD02] Data dictionary:** business rules consistent with the multiplicities.
* **[USBD05] Physical model and data.**

---

## 📥 Input & Output Data

### Inputs
| Input | Source |
| :--- | :--- |
| Domain model | [V2-domain-model.puml](../../domain-model/V2-domain-model.puml) |
| Project statement | Project Specification v1 (sections 4, 5 and 7) |
| Client clarifications | [Requirements Q&A log](../../requirements/requirements.md) |

### Outputs
| Output | Location |
| :--- | :--- |
| Conceptual model (PlantUML) | [doc/DataBase/conceptual-data-model/V2-conceptual-data-model.puml](../../DataBase/conceptual-data-model/V2-conceptual-data-model.puml) |
| Conceptual model (SVG) | [doc/DataBase/conceptual-data-model/V2-conceptual-data-model.svg](../../DataBase/conceptual-data-model/V2-conceptual-data-model.svg) |

---

## 🗺️ Conceptual Model

![Conceptual Data Model](../../DataBase/conceptual-data-model/V2-conceptual-data-model.svg)

### Entities (23)

| Area | Entities |
| :--- | :--- |
| Users | `SystemUser` (roles: Customer, Merchant, RetailerAdmin, MaintenanceOperator, SystemPlanner, SystemOperator) |
| Retail & Layout | `RetailerCompany`, `ShoppingLocation`, `Level`, `Area` (abstract) → `Shop`, `Amenity`, `DeliverySpot`, `RobotStation`, `Corridor`; `Elevator` |
| Robotics | `RobotStation`, `ControlTerminal`, `Robot`, `StationMovement` |
| Catalog, Cart & Orders | `Product`, `ShoppingCart`, `CartItem`, `Order`, `SubOrder`, `OrderItem` |
| Planning | `Route`, `Stop` |

### Relationships

| Relationship | Multiplicity | Kind |
| :--- | :--- | :--- |
| RetailerCompany owns ShoppingLocation | 1 — 1..* | association |
| ShoppingLocation has Level | 1 — 1..* | composition |
| Level contains Area | 1 — * | composition |
| Level connected by Elevator | 2..* — 1..* | association |
| Shop, Amenity, DeliverySpot, RobotStation, Corridor are Areas | — | generalization |
| SystemUser (RetailerAdmin) works for RetailerCompany | * — 0..1 | association |
| SystemUser (Merchant) manages Shop | 0..1 — * | association |
| SystemUser (MaintenanceOperator) operates ControlTerminal | 1 — * | association |
| SystemUser (Customer) places Order | 1 — * | association |
| SystemUser (Customer) fills ShoppingCart | 1 — 0..1 | association |
| SystemUser (SystemPlanner) plans Route | 1 — * | association |
| RobotStation has ControlTerminal | 1 — 1 | composition |
| RobotStation parks / charges Robot | 0..1 — * | aggregation |
| RobotStation schedules StationMovement | 1 — * | composition |
| StationMovement moves Robot | * — 1 | association |
| Robot operates in ShoppingLocation | * — 1 | association |
| Shop offers Product | 1 — * | association |
| ShoppingCart contains CartItem | 1 — * | composition |
| CartItem refers to Product | * — 1 | association |
| Order delivered at DeliverySpot | * — 1 | association |
| Order split into SubOrder | 1 — 1..* | composition |
| Order contains OrderItem | 1 — 1..* | composition |
| SubOrder carried by Robot | * — 0..1 | association |
| SubOrder groups OrderItem | 1 — 1..* | aggregation |
| SubOrder follows Route | 1 — 0..1 | association |
| Route sequence of Stop | 1 — 2..* | composition |
| Stop at Area | * — 1 | association |
| OrderItem refers to Product | * — 1 | association |

The shop of an order item is the shop of its product (`OrderItem → Product → Shop`), so no direct `OrderItem — Shop` relationship is kept.

---

## Edge Cases

* **Domain model changes:** the conceptual model, the data dictionary (USBD02) and the relational model (USBD04) must be updated together.
* **Concepts of the specification not used in Sprint 1** (shopping cart, route planning, schedule of station movements, corridors): represented anyway, because the model has to cover the entire specification (clarification of 2026-10-06).

---

## 🧪 Functional Tests / Validation

| Scenario ID | Check | Expected Result |
| :--- | :--- | :--- |
| **TC-01** | Compare the classes of the domain model with the entities of the conceptual model | Same entities, same names |
| **TC-02** | Compare the multiplicities with the domain model | Same multiplicities |
| **TC-03** | Check each business concept of sections 4 and 5 of the specification | Each one is an entity, a relationship or a role |
| **TC-04** | Check the entities needed by the Sprint 1 user stories | All present |
| **TC-05** | Render the PlantUML source and compare with the SVG | SVG up to date |

---

# 🧪 QA & Test Documentation

See Report at [QA & Test Documentation: USBD03 Conceptual Data Model](./qa-readme.md)

---

## 🧩 Related Artifacts

* **Domain model (USLP01):** [V2-domain-model.svg](../../domain-model/V2-domain-model.svg)
* **Relational model (USBD04):** [USBD04_ShopRobot_Relational_Model.svg](../../DataBase/Relational-Model/USBD04_ShopRobot_Relational_Model.svg)
* **Data dictionary (USBD02):** [DBglossary](../../Glossaries/DBglossary/readme.md)

---

## 🔄 Sequence Diagram

Not applicable: this user story produces a data model, not application behaviour.

---

## 🏁 Definition of Done (DoD)

* [x] PlantUML source and SVG in the repository.
* [ ] Consistent with the domain model (USLP01).
* [ ] Covers the entire specification.
* [ ] Acceptance criteria verified by QA.
