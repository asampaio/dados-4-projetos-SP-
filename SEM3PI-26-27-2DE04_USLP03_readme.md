# [USLP03] Front office Shopping application

* **Priority:** High
* **Story Points:** 3
* **Module / Component:** `Shopping app` / `UI` / `Persistence (JDBC) `
* **Status:** Draft

---

## 👤 User Story Description

**As a**  Product Owner  
**I want** the team to create the “skeleton” of the Shopping front-office application.

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

* **Question:** Could you elaborate what exactly the "skeleton" for the front and back-office applications is, and what
  must be included in it for each US to be considered complete?

* **Answer:** In both cases a skeleton application is an application that does not do anything functional for the
  system, but includes all the necessary libraries and support for non-functional requirements. And,of course, it must
  compile and execute. When executed the application can simply display its identification and exit.

---

## ✅ Acceptance Criteria

* [ ] ***AC1** - The application should be a text-based UI, developed in Java*
* [ ] ***AC2** - The application should be prepared to use JDBC to access a relational database*

---

## 🔒 Non-Functional Requirements & Security

* **Database (NFR02): Database operations via DAO/JDBC, ready to use PL/SQL API functions in the future.**

* **Integration (NFR03): Java main language; Other components must be integrated through files, program arguments or
  environment variables.**

* **Object-Oriented Programming(NFR04): Work developed must follow the OOP paradigm, making it easy to maintain and to
  add new functionalities.**

* **Documentation (NFR05): Javadocs, usage of CamelCase and SVG style figures.**

---

## 🔗 Found Out Dependencies

### Upstream Dependencies (Required before starting)

*

### Downstream Dependencies (Impacted by this story)

* **[USLP04] List available stores:** Uses the main menu and the DAO/JDBC layer developed in this US.

---

## 📥 Input & Output Data

### Inputs

| Field Name | Data Type | Required | Source | Example / Constraints |
|:-----------|:----------|:---------|:-------|:----------------------|
| None       |           |          |        |                       |
|

### Outputs

| Field Name                | Data Type | Destination | Example / Description                                                                      |
|:--------------------------|:----------|:------------|:-------------------------------------------------------------------------------------------|
| applicationIdentification | String    | Console     | "ShopRobot - Shopping application (Front-office) v0.1;" name and version of the aplication |

---

## Edge Cases

* **Unexpected exception during startup** Exits with an exit code and a short message

---

## 🧪 Functional Tests / Validation

| Scenario ID | Test Case / Steps | Expected Result |
|:------------|:------------------|:----------------|
|

---

# 🧪 QA & Test Documentation

See Report at [QA & Test Documentation: USLP01 Feature Title](./qa-readme.md)

---

## 🧩 Related Modules & Classes

* **Frontend:**
Identification: `ShoppingUI`

* **Backend:**
Database

---

## 🔄 Sequence Diagram

![Sequence Diagram](sd.svg)

## 🏁 Definition of Done (DoD)

* [ ] Acceptance criteria verified by QA / Product Owner.
* [ ] Unit tests written and passing (minimum 80% coverage on new service logic).
* [ ] Javadoc and coding conventions applied
* [ ] Diagrams/Graphs updated in SVG
* [ ] Merge in "Main" 