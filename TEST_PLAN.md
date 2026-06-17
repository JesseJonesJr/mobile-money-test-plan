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

## Flow 3: Money In

12 test cases covering agent cash-in, bank-to-wallet pulls, card top-up with 3DS, atomicity across user and agent ledgers, idempotency on network retries, wallet balance ceilings, reversal windows, and immutable transaction history.

### TC-CIN-001: Agent deposits a valid amount to user wallet (happy path)

**Category:** Functional
**Severity:** Critical

**Preconditions:**

- Agent has an active agent account with float balance of GHS 5,000
- User is a Standard tier customer with balance GHS 100 (well below wallet ceiling)
- Agent and user are physically present at the agent's location

**Steps:**

1. Agent opens the agent app and selects "Deposit"
2. Agent enters user's phone number `+233 24 555 1234`
3. Agent enters amount GHS 500
4. Agent presses "Confirm"
5. User's phone receives a push notification or USSD prompt: "Confirm deposit of GHS 500 from Agent KOJO ENT (#A12345). Enter your PIN to approve."
6. User enters their PIN on their own device to authorize the transaction

**Expected Result:** Transaction completes within 5 seconds of user PIN entry. User balance updates from GHS 100 to GHS 600. Agent float updates from GHS 5,000 to GHS 4,500. Both user and agent receive a confirmation SMS with the transaction reference. The transaction is logged in both user and agent histories with status "Completed".

**Notes:** The user authorizes the transaction with their PIN on their own device. The agent never sees the user's PIN and the user never shares any credential with the agent. This is the standard mobile money pattern in Ghana (MTN MoMo, AirtelTigo, Telecel) and most African markets, and it is the strongest of the common agent-deposit confirmation patterns. The agent cannot fake a deposit because the money only moves if the user actively approves on their own phone. Some other markets and platforms use QR-scan or OTP-share patterns; this test plan adopts the PIN-on-user-device pattern as the reference.

### TC-CIN-002: Cash-in updates user wallet and agent float atomically

**Category:** Money Math + Concurrency
**Severity:** Critical

**Preconditions:**

- Agent float balance = GHS 5,000
- User balance = GHS 100
- A controlled failure point is injected after the user wallet update but before the agent float update (test environment)

**Steps:**

1. Agent submits a GHS 500 deposit to the user
2. System updates user balance to GHS 600
3. System encounters the injected failure before updating agent float

**Expected Result:** Both updates roll back. User balance returns to GHS 100. Agent float remains GHS 5,000. The transaction is recorded as "Failed" with a rollback reason in the audit log. No state ever exists where user got money but the agent's float was not debited (or vice versa).

**Notes:** Mobile money ledgers must enforce all-or-nothing semantics across multiple account updates. Without transactional integrity (database transactions, sagas with compensating actions, or event-sourced ledgers), partial failures create money out of thin air or destroy it. This is one of the most common sources of catastrophic accounting drift in poorly-engineered FinTech systems.

### TC-CIN-003: Agent cannot deposit more than their float balance

**Category:** Negative
**Severity:** High

**Preconditions:**

- Agent float balance = GHS 200
- User account is active and able to receive

**Steps:**

1. Agent attempts to deposit GHS 500 to the user

**Expected Result:** Deposit is rejected with the message "Insufficient float balance. Please top up your float before processing more deposits." User balance is unchanged. Agent float is unchanged. No transaction is recorded except a "Failed: insufficient float" entry in the agent's transaction history.

**Notes:** Agent float is the agent's pre-funded liquidity. Each cash-in deducts from the float, each cash-out adds to it. Without this check, agents could "deposit" beyond their float, which means platform-side money is created without corresponding cash backing. That is effectively printing money and is the kind of failure that ends partnerships with regulators.

### TC-CIN-004: User receives SMS confirmation immediately on successful deposit

**Category:** Functional
**Severity:** Medium

**Preconditions:**

- A successful agent deposit just completed (TC-CIN-001 passed)
- User's registered phone has active SMS service

**Steps:**

1. Complete a successful deposit
2. Wait up to 60 seconds for SMS delivery

**Expected Result:** SMS arrives within 60 seconds. Message includes: transaction type (Cash-in), amount, agent name and ID, timestamp, new balance, and transaction reference number. Example: "You received GHS 500 from Agent KOJO ENT (#A12345) on 17/06/2026 at 14:32. New balance: GHS 600. Ref: TXN78451234. Was this not you? Call 100."

**Notes:** Mobile money users frequently rely on SMS as their primary record of transactions. Many users transact via USSD and do not even open the app during a transaction. The SMS IS the receipt. The "was this not you" line is a critical fraud tripwire that prompts the user to act immediately if the transaction was unauthorized.

### TC-CIN-005: Bank-to-wallet transfer succeeds for a valid linked bank account

**Category:** Functional
**Severity:** Critical

**Preconditions:**

- User has linked a bank account with valid Open Banking authorization (token not expired)
- Linked bank account has GHS 1,000 available balance
- User is on the "Add Money" screen and has selected "From Bank"

**Steps:**

1. Select the linked bank account
2. Enter amount GHS 500
3. Confirm
4. Complete bank-side authentication (in-app push approval or OTP)

**Expected Result:** Bank account debits GHS 500. Wallet credits GHS 500. Both sides are reconciled within 60 seconds (subject to bank settlement speed). User receives SMS and in-app notification. Transaction is logged with the source identifier (bank name, masked account number) and a separate bank transaction reference.

