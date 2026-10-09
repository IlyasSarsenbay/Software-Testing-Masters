# Software-Testing-Masters
Markdown
1. One-Page Test Plan: Money Transfer Feature 

1. Overview & Objectives
This Test Plan defines the scope, strategy, and criteria for validating the core logic and security authentication of the Financial Money Transfer Module. The objective is to ensure precise business rule compliance, robust risk mitigation, and seamless state management.

2. Test Scope
2.1 Scope In
Transfer Amount Validation: Single transaction range validation (100KZT to 500,000KZT).
Daily Limit Validation: Cumulative transfer cap validation (1,000,000KZT per user/day).
SMS Authentication Flow: Triggered for transfers >100,000KZT, enforce 120s OTP timeout, and enforce max 3 failed attempts lockout.
State Transitions: Dynamic lifecycle handling  (Waiting - attempt 1..3, Confirmed, Expired, Blocked).
Error Prioritization: Verification that single transfer amount errors strictly have priority over daily limit errors.
2.2 Scope Out
End-to-end Payment Gateway Integration processing.
Backend KYC/AML regulatory checks and background scoring.
Non-functional Performance, Load, and Stress Testing.
3. Test Approach & Techniques
Equivalence Partitioning (EP): Grouping transfer amounts into valid and invalid sets (e.g., <100, 100--500,000, >500,000).
Boundary Value Analysis (BVA): Testing exact boundaries (99, 100, 101, 499,999, 500,000, 500,001 KZT; 100,000KZT vs 100,001KZT for SMS trigger).
Decision Table Testing: Combinatorial coverage for multi-condition flows (Amount threshold  SMS attempt count  Daily total).
State Transition Testing: Validating valid state paths and rejecting invalid transitions e.g., executing actions from `Expired` or `Blocked` terminal states .
4. Criteria 
Criteria Type
Requirements
Entry Criteria
1. QA Environment build deployment completed & verified.
2. Test accounts with mock balances and daily caps populated.
3. SMS Gateway API stubs/endpoints active and accessible.
Exit Criteria
1. 100% execution of all Critical and High priority test cases.
2. 0 open Critical/Blocker defects in the tracking system.
3. Traceability Matrix completed mapping 100% of functional rules to tests.

5. Risk Assessment & Mitigation 

Risk ID
Product Risk Description
Severity
Mitigation Strategy / Test Focus
R-01
Unauthorized transfers via SMS auth bypass
Critical
Validate mandatory server-side session lockouts after 3 invalid OTP attempts; test API direct post bypass.
R-02
Double spending / Replay attacks
Critical
Execute rapid parallel API requests using identical transaction IDs and nonces to verify idempotency controls.
R-03
SMS delivery delay or gateway failure
High
Simulate high latency and API failure responses (50x/40x) on SMS endpoints; verify gracefully handled timeout flows (120s).


2. 12 test cases

