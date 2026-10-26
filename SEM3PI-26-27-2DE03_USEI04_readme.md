# [USEI04] Feasibility Check and Single-Robot Assignment

* **Priority:** High (critical for the order assignment workflow)
* **Story Points:** 5 (size M, as estimated on the project board)
* **Module / Component:** `Planning App` (`app-two`) / `Order Assignment`
* **Status:** In Review (implementation done, 19 JUnit tests passing, performance study done; review pending)

---

## 👤 User Story Description

**As a** logistics operator
**I want to** evaluate if a single purchase order can be fulfilled by an available robot and assign the most suitable robot based on weight, volume constraints, and highest battery level
**So that** orders that fit in one robot are dispatched automatically, and only the orders that do not fit are sent to partitioning ([USEI05](../USEI05/readme.md))

---

## 🚦 Definition of Ready (DoR)

* [x] **Clear User Story:** Story follows the standard *As a / I want to / So that* format.
* [x] **Acceptance Criteria Defined:** Taken directly from the project requirements document.
* [x] **Dependencies Identified:** See "Found Out Dependencies" below.
* [x] **Inputs/Outputs Mapped:** See "Input & Output Data" below.
* [x] **Estimation Complete:** Story estimated by the team (5 story points, size M).
* [ ] **Open Clarifications Resolved:** "Minimum required battery" is still open (a placeholder is used, see below). The robot state name is resolved (`IN_USE`).

---

## 💬 Customer Specifications & Clarifications

* **Requirements doc (ESINF, Sprint 1):** the use of `PriorityQueue` (or any other heap-based structure) is strictly prohibited. Sorting and priorities must use standard collections combined with `Comparator`, or explicit sorting through JCF utilities.
* **Requirements doc (ESINF, Sprint 1):** data may be processed only in memory (database integration is not mandatory in this sprint), but the classes must be designed with future database integration in mind.
* **Requirements doc (ESINF, Sprint 1):** a report is required, with the principal operations and data structures, justification of the JCF choices and temporal complexity (parameters R, S, O, I, Io, K), supported by an empirical study with the medium and large datasets.
* **Open question:** the requirements do not define how the *minimum required battery* is determined. To be clarified with the teacher/client (question to be posted on the Moodle forum).
* **Assumption (minimum battery):** until clarified, a fixed and configurable threshold is used (placeholder `MIN_BATTERY = 20`, to be agreed by the team). Route-based estimation is out of scope for Sprint 1.
* **Assumption (calculation):** order total weight = sum of (quantity × product weight); total volume = sum of (quantity × product volume), without considering packing. An order is *refrigerated* if all its lines require refrigeration, *ordinary* if none of them does, and *mixed* otherwise.
* **Assumption (assignment record):** when a robot is assigned, the order is linked to that robot and its status changes from `PENDING` to `ASSIGNED` (as in the domain model), otherwise the assignment would be lost.
* **Decision (robot state):** the story text says the robot state changes from `READY` to `BUSY`, but the glossary and domain model use `IN_USE`. The team uses `IN_USE`.
* **Client sessions (refrigeration):** refrigerated robots carry only refrigerated products and non-refrigerated robots carry only non-refrigerated products; there are no mixed robots. Therefore, in this story a refrigerated order is eligible only for refrigerated robots, an ordinary order only for non-refrigerated robots, and a mixed order is not feasible for a single robot (it is sent to [USEI05](../USEI05/readme.md)).
* **Decision (battery tie):** when several eligible robots have the same battery level, the tie is broken deterministically by the lowest robot id, so that results are reproducible.
* **Decision (best robot):** the eligible robots are collected in a list and the best one is picked with `Collections.min` and a `Comparator` (highest battery, then lowest id). No sorting and no heap-based structure is needed, which keeps the selection linear.
* **Decision (module):** the Planning App is hosted in `app-two`.
* **Out of Scope:** splitting an order across several robots ([USEI05](../USEI05/readme.md)), route planning, estimating battery consumption of a trip.

---

## ✅ Acceptance Criteria

* [x] **AC1 - Feasibility Result:** A boolean result indicates whether any available robot satisfies all constraints (max volume, max weight and minimum required battery). *(The minimum battery uses the placeholder `MIN_BATTERY = 20` until the rule is confirmed.)*
* [x] **AC2 - Optimal Assignment:** The order is automatically assigned to the optimal robot (`READY` status, sufficient capacity and highest current battery).
* [x] **AC3 - State Update:** The status of the selected robot is updated (from `READY` to `IN_USE`) and the station availability is updated.
* [x] **AC4 - Assignment Record (derived from the domain model):** The order is linked to the selected robot and its status changes from `PENDING` to `ASSIGNED`.

