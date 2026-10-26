# [USEI05] Complex Order Partitioning Across Multiple Robots

* **Priority:** TBD
* **Story Points:** 8 (size L, as estimated on the project board)
* **Module / Component:** `Planning App` (`app-two`) / `Order Assignment`
* **Status:** In Review (implemented, 23 JUnit tests passing; grouping limitation open, ESINF report pending)

---

## 👤 User Story Description

**As a** logistics operator
**I want to** split oversized or incompatible purchase orders into sub-orders so that they can be dispatched across multiple robots
**So that** orders that no single robot can fulfil ([USEI04](../USEI04/readme.md)) are still delivered, each part by a distinct available robot

---

## 🚦 Definition of Ready (DoR)

* [x] **Clear User Story:** Story follows the standard *As a / I want to / So that* format.
* [x] **Acceptance Criteria Defined:** Taken directly from the project requirements document.
* [x] **Dependencies Identified:** See "Found Out Dependencies" below.
* [x] **Inputs/Outputs Mapped:** See "Input & Output Data" below.
* [x] **Estimation Complete:** 8 story points (size L), as estimated on the project board.
* [ ] **Open Clarifications Resolved:** "Minimum required battery" is still open (the placeholder `MIN_BATTERY = 20` is used, shared with USEI04). The robot state name (`IN_USE`) and the refrigeration rule are resolved (see below).

---

## 💬 Customer Specifications & Clarifications

* **Requirements doc (ESINF, Sprint 1):** the use of `PriorityQueue` (or any other heap-based structure) is strictly prohibited. Sorting and priorities must use standard collections combined with `Comparator`, or explicit sorting through JCF utilities.
* **Requirements doc (ESINF, Sprint 1):** data may be processed only in memory (database integration is not mandatory in this sprint), but the classes must be designed with future database integration in mind.
* **Requirements doc (ESINF, Sprint 1):** a report is required, with the principal operations and data structures, justification of the JCF choices and temporal complexity (parameters R, S, O, I, Io, K), supported by an empirical study with the medium and large datasets. `K` (number of robots assigned to a partitioned order) is the parameter introduced specifically for this story.
* **Open question:** the requirements do not define how the *minimum required battery* is determined. To be clarified with the teacher/client (question to be posted on the Moodle forum).
* **Assumption (minimum battery):** same as [USEI04](../USEI04/readme.md): a fixed threshold is used (placeholder `MIN_BATTERY = 20`, the constant defined in `RobotAssignmentService` and reused by `OrderPartitioningService`).
* **Decision (robot state):** the requirements text says `BUSY`, but the glossary and domain model use `IN_USE`. The team uses `IN_USE`.
* **Decision (refrigeration, as clarified by the client):** refrigerated robots carry only refrigerated products and non-refrigerated robots carry only ordinary products; there is no mixed robot. A sub-order therefore never mixes both kinds, and its robot must have the same refrigeration capability.
* **Assumption (calculation):** sub-order total weight = sum of (quantity × product weight); total volume = sum of (quantity × product volume), without considering packing; refrigeration is needed if any item of the (sub-)order requires it.
* **Assumption (splitting granularity):** the unit is the indivisible item. A line with quantity N is expanded into N units. If a single unit exceeds the capacity of every eligible robot, no partition exists and the operation fails. To be confirmed with the teacher/client.
* **Implemented behaviour (partitioning):** every unit becomes its own sub-order (`<orderId>-part-<n>`), so for a split order K equals the number of units. Units are **not packed together** (see Known Limitations, D-01). The first-fit-decreasing packing that was initially planned is not implemented.
* **Implemented behaviour (matching):** every sub-order is linked to a **distinct** robot. The service searches a complete matching with backtracking over the eligible robots (`READY`, battery at least `MIN_BATTERY`, refrigeration equal, enough capacity), trying robots with higher battery first (tie: lowest id). The search does not fail just because a greedy choice used up a robot needed by a later sub-order.
* **Implemented behaviour (all-or-nothing):** no robot, station, order or sub-order state is changed until the partition and the matching are both complete and valid. Otherwise an `IllegalStateException` with a clear reason is thrown and nothing changes.
* **Implemented behaviour (assignment record):** each sub-order is linked to its robot (status `ASSIGNED`) and the robot becomes `IN_USE` and leaves its station. The original order **stays `PENDING`** when it is split, because `OrderStatus` has no `PARTITIONED` value and partitioning is an internal dispatch operation.
* **Implemented behaviour (tie-break):** when several robots have the same battery level, the lowest robot id wins, so results are reproducible.
* **Implemented behaviour (single-robot shortcut):** if the order is not mixed and fits one eligible robot, that robot is assigned directly (the order becomes `ASSIGNED`) and no sub-orders are created. The service applies the same rules as USEI04 itself; it does not call `RobotAssignmentService`, it only reuses `MIN_BATTERY`.
* **Out of Scope:** route planning, estimating battery consumption of a trip, order queue management ([USEI03](../USEI03/readme.md)), finding the provably minimum number of robots.

