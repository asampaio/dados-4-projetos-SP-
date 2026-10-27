# [USEI02] Parking Station Monitoring and Control Console

* **Priority:** Urgent
* **Story Points:** 3
* **Module / Component:** `Planning App` (`app-two`) / `Robot Station Monitoring`
* **Status:** Draft (documentation in progress)

---

## 👤 User Story Description

**As a** shopping facility operator (Maintenance Operator) <br>
**I want to**  consult the status of parking stations <br> 
**So that** I can monitor their total and available capacity, as well as the
robots currently parked, including their status and battery levels sorted in
descending order.

---

## 🚦 Definition of Ready (DoR)

* [ ] **Clear User Story:** Story follows the standard *As a / I want to / So that* format.
* [ ] **Acceptance Criteria Defined:** Clear, testable ACs agreed upon by Product and Engineering.
* [ ] **Dependencies Identified:** Upstream APIs, third-party integrations, or blocking tasks are resolved or scheduled.
* [ ] **Inputs/Outputs Mapped:** Required data fields, payloads, and error states are documented.
* [ ] **Estimation Complete:** Story has been refined and estimated (story points assigned) by the dev team.
* [ ] **Small & Actionable:** Story is scoped down enough to be completed within a single sprint.

---

## 💬 Customer Specifications & Clarifications

* `at Requirements doc/4.2 Robot Stations` Each shopping location includes one or more robot stations. (At most 1 per level, except the parking levels)
* `at Requirements doc/4.2 Robot Stations` A robot station is where robots park. It is used when robots are not in active use. <br> 
* There they can charge their batteries. These stations can also be used for other purposes, such as robot **maintenance**.
* `at Requirements doc/4.2 Robot Stations` Each station includes a control terminal that is used for several management tasks. <br>
* For instance, a Maintenance Operator could use the terminal to get information about its capacity, parked robots and their state. <br> 
* It could also be used by the operator to change the state of robots (for instance, set a robot in maintenance) or view a schedule of robot movements (i.e., robots entering or leaving the station).
* `at Requirements doc/4.2 Robots` Each robot should have a unique identification. Robots can be in one of several states. <br>
  For instance, they can be in use, satisfying a specific order of a customer; <br>
  they can be in
  maintenance; <br>
  they can be idle in a robot station; <br>
  they can be charging in a robot station.
<br><br>
* **Station Capacity Metrics**: For the accessed parking station, the console must display total maximum capacity ($S_{max}$), current occupied spots ($S_{occ}$), and remaining free spots ($S_{free} = S_{max} - S_{occ}$).
* **Parked Robot Attributes:** Displays the list of robots currently parked at the selected station, showing their unique `Robot ID`, operational status (`READY`, `CHARGING`, `MAINTENANCE`), and current battery level (%).
* **Custom Comparator Sorting Requirement:** Robot outputs must be sorted in descending order of battery level (highest battery first).
* **Strict JCF Constraint (Sprint 1 Rule):** The use of `PriorityQueue` or any heap-based data structure is **strictly prohibited** in Sprint 1\. Sorting must be implemented using standard Java Collection Framework (JCF) collections combined with an explicit `Comparator<Robot>` (or `List.sort()` / `Collections.sort()`).
* **In-Memory Operations:** In Sprint 1, database persistence is not required; data is managed entirely in memory via JCF data structures loaded during initialization (USEI01).
* **Out of Scope:** 

---

## ✅ Acceptance Criteria

* [ ] **AC1** - A clear status view per station detailing maximum capacity,
  occupied spots, and free spots. 
* [ ] **AC2** - A list of parked robots for a selected station showing their operational status (READY, CHARGING, MAINTENANCE) and battery levels.
* [ ] **AC3** -  Sorted robot outputs driven by custom Comparator implementation (e.g., highest battery level first)


---

## 🔒 Non-Functional Requirements & Security

* **Algorithmic Complexity &amp; Temporal Derivation:**
  * Robot Sorting: $O(R_{parked} \log R_{parked})$ where $R_{parked}$ is the number of robots parked at the selected station, using List.sort().
* **JCF Rules Compliance:** Strict adherence to Sprint 1 constraints — **No** **PriorityQueue** **or heap structures allowed**.
* **Memory Efficiency:** Avoids copying large object graphs when retrieving station views.
* **Coding Standards &amp; Documentation:** Follows Java naming conventions (CamelCase), includes Javadoc comments, and includes complexity derivations in the ESINF technical report.

---

## 🔗 Found Out Dependencies

### Upstream Dependencies (Required before starting)
* **[USEI01] Data Import and System Initialization:** Populates the `in-memory JCF collections` (stations and robots) from CSV/text input files.

