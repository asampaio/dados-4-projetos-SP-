# [USLP01] Domain Model (Conceptual Level)

* **Priority:** TBD (to be defined in Sprint Planning)
* **Story Points:** 5 (size M, as estimated on the project board)
* **Module / Component:** `Documentation` / `Domain Model` (PlantUML source and SVG rendering in `doc/domain-model/`, glossary in `doc/glossary/`)
* **Status:** In Review (diagram and glossary drafted; review and sign-off pending)

---

## 👤 User Story Description

**As a** Product Owner
**I want** the domain model to be created (conceptual level)
**So that** it is an essential communication element between all stakeholders, continuously reflecting their shared understanding of the domain

> Equivalent deliverable to [USBD03](../../BDDAD/USBD03/readme.md) — the requirements document states "Please refer to USLP01" for that story. Same artefact, counted for both LAPR3 and BDDAD: the diagram and the glossary are shared, and this documentation describes the same model.

---

## 🚦 Definition of Ready (DoR)

* [x] **Clear User Story:** Story follows the standard *As a / I want to / So that* format.
* [x] **Acceptance Criteria Defined:** Taken directly from the project requirements document.
* [x] **Dependencies Identified:** See "Found Out Dependencies" below.
* [x] **Inputs/Outputs Mapped:** See "Input & Output Data" below.
* [x] **Estimation Complete:** 5 story points (size M), as estimated on the project board.
* [ ] **Open Clarifications Resolved:** The robot states, the minimum battery, the payment and the identifiers in the GeoJSON file were clarified. The position of the robot stations is still open (see below).

---

## 💬 Customer Specifications & Clarifications