ID
Traces to Requirement
Preconditions
Inputs
Expected Result
TC_AMT_001
Min amount per transfer (100 KZT)
Logged in, balance 1,000 KZT
Transfer 100 KZT
Successful immediate transfer with no SMS code required. The balance changed to 900 KZT.
TC_AMT_002
Min amount per transfer (100 KZT)
Logged in, balance 1,000 KZT
Transfer 99 KZT
Transfer rejected. Amount error message displayed. The balance remains 1,000 KZT.
TC_AMT_003
Min amount per transfer (100 KZT)
Logged in, balance 1,000 KZT
Transfer 101 KZT
Successful immediate transfer with no SMS code required. The balance changed to 899 KZT.
TC_AMT_004
Max amount per transfer (500,000 KZT) & Daily limit (1,000,000 KZT)
Logged in, balance 1,000,000 KZT, daily total 500,000 KZT
Transfer 500,000 KZT
1. SMS code screen displayed.
2. Entering valid 6-digit code results in successful transfer. Balance becomes 500,000 KZT, daily total updates to 1,000,000 KZT.
TC_AMT_005
Max amount per transfer (500,000 KZT)
Logged in, balance 1,000,000 KZT, daily total 0 KZT
Transfer 500,001 KZT
Transfer rejected. Amount error message displayed. Balance remains 1,000,000 KZT.
TC_AMT_006
Daily limit for transfer (1,000,000 KZT)
Logged in, balance 500 KZT, daily total 1,000,000 KZT
Transfer 100 KZT
Transfer rejected. Daily limit error message displayed. Balance remains 500 KZT.
TC_DT_001
Mandatory SMS code for transfers > 100,000 KZT
Logged in, balance 1,000,000 KZT, daily total 0 KZT
Transfer 200,000 KZT, enter valid 6-digit SMS code
SMS screen displayed. After entering valid SMS code, transfer succeeds. Balance becomes 800,000 KZT, daily total updates to 200,000 KZT.
TC_DT_002
3 incorrect SMS code attempts lockout
Logged in, balance 1,000,000 KZT, daily total 0 KZT
Transfer 200,000 KZT, enter invalid SMS code 3 times
SMS screen displayed. After 3 wrong attempts, transfer is cancelled, error message shown. Balance remains 1,000,000 KZT.
TC_ST_001
Successful transfer on 1st SMS attempt
Logged in, balance 1,000,000 KZT, daily total 0 KZT
Transfer 500,000 KZT, enter valid code on Attempt 1
Transfer succeeds on first try. Balance becomes 500,000 KZT, daily total updates to 500,000 KZT.
TC_ST_002
Successful transfer on 2nd SMS attempt
Logged in, balance 1,000,000 KZT, daily total 0 KZT
Transfer 500,000 KZT, enter wrong code on Attempt 1, enter valid code on Attempt 2
Transfer succeeds on second try. Balance becomes 500,000 KZT, daily total updates to 500,000 KZT.
TC_ST_004
State Transition: Blocked after 3 invalid attempts
Logged in, balance 1,000,000 KZT, daily total 0 KZT
Transfer 500,000 KZT, enter invalid code 3 consecutive times
Transfer cancelled, user session/SMS flow blocked. Balance remains 1,000,000 KZT.
TC_ST_005
State Transition: SMS Timeout (120 seconds)
Logged in, balance 1,000,000 KZT, daily total 0 KZT
Transfer 500,000 KZT, enter valid code after 120 seconds pass
Transfer rejected due to expired OTP code. Error message shown. Balance remains 1,000,000 KZT.


3. Requirements Traceability Matrix 

Requirement ID
Requirement Description
Scope
Mapped Test Case IDs
Coverage Status
REQ-01
Transfer amount validation (100 – 500,000 KZT)
In Scope
TC_AMT_001, TC_AMT_002, TC_AMT_003, TC_AMT_004, TC_AMT_005, TC_DT_001,
TC_DT_003, TC_DT_005
Covered
REQ-02
Daily total transfer limit validation (1,000,000 KZT)
In Scope
TC_AMT_004, TC_AMT_006, TC_DT_004, TC_DT_005
Covered
REQ-03
Mandatory SMS OTP authentication for amounts > 100,000 KZT
In Scope
TC_AMT_004, TC_DT_001, TC_DT_002, TC_ST_001, TC_ST_002
Covered
REQ-04
SMS OTP state transitions (3 attempts limit, 120s timeout, session lockout)
In Scope
TC_DT_002, TC_ST_001, TC_ST_002, TC_ST_004, TC_ST_005
Covered
REQ-05
Payment gateway integration (Acquiring / Processing)
Out of Scope
None
Not Covered
REQ-06
KYC / AML compliance backend processing
Out of Scope
None
Not Covered
REQ-07
Performance, load, and stress testing
Out of Scope
None
Not Covered


