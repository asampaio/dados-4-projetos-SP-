# [USBD04] Relational Model (Logical Level)

* **Priority:** High
* **Story Points:** 5 (size M, as estimated on the project board)
* **Module / Component:** `Database` / `Relational Model`
* **Course Unit / Sprint:** BDDAD / Sprint 1
* **Status:** In Review (model complete, ER diagram and constraints committed; review and sign-off pending)

---

## 👤 User Story Description

**As a** Product Owner  
**I want** the relational model to be created (logical level)  
**So that** the physical model script for Oracle RDBMS can be automatically generated and the database can support all the Shopping, Planning and Administration applications

---

## 🚦 Definition of Ready (DoR)

* [x] **Clear User Story:** Story follows the standard *As a / I want to / So that* format.
* [x] **Acceptance Criteria Defined:** Taken directly from the project requirements document.
* [x] **Dependencies Identified:** Conceptual model ([USBD03]/[USLP01]), glossary ([USBD02]/[USLP02]) and requirements document ([USBD01]) are available.
* [x] **Inputs/Outputs Mapped:** Required inputs and deliverables are documented.
* [x] **Estimation Complete:** 5 story points (size M), as estimated on the project board.
* [x] **Small & Actionable:** Scoped to Sprint 1; the model evolves in the next sprints with the new requirements.

---

## 💬 Customer Specifications & Clarifications

* **Source (Project Description, §7.3.1):** The model must cover the requirements in the project document, plus any additional requirements resulting from user stories and client feedback.
* **Source (Project Description, §6.2 – NFR02):** The database is the main repository of information, must reflect the necessary data integrity, and is hosted in a remote relational DBMS (Oracle).
* **Source (Project Description, §6.1 – NFR01):** The business rules must be validated during data registration and updating, so the constraints of the model reflect them.
* **Source (Project Description, §4.1):** Several shopping locations may be registered in the database, and each retailer company may own several shopping locations.
* **Source (Project Description, §4.2):** Products available in ShopRobot may differ from those sold directly in the stores and are defined by the Merchant.
* **Clarification (BDDAD teacher, requirements):** the requirements of the data model come from the whole requirements document (all the user stories) and from the client clarifications.
* **Clarification (BDDAD teacher, iterative work):** LLMs and other tools may be used, but the repository must contain evidence of iterative and incremental development, with several versions of the artefacts in an open format. At least three development cycles of the data model are expected in Sprint 1.
* **Clarification (BDDAD teacher, IDs):** automatic generation of IDs is a bad idea when test data has to be injected. All the primary keys are explicit (no identity columns and no sequences).
* **Clarification (BDDAD teacher, prices):** changing the price of a product must not affect previous transactions, and a serious retail system keeps a price history.
* **Clarification (BDDAD teacher, passwords):** storing unencrypted passwords is a mistake with no excuse, and the teacher thinks storing passwords is not even necessary. The model stores no credentials.
* **Clarification (client sessions, scope):** the proof of concept covers a single shopping center (NorteShopping) in this sprint, but the model supports several shopping locations, as the requirements document describes.
* **Clarification (client sessions, refrigeration):** refrigerated robots carry only refrigerated products, and non-refrigerated robots carry only non-refrigerated products. There are no mixed robots.
* **Clarification (client sessions, robot states):** a state for a robot that is working on an order ("busy") is allowed, so the states are `READY`, `IN_USE`, `CHARGING` and `MAINTENANCE`.
* **Clarification (client sessions, orders):** orders are final once confirmed and paid. The sub-order is an explicit concept: each robot receives its own order or sub-order.
* **Clarification (client sessions, battery):** the minimum battery needed for an order is not stored in the model (input parameter or global property).
* **Clarification (Moodle forum, GeoJSON, as recorded in [USLP06](../../LAPR3/USLP06/readme.md)):** the shop features have the properties `id`, `name`, `shop_type` and `level`, and some `level` values are decimal numbers (for example 0.5). Stores need at least a name and a level.
* **Decision (price history):** the price of a product is kept in `PRODUCT_PRICE`, which is insert-only. A price change is a new row, and `ORDER_LINE.unit_price` is copied from the price in force at purchase, so past orders never change.
* **Decision (level number):** `SHOPPING_LEVEL.level_number` is `NUMBER(5,2)`, because levels can be fractional or negative.
* **Decision (merchant of a store):** `STORE.merchant_id` is optional, because the stores created by the GeoJSON import have no merchant until the retailer administrator assigns one.
* **Decision (area geometry):** the polygon of an area is stored in `AREA_VERTEX` (one row per vertex, ordered by `seq_no`). Only the exterior ring of a Polygon is stored, the closing vertex of the GeoJSON ring is discarded, and features that are not a Polygon are skipped with a warning. The Sprint 1 import (USLP06) loads levels and stores only, and the geometry is loaded in Sprint 2.
* **Decision (area identifier):** `AREA.area_id` is the `id` property of the GeoJSON feature. It is never generated by the database.
* **Decision (shop type):** the `shop_type` of the GeoJSON shops is not stored in Sprint 1 (the client asked for at least the name and the level).
* **Decision (units):** weight in kg, volume in litres, battery in %, coordinates in degrees (WGS 84), prices in euros, distance in metres and duration in seconds.
* **Decision (rules without constraints):** the business rules that cannot be expressed with PK, FK, UK or CHECK (R1 to R10, R16 and R17 in the diagram) are documented in the model and are implemented with triggers or in the PL/SQL API, as required by NFR01 and NFR02.
* **Out of Scope:** Physical-level tuning (indexes, partitioning, tablespaces) and Sprint 2/3 requirements not yet published.

