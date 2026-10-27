# [USEI03] Order Queue Management and Time-Window Filtering

* **Priority:** TBD (to be defined in Sprint Planning)
* **Story Points:** 5 (size M, as estimated on the project board)
* **Module / Component:** `Planning App` (`app-two`) / `Order Queue`
* **Status:** In Review (implementation done, 23 JUnit tests passing; review and ESINF report pending)

---

## 👤 User Story Description

**As a** logistics operator
**I want to** manage and filter the queue of pending purchase orders by delivery time windows and customer ID, while keeping orders prioritised by their deadline
**So that** the most urgent orders are always processed first and operators can quickly locate orders by time constraint or customer

---

## 🚦 Definition of Ready (DoR)

* [x] **Clear User Story:** Story follows the standard *As a / I want to / So that* format.
* [x] **Acceptance Criteria Defined:** Taken directly from the project requirements document.
* [x] **Dependencies Identified:** See "Found Out Dependencies" below.
* [x] **Inputs/Outputs Mapped:** See "Input & Output Data" below.
* [x] **Estimation Complete:** 5 story points, size M (project board).
* [x] **Open Clarifications Resolved:** Customer filter scope is PENDING-only by design (see "Customer Specifications" below).

---

## 💬 Customer Specifications & Clarifications

* **Requirements doc (ESINF, Sprint 1):** the use of `PriorityQueue` (or any other heap-based structure) is strictly prohibited. Sorting and priority management must use standard collections combined with `Comparator`, or explicit sorting through JCF utilities.
* **Requirements doc (ESINF, Sprint 1):** data may be processed only in memory (database integration is not mandatory in this sprint), but the classes must be designed with future database integration in mind.
* **Requirements doc (ESINF, Sprint 1):** a report is required, identifying the principal operations and data structures, justifying the JCF choices and temporal complexity (parameters R, S, O, I, Io, K), supported by an empirical study with the medium and large datasets.
* **Decision (ordered queue):** implemented as a `TreeSet<Order>` with a `Comparator` by earliest delivery deadline; the orderId is the secondary sort key, which prevents silent duplicate drops when two orders share the same deadline.
* **Decision (time window):** a time window is defined by an inclusive `[start, end]` interval of `LocalDateTime`; orders whose deadline falls within the interval are returned.
* **Assumption (processed order):** an order is considered processed when it is explicitly removed from the queue (e.g., after assignment by USEI04 or manual operator action); no status change is required within this story.
* **Domain model (delivery deadline):** the deadline of an order is the `requestedDeliveryTime` attribute of `Order` (domain model and glossary), represented in Java as a `LocalDateTime` and populated during the CSV import (USEI01).
* **Decision (customer filter scope):** `filterByCustomerId` returns only `PENDING` orders — not because of an explicit filter, but because `buildQueue` adds only `PENDING` orders, and an order assigned afterwards (e.g., by USEI04) must leave the queue through `removeOrder` (see "processed order" above). This is a consequence of the design, not an extra filter.
* **Decision (invalid orders on load):** `buildQueue` validates every order before changing the queue (an order without id or without `requestedDeliveryTime` cannot be placed in the queue, because these are its two sort keys). If any order is invalid, the call is rejected and the current queue is kept unchanged, so the queue is never left half-built.
* **Out of Scope:** order assignment to robots ([USEI04](../USEI04/readme.md)), order partitioning ([USEI05](../USEI05/readme.md)), route planning.

---

## ✅ Acceptance Criteria

* [x] **AC1 - Ordered Queue:** an ordered queue structure that automatically keeps pending orders sorted by their earliest delivery deadline.
* [x] **AC2 - Filtered Lists:** filtered lists of orders matching specific criteria (e.g., orders due within a given time window or orders belonging to a specific customer ID) are returned in deadline order.
* [x] **AC3 - Safe Removal:** functions to query and remove processed orders from the queue safely without breaking collection integrity (no `ConcurrentModificationException`, no silent data loss).

---

## 🔒 Non-Functional Requirements & Security

