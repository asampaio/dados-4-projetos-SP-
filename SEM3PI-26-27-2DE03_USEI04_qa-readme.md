# 🧪 QA & Test Documentation: [USEI04] Feasibility Check and Single-Robot Assignment

* **User Story Reference:** [USEI04: Feasibility Check and Single-Robot Assignment](readme.md)
* **QA Lead / Owner:** TBD (must be a team member other than the author, Hugo Coutinho, per section 3.4 of the requirements document)
* **Target Environment:** `Planning App (app-two, Java 21, JUnit 5)`
* **Build / Release Version:** Sprint 1
* **QA Status:** 🟡 Implemented, awaiting QA review (19 JUnit tests passing; performance study done; sign-off pending)

---

## 🎯 QA Strategy & Scope

### In-Scope
* Feasibility check (constraints: weight, volume, refrigeration, minimum battery).
* Selection of the optimal robot (highest battery among eligible).
* Update of the robot state and station availability.
* Boundary values and invalid input.
* Time complexity vs. measured execution time (medium and large datasets).

### Out-of-Scope
* Order partitioning across several robots ([USEI05]).
* Database integration (not mandatory in Sprint 1).
* User interface appearance.

---

## 🛠️ Test Environment & Setup

* **Tooling:** Java 21 + JUnit 5, module `app-two`, test class `RobotAssignmentServiceTest`.
* **Test data:** a small hand-made fleet (robots with different states, capacities, refrigeration and battery levels) and sample orders; the medium and large datasets (`clean` files) for the performance study.
* **Performance study:** `RobotAssignmentBenchmark` (in `app-two/src/test/java/com/example/apptwo/benchmark/`). It is not a JUnit test (the class name does not end with `Test`), so Maven does not run it; it is run by hand with the folder of the datasets as argument. Times are medians of 41 samples after a JIT warm-up.
* **Approach:** TDD. Each test is written first (Red), then the minimum code to pass (Green), then refactor.

---

## 📋 Test Matrix