---

## ✅ Acceptance Criteria

* [ ] **AC1 - Requirements Coverage:** The model covers all the requirements of the project document and any additional requirements resulting from user stories and client feedback. *(to be confirmed by the reviewer)*
* [x] **AC2 - ER Diagram:** A complete ER diagram is provided (PlantUML source + SVG rendering) in the repository: `doc/relational-model/relational-model.puml` and `relational-model.svg`.
* [x] **AC3 - Constraints:** All constraints are documented in the legend and notes of the diagram: primary keys, foreign keys, unique, not null, check constraints, a catalogue with the name of every constraint, and the business rules.
* [ ] **AC4 - Script Generation:** The model is complete enough for the Oracle physical model script (DDL) to be automatically generated by a tool or an LLM. *(verified in [USBD05])*
* [ ] **AC5 - Consistency:** The model is consistent with the conceptual model ([USBD03]/[USLP01]) and the glossary ([USBD02]/[USLP02]). *(to be confirmed by the reviewer)*

---

## 🔒 Non-Functional Requirements & Security

* **Data Integrity (NFR02):** Integrity must be enforced at database level (PK, FK, UK, NOT NULL, CHECK), not only in the applications.
* **Business Rules (NFR01):** Business rules must be validated during data registration and updating (via constraints and, where needed, PL/SQL).
* **Normalization:** All tables are at least in 3NF. The only exception is `ROBOT_STATION.level_id`, kept besides `area_id` because the area is optional, and documented in the diagram (rule R11).
* **Naming Conventions:** tables in upper case with underscores; constraints named `PK_<TABLE>`, `FK_<CHILD>_<PARENT>`, `UK_<TABLE>_<COLUMNS>` and `CK_<TABLE>_<COLUMN>`, with at most 30 characters.
* **Documentation (NFR05):** All figures are recorded in SVG format.
* **Security:** the model stores no credentials. If authentication is ever required, only a salted hash may be stored.
* **Portability:** Data types and constraints are Oracle (`VARCHAR2`, `NUMBER`, `TIMESTAMP`, `CHAR`).

---

## 🔗 Found Out Dependencies

### Upstream Dependencies (Required before starting)
* **[USBD01] Requirements Document:** Source of all functional and non-functional requirements.
* **[USBD02] / [USLP02] Glossary:** Business definitions used to name entities and attributes.
* **[USBD03] / [USLP01] Domain Model (conceptual):** The relational model is derived from it.

### Downstream Dependencies (Impacted by this story)
* **[USBD05] Instantiate Model in Oracle:** Needs the DDL generated from this model.
* **[USLP04] List Stores / [USLP06] Import GeoJSON:** Need the tables for levels, areas and stores.
* **[USEI01]–[USEI05] Planning App:** Future full database integration of robots, stations and orders.

