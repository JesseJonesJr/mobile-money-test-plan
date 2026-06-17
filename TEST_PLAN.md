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

## Flow Index

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

## Flow 1: Account & KYC

12 test cases covering account creation, phone validation, KYC tier upgrades, identity uniqueness, biometric verification, dormancy handling, and one mobile-money-specific edge case (phone number normalization).

### TC-KYC-001: New user can register with a valid Ghanaian phone number

**Category:** Functional
**Severity:** Critical

**Preconditions:**

- Phone number `+233 24 123 4567` is valid and not previously registered
- A SIM matching the phone number is active in the device
- The mobile money app is installed and on the registration screen

**Steps:**

1. Tap "Create new account"
2. Enter phone number `+233 24 123 4567`
3. Submit the form
4. Receive SMS OTP within 60 seconds
5. Enter the OTP in the verification screen
6. Set a 4-digit PIN
7. Confirm the PIN

**Expected Result:** Account is created with status `active`, KYC tier `Basic`, balance `GHS 0.00`. User lands on the wallet home screen. A welcome SMS is sent confirming registration.

### TC-KYC-002: Registration is rejected for an already-registered phone number

**Category:** Negative
**Severity:** High

**Preconditions:**

- Phone number `+233 24 555 7777` is already linked to an active account
- User is on the registration screen

**Steps:**

1. Tap "Create new account"
2. Enter phone number `+233 24 555 7777`
3. Submit the form

**Expected Result:** Registration is blocked. User sees the message "This phone number is already registered. Please log in or use account recovery." No OTP is sent to the existing account. No new account is created.

**Notes:** Without this guard, account takeover via re-registration is possible.

### TC-KYC-003: Registration is rejected for an invalid phone number format

**Category:** Negative
**Severity:** Medium

**Preconditions:**

- User is on the registration screen

**Steps:**

1. Tap "Create new account"
2. Enter phone number `1234abc` (contains letters, too short)
3. Submit the form

**Expected Result:** Inline validation error: "Please enter a valid phone number." Submit button remains disabled until a valid format is entered. No SMS is sent. No backend request is made.

### TC-KYC-004: Registration is rejected for users under the legal age

**Category:** Compliance
**Severity:** Critical

**Preconditions:**

- User has completed phone verification and is in the KYC step where date of birth is entered
- Today's date is 2026-05-21

**Steps:**

1. Enter a date of birth that makes the user 17 years and 11 months old
2. Submit the form

**Expected Result:** KYC submission is rejected with the message "You must be 18 or older to open an account." The account remains in an incomplete state and no transactions are possible. The audit log records the failed underage attempt for compliance reporting.

**Notes:** Bank of Ghana and most regulators require legal majority for financial account opening. Failed attempts must be logged for AML reviews.

### TC-KYC-005: New account defaults to Basic tier with correct transaction limits

**Category:** Compliance
**Severity:** Critical

**Preconditions:**

- User has just completed registration (TC-KYC-001 passed)
- Tier limits are configured per current regulator guidance:
  - Basic: GHS 1,000 per transaction, GHS 5,000 daily, GHS 20,000 monthly

**Steps:**

1. Open the account profile screen
2. Open the transaction limits screen

**Expected Result:** Account tier displays as `Basic`. Transaction limits screen shows per-transaction GHS 1,000, daily GHS 5,000, monthly GHS 20,000. Attempting any transaction above these limits is blocked (covered by TC-KYC-009).

**Notes:** Default tier limits are a compliance control. Misconfiguration here means the platform either over-restricts users (UX issue) or under-restricts (regulatory violation).

### TC-KYC-006: User can upgrade from Basic to Standard tier with a valid national ID

**Category:** Functional
**Severity:** High

**Preconditions:**

- User has an active Basic tier account
- User has a valid Ghana Card with a clear, legible photo of the front and back
- User is on the "Upgrade KYC" screen

**Steps:**

1. Tap "Upgrade to Standard"
2. Take a photo of the front of the Ghana Card
3. Take a photo of the back of the Ghana Card
4. Take a selfie when prompted (liveness check)
5. Submit for verification

