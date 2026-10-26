# 🧪 QA & Test Documentation: [USBD03] Conceptual Domain Model

* **User Story Reference:** [USBD03: Conceptual Domain Model](readme.md)
* **QA Lead / Owner:** TBD — name to be filled in; must be a team member other than the author (Hugo Coutinho), per section 3.4 of the requirements document
* **Target Environment:** `Repository / Documentation`
* **Build / Release Version:** Sprint 1
* **QA Status:** 🟡 In Review (waiting for the reviewer to run the checks)

---

## 🎯 QA Strategy & Scope

### In-Scope
* Coverage — all business entities from the enunciado are represented.
* Consistency with the Glossary (USBD02/USLP02).
* Diagram quality — multiplicities, directions and association names.
* Rendering — PlantUML source produces a valid, up-to-date SVG.

### Out-of-Scope
* Code review (no code produced by this story).
* Load/performance testing (not applicable to a static document).
* Physical or logical model details (those are USBD04 and USBD05).

---

## 🛠️ Test Environment & Setup

* **Location:** `../../domain-model/domain-model.puml` and `../../domain-model/domain-model.svg`
* **Reviewers needed:** At least one team member who did not author the domain model.
* **Pre-conditions:** Glossary (USBD02/USLP02) and requirements document (USBD01) must be in a stable state for cross-reference.

---

## 📋 Review Matrix

| Review ID | Title / Scenario                                          | Type         | Review Steps                                                                 | Expected Result                                                   |   Status   | Reviewed By | Notes |
|:----------|:----------------------------------------------------------|:-------------|:-----------------------------------------------------------------------------|:------------------------------------------------------------------|:----------:|:-----------:|:------|
| **RC-01** | All enunciado entities represented                        | Completeness | Compare domain concepts from enunciado §4 and §7 vs model classes            | No entity missing; no entity added without business justification | 🟡 Pending |     TBD     | —     |
| **RC-02** | All model entities have a glossary entry                  | Consistency  | Compare model classes vs `glossary/readme.md` index                          | No entity left undocumented in the glossary                       | 🟡 Pending |     TBD     | —     |
| **RC-03** | All associations have multiplicities, direction, and name | Quality      | Inspect each association in the PlantUML source                              | No association is anonymous or direction-less                     | 🟡 Pending |     TBD     | —     |
| **RC-04** | Enums are correct and consistent with glossary            | Consistency  | Compare `RobotState` and `OrderStatus` values vs glossary definitions        | Values match the agreed list                                      | 🟡 Pending |     TBD     | —     |
| **RC-05** | PlantUML renders without errors; SVG is up to date        | Rendering    | Render `domain-model.puml`; compare output with committed `domain-model.svg` | No rendering errors; SVG reflects the current source              | 🟡 Pending |     TBD     | —     |
| **RC-06** | Model consistent with requirements document               | Consistency  | Cross-check model entities and relationships vs USBD01 requirements table    | No entity or relationship contradicts a stated requirement        | 🟡 Pending |     TBD     | —     |

### Execution Log

NOTE: register here the outcome of the team review meeting once it happens.

| Date | Reviewed By | Result |
|:-----|:------------|:-------|
| TBD  | TBD         | TBD    |

---

## 🐞 Defect & Bug Tracking

| Issue ID | Summary / Description | Severity | Priority | Status | Linked Commit / Fix | Re-test Status |
|:---------|:----------------------|:---------|:---------|:-------|:--------------------|:--------------:|
| —        | None identified yet   | —        | —        | —      | —                   |       —        |

---

## 🤖 Unit Tests

Not applicable — no automated tests apply to a documentation artefact.

---

## 📊 Summary & QA Sign-Off

### Execution Results
* **Total Reviewed:** 0 / 6
* **Approved:** 0
* **Pending:** 6

### Final QA Recommendation

> 🟡 **Pending Team Review:** The domain model diagram is drafted and internally consistent with the current glossary, but requires sign-off from a reviewer other than the author before being marked Done, per the project's Quality Assurance process (section 3.4). The model must also be re-reviewed at the end of every sprint as new requirements are added.

* [ ] **QA Sign-off Approved**
* **Signed By:** TBD
* **Date:** TBD