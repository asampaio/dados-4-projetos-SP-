# [USLP04] List Available Stores

* **Priority:** TBD (to be defined in Sprint Planning)
* **Story Points:** 3 (size S, as estimated on the project board)
* **Module / Component:** `Shopping App` (`app-one`) / `Store Listing`
* **Status:** In Review (implementation done, 11 JUnit tests passing; review and QA sign-off pending)

---

## 👤 User Story Description

**As a** customer
**I want to** use the Shopping application to list the available stores
**So that** I can see which stores of the shopping location I can buy from with ShopRobot

---

## 🚦 Definition of Ready (DoR)

* [x] **Clear User Story:** Story follows the standard *As a / I want to / So that* format.
* [x] **Acceptance Criteria Defined:** Taken directly from the project requirements document.
* [x] **Dependencies Identified:** See "Found Out Dependencies" below.
* [x] **Inputs/Outputs Mapped:** See "Input & Output Data" below.
* [x] **Estimation Complete:** Story estimated by the team (3 story points, size S).
* [x] **Open Clarifications Resolved:** The shopping location and the use of H2 were clarified in the client sessions (see below).

---

## 💬 Customer Specifications & Clarifications

* **Requirements doc (LAPR3, Sprint 1):** for the database, H2 may be used, as illustrated in the template application code in the repository.
* **Requirements doc (envisioned process):** the Shopping App only includes the stores that joined the solution. Not every shop of the shopping center is a store of the system; the Retailer Administrator defines which stores are available.
* **Requirements doc (NFR02):** the DAO pattern must be used for JDBC; database operations should be done using an API of PL/SQL functions.
* **Client sessions (shopping location):** the proof of concept in Sprint 1 covers a single shopping center (NorteShopping), so the app works on a single shopping location.
* **Client sessions (database):** an in-memory H2 database is accepted to fulfil the non-functional requirements of Sprint 1, so JDBC with the DAO pattern is used without PL/SQL functions. A persistent database may be used in later sprints.
* **Client sessions (data):** the listing reads data that was already validated and stored in the database (the import is [USLP06]). The minimum expected content is the levels and the names of the stores.
* **Decision (decimal levels):** the level number is decimal, not integer. The relational model ([USBD04]) stores it as `DECIMAL(5,2)` (for example, level 0.5 for a mezzanine), and the levels imported in [USLP06] can have this format. The app therefore keeps the level number as a `BigDecimal` with at most two decimal places and accepts a decimal point (`0.5`) when the customer chooses a level.
* **Assumption (listing):** levels are listed by ascending number and stores by name (alphabetical, ignoring case); the customer chooses a level by typing its number (for example `1`, `-1` or `0.5`).
* **Assumption (empty results):** a level without stores, or a shopping location without levels, shows an informative message instead of an empty list.
* **Assumption (customer):** no authentication is required to list the stores in Sprint 1.
* **Note (template database):** the template H2 database is in memory (`jdbc:h2:mem`), so the data is lost when the application stops. Until [USLP06] is available, `ShoppingSchemaInitializer` creates the tables (a subset of the model of [USBD04], with `level_number DECIMAL(5,2)`) and `ShoppingDemoData` loads a small set of test data when the application starts. `ShoppingDemoData` is temporary and is to be replaced by the import.
* **Out of Scope:** products of a store, shopping cart, placing orders, store details beyond the name, importing the data ([USLP06]).

---

## ✅ Acceptance Criteria

* [x] **AC1 - Levels:** The app lists the levels of the shopping location and allows the user to select a level.
* [x] **AC2 - Stores of the Level:** The app displays the names of the stores of the selected level.
* [x] **AC3 - Database:** The app accesses the information from the database.

---

## 🔒 Non-Functional Requirements & Security

* **NFR01 (Business Rules):** invalid user input (non-numeric, more than two decimal places or non-existing level) must be validated and rejected with a clear message.
* **NFR02 (Database):** access through the Data Access Object (DAO) pattern with JDBC.
* **NFR03 (Integration):** Java; text-based user interface.
* **NFR04 (OOP):** the design should follow OOP best practices, allowing easy maintenance and new functionalities.
* **NFR05 (Documentation):** Javadoc on public classes/methods, camelCase coding standard, diagrams in SVG.
* **Process:** TDD (Red-Green-Refactor locally), at least one commit per day.
* **Security:** not applicable (no authentication or sensitive data involved).

---

## 🔗 Found Out Dependencies

