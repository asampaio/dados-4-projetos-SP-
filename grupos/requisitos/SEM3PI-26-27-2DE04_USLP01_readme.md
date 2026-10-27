# [USLP01] Feature Title

* **Priority:** High / Medium / Low
* **Story Points:** 5
* **Module / Component:** `Orders` / `Checkout`
* **Status:** Draft / Ready for Dev / In Progress / Done

---

## 👤 User Story Description

**As a** registered customer  
**I want to** apply a discount code during checkout  
**So that** I can get a reduction on my order total

---

## 🚦 Definition of Ready (DoR)

* [ ] **Clear User Story:** Story follows the standard *As a / I want to / So that* format.
* [ ] **Acceptance Criteria Defined:** Clear, testable ACs agreed upon by Product and Engineering.
* [ ] **Dependencies Identified:** Upstream APIs, third-party integrations, or blocking tasks are resolved or scheduled.
* [ ] **Inputs/Outputs Mapped:** Required data fields, payloads, and error states are documented.
* [ ] **Estimation Complete:** Story has been refined and estimated (story points assigned) by the dev team.
* [ ] **Small & Actionable:** Story is scoped down enough to be completed within a single sprint.

---

## 💬 Customer Specifications & Clarifications

* **Clarification (2026-03-15):** Codes are case-insensitive (`SAVE10` is the same as `save10`).
* **Clarification (2026-03-16):** Only one promo code can be applied per checkout session.
* **Out of Scope:** Stacking multiple discount codes or applying gift cards (handled in [US-108]).

---

## ✅ Acceptance Criteria

* [ ] **AC1 - Valid Code:** Applying a valid code reduces the order subtotal by the designated percentage/amount.
* [ ] **AC2 - Invalid Code:** Entering an invalid or expired code displays a clear error message: *"Invalid or expired promo code."*
* [ ] **AC3 - Single Use:** Entering a new code replaces any previously applied promo code.
* [ ] **AC4 - Order Persistence:** The applied code and discount amount persist if the user refreshes the page.

---

## 🔒 Non-Functional Requirements & Security

* **Performance:** Promo code validation API response time must be under **200ms** at $P_{95}$.
* **Security & Anti-Abuse:** Rate-limit the validation endpoint to **5 requests per minute per IP** to prevent brute-force attacks on promo codes.
* **Authorization:** Feature accessible to both `GUEST` and `AUTHENTICATED_USER` roles.

---

## 🔗 Found Out Dependencies

### Upstream Dependencies (Required before starting)
* **[US-089] Promo Code Management API:** Needs endpoint `GET /api/v1/promos/{code}/validate` available in staging.

### Downstream Dependencies (Impacted by this story)
* **[US-115] Order Receipt Generation:** Needs to read `discountAmount` from order payload to display savings.

---

## 📥 Input & Output Data

### Inputs
| Field Name | Data Type | Required | Source | Example / Constraints |
| :--- | :--- | :--- | :--- | :--- |
| `promoCode` | String | Yes | User Input | `"SUMMER20"`, max 20 chars |
| `cartTotal` | Decimal | Yes | Cart State | `120.50` |

### Outputs
| Field Name | Data Type | Destination | Example / Description |
| :--- | :--- | :--- | :--- |
| `discountAmount` | Decimal | Order Summary State | `12.05` |
| `appliedCode` | String | Checkout Payload | `"SUMMER20"` |
| `errorCode` | String | UI Alert Component | `"INVALID_PROMO"` |

---

## Edge Cases

* **Zero Balance Cart:** What happens if 100% off coupon makes the order total $0.00? (Bypass payment step).
* **Network Timeout:** Show retry button with message *"Network issue. Please try applying code again."*

---

## 🧪 Functional Tests / Validation

| Scenario ID | Test Case / Steps | Expected Result |
| :--- | :--- | :--- |
| **TC-01** | Apply valid 10% off code on a $100 cart | Cart total updates to $90; discount line item shows -$10.00 |
| **TC-02** | Apply code past its expiration date | Error banner appears; cart total remains unchanged |
| **TC-03** | Apply code `SUMMER10`, then apply code `FALL20` | `SUMMER10` is removed, `FALL20` discount is applied |

---

# 🧪 QA & Test Documentation

See Report at [QA & Test Documentation: USLP01 Feature Title](./qa-readme.md)

---

## 🧩 Related Modules & Classes

* **Frontend:**
  * Component: `CheckoutSummary.tsx`
  * State Hook: `usePromoCode.ts`
* **Backend:**
  * Controller: `PromoCodeController`
  * Service: `DiscountCalculationService`
  * Entity: `DiscountRule`

---

## 🔄 Sequence Diagram

![Sequence Diagram](sd.svg)

## 🏁 Definition of Done (DoD)

* [ ] Acceptance criteria verified by QA / Product Owner.
* [ ] Unit tests written and passing (minimum 80% coverage on new service logic).
* [ ] Integration/E2E tests added for promo application flow.
* [ ] API specs updated in OpenAPI / Swagger documentation.
* [ ] Feature flag configured (`enable-checkout-promos`).