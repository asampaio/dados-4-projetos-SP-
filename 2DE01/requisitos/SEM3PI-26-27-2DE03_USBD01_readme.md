# [USBD01] Database Requirements Analysis

* **Priority:** High — foundation for USBD04 (relational model) and USBD05 (physical model)
* **Story Points:** TBD (to be defined in Sprint Planning)
* **Module / Component:** `All modules` / `Persistent Data Layer`
* **Status:** In Progress (Sprint 1 scope; will be updated as domain is refined in subsequent sprints)

---

## 👤 User Story Description

**As a** development team
**I want to** have a clear and validated database requirements document
**So that** the relational model (USBD04) and physical model (USBD05) are built on accurate, agreed-upon domain rules — and common errors such as mutable historical prices, modifiable confirmed orders, or insecure password storage are prevented from the outset

---

## 🚦 Definition of Ready (DoR)

* [x] **Clear User Story:** Story follows the standard *As a / I want to / So that* format.
* [x] **Acceptance Criteria Defined:** see below.
* [x] **Dependencies Identified:** see "Found Out Dependencies" below.
* [x] **Inputs/Outputs Mapped:** inputs are client sessions, professor directives, and enunciado; output is this document.
* [ ] **Estimation Complete:** story points not yet assigned by the team.
* [x] **Open Clarifications Resolved:** price-per-store vs. price-per-product scope still open (see Open Decisions below); all other critical rules confirmed.

---

## 💬 Customer Specifications & Clarifications

