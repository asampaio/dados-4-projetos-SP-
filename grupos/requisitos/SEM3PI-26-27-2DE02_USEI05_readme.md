# [USEI05] Complex Order Partitioning Across Multiple Robots

* **Issue:** [#11](https://github.com/Departamento-de-Engenharia-Informatica/SEM3PI-26-27-2DE02/issues/11)
* **Priority:** High
* **Story Points:** 8
* **Module / Component:** ESINF / Planning area of the ShopRobot application (menu *2 - Planning → Complex order partitioning*)
* **Status:** Testing
* **Developer:** Gonçalo Abreu (GmpABR)
* **QA:** Francisco Marques (1212081)

---

## 👤 User Story Description

As a logistics operator, I want to split oversized or incompatible purchase orders into sub-orders so that they can be dispatched across multiple robots when a single robot cannot fulfill the request.

---

## 🚦 Definition of Ready (DoR)

* [x] **Clear User Story:** Story follows the standard *As a / I want to / So that* format (Issue #11).
* [x] **Acceptance Criteria Defined:** ACs taken from GitHub Issue #11 and Project Specification v1 (Section 7.2.1).
* [x] **Dependencies Identified:** Orders, items, robots, and stations imported by USEI01 (`CsvDataImporter`), and order queues managed by USEI03.
* [x] **Inputs/Outputs Mapped:** Inputs and outputs documented below.
* [x] **Estimation Complete:** 8 story points (Sprint 1 planning).
* [x] **Small & Actionable:** In-memory feature of the Planning area, completed within Sprint 1.

---

## 💬 Customer Specifications & Clarifications

* **Project statement (v1, Section 7.2.1):**
  * Choose the most appropriate JCF interfaces and classes for each entity/relation.
  * `PriorityQueue` (or any other heap-based structure) is **strictly prohibited** in Sprint 1; sorting and priorities must use standard collections with a `Comparator` or explicit sorting through JCF utilities.
  * Database integration is not mandatory in Sprint 1: data may be processed only in memory.
  * A report must identify the main operations and data structures, justify the JCF choices, and derive the time complexity in terms of input parameters ($R$: robots, $S$: stations, $O$: orders, $I$: total item lines, $I_o$: items in a particular order, $K$: robots assigned to a partitioned order), supported by an empirical study using the medium and large datasets.
* **Client Session (2026-10-12, LAPR3):**
  * Sub-orders of one order must be dispatched across **distinct** available robots (each assigned robot receives one sub-order).
  * Robot operational status: once assigned to a sub-order/order, the robot's status transitions to `BUSY`.
* **Domain / Business Constraints:**
  * **Refrigeration compatibility (strict segregation, client session of 2026-10-16):** items requiring cooling (`needsCooling == true`) are only placed into robots with cooling units (`hasCooling == true`), and non-refrigerated items only into robots without cooling units. Refrigerated robots never carry non-refrigerated items; there are no mixed-cargo robots.
  * **Capacity limits:** For each sub-order, total item weight $\le$ robot `maxWeightKg` and total item volume $\le$ robot `maxVolumeLitres`.
  * **Battery & status:** Robots considered for assignment must be in `READY` status with sufficient battery percentage. Preference/sorting is given to available robots with higher battery level.
* **Out of Scope for Sprint 1:** Autonomous path navigation, physical delivery timing simulation, and persistent DB transactions.

---

## ✅ Acceptance Criteria

* [x] Algorithmically partitioned sub-orders where items are grouped legally by compatibility (e.g., separating refrigerated items from non-refrigerated items if required) and weight/volume constraints.
* [x] Successful multi-robot assignment linking each sub-order to a distinct available robot.
* [x] A structured summary showing the original order, the generated sub-orders, and their respective assigned robots.

---

## 🔒 Non-Functional Requirements & Architecture

* **NFR01 - Business Rules:** Strictly enforce compatibility rules (cooling requirements) and capacity constraints (weight and volume).
* **NFR04 - Object-Oriented Design (MVC):**
  * **Model:** 
    * `SubOrder`: represents a partitioned slice of an `Order`, containing a subset of `OrderItem`, aggregate weight/volume, refrigeration requirement, and optional assigned `Robot`.
    * `OrderPartitioner` / `OrderPartitioningService`: encapsulates the bin packing / partitioning algorithm and multi-robot matching logic.
  * **Controller:** `OrderPartitioningController` in `controller/planning`, mediating between the UI and domain models.
  * **View:** `OrderPartitioningUI` in `view/planning`, accessible from `PlanningMenu`.
* **NFR05 - Documentation:** Javadoc on all public classes and methods; PlantUML sequence and class diagrams with SVG renderings.
* **ESINF Sprint 1 restriction:** No `PriorityQueue` or heap-based structures. Standard JCF collections (`ArrayList`, `HashMap`, `List.sort`) with `Comparator`.

---

## 🔗 Dependencies

### Upstream Dependencies
* **[USEI01] CSV Import:** Provides imported `Map<String, Robot>` and `Map<String, Order>` with `OrderItem` details.
* **[USEI02] Station & Robot Monitoring:** Provides `RobotBatteryComparator` and status tracking (`READY`, `BUSY`).
* **[USEI03] Order Queue:** Provides order management sorted by deadline.

### Downstream Dependencies
* **[USBD04 / USBD05]:** Relational schema constraints reflecting sub-order partition (`SUB_ORDER` table, `ux_sub_order_order_robot` constraint ensuring distinct robots per order).

---

## 📥 Input & Output Data

### Inputs
| Field Name | Data Type | Required | Source | Example / Constraints |
| :--- | :--- | :--- | :--- | :--- |
| `orderId` | `String` | Yes | Operator selection / Queue | e.g. `O0002` |
| `availableRobots` | `Collection<Robot>` | Yes | System state / Station monitor | Filtered by `status == READY` |
| `order` | `Order` | Yes | `OrderQueue` / Repository | Order containing list of `OrderItem` |

### Outputs
| Field Name | Data Type | Destination | Example / Description |
| :--- | :--- | :--- | :--- |
| `PartitionResult` | Object | Controller / UI | Success or failure (with the reason: order not pending, no items, an item that fits no robot of its type, or not enough suitable available robots), original order, and list of assigned `SubOrder`s |
| Sub-orders list | `List<SubOrder>` | Planning UI | Sub-order ID, items, total weight, total volume, cooling required, assigned robot |
| Updated robots | `Map<String, Robot>` | System state | Selected robots transitioned from `READY` to `BUSY` |

---

## 🧠 Design & Algorithm

### 1. Partitioning Strategy (First-Fit Decreasing / Bin Packing)
1. **Compatibility Grouping:** Separate items into two lists:
   - Refrigerated items (`needsCooling == true`)
   - Non-refrigerated items (`needsCooling == false`)
2. **Item Sorting:** Sort items within each category in descending order of volume/weight to optimize bin packing efficiency (First-Fit Decreasing).
3. **Sub-Order Bin Formation:** Group items into sub-orders such that no sub-order exceeds the capacity of the largest **available** robot of its type (`READY`, battery ≥ 20%; if none is available, the largest robot of that type in the fleet). Before partitioning, an item that no robot of its type in the fleet can carry makes the order fail with a message naming the item.
4. **Robot Matching:**
   - Candidates of each sub-order: available `READY` robots of the same type (`hasCooling == true` for refrigerated sub-orders, `hasCooling == false` for the others) with enough weight and volume, sorted by highest battery level (`RobotBatteryComparator`).
   - The sub-orders with fewer candidates are matched first, each to its unused candidate with the highest battery; if a sub-order has no candidate left, the previous choices are revised (backtracking), so an assignment is found whenever one exists.
   - Enforce distinct robot assignment: once a robot is assigned to a sub-order, it cannot be used for another sub-order of the same order.
5. **State Transition:** When all sub-orders are successfully matched, mark the assigned robots as `BUSY`.

### 2. Data Structures (JCF)

| Structure | Use | Justification |
| :--- | :--- | :--- |
| `List<OrderItem>` (`ArrayList`) | Holding items of each compatibility group | Sequential iteration and sorting by size/weight |
| `List<Robot>` (`ArrayList`) | Available robots and candidates of each sub-order | Read in battery order from the `TreeSet` of `RobotFleet` (`RobotBatteryComparator`), without the prohibited `PriorityQueue` |
| `Map<SubOrder, List<Robot>>` (`HashMap`) | Candidates of each sub-order | Computed once, used by the matching |
| `List<SubOrder>` (`ArrayList`) | Resulting sub-orders | Ordered representation of the partitions for display and tracking |
| `Set<String>` (`HashSet`) | Assigned robot IDs | $O(1)$ lookup ensuring distinct robots per order |

### 3. Time Complexity Analysis
* Let $I_o$ be the number of items in the order, and $R$ the total number of robots.
* Grouping items by refrigeration: $O(I_o)$.
* Sorting items for bin packing: $O(I_o \log I_o)$.
* Partitioning into $K$ sub-orders: $O(I_o \cdot K)$ where $K \le I_o$.
* Checking that every item fits some robot of its type in the fleet: $O(I_o \cdot R)$.
* Available robots, already sorted by battery in `RobotFleet`: $O(A)$, with $A \le R$ available robots.
* Matching $K$ sub-orders to distinct robots: $O(K \log K + K \cdot A)$ when no choice is revised; $O(A^K)$ in the worst case of the backtracking ($K \approx 2$ in the datasets).
* **Total Complexity:** $O(I_o \cdot R + I_o \log I_o + I_o \cdot K + K \cdot A)$ without backtracking, which operates efficiently well within real-time interactive thresholds.

---

## 🧪 Functional Tests / Validation

| Scenario ID | Test Case / Steps | Expected Result |
| :--- | :--- | :--- |
| **TC-01** | Order fits in a single robot without cooling | 1 sub-order created, assigned to best available standard robot |
| **TC-02** | Order contains both refrigerated and dry items | At least 2 sub-orders created; refrigerated sub-order assigned to cooling robot, dry items assigned to standard robot |
| **TC-03** | Order exceeds weight/volume of a single robot | Order is split into multiple sub-orders, each assigned to a distinct robot |
| **TC-04** | Not enough available robots to fulfill all sub-orders | Partition fails gracefully, reports insufficient robots, and robot states remain unchanged (atomic) |
| **TC-05** | Robot battery prioritization | Sub-orders receive robots with highest battery levels among valid candidates |
| **TC-06** | Non-refrigerated items and only refrigerated robots available | Not assigned: a refrigerated robot never carries non-refrigerated items (strict segregation) |
| **TC-07** | The largest robot is not available (e.g. `MAINTENANCE`) | Sub-orders are sized for the available robots and assigned to them |
| **TC-08** | Sub-orders that only fit if the robots are swapped (e.g. 50 kg / 100 L and 10 kg / 300 L; robots 80 kg / 320 L and 60 kg / 220 L) | Assigned: 50 kg / 100 L → 60 kg robot, 10 kg / 300 L → 80 kg robot |
| **TC-09** | An item that fits no robot of its type | `Item … does not fit any refrigerated / non-refrigerated robot of the fleet.`; nothing changes |

### How to run and values with `data/medium/clean`

Run `com.example.shoprobot.Main`, choose **2 - Planning** and then **4 - Complex order partitioning**; type an order id or press Enter for the most urgent order. Steps in this order, in the same run:

| Scenario ID | Test Case / Steps | Expected Result |
| :--- | :--- | :--- |
| **TC-10** | Press Enter (most urgent order `O00047`, 3 items, mixed) | `SUCCESS`: `O00047-S1` refrigerated (1 item, 1.05 kg) → `R0028` (93%, refrigerated); `O00047-S2` (2 items, 12.60 kg) → `R0018` (100%, non-refrigerated) |
| **TC-11** | Type `O00197` (118.40 kg, 626.00 L, no refrigeration) | `SUCCESS`: `O00197-S1` 58.40 kg / 310.00 L → `R0027`; `O00197-S2` 60.00 kg / 316.00 L → `R0003` |
| **TC-12** | Type `O00187` (refrigerated item of 95 kg) | `FAILED`: `Item Artigo sobredimensionado (95.00 kg, 180.00 L) does not fit any refrigerated robot of the fleet.`; the order stays pending |
| **TC-13** | Type `O00111` (6 items, mixed, 71.70 kg, 406.00 L) | `SUCCESS`: 3 sub-orders; refrigerated `O00111-S1` → `R0008`; `O00111-S2` (284.00 L, only fits the 80 kg / 320 L robots) → `R0023`; `O00111-S3` → `R0001` |
| **TC-14** | Type `O00047` again or `O99999` | `Order … is not pending in the queue.` |
| **TC-15** | Choose **1 - Station monitoring** | the stations of the assigned robots have one more free spot each (e.g. `S006`: occupied 5, free 7) |

**Unit tests:** `SubOrderTest`, `PartitionResultTest`, `OrderPartitioningServiceTest`, `OrderPartitioningControllerTest`, `OrderPartitioningUITest`, `PlanningMenuTest` and `PlanningMainTest`.

---

# 🧪 QA & Test Documentation

See Report at [QA & Test Documentation: USEI05 Complex Order Partitioning](./qa-readme.md)

---

## 🔄 Diagrams

### Sequence Diagram (SD)
![USEI05 Sequence Diagram](./sd.svg)
*Source file:* [sd.puml](./sd.puml)

### Class Diagram (CD)
![USEI05 Class Diagram](./cd.svg)
*Source file:* [cd.puml](./cd.puml)

---

## 🏁 Definition of Done (DoD)

* [x] SubOrder domain entity created.
* [x] Partitioning and multi-robot assignment service implemented using compliant JCF structures (no `PriorityQueue`).
* [x] Unit tests covering edge cases (mixed items, overweight orders, insufficient robots, distinct robot assignment).
* [x] Planning UI integration.
* [x] PlantUML diagrams (SD and CD) generated and committed.
* [x] ESINF report updated with theoretical and empirical complexity analysis.
