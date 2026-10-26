# 🧪 QA & Test Documentation: [USBD03] Conceptual Data Model

* **User Story Reference:** [USBD03: Conceptual Data Model](./readme.md)
* **QA Lead / Owner:** Francisco Marques (1212081)
* **Developer:** João Ferreira (1231752)
* **Artifacts Under Review:**
  * Review 1: `doc/DataBase/conceptual-data-model/domain-model.puml` (commit `54c61bf`), now [V1-conceptual-data-model.puml](../../DataBase/conceptual-data-model/V1-conceptual-data-model.puml)
  * Review 2: [V2-conceptual-data-model.puml](../../DataBase/conceptual-data-model/V2-conceptual-data-model.puml) and [V2-conceptual-data-model.svg](../../DataBase/conceptual-data-model/V2-conceptual-data-model.svg) (commits `e901711`, `2763b9d`, `ca8533d`, `ef5b21f`)
  * Review 3: fixes of QA-07, QA-08 and QA-09 in the developer's working copy (verified by QA on the developer's machine on 2026-10-15; push pending)
* **QA Status:** 🟢 Approved (review 3, 2026-10-15)

---

## 🎯 QA Strategy & Scope

### In-Scope
* Consistency with the domain model of USLP01 (entities and multiplicities).
* Coverage of the project specification (sections 4 and 5), the Sprint 1 user stories and the client clarifications.
* Conceptual-level notation (relationships named, with direction and multiplicities).
* PlantUML source and SVG in sync.

### Out-of-Scope
* Attributes and data types (domain model, USLP01).
* Keys and constraints (relational model, USBD04).

---

## 🛠️ Review Setup

* **Review 1 (2026-10-13):** domain model `domain-model.puml` at commit `d3417ee`.
* **Review 2 (2026-10-15):** [V2-domain-model.puml](../../domain-model/V2-domain-model.puml) at commit `a393cf1`.
* **Reference clarifications:** [Requirements Q&A log](../../requirements/requirements.md) (up to 2026-10-12).

---

## 📋 Test Matrix

