# [USLP06] Import Stores and Levels from GeoJSON

* **Priority:** *(as in issue [#6](https://github.com/Departamento-de-Engenharia-Informatica/SEM3PI-26-27-2DE02/issues/6))*
* **Module / Component:** `LAPR3` / `JavaApp` — Back-office area (MVC)
* **Status:** In Progress
* **Developer:** Tiago Borges (1231109)
* **QA:** Gonçalo Abreu (1231536)

---

## 👤 User Story Description

**As a** Retailer Administrator
**I want** to use the back-office application to import the stores and levels of a shopping location from a GeoJSON file
**So that** the stores available at the shopping location are created

---

## 🚦 Definition of Ready (DoR)

* [x] **Clear User Story:** Story follows the standard *As a / I want to / So that* format.
* [x] **Acceptance Criteria Defined:** ACs taken from issue #6 (project specification).
* [x] **Dependencies Identified:** Back-office skeleton (USLP05), Shopping database tables (USLP03 / USLP04, USBD04), GeoJSON layout of the repository.
* [x] **Inputs/Outputs Mapped:** GeoJSON fields, properties and messages documented below.
* [x] **Small & Actionable:** Fits in Sprint 1.

---

## 💬 Customer Specifications & Clarifications

* **Client session (2026-10-02):** the back-office imports JSON/GeoJSON using a **properties file for the input path**. The mandatory GeoJSON fields are, at least, the **store name** and the **floor/level**; a store without a level is **skipped and a warning is logged**. Sprint 1 uses the H2 in-memory database; Oracle comes in later sprints.
* **Client session (2026-10-16):** the GeoJSON import (US6) creates the levels and stores of the shopping location; the planning algorithms use the validated data of the database. H2 in memory is accepted for Sprint 1.
* **Team decisions (USLP06):**
  * Only the features with a `shop` property are stores; corridors, amenities, walls, etc. are not imported.
  * A store without a name is skipped with a warning, like a store without a level (both are mandatory).
  * A level must be one whole level (e.g. `0`, `-2`): features between levels (`0.5`) or on several levels (`0;1`) are skipped with a warning, because `level_number` is an integer in the database.
  * A store without geometry is skipped with a warning, because the polygon of an area is mandatory (`AREA.polygon_coordinates NOT NULL`).
  * Only the levels that have stores are imported. The layout has no name nor height for the levels: a new level is named `Level <n>` and has the default height of **4.0 m** (`height_meters` is mandatory and must be greater than 0). *To be confirmed with the database team (USBD).* Levels that already exist keep their name and height.
  * The area id of a store is the OpenStreetMap id of the feature (`@id`, e.g. `way/1368109278`); the category is the value of `shop`; the store is open; the shop code is empty (`NULL`), because the layout has none.
  * The data is saved in **one transaction**: if a store cannot be saved, nothing is saved. Importing the same file again fails because the stores already exist, and nothing is saved. *How to re-import a layout is still to be decided.*
  * The location of the stores is set in `shoprobot.properties` and must already exist in the database (`NS01`, NorteShopping, in the sample data).
* **Out of Scope:** other areas of the layout (corridors, robot stations, amenities); updating or deleting stores of an earlier import; Oracle (later sprints).

---

## ✅ Acceptance Criteria

* [ ] **AC1 - Import the GeoJSON file:** the application imports the data from the provided GeoJSON file (path set in `shoprobot.properties`).
* [ ] **AC2 - List the imported data:** the levels and the stores of each level are shown to the user.
* [ ] **AC3 - Insert into the database:** the imported levels and stores are inserted into the database.
* [ ] **AC4 - Show the import errors:** the features that were skipped (with the reason) and the errors of the file or of the database are clearly displayed.

---

## 🔒 Non-Functional Requirements & Security

* **NFR02 - Database:** JDBC with the DAO pattern (`LayoutImportDao` / `JdbcLayoutImportDao`), `PreparedStatement` for every statement, one transaction (`commit` / `rollback`); errors wrapped in `DaoException`.
* **NFR03 - Integration:** the file and the location come from `shoprobot.properties` (`backoffice.layout.file`, `backoffice.layout.location`), so another layout can be imported without code changes. A `shoprobot.properties` in the folder where the application is started replaces the included values.
* **NFR04 - OOP:** MVC: `view.backoffice.ImportLayoutUI` → `controller.backoffice.ImportLayoutController` → `model` (`model.importer` reads the GeoJSON with Jackson, `model.persistence` saves it). The parser does not print anything: the skipped features are returned as warnings and the UI shows them.

---

## 🔗 Found Out Dependencies

### Upstream Dependencies
* **[USLP05] Back-office App Skeleton:** Back-office menu, Jackson and `shoprobot.properties` (`LayoutFileSettings`).
* **[USLP03] / [USLP04]:** `ConnectionFactory`, the H2 tables `SHOPPING_LEVEL`, `AREA` and `SHOP` (`ShoppingSchema`) and the domain classes `ShoppingLevel` and `Store`.
* **[USBD04] Relational model:** keys and constraints of `SHOPPING_LEVEL`, `AREA` and `SHOP`.
* **Layout file:** `data/geojson/norteshopping.geojson` (OpenStreetMap export).

### Downstream Dependencies
* **[USLP04] List stores:** the imported levels and stores are listed in the Shopping area.
* **Planning (ESINF) and later sprints:** use the stores and levels of the database.

---

## 📥 Input & Output Data

### Inputs
| Input | Type | Source | Example / Constraints |
| :--- | :--- | :--- | :--- |
| Menu option | Text | Keyboard | `3 - Back-office`, then `3 - Import layout (stores and levels)` |
| `backoffice.layout.file` | Property of `shoprobot.properties` | Optional | default `data/geojson/norteshopping.geojson` (use `/` in the paths) |
| `backoffice.layout.location` | Property of `shoprobot.properties` | Optional | default `NS01`; must exist in `SHOPPING_LOCATION` |
| Feature `properties.shop` | GeoJSON | File | Category of the store (e.g. `clothes`); features without it are not stores |
| Feature `properties.name` | GeoJSON | File | Mandatory |
| Feature `properties.level` | GeoJSON | File | Mandatory; one whole level (e.g. `0`, `1`, `-2`) |
| Feature `properties.@id` | GeoJSON | File | Area id of the store (e.g. `way/1368109278`) |
| Feature `geometry.coordinates` | GeoJSON | File | Mandatory; saved as the polygon of the area |

### Outputs
| Situation | Output |
| :--- | :--- |
| Start of the import | `--- Import Layout ---` and `File: <file> (location <id>)` |
| Stores read | `<n> stores read on levels <l1>, <l2>, …`, then `Level <l> (<k> stores):` and `  - <name> (<category>)` for each store |
| Features skipped | `<n> features skipped:` and `  - <@id>: <reason>` (e.g. `way/1368109305: store without name, skipped`) |
| Data saved | `Saved in the database: <n> stores, <m> new levels` |
| Location not in the database | `Nothing was saved in the database. Location <id> does not exist in the database` |
| Store already imported | `Nothing was saved in the database. Store <name> (<area id>) already exists in the database` |
| Other database error | `Nothing was saved in the database. Failed to save the imported layout: <reason>` |
| No stores in the file | `No stores to save.` |
| Invalid path, missing file or invalid JSON | `Layout import failed for <file>: <reason>` |

With the layout of the repository: 591 features, 197 of them shops; **192 stores** imported (level `-2`: 1, level `0`: 118, level `1`: 73) and **5 warnings** (shops `vacant` without name). In the sample database, levels `0` and `1` of `NS01` already exist, so only level `-2` is created.

---

## Edge Cases

* **Store without name, level or geometry; level `0.5` or `0;1`:** skipped and listed as a warning; the other stores are imported.
* **Level that already exists in the database:** kept as it is (name, description and height).
* **Store that already exists (e.g. the same file imported twice):** nothing is saved and `Store <name> (<area id>) already exists in the database` is shown; the menu keeps working.
* **Location that does not exist in the database:** nothing is saved and `Location <id> does not exist in the database` is shown.
* **File without stores:** nothing is saved (`No stores to save.`).
* **Invalid path (e.g. `nope?.json`), missing file or invalid JSON:** `Layout import failed for …`; the application keeps running.
* **Shop code:** saved as `NULL` (the column is optional and unique, so two stores without a code do not collide).

---

## 🧩 Related Modules & Classes

* **App:** `JavaApp` (package `com.example.shoprobot`)
  * `Main` — creates the import objects and passes `ImportLayoutUI` to the Back-office menu.
  * `view.backoffice.BackofficeMenu` (USLP05) — option `3 - Import layout (stores and levels)`.
  * `view.backoffice.ImportLayoutUI` — shows the levels, stores, warnings and the result of saving.
  * `controller.backoffice.ImportLayoutController` — reads the file, parses it and saves the result.
  * `model.importer.GeoJsonParser` — reads the stores of the GeoJSON with Jackson and validates them.
  * `model.importer.GeoJsonImportResult` / `GeoJsonWarning` — stores, polygons, levels and warnings of an import.
  * `model.importer.LayoutFileSettings` (USLP05) / `LayoutLocationSettings` — file and location from `shoprobot.properties`.
  * `model.persistence.LayoutImportDao` / `JdbcLayoutImportDao` — saves the levels and stores in one transaction.
  * `model.domain.ShoppingLevel` / `Store` (USLP04) — domain classes.
* **Errors:** `java.io.IOException`, `IllegalArgumentException` (invalid JSON, `InvalidPathException`), `model.persistence.DaoException`.

---

## 🧱 Class Diagram

![USLP06 Class Diagram](./cd.svg)

*Source file:* [cd.puml](./cd.puml)

---

## 🔄 Sequence Diagram

Option `3 - Import layout (stores and levels)` of the Back-office menu:

![USLP06 Sequence Diagram](./sd.svg)

*Source file:* [sd.puml](./sd.puml)

---

## 🧪 Functional Tests / Validation

| Scenario ID | Test Case / Steps | Expected Result |
| :--- | :--- | :--- |
| **TC-01** | Start the application, choose `3 - Back-office`, then `3` | `File: data/geojson/norteshopping.geojson (location NS01)` and `192 stores read on levels -2, 0, 1` |
| **TC-02** | Same as TC-01 | `Level -2 (1 stores):` with `  - Tiendanimal (pet)`, `Level 0 (118 stores):`, `Level 1 (73 stores):` |
| **TC-03** | Same as TC-01 | `5 features skipped:` with the 5 shops without name |
| **TC-04** | Same as TC-01 | `Saved in the database: 192 stores, 1 new levels` |
| **TC-05** | After TC-04, choose `0`, `1 - Shopping`, `1`, level `-2` | `[-2] Level -2 (Imported from the GeoJSON layout)` and `  - Tiendanimal …` |
| **TC-06** | After TC-04, choose `3` again in the Back-office menu | `Nothing was saved in the database. Store Calzedonia (way/1368109278) already exists in the database`; no crash |
| **TC-07** | Put a `shoprobot.properties` with `backoffice.layout.file=nope.json` in the folder where the application is started, then TC-01 | `Layout import failed for nope.json: …`; no crash |
| **TC-08** | Same, with a file that is not JSON | `Layout import failed for …: The file is not valid JSON …` |
| **TC-09** | Same, with `backoffice.layout.location=XX99` (location not in the database) | `Nothing was saved in the database. Location XX99 does not exist in the database`; no crash |

Unit tests: `GeoJsonParserTest`, `ImportLayoutControllerTest`, `ImportLayoutUITest`, `JdbcLayoutImportDaoTest`, `LayoutLocationSettingsTest`, `BackofficeMenuTest`, `BackofficeMainTest`.

---

# 🧪 QA & Test Documentation

To be added by QA (`qa-readme.md`).

---

## 🏁 Definition of Done (DoD)

* [ ] Acceptance criteria verified by QA.
* [x] Unit tests written and passing (`mvn clean test`).
* [x] Class and sequence diagrams in PlantUML and SVG.
* [x] Code documented with Javadoc.
