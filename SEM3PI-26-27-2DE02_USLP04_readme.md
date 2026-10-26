# [USLP04] Store Listing by Floor via JDBC

* **Issue:** [#4](https://github.com/Departamento-de-Engenharia-Informatica/SEM3PI-26-27-2DE02/issues/4)
* **Priority:** High
* **Story Points:** 5
* **Module / Component:** LAPR3 / Shopping area of `JavaApp` (MVC)
* **Status:** Todo
* **Developer:** Gonçalo Abreu (GmpABR)
* **QA:** Tiago Marquez

---

## 👤 User Story Description

**As a** Customer  
**I want to** use the Shopping application to list the available stores  
**So that** I can browse the stores on a selected floor/level of the shopping centre.

---

## 🚦 Definition of Ready (DoR)

* [x] **Clear User Story:** Story follows the standard *As a / I want to / So that* format (Issue #4).
* [x] **Acceptance Criteria Defined:** ACs taken directly from GitHub Issue #4 and Project Specification v1 (Section 7.1.1).
* [x] **Dependencies Identified:** Shopping App skeleton (USLP03) and Database Schema / Data (USBD04 / USBD05 / USLP06).
* [x] **Inputs/Outputs Mapped:** User input (selected level) and output (list of stores) mapped and documented below.
* [x] **Estimation Complete:** 5 story points (assigned in Sprint 1 backlog / GitHub Projects).
* [x] **Small & Actionable:** Console feature realizable within Sprint 1.

---

## 💬 Customer Specifications & Clarifications

* **GitHub Issue #4 & Project Specification v1 (Section 7.1.1):**
  * The app must list the levels of the shopping and allow the user to select a level.
  * The app must display the names of the stores of the selected level.
  * The app must access the information from the database.
* **Client Session (02/10/2026):**
  * Console-based text UI connected via JDBC to relational database.
  * Stores are linked to shopping levels/floors.
* **Client Session (12/10/2026):**
  * Shopping App is the front-office application for customers (Shopping area, option `1` of the main menu).
  * Sprint 1 uses H2 in-memory database via JDBC (migrating to Oracle in future sprints).

---

## ✅ Acceptance Criteria

* [x] **AC1 - List and Select Levels:** The app must list the levels of the shopping and allow the user to select a level.
* [x] **AC2 - Display Store Names:** The app must display the names of the stores of the selected level.
* [x] **AC3 - Database Access:** The app must access the information from the database via JDBC / DAO.

---

## 🔒 Non-Functional Requirements & Architecture

* **NFR02 - Database & DAO Pattern:** The application must access the database using JDBC following the Data Access Object (DAO) pattern.
* **NFR04 - Object-Oriented Design:** Separation of concerns between UI (`ListStoresUI`), Application Controller (`ListStoresController`), Domain models (`ShoppingLevel`, `Store`), and Data Access Objects (`LevelDao`, `StoreDao`).
* **NFR-UI-01:** Text-based console UI in Java.
* **NFR-LAPR3-01:** H2 in-memory database used for Sprint 1.

---

## 🔗 Found Out Dependencies

### Upstream Dependencies (Required before starting)
* **[USLP03] Shopping App Skeleton:** Base Java console project (`JavaApp`, Shopping area) and configuration.
* **[USBD04 / USBD05] Database Schema & Seeding:** `SHOPPING_LEVEL`, `AREA`, and `SHOP` tables in the database.
* **[USLP06] GeoJSON Store Data Import:** Ingests store and floor data into the database.

### Downstream Dependencies (Impacted by this story)
* Future shopping stories (e.g. browsing store products, placing orders).

---

## 📥 Input & Output Data

### Inputs
| Input Field | Data Type | Source | Description / Constraints |
| :--- | :--- | :--- | :--- |
| `selectedLevelNumber` | Integer | User Input (Console) | The floor number chosen by the customer |

### Outputs
| Output Field | Data Type | Destination | Description |
| :--- | :--- | :--- | :--- |
| `levelsList` | List of Levels | Console UI | Displays level number and floor name (e.g. Piso 0, Piso 1) |
| `storesList` | List of Stores | Console UI | Displays store names, categories, and codes on the selected level |
| `errorMessage` | String | Console UI | Message shown if level selection is invalid or no stores exist |

---

## 📐 Design & Architecture Artifacts

### Sequence Diagram (SD)
![USLP04 Sequence Diagram](./sd.svg)
*Source file:* [sd.puml](./sd.puml)

### Class Diagram (CD)
![USLP04 Class Diagram](./cd.svg)
*Source file:* [cd.puml](./cd.puml)

---

## 🧪 Functional Tests / Validation Scenarios

| Scenario ID | Description | Input | Expected Output |
| :--- | :--- | :--- | :--- |
| **TC-01** | Successfully list stores for a valid floor | Choose Level `1` | Displays all stores located on Level 1 |
| **TC-02** | Select a level with no registered stores | Choose Level `3` | Displays *"No stores found on this level."* |
| **TC-03** | User enters invalid level number | Enter `-99` or non-existent floor | Displays *"Invalid level selection. Please try again."* |
| **TC-04** | Verify data is retrieved from database | Inspect SQL query execution | Verifies `SELECT` statements against `SHOPPING_LEVEL` and `SHOP` |

---

# 🧪 QA & Test Documentation

See Report at [QA & Test Documentation: USLP04 Store Listing by Floor via JDBC](./qa-readme.md) *(to be completed by QA)*

---

## 🏁 Definition of Done (DoD) Checklist

* [ ] Design documented with Sequence Diagram (SD) and Class Diagram (CD).
* [ ] Domain models implemented in `model.domain` and DAOs in `model.persistence`.
* [ ] Controller and UI implemented in `controller.shopping` and `view.shopping`.
* [ ] Unit tests for Controller and DAOs implemented (TDD approach).
* [ ] Acceptance criteria and manual testing verified and signed off by QA.