| Test ID | Title / Scenario | Type | Test Steps | Expected Result | Review 1 (2026-10-13) | Review 2 (2026-10-15) | Review 3 (2026-10-15) | Issues Found |
| :--- | :--- | :--- | :--- | :--- | :---: | :---: | :---: | :--- |
| **TC-01** | Entities match the domain model | Consistency | Compare the classes of both PlantUML files | Same entities, same names | 🟢 PASS | 🟢 PASS | 🟢 PASS | None |
| **TC-02** | Multiplicities match the domain model | Consistency | Compare the multiplicities of every association | Same multiplicities | 🔴 FAIL | 🟢 PASS | 🟢 PASS | [QA-01](#-issues-found) (fixed) |
| **TC-03** | Relationships are complete and named | Notation | Check every relationship of the domain model | All relationships present, with name, direction and multiplicities | 🟢 PASS | 🟢 PASS | 🟢 PASS | None |
| **TC-04** | Conceptual-level notation | Notation | Check generalization, composition, aggregation and roles | `Area` abstract with its subtypes; compositions/aggregations as in the domain model; user roles shown | 🟢 PASS | 🟢 PASS | 🟢 PASS | None |
| **TC-05** | Coverage of the specification | Coverage | Check each business concept of sections 4 and 5 | Every concept is an entity, relationship or role | 🔴 FAIL | 🟢 PASS | 🟢 PASS | [QA-02 to QA-05](#-issues-found) (fixed) |
| **TC-06** | Coverage of the Sprint 1 user stories | Coverage | Check the entities used by USLP04, USLP06, USEI01-USEI05 and USBD05 | All present | 🟢 PASS | 🟢 PASS | 🟢 PASS | None |
| **TC-07** | Consistency with client clarifications | Consistency | Check the clarifications of 2026-10-02 and 2026-10-12 | No contradiction | 🟢 PASS | 🟢 PASS | 🟢 PASS | None |
| **TC-08** | Deliverables in the repository | Deliverable | Check `.puml` and `.svg`; compare entities and relationships of the SVG with the source | Both present and in sync | 🟢 PASS | 🔴 FAIL | 🟢 PASS | [QA-06](#-issues-found) (fixed), [QA-07](#-issues-found), [QA-09](#-issues-found) |
| **TC-09** | US documentation is consistent with the model | Documentation | Compare the readme tables with the `.puml` | Same entities and relationships | — | 🔴 FAIL | 🟢 PASS | [QA-08](#-issues-found) |

### Execution Log

#### Review 1 — 2026-10-13 (commit `54c61bf`)

* **TC-01:** both files have the same 16 classes: `Amenity`, `Area`, `ControlTerminal`, `DeliverySpot`, `Elevator`, `Level`, `Order`, `OrderItem`, `Product`, `RetailerCompany`, `Robot`, `RobotStation`, `Shop`, `ShoppingLocation`, `SubOrder`, `SystemUser`.
* **TC-02:** differences between the domain model and the conceptual model:

  | Relationship | Domain model (USLP01) | Conceptual model (USBD03) |
  | :--- | :---: | :---: |
  | RetailerCompany — ShoppingLocation | 1 — `*` | 1 — `1..*` |
  | Level — Area | 1 — `1..*` | 1 — `*` |

  All other multiplicities are equal. The relational model (USBD04) follows the conceptual model.
* **TC-06:** USLP04 (`Level`, `Area`, `Shop`); USLP06 (`ShoppingLocation`, `Level`, `Area`); USEI01-USEI05 (`Robot`, `RobotStation`, `Order`, `SubOrder`, `OrderItem`, `Product`); USBD05 (all entities instantiated in Oracle).
* **TC-07:** `BUSY` robot state and refrigerated products are attributes/rules, not entities; "each robot receives its own sub-order" is supported by `SubOrder` *—0..1* `Robot`.
* **TC-08:** every entity and every multiplicity of the source appears in the SVG.

#### Review 2 — 2026-10-15 (V2: commits `e901711`, `2763b9d`, `ca8533d`, `ef5b21f`)

* **TC-01:** the V2 conceptual model and the V2 domain model have the same **22** classes (the 16 of review 1 plus `Corridor`, `StationMovement`, `ShoppingCart`, `CartItem`, `Route`, `Stop`).
* **TC-02:** the **27** associations have the same multiplicities in both models (`RetailerCompany 1 — 1..* ShoppingLocation` and `Level 1 — * Area` in both).
* **TC-05:** shopping cart (`ShoppingCart`, `CartItem`), route (`Route`, `Stop`), schedule of station movements (`StationMovement`), corridors (`Corridor`), retailer administrator (`SystemUser * — 0..1 RetailerCompany : works for`) and robot location (`Robot * — 1 ShoppingLocation`) are now modelled.
* **TC-08:** the source was renamed to `V2-conceptual-data-model.puml` (old version kept as `V1-…`), which solves QA-06. However, `V2-conceptual-data-model.svg` **is not in sync** with the source: the class `Stop` and the relationships `follows`, `at`, `sequence of`, `plans (SystemPlanner)` and `works for (RetailerAdmin)` are missing — the SVG was generated from an earlier version of the `.puml` (QA-07). The rendering of V1 is still named `conceptual-data-model.svg` (QA-09).
* **TC-09:** the readme says *"Entities (23)"*, but the model has **22** entities (QA-08).

#### Review 3 — 2026-10-15 (developer's working copy, push pending)

* **TC-08 / QA-07:** the SVG was regenerated from `V2-conceptual-data-model.puml` and now shows the class `Stop` and the relationships `follows`, `at`, `sequence of`, `plans (SystemPlanner)` and `works for (RetailerAdmin)`.
* **TC-08 / QA-09:** the V1 rendering was renamed to `V1-conceptual-data-model.svg`.
* **TC-09 / QA-08:** the readme states 22 entities.
* Verified by the QA on the developer's machine, before the push. The developer will push these fixes to the repository.

### Answers to the developer's review checklist (issue #14)

| Question | Answer (review 1) |
| :--- | :--- |
| Does the generalization of `Area` encompass all required floor space types? | **No.** Corridors (`indoor`) and elevator areas (`highway`) of the GeoJSON (Section 4.1) fit none of the 4 subtypes — see QA-05. |
| Is the composition between `Level` and `Area` appropriately scoped? | Composition is correct, but the multiplicity (`*`) differs from the domain model (`1..*`) — see QA-01. |
| Does the split from `Order` into `SubOrder` align with the dispatching rules? | **Yes**, structurally. Note: according to the client (2026-10-12), the split is driven by **robot capacity and refrigeration** (each robot gets its own sub-order; refrigerated items separated), not by shop/merchant. |
| Is the `SubOrder` → `Robot` cardinality (`*` — `0..1`) accurate? | **Yes.** A sub-order may still have no robot while pending, and a robot carries several sub-orders over time. |
| Are the links between `OrderItem`, `Shop` and `Product` correctly mapped? | **Yes.** `OrderItem — Shop` is derivable from `Product — Shop` (in V2 it was removed, keeping the relational model in 3NF). |
| Are the facility relationships properly bounded? | **Yes** for `Level — Elevator` (2..* — 1..*) and `ShoppingLocation — Level` (1 — 1..*). `RetailerCompany — ShoppingLocation` differs from the domain model — see QA-01. |
| Are user roles and operations correctly associated? | **Yes** for Customer, Merchant and MaintenanceOperator. The RetailerAdmin is not related to the retailer whose stores they define — see QA-05. |
| Any missing relations or edge cases? | **Yes:** shopping cart (QA-02), route / sequence of stops (QA-03), schedule of station movements (QA-04). |

---

## 🐞 Issues Found

| Issue ID | Summary / Description | Severity | Status | Linked Commit / Fix | Re-check Status |
| :--- | :--- | :--- | :--- | :--- | :---: |
| **QA-01** | Multiplicities of RetailerCompany—ShoppingLocation and Level—Area differ from the domain model (see TC-02). | Medium | Fixed | `e901711` (conceptual V2), domain model V2 | 🟢 Passed |
| **QA-02** | **Shopping cart** not modelled (Section 4.2). | Medium | Fixed | `e901711` | 🟢 Passed |
| **QA-03** | **Route / sequence of stops** not modelled (Section 4.2; client session of 2026-10-02). | Medium | Fixed | `e901711` | 🟢 Passed |
| **QA-04** | **Schedule of robot movements** not modelled (Section 4.2). | Low | Fixed | `e901711` | 🟢 Passed |
| **QA-05** | GeoJSON areas that are not shops, amenities, delivery spots or stations (corridors) cannot be represented; the Retailer Administrator is not related to its `RetailerCompany`. | Low | Fixed | `e901711` | 🟢 Passed |
| **QA-06** | The source of the conceptual model had the same name as the domain model (`domain-model.puml`). | Low | Fixed | `e901711` (renamed to `V1-…` / `V2-conceptual-data-model.puml`) | 🟢 Passed |
| **QA-07** | `V2-conceptual-data-model.svg` is not in sync with `V2-conceptual-data-model.puml`: class `Stop` and relationships `follows`, `at`, `sequence of`, `plans (SystemPlanner)`, `works for (RetailerAdmin)` are missing. Regenerate the SVG from the `.puml`. | Medium | Fixed | Developer's working copy (push pending) | 🟢 Passed |
| **QA-08** | The readme says *"Entities (23)"*; the model has 22 entities. | Low | Fixed | Developer's working copy (push pending) | 🟢 Passed |
| **QA-09** | The V1 rendering is still named `conceptual-data-model.svg`; rename it to `V1-conceptual-data-model.svg`, like the `.puml`. | Low | Fixed | Developer's working copy (push pending) | 🟢 Passed |

---

## 📊 Summary & QA Sign-Off

### Execution Results
| | Review 1 (2026-10-13) | Review 2 (2026-10-15) | Review 3 (2026-10-15) |
| :--- | :---: | :---: | :---: |
| Total executed | 8 | 9 | 9 |
| Passed | 6 | 7 | 9 |
| Failed | 2 | 2 | 0 |
| Blocked | 0 | 0 | 0 |

### Final QA Recommendation

> 🟢 **Approved:** the V2 model fixes all the issues of reviews 1 and 2 (QA-01 to QA-09): it is consistent with the V2 domain model, covers the entire specification, and the SVG and the readme are in sync with the source. QA-07 to QA-09 were verified by the QA on the developer's machine; the developer will push them to the repository.
>
> Note: the fixes of review 1 were proposed by QA and applied by the developer, who committed them; review 2 checks the committed artifacts.

* [x] **QA Sign-off Approved**
* **Signed By:** Francisco Marques (QA)
* **Date:** 2026-10-15
