# [USEI02] Parking Station Monitoring and Control Console

* **Priority:** Medium
* **Story Points:** 5
* **Module / Component:** `Monitoring`,`Control`
* **Status:** Draft 

---

## 👤 User Story Description

**As a** shopping facility operator, \
**I want** to consult the status of parking stations to monitor their total and available capacity, as well as the robots currently parked, including their status and battery levels sorted in descending order,\
**So that** i can understand how often they are used / experience failures during use.


## 🚦 Definition of Ready (DoR)

* [ ] **Clear User Story:** Story follows the standard *As a / I want to / So that* format.
* [ ] **Acceptance Criteria Defined:** Clear, testable ACs agreed upon by Product and Engineering.
* [ ] **Dependencies Identified:** Understanding the User Story's dependencies and what depends on it
* [ ] **Inputs/Outputs Mapped:** Required data fields, payloads, and error states are documented.
* [ ] **Estimation Complete:** Story has been refined and estimated (story points assigned) by the dev team.
* [ ] **Small & Actionable:** Story is scoped down enough to be completed within a single sprint.

---

## 💬 Customer Specifications & Clarifications
* **Info to Display (2026-10-09):** The information to be displayed to the shopping facility operator is the status of the parking stations and the robots
* **Robot Station (2026-10-09):** A robot station is where robots park when not in active use. There they can charge their batteries, and the station can also be used for other purposes, such as robot maintenance.
* **Station Control Terminal (2026-10-09):** Each station has a control terminal. An operator can use it to get information about the station's capacity, its parked robots and their state.
* **Stations per Location (2026-10-09):** Each shopping location has one or more robot stations, usually one per level (except customer parking levels).
* **Robots (2026-10-09):** Each robot has a unique identification and a battery state, among other properties (max volume, max weight, refrigerated unit).


---

## ✅ Acceptance Criteria

* [ ] **AC1 - Get Info per Station:** A clear status view per station detailing maximum capacity, occupied spots, and free spots.
* [ ] **AC2 - Get Info per Parked Robot:** A list of parked robots for a selected station showing their operational status (READY, CHARGING, MAINTENANCE) and battery levels.
* [ ] **AC3 - Sort Robots from custom Comparator:** Sorted robot outputs driven by custom Comparator implementation (e.g., highest battery level first)

---