---

## ✅ Acceptance Criteria

* [ ] **AC1 - Partitioning:** The order is algorithmically split into sub-orders where items are grouped legally by compatibility (e.g., refrigerated vs. non-refrigerated) and respect weight and volume constraints. *(Partially met: no sub-order mixes refrigerated and ordinary items and every sub-order respects the limits of its robot, but units are not grouped, there is one sub-order per unit. See D-01.)*
* [x] **AC2 - Multi-Robot Assignment:** Each sub-order is linked to a distinct available robot.
* [x] **AC3 - Structured Summary:** A `PartitionResult` holds the original order, the generated sub-orders (items, weight, volume) and their respective assigned robots.
* [x] **AC4 - State Update:** Every selected robot changes from `READY` to `IN_USE`, the station availability is updated and each sub-order is linked to its robot (`ASSIGNED`). The original order stays `PENDING` when it is split.
* [x] **AC5 - Integrity:** Every unit of the original order belongs to exactly one sub-order, and if the partition or the assignment is not possible nothing is changed.

---

## 🔒 Non-Functional Requirements & Security

* **NFR01 (Business Rules):** order and robot data must be validated (e.g., non-negative weights/volumes, battery between 0 and 100). The order is validated by the service (`IllegalArgumentException`); the battery range is validated by the `Robot` constructor.
* **NFR04 (OOP):** the design should follow OOP best practices, allowing easy maintenance and new functionalities.
* **NFR05 (Documentation):** Javadoc on public classes/methods, camelCase coding standard, diagrams in SVG.
* **Sprint 1 ESINF restriction:** no `PriorityQueue` or other heap-based structure.
* **Process:** TDD (Red-Green-Refactor locally), at least one commit per day.
* **Performance:** report must derive the time complexity (in Io, R and K) and compare it with measurements on the medium and large datasets.
* **Security:** not applicable (no authentication or sensitive data involved).

---

## 🔗 Found Out Dependencies

### Upstream Dependencies (Required before starting)
* **[USEI01](../USEI01/readme.md) Data Import and System Initialization:** provides the robots, stations and orders loaded into the JCF structures. Until it is ready, development can use hand-made test data.
* **[USEI04](../USEI04/readme.md) Feasibility Check and Single-Robot Assignment:** defines the eligibility rules (state, battery, refrigeration, capacity) and the `MIN_BATTERY` constant, which this story reuses.
* **Domain Model ([USBD03](../../BDDAD/USBD03/readme.md)/[USLP01](../../LAPR3/USLP01/readme.md)):** `Robot`, `RobotState`, `RobotStation`, `Order`, `OrderLine`, `SubOrder`, `Product`.