* **NFR01 (Business Rules):** input data must be validated: every order loaded into the queue must have an id and a requested delivery time, and invalid filters (a time window with `start > end` or a `null` bound, a `null` customer ID) are rejected with a clear error message.
* **NFR04 (OOP):** design must follow OOP best practices and allow easy extension (e.g., adding new filter criteria in a future sprint).
* **NFR05 (Documentation):** Javadoc on public classes and methods, camelCase coding standard, diagrams in SVG.
* **Sprint 1 ESINF restriction:** no `PriorityQueue` or other heap-based structure.
* **Process:** TDD (Red-Green-Refactor), at least one commit per day.
* **Performance:** report must derive the time complexity and compare it with measurements on the medium and large datasets.
* **Security:** not applicable (no authentication or sensitive data involved).

---

## 🔗 Found Out Dependencies

### Upstream Dependencies (Required before starting)
* **[USEI01](../USEI01/readme.md) Data Import and System Initialization:** provides the `Order` collection loaded into JCF structures. Until it is ready, development can use hand-made test data.
* **Domain Model ([USBD03](../../BDDAD/USBD03/readme.md)/[USLP01](../../LAPR3/USLP01/readme.md)):** `Order`, `OrderLine`, `OrderStatus` (already in `app-two`, created in [USEI04](../USEI04/readme.md)); the customer is referenced by `customerId` on `Order`.

### Downstream Dependencies (Impacted by this story)
* **[USEI04](../USEI04/readme.md) Feasibility Check and Single-Robot Assignment:** retrieves pending orders from this queue to attempt assignment.
* **[USEI05](../USEI05/readme.md) Complex Order Partitioning:** also retrieves pending orders from this queue when a single robot cannot fulfil them.

---

## 📥 Input & Output Data

### Input Data

| Data | Type | Description |
|:-----|:-----|:------------|
| Orders | `Collection<Order>` | Full order collection loaded by USEI01 |
| Time window start | `LocalDateTime` | Inclusive lower bound of the delivery deadline filter |
| Time window end | `LocalDateTime` | Inclusive upper bound of the delivery deadline filter |
| Customer ID | `String` | Identifier of the customer whose orders are to be retrieved |
| Order to remove | `Order` | Order to be removed after processing |

### Output Data

| Data | Type | Description |
|:-----|:-----|:------------|
| Ordered queue | `List<Order>` | All pending orders sorted by deadline (then orderId) |
| Time-window filtered list | `List<Order>` | Orders whose deadline falls within `[start, end]` |
| Customer filtered list | `List<Order>` | Pending orders belonging to the given customer |
| Removal result | `boolean` | `true` if the order was found and removed; `false` otherwise |

---

## Edge Cases

* **Empty queue:** every operation called with valid arguments returns an empty list or `false`; no exception is thrown.
* **No orders match the filter:** an empty list is returned, not an error.
* **Reload:** calling `buildQueue` again replaces the whole queue (it does not append); an empty collection gives an empty queue.
* **Invalid orders on load (`null` collection, `null` order, or order without id or without `requestedDeliveryTime`):** rejected with a clear error message; the current queue is kept unchanged (decision above, NFR01).
* **Two orders with equal deadline:** tie-broken by `orderId` (secondary `Comparator` key), so neither is silently dropped by the `TreeSet`. Two orders with the same deadline and the same `orderId` would count as one entry; this does not happen because USEI01 keeps only the first occurrence of a repeated `orderId`.
* **Time window bounds:** both bounds are inclusive and `start == end` is a valid single-instant window; `end = LocalDateTime.MAX` returns every order from `start` on. A `null` bound or `start` after `end` is rejected with a clear error message (NFR01).
* **Customer filter:** a `null` customer ID is rejected with a clear error message (NFR01); a blank ID is not rejected and matches no order. An order without `customerId` stays in the queue but is never returned by this filter (USEI01 makes `customerId` mandatory).
* **Remove:** an order counts as "in the queue" when one with the same `orderId` and `requestedDeliveryTime` is there (the `Comparator` decides, not `equals`); otherwise `false` is returned and the queue is unchanged. A `null` order is not accepted (precondition documented in the Javadoc).
* **Removal during iteration:** `getPendingOrders` and both filters return a new list (a copy), so the caller can iterate it and call `removeOrder` without `ConcurrentModificationException` (TC-18).

---

## 🧪 Functional Tests / Validation

