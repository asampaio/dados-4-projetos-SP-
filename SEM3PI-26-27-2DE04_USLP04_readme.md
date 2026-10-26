# [USLP04] Allow Client to see available stores by level

* **Priority:** Medium
* **Story Points:** -
* **Module / Component:** `Store Catalog , Front Office`
* **Status:** Draft 

---

## 👤 User Story Description

**As a** Customer \
**I want** to use the Shopping application to list the available stores \
**So that** I can choose where to buy.

---

## 🚦 Definition of Ready (DoR) 

* [ ] **Clear User Story:** Story follows the standard *As a / I want to / So that* format.
* [ ] **Acceptance Criteria Defined:** Clear, testable ACs agreed upon by Product and Engineering.
* [ ] **Dependencies Identified:** Import the GeoJSON, other USs that are affected, and USs that affect this US
* [ ] **Inputs/Outputs Mapped:** Required the data from the GeoJSON and needs to show the available stores on the floor.
* [ ] **Estimation Complete:** Story has been refined and estimated (story points assigned) by the dev team.
* [ ] **Small & Actionable:** Story is scoped down enough to be completed within a single sprint.x
---

## 💬 Customer Specifications & Clarifications

* **Info to Display (2026-10-01):** The information to be displayed to the customer is the store name and its level only.
* **Selection of Level (2026-10-08):** The program must allow the user to select one of the valid levels.
* **Level List (2026-10-08):** The application must display the list of available levels (floor numbers) for selection.
---

## ✅ Acceptance Criteria

* [ ] **AC1 - List levels** The app must list the levels of the shopping and allow the user to select a level
* [ ] **AC2 - Show only stores of the current level** The app must display the names of the stores of the selected level
* [ ] **AC3 - Get info from DB**  The app must access the information from the database



## 🔒 Non-Functional Requirements & Security
* **Business Rules (NFR01):** Business rules must be validated during data registration and updating (e.g., occupied spots cannot exceed a station's maximum capacity)
* **Database (NFR02):** Database operations via DAO/JDBC, ready to use PL/SQL API functions in the future.
* **Integration (NFR03):** Java main language; Other components must be integrated through files, program arguments or environment variables.
* **Object-Oriented Programming(NFR04):** Work developed must follow the OOP paradigm, making it easy to maintain and to add new functionalities.
* **Documentation (NFR05):** Recognized coding standards (e.g., CamelCase) must be adopted, the code must be documented with Javadoc, and all figures (e.g., the sequence diagram) must be recorded in SVG format.

---

## 🔗 Found Out Dependencies

### Upstream Dependencies (Required before starting)
* **[USLP06]  As a Retailer Administrator, I want to use the back-office application to import the stores and levels of a shopping location from a GeoJSON file, to create the stores available at a shopping location:** Need the file imported from USLP06 to list the stores by floor.

* **[USLP03] Front office Shopping application:** It will need the list of stores, given the user input.

---

## 📥 Input & Output Data

### Inputs
| Field Name | Data Type | Required | Source | Example / Constraints |
| :--- | :--- | :--- | :--- | :--- |
| `GeoJSONPath` | File | Yes | Retailer Administrator Input | the file must have the name and level of each stores (at least)  |
| `customerInputLevelToDisplayStore` | Double | Yes | Customer | the input must be a valid level and a number
### Outputs
| Field Name | Data Type | Destination | Example / Description |
| :--- | :--- | :--- | :--- |
| `ListOfStores` | List | Console | ordered list of store names on the selected floor

---

## Edge Cases

* **Invalid input type:** Displays a message indicating that the input type must be a double.
* **Invalid level:** Displays a message indicating that the input number must be one of the allowed numbers.
* **Not showing the level list:** Display a message stating that an unexpected problem occurred in the program and requesting the number of the current level.
---

<!--## 🧪 Functional Tests / Validation

| Scenario ID | Test Case / Steps | Expected Result |
| :--- | :--- | :--- |
| **TC-01** | Apply valid 10% off code on a $100 cart | Cart total updates to $90; discount line item shows -$10.00 |
| **TC-02** | Apply code past its expiration date | Error banner appears; cart total remains unchanged |
| **TC-03** | Apply code `SUMMER10`, then apply code `FALL20` | `SUMMER10` is removed, `FALL20` discount is applied |
---


# 🧪 QA & Test Documentation

See Report at [QA & Test Documentation: USLP04 Feature Title](./qa-readme.md)

---

## 🧩 Related Modules & Classes

* **Frontend:**
  * Component: `CheckoutSummary.tsx`
  * State Hook: `usePromoCode.ts`
* **Backend:**
  * Controller: `PromoCodeController`
  * Service: `DiscountCalculationService`
  * Entity: `DiscountRule`

--->

## 🔄 Sequence Diagram

![Sequence Diagram](sd.svg)

## 🏁 Definition of Done (DoD)

* [ ] Acceptance criteria verified by QA / Product Owner.
* [ ] Unit tests written and passing (minimum 80% coverage on new service logic).
* [ ] Integration/E2E tests added for promo application flow.
* [ ] API specs updated in OpenAPI / Swagger documentation.