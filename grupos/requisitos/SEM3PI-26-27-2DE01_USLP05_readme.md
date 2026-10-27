# [USLP05] Back-Office Application Skeleton

* **Priority:** High
* **Story Points:** TODO
* **Module / Component:** `app-two` / Administration (Back-Office) App
* **Status:** Draft

---

## 👤 User Story Description

**As a** Product Owner  
**I want** the team to create the "skeleton" of the back-office application  
**So that** back-office features (e.g., USLP06 GeoJSON import) can be built on a common, ready-to-use base

---

## 🚦 Definition of Ready (DoR)

* [x] **Clear User Story:** Story follows the standard *As a / I want to / So that* format.
* [x] **Acceptance Criteria Defined:** Taken from the project document (section 7.1.1) and client clarifications.
* [x] **Dependencies Identified:** See Found Out Dependencies.
* [x] **Inputs/Outputs Mapped:** See Input & Output Data.
* [ ] **Estimation Complete:** TODO
* [x] **Small & Actionable:** Split into sub-issues that fit in Sprint 1.

---

## 💬 Customer Specifications & Clarifications

**From the specifications document:**
* Administration application used as a generic back office (primarily BDDAD; Java/PL-SQL).
* All database operations must be done using an API of PL/SQL functions; the DAO pattern must be used with JDBC (NFR02).
* For Sprint 1, the H2 database may be used, as illustrated in the template application code.

**From the client clarifications:**
* **Clarification (2026-10-07):** A skeleton application does not do anything functional for the system, but includes all
  the necessary libraries and support for non-functional requirements. It must compile and execute; when executed, it can
  simply display its identification and exit, similar to the examples already in the repository.
* **Clarification (2026-10-06):** "Prepared to import JSON data" means the application supports importing JSON data,
  similar to being "prepared to use JDBC": the app is ready to start coding JSON import without requiring any further library.
* **Clarification (2026-10-06):** For this sprint, an in-memory database such as H2 is acceptable, but JDBC must be used.
* **Clarification (2026-10-06):** Authentication is not mandatory for this POC. The user's role must be clear (apps or
  menus specific to each role), and logs/error messages may need to identify the user/role performing an operation.
* **Client session (2026-10-02):** The back office must be ready to import JSON/GeoJSON. GeoJSON is a subset of JSON, and
  a GeoJSON-specific library is acceptable and preferred.
* **Client session (2026-10-02):** The import file path comes from a properties file with a default filename, configurable
  without code changes.
* **Client forum (2026-10-16):** The PoC considers only one shopping location (NorteShopping), whose data may be stored in a
  properties file. The database must still support several locations. Re-importing a GeoJSON adds new entries and updates
  existing ones (USLP06).
* **Out of Scope:** Functional features (e.g., importing stores/levels from GeoJSON – USLP06); authentication; Oracle /
  PL-SQL integration (later sprints).

---

## ✅ Acceptance Criteria

* [ ] **AC1 - Text-based UI:** The application is a text-based (console) Java application that compiles, executes, displays its identification ("ShopRobot Back Office - Retailer Administrator"), shows a menu and exits on option `0`.
* [ ] **AC2 - JDBC ready:** The application includes the JDBC dependencies and configuration (H2 in-memory for Sprint 1) and the DAO structure, so database access can be coded without adding libraries.
* [ ] **AC3 - JSON ready:** The application includes a JSON library (Jackson) and a GeoJSON library (geojson-jackson) as dependencies, and can already read a GeoJSON file, so the USLP06 import can be coded without adding libraries.
* [ ] **AC4 - Configurable import file:** The default import file path is read from `backoffice.properties`, (`app-two/src/main/resources`), and can be changed by editing that file, without any Java code changes.

---

## 🔒 Non-Functional Requirements & Security

* **NFR01 - Business Rules:** Data is validated before being registered (validation in the domain objects).
* **NFR02 - Database:** DAO pattern with JDBC (DAO interface + JDBC implementation per entity); H2 in-memory in Sprint 1, ready to switch to the remote Oracle DB / PL/SQL API by changing only the JDBC DAO classes.
* **NFR04 - OOP:** Layered design: `ui` → `controller` → `service` → `domain` / `dao`, with `dto` objects between controller and UI and an `infrastructure` package for configuration, file reading and the H2 schema.
* **NFR05 - Documentation:** Javadoc on public classes; CamelCase; diagrams in SVG.
* **Authorization:** Used by the Retailer Administrator.

---

## 🔗 Found Out Dependencies

### Upstream Dependencies (Required before starting)
* **shared-library:** `DatabaseConfig` (H2 DataSource) and DAO pattern from the repository template.

### Downstream Dependencies (Impacted by this US)
* **[USLP06] Import stores and levels from GeoJSON:** uses the back-office menu, the import controller, the GeoJSON reader and DAOs.

---

## 📥 Input & Output Data