Uncovered Requirements & Rationale
The following requirements are explicitly excluded from the current functional test execution scope, along with the technical rationale for their exclusion:
REQ-05: Payment Gateway Integration
Reason: Testing third-party payment acquiring and processing services requires dedicated external sandbox environments and payment network certification. This test plan focuses strictly on validating the internal transfer rules, boundary conditions, and OTP authorization flows within the money transfer module.
REQ-06: KYC / AML Compliance Backend Processing
Reason: Know Your Customer (KYC) and Anti-Money Laundering (AML) checks are handled by specialized backend compliance microservices. These systems undergo separate integration and security testing and fall outside the scope of this functional test suite.
REQ-07: Performance and Load Testing
Reason: Performance, stress, and load testing require a dedicated, isolated staging environment along with specialized load generation tools (e.g., JMeter, k6) to evaluate Non-Functional Requirements (NFRs). This test suite is restricted to functional correctness, state transitions, and business rule enforcement.
4. Release Checklist: SMS Authentication Flow 

Checklist ID
Category
Verification Item
Expected Behavior
CHK-SMS-01
Functional
Mandatory Trigger (>100,000KZT)
Transfer amounts strictly greater than 100,000KZT (e.g., 100,001KZT) immediately display the SMS OTP verification screen; amounts 100,000KZT bypass SMS verification.
CHK-SMS-02
UI/UX
Countdown Timer UI Display
A visible 120-second countdown timer starts immediately upon landing on the SMS verification screen and updates smoothly in real time.
CHK-SMS-03
Functional
Timer Expiration (120s Timeout)
 Entering the correct OTP after 120 seconds transitions the transaction to Expired state, showing an explicit timeout error and preventing completion.
CHK-SMS-04
Security
3-Attempt Invalid Lockout
Entering an incorrect OTP 3 consecutive times immediately transitions state to Blocked, invalidating the transaction and locking out the SMS session.
CHK-SMS-05
Security
Lockout Reset on Success
Entering a valid OTP on Attempt 1 or Attempt 2 resets the failed attempt counter and completes the transaction successfully.
CHK-SMS-06
Security
Anti-Replay & Idempotency
Re-submitting an already used or expired OTP payload via API interception fails with an HTTP 409/400 error; transaction execution is strictly idempotent.
CHK-SMS-07
Security
Payload Privacy & Logging
OTP codes and sensitive payloads are never exposed in plaintext across server logs, browser storage, analytics tools, or network query parameters.
CHK-SMS-08
Security
Direct API Post Bypass Prevention
Sending direct POST /api/transfer/execute requests without a valid, server-verified OTP token returns an HTTP 401/403 Forbidden error.
CHK-SMS-09
Edge Cases
SMS Gateway Timeout / Failure
High latency or 5xx/4xx failure responses from the SMS gateway are gracefully handled with a user-friendly error message, allowing retry without data corruption.
CHK-SMS-10
UI/UX
User Cancellation Flow
Clicking "Cancel" on the SMS screen safely aborts the authentication process, releases pre-authorization holds, and returns the user to the transaction form with balance unchanged.
CHK-SMS-11
Integration
Resend OTP Flow & Throttling
Requesting a new OTP code invalidates any previously issued active OTP for the current session and enforces rate-limiting timeouts between requests.


5. Defect Reports 

