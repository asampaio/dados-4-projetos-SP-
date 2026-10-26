# 🧪 QA & Test Documentation: [USEI05] Complex Order Partitioning Across Multiple Robots

* **User Story Reference:** [USEI05: Complex Order Partitioning Across Multiple Robots](./readme.md) — GitHub issue [#11](https://github.com/Departamento-de-Engenharia-Informatica/SEM3PI-26-27-2DE02/issues/11)
* **QA Lead / Owner:** Francisco Marques (1212081)
* **Developer:** Gonçalo Abreu (GmpABR)
* **Target Environment:** local, Java 21, Maven; ShopRobot application (`com.example.shoprobot.Main`), `data/medium/clean` and `data/large/clean`
* **Build / Release Version:** `main` at commit `b48cce1` (2026-10-21)
* **QA Status:** 🟠 Changes requested — the three acceptance criteria work on the datasets, 3 issues to fix before Done (QA-01 to QA-03)

---

## 🎯 QA Strategy & Scope

### In-Scope
* AC1 (sub-orders grouped legally by compatibility and by weight/volume), AC2 (each sub-order assigned to a distinct available robot) and AC3 (structured summary), as in `doc/requirements/requirements.md`.
* Client clarifications: sub-orders on distinct robots and refrigerated items carried separately (2026-10-12); **strict segregation** of refrigerated and non-refrigerated robots and minimum battery of 20-25% (2026-10-16).
* Functional scenarios TC-01 to TC-05 of the [US readme](./readme.md#-functional-tests--validation) on the datasets, plus edge cases that are not in the datasets (run with a small test program on the model classes).
* Integration with USEI02 (station availability), USEI03 (pending queue) and USEI04 (same robots).
* Unit tests, JaCoCo coverage, ESINF report section and US documentation.

### Out-of-Scope
* Single-robot assignment (USEI04), already tested in its own QA.
* Route proposal, delivery scheduling and database persistence (later sprints).

---

## 🛠️ Test Environment & Setup

* **How to run:** `com.example.shoprobot.Main` → **2 - Planning** → **4 - Complex order partitioning**; Enter for the most urgent order or an order id.
* **Data:** `data/medium/clean` by default; `SHOPROBOT_DATA_DIR=../data/large/clean` for the large dataset.
* **Edge cases:** small fleets and orders built in a test program with `OrderPartitioningService`, `RobotFleet`, `StationMonitor` and `OrderQueue` (P1 to P6 below).

---

## 📋 Test Matrix

| Test ID | Title / Scenario | Type | Pre-conditions | Test Steps | Expected Result | Status | Execution Date | Bugs Filed |
| :--- | :--- | :--- | :--- | :--- | :--- | :---: | :---: | :--- |
| **TC-01** | Mixed order split by compatibility (AC1) | Functional | medium | Option `4`, Enter (most urgent: `O00047`) | 2 sub-orders: `O00047-S1` refrigerated (1 item, 1.05 kg) on a refrigerated robot, `O00047-S2` (2 items, 12.60 kg) on a standard robot | 🟢 PASS | 2026-10-21 | None |
| **TC-02** | Order too big for one robot (AC1, AC2) | Functional | medium | Option `4`, `O00197` (118.40 kg, 626.00 L) | 2 sub-orders (58.40 kg / 310 L and 60.00 kg / 316 L), each ≤ 80 kg / 320 L, on 2 distinct robots (`R0027`, `R0003`) | 🟢 PASS | 2026-10-21 | None |
| **TC-03** | Distinct robots and battery priority (AC2) | Functional | medium | TC-01 | `R0028` (93%, highest refrigerated) and `R0018` (100%, highest standard); different robots | 🟢 PASS | 2026-10-21 | None |
| **TC-04** | Structured summary (AC3) | Functional | medium | TC-01, TC-02 | Status, original order (customer, deadline, items, weight, volume, refrigeration), each sub-order with its items and its robot | 🟢 PASS | 2026-10-21 | None |
| **TC-05** | Order that cannot be served (atomic) | Negative | medium | Option `4`, `O00187` (refrigerated item of 95 kg) | `Status: FAILED`, no robot changed, the order stays pending | 🟢 PASS | 2026-10-21 | [QA-04](#-defect--bug-tracking) |
| **TC-06** | Order not pending | Negative | medium | `O99999`; `O00047` after TC-01 | `Order … is not pending in the queue.` | 🟢 PASS | 2026-10-21 | None |
| **TC-07** | Integration with USEI02, USEI03 and USEI04 | Integration | medium | TC-01, then `1 - Station monitoring`, then `3`, `O00001` | `S006` and `S001` with one more free spot; `O00047` no longer pending; USEI04 does not reuse `R0028` (assigns `R0008`) | 🟢 PASS | 2026-10-21 | None |
| **TC-08** | Large dataset | Functional | large | Option `4`, Enter | `O00176` split into 2 sub-orders on `R0020` (refrigerated) and `R0009` | 🟢 PASS | 2026-10-21 | None |
| **TC-09** | End of input | Robustness | medium | Piped input that ends after option `4` | Back to the menus, `Exiting ShopRobot.` | 🟢 PASS | 2026-10-21 | None |
| **TC-10** | Strict segregation (client, 2026-10-16) | Edge case | P1 | Non-refrigerated order, only a refrigerated robot `READY` | Not assigned to the refrigerated robot | 🔴 FAIL | 2026-10-21 | [QA-01](#-defect--bug-tracking) |
| **TC-11** | Capacity of the available robots | Edge case | P2 | 70 kg order (2 × 35 kg); 80 kg robot in `MAINTENANCE`; two 60 kg robots `READY` | 2 sub-orders on the two 60 kg robots | 🔴 FAIL | 2026-10-21 | [QA-02](#-defect--bug-tracking) |
| **TC-12** | Matching of sub-orders to robots | Edge case | P6 | Sub-orders 50 kg / 100 L and 10 kg / 300 L; robots X 80 kg / 320 L (90%) and Y 60 kg / 220 L (80%) | Y → 50 kg / 100 L, X → 10 kg / 300 L | 🔴 FAIL | 2026-10-21 | [QA-03](#-defect--bug-tracking) |
| **TC-13** | Atomic failure | Edge case | P4 | Mixed order, only one refrigerated robot | `FAILED`; the robot stays `READY`; the order stays pending | 🟢 PASS | 2026-10-21 | None |
| **TC-14** | Minimum battery | Edge case | P5 | Only robot with 19% | Not used (`FAILED`) | 🟢 PASS | 2026-10-21 | None |
| **TC-15** | No `PriorityQueue` / heap | Code review | — | Read `OrderPartitioningService` | `ArrayList`, `List.sort`, `RobotBatteryComparator`, `TreeSet` of `RobotFleet` | 🟢 PASS | 2026-10-21 | None |
| **TC-16** | Unit tests | Automated | — | `mvn clean verify` in `JavaApp` | All tests pass (275; 21 of USEI05) | 🟢 PASS | 2026-10-21 | None |
| **TC-17** | Coverage ≥ 80% | Automated | — | JaCoCo report | Line coverage ≥ 80% in every USEI05 class | 🟢 PASS | 2026-10-21 | None |
| **TC-18** | ESINF report | Review | — | Read the USEI05 section of [report.md](../../ESINF/report.md#usei05--complex-order-partitioning-across-multiple-robots) | Operations, JCF justification, complexity, empirical study (medium and large), discussion | 🟡 OBSERVATION | 2026-10-21 | [QA-05](#-defect--bug-tracking) |
| **TC-19** | US documentation complete | Review | — | Read the [US readme](./readme.md) and the diagrams | Readme and diagrams match the application | 🟡 OBSERVATION | 2026-10-21 | [QA-06](#-defect--bug-tracking) |

### Execution Log

**TC-01 / TC-03 / TC-04 (medium, option `4`, Enter):**
```
Status: SUCCESS
Original Order: O00047 (Customer: C0014, Deadline: 2026-10-05T11:45)
Total: 3 items | 13.65 kg | 31.00 L | Refrigeration required: YES
Generated Sub-Orders (2 partitions):
  * Sub-Order: O00047-S1
    - Items: 1 | Weight: 1.05 kg | Volume: 1.00 L | Refrigerated: YES
    - Assigned Robot: R0028 (Battery: 93%, Status: BUSY, Max: 60.0 kg / 220.0 L, Cooling: YES, Station: S006)
  * Sub-Order: O00047-S2
    - Items: 2 | Weight: 12.60 kg | Volume: 30.00 L | Refrigerated: NO
    - Assigned Robot: R0018 (Battery: 100%, Status: BUSY, Max: 40.0 kg / 140.0 L, Cooling: NO, Station: S001)
Order O00047 was successfully fulfilled and removed from pending queue.
```

**TC-02 (`O00197`):** `O00197-S1` 58.40 kg / 310.00 L → `R0027` (73%); `O00197-S2` 60.00 kg / 316.00 L → `R0003` (78%).

**TC-05 (`O00187`):** `Status: FAILED`, `Reason: Insufficient suitable available robots to fulfill the partitioned sub-orders.`, `No robot states were modified. Order remains pending.` The order has one refrigerated item of 95 kg / 180 L (*Artigo sobredimensionado*), and every refrigerated robot carries at most 60 kg.

**TC-07:** after TC-01, *Station monitoring* shows `S001 | Entrada Norte | capacity 10 | occupied 3 | free 7` and `S006 | Hipermercado | capacity 12 | occupied 5 | free 7`; USEI04 assigns `O00001` to `R0008`.

**TC-10 to TC-14 — test program on the model classes:**
```
P1 standard items, only refrigerated robot READY -> SUCCESS
   P1-S1 cool=false 5,0kg 10,0L -> RC (cooling robot=true)
P2 70 kg, 80 kg robot in MAINTENANCE, two 60 kg READY -> FAILED: Insufficient suitable available robots ...
P6 heavy 50kg/100L + bulky 10kg/300L; X 80kg/320L 90%, Y 60kg/220L 80% -> FAILED: Insufficient suitable available robots ...
P4 mixed order, only one refrigerated robot -> FAILED; RC status after: READY, order still pending: true
P5 only robot has 19% battery -> FAILED
```

**TC-17 — JaCoCo line / branch coverage:** `OrderPartitioningService` 93% / 75%, `OrderPartitioningUI` 95% / 83%, `OrderPartitioningController` 92% / 67%, `SubOrder` 100% / 95%, `PartitionResult` 100% / 50%, `PlanningMenu` 100% / 100%.

---

## 🐞 Defect & Bug Tracking

| Bug ID | Summary / Description | Severity | Priority | Status | Linked Commit / Fix | Re-test Status |
| :--- | :--- | :--- | :--- | :--- | :--- | :---: |
| **QA-01** | **Strict segregation not respected.** In the client session of 2026-10-16 (`doc/requirements/requirements.md`), the client said that refrigerated robots **exclusively** carry refrigerated items and that there are no mixed-cargo robots. `assignRobotsToSubOrders` gives a non-refrigerated sub-order to a refrigerated robot when no standard robot fits ("If no standard robot fits, consider cooling robots"), and the readme (*Domain / Business Constraints*) still allows it. Suggestion: non-refrigerated sub-orders only on robots without refrigeration; update the readme. | High | High | Open | | 🔴 Failed |
| **QA-02** | **Sub-orders sized by robots that are not available.** `partitionOrder` uses the maximum weight and volume of **every** robot of the fleet (`fleet.all()`), also `BUSY`, `CHARGING`, `MAINTENANCE` or with low battery. When the largest robots are not available, it builds sub-orders that no available robot can carry and the order fails, although it could be split for the available robots (TC-11). Suggestion: size the sub-orders with the `READY` robots with enough battery of each type (refrigerated / standard). | Medium | High | Open | | 🔴 Failed |
| **QA-03** | **Greedy matching can fail when an assignment exists.** The sub-orders are matched heaviest first, each to the available robot with the highest battery that fits, without looking at the volume of the other sub-orders. With capacities that are not "the heavier, the bigger" (TC-12), the heaviest sub-order takes the only robot that fits the bulky one. In the datasets the capacities grow together (25/80, 40/140, 60/220, 80/320), so it does not happen there. Suggestion: process first the sub-orders with fewer candidate robots, or try the other robots when one fails (K is small). | Low | Medium | Open | | 🔴 Failed |
| **QA-04** | *(Observation)* The failure message is always `Insufficient suitable available robots …`, also when an item can never be carried, e.g. `O00187` has a refrigerated item of 95 kg and every refrigerated robot carries at most 60 kg. Suggestion: say which item does not fit any robot of its type (the client said those products should be blocked when they are registered, 2026-10-02). | Low | Low | Open | | 🟡 Observation |
| **QA-05** | *(Observation)* ESINF report: `partitionOrder` reads `fleet.all()` four times (a copy of the R robots each time) to find the maximum capacities, so it is O(Io log Io + Io · K + R), not independent of R; this explains the 1.38× growth from medium to large. The time of `partitionAndAssignOrder` (79.74 µs / 220.95 µs) also includes building the queue with every order, the station monitor and the fleet in each run, which is most of it, and only 200 runs of the first order are measured. The empirical study does not say how many runs and warm-up runs were used. | Low | Low | Open | | 🟡 Observation |
| **QA-06** | *(Observation)* Readme: header says `Status: Done` while the US is in Testing; *Outputs* describes `PartitionResult` statuses `SUCCESS`, `PARTIAL`, `INSUFFICIENT_ROBOTS`, but the class only has success / failure; the functional tests have no dataset values or "how to run" (option `4` of the Planning menu); no link to this QA report. In the Windows console the item bullet `•` of the summary is shown as `�`. | Low | Low | Open | | 🟡 Observation |

---

## 🤖 Unit Tests

* **Related App:** JavaApp
* **Related Code:** `model/domain/SubOrder`, `model/domain/PartitionResult`, `model/domain/OrderPartitioningService`, `controller/planning/OrderPartitioningController`, `view/planning/OrderPartitioningUI`, `view/planning/PlanningMenu`, `Main`
* **Automated Tests:** `SubOrderTest`, `PartitionResultTest`, `OrderPartitioningServiceTest`, `OrderPartitioningControllerTest`, `OrderPartitioningUITest`, `PlanningMenuTest`, `PlanningMainTest`
* **Execution:** `mvn clean test` (coverage: `mvn verify`, report in `JavaApp/target/site/jacoco`)

---

## 📊 Summary & QA Sign-Off

### Execution Results
* **Total Executed:** 19
* **Passed:** 14
* **Failed:** 3 (TC-10 — QA-01, TC-11 — QA-02, TC-12 — QA-03)
* **Observations:** 2 (TC-18 — QA-05, TC-19 — QA-06); TC-05 passed with observation QA-04
* **Blocked:** 0

### Final QA Recommendation

> 🟠 **Changes requested:** on the medium and large datasets the three acceptance criteria work — orders are split by refrigeration and by weight/volume (AC1), every sub-order gets a distinct `READY` robot with the highest battery and the robots become `BUSY` with the station availability updated (AC2), the summary shows the order, the sub-orders and their robots (AC3), and a failure changes nothing. Before **Done**: respect the strict segregation of refrigerated robots asked by the client on 2026-10-16 (QA-01) and size the sub-orders with the robots that are available (QA-02). QA-03 should be fixed or documented as a limitation; QA-04 to QA-06 do not block.

* [ ] **QA Sign-off Approved** *(pending QA-01, QA-02, QA-03)*
* **Signed By:** Francisco Marques (QA)
* **Date:** *(Pending re-test)*
