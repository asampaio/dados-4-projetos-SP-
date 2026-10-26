# 🧪 QA & Test Documentation: [USEI03] Order Queue Management and Time-Window Filtering

* **User Story Reference:** [USEI03: Order Queue Management and Time-Window Filtering](./readme.md)
* **QA Lead / Owner:** TBD (must be a team member other than the author, João Tavares, per section 3.4 of the requirements document)
* **Target Environment:** `Planning App (app-two, Java 21, JUnit 5)`
* **Build / Release Version:** Sprint 1
* **QA Status:** 🟡 Implemented, awaiting QA review (23 JUnit tests passing; sign-off pending)

---

## 🎯 QA Strategy & Scope

### In-Scope
* Ordered queue structure sorted by deadline (no `PriorityQueue`).
* Time-window filtering — inclusive bounds, deadline-ordered results.
* Customer ID filtering.
* Safe removal — no `ConcurrentModificationException`, no silent data loss.
* Edge cases: empty queue, no matches, invalid inputs, equal deadlines.
* Time complexity vs. measured execution time (medium and large datasets).

### Out-of-Scope
* Order assignment to robots ([USEI04](../USEI04/readme.md)).
* Order partitioning ([USEI05](../USEI05/readme.md)).
* Database integration (not mandatory in Sprint 1).
* User interface appearance.

---

## 🛠️ Test Environment & Setup

* **Tooling:** Java 21 + JUnit 5, module `app-two`, test class `OrderQueueServiceTest`.
* **Test data:** hand-made `Order` objects (one order line each) with varied deadlines and customer IDs; an order assigned to a robot with `Order.assignTo` (from USEI04) for the status check; medium and large datasets for the performance test.
* **Approach:** TDD. Each test is written first (Red), then the minimum code to pass (Green), then refactor.

---

## 📋 Test Matrix

| Test ID | Title / Scenario | Type | Steps | Expected Result | Status | Executed By | Notes |
|:--------|:-----------------|:-----|:------|:----------------|:------:|:-----------:|:------|
| **TC-01** | Single order in queue | Functional | Load one order; call `getPendingOrders` | That order is returned | ✅ Pass | João Tavares | — |
| **TC-02** | Multiple orders, different deadlines | Functional | Load orders A (later), B (earlier); call `getPendingOrders` | B before A | ✅ Pass | João Tavares | — |
| **TC-03** | Equal deadlines, different orderIds | Boundary | Load two orders with same deadline | Both present; ordered by orderId | ✅ Pass | João Tavares | Ensures no silent drop |
| **TC-04** | Filter by time window — some match | Functional | Orders inside and outside window; filter | Only inside orders returned, in deadline order | ✅ Pass | João Tavares | — |
| **TC-05** | Filter by time window — none match | Negative | All orders outside window | Empty list | ✅ Pass | João Tavares | — |
| **TC-06** | Filter by time window — boundary (inclusive) | Boundary | Orders at exactly `start` and `end` | Both included | ✅ Pass | João Tavares | — |
| **TC-07** | Filter by customer ID — some match | Functional | Mixed-customer queue; filter by one ID | Only that customer's orders | ✅ Pass | João Tavares | — |
| **TC-08** | Filter by customer ID — none match | Negative | No order belongs to given customer | Empty list | ✅ Pass | João Tavares | — |
| **TC-09** | Remove existing order | Functional | Add order; remove it | Returns `true`; order gone from queue | ✅ Pass | João Tavares | — |
| **TC-10** | Remove non-existing order | Negative | Remove an order never added | Returns `false`; queue unchanged | ✅ Pass | João Tavares | — |
| **TC-11** | Empty queue — `getPendingOrders` | Negative | No orders loaded | Empty list | ✅ Pass | João Tavares | — |
| **TC-12** | Empty queue — filter by time window | Negative | No orders loaded; filter | Empty list | ✅ Pass | João Tavares | — |
| **TC-13** | Empty queue — filter by customer | Negative | No orders loaded; filter | Empty list | ✅ Pass | João Tavares | — |
| **TC-14** | Empty queue — remove | Negative | No orders loaded; remove | Returns `false`; no exception | ✅ Pass | João Tavares | — |
| **TC-15** | Invalid time window (`start > end`) | Negative | Call filter with reversed bounds | Rejected with clear message | ✅ Pass | João Tavares | NFR01; also a window inverted by 1 ns, which only the explicit validation rejects |
| **TC-16** | Null time window bounds | Negative | Call filter with `null` start or end | Rejected with clear message | ✅ Pass | João Tavares | NFR01 |
| **TC-17** | Null customer ID | Negative | Call customer filter with `null` | Rejected with clear message | ✅ Pass | João Tavares | NFR01 |
| **TC-18** | Safe removal during iteration | Functional | Iterate and remove matching orders | No `ConcurrentModificationException`; only the non-matching orders remain | ✅ Pass | João Tavares | — |
| **TC-19** | Performance | Non-functional | Run over medium and large datasets | Growth consistent with O(O log O) load, O(log O + M) time-window filter (M = orders in the window), O(O) customer filter | ⚪ Not run | — | Results go to the ESINF report |
| **TC-20** | Only pending orders enter the queue | Functional | Load a `PENDING` order and an order already assigned to a robot (`ASSIGNED`); call `getPendingOrders` | Only the `PENDING` order is returned | ✅ Pass | João Tavares | Customer filter scope decision |
| **TC-21** | Invalid orders on load | Negative | Call `buildQueue` with `null`, with a `null` order, or with an order without id or without `requestedDeliveryTime` | Rejected with clear message; current queue unchanged | ✅ Pass | João Tavares | NFR01 |
| **TC-22** | Reload | Functional | Call `buildQueue` again with other orders, then with an empty collection | The queue is replaced each time (not appended); empty at the end | ✅ Pass | João Tavares | — |
| **TC-23** | Single-instant window (`start == end`) | Boundary | Filter with `start` equal to `end` | Only the orders due exactly at that instant | ✅ Pass | João Tavares | — |
| **TC-24** | Window up to `LocalDateTime.MAX` | Boundary | Filter with `end = LocalDateTime.MAX` | Every order from `start` on | ✅ Pass | João Tavares | Regression test for #86 |

