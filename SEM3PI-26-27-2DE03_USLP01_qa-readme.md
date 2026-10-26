# 🧪 QA & Test Documentation: [USLP01] Domain Model (Conceptual Level)

* **User Story Reference:** [USLP01: Domain Model](./readme.md)
* **QA Lead / Owner:** TBD (must be a team member other than the author, per section 3.4 of the requirements document; review requested)
* **Target Environment:** `PlantUML (source) + SVG (rendering), GitHub repository`
* **Build / Release Version:** Sprint 1
* **QA Status:** 🟡 Ready for review (diagram and documentation drafted; review requested, no test executed yet)

---

## 🎯 QA Strategy & Scope

### In-Scope
* Compliance of the diagram with the modeling guidelines (business concepts only, identities, value objects, associations, enumerations).
* Coverage of the business concepts of the requirements document and of the data needed by the Sprint 1 user stories.
* PlantUML source and SVG rendering available in the repository and in sync.
* Consistency between the diagram and the glossary ([USLP02]).

### Out-of-Scope
* Relational model and ER diagram ([USBD04]).
* Implementation classes, DAOs and user interface.
* Quality of the glossary wording (reviewed in [USLP02]).
* Route planning algorithm.

---

## 🛠️ Test Environment & Setup

* **Tooling:** PlantUML (local or IDE plugin) and an SVG viewer. The requirements document and the glossary are the references.
* **Test data:** worked scenarios taken from the requirements: a customer order with products from two stores, an order that exceeds the limits of one robot, an order mixing refrigerated and non-refrigerated products, a station with parked robots, and a level without a station.
* **Approach:** static review. Each test is a question answered by reading the diagram against the requirements, the modeling guidelines and the glossary. There is no code, so TDD (Red-Green-Refactor) does not apply.
* **Same artefact as [USBD03]:** the review of this model also serves the BDDAD story, so one execution of the checks below covers both.

---

## 📋 Test Matrix

| Test ID   | Title / Scenario                 | Type          | Steps                                                                                       | Expected Result                                                                          |   Status   | Executed By | Notes                                         |
|:----------|:---------------------------------|:--------------|:--------------------------------------------------------------------------------------------|:-----------------------------------------------------------------------------------------|:----------:|:-----------:|:----------------------------------------------|
| **TC-01** | PlantUML source renders          | Rendering     | Render `domain-model.puml` with PlantUML                                                    | SVG generated without syntax errors                                                      | ⚪ Not run |      —      | —                                             |
| **TC-02** | SVG in the repository is current | Deliverable   | Compare the committed SVG with a fresh render of the committed source                       | Both files exist and the SVG matches the source                                          | ⚪ Not run |      —      | Covers AC2 and AC3                            |
| **TC-03** | Only business concepts           | Guideline     | Review every class, enumeration and value object                                            | No technical classes (DAO, UI, database tables, applications)                            | ⚪ Not run |      —      | —                                             |
| **TC-04** | Identity on every entity         | Guideline     | Check each entity                                                                           | Each one has an identity attribute marked `{id}`; value objects and enums have none      | ⚪ Not run |      —      | Identity choices are proposals                |
| **TC-05** | Value objects                    | Guideline     | Look for attributes that hold structured data                                               | `Polygon`, `Coordinate` and `Address` are classes; no structured data left as an attribute | ⚪ Not run |      —      | Prices and emails kept as simple attributes   |
| **TC-06** | Associations                     | Guideline     | Check every association                                                                     | Direction, meaningful name and multiplicities on both ends; each label reads correctly   | ⚪ Not run |      —      | e.g., "Route fulfils Order"                   |
| **TC-07** | Enumerations                     | Guideline     | Check attributes with a fixed set of values                                                 | `RobotState` and `OrderStatus` are enumerations; area category stays free text           | ⚪ Not run |      —      | Category is free text by decision             |
| **TC-08** | Shopping location structure      | Coverage      | Compare with the layout description of the requirements                                     | Levels have areas with a polygon; a store is located in an area; an elevator is an area  | ⚪ Not run |      —      | —                                             |
| **TC-09** | Robots and stations              | Coverage      | Compare with the "Robots" section                                                           | Robot has id, state, max volume, max weight, refrigerated and battery; station has capacity and parks robots | ⚪ Not run |      —      | —                                             |
| **TC-10** | Orders                           | Coverage      | Follow a customer order through the model                                                   | Customer places an order with delivery time, delivery area and lines of products of stores | ⚪ Not run |      —      | —                                             |
| **TC-11** | Order split between robots       | Coverage      | Take an order above the volume or weight of one robot, mixing refrigeration, or above its battery | Sub-orders, each with one robot; a simple order has none; never both                | ⚪ Not run |      —      | Whole lines only (assumption)                 |
| **TC-12** | Sprint 1 user stories            | Traceability  | For USLP04, USLP06 and USEI01 to USEI05, list the data each one needs                      | Every item is in the model or derivable (e.g., free spots, order totals)                 | ⚪ Not run |      —      | Review again in Sprint 2                      |
| **TC-13** | Glossary consistency             | Consistency   | Compare the diagram and the glossary                                                        | Every class, enumeration and value object has an entry with the same name; rules match the multiplicities | ⚪ Not run |      —      | 20 elements (15 entities, 3 value objects, 2 enumerations) |
| **TC-14** | Exclusions documented            | Documentation | Check what is not in the model (payment, shopping cart, control terminal, stop, actors)     | Each exclusion is explained in the readme or in the glossary                             | ⚪ Not run |      —      | —                                             |

### Execution Log

NOTE: register here the outcome of each review once it is done.

---

## 🐞 Defect & Bug Tracking

| Issue ID | Summary / Description | Severity | Priority | Status | Linked Commit / Fix | Re-test Status |
|:---------|:----------------------|:---------|:---------|:-------|:--------------------|:--------------:|
| —        | None identified yet   | —        | —        | —      | —                   |       —        |

---

## 🤖 Automated Checks

There are no unit tests, because the deliverable is a diagram. Planned checks (none implemented yet), which could run in the pre-commit or in the nightly build:

| Planned Check                                                                              | Covers | Status     |
|:-------------------------------------------------------------------------------------------|:-------|:----------:|
| Run PlantUML in check-only mode on `domain-model.puml` and fail on syntax errors           | TC-01  | ⚪ Pending |
| Render the source again and warn if the committed SVG is outdated                          | TC-02  | ⚪ Pending |
| Compare the names of the classes in the source with the entries of the glossary            | TC-13  | ⚪ Pending |

---

## 📊 Summary & QA Sign-Off

### Execution Results
* **Total Tests:** 0 / 14
* **Passed:** 0
* **Failed:** 0
* **Not run:** 14

### Final QA Recommendation

> 🟡 **Ready for review:** the documentation is up to date and the review has been requested. QA sign-off must be given by a team member other than the author.

* [ ] **QA Sign-off Approved**
* **Signed By:** TBD
* **Date:** TBD