* **Requirements doc (USLP01):** the model is at conceptual level and is "an essential communication element between all stakeholders". It is a "dynamic" document that must continuously reflect the shared understanding of the domain, so it is updated whenever the requirements or the clarifications change.
* **Requirements doc (USLP01 acceptance criteria):** the model must cover all the business/domain aspects presented and discussed in the requirements document. The source of the diagram in PlantUML and its rendering as an SVG file must both be in the GitHub repository.
* **Requirements doc (NFR05):** all the images/figures produced during the project must be recorded in SVG format.
* **Requirements doc (USLP02/USBD02):** the glossary must be consistent with the model and focused on business concepts. Model and glossary are kept in sync.
* **Requirements doc (USBD03):** the BDDAD course asks for the same domain model and refers to USLP01, so there is a single model for both.
* **Requirements doc (layout):** a shopping location is a volume with several levels served by elevators. Each area (shop, corridor, toilet, parking, etc.) is represented by the coordinates of a polygon's vertices and is located in a specific level. The layout is described in a GeoJSON file.
* **Requirements doc (envisioned process):** only the stores that joined the solution are available in the Shopping App. Not every shop of the shopping center is a store of the system; the Retailer Administrator defines which stores are available.
* **Requirements doc (robots and orders):** each robot has a unique identification, a state, a maximum volume and weight, whether it is a refrigerated unit, and a battery level. An order may need several robots because of volume, weight, refrigeration or battery.
* **Modeling guidelines (provided to the team):** the model includes only business concepts; entities and their attributes; an identity attribute from the business domain on every entity; complex attributes modeled as classes (value objects); associations with direction, business-meaningful names and multiplicities; enumerations if necessary; and, optionally, meaningful business methods.
* **Clarification (client sessions, modeling priority):** the domain model is a priority. It must show the main entities and their attributes, the relations, the cardinalities and meaningful names on the associations.
* **Clarification (client sessions, duplicated stories):** the model and the glossary are required by more than one course (LAPR3 and BDDAD). They are treated as a single artefact, with one documentation entry that links to the other.
* **Clarification (client sessions, refrigeration):** refrigerated robots carry only refrigerated products, and non-refrigerated robots carry only non-refrigerated products. There are no mixed robots.
* **Clarification (client sessions, robot states):** a robot always has a state and it can change. Adding a state for a robot that is working on an order ("busy") is allowed and makes sense. The states are not limited by hardware constraints (the LEDs are still to be defined).
* **Clarification (client sessions, orders):** orders are final once confirmed and paid. Orders cannot be changed after payment.
* **Clarification (client sessions, sub-orders):** the sub-order is an explicit concept: each robot receives its own order or sub-order. How it is implemented internally (one order with a list of robots or separate sub-orders) is a decision of the team.
* **Clarification (client sessions, battery):** the minimum battery needed for an order is not in the acceptance criteria. It can be an input parameter or a global property (an indicative value is 20–25%). The unit is not defined in Sprint 1.
* **Clarification (client sessions, scope):** the proof of concept covers a single shopping center (NorteShopping) in this sprint.
* **Clarification (client sessions, GeoJSON):** the mandatory fields of the file are not fully defined. At a minimum it has the store name and the level.
* **Clarification (Moodle forum, GeoJSON, as recorded in [USLP06](../USLP06/readme.md)):** the shop features have the properties `id`, `name`, `shop_type` and `level`, and some `level` values are decimal numbers (for example 0.5) that are also levels to be imported.
* **Clarification (BDDAD teacher, prices):** changing the price of a product must not affect previous transactions, and a serious retail system keeps a price history.
* **Clarification (client sessions, actors):** the "Infrastructure Planner" of the ESINF stories is the System Planner of the use case diagram. The difference in naming was posted on the forum, and the answer is pending.
* **Decision (GeoJSON):** GeoJSON is the format used to import the layout, not a business concept. The layout is modeled as `ShoppingLocation` → `Level` → `Area`, and the shape of an area is a `Polygon` formed by `Coordinate`s.
* **Decision (area category):** stored as free text, because the requirements say the list of categories is not complete.
* **Decision (area identifier):** the `id` of an `Area` is the `id` property of its GeoJSON feature. It is never generated by the database.
* **Decision (level number):** the `number` of a `Level` is a decimal. It can be negative (for example a basement) or fractional (for example 0.5).
* **Decision (price history):** a `Product` has a history of prices (`ProductPrice`, identified by `validFrom` within its product) instead of a single price. A price change is a new `ProductPrice`, so it never alters past orders, and the unit price of an `OrderLine` is recorded at the time of purchase. The current price is the business method `currentPrice()` of `Product`. This entity is justified by the rule of the BDDAD teacher.
* **Decision (merchant of a store):** a `Store` created by the import of the GeoJSON file has no `Merchant` until the Retailer Administrator assigns one, so the merchant side of the association is `0..1`.
* **Decision (robot states):** `READY`, `IN_USE`, `CHARGING` and `MAINTENANCE`. The client accepted a "busy" state, so `IN_USE` is kept for the robot that is working on an order (`BUSY` in the assignment story USEI04).
* **Decision (sub-orders):** a sub-order exists only when no single robot can fulfil the order (volume, weight, refrigeration or battery). An order is fulfilled either by one robot directly or through its sub-orders, never both. The same applies to its route.
* **Decision (derived data):** the total weight and volume of an order and the occupied and free spots of a station are not stored. They are business methods of `Order` and `RobotStation`.
* **Decision (battery):** the minimum battery needed for an order is not stored in the model. It is an input parameter or a global property, as stated by the client.
* **Assumption (order lines):** a line is carried whole; it is never divided between sub-orders.
* **Assumption (order status):** `PENDING`, `ASSIGNED`, `IN_DELIVERY`, `DELIVERED` and `CANCELLED` are a proposal.
* **Assumption (payment):** not modeled as a class. The client only states that orders are final once confirmed and paid, and no payment data is requested in the requirements.
* **Assumption (robot station position):** a station is placed on a level and may be located in an area. The requirements do not say where stations are, so the area is optional.
* **Assumption (identities):** `Merchant` by `email`, `Retailer` and `ShoppingLocation` by `name`, `Store` by `name` within its shopping location, `Level` by `number` (decimal) within its shopping location, `Area` by an `id`, and `ProductPrice` by `validFrom` within its product. These are proposals.
* **Open question:** whether the position of a robot station is needed for route planning.
* **Out of Scope:** the relational model ([USBD04]), implementation classes, DAOs, the route planning algorithm, and payments.

