# 📖 Project Glossary (USLP02 / USBD02)

Quick reference guide for domain-specific concepts used across the **ShopRobot** project.
In accordance with acceptance criteria, this glossary is consistent with the [Domain Model Diagram](../../domain-model/V2-domain-model.svg), focuses on business domain terms, and provides relevant context for the system problem domain while avoiding generic definitions.

* **Last updated:** 2026-10-19 — Sprint 1, after the implementation of USEI01–USEI03, USLP03, USLP04 and the first part of USLP06 (GeoJSON import); the USEI04 and USEI05 terms come from the project statement, the client sessions and the USEI05 design (not implemented yet).
* **Sections:** [Domain & Business Terms](#-domain--business-terms) (concepts of the domain model) · [Planning & Operations Terms](#-planning--operations-terms) (rules and concepts introduced by the user stories) · [Actor Names Used in the User Stories](#-actor-names-used-in-the-user-stories).
* **Relational data dictionary (USBD02):** [DB Glossary](../DBglossary/readme.md).

---

## 💼 Domain & Business Terms

### A - D

* **Amenity**
  * **Definition:** A public service or non-retail facility situated on a shopping location level (e.g., restrooms, food courts, parking areas, ATMs, information desks). It is modeled as a specialized kind of Area.
  * **Key Attributes / Rules:** `amenityType`.

* **Area**
  * **Definition:** A distinct polygon-delimited spatial zone on a specific level of a shopping location as derived from its GeoJSON floor layout. Each area is categorized into a specific subtype (Shop, Amenity, DeliverySpot, RobotStation, or Corridor) and specifies robot accessibility.
  * **Key Attributes / Rules:** `areaId`, `name`, `polygonCoordinates`, `isAccessibleForRobots`. Areas are also the places where the stops of a route happen.

* **Cart Item**
  * **Definition:** A line of a customer's shopping cart, holding a product the customer intends to buy and the desired quantity, before the purchase is confirmed.
  * **Key Attributes / Rules:** `quantity`, associated `Product`. Only exists inside its shopping cart; becomes an order item when the purchase is confirmed.

* **Control Terminal**
  * **Definition:** A dedicated computational terminal physically or logically linked to a robot station, operated by maintenance staff to monitor parking capacity, supervise robot telemetry and status, trigger maintenance modes, and inspect movement schedules.
  * **Key Attributes / Rules:** `terminalId`, `ipAddress`, `lastHeartbeat`. Exactly one terminal is associated with each robot station.

* **Corridor**
  * **Definition:** A passage area of a level (GeoJSON `indoor: corridor`) that connects shops, amenities, elevators and robot stations, and through which robots travel between stops.
  * **Key Attributes / Rules:** `corridorType`. An area specialization.

* **Customer**
  * **Definition:** An end-user who utilizes the Shopping front-office application to browse available stores, view products, assemble shopping carts, and place delivery orders designated to a chosen delivery spot.
  * **Domain Role:** Subtype/role of `SystemUser` (`CUSTOMER`). Fills at most one shopping cart at a time.

* **Delivery Spot**
  * **Definition:** A designated physical handover location within a shopping location where autonomous robots drop off fulfilled customer orders or wait for customer pick-up.
  * **Key Attributes / Rules:** `spotCode`, `isDesignatedWaitingArea`. An area specialization.

### E - H

* **Elevator**
  * **Definition:** A vertical transit infrastructure connecting two or more levels of a shopping location, used by autonomous robots to navigate between different floors while respecting maximum payload weight and robot capacity constraints.
  * **Key Attributes / Rules:** `elevatorCode`, `maxPayloadWeight`, `maxRobotCapacity`.

### I - L

* **Level**
  * **Definition:** A horizontal floor or storey of a shopping location (including subterranean floors such as parking garages). Contains areas and connects to vertical elevators.
  * **Key Attributes / Rules:** `levelNumber`, `floorName`, `description`, `heightMeters`. In the Shopping application the customer first chooses a level and then sees the stores of that level (USLP04). Every store imported from the GeoJSON layout must have a level (USLP06).

### M - P

* **Maintenance Operator**
  * **Definition:** A staff member who uses the control terminal of a robot station to consult its capacity and parked robots, change the state of robots (e.g., set a robot in maintenance) and view the schedule of robot movements.
  * **Domain Role:** Subtype/role of `SystemUser` (`MAINTENANCE_OPERATOR`).

* **Merchant**
  * **Definition:** A registered business operator or store manager responsible for managing a specific shop, maintaining its profile, and configuring the catalog of products offered through the ShopRobot service.
  * **Domain Role:** Subtype/role of `SystemUser` (`MERCHANT`).

* **Order**
  * **Definition:** A validated customer purchase request containing one or more order items to be fetched from various stores within the shopping location and delivered to a chosen delivery spot by an agreed deadline.
  * **Key Attributes / Rules:** `orderId`, `orderDate`, `targetDeliveryTime`, `status` (`PENDING`, `IN_PROCESSING`, `READY_FOR_DELIVERY`, `DELIVERED`, `CANCELLED`). An order is partitioned into one or more sub-orders. Orders are final once confirmed and paid. The `targetDeliveryTime` is the order's **deadline** (see [Deadline](#deadline)). The weight, volume and refrigeration need of an order come from its items (see [Order Load](#order-load)).

* **OrderItem**
  * **Definition:** An individual line item in an order specifying the chosen product, purchase unit price, and requested quantity.
  * **Key Attributes / Rules:** `quantity`, `unitPriceAtPurchase`, associated `Product`. The shop of an order item is the shop that offers its product. In the ESINF datasets (`orders.csv`, USEI01) each line is one order item, with its store, weight, volume and whether it needs refrigeration; several lines with the same `order_id` belong to the same order.

* **Product**
  * **Definition:** A commercial merchandise item offered by a shop through the ShopRobot platform, defined by dimensions, weight, unit price, and special handling requirements (such as cold chain / refrigeration).
  * **Key Attributes / Rules:** `productId`, `name`, `description`, `unitPrice`, `weight`, `volume`, `requiresRefrigeration`. Products that no robot can carry should be blocked at registration.

### Q - T

* **Retailer Administrator**
  * **Definition:** An administrative authority representing the retailer company who configures shopping infrastructure, including importing store directories, levels, and geographic layouts via GeoJSON data files.
  * **Domain Role:** Subtype/role of `SystemUser` (`RETAILER_ADMIN`). Works for a retailer company.

* **Retailer Company**
  * **Definition:** The legal business entity or holding that owns and operates one or more shopping locations.
  * **Key Attributes / Rules:** `vatNumber`, `companyName`.

* **Robot**
  * **Definition:** An autonomous electric ground vehicle operating within the shopping facility responsible for transporting picked goods from merchant stores to designated delivery spots.
  * **Key Attributes / Rules:** `robotId`, `model`, `maxWeightCapacity`, `maxVolumeCapacity`, `isRefrigerated`, `batteryLevel` (0–100%), `status` (`READY`, `CHARGING`, `MAINTENANCE`, `BUSY`). Operates in one shopping location and belongs to a robot station where it parks. Refrigerated robots carry refrigerated products only. Only a `READY` robot can be assigned to an order; once assigned it becomes `BUSY` (see [Robot Assignment](#robot-assignment)).
  * **Status values:** `READY` — parked and available for an order; `CHARGING` — parked and recharging its battery; `MAINTENANCE` — parked and unavailable (set by the maintenance operator); `BUSY` — working on an order, so it is not parked (state accepted by the client on 2026-10-12).

* **Robot Station**
  * **Definition:** A dedicated base area on a level where robots park when idle, recharge their batteries, and undergo operational diagnostics and maintenance.
  * **Key Attributes / Rules:** `stationId`, `name`, `maxCapacity`. Equipped with a control terminal. Usually one per level, except customer parking levels. Its current use is described by its [Station Occupancy](#station-occupancy) and its [Parked Robots](#parked-robot) (USEI02).

* **Route**
  * **Definition:** The plan of the path a robot follows to fulfil a sub-order: the ordered sequence of stops to collect the products in the shops and deliver them at the delivery spot. Planned by the system with input from a system planner.
  * **Key Attributes / Rules:** `routeId`, `plannedAt`. Has at least two stops (one pickup and one delivery).

* **Shopping Cart**
  * **Definition:** The virtual cart a customer fills with products while browsing the stores in the Shopping application, before confirming the purchase and choosing the delivery time and spot.
  * **Key Attributes / Rules:** `cartId`, `createdAt`. Contains cart items.

* **Shopping Location**
  * **Definition:** A physical commercial complex (mall, shopping centre, hypermarket) containing multiple levels and stores managed within the ShopRobot ecosystem.
  * **Key Attributes / Rules:** `locationId`, `name`, `address`, `geoJsonData`.

* **Shop**
  * **Definition:** A retail store operating within a shopping location that participates in the ShopRobot automated fulfillment program and offers products to customers.
  * **Key Attributes / Rules:** `shopCode`, `category`, `isOpen`. Specialization of Area. Also called **store** in the user stories and in the code (e.g. "list the stores of a level", USLP04). Only the stores that joined the solution are available in the system; in the GeoJSON layout a store is a feature with a `shop` tag, whose value is its category (see [GeoJSON Layout](#geojson-layout)).

* **Station Movement**
  * **Definition:** A scheduled entry of a robot into, or exit from, a robot station, shown in the control terminal so that the maintenance operator knows which robots will arrive or leave.
  * **Key Attributes / Rules:** `movementId`, `scheduledAt`, `direction` (`ENTERING`, `LEAVING`).

* **Stop**
  * **Definition:** A point of a route where the robot has to stop at a specific area, either to collect products in a shop or to deliver them at the delivery spot.
  * **Key Attributes / Rules:** `sequenceNumber`, `plannedArrival`, `stopType` (`PICKUP`, `DELIVERY`).

* **SubOrder**
  * **Definition:** A discrete physical parcel/portion of a customer's order assigned to a single autonomous robot for transportation, ensuring volume, payload weight, and refrigeration compatibility rules are satisfied.
  * **Key Attributes / Rules:** `subOrderId`, `status` (`PENDING`, `ASSIGNED`, `PICKING`, `IN_TRANSIT`, `DELIVERED`). Each robot receives its own sub-order; refrigerated and non-refrigerated items are never mixed. Created by [Order Partitioning](#order-partitioning) when one robot cannot carry the whole order; the sub-orders of an order go to distinct robots.

* **System Operator**
  * **Definition:** A staff member who supervises the operation of the ShopRobot system in a shopping location (orders, robots and deliveries in progress).
  * **Domain Role:** Subtype/role of `SystemUser` (`SYSTEM_OPERATOR`).

* **System Planner**
  * **Definition:** A staff member (also called *infrastructure planner* in the ESINF user stories) who plans the robot loads and routes, including the sequence of stops needed to gather the products of an order.
  * **Domain Role:** Subtype/role of `SystemUser` (`SYSTEM_PLANNER`). Plans routes.

* **System User**
  * **Definition:** Any authenticated human agent interacting with the system under a designated functional role.
  * **Key Attributes / Rules:** `userId`, `fullName`, `email`, `phoneNumber`, `userRole` (`CUSTOMER`, `MERCHANT`, `RETAILER_ADMIN`, `MAINTENANCE_OPERATOR`, `SYSTEM_PLANNER`, `SYSTEM_OPERATOR`).

---

## 🤖 Planning & Operations Terms

Business rules and concepts introduced by the Sprint 1 user stories (ESINF planning and LAPR3 import). They are not separate classes of the domain model: they describe how the domain concepts above are used.

#### Battery-Based Priority

* **Definition:** The rule that, among robots that can serve an order, the one with the **highest battery level** comes first, so that the robots best prepared for a delivery are used first and the others can keep charging.
* **Rules:** Robots with the same battery level are ordered by robot id (ascending), so the result is always the same; the client accepted any tie-break (2026-10-12). Applied when listing the parked robots of a station (USEI02) and when choosing robots for an order (USEI04, USEI05). In Sprint 1 it is implemented with a custom `Comparator` — no `PriorityQueue` or other heap.

#### Deadline

* **Definition:** The date and time by which a customer wants an order delivered — the order's `targetDeliveryTime`. It is the priority of the order for planning: the order with the earliest deadline is served first.
* **Rules:** Orders with the same deadline are served in their order of arrival (first come, first served). In the datasets it is the `delivery_time` of `orders.csv` (`yyyy-MM-dd HH:mm`). Used by the [Pending Order Queue](#pending-order-queue) (USEI03).

#### Feasibility Check

* **Definition:** The verification of whether a single available robot can fulfil a whole order on its own.
* **Rules:** Feasible when at least one `READY` robot has enough maximum weight and volume for the [Order Load](#order-load), the minimum required battery and, if the order needs refrigeration, a refrigeration unit. If no robot is feasible, the order must be split by [Order Partitioning](#order-partitioning) (USEI04 → USEI05). Products that no robot can ever carry should be blocked when the product is registered (client, 2026-10-02).

#### GeoJSON Layout

* **Definition:** The file that describes the floor plan of a shopping location in the GeoJSON format: each area (shop, amenity, corridor, elevator, …) is a feature with a polygon and properties such as `level`, `name` and its type (`shop`, `amenity`, `indoor`, `highway`).
* **Rules:** Imported by the retailer administrator in the back-office application to create the levels and stores of a shopping location (USLP06). A store must have at least a name and a level; a store without them is skipped and reported to the user as a warning (client, 2026-10-02). The path of the file is configured in a properties file with a default value.

#### Import Error

* **Definition:** A problem found in one line of an input file during the initial data import (robots, stations and orders — USEI01), or in one feature of the GeoJSON layout (USLP06, where it is called a warning).
* **Rules:** It identifies the file, the line number, the field and the reason. The line is skipped and the import continues with the next lines (client, 2026-09-28). Duplicate robot or station identifiers and robots that refer to an unknown station are import errors; several lines with the same order id are not, because each one is an item of the order. A missing file or an invalid header stops the import.

#### Order Load

* **Definition:** What a robot has to carry to fulfil an order or a sub-order: the **total weight** and **total volume** of its items, and whether **refrigeration** is needed.
* **Rules:** Total weight and volume are the sums of the items. An order needs refrigeration if at least one of its items does. Compared with the robot's maximum weight and volume in the [Feasibility Check](#feasibility-check) and in [Order Partitioning](#order-partitioning).

#### Order Partitioning

* **Definition:** Splitting an order that is too big for one robot, or that mixes refrigerated and non-refrigerated items, into sub-orders that can each be carried by one robot (USEI05).
* **Rules:** Refrigerated items are separated from non-refrigerated ones and go only to robots with refrigeration (client, 2026-10-12). Each sub-order must fit the weight and volume of its robot, and the sub-orders of an order are assigned to **distinct** `READY` robots, chosen by [Battery-Based Priority](#battery-based-priority). Result: the original order, its sub-orders and the robot of each one.

#### Overdue Order

* **Definition:** A pending order whose deadline has already passed and has not been processed yet.
* **Rules:** An order due exactly now is not overdue. Overdue orders are the most urgent ones: they are shown in a separate section from the orders due in the next hour, so that they do not hide the orders that are about to be due (USEI03).

#### Parked Robot

* **Definition:** A robot that is at its robot station and occupies one of its spots: its station is that station and its status is `READY`, `CHARGING` or `MAINTENANCE`.
* **Rules:** A `BUSY` robot is out working on an order, so it is not parked and does not occupy a spot. The parked robots of a station are shown with their status and battery level, by [Battery-Based Priority](#battery-based-priority) (USEI02).

#### Pending Order Queue

* **Definition:** The queue of confirmed orders that are waiting to be served by the robots (status `PENDING`), always kept sorted by [Deadline](#deadline) — the most urgent order is the next one.
* **Rules:** An order id can only be once in the queue. It can be filtered by a [Time Window](#time-window) (e.g. orders due in the next hour) or by customer id. A [Processed Order](#processed-order) leaves the queue (USEI03).

#### Processed Order

* **Definition:** An order that has been handled by the logistics operator (e.g. assigned to robots) and therefore no longer waits in the pending order queue.
* **Rules:** It is removed from the queue and disappears from every query; removing an order that is not in the queue changes nothing (USEI03).

#### Robot Assignment

* **Definition:** Choosing the robot (or robots, for a partitioned order) that will fulfil an order and reserving it for that order.
* **Rules:** The most suitable robot is a `READY` robot with enough capacity and the highest battery level ([Battery-Based Priority](#battery-based-priority)); proximity is not considered in Sprint 1 (client, 2026-10-12). Once assigned, the robot changes from `READY` to `BUSY` and stops occupying a spot in its station, so the [Station Occupancy](#station-occupancy) is updated (USEI04, USEI05).

#### Station Occupancy

* **Definition:** How full a robot station is: its **maximum capacity**, its **occupied spots** (number of [Parked Robots](#parked-robot)) and its **free spots** (capacity minus occupied spots).
* **Rules:** Shown for every station in the station monitoring console (USEI02). If the data has more parked robots than the capacity, the free spots are shown as 0 and the station is marked as over capacity. A station id can be typed in upper or lower case (e.g. `s001`).

#### Time Window

* **Definition:** A period from the current time up to a given limit, used to find the orders that must be delivered soon — e.g. the **orders due in the next hour** are the orders with a deadline until now + 1 hour.
* **Rules:** The window also includes the [Overdue Orders](#overdue-order), shown apart. To demonstrate it with the provided datasets (whose deadlines are in the past), "now" can be fixed with the environment variable `SHOPROBOT_NOW` (USEI03).

---

## 👥 Actor Names Used in the User Stories

The user stories use some actor names that are not in the domain model. They correspond to these roles of [System User](#q---t):

| Name in the user stories | Role in the domain model | Where | Notes |
| :--- | :--- | :--- | :--- |
| **Infrastructure Planner** | System Planner (`SYSTEM_PLANNER`) | USEI01 | Confirmed by the client (2026-10-02). |
| **Shopping Facility Operator** | Maintenance Operator (`MAINTENANCE_OPERATOR`) | USEI02 | The project statement gives the maintenance operator the task of consulting the capacity and the parked robots of a station. |
| **Logistics Operator** | System Operator (`SYSTEM_OPERATOR`) | USEI03, USEI04, USEI05 | Team interpretation: the system operator supervises orders, robots and deliveries; to be confirmed with the client. |
| **Customer** | Customer (`CUSTOMER`) | USLP04 | Uses the Shopping application. |
| **Retailer Administrator** | Retailer Administrator (`RETAILER_ADMIN`) | USLP06 | Uses the back-office application. |
