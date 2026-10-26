# [USBD04] Relational Model (Logical Level)

* **Priority:** High
* **Story Points:** 5
* **Module / Component:** `BDDAD` / `Database`
* **Status:** In Progress
* **Developer:** Tiago Marquez (1221508)
* **QA:** Tiago Borges (1231109)

---

## 👤 User Story Description

**As a** Product Owner
**I want** the relational model to be created (logical level)
**So that** the physical model script for Oracle can be generated from it and the ShopRobot applications share a database with full data integrity

---

## 🚦 Definition of Ready (DoR)

* [x] **Clear User Story:** Story follows the standard *As a / I want to / So that* format.
* [x] **Acceptance Criteria Defined:** ACs taken from the project specification (Section 7.3.1).
* [x] **Dependencies Identified:** Conceptual data model (USBD03) and domain model (USLP01).
* [x] **Inputs/Outputs Mapped:** Mapping of classes and associations to tables documented below.
* [x] **Estimation Complete:** 5 story points.
* [x] **Small & Actionable:** Fits in Sprint 1.

---

## 💬 Customer Specifications & Clarifications

* **Specification (Project Specification v1, 7.3.1):** the model must include an ER diagram and all constraints, so that the physical model script for Oracle RDBMS can be generated automatically by a specific tool or an LLM.
* **Specification (Sections 4.1 and 4.2):** source of the business rules (shopping locations, levels, elevators, robot stations, robots, orders, shopping cart, routes, schedule of robot movements).
* **Clarification (2026-10-02, client session):** route planning is done by the system with operator input; items no robot can carry should be blocked at product registration; the GeoJSON import only guarantees the store name and its level.
* **Clarification (2026-10-06, Moodle BDDAD):** the model must cover the entire specification and all user stories.
* **Clarification (2026-10-09, Moodle BDDAD):** fixed, deterministic IDs — no auto-generated IDs.
* **Clarification (2026-10-12, client session):** refrigerated and normal products are not mixed; each robot receives its own sub-order; `BUSY` robot state allowed.
* **Out of Scope:** physical instantiation and data (USBD05); PL/SQL API and triggers (future sprints).

---

## ✅ Acceptance Criteria

* [ ] **AC1 - Coverage:** the model covers the requirements of the project specification, the user stories and the client clarifications (every class and association of the [conceptual data model](../../DataBase/conceptual-data-model/V2-conceptual-data-model.puml) is mapped).
* [ ] **AC2 - Complete model:** ER diagram ([USBD04_ShopRobot_Relational_Model.puml](../../DataBase/Relational-Model/USBD04_ShopRobot_Relational_Model.puml) / [svg](../../DataBase/Relational-Model/USBD04_ShopRobot_Relational_Model.svg)) and all constraints (below).
* [ ] **AC3 - Generable:** the Oracle DDL script ([01_create_tables.sql](../../DataBase/scripts/01_create_tables.sql)) was generated from this model (LLM) and runs on Oracle FreeSQL (validated in USBD05).

---

## 🔒 Non-Functional Requirements & Security

