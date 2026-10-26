# [USLP03] Shopping Front-Office Application Skeleton

* **Priority:** High
* **Story Points:** 3
* **Module / Component:** `Shopping App` (front-office, Java)
* **Status:** Draft

---

## 👤 User Story Description

**As a** Product Owner  
**I want** the team to create the "skeleton" of the Shopping front-office application  
**So that** the following customer features (e.g. USLP04) can be built on a common, database-ready structure

---

## 🚦 Definition of Ready (DoR)

* [x] **Clear User Story:** Story follows the standard *As a / I want to / So that* format.
* [x] **Acceptance Criteria Defined:** Clear, testable ACs agreed upon by Product and Engineering.
* [x] **Dependencies Identified:** Template application code (Maven multi-module project) available in the team repository; Oracle database defined in the relational model (USBD04/USBD05).
* [x] **Inputs/Outputs Mapped:** Menu options, JDBC configuration and error states are documented.
* [ ] **Estimation Complete:** Story has been refined and estimated (story points assigned) by the dev team.
* [x] **Small & Actionable:** Story is scoped down enough to be completed within a single sprint.

---

## 💬 Customer Specifications & Clarifications

* **Source (project description, USLP03):** The application must be a text-based UI developed in Java and prepared to use JDBC to access a relational database.
* **Team decision:** The team does not use H2 in the application. The target database is Oracle (DDL in `database/ddl.sql`). The automated unit tests use an in-memory H2 database (already provided by the template's `shared-library`), so they do not need Oracle.
* **Out of Scope:** Any business feature (listing levels/stores is [USLP04]); the application only provides the structure and a menu.

---

## ✅ Acceptance Criteria

* [ ] **AC1 - Text-based UI:** The application runs in the console and shows a main menu implemented in Java.
* [ ] **AC2 - JDBC Ready:** The application contains the infrastructure to open/close a JDBC connection to a relational database (configurable, no hard-coded credentials).
* [ ] **AC3 - Navigation:** Invalid menu options display a clear message and the menu is shown again; the user can exit cleanly.
* [ ] **AC4 - Connection Failure:** If the database is unreachable, a clear error message is displayed and the application terminates gracefully.

---

## 🔒 Non-Functional Requirements & Security

* **NFR02 - Database:** Database access must follow the Data Access Object (DAO) pattern and, as far as possible, use PL/SQL function APIs.
* **NFR03 - Integration:** Main language is Java.
* **NFR04 - OOP:** Design must allow easy addition of new features (new menu options).
* **NFR05 - Documentation:** Javadoc and coding standards (CamelCase); figures in SVG.
* **Security:** Database credentials must be read from configuration, not committed in source code.
* **Authorization:** Feature accessible to the `CUSTOMER` role (no authentication in this sprint).

---

## 🔗 Found Out Dependencies

### Upstream Dependencies (Required before starting)
* **Repository template:** Provides the Maven multi-module structure (`shared-library`, `app-one`) and the `DaoException`/DAO conventions.
* **Oracle schema (USBD05):** `database/ddl.sql` creates the tables the next stories will query.

### Downstream Dependencies (Impacted by this story)
* **[USLP04] List available stores:** Will add a menu option and DAO on top of this skeleton.

---

## 📥 Input & Output Data

### Inputs
| Field Name              | Data Type | Required | Source                                                 | Example / Constraints                    |
|:------------------------|:----------|:---------|:-------------------------------------------------------|:-----------------------------------------|
| `menuOption`            | Integer   | Yes      | User Input (console)                                   | `0` = Exit; must belong to the menu      |
| `jdbcUrl`               | String    | Yes      | `db.url` in `shopping.properties` or env `DB_URL`      | `jdbc:oracle:thin:@//host:1521/service`  |
| `dbUser` / `dbPassword` | String    | Yes      | `db.user`/`db.password` or env `DB_USER`/`DB_PASSWORD` | Never hard-coded; env overrides the file |

### Outputs
| Field Name     | Data Type | Destination | Example / Description                                        |
|:---------------|:----------|:------------|:-------------------------------------------------------------|
| `mainMenu`     | Text      | Console     | List of numbered options                                     |
| `errorMessage` | String    | Console     | `"Invalid option."` / `"Could not connect to the database."` |

---

## Edge Cases

* **Non-numeric input:** Typing text instead of a number must not crash the application.
* **Missing configuration:** Missing/invalid JDBC properties produce a clear error message.
* **Connection lost during use:** Error is reported and the user returns to the menu or exits safely.

---

## 🧪 Functional Tests / Validation

| Scenario ID | Test Case / Steps                                   | Expected Result                                        |
|:------------|:----------------------------------------------------|:-------------------------------------------------------|
| **TC-01**   | Start the application with the database available   | Main menu is displayed                                 |
| **TC-02**   | Enter an invalid option (`99`, `abc`)               | Error message; menu is displayed again                 |
| **TC-03**   | Start the application with the database unavailable | Clear error message; application terminates gracefully |
| **TC-04**   | Select "Exit"                                       | Connection is closed and application ends              |
| **TC-05**   | Search the source code for credentials              | Credentials only in the configuration                  |
| **TC-06**   | Start the application without configuration         | Clear error message naming `DB_URL` / `db.url`; exit 1 |

---

# 🧪 QA & Test Documentation

See Report at [QA & Test Documentation: USLP03 Shopping Front-Office Skeleton](./qa-readme.md)

---

## 🧩 Related Modules & Classes

* **Shopping App (Java, module `app-one`):**
  * Entry point: `Main`
  * UI: `ShoppingUI` (the options are `MenuItem`s of the shared library, so new options are added without changing `ShoppingUI`)
* **Shared library (module `shared-library`):**
  * Infrastructure: `DatabaseConnection` (JDBC, configuration from file/environment), `DaoException`, DAO interfaces for future features
  * UI utilities: `MenuItem` (menu option shared by the three applications)

---

## 🔄 Sequence Diagram

![Sequence Diagram](./sd.svg)

## 🏁 Definition of Done (DoD)

* [ ] Acceptance criteria verified by someone other than the author.
* [ ] Unit tests written and passing (TDD).
* [ ] Javadoc written and coding standards applied.
* [ ] Sequence diagram `sd.puml` and `sd.svg` committed to the repository.
* [ ] Feature merged into `main` (trunk-based) with the release tagged.