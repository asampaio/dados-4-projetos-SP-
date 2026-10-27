# 🧪 QA & Test Documentation: [USLP02] Project Glossary

* **User Story Reference:** [USLP02: Project Glossary](./readme.md)
* **QA Lead / Owner:** TBD (must be a team member other than the author, per section 3.4 of the requirements document; review requested)
* **Target Environment:** `Repository / Documentation`
* **Build / Release Version:** Sprint 1
* **QA Status:** 🟡 Ready for review (glossary updated; review requested, no check executed yet)

---

## 🎯 QA Strategy & Scope

### In-Scope
* Consistency between the glossary terms and the domain model ([USLP01]).
* Business focus and clarity of each definition.
* Presence of the client clarifications in the affected entries.
* Internal link integrity (index and anchors).

### Out-of-Scope
* Code review (no code is produced by this story).
* The domain model diagram itself ([USLP01]).
* Performance or load testing (not applicable to a static document).

---

## 🛠️ Test Environment & Setup

* **Location:** `doc/glossary/readme.md`, rendered on GitHub.
* **References:** the requirements document, `doc/domain-model/domain-model.puml` and the client clarifications.
* **Approach:** static review. Each check is answered by reading the glossary against the references. There is no code, so TDD does not apply.
* **Same artefact as [USBD02]:** one execution of the checks below covers both stories.

---

## 📋 Review Matrix

| Review ID | Title / Scenario | Type | Review Steps | Expected Result | Status | Reviewed By | Notes |
| :--- | :--- | :--- | :--- | :--- | :---: | :---: | :--- |
| **RC-01** | Every term has a definition and related concepts | Completeness | Scan every entry | No missing section; entities also have an identity | ⚪ Not run | — | Actors in "Other Actors" have no identity |
| **RC-02** | Every class, enumeration and value object of the model has an entry | Consistency | Compare `domain-model.puml` with the glossary index | All 20 elements have an entry with the same name | ⚪ Not run | — | 15 entities, 3 value objects, 2 enumerations |
| **RC-03** | Definitions are business-specific | Quality | Read each definition | No generic, dictionary-style definition | ⚪ Not run | — | — |
| **RC-04** | Index and anchors | Link integrity | Follow every link of the index and of the entries | No broken link; every entry is in the index once | ⚪ Not run | — | — |
| **RC-05** | Client clarifications are reflected | Consistency | Read Refrigerated Unit, Robot State, Battery Level, Order, Route, Product Price, Area and Level | Each entry matches what the client or the teacher said | ⚪ Not run | — | Refrigeration, "busy" state, minimum battery, final orders, route planning, price history, GeoJSON identifier and decimal levels |
| **RC-06** | Rules match the model | Consistency | Compare the rules of each entry with the multiplicities of the diagram | No rule contradicts the diagram | ⚪ Not run | — | e.g., an order is fulfilled by one robot or by sub-orders, never both |
| **RC-07** | Formatting | Quality | Check the entries of "Customer" and "GeoJSON" | One definition per term; lists correctly indented | ⚪ Not run | — | Errors found and fixed in Sprint 1 |

### Execution Log

NOTE: register here the outcome of the review once it is done.

---

## 🐞 Defect & Bug Tracking

| Issue ID | Summary / Description | Severity | Priority | Status | Linked Commit / Fix | Re-test Status |
| :--- | :--- | :--- | :--- | :--- | :--- | :---: |
| — | None identified yet | — | — | — | — | — |

---

## 🤖 Unit Tests

Not applicable, there are no automated tests for a documentation artefact.

---

## 📊 Summary & QA Sign-Off

### Execution Results
* **Total Reviewed:** 0 / 7
* **Approved:** 0
* **Pending:** 7

### Final QA Recommendation

> 🟡 **Ready for review:** the glossary is up to date with the client clarifications and the review has been requested. QA sign-off must be given by a team member other than the author.

* [ ] **QA Sign-off Approved**
* **Signed By:** TBD
* **Date:** TBD