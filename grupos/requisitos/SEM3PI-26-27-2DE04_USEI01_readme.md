# [USEI01] Data Import and System Initialization

* **Priority:**  Medium
* **Story Points:** 5
* **Module / Component:** `Planning App`/ `Data Import + Domain`
* **Status:** In Review

---

## 👤 User Story Description

**As a** infrastructure planner
**I want to**  import initial information from text/CSV
files (robots, parking stations, and user purchase requests) and load it into the
most appropriate data structures.

---

## 🚦 Definition of Ready (DoR)

* [ ] **Acceptance Criteria Defined:** Clear, testable ACs agreed upon by Product and Engineering.
* [ ] **Dependencies Identified:** Upstream APIs, third-party integrations, or blocking tasks are resolved or scheduled.
* [ ] **Inputs/Outputs Mapped:** Required data fields, payloads, and error states are documented.
* [ ] **Estimation Complete:** Story has been refined and estimated (story points assigned) by the dev team.
* [ ] **Small & Actionable:** Story is scoped down enough to be completed within a single sprint.

---

## 💬 Customer Specifications & Clarifications

* The ESINF Sprint 1 description states that it is not mandatory to integrate the data into the database: the data may
  be processed only in memory. Full database integration will be required in later sprints, so the classes were designed
  to make that change easy.

* The import must report how many lines were loaded and which lines failed, with line numbers, without aborting on the
  first error.

* Input files have a header line, which is skipped. Fields are separated by commas and decimals use a dot.

---

## ✅ Acceptance Criteria

* [ ] **AC1** Successful parsing and loading of all entities (Robots, Stations, Orders) into Java Collection Framework
  structures without data
  loss or corruption.
* [ ] **AC2** Appropriate collection choices based on access patterns.
* [ ] **AC3** Informative error messages or logs for malformed lines or missing
  values during the import process.

---

## 🔒 Non-Functional Requirements & Security

* **Business Rules (NFR1): Business rules validation is applied during data registration.**

* **Integration (NFR03): Java main language; Other components must be integrated through files, program arguments or
  environment variables.**

* **Object Oriented Programming(NFR04): Work developed must follow the OOP paradigm, making it easy to maintain and to
  add new functionalities.**

* **Documentation (NFR05): Javadocs, usage of CamelCase and SVG style figures.**

---

## 🔗 Found Out Dependencies

### Upstream Dependencies (Required before starting)

* **None** The relational model (USBD04) is only needed for the database-backed DAO

### Downstream Dependencies (Impacted by this story)

* **[USEI02] Station Monitoring:** uses the stations and robots loaded in this US
* **[USEI03] Delivery queue management:** uses the imported deliveries
* **[USEI04] Viability check and robot attribution:** uses the robots, stations and deliveries
* **[USEI05] Delivery split between multiple robots:** uses deliveries (and respective items) and robots

---

## 📥 Input & Output Data

### Inputs

| Field Name       | Data Type | Required | Source  | Example / Constraints                                   |
|:-----------------|:----------|:---------|:--------|:--------------------------------------------------------|
| robotsFilePath   | String    | yes      | console | data/robots.csv;<br/>file must exist and be ledgible    |
| stationsFilePath | String    | yes      | console | data/stations.csv;<br/> file must exist and be ledgible |
| ordersFilePath   | String    | yes      | console | data/orders.csv;<br/>file must exist and be ledgible    |

### Outputs

| Field Name    | Data Type                     | Destination | Example / Description                                                                                                  |
|:--------------|:------------------------------|:------------|:-----------------------------------------------------------------------------------------------------------------------|
| robots        | Map<String,Robot>             | RobotDAO    | {"R001" -> Robot};<br/>Valid robots organized for efficient search by robotID                                          |
| stations      | LinkedHashMap<String,Station> | StationDAO  | {"S001" -> ParkingStation};<br/>Valid stations each with/without the respective robots parked, with a unique stationId |
| orders        | LinkedHashMap<String,Order>   | OrderDAO    | {"O001" -> Robot};<br/>Valid orders each with its own list of items                                                    |
| importSummary | String                        | Console     | "Robots:50 read, 48 loaded, 2 rejected";<br/>Register of the read,loaded and rejected entities                         |
| importErrors  | List<String>                  | Console     | "Robots.csv, line:14, batteryLevel invalid ("...")"                                                                    |
|

---

## Edge Cases

* **Non-existing/unintelligible file:** Clear error message, show retry option with text "input file path again?", show
  abort import option with "abort"
* **Empty file:** Report that no data was loaded
* **Malformed lines:** Register the error along with the line number on which it occurred
* **Duplicate identifiers:** Keep the first register and reject the following, reporting them
* **Robot with missing station:** Reject the robot and report the invalid reference
* **Station with more robots than max capacity:** Reject the robot and report the invalid reference
* **Order with inconsistent items:** (different deadline or client for the same orderId), Reject inconsistent lines and
  report
* **Repeated import:** (reloading the same file) Replace the existing data to avoid duplication

---

## 🧪 Functional Tests / Validation

| Scenario ID | Test Case / Steps | Expected Result |
|:------------|:------------------|:----------------|
|             |                   |                 |

---

# 🧪 QA & Test Documentation

See Report at [QA & Test Documentation: USLP01 Feature Title](./qa-readme.md)

---

## 🧩 Related Modules & Classes

* **Frontend:**
* ImportUI: (console menu, importStations,robots and orders; shows the summary and the error list)
* **Backend:**
* ImportController: delegates each import to the matching **DataImporter**
* DataImporter<T>: reads the file, skips the header lines and blank lines, collects errors and stores valid entities
* Parser<T>,StationParser,TransactionParser,RobotParser: convert one CSV line into an entity
* Station, Robot, Transaction(order): domain entities plus the list of error messages
* Dao<T>,StationDAO: robot and order DAO's: storage
* ImportResult: number of imported entities plus the list of error messages

---

## 🔄 Sequence Diagram

![Sequence Diagram](sd.svg)

## 🏁 Definition of Done (DoD)

* [ ] Acceptance criteria verified by QA / Product Owner.
* [ ] Unit tests written and passing (minimum 80% coverage on new service logic).
* [ ] Javadoc and coding conventions applied
* [ ] Diagrams/Graphs updated in SVG
* [ ] Merge in "Main" 