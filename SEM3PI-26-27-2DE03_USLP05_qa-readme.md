# 🧪 QA & Test Documentation: [USLP05] Back-Office Application Skeleton

* **User Story Reference:** [USLP05: Back-Office Application Skeleton](./readme.md)
* **QA Lead / Owner:** TBD (must be a team member other than the author, João Tavares, per section 3.4 of the requirements document)
* **Target Environment:** `Back-Office App (app-three, Java 21, JUnit 5, H2)`
* **Build / Release Version:** Sprint 1
* **QA Status:** 🟡 Implemented, awaiting QA review (15 JUnit tests passing; manual functional tests and sign-off pending)

---

## 🎯 QA Strategy & Scope

### In-Scope
* Start-up of the application: `DataSource` from `DatabaseConfig` and shopping schema prepared through JDBC (`ShoppingSchemaInitializer`).
* Main menu: valid options, invalid options, clean exit, end of input and menu options without text or UI.
* "Import Stores & Levels" option: reading the file path from `backoffice.properties` and running the stub `ImportGeoJSONUI`.
* JSON readiness: the JSON library of `app-three` reads a GeoJSON `FeatureCollection`.

### Out-of-Scope
* Real GeoJSON import and insertion of levels and stores ([USLP06](../USLP06/readme.md)).
* Oracle integration (the applications move to Oracle in later sprints).
* Visual layout beyond the menu text (colours, spacing).

---

## 🛠️ Test Environment & Setup

* **Tooling:** Java 21 + JUnit 5, module `app-three`, test classes `MainTest`, `AppConfigTest`, `JsonLibraryTest`, `ImportGeoJSONUITest`, `MainMenuUITest` and `MenuItemTest`; H2 in memory (as in the template).
* **Test data:** no domain data needed (skeleton only); configurations created in the tests with and without `geojson.file.path`; a `DataSource` that always fails, to simulate the database being unavailable; a small GeoJSON `FeatureCollection` with one store on level 0.5; a test properties file with a Windows path written with `\` (`src/test/resources/backoffice-malformed.properties`).
* **Approach:** TDD. Each test is written first (Red), then the minimum code to pass (Green), then refactor. Console input/output is simulated in the tests (no manual typing), so no test waits for user input.
* **Roles (client session, 28 Sep 2026):** the developer writes and runs the unit tests (JUnit); the QA, a team member other than the author, runs the functional tests of the matrix manually and records the result (pass/fail).

### Manual execution (QA)

How to start the application (in the `JavaApp` folder):

```
mvn -pl app-three -am compile exec:java -Dexec.mainClass=com.example.appthree.Main
```

The functional scenarios F-01 to F-09 of the [readme](./readme.md) are run by the QA as follows:

| Scenario | Set-up | What to do | Expected result | Test case |
|:---------|:-------|:-----------|:----------------|:----------|
| **F-01** | — | Start the application | Header `--- BACK-OFFICE MAIN MENU --------------`, then `1 - Import Stores & Levels` and `0 - Cancel` | TC-01 |
| **F-02** | — | Type `0` | `Goodbye.` and the application ends | TC-02 |
| **F-03** | — | Type `abc`, then an empty line, then `2`, then `-1` | `Invalid option '…'` for each one, and the menu is shown again | TC-03 |
| **F-04** | — | Start the application | The menu is shown (the tables were created without errors) | TC-04 |
| **F-05** | — | Cannot be reproduced manually with H2 in memory (the database is inside the application) | Covered by `start_databaseFailure_showsErrorAndExits` (JUnit): `Error: could not connect to the database. The back-office application will close.` | TC-05 |
| **F-06** | — | Type `1` | `GeoJSON import of 'export-norteshopping.geojson' is not implemented yet (USLP06).` and the menu is shown again | TC-06 |
| **F-07** | In `app-three/src/main/resources/backoffice.properties`, comment out the line `geojson.file.path=…` (put `#` at the start) | Start the application and type `1` | `Error: the GeoJSON file path is not configured (…)` and the menu is shown again | TC-07 |
| **F-08** | — | Press Ctrl+D (Linux/macOS) or Ctrl+Z and Enter (Windows) while the menu is shown | The application ends without an exception | TC-09 |
| **F-09** | In `backoffice.properties`, set `geojson.file.path=C:\users\test\file.geojson` | Start the application and type `1` | The application starts normally and shows the "path is not configured" error | TC-10 |

After F-07 and F-09, restore `backoffice.properties` (`geojson.file.path=export-norteshopping.geojson`).

---

## 📋 Test Matrix