| Test ID   | Title / Scenario                                | Type           | Steps                                                                                                       | Expected Result                                   |   Status   |  Executed By  | Notes                                                                                              |
|:----------|:------------------------------------------------|:---------------|:------------------------------------------------------------------------------------------------------------|:--------------------------------------------------|:----------:|:-------------:|:---------------------------------------------------------------------------------------------------|
| **TC-01** | Single eligible robot                           | Functional     | Fleet with one eligible robot; assign                                                                       | That robot is returned                            |  ✅ Pass   | Hugo Coutinho | —                                                                                                  |
| **TC-02** | Several eligible robots                         | Functional     | Fleet with eligible robots of different battery                                                             | Robot with the highest battery is returned        |  ✅ Pass   | Hugo Coutinho | —                                                                                                  |
| **TC-03** | Robots not `READY`                              | Negative       | Fleet in `CHARGING`, `MAINTENANCE`, in use                                                                  | They are ignored; not feasible                    |  ✅ Pass   | Hugo Coutinho | —                                                                                                  |
| **TC-04** | Order heavier than every robot                  | Negative       | Order weight above all max weights                                                                          | Not feasible; nothing changes                     |  ✅ Pass   | Hugo Coutinho | —                                                                                                  |
| **TC-05** | Order bigger than every robot                   | Negative       | Order volume above all max volumes                                                                          | Not feasible; nothing changes                     |  ✅ Pass   | Hugo Coutinho | —                                                                                                  |
| **TC-06** | Refrigerated item, no refrigerated robot        | Negative       | Order with refrigerated item; only non-refrigerated robots                                                  | Not feasible                                      |  ✅ Pass   | Hugo Coutinho | —                                                                                                  |
| **TC-07** | Refrigerated item, refrigerated robot available | Functional     | Same order; one refrigerated robot                                                                          | The refrigerated robot is returned                |  ✅ Pass   | Hugo Coutinho | —                                                                                                  |
| **TC-08** | Order exactly at robot limits                   | Boundary       | Weight and volume equal to the robot maximums                                                               | Robot is eligible                                 |  ✅ Pass   | Hugo Coutinho | —                                                                                                  |
| **TC-09** | Battery tie                                     | Boundary       | Two eligible robots with equal battery                                                                      | Deterministic choice (tie-break rule)             |  ✅ Pass   | Hugo Coutinho | Tie-break: lowest robot id, whatever the fleet order                                               |
| **TC-10** | Empty fleet                                     | Negative       | No robots loaded                                                                                            | Not feasible; no exception                        |  ✅ Pass   | Hugo Coutinho | —                                                                                                  |
| **TC-11** | State and station update                        | Functional     | Assign an order                                                                                             | Robot state changes; station availability updated |  ✅ Pass   | Hugo Coutinho | State name: `IN_USE`                                                                               |
| **TC-12** | No side effects when not feasible               | Negative       | Run a not feasible case                                                                                     | No robot or station changed                       |  ✅ Pass   | Hugo Coutinho | Also checks that the order stays `PENDING`                                                         |
| **TC-13** | Robot below minimum battery                     | Functional     | Robot with battery under the threshold                                                                      | Robot is ignored                                  | 🚫 Blocked |       —       | Blocked until the rule is defined; the code uses the placeholder `MIN_BATTERY = 20`, no JUnit test |
| **TC-14** | Invalid order                                   | Negative       | `null` or empty order                                                                                       | Rejected with a clear message                     |  ✅ Pass   | Hugo Coutinho | —                                                                                                  |
| **TC-15** | Performance                                     | Non-functional | Run over medium and large datasets                                                                          | Growth consistent with the derived complexity     |  ✅ Pass   | Hugo Coutinho | `RobotAssignmentBenchmark`, 1 run (2026-10-21): linear in R and in Io, as derived. Results in the readme and in the ESINF report |
| **TC-16** | `isFeasible` with eligible robot                | Functional     | Fleet with one eligible robot; call `isFeasible`                                                            | `true`, no state changes                          |  ✅ Pass   | Hugo Coutinho | —                                                                                                  |
| **TC-17** | `isFeasible` without eligible robot             | Negative       | Fleet with no eligible robot; call `isFeasible`                                                             | `false`, no state changes                         |  ✅ Pass   | Hugo Coutinho | —                                                                                                  |
| **TC-18** | Invalid data                                    | Negative       | Negative weight/volume or battery outside 0–100                                                             | Rejected with a clear message                     |  ✅ Pass   | Hugo Coutinho | NFR01; also covers a negative order line quantity                                                  |
| **TC-19** | Order linked to robot                           | Functional     | Assign an order                                                                                             | Order linked to the robot, status `ASSIGNED`      |  ✅ Pass   | Hugo Coutinho | —                                                                                                  |
| **TC-20** | Ordinary item, refrigerated robot ignored       | Functional     | Order with only ordinary items; fleet with a refrigerated robot (higher battery) and a non-refrigerated one | The non-refrigerated robot is returned            |  ✅ Pass   | Hugo Coutinho | Client rule: refrigerated robots carry only refrigerated products                                  |
| **TC-21** | Mixed order                                     | Negative       | Order with refrigerated and ordinary items; fleet with both types of robot                                  | Not feasible; nothing changes                     |  ✅ Pass   | Hugo Coutinho | Client rule: no mixed robots; the order goes to [USEI05]                                           |

### Execution Log

| Date       | Run by        | Command                                                                  | Result                                                                                                          |
|:-----------|:--------------|:-------------------------------------------------------------------------|:----------------------------------------------------------------------------------------------------------------|
| 2026-10-16 | Hugo Coutinho | `mvn -B clean verify` (in `JavaApp`)                                     | 17 JUnit tests: 17 passed, 0 failed, 0 skipped                                                                  |
| 2026-10-19 | Hugo Coutinho | `mvn -B clean verify` (in `JavaApp`)                                     | 19 JUnit tests: 19 passed, 0 failed, 0 skipped                                                                  |
| 2026-10-21 | Hugo Coutinho | `RobotAssignmentBenchmark <datasets-folder>` (`app-two`, Java 21, 8 processors) | TC-15: `isFeasible` pass over all orders 26.8 µs (medium) / 96.9 µs (large); `assignBestRobot` 38.4 µs / 137.8 µs; time ratio ≈ 2 when R doubles and ≈ 10 when Io is multiplied by 10 (linear) |
| 2026-10-21 | Hugo Coutinho | `mvn -B clean verify` (in `JavaApp`), after #111 (`Collections.min`) and #112 | 19 JUnit tests: 19 passed, 0 failed, 0 skipped                                                                  |

