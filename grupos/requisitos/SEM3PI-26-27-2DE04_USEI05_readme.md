# [USEI05] Delivery Split Between Multiple Robots

* **Priority:** Medium
* **Story Points:** 8
* **Module / Component:** `Planning App` / `Order Splitting + Robot Assignment`
* **Status:** Draft

---

## 👤 User Story Description

**As a** logistics operator
**I want to** split oversized or incompatible purchase orders into sub-orders
**So that** they can be dispatched across multiple robots when a single robot cannot fulfill the request.

---

## 🚦 Definition of Ready (DoR)

* [ ] **Acceptance Criteria Defined:** Clear, testable ACs agreed upon by Product and Engineering.
* [ ] **Dependencies Identified:** Upstream APIs, third-party integrations, or blocking tasks are resolved or scheduled.
* [ ] **Inputs/Outputs Mapped:** Required data fields, payloads, and error states are documented.
* [ ] **Estimation Complete:** Story has been refined and estimated (story points assigned) by the dev team.
* [ ] **Small & Actionable:** Story is scoped down enough to be completed within a single sprint.

---

## 💬 Customer Specifications & Clarifications

 - The client expects the operator to enter the order to be processed and the system to provide the sub-orders (and robots) that satisfy the goal of the US. If not possible, the system should display an information message.

---

## ✅ Acceptance Criteria

* [ ] **AC1** Algorithmically partitioned sub-orders where items are grouped legally by compatibility (e.g., separating refrigerated items from non-refrigerated items if required) and weight/volume constraints.
* [ ] **AC2** Successful multi-robot assignment linking each sub-order to a distinct available robot.
* [ ] **AC3** A structured summary showing the original order, the generated sub-orders, and their respective assigned robots.he original order's items.

---

## 🔒 Non-Functional Requirements & Security

* **Business Rules (NFR01):** The validation of business rules must be followed during data registration and updating

* **Database (NFR02):** The database will be the main repository of information for the system and must reflect the necessary data integrity. The information should be maintained in a remote relational database management system (DBMS). All database operations must be done using an API of PL/SQL functions. As far as possible, all applications should use the same database. You should use the Data Access Object (DAO) pattern for working with JDBC.

* **Integration (NFR03):** To enhance interoperability between existing or developing systems, the main programming language should be Java. However, some components will need to be developed in other languages, such as PL/SQL for the database or C/Assembly for some specific components. Therefore, a significant part of the integration must be carried out through files, program arguments, or environment variables.

* **Object-Oriented Programming (NFR04):** As far as possible, the team should follow the object-oriented programming (OOP) paradigm, aiming at a design that allows easy maintenance and the addition of new functionalities, in line with OOP best practices.

* **Documentation (NFR05):** The team should follow the templates provided to document the project activities and issues. It should also adopt recognized coding standards (e.g., CamelCase) and code documentation tools, such as Javadoc. All the images/figures produced during the project should be recorded in SVG format

---

## 🔗 Found Out Dependencies

### Upstream Dependencies (Required before starting)

* **[USEI01] Data Import and System Initialization:** provides the robots, stations and orders (with their items)
  loaded into the Java collections.
* **[USEI04] Viability check and robot attribution:** provides the feasibility check (max volume, max weight, minimum
  battery) and the robot assignment/state update logic (READY → BUSY, station availability), which should be **reused**,
  not duplicated.

### Downstream Dependencies (Impacted by this story)

* None.

---

## 📥 Input & Output Data

### Inputs

| Field Name | Data Type                     | Required             | Source           | Example / Constraints                                                                    |
|:-----------|:------------------------------|:---------------------|:-----------------|:-----------------------------------------------------------------------------------------|
| oversizedOrder | Order | Yes                  | OrderRepository  | {O0001};<br/>Oversized order chosen by the logistics operator, that will be split so it's manageable by the available robots |
| robots        | Map<String,Robot>             | Yes |  RobotRepository | {"R001" -> Robot};<br/>Valid robots organized for efficient search by robotID   |
### Outputs

| Field Name        | Data Type                     | Destination                          | Example / Description                                                                                     |
|:------------------|:------------------------------|:-------------------------------------|:----------------------------------------------------------------------------------------------------------|
| oversizedOrder | Order | OrderRepository(?) | {O0001};<br/>Order that now owns a list/map of it's sub-orders, that will be distributed between available robots and dispatched |

---

## Edge Cases

* **Order not found / already assigned:** Clear error message; nothing is changed.
* **Order empty (no items):** Report that there is nothing to split.
* **Order fits in a single robot:** Inform that no split is needed and delegate to [USEI04].
* **Single item exceeds the capacity of every robot:** The order is unsplittable; reject with the item identified.
* **Not enough available robots for the number of sub-orders:** Reject the whole order; no state is changed
  (all-or-nothing); report how many robots were needed vs. available.
* **No robot with the required capability (e.g. refrigeration):** Reject with a message identifying the missing
  capability.
* **Robots with insufficient battery:** Excluded from candidates; reported if this is the cause of rejection.
* **Item with invalid weight/volume (zero or negative):** Rejected by validation (already prevented in [USEI01]).
* **Robot becomes unavailable during the operation:** Assignment is rolled back.

---

## 🧪 Functional Tests / Validation

| Scenario ID | Test Case / Steps                                                                               | Expected Result                                                                                   |
|:------------|:------------------------------------------------------------------------------------------------|:--------------------------------------------------------------------------------------------------|

---

# 🧪 QA & Test Documentation

---

## 🧩 Related Modules & Classes

* **Frontend:** Console menu option "Split and assign order" (reads `orderId`, prints `splitSummary`).

* **Backend:** `OrderSplitService`, `OrderSplitStrategy` (interface), `FirstFitDecreasingSplitStrategy`,
  `SubOrder`, `Item`, `CompatibilityType`, `RobotAssignmentService` (reused from USEI04), `Order`/`Robot`/`Station`
  repositories (from USEI01).

---

## 🔄 Sequence Diagram

![Sequence Diagram](sd.svg)

---

## 🏁 Definition of Done (DoD)

* [ ] Acceptance criteria verified by QA / Product Owner.
* [ ] Unit tests written and passing (minimum 80% coverage on new service logic).
* [ ] Javadoc and coding conventions applied
* [ ] Algorithm and its complexity documented
* [ ] Diagrams/Graphs updated in SVG
* [ ] Merge in "Main"