# [USLP06] Import stores and levels

* **Priority:** Medium
* **Story Points:** 5
* **Module / Component:** `Import stores and levels` / `Import from GeoJSON`
* **Status:** Draft

---

## 👤 User Story Description

**As a** Retailer Administrator
**I want to** use the back-office application to import the stores and levels of a shopping location from a GeoJSON file, to create the stores available at a shopping location.

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

* **Clarification (2026-10-02):** Float levels should be considered

---

## ✅ Acceptance Criteria

* [ ] **AC1 - Import:** The app must import the data from the provided GeoJSON file.
* [ ] **AC2 - Shown Data:** A list of the imported levels and stores must be presented to the user.
* [ ] **AC3 - Data management:** The app must insert the imported data into the database.
* [ ] **AC4 - Error Management:** Importing errors should be clearly displayed

---

## 🔒 Non-Functional Requirements & Security

* **Business Rules (NFR1):** Business rules validation is applied during data registration (only *pending* orders enter the queue).
* **Performance (NFR2):** Insertion and removal of the head must run in logarithmic time (heap-based structure).
* **Integration (NFR3):** Java main language; other components must be integrated through files, program arguments or environment variables.
* **Object Oriented Programming (NFR4):** Work developed must follow the OOP paradigm, making it easy to maintain and to add new functionalities.
* **Documentation (NFR5):** Javadocs, usage of CamelCase and SVG style figures.

---

## 🔗 Found Out Dependencies

### Upstream Dependencies (Required before starting)
* **[USLP05] Back-Office Application Skeleton:** Needs the back-office to be accessed.

### Downstream Dependencies (Impacted by this story)
* **n/a**

---

## 📥 Input & Output Data

### Inputs
| Field Name | Data Type | Required | Source     | Example / Constraints   |
|:-----------|:----------| :--- |:-----------|:------------------------|
| `filePath` | String    | Yes | User Input | `"File1.geojson"`       |
| `GeoJSON`  | File      | Yes | User Input | The file to be imported |

### Outputs
| Field Name       | Data Type | Destination | Example / Description                   |
|:-----------------|:----------|:------------|:----------------------------------------|
| `importedStores` | String    | console     | `"clothes store imported"`              |
| `importedLevels` | String    | console     | `"level 2 imported"`                    |
| `errorMessage`   | String    | console     | `"There was an  error with the import"` |

---

## Edge Cases

* **File Not Found:** Tells the user to try another file path
* **Invalid File:** The file is not a geojson file
* **Wrong format:** What is written on the file is not written in the specified format making it impossible to get the data

---

## 🧪 Functional Tests / Validation

| Scenario ID | Test Case / Steps | Expected Result |
| :--- | :--- |:----------------|
| |  |                 |


---

# 🧪 QA & Test Documentation

See Report at [QA & Test Documentation: USLP06 Import stores and levels](./qa-readme.md)

---

## 🧩 Related Modules & Classes

* **Frontend:**

* **Backend:**


---

## 🔄 Sequence Diagram

![Sequence Diagram](sd.svg)

## 🏁 Definition of Done (DoD)

* [ ] Acceptance criteria verified by QA / Product Owner.
* [ ] Unit tests written and passing (minimum 80% coverage on new service logic).
* [ ] Edge cases covered by tests.
* [ ] Javadoc and coding conventions applied
* [ ] Diagrams/Graphs updated in SVG.
* [ ] Merge in "Main"