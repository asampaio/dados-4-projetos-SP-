# QA & Test Execution Report: USEI03

## USEI03 — Order Queue and Temporal Filters

* **User Story Reference:** [USEI03: Order Queue and Temporal Filters](./readme.md)
* **Issue:** [#9 — USEI03 Order Queue and Temporal Filters](https://github.com/Departamento-de-Engenharia-Informatica/SEM3PI-26-27-2DE02/issues/9)
* **Developer:** Tiago Borges
* **QA Lead / Owner:** João Ferreira
* **Implementation Area:** Planning area (menu *2 - Planning → 2 - Order queue*)
* **Execution Date:** 2026-10-19 (first run, commit `ca13f09`); re-test 2026-10-19 (commit `dfc62b3`)
* **Environment:** Java 21, Maven, JUnit 5, JaCoCo; dataset `data/medium/clean`
* **QA Status:** 🟢 Passed — QA-01 and QA-02 fixed and re-tested

---

## 🎯 QA Strategy & Scope

### In-Scope

* Ordering of the pending orders by earliest deadline (AC1).
* Orders due in the next hour, overdue orders and orders of a customer (AC2).
* Querying and removing processed orders without breaking the queue (AC3).
* Business rules documented in the readme (same deadline, repeated ids, orders due exactly now).
* Input validation in the console UI.
* Sprint 1 ESINF restriction: no `PriorityQueue` or heap-based structures.
* Automated unit tests and code coverage.

### Out-of-Scope

* CSV parsing and error handling of the import, covered by USEI01.
* Assignment of robots to orders (USEI04, USEI05).
* Database persistence (not required in Sprint 1).
* Review of the complexity analysis and empirical study in the ESINF report.

---

## 🛠️ Review Setup

* **Specification:** Project Specification v1, Section 7.2.1.
* **User story requirements:** [USEI03 readme](./readme.md).
* **Implementation under review:**
  * `OrderQueue` (`model/domain`)
  * `OrderQueueController` (`controller/planning`)
  * `OrderQueueUI` and `PlanningMenu` (`view/planning`)
  * `Main` (loading of the imported orders into the queue)
  * `ImportResult` and `PlanningClockSettings` (`model/importer`), changed by the fixes of QA-01 and QA-02
* **Automated tests:**
  * `OrderQueueTest`
  * `OrderTest`
  * `OrderQueueControllerTest`
  * `OrderQueueUITest`
  * `PlanningMenuTest`
  * `ImportResultTest` and `PlanningClockSettingsTest` (added with the fixes)

---

## ✅ Acceptance Criteria

* **AC1 — Ordered queue:** an ordered queue structure that automatically keeps pending orders sorted by their earliest delivery deadline.
* **AC2 — Filters:** filtered lists of orders matching specific criteria (e.g. orders due within the next hour or orders belonging to a specific customer ID).
* **AC3 — Safe query and removal:** functions to query and remove processed orders from the queue safely without breaking collection integrity.

---

## 📋 Test Matrix

| Test ID | Title / Scenario | Type | Test Steps | Expected Result | Status | Execution Date | Issues Found |
| :--- | :--- | :--- | :--- | :--- | :---: | :---: | :--- |
| **TC-01** | List pending orders | Functional | *Order queue* with `data/medium/clean`, option **1** | 200 orders, earliest deadline first, starting with `O00047`, `O00156`, `O00167` | 🟢 PASS | 2026-10-19 | None |
| **TC-02** | Orders due in the next hour after the last deadline | Functional | Option **2** (run on 2026-10-19, after the last deadline 2026-10-18 18:45) | *Overdue orders*: all 200, marked `(OVERDUE)`; *Orders due in the next hour*: `(None)` | 🟢 PASS | 2026-10-19 | QA-02 |
| **TC-03** | Orders of a customer | Functional | Option **3**, customer `C0146` | `O00167`, `O00091`, `O00169`, in this order | 🟢 PASS | 2026-10-19 | None |
| **TC-04** | Mark an order as processed | Functional | Option **4**, order `O00047`, then option **1** | `Order O00047 removed from the queue.`; the list starts with `O00156` and has 199 orders | 🟢 PASS | 2026-10-19 | None |
| **TC-05** | Remove an order that is no longer in the queue | Functional | Option **4**, order `O00047` again | `Order O00047 is not in the queue.`; nothing changes | 🟢 PASS | 2026-10-19 | None |
| **TC-06** | Empty customer id / order id | Functional | Options **3** and **4** with an empty line | `Customer id cannot be empty.` / `Order id cannot be empty.`; nothing changes | 🟢 PASS | 2026-10-19 | None |
| **TC-07** | Invalid menu option | Functional | Option **9** | `Invalid option.`; the menu is shown again | 🟢 PASS | 2026-10-19 | None |
| **TC-08** | Whole list sorted by deadline | Functional | Compare the deadlines of the 200 orders listed in TC-01 | Deadlines in non-decreasing order | 🟢 PASS | 2026-10-19 | None |
| **TC-09** | Same deadline served in arrival order (integrated app) | Functional | Compare the 27 deadlines shared by several orders in TC-01 with their order in `orders.csv` | Orders with the same deadline appear in the order of `orders.csv` | 🟢 PASS (re-test) | 2026-10-19 | QA-01 (fixed) |
| **TC-10** | Same deadline, due exactly now, repeated ids, null order | Unit | `OrderQueueTest` | Arrival order kept; due now is not overdue; repeated id and null rejected | 🟢 PASS | 2026-10-19 | None |
| **TC-11** | Returned lists cannot change the queue | Unit | `OrderQueueTest#returnedListsCannotChangeTheQueue` | The lists are unmodifiable | 🟢 PASS | 2026-10-19 | None |
| **TC-12** | Orders due in the next hour with a fixed clock | Unit | `OrderQueueUITest` (fixed `Clock`) | Overdue orders and orders due in the next hour shown in separate sections | 🟢 PASS | 2026-10-19 | None |
| **TC-13** | No `PriorityQueue` / heap-based structures | Static review | Inspect `OrderQueue` | Uses `TreeMap` of `ArrayList` and `HashMap` only | 🟢 PASS | 2026-10-19 | None |
| **TC-14** | Both internal structures updated together | Static review | Inspect `add`, `pollNext` and `remove` | `byId` and `byDeadline` always change together; empty deadline lists are removed | 🟢 PASS | 2026-10-19 | None |
| **TC-15** | Orders due in the next hour with a fixed "now" | Functional | Run with `SHOPROBOT_NOW=2026-10-06T10:00`, option **2** | *Overdue orders*: the 12 orders of 2026-10-05 (`O00047` … `O00154`); *Orders due in the next hour*: `O00018`, `O00034` (10:15) and `O00116` (11:00) | 🟢 PASS | 2026-10-19 | QA-02 (fixed) |
| **TC-16** | "Now" written with a space | Functional | Run with `SHOPROBOT_NOW="2026-10-06 10:00"`, option **2** | Same result as TC-15 | 🟢 PASS | 2026-10-19 | None |
| **TC-17** | Invalid "now" | Negative | Run with `SHOPROBOT_NOW=amanha`, option **2** | Message `SHOPROBOT_NOW must be a date and time such as 2026-10-06T10:00, but was: amanha. Using the current time.`; the application keeps working with the current time | 🟢 PASS | 2026-10-19 | None |
| **TC-18** | Import order kept for orders, robots and stations | Unit | `ImportResultTest` | The maps keep the import order and cannot be changed | 🟢 PASS | 2026-10-19 | None |

---

## 🧾 Execution Log

### Automated tests and coverage

Command executed (in `JavaApp`):

```text
mvn -B clean verify
```

Result:

```text
Tests run: 132, Failures: 0, Errors: 0, Skipped: 0
BUILD SUCCESS
```

USEI03 tests: `OrderQueueTest` (17), `OrderQueueUITest` (14), `OrderQueueControllerTest` (9), `PlanningMenuTest` (7), `OrderTest` (4) — all passing.

JaCoCo coverage of the USEI03 classes:

| Class | Instructions | Branches | Lines |
| :--- | :---: | :---: | :---: |
| `OrderQueue` | 96.4% | 94.4% | 96.7% |
| `OrderQueueController` | 100.0% | 100.0% | 100.0% |
| `OrderQueueUI` | 99.3% | 92.9% | 100.0% |
| `Order` | 86.5% | 91.7% | 91.3% |

Total project instruction coverage: 87.1% (goal > 80%).

### TC-01 to TC-09 — Functional tests

The application was run with `com.example.shoprobot.Main` (default dataset `data/medium/clean`), choosing **2 - Planning → 2 - Order queue**, with the input sequence `1`, `2`, `3`/`C0146`, `4`/`O00047`, `1`, `4`/`O00047`, `3`/empty, `4`/empty, `9`, `0`.

All the results of the readme scenarios TC-01 to TC-06 were obtained as documented.

For TC-09, the 200 orders listed by option **1** were compared with `orders.csv`. 27 deadlines are shared by more than one order, and in 18 of them the orders are not in the order of the file. Examples:

| Deadline | Order in `orders.csv` | Order shown by the application |
| :--- | :--- | :--- |
| 2026-10-06 10:15 | `O00018`, `O00034` | `O00034`, `O00018` |
| 2026-10-08 12:00 | `O00112`, `O00173`, `O00178` | `O00112`, `O00178`, `O00173` |
| 2026-10-13 12:15 | `O00024`, `O00033`, `O00085` | `O00085`, `O00033`, `O00024` |

`OrderQueue` itself keeps the arrival order (TC-10 passes). The cause is in the data that reaches it: `ImportResult` (USEI01) stores the imported maps with `Map.copyOf`, which does not keep the insertion order of the importer's `LinkedHashMap`. `Main` then adds the orders to the queue with `data.orders().values().forEach(queue::add)`, so the arrival order is lost before the queue receives the orders.

### Re-test after the fixes (commit `dfc62b3`)

Fixes delivered by the developer:

* `30aa7a4` — *keep the import order of the planning data*: `ImportResult` now stores the maps as `Collections.unmodifiableMap(new LinkedHashMap<>(...))`, with `ImportResultTest` (QA-01).
* `90dbdd1` — *allow fixing the current time of the order queue*: `PlanningClockSettings` reads the environment variable `SHOPROBOT_NOW` and `Main` gives that clock to `OrderQueueUI`, with `PlanningClockSettingsTest` and new readme scenarios (QA-02).

Command executed (in `JavaApp`):

```text
mvn -B clean verify
```

Result:

```text
Tests run: 141, Failures: 0, Errors: 0, Skipped: 0
BUILD SUCCESS
```

New tests: `ImportResultTest` (3) and `PlanningClockSettingsTest` (5) — all passing. Coverage: `ImportResult` 100% lines, `PlanningClockSettings` 100% lines; total project instruction coverage 87.2%.

Functional re-test with the same input sequence as before: TC-01 to TC-08 give the same results. **TC-09:** the 27 deadlines shared by several orders now appear in the order of `orders.csv`, with no mismatch (e.g. 2026-10-06 10:15: `O00018`, `O00034`).

**TC-15 (`SHOPROBOT_NOW=2026-10-06T10:00`):** the results were recomputed from `orders.csv` (12 orders before 10:00; 3 orders between 10:00 and 11:00):

```text
Orders due in the next hour:
  O00018 | customer C0093 | deadline 2026-10-06T10:15
  O00034 | customer C0111 | deadline 2026-10-06T10:15
  O00116 | customer C0134 | deadline 2026-10-06T11:00
```

---

## 🔍 Traceability to Acceptance Criteria

| Acceptance Criterion | Implementation Evidence | Test Evidence |
| :--- | :--- | :--- |
| **AC1 — Ordered queue** | `OrderQueue.add`, `peekNext`, `pollNext` (`TreeMap` by deadline); import order kept by `ImportResult` | TC-01, TC-08, TC-09, TC-10, TC-13, TC-18 |
| **AC2 — Filters** | `OrderQueue.dueWithin`, `byCustomer`, `filter`; `OrderQueueUI` sections; "now" from `PlanningClockSettings` | TC-02, TC-03, TC-12, TC-15, TC-16, TC-17 |
| **AC3 — Safe query and removal** | `OrderQueue.find`, `remove`, `removeProcessed`; unmodifiable result lists | TC-04, TC-05, TC-06, TC-11, TC-14 |

---

## 🐞 Issues Found

| Issue ID | Summary / Description | Severity | Status | Fix | Re-test |
| :--- | :--- | :--- | :--- | :--- | :---: |
| **QA-01** | In the integrated application, orders with the same deadline are not served in arrival order (business rule of the readme). `ImportResult` used `Map.copyOf`, which lost the insertion order of the imported orders before they reached the queue. It also contradicted the USEI01 readme ("`LinkedHashMap` ... preserves input order") and affected the order of robots and stations used by other user stories. | Medium | ✅ Closed | `30aa7a4` — `ImportResult` keeps the import order (`LinkedHashMap`), with `ImportResultTest` | 🟢 PASS (TC-09, TC-18) |
| **QA-02** | With the provided datasets every deadline is before the current date, so in a live run the section *Orders due in the next hour* was always `(None)`; that path was only verified by unit tests with a fixed clock. | Low | ✅ Closed | `90dbdd1` — "now" can be fixed with `SHOPROBOT_NOW`, documented in the readme (its TC-08 and TC-09) | 🟢 PASS (TC-15 to TC-17) |

---

## 📊 Summary & QA Sign-Off

### Execution Results

* **Total test cases:** 18
* **Executed:** 18
* **Passed:** 18 (TC-09 after re-test)
* **Failed:** 0
* **Blocked:** 0
* **Not executed:** 0
* **Open QA issues:** 0 (QA-01 and QA-02 closed)

### Final QA Recommendation

> 🟢 **Approved:** `OrderQueue`, the controller and the console UI satisfy the three acceptance criteria and are well covered by unit tests. After the fixes, orders with the same deadline are served in arrival order in the integrated application (QA-01), and *Orders due in the next hour* can be demonstrated with the provided datasets through `SHOPROBOT_NOW` (QA-02).

* [x] **QA Sign-off Approved**
* **Signed By:** João Ferreira
* **Date:** 2026-10-19

---

*AI usage: test execution and the first draft of this report were prepared with the help of Claude (Anthropic) and reviewed by the QA owner.*
