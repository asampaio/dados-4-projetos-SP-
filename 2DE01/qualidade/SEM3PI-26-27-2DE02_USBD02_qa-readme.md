# 🧪 QA & Test Documentation: [USBD02] Data Dictionary

* **User Story Reference:** [USBD02: Data Dictionary / Glossary](./readme.md)
* **QA Lead / Owner:** Tiago Marquez (1221508)
* **Artifact Under Review:** [doc/Glossaries/DBglossary/readme.md](../../Glossaries/DBglossary/readme.md)
* **QA Status:** 🟡 In Progress

---

## 🎯 QA Strategy & Scope

### In-Scope
* Consistency of the dictionary with the domain model (entities, attributes, enumerations, multiplicities).
* Completeness of each attribute row (description, units, allowed values).
* Business focus of the definitions (no generic definitions).
* Traceability of the assumptions marked *(to confirm)*.

### Out-of-Scope
* SQL data types, keys and constraints of the relational model (verified in USBD04).

---

## 🛠️ Review Setup

* **Domain model version:** [V2-domain-model.puml](../../domain-model/V2-domain-model.puml) at the commit under review.
* **Reference clarifications:** [Requirements Q&A log](../../requirements/requirements.md).

---

## 📋 Test Matrix

| Test ID | Title / Scenario | Type | Test Steps | Expected Result | Status | Execution Date | Issues Found |
| :--- | :--- | :--- | :--- | :--- | :---: | :---: | :--- |
| **TC-01** | Entities match the domain model | Consistency | Compare the classes of the domain model with the entity headings | Every class has an entry with the same name | ⏳ Pending | | |
| **TC-02** | Attributes match the domain model | Consistency | Compare the attributes of each class with its attribute table | Every attribute appears, with the same name | ⏳ Pending | | |
| **TC-03** | Enumerations match | Consistency | Compare `OrderStatus`, `SubOrderStatus`, `RobotStatus` and `UserRole` with the allowed values | Same values, same names | ⏳ Pending | | |
| **TC-04** | Attribute rows are complete | Completeness | Check every attribute row | Description, units (or `—`) and allowed values filled | ⏳ Pending | | |
| **TC-05** | Rules respect the multiplicities | Consistency | Compare the business rules with the multiplicities of the domain model | No rule contradicts a multiplicity | ⏳ Pending | | |
| **TC-06** | Assumptions are traceable | Traceability | Check every *(to confirm)* mark | Each one has a matching open question | ⏳ Pending | | |
| **TC-07** | Definitions are business-focused | Content | Read every definition | Definitions refer to the ShopRobot context, not generic meanings | ⏳ Pending | | |

### Execution Log

NOTE: add here evidences of the review, such as comments or screenshots.

---

## 🐞 Issues Found

| Issue ID | Summary / Description | Severity | Status | Linked Commit / Fix | Re-check Status |
| :--- | :--- | :--- | :--- | :--- | :---: |
| | | | | | |

---

## 📊 Summary & QA Sign-Off

### Execution Results
* **Total Executed:** 0
* **Passed:** 0
* **Failed:** 0

### Final QA Recommendation

> *(Pending review)*

* [ ] **QA Sign-off Approved**
* **Signed By:** Tiago Marquez (QA)
* **Date:**