### Downstream Dependencies (Impacted by this story)
* **[USEI04] Feasibility Check and Single-Robot Assignment:** Uses station occupancy and robot battery/status data to select optimal robots for order dispatch.

---

## 📥 Input & Output Data

### Inputs
| Field Name                        | Data Type | Required       | Source                   | Example / Constraints                                           |
|:----------------------------------|:----------|:---------------|:-------------------------|:----------------------------------------------------------------|
| `stationIndex` / `selectedOption` | int       | Yes            | User Selection from Menu | `1`, `2` or `3` (Selecting any of those from the presented list |
| `selectedStationID`               | String    | Yes (Internal) | Derived from Selection   | `"ST-O1"` (Mapped from the selected list option                 |


### Outputs
| Field Name          | Data Type           | Destination      | Example / Description                                                        |
|:--------------------|:--------------------|:-----------------|:-----------------------------------------------------------------------------|
| `stationSummaryDTO` | `StationSummaryDTO` | Console UI View  | Max: 10; Occupied: 4; Free: 6                                                |
| `parkedRobotsList`  | `List<RobotDTO>`    | Console UI Table | `R-03 (READY, 98%)` <br>`R-01 (CHARGING, 75%)` <br>`R-08 (MAINTENANCE, 40%)` |
| `statusMessage`     | `String`            | Console UI Alert | `"Station has no parked robots"`                                             |

---

## Edge Cases

* **Empty Station (0 Parked Robots):** The console displays capacity (`Max: 10, Occupied: 0, Free: 10`) and shows an informative message: `"No robots currently parked at this station."`
* **Station at 100% Capacity:** Correctly reports `Free: 0` without throwing index or collection bounds errors.
* **Robots with Equal Battery Levels:** The custom `Comparator` includes a secondary tie-breaker comparison (e.g., natural ordering of `Robot ID`) to maintain consistent, deterministic sorting.

---

## 🧪 Functional Tests / Validation

| Scenario ID | Test Case / Steps                                                          | Expected Result                                                                                            |
|:------------|:---------------------------------------------------------------------------|:-----------------------------------------------------------------------------------------------------------|
| **TC-01**   | Query station with 3 parked robots (Batteries: 45%, 92%, 78%)              | Station capacity summary displayed; Robots listed in descending order: 92%, 78%, 45%.                      |
| **TC-02**   | Query station with 0 parked robots                                         | Capacity summary shows `Max: 10, Occupied: 0, Free: 10`; Displays `"No robots parked" message.`            |
| **TC-03**   | Query full station (same number of parked robots as the station's capacity | Capacity summary shows `Max: 10, Occupied: 10, Free: 0`; Displays `"No free spots, all occupied"` message. |                                                                                     |
| **TC-04**   | Query station with 2 robots having identical battery levels                | Both robots displayed with secondary sort by `Robot ID` applied deterministically.                         |

---

# 🧪 QA & Test Documentation

See Report at [QA & Test Documentation: USLP01 Feature Title](qa-readme.md)

---

## 🧩 Related Modules & Classes

* **Frontend:**
  * Component: `StationConsoleUI`
* **Backend:**
  * Controller: `StationMonitoringController`
  * Service: `StationMonitoringService` (responsible for business logic and operations, like computing station's capacity, the sorting logic, etc)
  * Repository/Store: StationRepository (`in-Memory JCF Store`)
* Domain & Comparators: 
  * Domain Entities: `RobotStation`, `Robot`, `RobotStatus` (Enum: `READY`, `CHARGING`, `MAINTENANCE`)
  * Comparator: `RobotBatteryDescendingComparator`(`Comparator<Robot>`)

---

## 🔄 Sequence Diagram

![Sequence Diagram](sd.svg)

## 🏁 Definition of Done (DoD)

* [ ] Acceptance criteria verified by QA / Product Owner.
* [ ] Unit tests written and passing (minimum 80% coverage on new service logic).
* [ ] Operator selects a station in `StationConsoleUI`.
* [ ] `StationConsoleUI` requests station details from `MonitorStationController`.
* [ ] `MonitorStationController` queries `StationRepository` / `StationMonitoringService` for the station instance.
* [ ] `StationMonitoringService` retrieves the list of parked `Robot` entities for that station.
* [ ] `StationMonitoringService` applies `List.sort(new RobotBatteryDescendingComparator())` to sort the robots in $O(R \log R)$ time.
* [ ] The sorted DTO list and station summary metrics are returned to `StationConsoleUI` for formatted console rendering.