### TC-CIN-006: Card top-up requires 3D Secure authentication for first-time card use

**Category:** Security
**Severity:** High

**Preconditions:**

- User is adding a new Visa or Mastercard for the first time on this platform
- Card is enrolled in 3DS (most cards in Ghana are via GhIPSS and the issuer)

**Steps:**

1. Enter card number, expiry, CVV
2. Enter top-up amount GHS 200
3. Submit
4. 3DS challenge page loads on the issuer's domain
5. Complete the challenge (typically OTP sent to the bank-registered phone or biometric in the bank app)
6. Return to the platform

**Expected Result:** The 3DS challenge MUST appear before the transaction processes. If user fails 3DS or cancels, no charge is made and no credit appears in the wallet. On successful 3DS, the card is charged GHS 200 and the wallet credits GHS 200. The card is tokenized for future use, but subsequent transactions may also require 3DS depending on amount and issuer policy.

**Notes:** 3DS (SCA in Europe) shifts liability for fraudulent transactions from the merchant to the issuer. Without 3DS, fraudulent card top-ups become chargebacks against the platform and can put the card-processing relationship with the acquirer at risk. Some markets allow exemptions for low-value or recurring transactions, but first-time use should always require it.

### TC-CIN-007: Idempotent retry on cash-in does not double-credit the user

**Category:** Money Math + Concurrency
**Severity:** Critical

**Preconditions:**

- Agent float balance = GHS 5,000
- User balance = GHS 100
- Network conditions are unreliable (test environment simulates packet loss on response path)
- The deposit API requires an idempotency key per request

**Steps:**

1. Agent submits a GHS 500 deposit with idempotency key `idem-abc-001`
2. Server processes the deposit successfully: user balance becomes GHS 600, agent float becomes GHS 4,500
3. The HTTP response is lost in transit (server-side success, client-side timeout)
4. Agent's client automatically retries the same deposit with the same idempotency key `idem-abc-001`
5. Server receives the retry

**Expected Result:** Server recognizes idempotency key `idem-abc-001` as already-processed. Server returns the cached response of the original successful transaction WITHOUT re-executing the deposit. User balance remains GHS 600 (NOT GHS 1,100). Agent float remains GHS 4,500 (NOT GHS 4,000). Only ONE transaction is recorded in the audit log. Both the original response and the retry response refer to the same transaction reference number.

**Notes:** This is the most operationally important test in the entire cash-in flow. Network drops are routine in mobile data conditions, especially in markets where users rely on 3G/4G with weak signal. Without idempotency, the natural client behavior of "retry on timeout" creates duplicate transactions. The user gets credited twice and the agent's float depletes twice for one cash payment. Multiplied across millions of transactions per day, this is how platforms lose serious money to operational drift. Industry standard: idempotency keys with at least 24-hour retention, returning the original response (status code and body) on retry. The Stripe API is the public reference implementation of this pattern.

### TC-CIN-008: Deposit is blocked when wallet would exceed tier balance ceiling

**Category:** Compliance
**Severity:** High

**Preconditions:**

- User is Basic tier
- Basic tier wallet balance ceiling = GHS 10,000 (distinct from per-transaction or daily/monthly transaction limits)
- User current balance = GHS 9,500
- Agent has sufficient float

**Steps:**

1. Agent attempts to deposit GHS 800 (would bring balance to GHS 10,300, above ceiling)

**Expected Result:** Deposit is rejected with the message "This deposit would exceed the recipient's wallet balance limit. The customer can receive up to GHS 500 more, or should upgrade to Standard tier for a higher limit." Agent float is not debited. No partial deposit is made. The user can still receive deposits up to the available headroom (GHS 500 in this case).

**Notes:** Wallet balance ceilings exist separately from transaction limits and are a regulatory compliance feature. They limit the amount of money mobile money operators hold for any one customer at any one time, both to limit consumer exposure and to manage the operator's prudential reserve requirements with the central bank.

### TC-CIN-009: Agent can reverse a deposit within the 15-minute correction window

**Category:** Functional
**Severity:** High

**Preconditions:**

- Agent completed a GHS 500 deposit to user 3 minutes ago
- Reversal window policy: agent can self-initiate reversal within 15 minutes of the original transaction, conditional on the recipient not having spent the funds
- User has not initiated any transaction since the deposit

**Steps:**

1. Agent opens the transaction history
2. Selects the recent deposit
3. Taps "Reverse"
4. Confirms the reversal by entering agent PIN

**Expected Result:** Reversal completes immediately. User balance decreases by GHS 500. Agent float increases by GHS 500. Both parties receive SMS notification of the reversal. The original transaction is marked "Reversed" with a link to the reversal transaction. The reversal itself cannot be reversed. If the user had spent any portion of the deposit, the reversal is blocked with the message "Cannot reverse: funds have been used. Escalate to support."

### TC-CIN-010: Reversal beyond the 15-minute window requires recipient consent

**Category:** Security
**Severity:** High

**Preconditions:**

- Agent completed a GHS 500 deposit to user 30 minutes ago
- The 15-minute reversal window has elapsed
- User still has the funds available (not spent)

**Steps:**

1. Agent initiates reversal via the support escalation flow
2. System sends a consent request to the recipient: "Agent KOJO ENT is requesting to reverse a GHS 500 deposit made at 14:32. Approve / Deny?"
3. User responds via app or SMS reply

**Expected Result:** Reversal does NOT proceed without user consent. If user approves, reversal executes (same flow as TC-CIN-009). If user denies, reversal is blocked and a support ticket is created for manual review. If user does not respond within 24 hours, the request expires and the reversal must be processed manually through support with documented justification.