Defect Report 1: Critical / Security
Defect ID: BUG-TRF-001 Title / Summary: SMS OTP accepted and transaction completed after 120-second expiration timeout (Terminal State Violation) Severity: Critical Priority: P1 - Urgent Component / Module: Authentication Service / SMS OTP Module Environment: Staging (v2.4.0-rc1), Mobile App (iOS 17.4 / Android 14), Backend API v2
Preconditions:
User is authenticated and logged into the mobile banking application.
User account balance is at least 200,000 KZT.
User has non-zero daily transfer limit remaining.
Steps to Reproduce:
Initiate a transfer of 200,000 KZT to a valid recipient.
Observe navigation to the SMS OTP Verification screen with a 120-second countdown timer.
Wait for 125 seconds without entering any input until the UI countdown reaches 00:00 and displays "Code Expired".
Enter the original 6-digit OTP code received via SMS during step 1.
Tap the "Confirm Transfer" button.
Expected Result: The system must reject the expired OTP, display an error message ("OTP has expired. Please request a new code"), keep the transaction status as EXPIRED, and leave the account balance unchanged.
Actual Result: The backend accepts the expired OTP code, returns 200 OK with status TRANSACTION_SUCCESS, deducts 200,000 KZT from the user balance, and transfers the funds to the recipient.
Workaround: No client-side or user workaround available. Server-side session validation must be enforced immediately.
Defect Report 2: High / Business Logic
Defect ID: BUG-TRF-002 Title / Summary: Incorrect error message displayed when both single transfer limit and cumulative daily limit are exceeded Severity: High Priority: P2 - High Component / Module: Transfer Validation Engine / Business Rules Engine Environment: Staging (v2.4.0-rc1), Web & Mobile Clients, Backend API v2
Preconditions:
User is logged into the application with an account balance of at least 1,500,000 KZT.
User's accumulated daily transfer total is currently 800,000 KZT (Daily limit: 1,000,000 KZT).
Steps to Reproduce:
Navigate to the Money Transfer screen.
Enter 600,000 KZT in the transfer amount field (violates single transfer limit of 500,000 KZT AND cumulative daily limit of 1,000,000 KZT).
Tap "Continue" / "Transfer".
Expected Result: The input validation rule priority should flag the single transaction limit first: "Transfer amount exceeds single transaction limit of 500,000 KZT".
Actual Result: The system evaluates cumulative limit rules first and displays the incorrect error: "Daily transfer limit of 1,000,000 KZT exceeded", confusing the user about whether a smaller amount below 500,000 KZT is permissible.
Workaround: Users can manually recalculate their remaining daily quota and enter amounts below 500,000 KZT, but the misleading messaging degrades user experience and trust.
Defect Report 3: Medium / UI & Lack of Requirements
Defect ID: BUG-TRF-003 Title / Summary: Lack of client-side input max-length constraint causes transfer button overlap with balance card on small mobile screens Severity: Medium Priority: P3 - Medium Component / Module: Frontend UI / Money Transfer View Environment: Staging (v2.4.0-rc1), iOS App (iPhone SE / 375px viewport width)
Preconditions:
User is logged into the mobile app on a screen with a display width of 375px or smaller.
User opens the Money Transfer screen.
Steps to Reproduce:
Focus on the "Transfer Amount" input field.
Enter a 10-digit numeric string (e.g., 9999999999).
Observe the layout rendering as the text scales.
Expected Result:
Client-side input mask restricts digits to maximum allowed system limit (8 digits / up to 500,000 KZT).
UI layout remains responsive and intact without element collision or overlapping.
Actual Result: The input field accepts unlimited digits without client-side max-length validation. The unformatted text expands vertically and pushes adjacent UI elements down, causing the primary "Transfer" CTA button to visually overlap and obscure the current balance display card.
Workaround: User can delete the extra digits manually to restore normal UI layout rendering.

AI Appendix
Prompt used for Task 1: "Act as a Lead QA Engineer. Generate a concise, professional, one-page Test Plan section in Markdown format for a financial money transfer module based on these specifications: 
Scope In: Transfer amount validation (100–500,000 KZT), daily total limit validation (1,000,000 KZT), SMS authentication flow (> 100,000 KZT, 120s timeout, 3 attempts), state transitions, and error prioritization. 
Scope Out: Payment gateway integration, KYC/AML backend processing, performance/load testing. 
Approach: EP, BVA, Decision Table Testing, State Transition Testing.
Entry Criteria: QA environment deployment ready, test data available, SMS API accessible. 
Exit Criteria: 100% execution of critical test cases, 0 open Critical/Blocker defects, Traceability Matrix completed. 
Top 3 Product Risks: Unauthorized transfers via SMS bypass, Double spending (Replay attacks), SMS delivery delay/failure. 
Keep it strictly one page and structured." 
Raw output: Test Plan: Financial Money Transfer Module

1. Overview & Objectives
This Test Plan defines the scope, strategy, and criteria for validating the core logic and security authentication of the Financial Money Transfer Module. The objective is to ensure precise business rule compliance, robust risk mitigation, and seamless state management.

