# 🧪 QA & Test Documentation: [USLP04] Import stores and levels from GeoJSON file

* **User Story Reference:** [USLP01: Import stores and levels from GeoJSON file](readme.md)
* **QA Lead / Owner:** TBD (must be a team member other than the author, Daniel Rey, per section 3.4 of the requirements document)
* **Target Environment:** `Back-office app(Java, JUnit, H2)`
* **Build / Release Version:** `Sprint 1`
* **QA Status:** ⚪ Not Started (implementation pending)

---

## 🎯 QA Strategy & Scope

### In-Scope
* Verification of GeoJSON parsing logic for shopping levels and stores.
* Verification of data insertion into Oracle DB via JDBC using the DAO pattern.
* Validation of console UI outputs displaying lists of imported levels, stores, and error logs.
* Negative testing for malformed GeoJSON files, missing properties, and invalid file paths.
* Database transaction rollback verification upon persistence or connection failure.

### Out-of-Scope
* Navigation graph generation, elevators, and autonomous robot routes (deferred to dedicated navigation USs).
* Customer front-office UI listing of stores (covered under [USLP04]).

---

## 🛠️ Test Environment & Setup

* **Endpoint:** `Back-office (app-three)`
* **Tooling:** Java+JUnit, H2 database
* **Pre-conditions:**
  * The database must be developed and instantiated
* **Test data:**
  * Valid GeoJSON file (containing 2 Levels and 12 Stores)
  * Invalid path to a GeoJSON file
  * Empty GeoJSON file
  * GeoJSON with malformed syntax
  * GeoJSON file containing Levels or Stores already existing in the database
* **Approach:** TDD. Each test is written first (Red), then the minimum code to pass (Green), then refactor.
---

## 📋 Test Matrix

| Test ID   | Title / Scenario                                      | Type                              | Pre-conditions                                                                                                                                  | Test Steps                                                                                                                                            | Expected Result                                                                                                                                                                                                                                                                                                        |     Status     | Execution Date | Bugs Filed |
|:----------|:------------------------------------------------------|:----------------------------------|:------------------------------------------------------------------------------------------------------------------------------------------------|:------------------------------------------------------------------------------------------------------------------------------------------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:--------------:|:--------------:|:-----------|
| **TC-01** | Totally valid GeoJSON                                 | Functional                        | Database must be instantiated                                                                                                                   | 1. Enter existing and valid `GeoJSON file path` for one that contains 2 Levels and 12 Stores, all valid                                               | The Levels and Stores are recorded in the database, and the summary list is successfully displayed in the console.                                                                                                                                                                                                     | ⚪ NOT RUN YET |       —        | None       |
| **TC-02** | Invalid path to the GeoJSON file                      | Negative / File and data handling | 1. Successful connection to the database <br> 2.There must be no GeoJSON file with the same name as the test data input one                     | 1. Enter a non-existent file path for the GeoJSON file                                                                                                | Error alert: `"Could not find GeoJSON file on the provided file path."`                                                                                                                                                                                                                                                | ⚪ NOT RUN YET |       —        | None       |
| **TC-03** | GeoJSON file exists but is empty                      | Negative / File and data handling | 1. Successful connection to the database <br> 2.The input GeoJSON file must exist but be empty                                                  | 1. Enter existing and empty GeoJSON file                                                                                                              | `GeoJSONParserService` picks up on the file being empty and aborts the process, sending an error message `"Error: GeoJSON file is empty."`                                                                                                                                                                             | ⚪ NOT RUN YET |       —        | None       |
| **TC-04** | Invalid JSON syntax format                            | Negative / File and data handling | 1. Successful connection to the database <br> 2. The input GeoJSON file must contain syntax with invalid format                                 | 1. Enter path to a GeoJSON file with invalid format syntax                                                                                            | `GeoJSONParserService` picks up on the invalid formatted syntax and aborts the process, sending an error message `"Error: wrongly formatted syntax in the GeoJSON file"`                                                                                                                                               | ⚪ NOT RUN YET |       —        | None       |
| **TC-05** | Duplicate Levels or Stores                            | Duplicate data                    | 1. Successful connection to the database <br> 2. The input GeoJSON file must contain Level or Store elements that already exist in the database | 1. Enter path to a GeoJSON file containing elements already present in the database <br> 2. Selection those duplicate elements to add to the database | `ImportGeoJSONController` picks up on the duplicate geographical elements, does not add them to the database (still adds the non-duplicate ones) and displays a warning message `"Warning: Duplicate element ´element_name´ found. Not added to the database."` for each duplicate found.                              | ⚪ NOT RUN YET |       —        | None       |
| **TC-06** | Failed connection to database                         | Integration                       | The connection to the database is manual and intentionally severed                                                                              | 1. Enter path to a valid GeoJSON file                                                                                                                 | `ImportGeoJSONController` fails to connect to the database, aborting the process and display the error message `"Error: failed to connect to the database."`                                                                                                                                                           | ⚪ NOT RUN YET |       —        | None       |



### Execution Log

NOTE: add here evidences of the tests, such as logs or screenshots.

---

## 🐞 Defect & Bug Tracking

| Bug ID   | Summary / Description | Severity | Priority | Status | Linked Commit / Fix | Re-test Status |
|:---------|:----------------------|:---------|:---------|:-------|:--------------------|:--------------:|
| **BUG-** |                       |          |          |        |                     |                |


---

## 🤖 Unit Tests

| Planned Test                        | Covers |   Status   |
|:------------------------------------|:-------|:----------:|
| `inputTotallyValidGeoJSON`          | TC-01  | ⚪ Pending |
| `inputNonExistentFilePath`          | TC-02  | ⚪ Pending |
| `inputEmptyGeoJSON`                 | TC-03  | ⚪ Pending |
| `inputBadSyntaxGeoJSON`             | TC-04  | ⚪ Pending |
| `inputGeoJSONWithDuplicateElements` | TC-05  | ⚪ Pending |
| `severedDatabaseConnection`         | TC-06  | ⚪ Pending |


---

## 📊 Summary & QA Sign-Off

### Execution Results
* **Total Executed:** 0/6
* **Passed:** 0/6
* **Failed:** 0/6
* **Not run:** 6/6 

### Final QA Recommendation

> ⚪ Not Started: the story is documented, but implementation and testing have not begun. QA sign-off must be given by a team member other than the author.

* [ ] **QA Sign-off Approved**
* **Signed By:** TBD
* **Date:** TBD