---

## 🔒 Non-Functional Requirements & Security

* **NFR01 (Business Rules):** order and robot data must be validated (e.g., non-negative weights/volumes, battery between 0 and 100).
* **NFR04 (OOP):** the design should follow OOP best practices, allowing easy maintenance and new functionalities.
* **NFR05 (Documentation):** Javadoc on public classes/methods, camelCase coding standard, diagrams in SVG.
* **Sprint 1 ESINF restriction:** no `PriorityQueue` or other heap-based structure.
* **Process:** TDD (Red-Green-Refactor locally), at least one commit per day.
* **Performance:** report must derive the time complexity and compare it with measurements on the medium and large datasets.
* **Security:** not applicable (no authentication or sensitive data involved).

---

## 🔗 Found Out Dependencies

### Upstream Dependencies (Required before starting)
* **[USEI01](../USEI01/readme.md) Data Import and System Initialization:** provides the robots, stations and orders loaded into the JCF structures. Until it is ready, development can use hand-made test data.
* **Domain Model ([USBD03](../../BDDAD/USBD03/readme.md)/[USLP01](../../LAPR3/USLP01/readme.md)):** `Robot`, `RobotState`, `RobotStation`, `Order`, `OrderLine`, `Product`.

### Downstream Dependencies (Impacted by this story)
* **[USEI05](../USEI05/readme.md) Complex Order Partitioning:** applies when this story finds no single robot able to fulfil the order.
* **[USEI02](../USEI02/readme.md) Parking Station Monitoring:** displays the robot states and station occupancy that this story updates.
* **[USEI03](../USEI03/readme.md) Order Queue Management:** expected to provide the pending orders to be assigned.

---

## 📥 Input & Output Data

### Input Data
| Data            | Type                          | Description                                                                                              |
|:----------------|:------------------------------|:---------------------------------------------------------------------------------------------------------|
| Order           | `Order` (list of `OrderLine`) | Order to be fulfilled; each line has a `Product` (weight, volume, requires refrigeration) and a quantity |
| Robots          | Collection of `Robot`         | Fleet loaded by USEI01; each has state, max weight, max volume, refrigerated flag and battery level      |
| Minimum battery | `int` constant                | `MIN_BATTERY` (placeholder `20`); the real rule is still open                                            |

### Output Data
| Data                 | Type              | Description                                                                 |
|:---------------------|:------------------|:----------------------------------------------------------------------------|
| Feasibility          | `boolean`         | `true` if at least one robot satisfies all constraints                      |
| Assigned robot       | `Optional<Robot>` | The optimal robot, or empty if none is eligible                             |
| Robot state          | Side effect       | Selected robot changes from `READY` to `IN_USE`                             |
| Station availability | Side effect       | Free spots of the station of the selected robot are updated                 |
| Order                | Side effect       | The order is linked to the selected robot and its status becomes `ASSIGNED` |

---

## Edge Cases

* **Empty fleet:** no robots loaded, so the result is "not feasible" (no exception).
* **No robot in `READY` state:** all are charging, in maintenance or in use, so the result is "not feasible".
* **Order larger than every robot:** no single robot fits it, so the result is "not feasible" (the order goes to [USEI05](../USEI05/readme.md)).
* **Order exactly at a robot's limit:** an order whose total weight and volume are equal to the robot's `maxWeight` and `maxVolume` is eligible (order weight ≤ `maxWeight` and order volume ≤ `maxVolume`).
* **Invalid data:** negative weights or volumes, or a battery level outside 0–100, are rejected with a clear error message (NFR01).
* **Refrigerated order, no refrigerated robot available:** "not feasible".
* **Ordinary order, refrigerated robots available:** refrigerated robots are not eligible, even if they have more battery than the non-refrigerated ones (client rule above).
* **Mixed order (refrigerated and ordinary items):** "not feasible" for a single robot, even if both types of robot are available; the order goes to [USEI05](../USEI05/readme.md). The fleet is not even scanned.
* **Battery tie between eligible robots:** broken by the lowest robot id (decision above).
* **Empty or invalid order:** rejected with a clear error message.
* **Not feasible result:** no robot or station state may be changed.

---

## 🧪 Functional Tests / Validation