---

## 📥 Input & Output Data

### Inputs
| Artifact            | Format         | Source              | Description                     |
|:--------------------|:---------------|:--------------------|:--------------------------------|
| Project description | PDF            | Teachers            | Requirements and clarifications |
| Domain model        | PlantUML / SVG | [USLP01] / [USBD03] | Conceptual model                |
| Glossary            | Markdown       | [USLP02] / [USBD02] | Business concepts               |
| Client feedback     | Text           | Moodle / meetings   | Additional requirements         |

### Outputs
| Artifact                      | Format             | Destination                          | Description                                                           |
|:------------------------------|:-------------------|:-------------------------------------|:----------------------------------------------------------------------|
| ER diagram source             | PlantUML (`.puml`) | `doc/relational-model/`              | Logical model source, with the legend, the rules and the constraints |
| ER diagram                    | SVG                | `doc/relational-model/`              | Rendered diagram                                                      |
| Constraints / data dictionary | In the diagram     | `doc/relational-model/`              | Tables, columns, types and constraints are in the diagram; the business definitions are in the glossary |
| DDL script                    | SQL (Oracle)       | [USBD05]                             | Generated from the model                                              |

---

## 📐 Tables of the Model

The model has 16 tables, grouped as in the diagram.

| Group                  | Table               | Primary key                | Main foreign keys and notes                                                                                         |
|:-----------------------|:--------------------|:---------------------------|:--------------------------------------------------------------------------------------------------------------------|
| Shopping structure     | `RETAILER`          | `retailer_id`              | `name` unique                                                                                                       |
| Shopping structure     | `SHOPPING_LOCATION` | `location_id`              | `retailer_id`; `name` unique; address columns (street, postal code, city)                                           |
| Shopping structure     | `SHOPPING_LEVEL`    | `level_id`                 | `location_id`; unique (`location_id`, `level_number`); decimal `level_number`                                       |
| Shopping structure     | `AREA`              | `area_id` (GeoJSON `id`)   | `level_id`; unique (`area_id`, `level_id`); free-text `category`                                                    |
| Shopping structure     | `AREA_VERTEX`       | (`area_id`, `seq_no`)      | `area_id`; longitude and latitude with range checks                                                                 |
| Merchants and catalogue| `MERCHANT`          | `merchant_id`              | `email` unique                                                                                                      |
| Merchants and catalogue| `STORE`             | `store_id`                 | `area_id` (unique); `merchant_id` (optional)                                                                        |
| Merchants and catalogue| `PRODUCT`           | `product_id`               | `store_id`; weight and volume greater than 0; refrigeration flag                                                    |
| Merchants and catalogue| `PRODUCT_PRICE`     | (`product_id`, `valid_from`)| `product_id`; price greater than or equal to 0; insert-only (price history)                                        |
| Orders                 | `CUSTOMER`          | `customer_id`              | `email` unique                                                                                                      |
| Orders                 | `CUSTOMER_ORDER`    | `order_id`                 | `customer_id`, `delivery_area_id`, `robot_id` (optional); status check                                              |
| Orders                 | `SUB_ORDER`         | `sub_order_id`             | `order_id`, `robot_id`; unique (`sub_order_id`, `order_id`); status check                                           |
| Orders                 | `ORDER_LINE`        | (`order_id`, `line_number`)| `product_id`; composite FK (`sub_order_id`, `order_id`) to `SUB_ORDER` (optional); `unit_price` copied at purchase   |
| Robots and stations    | `ROBOT_STATION`     | `station_id`               | `level_id` (unique); composite FK (`area_id`, `level_id`) to `AREA` (optional); capacity greater than 0             |
| Robots and stations    | `ROBOT`             | `robot_id`                 | `station_id` (optional); state check; battery 0 to 100; state and station consistent                                |
| Routes                 | `ROUTE`             | `route_id`                 | `order_id` or `sub_order_id` (exactly one, both unique); distance and duration greater than or equal to 0           |

---

## Edge Cases

