# [USEI04] Single-Robot Feasibility and Allocation

* **Issue:** [#10](https://github.com/Departamento-de-Engenharia-Informatica/SEM3PI-26-27-2DE02/issues/10)
* **Priority:** Urgent
* **Story Points:** 5
* **Module / Component:** ESINF / Planning area of the ShopRobot application (menu *2 - Planning → 3 - Single-robot assignment*)
* **Status:** Testing
* **Developer:** Francisco Marques (1212081)
* **QA:** To be assigned

---

## 👤 User Story Description

**As a** logistics operator  
**I want to** evaluate if a single purchase order can be fulfilled by an available robot and assign the most suitable robot based on weight, volume constraints, and highest battery level  
**So that** each order that fits in one robot is served by the robot best prepared for it, and the orders that do not fit can be split (USEI05)

---

## 🚦 Definition of Ready (DoR)

* [x] **Clear User Story:** Story follows the standard *As a / I want to / So that* format (Issue #10).
* [x] **Acceptance Criteria Defined:** ACs taken from GitHub Issue #10 and Project Specification v1 (Section 7.2.1).
* [x] **Dependencies Identified:** robots, stations and orders imported by USEI01; station status of USEI02; order queue of USEI03.
* [x] **Inputs/Outputs Mapped:** Inputs and outputs documented below.
* [x] **Estimation Complete:** 5 story points.
* [x] **Small & Actionable:** In-memory feature of the Planning area, completed within Sprint 1.

---

## 💬 Customer Specifications & Clarifications

* **Project statement (v1, Section 7.2.1):**
  * The feasibility result is a boolean: whether any available robot satisfies all the constraints (max volume, max weight and minimum required battery).
  * The order is assigned automatically to the optimal robot: `READY`, enough capacity and the highest current battery.
  * The selected robot changes from `READY` to `BUSY` and the station availability is updated.
  * `PriorityQueue` and any other heap-based structure are not allowed in Sprint 1.
* **Clarification (2026-10-12, LAPR3 client session):** the robot with the highest battery is chosen; ties can be broken arbitrarily (we use the robot id); geographic proximity is out of the current scope. A `BUSY` state (working on an order) is allowed.
* **Clarification (2026-10-12):** refrigerated products are carried by refrigeration robots separately from normal items (do not mix).
* **Clarification (2026-10-16): strict segregation** — refrigerated robots exclusively carry refrigerated items and non-refrigerated robots carry non-refrigerated items; there are no mixed-cargo robots (see `doc/requirements/requirements.md`). So an order with only refrigerated items needs a refrigerated robot, an order with no refrigerated items needs a non-refrigerated robot, and an order that **mixes** both is never feasible for one robot: it must be split (USEI05).
* **Clarification (2026-10-16):** the minimum battery for the US4 feasibility check can be an input parameter or a global system property, e.g. a 20-25% threshold. We use **20%** (`SingleRobotAllocator.DEFAULT_MINIMUM_BATTERY`, the same value as query B12 of USBD05), passed to the allocator so it can be changed in one place.
* **Out of Scope:** splitting orders that do not fit in one robot (USEI05, Gonçalo Abreu); the robot coming back to its station; database persistence.

---

## ✅ Acceptance Criteria

* [ ] **AC1 - Feasibility result:** a boolean that says whether any available robot satisfies every constraint: `READY`, battery ≥ minimum, max weight ≥ total weight, max volume ≥ total volume, and a robot of the same type as the items (refrigerated robot for refrigerated items, non-refrigerated robot for the others; an order that mixes both is not feasible).
* [ ] **AC2 - Automatic assignment:** the order is assigned to the optimal robot: among the robots that satisfy every constraint, the one with the highest battery (same battery: lowest robot id).
* [ ] **AC3 - Updated status and station availability:** the selected robot changes from `READY` to `BUSY`, it no longer occupies a spot in its station (one more free spot in *Station monitoring*, USEI02), and the order leaves the pending queue (USEI03).

---

## 🔒 Non-Functional Requirements & Architecture

* **NFR04 - Object-Oriented Design (MVC):**
  * **Model:** `RobotFleet` (robots and the `READY` ones sorted by battery), `SingleRobotAllocator` (constraints, feasibility and assignment), `StationMonitor#update` (station availability), `Robot#withStatus`.
  * **Controller:** `SingleRobotAssignmentController` — connects the allocator with the order queue and the station monitor.
  * **View:** `SingleRobotAssignmentUI` — option *3* of the Planning menu.
* **NFR05 - Documentation:** Javadoc on all public classes and methods, including the time complexity of the operations.
* **ESINF Sprint 1 restriction:** no `PriorityQueue` or heap-based structures (a `TreeSet` with a `Comparator` is used).

---

## 🔗 Found Out Dependencies

### Upstream Dependencies (Required before starting)
* **[USEI01] CSV import:** `ImportResult` with the robots, stations and orders (with their items).
* **[USEI02] Station monitoring:** `RobotBatteryComparator` (highest battery first, then by id) and `StationMonitor`, which gets the method `update(Robot)` agreed in the USEI02 readme (QA-03 of USEI02).
* **[USEI03] Order queue:** the most urgent order (`peekNext`), `find` and `remove` of `OrderQueue`.

### Downstream Dependencies (Impacted by this story)
* **[USEI05] Multi-robot assignment:** the orders that are not feasible here must be split; it can reuse `RobotFleet` (ready robots by battery, `markBusy`) and `SingleRobotAllocator#canCarry` for each sub-order.

---

## 📥 Input & Output Data

### Inputs
| Field Name | Data Type | Required | Source | Example / Constraints |
| :--- | :--- | :--- | :--- | :--- |
| `robots` | `Map<String, Robot>` | Yes | USEI01 import (`robots.csv`) | `maxWeightKg`, `maxVolumeLitres`, `hasCooling`, `batteryPercentage`, `status`, `stationId` |
| `stations` | `Map<String, RobotStation>` | Yes | USEI01 import (`stations.csv`) | `maxCapacity` |
| Pending orders | `OrderQueue` | Yes | USEI03 (all imported orders) | items with `weightKg`, `volumeLitres`, `needsCooling` |
| `orderId` | `String` | No | User input | e.g. `O00156`; Enter = the most urgent pending order |
| Minimum battery | `int` | Yes | `SingleRobotAllocator.DEFAULT_MINIMUM_BATTERY` | 20 (%) |

### Outputs
| Field Name | Data Type | Destination | Example / Description |
| :--- | :--- | :--- | :--- |
| Order load | text | Planning text UI | `Order O00156 \| 2 items \| 18.00 kg \| 76.00 L \| refrigeration: no` |
| Feasibility (AC1) | `boolean` | Planning text UI | `Feasible: true` / `Feasible: false`, with the reason when it is not feasible (mixed order, or no robot of the right type) |
| Assigned robot (AC2) | `Robot` | Planning text UI | `Assigned to robot R0018 (battery 100%, max 40.0 kg / 140.0 L, refrigeration: no)` |
| New status and station availability (AC3) | text | Planning text UI | `Robot R0018: READY -> BUSY`, `Station S001 (Entrada Norte): capacity 10 \| occupied 3 \| free 7` |
| Order processed | text | Planning text UI | `Order O00156 removed from the pending orders.` |

---

## 🧠 Design

### Business rules

* A robot **can carry** an order when it is `READY`, its battery is at least the minimum (20%), its max weight is not lower than the total weight of the order, its max volume is not lower than the total volume, and it is of the same type as the items: a refrigerated robot for an order with only refrigerated items, a non-refrigerated robot for an order with no refrigerated items (strict segregation, 2026-10-16). Values equal to the maximum are accepted.
* An order that **mixes** refrigerated and non-refrigerated items needs two kinds of robots, so it is never feasible for one robot.
* The order is **feasible** (AC1) if at least one robot can carry it.
* The **most suitable robot** (AC2) is, among the robots that can carry the order, the one with the highest battery; with the same battery, the lowest robot id ([Battery-Based Priority](../../Glossaries/glossary/readme.md#battery-based-priority)).
* After the **assignment** (AC3), the robot is `BUSY`: it is no longer available for other orders and it no longer occupies a spot in its station. The order is processed, so it leaves the pending queue.
* If the order is **not feasible**, nothing changes: the order stays pending and must be split into sub-orders (USEI05).

### Data structures (JCF)

| Structure | Use | Justification |
| :--- | :--- | :--- |
| `NavigableSet<Robot>` (`TreeSet` with `RobotBatteryComparator`) | `READY` robots, highest battery first | the search visits the robots from the highest battery down, so the first robot that can carry the order is the optimal one, and it stops as soon as the battery is below the minimum; removing the assigned robot is O(log R); no heap |
| `Map<String, Robot>` (`HashMap`) | every robot by id, with its current status | O(1) to find a robot and to replace it by its `BUSY` copy (`Robot` is an immutable record) |
| `Map<String, List<Robot>>` of `StationMonitor` (USEI02) | parked robots by station | `update` removes the busy robot from the list of its station in O(k) |

### Main operations and time complexity

R = number of robots, Io = number of items of the order, j = number of `READY` robots visited by the search (j ≤ R), k = robots parked at the station of the assigned robot, S = number of stations.

| Operation | Complexity |
| :--- | :--- |
| Build the fleet | O(R log R) |
| Feasibility (`isFeasible`) / most suitable robot (`findBestRobot`) | O(Io + j), O(Io + R) in the worst case |
| Assignment (`assign`) | O(Io + j + log R + k) |
| Station availability shown after the assignment | O(S log S) |

The justification and the empirical study (medium and large datasets) are in the [ESINF report](../../ESINF/report.md#usei04--feasibility-check-and-single-robot-assignment).

### How to run

1. Run `com.example.shoprobot.Main` (from the repository root or from `JavaApp`).
2. Choose **2 - Planning** and then **3 - Single-robot assignment**.
3. Type the id of a pending order, or press Enter for the most urgent pending order.

The data is imported from `data/medium/clean` (see [data/README.md](../../../data/README.md)). Another folder can be used with the environment variable `SHOPROBOT_DATA_DIR`.

### Edge cases

* **Order not pending** (unknown or already assigned): `Order … is not pending.`; nothing changes.
* **No pending orders** and Enter: `No pending orders.`
* **Mixed order** (refrigerated and non-refrigerated items): `Feasible: false`, `The order has refrigerated and non-refrigerated items, which go on different robots; it must be split into sub-orders (USEI05).`
* **Not feasible** (too heavy, too big, or no `READY` robot of the right type with enough battery): `Feasible: false`, `No READY robot of the right type …`, and the order stays pending.
* **Non-refrigerated order and only refrigerated robots available:** not feasible (a refrigerated robot never carries non-refrigerated items).
* **Weight or volume equal to the maximum of the robot:** accepted.
* **Battery equal to the minimum (20%):** accepted.
* **Robots with the same battery:** the lowest robot id is chosen.
* **All the robots that could carry the order are already `BUSY`:** `Feasible: false`.
* **End of input:** back to the Planning menu.

---

## 🧪 Functional Tests / Validation

With `data/medium/clean` (16 `READY` robots, 4 of them refrigerated, all with at least 50% battery; 131 of the 200 orders mix refrigerated and non-refrigerated items; 59 orders are feasible for one robot). Do the steps in this order, in the same run:

| Scenario ID | Test Case / Steps | Expected Result |
| :--- | :--- | :--- |
| **TC-01** | Open *Single-robot assignment* and press Enter | Most urgent order `O00047` (3 items, 13.65 kg, 31.00 L, refrigeration: yes) mixes refrigerated and non-refrigerated items: `Feasible: false`, `The order has refrigerated and non-refrigerated items, which go on different robots; it must be split into sub-orders (USEI05).`; the order stays pending |
| **TC-02** | Type `O00156` | 18.00 kg, 76.00 L, refrigeration: no; `Feasible: true`; assigned to `R0018` (100%, non-refrigerated); `READY -> BUSY`; `S001 (Entrada Norte)`: occupied 3, free 7 |
| **TC-03** | Choose **1 - Station monitoring** | `S001 \| Entrada Norte \| capacity 10 \| occupied 3 \| free 7` (it was occupied 4, free 6) |
| **TC-04** | Type `O00091` | 1 item, 0.20 kg, only refrigerated: assigned to `R0028` (93%, the refrigerated robot with the highest battery; the non-refrigerated robots are not used); `S006`: occupied 5, free 7 |
| **TC-05** | Type `O00197` | 118.40 kg, no refrigeration; `Feasible: false`, `No READY robot of the right type …` (the largest robot carries 80 kg); the order stays pending |
| **TC-06** | Type `O00167` | 5.60 kg, 39.00 L, no refrigeration; assigned to `R0001` (95%; `R0018` is already `BUSY` and `R0028` is refrigerated) |
| **TC-07** | Type `O00156` again (already assigned) or `O99999` | `Order … is not pending.` |
| **TC-08** | Choose **2 - Order queue → 1** | `O00156`, `O00091` and `O00167` are no longer pending; `O00047` and `O00197` are still pending |
| **TC-09** | Battery below / equal to the minimum, ties, values equal to the maximum, robot types (covered by unit tests) | below: not used; equal: used; ties: lowest id; equal to the maximum: accepted; refrigerated order: only a refrigerated robot; non-refrigerated order: never a refrigerated robot; mixed order: never feasible |

**Unit tests:** `SingleRobotAllocatorTest`, `RobotFleetTest`, `StationMonitorTest` (update), `SingleRobotAssignmentControllerTest`, `SingleRobotAssignmentUITest`, `PlanningMenuTest` and `PlanningMainTest` (medium dataset).

---

# 🧪 QA & Test Documentation

To be completed by QA in `qa-readme.md`.

---

## 🧩 Related Modules & Classes

* **Model:** new `model/domain/RobotFleet` and `model/domain/SingleRobotAllocator`; `model/domain/Robot#withStatus` and `model/domain/StationMonitor#update` added; `RobotBatteryComparator`, `Order`, `OrderQueue` reused.
* **Controller:** new `controller/planning/SingleRobotAssignmentController`.
* **View:** new `view/planning/SingleRobotAssignmentUI`; option *3 - Single-robot assignment* added to `view/planning/PlanningMenu`.
* **Main:** the station monitoring, the order queue and the single-robot assignment now share the same `StationMonitor` and `OrderQueue`, so an assignment is seen in the other two options.
* **Benchmark:** `benchmark/SingleRobotAllocatorBenchmark` (test sources), for the empirical study.

---

## 🔄 Diagrams

### Class Diagram (CD)

![USEI04 Class Diagram](./cd.png)

*Source file:* [cd.puml](./cd.puml)

### Sequence Diagram (SD) — Feasibility check and assignment

![USEI04 Sequence Diagram](./sd.png)

*Source file:* [sd.puml](./sd.puml)

---

## 🏁 Definition of Done (DoD)

* [ ] Acceptance criteria verified by QA.
* [x] Unit tests written and passing (constraints, optimal robot, status and station update, edge cases).
* [x] Javadoc on all public classes and methods.
* [x] Complexity analysis and empirical study (medium and large datasets) included in the ESINF report.
* [x] Class and sequence diagrams in the repository (PlantUML + PNG).
