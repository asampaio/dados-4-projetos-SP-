# 🧪 QA & Test Documentation: [USBD05] Instantiate the Relational Model in Oracle

* **User Story Reference:** [USBD05: Instantiate the Relational Model in Oracle](readme.md)
* **QA Lead / Owner:** TBD — assign a reviewer other than the author (Hugo Coutinho), per section 3.4 of the requirements document
* **Target Environment:** `Oracle FreeSQL`
* **Build / Release Version:** Sprint 1 (tests run on `database/ddl.sql` v1 and `database/seed.sql` v1; version 2 of both scripts and version 3 of the seed run again, see the Execution Log)
* **QA Status:** 🟡 Executed (12 / 12 tests passed on Oracle FreeSQL, 2026-10-20 and 2026-10-21); review and sign-off by another team member pending

---

## 🎯 QA Strategy & Scope

### In-Scope
* DDL script correctness — all tables created without errors.
* Constraint enforcement — PK, UK, FK, CHECK and NOT NULL constraints active.
* Data completeness — all tables populated with meaningful seed data.
* Requirements verifiability — seed data allows key business scenarios to be queried.
* Price history — a price change does not affect previous transactions.
* Oracle FreeSQL demonstration.

### Out-of-Scope
* PL/SQL API functions and triggers (future stories). The business rules of the model that need them (R1 to R10, R16 and R17) are not enforced by the DDL, so they are not tested here; the seed data respects them.
* Java JDBC integration (USLP03/USLP04).
* Performance benchmarking.

---

## 🛠️ Test Environment & Setup

* **Tooling:** Oracle FreeSQL (SQL Worksheet).
* **Pre-conditions:** the USBD04 relational model (version 2) is available and the team confirmed the approach to generate the DDL (LLM, manual review and execution on Oracle FreeSQL).
* **Approach:** run the DDL on a clean schema (Run Script), verify with SQL queries, run the seed data (Run Script), re-verify. Both scripts can be run again, because the DDL drops the tables and the seed empties them first.
* **Scripts:** `database/ddl.sql` and `database/seed.sql`.
* **Credentials:** must not be committed to the repository — use a `.env` file or an environment variable (excluded via `.gitignore`).

---

## 📋 Validation Matrix

| Test ID   | Title / Scenario                      | Type          | Steps                                                                                                                         | Expected Result                                                               |  Status   |  Executed By  | Notes                                                                                                                                                               |
|:----------|:--------------------------------------|:--------------|:------------------------------------------------------------------------------------------------------------------------------|:------------------------------------------------------------------------------|:---------:|:-------------:|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **TC-01** | DDL executes without errors           | Functional    | Run the full DDL script on a clean schema                                                                                     | All tables created; no ORA- errors                                            | ✅ Passed | Hugo Coutinho | No `ORA-` line in the Script Output                                                                                                                                 |
| **TC-02** | All USBD04 tables exist               | Completeness  | `SELECT table_name FROM user_tables ORDER BY 1`                                                                               | Every table in the relational model appears                                   | ✅ Passed | Hugo Coutinho | 16 tables listed                                                                                                                                                    |
| **TC-03** | All constraints present and enabled   | Correctness   | Count the named constraints in `user_constraints` (name not starting with `SYS`); list the ones whose status is not `ENABLED` | All named constraints present; none disabled                                  | ✅ Passed | Hugo Coutinho | 69 named constraints (PK, UK, FK and CHECK; `NOT NULL` constraints are inline and named by Oracle, so they are not counted); no constraint different from `ENABLED` |
| **TC-04** | All tables have seed data             | Completeness  | `SELECT COUNT(*)` for every table (one `UNION ALL` query)                                                                     | Every table has ≥ 1 row                                                       | ✅ Passed | Hugo Coutinho | 16 tables, counts as expected (see Execution Log); screenshots re-captured with seed version 3                                                                      |
| **TC-05** | Robots of a station sorted by battery | Functional    | Query robots parked in a station, ordered by battery DESC                                                                     | Correct robots, correct descending order                                      | ✅ Passed | Hugo Coutinho | `R-001` (90), `R-003` (80), `R-002` (35)                                                                                                                            |
| **TC-06** | Stores of a level                     | Functional    | Query stores located in a given level                                                                                         | Only stores of that level, by name                                            | ✅ Passed | Hugo Coutinho | Demo Fashion, Demo Fresh Market, Demo Sports                                                                                                                        |
| **TC-07** | Pending orders of a customer          | Functional    | Query orders with status `PENDING` for a customer                                                                             | Only that customer's pending orders                                           | ✅ Passed | Hugo Coutinho | Customer `C-0003`: order `O-0004`                                                                                                                                   |
| **TC-08** | PK constraint enforced                | Negative      | Insert a robot with a duplicate ID                                                                                            | Oracle raises ORA-00001 (unique constraint violated)                          | ✅ Passed | Hugo Coutinho | `PK_ROBOT` violated                                                                                                                                                 |
| **TC-09** | FK constraint enforced                | Negative      | Insert an order line referencing a non-existent product                                                                       | Oracle raises ORA-02291 (integrity constraint violated)                       | ✅ Passed | Hugo Coutinho | `FK_ORDER_LINE_PRODUCT` violated - parent key not found                                                                                                             |
| **TC-10** | NOT NULL constraint enforced          | Negative      | Insert a robot with NULL battery level                                                                                        | Oracle raises ORA-01400 (cannot insert NULL)                                  | ✅ Passed | Hugo Coutinho | Column `BATTERY_LEVEL`                                                                                                                                              |
| **TC-11** | Oracle FreeSQL demonstration          | Acceptance    | Run TC-04 to TC-07 live on Oracle FreeSQL; record results                                                                     | All queries return expected results; no errors                                | ✅ Passed | Hugo Coutinho | Evidence in the `evidence` folder                                                                                                                                   |
| **TC-12** | Price change keeps previous orders    | Business rule | Change the current price of a product that is in an existing order; query that order and the price history                    | The previous order keeps the price paid; the old price remains in the history | ✅ Passed | Hugo Coutinho | Rule of the BDDAD teacher                                                                                                                                           |

