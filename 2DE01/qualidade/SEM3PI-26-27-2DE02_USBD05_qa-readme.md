# 🧪 QA & Test Documentation: [USBD05] Physical Instantiation and Data Population in Oracle SQL

* **User Story Reference:** [USBD05: Physical Instantiation and Data Population in Oracle SQL](./readme.md) — GitHub issue #16
* **QA Lead / Owner:** Tiago Marquez
* **Developer:** Francisco Marques
* **Artifacts Under Test:** [00_drop_tables.sql](../../DataBase/scripts/00_drop_tables.sql), [01_create_tables.sql](../../DataBase/scripts/01_create_tables.sql), [02_insert_data.sql](../../DataBase/scripts/02_insert_data.sql), [03_verification_queries.sql](../../DataBase/scripts/03_verification_queries.sql), [04_constraint_tests.sql](../../DataBase/scripts/04_constraint_tests.sql), [US readme](./readme.md)
* **Tested Version:** commit `f2d84a3` (2026-10-19)
* **Target Environment:** `Oracle FreeSQL (Oracle Database 23ai)`
* **Build / Release Version:** `Sprint 1`
* **QA Status:** 🟡 In Progress

---

## 🎯 QA Strategy & Scope

### In-Scope
* Creation of the schema from the DDL script without errors (AC3).
* Every table populated with data (AC1).
* Queries that show the model answers the Sprint 1 requirements (AC2, Part B).
* Business rules: declarative constraints (negative and positive tests) and rules checked by query (AC2, Part C).
* Edge cases documented in the US (re-run of the scripts, location without GeoJSON, deleting a station with parked robots).
* Naming conventions of NFR05.
* Consistency of the tables with the domain model and the relational model (USBD04).

### Out-of-Scope
* PL/SQL API and triggers (future sprints).
* Performance of the queries.

---

## 🛠️ Test Environment & Setup

* **Environment:** [Oracle FreeSQL](https://freesql.com), personal schema of the tester.
* **Pre-conditions:** run, as a script and in this order, `00_drop_tables.sql`, `01_create_tables.sql` and `02_insert_data.sql` from [doc/DataBase/scripts/](../../DataBase/scripts/).

---

## 📋 Test Matrix

| Test ID | Title / Scenario | AC | Type | Pre-conditions | Test Steps | Expected Result | Status | Execution Date | Bugs Filed |
| :--- | :--- | :---: | :--- | :--- | :--- | :--- | :---: | :---: | :--- |
| **TC-01** | Create schema and insert data | AC3 | Functional | Empty schema | Run `00`, `01`, `02` | No errors; 23 tables created; `COMMIT` complete | ⏳ Pending | | |
| **TC-02** | All tables populated | AC1 | Functional | TC-01 | Run Part A of `03` | 23 rows, every `num_rows` > 0 | ⏳ Pending | | |
| **TC-03** | Stores by level (USLP04) | AC2 | Functional | TC-01 | Run B3 | Levels −1, 0, 1 of NorteShopping; level 0 stores: Desporto Total, Mercearia do Norte, Moda Viva | ⏳ Pending | | |
| **TC-04** | Station status (USEI02) | AC2 | Functional | TC-01 | Run B6, B7 | `ST-NS-0` capacity 3, occupied 3, free 0; robots R001 (95), R003 (70), R002 (40) | ⏳ Pending | | |
| **TC-05** | Pending queue and orders by deadline / customer (USEI03) | AC2 | Functional | TC-01 | Run B9, B10 | B9: `O0008` (12:15) before `O0003` (13:00). B10: due within 1 h of 11:30 → `O0002` (11:30), `O0004` (12:00), `O0008` (12:15); orders of `U002` → `O0005`, `O0002`, `O0008` | ⏳ Pending | | |
| **TC-06** | Feasibility (USEI04) | AC2 | Functional | TC-01 | Run B12 | `O0003` feasible with R001 (95) then R005 (76); `O0008` NOT FEASIBLE | ⏳ Pending | | |
| **TC-07** | Partitioned orders (USEI05) | AC2 | Functional | TC-01 | Run B13 | `O0002` split in `SO0002` (R007, refrigerated) and `SO0003` (R008); also `O0004`, `O0007`, `O0008`; sub-orders of one order use distinct robots; loads within capacity | ⏳ Pending | | |
| **TC-08** | Business rules by query | AC2 | Integrity | TC-01 | Run Part C (C1-C16) of `03` | Every query returns no rows | ⏳ Pending | | |
| **TC-09** | Constraint tests | AC2 | Negative / positive | TC-01 | Run `04` with DBMS output on; then run Part A of `03` | `Passed: 47   Failed: 0` (43 negative + 4 positive); row counts equal to TC-02 (tests rolled back) | ⏳ Pending | | |
| **TC-10** | Re-run scripts | AC3 | Regression | TC-01 | Run `00`, `01`, `02` again | No errors; same row counts as TC-02 | ⏳ Pending | | |
| **TC-11** | Location without GeoJSON | — | Edge case | TC-01 | `SELECT location_id FROM SHOPPING_LOCATION WHERE geo_json_data IS NULL` | `LOC-OA` returned (optional GeoJSON accepted) | ⏳ Pending | | |
| **TC-12** | Delete a station with parked `READY` robots | — | Edge case / negative | TC-01 | `DELETE FROM ROBOT_STATION WHERE station_id = 'ST-NS-0'`; then `ROLLBACK` | Rejected with `ORA-02290` on `ck_robot_status_station`; data unchanged | ⏳ Pending | | |
| **TC-13** | Naming conventions (NFR05) | — | Review | TC-01 | Check `USER_TABLES`, `USER_CONSTRAINTS` and `USER_INDEXES` of the 23 tables | Tables in `UPPER_SNAKE_CASE`; constraints prefixed `pk_`, `uk_`, `fk_`, `ck_`, `nn_`; indexes `ux_` | ⏳ Pending | | |
| **TC-14** | Consistency with models | — | Review | USBD04 done | Compare tables with the domain model, the relational model and the mapping table of the US | Every class/association mapped | ⏳ Pending | | |

### Execution Log

NOTE: add here the evidences of the tests (DoD), such as screenshots of Oracle FreeSQL, one per test case.

---

## 🐞 Defect & Bug Tracking

| Bug ID | Summary / Description | Severity | Priority | Status | Linked Commit / Fix | Re-test Status |
| :--- | :--- | :--- | :--- | :--- | :--- | :---: |
| | | | | | | |

---

## 🤖 Unit Tests

* **Related App:** Database (Oracle)
* **Related Code:** [01_create_tables.sql](../../DataBase/scripts/01_create_tables.sql), [02_insert_data.sql](../../DataBase/scripts/02_insert_data.sql)
* **Automated Tests:** [04_constraint_tests.sql](../../DataBase/scripts/04_constraint_tests.sql) (43 negative and 4 positive tests, rolled back at the end)
* **Execution:** run on Oracle FreeSQL

---

## 📊 Summary & QA Sign-Off

### Execution Results
* **Total Executed:** 0 / 14
* **Passed:** 0
* **Failed:** 0
* **Blocked:** 0

### Acceptance Criteria
* [ ] **AC1 - All tables populated:** TC-02
* [ ] **AC2 - Data verifies the requirements:** TC-03 to TC-09
* [ ] **AC3 - Shown on Oracle FreeSQL:** TC-01, TC-10

### Final QA Recommendation

> ⏳ Pending execution.

* [ ] **QA Sign-off Approved**
* **Signed By:**
* **Date:**
