# 🧪 QA & Test Documentation: [USBD04] Relational Model (Logical Level)

* **User Story Reference:** [USBD04: Relational Model (Logical Level)](./readme.md) — GitHub issue #15
* **QA Lead / Owner:** Tiago Borges (1231109)
* **Developer:** Tiago Marquez (1221508)
* **Artifacts Under Review:** [ER diagram](../../DataBase/Relational-Model/USBD04_ShopRobot_Relational_Model.puml) ([svg](../../DataBase/Relational-Model/USBD04_ShopRobot_Relational_Model.svg)), [US readme](./readme.md), [01_create_tables.sql](../../DataBase/scripts/01_create_tables.sql), [04_constraint_tests.sql](../../DataBase/scripts/04_constraint_tests.sql)
* **Reviewed Versions:**
  * Review 1: commit `d3417ee` (2026-10-13)
  * Review 2: commit `91e9660` (2026-10-15) — fixes in `7899544` (DDL), `e06680e` (constraint tests), `73904c5` (diagram and readme) and the V2 domain / conceptual models
* **QA Status:** 🟢 Approved (review 2, 2026-10-15)

---

## 🎯 QA Strategy & Scope

### In-Scope
* Coverage of the domain model (USLP01), the conceptual model (USBD03) and the requirements document (USBD01).
* Normalization (3NF) of every table.
* Correctness of keys, foreign keys, `ON DELETE` actions and `CHECK` constraints.
* Consistency with the data dictionary (USBD02) and with the user stories that will use the database (USLP04, USLP06, USEI01-05).
* Completeness of the US documentation.

### Out-of-Scope
* Execution of the scripts on Oracle FreeSQL and the test data — validated in USBD05.

---

## 🛠️ Review Setup

* **References:** [domain model](../../domain-model/V2-domain-model.puml), [conceptual model](../../DataBase/conceptual-data-model/V2-conceptual-data-model.puml), [data dictionary](../../Glossaries/DBglossary/readme.md), [requirements document](../../requirements/requirements.md).
* **Method:** static review of the diagram, the readme and the DDL; the scripts were not run on Oracle in either review.

---

## 📋 Test Matrix

