# [USBD03] Conceptual Domain Model

* **Priority:** High
* **Story Points:** 2 (size S, as estimated on the project board)
* **Module / Component:** `Documentation` / `Domain Model`
* **Status:** In Review (model and glossary drafted; review and sign-off pending)

---

## 👤 User Story Description

**As a** Product Owner
**I want to** have the domain model created at the conceptual level
**So that** all stakeholders share a consistent understanding of the business domain throughout the project

> Equivalent deliverable to [USLP01](../../LAPR3/USLP01/readme.md) — the requirements document explicitly states "Please refer to USLP01" for this story. Same artefact, counted for both LAPR3 and BDDAD. As agreed with the client, it is treated as a single artefact: the diagram and the glossary are shared, and the documentation of [USLP01](../../LAPR3/USLP01/readme.md) describes the same model.

---

## 🚦 Definition of Ready (DoR)

* [x] **Clear User Story:** Story follows the standard *As a / I want to / So that* format.
* [x] **Acceptance Criteria Defined:** Taken directly from the project requirements document.
* [x] **Dependencies Identified:** Must stay consistent with the Glossary (USBD02/USLP02) at all times.
* [x] **Inputs/Outputs Mapped:** Not applicable — this is a documentation deliverable, not a data-processing feature.
* [x] **Estimation Complete:** 2 story points (size S), as estimated on the project board.
* [x] **Small & Actionable:** Initial version scoped to Sprint 1; model evolves in later sprints as new requirements emerge.

---

## 💬 Customer Specifications & Clarifications

* **Clarification (enunciado, USLP01):** The domain model is a "dynamic" document — it must continuously reflect the stakeholders' shared understanding of the domain and be updated whenever the domain evolves.
* **Clarification (enunciado, USLP01):** What is expected is the source of the diagram in PlantUML **and** its rendering as an SVG file, both committed to the GitHub repository.
* **Clarification (doc/readme.md):** The domain model should include only business/domain concepts; all entities must have an identity attribute from the business domain; "complex" attributes should be modelled as classes (value objects); associations must include direction, business-meaningful names, and multiplicities; enumerations should be included where necessary; meaningful business methods may also be included.
* **Clarification (client sessions):** Refrigerated robots carry only refrigerated products and non-refrigerated robots only non-refrigerated ones; there are no mixed robots.
* **Clarification (client sessions):** A robot always has a state, and a "busy" state (working on an order) is allowed. The team keeps `IN_USE` for it.
* **Clarification (client sessions):** Orders are final once confirmed and paid. Payment is not modelled (see the assumption in [USLP01](../../LAPR3/USLP01/readme.md)).
* **Clarification (client sessions):** The domain model is a priority: main entities and attributes, relationships, multiplicities and meaningful association names.
* **Out of Scope:** Physical or logical database schema — those belong to USBD04 and USBD05. Technical infrastructure and implementation classes.

---

## ✅ Acceptance Criteria

* [x] **AC1 - Coverage:** The data model covers all the business/domain aspects presented and discussed in the context of the requirements document.
* [x] **AC2 - Repository Artefact:** The PlantUML source (`domain-model.puml`) and its SVG rendering (`domain-model.svg`) are both committed to the GitHub repository at `doc/domain-model/`.

---

## 🔒 Non-Functional Requirements & Security

* **NFR05 (Documentation):** Must follow team documentation standards; all diagrams must be recorded in SVG format.
* **Not applicable:** Performance, security and authorisation NFRs do not apply — this is a static documentation deliverable, not a running feature.

---

## 🔗 Found Out Dependencies

### Upstream Dependencies (Required before starting)
* **Project Enunciado (sem3-integrative-project-26-27-v1.pdf):** Primary source of domain knowledge; all entities must be traceable to it.
* **Sprint Planning and client clarifications:** Any new business concept arising from questions or discussions must be reflected in the model before the sprint closes.

### Downstream Dependencies (Impacted by this story)
* **[USBD02]/[USLP02] Glossary:** Must always be in sync with this model — every entity in the model must have a glossary entry and vice versa.
* **[USBD04] Relational Model:** The logical model must be derived from this conceptual model.
* **[USBD05] Physical Model:** The physical Oracle instantiation must implement what is represented here.
* **[USEI04](../../ESINF/USEI04/readme.md) and all ESINF stories:** Class names, attributes and relationships used in the Java implementation must follow this model.

---

## 📥 Input & Output Data

Not applicable. This story produces a documentation artefact (PlantUML source + SVG), not a runtime data flow.

---

## Edge Cases

* **New entities discovered mid-sprint:** Any entity that emerges from clarifications or team decisions must be added to both the model and the glossary before the sprint closes.
* **Terminology disagreements:** If a class name differs between the model and the glossary, the discrepancy must be resolved and both artefacts updated together.
* **Derived attributes:** Attributes that can be computed from others (e.g., free spots of a robot station = capacity − parked robots count) are not stored; this is reflected by their absence from the model.
* **Actors not modelled as classes:** System Operator, System Planner, Maintenance Operator, and Public are intentionally excluded from the class diagram and documented only in the Glossary (under "Other Actors").

---

## 🧪 Functional Tests / Validation

Since this is a documentation deliverable, "testing" here means a **review checklist** rather than functional test cases:

| Scenario ID | Review Check                                                                            | Expected Result                                                     |
|:------------|:----------------------------------------------------------------------------------------|:--------------------------------------------------------------------|
| **RC-01**   | All business entities from the enunciado are represented in the model                   | No entity missing; no entity added without a business justification |
| **RC-02**   | Every entity in the model has a matching entry in the Glossary (USBD02)                 | No entity left undocumented in the glossary                         |
| **RC-03**   | All associations have multiplicities, direction and a business-meaningful name          | No association is anonymous or direction-less                       |
| **RC-04**   | Enumerations (`RobotState`, `OrderStatus`) are correct and consistent with the glossary | Values match the agreed list in the glossary                        |
| **RC-05**   | PlantUML source renders without errors and the SVG is up to date                        | No rendering errors; committed SVG reflects the current source      |
| **RC-06**   | Model is consistent with the requirements document (USBD01)                             | No entity or relationship contradicts a stated requirement          |

---

# 🧪 QA & Test Documentation

See Report at [QA & Test Documentation: USBD03 Conceptual Domain Model](qa-readme.md)

---

## 🧩 Related Modules & Classes

Not applicable — no code module. The artefacts are:
* `../../domain-model/domain-model.puml` — PlantUML source
* `../../domain-model/domain-model.svg` — rendered SVG

---

## 🔄 Sequence Diagram

Not applicable. This story has no runtime interaction/flow to diagram.

---

## 🏁 Definition of Done (DoD)

* [x] Acceptance criteria verified against the requirements document.
* [ ] Reviewed and approved by a team member other than the author (Hugo Coutinho).
* [x] Consistent with the current state of the Glossary (USBD02/USLP02).
* [ ] Final sign-off recorded in `qa-readme.md`.
* [ ] Model updated at the end of every sprint when new requirements emerge.