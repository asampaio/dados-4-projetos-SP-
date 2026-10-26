# [USEI02] Station Monitoring and Battery-Based Sorting

* **Issue:** [#8](https://github.com/Departamento-de-Engenharia-Informatica/SEM3PI-26-27-2DE02/issues/8)
* **Priority:** Urgent
* **Story Points:** 5
* **Module / Component:** ESINF / Planning area of the ShopRobot application (menu *2 - Planning → 1 - Station monitoring*)
* **Status:** In Progress
* **Developer:** João Ferreira (1231752)
* **QA:** Tiago Borges (1231109)

---

## 👤 User Story Description

**As a** shopping facility operator  
**I want to** consult the status of parking stations to monitor their total and available capacity, as well as the robots currently parked, including their status and battery levels sorted in descending order  
**So that** I know which stations have free spots and which parked robots are best prepared for the next orders

---

## 🚦 Definition of Ready (DoR)

* [x] **Clear User Story:** Story follows the standard *As a / I want to / So that* format (Issue #8).
* [x] **Acceptance Criteria Defined:** ACs taken from GitHub Issue #8 and Project Specification v1 (Section 7.2.1).
* [x] **Dependencies Identified:** robots and stations imported by USEI01 (`CsvDataImporter`).
* [x] **Inputs/Outputs Mapped:** Inputs and outputs documented below.
* [x] **Estimation Complete:** 5 story points (Sprint 1 planning).
* [x] **Small & Actionable:** In-memory feature of the Planning area, completed within Sprint 1.

---

## 💬 Customer Specifications & Clarifications

* **Project statement (v1, Section 7.2.1):**
  * Choose the most appropriate JCF interfaces and classes for each entity/relation.
  * `PriorityQueue` (or any other heap-based structure) is **not allowed** in Sprint 1; sorting uses standard collections with a `Comparator` or JCF sorting utilities.
  * Database integration is not mandatory in Sprint 1: data may be processed only in memory.
  * A report must identify the main operations and data structures, justify the JCF choices and derive the time complexity (R: robots, S: stations), supported by an empirical study with the medium and large datasets.
* **Clarification (2026-10-02, LAPR3 client session):** robots always have a status; for the first ESINF user stories the status is treated as fixed. The medium and large datasets (clean and with errors) are provided for the demo and for error-handling tests.
* **Clarification (2026-10-12, LAPR3 client session):** a `BUSY` state (robot working on an order) is allowed.
* See the [requirements Q&A log](../../requirements/requirements.md).
* **Out of Scope:** changing the status of robots, moving robots between stations, and database persistence.

---

## ✅ Acceptance Criteria

* [x] **AC1 - Station status:** a clear status view per station detailing maximum capacity, occupied spots and free spots.
* [x] **AC2 - Parked robots:** a list of the parked robots of a selected station showing their operational status (`READY`, `CHARGING`, `MAINTENANCE`) and battery level.
* [x] **AC3 - Sorting:** the robot list is sorted by a custom `Comparator` implementation (highest battery level first).

---

## 🔒 Non-Functional Requirements & Architecture

* **NFR04 - Object-Oriented Design (MVC):**
  * **Model:** `StationMonitor` (builds the station views from the imported robots and stations), `StationStatus` (status of one station) and `RobotBatteryComparator` (sorting rule), in `model/domain`; `PlanningDataSettings` (folder of the CSV files), in `model/importer`.
  * **Controller:** `StationMonitoringController` (`controller/planning`), called by the view and delegating to the model.
  * **View:** `StationMonitoringUI` (`view/planning`), opened from `PlanningMenu`, the text menu of the Planning area.
* **NFR05 - Documentation:** Javadoc on all public classes and methods.
* **ESINF Sprint 1 restriction:** no `PriorityQueue` or heap-based structures.

---

## 🔗 Found Out Dependencies

### Upstream Dependencies (Required before starting)
* **[USEI01] CSV import:** `CsvDataImporter#importDirectory(Path)` returns an `ImportResult` with `Map<String, Robot>` and `Map<String, RobotStation>` — [USEI01 readme](../USEI01/readme.md).

### Downstream Dependencies (Impacted by this story)
* **[USEI04] Single-robot assignment:** can reuse the station index and the battery comparator (highest battery first) to find the most suitable `READY` robot, and needs the updated station availability.
  * **To keep in mind for the integration (QA-03):** `StationMonitor` builds its index of parked robots once, when it is created, because in this story the robot status is fixed (client session of 2026-10-02). When USEI04 changes a robot from `READY` to `BUSY`, the station view only shows it after the index is updated: either the `StationMonitor` is created again from the current robots each time the console is opened, or it gets a method to update one robot (O(k) for its station). The choice is to be agreed with the USEI04 developer.
* **[USEI05] Multi-robot assignment:** same as USEI04.

---

## 📥 Input & Output Data

### Inputs
| Field Name | Data Type | Required | Source | Example / Constraints |
| :--- | :--- | :--- | :--- | :--- |
| `robots` | `Map<String, Robot>` | Yes | USEI01 import (`robots.csv`) | `id`, `batteryPercentage` (0–100), `status`, `stationId` |
| `stations` | `Map<String, RobotStation>` | Yes | USEI01 import (`stations.csv`) | `id`, `name`, `maxCapacity` (> 0) |
| `stationId` | `String` | Yes (AC2) | User input | Identifier of an existing station, e.g. `S001` |

### Outputs
| Field Name | Data Type | Destination | Example / Description |
| :--- | :--- | :--- | :--- |
| Station status list | `List<StationStatus>` | Planning text UI | `S001 – Entrada Norte – capacity 10 – occupied 4 – free 6` |
| Parked robots | `List<Robot>` (sorted) | Planning text UI | `R0018 – READY – 100%`, `R0029 – CHARGING – 82%`, … |
| Error message | `String` | Planning text UI | `Station ST99 not found.` |

---

## 🧠 Design

### Business rules

* A robot is **parked** at a station when its `stationId` is the station's id and its status is `READY`, `CHARGING` or `MAINTENANCE`. A `BUSY` robot is working on an order, so it does not occupy a spot.
* **Occupied spots** = number of parked robots; **free spots** = `maxCapacity` − occupied spots.
* **Sorting:** battery level in descending order; robots with the same battery are ordered by `id` (ascending), so the output is deterministic.

### Data structures (JCF)

| Structure | Use | Justification |
| :--- | :--- | :--- |
| `Map<String, RobotStation>` (immutable copy, `Map.copyOf`) | stations by id | O(1) search of the selected station. The map does not keep the input order, so the status view is sorted by station id |
| `Map<String, String>` (`HashMap`) | station ids indexed by their upper-case form | built once in O(S); finds the station typed by the operator ignoring case (e.g. `s001` → `S001`) in O(1) |
| `Map<String, List<Robot>>` (`HashMap` of `ArrayList`) | parked robots indexed by station id | built once in O(R); avoids going through all robots for each station |
| `List<StationStatus>` (`ArrayList`, sorted by station id) | status view of all stations | one element per station, read sequentially by the UI; returned as an unmodifiable list |
| `List<Robot>` (`ArrayList`) + `RobotBatteryComparator` | sorted parked robots of one station | copy of the station list sorted with `List.sort` (TimSort) and a custom `Comparator`; no heap-based structure; returned as an unmodifiable list |

### Main operations and time complexity

R = number of robots, S = number of stations, k = number of robots parked in the selected station (k ≤ R).

| Operation | Steps | Complexity |
| :--- | :--- | :--- |
| Build the station index | index the station ids ignoring case (O(S)), then go through all robots once and add each parked robot to the list of its station (O(R)) | O(S + R) |
| Status of all stations (AC1) | for each station, get the size of its list in the index (O(S)), then sort the statuses by station id | O(S log S) |
| Parked robots of one station (AC2, AC3) | find the station's list in O(1), copy it and sort it with the comparator | O(k log k) |
| Whole use case (index + status + one station) | | O(R + S log S + k log k) |

The empirical study measured each operation on the clean datasets (medium R = 30, S = 6; large R = 72, S = 12). Building the index grew 2.3× for 2.4× more robots, which confirms O(R); the other two operations stay almost constant at these small sizes, because their time is dominated by constant costs and not by the sorting. The justification, the measurements and the discussion are in the [ESINF report](../../ESINF/report.md#usei02--parking-station-monitoring-and-control-console).

### How to run

1. Run `com.example.shoprobot.Main` (from the repository root or from `JavaApp`).
2. Choose **2 - Planning** and then **1 - Station monitoring**.
3. Type a station id (e.g. `S001`), or press Enter to go back.

The data is imported from `data/medium/clean` (see [data/README.md](../../../data/README.md)). Another folder can be used with the environment variable `SHOPROBOT_DATA_DIR` (e.g. `data/large/clean` for the empirical study).

### Edge cases

* **Station not found:** the UI shows an error message and lists nothing.
* **Station id in another case** (e.g. `s001`): the station is found ignoring upper and lower case (QA-04); an id that matches exactly is preferred.
* **Station with no parked robots:** occupied = 0, free = `maxCapacity`, and the robot list is empty.
* **Robot whose `stationId` does not exist in `stations`:** not counted in any station.
* **More parked robots than the capacity** (invalid data): free spots shown as 0 and the station marked as over capacity.

---

## 🧪 Functional Tests / Validation

| Scenario ID | Test Case / Steps | Expected Result |
| :--- | :--- | :--- |
| **TC-01** | Open *Station monitoring* with `data/medium/clean` | 6 stations (`S001`–`S006`) sorted by id, e.g. `S001`: capacity 10, occupied 4, free 6; occupied + free = capacity in every station |
| **TC-02** | Select `S001` | `R0018` (READY, 100%), `R0029` (CHARGING, 82%), `R0003` (READY, 78%), `R0006` (READY, 66%) |
| **TC-03** | Check the order of the list of TC-02 | Battery levels in descending order |
| **TC-04** | Check `S001`, which has the `BUSY` robot `R0010` in `robots.csv` | `R0010` is not listed and is not counted as occupied (4 occupied, not 5) |
| **TC-05** | Station with no parked robots (not in the dataset; covered by `StationMonitorTest`) | Empty list; free spots = capacity |
| **TC-06** | Type `S099` | Error message `Station S099 not found.` |
| **TC-07** | Press Enter without a station id | Back to the Planning menu |
| **TC-08** | Type `s001` (lower case) | Same robots as TC-02, under `Parked robots at S001 (Entrada Norte)` |

**Unit tests:** `RobotBatteryComparatorTest`, `StationStatusTest`, `StationMonitorTest`, `StationMonitoringControllerTest`, `StationMonitoringUITest`, `PlanningMenuTest`, `PlanningDataSettingsTest` and `PlanningMainTest`.

---

# 🧪 QA & Test Documentation

See Report at [QA & Test Documentation: USEI02 Station Monitoring and Battery-Based Sorting](./qa-readme.md) (Tiago Borges, 2026-10-19).

---

## 🧩 Related Modules & Classes

* **Model:** `model/domain/Robot`, `model/domain/RobotStation`, `model/domain/RobotStatus`, `model/importer/CsvDataImporter`; new: `model/domain/StationMonitor`, `model/domain/StationStatus`, `model/domain/RobotBatteryComparator`, `model/importer/PlanningDataSettings`.
* **Controller:** new `controller/planning/StationMonitoringController`.
* **View:** new `view/planning/StationMonitoringUI` and `view/planning/PlanningMenu` (option *2 - Planning* of `MainMenu`).
* **Main:** imports the planning data and creates the station monitoring UI.

---

## 🔄 Diagrams

### Class Diagram (CD)

![USEI02 Class Diagram](./cd.svg)

*Source file:* [cd.puml](./cd.puml)

The diagram shows the MVC classes of the user story: `PlanningMenu` opens the `StationMonitoringUI`, which only talks to the `StationMonitoringController`; the controller delegates to the `StationMonitor`, which keeps the stations and the index of parked robots, creates the `StationStatus` of each station and sorts the parked robots with the `RobotBatteryComparator` (a `Comparator<Robot>`). The robots and stations come from the `ImportResult` of USEI01.

### Sequence Diagram (SD) — Station status and parked robots of a station

![USEI02 Sequence Diagram](./sd.svg)

*Source file:* [sd.puml](./sd.puml)

The diagram follows the two parts of the user story:

* **AC1 (steps 3–13):** the controller asks the `StationMonitor` for the status of every station. For each station, the monitor reads its parked robots from the index (`parkedByStation`) and creates a `StationStatus`; the statuses are sorted by station id and returned as an unmodifiable list. The UI shows the capacity, occupied and free spots of each station.
* **AC2 and AC3 (steps 14–32):** the operator types a station id. An empty input goes back to the menu; an unknown station shows an error; otherwise the monitor copies the parked robots of the station from the index and sorts them with the `RobotBatteryComparator` (highest battery first, then robot id) before the UI shows them.
* The index itself is built once, when the `StationMonitor` is created in `Main` (see the note at the top).

---

## 🏁 Definition of Done (DoD)

* [x] Acceptance criteria verified by QA ([QA report](./qa-readme.md), 2026-10-19).
* [x] Unit tests written and passing (comparator, station status, parked robots, edge cases).
* [x] Javadoc on all public classes and methods.
* [x] Complexity analysis and empirical study (medium and large datasets) included in the ESINF report.
* [x] Class and sequence diagrams in the repository (PlantUML + SVG).
