# [USBD02] Project Glossary / Data Dictionary

* **Priority:** High
* **Story Points:** TBD (a definir pela equipa no Sprint Planning)
* **Module / Component:** `Documentation` / `Glossary`
* **Status:** In Progress

---

## 👤 User Story Description

**As a** Product Owner
**I want to** have a data dictionary/glossary created
**So that** all team members and course units share a consistent understanding of business concepts throughout the project

> Equivalent deliverable to [USLP02] — the requirements document explicitly states "Please refer to USLP02" for this story. Same artifact, counted for both LAPR3 and BDDAD.

---

## 🚦 Definition of Ready (DoR)

* [x] **Clear User Story:** Story follows the standard *As a / I want to / So that* format.
* [x] **Acceptance Criteria Defined:** Taken directly from the project requirements document.
* [x] **Dependencies Identified:** Must stay consistent with the Domain Model (USBD03/USLP01).
* [x] **Inputs/Outputs Mapped:** Not applicable, this is a documentation deliverable, not a data-processing feature.
* [ ] **Estimation Complete:** Story points not yet assigned by the team.
* [x] **Small & Actionable:** Scoped to Sprint 1.

---

## 💬 Customer Specifications & Clarifications

* **Clarification (requirements doc, USLP02):** The glossary should be consistent with the Conceptual Model Diagram.
* **Clarification (requirements doc, USLP02):** Focus should be on business concepts — avoid generic, Wikipedia-like definitions.
* **Out of Scope:** Technical/implementation terms (e.g., database column names, API endpoints) — those belong in code documentation, not here.

---

## ✅ Acceptance Criteria

* [x] **AC1 - Consistency:** The Glossary is consistent with the Conceptual Model Diagram.
* [x] **AC2 - Business Focus:** The focus is on business concepts, not technical implementation details.
* [x] **AC3 - Relevant Definitions:** Clarifications/definitions are relevant for problem interpretation and avoid generic ("Wikipedia-like") definitions.

---

## 🔒 Non-Functional Requirements & Security

* **NFR05 (Documentation):** Must follow team documentation standards; all diagrams referenced must be in SVG format.
* **Not applicable:** Performance, security and authorization NFRs do not apply — this is a static document, not a running feature.

---

## 🔗 Found Out Dependencies

### Upstream Dependencies (Required before starting)
* **[USBD03]/[USLP01] Domain Model:** The glossary's terms and relationships must match the entities in the conceptual domain model.

### Downstream Dependencies (Impacted by this story)
* **All future documentation and code (all course units):** Class names, attribute names and business terminology used across the project should follow this glossary, to keep the whole team using the same vocabulary.

---

## 📥 Input & Output Data

Not applicable. This story produces a documentation artifact (Markdown file), not a runtime data flow.

---

## Edge Cases

* **Terms not yet confirmed with the client:** the `Order Status` values, the `Robot State` names (`READY`/`IN_USE` vs. idle/`BUSY`) and the minimum battery rule are flagged in the glossary as assumptions, pending validation.
* **Terms reserved for a future sprint:** `Route` and `Stop` are documented now (to avoid terminology drift later) but are out of scope for Sprint 1. They belong to the future Planning App (ESINF) work.
* **Terms pending team agreement:** `Retailer Administrator`, `Shopping Cart`, `Route` and `Stop` were added based on the requirements document but have not yet been formally reviewed/approved by the whole team (see RC-04).

---

## 🧪 Functional Tests / Validation

Since this is a documentation deliverable, "testing" here means a **review checklist** rather than functional test cases:

| Scenario ID | Review Check | Expected Result |
| :--- | :--- | :--- |
| **RC-01** | Every term has both a *Definition* and *Related concepts* | No term is left as an orphaned definition |
| **RC-02** | Every class/enum in the Domain Model has a matching glossary entry | No entity exists in the model without a glossary term |
| **RC-03** | No definition is a generic/dictionary-style definition | Definitions are specific to the ShopRobot domain |
| **RC-04** | Index links at the top match the anchors used in each entry | No broken in-document links |

---

# 🧪 QA & Test Documentation

See Report at [QA & Test Documentation: USBD02 Project Glossary](qa-readme.md)

---

## 🧩 Related Modules & Classes

Not applicable — no code module. The artifact is `../../glossary/readme.md`.

---

## 🔄 Sequence Diagram

Not applicable. This story has no runtime interaction/flow to diagram — it documents static business concepts.

## 🏁 Definition of Done (DoD)

* [x] Acceptance criteria verified against the requirements document.
* [ ] Reviewed and approved by the whole team (pending — see Edge Cases above).
* [x] Consistent with the current state of the Domain Model.
* [ ] Final sign-off recorded in `qa-readme.md`.