# DPDP Act, 2023 — Frontend/UX Compliance Reference for Sparkle

This document culls out the parts of India's **Digital Personal Data Protection Act, 2023** (DPDP Act) and the **DPDP Rules, 2025** that fall on the *frontend/product/UX* layer of a consumer digital product — i.e. what needs to be **built and shown to the user**, as opposed to backend/legal/organizational obligations (DPO appointment paperwork, audit trails, government filings, etc.).

It is written against the current state of `sparklequicksession_2.html`, Sparkle's new-user onboarding/quick-session flow, which today collects **phone number (+91, OTP-style), name, email, birthday, and anniversary** with no visible consent notice, purpose disclosure, age gate, or privacy controls.

> **This is a product/engineering reference, not legal advice.** Treat citations and dates as a starting point for legal counsel to verify against the primary Gazette text before this becomes a locked compliance spec.

---

## 1. Timeline

| Date | Milestone |
|---|---|
| 13 Nov 2025 | DPDP Rules 2025 notified in final form; Data Protection Board of India (DPBI) operationalized |
| 13 Nov 2026 | Consent Manager registration regime opens |
| **13 May 2027** | **Full compliance deadline** for notice (Rule 3), consent-flow obligations, children's data rules, Significant Data Fiduciary duties, breach/retention mechanics, and rights/grievance mechanics |

The Act's substantive principles (Sections 5, 6, 9, 11–14) are law today; the Rules mainly operationalize *how* and set the grace period before the detailed mechanics are enforced. Building to the final Rules now (rather than waiting for the deadline) is the safer path, since retrofitting consent/onboarding flows later is expensive.

---

## 2. Consent & Notice (Section 5, Section 6, Rule 3 + Schedule I)

**What the law requires:**
- Notice must be **itemized** — each data category paired with its specific purpose (e.g. "Phone number — account verification / OTP login", "Birthday — personalized offers"). Generic language like "your information" is non-compliant.
- Notice must be **standalone**, in **clear plain language**, not buried inside Terms & Conditions.
- Must be offered in **English or an Eighth-Schedule Indian language**, with a language option on request.
- Consent must be given by **clear affirmative action** — never inferred from silence, scrolling, or continued use.
- Consent must be **specific per purpose** — no bundling unrelated purposes under one checkbox.
- Consent **cannot be a precondition** for using the core product, and declining optional processing must not degrade the core experience.
- **Withdrawal must be as easy as giving consent** (Section 6(4)) — a one-tap grant needs a one-tap revoke.

**What this means for Sparkle's onboarding flow:**
- [ ] Add a dedicated, itemized consent screen before/alongside phone-number capture — not a footer link. List each field collected (phone, name, email, birthday, anniversary) with its purpose.
- [ ] Separate **essential** (phone/OTP for login) from **optional** (birthday/anniversary for personalized offers, marketing emails) — optional items get their own **off-by-default toggle**, not one bundled "I agree" checkbox.
- [ ] No pre-ticked boxes anywhere in the form.
- [ ] Add a language selector on the consent screen.
- [ ] Build a **Privacy/Consent settings screen** reachable from the main app menu where every toggle granted at onboarding can be flipped off in equivalent effort — and have that flip actually stop downstream processing (marketing sends, personalization), not just update a UI switch.
- [ ] Version the consent-notice copy and send the version ID with each consent event, so what the user actually saw can be reconstructed later.

---

## 3. Children's Data (Section 9, Rule 10)

**What the law requires:**
- Anyone **under 18** is a "child" — one bright-line age, no tiered bands.
- **Verifiable parental consent** is required — not a self-declared "I am 18+" checkbox; the fiduciary must independently verify the parent is a real, identifiable adult.
- **Absolute ban** on tracking, behavioural monitoring, and targeted advertising directed at children — this cannot be unlocked by parental consent at all.

