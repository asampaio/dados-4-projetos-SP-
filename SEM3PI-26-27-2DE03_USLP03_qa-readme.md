# 🧪 QA & Test Documentation: [USLP03] Shopping Front-Office Application Skeleton

* **User Story Reference:** [USLP03: Shopping Front-Office Application Skeleton](readme.md)
* **QA Lead / Owner:** TBD (must be a team member other than the author, per section 3.4 of the requirements document)
* **Target Environment:** `Front-office app (Java, JUnit; H2 in memory for the unit tests, Oracle for the manual tests)`
* **Build / Release Version:** `Sprint 1`
* **QA Status:** 🟡 In Progress (unit tests passing, manual tests pending)

---

## 🎯 QA Strategy & Scope

### In-Scope
* Console menu navigation: valid options, invalid options (numbers outside the menu, text, empty line) and exit.
* Opening and closing the JDBC connection, and handling of a database that cannot be reached.
* Configuration of the connection parameters (properties file and/or environment variables), with no hard-coded credentials.
* Handling of a missing configuration, with a clear error message.
* Code quality: Javadoc, coding standards (CamelCase) and OOP structure (new menu options added without changing the menu class).

### Out-of-Scope
* Business features: the store listing is covered by [USLP04]. In this story its menu option is only a placeholder.
* Authentication of the customer (not required in this sprint).

---

## 🛠️ Test Environment & Setup

* **Endpoint:** `Front-office (app-one)`
* **Tooling:** Java + JUnit 5, Maven (H2 for the unit tests, Oracle for the manual tests)
* **Pre-conditions:**
  * Oracle database instantiated with `database/ddl.sql`, and the `ojdbc11` driver in the project
  * Connection settings in a properties file or in `DB_URL`, `DB_USER` and `DB_PASSWORD` (never committed)
* **Test data:**
  * Valid configuration
  * Invalid JDBC URL
  * No configuration
  * Menu inputs: `99`, `abc`, an empty line and `0`
* **Approach:** TDD. Each test is written first (Red), then the minimum code to pass (Green), then refactor.

---

## 📋 Test Matrix

| Test ID   | Title / Scenario          | Type                   | Pre-conditions                                                | Test Steps                                                                | Expected Result                                                                                                                              |     Status     | Execution Date | Bugs Filed |
|:----------|:--------------------------|:-----------------------|:--------------------------------------------------------------|:--------------------------------------------------------------------------|:---------------------------------------------------------------------------------------------------------------------------------------------|:--------------:|:--------------:|:-----------|
| **TC-01** | Start with DB available   | Functional             | Valid configuration; database instantiated                    | 1. Run the application                                                    | The connection is opened and the main menu is displayed, with the option `1 - List available stores` and `0 - Exit`.                         | ⚪ NOT RUN YET |       —        | None       |
| **TC-02** | Invalid menu option       | Negative               | Application running and main menu displayed                   | 1. Enter `99` <br> 2. Enter `abc` <br> 3. Enter an empty line             | For each input: `"Invalid option. Please choose a number from 0 to 1."` and the menu is displayed again. The application does not crash.     | ⚪ NOT RUN YET |       —        | None       |
| **TC-03** | DB unavailable            | Negative               | Configuration with an invalid JDBC URL                        | 1. Run the application                                                    | `"Error: Could not connect to the database."` is displayed and the application terminates with exit code 1.                                  | ⚪ NOT RUN YET |       —        | None       |
| **TC-04** | Exit                      | Functional             | Application running and main menu displayed                   | 1. Select `0` (Exit)                                                      | `"Goodbye!"` is displayed, the connection is closed and the application ends with exit code 0.                                               | ⚪ NOT RUN YET |       —        | None       |
| **TC-05** | No hard-coded credentials | Security / Code review | Source code available                                         | 1. Search the source code (excluding tests) for the URL, user and password | The database URL, user and password appear only in the configuration (properties file or environment variables), never in the source code.   | ⚪ NOT RUN YET |       —        | None       |
| **TC-06** | Missing configuration     | Negative               | No configuration file and no environment variables            | 1. Run the application                                                    | `"Error: Missing database configuration: set DB_URL or 'db.url' in the configuration file."` is displayed and the application exits with code 1. | ⚪ NOT RUN YET |       —        | None       |