### Execution Log

| Date       | Run by       | Command                              | Result                                                                   |
|:-----------|:-------------|:-------------------------------------|:-------------------------------------------------------------------------|
| 2026-10-19 | João Tavares | `mvn -B clean verify` (in `JavaApp`) | 23 JUnit tests (`OrderQueueServiceTest`): 23 passed, 0 failed, 0 skipped; BUILD SUCCESS |

---

## 🐞 Defect & Bug Tracking

| Issue ID | Summary / Description | Severity | Priority | Status | Linked Commit / Fix | Re-test Status |
|:---------|:----------------------|:---------|:---------|:-------|:--------------------|:--------------:|
| #86 | `filterByTimeWindow(start, LocalDateTime.MAX)` threw `DateTimeException`, because `end.plusNanos(1)` goes past the largest `LocalDateTime` | Low | Medium | Fixed | `d4eab04` (uses `tailSet()` when `end` is `LocalDateTime.MAX`) | ✅ Pass (TC-24) |

---

## 🤖 Unit Tests

Implemented JUnit tests (`OrderQueueServiceTest`, module `app-two`):

| Test | Covers | Status |
|:-------------|:-------|:------:|
| `getPendingOrders_singleOrder_returnsIt` | TC-01 | ✅ Passing |
| `getPendingOrders_multipleOrders_sortedByDeadline` | TC-02 | ✅ Passing |
| `getPendingOrders_equalDeadlines_sortedByOrderId` | TC-03 | ✅ Passing |
| `filterByTimeWindow_someMatch_returnsMatchingInOrder` | TC-04 | ✅ Passing |
| `filterByTimeWindow_noneMatch_returnsEmpty` | TC-05 | ✅ Passing |
| `filterByTimeWindow_boundaryInclusive_includesBothEnds` | TC-06 | ✅ Passing |
| `filterByCustomerId_someMatch_returnsCustomerOrders` | TC-07 | ✅ Passing |
| `filterByCustomerId_noneMatch_returnsEmpty` | TC-08 | ✅ Passing |
| `removeOrder_existingOrder_returnsTrueAndRemoves` | TC-09 | ✅ Passing |
| `removeOrder_nonExistingOrder_returnsFalse` | TC-10 | ✅ Passing |
| `getPendingOrders_emptyQueue_returnsEmpty` | TC-11 | ✅ Passing |
| `filterByTimeWindow_emptyQueue_returnsEmpty` | TC-12 | ✅ Passing |
| `filterByCustomerId_emptyQueue_returnsEmpty` | TC-13 | ✅ Passing |
| `removeOrder_emptyQueue_returnsFalse` | TC-14 | ✅ Passing |
| `filterByTimeWindow_startAfterEnd_throwsException` | TC-15 | ✅ Passing |
| `filterByTimeWindow_nullBounds_throwsException` | TC-16 | ✅ Passing |
| `filterByCustomerId_nullId_throwsException` | TC-17 | ✅ Passing |
| `removeOrder_duringIteration_noException` | TC-18 | ✅ Passing |
| `buildQueue_nonPendingOrders_areIgnored` | TC-20 | ✅ Passing |
| `buildQueue_invalidOrders_throwsException` | TC-21 | ✅ Passing |
| `buildQueue_calledAgain_replacesQueue` | TC-22 | ✅ Passing |
| `filterByTimeWindow_startEqualsEnd_returnsOrdersAtThatInstant` | TC-23 | ✅ Passing |
| `filterByTimeWindow_endAtMax_returnsAllFromStart` | TC-24 | ✅ Passing |

TC-19 (performance, reported in the ESINF report) has no JUnit test.

---

## 📊 Summary & QA Sign-Off

### Execution Results
* **Total Test Cases:** 24
* **JUnit tests:** 23 / 23 implemented
* **Passed:** 23
* **Failed:** 0
* **Not run:** 1 (TC-19 performance test)

### Final QA Recommendation

> 🟡 **Awaiting review:** the story is implemented and its 23 JUnit tests pass. QA sign-off must be given by a team member other than the author.

* [ ] **QA Sign-off Approved**
* **Signed By:** TBD
* **Date:** TBD