---

## ✅ Acceptance Criteria

* [ ] **AC1 - Coverage:** The model covers all the business/domain aspects presented and discussed in the requirements document. *(to be confirmed by the reviewer)*
* [x] **AC2 - PlantUML source:** The source of the diagram, in PlantUML, is in the GitHub repository (`doc/domain-model/domain-model.puml`).
* [x] **AC3 - SVG rendering:** The rendering of the diagram, as an SVG file, is in the GitHub repository (`doc/domain-model/domain-model.svg`).

---

## 🔒 Non-Functional Requirements & Security

* **NFR01 (Business Rules):** not applicable to the diagram. Business rules that cannot be drawn (for example, an order is fulfilled by one robot or by sub-orders, never both) are written in the glossary.
* **NFR02 (Database):** not applicable. The relational model ([USBD04]) is derived from this model.
* **NFR03 (Integration):** not applicable.
* **NFR04 (OOP):** the model is the starting point of the object-oriented design, so it contains only business concepts, without technical classes.
* **NFR05 (Documentation):** the diagram follows the project templates and is recorded in SVG format, together with its PlantUML source.
* **Process:** trunk-based development, at least one commit per day. The source and the SVG are committed together on every change. Review by a team member other than the author (section 3.4).
* **Security:** not applicable (no authentication or sensitive data involved).

---

## 🔗 Found Out Dependencies

### Upstream Dependencies (Required before starting)
* **Requirements document:** source of all the business concepts, rules and user stories.
* **Main use cases (Figure 2):** source of the actors (Customer, Merchant, Retailer Administrator, Maintenance Operator, System Planner, System Operator and Public). Customer and Merchant are modeled as classes; the others are recorded only in the glossary.
* **Modeling guidelines:** the rules the diagram must follow.
* **Clarifications from the teachers/client:** the open questions above.

### Parallel Dependencies
* **[USLP02]/[USBD02] Glossary:** must stay consistent with the model. Both are updated together.
* **[USBD03] Conceptual domain model:** same deliverable, documented in the BDDAD folder.

### Downstream Dependencies (Impacted by this story)
* **[USBD04] Relational model:** derived from this model.
* **[USLP04] List available stores:** uses `ShoppingLocation`, `Level`, `Area` and `Store`.
* **[USLP06] Import of levels and stores:** uses `Level`, `Area`, `Polygon`, `Coordinate` and `Store`.
* **[USEI01]-[USEI05]:** use `Robot`, `RobotStation`, `Order`, `SubOrder`, `OrderLine` and `Product`.

---

## 📥 Input & Output Data

### Input Data
| Data                          | Type            | Description                                                        |
|:------------------------------|:----------------|:-------------------------------------------------------------------|
| Requirements document         | PDF             | Business concepts, rules and user stories                          |
| Main use cases (Figure 2)     | Figure          | Actors and use cases of the solution                               |
| Area categories               | Requirements    | `amenity`, `indoor`, `shop` and `highway` values of the layout     |
| Modeling guidelines           | Checklist       | Rules the model must follow                                        |
| Clarifications                | Teachers/client | Answers to the open questions                                      |

### Output Data
| Data                    | Type                 | Description                                              |
|:------------------------|:---------------------|:---------------------------------------------------------|
| Domain model source     | PlantUML (`.puml`)   | Source of the diagram, versioned in the repository       |
| Domain model rendering  | SVG (`.svg`)         | Rendering of the source, versioned in the repository     |

---

## Edge Cases