### Execution Log

NOTE: add here evidences of the tests, such as logs or screenshots.

**TDD (unit tests)**
* **Red:** 17 tests written first (`ShoppingUITest` 6, `DatabaseConnectionTest` 7, `MainTest` 4) with empty class stubs. The code compiled; 15 tests failed and 2 passed (trivially, with empty stubs).
* **Green:** `ShoppingUI`, `Main` and `DatabaseConnection` implemented. `mvn test -pl shared-library,app-one -am`: BUILD SUCCESS (2026-10-22).

---

## 🐞 Defect & Bug Tracking

| Bug ID   | Summary / Description | Severity | Priority | Status | Linked Commit / Fix | Re-test Status |
|:---------|:----------------------|:---------|:---------|:-------|:--------------------|:--------------:|
| **BUG-** |                       |          |          |        |                     |                |

---

## 🤖 Unit Tests

Execution: `mvn test -pl shared-library,app-one -am` (from the `JavaApp` folder).

| Test class               | Test                                                   | Covers                    |  Status   |
|:-------------------------|:-------------------------------------------------------|:--------------------------|:---------:|
| `ShoppingUITest`         | `run_showsMainMenuWithExitOption`                      | TC-01                     | ✅ Passing |
| `ShoppingUITest`         | `run_showsTheGivenOptionsNumberedFromOne`              | TC-01                     | ✅ Passing |
| `ShoppingUITest`         | `run_invalidOptions_showMessageAndMenuAgain`           | TC-02                     | ✅ Passing |
| `ShoppingUITest`         | `run_validOption_runsItsActionAndShowsMenuAgain`       | Navigation (valid option) | ✅ Passing |
| `ShoppingUITest`         | `run_exitOption_endsWithGoodbye`                       | TC-04                     | ✅ Passing |
| `ShoppingUITest`         | `run_inputEnds_returnsWithoutError`                    | Edge case (input ends)    | ✅ Passing |
| `DatabaseConnectionTest` | `openAndClose_validDatabase`                           | TC-01, TC-04              | ✅ Passing |
| `DatabaseConnectionTest` | `close_withoutOpen_doesNothing`                        | TC-04 (edge case)         | ✅ Passing |
| `DatabaseConnectionTest` | `open_unreachableDatabase_throwsClearError`            | TC-03                     | ✅ Passing |
| `DatabaseConnectionTest` | `fromConfiguration_readsTheFile`                       | TC-05                     | ✅ Passing |
| `DatabaseConnectionTest` | `fromConfiguration_environmentOverridesTheFile`        | TC-05                     | ✅ Passing |
| `DatabaseConnectionTest` | `fromConfiguration_noFileButCompleteEnvironment_works` | TC-05                     | ✅ Passing |
| `DatabaseConnectionTest` | `fromConfiguration_missingParameters_throwsClearError` | TC-06                     | ✅ Passing |
| `MainTest`               | `run_databaseAvailable_showsMenuAndExitsCleanly`       | TC-01, TC-04              | ✅ Passing |
| `MainTest`               | `run_databaseUnreachable_exitsWithClearMessage`        | TC-03                     | ✅ Passing |
| `MainTest`               | `run_missingConfiguration_exitsWithClearMessage`       | TC-06                     | ✅ Passing |
| `MainTest`               | `run_storeListingOption_isPlaceholderForNow`           | Placeholder for [USLP04]  | ✅ Passing |

---

## 📊 Summary & QA Sign-Off

### Execution Results
* **Manual tests (TC-01 to TC-06):** Total executed: 0/6 · Passed: 0/6 · Failed: 0/6 · Not run: 6/6
* **Unit tests:** Total executed: 17/17 · Passed: 17/17 · Failed: 0/17

### Final QA Recommendation

> 🟡 In Progress: the unit tests pass, but the manual tests TC-01 to TC-06 have not been executed yet. QA sign-off must be given by a team member other than the author.

* [ ] **QA Sign-off Approved**
* **Signed By:** TBD
* **Date:** TBD