| # | Rule / Clarification | Source |
|:--|:---------------------|:-------|
| C-01 | Orders are **final once confirmed and paid**; no post-payment changes are in scope | Client session 2 Oct 2026 |
| C-02 | Oversized items (exceeding any robot's cargo capacity) must be **blocked at product registration**; if no capable robot is available at delivery time, log the error and notify the user | Client session 2 Oct 2026 |
| C-03 | Refrigerated products: robots are equipped to handle them; **refrigerated and non-refrigerated robots must not be mixed** for the same sub-order | Client session 12 Oct 2026 |
| C-04 | Robot selection criterion: **highest battery level**; ties are resolved arbitrarily | Client session 12 Oct 2026 |
| C-05 | Sub-orders are an **explicit concept**: each robot receives its own sub-order. Internal structure (one order with a list of robots vs. separate sub-order records) is an implementation decision | Client session 12 Oct 2026 |
| C-06 | Robot status must always be assigned; **"busy" (working on an order) is a valid and required status** | Client session 12 Oct 2026 |
| C-07 | GeoJSON import: **store name and level are mandatory**; a store without a level must be skipped and a warning logged to the user | Client session 2 Oct 2026 |
| C-08 | Sprint 1 database: **H2 in-memory** via JDBC; migration to Oracle is planned for a later sprint | Client session 12 Oct 2026 |
| C-09 | **Price history is mandatory** in any serious retail system. A product's price change must never affect past transactions. Violating this is considered an unacceptable error | Prof Angelo Martins email, 13 Oct 2026 |
| C-10 | **Plain-text password storage is strictly forbidden**. If passwords are stored, they must be hashed with a secure algorithm. The professor considers that this project probably does not need to store passwords | Prof Angelo Martins email, 13 Oct 2026 |

---

## ✅ Acceptance Criteria

* [ ] **AC1 — Entity Catalogue:** all persistent entities are identified with their attributes, primary keys, and data types.
* [ ] **AC2 — Relationships:** all relationships between entities are documented with cardinality and participation constraints.
* [ ] **AC3 — Business Rules:** all business rules with direct database impact (especially C-01, C-09, C-10) are documented and traceable to their source.
* [ ] **AC4 — Open Decisions:** all unresolved design choices that affect the schema are listed explicitly with their options and tradeoffs.
* [ ] **AC5 — Consistency with Domain Model:** entity names and attributes are aligned with USBD03 / LAPR3 US02 (domain model).

---

## 🔒 Non-Functional Requirements & Security

* **NFR-DB-01 (Price Integrity):** product price changes must be tracked with temporal validity. Confirmed order lines capture the price at creation time and this value is immutable. Violating either rule is considered unacceptable by the BDDAD lecturers.
* **NFR-DB-02 (Password Security):** if any actor (customer, operator) requires authentication, passwords must be stored as a cryptographic hash (e.g. bcrypt). Plain-text storage is strictly forbidden.
* **NFR-DB-03 (Order Immutability):** the database design must enforce that confirmed orders cannot be modified — either through application-level logic or database constraints (triggers / no UPDATE permission on confirmed rows).
* **NFR-DB-04 (Referential Integrity):** all foreign key relationships must be enforced; orphaned records are not acceptable.
* **NFR-DB-05 (Normalisation):** schema must be in at least 3NF; any BCNF deviations must be explicitly justified.

---

## 🔗 Found Out Dependencies

### Upstream Dependencies (Required before starting)
* **Domain Model (USBD03 / LAPR3 US02):** entity names, associations and cardinalities; this document is aligned with but does not replace the domain model.
* **Client sessions and enunciado:** source of all business rules documented here.

### Downstream Dependencies (Impacted by this story)
* **[USBD04](../USBD04/readme.md) Relational Model:** built directly from this document — any entity or rule missing here will be missing in the ERD.
* **[USBD05](../USBD05/readme.md) Physical Model (DDL):** SQL schema derived from USBD04.
* **All application modules:** entity names and constraints defined here must match the Java domain model used in ESINF and LAPR3.

---

## 🗃️ Entity Catalogue

### ShoppingCenter

Represents the mall building. Sprint 1 targets Norte Shopping; model supports multiple centres.

| Attribute | Type | Constraints | Notes |
|:----------|:-----|:------------|:------|
| shoppingCenterId | INTEGER | PK, NOT NULL, AUTO | Surrogate key |
| name | VARCHAR(100) | NOT NULL, UNIQUE | e.g. "Norte Shopping" |
| address | VARCHAR(255) | NULL | Optional for Sprint 1 |

---

### Level

A floor within a shopping centre. Imported from GeoJSON.

| Attribute | Type | Constraints | Notes |
|:----------|:-----|:------------|:------|
| levelId | INTEGER | PK, NOT NULL, AUTO | Surrogate key |
| shoppingCenterId | INTEGER | FK → ShoppingCenter, NOT NULL | |
| floorNumber | INTEGER | NOT NULL | e.g. 0 = ground floor |
| name | VARCHAR(100) | NULL | Descriptive label if present in GeoJSON |

> **Rule (C-07):** a `Store` without a `Level` assignment must be rejected on import.

---

### Store

A retail store located on a floor of a shopping centre. Imported from GeoJSON (USLP06).

| Attribute | Type | Constraints | Notes |
|:----------|:-----|:------------|:------|
| storeId | INTEGER | PK, NOT NULL, AUTO | Surrogate key |
| levelId | INTEGER | FK → Level, NOT NULL | Cannot be NULL — enforces C-07 |
| name | VARCHAR(150) | NOT NULL | Mandatory GeoJSON field |
| category | VARCHAR(100) | NULL | Optional — type of store |
| geoJsonGeometry | TEXT | NULL | Raw geometry string from GeoJSON if useful |

---

### Product

An item available for purchase, associated with a specific store. Physical dimensions are required to validate robot cargo capacity (C-02).

| Attribute | Type | Constraints | Notes |
|:----------|:-----|:------------|:------|
| productId | INTEGER | PK, NOT NULL, AUTO | Surrogate key |
| storeId | INTEGER | FK → Store, NOT NULL | Product belongs to one store |
| name | VARCHAR(150) | NOT NULL | |
| description | TEXT | NULL | |
| weightKg | DECIMAL(8,3) | NOT NULL, > 0 | Used for robot cargo validation |
| volumeLiters | DECIMAL(8,3) | NOT NULL, > 0 | Used for robot cargo validation |
| requiresRefrigeration | BOOLEAN | NOT NULL, DEFAULT FALSE | Drives robot type selection (C-03) |

> **Rule (C-02):** at registration, if a product's weight or volume exceeds the maximum capacity of all registered robots, the product must be blocked from orders and an error must be recorded.

---

### ProductPrice

Temporal price history for each product. A price record is never updated — changes are made by inserting a new record and closing the previous one.

| Attribute | Type | Constraints | Notes |
|:----------|:-----|:------------|:------|
| productPriceId | INTEGER | PK, NOT NULL, AUTO | Surrogate key |
| productId | INTEGER | FK → Product, NOT NULL | |
| price | DECIMAL(10,2) | NOT NULL, ≥ 0 | |
| validFrom | TIMESTAMP | NOT NULL | Instant the price became active |
| validTo | TIMESTAMP | NULL | NULL = currently active price |

> **Additional constraint:** `UNIQUE (productId, validFrom)` — prevents duplicate price records for the same start instant. A **partial unique index on `(productId) WHERE validTo IS NULL`** must ensure at most one active price record per product exists at any time.

> **Rule (C-09 / NFR-DB-01):** `validTo` of the previous record must be set to the same value as the `validFrom` of the new record. No two records for the same product may have `validTo = NULL` simultaneously — enforce with a partial unique index.  
> **Design constraint:** NEVER use `UPDATE ProductPrice SET price = X`. Always INSERT a new record.

---

### Customer

The person who places orders.

| Attribute | Type | Constraints | Notes |
|:----------|:-----|:------------|:------|
| customerId | INTEGER | PK, NOT NULL, AUTO | Surrogate key |
| name | VARCHAR(150) | NOT NULL | |
| email | VARCHAR(255) | NOT NULL, UNIQUE | Login identifier |
| phoneNumber | VARCHAR(30) | NULL | |
| passwordHash | VARCHAR(255) | NULL | Hashed only (bcrypt). NULL if auth not implemented (C-10) |

> **Rule (C-10):** if `passwordHash` is used, it must store only the output of a secure one-way hash function. Plain-text storage in any column is strictly forbidden.

---

### Order

A purchase order placed by a customer. Immutable after confirmation.

| Attribute | Type | Constraints | Notes |
|:----------|:-----|:------------|:------|
| orderId | INTEGER | PK, NOT NULL, AUTO | Surrogate key |
| customerId | INTEGER | FK → Customer, NOT NULL | |
| createdAt | TIMESTAMP | NOT NULL | Moment the order was created |
| confirmedAt | TIMESTAMP | NULL | Set when payment is registered; after this, immutable |
| status | VARCHAR(30) | NOT NULL | Values: PENDING, CONFIRMED, IN_PROGRESS, COMPLETED, CANCELLED |

> **Rule (C-01 / NFR-DB-03):** once `confirmedAt` is set, no UPDATE to this row or its `OrderLine` records is permitted. Enforcement may be at the application layer, via database triggers, or both.

---

### OrderLine

An individual product within an order. Captures the price at creation time — this value is permanent.

| Attribute | Type | Constraints | Notes |
|:----------|:-----|:------------|:------|
| orderLineId | INTEGER | PK, NOT NULL, AUTO | Surrogate key |
| orderId | INTEGER | FK → Order, NOT NULL | |
| productId | INTEGER | FK → Product, NOT NULL | |
| quantity | INTEGER | NOT NULL, > 0 | |
| unitPriceAtOrder | DECIMAL(10,2) | NOT NULL, ≥ 0 | **Snapshot of price at line creation — never updated** |

> **Rule (C-09 / NFR-DB-01):** `unitPriceAtOrder` must be populated from `ProductPrice` (the active record at `createdAt`) at the moment the line is created and must never be recalculated from the current product price. Any query that derives order totals must use `unitPriceAtOrder`, never `ProductPrice.price`.

---

### Robot

A delivery robot in the system.

| Attribute | Type | Constraints | Notes |
|:----------|:-----|:------------|:------|
| robotId | INTEGER | PK, NOT NULL, AUTO | Surrogate key |
| label | VARCHAR(50) | NOT NULL, UNIQUE | Human-readable name (e.g. "R-07") |
| batteryLevel | INTEGER | NOT NULL, CHECK (batteryLevel BETWEEN 0 AND 100) | Updated in real time; used for selection (C-04) |
| status | VARCHAR(30) | NOT NULL | Values: AVAILABLE, BUSY, MAINTENANCE, OFFLINE (C-06) |
| isRefrigerated | BOOLEAN | NOT NULL, DEFAULT FALSE | TRUE = can carry refrigerated products (C-03) |
| maxCargoWeightKg | DECIMAL(8,3) | NOT NULL, > 0 | Maximum weight per delivery |
| maxCargoVolumeLiters | DECIMAL(8,3) | NOT NULL, > 0 | Maximum volume per delivery |

> **Rule (C-06):** a robot without a status is invalid — `status` is NOT NULL and must always be set.  
> **Rule (C-03):** a sub-order containing a product with `requiresRefrigeration = TRUE` may only be assigned to a robot with `isRefrigerated = TRUE`.  
> **Rule (C-04):** robot selection picks the robot with the highest `batteryLevel`; ties are resolved arbitrarily.

---

### SubOrder

A portion of an `Order` assigned to a single robot. Created when an order is dispatched (USEI04). Each robot handles one sub-order.

| Attribute | Type | Constraints | Notes |
|:----------|:-----|:------------|:------|
| subOrderId | INTEGER | PK, NOT NULL, AUTO | Surrogate key |
| orderId | INTEGER | FK → Order, NOT NULL | Parent order |
| robotId | INTEGER | FK → Robot, NULL | NULL until a robot is assigned |
| status | VARCHAR(30) | NOT NULL | Values: PENDING_ASSIGNMENT, ASSIGNED, IN_DELIVERY, COMPLETED |
| assignedAt | TIMESTAMP | NULL | Set when robotId is populated |

> **Rule (C-05):** the relationship between SubOrder and OrderLine (which lines / quantities go to which sub-order) is modelled via `SubOrderLine` (see below). Whether order lines can be split across sub-orders is an open decision.

---

### SubOrderLine *(tentative — Open Decision #3)*

Resolves the many-to-many between `SubOrder` and `OrderLine`. Required if a single `OrderLine` can be split across multiple robots (e.g. 5 units: 3 to robot A, 2 to robot B).

| Attribute | Type | Constraints | Notes |
|:----------|:-----|:------------|:------|
| subOrderId | INTEGER | PK (composite), FK → SubOrder | |
| orderLineId | INTEGER | PK (composite), FK → OrderLine | |
| quantity | INTEGER | NOT NULL, > 0 | Portion of the OrderLine assigned to this sub-order |

> **Constraint:** `SUM(SubOrderLine.quantity)` for a given `orderLineId` must not exceed `OrderLine.quantity`.  
> If order lines are never split (each line goes entirely to one robot), this entity collapses to a simple FK on `SubOrder` pointing to `OrderLine`. Decision pending (see Open Decisions).

---

## 🔗 Relationship Summary

| Relationship | Cardinality | Notes |
|:-------------|:------------|:------|
| ShoppingCenter → Level | 1 : N | A mall has many floors |
| Level → Store | 1 : N | A floor has many stores |
| Store → Product | 1 : N | A store has many products |
| Product → ProductPrice | 1 : N | Full temporal price history |
| Customer → Order | 1 : N | A customer places many orders |
| Order → OrderLine | 1 : N | An order has multiple lines |
| OrderLine → Product | N : 1 | Each line references one product |
| Order → SubOrder | 1 : N | An order splits into sub-orders |
| Robot → SubOrder | 1 : N (optional) | A robot handles many sub-orders over time; one at a time (BUSY status enforces this) |
| SubOrder ↔ OrderLine | M : N via SubOrderLine | Tentative — see Open Decision #3 |

---

## 📐 Business Rules & Constraints Summary

| ID | Rule | Enforcement | Source |
|:---|:-----|:------------|:-------|
| BR-01 | Product price changes never affect confirmed order lines | `unitPriceAtOrder` is set once and never updated | Prof email 13 Oct |
| BR-02 | Price history is always maintained for products | INSERT-only `ProductPrice`; UNIQUE partial index on `(productId, validTo IS NULL)` | Prof email 13 Oct |
| BR-03 | Orders are immutable after confirmation | No UPDATE on confirmed `Order` or its `OrderLine` rows | Client 2 Oct |
| BR-04 | Refrigerated products → refrigerated robots only | `Robot.isRefrigerated` must be TRUE if any `SubOrderLine.Product.requiresRefrigeration = TRUE` | Client 12 Oct |
| BR-05 | Robot selection: highest battery level | Application logic (USEI04); battery level stored in DB | Client 12 Oct |
| BR-06 | Oversized products blocked at registration | Application-level validation against `Robot.maxCargoWeightKg` and `maxCargoVolumeLiters` | Client 2 Oct |
| BR-07 | Store must have a Level | `Store.levelId NOT NULL` + FK constraint | Client 2 Oct |
| BR-08 | Robot always has a status | `Robot.status NOT NULL` | Client 12 Oct |
| BR-09 | No plain-text passwords | `passwordHash` only; plain-text columns forbidden | Prof email 13 Oct |
| BR-10 | Sub-order quantities ≤ order line quantities | **Cannot be enforced by a simple column constraint** — requires a database trigger (`AFTER INSERT/UPDATE ON SubOrderLine`) or application-layer validation before any insert/update on `SubOrderLine` | Domain logic (C-05) |

---

## 🔓 Open Decisions

These must be resolved before USBD04 is finalised.

| # | Question | Option A | Option B | Impact |
|:--|:---------|:---------|:---------|:-------|
| OD-1 | Is price **global per product** or **per (store, product)**? | `ProductPrice(productId, price, validFrom, validTo)` — same price everywhere | `ProductPrice(storeId, productId, price, validFrom, validTo)` — store sets its own price | Affects `ProductPrice` PK and `OrderLine.unitPriceAtOrder` derivation |
| OD-2 | Does `Customer` need authentication (passwordHash)? | Yes — include `passwordHash VARCHAR(255)` (hashed only) | No — remove the column entirely for Sprint 1 | Affects `Customer` schema; prof recommends skipping |
| OD-3 | Can an `OrderLine` be split across multiple robots? | Yes — `SubOrderLine` with `quantity`; SUM constraint enforced | No — each `OrderLine` goes entirely to one `SubOrder`; `SubOrderLine` collapses to FK | Affects `SubOrderLine` design and complexity |
| OD-4 | How is "current price" queried efficiently? | `WHERE validTo IS NULL` — partial unique index | Separate `Product.currentPrice` column (denormalised, updated on price change) | Performance vs. normalisation tradeoff; Option A is safer |
| OD-5 | Does a `Product` belong to exactly **one** store, or can the same product be sold by multiple stores? | `Product.storeId` FK — one product, one store (current assumption) | `StoreProduct (storeId, productId)` bridge table — product can appear in multiple stores at different prices | **Critical — affects entire schema.** If Option B, `OrderLine` must reference `(storeId, productId)` and `ProductPrice` must include `storeId`. Decision needed before USBD04 |

---

## 📊 Data Volume (Sprint 1 estimates)

| Entity | Approximate scale | Notes |
|:-------|:------------------|:------|
| ShoppingCenter | 1 | Norte Shopping only in Sprint 1 |
| Level | ~5 | Typical mall floor count |
| Store | ~50–200 | From GeoJSON; Norte Shopping has ~160 stores |
| Product | ~hundreds | One CSV per store data file |
| ProductPrice | ~same as Product | One initial record per product |
| Customer | ~tens | Test data only in Sprint 1 |
| Order | ~hundreds | Medium/large test datasets from ESINF |
| OrderLine | ~thousands | Multiple lines per order |
| Robot | ~tens | From ESINF data files |
| SubOrder | ~same as Order | One per robot per order |

---

## 🧪 Functional Tests / Validation

| Scenario ID | Scenario | Expected Result |
|:------------|:---------|:----------------|
| **F-01** | Insert a product, confirm an order with that product, change the product price | Confirmed order line `unitPriceAtOrder` remains unchanged; new `ProductPrice` record created |
| **F-02** | Query total value of a confirmed order after a price change | Total calculated from `unitPriceAtOrder` — matches original order value |
| **F-03** | Import GeoJSON store with no level | Store skipped; warning logged; other stores imported normally |
| **F-04** | Register a product heavier than all robot capacities | Product registration blocked; error recorded |
| **F-05** | Assign a refrigerated product's sub-order to a non-refrigerated robot | Assignment rejected (BR-04) |
| **F-06** | Two robots with the same battery level available | Either robot selected; no exception thrown |

---

## 🧩 Related Modules & Classes

* **Domain model diagram:** see USBD03 / LAPR3 US02 (to be kept in sync with this document).
* **Java domain classes** (ESINF / LAPR3): `Order`, `OrderLine`, `Product`, `Robot`, `Store`, `Level` — attribute names and types must match this catalogue.
* **USBD04:** relational model derived from this document — ERD in 3NF / BCNF.
* **USBD05:** physical SQL DDL; includes `CREATE TABLE` statements, indexes, constraints.

---

## 🏁 Definition of Done (DoD)

* [ ] All entities defined with attributes, PKs, FKs, and types.
* [ ] All business rules documented with source traceability.
* [ ] Open decisions listed explicitly (none silently resolved without team agreement).
* [ ] Consistent with domain model (USBD03 / LAPR3 US02).
* [ ] Reviewed by at least one team member other than the author.
* [ ] Updated whenever a client session or professor directive introduces a new constraint.