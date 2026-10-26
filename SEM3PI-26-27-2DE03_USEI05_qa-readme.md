# 🧪 QA & Test Documentation: [USEI05] Complex Order Partitioning Across Multiple Robots

* **User Story Reference:** [USEI05: Complex Order Partitioning Across Multiple Robots](readme.md)
* **QA Lead / Owner:** Hugo Coutinho (a team member other than the author of this story, Ivo Lobarinhas, per section 3.4 of the requirements document)
* **Target Environment:** `Planning App (app-two, Java 21, JUnit 5)`
* **Build / Release Version:** Sprint 1
* **QA Status:** 🟡 Implemented and reviewed, sign-off not approved yet (23 JUnit tests passing; defects D-01 and D-02 open)

---

## 🎯 QA Strategy & Scope

### In-Scope
* Partitioning of an order into sub-orders by legal compatibility (refrigerated / non-refrigerated) and weight/volume limits.
* Completeness and integrity of the partition (every unit in exactly one sub-order).
* Assignment of each sub-order to a distinct eligible robot (including the case where the matching needs a non-greedy choice).
* Update of robot states, station availability and sub-order links while the parent order stays `PENDING`.
* All-or-nothing behaviour: no change of state when the partition or the assignment fails.
* Structured summary (original order, sub-orders, assigned robots).
* Boundary values and invalid input.
* Time complexity vs. measured execution time (medium and large datasets).

### Out-of-Scope
* Single-robot feasibility and assignment ([USEI04](../USEI04/readme.md)), except for the shortcut when the order fits in one robot.
* Order queue management ([USEI03](../USEI03/readme.md)).
* Database integration (not mandatory in Sprint 1).
* Route planning and battery consumption estimation.
* User interface appearance.

---

## 🛠️ Test Environment & Setup

* **Tooling:** Java 21 + JUnit 5, module `app-two` (class `OrderPartitioningServiceTest`).
* **Test data:** small hand-made fleets (robots with different states, capacities, refrigeration and battery levels) and sample orders (fits in one robot, needs 2–3 sub-orders, mixed refrigerated/non-refrigerated, impossible); the medium and large datasets for the performance test.
* **Approach:** TDD. Each test is written first (Red), then the minimum code to pass (Green), then refactor.

---

## 📋 Test Matrix

