# Mobile Money Platform – Test Plan

A comprehensive QA test plan for a fictional mobile money platform, designed to demonstrate FinTech QA depth across functional, security, compliance, and money math concerns. The plan covers 84 test cases across 7 flows, drawing on industry best practice from leaders like Wise (FX transparency), Stripe (idempotency patterns), and the major mobile money operators in Africa and Asia.

## What this is

This is a QA design and documentation artifact. The repo demonstrates the kind of thinking that goes into testing a regulated financial platform end to end: how to design tests that catch real bugs (race conditions, partial debits, double-credits), how to enforce regulatory requirements (KYC tiers, AML reporting, sanctions screening), and how to design audit trails that survive a forensic review.

The platform under test is fictional but designed to mirror real-world mobile money systems operating in emerging markets (MTN MoMo, M-Pesa, Wave, AirtelTigo, Telecel, and others). Ghana is used as the default jurisdiction for currency, regulator references, and tier structure.

## Skills demonstrated

- **Money math precision**: minor-unit storage, banker's rounding, fee-inclusive balance checks, daily e-money conservation invariants
- **Atomicity and concurrency**: race conditions on daily limit aggregation, atomic dual-account updates, structuring pattern detection
- **Idempotency**: retry-safe APIs with cached-response semantics across cash-in, cash-out, and cross-border flows
- **Authentication and fraud prevention**: PIN-on-user-device flows, step-up auth for high-value transactions, SIM swap detection, new-device verification, multi-factor account recovery
- **Compliance**: tiered KYC, AML triggers (structuring, CTR, SAR, PEP), sanctions screening (OFAC, UN, EU, UK), data subject access requests, retention-aware deletion, regulatory reporting
- **Cross-border FX**: rate locking with quote validity windows, transparent margin disclosure, multi-currency precision handling, FX-favorable refund on failure
- **Audit and reconciliation**: hash-chained immutable ledgers, daily e-money conservation reconciliation, balance reconstruction from ledger at any timestamp
- **Agent network mechanics**: agent float ceilings, commission calculation and end-of-day settlement, reversal windows with consent flows

## Applicable industries

The patterns and skills demonstrated apply broadly across FinTech, not just to mobile money operators:

- **Mobile money platforms**: M-Pesa, MTN MoMo, AirtelTigo, Telecel, Wave, Orange Money, OPay, PalmPay, bKash, GCash, Paytm
- **African FinTech**: Paystack, Flutterwave, Chipper Cash, Eversend, Kuda, Sendy
- **Global FinTech**: Stripe, Wise, Revolut, Adyen, Mercury, Cash App, Mercado Pago
- **Neobanks, crypto exchanges, remittance providers, payment processors**: any platform that handles money and is subject to AML, sanctions, and prudential regulation

## Test plan structure

The full test plan is in [TEST_PLAN.md](./TEST_PLAN.md). It is organized into 7 flows, each with approximately 12 test cases:

| Code | Flow | Cases |
| --- | --- | --- |
| `KYC` | Account & KYC | 12 |
| `AUTH` | Authentication & Security | 12 |
| `CIN` | Money In (cash-in, deposits) | 12 |
| `MOV` | Money Movement (P2P, bills, merchants) | 12 |
| `COUT` | Money Out (cash-out, withdrawals) | 12 |
| `XBD` | Cross-Border & FX | 12 |
| `COMP` | Compliance, Limits & Audit | 12 |

Each test case includes: ID, title, category, severity, preconditions, steps, expected result, and (where relevant) notes explaining the rationale or pointing to related regulatory or industry references.

## Notable test cases

These are the test cases that go beyond functional coverage and demonstrate domain depth:

- **TC-KYC-009**: daily transaction limit cannot be bypassed by concurrent aggregation. Race condition test that catches non-atomic limit checks.
- **TC-AUTH-007**: SIM swap detection enforces a 48-hour transaction cooldown. SIM swap fraud is the single largest fraud vector in mobile money.
- **TC-CIN-007**: idempotent retry on cash-in does not double-credit. Network drops are routine in mobile data conditions and "retry on timeout" is the default client behavior, so without idempotency every transient failure becomes a duplicate transaction.
- **TC-MOV-006 to 007**: send-to-non-user escrow with expiration-return. How mobile money serves the unbanked while preserving ledger integrity.
- **TC-COUT-007 to 008**: agent commission calculation with banker's rounding and idempotent end-of-day settlement. Agent commission is the single most operationally complex part of a mobile money system.
- **TC-XBD-003**: FX margin disclosed transparently with both mid-market and platform rate visible. The trust-building feature that Wise built a brand on.
- **TC-COMP-001**: hash-chained transaction ledger with tamper-evidence verification. The test that proves the audit trail can survive forensic investigation.
- **TC-COMP-002**: daily reconciliation of the e-money conservation invariant. The fundamental rule that money is neither created nor destroyed in a closed-loop system.

## Scope and limitations

This repo is a design artifact. It does NOT include:

- An associated automation suite (test cases are documented for execution by a QA team, not executed by code in this repo)
- A real running mobile money platform (the system under test is fictional)
- Performance and load testing methodology (separate test plan)
- Penetration testing methodology (separate engagement)

What it DOES provide is a thorough, structured, regulator-aware test plan that demonstrates how to think about testing a regulated financial system end to end.

## References cited

The test plan cites the following frameworks and reference implementations:

- **PSD2** (EU): strong customer authentication
- **FATF AML/CFT** guidance
- **PCI DSS** scope considerations for card top-up
- **GSMA Mobile Money guidelines**
- **Bank of Ghana** mobile money operator regulations
- **OFAC, UN, EU, UK** sanctions lists
- **GhIPSS** (Ghana Interbank Payment and Settlement Systems) for interoperability
- **Wise** as the reference implementation for FX margin transparency
- **Stripe** as the reference implementation for idempotency keys

## About

Built and maintained by Jesse Jones, a frontend developer and QA engineer focused on FinTech and regulated financial systems. See the full [QA portfolio](https://github.com/JesseJonesJr/qa-portfolio) for other projects.