| Scenario ID | Scenario | Expected Result |
|:------------|:---------|:----------------|
| **F-01** | Queue with orders of different deadlines | Orders returned in strict deadline order |
| **F-02** | Two orders with equal deadlines | Both present; ordered by orderId |
| **F-03** | Filter by time window — some orders match | Only matching orders returned, in deadline order |
| **F-04** | Filter by time window — no orders match | Empty list |
| **F-05** | Filter by time window — boundary values | Orders at exactly `start` and `end` are included |
| **F-06** | Filter by customer ID — some match | Only that customer's pending orders returned |
| **F-07** | Filter by customer ID — no match | Empty list |
| **F-08** | Remove existing order | Returns `true`; order no longer in queue |
| **F-09** | Remove non-existing order | Returns `false`; queue unchanged |
| **F-10** | All operations on empty queue | Empty results / `false`; no exception |
| **F-11** | Invalid time window (`start > end`) | Rejected with clear error; queue unchanged |
| **F-12** | Load orders where one has no id or no requested delivery time | Rejected with clear error; current queue unchanged |

---

# 🧪 QA & Test Documentation

See Report at [QA & Test Documentation: USEI03 Order Queue Management](qa-readme.md)

---

## 🧩 Related Modules & Classes

* **Module:** `app-two` (Planning App), as agreed by the team.
* **Packages:** `com.example.apptwo.domain` and `com.example.apptwo.service`.
* **Classes:**
    * `Order` — shared with [USEI04](../USEI04/readme.md) and [USEI05](../USEI05/readme.md); has `id` (referred to as orderId in this document), `status` (`OrderStatus`) and its list of `OrderLine`. USEI03 adds `requestedDeliveryTime` (`LocalDateTime`) and `customerId` (identifier of its `Customer`), with a new constructor and getters; the USEI04 constructor is kept
    * `OrderStatus` — enum (`PENDING`, `ASSIGNED`, …); only `PENDING` orders enter the queue
    * `OrderQueueService` — keeps the queue in a `TreeSet<Order>` instance field (declared as `NavigableSet<Order>`); exposes `buildQueue`, `getPendingOrders`, `filterByTimeWindow`, `filterByCustomerId`, `removeOrder`; the validations are in the private methods `validateOrders` and `validateTimeWindow`, and the `subSet()` / `tailSet()` sentinels are created by `windowBoundary`
    * `DeadlineComparator` — implements `Comparator<Order>`, sorts by `requestedDeliveryTime` ASC, tiebreak by `orderId`; used as the `TreeSet` comparator (replaces `equals()` for set membership — two orders with the same deadline and orderId are considered the same entry)
* **Tests:** `OrderQueueServiceTest` (23 JUnit tests, see [qa-readme.md](qa-readme.md)).

---

## ⏱️ Expected Complexity

`buildQueue` validates the O orders (O(O)) and then performs at most O insertions into a new `TreeSet`, each costing O(log O) — total **O(O log O)**. `getPendingOrders` iterates all O elements: **O(O)**. `filterByTimeWindow` uses `NavigableSet.subSet()` — a O(log O) range lookup plus O(M) traversal of the M orders inside the window — **O(log O + M)**, where M ≤ O is an auxiliary output-size parameter (not one of the requirement parameters; R is reserved for the number of robots). **Caveat:** `subSet()` needs two sentinel `Order` objects whose `orderId` is the empty string `""` (guaranteed to sort below any real orderId): one at `start` (inclusive) and one at `end` plus one nanosecond (exclusive), so that the orders due exactly at `end` are included; this is a structural constraint of using `TreeSet` with a compound `Comparator`. When `end` is `LocalDateTime.MAX` there is no instant after it, so `tailSet()` from the `start` sentinel is used instead, with the same cost. `filterByCustomerId` is a full linear scan: **O(O)**. `removeOrder` is **O(log O)** via `TreeSet.remove()` (which also uses the `Comparator`, so `deadline + orderId` must match exactly). To be confirmed and measured with the empirical study in the ESINF report (parameters R, S, O, I, Io, K as defined in the requirements).

---

## 🔄 Sequence Diagram

![Sequence Diagram](sd.svg)

## 🏁 Definition of Done (DoD)

* [x] All acceptance criteria verified.
* [x] Unit tests written (TDD) and passing.
* [ ] Reviewed by a team member other than the author.
* [x] Javadoc written for public classes and methods.
* [ ] Complexity analysis and empirical study included in the ESINF report.
* [ ] Final sign-off recorded in `qa-readme.md`.