### Downstream Dependencies (Impacted by this story)
* **[USEI02](../USEI02/readme.md) Parking Station Monitoring:** displays the robot states and station occupancy that this story updates.
* **[USEI03](../USEI03/readme.md) Order Queue Management:** expected to provide the pending orders.

---

## 📥 Input & Output Data

### Input Data
| Data            | Type                          | Description                                                                                              |
|:----------------|:------------------------------|:---------------------------------------------------------------------------------------------------------|
| Order           | `Order` (list of `OrderLine`) | Order to be fulfilled; each line has a `Product` (weight, volume, requires refrigeration) and a quantity |
| Robots          | `List<Robot>` (service field) | Fleet given to the service; each has state, max weight, max volume, refrigerated flag and battery level |
| Stations        | `List<RobotStation>`          | Stations from which the assigned robots are removed                                                      |
| Minimum battery | `int` constant                | `RobotAssignmentService.MIN_BATTERY` (placeholder `20`); the real rule is still open                     |

### Output Data
| Data                 | Type                           | Description                                                                                          |
|:---------------------|:-------------------------------|:-----------------------------------------------------------------------------------------------------|
| Partition result     | `PartitionResult`              | Original order, generated sub-orders (items, weight, volume) and the assigned robots                 |
| Failure reason       | `IllegalStateException`        | Why it was not possible (item fits no robot, no distinct robots available)                           |
| Invalid input        | `IllegalArgumentException`     | Null/empty order, negative quantities, weights or volumes, null robots in the fleet                  |
| Robot state          | Side effect                    | Each selected robot changes from `READY` to `IN_USE`                                                 |
| Station availability | Side effect                    | Each selected robot is removed from its station                                                      |
| Sub-orders           | Side effect                    | Each sub-order is linked to its robot with status `ASSIGNED`                                         |
| Order                | Side effect                    | Split order: stays `PENDING`. Single-robot shortcut: becomes `ASSIGNED`                              |

---

## Edge Cases

* **Order fits in one robot:** handled as a single-robot assignment; no sub-orders are generated.
* **Empty fleet or no robot in `READY` state:** fails with a clear reason; nothing changes.
* **A single unit exceeds every eligible robot's capacity:** no valid partition exists, so the operation fails and nothing changes.
* **Refrigerated items, no refrigerated robot available (or the opposite):** the sub-order cannot be assigned, so the whole operation fails and nothing changes.
* **Fewer eligible robots than sub-orders:** no complete distinct-robot matching, so the operation fails and nothing changes (no partial assignment). Because there is one sub-order per unit, this happens more often than with a packed partition (D-01).
* **Sub-order exactly at a robot's limit:** weight and volume equal to `maxWeight` and `maxVolume` is eligible (≤).
* **Battery tie between candidate robots:** the lowest robot id wins.
* **Matching requires a non-greedy choice:** the backtracking search undoes a choice that leaves a later sub-order without a robot.
* **Robot below minimum battery:** ignored as a candidate.
* **Invalid data:** negative weights, volumes or quantities are rejected with `IllegalArgumentException`; a battery outside 0–100 is rejected by the `Robot` constructor.
* **Empty or invalid order (`null`, no lines, or no unit at all):** rejected with `IllegalArgumentException`.

---

## 🧪 Functional Tests / Validation

| Scenario ID | Scenario                                                  | Expected Result                                                                 |
|:------------|:----------------------------------------------------------|:--------------------------------------------------------------------------------|
| **F-01**    | Order heavier or bigger than every robot, but splittable  | Sub-orders within weight/volume limits, each with a distinct robot              |
| **F-02**    | Order with refrigerated and non-refrigerated items        | Items separated into compatible sub-orders; refrigerated ones get refrigerated robots |
| **F-03**    | Every item of the original order                          | Belongs to exactly one sub-order (quantities add up to the original)            |
| **F-04**    | Fewer eligible robots than sub-orders                     | Operation fails with a reason; nothing changes                                  |
| **F-05**    | A single item bigger than every robot                     | Operation fails with a reason; nothing changes                                  |
| **F-06**    | Partition and assignment succeed                          | Robot states and station availability updated; sub-orders `ASSIGNED`; order stays `PENDING` |
| **F-07**    | Partition and assignment succeed                          | `PartitionResult` shows original order, sub-orders and assigned robots          |
| **F-08**    | Order fits in a single robot                              | Single-robot assignment, no sub-orders                                          |

