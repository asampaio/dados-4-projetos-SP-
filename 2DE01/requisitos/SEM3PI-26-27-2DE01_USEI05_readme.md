# [USEI05] Complex Order Partitioning

* **Priority:** High
* **Effort:** L
* **Status:** In progress

---

## 👤 User Story Description

**As a** Logistics Operator  
**I want to** automatically partition oversized or incompatible purchase orders into multiple sub‑orders.
**So that** each sub‑order can be assigned to a suitable robot that respects weight, volume, cooling, and battery constraints.
---

## 🚦 Definition of Ready (DoR)

* [x] **Clear User Story:** Story follows the standard *As a / I want to / So that* format.
* [x] **Acceptance Criteria Defined:** ACs are explicit and testable.
* [x] **Dependencies Identified:** Robots, stations, and order items must already be imported (USEI01).
* [x] **Inputs/Outputs Mapped:**  Items, robots, and sub‑orders clearly defined.
* [x] **Estimation Complete:** Story has been refined and estimated (story points assigned) by the dev team.
* [x] **Small & Actionable:** Story is scoped down enough to be completed within a single sprint.

---

## 💬 Customer Specifications & Clarifications

* **Clarification (2026-xx-xx):** 

---

## ✅ Acceptance Criteria

* [x] **AC1** - Algorithmically partitioned sub-orders where items are grouped legally by compatibility (e.g., separating refrigerated items from non-refrigerated items if required) and weight/volume constraints.
* [x] **AC2** - Successful multi-robot assignment linking each sub-order to a distinct available robot.
* [ ] **AC3** - A structured summary showing the original order, the generated sub-orders, and their respective assigned robots.

---

## 🔒 Non-Functional Requirements & Security

+ Code must follow Java best practices (packages, separation of concerns, naming conventions).
+ No external frameworks allowed (only standard Java).
+ Application must compile and run in a standard Java environment (JDK 17+).
+ Must comply with ESINF Sprint 1 restrictions (no streams, no heaps).
+ Must include Javadoc for all public classes and methods.

---

## 🔗 Found Out Dependencies

### Upstream Dependencies (Required before starting)
* USEI01 – Robots, stations, and orders must be imported and stored in memory. 
* USEI04 – Single‑robot feasibility logic already implemented (can be reused).

### Downstream Dependencies (Impacted by this story)
* No found dependencies.

---

## 📥 Input & Output Data

### Inputs
| Field Name      | Data Type   | Required | Source | Example / Constraints     |
|-----------------|-------------|----------|--------|---------------------------|
| Order           | Order       | Yes      | Memory | Must contain ≥1 item      |
| Robots          | List<Robot> | Yes      | Memory | Robots imported in USEI01 |
| Minimum Battery | double      | Yes      | Config | 50.0                      |

### Outputs
| Field Name          | Data Type                | Destination | Example / Description |
|---------------------|--------------------------|-------------|-----------------------|
| Sub‑Orders          | List<SubOrderAssignment> | Memory      | Robot and items       |
| Error Messages      | String                   | Console     | “No robots available” |
| Robot Status Update | STATUS                   | Memory      | READY → BUSY          |
| Station Update      | int                      | Memory      | occupiedSpots--       |

---

## Edge Cases

* No eligible robot: Throw IllegalStateException with clear message.
* Item too large for any robot: Must fail immediately.
* All robots below battery threshold: Must fail immediately.
* Refrigerated items but no cooling robots: Must fail immediately.
* Robot capacity exactly matches item: Must succeed.
* Multiple robots with same battery: Any of them may be chosen (stable sort).

---

## 🧪 Functional Tests / Validation

| Scenario ID | Test Case / Steps                                | Expected Result                      |
|-------------|--------------------------------------------------|--------------------------------------|
| TC‑01       | Order with refrigerated + non‑refrigerated items | Items separated into two groups      |
| TC‑02       | Items exceed robot capacity                      | Items split into multiple sub‑orders |
| TC‑03       | Multiple robots available                        | Robot with highest battery chosen    |
| TC‑04       | Robot assigned → station spot released           | ``occupiedSpots`` decreases by 1     |
| TC‑05       | No robot can carry refrigerated items            | Exception thrown                     |
| TC‑06       | No robot meets battery requirement               | Exception thrown                     |
| TC‑07       | Sub‑order items fit exactly                      | Assignment succeeds                  |
| TC‑08       | Items sorted by volume descending                | Largest items assigned first         |

---

# 🧪 QA & Test Documentation

See Report at [QA & Test Documentation: USLP03](./qa-readme.md)

---

## 🧩 Related Modules & Classes

* **Core Logic:**
  - OrderPartitionService
  - SubOrderAssignment
  - RobotBatteryComparator

* **Domain Entities:**
  - Order
  - OrderItem
  - Robot
  - Station

* **Utilities:**
  - STATUS
  - STATUSLIGHT

---

## 🔄 Sequence Diagram

![Sequence Diagram](sd.svg)

## 🏁 Definition of Done (DoD)

* [ ] Acceptance criteria verified by QA / Product Owner.
* [ ] Unit tests written and passing (minimum 80% coverage on new service logic).
* [ ] No streams or heap-based structures used.
* [ ] Robot and station states updated correctly.
* [ ] Javadoc added to all public classes and methods.
* [ ] Performance validated with medium and large datasets.
* [ ] Complexity analysis documented in ESINF report.
* [ ] API specs updated in OpenAPI / Swagger documentation.
