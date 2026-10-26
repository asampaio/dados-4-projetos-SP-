# QA & Test Execution Report: USLP04

## USLP04 — Store Listing by Floor via JDBC

* **User Story Reference:** [USLP04: Store Listing by Floor via JDBC](./readme.md)
* **Issue:** [#4 — USLP04 Store Listing by Floor via JDBC](https://github.com/Departamento-de-Engenharia-Informatica/SEM3PI-26-27-2DE02/issues/4)
* **QA Lead / Owner:** Tiago Marquez
* **Implementation Area:** Shopping application
* **Execution Date:** 2026-10-15
* **Environment:** Java 21, Maven, JUnit 5, JDBC and H2 configuration
* **QA Status:** 🟡 Passed at unit/component level; end-to-end database verification pending

---

## 🎯 QA Strategy & Scope

### In-Scope

* Listing the shopping levels available to the customer.
* Selecting a level through the console interface.
* Listing the stores associated with the selected level.
* Handling a level with no stores.
* Validating controller dependencies and location input.
* Verifying the JDBC/DAO implementation by static inspection.
* Confirming that the automated USLP04 tests execute successfully.

### Out-of-Scope

* GeoJSON import and population of the database, covered by USLP06.
* Oracle FreeSQL execution.
* End-to-end execution against a populated production-like database.
* Persistence of shopping data, which belongs to USBD04/USBD05.
* Future product browsing and purchase workflows.

---

## 🛠️ Review Setup

* **Specification:** Project Specification v1, Section 7.1.1.
* **User story requirements:** [USLP04 readme](./readme.md).
* **Implementation under review:**
  * `ListStoresController`
  * `ListStoresUI`
  * `LevelDao` / `JdbcLevelDao`
  * `StoreDao` / `JdbcStoreDao`
  * `ShoppingLevel`
  * `Store`
* **Automated tests:**
  * `ListStoresControllerTest`
  * `ListStoresUITest`

---

## ✅ Acceptance Criteria

* **AC1 — List and select levels:** the application lists the shopping levels and allows the customer to select one.
* **AC2 — Display store names:** the application displays the stores located on the selected level.
* **AC3 — Database access:** the application retrieves the information through JDBC and the DAO pattern.

---

## 📋 Test Matrix

| Test ID | Title / Scenario | Type | Test Steps | Expected Result | Status | Execution Date | Issues Found |
| :--- | :--- | :--- | :--- | :--- | :---: | :---: | :--- |
| **TC-01** | Retrieve available shopping levels | Unit | Call `ListStoresController.getShoppingLevels()` using a test DAO | The available levels are returned with their numbers and names | 🟢 PASS | 2026-10-15 | None |
| **TC-02** | List stores for a selected level | Unit | Call `getStoresByLevel("NS01", 1)` | Stores associated with level 1 are returned | 🟢 PASS | 2026-10-15 | None |
| **TC-03** | Select a level with no stores | Unit | Call `getStoresByLevel("NS01", 0)` | An empty list is returned without an exception | 🟢 PASS | 2026-10-15 | None |
| **TC-04** | Reject an invalid location identifier | Unit | Call `getStoresByLevel` with `null` and blank location IDs | `IllegalArgumentException` is raised for both invalid inputs | 🟢 PASS | 2026-10-15 | None |
| **TC-05** | Validate controller dependencies | Unit | Construct the controller with a null `LevelDao` and a null `StoreDao` | Invalid dependencies are rejected | 🟢 PASS | 2026-10-15 | None |
| **TC-06** | Successful console flow | UI/component | Run `ListStoresUI` with input `1` and test DAOs | Available levels and the selected store names are displayed | 🟢 PASS | 2026-10-15 | None |
| **TC-07** | Cancel level selection | UI/component | Run `ListStoresUI` with an empty input line | The level list is shown and the operation ends safely | 🟢 PASS | 2026-10-15 | None |
| **TC-08** | JDBC level retrieval | Static review | Inspect `JdbcLevelDao.findAll` and `findByLocation` | Parameterized JDBC queries retrieve and order levels | 🟢 PASS | 2026-10-15 | None |
| **TC-09** | JDBC store retrieval by level | Static review | Inspect `JdbcStoreDao.findByLevel` | Parameterized JDBC query joins `SHOP` and `AREA` and filters by location/level | 🟢 PASS | 2026-10-15 | None |
| **TC-10** | End-to-end retrieval from seeded database | Integration | Execute the feature against a populated H2/Oracle database | Real levels and stores are displayed from persisted data | ⚪ NOT EXECUTED | — | QA-01 |

---

## 🧾 Execution Log

### TC-01 to TC-07 — Automated controller and UI tests

Command executed:

```text
mvn -B -Dtest=ListStoresControllerTest,ListStoresUITest test
```

Result:

```text
Tests run: 6, Failures: 0, Errors: 0, Skipped: 0
BUILD SUCCESS
```

The automated tests verified:

* retrieval of two shopping levels;
* retrieval of `Zara` and `Fnac` for level 1;
* an empty result for a level without stores;
* rejection of null and blank location IDs;
* rejection of null DAO dependencies;
* successful console output;
* safe cancellation with an empty input.

### TC-08 — JDBC level DAO review

`JdbcLevelDao` was reviewed and confirmed to:

* use `PreparedStatement`;
* retrieve levels from `SHOPPING_LEVEL`;
* support retrieval of all levels and levels for one location;
* order levels by location and level number;
* map database columns into `ShoppingLevel`.

### TC-09 — JDBC store DAO review

`JdbcStoreDao` was reviewed and confirmed to:

* use `PreparedStatement`;
* join `SHOP` and `AREA`;
* filter by `location_id` and `level_number`;
* order stores by name;
* map the database open/closed value into the `Store` domain object.

### TC-10 — Integration verification

An end-to-end execution against a populated database was not performed during this QA run. The automated tests use test doubles for the DAO interfaces and therefore do not prove that the configured database contains compatible seeded data.

---

## 🔍 Traceability to Acceptance Criteria

| Acceptance Criterion | Implementation Evidence | Test Evidence |
| :--- | :--- | :--- |
| **AC1 — List and select levels** | `ListStoresController.getShoppingLevels()` and `ListStoresUI.run()` / `displayLevels()` | TC-01, TC-06, TC-07 |
| **AC2 — Display store names** | `ListStoresController.getStoresByLevel()` and `ListStoresUI.displayStores()` | TC-02, TC-03, TC-06 |
| **AC3 — Access data through JDBC/DAO** | `LevelDao`, `StoreDao`, `JdbcLevelDao` and `JdbcStoreDao` | TC-08, TC-09; TC-10 pending |

---

## 🐞 Issues Found

| Issue ID | Summary / Description | Severity | Status | Recommended Action |
| :--- | :--- | :--- | :--- | :--- |
| **QA-01** | No end-to-end integration test currently executes the complete path from a populated database through JDBC, controller and console UI. | Medium | Open | Add an H2 integration test using the project schema and deterministic seed data. |
| **QA-02** | The functional scenarios in `readme.md` mention specific user-facing messages that are not all asserted by automated tests. | Low | Open | Add UI tests for invalid numeric input and non-existent level selection. |

No failures were found in the automated tests executed for this report.

---

## 📊 Summary & QA Sign-Off

### Execution Results

* **Total test cases:** 10
* **Executed:** 9
* **Passed:** 9
* **Failed:** 0
* **Blocked:** 0
* **Not executed:** 1
* **Open QA issues:** 2

### Final QA Recommendation

> 🟡 **Conditionally approved:** the controller and console UI satisfy the acceptance criteria at unit/component level, and the JDBC DAO implementations follow the expected access pattern. Final approval should wait for TC-10, an end-to-end test against a populated database.

* [ ] **QA Sign-off Approved**
* **Signed By:** Tiago Marquez
* **Date:** 2026-10-15