### Inputs
| Field Name | Data Type | Required | Source | Example / Constraints |
| :--- | :--- | :--- | :--- | :--- |
| `menuOption` | Integer | Yes | User Input | `1`..`N`, `0` = Exit |
| `jsonFilePath` | String | No (Enter = default) | User Input | `data/export-norteshopping.geojson` or `C:/Users/me/x.geojson`. Looked up on disk first, then inside the application's resources. |
| `import.file` | String | Yes | `backoffice.properties` | Default import file path (`data/export-norteshopping.geojson`, shipped in the app's resources) |

### Outputs
| Field Name | Data Type | Destination | Example / Description |
| :--- | :--- | :--- | :--- |
| Main menu | Text | Console | Numbered list of available options |
| Import result | `ImportResultDTO` | Console | `"File parsed successfully: N features"` |
| Error message | Text | Console | `"Invalid option"`, `"Database connection failed: ..."`, `"Import failed: ..."` |

---

## Edge Cases

* **Invalid menu input:** Letters or out-of-range numbers → error message, menu shown again.
* **Database unavailable:** Clear message at startup instead of a stack trace.
* **Missing / malformed JSON file:** Clear error message; the user returns to the menu.
* **Invalid file path** (e.g. `a<b` on Windows): Clear error message; the user returns to the menu.
* **Missing `backoffice.properties` or `import.file` key:** Clear error message instead of a crash.
* **End of input:** The application exits cleanly (no exception).
* **Quoted path** (Windows "Copy as path"): Surrounding quotes are removed before the path is used.

---

## 🧪 Functional Tests / Validation

| Scenario ID | Test Case / Steps | Expected Result |
| :--- | :--- | :--- |
| **TC-01** | Start the application | Identification is shown; DB connection opens; main menu is displayed |
| **TC-02** | Type `abc` or `99` in the menu | "Invalid option" shown; menu displayed again |
| **TC-03** | Choose Exit | Application ends cleanly |
| **TC-04** | Load a valid GeoJSON file | "File parsed successfully: N features" (the lines after it, with imported levels and stores, belong to USLP06) |
| **TC-05** | Load a non-existent or malformed file, or type an invalid path | Clear error message; back to the menu |
| **TC-06** | Choose option 1 and press Enter without typing a path | The default file from `backoffice.properties` is shown and used. Change `import.file` in `backoffice.properties` and run the app again: the new default appears, with no Java code change. Use forward slashes in the path (`C:/Users/me/x.geojson`), because backslashes are escape characters in `.properties` files. Revert the change after the test. |
| **TC-07** | Option 1, paste a path copied with Windows "Copy as path" (wrapped in quotes) | The quotes are ignored and the file is loaded |

---

# 🧪 QA & Test Documentation

See Report at [QA & Test Documentation: USLP05 Back-Office Application Skeleton](./qa-readme.md)

---

## 🧩 Related Modules & Classes

* **app-two (USLP05):**
  * Entry point: `Main`
  * UI: `MainMenuUI` (built from a list of `MenuItem`), `ImportGeoJsonUI`
  * Controller: `ImportGeoJsonController`
  * DTO: `ImportResultDTO`
  * Infrastructure: `PropertiesReader`, `GeoJsonReader`, `ImportException`
  * Resources: `backoffice.properties` and `data/export-norteshopping.geojson`; tests use `test-backoffice.properties` and `test-data/sample.geojson`
* **app-two (added by USLP06, documented there):**
  * Service: `GeoJsonImportService`, `GeoJsonMapper`, `GeoJsonMappingResult`
  * Domain: `ShoppingLocation`, `Level`, `Area`, `AreaType`, `Coordinate`, `Store`
  * DAO: `GeoJsonImportDao`, `JdbcShoppingLocationDao`, `JdbcLevelDao`, `JdbcAreaDao`, `JdbcStoreDao`
  * Infrastructure: `BackOfficeSchema` (creates the H2 tables)
* **shared-library:**
  * `DatabaseConfig`, `DaoException` (`com.example.dao`)
  * `UserRole` (`com.example.domain`)
  * `MenuItem` (`UI`)

---

## ▶️ How to Run

* **Database:** nothing to configure. Sprint 1 uses H2 in-memory, set up in `DatabaseConfig` (shared-library). It starts empty on every run.
* **Run in IntelliJ:** Maven panel → `java-apps` → Lifecycle → `install` (builds shared-library first), then run `com.example.apptwo.Main`. The working directory doesn't matter.
* **Change the default import file:** edit `import.file` in `app-two/src/main/resources/backoffice.properties` and run again. Use an absolute path with forward slashes (`C:/Users/me/file.geojson`) for a file on disk, or a path inside the resources (`data/export-norteshopping.geojson`).
* **Tests and coverage:** Maven panel → Lifecycle → `verify`; the report is in `app-two/target/site/jacoco/index.html`.

---

## 🔄 Sequence Diagram

![Sequence Diagram](sd.svg)

## 🏁 Definition of Done (DoD)

* [ ] Acceptance criteria verified by QA (a team member other than the author).
* [ ] Unit tests written and passing; coverage of the USLP05 classes above 80%.
* [ ] `mvn -B clean verify` passes.
* [ ] Javadoc on all new public classes and methods.
* [ ] Sequence diagram committed as `sd.puml` and `sd.svg`.