2. Test Scope
2.1 Scope In
Transfer Amount Validation: Single transaction range validation (100KZT to 500,000KZT).
Daily Limit Validation: Cumulative transfer cap validation (1,000,000KZT per user/day).
SMS Authentication Flow: Triggered for transfers >100,000KZT, enforce 120s OTP timeout, and enforce max 3 failed attempts lockout.
State Transitions: Lifecycle handling (Initiated  Pending Auth  Processing  Completed / Failed).
Error Prioritization: Verification of accurate error codes and user messaging based on failure conditions.
2.2 Scope Out
End-to-end Payment Gateway Integration processing.
Backend KYC/AML regulatory checks and background scoring.
Non-functional Performance, Load, and Stress Testing.
3. Test Approach & Techniques
Equivalence Partitioning (EP): Grouping transfer amounts into valid and invalid sets (e.g., <100, 100--500,000, >500,000).
Boundary Value Analysis (BVA): Testing exact boundaries (99, 100, 101, 499,999, 500,000, 500,001 KZT; 99,999 vs. 100,001 KZT for SMS trigger).
Decision Table Testing: Combinatorial coverage for multi-condition flows (Amount threshold  SMS attempt count  Daily total).
State Transition Testing: Validating valid state paths and rejecting invalid transitions (e.g., Failed directly to Completed).
4. Criteria 
Criteria Type
Requirements
Entry Criteria
1. QA Environment build deployment completed & verified.
2. Test accounts with mock balances and daily caps populated.
3. SMS Gateway API stubs/endpoints active and accessible.
Exit Criteria
1. 100% execution of all Critical and High priority test cases.
2. 0 open Critical/Blocker defects in the tracking system.
3. Traceability Matrix completed mapping 100% of functional rules to tests.

5. Risk Assessment & Mitigation 

Risk ID
Product Risk Description
Severity
Mitigation Strategy / Test Focus
R-01
Unauthorized transfers via SMS auth bypass
Critical
Validate mandatory server-side session lockouts after 3 invalid OTP attempts; test API direct post bypass.
R-02
Double spending / Replay attacks
Critical
Execute rapid parallel API requests using identical transaction IDs and nonces to verify idempotency controls.
R-03
SMS delivery delay or gateway failure
High
Simulate high latency and API failure responses (50x/40x) on SMS endpoints; verify gracefully handled timeout flows (120s).


What I Changed & Why

1. Fixed State Transition Lifecycle Names:
Change: Replaced generic states (Initiated -> Pending Auth -> Processing -> Completed / Failed) with the explicit state machine defined in Task C (Waiting - attempt 1..3, Confirmed, Expired, Blocked).
Reason: The raw AI output used standard order-processing state names instead of the precise SMS verification states.

2. Corrected Error Prioritization Rule:
Change: Explicitly specified that Amount errors take priority over Daily Limit errors.
Reason: The raw AI output gave a generic description ("Verification of accurate error codes"). It was necessary to detail specific rule priority logic (Amount > Daily limit).
3. Adjusted BVA Threshold for SMS Authentication: 
Change: Fixed the SMS boundary threshold from 99,999KZT vs 100,001KZT to 100,000KZT vs 100,001KZT. 
Reason: SMS is required for amounts strictly greater than 100,000KZT. Therefore, 100,000KZT does not require SMS, while 100,001KZT does.

Prompt used for Task 3: "Act as a Senior QA Engineer. Create a concise, professional Traceability Matrix (RTM) section in Markdown for a financial money transfer module based on the deliverables from Week 3 and Week 4.

Context & Inputs:
1. Requirements / Scope In:
   - REQ-01: Transfer amount validation (100–500,000 KZT).
   - REQ-02: Daily total transfer limit validation (1,000,000 KZT).
   - REQ-03: Mandatory SMS authentication flow for amounts > 100,000 KZT (120s timeout, max 3 invalid attempts before lockout).
   - REQ-04: State transitions for SMS OTP verification (Waiting attempt 1..3, Confirmed, Blocked, Expired).