**What this means for Sparkle:**
- [ ] The birthday field currently collected should be used to **compute age at signup**, not just stored as a preference field.
- [ ] If the computed age is under 18, branch to a **parental-consent flow** (e.g., OTP/verification sent to a parent-provided, independently registered contact) instead of proceeding with normal onboarding.
- [ ] Disable analytics/ad-personalization SDKs and any behavior-based recommendation logic for accounts flagged as minors.
- [ ] Do not show personalized offers, streak/engagement nudges, or targeted promotions to minor accounts.

---

## 4. Data Principal Rights (Sections 11–14, Rule 14)

| Right | Section | Sparkle UI needed |
|---|---|---|
| Access a summary of held data & processing | S.11 | "My Data" screen: show/export name, phone, email, birthday, anniversary and how each is used |
| Correction / update / erasure | S.12 | Editable profile fields + a real "Delete my account & data" action |
| Grievance redressal (≤90 days, published SLA) | S.13, Rule 14(3) | Visible "Contact Grievance Officer" entry point (settings + footer) with a stated response-time commitment |
| Nominate someone to act on your behalf (death/incapacity) | S.14 | A "Nominee" field in account settings |

- [ ] Consolidate the above into one **"Privacy Center"** screen rather than scattering them — this is also where consent toggles (Section 2) live.

---

## 5. Grievance Redressal (Section 13, Rule 14)

- [ ] Publish Grievance Officer / DPO contact details (name, contact channel) in the Privacy Center and app footer — not only in a PDF privacy policy.
- [ ] Display the response-time commitment at the point of submission (e.g., "We aim to resolve this within X days, no later than 90 days").

---

## 6. Dark Patterns (CCPA Dark Patterns Guidelines, 2023 — overlaps with DPDP Section 6)

Not part of the DPDP Act itself, but a parallel regime whose banned patterns are exactly the failure modes Section 6 ("free, unambiguous, clear affirmative action") also prohibits. Audit the onboarding flow against:

- [ ] No pre-ticked boxes (basket sneaking).
- [ ] No guilt-trip copy on decline/skip buttons ("No, I don't want great offers" style confirm-shaming).
- [ ] No fake urgency/countdown timers pressuring consent.
- [ ] No disguising a marketing-consent ask as a plain "Continue" button.
- [ ] Decline/opt-out affordance as visually prominent as accept.

---

## 7. Data Breach Notification (Section 8(6), Rule 7)

- [ ] Have a ready-to-fire, plain-language breach notification template (email/SMS + in-app), including: nature/extent of breach, data categories affected, likely consequences, remedial steps, and a contact point. No severity threshold — even a small breach triggers this.

---

## 8. Retention / Erasure Notice (Rule 8)

- [ ] Before deleting a user's data at end of retention/inactivity, send a **48-hours-before-erasure notice** (email/SMS/push — not just in-app, since the user may not be actively logged in) with a one-tap "keep my account active" action.

---

## 9. Cross-Border Transfer (Section 16, Rule 15)

No country is currently on a restricted list, so no consumer-facing disclosure is mandatory today. Low priority, but:
- [ ] Mention in the itemized notice (Section 2) if any processor/vendor is located outside India.
- [ ] Keep a config layer that can quickly geofence a jurisdiction if the government issues a restricted-country notification — this can happen with no advance warning.

---

## 10. Out of scope for frontend (flagging only)

These are real DPDP obligations but sit with legal/backend/ops, not the UI layer — noted here so they aren't lost, not because the frontend team owns them:
- Significant Data Fiduciary designation, DPIA, independent data audits (Section 10, Rule 13) — only the *published DPO contact* is a frontend surface if Sparkle is ever designated.
- Consent Manager registration (Rule 4) — relevant only to the backend consent-data architecture (keep it API-first/decoupled so a future Consent Manager could integrate).
- Board notification mechanics for breaches (72-hour detailed report) — backend/legal process.

---

## Sources

Research was triangulated across multiple secondary legal/compliance sources (primary Gazette PDF was not reachable in this environment); flagged uncertainties above (exact Schedule I wording, a disputed "7-day acknowledgment" figure, SDF designation) should be verified against the Gazette text or by counsel before this is treated as a final spec.