**Notes:** This control prevents abusive reversal by agents. Without consent, an agent could effectively cash someone out indirectly by depositing then reversing after the user has already spent money in reliance on the deposit. The consent step puts the user in control of their wallet state.

### TC-CIN-011: Bank deposit failure does not partial-credit the wallet

**Category:** Money Math + Negative
**Severity:** Critical

**Preconditions:**

- User's linked bank account has only GHS 100 available
- User attempts to deposit GHS 500 from the bank

**Steps:**

1. Initiate bank-to-wallet transfer of GHS 500
2. Authenticate with the bank
3. Bank API returns "INSUFFICIENT_FUNDS" response

**Expected Result:** Wallet balance is unchanged (no partial credit of GHS 100). Bank account balance is unchanged (no partial debit). User sees a clear error: "Your bank reported insufficient funds. Please check your account balance and try again." The failed transaction is logged but does not appear in normal transaction history (or is clearly marked "Failed - Not Processed").

**Notes:** Partial credit is the classic mobile money money-math failure. If the platform credits whatever portion the bank approved but logs the user-requested amount, the ledger is corrupted. If the platform debits the bank but the wallet credit fails, money disappears. The cleanest pattern is: bank either authorizes the full amount or nothing at all, and the wallet credit happens AFTER successful bank debit confirmation.

### TC-CIN-012: Deposit transaction is recorded immutably with full audit details

**Category:** Compliance
**Severity:** High

**Preconditions:**

- A successful deposit just completed (any channel: agent, bank, or card)

**Steps:**

1. Open the user's transaction history
2. Find the deposit transaction
3. Open the transaction detail view

**Expected Result:** Transaction record displays: transaction ID (system-generated unique reference), channel (Cash-in via agent / Bank transfer / Card top-up), amount and currency, source identifier (agent ID and name, or bank name and masked account, or card last 4), timestamp in both UTC and the user's local timezone, status "Completed", balance before and balance after, and a cryptographic hash linking this transaction to the previous one in the ledger (tamper-evidence). The record cannot be edited or deleted by any user role including platform admins. Only annotations can be added (e.g. "Customer disputed, reviewed and confirmed valid"). Any modification attempt is logged separately as an audit event.

**Notes:** Immutability is required by AML regulations and is essential for dispute resolution. Many jurisdictions require transaction records to be retained for 5 to 7 years. Hash-chained or append-only logs (similar in concept to blockchain or write-once storage) make tampering detectable even by privileged insiders.

## Flow 4: Money Movement

12 test cases covering on-network P2P transfers, off-network interoperability transfers, send-to-non-user with escrow and expiration-return, bill payments with biller-failure refund, merchant payments, recipient name verification, duplicate transfer detection, and tier-based fee calculation.

### TC-MOV-001: P2P transfer to another user on the same network completes successfully

**Category:** Functional
**Severity:** Critical

**Preconditions:**

- Sender: Standard tier, balance GHS 1,000, daily limit GHS 10,000 (untouched today)
- Recipient: Standard tier, balance GHS 200, account active and not blocked
- Both users on the same mobile money network

**Steps:**

1. Sender opens "Send Money"
2. Enters recipient phone number `+233 24 555 7777`
3. System displays the recipient's masked name as registered (e.g. "Kojo A.")
4. Sender confirms recipient and enters amount GHS 300
5. System displays a transaction summary including the applicable fee
6. Sender enters PIN to confirm

**Expected Result:** Transfer completes within 3 seconds. Sender balance decreases by GHS 300 plus the fee. Recipient balance increases by GHS 300 (recipient does not pay receive fees on-network). Both receive SMS confirmation. A transaction reference is generated and recorded in both transaction histories.

### TC-MOV-002: P2P transfer fails cleanly when sender has insufficient balance

**Category:** Money Math + Negative
**Severity:** Critical

**Preconditions:**

- Sender balance: GHS 100
- Sender attempts to send GHS 300 plus fee

**Steps:**

1. Initiate send of GHS 300 to a valid recipient
2. Confirm with PIN

**Expected Result:** Transfer is rejected with the message "Insufficient balance. You have GHS 100 available." Sender balance remains GHS 100. No funds are deducted, not even partially. Recipient balance is unchanged and no notification is sent. Transaction is logged as "Failed: insufficient balance" in the sender's transaction history with no impact on daily limit counters.

**Notes:** Partial debits on failed transfers are one of the most common money-math bugs in poorly-engineered wallets. The balance check must happen BEFORE any state mutation, and ideally inside the same atomic database transaction that performs the debit.

### TC-MOV-003: P2P transfer requires fresh PIN entry, even within an active session

**Category:** Security
**Severity:** High

**Preconditions:**

- User is logged in with an active PIN-based session
- User completed a transfer 1 minute ago

**Steps:**

1. Initiate a new transfer to a different recipient
2. Confirm recipient and amount
3. Reach the PIN entry screen

**Expected Result:** PIN entry is required for every transfer, regardless of session age or recent transactions. There is no "remember PIN for this session" option. After PIN entry, transfer proceeds. After 3 incorrect PIN entries on a single transfer, the transfer is cancelled and the user is returned to the home screen (without locking the entire account, unless the global lockout counter reaches its threshold per TC-AUTH-003).

**Notes:** Per-transaction PIN entry is the floor of mobile money security. The PIN is the "what you know" factor that, combined with possession of the SIM, provides two-factor authentication for each transfer. Skipping PIN even once breaks this guarantee.