| Scenario ID | Scenario                                                                    | Expected Result                                           |
|:------------|:----------------------------------------------------------------------------|:----------------------------------------------------------|
| **F-01**    | Only one robot is eligible                                                  | That robot is assigned                                    |
| **F-02**    | Several robots are eligible                                                 | The one with the highest battery is assigned              |
| **F-03**    | Robots in `CHARGING`, `MAINTENANCE` or in use                               | They are ignored                                          |
| **F-04**    | Order heavier or bigger than every robot                                    | Not feasible, nothing changes                             |
| **F-05**    | Refrigerated item and only non-refrigerated robots                          | Not feasible                                              |
| **F-06**    | Robot assigned                                                              | Its state changes and the station availability is updated |
| **F-07**    | `isFeasible` with an eligible robot / without one                           | `true` / `false`, and no state changes in both cases      |
| **F-08**    | Robot assigned                                                              | Order is linked to the robot and its status is `ASSIGNED` |
| **F-09**    | Ordinary items; a refrigerated robot with more battery and a non-refrigerated one | The non-refrigerated robot is assigned              |
| **F-10**    | Refrigerated and ordinary items in the same order; both types of robot available | Not feasible, nothing changes                       |

---

# 🧪 QA & Test Documentation

See Report at [QA & Test Documentation: USEI04 Feasibility Check and Single-Robot Assignment](qa-readme.md)

---

## 🧩 Related Modules & Classes

* **Module:** `app-two` (Planning App), using `shared-library` where needed.
* **Packages:** `com.example.apptwo.domain` and `com.example.apptwo.service`.
* **Classes:** `Robot`, `RobotState`, `RobotStation`, `Order`, `OrderLine`, `OrderStatus`, `Product`, `RobotAssignmentService`.
* **Tests:** `RobotAssignmentServiceTest` (19 JUnit tests, see [qa-readme.md](qa-readme.md)).
* **Performance study:** `RobotAssignmentBenchmark` (`com.example.apptwo.benchmark`, in the test sources; run by hand, not by Maven).

---

## ⏱️ Complexity

Parameters (as in the ESINF report): **R** robots, **S** stations, **Io** lines of the order, **E** eligible robots (E ≤ R).

| Step                                                    | Cost            | Explanation                                                                                                                           |
|:--------------------------------------------------------|:----------------|:--------------------------------------------------------------------------------------------------------------------------------------|
| Validate the order, totals and mixed check              | O(Io)           | A constant number of passes over the order lines (weight, volume, refrigeration, mixed check)                                         |
| Scan the fleet                                          | O(R)            | One pass; each robot is checked in O(1) and the eligible ones are added to a list                                                     |
| Pick the best robot                                     | O(E) ⊆ O(R)    | `Collections.min` with a `Comparator` over the eligible robots (one pass, no sorting)                                                 |
| **`isFeasible`**                                        | **O(Io + R)**   | Sum of the steps above. A mixed order ends after O(Io), because the fleet is not scanned                                              |
| Find the station and remove the robot (assignment only) | O(S + R)        | The stations are scanned until the one that parks the robot is found (at most R parked robots in total); removal from the station list is bounded by R |
| **`assignBestRobot`**                                   | **O(Io + R + S)** | `isFeasible` plus the update of the robot, the station and the order                                                                |

Extra memory: O(E) for the list of eligible robots.

### Empirical study (`RobotAssignmentBenchmark`)

Median time of one pass over all the orders of the `clean` datasets (one run, 8 processors, JDK 21):

| Dataset | R  | S  | Orders | `isFeasible` (µs) | `assignBestRobot` (µs) |
|:--------|---:|---:|-------:|------------------:|-----------------------:|
| Medium  | 30 | 6  | 200    | 26.8              | 38.4                   |
| Large   | 72 | 12 | 600    | 96.9              | 137.8                  |

Repeating the fleet and stations k times with the same orders, the time ratio tends to 2 when R doubles; multiplying Io by 10 makes the time ratio tend to 10. Both are consistent with the linear growth derived above. The full analysis is in the ESINF report.

---

## 🔄 Sequence Diagram

![Sequence Diagram](sd.svg)

## 🏁 Definition of Done (DoD)

* [x] All acceptance criteria verified.
* [x] Unit tests written (TDD) and passing.
* [ ] Reviewed by a team member other than the author.
* [x] Javadoc written for public classes and methods.
* [x] Complexity analysis and empirical study included in the ESINF report.
* [ ] Final sign-off recorded in `qa-readme.md`.