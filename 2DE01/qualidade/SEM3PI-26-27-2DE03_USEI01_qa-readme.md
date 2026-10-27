# 🧪 QA & Test Documentation: [USEI01] Data Import and System Initialization

* **User Story Reference:** [US-USEI01: Data Import and System Initialization](./readme.md)
* **QA Lead / Owner:** *(TBD – must be someone other than the author)*
* **Target Environment:** `Local / CI (Integration Server)`
* **Build / Release Version:** *(sprint tag, e.g. `sprint1`)*
* **QA Status:** ⚪ Not Started / 🟡 In Progress / 🟢 Passed / 🔴 Blocked / 🟠 Failed

---

## 🎯 QA Strategy & Scope

### In-Scope
* Parsing and loading of robots, stations and orders (with items) from text/CSV files.
* No data loss or corruption (counts and field values match the files).
* Error handling and logging for malformed lines and missing values.
* Referential consistency (robot → station, order → items).
* Appropriate JCF collections and absence of heap-based structures (`PriorityQueue`).
* Empirical performance with medium and large datasets.

### Out-of-Scope
* Station monitoring ([USEI02]), order queue ([USEI03]), assignment ([USEI04]) and partitioning ([USEI05]).
* Database persistence (not mandatory in Sprint 1).

---

## 🛠️ Test Environment & Setup

* **App URL / Endpoint:** Console application (Java) — `mvn test` for automated tests
* **Test Accounts Used:**
  * Not applicable (no authentication)
* **Pre-conditions & Test Data:**
  * Valid datasets: small, medium and large (robots, stations, orders)
  * Custom test files: malformed lines, missing values, duplicates, unknown station, empty file
  * Java and Maven installed; build passes locally and on the integration server

---

## 📋 Test Matrix

| Test ID | Title / Scenario | Type | Pre-conditions | Test Steps | Expected Result | Status | Execution Date | Bugs Filed |
| :--- | :--- | :--- | :--- | :--- | :--- | :---: | :---: | :--- |
| **TC-01** | Import valid small dataset | Functional | Valid files | 1. Run import for robots, stations and orders | Counts of entities equal the number of data lines; fields correctly parsed | ⚪ NOT RUN | — | — |
| **TC-02** | Malformed line (wrong number of fields) | Negative | File with 1 bad line | 1. Run import | Error logged with file and line number; other lines loaded; no crash | ⚪ NOT RUN | — | — |
| **TC-03** | Missing required value | Negative | Line with empty field | 1. Run import | Error logged; line not loaded | ⚪ NOT RUN | — | — |
| **TC-04** | Invalid values (non-numeric weight, battery > 100, unknown status, invalid date) | Negative | File with invalid values | 1. Run import | Each invalid line reported and skipped | ⚪ NOT RUN | — | — |
| **TC-05** | Duplicate ids | Negative | Repeated `robotId`/`stationId`/`orderId` | 1. Run import | Duplicate reported; behavior as documented (keep first/reject) | ⚪ NOT RUN | — | — |
| **TC-06** | Robot with unknown station | Negative | Robot references non-existent station | 1. Run import | Inconsistency reported; robot not loaded (or flagged) | ⚪ NOT RUN | — | — |
| **TC-07** | Empty file / file not found | Negative | Empty file; wrong path | 1. Run import | Clear message; application does not crash | ⚪ NOT RUN | — | — |
| **TC-08** | Header and blank lines | Functional | File with header and blank lines | 1. Run import | Header and blank lines ignored; no false errors | ⚪ NOT RUN | — | — |
| **TC-09** | Orders with several items | Functional | Order with multiple item lines | 1. Run import | All items associated with the correct order | ⚪ NOT RUN | — | — |
| **TC-10** | Medium and large datasets (performance) | Performance | Medium and large files | 1. Run import several times<br>2. Record times | No data loss; time growth consistent with O(R+S+O+I); results added to report | ⚪ NOT RUN | — | — |
| **TC-11** | Code review of collections | Review | Code available | 1. Inspect collections used<br>2. Search for `PriorityQueue`/heap structures | Collections justified by access patterns; no prohibited structure | ⚪ NOT RUN | — | — |

### Execution Log

NOTE: add here evidences of the tests, such as logs or screenshots (e.g. import console output, execution times table, `mvn test` result).

---

## 🐞 Defect & Bug Tracking

| Bug ID | Summary / Description | Severity | Priority | Status | Linked Commit / Fix | Re-test Status |
| :--- | :--- | :--- | :--- | :--- | :--- | :---: |
| *(none registered yet)* | | | | | | |

---

## 🤖 Unit Tests

* **Related App:** Planning App (Java)
* **Related Code:** *(update with real package names)* `Robot`, `Station`, `Order`, `OrderItem`, `DataImporter`, `ImportReport`
* **Automated Tests:** *(update with real test classes)* `DataImporterTest`, `RobotTest`, `StationTest`, `OrderTest`
* **Execution:** `mvn test`
* **CI Build Link:** *(link to the nightly build on the integration server)*

---

## 📊 Summary & QA Sign-Off

### Execution Results
* **Total Executed:** 0
* **Passed:** 0
* **Failed:** 0
* **Blocked:** 0

### Final QA Recommendation

> ⏳ **Pending:** QA not executed yet. Update this section after running the test matrix.

* [ ] **QA Sign-off Approved**
* **Signed By:** *(name, role)*
* **Date:** *(pending)*