| Test ID   | Title / Scenario                     | Type        | Steps                                                         | Expected Result                                         |   Status   | Executed By | Notes |
|:----------|:-------------------------------------|:------------|:--------------------------------------------------------------|:--------------------------------------------------------|:----------:|:-----------:|:------|
| **TC-01** | Main menu is shown                   | Functional  | Start the application                                         | Menu with "1 - Import Stores & Levels" and "0 - Cancel"| ✅ Pass (JUnit) | João Tavares (JUnit) | AC1, F-01 |
| **TC-02** | Exit (option 0)                      | Functional  | Choose "0 - Cancel"                                           | The application exits cleanly                           | ✅ Pass (JUnit) | João Tavares (JUnit) | AC1, F-02 |
| **TC-03** | Invalid option                       | Negative    | Type letters, empty, a negative number or a number above the last option | Error message and the menu is shown again   | ✅ Pass (JUnit) | João Tavares (JUnit) | AC1, F-03 |
| **TC-04** | Database available                   | Functional  | Start the application with the database available            | `levels`, `areas` and `stores` tables created and the menu is shown | ✅ Pass (JUnit) | João Tavares (JUnit) | AC2, F-04 |
| **TC-05** | Database unavailable                 | Negative    | Simulate a database failure on start-up                       | Clear error message, the application exits without crashing | ✅ Pass (JUnit) | João Tavares (JUnit) | AC2, F-05 |
| **TC-06** | Import Stores & Levels (placeholder) | Functional  | Choose "Import Stores & Levels" with the path configured      | Placeholder message with the path; the menu is shown again | ✅ Pass (JUnit) | João Tavares (JUnit) | AC3, F-06 |
| **TC-07** | GeoJSON path not configured          | Negative    | Choose "Import Stores & Levels" without `geojson.file.path` (or blank) | Clear error message and the menu is shown again  | ✅ Pass (JUnit) | João Tavares (JUnit) | F-07 |
| **TC-08** | JSON library ready                   | Structural  | Read a GeoJSON `FeatureCollection` with the JSON library      | Type, number of features, store name and level 0.5 read correctly | ✅ Pass (JUnit) | João Tavares (JUnit) | AC3 |
| **TC-09** | End of input                         | Negative    | Close the input while the menu is shown                       | The application exits cleanly, without an exception     | ✅ Pass (JUnit) | João Tavares (JUnit) | AC1, F-08 |
| **TC-10** | Properties file missing or malformed | Negative    | Load the configuration from a file that does not exist, or from a file with a Windows path such as `C:\users\…` (`\u` is an invalid escape) | No exception; the GeoJSON path is reported as not configured | ✅ Pass (JUnit) | João Tavares (JUnit) | F-07, F-09 |
| **TC-11** | Menu option without text or UI      | Negative    | Create a menu option with a null or blank text, or a null UI | Rejected (`IllegalArgumentException`); a valid option shows its text and runs its UI | ✅ Pass (JUnit) | João Tavares (JUnit) | NFR04 |

### Execution Log

| Date       | Run by       | Command                              | Result                                                                   |
|:-----------|:-------------|:-------------------------------------|:-------------------------------------------------------------------------|
| 2026-10-21 | João Tavares | `mvn -B clean verify` (in `JavaApp`) | 15 JUnit tests in `app-three` (`MainTest`, `AppConfigTest`, `JsonLibraryTest`, `ImportGeoJSONUITest`, `MainMenuUITest`, `MenuItemTest`): 15 passed, 0 failed, 0 skipped; BUILD SUCCESS |

NOTE: the manual functional scenarios (F-01 to F-09, see "Manual execution (QA)") are still to be executed by the QA team member.

---

## 🐞 Defect & Bug Tracking

| Issue ID | Summary / Description | Severity | Priority | Status | Linked Commit / Fix | Re-test Status |
|:---------|:----------------------|:---------|:---------|:-------|:--------------------|:--------------:|
| —        | None identified yet   | —        | —        | —      | —                   |       —        |

---

## 🤖 Unit Tests

Implemented JUnit tests:

| Test                                                      | Class                 | Covers       |   Status   |
|:----------------------------------------------------------|:----------------------|:-------------|:----------:|
| `start_databaseAvailable_showsMainMenu`                   | `MainTest`            | TC-01, TC-04 | ✅ Passing |
| `start_databaseFailure_showsErrorAndExits`                | `MainTest`            | TC-05        | ✅ Passing |
| `menu_exitOption_endsLoop`                                | `MainMenuUITest`      | TC-02        | ✅ Passing |
| `menu_invalidOption_showsErrorAndShowsMenuAgain`          | `MainMenuUITest`      | TC-03        | ✅ Passing |
| `menu_validOption_runsItsUiAndShowsMenuAgain`             | `MainMenuUITest`      | TC-06, TC-07 | ✅ Passing |
| `menu_endOfInput_exitsCleanly`                            | `MainMenuUITest`      | TC-09        | ✅ Passing |
| `importGeoJson_pathConfigured_showsPlaceholderMessage`    | `ImportGeoJSONUITest` | TC-06        | ✅ Passing |
| `importGeoJson_pathMissing_showsError`                    | `ImportGeoJSONUITest` | TC-07        | ✅ Passing |
| `appConfig_defaultFile_hasGeoJsonPath`                    | `AppConfigTest`       | TC-06        | ✅ Passing |
| `appConfig_missingGeoJsonPath_returnsEmpty`               | `AppConfigTest`       | TC-07        | ✅ Passing |
| `appConfig_missingFile_returnsEmpty`                      | `AppConfigTest`       | TC-10        | ✅ Passing |
| `appConfig_malformedFile_returnsEmpty`                    | `AppConfigTest`       | TC-10        | ✅ Passing |
| `jsonLibrary_readsGeoJsonFeatureCollection`               | `JsonLibraryTest`     | TC-08        | ✅ Passing |
| `menuItem_invalidArguments_throwsException`              | `MenuItemTest`        | TC-11        | ✅ Passing |
| `menuItem_valid_showsTextAndRunsUi`                      | `MenuItemTest`        | TC-11        | ✅ Passing |

---

## 📊 Summary & QA Sign-Off

### Execution Results
* **Total Test Cases:** 11 (TC-01 to TC-11, all covered by JUnit tests)
* **JUnit tests:** 15 / 15 passing
* **Passed:** 11 test cases (in JUnit)
* **Failed:** 0
* **Not run:** 0 in JUnit; the manual functional scenarios F-01 to F-09 are pending for the QA

### Final QA Recommendation

> 🟡 **Awaiting review:** the story is implemented and its 15 JUnit tests pass, covering TC-01 to TC-11. The QA (a team member other than the author) runs the functional scenarios manually (see "Manual execution (QA)"), records the result in the matrix and gives the sign-off.

* [ ] **QA Sign-off Approved**
* **Signed By:** TBD
* **Date:** TBD