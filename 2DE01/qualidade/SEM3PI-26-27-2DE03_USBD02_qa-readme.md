# 🧪 QA & Test Documentation: [USBD02] Project Glossary

* **User Story Reference:** [USBD02: Project Glossary](readme.md)
* **QA Lead / Owner:** TBD — assign a reviewer other than the author, per section 3.4 of the requirements document
* **Target Environment:** `Repository / Documentation`
* **Build / Release Version:** Sprint 1
* **QA Status:** 🟡 In Progress

---

## 🎯 QA Strategy & Scope

### In-Scope
* Consistency between glossary terms and the Domain Model entities.
* Business-focus and clarity of each definition.
* Internal link integrity (index ↔ anchors).

### Out-of-Scope
* Code review (no code produced by this story).
* Load/performance testing (not applicable to a static document).

---

## 🛠️ Test Environment & Setup

* **Location:** `../../glossary/readme.md`
* **Reviewers needed:** At least one team member who did not author the glossary.
* **Pre-conditions:** Domain Model (USBD03/USLP01) should be in a stable-enough state to compare against.

---

## 📋 Review Matrix

| Review ID | Title / Scenario | Type | Review Steps | Expected Result | Status | Reviewed By | Notes |
| :--- | :--- | :--- | :--- | :--- | :---: | :---: | :--- |
| **RC-01** | All glossary terms have Definition + Related concepts | Completeness | Scan every entry | No missing sections | 🟡 Pending | — | — |
| **RC-02** | All Domain Model entities have a glossary entry | Consistency | Compare `domain-model.puml` classes vs glossary index | No entity left undocumented | 🟡 Pending | — | — |
| **RC-03** | Definitions are business-specific, not generic | Quality | Read each definition | No "dictionary style" definitions | 🟡 Pending | — | — |
| **RC-04** | Team agrees on new/disputed terms (Retailer Administrator, Shopping Cart, Route, Stop) | Team Validation | Discuss in team meeting | Consensus reached or terms revised | 🟡 Pending | — | — |

### Execution Log

NOTE: register here the outcome of the team review meeting once it happens.

---

## 🐞 Defect & Bug Tracking

| Issue ID | Summary / Description | Severity | Priority | Status | Linked Commit / Fix | Re-test Status |
| :--- | :--- | :--- | :--- | :--- | :--- | :---: |
| — | None identified yet | — | — | — | — | — |

---

## 🤖 Unit Tests

Not applicable — no automated tests apply to a documentation artifact.

---

## 📊 Summary & QA Sign-Off

### Execution Results
* **Total Reviewed:** 0 / 4
* **Approved:** 0
* **Pending:** 4

### Final QA Recommendation

> 🟡 **Pending Team Review:** Glossary content is drafted and internally consistent with the current domain model, but requires sign-off from a reviewer other than the author before being marked Done, per the project's Quality Assurance process (section 3.4).

* [ ] **QA Sign-off Approved**
* **Signed By:** TBD
* **Date:** TBD