### Execution Log

**Dates:** 2026-10-20 and 2026-10-21 · **Environment:** Oracle FreeSQL (SQL Worksheet) · **Executed by:** Hugo Coutinho

The tests were run on 2026-10-20 with version 1 of the scripts. On 2026-10-21 the DDL and the seed were run again to capture the evidence of TC-01, TC-02 and TC-04, and TC-09 was run and captured. Version 2 of the scripts (new header and fixed indentation, same tables, constraints and data) was then run and checked again. Finally, the seed was corrected to respect rule R17 (a store without a merchant has no products) and version 3 was run and checked (steps 20 and 21).

| Step | Script / Query | Result |
|:-----|:---------------|:-------|
| 1 | `database/ddl.sql` (Run Script) on the schema | No `ORA-` errors (TC-01) |
| 2 | `SELECT table_name FROM user_tables ORDER BY 1` | 16 tables (TC-02) |
| 3 | `SELECT COUNT(*) FROM user_constraints WHERE constraint_name NOT LIKE 'SYS%'` | 69 (TC-03) |
| 4 | `SELECT constraint_name, status FROM user_constraints WHERE status <> 'ENABLED'` | No items to display (TC-03) |
| 5 | `database/seed.sql` (Run Script) | No `ORA-` errors; ends with `Commit complete.` |
| 6 | `SELECT COUNT(*)` of the 16 tables (one `UNION ALL` query), seed versions 1 and 2 | RETAILER 2, SHOPPING_LOCATION 2, SHOPPING_LEVEL 5, AREA 15, AREA_VERTEX 60, MERCHANT 4, STORE 7, PRODUCT 11, PRODUCT_PRICE 12, CUSTOMER 3, ROBOT_STATION 4, ROBOT 8, CUSTOMER_ORDER 5, SUB_ORDER 2, ORDER_LINE 10, ROUTE 4 (TC-04) |
| 7 | Robots of station `RS-NS-L0` by battery DESC | `R-001` 90, `R-003` 80, `R-002` 35 (TC-05) |
| 8 | Stores of level 2 (level 0 of NorteShopping) | Demo Fashion, Demo Fresh Market, Demo Sports (TC-06) |
| 9 | `PENDING` orders of customer `C-0003` | `O-0004` (TC-07) |
| 10 | `INSERT` of a robot with the existing id `R-001` | `ORA-00001: unique constraint (PK_ROBOT) violated` (TC-08) |
| 11 | `INSERT` of an order line with product `P-NAO-EXISTE` | `ORA-02291: integrity constraint (FK_ORDER_LINE_PRODUCT) violated - parent key not found` (TC-09) |
| 12 | `INSERT` of a robot with `battery_level` NULL | `ORA-01400: cannot insert NULL into ("ROBOT"."BATTERY_LEVEL")` (TC-10) |
| 13 | Line 1 of order `O-0001` before the price change | `unit_price` = 15 (TC-12) |
| 14 | `INSERT` of a new price 19.00 for `P-FASH-001` (valid from 2026-10-20) | 1 row inserted (TC-12). Running the same `INSERT` again raised `ORA-00001` on `PK_PRODUCT_PRICE`, since the key is `(product_id, valid_from)` |
| 15 | Line 1 of order `O-0001` after the price change | `unit_price` still 15 (TC-12) |
| 16 | Price history of `P-FASH-001` | 15, 17.5 and 19, in this order (TC-12) |
| 17 | `database/seed.sql` (Run Script) again | The test data of TC-12 is removed and the seed data is restored |
| 18 | `database/ddl.sql` version 2 (Run Script), then `database/seed.sql` version 2 (Run Script) | No `ORA-` errors in either script |
| 19 | `SELECT COUNT(*)` of the 16 tables after version 2 | Same counts as in step 6 |
| 20 | `database/ddl.sql` version 2 (Run Script), then `database/seed.sql` version 3 (Run Script) | No `ORA-` errors in either script; the seed ends with `Commit complete.` (TC-04) |
| 21 | `SELECT COUNT(*)` of the 16 tables after seed version 3 | RETAILER 2, SHOPPING_LOCATION 2, SHOPPING_LEVEL 5, AREA 16, AREA_VERTEX 64, MERCHANT 5, STORE 8, PRODUCT 11, PRODUCT_PRICE 12, CUSTOMER 3, ROBOT_STATION 4, ROBOT 8, CUSTOMER_ORDER 5, SUB_ORDER 2, ORDER_LINE 10, ROUTE 4. Only AREA, AREA_VERTEX, MERCHANT and STORE changed (TC-04) |