### TC-MOV-004: Recipient name is displayed before confirmation to prevent typo errors

**Category:** Functional
**Severity:** High

**Preconditions:**

- Recipient `+233 24 555 7777` is registered as "Kojo Asante"
- Sender enters `+233 24 555 7778` (off by one digit), which is registered as "Ama Mensah"

**Steps:**

1. Sender enters phone number `+233 24 555 7778`
2. System looks up the recipient

**Expected Result:** System displays the recipient's masked name (e.g. "Ama M." or "A** M*****") before allowing sender to proceed. The sender can clearly see this is NOT the intended recipient and can cancel or correct the number. The mask balances privacy (avoid revealing full PII to anyone with a phone number) with verification (sender can confirm the right person).

**Notes:** Sending to the wrong number is the single most common mobile money user error. Recipient name display is a UX-level fraud and error prevention control. The mask format varies: some platforms show first name and last initial, others mask everything but a few characters. The mask must be informative enough to verify and opaque enough to prevent name enumeration attacks.

### TC-MOV-005: Off-network transfer to a user on a different mobile money network

**Category:** Integration
**Severity:** High

**Preconditions:**

- Sender on platform A (this platform)
- Recipient on platform B (different operator), recipient phone is registered and active
- Cross-network interoperability is enabled (in Ghana, via GhIPSS Mobile Money Interoperability switch)

**Steps:**

1. Sender initiates send to recipient on platform B
2. System detects the recipient is off-network and displays the cross-network fee (typically higher than on-network)
3. Sender confirms with PIN

**Expected Result:** Transfer routes through the interoperability switch. Sender wallet debits the amount plus the cross-network fee. Recipient receives the amount in their platform B wallet, typically within 30 seconds. Both sender and recipient receive SMS confirmation. The transaction is logged with the cross-network indicator and the interoperability switch reference number.

**Notes:** Cross-network transfers depend on the interoperability switch (GhIPSS MMI in Ghana, similar systems in other markets). The switch may be slower than on-network transfers (30 seconds to several minutes versus sub-second). Tests should verify behavior at the boundary, including timeout handling if the switch is slow to respond and split-brain handling if the switch cannot confirm delivery.

### TC-MOV-006: Send-to-non-user creates an escrow with a redemption code

**Category:** Functional + Edge
**Severity:** High

**Preconditions:**

- Sender balance: GHS 1,000
- Recipient phone number `+233 24 999 0001` is NOT registered with any mobile money platform

**Steps:**

1. Sender enters recipient phone `+233 24 999 0001`
2. System recognizes the number is not registered and offers "Send to non-user"
3. Sender confirms intent, enters amount GHS 500, and confirms with PIN

**Expected Result:** Funds are debited from sender's wallet (GHS 500 plus applicable fee) and held in an escrow account, NOT credited to anyone yet. Recipient receives an SMS: "Kojo A. sent you GHS 500. Visit any Agent with your phone, ID, and this code: 8472-3691 to collect. Or download the app and register your number to receive directly. Code expires in 7 days." Sender receives confirmation: "GHS 500 held for recipient. They have 7 days to claim. If unclaimed, funds will return to your wallet." Both records reference the escrow ID.

**Notes:** Send-to-non-user is one of mobile money's killer features for financial inclusion. It lets banked users transfer to unbanked recipients without requiring upfront registration. The escrow pattern is essential: money is debited from sender but not credited until recipient claims, so the platform tracks an in-flight liability rather than money that "exists" in any wallet. Without this, the platform either delays sender debit (sender doesn't know if the transfer succeeded) or credits nowhere (loses money). The 7-day window is industry standard, with codes typically 6 to 10 digits to balance memorability against guessing risk.

### TC-MOV-007: Unclaimed send-to-non-user funds return to sender after expiration

**Category:** Money Math + Functional
**Severity:** Critical

**Preconditions:**

- An escrow transaction from TC-MOV-006 is 7 days and 1 minute old
- Recipient never registered or visited an agent to claim
- Sender balance is currently GHS 500 (originally GHS 1,000 before the held send, after fees)

**Steps:**

1. Background reconciliation job runs at its configured interval (e.g. hourly)

**Expected Result:** The expired escrow is returned to the sender's wallet. Sender balance goes from GHS 500 back to GHS 1,000 (with the principal returned; fees may or may not be refunded depending on policy). Sender receives SMS: "The GHS 500 you sent to +233 24 999 0001 was not claimed. Funds have been returned to your wallet." The escrow record is marked "Returned to sender" with the return timestamp and reconciliation job ID. The redemption code is invalidated and cannot be used.

**Notes:** Return-to-sender on expiration is critical for two reasons: the user trusts that unclaimed money is not lost forever, and the platform avoids accumulating untracked escrow liabilities on its books. Fee policy varies: some platforms refund fees on returns, others keep them as a processing cost. Either is defensible if disclosed clearly upfront. The reconciliation job must be idempotent: if it runs twice on the same expired escrow (e.g. due to job retry on transient failure), the funds are returned exactly once, not twice.

### TC-MOV-008: Bill payment to a registered biller completes with reference number

**Category:** Integration
**Severity:** High

**Preconditions:**

- User has GHS 200 balance
- Biller (e.g. ECG electricity) is integrated with the platform
- User has the biller account number (e.g. meter number) ready

**Steps:**

