# 🧪 QA & Test Documentation: [USLP04] List Available Stores

* **User Story Reference:** [USLP04: List Available Stores](./readme.md)
* **QA Lead / Owner:** TBD (must be a team member other than the author, Hugo Coutinho, per section 3.4 of the requirements document)
* **Target Environment:** `Shopping App (Java, JUnit, H2)`
* **Build / Release Version:** Sprint 1
* **QA Status:** 🟡 In Progress (11 unit tests passing; manual functional tests and QA sign-off pending)

---

## 🎯 QA Strategy & Scope

### In-Scope
* Listing of the levels (integer and decimal numbers) and selection of a level.
* Listing of the names of the stores of the selected level.
* Access to the data through DAOs and JDBC.
* Invalid input and error handling.

### Out-of-Scope
* Products, shopping cart and orders.
* Import of the levels and stores ([USLP06]).
* User interface appearance.

---

## 🛠️ Test Environment & Setup

* **Tooling:** Java + JUnit, H2 database (as in the template).
* **Test data:** a shopping location with several levels (one of them decimal, 0.5), stores in different levels, one level without stores and one shop area that is not a store of the system.
* **Approach:** TDD. Each test is written first (Red), then the minimum code to pass (Green), then refactor.

---

## 📋 Test Matrix

| Test ID   | Title / Scenario             | Type        | Steps                                              | Expected Result                               |      Status      |  Executed By   | Notes                                                                                  |
|:----------|:-----------------------------|:------------|:---------------------------------------------------|:----------------------------------------------|:----------------:|:--------------:|:---------------------------------------------------------------------------------------|
| **TC-01** | List the levels              | Functional  | Database with several levels; list the levels      | All levels, by ascending number               |    ✅ Passed     | Hugo (JUnit)   | —                                                                                      |
| **TC-02** | Select a valid level         | Functional  | Select an existing level                           | The names of its stores are shown             |    ✅ Passed     | Hugo (JUnit)   | —                                                                                      |
| **TC-03** | Stores in alphabetical order | Functional  | Level with several stores                          | Names sorted alphabetically                   |    ✅ Passed     | Hugo (JUnit)   | Order is an assumption                                                                 |
| **TC-04** | Only stores of the system    | Functional  | Level with a shop area that is not a store         | That area is not listed                       |    ✅ Passed     | Hugo (JUnit)   | —                                                                                      |
| **TC-05** | Level without stores         | Boundary    | Select a level with no stores                      | Informative message                           | 🟡 Partial       | Hugo (JUnit)   | DAO returns an empty list (tested); the message is to be checked manually (F-03)       |
| **TC-06** | No levels                    | Negative    | Database without levels                            | Informative message                           | 🟡 Partial       | Hugo (JUnit)   | DAO returns an empty list (tested); the message is to be checked manually (F-05)       |
| **TC-07** | Invalid level choice         | Negative    | Type letters, empty, a comma, more than two decimal places or a level that does not exist | Error message and the customer is asked again | 🟡 Partial | Hugo (JUnit) | Validation tested (`parseLevelChoice`); asking again is to be checked manually (F-04) |
| **TC-08** | Data comes from the database | Integration | Run the DAOs against H2 with test data             | Results match the data in the database        |    ✅ Passed     | Hugo (JUnit)   | Covered by the DAO tests                                                               |
| **TC-09** | Database error               | Negative    | Simulate a failure in the query                    | Clear error message, no crash                 | 🟡 Partial       | Hugo (JUnit)   | DAO throws `DaoException` (tested); the message shown to the user is to be checked manually (F-06) |
| **TC-10** | Stores of other levels       | Negative    | Select a level when other levels have stores       | Only the stores of the selected level         |    ✅ Passed     | Hugo (JUnit)   | —                                                                                      |
| **TC-11** | Decimal levels in the database | Integration | Database with levels 1, -1, 0.5 and 0; list the levels | Levels returned by ascending number: -1, 0, 0.5, 1 | ✅ Passed | Hugo (JUnit)   | Level numbers are `DECIMAL(5,2)` in the database and `BigDecimal` in the code          |
| **TC-12** | Select a decimal level       | Functional  | Select the level `0.5`                             | The names of its stores are shown, not those of other levels | ✅ Passed | Hugo (JUnit) | The customer types `0.5` (decimal point)                                               |