**Expected Result:** Submission is accepted. Status shows "Verification in progress" with a target SLA of under 5 minutes for automated review or under 24 hours for manual review. On approval, tier updates to `Standard` and new limits take effect immediately. User receives an SMS confirming the upgrade.

### TC-KYC-007: Tier upgrade fails gracefully when ID document is unreadable

**Category:** Negative
**Severity:** High

**Preconditions:**

- User has a Basic tier account
- User is on the KYC upgrade screen

**Steps:**

1. Tap "Upgrade to Standard"
2. Upload a blurry photo of an ID (image where text is illegible)
3. Submit for verification

**Expected Result:** Verification fails with the message "We could not read your document clearly. Please retake the photo in good lighting." User remains on Basic tier. No tier downgrade or account lock occurs. User can immediately retry.

**Notes:** Soft failure with retry is critical. A hard failure that locks the user out for hours after one bad photo upload damages trust without improving security.

### TC-KYC-008: A single national ID cannot be linked to more than one active account

**Category:** Compliance
**Severity:** Critical

**Preconditions:**

- A Standard tier account already exists, verified against Ghana Card `GHA-123456789-0`
- A new user is attempting a tier upgrade with the same Ghana Card `GHA-123456789-0` (different phone number)

**Steps:**

1. Submit a KYC upgrade with photos of Ghana Card `GHA-123456789-0`
2. Submit selfie

**Expected Result:** Upgrade is rejected with the message "This ID is already linked to another account. If this is an error, please contact support." Tier upgrade is blocked. The attempted match is logged for AML review. The original account is not affected or notified.

**Notes:** One identity = one account is a fundamental AML control. Multiple accounts per identity enables structuring (splitting large transactions to evade reporting thresholds).

### TC-KYC-009: Daily transaction limit cannot be bypassed by aggregating multiple smaller transactions

**Category:** Compliance + Concurrency
**Severity:** Critical

**Preconditions:**

- Basic tier user with daily limit GHS 5,000
- User has already sent GHS 4,500 in transactions today (multiple smaller transactions)
- User has GHS 10,000 in wallet balance

**Steps:**

1. Attempt to send GHS 700 to user A
2. Without waiting for confirmation, attempt to send GHS 700 to user B (simulating concurrent submission where both individually fit under remaining limit but combined exceed it)

**Expected Result:** Combined total of completed transactions cannot exceed GHS 5,000 for the day. Either one transaction succeeds for up to GHS 500 (the remaining limit) and the other is rejected, or both are rejected. The backend serializes the limit check atomically. The user receives a clear message: "This would exceed your daily limit. You have GHS 500 remaining today."

**Notes:** This is the structuring and race-condition test. Real-world attacks fire many simultaneous transactions hoping the limit check is non-atomic. Without an atomic check (database lock or transactional aggregate), users could bypass daily limits, which is both a compliance violation and a fraud vector.

### TC-KYC-010: Selfie liveness check fails when face does not match the ID photo

**Category:** Security
**Severity:** High

**Preconditions:**

- User is mid-tier-upgrade after submitting a valid Ghana Card
- The Ghana Card photo shows person A
- The user submitting the selfie is person B (different individual)

**Steps:**

1. Take the selfie as prompted
2. Submit

**Expected Result:** Verification fails with the message "We could not verify your identity. Please ensure you are submitting your own ID and try again." Tier upgrade is blocked. The mismatched attempt is logged with both image hashes for AML and fraud review. Excessive failed attempts (threshold configurable, e.g. 3 in 24 hours) temporarily lock further KYC submissions and trigger manual review.

**Notes:** Biometric face match is the primary defense against ID theft. Mismatch tolerance must be tight enough to catch impersonation but loose enough to accommodate normal variations (lighting, age, glasses). A confidence score threshold around 90 to 95% is typical.

### TC-KYC-011: Dormant accounts require re-verification before any transaction

**Category:** Compliance + Security
**Severity:** High

**Preconditions:**

- User account has had no logins or transactions for 13 months
- Account status is `dormant` (set by the inactivity job)
- User attempts to log in and immediately initiate a transaction

**Steps:**

1. Log in with phone number and PIN
2. Attempt to send GHS 100 to another user