1. Open "Pay Bills", select "Electricity", select "ECG"
2. Enter meter number
3. System fetches account details from the biller API (account holder name, current balance or amount due if applicable)
4. User confirms the account
5. Enter amount GHS 150
6. Confirm with PIN

**Expected Result:** Wallet debits GHS 150 plus any service fee. Biller API is called with the payment instruction. Biller returns a confirmation reference (e.g. ECG receipt number). User sees the success screen with: amount paid, biller reference, meter number, and timestamp. SMS confirmation is sent. Transaction is logged with both the platform reference and the biller reference for reconciliation.

### TC-MOV-009: Failed bill payment auto-refunds the wallet within 60 seconds

**Category:** Money Math + Negative
**Severity:** Critical

**Preconditions:**

- User has GHS 200 balance
- Bill payment to a biller is initiated; the biller API will return an error mid-transaction (test environment)

**Steps:**

1. Initiate bill payment of GHS 150
2. PIN confirmation passes
3. Wallet debit succeeds (GHS 200 becomes GHS 50)
4. Biller API call fails with a clean error (e.g. "BILLER_TIMEOUT" or "ACCOUNT_NOT_FOUND")

**Expected Result:** Within 60 seconds of biller failure, the platform auto-reverses the wallet debit. Balance returns from GHS 50 to GHS 200. User receives SMS: "Your bill payment of GHS 150 failed. Funds have been returned to your wallet." If the platform cannot determine within the timeout whether the biller succeeded or failed (network split-brain), the transaction is held in a "Pending reconciliation" state and a manual review is queued. Auto-refund only fires on a clean failure response.

**Notes:** Bill payments are notorious for ambiguous failure modes. The biller may have received and processed the payment but failed to respond. Auto-refunding in that case would double-pay the bill (or worse, refund the user without the bill ever being paid, then the biller posts it later, creating a negative balance). The safest pattern is: clean error response triggers auto-refund; timeout or no response holds the funds in a reconciliation state until automated or manual reconciliation with the biller confirms which side has the money.

### TC-MOV-010: Merchant payment via merchant code completes successfully

**Category:** Functional
**Severity:** High

**Preconditions:**

- Merchant is registered with the platform and has a merchant code (e.g. `MOMO-12345`) or QR code
- User has sufficient balance

**Steps:**

1. Open "Pay Merchant", enter merchant code OR scan QR
2. System displays the merchant name and category
3. Enter amount (or amount is pre-filled if the QR is a static-amount code)
4. Confirm with PIN

**Expected Result:** Wallet debits the amount. Merchant wallet credits the amount minus the merchant processing fee (typically 0.5% to 1% in Ghana). User sees the success screen with merchant name, amount, and transaction reference. Both user and merchant receive SMS notifications. The transaction is logged with the merchant category code (MCC) for the merchant's reconciliation and for the platform's reporting.

**Notes:** Merchant payments differ from P2P transfers in that the merchant pays the processing fee out of the received amount, similar to card payment processing. This fee structure is what makes mobile money economically sustainable for the platform. Merchants in Ghana generally accept this fee in exchange for the volume and convenience of mobile money over cash.

### TC-MOV-011: Duplicate transfer within 30 seconds prompts confirmation

**Category:** Security + Edge
**Severity:** High

**Preconditions:**

- Sender just sent GHS 100 to `+233 24 555 7777` (recipient Kojo Asante) 20 seconds ago

**Steps:**

1. Sender initiates another send of GHS 100 to `+233 24 555 7777`
2. Reaches the confirmation screen

**Expected Result:** Before requesting PIN, the system displays a warning: "You sent GHS 100 to Kojo A. 20 seconds ago. Are you sure you want to send the same amount again?" with options "Yes, continue" and "No, cancel." Only if the user confirms does the transaction proceed to PIN entry. If the user cancels, no transaction is initiated. The detection window (30 seconds), the amount-and-recipient match criteria, and the warning text are configurable.

**Notes:** Accidental double-sends are a frequent user complaint. They happen when the network is slow and the user assumes the first transaction did not go through, then retries manually. This warning costs one extra tap but prevents thousands of support tickets and accidental losses per day at scale. The warning should NOT block intentional duplicates (e.g. a user genuinely sending two GHS 100 transfers in a row), only flag them for explicit confirmation.

### TC-MOV-012: Transfer fees are calculated correctly per tier and displayed before confirmation

**Category:** Money Math
**Severity:** High

**Preconditions:**

- Standard tier user
- Fee schedule for Standard tier on-network transfers (example):
  - GHS 1 to GHS 50: GHS 0.50
  - GHS 51 to GHS 250: GHS 1.00
  - GHS 251 to GHS 1,000: GHS 2.50
  - GHS 1,001 and above: GHS 5.00

**Steps:**

1. User enters send amount GHS 100
2. System displays the transaction summary

**Expected Result:** Summary shows: amount GHS 100, fee GHS 1.00, total deducted GHS 101.00, recipient receives GHS 100. The fee matches the published schedule exactly for the tier and amount band. If the user changes the amount to GHS 300, the fee updates to GHS 2.50 in real time. The fee is debited at the same time as the principal (atomically) and is recorded as a separate line item in the transaction history for auditability and any future fee reversal logic.

**Notes:** Fee transparency is both a regulatory requirement and a consumer trust issue. Users should never see a fee they did not see before confirming. Money math must use fixed-point or decimal arithmetic, not floating point: a fee of GHS 0.50 represented as a float can produce rounding errors that accumulate into ledger drift over millions of transactions. The cleanest implementation stores all amounts as integer minor units (pesewas in Ghana) and only converts to decimal for display.

