# [USEI01] Data Import and System Initialization

* **Priority:** High
* **Story Points:** TBD (to be estimated by the team)
* **Module / Component:** `Planning App` / `Data Import`
* **Course Unit / Sprint:** ESINF / Sprint 1
* **Status:** Draft

---

## 👤 User Story Description

**As an** infrastructure planner  
**I want to** import initial information from text/CSV files (robots, parking stations and user purchase requests) and load it into the most appropriate data structures  
**So that** the system is initialized and the following planning features (USEI02–USEI05) can operate on the data

---

## 🚦 Definition of Ready (DoR)

* [ ] **Clear User Story:** Story follows the standard *As a / I want to / So that* format.
* [ ] **Acceptance Criteria Defined:** Clear, testable ACs agreed upon by Product and Engineering.
* [ ] **Dependencies Identified:** Datasets (small/medium/large) and their file formats are available.
* [ ] **Inputs/Outputs Mapped:** Required data fields, payloads, and error states are documented.
* [ ] **Estimation Complete:** Story has been refined and estimated (story points assigned) by the dev team.
* [ ] **Small & Actionable:** Story is scoped down enough to be completed within a single sprint.

---

## 💬 Customer Specifications & Clarifications

* **Source (Project Description, §7.2.1):** Data is imported from text/CSV files containing robots, parking stations and purchase orders.
* **Source (Project Description, §7.2.1):** Data may be processed only in memory in Sprint 1, but classes must be designed considering full database integration in the future.
* **Source (Project Description, §7.2.1):** `PriorityQueue` (or any other heap-based structure) is **strictly prohibited** in this sprint. Use standard collections with `Comparator` or explicit sorting through JCF utilities.
* **Clarification:** *(add here the clarifications obtained from teachers, with date, e.g. exact CSV headers, delimiter, date format, handling of duplicates)*
* **Out of Scope:** Station monitoring ([USEI02]), order queue ([USEI03]), robot assignment ([USEI04]) and order partitioning ([USEI05]).

---

## ✅ Acceptance Criteria

* [ ] **AC1 - Complete Loading:** All entities (Robots, Stations, Orders) are successfully parsed and loaded into Java Collection Framework structures, without data loss or corruption.
* [ ] **AC2 - Appropriate Collections:** Collections are chosen according to the access patterns of the system and the choice is justified in the report.
* [ ] **AC3 - Error Reporting:** Informative error messages or logs are produced for malformed lines or missing values during import, and the import continues with the remaining valid lines.

---

## 🔒 Non-Functional Requirements & Security

* **Performance:** Import must scale linearly with the input size; empirical study with the medium and large datasets is required.![img.png](img.png)
* **Complexity Analysis:** Theoretical complexity expressed in terms of R (robots), S (stations), O (orders), I (total order-item lines), Io (items in an order).
* **Restriction:** No `PriorityQueue` or heap-based structure in this sprint.
* **Object-Oriented Design (NFR04):** Domain classes and import logic must be well separated to ease the future DAO/database integration (NFR02).
* **Documentation (NFR05):** Javadoc, coding standards (CamelCase) and SVG figures.
* **Authorization:** Not applicable (console application operated by the infrastructure planner).

---

## 🔗 Found Out Dependencies

### Upstream Dependencies (Required before starting)
* **Datasets:** CSV/text files for robots, stations and orders (small, medium and large).
* **[USLP01] Domain Model:** Reference for naming the domain classes and attributes.

### Downstream Dependencies (Impacted by this story)
* **[USEI02] Station Monitoring:** Needs the loaded robots and stations.
* **[USEI03] Order Queue:** Needs the loaded orders.
* **[USEI04] / [USEI05] Assignment and Partitioning:** Need robots, stations and orders (with their items).

---

## 📥 Input & Output Data

> ⚠️ The fields below are assumed from the project description and must be confirmed against the headers of the provided files.

### Inputs
| File        | Field Name (assumed)                                         | Data Type       | Required | Example / Constraints              |
|:------------|:-------------------------------------------------------------|:----------------|:---------|:-----------------------------------|
| Robots      | `robotId`                                                    | String          | Yes      | Unique                             |
| Robots      | `maxVolume`                                                  | Decimal         | Yes      | > 0                                |
| Robots      | `maxWeight`                                                  | Decimal         | Yes      | > 0                                |
| Robots      | `refrigerated`                                               | Boolean         | Yes      | `true` / `false`                   |
| Robots      | `batteryLevel`                                               | Integer/Decimal | Yes      | 0–100                              |
| Robots      | `status`                                                     | Enum            | Yes      | `READY`, `CHARGING`, `MAINTENANCE` |
| Robots      | `stationId`                                                  | String          | Yes      | Must exist in Stations             |
| Stations    | `stationId`                                                  | String          | Yes      | Unique                             |
| Stations    | `capacity`                                                   | Integer         | Yes      | > 0                                |
| Orders      | `orderId`                                                    | String          | Yes      | Unique                             |
| Orders      | `customerId`                                                 | String          | Yes      |                                    |
| Orders      | `deliveryDeadline`                                           | Date/Time       | Yes      | Valid date/time                    |
| Order items | `itemId` / `quantity` / `weight` / `volume` / `refrigerated` | Mixed           | Yes      | Quantities and dimensions > 0      |