**Expected Result:** Login succeeds but a re-verification screen appears before any transaction is allowed. User must complete a fresh KYC selfie (and optionally re-confirm ID). After successful re-verification, the account is reactivated and transactions proceed normally. Without re-verification, transactions are blocked with the message "Please reverify your identity to continue using your account."

**Notes:** Dormant account handling is a regulatory requirement in most jurisdictions. Dormant accounts are frequently targeted by fraudsters via account takeover with leaked credentials. Re-verification before money movement is the standard mitigation.

### TC-KYC-012: Phone number with leading zero is normalized correctly during registration

**Category:** Edge
**Severity:** Medium

**Preconditions:**

- User is on the registration screen
- Phone number `0244 123 456` (local Ghana format) is valid and not previously registered

**Steps:**

1. Enter phone number as `0244123456` (local format with leading zero, no country code)
2. Submit the form

**Expected Result:** The system normalizes the input to E.164 format (`+233244123456`), validates uniqueness against that normalized form, and proceeds with registration. The same number entered later as `+233 24 412 3456` should match the existing account, not allow a second registration.

**Notes:** Leading-zero versus international format is a common source of duplicate accounts in mobile money systems. If `0244123456` and `+233244123456` are treated as different keys, the same person can register twice. This breaks the "one identity per phone" guarantee.

## Flow 2: Authentication & Security

12 test cases covering login flows, account lockout and recovery, session management, new-device verification, SIM swap fraud detection, PIN change security, biometric versus PIN policy, step-up authentication for high-value transactions, and weak PIN rejection.

### TC-AUTH-001: User can log in with correct phone number and PIN

**Category:** Functional
**Severity:** Critical

**Preconditions:**

- Active Standard tier account exists with phone `+233 24 555 1234` and PIN `4729`
- User is on the login screen on their registered device

**Steps:**

1. Enter phone number `+233 24 555 1234`
2. Enter PIN `4729`
3. Submit

**Expected Result:** User is authenticated, session is established, and the wallet home screen displays the current balance. A login event is recorded in the audit log with device ID, IP address, and timestamp.

### TC-AUTH-002: Login fails with a generic message when PIN is incorrect

**Category:** Security
**Severity:** High

**Preconditions:**

- Account exists with phone `+233 24 555 1234`
- User is on the login screen

**Steps:**

1. Enter phone number `+233 24 555 1234`
2. Enter incorrect PIN `9999`
3. Submit

**Expected Result:** Login is rejected with the generic message "Phone or PIN is incorrect." The error message does NOT confirm whether the phone number exists in the system. The same generic message must appear for unknown phone numbers. The failed attempt is recorded with timestamp and device ID.

**Notes:** Generic error messages prevent account enumeration. A message like "Incorrect PIN for this account" confirms the phone is registered, which attackers exploit to target known accounts before attempting credential stuffing or social engineering.

### TC-AUTH-003: Account locks after 5 consecutive failed PIN attempts within 15 minutes

**Category:** Security
**Severity:** Critical

**Preconditions:**

- Account exists, currently unlocked
- Lockout policy is configured: 5 failures within a 15-minute rolling window triggers lockout

**Steps:**

1. Attempt login with incorrect PIN 5 times in rapid succession
2. Wait 1 minute
3. Attempt login with the CORRECT PIN

**Expected Result:** After the 5th failed attempt, the account is locked. The 6th attempt (even with the correct PIN) is rejected with the message "Your account is temporarily locked. Please use Account Recovery to regain access." The lockout event is logged. The user is NOT auto-unlocked by simply waiting (covered by TC-AUTH-004).

**Notes:** Lockout thresholds balance security against UX. Industry standard is 3 to 5 attempts. Too many attempts allowed creates brute-force risk. Too few locks out legitimate users on typos.

### TC-AUTH-004: Locked account cannot be unlocked by waiting; recovery is required

**Category:** Security
**Severity:** High

**Preconditions:**

- Account is locked due to failed PIN attempts (from TC-AUTH-003)
- 24 hours have passed since lockout

**Steps:**

1. Attempt login with the correct PIN

**Expected Result:** Login is still rejected. The user must complete the Account Recovery flow (TC-AUTH-011) to unlock. Time alone does not unlock the account.

**Notes:** Time-based auto-unlock allows persistent attackers to retry indefinitely on a slow-and-low schedule. Recovery-based unlock requires the legitimate account holder to prove identity, which raises the bar significantly.

