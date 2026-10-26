# QA & Test Execution Report: USEI04

## USEI04 — Single-Robot Feasibility and Allocation

* **User Story Reference:** [USEI04: Single-Robot Feasibility and Allocation](./readme.md)
* **Issue:** [#10 — USEI04 Single-Robot Feasibility and Allocation](https://github.com/Departamento-de-Engenharia-Informatica/SEM3PI-26-27-2DE02/issues/10)
* **Developer:** Francisco Marques
* **QA Lead / Owner:** João Ferreira
* **Implementation Area:** Planning area (menu *2 - Planning → 3 - Single-robot assignment*)
* **Execution Date:** 2026-10-20 (commit `c918852`)
* **Environment:** Java 21, Maven, JUnit 5, JaCoCo; datasets `data/medium/clean` and `data/large/clean`
* **QA Status:** 🟢 Passed — functionality approved; documentation fixes requested (QA-01, QA-02)

---

## 🎯 QA Strategy & Scope

### In-Scope

* Feasibility result of an order (AC1): `READY`, minimum battery, max weight, max volume and refrigeration.
* Assignment to the most suitable robot (AC2): highest battery, ties by robot id.
* Robot status `READY → BUSY`, station availability and pending order queue after an assignment (AC3), including the integration with USEI02 (*Station monitoring*) and USEI03 (*Order queue*).
* Edge cases of the readme (values equal to the maximum, minimum battery, already assigned or unknown orders, not feasible orders).
* Sprint 1 ESINF restriction (no `PriorityQueue` or heap-based structures).
* Automated unit tests and code coverage; completeness of the user story documentation.

### Out-of-Scope

* Splitting the orders that are not feasible (USEI05).
* The robot coming back to its station after the delivery.
* Database persistence (not required in Sprint 1).
* Review of the complexity analysis and empirical study in the ESINF report.

---

## 🛠️ Review Setup

* **Specification:** Project Specification v1, Section 7.2.1; client sessions of 2026-10-02 and 2026-10-12.
* **User story requirements:** [USEI04 readme](./readme.md).
* **Implementation under review:**
  * `SingleRobotAllocator`, `RobotFleet`, `Robot#withStatus` and `StationMonitor#update` (`model/domain`)
  * `SingleRobotAssignmentController` (`controller/planning`)
  * `SingleRobotAssignmentUI` and option *3* of `PlanningMenu` (`view/planning`)
  * `Main` (the three Planning options share the same `StationMonitor` and `OrderQueue`)
* **Automated tests:** `SingleRobotAllocatorTest`, `RobotFleetTest`, `StationMonitorTest`, `SingleRobotAssignmentControllerTest`, `SingleRobotAssignmentUITest`, `PlanningMenuTest`, `PlanningMainTest`.
* **Independent check:** the expected values were recomputed directly from `robots.csv` and `orders.csv` with a script written by QA (totals per order, `READY` robots sorted by battery then id, minimum battery 20%), not with the application code.

---

## ✅ Acceptance Criteria

* **AC1 — Feasibility result:** a boolean that says whether any available robot satisfies every constraint (max volume, max weight and minimum required battery; refrigeration if needed).
* **AC2 — Automatic assignment:** the order is assigned to the optimal robot (`READY`, sufficient capacity, highest current battery).
* **AC3 — Updated status and station availability:** the selected robot changes from `READY` to `BUSY` and the station availability is updated.

---

## 📋 Test Matrix

| Test ID | Title / Scenario | Type | Test Steps | Expected Result | Status | Execution Date | Issues Found |
| :--- | :--- | :--- | :--- | :--- | :---: | :---: | :--- |
| **TC-01** | Most urgent order, refrigerated | Functional | `medium/clean`, option **3**, Enter | `O00047` (3 items, 13.65 kg, 31.00 L, refrigeration: yes); `Feasible: true`; `R0028` (93%, refrigerated); `READY -> BUSY`; `S006`: occupied 5, free 7 | 🟢 PASS | 2026-10-20 | None |
| **TC-02** | Station availability seen in USEI02 | Functional | After TC-01, option **1** | `S006 \| Hipermercado \| capacity 12 \| occupied 5 \| free 7` (was 6 / 6) | 🟢 PASS | 2026-10-20 | None |
| **TC-03** | Next order, not refrigerated | Functional | Option **3**, Enter | `O00156` (18.00 kg, 76.00 L); `R0018` (100%); `S001`: occupied 3, free 7 | 🟢 PASS | 2026-10-20 | None |
| **TC-04** | Best robot already `BUSY` | Functional | Option **3**, `O00001` | 6.90 kg, 21.30 L, refrigeration: yes; `R0028` is `BUSY`, so `R0008` (87%) | 🟢 PASS | 2026-10-20 | None |
| **TC-05** | Not feasible | Functional | Option **3**, `O00197` | 118.40 kg, 626.00 L; `Feasible: false`; message pointing to USEI05; the order stays pending | 🟢 PASS | 2026-10-20 | None |
| **TC-06** | Order already assigned / unknown | Negative | Option **3**, `O00047` and `O99999` | `Order … is not pending.`; nothing changes | 🟢 PASS | 2026-10-20 | None |
| **TC-07** | Order leaves the pending queue (USEI03) | Functional | After TC-01, TC-03 and TC-04, option **2 → 1** | `O00047` not listed; 197 pending orders | 🟢 PASS | 2026-10-20 | None |
| **TC-08** | Spaces around the order id | Robustness | Option **3**, `  O00002  ` | Same as `O00002`: 25.50 kg, 153.00 L; `R0001` (95%); `S004`: occupied 2, free 10 | 🟢 PASS | 2026-10-20 | None |
| **TC-09** | Order id in lower case | Robustness | Option **3**, `o00002` | `Order o00002 is not pending.` (ids are case-sensitive, as in USEI03) | 🟡 OBS | 2026-10-20 | QA-04 |
| **TC-10** | Feasibility of every order | Independent check | Recompute AC1 for the 200 orders of `medium/clean` and the 600 of `large/clean` at the start | 134 / 200 and 417 / 600 feasible; same best robot for `O00047`, `O00156`, `O00001`, `O00002` (`R0028`, `R0018`, `R0028`, `R0001` before any assignment) and none for `O00197` | 🟢 PASS | 2026-10-20 | None |
| **TC-11** | Totals equal to the robot maximum | Independent check | For every order/robot pair whose total weight or volume equals the robot maximum, check `canCarry` in Java | 14 pairs (medium) and 95 pairs (large), none rejected by floating-point rounding | 🟢 PASS | 2026-10-20 | None |
| **TC-12** | Large dataset | Functional | `SHOPROBOT_DATA_DIR=data/large/clean`, option **3**, Enter | `O00176` (earliest deadline, 10:30) → `R0020` (100%, refrigerated); `S012`: occupied 6, free 6 | 🟢 PASS | 2026-10-20 | None |
| **TC-13** | Most urgent order not feasible | Functional | `large/clean`, after TC-12, Enter twice | `O00089` (86.30 kg > 80 kg, the largest robot): `Feasible: false` both times; Enter keeps selecting the same order | 🟡 OBS | 2026-10-20 | QA-03 |
| **TC-14** | Minimum battery, ties, values equal to the maximum, refrigeration | Unit | `SingleRobotAllocatorTest`, `RobotFleetTest` | Below the minimum: not used; equal: used; ties: lowest id; equal to the maximum: accepted; refrigerated order: only robots with refrigeration | 🟢 PASS | 2026-10-20 | None |
| **TC-15** | Station update | Unit | `StationMonitorTest#busyRobotFreesItsSpotAfterAnUpdate`, `#updatedRobotThatIsStillParkedShowsItsNewStatus` | A `BUSY` robot frees its spot; a robot that is still parked shows its new status | 🟢 PASS | 2026-10-20 | None |
| **TC-16** | No heap-based structures | Static review | Inspect `RobotFleet`, `SingleRobotAllocator` | `TreeSet` with `RobotBatteryComparator`, `HashMap`; no `PriorityQueue` | 🟢 PASS | 2026-10-20 | None |
| **TC-17** | Unit tests and coverage | Automated | `mvn -B clean verify` in `JavaApp` | All tests pass; every new USEI04 class ≥ 80% line coverage | 🟢 PASS | 2026-10-20 | None |
| **TC-18** | US documentation complete | Review | Read the [US readme](./readme.md) | NFR05 respected (figures in SVG); QA and status fields up to date | 🟠 FAIL | 2026-10-20 | QA-01, QA-02 |

---

## 🧾 Execution Log

### Automated tests and coverage (TC-14 to TC-17)

Command executed (in `JavaApp`):

```text
mvn -B clean verify
```

Result:

```text
Tests run: 248, Failures: 0, Errors: 0, Skipped: 0
BUILD SUCCESS
```

USEI04 tests: `SingleRobotAllocatorTest` (16), `StationMonitorTest` (14), `PlanningMenuTest` (9), `RobotFleetTest` (7), `SingleRobotAssignmentControllerTest` (7), `SingleRobotAssignmentUITest` (6), `PlanningMainTest` (3) — all passing.

JaCoCo line / branch coverage:

| Class | Lines | Branches |
| :--- | :---: | :---: |
| `SingleRobotAllocator` | 100.0% | 97.1% |
| `RobotFleet` | 100.0% | 100.0% |
| `StationMonitor` | 100.0% | 100.0% |
| `SingleRobotAssignmentController` | 100.0% | 100.0% |
| `SingleRobotAssignmentUI` | 100.0% | 100.0% |
| `PlanningMenu` | 100.0% | 100.0% |

Total project instruction coverage: 90.5%.

### Functional tests (TC-01 to TC-09)

The application was run with `com.example.shoprobot.Main` (default dataset `data/medium/clean`), choosing **2 - Planning**, with the input sequence `3`/Enter, `1`/Enter, `3`/Enter, `3`/`O00001`, `3`/`O00197`, `3`/`O00047`, `3`/`O99999`, `2`/`1`/`0`, `3`/`o00002`, `3`/`  O00002  `. Output of TC-01:

```text
Order O00047 | 3 items | 13.65 kg | 31.00 L | refrigeration: yes
Feasible: true
Assigned to robot R0028 (battery 93%, max 60.0 kg / 220.0 L, refrigeration: yes)
Robot R0028: READY -> BUSY
Station S006 (Hipermercado): capacity 12 | occupied 5 | free 7
Order O00047 removed from the pending orders.
```

All the results of the readme scenarios TC-01 to TC-07 were obtained as documented.

### Independent check against the CSV files (TC-10 to TC-12)

* `medium/clean`: 16 `READY` robots (lowest battery 50%), 200 orders, **134 feasible** at the start — the value stated in the readme.
* `large/clean`: 40 `READY` robots (lowest battery 45%), 600 orders, **417 feasible** at the start; most urgent order `O00176` (2026-10-05 10:30); best robot `R0020` (100%, refrigerated, station `S012` with 7 parked robots before the assignment).
* Because the total weight and volume of an order are sums of `double` values, every order/robot pair whose total is equal to the robot maximum was checked in Java: none is rejected by rounding, so "values equal to the maximum are accepted" holds for the provided datasets.

### TC-13 — Most urgent order not feasible

With `large/clean`, after the first assignment the most urgent pending order is `O00089` (86.30 kg), which no robot can carry. It stays pending (correct, according to the readme), so pressing Enter again selects it again: to assign the next orders the operator has to type their ids.

---

## 🔍 Traceability to Acceptance Criteria

| Acceptance Criterion | Implementation Evidence | Test Evidence |
| :--- | :--- | :--- |
| **AC1 — Feasibility result** | `SingleRobotAllocator.canCarry`, `findBestRobot`, `isFeasible` | TC-01, TC-05, TC-10, TC-11, TC-13, TC-14 |
| **AC2 — Automatic assignment** | `RobotFleet.readyByBattery` (`TreeSet` + `RobotBatteryComparator`), `SingleRobotAllocator.assign` | TC-01, TC-03, TC-04, TC-08, TC-10, TC-12, TC-14 |
| **AC3 — Status and station availability** | `RobotFleet.markBusy`, `Robot.withStatus`, `StationMonitor.update`, `OrderQueue.remove` | TC-01, TC-02, TC-03, TC-07, TC-15 |

---

## 🐞 Issues Found

| Issue ID | Summary / Description | Severity | Status | Recommended Action |
| :--- | :--- | :--- | :--- | :--- |
| **QA-01** | The class and sequence diagrams are rendered as **PNG** (`cd.png`, `sd.png`), and the DoD says "PlantUML + PNG". NFR05 requires all images/figures in **SVG**, as in the other user stories. | Low | Open | Generate `cd.svg` and `sd.svg` from the `.puml` files (e.g. `generate-diagrams.sh` with `lib/plantuml.jar`), update the readme links and the DoD, and remove the PNG files. |
| **QA-02** | Readme out of date: header *QA: To be assigned* (the QA is João Ferreira) and the QA section still says *To be completed by QA*. | Low | Open | Set the QA name and link this report; tick AC1–AC3 and the DoD item "Acceptance criteria verified by QA". |
| **QA-03** | *(Observation)* When the most urgent pending order is not feasible, pressing Enter keeps selecting it, so the operator cannot move on to the next orders without typing their ids (TC-13). | Low | Open | Optional: show a hint (e.g. "type the id of another order"), or let Enter pick the most urgent **feasible** order. Not a defect of the acceptance criteria; to be considered together with USEI05. |
| **QA-04** | *(Observation)* Order ids are case-sensitive (`o00002` → *not pending*), as in the order queue of USEI03. | Low | Open | Optional: find the order ignoring case, as was done for the station ids of USEI02. |
| **QA-05** | *(Observation)* The minimum battery (20%) is a team assumption, documented in the readme as *to confirm with the client*. With the provided datasets it never changes the result (every `READY` robot has at least 45%). | Low | Open | Ask the client in the Moodle forum; the value is a single constant (`SingleRobotAllocator.DEFAULT_MINIMUM_BATTERY`). |

No failures were found in the automated tests nor in the functional tests of the acceptance criteria.

---

## 📊 Summary & QA Sign-Off

### Execution Results

* **Total test cases:** 18
* **Executed:** 18
* **Passed:** 15
* **Failed:** 1 (TC-18, documentation)
* **Observations:** 2 (TC-09, TC-13)
* **Blocked:** 0
* **Open QA issues:** 5 (QA-01 and QA-02 to fix; QA-03 to QA-05 are observations)

### Final QA Recommendation

> 🟢 **Approved — functionality:** the three acceptance criteria are met on the medium and large datasets; the results match an independent recomputation from the CSV files; the robot status, the station availability (USEI02) and the pending order queue (USEI03) are updated after each assignment; all unit tests pass with 100% line coverage of the new classes; no heap-based structure is used.
>
> 🟠 **Documentation:** QA-01 and QA-02 should be fixed before closing the issue. QA-03 to QA-05 are observations and do not block the user story.

* [x] **QA Sign-off Approved** (functionality)
* **Signed By:** João Ferreira
* **Date:** 2026-10-20

---

*AI usage: test execution, the independent check and the first draft of this report were prepared with the help of Claude (Anthropic) and reviewed by the QA owner.*
