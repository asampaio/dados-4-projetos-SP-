# [USLP05] Back-Office Application Skeleton

* **Priority:** TBD (to be defined in Sprint Planning)
* **Story Points:** 3 (size S, as estimated on the project board)
* **Module / Component:** `Back-Office App` (`app-three`) / `Application Skeleton`
* **Status:** In Review (implementation done, 15 JUnit tests passing; manual functional tests and review by another team member pending)

---

## 👤 User Story Description

**As a** Product Owner
**I want** the team to create the "skeleton" of the back-office application
**So that** the team has a working foundation (text-based UI, database access through JDBC and JSON import readiness) on which the next back-office stories, starting with USLP06, can be built

---

## 🚦 Definition of Ready (DoR)

* [x] **Clear User Story:** Story follows the standard *As a / I want to / So that* format.
* [x] **Acceptance Criteria Defined:** Taken directly from the project requirements document.
* [x] **Dependencies Identified:** See "Found Out Dependencies" below.
* [x] **Inputs/Outputs Mapped:** See "Input & Output Data" below.
* [x] **Estimation Complete:** 3 story points, size S (project board).
* [x] **Open Clarifications Resolved:** the GeoJSON path and the database in Sprint 1 were clarified by the client (see below).

---

## 💬 Customer Specifications & Clarifications

