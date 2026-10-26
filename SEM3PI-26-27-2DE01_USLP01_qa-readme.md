# 🧪 QA & Test Documentation: [USLP01] Feature Title

* **User Story Reference:** [US-USLP01: US Title](./readme.md)
* **QA Lead / Owner:** Jane Smith
* **Target Environment:** `Staging / QA-01`
* **Build / Release Version:** `v2.4.0-rc.1`
* **QA Status:** 🟡 In Progress / 🟢 Passed / 🔴 Blocked / 🟠 Failed

---

## 🎯 QA Strategy & Scope

### In-Scope
* Functional verification of valid, invalid, and expired promo codes.
* Cart total recalculation accuracy.
* Rate-limiting and security brute-force checks.
* UI component responsiveness and screen reader accessibility (`aria-live`).

### Out-of-Scope
* Stacking multiple promo codes (deferred to [US-108]).
* Payment gateway processor failure handling (covered under [US-055]).

---

## 🛠️ Test Environment & Setup

* **App URL / Endpoint:** `https://staging.app.example.com/checkout`
* **Test Accounts Used:**
  * Guest User (No login)
  * Standard User: `qa_user_01@example.com`
* **Pre-conditions & Test Data:**
  * Active Promo Code: `SAVE10` (10% off)
  * Fixed Value Code: `FLAT20` ($20 off)
  * Expired Promo Code: `EXPIRED2025`

---

## 📋 Test Matrix

| Test ID | Title / Scenario | Type | Pre-conditions | Test Steps | Expected Result | Status | Execution Date | Bugs Filed |
| :--- | :--- | :--- | :--- | :--- | :--- | :---: | :---: | :--- |
| **TC-01** | Apply valid percentage code | Functional | Cart total = $100 | 1. Enter `SAVE10`<br>2. Click Apply | Total updates to $90.00 (-$10.00) | 🟢 PASS | 2026-09-04 | None |
| **TC-02** | Apply expired code | Negative | Cart total = $50 | 1. Enter `EXPIRED2025`<br>2. Click Apply | Error alert: *"Invalid or expired code"* | 🟢 PASS | 2026-09-04 | None |
| **TC-03** | Replace applied code | Functional | Code `SAVE10` applied | 1. Enter `FLAT20`<br>2. Click Apply | `SAVE10` is removed; $20 discount applied | 🟢 PASS | 2026-09-04 | None |
| **TC-04** | Exceed rate limit | Security | IP clean | 1. Submit invalid code 6 times in 60s | 6th attempt blocked with HTTP 429 | 🔴 FAIL | 2026-09-04 | [BUG-304](#-defect--bug-tracking) |
| **TC-05** | Refresh cart with code | Regression | Code `SAVE10` applied | 1. Refresh browser | Applied code and discount persist | 🟢 PASS | 2026-09-04 | None |

### Execution Log

NOTE: add here evidences of the tests, such as logs or screenshots.

---

## 🐞 Defect & Bug Tracking

| Bug ID | Summary / Description | Severity | Priority | Status | Linked Commit / Fix | Re-test Status |
| :--- | :--- | :--- | :--- | :--- | :--- | :---: |
| **BUG-304** | Rate limiter returns 500 Server Error instead of HTTP 429 | High | High | Open | PR `#412` pending | ⏳ Pending |
| **BUG-305** | Discount text overlaps on mobile screens (<375px) | Low | Medium | Fixed | PR `#415` merged | 🟢 Passed |

---

## 🤖 Unit Tests

* **Related App:** JavaApp
* **Related Code:** com.example.dao.User, com.example.dao.UserDao, com.example.dao.jdbcUserDao  
* **Automated Tests:** com.example.dao.UserDaoTest
* **Execution:** mvn test
* **CI Build Link:** [GitHub Pipeline #1204](https://ci.example.com/build/1204)

---

## 📊 Summary & QA Sign-Off

### Execution Results
* **Total Executed:** 5
* **Passed:** 4
* **Failed:** 1
* **Blocked:** 0

### Final QA Recommendation

> ⚠️ **Conditional Approval:** Core functional scenarios (`TC-01` to `TC-03`, `TC-05`) have passed. However, deployment to Production is **blocked** pending the resolution of [BUG-304] (security rate-limiting fix).

* [ ] **QA Sign-off Approved**
* **Signed By:** Jane Smith (QA Lead)
* **Date:** *(Pending Bug Resolution)*