# [USLP03] Shopping App Skeleton (Java Text UI)

* **Priority:** Urgent
* **Story Points:** 3
* **Module / Component:** `LAPR3` / `JavaApp` — Shopping area (MVC)
* **Status:** Done
* **Developer:** Francisco Marques (1212081)
* **QA:** Tiago Borges (1231109)

---

## 👤 User Story Description

**As a** Product Owner
**I want** the team to create the "skeleton" of the Shopping front-office application
**So that** the customer features (e.g. USLP04) can be added on top of a text UI that is already prepared to access the relational database

---

## 🚦 Definition of Ready (DoR)

* [x] **Clear User Story:** Story follows the standard *As a / I want to / So that* format.
* [x] **Acceptance Criteria Defined:** ACs taken from the project specification (Section 7.1.1).
* [x] **Dependencies Identified:** `JavaApp` Maven project (single MVC application).
* [x] **Inputs/Outputs Mapped:** Menu options and database settings documented below.
* [x] **Estimation Complete:** 3 story points.
* [x] **Small & Actionable:** Fits in Sprint 1.

---

## 💬 Customer Specifications & Clarifications

* **Specification (Section 5.1):** the Shopping application (front office) is used by shopping center customers and is primarily developed in Java/PL-SQL.
* **Specification (Section 7.1.1):** in Sprint 1 the H2 database may be used, as in the template application.
* **Team decision (2026-10-14):** ShopRobot is one Java application in MVC; the Shopping area is option `1` of its main menu (the former `app-one`).
* **Client session (2026-10-02):** the skeleton is a text-based Java console app connected to a relational database via JDBC; in Sprint 1 the database is H2 in memory (confirmed on 2026-10-12).
* **Out of Scope:** listing levels and stores (USLP04, Gonçalo Abreu, already plugged into option `1`); Oracle JDBC driver and PL/SQL API (later sprints).

---

## ✅ Acceptance Criteria

* [x] **AC1 - Text-based UI in Java:** the application shows a console menu, reads the chosen option, handles invalid options and goes back to the main menu on `0` or at the end of the input.
* [x] **AC2 - Prepared to use JDBC:** the application opens a JDBC connection to a relational database configured by environment variables (H2 in memory by default) and option `2` shows the database it is connected to.

---

## 🔒 Non-Functional Requirements & Security

* **NFR02 - Database:** access through JDBC using the DAO pattern (`DatabaseInfoDao` / `JdbcDatabaseInfoDao`); errors wrapped in `DaoException`. One `ConnectionFactory` (also a `DataSource`) is shared by every DAO of the Shopping area, so checking the connection and listing the stores use the same database.
* **NFR03 - Integration:** the database URL, user and password come from environment variables, so the same build can use H2 or Oracle.
* **NFR04 - OOP:** MVC: `view` → `controller` → `model` (`model.persistence` for the DAOs, `model.domain` for the business classes); each class has one responsibility and receives its dependencies in the constructor (testable without a real console or database).
* **Security:** the password is never printed (`DatabaseSettings.toString()` omits it).

---

## 🔗 Found Out Dependencies

### Upstream Dependencies
* **JavaApp:** Maven project, H2 driver and `model.persistence.DaoException`.
* **[USBD04] Relational model:** the H2 tables of the Shopping area (`ShoppingSchema`) follow its tables, keys and checks.

### Downstream Dependencies
* **[USLP04] List stores by level:** option `1` of the Shopping menu opens `view.shopping.ListStoresUI`, which uses `LevelDao` / `StoreDao` (`model.persistence`) with the same `ConnectionFactory`.
* **[USBD05] Oracle database:** to use it, add the Oracle JDBC driver and set the environment variables below.

---

## 📥 Input & Output Data

### Inputs
| Input | Type | Source | Example / Constraints |
| :--- | :--- | :--- | :--- |
| Menu option | Text | Keyboard | `1`, `2` or `0`; anything else is invalid |
| `SHOPROBOT_DB_URL` | Environment variable | Optional | default `jdbc:h2:mem:shoprobot;DB_CLOSE_DELAY=-1` |
| `SHOPROBOT_DB_USER` | Environment variable | Optional | default `sa` |
| `SHOPROBOT_DB_PASSWORD` | Environment variable | Optional | default empty |

