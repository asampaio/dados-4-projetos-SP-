# [USBD02] Data Dictionary / Glossary

* **Priority:** Must
* **Story Points:** 3
* **Module / Component:** BDDAD / Data Documentation
* **Status:** In Progress
* **Developer:** Tiago Borges (1231109)
* **QA:** Tiago Marquez (1221508)

---

## 👤 User Story Description

**As a** Product Owner
**I want** a data dictionary/glossary to be created
**So that** everyone working on the database shares the same meaning, units and allowed values for the data of ShopRobot

---

## 🚦 Definition of Ready (DoR)

* [x] **Clear User Story:** Story follows the standard *As a / I want to / So that* format.
* [x] **Acceptance Criteria Defined:** ACs below, based on the project statement (USLP02) and the BDDAD clarifications.
* [x] **Dependencies Identified:** Domain model (USLP01/USBD03) available in `doc/domain-model`.
* [x] **Inputs/Outputs Mapped:** Inputs and output documented below.
* [x] **Estimation Complete:** 3 story points (Sprint 1 planning).
* [x] **Small & Actionable:** One document, completed within Sprint 1.

---

## 💬 Customer Specifications & Clarifications

* **Project statement (v1, Section 7.3.1):** USBD02 refers to USLP02 — the glossary must be consistent with the conceptual model, focused on business concepts and without generic (Wikipedia-like) definitions.
* **Clarification (2026-10-02, BDDAD professor):** in BDDAD, the glossary must include the business constraints of each concept.
* **Clarification (2026-10-07, Moodle BDDAD, Prof. Angelo Martins):** a simple tabular format following the Harvard Medical School data dictionary template is acceptable (see the [requirements Q&A log](../../requirements/requirements.md)).
* **Out of Scope:** SQL data types, keys and `NOT NULL`/`CHECK` constraints of the relational model (USBD04).

---

## ✅ Acceptance Criteria

* [x] **AC1 - Consistent with the domain model:** Every entity, attribute and enumeration of the domain model is described, with the same names.
* [x] **AC2 - Business focus:** Definitions explain the concepts in the ShopRobot context, not generic definitions.
* [x] **AC3 - Attribute details:** Each attribute has a description, units (or `—`) and allowed values.
* [x] **AC4 - Business constraints:** Each entity lists the business rules that involve more than one attribute or entity, consistent with the multiplicities of the domain model.
* [x] **AC5 - Traceability of assumptions:** Values and rules not stated by the client are marked *(to confirm)* and listed as open questions.

---

## 🔒 Non-Functional Requirements

* **NFR01 - Business rules:** The dictionary is the reference for the business rules validated when data is registered or updated.
* **NFR02 - Database:** The allowed values and rules support the data integrity of the relational database.
* **NFR05 - Documentation:** Written in Markdown in the repository, following the project templates.

---

## 🔗 Found Out Dependencies

### Upstream Dependencies (Required before starting)
* **[USLP01 / USBD03] Domain model:** the dictionary follows its entities, attributes, enumerations and multiplicities.

### Downstream Dependencies (Impacted by this story)
* **[USBD04] Relational model:** turns the allowed values and business rules into keys and constraints.
* **[USBD05] Physical model and data:** inserted data must respect the allowed values.
* **[USBD01] Requirements document:** receives the open questions and the client's answers.
* **[USLP02] Glossary:** definitions of the same concepts must not contradict each other.

---

## 📥 Input & Output Data

### Inputs
| Input | Source |
| :--- | :--- |
| Domain model | [V2-domain-model.puml](../../domain-model/V2-domain-model.puml) |
| Project statement | Project Specification v1 (sections 4 and 7) |
| Client clarifications | [Requirements Q&A log](../../requirements/requirements.md) |

### Outputs
| Output | Location |
| :--- | :--- |
| Data dictionary | [doc/Glossaries/DBglossary/readme.md](../../Glossaries/DBglossary/readme.md) |

---

## Edge Cases

* **Domain model changes:** the dictionary must be updated whenever the domain model changes (done on 2026-10-08, after the new domain model).
* **Information not given by the client:** the value or rule is marked *(to confirm)* and added to the open questions, instead of being invented.
* **Conflicting sources:** e.g. the ESINF user stories mention delivery *time windows*, while the domain model has a single `targetDeliveryTime`; registered as an open question.

---

## 🧪 Functional Tests / Validation

| Scenario ID | Check | Expected Result |
| :--- | :--- | :--- |
| **TC-01** | Compare the classes of the domain model with the entities of the dictionary | Every class has an entry with the same name |
| **TC-02** | Compare the attributes of each class with the attribute tables | Every attribute appears, with the same name |
| **TC-03** | Compare the enumerations with the allowed values of the status and role attributes | Same values, same names |
| **TC-04** | Check each attribute row | Description, units (or `—`) and allowed values filled |
| **TC-05** | Compare the business rules with the multiplicities of the domain model | No rule contradicts a multiplicity |
| **TC-06** | Check every *(to confirm)* mark | Each one has a matching open question |

---

# 🧪 QA & Test Documentation

See Report at [QA & Test Documentation: USBD02 Data Dictionary](./qa-readme.md)

---

## 🧩 Related Artifacts

* **Deliverable:** [Data Dictionary](../../Glossaries/DBglossary/readme.md)
* **Domain model:** [V2-domain-model.svg](../../domain-model/V2-domain-model.svg)
* **Relational model (USBD04):** [USBD04_ShopRobot_Relational_Model.svg](../../DataBase/Relational-Model/USBD04_ShopRobot_Relational_Model.svg)
* **Glossary (USLP02):** [Project Glossary](../../Glossaries/glossary/readme.md)

---

## 🔄 Sequence Diagram

Not applicable: this user story produces documentation, not application behaviour.

---

## 🏁 Definition of Done (DoD)

* [x] Dictionary consistent with the current domain model.
* [x] Open questions listed in the dictionary.
* [ ] Open questions registered in the requirements document (USBD01).
* [ ] Acceptance criteria verified by QA.
