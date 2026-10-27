# [USEI03] Order Queue and Temporal Filters

* **Issue:** [#9](https://github.com/Departamento-de-Engenharia-Informatica/SEM3PI-26-27-2DE02/issues/9)
* **Priority:** High
* **Story Points:** 5
* **Module / Component:** ESINF / Planning area of the ShopRobot application (menu *2 - Planning → 2 - Order queue*)
* **Status:** In Progress
* **Developer:** Tiago Borges (1231109)
* **QA:** João Ferreira (1231752)

---

## 👤 User Story Description

**As a** logistics operator  
**I want to** manage and filter the queue of pending purchase orders by delivery time windows and customer ID, while keeping orders prioritized by their deadline  
**So that** the most urgent orders are always handled first and I can quickly find the orders I need

---

## 🚦 Definition of Ready (DoR)

* [x] **Clear User Story:** Story follows the standard *As a / I want to / So that* format (Issue #9).
* [x] **Acceptance Criteria Defined:** ACs taken from GitHub Issue #9 and Project Specification v1 (Section 7.2.1).
* [x] **Dependencies Identified:** orders imported by USEI01 (`CsvDataImporter`).
* [x] **Inputs/Outputs Mapped:** Inputs and outputs documented below.
* [x] **Estimation Complete:** 5 story points (Sprint 1 planning).
* [x] **Small & Actionable:** In-memory feature of the Planning area, completed within Sprint 1.

---

## 💬 Customer Specifications & Clarifications

* **Project statement (v1, Section 7.2.1):**
  * Choose the most appropriate JCF interfaces and classes for each entity/relation.
  * `PriorityQueue` (or any other heap-based structure) is **not allowed** in Sprint 1; sorting and priorities use standard collections with a `Comparator` or JCF utilities.
  * Database integration is not mandatory in Sprint 1: data may be processed only in memory.
  * A report must identify the main operations and data structures, justify the JCF choices and derive the time complexity, supported by an empirical study with the medium and large datasets.
* **Clarification (2026-10-02, LAPR3 client session):** orders are final once confirmed and paid.
* **Clarification (2026-10-12, LAPR3 client session):** delivery scheduling is out of Sprint 1 scope.
* See the [requirements Q&A log](../../requirements/requirements.md).
* **Out of Scope:** assigning robots to orders (USEI04, USEI05), changing the status of orders, and database persistence.

---

## ✅ Acceptance Criteria

* [x] **AC1 - Ordered queue:** an ordered queue structure that automatically keeps pending orders sorted by their earliest delivery deadline.
* [x] **AC2 - Filters:** filtered lists of orders matching specific criteria (e.g. orders due within the next hour or orders belonging to a specific customer ID).
* [x] **AC3 - Safe query and removal:** functions to query and remove processed orders from the queue safely without breaking collection integrity.

---

## 🔒 Non-Functional Requirements & Architecture

* **NFR04 - Object-Oriented Design (MVC):**
  * **Model:** `OrderQueue` (queue of pending orders, filters and removal), in `model/domain`; uses `Order`, imported by USEI01.
  * **Controller:** `OrderQueueController` (`controller/planning`), called by the view and delegating to the model.
  * **View:** `OrderQueueUI` (`view/planning`), opened from `PlanningMenu`, the text menu of the Planning area.
* **NFR05 - Documentation:** Javadoc on all public classes and methods, including the time complexity of the `OrderQueue` operations.
* **ESINF Sprint 1 restriction:** no `PriorityQueue` or heap-based structures.

---

## 🔗 Found Out Dependencies

### Upstream Dependencies (Required before starting)
* **[USEI01] CSV import:** `CsvDataImporter#importDirectory(Path)` returns an `ImportResult` with `Map<String, Order>` — [USEI01 readme](../USEI01/readme.md).

### Downstream Dependencies (Impacted by this story)
* **[USEI04] Single-robot assignment:** takes the most urgent pending order (`peekNext` / `pollNext`) and removes it from the queue once assigned (`remove`).
* **[USEI05] Multi-robot assignment:** same as USEI04, for orders split into sub-orders.

---

## 📥 Input & Output Data

### Inputs
| Field Name | Data Type | Required | Source | Example / Constraints |
| :--- | :--- | :--- | :--- | :--- |
| `orders` | `Map<String, Order>` | Yes | USEI01 import (`orders.csv`) | `orderId`, `customerId`, `targetDeliveryTime` (deadline) |
| `now` | `LocalDateTime` | Yes (next hour) | System clock | Current date and time |
| `customerId` | `String` | Yes (customer filter) | User input | e.g. `C0146` |
| `orderId` | `String` | Yes (removal) | User input | e.g. `O00047` |

### Outputs
| Field Name | Data Type | Destination | Example / Description |
| :--- | :--- | :--- | :--- |
| Pending orders | `List<Order>` (earliest deadline first) | Planning text UI | `O00047 \| customer C0014 \| deadline 2026-10-05T11:45 (OVERDUE)` |
| Overdue orders / orders due in the next hour | two `List<Order>` | Planning text UI | Two sections; `(None)` when a section is empty |
| Removal result | `boolean` | Planning text UI | `Order O00047 removed from the queue.` / `Order O99999 is not in the queue.` |

---

## 🧠 Design

### Business rules

* The **deadline** of an order is its `targetDeliveryTime`; the queue serves the earliest deadline first.
* Orders with the **same deadline** are served in their order of arrival (first come, first served).
* An order id can only be **once** in the queue; a null order or a repeated id is rejected.
* **Orders due in the next hour** are the orders with a deadline until `now + 1 hour`. The overdue orders (deadline before `now`) are also included, because they are the most urgent; the UI shows them in a separate section, so that they do not hide the orders that are about to be due. An order due exactly at `now` is not overdue.
* A **processed** order is removed from the queue and disappears from every query.

### Data structures (JCF)

| Structure | Use | Justification |
| :--- | :--- | :--- |
| `NavigableMap<LocalDateTime, List<Order>>` (`TreeMap` of `ArrayList`) | pending orders grouped by deadline | keeps the deadlines sorted on every insertion (AC1) without a heap; `firstEntry` gives the most urgent order and `headMap` the time windows (AC2); the list per deadline keeps orders with the same deadline in arrival order |
| `Map<String, Order>` (`HashMap`) | pending orders by id | O(1) duplicate check, search and location for removal (AC3); updated together with the `TreeMap` |
| `List<Order>` (unmodifiable, `List.copyOf`) | results of the queries | the view cannot change the queue (AC3) |

### Main operations and time complexity

O = number of orders, D = number of distinct deadlines (D ≤ O), k = number of orders returned, b = number of orders with the same deadline.

| Operation | Complexity |
| :--- | :--- |
| Add an order | O(log D) |
| Most urgent order (`peekNext` / `pollNext`) | O(log D) / O(log D + b) |
| Orders due within a time window (`dueWithin`) | O(log D + k) |
| Orders of a customer (`byCustomer`) | O(O) |
| Find an order (`find`) | O(1) |
| Remove a processed order (`remove`) | O(log D + b) |

The justification and the empirical study (medium and large datasets) are in the [ESINF report](../../ESINF/report.md#usei03--order-queue-management-and-time-window-filtering).

### How to run

1. Run `com.example.shoprobot.Main` (from the repository root or from `JavaApp`).
2. Choose **2 - Planning** and then **2 - Order queue**.
3. Choose an option: **1** list pending orders, **2** orders due in the next hour, **3** orders of a customer, **4** mark an order as processed, **0** back.

The data is imported from `data/medium/clean` (see [data/README.md](../../../data/README.md)). Another folder can be used with the environment variable `SHOPROBOT_DATA_DIR`.

"Now" is the current time. Because the deadlines of the provided datasets are already in the past, it can be fixed with the environment variable `SHOPROBOT_NOW` (e.g. `SHOPROBOT_NOW=2026-10-06T10:00`) to demonstrate *Orders due in the next hour*.

### Edge cases

* **Empty queue:** the lists show `(No pending orders)`.
* **No orders in a section of the next hour:** the section shows `(None)`.
* **Customer without pending orders:** empty list.
* **Order not in the queue** (unknown or already processed): the UI says so and nothing changes.
* **Empty customer id or order id:** error message; the controller is not called.
* **Repeated order id:** rejected by `OrderQueue#add` with `IllegalArgumentException`.

---

## 🧪 Functional Tests / Validation

| Scenario ID | Test Case / Steps | Expected Result |
| :--- | :--- | :--- |
| **TC-01** | Open *Order queue* with `data/medium/clean` and choose **1** | 200 pending orders, earliest deadline first, starting with `O00047` (2026-10-05T11:45), `O00156` (12:00), `O00167` (13:00) |
| **TC-02** | Choose **2** after the last deadline of the dataset (deadlines from 2026-10-05 11:45 to 2026-10-18 18:45) | *Overdue orders*: all orders, marked `(OVERDUE)`; *Orders due in the next hour*: `(None)` |
| **TC-03** | Choose **3** and type `C0146` | `O00167` (2026-10-05T13:00), `O00091` (2026-10-05T16:45), `O00169` (2026-10-08T15:15), in this order |
| **TC-04** | Choose **4** and type `O00047`, then choose **1** | `Order O00047 removed from the queue.`; the list now starts with `O00156` and has 199 orders |
| **TC-05** | Choose **4** and type `O00047` again | `Order O00047 is not in the queue.` |
| **TC-06** | Choose **3** or **4** and press Enter without an id | Error message; nothing changes |
| **TC-07** | Orders with the same deadline, orders due exactly now, repeated ids (not in the dataset; covered by unit tests) | Same deadline: arrival order; due now: not overdue; repeated id: rejected |
| **TC-08** | Run with `SHOPROBOT_NOW=2026-10-06T10:00` and choose **2** | *Overdue orders*: the 12 orders of 2026-10-05 (`O00047` … `O00154`); *Orders due in the next hour*: `O00018`, `O00034` (10:15) and `O00116` (11:00) |
| **TC-09** | Choose **1** and compare the orders with the same deadline with `orders.csv` | Same order as in the file (e.g. 2026-10-06 10:15: `O00018`, `O00034`) |

**Unit tests:** `OrderQueueTest`, `OrderTest`, `OrderQueueControllerTest`, `OrderQueueUITest` and `PlanningMenuTest`.

---

# 🧪 QA & Test Documentation

To be completed by QA (João Ferreira) in `qa-readme.md`.

---

## 🧩 Related Modules & Classes

* **Model:** `model/domain/Order`, `model/importer/CsvDataImporter`; new: `model/domain/OrderQueue`.
* **Controller:** new `controller/planning/OrderQueueController`.
* **View:** new `view/planning/OrderQueueUI`; option *2 - Order queue* added to `view/planning/PlanningMenu`.
* **Main:** imports the planning data once and creates the order queue UI with the imported orders.
* **Benchmark:** `benchmark/OrderQueueBenchmark` (test sources), for the empirical study.

---

## 🔄 Diagrams

### Sequence Diagram (SD) — Orders due in the next hour

![USEI03 Sequence Diagram](./sd.svg)

*Source file:* [sd.puml](./sd.puml)

---

## 🏁 Definition of Done (DoD)

* [ ] Acceptance criteria verified by QA.
* [x] Unit tests written and passing (queue order, filters, removal, edge cases).
* [x] Javadoc on all public classes and methods.
* [x] Complexity analysis and empirical study (medium and large datasets) included in the ESINF report.
* [x] Sequence diagram in the repository (PlantUML + SVG).