* **Shop that is not a store of the system:** only the stores that joined ShopRobot are a `Store`. The other shops of the shopping location are just areas.
* **Store without a merchant:** a store created by the import has no `Merchant` until one is assigned, and a store without a merchant has no products.
* **Elevators:** an elevator is an `Area` that connects levels. It is not a separate class, and level changes in a route are derived.
* **Levels without a robot station:** customer parking levels usually have none, so a level has at most one station.
* **Levels with a decimal or negative number:** accepted, because the `number` of a `Level` is a decimal.
* **Station without a known position:** the area of a station is optional.
* **Robot that is delivering:** it is not parked in any station.
* **Order that one robot cannot carry:** it is split into sub-orders, each carried by one robot. A simple order has no sub-orders.
* **Order that mixes refrigerated and non-refrigerated products:** it may need two sub-orders, one with a refrigerated robot.
* **Product whose price changes:** a new `ProductPrice` is added and the previous ones are kept; past order lines keep their unit price.
* **Areas without a name (corridors):** the name cannot identify an area, so each area has an `id`.
* **Area category not in the list:** accepted, because the category is free text.
* **Requirements change during the semester:** the diagram is a dynamic document. The source, the SVG and the glossary are updated together.

---

## 🧪 Functional Tests / Validation

| Scenario ID | Scenario                                              | Expected Result                                                                           |
|:------------|:------------------------------------------------------|:------------------------------------------------------------------------------------------|
| **F-01**    | Render the PlantUML source                            | The SVG is generated without errors                                                       |
| **F-02**    | Customer order with products from two stores          | `Customer`, `Order`, `OrderLine`, `Product`, `Store` and the delivery `Area` represent it |
| **F-03**    | Order above the limits of one robot                   | It is represented by sub-orders, each with one robot                                      |
| **F-04**    | Order with refrigerated and non-refrigerated products | The product and robot refrigeration flags allow the split                                 |
| **F-05**    | Import of levels and stores (USLP06)                  | `ShoppingLocation`, `Level`, `Area` with its `Polygon`, and `Store` hold the data         |
| **F-06**    | Station monitoring (USEI02)                           | Capacity, parked robots with state and battery are available; free spots are derived      |
| **F-07**    | Compare diagram and glossary                          | Every class, enumeration and value object has a glossary entry with the same name         |
| **F-08**    | Change the price of a product                         | `ProductPrice` keeps the previous prices and past order lines keep their unit price       |

---

# 🧪 QA & Test Documentation

See Report at [QA & Test Documentation: USLP01 Domain Model](./qa-readme.md)

---

## 🧩 Related Modules & Classes

* **Module:** documentation of the domain model (`doc/domain-model/`) and of the glossary (`doc/glossary/`).
* **Files:** `doc/domain-model/domain-model.puml` (source), `doc/domain-model/domain-model.svg` (rendering) and `doc/glossary/readme.md` (glossary).
* **Entities (15):** `Robot`, `RobotStation`, `Customer`, `Merchant`, `Retailer`, `ShoppingLocation`, `Level`, `Area`, `Order`, `SubOrder`, `OrderLine`, `Store`, `Product`, `ProductPrice` and `Route`.
* **Value objects (3):** `Address`, `Coordinate` and `Polygon`.
* **Enumerations (2):** `RobotState` and `OrderStatus`.

---

## 🗺️ Domain Model Diagram

![Domain-Model](../../domain-model/domain-model.svg)

Source: [domain-model.puml](../../domain-model/domain-model.puml)

## 🏁 Definition of Done (DoD)

* [ ] All acceptance criteria verified.
* [ ] Diagram checked against the modeling guidelines (TC-03 to TC-07 in `qa-readme.md`).
* [x] PlantUML source and SVG committed (`doc/domain-model/`).
* [ ] Glossary consistent with the diagram.
* [x] Assumptions and open questions recorded in this readme.
* [ ] Reviewed by a team member other than the author.
* [ ] Final sign-off recorded in `qa-readme.md`.