* **Requirements doc (LAPR3, Sprint 1):** for the database, H2 may be used, as illustrated in the template application code in the repository.
* **Requirements doc (envisioned architecture):** the back-office is the *Administration application* (primarily BDDAD; Java/PL-SQL), a generic back office that shares the relational database with the Shopping and Planning applications.
* **Requirements doc (USLP06):** the back-office user is the Retailer Administrator, who will use it to import the stores and levels of a shopping location from a GeoJSON file.
* **Client session (2 Oct 2026):** the skeleton is a compilable, text-based Java console app connected to a relational database through JDBC; the back-office skeleton must also be ready to import JSON/GeoJSON.
* **Client session (2 Oct 2026):** GeoJSON is a subset of JSON; a GeoJSON-specific library is acceptable and preferred.
* **Client session (2 Oct 2026):** "File path for import: use a properties file with a default filename, configurable without code changes."
* **Client session (12 Oct 2026):** the back-office is the Administrative application (back office and management); in Sprint 1 the database is H2 in memory, accessed through JDBC.
* **Client session (16 Oct 2026):** an in-memory H2 database is accepted to fulfil the non-functional requirements of Sprint 1; the proof of concept targets a single shopping location (NorteShopping).
* **Requirements doc (NFR02):** the information is kept in a remote relational DBMS, "all database operations must be done using an API of PL/SQL functions" and the DAO pattern should be used for working with JDBC. In Sprint 1 the client accepted H2 in memory to fulfil the non-functional requirements (16 Oct 2026), so JDBC with the DAO pattern is used without PL/SQL functions.
* **Requirements doc (NFR03):** "a significant part of the integration must be carried out through files, program arguments, or environment variables", which supports reading the GeoJSON path from a properties file.
* **Team decision (module):** `app-one` is the Shopping app, `app-two` the Planning app (ESINF) and `app-three` the back-office (Administration app), in package `com.example.appthree`; the template `JavaApp/README.md` asks not to rename the modules.
* **Decision (JSON readiness):** "prepared to import JSON data" means that a JSON library is available in `app-three` and can read a GeoJSON `FeatureCollection`, and that the menu already has an "Import Stores & Levels" option, which runs the stub `ImportGeoJSONUI`; the real import is implemented in [USLP06](../USLP06/readme.md), which replaces the body of the stub.
* **Decision (library, with USLP06):** Jackson Databind (`com.fasterxml.jackson.core:jackson-databind`), added to `app-three/pom.xml` by USLP06 (#74). The GeoJSON-specific library `geojson-jackson` was not used: its last version for Jackson 2 (1.14, from 2020) does not work with current Jackson 2 releases, and the newer versions require Jackson 3.
* **Decision (configuration file):** the properties file is `backoffice.properties`, on the classpath of `app-three`, with the key `geojson.file.path` (default value `export-norteshopping.geojson`, relative to the folder where the application is started; USLP06 can change it without changing the code). Paths use `/`, also on Windows, because `\` is an escape character in a properties file. A missing or malformed file, or a missing key, does not stop the application; the error is shown only when "Import Stores & Levels" is chosen.
* **Decision (database on start-up):** the application gets the `DataSource` from `DatabaseConfig` (shared-library) and creates the `levels`, `areas` and `stores` tables with `ShoppingSchemaInitializer` (shared-library, created in [USLP04](../USLP04/readme.md)), which USLP06 needs to insert the imported data. The tables follow the domain model (a Level contains Areas and a Store is located in an Area); there is no shopping location table, because the proof of concept has a single shopping location (client session, 16 Oct 2026). The template `DatabaseConfig.initialize()` is not called, because it only creates the template `users` table.
* **Decision (menu format, agreed by the team):** the same console structure in the three apps: `ui.console.menu.MainMenuUI` creates the options as `ui.console.menu.MenuItem` (text and the UI it runs), there is one UI per user story implementing `Runnable` (in `ui.console`), and the menu has the same look as the `Utils` class of `ui.console.utils` (`  1 - …`, `  0 - Cancel`, `Type your option:`). The menu reads and writes through the `Scanner` and `PrintStream` it receives, as `StoreListing` in [USLP04](../USLP04/readme.md), so it can be tested with JUnit; `Utils` reads `System.in` directly and cannot be used with simulated input.
* **Team decision (database, 21 Oct 2026):** Oracle is used for the BDDAD stories (USBD05, on Oracle FreeSQL); the applications and their unit tests use H2 in memory in Sprint 1, as accepted by the client. The move of the applications to Oracle is planned for later sprints and only changes `DatabaseConfig` (shared-library), not the back-office code, because the skeleton only asks `DatabaseConfig` for a `DataSource`.
* **Note (start-up tables vs. relational model):** `ShoppingSchemaInitializer` creates a simplified subset of the relational model: the tables `levels`, `areas` and `stores`, without a shopping location table and with `level_number INTEGER`. The DDL of USBD05 has `SHOPPING_LOCATION`, `SHOPPING_LEVEL` (with `level_number NUMBER(5,2)`, because the GeoJSON has levels such as 0.5), `AREA` and `STORE`. The shared schema (USLP04) has to be aligned with it before the import of USLP06 and the move to Oracle; the skeleton only calls `ShoppingSchemaInitializer`, so its code does not change.
* **Out of Scope:** the real GeoJSON import and the insertion of levels and stores ([USLP06](../USLP06/readme.md)), the move of the applications to Oracle (later sprints), any front-office feature (USLP03).

---

## ✅ Acceptance Criteria

* [x] **AC1 - Text-Based UI:** The application is a text-based UI, developed in Java; it shows a main menu, accepts the user's option, rejects invalid options with a clear message and exits cleanly.
* [x] **AC2 - JDBC Readiness:** The application is prepared to use JDBC to access a relational database: on start-up it obtains the `DataSource` from `DatabaseConfig` (shared-library) and prepares the shopping schema through JDBC (`ShoppingSchemaInitializer`); a connection error is reported with a clear message and the application exits.
* [x] **AC3 - JSON Readiness:** The application is prepared to import JSON data: a JSON library (Jackson Databind) is available in `app-three` and reads a GeoJSON `FeatureCollection`; the "Import Stores & Levels" option reads the path from `backoffice.properties` and can be selected without throwing an exception.

---

## 🔒 Non-Functional Requirements & Security

* **NFR01 (Business Rules):** the skeleton does not register or update data; the business rules of the import (e.g., a store without a level is skipped) are validated in USLP06.
* **NFR02 (Database):** the skeleton accesses the database through JDBC (`DatabaseConfig` and `ShoppingSchemaInitializer`); the data access of the next back-office stories uses the DAO pattern of the `shared-library` (as `LevelDao` and `StoreDao` in USLP04). The skeleton does not call PL/SQL functions: it does not read or write data, and H2 is used in Sprint 1 (see above).
* **NFR03 (Integration):** Java; text-based user interface; the GeoJSON path is read from a properties file.
* **NFR04 (OOP):** the menu, the menu options, the import UI and the configuration are separate classes, so the next back-office stories add a menu option or replace the stub with few changes.
* **NFR05 (Documentation):** Javadoc on public classes/methods, camelCase coding standard, diagrams in SVG.
* **Process:** TDD (Red-Green-Refactor locally; Red tests disabled when merged into `main`), at least one commit per day.
* **Security:** not applicable (no authentication or sensitive data involved).

---

## 🔗 Found Out Dependencies

### Upstream Dependencies (Required before starting)
* **Repository template (`app-three` and `shared-library`):** `app-three` already exists as a Maven module and `shared-library` already provides `DatabaseConfig`, `DaoException` and the DAO structure. The template `Main` of `app-three` is replaced by the back-office code.
* **[USLP04](../USLP04/readme.md) List available stores:** `ShoppingSchemaInitializer` (shared-library) creates the `levels`, `areas` and `stores` tables used on start-up.
* **[USLP06](../USLP06/readme.md) GeoJSON import (#74):** Jackson Databind declared in `app-three/pom.xml`.
* **[USBD05](../../BDDAD/USBD05/readme.md) Relational model instantiated:** the tables in Oracle that the shared schema has to follow when the applications move to Oracle (see the note above).
* **Domain Model ([USBD03](../../BDDAD/USBD03/readme.md)/[USLP01](../USLP01/readme.md)):** `Level`, `Area`, `Store`, followed by the tables prepared on start-up.

### Downstream Dependencies (Impacted by this story)
* **[USLP06](../USLP06/readme.md) GeoJSON Import:** replaces the body of the stub `ImportGeoJSONUI` with the real import, using the JSON library, the configured path and the menu created here.
* **Future back-office stories:** add new options (`MenuItem`) to the menu created here.

---

## 📥 Input & Output Data

### Input Data
| Data             | Type                | Description                                        |
|:-----------------|:--------------------|:---------------------------------------------------|
| Menu option      | `String` (console)  | Option typed by the user                           |
| GeoJSON path     | Properties file     | Value of `geojson.file.path` in `backoffice.properties` |

### Output Data
| Data      | Type    | Description                                                                   |
|:----------|:--------|:------------------------------------------------------------------------------|
| Main menu | Console | Available options ("1 - Import Stores & Levels" and "0 - Cancel")              |
| Messages  | Console | Import not yet implemented, invalid option, missing configuration, database error or "Goodbye." |

---

## Edge Cases

* **Database unavailable on start-up:** a clear error message is shown and the application exits without an unhandled exception.
* **Invalid menu option (letters, empty, out of range):** rejected with a clear message and the menu is shown again.
* **Properties file missing or malformed (e.g., a Windows path such as `C:\users\…`, where `\u` is an invalid escape), or `geojson.file.path` missing or blank:** the application starts normally; a clear message is shown when "Import Stores & Levels" is selected and the menu is shown again.
* **End of input (e.g., the input stream is closed):** the application exits cleanly, without an exception.
* **Menu option without text or without UI:** rejected when the option is created (`IllegalArgumentException`).
* **JSON library missing:** a JUnit test reads a GeoJSON `FeatureCollection` with the library, so removing the dependency breaks the build.

---

## 🧪 Functional Tests / Validation

| Scenario ID | Scenario | Expected Result |
| :--- | :--- | :--- |
| **F-01** | Start the application | The main menu is shown with "1 - Import Stores & Levels" and "0 - Cancel" |
| **F-02** | Select "0 - Cancel" | The application exits cleanly |
| **F-03** | Type an invalid option | Error message and the menu is shown again |
| **F-04** | Database available on start-up | The connection is established, the tables are created and the menu is shown |
| **F-05** | Database unavailable on start-up | Clear error message and the application exits without crashing |
| **F-06** | Select "Import Stores & Levels" | Placeholder message ("not implemented yet") and the menu is shown again; no exception |
| **F-07** | Select "Import Stores & Levels" without `geojson.file.path` configured | Clear error message and the menu is shown again |
| **F-08** | Input ends while the menu is shown | The application exits cleanly |
| **F-09** | Start the application with a malformed `backoffice.properties` (e.g., `geojson.file.path=C:\users\…`) and select "Import Stores & Levels" | The application starts normally; the "path is not configured" error is shown and the menu is shown again |

---

# 🧪 QA & Test Documentation

See Report at [QA & Test Documentation: USLP05 Back-Office Application Skeleton](./qa-readme.md)

---

## 🧩 Related Modules & Classes

* **Module:** `app-three` (package `com.example.appthree`), using `shared-library` for `DatabaseConfig` and the DAO code.
* **Classes:**
    * `Main` — entry point; gets the `DataSource` from `DatabaseConfig`, loads `AppConfig`, prepares the schema with `ShoppingSchemaInitializer` and starts the main menu; if the database cannot be prepared, shows a clear message and exits
    * `AppConfig` — reads `backoffice.properties` from the classpath and returns the value of `geojson.file.path`, if any
    * `ui.console.menu.MainMenuUI` — main menu: creates the options, shows them numbered with `0 - Cancel`, runs the chosen one, rejects invalid options and ends at the end of input; receives `AppConfig`, `Scanner` and `PrintStream`
    * `ui.console.menu.MenuItem` — one menu option: the text shown and the UI it runs (`Runnable`); rejects a blank text or a null UI
    * `ui.console.ImportGeoJSONUI` — stub of the import UI (name from [USLP06](../USLP06/readme.md)); shows the configured path and a "not implemented yet" message, or an error if the path is not configured; USLP06 replaces its body
* **Tests:** `MainTest`, `AppConfigTest`, `JsonLibraryTest`, `ImportGeoJSONUITest`, `MainMenuUITest` and `MenuItemTest` (15 JUnit tests).
* **Configuration:** `app-three/src/main/resources/backoffice.properties`.

---

## 🔄 Sequence Diagram

![Sequence Diagram](./sd.svg)

## 🏁 Definition of Done (DoD)

* [x] All acceptance criteria verified.
* [x] Unit tests written (TDD) and passing.
* [ ] Reviewed by a team member other than the author.
* [x] Javadoc written for public classes and methods.
* [ ] Final sign-off recorded in `qa-readme.md`.