* **NFR01 - Business Rules:** every rule that can be declared is a `PRIMARY KEY`, `UNIQUE`, `FOREIGN KEY`, `CHECK` or `NOT NULL` constraint; the others are listed in [Rules not expressible as constraints](#rules-not-expressible-as-constraints).
* **NFR02 - Database:** one Oracle schema shared by all applications.
* **NFR05 - Documentation:** diagram in PlantUML + SVG; names in `UPPER_SNAKE_CASE`; constraint prefixes `pk_`, `uk_`, `fk_`, `ck_`, `nn_`, unique indexes `ux_`.
* **Normalization:** every table is in 3NF (no partial or transitive dependencies).
* **Generative AI (Section 3.6):** the mapping and the DDL were drafted with Claude (Anthropic) from the conceptual model and reviewed by the team.

---

## 🔗 Found Out Dependencies

### Upstream Dependencies
* **[USBD03] Conceptual Data Model:** [V2-conceptual-data-model.puml](../../DataBase/conceptual-data-model/V2-conceptual-data-model.puml)
* **[USLP01] Domain Model** (attributes): [V2-domain-model.puml](../../domain-model/V2-domain-model.puml)
* **[USBD02] Data Dictionary** (allowed values): [DBglossary](../../Glossaries/DBglossary/readme.md)

### Downstream Dependencies
* **[USBD05] Physical instantiation and data:** [scripts](../../DataBase/scripts/)
* **[USLP04], [USLP06]** and the future database integration of ESINF (USEI01-05).

---

## 🗺️ ER Diagram

![Relational Model](../../DataBase/Relational-Model/USBD04_ShopRobot_Relational_Model.svg)

Source: [USBD04_ShopRobot_Relational_Model.puml](../../DataBase/Relational-Model/USBD04_ShopRobot_Relational_Model.puml) — `*` mandatory, `<<PK>>` primary key, `<<FK>>` foreign key, `<<UK>>` unique.

---

## 🔁 Mapping Rules (conceptual → relational)

| Conceptual element | Relational mapping |
| :--- | :--- |
| Class | Table; identity attribute → `PRIMARY KEY` (fixed business IDs, no `IDENTITY`) |
| Attribute `Text` / `Integer` / `Number` / `DateTime` | `VARCHAR2` / `NUMBER(p)` / `NUMBER(p,s)` / `TIMESTAMP` |
| Attribute `Boolean` | `CHAR(1)` + `CHECK IN ('Y','N')` |
| Enumeration | `VARCHAR2` + `CHECK IN (...)` |
| GeoJSON / polygon | `CLOB` + `CHECK (... IS JSON)` |
| Association 1 — * | `FOREIGN KEY` on the "many" side (`NOT NULL` if the "one" side is mandatory) |
| Association 1 — 1 / 1 — 0..1 | `FOREIGN KEY` + `UNIQUE` |
| Association * — * | Association table with composite PK |
| Composition (filled diamond) | `FOREIGN KEY ... ON DELETE CASCADE`; weak entities get a composite PK (`SHOPPING_LEVEL`, `CART_ITEM`, `STOP`) |
| Aggregation / association | `FOREIGN KEY` without cascade |
| Generalization `Area` | One table per class; subtype PK = FK to `AREA`; discriminator `area_type` |
| Role of `SystemUser` in an association | FK `(user_id, role)` to `APP_USER (user_id, user_role)` with the role fixed by `CHECK` |
| Derivable association (`OrderItem — Shop`) | Not stored: the shop of an item is the shop of its product (3NF) |
| Oracle reserved words | `Order` → `CUSTOMER_ORDER`, `Level` → `SHOPPING_LEVEL`, `SystemUser` → `APP_USER` |

---

## 📋 Tables and Constraints

### RETAILER_COMPANY — `RetailerCompany`
| Column | Type | Null | Constraints |
| :--- | :--- | :---: | :--- |
| vat_number | VARCHAR2(15) | N | `pk_retailer_company`; `ck_retailer_company_vat` (country code + 2-13 alphanumerics) |
| company_name | VARCHAR2(100) | N | `uk_retailer_company_name` |

### APP_USER — `SystemUser`
| Column | Type | Null | Constraints |
| :--- | :--- | :---: | :--- |
| user_id | VARCHAR2(20) | N | `pk_app_user` |
| full_name | VARCHAR2(100) | N | |
| email | VARCHAR2(100) | N | `uk_app_user_email`; `ck_app_user_email` (format `x@y.zz`) |
| phone_number | VARCHAR2(20) | Y | `ck_app_user_phone` (9-15 digits, optional `+`) |
| user_role | VARCHAR2(25) | N | `ck_app_user_role` IN (`CUSTOMER`, `MERCHANT`, `RETAILER_ADMIN`, `MAINTENANCE_OPERATOR`, `SYSTEM_PLANNER`, `SYSTEM_OPERATOR`) |
| retailer_vat_number | VARCHAR2(15) | Y | `fk_app_user_retailer` → RETAILER_COMPANY |

Table: `uk_app_user_id_role UNIQUE (user_id, user_role)` — target of the role FKs; `ck_app_user_retailer` — only (and every) `RETAILER_ADMIN` works for a retailer company.

### SHOPPING_LOCATION — `ShoppingLocation`
| Column | Type | Null | Constraints |
| :--- | :--- | :---: | :--- |
| location_id | VARCHAR2(20) | N | `pk_shopping_location` |
| name | VARCHAR2(100) | N | `uk_shopping_location_name` |
| address | VARCHAR2(255) | N | |
| geo_json_data | CLOB | Y | `ck_shopping_location_geojson` (IS JSON) |
| vat_number | VARCHAR2(15) | N | `fk_shopping_location_retailer` → RETAILER_COMPANY |

### SHOPPING_LEVEL — `Level`
| Column | Type | Null | Constraints |
| :--- | :--- | :---: | :--- |
| location_id | VARCHAR2(20) | N | PK; `fk_shopping_level_location` → SHOPPING_LOCATION, ON DELETE CASCADE |
| level_number | NUMBER(3) | N | PK |
| floor_name | VARCHAR2(50) | Y | |
| description | VARCHAR2(255) | Y | |
| height_meters | NUMBER(5,2) | N | `ck_shopping_level_height` > 0 |

Table: `pk_shopping_level (location_id, level_number)`.

### AREA — `Area` (abstract)
| Column | Type | Null | Constraints |
| :--- | :--- | :---: | :--- |
| area_id | VARCHAR2(30) | N | `pk_area` |
| location_id, level_number | VARCHAR2(20), NUMBER(3) | N | `fk_area_level` → SHOPPING_LEVEL, ON DELETE CASCADE |
| name | VARCHAR2(100) | Y | |
| polygon_coordinates | CLOB | N | `ck_area_polygon` (IS JSON) |
| is_accessible_for_robots | CHAR(1) | N | `ck_area_accessible` IN ('Y','N'), default 'Y' |
| area_type | VARCHAR2(15) | N | `ck_area_type` IN (`SHOP`, `ROBOT_STATION`, `AMENITY`, `DELIVERY_SPOT`, `CORRIDOR`) |

Table: `uk_area_id_type UNIQUE (area_id, area_type)` — target of the subtype FKs.

### SHOP — `Shop`
| Column | Type | Null | Constraints |
| :--- | :--- | :---: | :--- |
| area_id | VARCHAR2(30) | N | `pk_shop` |
| area_type | VARCHAR2(15) | N | `ck_shop_area_type` = 'SHOP' |
| shop_code | VARCHAR2(20) | Y | `uk_shop_code` |
| category | VARCHAR2(50) | N | |
| is_open | CHAR(1) | N | `ck_shop_is_open` IN ('Y','N') |
| merchant_id | VARCHAR2(20) | Y | |
| merchant_role | VARCHAR2(25) | N | `ck_shop_merchant_role` = 'MERCHANT' |

Table: `fk_shop_area (area_id, area_type)` → AREA, ON DELETE CASCADE; `fk_shop_merchant (merchant_id, merchant_role)` → APP_USER. `shop_code` and `merchant_id` are optional because the GeoJSON import (USLP06) creates the shops without them (`SystemUser 0..1 — * Shop`).

### AMENITY — `Amenity`
| Column | Type | Null | Constraints |
| :--- | :--- | :---: | :--- |
| area_id | VARCHAR2(30) | N | `pk_amenity` |
| area_type | VARCHAR2(15) | N | `ck_amenity_area_type` = 'AMENITY' |
| amenity_type | VARCHAR2(30) | N | (open list: toilets, food_court, parking, bank, ...) |

Table: `fk_amenity_area (area_id, area_type)` → AREA, ON DELETE CASCADE.

### DELIVERY_SPOT — `DeliverySpot`
| Column | Type | Null | Constraints |
| :--- | :--- | :---: | :--- |
| area_id | VARCHAR2(30) | N | `pk_delivery_spot` |
| area_type | VARCHAR2(15) | N | `ck_delivery_spot_area_type` = 'DELIVERY_SPOT' |
| spot_code | VARCHAR2(20) | N | `uk_delivery_spot_code` |
| is_designated_waiting_area | CHAR(1) | N | `ck_delivery_spot_waiting` IN ('Y','N') |

Table: `fk_delivery_spot_area (area_id, area_type)` → AREA, ON DELETE CASCADE.

### CORRIDOR — `Corridor`
| Column | Type | Null | Constraints |
| :--- | :--- | :---: | :--- |
| area_id | VARCHAR2(30) | N | `pk_corridor` |
| area_type | VARCHAR2(15) | N | `ck_corridor_area_type` = 'CORRIDOR' |
| corridor_type | VARCHAR2(30) | N | (GeoJSON `indoor` value: corridor, room, ...) |

Table: `fk_corridor_area (area_id, area_type)` → AREA, ON DELETE CASCADE.

### ROBOT_STATION — `RobotStation`
| Column | Type | Null | Constraints |
| :--- | :--- | :---: | :--- |
| area_id | VARCHAR2(30) | N | `pk_robot_station` |
| area_type | VARCHAR2(15) | N | `ck_robot_station_area_type` = 'ROBOT_STATION' |
| station_id | VARCHAR2(20) | N | `uk_robot_station_id` |
| name | VARCHAR2(100) | Y | |
| max_capacity | NUMBER(3) | N | `ck_robot_station_capacity` > 0 |

Table: `fk_robot_station_area (area_id, area_type)` → AREA, ON DELETE CASCADE.

### ELEVATOR — `Elevator`
| Column | Type | Null | Constraints |
| :--- | :--- | :---: | :--- |
| elevator_code | VARCHAR2(20) | N | `pk_elevator` |
| max_payload_weight | NUMBER(7,2) | N | `ck_elevator_payload` > 0 (kg) |
| max_robot_capacity | NUMBER(2) | N | `ck_elevator_robot_capacity` > 0 |

### ELEVATOR_LEVEL — `Level` 2..* — 1..* `Elevator`
| Column | Type | Null | Constraints |
| :--- | :--- | :---: | :--- |
| elevator_code | VARCHAR2(20) | N | PK; `fk_elevator_level_elevator` → ELEVATOR, ON DELETE CASCADE |
| location_id, level_number | VARCHAR2(20), NUMBER(3) | N | PK; `fk_elevator_level_level` → SHOPPING_LEVEL, ON DELETE CASCADE |

### CONTROL_TERMINAL — `ControlTerminal`
| Column | Type | Null | Constraints |
| :--- | :--- | :---: | :--- |
| terminal_id | VARCHAR2(20) | N | `pk_control_terminal` |
| ip_address | VARCHAR2(45) | N | `uk_control_terminal_ip`; `ck_control_terminal_ip` (IPv4 or IPv6) |
| last_heartbeat | TIMESTAMP | Y | |
| station_area_id | VARCHAR2(30) | N | `uk_control_terminal_station` (1:1); `fk_control_terminal_station` → ROBOT_STATION, ON DELETE CASCADE |
| operator_id | VARCHAR2(20) | N | |
| operator_role | VARCHAR2(25) | N | `ck_control_terminal_op_role` = 'MAINTENANCE_OPERATOR' |

Table: `fk_control_terminal_operator (operator_id, operator_role)` → APP_USER.

### ROBOT — `Robot`
| Column | Type | Null | Constraints |
| :--- | :--- | :---: | :--- |
| robot_id | VARCHAR2(20) | N | `pk_robot` |
| model | VARCHAR2(50) | Y | |
| max_weight_capacity | NUMBER(6,2) | N | `ck_robot_max_weight` > 0 (kg) |
| max_volume_capacity | NUMBER(7,2) | N | `ck_robot_max_volume` > 0 (L) |
| is_refrigerated | CHAR(1) | N | `ck_robot_refrigerated` IN ('Y','N') |
| battery_level | NUMBER(5,2) | N | `ck_robot_battery` BETWEEN 0 AND 100 (%) |
| status | VARCHAR2(15) | N | `ck_robot_status` IN (`READY`, `CHARGING`, `MAINTENANCE`, `BUSY`) |
| location_id | VARCHAR2(20) | N | `fk_robot_location` → SHOPPING_LOCATION |
| station_area_id | VARCHAR2(30) | Y | `fk_robot_station` → ROBOT_STATION, ON DELETE SET NULL |

Table: `ck_robot_status_station` — `READY`/`CHARGING` ⇒ in a station; `BUSY` ⇒ not in a station; `MAINTENANCE` ⇒ either. Consequence (intended): a station with `READY`/`CHARGING` robots cannot be deleted; its robots must first be moved to another station.

### STATION_MOVEMENT — `StationMovement`
| Column | Type | Null | Constraints |
| :--- | :--- | :---: | :--- |
| movement_id | VARCHAR2(20) | N | `pk_station_movement` |
| station_area_id | VARCHAR2(30) | N | `fk_station_movement_station` → ROBOT_STATION, ON DELETE CASCADE |
| robot_id | VARCHAR2(20) | N | `fk_station_movement_robot` → ROBOT |
| scheduled_at | TIMESTAMP | N | |
| direction | VARCHAR2(10) | N | `ck_station_movement_direction` IN (`ENTERING`, `LEAVING`) |

Table: `uk_station_movement_robot_time UNIQUE (robot_id, scheduled_at)`.

### PRODUCT — `Product`
| Column | Type | Null | Constraints |
| :--- | :--- | :---: | :--- |
| product_id | VARCHAR2(20) | N | `pk_product` |
| name | VARCHAR2(100) | N | |
| description | VARCHAR2(500) | Y | |
| unit_price | NUMBER(8,2) | N | `ck_product_price` > 0 (€) |
| weight | NUMBER(7,3) | N | `ck_product_weight` > 0 (kg) |
| volume | NUMBER(8,3) | N | `ck_product_volume` > 0 (L) |
| requires_refrigeration | CHAR(1) | N | `ck_product_refrigeration` IN ('Y','N') |
| shop_area_id | VARCHAR2(30) | N | `fk_product_shop` → SHOP |

Table: `uk_product_shop_name UNIQUE (shop_area_id, name)`.

### SHOPPING_CART — `ShoppingCart`
| Column | Type | Null | Constraints |
| :--- | :--- | :---: | :--- |
| cart_id | VARCHAR2(20) | N | `pk_shopping_cart` |
| customer_id | VARCHAR2(20) | N | `uk_shopping_cart_customer` (a customer has at most one cart) |
| customer_role | VARCHAR2(25) | N | `ck_shopping_cart_cust_role` = 'CUSTOMER' |
| created_at | TIMESTAMP | N | |

Table: `fk_shopping_cart_customer (customer_id, customer_role)` → APP_USER, ON DELETE CASCADE.

### CART_ITEM — `CartItem`
| Column | Type | Null | Constraints |
| :--- | :--- | :---: | :--- |
| cart_id | VARCHAR2(20) | N | PK; `fk_cart_item_cart` → SHOPPING_CART, ON DELETE CASCADE |
| product_id | VARCHAR2(20) | N | PK; `fk_cart_item_product` → PRODUCT |
| quantity | NUMBER(4) | N | `ck_cart_item_quantity` > 0 |

### CUSTOMER_ORDER — `Order`
| Column | Type | Null | Constraints |
| :--- | :--- | :---: | :--- |
| order_id | VARCHAR2(20) | N | `pk_customer_order` |
| order_date | TIMESTAMP | N | |
| target_delivery_time | TIMESTAMP | N | `ck_customer_order_target` > order_date |
| status | VARCHAR2(20) | N | `ck_customer_order_status` IN (`PENDING`, `IN_PROCESSING`, `READY_FOR_DELIVERY`, `DELIVERED`, `CANCELLED`) |
| customer_id | VARCHAR2(20) | N | |
| customer_role | VARCHAR2(25) | N | `ck_customer_order_cust_role` = 'CUSTOMER' |
| delivery_spot_area_id | VARCHAR2(30) | N | `fk_customer_order_spot` → DELIVERY_SPOT |

Table: `fk_customer_order_customer (customer_id, customer_role)` → APP_USER.

### SUB_ORDER — `SubOrder`
| Column | Type | Null | Constraints |
| :--- | :--- | :---: | :--- |
| sub_order_id | VARCHAR2(20) | N | `pk_sub_order` |
| order_id | VARCHAR2(20) | N | `fk_sub_order_order` → CUSTOMER_ORDER, ON DELETE CASCADE |
| status | VARCHAR2(15) | N | `ck_sub_order_status` IN (`PENDING`, `ASSIGNED`, `PICKING`, `IN_TRANSIT`, `DELIVERED`) |
| robot_id | VARCHAR2(20) | Y | `fk_sub_order_robot` → ROBOT |

Table:
* `uk_sub_order_id_order UNIQUE (sub_order_id, order_id)` — target of the ORDER_ITEM FK.
* `ck_sub_order_robot` — only a `PENDING` sub-order can have no robot.
* `ux_sub_order_order_robot` (unique index) — the sub-orders of one order are carried by distinct robots (USEI05; client session 2026-10-12).
* `ux_sub_order_active_robot` (unique index) — a robot has at most one active (`ASSIGNED`, `PICKING`, `IN_TRANSIT`) sub-order: when assigned it becomes `BUSY` and is no longer available (USEI04). *(to confirm)*

### ORDER_ITEM — `OrderItem`
| Column | Type | Null | Constraints |
| :--- | :--- | :---: | :--- |
| order_id | VARCHAR2(20) | N | PK; `fk_order_item_order` → CUSTOMER_ORDER, ON DELETE CASCADE |
| product_id | VARCHAR2(20) | N | PK; `fk_order_item_product` → PRODUCT |
| quantity | NUMBER(4) | N | `ck_order_item_quantity` > 0 |
| unit_price_at_purchase | NUMBER(8,2) | N | `ck_order_item_price` > 0 (€) |
| sub_order_id | VARCHAR2(20) | N | |

Table:
* `pk_order_item (order_id, product_id)`.
* `fk_order_item_sub_order (sub_order_id, order_id)` → SUB_ORDER, **no cascade** (aggregation) — the sub-order belongs to the same order.
* The shop of an item is not stored: it is the shop of its product (keeps the table in 3NF).

### ROUTE — `Route`
| Column | Type | Null | Constraints |
| :--- | :--- | :---: | :--- |
| route_id | VARCHAR2(20) | N | `pk_route` |
| sub_order_id | VARCHAR2(20) | N | `uk_route_sub_order` (1 — 0..1); `fk_route_sub_order` → SUB_ORDER |
| planned_at | TIMESTAMP | N | |
| planner_id | VARCHAR2(20) | N | |
| planner_role | VARCHAR2(25) | N | `ck_route_planner_role` = 'SYSTEM_PLANNER' |

Table: `fk_route_planner (planner_id, planner_role)` → APP_USER.

### STOP — `Stop`
| Column | Type | Null | Constraints |
| :--- | :--- | :---: | :--- |
| route_id | VARCHAR2(20) | N | PK; `fk_stop_route` → ROUTE, ON DELETE CASCADE |
| sequence_number | NUMBER(3) | N | PK; `ck_stop_sequence` > 0 |
| stop_type | VARCHAR2(10) | N | `ck_stop_type` IN (`PICKUP`, `DELIVERY`) |
| planned_arrival | TIMESTAMP | N | |
| area_id | VARCHAR2(30) | N | `fk_stop_area` → AREA |

---

## Rules not expressible as constraints

These rules involve several rows or tables. They are verified by queries in USBD05 ([03_verification_queries.sql](../../DataBase/scripts/03_verification_queries.sql), Part C) and will be enforced by triggers / the PL/SQL API:

* Number of robots in a station ≤ `max_capacity` (C1).
* An elevator connects ≥ 2 levels, all of the same shopping location; every level has ≥ 1 elevator (C2, C3).
* Every retailer has ≥ 1 shopping location; every shopping location has ≥ 1 level (C4).
* Every `AREA` row has a row in the table of its subtype (`Area` is abstract) (C5).
* Every robot station has a control terminal (C6).
* Every order has ≥ 1 item and ≥ 1 sub-order; every sub-order has ≥ 1 item (C7).
* Total weight/volume of a sub-order ≤ capacity of its robot; refrigerated products only in refrigerated robots (C8, C9).
* The shops of an order are in the shopping location of its delivery spot (C10).
* A robot with an active sub-order is `BUSY` (and vice-versa); a `DELIVERED` order has all its sub-orders `DELIVERED` (C11, C12).
* Refrigerated and normal products are never in the same sub-order (C13).
* Every product can be carried by at least one robot of its shopping location (C14).
* A robot's station, the orders it carries and its movements are in the robot's shopping location (C15).
* A route has ≥ 2 stops, pickups at shops and ends with the delivery at the order's delivery spot (C16).

---

## 🛠️ Changes after the QA review of 2026-10-13

| QA issue | Change |
| :--- | :--- |
| QA-01 (3NF) | `ORDER_ITEM.shop_area_id` and `uk_product_id_shop` removed; the shop of an item is the shop of its product. The redundant association `OrderItem — Shop` was removed from the domain and conceptual models. |
| QA-02 (cascade on aggregation) | `fk_order_item_sub_order` has no `ON DELETE CASCADE`. |
| QA-03 (GeoJSON import) | `SHOP.shop_code` and `SHOP.merchant_id` are optional; `SystemUser 0..1 — * Shop` in the models. |
| QA-04 (gaps) | Added `STATION_MOVEMENT` (schedule of robot movements) and `ROBOT.location_id` (robot ↔ shopping location). `SHOP` holds only shops that joined ShopRobot (defined by the retailer administrator through the import). The delivery time window of USEI03 is a query filter on `target_delivery_time` (open question in USBD02). Also added `SHOPPING_CART`, `CART_ITEM`, `ROUTE`, `STOP`, `CORRIDOR` and the retailer of the `RETAILER_ADMIN` (USBD03 QA). |
| QA-05 (`ux_sub_order_active_robot`) | Kept, justified by USEI04 and marked *(to confirm)*. |
| QA-06 (`ON DELETE SET NULL`) | Behaviour made explicit: a station with `READY`/`CHARGING` robots cannot be deleted. |
| QA-07 (multiplicities) | Domain model aligned with the conceptual model: `RetailerCompany 1 — 1..* ShoppingLocation`, `Level 1 — * Area`. |
| QA-08 (data dictionary) | `floor_name`, `ROBOT.model` and `ROBOT_STATION.name` optional; `ip_address` IPv4 or IPv6 (`VARCHAR2(45)`); `unit_price_at_purchase > 0`. |
| QA-09 (links) | Links and the DDL header fixed. |
| QA-10 (Oracle evidence) | Pending: run on Oracle FreeSQL (USBD05). |

---

## 🧪 Functional Tests / Validation

| Scenario ID | Test Case / Steps | Expected Result |
| :--- | :--- | :--- |
| **TC-01** | Every class and association of the conceptual model has a table / FK in the diagram | Full coverage (23 tables) |
| **TC-02** | Every multiplicity is reflected (`NOT NULL`, `UNIQUE`, association table or rule in the list above) | No multiplicity left unmapped |
| **TC-03** | Every table is in 3NF | No partial or transitive dependency |
| **TC-04** | DDL generated from the model runs on Oracle FreeSQL | No errors (see USBD05) |
| **TC-05** | Constraint tests of [04_constraint_tests.sql](../../DataBase/scripts/04_constraint_tests.sql) | `Passed: 47   Failed: 0` |

---

# 🧪 QA & Test Documentation

See Report at [QA & Test Documentation: USBD04 Relational Model](./qa-readme.md)

---

## 🏁 Definition of Done (DoD)

* [ ] ER diagram in PlantUML and SVG in the repository.
* [ ] All constraints documented and present in the DDL.
* [ ] DDL runs on Oracle FreeSQL (USBD05).
* [ ] Reviewed by QA (Tiago Borges).