| Test ID | Title / Scenario | Type | Test Steps | Expected Result | Review 1 (2026-10-13) | Review 2 (2026-10-15) | Issues Found |
| :--- | :--- | :--- | :--- | :--- | :---: | :---: | :--- |
| **TC-01** | Every domain model class is mapped | Coverage | Compare the classes of the domain model with the tables | One table per class (+ association tables) | 🟢 PASS | 🟢 PASS | None (review 2: 22 classes + `ELEVATOR_LEVEL` = 23 tables) |
| **TC-02** | Every attribute and enumeration is mapped | Coverage | Compare attributes and enums with columns and `CHECK IN` | All mapped with consistent names | 🟢 PASS | 🟢 PASS | None |
| **TC-03** | Requirements are covered (AC1) | Coverage | Check the spec, user stories and clarifications against the model | Every requirement has a table/column or a documented rule | 🟠 FAIL | 🟢 PASS | [QA-04](#-defect--bug-tracking) (fixed) |
| **TC-04** | Every table is in 3NF | Normalization | Check partial and transitive dependencies of each table | No partial or transitive dependency | 🔴 FAIL | 🟢 PASS | [QA-01](#-defect--bug-tracking) (fixed) |
| **TC-05** | Multiplicities are reflected | Consistency | Compare multiplicities with `NOT NULL`, `UNIQUE`, association tables and rules | Every multiplicity mapped as in the domain model | 🟠 FAIL | 🟢 PASS | [QA-07](#-defect--bug-tracking) (fixed) |
| **TC-06** | `ON DELETE` actions follow the mapping rules | Correctness | Check every FK against composition / aggregation | Cascade only on compositions; no action conflicts with a `CHECK` | 🔴 FAIL | 🟢 PASS | [QA-02](#-defect--bug-tracking), [QA-06](#-defect--bug-tracking) (fixed) |
| **TC-07** | Constraints come from the client or are marked as assumptions | Traceability | Check each `CHECK` / unique index against the spec and the domain model | No undocumented assumption | 🟠 FAIL | 🟢 PASS | [QA-05](#-defect--bug-tracking) (fixed) |
| **TC-08** | The model supports the user stories that use it | Integration | Check the data needed by USLP06 (GeoJSON import) | Imported data can be inserted | 🔴 FAIL | 🟢 PASS | [QA-03](#-defect--bug-tracking) (fixed) |
| **TC-09** | Consistency with the data dictionary (USBD02) | Consistency | Compare mandatory columns and allowed values with the dictionary | Same rules in both documents | 🟠 FAIL | 🟢 PASS | [QA-08](#-defect--bug-tracking) (fixed) |
| **TC-10** | IDs are fixed and deterministic | Requirements | Check the PKs (clarification of 2026-10-09) | No auto-generated IDs | 🟢 PASS | 🟢 PASS | None |
| **TC-11** | Documentation is complete | Documentation | Check the readme sections and links | All sections filled, no broken links | 🟠 FAIL | 🟢 PASS | [QA-09](#-defect--bug-tracking) (fixed) |
| **TC-12** | The DDL runs on Oracle FreeSQL and the constraint tests pass (AC3) | Execution | Run `01_create_tables.sql`, `02_insert_data.sql` and `04_constraint_tests.sql` | No errors; `Passed: 47   Failed: 0` | ⏳ Pending | ➖ Out of scope | [QA-10](#-defect--bug-tracking) (validated in USBD05) |

### Execution Log

#### Review 1 — 2026-10-13 (commit `d3417ee`)

* Static review of the diagram, the readme and the DDL; findings QA-01 to QA-10 recorded in the table below.

#### Review 2 — 2026-10-15 (commit `91e9660`)

* **TC-01:** the diagram and the DDL have the same **23** tables (the 17 of review 1 plus `CORRIDOR`, `STATION_MOVEMENT`, `SHOPPING_CART`, `CART_ITEM`, `ROUTE`, `STOP`); the SVG shows all 23 and is in sync with the `.puml`.
* **TC-03 / QA-04:** `STATION_MOVEMENT` covers the schedule of robot movements and `ROBOT.location_id` the robot ↔ shopping location; shops that joined ShopRobot are the rows of `SHOP`; the delivery time window of USEI03 is documented as a query filter on `target_delivery_time`.
* **TC-04 / QA-01:** `ORDER_ITEM.shop_area_id` and `uk_product_id_shop` were removed; the shop of an item is obtained through its product. The redundant association `OrderItem — Shop` is no longer in the domain and conceptual models. All 23 tables are in 3NF.
* **TC-05 / QA-07:** domain model, conceptual model and relational model use `RetailerCompany 1 — 1..* ShoppingLocation` and `Level 1 — * Area`.
* **TC-06 / QA-02:** `fk_order_item_sub_order` has no `ON DELETE CASCADE`; test T37 checks that a sub-order with items cannot be deleted.
* **TC-06 / QA-06:** the behaviour is now explicit (readme and DDL comment): a station with `READY`/`CHARGING` robots cannot be deleted; its robots must be moved first.
* **TC-07 / QA-05:** `ux_sub_order_active_robot` is kept, justified by USEI04 and marked *(to confirm)* in the readme and the DDL.
* **TC-08 / QA-03:** `SHOP.shop_code` and `SHOP.merchant_id` are optional (`SystemUser 0..1 — * Shop` in all models); positive tests P02 and P03 insert an imported shop without merchant and shop code.
* **TC-09 / QA-08:** `floor_name`, `ROBOT.model` and `ROBOT_STATION.name` are optional, `ip_address` accepts IPv4 and IPv6 (`VARCHAR2(45)`, test P01), and `unit_price_at_purchase > 0`, as in the data dictionary.
* **TC-11 / QA-09:** every relative link of the readme resolves; the DDL header points to `doc/UserStories/USBD04/readme.md`.
* **TC-12 / QA-10:** the execution of the scripts on Oracle FreeSQL belongs to USBD05 (AC3 of this US: "validated in USBD05"), so it is tracked there and is not part of this review.

---

## 🐞 Defect & Bug Tracking

| Bug ID | Summary / Description | Severity | Priority | Status | Linked Commit / Fix | Re-test Status |
| :--- | :--- | :--- | :--- | :--- | :--- | :---: |
| **QA-01** | `ORDER_ITEM` is not in 3NF (not even 2NF): `shop_area_id` depends only on `product_id`, part of the key `(order_id, product_id)`. Remove the column (and `uk_product_id_shop`) or document it as an intentional denormalisation. | High | High | Fixed | `7899544`, `73904c5` | 🟢 Passed |
| **QA-02** | `fk_order_item_sub_order ... ON DELETE CASCADE` on an aggregation (`SubOrder o-- OrderItem`): deleting a sub-order deletes the order's items. Use a FK without cascade. | High | High | Fixed | `7899544` (test T37 in `e06680e`) | 🟢 Passed |
| **QA-03** | `SHOP.merchant_id` and `shop_code` are `NOT NULL`, but the GeoJSON file (USLP06) has neither, so imported shops cannot be inserted. Decide between `0..1` (domain model + relational) or requiring a merchant during the import. | High | High | Fixed | `7899544`, `73904c5` (tests P02, P03) | 🟢 Passed |
| **QA-04** | Requirements with no table/column (gaps in the domain model): delivery time window (USEI03), schedule of robot movements (spec 4.2), shops that joined ShopRobot, robot ↔ shopping location. To be added to the domain model first (USLP01). | Medium | Medium | Fixed | `7899544`, `73904c5`, V2 domain model | 🟢 Passed |
| **QA-05** | `ux_sub_order_active_robot` (a robot has at most one active sub-order) is not in the spec nor in the domain model (`SubOrder * -- 0..1 Robot`). Mark it as *to confirm* or remove it. | Medium | Medium | Fixed | `7899544`, `73904c5` (marked *to confirm*) | 🟢 Passed |
| **QA-06** | `fk_robot_station ON DELETE SET NULL` conflicts with `ck_robot_status_station`: deleting a station with `READY`/`CHARGING` robots fails. Choose an explicit behaviour. | Medium | Low | Fixed | `7899544`, `73904c5` (behaviour documented) | 🟢 Passed |
| **QA-07** | Multiplicities differ: USBD03 and the diagram use `RetailerCompany 1..* ShoppingLocation` and `Level * Area`; the domain model (USLP01) uses `*` and `1..*`. | Low | Medium | Fixed | V2 domain model | 🟢 Passed |
| **QA-08** | Differences from the data dictionary: `floor_name`, `robot.model`, `robot_station.name` mandatory here and optional there; IP only IPv4 (`VARCHAR2(15)`) vs IPv4/IPv6; `unit_price_at_purchase >= 0` vs product price `> 0`. | Low | Medium | Fixed | `7899544`, `73904c5` (test P01) | 🟢 Passed |
| **QA-09** | Broken links in the readme (`../USBD05/readme.md`, `./qa-readme.md` — the latter fixed by this file); the DDL header points to the old path `doc/USBD05/readme.md`; DoD not ticked. | Low | Low | Fixed | `7899544`, `73904c5` | 🟢 Passed |
| **QA-10** | No evidence of the Oracle FreeSQL run (DDL + constraint tests). Add a screenshot or the output. | Medium | Medium | Moved to USBD05 | [USBD05](../USBD05/readme.md) | ➖ N/A |

---

## 🤖 Automated Tests

* **Related Scripts:** [01_create_tables.sql](../../DataBase/scripts/01_create_tables.sql), [04_constraint_tests.sql](../../DataBase/scripts/04_constraint_tests.sql)
* **Automated Tests:** 47 tests — 43 negative (`expect_error`), each breaking one constraint, and 4 positive (`expect_ok`), each checking a statement that must be accepted (IPv6 address, imported shop, cascade of an order).
* **Execution:** Oracle FreeSQL, after `02_insert_data.sql`; results in the `DBMS_OUTPUT` panel.

---

## 📊 Summary & QA Sign-Off

### Execution Results
| | Review 1 (2026-10-13) | Review 2 (2026-10-15) |
| :--- | :---: | :---: |
| Total executed | 11 | 11 |
| Passed | 3 | 11 |
| Failed | 8 | 0 |
| Pending / out of scope | 1 (TC-12) | 1 (TC-12, USBD05) |

### Final QA Recommendation

> 🟢 **Approved:** the issues of review 1 (QA-01 to QA-09) are fixed in the repository — the model is in 3NF, cascades follow the mapping rules, imported shops can be inserted and the diagram, the readme and the DDL are consistent. The execution on Oracle FreeSQL (QA-10, TC-12) is validated in USBD05.

* [x] **QA Sign-off Approved**
* **Signed By:** Tiago Borges (QA)
* **Date:** 2026-10-15