2. Scope Out (Uncovered Requirements):
   - REQ-05: Payment gateway integration (Processing / acquiring).
   - REQ-06: KYC/AML compliance backend checks.
   - REQ-07: Performance, load, and stress testing.
3. Test Cases (Week 3 Deliverables):
   - TC_AMT_001 to TC_AMT_006 (Equivalence Partitioning & Boundary Value Analysis)
   - TC_DT_001 to TC_DT_005 (Decision Table testing)
   - TC_ST_001 to TC_ST_005 (State Transition testing)

Task Requirements:
1. Generate a clean Markdown Traceability Matrix table mapping Requirement ID, Requirement Description, Scope Status (In-Scope / Out-of-Scope), Mapped Test Case IDs, and Coverage Status (Covered / Not Covered).
2. Add a clear, concise section explicitly listing all uncovered requirements (Scope Out) and providing a professional engineering rationale for why they are excluded from functional test execution at this stage."

Raw output: Requirements Traceability Matrix 

Requirement ID
Requirement Description
Scope
Mapped Test Case IDs
Coverage Status
REQ-01
Transfer amount validation (100 – 500,000 KZT)
In Scope
TC_AMT_001, TC_AMT_002, TC_AMT_003, TC_AMT_004, TC_AMT_005, TC_DT_003, TC_DT_005
Covered
REQ-02
Daily total transfer limit validation (1,000,000 KZT)
In Scope
TC_AMT_004, TC_AMT_006, TC_DT_004, TC_DT_005
Covered
REQ-03
Mandatory SMS OTP authentication for amounts > 100,000 KZT
In Scope
TC_AMT_004, TC_DT_001, TC_DT_002, TC_ST_001, TC_ST_002
Covered
REQ-04
SMS OTP state transitions (3 attempts limit, 120s timeout, session lockout)
In Scope
TC_DT_002, TC_ST_001, TC_ST_002, TC_ST_004, TC_ST_005
Covered
REQ-05
Payment gateway integration (Acquiring / Processing)
Out of Scope
None
Not Covered
REQ-06
KYC / AML compliance backend processing
Out of Scope
None
Not Covered
REQ-07
Performance, load, and stress testing
Out of Scope
None
Not Covered


Uncovered Requirements & Rationale
The following requirements are explicitly excluded from the current functional test execution scope, along with the technical rationale for their exclusion:
REQ-05: Payment Gateway Integration
Reason: Testing third-party payment acquiring and processing services requires dedicated external sandbox environments and payment network certification. This test plan focuses strictly on validating the internal transfer rules, boundary conditions, and OTP authorization flows within the money transfer module.
REQ-06: KYC / AML Compliance Backend Processing
Reason: Know Your Customer (KYC) and Anti-Money Laundering (AML) checks are handled by specialized backend compliance microservices. These systems undergo separate integration and security testing and fall outside the scope of this functional test suite.
REQ-07: Performance and Load Testing
Reason: Performance, stress, and load testing require a dedicated, isolated staging environment along with specialized load generation tools (e.g., JMeter, k6) to evaluate Non-Functional Requirements (NFRs). This test suite is restricted to functional correctness, state transitions, and business rule enforcement.
What I Changed & Why

Aligned Decision Table Test Case Mapping for REQ-01
Change: Synchronized test case ID with REQ-01 
Reason: The raw AI output missed linking `TC_DT_001` (valid amount over threshold) to REQ-01.

Prompt used for Task 4: "Act as a Lead QA Engineer. Generate a concise, professional Release Checklist for the SMS Code Authentication Flow in a financial money transfer module.

Requirements:
1. Format: Markdown table or bulleted checklist with exactly 10 to 12 items (at most 12).
2. Business Rules to Cover:
   - Mandatory trigger for amounts > 100,000 KZT
   - 120-second OTP expiration timer & countdown UI
   - Hard lockout after 3 consecutive invalid code attempts
   - Anti-replay / Idempotency protection & secure SMS payload (no plaintext logging)
   - Graceful error handling for gateway timeouts and cancellation