## Flow 5: Money Out

12 test cases covering cash-out via agent, wallet-to-bank withdrawals, agent e-money float ceilings, per-tier daily cash-out limits, agent commission calculation and end-of-day settlement, idempotent retries, fraud-pattern holds, reversal handling, and multi-party reconciliation.

### TC-COUT-001: User cashes out via agent successfully

**Category:** Functional
**Severity:** Critical

**Preconditions:**

- User: Standard tier, wallet balance GHS 1,000
- Agent: e-money float at GHS 1,000 (capacity to accept up to the regulatory ceiling of GHS 10,000), commission wallet GHS 50, status active
- Cash-out fee for the GHS 500 band: GHS 5 total (GHS 2 agent commission, GHS 3 platform fee)
- Agent and user are physically present at the agent's location

**Steps:**

1. User tells agent they want to withdraw GHS 500
2. Agent opens the agent app, selects "Withdraw"
3. Agent enters user phone `+233 24 555 1234` and amount GHS 500
4. Agent presses Confirm
5. User's phone receives push notification or USSD prompt: "Confirm withdrawal of GHS 500 at Agent KOJO ENT. Fee: GHS 5. Enter PIN to approve."
6. User enters PIN on their own device to authorize
7. Agent counts and hands the user GHS 500 in physical cash

**Expected Result:** Transaction completes within 5 seconds of user PIN entry. Final state:

- User wallet: GHS 1,000 becomes GHS 495 (debited GHS 500 principal plus GHS 5 fee)
- Agent e-money float: GHS 1,000 becomes GHS 1,500 (received GHS 500 equivalent of cash disbursed)
- Agent commission wallet: GHS 50 becomes GHS 52 (accrued GHS 2 commission)
- Platform fee revenue account: increases by GHS 3

Money is conserved end-to-end: wallet debit GHS 505 equals agent float credit GHS 500 plus commission GHS 2 plus platform fee GHS 3. Both user and agent receive SMS confirmation with the transaction reference and full receipt details.

### TC-COUT-002: Cash-out is rejected when balance is below requested amount plus fee

**Category:** Money Math + Negative
**Severity:** Critical

**Preconditions:**

- User balance: GHS 200
- User attempts cash-out of GHS 200 (fee on GHS 200 amount band = GHS 2, so total required = GHS 202)

**Steps:**

1. Agent submits withdrawal of GHS 200 for the user
2. User enters PIN to authorize

**Expected Result:** Transaction is rejected with the message "Insufficient balance. You need GHS 202 (GHS 200 plus GHS 2 fee) but have GHS 200." User can reduce the requested amount or cancel. User wallet unchanged. Agent float unchanged. No partial transaction recorded.

**Notes:** This case catches a subtle bug: a system that compares balance only against the principal (200 ≥ 200, OK) but then debits principal plus fee (202) drives the balance negative. The check must compare balance to the FULL deduction including all fees.

### TC-COUT-003: Cash-out is blocked when agent's e-money float would exceed the regulatory ceiling

**Category:** Compliance
**Severity:** High

**Preconditions:**

- Agent e-money float regulatory ceiling: GHS 10,000
- Agent current e-money float: GHS 9,800
- User requests withdrawal of GHS 500 (would push agent float to GHS 10,300, above ceiling)

**Steps:**

1. Agent submits the withdrawal request
2. System checks agent capacity before proceeding to user PIN prompt

**Expected Result:** Transaction is rejected at the agent side before reaching the user with the message "You cannot accept this transaction. Your float is near its limit. Please cash out at a super-agent first." User receives SMS: "Your withdrawal at this agent could not be processed. Please try another agent." User wallet unchanged. The block applies only to this transaction; the agent's account is not locked.

**Notes:** Agent e-money float ceilings are a prudential regulatory control. Mobile money operators are required to maintain trust account balances at the central bank equal to all outstanding e-money. Capping individual agent floats limits operational risk if an agent's account is compromised. When an agent hits the ceiling, they must "cash out" their float at a super-agent or bank, converting e-money back to cash, before they can accept more customer withdrawals.

### TC-COUT-004: PIN is required for every cash-out, with no remember-me option

**Category:** Security
**Severity:** Critical

**Preconditions:**

- User has an active session and just completed a different cash-out 1 minute ago

**Steps:**

1. Agent initiates another cash-out for the same user
2. User reaches the PIN entry step

**Expected Result:** PIN entry is required on every cash-out regardless of session age or recent cash-out activity. There is no "trust this agent for the next N minutes" option. After 3 incorrect PIN entries on a single cash-out, the cash-out is cancelled. Failed attempts count toward the global lockout threshold from TC-AUTH-003.

**Notes:** Cash-out is irreversible the moment the agent disburses physical cash. Skipping PIN even within a recent session would let anyone with brief access to an unlocked phone authorize a cash-out at a colluding agent. Per-transaction PIN is non-negotiable for any withdrawal flow.

### TC-COUT-005: Daily cash-out limit per tier is enforced atomically

**Category:** Compliance + Concurrency
**Severity:** Critical

**Preconditions:**

- Basic tier user, daily cash-out limit GHS 3,000 (independent from transfer limits)
- User has already cashed out GHS 2,800 today across two earlier transactions
- User attempts a new cash-out of GHS 400 (would push daily total to GHS 3,200, over limit)

**Steps:**

1. Agent initiates the cash-out of GHS 400
2. System aggregates today's cash-outs and checks against the limit