| Test ID   | Title / Scenario                                 | Type           | Steps                                                                   | Expected Result                                                           |   Status   | Executed By   | Notes                                                |
|:----------|:-------------------------------------------------|:---------------|:------------------------------------------------------------------------|:--------------------------------------------------------------------------|:----------:|:-------------:|:-----------------------------------------------------|
| **TC-01** | Order fits in a single robot                     | Functional     | Order within the limits of one eligible robot; partition and assign     | Single-robot assignment; no sub-orders created                            | ✅ Passed  | Hugo Coutinho | Shortcut; a second test covers the refrigerated case |
| **TC-02** | Order heavier than every robot (splittable)      | Functional     | Order weight above any robot's max weight, items small enough           | Sub-orders created; each one within the weight limit                      | ✅ Passed  | Hugo Coutinho | The test checks the product weight, not the sub-order total (see D-04) |
| **TC-03** | Order bigger than every robot (splittable)       | Functional     | Order volume above any robot's max volume, items small enough           | Sub-orders created; each one within the volume limit                      | ✅ Passed  | Hugo Coutinho | Same remark as TC-02 (see D-04)                      |
| **TC-04** | Mixed refrigerated and non-refrigerated items    | Functional     | Order with both kinds; fleet with refrigerated and standard robots      | Refrigerated and non-refrigerated items are in different sub-orders       | ✅ Passed  | Hugo Coutinho | Client rule: refrigerated robots carry only refrigerated items |
| **TC-05** | Partition completeness                           | Functional     | Partition an order; sum quantities, weight and volume of all sub-orders | Every unit in exactly one sub-order; totals equal the original order      | ✅ Passed  | Hugo Coutinho | —                                                    |
| **TC-06** | Sub-order exactly at robot limits                | Boundary       | Items that fill a robot exactly in weight and volume                    | Sub-order accepted and robot is eligible                                  | ✅ Passed  | Hugo Coutinho | —                                                    |
| **TC-07** | Distinct robot per sub-order                     | Functional     | Order that needs several sub-orders; fleet with enough eligible robots  | Different robots assigned; none repeated                                  | ✅ Passed  | Hugo Coutinho | The test uses 2 sub-orders and 2 robots              |
| **TC-08** | Highest battery preferred                        | Functional     | Several eligible robots with different battery                          | The robot with the highest battery (not yet used) is chosen               | ✅ Passed  | Hugo Coutinho | —                                                    |
| **TC-09** | Fewer eligible robots than sub-orders            | Negative       | Order needs 2 sub-orders; only 1 eligible robot                         | Fails with a clear reason; nothing changes                                | ✅ Passed  | Hugo Coutinho | `IllegalStateException`                              |
| **TC-10** | Single item bigger than every robot              | Negative       | One unit heavier or bigger than the largest robot                       | No valid partition; fails with a reason; nothing changes                  | ✅ Passed  | Hugo Coutinho | Depends on the splitting granularity assumption      |
| **TC-11** | Refrigerated sub-order, no refrigerated robot    | Negative       | Refrigerated items; only non-refrigerated robots                        | Fails with a reason; nothing changes                                      | ✅ Passed  | Hugo Coutinho | A second test covers the opposite mismatch           |
| **TC-12** | Robots not `READY`                               | Negative       | Robot in use                                                            | It is ignored as a candidate                                              | ✅ Passed  | Hugo Coutinho | Only the `IN_USE` state is tested                    |
| **TC-13** | Robot below minimum battery                      | Functional     | Robot with battery under the threshold                                  | Robot is ignored as candidate                                             | ⚪ Not run |       —       | No JUnit test; the minimum battery rule is still open |
| **TC-14** | Robot state and station update                   | Functional     | Successful partition and assignment                                     | Each selected robot changes state; station availability updated           | ✅ Passed  | Hugo Coutinho | `IN_USE`                                             |
| **TC-15** | Order and sub-order status                       | Functional     | Successful partition and assignment                                     | Sub-orders linked to their robots (`ASSIGNED`); parent order remains `PENDING` | ✅ Passed | Hugo Coutinho | Partitioning is an internal dispatch operation; the test name still says "MarksOrderPartitioned" (see D-04) |
| **TC-16** | No side effects on failure (all-or-nothing)      | Negative       | Run a failing partition and compare the state before and after          | No robot, station or order changed; no sub-order left linked              | ✅ Passed  | Hugo Coutinho | The test only checks the state of one robot (see D-04) |
| **TC-17** | Empty fleet                                      | Negative       | No robots loaded                                                        | Fails with a reason                                                       | ✅ Passed  | Hugo Coutinho | `IllegalStateException`                              |
| **TC-18** | Invalid order                                    | Negative       | `null` or empty order                                                   | Rejected with a clear message                                             | ✅ Passed  | Hugo Coutinho | `IllegalArgumentException`                           |
| **TC-19** | Invalid data                                     | Negative       | Negative weight                                                         | Rejected with a clear message                                             | ✅ Passed  | Hugo Coutinho | NFR01                                                |
| **TC-20** | Structured summary                               | Functional     | Successful partition and assignment                                     | Summary shows the original order, the sub-orders and their robots         | ✅ Passed  | Hugo Coutinho | `PartitionResult`                                    |
| **TC-21** | Deterministic choice on ties                     | Boundary       | Candidate robots with equal battery                                     | Same result on every run (tie-break rule)                                 | ✅ Passed  | Hugo Coutinho | Lowest id wins                                       |
| **TC-22** | Performance                                      | Non-functional | Run over medium and large datasets                                      | Growth consistent with the derived complexity (Io, R, K)                  | ⚪ Not run |       —       | No JUnit test; results go to the ESINF report        |
| **TC-23** | Matching needing a non-greedy choice             | Functional     | Sub-order A fits robots r1 and r2 (r1 has more battery); sub-order B fits only r1 | A gets r2 and B gets r1 (complete matching found)           | ✅ Passed  | Hugo Coutinho | The test does not really exercise the backtracking (see D-03) |

### Execution Log

| Date       | Run by        | Command                                | Result                                          |
|:-----------|:--------------|:---------------------------------------|:------------------------------------------------|
| 2026-10-21 | Hugo Coutinho | `mvn -B clean verify` (in `JavaApp`)   | 23 JUnit tests: 23 passed, 0 failed, 0 skipped  |

---

## 🐞 Defect & Bug Tracking

| Issue ID | Summary / Description | Severity | Priority | Status | Linked Commit / Fix | Re-test Status |
|:---------|:----------------------|:---------|:---------|:-------|:--------------------|:--------------:|
| D-01     | No grouping: each unit becomes its own sub-order (AC1 and the planned first-fit-decreasing are not met). It uses more robots than necessary and rejects feasible orders, for example 3 units of 4 kg with 2 robots of 10 kg capacity, or the order `O00199` of the medium dataset (5 refrigerated units but only 4 eligible refrigerated robots) | High | High | Open | — | — |
| D-02     | The backtracking matching is exponential in K when no complete matching exists; it also copies and sorts the candidates at every level. `O00199` takes about 27 ms against tens of µs for the other orders (preliminary measurement, not on the reference machine) | Medium | Medium | Open | — | — |
| D-03     | TC-23 does not exercise a non-greedy choice: the refrigeration rule already separates the two robots, so a greedy choice would also pass | Low | Low | Open | — | — |
| D-04     | Test quality: TC-02/TC-03 check the weight/volume of the product, not the sub-order total; TC-16 checks the state of one robot only; the test `..._linksSubOrdersAndMarksOrderPartitioned` verifies that the order stays `PENDING` | Low | Low | Open | — | — |