3. Structure per Item: Checklist ID, Category (e.g., Security, Functional, UI/UX, Edge Cases), Verification Item, Expected Behavior. "

Raw output:

Checklist ID
Category
Verification Item
Expected Behavior
CHK-SMS-01
Functional
Mandatory Trigger (> 100,000 KZT)
Transfer amounts strictly greater than 100,000 KZT (e.g., 100,001 KZT) immediately display the SMS OTP verification screen; amounts  100,000 KZT bypass SMS verification.
CHK-SMS-02
UI/UX
Countdown Timer UI Display
A visible 120-second countdown timer starts immediately upon landing on the SMS verification screen and updates smoothly in real time.
CHK-SMS-03
Functional
Timer Expiration (120s Timeout)
Entering the correct OTP after 120 seconds results in an "OTP Expired" error, preventing transaction completion and requiring a new OTP request.
CHK-SMS-04
Security
3-Attempt Invalid Lockout
Entering an incorrect OTP 3 consecutive times immediately triggers a hard session lockout, invalidates the transaction, and displays a security block notification.
CHK-SMS-05
Security
Lockout Reset on Success
Entering a valid OTP on Attempt 1 or Attempt 2 resets the failed attempt counter and completes the transaction successfully.
CHK-SMS-06
Security
Anti-Replay & Idempotency
Re-submitting an already used or expired OTP payload via API interception fails with an HTTP 409/400 error; transaction execution is strictly idempotent.
CHK-SMS-07
Security
Payload Privacy & Logging
OTP codes and sensitive payloads are never exposed in plaintext across server logs, browser storage, analytics tools, or network query parameters.
CHK-SMS-08
Security
Direct API Post Bypass Prevention
Sending direct POST /api/transfer/execute requests without a valid, server-verified OTP token returns an HTTP 401/403 Forbidden error.
CHK-SMS-09
Edge Cases
SMS Gateway Timeout / Failure
High latency or 5xx/4xx failure responses from the SMS gateway are gracefully handled with a user-friendly error message, allowing retry without data corruption.
CHK-SMS-10
UI/UX
User Cancellation Flow
Clicking "Cancel" on the SMS screen safely aborts the authentication process, releases pre-authorization holds, and returns the user to the transaction form with balance unchanged.
CHK-SMS-11
Integration
Resend OTP Flow & Throttling
Requesting a new OTP code invalidates any previously issued active OTP for the current session and enforces rate-limiting timeouts between requests.



What I Changed & Why

Aligned Behavior Statements with Task C State Machine: 
Change: Explicitly mapped `CHK-SMS-03` (120s timeout) and `CHK-SMS-04` (3 invalid attempts) to their respective formal states (`Expired` and `Blocked`). 
Reason: The raw AI output used generic system failure descriptions ("security block notification", "preventing transaction completion"). 

Prompt used for Task 5: "Act as a Senior QA Engineer. Generate exactly 3 realistic, highly detailed defect reports (bug reports) in Markdown format for a financial money transfer module.

Defect Scenarios to Cover:
1. Defect 1 (Critical / Security): Terminal State Bypass — SMS code is accepted and transfer completes successfully even after the 120-second OTP timer has expired (Expired state violation).
2. Defect 2 (High / Business Logic): Rule Priority Violation — When both the single transfer amount limit (> 500,000 KZT) and cumulative daily limit (> 1,000,000 KZT) are violated, the system incorrectly displays the Daily Limit error instead of the Amount error.
3. Defect 3 (Medium / UI & Lack of Requirements): Undefined Behavior / UI Truncation — Entering a 10-digit number in the amount field causes the transfer button to overlap with the balance display on mobile screens, with no client-side max-length validation.