### TC-AUTH-005: Session expires after 5 minutes of inactivity and requires re-authentication

**Category:** Security
**Severity:** High

**Preconditions:**

- User is logged in with an active session
- Idle timeout policy: 5 minutes

**Steps:**

1. Leave the app open without interaction for 5 minutes and 10 seconds
2. Return to the app and attempt to view balance or initiate a transaction

**Expected Result:** Session is invalidated. The user is prompted to re-enter PIN. The backend has already revoked the session token. Sensitive data (balance, transaction history) is not visible until re-authentication completes.

**Notes:** Mobile money sessions must be shorter than typical app sessions because the device may be shared, lost, or briefly unattended. 3 to 5 minutes is the industry standard for financial apps.

### TC-AUTH-006: Login on an unrecognized device requires step-up verification

**Category:** Security
**Severity:** Critical

**Preconditions:**

- Account is active on device A (recognized, previously used)
- User attempts login on device B (not seen before, different device ID)

**Steps:**

1. On device B, enter phone number and the correct PIN
2. Submit

**Expected Result:** Login is held in a pending state. The user receives an SMS OTP on the registered phone. The user must enter the OTP to complete login. After login, money movement is restricted for 24 hours (read-only access allowed). The user on device A receives a notification: "New login from device B in Accra at 14:32. Was this you?"

**Notes:** New-device login is one of the highest-signal indicators of account takeover. Many real-world breaches happen via leaked credentials reused on new devices. The 24-hour cooldown gives the legitimate user a window to detect and respond before funds can be drained.

### TC-AUTH-007: SIM swap on the registered phone triggers a transaction cooldown

**Category:** Security
**Severity:** Critical

**Preconditions:**

- Account active and verified
- Telecom integration is in place to detect SIM swaps (IMSI change for the registered phone number)
- A SIM swap event was detected for the registered number within the last 48 hours

**Steps:**

1. User attempts to log in (successfully or in an existing session)
2. User attempts to send GHS 500 to another user

**Expected Result:** Login is allowed for read-only operations (view balance, view history). All money movement (send, withdraw, bill pay) is blocked for 48 hours after the SIM swap event. The user sees: "We detected a recent change to your SIM. As a security measure, transactions are temporarily restricted. Please contact support if this was you." The block can only be lifted via in-person or video KYC re-verification with support.

**Notes:** SIM swap fraud is the single largest fraud vector in mobile money. An attacker who convinces a telco agent to port the victim's number to their SIM can receive OTPs, reset PINs, and drain wallets within minutes. This control alone prevents catastrophic losses. Industry leaders (Safaricom, MTN, Vodafone) all implement IMSI-change detection feeds. Cooldown windows of 24 to 72 hours are standard.

### TC-AUTH-008: PIN change requires current PIN AND OTP confirmation

**Category:** Security
**Severity:** High

**Preconditions:**

- User is logged in with an active session
- User is on the "Change PIN" screen

**Steps:**

1. Enter current PIN
2. Enter new PIN (twice, with confirmation)
3. Submit
4. Receive SMS OTP on the registered phone
5. Enter OTP to confirm the change

**Expected Result:** PIN is changed only after both factors verify successfully. The old PIN is invalidated immediately. The user receives a confirmation SMS: "Your MoMo PIN was changed at 14:32. If this wasn't you, call 100 immediately." All active sessions on other devices are terminated.

**Notes:** Two-factor PIN change is critical because PIN is the primary auth credential. Single-factor change (just current PIN) means anyone who briefly accesses an unlocked phone can lock the legitimate owner out. The SMS confirmation also serves as a tripwire so the legitimate user can detect unauthorized changes within minutes.

### TC-AUTH-009: Biometric login is permitted, but high-value transactions still require PIN

**Category:** Security
**Severity:** High

**Preconditions:**

- User has enabled fingerprint authentication for app login
- High-value transaction threshold is configured at GHS 500
- User is logged in via fingerprint

**Steps:**

1. Send GHS 100 to another user (below threshold)
2. Send GHS 1,000 to another user (above threshold)

