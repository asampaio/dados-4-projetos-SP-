# 🧪 QA & Test Documentation: [USEI02] Parking Station Monitoring and Control Console

* **User Story Reference:** [USEI02: Parking Station Monitoring and Control Console](./readme.md)
* **QA Lead / Owner:** TBD (must be a team member other than the author, Daniel, per section 3.4 of the requirements document)
* **Target Environment:** `Planning app` / `app-two` (Java, JUnit, in-memory JFC Stores)
* **Build / Release Version:** `Sprint 1`
* **QA Status:** ⚪ Not Started (implementation pending)

---

## 🎯 QA Strategy & Scope

### In-Scope
* Successful query to the in-memory JFC Stores containing all `RobotStation` objects and listing of all of them. 
* Successful query of internal `RobotStation`information, such as `totalCapacity`, `occupiedSpots` and `freeSpots`.
* Successful query, sorting and presentation of `parkedRobotsList`, with the sorting being made with a Comparator and the sorting requirement being the Robot's `batteryLevel`, descending (and the `Robot ID` for when two Robots have the same battery level).
* Valid output for an empty station and a full station.

### Out-of-Scope
* Access of RobotStation internal data from the database (H2 or Oracle, for this US all internal data is access from `in-memory JCF Stores`)

---

## 🛠️ Test Environment & Setup

* **Endpoint:** J
* Tooling: Java+Junit, in-memory JCF Stores
* **Pre-conditions:**
  * `RobotStation`, `Robot` and other objects must be instantiated in the `in-memory JCF Stores`, after being imported to the application through the provided csv files (this is related to USEI01)
* **Test data:** 
  * Queried Station with many parked robots, but not full
  * Queried full station
  * Queried empty station
  * Queried station with two parked robots with the same battery level
* **Approach:** 
  * TDD. Each test is written first (Red), then the minimum code to pass (Green), then refactor.
  * For each test data you must manually check the imported `RobotStations` in the CSV file, in order to identify which ones to select to query in the beginning of the test (refer to `Pre-conditions` and `Test Steps` columns of the Test Matrix table below) 
---

## 📋 Test Matrix

| Test ID   | Title / Scenario                                                  | Type                  | Pre-conditions                                                                                                                                         | Test Steps                                                            | Expected Result                                                                                                                                                                                                                                                                                    |     Status     | Execution Date | Bugs Filed |
|:----------|:------------------------------------------------------------------|:----------------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------|:----------------------------------------------------------------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:--------------:|:--------------:|:-----------|
| **TC-01** | Query non-empty, non-full, valid RobotStation                     | Functional            | There must be at least one not empty or full, and valid `RobotStation` in the `in-memory JCF Store` to choose from                                     | 1. Select the valid non-empty, non-full, `RobotStation` from the list | 1. Station capacity summary displayed  (ex: `"Max: 10; Occupied: 4; Free: 6"`) <br> 2.  Robots listed in descending order (ex: <br> `R-03 (READY, 98%)` <br>`R-01 (CHARGING, 75%)` <br>`R-08 (MAINTENANCE, 40%)`)                                                                                  | ⚪ NOT RUN YET |       —        | None       |
| **TC-02** | Query valid empty RobotStation                                    | Functional / Boundary | There must be at least one empty and valid `RobotStation` in the `in-memory JCF Store` to choose from                                                  | 1. Select the valid and empty `RobotStation` from the list            | 1. Station capacity summary displayed  (ex: `"Max: 10; Occupied: 0; Free: 10"`) <br> 2. No parked robots listing <br> 3. Message `"Station has no parked robots"` displayed                                                                                                                        | ⚪ NOT RUN YET |       —        | None       |
| **TC-03** | Query valid full RobotStation                                     | Functional / Boundary | There must be at least one full and valid `RobotStation` in the `in-memory JCF Store` to choose from                                                   | 1. Select the valid and empty `RobotStation` from the list            | 1. Station capacity summary displayed  (ex: `"Max: 10; Occupied: 10; Free: 0"`) <br> 2. Normal expected parked robots listing, sorted by battery level (list lenght must be equal to number of parked robots and station's max capacity) <br> 3. Message `"No free spots, all occupied"` displayed | ⚪ NOT RUN YET |       —        | None       |
| **TC-04** | Query valid station with 2 robots having identical battery levels | Functional / Ordering | There must be at least one valid `RobotStation` in the `in-memory JCF Store`, containing two parked robots with the same battery level, to choose from | 1. Select the valid desired `RobotStation` from the list              | 1. Station capacity summary displayed  <br> 2. Expect valid sorting by battery level. The pair of Robots who have the same battery level is sorted by ID.                                                                                                                                          | ⚪ NOT RUN YET |       —        | None       |

### Execution Log

NOTE: add here evidences of the tests, such as logs or screenshots.

---

## 🐞 Defect & Bug Tracking

| Bug ID   | Summary / Description | Severity | Priority | Status | Linked Commit / Fix | Re-test Status |
|:---------|:----------------------|:---------|:---------|:-------|:--------------------|:--------------:|
| **BUG-** |                       |          |          |        |                     |                |

---

## 🤖 Unit Tests

| Planned Test                      | Covers |   Status   |
|:----------------------------------|:-------|:----------:|
| `functionalNormalStation`         | TC-01  | ⚪ Pending |
| `emptyStation`                    | TC-02  | ⚪ Pending |
| `fullStation`                     | TC-03  | ⚪ Pending |
| `stationWithSameBatteryRobotPair` | TC-04  | ⚪ Pending |
---

## 📊 Summary & QA Sign-Off

### Execution Results
* **Total Executed:** 0/4
* **Passed:** 0/4
* **Failed:** 0/4
* **Not run:** 4/4

### Final QA Recommendation

> ⚪ Not Started: the story is documented, but implementation and testing have not begun. QA sign-off must be given by a team member other than the author.


* [ ] **QA Sign-off Approved**
* **Signed By:** TBD
* **Date:** TBD