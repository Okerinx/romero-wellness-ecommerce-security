# Security & Compliance Assessment — Romero Wellness (E-commerce)

**A GRC case study on securing customer PII and payment flows in a WooCommerce store**

*Author: [Full Name] · Role: Security & Compliance (self-directed assessment) · Platform: romerow.com*

---

## 1. Executive Summary

Romero Wellness is a live e-commerce store (wellness/supplements) built on **WooCommerce (WordPress + Elementor)**, taking payments through **Wompi** over TLS. Unlike a content site, an online store continuously collects **customer personal data (PII)** — names, contact details, shipping addresses, and order history — and sits directly in a **payment flow**, which raises two distinct obligations: **data protection** (Colombia's Law 1581) and **payment security** (PCI-DSS).

This assessment maps the store's data and payment flows, determines its **PCI-DSS scope**, evaluates the primary e-commerce risks (account takeover, third-party script skimming, admin compromise, and availability), and produces a prioritized remediation roadmap. Controls are mapped to **PCI-DSS v4.0.1** and **ISO/IEC 27001:2022 Annex A**.

**Headline finding:** because card data is handled by Wompi (a third-party processor) rather than stored on the store, the merchant is a **reduced-scope (SAQ-A / SAQ-A-EP) merchant** — but reduced scope is *not* zero scope. The highest practical risks are **client-side (payment-page script integrity) and account/administrator security**, exactly the areas PCI-DSS v4.0 tightened.

---

## 2. Scope & Systems

| Item | Detail |
|------|--------|
| Platform | WooCommerce on WordPress + Elementor |
| Payments | Wompi (third-party processor), TLS/SSL enabled |
| Data collected | Customer PII, shipping/billing details, order history, account credentials |
| In scope | Customer data lifecycle, payment flow, CMS/admin security, third-party scripts, privacy & consumer-protection notices |
| Out of scope | Active exploitation/penetration testing (defensive review of an owned asset) |
| Frameworks | PCI-DSS v4.0.1 · ISO/IEC 27001:2022 · Law 1581/2012 · Consumer Statute (Law 1480/2011) |

---

## 3. Data & Payment Flows

```
Customer → Store (account, cart, checkout) → Wompi (card data entry/authorization) → Bank
                     │
                     └─ Store retains: PII, addresses, order history (NOT card numbers)
```

**Key determination — where does card data live?**
- If the customer enters card details on **Wompi's hosted page or a properly isolated iframe/redirect**, cardholder data never touches the store → **SAQ-A** eligibility (lowest scope).
- If card fields are rendered **inline on the store's own checkout page** (even if posted directly to Wompi), the page can affect the security of card data → **SAQ-A-EP** (broader scope, more requirements).

*This distinction must be confirmed against the actual Wompi integration; it changes the compliance obligations materially.*

---

## 4. Data Classification

| Data element | Classification | Notes |
|--------------|----------------|-------|
| Cardholder data (PAN, CVV) | Restricted — **not stored by merchant** | Handled by Wompi; goal is to keep it out of store scope |
| Customer PII (name, email, phone, address) | Confidential / personal data | Governed by Law 1581 |
| Order history & account credentials | Confidential | Account-takeover target; credentials must be hashed |
| Payment-page third-party scripts | Security-critical | In scope for PCI-DSS v4.0 client-side requirements |

---

## 5. Regulatory & Compliance Mapping

**PCI-DSS v4.0.1 (payment security):**
- Even reduced-scope (SAQ-A) merchants must now address **payment-page script management and integrity** (req. 6.4.3) and **change/tamper detection on the payment page** (req. 11.6.1) — a direct response to digital-skimming (Magecart) attacks.
- No storage of prohibited data (CVV) anywhere on the store.

**Colombia — Law 1581/2012 (data protection):** lawful, informed collection of PII; privacy notice; honoring habeas-data rights (access, correction, deletion). Verify RNBD obligations with the SIC.

**Colombia — Consumer Statute, Law 1480/2011:** clear terms, pricing, right of withdrawal/returns, and accurate product information for e-commerce.

**GDPR:** applies only if EU customers are served.

---

## 6. Threat & Risk Assessment (Risk Register)

*Qualitative: Likelihood (L) / Impact (I) — Low / Medium / High.*

| ID | Risk | L | I | Rating | Concern |
|----|------|---|---|--------|---------|
| R1 | Third-party/payment-page script skimming (Magecart-style) | M | High | **High** | Card & PII theft at checkout |
| R2 | Customer account takeover (credential stuffing, weak auth) | High | Medium | **High** | Fraud, PII exposure |
| R3 | Admin (wp-admin) compromise via weak/single-factor auth | M | High | **High** | Full store & data control |
| R4 | Vulnerable/outdated plugins or themes | High | Medium | **High** | Common WordPress entry point |
| R5 | Store downtime / availability loss | M | High | **Medium** | Direct revenue loss |
| R6 | Excess PII retention with no deletion policy | High | Medium | **Medium** | Larger breach blast radius |
| R7 | Weak backup/recovery process | M | High | **Medium** | No clean restore after incident |
| R8 | Incomplete privacy/consumer notices | M | Medium | **Medium** | Legal non-compliance |

---

## 7. Control Gap Analysis — PCI-DSS & ISO/IEC 27001:2022

| Risk | Recommended control | Reference |
|------|---------------------|-----------|
| R1 | Payment-page script inventory + integrity (SRI), Content Security Policy, tamper monitoring | PCI-DSS 6.4.3 / 11.6.1 · ISO A.8.26 |
| R2 | Strong customer auth, rate limiting, optional MFA, breach-password checks | ISO A.8.5 |
| R3 | MFA on all admin accounts + least-privilege roles | ISO A.8.5, A.5.15 |
| R4 | Patch/update management for plugins, themes, core | ISO A.8.8 |
| R5 | Availability monitoring, WAF/CDN, DDoS protection | ISO A.8.6, A.8.16 |
| R6 | Data-retention schedule & secure deletion | ISO A.5.10, A.8.10 |
| R7 | Tested backups with defined RPO/RTO | ISO A.8.13 |
| R8 | Compliant privacy notice + consumer-protection terms | Law 1581 / Law 1480 |

---

## 8. Prioritized Remediation Roadmap

**Now:**
- Confirm the **Wompi integration type** and lock the store to the lowest PCI scope (redirect/iframe, no inline card fields).
- Enforce **MFA on all wp-admin accounts**; remove unused admin users.
- Inventory every **third-party script on the checkout page**; remove anything non-essential.

**Within 30 days:**
- Implement **Subresource Integrity + a Content Security Policy** on payment pages; enable tamper/change detection (PCI 11.6.1).
- Establish **plugin/theme/core patch management** on a regular cadence.
- Enable **tested, off-site backups** with defined recovery objectives.

**Within 90 days:**
- Add customer-account protections (rate limiting, optional MFA, compromised-password checks).
- Define **data retention & deletion** and align privacy + consumer-protection notices (Law 1581 / Law 1480).
- Add availability protection (WAF/CDN) and a lightweight incident-response plan.

---

## 9. Residual Risk & Conclusion

Keeping cardholder data entirely with Wompi already removes the store from the highest-risk PCI obligations. With the "Now" and 30-day controls — payment-page script integrity, admin MFA, and patching — the dominant High risks (R1–R4) drop to acceptable residual levels. For an e-commerce store, **security is also availability and trust**: the same controls that protect data also protect revenue.

This assessment demonstrates the ability to **determine PCI-DSS scope, secure a real payment flow, and translate e-commerce risk into a prioritized, framework-aligned remediation plan**.

---

### Skills demonstrated (note for reviewers)

- **PCI-DSS scoping** (SAQ-A vs SAQ-A-EP) and current v4.0 client-side script requirements.
- E-commerce threat modeling: **digital skimming, account takeover, availability**.
- Control mapping across **PCI-DSS** and **ISO/IEC 27001:2022**.
- Multi-regulation awareness: **data protection (Law 1581)** and **consumer protection (Law 1480)**.