**Expected Result:** First transaction completes after fingerprint confirmation. Second transaction prompts for PIN entry before proceeding, regardless of the fingerprint already being verified for app open. PIN entry happens at the transaction step, not as a full re-login.

**Notes:** Biometric convenience is fine for low-stakes operations but biometrics can be bypassed (sleeping partner's finger, photo unlock on weaker face systems). PIN-required step-up for sensitive operations balances usability with risk. The threshold is configurable per regulation or per platform policy.

### TC-AUTH-010: Transaction above threshold prompts PIN re-entry even within active session

**Category:** Security
**Severity:** High

**Preconditions:**

- User is logged in with an active PIN-based session (no biometric in use)
- High-value threshold: GHS 500
- User sent GHS 100 to user A 30 seconds ago (PIN was entered at login)

**Steps:**

1. Initiate send of GHS 1,000 to user B
2. Confirm recipient and amount

**Expected Result:** PIN re-entry is prompted before the transaction completes. Even though the session is active and a transaction just occurred, the high-value tier requires fresh PIN entry. After successful PIN entry, the transaction proceeds.

**Notes:** Session-only auth means anyone who grabs an unlocked phone can drain the wallet. Per-transaction PIN for high-value transactions is the standard mitigation. Threshold can be tier-dependent: higher KYC tiers may have higher step-up thresholds because the legitimate user is expected to transact larger amounts more often.

### TC-AUTH-011: Forgot-PIN recovery requires multi-step identity verification

**Category:** Security
**Severity:** Critical

**Preconditions:**

- Account exists with verified KYC (Standard tier)
- User taps "Forgot PIN" on the login screen

**Steps:**

1. Enter phone number
2. Receive and enter OTP
3. Submit a fresh selfie (matched against the KYC photo on file)
4. Answer at least one security question (set at KYC time or derived from transaction history, e.g. "What was the amount of your last received transaction?")
5. Set a new PIN

**Expected Result:** Recovery succeeds only if all steps pass: OTP delivered to the current SIM, selfie matches KYC photo with high confidence, security answer correct. PIN is reset. All sessions are terminated. The user receives a confirmation SMS. Failed recovery attempts are rate-limited (max 3 per day) and logged for fraud review.

**Notes:** OTP alone is insufficient because of SIM swap risk. Selfie match defeats SIM-swap-only attacks because the attacker has the SIM but not the face. The security question adds a third factor that resists shoulder surfing and casual social engineering. This three-factor recovery is the strongest reasonable barrier short of in-person verification.

### TC-AUTH-012: Weak PINs are rejected at creation and at change

**Category:** Security + Edge
**Severity:** Medium

**Preconditions:**

- User is creating a PIN during registration OR changing PIN
- Weak-PIN policy is configured to reject:
  - All-same digits (0000, 1111, 2222, etc.)
  - Simple sequences (1234, 4321, 9876, 0987)
  - The top 10 most common PINs from public breach datasets
  - The user's date of birth (DDMM or YYYY)
  - The last 4 digits of the user's registered phone number

**Steps:**

1. Attempt to set PIN `1234`
2. After rejection, attempt `1111`
3. After rejection, attempt the last 4 digits of the registered phone number
4. After rejection, attempt a non-trivial PIN like `4729`

**Expected Result:** The first three attempts are rejected with the message "Please choose a stronger PIN. Avoid simple patterns or numbers tied to your identity." The fourth attempt (4729) is accepted. The rejection happens client-side for UX AND is validated server-side so the client cannot be bypassed.

**Notes:** Weak PINs are a major fraud vector. Research on breached PIN datasets shows roughly 10% of users default to "1234" or "0000" if not blocked. Banning common patterns dramatically reduces guessing attacks. The phone-last-4-digits check is especially important because attackers who get the phone number get a high-probability PIN guess for free.

## References

This plan is informed by, but not formally compliant with, the following frameworks. Mentioned to make the regulatory awareness explicit:

- **PSD2** (EU): strong customer authentication requirements
- **AML/CFT** guidance from FATF
- **PCI DSS** scope considerations (for card top-up integration)
- **GSMA Mobile Money guidelines**
- **Bank of Ghana** mobile money operator regulations (for local context)

---

*Test cases for Flows 3 through 7 are pending and will be added incrementally.*