Required Structure for EACH Defect Report:
- Defect ID (e.g., BUG-TRF-001)
- Title / Summary
- Severity & Priority
- Component / Module
- Environment
- Preconditions
- Steps to Reproduce (numbered list)
- Expected Result
- Actual Result
- Workaround (if any)
Raw output: 
Defect Report 1: Critical / Security
Defect ID: BUG-TRF-001 Title / Summary: SMS OTP accepted and transaction completed after 120-second expiration timeout (Terminal State Violation) Severity: Critical Priority: P1 - Urgent Component / Module: Authentication Service / SMS OTP Module Environment: Staging (v2.4.0-rc1), Mobile App (iOS 17.4 / Android 14), Backend API v2
Preconditions:
User is authenticated and logged into the mobile banking application.
User account balance is at least 200,000 KZT.
User has non-zero daily transfer limit remaining.
Steps to Reproduce:
Initiate a transfer of 200,000 KZT to a valid recipient.
Observe navigation to the SMS OTP Verification screen with a 120-second countdown timer.
Wait for 125 seconds without entering any input until the UI countdown reaches 00:00 and displays "Code Expired".
Enter the original 6-digit OTP code received via SMS during step 1.
Tap the "Confirm Transfer" button.
Expected Result: The system must reject the expired OTP, display an error message ("OTP has expired. Please request a new code"), keep the transaction status as EXPIRED, and leave the account balance unchanged.
Actual Result: The backend accepts the expired OTP code, returns 200 OK with status TRANSACTION_SUCCESS, deducts 200,000 KZT from the user balance, and transfers the funds to the recipient.
Workaround: No client-side or user workaround available. Server-side session validation must be enforced immediately.
Defect Report 2: High / Business Logic
Defect ID: BUG-TRF-002 Title / Summary: Incorrect error message displayed when both single transfer limit and cumulative daily limit are exceeded Severity: High Priority: P2 - High Component / Module: Transfer Validation Engine / Business Rules Engine Environment: Staging (v2.4.0-rc1), Web & Mobile Clients, Backend API v2
Preconditions:
User is logged into the application with an account balance of at least 1,500,000 KZT.
User's accumulated daily transfer total is currently 800,000 KZT (Daily limit: 1,000,000 KZT).
Steps to Reproduce:
Navigate to the Money Transfer screen.
Enter 600,000 KZT in the transfer amount field (violates single transfer limit of 500,000 KZT AND cumulative daily limit of 1,000,000 KZT).
Tap "Continue" / "Transfer".
Expected Result: The input validation rule priority should flag the single transaction limit first: "Transfer amount exceeds single transaction limit of 500,000 KZT".
Actual Result: The system evaluates cumulative limit rules first and displays the incorrect error: "Daily transfer limit of 1,000,000 KZT exceeded", confusing the user about whether a smaller amount below 500,000 KZT is permissible.
Workaround: Users can manually recalculate their remaining daily quota and enter amounts below 500,000 KZT, but the misleading messaging degrades user experience and trust.
Defect Report 3: Medium / UI & Lack of Requirements
Defect ID: BUG-TRF-003 Title / Summary: Lack of client-side input max-length constraint causes transfer button overlap with balance card on small mobile screens Severity: Medium Priority: P3 - Medium Component / Module: Frontend UI / Money Transfer View Environment: Staging (v2.4.0-rc1), iOS App (iPhone SE / 375px viewport width)
Preconditions:
User is logged into the mobile app on a screen with a display width of 375px or smaller.
User opens the Money Transfer screen.
Steps to Reproduce:
Focus on the "Transfer Amount" input field.
Enter a 10-digit numeric string (e.g., 9999999999).
Observe the layout rendering as the text scales.
Expected Result:
Client-side input mask restricts digits to maximum allowed system limit (8 digits / up to 500,000 KZT).
UI layout remains responsive and intact without element collision or overlapping.
Actual Result: The input field accepts unlimited digits without client-side max-length validation. The unformatted text expands vertically and pushes adjacent UI elements down, causing the primary "Transfer" CTA button to visually overlap and obscure the current balance display card.
Workaround: User can delete the extra digits manually to restore normal UI layout layout rendering.
No functional or logical changes were made. The raw AI output correctly covered all three required defect scenarios 