**Expected Result:** Cash-out is rejected with the message "This withdrawal would exceed your daily cash-out limit. You can withdraw up to GHS 200 more today." User can request a smaller withdrawal (up to GHS 200) or wait until midnight reset. The aggregation is atomic across concurrent requests: two simultaneous cash-out attempts at different agents cannot both succeed if their sum would exceed the daily limit.

**Notes:** Daily cash-out limits are typically lower than transfer limits because cash leaves the platform's view the moment it is disbursed. The atomic aggregation must work even across different agents the user might visit in quick succession, which means the limit check needs a shared state lock or a transactional aggregate query.

### TC-COUT-006: Wallet-to-bank withdrawal succeeds for a linked bank account

**Category:** Integration
**Severity:** High

**Preconditions:**

- User has linked a bank account with valid Open Banking authorization (token not expired)
- User wallet balance: GHS 1,500
- Wallet-to-bank fee for the GHS 1,000 band: GHS 8

**Steps:**

1. User selects "Withdraw to bank"
2. Selects the linked bank account
3. Enters amount GHS 1,000
4. Confirms with PIN

**Expected Result:** Wallet debits GHS 1,008 (GHS 1,000 plus GHS 8 fee). Platform initiates a credit transfer to the user's bank account via the relevant rail (e.g. GhIPSS Instant Pay in Ghana). Bank credits the GHS 1,000 to the user's bank account. Settlement is typically within minutes for instant rails, or up to next business day for slower rails. User receives SMS at each settlement step: wallet debit, transfer initiated, bank credit confirmed.

**Notes:** Wallet-to-bank withdrawals are an alternative to agent cash-out, popular with users who prefer keeping money in a bank account rather than physical cash. They have different settlement characteristics than agent cash-out (typically slower) and different fee economics (often more expensive for the platform because the bank rail charges interchange or per-transaction fees).

### TC-COUT-007: Agent commission is calculated correctly per the published commission schedule

**Category:** Money Math
**Severity:** High

**Preconditions:**

- Agent commission schedule for cash-out (example, share of customer fee):
  - GHS 1 to GHS 100: 40% of customer fee
  - GHS 101 to GHS 500: 35% of customer fee
  - GHS 501 to GHS 2,000: 30% of customer fee
  - GHS 2,001 and above: 25% of customer fee
- Customer fee schedule for cash-out is already configured

**Steps:**

1. Process cash-out of GHS 75 (customer fee GHS 1.50, expected agent commission GHS 0.60)
2. Process cash-out of GHS 250 (customer fee GHS 3.00, expected agent commission GHS 1.05)
3. Process cash-out of GHS 1,000 (customer fee GHS 10.00, expected agent commission GHS 3.00)
4. Process cash-out of GHS 3,000 (customer fee GHS 25.00, expected agent commission GHS 6.25)

**Expected Result:** For each transaction, the commission accrued to the agent matches the schedule exactly. Commission is stored in integer minor units (pesewas), not as a floating point percentage of the fee, to avoid rounding drift. The agent's commission ledger records each accrual with: transaction reference, customer fee paid, commission rate applied, commission amount in minor units, and timestamp.

**Notes:** Agent commission accounting is the single most operationally complex part of a mobile money system. Agents are independent contractors whose income depends on accurate, transparent commission calculation. Small rounding errors at scale (millions of transactions per day) compound into disputes that damage the agent network. The standard pattern: define commission rates as integer numerator over integer denominator (e.g. 40/100), apply to the fee in minor units, and use banker's rounding (round half-to-even) at the final step to avoid systematic over-rounding in either direction.

### TC-COUT-008: Agent commission settles to the commission wallet at end-of-day

**Category:** Money Math + Functional
**Severity:** High

**Preconditions:**

- Agent has accrued GHS 47.25 in commission across 23 transactions over the course of a day
- Commission settlement policy: daily, at 23:59 local time, into the agent's commission wallet (separate from the e-money float)

**Steps:**

1. Wait for the end-of-day settlement job to run
2. Inspect the agent's commission wallet, accrual ledger, and settlement ledger

**Expected Result:** At end-of-day, the GHS 47.25 in accrued commission moves from the "Accrued" state into the agent's commission wallet (becoming spendable or withdrawable). The agent's commission ledger shows: 23 individual accruals during the day, one settlement entry of GHS 47.25, and a balance carried forward of GHS 0.00 in the accrual bucket until the next transaction accrues. The settlement job is idempotent: if it runs twice for the same day (e.g. due to a retry after transient failure), funds settle exactly once.

**Notes:** Daily settlement balances cash flow for agents (small-business agents depend on daily income) with operational efficiency for the platform (batch settle is cheaper than per-transaction settle). Some platforms settle hourly, some weekly. Whatever the cadence, accrual and settlement must be tested separately, and the settlement job must be idempotent because batch jobs commonly retry on transient infrastructure failures.

### TC-COUT-009: Idempotent retry on cash-out does not double-debit the user

**Category:** Money Math + Concurrency
**Severity:** Critical

**Preconditions:**

- Agent submits a GHS 500 cash-out with idempotency key `idem-cout-001`
- Server processes successfully but the response is lost due to network failure
- Agent client retries with the same idempotency key

**Steps:**

1. Original cash-out succeeds server-side: user debited GHS 505, agent float credited GHS 500, commission accrued GHS 2
2. HTTP response is lost in transit (server-side success, client-side timeout)
3. Agent client automatically retries with key `idem-cout-001`
4. Server receives the retry

