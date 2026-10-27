# 🧪 QA & Test Documentation: [USEI02] Station Monitoring and Battery-Based Sorting

* **User Story Reference:** [USEI02: Station Monitoring and Battery-Based Sorting](./readme.md) — GitHub issue [#8](https://github.com/Departamento-de-Engenharia-Informatica/SEM3PI-26-27-2DE02/issues/8)
* **QA Lead / Owner:** Tiago Borges (1231109)
* **Developer:** João Ferreira (1231752)
* **Target Environment:** local, Java 21, Maven; ShopRobot application (`com.example.shoprobot.Main`)
* **Build / Release Version:** `main` at commit `82e337f` (2026-10-19); re-test at commit `76ab076` (2026-10-19)
* **QA Status:** 🟢 Passed — all issues closed after re-test (see [issues](#-defect--bug-tracking))

---

## 🎯 QA Strategy & Scope

### In-Scope
* Acceptance criteria AC1 (station status), AC2 (parked robots with status and battery) and AC3 (sorting with a custom `Comparator`).
* Functional scenarios TC-01 to TC-07 of the [US readme](./readme.md#-functional-tests--validation), plus extra scenarios on the medium and large datasets.
* Unit tests and JaCoCo coverage of the USEI02 classes.
* ESINF Sprint 1 restriction (no `PriorityQueue` or heap-based structures) and the USEI02 section of the [ESINF report](../../ESINF/report.md#usei02--parking-station-monitoring-and-control-console).
* Completeness of the US documentation.

### Out-of-Scope
* Changing the status of robots and updating the station availability after an assignment (USEI04).
* Data import and its error handling (USEI01).

---

## 🛠️ Test Environment & Setup

* **How to run:** `com.example.shoprobot.Main` → **2 - Planning** → **1 - Station monitoring**.
* **Datasets:** `data/medium/clean` (default) and `data/large/clean` (with `SHOPROBOT_DATA_DIR`).
* **Expected values** were recomputed directly from `robots.csv` and `stations.csv` (robots per station, `BUSY` robots, battery levels).

---

## 📋 Test Matrix

| Test ID | Title / Scenario | Type | Pre-conditions | Test Steps | Expected Result | Status | Execution Date | Bugs Filed |
| :--- | :--- | :--- | :--- | :--- | :--- | :---: | :---: | :--- |
| **TC-01** | Status of every station (AC1) | Functional | `medium/clean` | Open *Station monitoring* | 6 stations `S001`–`S006` sorted by id; occupied + free = capacity in every station (e.g. `S001`: 10 / 4 / 6, `S005`: 12 / 7 / 5) | 🟢 PASS | 2026-10-19 | None |
| **TC-02** | Parked robots of a station (AC2) | Functional | `medium/clean` | Type `S001` | `R0018` READY 100%, `R0029` CHARGING 82%, `R0003` READY 78%, `R0006` READY 66% | 🟢 PASS | 2026-10-19 | None |
| **TC-03** | Sorting by battery (AC3) | Functional | TC-02 | Check the order of TC-02 | Battery levels in descending order | 🟢 PASS | 2026-10-19 | None |
| **TC-04** | `BUSY` robot not parked | Functional | `S001` has `R0010` (BUSY, 44%) in `robots.csv` | Check `S001` in TC-01 and TC-02 | `R0010` not listed and not counted (4 occupied, not 5); same for `S003` / `R0016` | 🟢 PASS | 2026-10-19 | None |
| **TC-05** | Station with no parked robots | Unit | Not in the datasets | `StationMonitorTest#stationWithoutRobotsHasAllSpotsFree`, `#stationWithoutRobotsHasNoParkedRobots` | Free spots = capacity; empty list | 🟢 PASS | 2026-10-19 | None |
| **TC-06** | Unknown station | Negative | `medium/clean` | Type `S099` | `Station S099 not found.` | 🟢 PASS | 2026-10-19 | None |
| **TC-07** | Cancel | Functional | `medium/clean` | Press Enter without an id | Back to the Planning menu | 🟢 PASS | 2026-10-19 | None |
| **TC-08** | `MAINTENANCE` robot shown (AC2) | Functional | `medium/clean`, `S002` has `R0004` in maintenance | Type `S002` | `R0026` 60%, `R0009` 50%, `R0004` MAINTENANCE 43%, `R0011` CHARGING 18% | 🟢 PASS | 2026-10-19 | None |
| **TC-09** | Same battery ordered by id (AC3) | Functional | `large/clean`, `S010` has `R0018` and `R0024` with 82% | Type `S010` | `R0018` before `R0024`; 9 robots in descending battery order | 🟢 PASS | 2026-10-19 | None |
| **TC-10** | Large dataset | Functional | `SHOPROBOT_DATA_DIR=data/large/clean` | Open *Station monitoring* | 12 stations `S001`–`S012` sorted by id; occupied + free = capacity | 🟢 PASS | 2026-10-19 | None |
| **TC-11** | Spaces around the id | Robustness | `medium/clean` | Type `  S001  ` | Same result as TC-02 | 🟢 PASS | 2026-10-19 | None |
| **TC-12** | Id in lower case (readme TC-08 after QA-04) | Robustness | `medium/clean` | Type `s001` | Same robots as TC-02, under `Parked robots at S001 (Entrada Norte)` | 🟢 PASS (re-test) | 2026-10-19 | [QA-04](#-defect--bug-tracking) |
| **TC-13** | Unit tests | Automated | — | `mvn clean test` in `JavaApp` | All USEI02 tests pass | 🟢 PASS | 2026-10-19 | None |
| **TC-14** | Coverage ≥ 80% | Automated | — | `mvn verify` (JaCoCo) | Line coverage ≥ 80% in every USEI02 class | 🟢 PASS | 2026-10-19 | None |
| **TC-15** | No heap-based structures | Code review | — | Review `StationMonitor`, `RobotBatteryComparator` | Only `HashMap`, `ArrayList`, `Map.copyOf` and `List.sort` with a `Comparator`; no `PriorityQueue` | 🟢 PASS | 2026-10-19 | None |
| **TC-16** | ESINF report and benchmark | Review | — | Read the USEI02 section; run `StationMonitorBenchmark` twice | Section complete; growth from medium to large reproducible | 🟢 PASS | 2026-10-19 | None |
| **TC-17** | US documentation complete | Review | — | Read the [US readme](./readme.md) | DoD and texts up to date; diagrams listed in the DoD present | 🟢 PASS (re-test) | 2026-10-19 | [QA-01](#-defect--bug-tracking), [QA-02](#-defect--bug-tracking) |
| **TC-18** | Regression after the fixes | Regression | `medium/clean`, `large/clean` | Repeat TC-01, TC-02, TC-06 and TC-09 | Same results as in the first run | 🟢 PASS | 2026-10-19 | None |

### Execution Log

**TC-01 / TC-02 (`medium/clean`):**
```
Parking stations:
  S001 | Entrada Norte | capacity 10 | occupied 4 | free 6
  S002 | Entrada Sul | capacity 12 | occupied 4 | free 8
  S003 | Praca Central | capacity 10 | occupied 4 | free 6
  S004 | Zona Restauracao | capacity 12 | occupied 3 | free 9
  S005 | Cinema | capacity 12 | occupied 7 | free 5
  S006 | Hipermercado | capacity 12 | occupied 6 | free 6

Parked robots at S001 (Entrada Norte):
  - R0018 | READY | battery 100%
  - R0029 | CHARGING | battery 82%
  - R0003 | READY | battery 78%
  - R0006 | READY | battery 66%
```
Recomputed from `robots.csv`: `S001` has 5 robots, one of them `BUSY` (`R0010`); `S003` has 5, one `BUSY` (`R0016`); the other stations have no `BUSY` robots — matches the occupied spots above.

**TC-09 (`large/clean`, `S010`):**
```
  - R0001 | CHARGING | battery 87%
  - R0011 | READY | battery 85%
  - R0061 | MAINTENANCE | battery 84%
  - R0018 | MAINTENANCE | battery 82%
  - R0024 | MAINTENANCE | battery 82%
  - R0068 | CHARGING | battery 68%
  - R0051 | READY | battery 66%
  - R0038 | READY | battery 45%
  - R0060 | MAINTENANCE | battery 40%
```

**TC-13 — unit tests (all passing):** `RobotBatteryComparatorTest` (4), `StationStatusTest` (7), `StationMonitorTest` (8), `StationMonitoringControllerTest` (6), `StationMonitoringUITest` (6), `PlanningMenuTest` (7), `PlanningDataSettingsTest` (3), `PlanningMainTest` (2).

**TC-14 — JaCoCo line / branch coverage:** `StationMonitor` 100% / 100%, `StationStatus` 100% / 90%, `RobotBatteryComparator` 100% / 100%, `StationMonitoringController` 100% / 100%, `StationMonitoringUI` 92.9% / 87.5%, `PlanningMenu` 100% / 100%, `PlanningDataSettings` 100% / 100%.

**TC-16 — `StationMonitorBenchmark` on the QA machine (2 runs, µs):**

| Dataset | Build index | Status of all stations | Parked robots of one station |
| :--- | ---: | ---: | ---: |
| medium | 1.89 / 2.12 | 0.98 / 0.94 | 0.64 / 0.58 |
| large | 4.54 / 4.51 | 1.37 / 1.47 | 0.40 / 0.33 |

The absolute times are higher than in the report (different machine), but the conclusions hold: building the index grows ≈ 2.2× (report: 2.3×, expected 2.4×), and the other two operations do not follow O(S log S) / O(k log k) at these small sizes (≈ 1.5× and ≈ 0.6×).

### Re-test after the fixes (commit `76ab076`, 2026-10-19)

Fixes by the developer: `dfc62b3` (readme updated), `7839298` (class diagram), `748d828` (station ids ignoring case) and `ca8f3cb` (QA-03 and QA-04 documented in the readme and in the ESINF report).

* **Unit tests:** all pass — `StationMonitorTest` now 10, `StationMonitoringControllerTest` 7 and `StationMonitoringUITest` 7 (new tests for ids in another case); the other classes as before.
* **Coverage:** `StationMonitor` 100% / 100%, `StationMonitoringController` 100% / 100%, `StationMonitoringUI` 93.0% / 87.5%, the others unchanged.
* **TC-12 (`s001`):**
  ```
  Parked robots at S001 (Entrada Norte):
    - R0018 | READY | battery 100%
    - R0029 | CHARGING | battery 82%
    - R0003 | READY | battery 78%
    - R0006 | READY | battery 66%
  ```
* **TC-18 (regression):** `S001` gives the same status and robots as before, `S099` still gives `Station S099 not found.`, and `s010` on `large/clean` gives the same order as TC-09 (`R0018` before `R0024`).
* **TC-17 (documentation):** the readme DoD is up to date, the "will measure" sentence was replaced by the results, `cd.puml` / `cd.svg` and `sd.puml` / `sd.svg` are in the folder and render correctly, all relative links work. The complexity of building the index was updated to O(S + R) in the readme and in the ESINF report, with a new benchmark run.

---

## 🐞 Defect & Bug Tracking

| Bug ID | Summary / Description | Severity | Priority | Status | Linked Commit / Fix | Re-test Status |
| :--- | :--- | :--- | :--- | :--- | :--- | :---: |
| **QA-01** | Readme out of date: the DoD item "Complexity analysis and empirical study … included in the ESINF report" is not ticked, although the report section is complete; the *Main operations and time complexity* section still says "The empirical study … **will** measure each operation". | Low | Medium | Fixed | `dfc62b3` | 🟢 Passed |
| **QA-02** | The DoD asks for **class and sequence diagrams**, but only the sequence diagram exists ("Class diagram: to be added"). Add the class diagram or change the DoD item. | Low | Medium | Fixed | `7839298` | 🟢 Passed |
| **QA-03** | *(Observation, not a defect of USEI02)* `StationMonitor` indexes the parked robots once, when it is created. If USEI04 changes a robot from `READY` to `BUSY` during the session, the station view will not reflect it until the monitor is created again. To be considered in the integration with USEI04 ("updated station availability"). | Low | Low | Accepted — documented for the integration with USEI04 | `ca8f3cb` | 🟢 Passed |
| **QA-04** | *(Observation)* Station ids are case-sensitive: `s001` is reported as not found. Optional: compare ids ignoring case in the controller. | Low | Low | Fixed | `748d828`, `ca8f3cb` | 🟢 Passed |

---

## 🤖 Unit Tests

* **Related App:** JavaApp
* **Related Code:** `model/domain/StationMonitor`, `StationStatus`, `RobotBatteryComparator`; `controller/planning/StationMonitoringController`; `view/planning/StationMonitoringUI`, `PlanningMenu`; `model/importer/PlanningDataSettings`
* **Automated Tests:** `RobotBatteryComparatorTest`, `StationStatusTest`, `StationMonitorTest`, `StationMonitoringControllerTest`, `StationMonitoringUITest`, `PlanningMenuTest`, `PlanningDataSettingsTest`, `PlanningMainTest`
* **Execution:** `mvn clean test` (coverage: `mvn verify`, report in `JavaApp/target/site/jacoco`)

---

## 📊 Summary & QA Sign-Off

### Execution Results
* **Total Executed:** 18 (after re-test)
* **Passed:** 18
* **Failed:** 0
* **Observations:** 0
* **Blocked:** 0

*First run: 15 passed, 1 failed (TC-17, documentation), 1 observation (TC-12).*

### Final QA Recommendation

> 🟢 **Approved:** the three acceptance criteria are met on the medium and large datasets, the results match the CSV files, the unit tests pass with coverage ≥ 80% and no heap-based structure is used. After the re-test, QA-01, QA-02 and QA-04 are fixed and QA-03 is documented for the integration with USEI04. The user story can be moved to **Done**.

* [x] **QA Sign-off Approved**
* **Signed By:** Tiago Borges (QA)
* **Date:** 2026-10-19