### Outputs
| Situation | Output |
| :--- | :--- |
| Start / after each option | Menu: `=== ShopRobot - Shopping ===`, options `1`, `2`, `0 - Back` |
| Option `1` | Store listing of USLP04 (levels, then the stores of the chosen level) |
| Option `2`, database reachable | `Connected to <product> <version>` (e.g. `Connected to H2 2.2.224 (2023-09-17)`) |
| Option `2`, database unreachable | `Database connection failed: Failed to connect to <url>` |
| Invalid option | `Invalid option.` |
| Option `0` or end of input | `Leaving Shopping.` and back to the main menu |

---

## Edge Cases

* **End of input (e.g. Ctrl+Z / Ctrl+D or piped input):** the menu ends normally with `Leaving Shopping.`.
* **Spaces around the option:** ignored (` 2 ` is option `2`).
* **Database unreachable or unknown JDBC URL:** the application does not crash, neither at start-up (it shows `Shopping database not available: …` or `Using the configured database …`) nor in option `2` (`Database connection failed: …`); the other areas of the application keep working.
* **Blank environment variable** (e.g. `SHOPROBOT_DB_URL=`): treated as not set, so the H2 default is used.
* **Sample tables only in H2 in memory:** `ShoppingSchema` creates the Shopping tables and sample data only when the database is H2 in memory. With any other database (e.g. the Oracle database of USBD05) the existing tables and data are never dropped.

---

## 🧩 Related Modules & Classes

* **App:** `JavaApp` (package `com.example.shoprobot`)
  * `Main` — builds the objects and starts the menu.
  * `view.shopping.ShoppingMenu` — text UI.
  * `controller.shopping.ShoppingController` — receives the UI requests.
  * `model.persistence.DatabaseSettings` — database settings from the environment.
  * `model.persistence.ConnectionFactory` — opens JDBC connections.
  * `model.persistence.DatabaseInfoDao` / `model.persistence.JdbcDatabaseInfoDao` — DAO that describes the connected database.
  * `model.persistence.ShoppingSchema` — creates the Shopping tables in H2 in memory, aligned with the relational model (USBD04), with sample data; never drops the tables of another database.
* **Errors:** `model.persistence.DaoException`.

---

## 🧱 Class Diagram

![USLP03 Class Diagram](./cd.png)

*Source file:* [cd.puml](./cd.puml)

---

## 🔄 Sequence Diagram

Option `2` - Check database connection:

![USLP03 Sequence Diagram](./sd.png)

*Source file:* [sd.puml](./sd.puml)

---

## 🧪 Functional Tests / Validation

| Scenario ID | Test Case / Steps | Expected Result |
| :--- | :--- | :--- |
| **TC-01** | Start the application and choose `1 - Shopping` in the main menu | Shopping menu with title and options `1`, `2`, `0 - Back` |
| **TC-02** | Choose `2` | `Connected to H2 ...` |
| **TC-03** | Choose `1` | Store listing of USLP04 opens; after it, the Shopping menu again |
| **TC-04** | Choose `9` | `Invalid option.` and the menu again |
| **TC-05** | Choose `0` | `Leaving Shopping.` and back to the main menu |
| **TC-06** | Set `SHOPROBOT_DB_URL=jdbc:unknown:db`, start the application and choose `1`, `2` | `Using the configured database …` at start-up, then `Database connection failed: ...`; no crash |
| **TC-07** | Choose `1`, list the stores of a level, then choose `2` | Stores are listed and `Connected to H2 ...` refers to the same database |
| **TC-08** | Set `SHOPROBOT_DB_URL=jdbc:h2:tcp://localhost:9/nope` (server not running), start and choose `1`, `2` | `Database connection failed: ...`; the menus keep working |
| **TC-09** | Set `SHOPROBOT_DB_URL=` (blank), start and choose `1`, `2` | `Connected to H2 ...` (default database) |

---

# 🧪 QA & Test Documentation

See Report at [QA & Test Documentation: USLP03 Shopping App Skeleton](./qa-readme.md)

---

## 🏁 Definition of Done (DoD)

* [x] Acceptance criteria verified by QA (Tiago Borges, approved on 2026-10-19).
* [x] Unit tests written and passing (`mvn clean test`).
* [x] Class and sequence diagrams in PlantUML and PNG.
* [x] Code documented with Javadoc.
