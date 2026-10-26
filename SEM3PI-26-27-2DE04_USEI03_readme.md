# [USEI03] Pending Purchase Orders Queue Management

* **Priority:** Medium
* **Story Points:** 5
* **Module / Component:** `Planning App` / `Order Queue + Domain`
* **Status:** Draft

---

## 👤 User Story Description

**As a** logistics operator, **I want to** manage and filter the queue of pending purchase orders by delivery time window and customer ID, while keeping the orders prioritized by their delivery deadline.
**So that** I can process the most urgent orders first and quickly find the orders relevant to a given time window or customer.

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

* **Clarification:** The queue only contains orders with status *pending*.
* **Clarification:** Orders with the same delivery deadline are ordered deterministically by `orderId`.
* **Clarification:** Orders whose deadline has already passed remain in the queue, at the head (most urgent first).
* **Assumption (to confirm with the customer):** "Due within the next hour" means `deliveryDeadline <= now + timeWindow`.
* **Out of Scope:** Order creation, payment, delivery routing and robot attribution (see [USEI04] and [USEI05]).

---

## ✅ Acceptance Criteria

* [ ] **AC1 - Ordered Queue:** An ordered queue structure automatically keeps the pending orders sorted by their earliest delivery deadline.
* [ ] **AC2 - Filtered Lists:** It is possible to obtain filtered lists of orders matching specific criteria, such as:
    * orders due within a delivery time window (e.g., the next hour);
    * orders belonging to a specific customer ID.
* [ ] **AC3 - Safe Query & Removal:** Functions exist to query and remove processed orders from the queue safely, without breaking collection integrity (the queue remains correctly ordered and consistent after every operation).

---

## 🔒 Non-Functional Requirements & Security

* **Business Rules (NFR1):** Business rules validation is applied during data registration (only *pending* orders enter the queue).
* **Performance (NFR2):** Insertion and removal of the head must run in logarithmic time (heap-based structure).
* **Integration (NFR3):** Java main language; other components must be integrated through files, program arguments or environment variables.
* **Object Oriented Programming (NFR4):** Work developed must follow the OOP paradigm, making it easy to maintain and to add new functionalities.
* **Documentation (NFR5):** Javadocs, usage of CamelCase and SVG style figures.
* **Integrity:** Filtering must not modify the queue; removals must keep the ordering invariant.
* **Robustness:** Operations on an empty queue or on a non-existent order are handled safely (no unhandled exceptions or corrupted state).

---

## 🔗 Found Out Dependencies

### Upstream Dependencies (Required before starting)
* **[USEI01] Data Import and System Initialization:** uses the data imported to setup queues


### Downstream Dependencies (Impacted by this story)

* **[USEI04] Viability check and robot attribution:** consumes the orders at the head of the queue.
* **[USEI05] Delivery split between multiple robots:** consumes the orders (and respective items) taken from the queue.

---

## 📥 Input & Output Data

### Inputs

| Field Name         | Data Type         | Required | Source           | Example / Constraints                          |
|:-------------------|:------------------|:---------|:-----------------|:-----------------------------------------------|
| `orderId`          | String            | Yes      | Order            | Unique identifier                              |
| `customerId`       | String            | Yes      | Order            | Used to filter by customer                     |
| `deliveryDeadline` | LocalDateTime     | Yes      | Order            | Earliest delivery deadline; defines queue order |
| `status`           | Enum              | Yes      | Order            | Only `PENDING` orders are queued               |
| `timeWindow`       | Duration          | No       | Operator input   | `1 hour` (orders due within the next hour)     |

### Outputs

| Field Name       | Data Type              | Destination   | Example / Description                                  |
|:-----------------|:-----------------------|:--------------|:-------------------------------------------------------|
| `pendingQueue`   | PriorityQueue\<Order\> | Operator view | Orders sorted by `deliveryDeadline` (tie-break `orderId`) |
| `filteredOrders` | List\<Order\>          | Operator view | Orders matching the time window and/or customer ID     |
| `removedOrder`   | Order                  | Caller        | Processed order removed from the queue                 |

---

## Edge Cases

* **Empty Queue:** Querying or removing from an empty queue returns an empty result / clear message instead of failing.
* **Same Deadline:** Two orders with identical deadlines keep a deterministic order (tie-break by `orderId`).
* **No Matches:** A filter with no matching orders returns an empty list.
* **Unknown Order:** Removing an order that is not in the queue does not change the queue.
* **Overdue Orders:** Orders whose deadline has already passed stay at the head of the queue.
* **Non-pending Order:** Inserting an order that is not *pending* is rejected with an informative message.

---

## 🧪 Functional Tests / Validation

| Scenario ID | Test Case / Steps                                | Expected Result                                                  |
|:------------|:-------------------------------------------------|:-----------------------------------------------------------------|
|    |    |    |

---

# 🧪 QA & Test Documentation

See Report at [QA & Test Documentation: USEI03 Pending Purchase Orders Queue](./qa-readme.md)

---

## 🧩 Related Modules & Classes

* **Frontend:**

* **Backend:**

---

## 🔄 Sequence Diagram

![Sequence Diagram](sd.svg)

---

## 🏁 Definition of Done (DoD)

* [ ] Acceptance criteria verified by QA / Product Owner.
* [ ] Unit tests written and passing (minimum 80% coverage on new service logic).
* [ ] Edge cases (empty queue, ties, non-existent order) covered by tests.
* [ ] Javadoc and coding conventions applied.
* [ ] Diagrams/Graphs updated in SVG.