* **Area geometry:** the polygon is stored in `AREA_VERTEX`; the closing vertex of the GeoJSON ring is discarded, so an area has at least 3 distinct vertices.
* **Robot without station:** a robot in use is not parked, so `station_id` is nullable, and a check keeps the state and the station consistent.
* **Station on parking levels:** a level has at most one station and may have none, so the model does not force one per level.
* **Order split:** one order may be fulfilled by several robots through `SUB_ORDER`; a split order has all its lines linked to a sub-order.
* **Refrigerated items:** refrigerated products only go in refrigerated robots (rule R13), checked through the order lines and the robot of the order or sub-order.
* **Price change:** a new `PRODUCT_PRICE` row is added and the previous rows are kept; the order lines already placed keep their `unit_price`.
* **Store without merchant:** accepted (stores created by the GeoJSON import); such a store has no products.
* **Level with a decimal or negative number:** accepted by `level_number`.
* **Multiple locations:** levels, areas, stores and stations belong to a single `SHOPPING_LOCATION`. Robots are linked to a location only through the station where they are parked, which is enough for the proof of concept with one shopping center.
* **Passwords:** no credentials are stored in the model.

---

## 🔁 Model Versions (iterative development)

* **Version 1 (#19):** first logical model, with the 15 tables derived from the domain model.
* **Version 2 (#97):** price history (`PRODUCT_PRICE`), decimal level number, optional store merchant, decisions about the geometry and the units, and constraint catalogue updated.

---

## 🧪 Functional Tests / Validation

The checks are the same as in the [qa-readme](./qa-readme.md):

| Scenario ID | Test Case / Steps                                                  | Expected Result                                               |
|:------------|:-------------------------------------------------------------------|:--------------------------------------------------------------|
| **TC-01**   | Compare each requirement of the project description with the model | Every requirement has a table, attribute or relationship      |
| **TC-02**   | Compare each concept of the glossary and domain model with the model | Same concepts and terminology; no missing or extra entity   |
| **TC-03**   | Inspect every table for a primary key                              | No table without PK                                           |
| **TC-04**   | Inspect every relationship for FK and nullability                  | All relationships have the correct FK                         |
| **TC-05**   | Check the CHECK, UNIQUE and NOT NULL rules                         | All documented rules are present                              |
| **TC-06**   | Analyze the functional dependencies of each table                  | All tables in 3NF (or justified exception)                    |
| **TC-07**   | Generate DDL from the model and run it in Oracle                   | Script runs without errors                                    |
| **TC-08**   | Insert data violating a rule (e.g. battery = 150)                  | Database rejects the insert                                   |
| **TC-09**   | Open the `.puml` and the `.svg`                                    | SVG matches the source and renders correctly                  |
| **TC-10**   | Check the price history                                            | `PRODUCT_PRICE` is insert-only and `ORDER_LINE` keeps its price |
| **TC-11**   | Search the model for "pending" and "to confirm"                    | No open decision left                                         |

---

# 🧪 QA & Test Documentation

See Report at [QA & Test Documentation: USBD04 Relational Model](./qa-readme.md)

---

## 🧩 Related Modules & Classes

* **Documentation:**
  * `doc/relational-model/relational-model.puml`
  * `doc/relational-model/relational-model.svg`
* **Database:**
  * DDL script (Oracle) generated from the model — see [USBD05]
* **Frontend / Backend:** Not applicable (documentation/modelling story)

---

## 🔄 Model Diagram

![ER Diagram](../../relational-model/relational-model.svg)

Source: [relational-model.puml](../../relational-model/relational-model.puml)

*(Sequence diagram not applicable for this story; the ER diagram replaces it.)*

## 🏁 Definition of Done (DoD)

* [ ] Acceptance criteria verified by someone other than the author (QA / Product Owner).
* [x] ER diagram source (PlantUML) and SVG committed to the repository.
* [x] All constraints documented (PK, FK, UK, NOT NULL, CHECK, business rules).
* [ ] Model consistent with the conceptual model and glossary.
* [ ] DDL generated from the model runs successfully in Oracle.
* [ ] Requirements document ([USBD01]) updated with any new requirement/clarification found.
* [ ] Generative AI usage declared (tool, purpose, parts supported), if applicable.