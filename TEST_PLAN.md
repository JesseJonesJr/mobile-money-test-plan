# Test Plan – Mobile Money Platform

## Overview

End-to-end test plan for a mobile money platform supporting peer-to-peer transfers, cash-in and cash-out via agents, bill payments, and cross-border remittance. The plan emphasizes the domain-specific concerns of mobile money: money math precision, transaction atomicity, idempotency, agent commission flows, tiered KYC compliance, and offline transaction handling.

This test plan was authored as a portfolio demonstration of FinTech QA depth. The platform under test is fictional but designed to mirror real-world mobile money systems operating in emerging markets (e.g. M-Pesa, MTN MoMo, AirtelTigo Money, Wave).

## System Under Test

A mobile money platform with:

- User accounts identified by phone number
- Tiered KYC (3 tiers: Basic, Standard, Premium, with increasing transaction limits)
- Wallet balance held in local currency (GHS for default test scope)
- Agent network for cash-in and cash-out
- Real-time peer-to-peer transfers (on-network and off-network)
- Bill payment and merchant payment integrations
- Cross-border remittance with FX conversion
- Transaction history, statements, and audit trails

## Scope

**In scope:**

- All flows listed below
- Functional, negative, security, compliance, and data integrity testing
- Money math precision and rounding behavior
- Concurrency and atomicity at the transaction level

**Out of scope:**

- Performance and load testing (separate test plan)
- Penetration testing (separate engagement)
- Mobile app UI testing (this plan is API and system-level)
- Telecom carrier integration testing (would require carrier test environments)

## Conventions

### Test Case ID Format

`TC-{FLOW}-{NUMBER}` (e.g. `TC-KYC-001`, `TC-CIN-014`)

Flow codes:

| Code | Flow |
| --- | --- |
| `KYC` | Account & KYC |
| `AUTH` | Authentication & Security |
| `CIN` | Money In (cash-in, deposits) |
| `MOV` | Money Movement (P2P, bills, merchants) |
| `COUT` | Money Out (cash-out, withdrawals) |
| `XBD` | Cross-Border & FX |
| `COMP` | Compliance, Limits & Audit |

### Severity Scale

| Level | Definition |
| --- | --- |
| **Critical** | Money loss, security breach, data corruption, regulatory violation |
| **High** | Major feature broken, significant user impact, partial money issues |
| **Medium** | Minor feature issue, workaround exists, low user impact |
| **Low** | Cosmetic, minor edge case, nice-to-have |

### Test Case Categories

- **Functional**: happy-path validation that the system does what it should
- **Negative**: the system fails correctly when it should fail
- **Edge**: boundary values, unusual inputs, limit testing
- **Security**: authentication, authorization, fraud prevention
- **Compliance**: regulatory and policy enforcement (KYC, AML, sanctions)
- **Concurrency**: race conditions, double-spend prevention, atomicity
- **Money Math**: decimal precision, rounding, currency-specific behavior
- **Integration**: cross-system interactions (banks, agents, billers)

### Test Case Structure

Each test case follows this structure:

- **ID:** `TC-{FLOW}-{NUMBER}` (e.g. `TC-KYC-001`)
- **Title:** short descriptive title
- **Category:** one of the categories listed above
- **Severity:** Critical, High, Medium, or Low
- **Preconditions:** bulleted list of state required before the test
- **Steps:** numbered list of actions to perform
- **Expected Result:** what should happen
- **Notes:** *(optional)* rationale, related compliance reference, or edge case explanation

Example:

> **TC-KYC-001: User can register with a valid phone number**
>
> **Category:** Functional
> **Severity:** Critical
>
> **Preconditions:**
>
> - Phone number is valid and not previously registered
> - Mobile money app is installed on the device
>
> **Steps:**
>
> 1. Open the app and select "Register"
> 2. Enter the phone number
> 3. Submit the form
>
> **Expected Result:** User receives an SMS verification code and is taken to the code verification screen.

## Flows

The plan covers seven flows. Each flow contains approximately 10 test cases for a total of ~70 cases.

### 1. Account & KYC (`KYC`)

Account registration, identity verification, KYC tier upgrades, tier-based feature gating.

### 2. Authentication & Security (`AUTH`)

PIN authentication, biometric, session management, account recovery, suspicious-activity lockout.

### 3. Money In (`CIN`)

Cash-in via agent, bank deposit, card top-up, idempotency on retries, deposit reversal handling.

### 4. Money Movement (`MOV`)

P2P transfers on-network and off-network, bill payments, merchant payments, send-to-non-user flows.

### 5. Money Out (`COUT`)

Cash-out via agent, bank withdrawal, withdrawal limits, agent commission accounting.

### 6. Cross-Border & FX (`XBD`)

International transfers, currency conversion math, FX margin handling, sanctioned-country blocks.

### 7. Compliance, Limits & Audit (`COMP`)

KYC tier transaction limits, AML alert triggers, sanctions screening, transaction history immutability, reconciliation.

## References

This plan is informed by, but not formally compliant with, the following frameworks. Mentioned to make the regulatory awareness explicit:

- **PSD2** (EU): strong customer authentication requirements
- **AML/CFT** guidance from FATF
- **PCI DSS** scope considerations (for card top-up integration)
- **GSMA Mobile Money guidelines**
- **Bank of Ghana** mobile money operator regulations (for local context)

---

*Test cases for each flow are written in the sections below as they are authored.*
