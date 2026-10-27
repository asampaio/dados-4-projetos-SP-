# 🧪 QA & Test Documentation: [USLP01] Conceptual Domain Model

* **User Story Reference:** [USLP01: Conceptual Domain Model](./readme.md)
* **QA Lead / Owner:** João Ferreira (1231752)
* **Artifact Under Review:** [V2-domain-model.puml](../../domain-model/V2-domain-model.puml) and [V2-domain-model.svg](../../domain-model/V2-domain-model.svg) (commit `e2cfaa8`)
* **QA Status:** 🟢 Passed

---

## 🎯 QA Strategy & Scope

### In-Scope
* Coverage of the project specification (sections 4 and 5), the Sprint 1 user stories and the client clarifications.
* Conceptual-level notation (associations named, with direction and multiplicities; generalization, composition, aggregation and enumerations).
* PlantUML source and SVG in sync.
* Consistency with the conceptual data model (USBD03) and the glossary (USLP02).

### Out-of-Scope
* Keys, database data types and constraints (relational model, USBD04).
* Application classes of the Sprint 1 user stories.

---

## 🛠️ Review Setup

* **Specification:** Project Specification v1 (sections 4, 5 and 7).
* **Reference clarifications:** [Requirements Q&A log](../../requirements/requirements.md) (client sessions of 2026-09-28, 2026-10-02 and 2026-10-12).
* **Conceptual data model version:** [V2-conceptual-data-model.puml](../../DataBase/conceptual-data-model/V2-conceptual-data-model.puml).

---

## 📋 Test Matrix

| Test ID | Title / Scenario | Type | Test Steps | Expected Result | Status | Execution Date | Issues Found |
| :--- | :--- | :--- | :--- | :--- | :---: | :---: | :--- |
| **TC-01** | Coverage of the specification | Coverage | Check each business concept of sections 4 and 5 | Every concept is a class, association, attribute or role | 🟢 PASS | 2026-10-14 | [QA-01](#-issues-found), [QA-02](#-issues-found) (minor) |
| **TC-02** | Consistency with client clarifications | Consistency | Check the clarifications of 2026-09-28, 2026-10-02 and 2026-10-12 | No contradiction | 🟢 PASS | 2026-10-14 | None |
| **TC-03** | Coverage of the Sprint 1 user stories | Coverage | Check the concepts used by USLP04, USLP06 and USEI01–USEI05 | All present | 🟢 PASS | 2026-10-14 | None |
| **TC-04** | Conceptual-level notation | Notation | Check every association, generalization, composition, aggregation and enumeration | Named, with direction and multiplicities; whole/part on the correct side | 🟢 PASS | 2026-10-14 | None |
| **TC-05** | Deliverables in the repository | Deliverable | Check `.puml` and `.svg`; compare classes, enumerations, associations and multiplicities of the SVG with the source | Both present and in sync | 🟢 PASS | 2026-10-14 | [QA-04](#-issues-found) |
| **TC-06** | Consistency with the conceptual data model (USBD03) | Consistency | Compare the classes and associations of both V2 PlantUML files | Same classes and associations | 🟢 PASS | 2026-10-14 | None |
| **TC-07** | Consistency with the glossary (USLP02) | Consistency | Check that every class and role of the model is defined in the glossary | All defined, same names | 🟢 PASS | 2026-10-14 | [QA-03](#-issues-found) (fixed) |

### Execution Log

* **TC-01:** the model has 22 classes and 6 enumerations in 5 packages (Retail & Mall Layout, Robotics & Infrastructure, Identity & Access, Catalog & Orders, Planning). Shopping locations, retailers, levels, elevators, areas from the GeoJSON layout, stores, products, shopping cart, orders, delivery spot and time, robot stations, control terminals, station movements, robots, sub-orders and routes are all represented.
* **TC-02:** `BUSY` robot state (`RobotStatus`); refrigerated products and robots (`Product.requiresRefrigeration`, `Robot.isRefrigerated`); each robot receives its own sub-order (`SubOrder` *—0..1* `Robot`); route planning by the system with operator input (`SystemUser` plans `Route`); oversized items can be checked at product registration (`Product.weight`, `Product.volume`).
* **TC-03:** USLP04 and USLP06 (`ShoppingLocation`, `Level`, `Area`, `Shop`); USEI01–USEI02 (`Robot`, `RobotStation`, `RobotStatus`, `batteryLevel`, `maxCapacity`); USEI03 (`Order.targetDeliveryTime`, customer through `SystemUser`); USEI04 (`maxWeightCapacity`, `maxVolumeCapacity`, `batteryLevel`, `READY`/`BUSY`); USEI05 (`SubOrder`, `isRefrigerated`).
* **TC-04:** compositions (◆) on the whole side: ShoppingLocation–Level, Level–Area, RobotStation–ControlTerminal, RobotStation–StationMovement, ShoppingCart–CartItem, Order–OrderItem, Order–SubOrder, Route–Stop; aggregations (◇): RobotStation–Robot, SubOrder–OrderItem; `Area` abstract with 5 subtypes.
* **TC-05:** the SVG has the same 22 classes, 6 enumerations and 38 links (27 associations, 5 generalizations, 6 enumeration dependencies) as the source, with the same multiplicities and composition/aggregation ends.
* **TC-06:** same 22 classes and same associations and multiplicities; the enumerations only appear in the domain model, as expected at the conceptual data level.
* **TC-07:** `Corridor`, `ShoppingCart`, `CartItem`, `StationMovement`, `Route`, `Stop` and the roles Maintenance Operator, System Planner and System Operator were not defined in the [glossary](../../Glossaries/glossary/readme.md) and `Product.description` was missing (first run). Re-check after commit `3ce06ec`: all 22 classes and 6 roles are defined, with the same names.

---

## 🐞 Issues Found

| Issue ID | Summary / Description | Severity | Status | Linked Commit / Fix | Re-check Status |
| :--- | :--- | :--- | :--- | :--- | :---: |
| **QA-01** | **Robot information screen and status light** (Section 4.2: status and next destination) are not modelled. Acceptable for now: the client said on 2026-10-12 that the LEDs are not yet defined. Revisit when ARQCP starts (Sprint 2). | Low | Open | | ⏳ Pending |
| **QA-02** | GeoJSON `indoor: room` areas (Section 4.1) have no explicit subtype of `Area`; elevators are modelled as `Elevator`, not as an area. Confirm that rooms are represented as `Amenity` or add a subtype. | Low | Open | | ⏳ Pending |
| **QA-03** | The glossary (USLP02) is not updated to V2 (see TC-07). Fixed in USLP02 (issue #2). | Medium | Closed | `3ce06ec` | 🟢 Passed |
| **QA-04** | The V2 SVG was drawn by the QA of this user story. The SVG was checked against the PlantUML source of the developer, but another team member should confirm TC-05. | Low | Open | | ⏳ Pending |

---

## 📊 Summary & QA Sign-Off

### Execution Results
* **Total Executed:** 7
* **Passed:** 7
* **Failed:** 0
* **Blocked:** 0

### Final QA Recommendation

> 🟢 **Approved:** the model covers the specification, the Sprint 1 user stories and the client clarifications, the PlantUML source and SVG are in sync, and the glossary is consistent with V2 (QA-03 closed). QA-01, QA-02 and QA-04 are low severity and do not block the acceptance criteria; they stay open for the next versions of the model.

* [x] **QA Sign-off Approved**
* **Signed By:** João Ferreira (QA)
* **Date:** 2026-10-14