### Upstream Dependencies (Required before starting)
* **[USLP03] Shopping App skeleton:** provides the text-based UI and the JDBC setup that this story extends.
* **[USLP06] Import of levels and stores:** provides the levels and stores in the database. Until it is ready, the application uses the hand-made demo data.
* **[USBD05] Relational model instantiated:** defines the tables with the levels and stores.
* **Domain Model ([USBD03]/[USLP01]):** `ShoppingLocation`, `Level`, `Area`, `Store`.

### Downstream Dependencies (Impacted by this story)
* Future stories that let the customer browse the products of a store are expected to reuse this listing (not yet defined).
* `Level` is in `shared-library`: its number is a `BigDecimal`, so any other story that creates or reads a `Level` (for example [USLP05] and [USLP06]) must use this type.

---

## 📥 Input & Output Data

### Input Data
| Data         | Type                          | Description                                                          |
|:-------------|:------------------------------|:---------------------------------------------------------------------|
| Level choice | `Decimal` (console)           | Number of the level selected by the customer (for example `1`, `0.5`) |
| Levels       | Database (`DECIMAL(5,2)`)     | Levels of the shopping location                                      |
| Stores       | Database                      | Stores located in the selected level                                 |

### Output Data
| Data        | Type    | Description                                                         |
|:------------|:--------|:--------------------------------------------------------------------|
| Levels list | Console | Levels of the shopping location, by ascending number                |
| Stores list | Console | Names of the stores of the selected level, in alphabetical order    |
| Messages    | Console | No levels, no stores in the level, invalid choice or database error |

---

## Edge Cases

* **No levels in the database:** an informative message is shown (no exception).
* **Level without stores:** an informative message is shown.
* **Shop areas that are not stores of the system:** they are not listed.
* **Decimal level (for example 0.5):** it is listed in its place by number and can be selected; `0.50` and `0.5` are the same level.
* **Invalid choice (letters, empty, out of range, comma instead of point, more than two decimal places):** rejected with a clear message and the customer is asked again.
* **Database unavailable or query error:** a clear error message is shown, without exposing technical details.
* **Stores of other levels:** never shown for the selected level.

---

## 🧪 Functional Tests / Validation

| Scenario ID | Scenario | Expected Result |
| :--- | :--- | :--- |
| **F-01** | Open the store listing | All levels are shown, by ascending number (including the decimal level 0.5) |
| **F-02** | Select a valid level | The names of its stores are shown |
| **F-03** | Select a level without stores | Informative message |
| **F-04** | Type an invalid level (for example `abc`, `0,5`, `0.555`) | Error message and the customer is asked again |
| **F-05** | Database without levels | Informative message |
| **F-06** | Database unavailable | Clear error message, the application does not crash |
| **F-07** | Select the decimal level `0.5` | The names of its stores are shown (Bertrand in the demo data) |

---

# 🧪 QA & Test Documentation

See Report at [QA & Test Documentation: USLP04 List Available Stores](./qa-readme.md)

---

## 🧩 Related Modules & Classes

* **`shared-library`** (`com.example.dao`):
  * `Level`, `Store`: records with the data shown to the customer. The number of a `Level` is a `BigDecimal` with two decimal places; `displayNumber()` gives the text shown to the customer (`1`, `-1`, `0.5`).
  * `LevelDao` / `JdbcLevelDao`: lists the levels by ascending number.
  * `StoreDao` / `JdbcStoreDao`: lists the stores of a level in alphabetical order.
  * `ShoppingSchemaInitializer`: creates the `levels`, `areas` and `stores` tables (subset of the [USBD04] model).
  * `DaoException`: error raised when the database cannot be read (from the template).
* **`app-one`** (`com.example.appone`):
  * `StoreListing`: text-based user interface of this story.
  * `ShoppingDemoData`: temporary demo data (to be replaced by the import of [USLP06]).
  * `Main`: starts the database and runs the store listing.
* **Tests:** `JdbcLevelDaoTest`, `JdbcStoreDaoTest` and the helper `ShoppingTestDatabase` (`shared-library`); `StoreListingTest` (`app-one`).

---

## 🔄 Sequence Diagram

![Sequence Diagram](./sd.svg)

## 🏁 Definition of Done (DoD)

* [x] All acceptance criteria verified.
* [x] Unit tests written (TDD) and passing.
* [ ] Reviewed by a team member other than the author.
* [x] Javadoc written for public classes and methods.
* [ ] Final sign-off recorded in `qa-readme.md`.