**Expected Result:** Server recognizes the idempotency key as already processed and returns the cached original response without re-executing the transaction. User balance, agent float, and commission ledger are all unchanged from the first execution. The original transaction reference appears exactly once in the audit log. The agent disburses physical cash once, not twice.

**Notes:** This is the cash-out mirror of TC-CIN-007. Both directions need idempotency, but cash-out is arguably more user-visible because the agent has already disbursed physical cash by the time the retry happens. Without idempotency, the user gets debited twice for one cash payment, which is the kind of failure that triggers the loudest support tickets.

### TC-COUT-010: Suspicious cash-out pattern triggers a fraud review hold

**Category:** Security + Compliance
**Severity:** High

**Preconditions:**

- User has been withdrawing larger amounts at different agents over the past hour:
  - 14:00: GHS 800 at Agent A
  - 14:20: GHS 1,500 at Agent B
  - 14:40: GHS 2,000 at Agent C
- Fraud rules engine threshold: more than GHS 4,000 in cash-outs across 3 or more agents within 1 hour triggers a review hold

**Steps:**

1. User attempts to cash out GHS 1,000 at a 4th agent (Agent D)

**Expected Result:** Transaction is held in a "Pending fraud review" state. User receives SMS: "For your security, we are reviewing this withdrawal. You will hear from us within 30 minutes. Reference: TXN-XYZ." Agent receives a notification to wait or refer the user to support. The fraud team can approve (release) or block (cancel and refund the held amount) within the SLA. No physical cash is disbursed until the hold clears.

**Notes:** Rapid cash-outs across multiple agents are a classic account-takeover pattern: an attacker who has gained access drains the wallet through a series of withdrawals at different agents to stay under per-transaction or per-agent limits. Holding the 4th transaction for review costs the user a few minutes if legitimate but can save thousands in stolen funds if not. Production fraud rules are far more sophisticated (behavioral baselines, device-and-location risk scores, peer comparison) but a clear rule like this is sufficient as a baseline test.

### TC-COUT-011: Cash-out reversal requires user consent AND support authorization

**Category:** Security
**Severity:** High

**Preconditions:**

- A cash-out of GHS 500 was processed 10 minutes ago
- Agent claims they over-disbursed (handed the user GHS 600 instead of GHS 500) and wants to reverse and re-process

**Steps:**

1. Agent contacts support to report the over-disbursement
2. Support opens a reversal ticket and contacts the user
3. User confirms (via SMS reply or in-app prompt) that they received GHS 600 instead of GHS 500
4. Support initiates the reversal of the original cash-out
5. Support then processes a corrected cash-out of GHS 600

**Expected Result:** Cash-out reversal does NOT auto-execute from the agent app. BOTH user consent AND support authorization are required before any state changes. Once both are present, the original transaction is reversed (user wallet returned to pre-transaction state, agent float and commission also reversed). The corrected cash-out is processed as a fresh transaction. If the user denies the over-disbursement, the reversal does not proceed; the agent must pursue recovery offline (e.g. calling the user back) and absorbs the loss if recovery fails.

**Notes:** Unlike cash-in reversals (TC-CIN-009 and TC-CIN-010), cash-out reversal is operationally much harder because the physical cash has already left the agent's till. The reversal mechanism only corrects the electronic ledger to match what happened physically. Without user consent, a dishonest agent could falsely claim over-disbursement and reverse cash-outs that were correctly processed, effectively stealing back the e-money equivalent of cash they genuinely owed.

### TC-COUT-012: Cash-out record is consistent and reconcilable across user, agent, and audit views

**Category:** Compliance
**Severity:** High

**Preconditions:**

- A successful cash-out has just completed

**Steps:**

1. View the transaction in the user's transaction history
2. View the corresponding entry in the agent's transaction history
3. Inspect the back-end audit record

**Expected Result:** All three views show consistent details for the same transaction ID:

- Transaction ID (identical across all three views)
- Type: Cash-out
- Principal amount, customer fee, total debited from user
- Agent commission amount (visible in agent view and audit view, not in user view)
- Platform fee amount (visible in audit view only)
- Agent ID, name, and location
- User phone (masked in agent view, full in audit view)
- Timestamps in both UTC and local time
- Status: Completed
- Cryptographic hash linking this transaction to the previous one in the respective ledger (user, agent, and audit ledgers each chain independently)

The user view shows what the user paid. The agent view shows what they earned. The audit view shows the full money flow including platform fee. This separation respects privacy and operational need-to-know while remaining fully reconcilable across all parties.

**Notes:** Multi-party reconciliation is essential. The user, agent, platform, and regulator each need a consistent view of the transaction at their appropriate level of detail. The cleanest implementation uses a single source of truth (the audit ledger) with role-based views derived from it. This avoids drift where "the agent's view shows GHS 2.00 commission but the audit shows GHS 2.05" can destroy trust at the agent network level. End-of-day reconciliation should cross-check that the sum of all user debits equals the sum of all agent float credits plus all commission accruals plus all platform fees, on every settlement run.

## References

This plan is informed by, but not formally compliant with, the following frameworks. Mentioned to make the regulatory awareness explicit:

- **PSD2** (EU): strong customer authentication requirements
- **AML/CFT** guidance from FATF
- **PCI DSS** scope considerations (for card top-up integration)
- **GSMA Mobile Money guidelines**
- **Bank of Ghana** mobile money operator regulations (for local context)

---

*Test cases for Flows 6 and 7 are pending and will be added incrementally.*
