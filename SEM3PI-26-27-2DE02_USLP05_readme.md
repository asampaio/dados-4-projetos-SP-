# [USLP05] Admin App Skeleton (Text UI + JSON)

* **Priority:** Urgent
* **Story Points:** 3
* **Module / Component:** `LAPR3` / `JavaApp` — Back-office area (MVC)
* **Status:** Testing
* **Developer:** Francisco Marques (1212081)
* **QA:** Tiago Borges (1231109)

---

## 👤 User Story Description

**As a** Product Owner
**I want** the team to create the "skeleton" of the back-office application
**So that** the administration features (e.g. importing the layout of USLP06) can be added on top of a text UI that is already prepared to access the relational database and to import JSON data

---

## 🚦 Definition of Ready (DoR)

* [x] **Clear User Story:** Story follows the standard *As a / I want to / So that* format.
* [x] **Acceptance Criteria Defined:** ACs taken from issue #5 (project specification).
* [x] **Dependencies Identified:** `JavaApp` Maven project (single MVC application), Jackson, layout file of the repository.
* [x] **Inputs/Outputs Mapped:** Menu options, database settings and JSON file documented below.
* [x] **Estimation Complete:** 3 story points.
* [x] **Small & Actionable:** Fits in Sprint 1.

---

## 💬 Customer Specifications & Clarifications

* **Specification (Section 5.1):** the back-office application is used by the administrators of the shopping center and is primarily developed in Java.
* **Specification (Section 7.1.1):** in Sprint 1 the H2 database may be used, as in the template application.
* **Client session (2026-10-02):** the back-office imports JSON/GeoJSON using a **properties file for the input path** (see `doc/requirements/requirements.md`). The path is the property `backoffice.layout.file` of `shoprobot.properties`.
* **Team decision (2026-10-14):** ShopRobot is one Java application in MVC; the Back-office area is option `3` of its main menu (it replaces the template user example, whose classes are kept).
* **Team decision:** the Back-office uses the same database as the Shopping area (`SHOPROBOT_DB_*`, see USLP03), so both areas see the same data.
* **Out of Scope:** importing the stores of the layout into the database (USLP06, Tiago Borges, with `GeoJsonParser`); Oracle JDBC driver (later sprints).

---

## ✅ Acceptance Criteria

* [ ] **AC1 - Text-based UI in Java:** the application shows a console menu, reads the chosen option, handles invalid options and goes back on `0` or at the end of the input.
* [ ] **AC2 - Prepared to use JDBC:** the application opens a JDBC connection to the relational database configured by environment variables (H2 in memory by default) and option `1` shows the database it is connected to.
* [ ] **AC3 - Prepared to import JSON data:** the application reads a JSON file with Jackson (the GeoJSON layout of the repository by default, set in the properties file `shoprobot.properties`) and option `2` shows how many features were read.

---

## 🔒 Non-Functional Requirements & Security

* **NFR02 - Database:** access through JDBC using the DAO pattern (`DatabaseInfoDao` / `JdbcDatabaseInfoDao`) and the same `ConnectionFactory` of the Shopping area; errors wrapped in `DaoException`.
* **NFR03 - Integration:** the database settings come from environment variables (USLP03) and the JSON file from `shoprobot.properties`, so another database or layout can be used without code changes. The properties file included in the application (`JavaApp/src/main/resources/shoprobot.properties`) has the default layout; a `shoprobot.properties` in the folder where the application is started replaces its values without recompiling.
* **NFR04 - OOP:** MVC: `view.backoffice` → `controller.backoffice` → `model` (`model.persistence` for JDBC, `model.importer` for JSON); each class has one responsibility and receives its dependencies in the constructor (testable without a real console, database or file).
* **Security:** the database password is never printed.

---

## 🔗 Found Out Dependencies

### Upstream Dependencies
* **JavaApp:** Maven project, H2 driver and Jackson (`jackson-databind`).
* **[USLP03] Shopping App Skeleton:** `DatabaseSettings`, `ConnectionFactory` and `JdbcDatabaseInfoDao` are reused.
* **Layout file:** `data/geojson/norteshopping.geojson`.

### Downstream Dependencies
* **[USLP06] Import the layout:** can use `JsonFileReader` / `LayoutFileSettings` and add its option to the Back-office menu.

---

## 📥 Input & Output Data

### Inputs
| Input | Type | Source | Example / Constraints |
| :--- | :--- | :--- | :--- |
| Menu option | Text | Keyboard | `1`, `2` or `0`; anything else is invalid |
| `SHOPROBOT_DB_URL` / `USER` / `PASSWORD` | Environment variables | Optional | as in USLP03 (default H2 in memory) |
| `backoffice.layout.file` | Property of `shoprobot.properties` | Optional | default `data/geojson/norteshopping.geojson`; a relative path is also looked up from the parent folder, so it works from the repository root and from `JavaApp`; use `/` in the paths |

