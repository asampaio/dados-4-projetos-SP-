# 🧪 QA & Test Documentation: [USLP05] Admin App Skeleton (Text UI + JSON)

* **User Story Reference:** [USLP05: Admin App Skeleton (Text UI + JSON)](./readme.md) — GitHub issue [#5](https://github.com/Departamento-de-Engenharia-Informatica/SEM3PI-26-27-2DE02/issues/5)
* **QA Lead / Owner:** Tiago Borges (1231109)
* **Developer:** Francisco Marques (1212081)
* **Target Environment:** local, Java 21, Maven; ShopRobot application (`com.example.shoprobot.Main`), H2 in memory
* **Build / Release Version:** `main` at commit `f09aa5f` (2026-10-20); re-test at commit `1015997` (2026-10-21)
* **QA Status:** 🟢 Approved — QA-01 to QA-04 fixed and re-tested (2026-10-21)

---

## 🎯 QA Strategy & Scope

### In-Scope
* AC1 (text-based UI in Java), AC2 (prepared to use JDBC to access a relational database) and AC3 (prepared to import JSON data), as in `doc/requirements/requirements.md`.
* Client clarification of 2026-10-02 for USLP03-USLP06: the back-office imports JSON/GeoJSON using a properties file for the input path.
* Functional scenarios TC-01 to TC-08 of the [US readme](./readme.md#-functional-tests--validation), plus extra scenarios on the settings and the JSON file.
* Unit tests and JaCoCo coverage of the Back-office skeleton classes.
* Completeness of the US documentation.

### Out-of-Scope
* Importing the stores and levels of the layout into the database (USLP06).
* Database classes reused from USLP03 (`DatabaseSettings`, `ConnectionFactory`, `JdbcDatabaseInfoDao`), already approved in its [QA report](../USLP03/qa-readme.md).
* Oracle JDBC driver (later sprints).

---

## 🛠️ Test Environment & Setup

* **How to run:** `com.example.shoprobot.Main` → **3 - Back-office**.
* **Database settings:** H2 in memory by default; `SHOPROBOT_DB_URL` for the error scenarios.
* **JSON file:** `data/geojson/norteshopping.geojson` by default (591 features); `SHOPROBOT_LAYOUT_FILE` for the error scenarios (files created in a temporary folder: not JSON, empty, `[1,2]`, `{"type":"FeatureCollection"}`).

---

## 📋 Test Matrix

| Test ID | Title / Scenario | Type | Pre-conditions | Test Steps | Expected Result | Status | Execution Date | Bugs Filed |
| :--- | :--- | :--- | :--- | :--- | :--- | :---: | :---: | :--- |
| **TC-01** | Back-office menu (AC1) | Functional | Default settings | Choose `3 - Back-office` in the main menu | Title `=== ShopRobot - Back-office ===` and options `1`, `2`, `0 - Back` | 🟢 PASS | 2026-10-20 | None |
| **TC-02** | Check database connection (AC2) | Functional | Default settings | Choose `1` | `Connected to H2 2.2.224 (2023-09-17)` | 🟢 PASS | 2026-10-20 | None |
| **TC-03** | Check JSON import (AC3) | Functional | Default settings | Choose `2` | `JSON file ..\data\geojson\norteshopping.geojson read: 591 features` (591 = features in the file) | 🟢 PASS | 2026-10-20 | None |
| **TC-04** | Invalid option | Negative | Default settings | Choose `9`, then an empty line | `Invalid option.` and the menu again | 🟢 PASS | 2026-10-20 | None |
| **TC-05** | Option `0` | Functional | Default settings | Choose `0` | `Leaving Back-office.` and back to the main menu | 🟢 PASS | 2026-10-20 | None |
| **TC-06** | Unknown JDBC URL (AC2) | Negative | `SHOPROBOT_DB_URL=jdbc:unknown:db` | Start the app, choose `3`, `1` | `Database connection failed: Failed to connect to jdbc:unknown:db`; no crash | 🟢 PASS | 2026-10-20 | None |
| **TC-07** | Missing JSON file (AC3) | Negative | `SHOPROBOT_LAYOUT_FILE=nope.json` | Start the app, choose `3`, `2` | `JSON import failed for nope.json: …`; no crash | 🟢 PASS | 2026-10-20 | None |
| **TC-08** | File that is not JSON (AC3) | Negative | `SHOPROBOT_LAYOUT_FILE` = file with `not json {` | Start the app, choose `3`, `2` | `JSON import failed for …: Unrecognized token 'not' …`; no crash | 🟢 PASS | 2026-10-20 | None |
| **TC-09** | Unreachable database server | Negative | `SHOPROBOT_DB_URL=jdbc:h2:tcp://localhost:9/nope` | Start the app, choose `3`, `2`, `1` | Option `2` still works; option `1` shows `Database connection failed: …` | 🟢 PASS | 2026-10-20 | None |
| **TC-10** | Spaces around the option | Robustness | Default settings | Type ` 2 ` | Same as option `2` | 🟢 PASS | 2026-10-20 | None |
| **TC-11** | End of input | Robustness | Default settings | Piped input that ends inside the Back-office menu | `Leaving Back-office.`, then `Exiting ShopRobot.` | 🟢 PASS | 2026-10-20 | None |
| **TC-12** | Blank or padded layout variable | Negative | `SHOPROBOT_LAYOUT_FILE="   "` and `SHOPROBOT_LAYOUT_FILE=" ../data/geojson/norteshopping.geojson "` | Start the app, choose `3`, `2` | Blank: default layout used; padded: spaces ignored; `read: 591 features` in both | 🟢 PASS | 2026-10-20 | None |
| **TC-13** | Default layout from the repository root and from `JavaApp` | Functional | Default settings | Start the app in each folder, choose `3`, `2` | `data\geojson\…` and `..\data\geojson\…` found; `read: 591 features` | 🟢 PASS | 2026-10-20 | None |
| **TC-14** | Path is a folder | Negative | `SHOPROBOT_LAYOUT_FILE` = a folder | Start the app, choose `3`, `2` | `JSON import failed for …`; no crash | 🟢 PASS | 2026-10-20 | None |
| **TC-15** | Valid JSON that is not a layout | Negative | Empty file, `[1,2]`, `{"type":"FeatureCollection"}` | Start the app, choose `3`, `2` | Reported as not a GeoJSON layout; documented in the readme (*Outputs*, *Edge Cases*) | 🟢 PASS (re-test) | 2026-10-21 | [QA-03](#-defect--bug-tracking) |
| **TC-16** | Invalid layout path | Negative | First run: `SHOPROBOT_LAYOUT_FILE=nope?.json`; re-test: `backoffice.layout.file=nope?.json` | Start the app, choose `3`, `2` | A message and the menus still usable (as in TC-07) | 🟢 PASS (re-test) | 2026-10-21 | [QA-02](#-defect--bug-tracking) |
| **TC-17** | Input path from a properties file (client, 2026-10-02) | Review | — | Read `LayoutFileSettings`, `Main` and the readme | The JSON input path is read from a properties file | 🟢 PASS (re-test) | 2026-10-21 | [QA-01](#-defect--bug-tracking) |
| **TC-18** | Unit tests | Automated | — | `mvn clean verify` in `JavaApp` | All tests pass | 🟢 PASS | 2026-10-20 | None |
| **TC-19** | Coverage ≥ 80% | Automated | — | JaCoCo report | Line coverage ≥ 80% in every class of the skeleton | 🟢 PASS | 2026-10-20 | None |
| **TC-20** | US documentation complete | Review | — | Read the [US readme](./readme.md), the diagrams and the Javadoc | Readme, `cd.puml` / `cd.png` and `sd.puml` / `sd.png` match the application | 🟢 PASS (re-test) | 2026-10-21 | [QA-04](#-defect--bug-tracking) |

### Execution Log

**TC-01 to TC-05 / TC-10 / TC-11 (input `3`, empty line, `2`, `9`, ` 2 `, `1`, `0`, `0`):**
```
=== ShopRobot - Back-office ===
1 - Check database connection
2 - Check JSON import
0 - Back
Option: Invalid option.
=== ShopRobot - Back-office ===
Option: JSON file ..\data\geojson\norteshopping.geojson read: 591 features
=== ShopRobot - Back-office ===
Option: Invalid option.
=== ShopRobot - Back-office ===
Option: JSON file ..\data\geojson\norteshopping.geojson read: 591 features
=== ShopRobot - Back-office ===
Option: Connected to H2 2.2.224 (2023-09-17)
=== ShopRobot - Back-office ===
Option: Leaving Back-office.
=== ShopRobot ===
Option: Exiting ShopRobot.
```

**TC-06 / TC-09 — database errors (the JSON option still works):**
```
SHOPROBOT_DB_URL=jdbc:unknown:db
  Using the configured database jdbc:unknown:db (sample Shopping tables not created).
  Option: JSON file ..\data\geojson\norteshopping.geojson read: 591 features
  Option: Database connection failed: Failed to connect to jdbc:unknown:db
SHOPROBOT_DB_URL=jdbc:h2:tcp://localhost:9/nope
  Using the configured database jdbc:h2:tcp://localhost:9/nope (sample Shopping tables not created).
  Option: JSON file ..\data\geojson\norteshopping.geojson read: 591 features
  Option: Database connection failed: Failed to connect to jdbc:h2:tcp://localhost:9/nope
```

**TC-07 / TC-08 / TC-14 — JSON errors (the database option still works):**
```
SHOPROBOT_LAYOUT_FILE=nope.json
  Option: JSON import failed for nope.json: nope.json (O sistema não conseguiu localizar o ficheiro especificado)
SHOPROBOT_LAYOUT_FILE=<temp>\bad.json
  Option: JSON import failed for <temp>\bad.json: Unrecognized token 'not': was expecting (JSON String, Number, Array, Object or token 'null', 'true' or 'false')
SHOPROBOT_LAYOUT_FILE=<temp folder>
  Option: JSON import failed for <temp folder>: <temp folder> (Acesso negado)
```

**TC-15 — valid JSON that is not a layout:**
```
empty file                     → Option: JSON file <temp>\empty.json read: 0 features
[1,2]                          → Option: JSON file <temp>\array.json read: 0 features
{"type":"FeatureCollection"}   → Option: JSON file <temp>\nofeatures.json read: 0 features
```

**TC-16 — the application stops before showing any menu:**
```
SHOPROBOT_LAYOUT_FILE=nope?.json
Exception in thread "main" java.nio.file.InvalidPathException: Illegal char <?> at index 4: nope?.json
	at com.example.shoprobot.model.importer.LayoutFileSettings.fromEnvironment(LayoutFileSettings.java:34)
	at com.example.shoprobot.Main.main(Main.java:63)
```
`LayoutFileSettings.fromEnvironment` calls `Path.of(configured.trim())` without handling `InvalidPathException`, and `Main` does not catch it, so Shopping and Planning cannot be used either.

**TC-17 — review:** the client session of 2026-10-02 (see `doc/requirements/requirements.md`) says that the back-office imports JSON/GeoJSON *using a properties file for the input path*. `LayoutFileSettings` reads the path from the environment variable `SHOPROBOT_LAYOUT_FILE`; there is no `.properties` file in the project, and the readme (*Customer Specifications & Clarifications*) does not mention this clarification.

**TC-18 — unit tests (all passing, 230 in total):** `BackofficeMenuTest` (7), `BackofficeControllerTest` (6), `JsonFileReaderTest` (3), `LayoutFileSettingsTest` (3), `MainMenuTest` (6), `BackofficeMainTest` (2).

**TC-19 — JaCoCo line / branch coverage:** `BackofficeMenu` 100% / 100%, `BackofficeController` 100% / —, `JsonFileReader` 100% / —, `LayoutFileSettings` 100% / 100%, `MainMenu` 100% / 100%.

**TC-20 — documentation:** the class and sequence diagrams match the code, and the readme matches the application (menu, messages, edge cases, 591 features). The header still says `Status: In Progress` and `QA: To be assigned`, and the constructors of `BackofficeMenu` and `BackofficeController` have no Javadoc.

**Note (out of scope, not a USLP05 defect):** started from a folder outside the repository, the application stops at startup with `IllegalArgumentException: Input directory does not exist: data\medium\clean`, thrown by the Planning data import in `Main` (it only catches `IOException`).

### Re-test after the fixes (commit `1015997`, 2026-10-21)

Fixes by the developer: `3e1b47f` (`JavaApp/src/main/resources/shoprobot.properties` with `backoffice.layout.file`), `0678a97` (`LayoutFileSettings` reads the path from the properties file — the included one, replaced by a `shoprobot.properties` in the folder where the application is started — and `BackofficeController` reports an invalid path), `6b70dc5` (an empty file is read as a missing node), `4e43767` (Javadoc of the `BackofficeMenu` constructor), `669548a`, `22a8abf`, `b48cce1` (readme, class and sequence diagrams). The commit tested also has the USLP06 option `3 - Import layout` in the Back-office menu, which does not change options `1`, `2` and `0`.

**TC-03 / TC-07 / TC-08 / TC-11 to TC-17 — repeated with a temporary `shoprobot.properties` in the folder where the application is started (removed after the test):**
```
(no external file)                                → JSON file data/geojson/norteshopping.geojson read: 591 features
backoffice.layout.file=nope.json                  → JSON import failed for nope.json: nope.json (O sistema não conseguiu localizar o ficheiro especificado)
backoffice.layout.file=<temp>/bad.json            → JSON import failed for <temp>/bad.json: Unrecognized token 'not': …
backoffice.layout.file=nope?.json                 → JSON import failed for nope?.json: invalid path (Illegal char <?>)
backoffice.layout.file=<temp>/array.json  ([1,2]) → JSON file <temp>/array.json read, but it is not a GeoJSON layout (no "features" list)
backoffice.layout.file=<temp>/empty.json          → JSON file <temp>/empty.json read, but it is not a GeoJSON layout (no "features" list)
backoffice.layout.file=<temp>/zero.json ([] list) → JSON file <temp>/zero.json read: 0 features
backoffice.layout.file=   (blank)                 → JSON file data/geojson/norteshopping.geojson read: 591 features
backoffice.layout.file=<temp folder>              → JSON import failed for <temp folder>: … (Acesso negado)
```
In every case the application starts, option `1` still shows `Connected to H2 2.2.224 (2023-09-17)` and the menu keeps working. From `JavaApp` the default layout is also found (`read: 591 features`), and an external file in `JavaApp` replaces the included value. `SHOPROBOT_DB_URL=jdbc:unknown:db` still gives `Database connection failed: …` (TC-06).

**TC-17 — review:** the path is read from the property `backoffice.layout.file` of `shoprobot.properties`; the readme documents the client clarification of 2026-10-02 (*Customer Specifications & Clarifications*, NFR03, *Inputs*). Paths with `\` in the properties file are read as escape characters (e.g. `data\geojson\…` → `datageojson…`); this is how properties files work, and the file and the readme say to use `/`.

**TC-18 — unit tests (all passing, 285 in total):** `BackofficeMenuTest` (9), `BackofficeControllerTest` (9), `JsonFileReaderTest` (3), `LayoutFileSettingsTest` (10), `MainMenuTest` (6), `BackofficeMainTest` (3).

**TC-19 — JaCoCo line / branch coverage:** `BackofficeController` 100% / 100%, `BackofficeMenu` 100% / 100%, `MainMenu` 100% / 100%, `JsonFileReader` 100% / 50% (the empty-file branch of `6b70dc5` has no unit test; checked manually), `LayoutFileSettings` 92.9% / 87.5% (only the `catch` of an unreadable properties file is not covered).

**TC-20 — documentation:** readme header updated (`Status: Testing`, `QA: Tiago Borges`); the readme, `cd.puml` / `cd.png` (with `shoprobot.properties`) and `sd.puml` / `sd.png` (with the invalid path) match the application; `BackofficeMenu` and `BackofficeController` constructors have Javadoc.

---

## 🐞 Defect & Bug Tracking

| Bug ID | Summary / Description | Severity | Priority | Status | Linked Commit / Fix | Re-test Status |
| :--- | :--- | :--- | :--- | :--- | :--- | :---: |
| **QA-01** | The JSON input path is read from the environment variable `SHOPROBOT_LAYOUT_FILE`, but the client (session of 2026-10-02) said the back-office imports JSON/GeoJSON using a **properties file** for the input path. There is no properties file in the project and the readme does not mention this clarification. Suggestion: read the layout file from a properties file (e.g. in `JavaApp`), keep the default layout, and document it in the readme. USLP06 depends on it. | Medium | High | Fixed | `3e1b47f`, `0678a97`, `669548a` | 🟢 Passed |
| **QA-02** | The application crashes at startup when the layout path is not valid (e.g. `SHOPROBOT_LAYOUT_FILE=nope?.json` → `InvalidPathException`), before any menu is shown. Suggestion: handle the error so that option `2` shows `JSON import failed for …` (as for a missing file) and the menus keep running. | Medium | Medium | Fixed | `0678a97` | 🟢 Passed |
| **QA-03** | *(Observation)* An empty file or a JSON that is not an object (e.g. `[1,2]`) is reported as `read: 0 features`. Suggestion: document it in the readme (*Edge Cases*) or report that the file is not a layout. | Low | Low | Fixed | `0678a97`, `6b70dc5`, `669548a` | 🟢 Passed |
| **QA-04** | *(Observation)* Readme header: `Status: In Progress` and `QA: To be assigned` (QA: Tiago Borges). The constructors of `BackofficeMenu` and `BackofficeController` have no Javadoc. | Low | Low | Fixed | `669548a`, `4e43767`, `0678a97` | 🟢 Passed |

---

## 🤖 Unit Tests

* **Related App:** JavaApp
* **Related Code:** `view/MainMenu`, `view/backoffice/BackofficeMenu`, `controller/backoffice/BackofficeController`, `model/importer/JsonFileReader`, `model/importer/LayoutFileSettings`, `Main`
* **Automated Tests:** `BackofficeMenuTest`, `BackofficeControllerTest`, `JsonFileReaderTest`, `LayoutFileSettingsTest`, `MainMenuTest`, `BackofficeMainTest`
* **Execution:** `mvn clean test` (coverage: `mvn verify`, report in `JavaApp/target/site/jacoco`)

---

## 📊 Summary & QA Sign-Off

### Execution Results
* **Total Executed:** 20 (after re-test)
* **Passed:** 20
* **Failed:** 0
* **Observations:** 0
* **Blocked:** 0

*First run: 16 passed, 2 failed (TC-16 — QA-02, TC-17 — QA-01), 2 observations (TC-15 — QA-03, TC-20 — QA-04).*

### Final QA Recommendation

> 🟢 **Approved:** the three acceptance criteria are met — text menu in Java (AC1), JDBC connection to the configured database (AC2) and JSON file read with Jackson (AC3). The input path now comes from `shoprobot.properties`, as asked by the client (QA-01), an invalid path no longer stops the application (QA-02), a JSON that is not a layout is reported as such (QA-03) and the documentation is up to date (QA-04). The user story can be moved to **Done**. Not blocking: a unit test for the empty file in `JsonFileReaderTest` would cover the branch added in `6b70dc5`.

* [x] **QA Sign-off Approved**
* **Signed By:** Tiago Borges (QA)
* **Date:** 2026-10-21
