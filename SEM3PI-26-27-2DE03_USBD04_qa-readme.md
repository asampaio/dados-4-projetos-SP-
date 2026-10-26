# 🧪 QA & Test Documentation: [USBD04] Relational Model (Logical Level)

* **User Story Reference:** [US-USBD04: Relational Model (Logical Level)](./readme.md)
* **QA Lead / Owner:** *(TBD – must be someone other than the author)*
* **Target Environment:** `Oracle FreeSQL`
* **Build / Release Version:** Sprint 1
* **QA Status:** 🟡 Ready for review (model complete, no check executed yet)

---

## 🎯 QA Strategy & Scope

### In-Scope
* Coverage of the project requirements and user stories by the model.
* Consistency between relational model, conceptual model and glossary.
* Presence and correctness of all constraints (PK, FK, UK, NOT NULL, CHECK).
* Normalization review (3NF) and naming conventions.
* Price history (a price change never alters past transactions).
* Executability of the DDL script generated from the model in Oracle.
* ER diagram delivered as PlantUML source and SVG.

### Out-of-Scope
* Population of the database with test data (covered by [USBD05]).
* Performance tuning, indexes and physical storage.
* Requirements of Sprint 2 and 3 (not yet published).

---

## 🛠️ Test Environment & Setup

* **App URL / Endpoint:** Oracle FreeSQL (`https://freesql.com`)
* **Test Accounts Used:**
  * Personal Oracle FreeSQL account
* **Pre-conditions & Test Data:**
  * Conceptual model and glossary available (`doc/domain-model/`, `doc/glossary/`)
  * ER diagram (`doc/relational-model/relational-model.puml` and `.svg`) committed, with the constraints in its legend and notes
  * DDL script generated from the model ([USBD05])

---

## 📋 Test Matrix

| Test ID | Title / Scenario | Type | Pre-conditions | Test Steps | Expected Result | Status | Execution Date | Bugs Filed |
| :--- | :--- | :--- | :--- | :--- | :--- | :---: | :---: | :--- |
| **TC-01** | Requirements coverage | Review | Requirements document available | 1. List all requirements/entities in the project description<br>2. Trace each to a table/attribute/relationship | No requirement without model counterpart | ⚪ NOT RUN | — | — |
| **TC-02** | Consistency with glossary and conceptual model | Review | Glossary and domain model available | 1. Compare names and concepts | Same concepts and terminology; no missing or extra entities | ⚪ NOT RUN | — | — |
| **TC-03** | Primary keys | Review | Model completed | 1. Inspect every table | Every table has a PK | ⚪ NOT RUN | — | — |
| **TC-04** | Foreign keys and cardinalities | Review | Model completed | 1. Inspect every relationship in the ER diagram | Each relationship has the correct FK and nullability | ⚪ NOT RUN | — | — |
| **TC-05** | Business-rule constraints | Review | Constraints in the diagram | 1. Check CHECK/UK/NOT NULL (battery 0–100, capacities > 0, quantity > 0, unique level per location, etc.) | All documented rules are present | ⚪ NOT RUN | — | — |
| **TC-06** | Normalization | Review | Model completed | 1. Analyze functional dependencies per table | All tables in 3NF (or justified exception) | ⚪ NOT RUN | — | — |
| **TC-07** | DDL execution | Functional | DDL script generated | 1. Run script in Oracle FreeSQL | Script runs without errors; all tables and constraints created | ⚪ NOT RUN | — | — |
| **TC-08** | Constraint enforcement | Negative | Schema created | 1. Insert robot with battery = 150<br>2. Insert row with non-existent FK<br>3. Insert duplicate PK | All three inserts rejected with the proper constraint error | ⚪ NOT RUN | — | — |
| **TC-09** | Diagram deliverables | Documentation | Files committed | 1. Open `.puml`<br>2. Render and compare with `.svg` | SVG matches the source and renders correctly | ⚪ NOT RUN | — | — |
| **TC-10** | Price history | Business rule | Model completed | 1. Check that `PRODUCT_PRICE` has the product and `valid_from` as key and no `PRODUCT.price` column exists<br>2. Check that `ORDER_LINE` has its own `unit_price` | A price change is a new row and past order lines keep their price (BDDAD teacher rule) | ⚪ NOT RUN | — | — |
| **TC-11** | No open decisions | Review | Model completed | 1. Search the `.puml` for "pending" and "to confirm" | No open decision left in the model | ⚪ NOT RUN | — | — |

### Execution Log

NOTE: add here evidences of the tests, such as logs or screenshots (e.g. Oracle execution output, review checklist).

---

## 🐞 Defect & Bug Tracking

| Bug ID | Summary / Description | Severity | Priority | Status | Linked Commit / Fix | Re-test Status |
| :--- | :--- | :--- | :--- | :--- | :--- | :---: |
| *(none registered yet)* | | | | | | |

---

## 🤖 Unit Tests

* **Related App:** Not applicable (modelling/database story)
* **Related Code:** DDL script generated from the relational model
* **Automated Tests:** *(optional)* SQL script with negative inserts (TC-08)
* **Execution:** Manual run on Oracle FreeSQL
* **CI Build Link:** N/A

---

## 📊 Summary & QA Sign-Off

### Execution Results
* **Total Executed:** 0 / 11
* **Passed:** 0
* **Failed:** 0
* **Blocked:** 0

### Final QA Recommendation

> 🟡 **Ready for review:** the model is complete and the review has been requested. QA must run the test matrix and sign off, and the reviewer must be a team member other than the author.

* [ ] **QA Sign-off Approved**
* **Signed By:** *(name, role)*
* **Date:** *(pending)*