### Evidence

Screenshots of the runs, in the [evidence](./evidence/) folder. The two screenshots of TC-04 were captured again with seed version 3 (the first capture of the counts showed the list of tables instead of the counts):

| Test | Screenshot |
|:-----|:-----------|
| TC-01 | [TC-01-ddl-run.png](./evidence/TC-01-ddl-run.png) |
| TC-02 | [TC-02-tables.png](./evidence/TC-02-tables.png) |
| TC-03 | [TC-03-constraints-count.png](./evidence/TC-03-constraints-count.png), [TC-03-constraints-enabled.png](./evidence/TC-03-constraints-enabled.png) |
| TC-04 | [TC-04-seed-run-end.png](./evidence/TC-04-seed-run-end.png), [TC-04-counts.png](./evidence/TC-04-counts.png) |
| TC-05 | [TC-05-robots-by-battery.png](./evidence/TC-05-robots-by-battery.png) |
| TC-06 | [TC-06-stores-of-level.png](./evidence/TC-06-stores-of-level.png) |
| TC-07 | [TC-07-pending-orders.png](./evidence/TC-07-pending-orders.png) |
| TC-08 | [TC-08-error-pk.png](./evidence/TC-08-error-pk.png) |
| TC-09 | [TC-09-error-fk.png](./evidence/TC-09-error-fk.png) |
| TC-10 | [TC-10-error-not-null.png](./evidence/TC-10-error-not-null.png) |
| TC-12 | [step 1](./evidence/TC-12-step1-price-before.png), [step 2](./evidence/TC-12-step2-insert-price.png), [step 3](./evidence/TC-12-step3-order-after.png), [step 4](./evidence/TC-12-step4-history.png) |

### Notes

* The postal code of NorteShopping in the seed data is `A CONFIRMAR` (not verified); the other seed data is fictional and labelled as such in the script.
* The units used in the seed data (kg, litres, euros, metres and seconds) are an assumption of the team.
* Seed version 3 fixes rule R17 (a store without a merchant has no products): the store `Demo Gadgets` now has a merchant, and the store without a merchant is the new `Demo Corner`, with no products. The data used by TC-05, TC-06, TC-07 and TC-12 is not changed by version 3.

---

## 🐞 Defect & Bug Tracking

| Issue ID | Summary / Description | Severity | Priority | Status | Linked Commit / Fix | Re-test Status |
|:---------|:----------------------|:---------|:---------|:-------|:--------------------|:--------------:|
| #117     | Seed version 2 broke rule R17 (a store without a merchant sold a product); found in the review of the seed data | Medium | Medium | Fixed | `seed.sql` version 3 (#117) | ✅ Passed (steps 20 and 21) |

---

## 🤖 Unit Tests

Not applicable — this story produces SQL scripts and a database instantiation, not Java code. Validation is done through SQL queries on Oracle FreeSQL (see Validation Matrix above).

---

## 📊 Summary & QA Sign-Off

### Execution Results
* **Total Tests:** 12 / 12
* **Passed:** 12
* **Failed:** 0
* **Not run:** 0

### Final QA Recommendation

> 🟡 **Executed, pending review:** All 12 validation checks passed on Oracle FreeSQL (DDL v1 and seed v1), with the evidence in the `evidence` folder. Version 2 of both scripts and version 3 of the seed (rule R17 fixed) ran again without errors, and the counts of version 3 were checked. The sign-off depends on the review by a team member other than the author.

* [ ] **QA Sign-off Approved**
* **Signed By:** TBD
* **Date:** TBD