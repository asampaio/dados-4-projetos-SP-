# [USLP03] Front-office application

* **Priority:** High
* **Effort:** M
* **Status:** Done

---

## 👤 User Story Description

**As a** Product Owner  
**I want to** the team to create the “skeleton” of the Shopping front-office application.  
**So that** future Sprints can extend the application with business logic, database integration, and UI flows.
---


## 🚦 Definition of Ready (DoR)

* [x] **Clear User Story:** Story follows the standard *As a / I want to / So that* format.
* [x] **Acceptance Criteria Defined:** Clear, testable ACs agreed upon by Product and Engineering.
* [x] **Dependencies Identified:** No blocking upstream dependencies.
* [x] **Inputs/Outputs Mapped:** Basic UI interactions and DAO structure clarified.
* [x] **Estimation Complete:** Story has been refined and estimated (story points assigned) by the dev team.
* [x] **Small & Actionable:** Story is scoped down enough to be completed within a single sprint.

---

## 💬 Customer Specifications & Clarifications

* **Clarification (2026-10-01):** For this POC, authentication is not mandatory. However, there are aspects related to that I consider important:
  - The role of the user should be clear; this can be achieved by using apps specific for each role or having menus specific for each role;
  - It may be also important for some user stories to provide logs or error messages that clearly identify the user/role performing the operation.


---

## ✅ Acceptance Criteria

* [x] **AC1** - The application should be a text-based UI, developed in Java.
* [x] **AC2** - The application should be prepared to use JDBC to access a relational database.

---

## 🔒 Non-Functional Requirements & Security

+ Code must follow Java best practices (packages, separation of concerns, naming conventions).
+ No external frameworks allowed (only standard Java + JDBC).
+ Application must compile and run in a standard Java environment (JDK 17+).
+ No sensitive data stored in plain text.
+ Database credentials must be configurable (not hard‑coded in multiple places).

---

## 🔗 Found Out Dependencies

### Upstream Dependencies (Required before starting)
* No found dependencies.

### Downstream Dependencies (Impacted by this story)
* USLP04 (Store Management)
* USLP06 (Level & GeoJSON Integration)
---

## 📥 Input & Output Data

### Inputs
| Field Name                                | Data Type | Required | Source     | Example / Constraints |
|:------------------------------------------|:----------|:---------|:-----------|:----------------------|
| Menu Option                               | Integer   | Yes      | User Input | 1, 2, 3               |
| User Name (placeholder for testing DAO)   | String    | Yes      | User Input | "Alice"               |
| User Email (placeholder for testing DAO)	 | String    | Yes      | User Input | "alice@example.com"   |

### Outputs
| Field Name              | Data Type | Destination | Example / Description |
|:------------------------|:----------|:------------|:----------------------|
| Menu Text               | String	   | Console	    | "1. Register User"    |
| Validation Messages     | String    | Console	    | "Invalid input"       |
| Placeholder DAO Results | String    | Console	    | "User saved"          |

---

## Edge Cases

Since Sprint 1 only builds the skeleton, edge cases are limited to UI behavior:
+ **Empty Input:** User enters empty name/email → controller rejects input.
+ **Invalid Menu Option:** User enters a non‑numeric or out‑of‑range option → show error message.
+ **DAO Not Implemented:** DAO methods may return mock data or placeholders.
(No network, no API, no timeout — this is a local Java console app.)

---

## 🧪 Functional Tests / Validation

| Scenario ID | Test Case / Steps                   | Expected Result                       |
|:------------|:------------------------------------|:--------------------------------------|
| TC‑01       | Start application → show main menu  | Menu is displayed                     |
| TC‑02       | Select valid menu option            | Correct controller method is executed |
| TC‑03       | Select invalid menu option          | Error message shown                   |
| TC‑04       | Enter empty name/email              | `validateUserData()` returns false    |
| TC‑05       | Enter valid name/email              | `validateUserData()` returns true     |
| TC‑06       | Call `saveUser()` with valid data   | DAO `save()` is invoked               |
| TC‑07       | Call `saveUser()` with invalid data | DAO `save()` is NOT invoked           |

---

# 🧪 QA & Test Documentation

See Report at [QA & Test Documentation: USLP03](./qa-readme.md)

---

## 🧩 Related Modules & Classes

* **Frontend:**
  * MainAppUI
  * MenuItem
  * UserRegistrationUI
* **Backend:**
  * UserRegistrationController
  * UserDao (interface)
  * JdbcUserDao (JDBC implementation)
  * DatabaseConfig (connection setup)
  * User (entity)

---

## 🔄 Sequence Diagram

![Sequence Diagram](sd.svg)

## 🏁 Definition of Done (DoD)

* [ ] Acceptance criteria verified by QA / Product Owner.
* [ ] Unit tests written and passing (minimum 80% coverage on new service logic).
* [ ] Integration/E2E tests added for promo application flow.
* [ ] API specs updated in OpenAPI / Swagger documentation.