---

# 🧪 QA & Test Documentation

See Report at [QA & Test Documentation: USEI05 Complex Order Partitioning Across Multiple Robots](qa-readme.md)

---

## 🧩 Related Modules & Classes

* **Module:** `app-two` (Planning App), packages `com.example.apptwo.domain` and `com.example.apptwo.service`.
* **Classes:** `OrderPartitioningService` (the participant of the sequence diagram), `SubOrder`, `PartitionResult`, `Order`, `OrderLine`, `Product`, `Robot`, `RobotState`, `RobotStation`, and `RobotAssignmentService.MIN_BATTERY` (reused from USEI04).
* **Tests:** `OrderPartitioningServiceTest` (23 JUnit tests).

---

## ⏱️ Expected Complexity

Parameters: Io is the number of **units** of the order (the quantity of each line counts, because each line is expanded into units), K is the number of sub-orders (K = Io when the order is split), E is the number of eligible robots (E ≤ R) and S the number of stations.

| Step                                      | Worst case                         |
|:------------------------------------------|:-----------------------------------|
| Validate order and fleet                  | O(Io + R)                          |
| Filter and sort the eligible robots       | O(R + E log E)                     |
| Single-robot shortcut                     | O(E · Io)                          |
| Expand lines into units                   | O(Io)                              |
| Check that each unit fits some robot      | O(Io · E)                          |
| Create the sub-orders                     | O(K)                               |
| Matching, when no backtracking is needed  | O(K · E · (log E + K))             |
| Matching, when it fails                   | exponential in K (up to E!/(E − K)! partial assignments) |
| Update robots and stations                | O(K · (S + R))                     |

The matching copies and sorts the candidates at every level and checks the used robots with `List.contains`, which explains the `log E + K` factor. Total when the order is split and the matching succeeds without backtracking: O(Io · R · (log R + Io) + Io · S). When the order fits one robot: O(R log R + Io · R). When a unit fits no robot: O(R log R + Io · R). Auxiliary space: O(E + K). To be confirmed with the empirical study in the ESINF report.

---

## ⚠️ Known Limitations

* **D-01 - No grouping of units:** each unit becomes its own sub-order instead of being packed with other units of the same compatibility group (first-fit-decreasing, as planned). The result uses more robots than necessary and can fail where a packed partition would succeed (for example, 3 units of 4 kg with 2 robots of 10 kg capacity). In the medium dataset the order `O00199` fails for this reason.
* **D-02 - Cost of a failing matching:** when no complete matching exists, the backtracking explores many partial assignments (exponential in K).
* **Minimum battery:** placeholder `MIN_BATTERY = 20` (rule open, see above).
* **Single-robot shortcut:** reimplemented in the service instead of calling `RobotAssignmentService` (the rules are the same).

The defects are tracked in the [qa-readme](qa-readme.md).

---

## 🔄 Sequence Diagram

![Sequence Diagram](sd.svg)

## 🏁 Definition of Done (DoD)

* [ ] All acceptance criteria verified. *(AC1 is partially met, see D-01.)*
* [x] Unit tests written (TDD) and passing (23 tests).
* [x] Reviewed by a team member other than the author.
* [ ] Javadoc written for public classes and methods. *(Missing on the `PartitionResult` constructor and getters.)*
* [ ] Complexity analysis and empirical study included in the ESINF report.
* [ ] Final sign-off recorded in `qa-readme.md`.