### Outputs
| Situation | Output |
| :--- | :--- |
| Start / after each option | Menu: `=== ShopRobot - Back-office ===`, options `1`, `2`, `0 - Back` |
| Option `1`, database reachable | `Connected to <product> <version>` (e.g. `Connected to H2 2.2.224 (2023-09-17)`) |
| Option `1`, database unreachable | `Database connection failed: Failed to connect to <url>` |
| Option `2`, valid JSON | `JSON file <file> read: <n> features` (e.g. `591 features` for NorteShopping) |
| Option `2`, invalid path, missing file or invalid JSON | `JSON import failed for <file>: <reason>` (e.g. `invalid path (Illegal char <?>)`) |
| Option `2`, JSON that is not a layout (no `features` list, e.g. `[1,2]` or an empty file) | `JSON file <file> read, but it is not a GeoJSON layout (no "features" list)` |
| Invalid option | `Invalid option.` |
| Option `0` or end of input | `Leaving Back-office.` and back to the main menu |

---

## Edge Cases

* **End of input (e.g. Ctrl+Z / Ctrl+D or piped input):** the menu ends normally with `Leaving Back-office.`.
* **Spaces around the option:** ignored (` 2 ` is option `2`).
* **Database unreachable or unknown JDBC URL:** option `1` shows `Database connection failed: …`; the menu keeps working.
* **Invalid path (e.g. `nope?.json` on Windows), missing file, folder or invalid JSON:** option `2` shows `JSON import failed for …`; the application starts and the menu keeps working.
* **JSON that is not a layout** (no `features` list: `[1,2]`, `{"name": …}`, an empty file): option `2` says that it is not a GeoJSON layout. A layout with an empty `features` list is read as `0 features`.
* **Missing or blank `backoffice.layout.file`:** the default layout is used.
* **No `shoprobot.properties` in the folder where the application is started:** the values of the included file are used.

---

## 🧩 Related Modules & Classes

* **App:** `JavaApp` (package `com.example.shoprobot`)
  * `Main` — builds the Back-office objects and passes the menu to `MainMenu` (option `3`).
  * `view.backoffice.BackofficeMenu` — text UI.
  * `controller.backoffice.BackofficeController` — receives the UI requests.
  * `model.importer.JsonFileReader` — reads JSON files with Jackson.
  * `model.importer.LayoutFileSettings` — JSON layout file from `shoprobot.properties`.
  * `JavaApp/src/main/resources/shoprobot.properties` — settings file with the path of the layout.
  * `model.persistence.DatabaseInfoDao` / `JdbcDatabaseInfoDao` / `ConnectionFactory` — JDBC (USLP03).
* **Errors:** `model.persistence.DaoException`, `java.io.IOException`, `java.nio.file.InvalidPathException`.

---

## 🧱 Class Diagram

![USLP05 Class Diagram](./cd.png)

*Source file:* [cd.puml](./cd.puml)

---

## 🔄 Sequence Diagram

Options `1` - Check database connection and `2` - Check JSON import:

![USLP05 Sequence Diagram](./sd.png)

*Source file:* [sd.puml](./sd.puml)

---

## 🧪 Functional Tests / Validation

| Scenario ID | Test Case / Steps | Expected Result |
| :--- | :--- | :--- |
| **TC-01** | Start the application and choose `3 - Back-office` in the main menu | Back-office menu with title and options `1`, `2`, `0 - Back` |
| **TC-02** | Choose `1` | `Connected to H2 ...` |
| **TC-03** | Choose `2` | `JSON file ...norteshopping.geojson read: 591 features` |
| **TC-04** | Choose `9` | `Invalid option.` and the menu again |
| **TC-05** | Choose `0` | `Leaving Back-office.` and back to the main menu |
| **TC-06** | Set `SHOPROBOT_DB_URL=jdbc:unknown:db`, start and choose `3`, `1` | `Database connection failed: ...`; no crash |
| **TC-07** | Put a `shoprobot.properties` with `backoffice.layout.file=nope.json` in the folder where the application is started, start and choose `3`, `2` | `JSON import failed for nope.json: ...`; no crash |
| **TC-08** | Same, with `backoffice.layout.file` set to a file that is not JSON (e.g. `pom.xml`) | `JSON import failed for ...: Unexpected character ...`; no crash |
| **TC-09** | Same, with `backoffice.layout.file=nope?.json` (invalid path on Windows) | The application starts; option `2` shows `JSON import failed for nope?.json: invalid path (Illegal char <?>)` |
| **TC-10** | Same, with `backoffice.layout.file` set to a JSON that is not a layout (e.g. a file with `[1,2]`) | `JSON file ... read, but it is not a GeoJSON layout (no "features" list)` |
| **TC-11** | Same, with a blank `backoffice.layout.file`, or without `shoprobot.properties` in the folder | Default layout: `read: 591 features` |

Unit tests: `BackofficeMenuTest`, `BackofficeControllerTest`, `JsonFileReaderTest`, `LayoutFileSettingsTest`, `MainMenuTest`, `BackofficeMainTest`.

---

# 🧪 QA & Test Documentation

See Report at [QA & Test Documentation: USLP05 Admin App Skeleton](./qa-readme.md)

---

## 🏁 Definition of Done (DoD)

* [ ] Acceptance criteria verified by QA.
* [x] Unit tests written and passing (`mvn clean test`).
* [x] Class and sequence diagrams in PlantUML and PNG.
* [x] Code documented with Javadoc.
