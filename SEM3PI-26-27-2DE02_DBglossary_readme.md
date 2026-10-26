# 📖 Data Dictionary (USBD02)

Data dictionary of ShopRobot for BDDAD. For each entity of the [domain model](../../domain-model/V2-domain-model.svg) it gives a definition, a table with the meaning, units and allowed values of each attribute, and the business rules that involve more than one attribute or entity.

* Entity and attribute names are the ones used in the domain model (V2).
* Allowed values are business constraints, not SQL: the relational model (USBD04) turns them into keys, `NOT NULL`, `CHECK` constraints and validations in the PL/SQL API (NFR01).
* Date-times follow ISO 8601 (e.g. `2026-10-07T15:30`); coordinates are decimal degrees (WGS 84, as in GeoJSON); money is in euros.
* Rules marked *(client, date)* come from the [client clarifications](../../requirements/requirements.md) of that date.
* Values and rules marked *(to confirm)* are not stated by the client and must be confirmed (see [Open Questions](#-open-questions)).

## 🗂️ Entities

### Amenity

Area of a level that offers a service to the public (e.g. toilets, food court, parking, bank, pharmacy). It is a kind of [Area](#area).

| Attribute | Description | Units | Allowed values |
| :--- | :--- | :--- | :--- |
| amenityType | Kind of service offered | — | e.g. `toilets`, `food_court`, `parking`, `post_office`, `bank`, `pharmacy`, `bureau_de_change`, `restaurant`; the list is not closed |

**Business rules**
* Has all the attributes and rules of an area.

### Area

Delimited zone of a level of a shopping location, as described in the GeoJSON layout. It is abstract: every area is a [Shop](#shop), an [Amenity](#amenity), a [DeliverySpot](#deliveryspot), a [Corridor](#corridor) or a [RobotStation](#robotstation).

| Attribute | Description | Units | Allowed values |
| :--- | :--- | :--- | :--- |
| areaId | Identification of the area in the GeoJSON file | — | mandatory; unique within the shopping location *(to confirm)* |
| name | Name of the area | — | optional (mandatory for shops) |
| polygonCoordinates | Vertices of the polygon that delimits the area | decimal degrees | at least 3 vertices; longitude −180 to 180, latitude −90 to 90 |
| isAccessibleForRobots | Whether robots can enter or cross the area | — | yes / no |

**Business rules**
* Belongs to exactly one level.
* Every area is exactly one of: shop, amenity, delivery spot, corridor or robot station.
* Robots only stop at, or go through, areas that are accessible for robots *(to confirm)*.

### CartItem

A product and the quantity of it that a customer put in the shopping cart.

| Attribute | Description | Units | Allowed values |
| :--- | :--- | :--- | :--- |
| quantity | Number of units of the product | units | integer > 0 |

**Business rules**
* Belongs to exactly one shopping cart and references exactly one product.
* A cart has at most one item per product (the quantity is increased instead).

### ControlTerminal

Device of a robot station used for management tasks.

| Attribute | Description | Units | Allowed values |
| :--- | :--- | :--- | :--- |
| terminalId | Identification of the terminal | — | unique, mandatory |
| ipAddress | Network address of the terminal | — | valid IPv4 or IPv6 address; unique |
| lastHeartbeat | Last time the terminal reported that it is working | date-time | not in the future |

**Business rules**
* Belongs to exactly one robot station; each robot station has exactly one control terminal.
* Operated by exactly one system user with role `MAINTENANCE_OPERATOR` *(to confirm)*.
* Through it, the maintenance operator consults the station capacity, the parked robots and their status, changes the status of a robot and views the schedule of station movements.

### Corridor

Indoor area that robots go through between other areas, such as a corridor or a room of the GeoJSON layout. It is a kind of [Area](#area).

| Attribute | Description | Units | Allowed values |
| :--- | :--- | :--- | :--- |
| corridorType | Kind of indoor area | — | e.g. `corridor`, `room` (GeoJSON `indoor` values); the list is not closed |

**Business rules**
* Has all the attributes and rules of an area.

### DeliverySpot

Area of the shopping location where the robots deliver an order to the customer. It is a kind of [Area](#area).

| Attribute | Description | Units | Allowed values |
| :--- | :--- | :--- | :--- |
| spotCode | Code of the delivery spot | — | unique, mandatory |
| isDesignatedWaitingArea | Whether it is an area where customers wait for their orders *(to confirm)* | — | yes / no |

**Business rules**
* Has all the attributes and rules of an area.
* Receives zero or more orders; every order is delivered at exactly one delivery spot.
* It is in the same shopping location as the shops of the orders delivered there *(to confirm)*.

### Elevator

Means the robots use to move between levels.

| Attribute | Description | Units | Allowed values |
| :--- | :--- | :--- | :--- |
| elevatorCode | Code of the elevator | — | unique, mandatory |
| maxPayloadWeight | Maximum weight the elevator can lift | kg | > 0 |
| maxRobotCapacity | Maximum number of robots inside the elevator at the same time | robots | integer > 0 |

**Business rules**
* Connects two or more levels of the same shopping location; every level is served by at least one elevator.
* Robots can only change level through an elevator.
* The robots inside it never exceed `maxRobotCapacity`, and their total weight (with their load) never exceeds `maxPayloadWeight` *(to confirm)*.

### Level

Floor of a shopping location.

| Attribute | Description | Units | Allowed values |
| :--- | :--- | :--- | :--- |
| levelNumber | Number of the floor (negative below ground) | — | integer; unique within the shopping location |
| floorName | Name of the floor, e.g. "Piso 0" | — | optional |
| description | Description of the floor | — | optional |
| heightMeters | Height of the floor *(to confirm)* | m | > 0 |

**Business rules**
* Belongs to exactly one shopping location and contains zero or more areas.
* Usually has one robot station, except customer parking levels, which usually have none.

### Order

Purchase confirmed by a customer, to be collected and delivered by one or more robots.

| Attribute | Description | Units | Allowed values |
| :--- | :--- | :--- | :--- |
| orderId | Identification of the order | — | unique, mandatory |
| orderDate | When the customer confirmed the purchase | date-time | mandatory |
| targetDeliveryTime | When the customer wants the order delivered; it is the order's deadline | date-time | mandatory; after `orderDate` |
| status | Stage of the order (enum `OrderStatus`) | — | `PENDING`, `IN_PROCESSING`, `READY_FOR_DELIVERY`, `DELIVERED`, `CANCELLED` |

**Business rules**
* Placed by exactly one system user with role `CUSTOMER`.
* Contains one or more order items and is partitioned into one or more sub-orders.
* Delivered at exactly one delivery spot, chosen when the purchase is confirmed.
* Pending orders are served by earliest `targetDeliveryTime`.
* Is final once confirmed and paid: its items cannot be changed afterwards *(client, 2026-10-02)*.

### OrderItem

A product, its quantity and its price within an order.

| Attribute | Description | Units | Allowed values |
| :--- | :--- | :--- | :--- |
| quantity | Number of units of the product | units | integer > 0 |
| unitPriceAtPurchase | Price of one unit at the moment of the purchase | € | > 0, two decimal places |

**Business rules**
* Belongs to exactly one order and references exactly one product; an order has at most one item per product.
* Its shop is the shop that offers the product (it is not recorded separately).
* Bundled in exactly one sub-order of its order.

### Product

Item that a shop sells through ShopRobot.

| Attribute | Description | Units | Allowed values |
| :--- | :--- | :--- | :--- |
| productId | Identification of the product | — | unique, mandatory |
| name | Name of the product | — | mandatory; unique within its shop |
| description | Description shown to the customer | — | optional |
| unitPrice | Selling price of one unit through ShopRobot | € | > 0 *(to confirm)*, two decimal places |
| weight | Weight of one unit | kg | > 0 |
| volume | Volume of one unit | L | > 0 |
| requiresRefrigeration | Whether the product must travel refrigerated | — | yes / no |

**Business rules**
* Offered by exactly one shop and defined by that shop's merchant; it may differ from the products sold directly at the shop.
* A product that requires refrigeration can only be carried by a refrigerated robot *(client, 2026-10-12)*.
* A product larger or heavier than any robot can carry should be rejected when it is registered *(client, 2026-10-02)*.

### RetailerCompany

Company that owns shopping locations.

| Attribute | Description | Units | Allowed values |
| :--- | :--- | :--- | :--- |
| vatNumber | Tax identification number of the company | — | unique, mandatory *(to confirm)* |
| companyName | Name of the company | — | mandatory; unique |

**Business rules**
* Owns one or more shopping locations; every shopping location is owned by exactly one company.
* Its retailer administrators work for it.

### Robot

Autonomous electric robot that collects the products of an order in the shops and delivers them to the customer.

| Attribute | Description | Units | Allowed values |
| :--- | :--- | :--- | :--- |
| robotId | Identification of the robot | — | unique, mandatory |
| model | Model of the robot | — | optional |
| maxWeightCapacity | Maximum weight the robot can carry | kg | > 0 |
| maxVolumeCapacity | Maximum volume the robot can carry | L | > 0 |
| isRefrigerated | Whether the robot is a refrigerated unit | — | yes / no |
| batteryLevel | Current battery charge | % | 0 to 100 |
| status | Current operational status (enum `RobotStatus`) | — | `READY` (idle at a station), `CHARGING`, `MAINTENANCE`, `BUSY` (working on an order) |

**Business rules**
* Operates in exactly one shopping location.
* Is parked or charging in at most one robot station: `READY` and `CHARGING` robots are in a station, `BUSY` robots are not.
* It can be assigned a sub-order only if it is `READY`, the sub-order's total weight and volume do not exceed `maxWeightCapacity` and `maxVolumeCapacity`, and its battery is at least the required minimum.
* Among the robots that can carry a sub-order, the one with the highest `batteryLevel` is chosen; ties can be broken arbitrarily *(client, 2026-10-12)*.
* When assigned, its status changes from `READY` to `BUSY` and its station gets one more free spot.
* Its entries into and departures from stations are scheduled as station movements.
* Shows its status and next destination on an information screen, and its status on a light.

### RobotStation

Area where robots park when not in use, charge their batteries and undergo maintenance. It is a kind of [Area](#area).

| Attribute | Description | Units | Allowed values |
| :--- | :--- | :--- | :--- |
| stationId | Identification of the station | — | unique, mandatory |
| name | Name of the station | — | optional |
| maxCapacity | Maximum number of robots it can hold | robots | integer > 0 |

**Business rules**
* Has all the attributes and rules of an area.
* Is equipped with exactly one control terminal and schedules zero or more station movements.
* The number of parked robots never exceeds `maxCapacity`; occupied spots plus free spots equal `maxCapacity`.
* A station with parked `READY` or `CHARGING` robots cannot be removed; its robots must be moved first.

### Route

Planned sequence of stops a robot follows to collect the products of a sub-order and deliver them.

| Attribute | Description | Units | Allowed values |
| :--- | :--- | :--- | :--- |
| routeId | Identification of the route | — | unique, mandatory |
| plannedAt | When the route was planned | date-time | mandatory |

**Business rules**
* Followed by exactly one sub-order; a sub-order follows at most one route.
* Planned by exactly one system user with role `SYSTEM_PLANNER`; planning is done by the system with the operator's input, while the robot's physical navigation is out of scope *(client, 2026-10-02)*.
* Is a sequence of two or more stops.

### Shop

Shop of a shopping location that joined ShopRobot. It is a kind of [Area](#area).

| Attribute | Description | Units | Allowed values |
| :--- | :--- | :--- | :--- |
| shopCode | Code of the shop | — | unique; optional until assigned (shops imported from GeoJSON have none) |
| category | Kind of shop | — | e.g. `clothes`, `sports`, `bag`, `fashion_accessories`; the list is not closed |
| isOpen | Whether the shop is currently open | — | yes / no |

**Business rules**
* Has all the attributes and rules of an area.
* Managed by at most one system user with role `MERCHANT`: shops imported from GeoJSON have no merchant until one is assigned.
* Offers zero or more products.
* Is only available in ShopRobot if a user with role `RETAILER_ADMIN` defined it.
* When imported from GeoJSON it must have at least a name and a level; a shop without a level is skipped and a warning is logged *(client, 2026-10-02)*.
* Products of a closed shop cannot be ordered *(to confirm)*.

### ShoppingCart

Virtual cart where a customer gathers the products to buy before confirming the purchase.

| Attribute | Description | Units | Allowed values |
| :--- | :--- | :--- | :--- |
| cartId | Identification of the cart | — | unique, mandatory |
| createdAt | When the cart was created | date-time | mandatory |

**Business rules**
* Belongs to exactly one system user with role `CUSTOMER`; a customer has at most one cart.
* Contains zero or more cart items.
* Confirming the purchase creates an order, with its target delivery time and delivery spot.

### ShoppingLocation

Shopping centre, outlet or hypermarket where ShopRobot operates.

| Attribute | Description | Units | Allowed values |
| :--- | :--- | :--- | :--- |
| locationId | Identification of the shopping location | — | unique, mandatory |
| name | Name of the shopping location, e.g. "NorteShopping" | — | mandatory; unique |
| address | Postal address | — | mandatory |
| geoJsonData | GeoJSON layout from which its levels and areas are imported | — | valid GeoJSON; optional |

**Business rules**
* Owned by exactly one retailer company.
* Contains one or more levels.
* Zero or more robots operate in it.

### StationMovement

Scheduled entry of a robot into, or departure from, a robot station.

| Attribute | Description | Units | Allowed values |
| :--- | :--- | :--- | :--- |
| movementId | Identification of the movement | — | unique, mandatory |
| scheduledAt | When the movement is scheduled | date-time | mandatory |
| direction | Whether the robot enters or leaves (enum `MovementDirection`) | — | `ENTERING`, `LEAVING` |

**Business rules**
* Scheduled by exactly one robot station and moves exactly one robot.
* A robot has at most one movement scheduled for the same date-time.
* A robot can only enter a station that has a free spot *(to confirm)*.

### Stop

Point of a route where the robot picks up products or delivers the order.

| Attribute | Description | Units | Allowed values |
| :--- | :--- | :--- | :--- |
| sequenceNumber | Position of the stop in the route | — | integer > 0; unique within the route |
| stopType | What the robot does at the stop (enum `StopType`) | — | `PICKUP` (at a shop), `DELIVERY` (at the delivery spot) |
| plannedArrival | When the robot is expected at the stop | date-time | mandatory; not earlier than the previous stop's *(to confirm)* |

**Business rules**
* Belongs to exactly one route and takes place at exactly one area.

### SubOrder

Part of an order carried by one robot. Every order is partitioned into one or more sub-orders.

| Attribute | Description | Units | Allowed values |
| :--- | :--- | :--- | :--- |
| subOrderId | Identification of the sub-order | — | unique, mandatory |
| status | Stage of the sub-order (enum `SubOrderStatus`) | — | `PENDING`, `ASSIGNED`, `PICKING`, `IN_TRANSIT`, `DELIVERED` |

**Business rules**
* Belongs to exactly one order and bundles one or more of its order items.
* Assigned to at most one robot: none while `PENDING`, exactly one from `ASSIGNED` onwards.
* Each robot receives its own sub-order: the sub-orders of the same order are assigned to distinct robots *(client, 2026-10-12)*.
* A robot carries at most one active (`ASSIGNED`, `PICKING`, `IN_TRANSIT`) sub-order at a time *(to confirm)*.
* Its total weight and volume do not exceed the `maxWeightCapacity` and `maxVolumeCapacity` of its robot.
* Refrigerated items are never mixed with non-refrigerated items: they go in a separate sub-order carried by a refrigerated robot *(client, 2026-10-12)*.
* Follows at most one route.

### SystemUser

Person who uses ShopRobot. What the user can do depends on the role.

| Attribute | Description | Units | Allowed values |
| :--- | :--- | :--- | :--- |
| userId | Identification of the user | — | unique, mandatory |
| fullName | Name of the user | — | mandatory |
| email | Contact e-mail | — | valid e-mail address; unique |
| phoneNumber | Contact phone number | — | optional; 9 to 15 digits, optionally preceded by `+` |
| userRole | Role of the user (enum `UserRole`) | — | `CUSTOMER`, `MERCHANT`, `RETAILER_ADMIN`, `MAINTENANCE_OPERATOR`, `SYSTEM_PLANNER`, `SYSTEM_OPERATOR` |

**Business rules**
* Has exactly one role.
* `CUSTOMER`: places zero or more orders and fills at most one shopping cart.
* `MERCHANT`: manages zero or more shops and defines the products they offer through ShopRobot.
* `RETAILER_ADMIN`: works for exactly one retailer company (and only retailer administrators do); defines the shops available in the system, namely by importing the levels and shops of a shopping location from a GeoJSON file.
* `MAINTENANCE_OPERATOR`: operates zero or more control terminals and can change the status of a robot, for instance to `MAINTENANCE`.
* `SYSTEM_PLANNER`: plans zero or more routes; it is the "infrastructure planner" of the ESINF user stories *(client, 2026-10-02)*.

## ❓ Open Questions

Values and rules to confirm with the client; the answers go to the requirements document (USBD01) with their source.

* What is the minimum battery level a robot needs to be assigned to a sub-order? (The client said battery constraints are for future sprints.)
* Which states does an order go through, who changes each one, and how do they relate to the states of its sub-orders?
* Does an order keep a single target delivery time, or will it have a delivery time window (start and end), as the ESINF user stories mention? (Delivery scheduling is out of Sprint 1 scope.)
* Can a robot carry more than one active sub-order at the same time?
* What does a `SYSTEM_OPERATOR` do, and how is it different from a `SYSTEM_PLANNER`?
* Can a user have more than one role?
* Can a control terminal be operated by more than one maintenance operator?
* What does `heightMeters` measure, and what is a "designated waiting area"?
* What identifies a retailer company, a shopping location and an area in the business (VAT number, GeoJSON id, …)?
* Can a product be free (price 0), e.g. in a promotion?
* Should products that no robot can carry be rejected at registration already in Sprint 1?