---

## 🤖 Unit Tests

JUnit tests in `OrderPartitioningServiceTest` (23 tests):

| Test                                                                      | Covers |   Status   |
|:--------------------------------------------------------------------------|:-------|:----------:|
| `partitionAndAssign_orderFitsOneRobot_assignsSingleRobotWithoutSubOrders` | TC-01  | ✅ Passing |
| `partitionAndAssign_refrigeratedOrderFitsRefrigeratedRobot_directly`      | TC-01 (refrigerated) | ✅ Passing |
| `partitionAndAssign_overWeight_createsSubOrdersWithinWeightLimit`         | TC-02  | ✅ Passing |
| `partitionAndAssign_overVolume_createsSubOrdersWithinVolumeLimit`         | TC-03  | ✅ Passing |
| `partitionAndAssign_mixedRefrigeration_separatesCompatibilityGroups`      | TC-04  | ✅ Passing |
| `partitionAndAssign_partitionContainsEveryItemExactlyOnce`                | TC-05  | ✅ Passing |
| `partitionAndAssign_exactLimits_subOrderIsAccepted`                       | TC-06  | ✅ Passing |
| `partitionAndAssign_subOrdersGetDistinctRobots`                           | TC-07  | ✅ Passing |
| `partitionAndAssign_severalCandidates_choosesHighestBattery`              | TC-08  | ✅ Passing |
| `partitionAndAssign_fewerRobotsThanSubOrders_fails`                       | TC-09  | ✅ Passing |
| `partitionAndAssign_itemBiggerThanAnyRobot_fails`                         | TC-10  | ✅ Passing |
| `partitionAndAssign_refrigeratedWithoutRefrigeratedRobot_fails`           | TC-11  | ✅ Passing |
| `partitionAndAssign_refrigerationMismatch_failsWithoutPartitioning`       | TC-11 (both directions) | ✅ Passing |
| `partitionAndAssign_robotNotReady_isIgnored`                              | TC-12  | ✅ Passing |
| `partitionAndAssign_assigned_updatesRobotsAndStations`                    | TC-14  | ✅ Passing |
| `partitionAndAssign_assigned_linksSubOrdersAndMarksOrderPartitioned`      | TC-15  | ✅ Passing |
| `partitionAndAssign_failure_changesNothing`                               | TC-16  | ✅ Passing |
| `partitionAndAssign_emptyFleet_failsWithoutException`                     | TC-17  | ✅ Passing |
| `partitionAndAssign_invalidOrder_isRejected`                              | TC-18  | ✅ Passing |
| `partitionAndAssign_invalidData_isRejected`                               | TC-19  | ✅ Passing |
| `partitionAndAssign_success_returnsStructuredSummary`                     | TC-20  | ✅ Passing |
| `partitionAndAssign_batteryTie_isDeterministic`                           | TC-21  | ✅ Passing |
| `partitionAndAssign_matchingNeedsNonGreedyChoice_findsCompleteMatching`   | TC-23  | ✅ Passing |

Two test names differ from the ones planned in the first draft of this document: `partitionAndAssign_partitionContainsEveryItemExactlyOnce` (planned `partitionAndAssign_partition_containsEveryItemExactlyOnce`) and `partitionAndAssign_subOrdersGetDistinctRobots` (planned `partitionAndAssign_subOrders_getDistinctRobots`). The two `refrigerat...` tests marked above were added beyond the plan.

TC-13 (minimum battery rule open) and TC-22 (performance, reported in the ESINF report) have no JUnit test.

---

## 📊 Summary & QA Sign-Off

### Execution Results
* **Total Test Cases:** 23
* **JUnit tests:** 23 implemented (21 test cases covered, plus 2 extra tests)
* **Passed:** 21 test cases (23 / 23 JUnit tests)
* **Failed:** 0
* **Not run:** 2 (TC-13 no test, TC-22 performance test)
* **Open defects:** 4 (D-01 and D-02 affect the behaviour; D-03 and D-04 affect test quality)

### Final QA Recommendation

> 🟡 **Awaiting approval:** the story is implemented and its 23 JUnit tests pass, but AC1 is only partially met (D-01) and the performance study is still pending. Sign-off should be approved only after D-01 is fixed or accepted by the team as a known limitation.

* [ ] **QA Sign-off Approved**
* **Signed By:** Hugo Coutinho
* **Date:** 2026-10-21