### Outputs
| Output              | Data Type                                          | Destination            | Description                                                            |
|:--------------------|:---------------------------------------------------|:-----------------------|:-----------------------------------------------------------------------|
| Robots collection   | Map keyed by `robotId`                             | In-memory system state | Fast lookup by id                                                      |
| Stations collection | Map keyed by `stationId`                           | In-memory system state | Fast lookup by id; holds the robots parked                             |
| Orders collection   | Map keyed by `orderId` (+ list of items per order) | In-memory system state | Base for the queue of [USEI03]                                         |
| Import report       | Log / console messages                             | User                   | Number of entities loaded and list of errors with file and line number |

---

## ⚙️ Design Notes (to be justified in the report)

| Data        | Candidate Structure                                                | Rationale (draft)                                                              | Import Complexity |
|:------------|:-------------------------------------------------------------------|:-------------------------------------------------------------------------------|:------------------|
| Robots      | `HashMap<String, Robot>`                                           | Access by id is the dominant operation; no ordering needed at load time        | O(R) expected     |
| Stations    | `HashMap<String, Station>` (or `LinkedHashMap` to keep file order) | Same as above; robots parked per station referenced by station                 | O(S) expected     |
| Orders      | `HashMap<String, Order>` / `LinkedHashMap`                         | Access by id; ordering by deadline handled in [USEI03] without heap structures | O(O) expected     |
| Order items | `List<OrderItem>` inside each `Order`                              | Iteration is the dominant operation                                            | O(I) in total     |

**Overall expected import complexity:** O(R + S + O + I). The team must confirm it with the empirical study (medium and large datasets).

---

## Edge Cases

* **Malformed line:** Wrong number of fields → log with file name and line number, skip the line, continue.
* **Missing value:** Required field empty → log and skip.
* **Invalid format:** Non-numeric volume/weight/battery, battery outside 0–100, invalid status or date → log and skip.
* **Duplicate id:** Repeated `robotId`, `stationId` or `orderId` → log and keep the first (or reject, per team decision documented).
* **Unknown reference:** Robot assigned to a non-existent station → log and skip.
* **Station over capacity:** More robots than the station capacity → log the inconsistency.
* **Empty file / file not found:** Clear message; the application must not crash.
* **Header line and blank lines:** Header correctly ignored; blank lines ignored.

---

## 🧪 Functional Tests / Validation

| Scenario ID | Test Case / Steps                           | Expected Result                                                                           |
|:------------|:--------------------------------------------|:------------------------------------------------------------------------------------------|
| **TC-01**   | Import a valid small dataset                | Number of robots, stations and orders loaded equals the number of data lines in each file |
| **TC-02**   | Import a file with one malformed line       | Error logged with line number; remaining lines loaded                                     |
| **TC-03**   | Import a line with a missing required value | Error logged; line not loaded                                                             |
| **TC-04**   | Import medium and large datasets            | Loaded without data loss; execution time recorded for the report                          |

---

# 🧪 QA & Test Documentation

See Report at [QA & Test Documentation: USEI01 Data Import and System Initialization](./qa-readme.md)

---

## 🧩 Related Modules & Classes

* **Domain (proposed):**
  * `Robot`, `RobotStatus` (enum)
  * `Station`
  * `Order`, `OrderItem`
* **Import / Services (proposed):**
  * `DataImporter` (or one importer per file type: `RobotImporter`, `StationImporter`, `OrderImporter`)
  * `ImportReport` (counts and errors)
* **Repositories / Storage (proposed):**
  * In-memory repositories (designed to be replaced by DAOs in the future)
* **UI:**
  * Console menu option “Import data”

---

## 🔄 Sequence Diagram

![Sequence Diagram](sd.svg)

## 🏁 Definition of Done (DoD)

* [ ] Acceptance criteria verified by someone other than the author (QA / Product Owner).
* [ ] Unit tests written and passing (TDD; minimum 80% coverage on new import logic).
* [ ] Tests cover valid, malformed, missing-value, duplicate and unknown-reference scenarios.
* [ ] Javadoc and coding standards applied; code reviewed by another team member.
* [ ] Report section written: operations, data structures, JCF justification, temporal complexity (R, S, O, I, Io) and empirical study with medium and large datasets.
* [ ] No heap-based structure (`PriorityQueue`) used.
* [ ] Feature flag configured if the feature is merged into `main` incomplete.
* [ ] Sequence diagram committed in SVG format.
* [ ] Generative AI usage declared (tool, purpose, parts supported), if applicable.