## 🔒 Non-Functional Requirements & Security
* **Business Rules (NFR01):** Business rules must be validated during data registration and updating (e.g., occupied spots cannot exceed a station's maximum capacity).
* **Database (NFR02):** Database operations via DAO/JDBC, ready to use PL/SQL API functions in the future.
* **Integration (NFR03):** A significant part of the integration between components/US must be carried out through files, program arguments, or environment variables.
* **Object-Oriented Programming (NFR04):** The design must follow OOP best practices, allowing easy maintenance and the addition of new functionalities.
* **Documentation (NFR05):** Recognized coding standards (e.g., CamelCase) must be adopted, the code must be documented with Javadoc, and all figures (e.g., the sequence diagram) must be recorded in SVG format.

---

## 🔗 Found Out Dependencies

### Upstream Dependencies (Required before starting)
* **[USEI01] Data Import and System Initialization:** Needs the .csv file to know the number of parking stations and robots parking, and the functions currently being executed.
---

## 📥 Input & Output Data

### Inputs
| Field Name | Data Type | Required | Source | Example / Constraints |
| :--- | :--- | :--- | :--- | :--- |
| `stationId` | Integer | Yes | USEI01 (CSV import) | Unique identifier of the station |
| `stationHashMap` | LinkedHashMap<String,Station> | Yes | USEI01 (CSV import) | Info about robot stations |
| `totalCapacity` | Integer | Yes | USEI01 (CSV import) | `> 0`; number of robots the station can hold |
| `robotId` | Integer | Yes | USEI01 (CSV import) | Unique identification of the robot |
| `status` | RoboStatus | Yes | USEI01 (CSV import) | `READY`, `CHARGING`, `MAINTENANCE` |
| `batteryLevel` | Integer | Yes | USEI01 (CSV import) | Battery state of the robot; range/unit to be confirmed |
| `selectedStationId` | Integer | Yes | User Input (console) | Must match an existing station |

### Outputs
* **Station status (AC1):** for each station, `stationId`, `totalCapacity`, `occupiedSpots` and `freeSpots` (`freeSpots = maxCapacity - occupiedSpots`).
* **Parked robots of the selected station (AC2):** for each parked robot, `robotId`, `status` and `batteryLevel`.
* **Ordering (AC3):** robots sorted by a custom `Comparator`: by status (`READY`, `CHARGING`, `MAINTENANCE`), then by highest battery level first.

---

## Edge Cases

* **All robots are in maintenance:** If every robot in a station has the status `MAINTENANCE`, the system must still display them in the monitoring view, but they must be marked as unavailable for assignment/use. 

* **Station has invalid or zero capacity:** If the `totalCapacity` shows zero, a negative number, or an invalid value, the station must be flagged as invalid. The system should reject the data during import or update and log an informative error.

* **Station is full:** If `occupiedSpots == totalCapacity`, then `freeSpots` must be `0`. The system should display the station as full and prevent any new robot from being parked there.

* **Station is empty:** If `occupiedSpots == 0`, then `freeSpots` must equal `totalCapacity`. The parked robot list for that station should be shown as empty with a clear message.

* **Selected station does not exist:** If the shopping facility operator selects a `stationId` that is not registered, the system must show an error message and ask for a valid station identifier.

* **No stations are registered:** If there are no parking stations in the system, the console must display an informative message instead of an empty or broken screen.

* **No robots parked in the selected station:**   If a valid station has no parked robots, the system must show an empty list and indicate that no robots are currently parked there.

* **Robot has unknown or invalid status:** If a robot’s status is missing or not one of `READY`, `CHARGING`, or `MAINTENANCE`, the system should display it as `UNKNOWN`, log a warning, and exclude it from normal sorting/assignment logic.

* **Robot has invalid battery level:** If `batteryLevel` is missing, below `0`, or above the maximum allowed value, the system should flag the robot as invalid, log the problem, and avoid using that value for sorting or assignment decisions.

* **Duplicate `robotId` in import data:** If the same `robotId` appears more than once, the system must reject the duplicate, keep only one valid record, and log an import error.

* **Robot assigned to more than one station at the same time:** A robot cannot be parked in multiple stations simultaneously. If this occurs, the system must log a business rule violation and keep only the most recent valid station assignment.

* **Sorting ties in robot list:** If two robots have the same status and the same battery level, the system must apply a deterministic secondary sort, such as ascending `robotId`, to ensure stable and predictable output when a customer requests a robot.

* **Robot in `CHARGING` status with full battery:** If a robot is `CHARGING` but its battery is already at maximum, the system should flag this as an inconsistent state and suggest that the robot may be ready (`READY`) instead.

* **Robot in `READY` status with zero battery:** If a robot is `READY` but has `batteryLevel == 0`, the system must flag the inconsistency and prevent the robot from being considered available for assignment.

<!--
---

## 🧪 Functional Tests / Validation

| Scenario ID | Test Case / Steps | Expected Result |
| :--- | :--- | :--- |
| **TC-01** | Apply valid 10% off code on a $100 cart | Cart total updates to $90; discount line item shows -$10.00 |
| **TC-02** | Apply code past its expiration date | Error banner appears; cart total remains unchanged |
| **TC-03** | Apply code `SUMMER10`, then apply code `FALL20` | `SUMMER10` is removed, `FALL20` discount is applied |

---

# 🧪 QA & Test Documentation

See Report at [QA & Test Documentation: USLP01 Feature Title](./qa-readme.md)

---

## 🧩 Related Modules & Classes

* **Frontend:**
  * Component: `CheckoutSummary.tsx`
  * State Hook: `usePromoCode.ts`
* **Backend:**
  * Controller: `PromoCodeController`
  * Service: `DiscountCalculationService`
  * Entity: `DiscountRule`

-->

## 🔄 Sequence Diagram

![Sequence Diagram](sd.svg)

## 🏁 Definition of Done (DoD)

* [ ] Acceptance criteria verified by QA / Product Owner.
* [ ] Unit tests written and passing (minimum 80% coverage on new service logic).
* [ ] Integration/E2E tests added for promo application flow.
* [ ] API specs updated in OpenAPI / Swagger documentation.
* [ ] Feature flag configured (`enable-checkout-promos`).