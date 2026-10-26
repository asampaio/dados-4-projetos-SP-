# 🧪 QA & Test Documentation: [USLP03] Shopping App Skeleton (Java Text UI)

* **User Story Reference:** [USLP03: Shopping App Skeleton (Java Text UI)](./readme.md) — GitHub issue [#3](https://github.com/Departamento-de-Engenharia-Informatica/SEM3PI-26-27-2DE02/issues/3)
* **QA Lead / Owner:** Tiago Borges (1231109)
* **Developer:** Francisco Marques (1212081)
* **Target Environment:** local, Java 21, Maven; ShopRobot application (`com.example.shoprobot.Main`), H2 in memory
* **Build / Release Version:** `main` at commit `251119c` (2026-10-19); re-test at commit `8995e47` (2026-10-19)
* **QA Status:** 🟢 Approved — QA-03 fixed in the developer's working copy (verified on the developer's machine on 2026-10-19; push pending)

---

## 🎯 QA Strategy & Scope

### In-Scope
* AC1 (text-based UI in Java) and AC2 (prepared to use JDBC to access a relational database).
* Functional scenarios TC-01 to TC-07 of the [US readme](./readme.md#-functional-tests--validation), plus extra scenarios on the database settings.
* Unit tests and JaCoCo coverage of the Shopping skeleton classes.
* Completeness of the US documentation.

### Out-of-Scope
* Listing levels and stores (USLP04); only checked that option `1` opens it and returns to the menu.
* Oracle JDBC driver and PL/SQL API (later sprints).

---

## 🛠️ Test Environment & Setup

* **How to run:** `com.example.shoprobot.Main` → **1 - Shopping**.
* **Database settings:** H2 in memory by default; `SHOPROBOT_DB_URL`, `SHOPROBOT_DB_USER` and `SHOPROBOT_DB_PASSWORD` for the error scenarios.

---

## 📋 Test Matrix

| Test ID | Title / Scenario | Type | Pre-conditions | Test Steps | Expected Result | Status | Execution Date | Bugs Filed |
| :--- | :--- | :--- | :--- | :--- | :--- | :---: | :---: | :--- |
| **TC-01** | Shopping menu (AC1) | Functional | Default settings | Choose `1 - Shopping` in the main menu | Title and options `1`, `2`, `0` | 🟢 PASS | 2026-10-19 | None |
| **TC-02** | Check database connection (AC2) | Functional | Default settings | Choose `2` | `Connected to H2 2.2.224 (2023-09-17)` | 🟢 PASS | 2026-10-19 | None |
| **TC-03** | Store listing opens | Functional | Default settings | Choose `1`, then level `1` | Levels and stores of USLP04, then the Shopping menu again | 🟢 PASS | 2026-10-19 | None |
| **TC-04** | Invalid option | Negative | Default settings | Choose `9` | `Invalid option.` and the menu again | 🟢 PASS | 2026-10-19 | None |
| **TC-05** | Option `0` | Functional | Default settings | Choose `0` | Readme: `Goodbye.` and **the application ends**. Observed after re-test: `0 - Back` / `Leaving Shopping.` and **back to the main menu**; readme updated in the developer's working copy | 🟢 PASS (verified on the developer's machine) | 2026-10-19 | [QA-03](#-defect--bug-tracking) |
| **TC-06** | Unknown JDBC URL (AC2) | Negative | `SHOPROBOT_DB_URL=jdbc:unknown:db` | Start the app, choose `1`, `2` | `Database connection failed: …`, no crash | 🟢 PASS (re-test) | 2026-10-19 | [QA-01](#-defect--bug-tracking) |
| **TC-07** | Same database for both options | Functional | Default settings | Choose `1` (list a level), then `2` | Stores listed; `Connected to H2 …` | 🟢 PASS | 2026-10-19 | None |
| **TC-08** | Spaces around the option | Robustness | Default settings | Type ` 2 ` | Same as option `2` | 🟢 PASS | 2026-10-19 | None |
| **TC-09** | Unreachable database server | Negative | `SHOPROBOT_DB_URL=jdbc:h2:tcp://localhost:9/nope` | Start the app | Message and menus still usable | 🟢 PASS (re-test) | 2026-10-19 | [QA-01](#-defect--bug-tracking) |
| **TC-10** | Blank environment variable | Negative | `SHOPROBOT_DB_URL=` (empty) | Start the app | Default H2 database used | 🟢 PASS (re-test) | 2026-10-19 | [QA-01](#-defect--bug-tracking), [QA-05](#-defect--bug-tracking) |
| **TC-11** | Schema initialization is safe | Code review | — | Review `ShoppingSchema.initialize` and `Main` | Tables are only dropped/recreated in the H2 in-memory database | 🟢 PASS (re-test) | 2026-10-19 | [QA-02](#-defect--bug-tracking) |
| **TC-12** | Unit tests | Automated | — | `mvn clean verify` in `JavaApp` | All tests pass | 🟢 PASS | 2026-10-19 | None |
| **TC-13** | Coverage ≥ 80% | Automated | — | JaCoCo report | Line coverage ≥ 80% in every class of the skeleton | 🟢 PASS (re-test) | 2026-10-19 | [QA-04](#-defect--bug-tracking) |
| **TC-14** | US documentation complete | Review | — | Read the [US readme](./readme.md) and the folder | Diagrams referenced by the readme are in the repository; texts match the application | 🟢 PASS (verified on the developer's machine) | 2026-10-19 | [QA-03](#-defect--bug-tracking) |
| **TC-15** | Configured database other than H2 in memory | Negative | `SHOPROBOT_DB_URL=jdbc:h2:./target/qa-test-db` | Start the app, choose `1`, `2` | Message `Using the configured database … (sample Shopping tables not created)`; tables not dropped; `Connected to H2 …` | 🟢 PASS | 2026-10-19 | None |
| **TC-16** | Regression after the fixes | Regression | Default settings | Repeat TC-01 to TC-04 and TC-07; open Planning after a database error | Same results as in the first run; Planning still works | 🟢 PASS | 2026-10-19 | None |

### Execution Log

**TC-01 / TC-02 / TC-04 / TC-05 / TC-08 (input `1`, ` 2 `, `9`, `0`, `0`):**
```
=== ShopRobot - Shopping ===
Option: Connected to H2 2.2.224 (2023-09-17)
=== ShopRobot - Shopping ===
Option: Invalid option.
=== ShopRobot - Shopping ===
Option: Goodbye.
=== ShopRobot ===
Option: Exiting ShopRobot.
```

**TC-03 / TC-07 (input `1`, `1`, level `1`, then `2`):**
```
--- List Stores by Level ---
Available shopping levels:
  [0] Piso 0 (Entrada Principal e Restauracao) - Height: 4,50m
  [1] Piso 1 (Moda e Tecnologia) - Height: 4,00m

Stores at Level 1 (Piso 1):
  - Fnac [Code: FNC01 | Category: Tecnologia / Livros | Status: OPEN]
  - Zara [Code: ZAR01 | Category: Moda / Vestuario | Status: OPEN]
...
Option: Connected to H2 2.2.224 (2023-09-17)
```

**TC-06 / TC-09 / TC-10 — the application stops before showing any menu:**
```
SHOPROBOT_DB_URL=jdbc:unknown:db
  Failed to initialize the Shopping tables: No suitable driver found for jdbc:unknown:db
SHOPROBOT_DB_URL=jdbc:h2:tcp://localhost:9/nope
  Failed to initialize the Shopping tables: Connection is broken: "java.net.ConnectException: Connection refused ..."
SHOPROBOT_DB_URL=   (blank)
  Failed to initialize the Shopping tables: No suitable driver found for
```
`Main` calls `ShoppingSchema.initialize(connectionFactory)` without handling the `DaoException`, so the error ends the program. `ShoppingController.checkDatabaseConnection()` and `ShoppingMenuTest#checkConnectionShowsFailure` handle the failure correctly, but the application never reaches that point.

**TC-12 — unit tests (all passing):** `ShoppingMenuTest` (7), `DatabaseSettingsTest` (3), `JdbcDatabaseInfoDaoTest` (2), `ShoppingSchemaTest` (6), `MainMenuTest` (5), `MainTest` (1).

**TC-13 — JaCoCo line / branch coverage:** `ShoppingController` 100% / 100%, `DatabaseSettings` 100% / 100%, `JdbcDatabaseInfoDao` 100% / 100%, `ShoppingMenu` 96.9% / 90%, `ShoppingSchema` 93.5% / 100%, **`ConnectionFactory` 47.1% / 50%**.

### Re-test after the fixes (commit `8995e47`, 2026-10-19)

Fixes by the developer: `d49540b` (blank variables treated as missing; `DatabaseSettings.isInMemoryH2()`), `5e30b85` (sample tables only created in H2 in memory), `76e5da5` (`Main` keeps running when the Shopping database is not available), `cc7cc34` (`0 - Back` / `Leaving Shopping.`) and `710925c` (tests for `ConnectionFactory` and `ShoppingController`).

**TC-06 / TC-09 / TC-10 / TC-15 — the application starts and option `2` reports the problem; Planning still opens:**
```
SHOPROBOT_DB_URL=jdbc:unknown:db
  Using the configured database jdbc:unknown:db (sample Shopping tables not created).
  Option: Database connection failed: Failed to connect to jdbc:unknown:db
SHOPROBOT_DB_URL=jdbc:h2:tcp://localhost:9/nope
  Using the configured database jdbc:h2:tcp://localhost:9/nope (sample Shopping tables not created).
  Option: Database connection failed: Failed to connect to jdbc:h2:tcp://localhost:9/nope
SHOPROBOT_DB_URL=   (blank)
  Option: Connected to H2 2.2.224 (2023-09-17)
SHOPROBOT_DB_URL=jdbc:h2:./target/qa-test-db
  Using the configured database jdbc:h2:./target/qa-test-db (sample Shopping tables not created).
  Option: Connected to H2 2.2.224 (2023-09-17)
```

**TC-11 — code review:** `Main` now calls `ShoppingSchema.initializeIfInMemory`, which only drops and recreates the tables when the URL starts with `jdbc:h2:mem:`; a `DaoException` is caught and reported, and the menus keep running.

**TC-12 — unit tests (all passing):** `ShoppingMenuTest` (7), `ShoppingControllerTest` (2, new), `DatabaseSettingsTest` (5), `ConnectionFactoryTest` (7, new), `JdbcDatabaseInfoDaoTest` (2), `ShoppingSchemaTest` (8), `ShoppingDatabaseStartupTest` (3, new), `MainMenuTest` (5), `MainTest` (1).

**TC-13 — JaCoCo line / branch coverage:** `ConnectionFactory` 100% / 100%, `ShoppingSchema` 100% / 100%, `ShoppingController` 100% / 100%, `DatabaseSettings` 100% / 87.5%, `JdbcDatabaseInfoDao` 100% / 100%, `ShoppingMenu` 96.9% / 90%.

**TC-05 / TC-14 — documentation:** the readme was not changed since `251119c`: it still links `cd.puml` and `sd.puml`, which are not in the folder, and it still describes option `0` as `Goodbye.` / "the application ends" (also in *Outputs* and *Edge Cases*), while the application now shows `0 - Back` / `Leaving Shopping.` and goes back to the main menu.

### Review 3 — QA-03 (2026-10-19)

The documentation fixes of QA-03 (class and sequence diagrams `cd.puml` / `sd.puml` with their SVGs, and the readme updated to `0 - Back` / `Leaving Shopping.` in TC-05, *Outputs* and *Edge Cases*) were checked on the developer's machine and are not in the repository yet. They must be pushed so that the repository matches the approved version.

---

## 🐞 Defect & Bug Tracking

| Bug ID | Summary / Description | Severity | Priority | Status | Linked Commit / Fix | Re-test Status |
| :--- | :--- | :--- | :--- | :--- | :--- | :---: |
| **QA-01** | The application crashes at startup when the database cannot be used (unknown JDBC URL, unreachable server or blank URL): `Main` does not handle the `DaoException` of `ShoppingSchema.initialize`. Option `2` never shows `Database connection failed: …` (TC-06 of the readme), and Planning and Back-office cannot be used either. Suggestion: catch the error in `Main`, show a message and keep the menus running. | High | High | Fixed | `76e5da5` | 🟢 Passed |
| **QA-02** | `ShoppingSchema.initialize` drops and recreates the Shopping tables (`DROP TABLE … CASCADE` + sample data) on every start. With `SHOPROBOT_DB_URL` pointing to the Oracle database, it would delete the shared tables and data of USBD05. Suggestion: only run it for the H2 in-memory URL (or behind a setting) and document it. | High | High | Fixed | `d49540b`, `5e30b85`, `76e5da5` | 🟢 Passed |
| **QA-03** | Documentation: the readme links `cd.puml` and `sd.puml`, which are not in the folder (no SVG either); TC-05 says the application ends after `0`, but it goes back to the main menu. After `cc7cc34`, the readme must also show `0 - Back` / `Leaving Shopping.` instead of `Goodbye.` (*Outputs*, *Edge Cases*, TC-05). | Low | Medium | Fixed | Developer's working copy (push pending) | 🟢 Passed (verified on the developer's machine) |
| **QA-04** | `ConnectionFactory` has 47.1% line coverage and no test class; `ShoppingController` is only tested through the menu. Suggestion: add `ConnectionFactoryTest` (connection opened with the settings, `unwrap` / `isWrapperFor`). | Low | Low | Fixed | `710925c` | 🟢 Passed |
| **QA-05** | An environment variable set but blank (e.g. `SHOPROBOT_DB_URL=`) is used as is instead of the H2 default (see TC-10). Suggestion: treat blank values as missing, as `PlanningDataSettings` does. | Low | Low | Fixed | `d49540b` | 🟢 Passed |
| **QA-06** | *(Observation)* The Shopping menu shows `0 - Exit` / `Goodbye.`, but goes back to the main menu; the Planning menu uses `0 - Back` / `Leaving Planning.`. | Low | Low | Fixed | `cc7cc34` | 🟢 Passed |

---

## 🤖 Unit Tests

* **Related App:** JavaApp
* **Related Code:** `view/shopping/ShoppingMenu`, `controller/shopping/ShoppingController`, `model/persistence/DatabaseSettings`, `ConnectionFactory`, `DatabaseInfoDao`, `JdbcDatabaseInfoDao`, `ShoppingSchema`, `Main`
* **Automated Tests:** `ShoppingMenuTest`, `ShoppingControllerTest`, `DatabaseSettingsTest`, `ConnectionFactoryTest`, `JdbcDatabaseInfoDaoTest`, `ShoppingSchemaTest`, `ShoppingDatabaseStartupTest`, `MainMenuTest`, `MainTest`
* **Execution:** `mvn clean test` (coverage: `mvn verify`, report in `JavaApp/target/site/jacoco`)

---

## 📊 Summary & QA Sign-Off

### Execution Results
* **Total Executed:** 16 (after review 3)
* **Passed:** 16
* **Failed:** 0
* **Observations:** 0
* **Blocked:** 0

*First run: 7 passed, 6 failed (TC-05, TC-06, TC-09, TC-10, TC-11, TC-14), 1 observation (TC-13). Re-test: 14 passed, 2 failed (TC-05, TC-14 — QA-03).*

### Final QA Recommendation

> 🟢 **Approved:** both acceptance criteria are met; the application no longer crashes when the database cannot be used (QA-01), the sample tables are only created in H2 in memory (QA-02), QA-04 to QA-06 are fixed, and the documentation of QA-03 was verified on the developer's machine. The user story can be moved to **Done**; the QA-03 fixes must be pushed so that the repository matches the approved version.

* [x] **QA Sign-off Approved**
* **Signed By:** Tiago Borges (QA)
* **Date:** 2026-10-19
