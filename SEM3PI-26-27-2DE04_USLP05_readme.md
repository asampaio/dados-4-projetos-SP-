# [USLP05] Back-Office Application Skeleton

* **Priority:** High
* **Story Points:** 3
* **Module / Component:** `Administration app` / `UI` / `Persistence (JDBC) / Import (JSON)`
* **Status:** Draft

---

## 👤 User Story Description

**As a** Product Owner
**I want** the team to create the "skeleton" of the back-office application

---

## 🚦 Definition of Ready (DoR)

* [ ] **Clear User Story:** Story follows the standard *As a / I want to / So that* format.
* [ ] **Acceptance Criteria Defined:** Clear, testable ACs agreed upon by Product and Engineering.
* [ ] **Dependencies Identified:** Upstream APIs, third-party integrations, or blocking tasks are resolved or scheduled.
* [ ] **Inputs/Outputs Mapped:** Required data fields, payloads, and error states are documented.
* [ ] **Estimation Complete:** Story has been refined and estimated (story points assigned) by the dev team.
* [ ] **Small & Actionable:** Story is scoped down enough to be completed within a single sprint.

---

## 💬 Customer Specifications & Clarifications

 - "The application should be prepared to import JSON data", means that the application should support importing JSON data. Basically, is the app ready to start coding importing JSON data or it may require a library?
 - A skeleton application is an application that does not do anything functional for the system but includes all the necessary libraries and support for non-functional requirements. And, of course, it must compile and execute. When executed the application can simply display its identification and exit.
---

## ✅ Acceptance Criteria

* [ ] **AC1** The application should be a text-based UI, developed in Java
* [ ] **AC2** The application should be prepared to use JDBC to access a relational database
* [ ] **AC3** The application should be prepared to import JSON data
---

## 🔒 Non-Functional Requirements & Security

* **Business Rules (NFR01):** The validation of business rules must be followed during data registration and updating

* **Database (NFR02):** The database will be the main repository of information for the system and must reflect the necessary data integrity. The information should be maintained in a remote relational database management system (DBMS). All database operations must be done using an API of PL/SQL functions. As far as possible, all applications should use the same database. You should use the Data Access Object (DAO) pattern for working with JDBC.

* **Integration (NFR03):** To enhance interoperability between existing or developing systems, the main programming language should be Java. However, some components will need to be developed in other languages, such as PL/SQL for the database or C/Assembly for some specific components. Therefore, a significant part of the integration must be carried out through files, program arguments, or environment variables.

* **Object-Oriented Programming (NFR04):** As far as possible, the team should follow the object-oriented programming (OOP) paradigm, aiming at a design that allows easy maintenance and the addition of new functionalities, in line with OOP best practices.

* **Documentation (NFR05):** The team should follow the templates provided to document the project activities and issues. It should also adopt recognized coding standards (e.g., CamelCase) and code documentation tools, such as Javadoc. All the images/figures produced during the project should be recorded in SVG format

---

## 🔗 Found Out Dependencies

### Upstream Dependencies (Required before starting)

### Downstream Dependencies (Impacted by this story)

* **[USLP06] Import the stores and levels of a shopping location from a GeoJSON file, to create the stores available at a shopping location.

---

## 📥 Input & Output Data

### Inputs

| Field Name  | Data Type | Required | Source                                 | Example / Constraints                                            |
|:------------|:----------|:---------|:---------------------------------------|:-----------------------------------------------------------------|

### Outputs

| Field Name         | Data Type  | Destination | Example / Description                                                                           |
|:-------------------|:-----------|:------------|:------------------------------------------------------------------------------------------------|
| applicationIdentification | String    | Console     | "ShopRobot - Administration application (Back-office) v0.1;" name and version of the aplication |

---

## Edge Cases

* **Network Timeout:** Show retry button with message *"Network issue. Please try applying code again."*
* **Database unreachable or wrong credentials:** Clear error message without exposing the password; option to retry
  or exit.
* **JSON file not found / not readable / malformed JSON:** Clear error message (file path and, if available, line
  of the syntax error); application remains running.

---

## 🧪 Functional Tests / Validation

| Scenario ID | Test Case / Steps                                                  | Expected Result                                                       |
|:------------|:-------------------------------------------------------------------|:----------------------------------------------------------------------|

---

# 🧪 QA & Test Documentation

---

## 🧩 Related Modules & Classes

* **Frontend:** 
  * Text-based UI (`ui` package): `MainMenu`, `MenuOption`.

* **Backend:** 
  * `ConfigLoader`, `ConnectionFactory` (JDBC), repository interfaces (`StoreRepository`,
    `LevelRepository`), `JsonFileReader`, `DataImporter<T>`.

---

## 🔄 Sequence Diagram

![Sequence Diagram](sd.svg)

---

## 🏁 Definition of Done (DoD)

* [ ] Acceptance criteria verified by QA / Product Owner.
* [ ] Unit tests written and passing (minimum 80% coverage on new service logic).
* [ ] Javadoc and coding conventions applied.
* [ ] Diagrams/Graphs updated in SVG.
* [ ] Merge in "Main".