---

## 🐞 Defect & Bug Tracking

| Issue ID | Summary / Description | Severity | Priority | Status | Linked Commit / Fix | Re-test Status |
|:---------|:----------------------|:---------|:---------|:-------|:--------------------|:--------------:|
| —        | None identified yet   | —        | —        | —      | —                   |       —        |

---

## 🤖 Unit Tests

Implemented JUnit tests (`RobotAssignmentServiceTest`, module `app-two`):

| Test                                                                    | Covers |   Status   |
|:------------------------------------------------------------------------|:-------|:----------:|
| `assignBestRobot_singleEligibleRobot_isAssigned`                        | TC-01  | ✅ Passing |
| `assignBestRobot_severalEligible_choosesHighestBattery`                 | TC-02  | ✅ Passing |
| `assignBestRobot_robotNotReady_isIgnored`                               | TC-03  | ✅ Passing |
| `assignBestRobot_overWeight_returnsEmpty`                               | TC-04  | ✅ Passing |
| `assignBestRobot_overVolume_returnsEmpty`                               | TC-05  | ✅ Passing |
| `assignBestRobot_refrigeratedItemWithoutRefrigeratedRobot_returnsEmpty` | TC-06  | ✅ Passing |
| `assignBestRobot_refrigeratedItem_choosesRefrigeratedRobot`             | TC-07  | ✅ Passing |
| `assignBestRobot_exactLimits_robotIsEligible`                           | TC-08  | ✅ Passing |
| `assignBestRobot_batteryTie_isDeterministic`                            | TC-09  | ✅ Passing |
| `assignBestRobot_emptyFleet_returnsEmpty`                               | TC-10  | ✅ Passing |
| `assignBestRobot_assigned_updatesRobotAndStation`                       | TC-11  | ✅ Passing |
| `assignBestRobot_notFeasible_changesNothing`                            | TC-12  | ✅ Passing |
| `assignBestRobot_invalidOrder_isRejected`                               | TC-14  | ✅ Passing |
| `isFeasible_eligibleRobotExists_returnsTrue`                            | TC-16  | ✅ Passing |
| `isFeasible_noEligibleRobot_returnsFalseAndChangesNothing`              | TC-17  | ✅ Passing |
| `assignBestRobot_invalidData_isRejected`                                | TC-18  | ✅ Passing |
| `assignBestRobot_assigned_linksOrderToRobotAndSetsAssigned`             | TC-19  | ✅ Passing |
| `assignBestRobot_ordinaryItem_ignoresRefrigeratedRobot`                 | TC-20  | ✅ Passing |
| `assignBestRobot_mixedOrder_returnsEmpty`                               | TC-21  | ✅ Passing |

TC-13 (blocked) has no JUnit test. TC-15 (performance) is the manual benchmark `RobotAssignmentBenchmark`, which is not a JUnit test.

---

## 📊 Summary & QA Sign-Off

### Execution Results
* **Total Test Cases:** 21
* **JUnit tests:** 19 / 19 implemented
* **Passed:** 20 (19 JUnit tests + TC-15, the benchmark)
* **Failed:** 0
* **Not run:** 1 (TC-13, blocked until the minimum battery rule is defined)

### Final QA Recommendation

> 🟡 **Awaiting review:** the story is implemented, its 19 JUnit tests pass and the performance study is done. QA sign-off must be given by a team member other than the author.

* [ ] **QA Sign-off Approved**
* **Signed By:** TBD
* **Date:** TBD