### Execution Log

| Date       | Executed By  | Command                          | Result                                                                                                           |
|:-----------|:-------------|:---------------------------------|:-----------------------------------------------------------------------------------------------------------------|
| 2026-10-17 | Hugo         | `mvn -B clean verify` (`JavaApp`) | `JdbcLevelDaoTest` 3/3, `JdbcStoreDaoTest` 4/4, `StoreListingTest` 2/2: BUILD SUCCESS, 0 failures, 0 skipped      |
| 2026-10-21 | Hugo         | `mvn -B clean verify` (`JavaApp`) | `JdbcLevelDaoTest` 4/4, `JdbcStoreDaoTest` 4/4, `StoreListingTest` 3/3: BUILD SUCCESS, 0 failures, 0 skipped (decimal levels, #118 / #119) |

NOTE: the manual functional scenarios (F-01 to F-07 in the [readme](./readme.md)) are still to be executed by the QA team member.

---

## 🐞 Defect & Bug Tracking

| Issue ID | Summary / Description | Severity | Priority | Status | Linked Commit / Fix | Re-test Status |
|:---------|:----------------------|:---------|:---------|:-------|:--------------------|:--------------:|
| #118 / #119 | The level number was an integer, so a decimal level allowed by the model (for example 0.5) could not be listed or selected | Medium | TBD | Fixed | #119 (`Level` with `BigDecimal`, `DECIMAL(5,2)`) | ✅ Passed (TC-11, TC-12) |

---

## 🤖 Unit Tests

| Test                                                         | Class                | Covers       |  Status   |
|:-------------------------------------------------------------|:---------------------|:-------------|:---------:|
| `findLevels_severalLevels_returnsOrderedByNumber`            | `JdbcLevelDaoTest`   | TC-01, TC-08 | ✅ Passed |
| `selectLevel_validNumber_showsStoresOfThatLevel`             | `StoreListingTest`   | TC-02        | ✅ Passed |
| `findStoresByLevel_severalStores_returnsOrderedByName`       | `JdbcStoreDaoTest`   | TC-03        | ✅ Passed |
| `findStoresByLevel_shopAreaNotRegisteredAsStore_isNotListed` | `JdbcStoreDaoTest`   | TC-04        | ✅ Passed |
| `findStoresByLevel_levelWithoutStores_returnsEmpty`          | `JdbcStoreDaoTest`   | TC-05        | ✅ Passed |
| `findLevels_noLevels_returnsEmpty`                           | `JdbcLevelDaoTest`   | TC-06        | ✅ Passed |
| `parseLevelChoice_invalidInput_isRejected`                   | `StoreListingTest`   | TC-07        | ✅ Passed |
| `findLevels_databaseError_throwsDaoException`                | `JdbcLevelDaoTest`   | TC-09        | ✅ Passed |
| `findStoresByLevel_otherLevelsStores_areNotReturned`         | `JdbcStoreDaoTest`   | TC-10        | ✅ Passed |
| `findLevels_decimalLevel_returnsOrderedByNumber`             | `JdbcLevelDaoTest`   | TC-11        | ✅ Passed |
| `selectLevel_decimalNumber_showsStoresOfThatLevel`           | `StoreListingTest`   | TC-12        | ✅ Passed |

---

## 📊 Summary & QA Sign-Off

### Execution Results
* **Unit tests (JUnit):** 11 / 11 passing
* **Test cases:** 12
* **Passed:** 8 (TC-01, TC-02, TC-03, TC-04, TC-08, TC-10, TC-11, TC-12)
* **Partially verified:** 4 (TC-05, TC-06, TC-07, TC-09). The automated part passes; the messages shown to the customer are still to be checked manually.
* **Failed:** 0
* **Not run:** 0

### Final QA Recommendation

> 🟡 **In Progress:** the unit tests pass, but the manual functional scenarios have not been executed yet. QA sign-off must be given by a team member other than the author.

* [ ] **QA Sign-off Approved**
* **Signed By:** TBD
* **Date:** TBD