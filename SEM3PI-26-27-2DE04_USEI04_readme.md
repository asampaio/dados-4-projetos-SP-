# [USEI04] Viability check and robot attribution

* **Priority:** Medium
* **Story Points:** 3
* **Module / Component:** `Order` / `Order Assesment + Robot assignment`
* **Status:** Draft

---

## 👤 User Story Description

**As a** logistics operator
**I want to** evaluate if a single purchase order can be fulfilled by an available robot and assign the most suitable robot based on weight, volume constraints, and highest battery level.
**So that** an order can be completed

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

* **Clarification (2026-10-06):** USEI04 checks if an order can be fulfilled by one robot. If not the system should display that information.

---

## ✅ Acceptance Criteria

* [ ] **AC1 - Order Feasibility:** A boolean feasibility result indicating whether any available robot satisfies all constraints (max volume, max weight, and minimum required battery).
* [ ] **AC2 - Automatic Assignment:** Automatic assignment of the order to the optimal robot (READY status, sufficient capacity, and highest current battery).
* [ ] **AC3 - Updated Status:** Updated status of the selected robot (e.g., state changed from READY to BUSY) and updated station availability.

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
* **[USEI01] Data Import and System Initialization:** provides the robots, stations and orders (with their items)
loaded into the Java collections.
* **[USLP05] Back-Office Application Skeleton:** this functionality will reside in the Back-Office Application.

### Downstream Dependencies (Impacted by this story)
* **[USEI05] Delivery Split Between Multiple Robots:** if there are not any robot that can fulfill the order, it will be sent to the USEI05 functionality where it will receive the order so it can be divided

---

## 📥 Input & Output Data

### Inputs
| Field Name | Data Type     | Required | Source | Example / Constraints                           |
|:-----------|:--------------|:---------|:-------|:------------------------------------------------|
| `orderId`  | String        | Yes      | Order  | Unique identifier                               |

### Outputs
| Field Name       | Data Type         | Destination   | Example / Description                                                                       |
|:-----------------|:------------------|:--------------|:--------------------------------------------------------------------------------------------|
| `robotsAssigned` | List<Robot>       | OrderDelivery | all the robots needed to fulfill the order                                                  |
| `resultsMessage` | String            | console       | `Not enough robots available for that order` or `it was dispatched * robots for this order` |

---

## Edge Cases

* **Invalid Order:** Rejects the order.
* **Not Enough Robots:** The order is too big or too heavy for the available robots.
* **No Robots Available:** There are no robots with the READY status,

---

## 🧪 Functional Tests / Validation

| Scenario ID | Test Case / Steps                                | Expected Result                                                  |
|:------------|:-------------------------------------------------|:-----------------------------------------------------------------|
|    |    |    |

---

# 🧪 QA & Test Documentation

See Report at [QA & Test Documentation: USEI04 Viability check and robot attribution](./qa-readme.md)

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