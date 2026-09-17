# Accounting System Development Roadmap

## Feature Sequence

1.  **Chart of Accounts**
2.  **Accounting Periods**
3.  **General Ledger / Journal Entry**
4.  **Cash & Bank**
5.  **Accounts Receivable**
6.  **Accounts Payable**
7.  **Sales**
8.  **Purchasing**
9.  **Inventory**
10. **Fixed Assets**
11. **Payroll Integration**
12. **Tax / VAT**
13. **Financial Statements**
14. **Year-End Closing**
15. **Audit & Internal Controls**

------------------------------------------------------------------------

# 1. Accounting Periods

Accounting Period Management should be implemented immediately after the
Chart of Accounts because every accounting transaction must belong to an
accounting period.

### Main Functions

-   Create accounting periods
-   Define period start and end dates
-   Open a period
-   Close a period
-   Reopen a period when authorized
-   Prevent posting to closed periods
-   Validate transaction dates against periods
-   Manage fiscal years
-   Track period status
-   Maintain an audit trail

### Example

  Period    Start Date   End Date     Status
  --------- ------------ ------------ --------
  2026-09   09/01/2026   09/30/2026   OPEN
  2026-10   10/01/2026   10/31/2026   OPEN

### Recommended Status Flow

``` text
OPEN
  ↓
CLOSING
  ↓
CLOSED
```

A closed period should prevent normal transaction posting.

------------------------------------------------------------------------

# 2. General Ledger / Journal Entry

The **General Ledger (GL)** is the accounting engine of the system.

The Chart of Accounts defines **where** transactions are classified,
while Journal Entries define **how** financial transactions are
recorded.

## Journal Entry Lifecycle

``` text
DRAFT
  ↓
SUBMITTED
  ↓
APPROVED
  ↓
POSTED
```

Alternative paths:

``` text
DRAFT → CANCELLED

SUBMITTED → REJECTED

POSTED → REVERSED
```

A posted journal entry should generally **not be edited or physically
deleted**. Corrections should be performed through a reversal and/or
correcting journal entry.

------------------------------------------------------------------------

# 3. Journal Entry Core Functions

## 3.1 Create Journal Entry

The system should allow authorized users to create a journal entry
containing:

-   Journal Number
-   Journal Date
-   Accounting Period
-   Journal Type
-   Description
-   Reference Number
-   Source Module
-   Currency
-   Exchange Rate, if applicable
-   Supporting Document
-   Journal Lines

### Example

**Journal Number:** JE-2026-000001

**Date:** September 18, 2026

**Description:** Purchase of office supplies

  Account                           Debit         Credit
  ------------------------ -------------- --------------
  6100 - Office Supplies         5,000.00           0.00
  1100 - Cash in Bank                0.00       5,000.00
  **Total**                  **5,000.00**   **5,000.00**

------------------------------------------------------------------------

# 4. Journal Entry Validation

Before a journal entry can be posted, the system should validate:

1.  Debit total equals Credit total
2.  At least one debit exists
3.  At least one credit exists
4.  Accounts exist
5.  Accounts are active
6.  Accounts are posting accounts
7.  Accounting period is open
8.  Transaction date belongs to the selected period
9.  User has permission to post
10. Required fields are completed
11. Journal number is unique
12. Duplicate transaction references are prevented where required

### Core Accounting Rule

``` text
Total Debit = Total Credit
```

Example:

``` text
Debit  = 10,000
Credit = 10,000

VALID
```

But:

``` text
Debit  = 10,000
Credit = 9,500

INVALID
```

------------------------------------------------------------------------

# 5. Journal Types

The system should support different journal types.

Recommended types:

-   General Journal
-   Adjusting Journal
-   Closing Journal
-   Reversing Journal
-   Recurring Journal
-   Opening Balance Journal
-   Reclassification Journal
-   Intercompany Journal, if required

------------------------------------------------------------------------

# 6. Journal Entry Approval

For organizations requiring internal controls, journal entries should
support an approval workflow.

### Example

``` text
DRAFT
   ↓
SUBMITTED
   ↓
APPROVED
   ↓
POSTED
```

The system should record:

-   Created By
-   Submitted By
-   Approved By
-   Posted By
-   Approval Date
-   Posting Date
-   Rejection Reason
-   Reversal Reason

Users should not be allowed to approve their own journal entries when
segregation-of-duties rules prohibit it.

------------------------------------------------------------------------

# 7. Journal Posting

Posting is the process that makes the journal entry part of the official
accounting records.

When a journal entry is posted:

1.  Validate the journal
2.  Validate the accounting period
3.  Confirm the user has posting permission
4.  Lock the journal against normal editing
5.  Update the General Ledger
6.  Update account balances
7.  Mark the journal as POSTED
8.  Record posting information
9.  Create an audit trail

### Posting Flow

``` text
Journal Entry
      ↓
Validation
      ↓
Approval
      ↓
Posting
      ↓
General Ledger
      ↓
Trial Balance
      ↓
Financial Statements
```

------------------------------------------------------------------------

# 8. Reversal of Posted Journal Entries

A posted journal entry should not simply be deleted.

Instead, the system should provide a **Reverse Journal Entry** function.

Example original transaction:

``` text
Office Supplies     Debit   5,000
Cash in Bank        Credit  5,000
```

Reversal:

``` text
Office Supplies     Credit  5,000
Cash in Bank        Debit   5,000
```

The system should record:

-   Original Journal Number
-   Reversal Journal Number
-   Reversal Date
-   Reversal Reason
-   Reversed By
-   Reversal Timestamp

------------------------------------------------------------------------

# 9. Recurring Journal Entries

The system should support recurring accounting transactions such as:

-   Monthly rent
-   Monthly depreciation
-   Insurance expense
-   Subscription expenses
-   Accruals
-   Management fees

A recurring journal template may contain:

-   Template Code
-   Description
-   Frequency
-   Start Date
-   End Date
-   Number of occurrences
-   Journal Lines
-   Default amounts
-   Active/Inactive status

The system should generate a draft journal for review before posting
when organizational controls require it.

------------------------------------------------------------------------

# 10. Journal Templates

Users should be able to create reusable journal templates.

Example:

**Monthly Rent**

``` text
Rent Expense       Debit
Cash/Bank          Credit
```

Template benefits:

-   Faster data entry
-   Reduced errors
-   Consistent account usage
-   Standardized recurring transactions

------------------------------------------------------------------------

# 11. Opening Balances

The system should support opening balances when starting a new fiscal
year or migrating from another accounting system.

Opening balances should be recorded through controlled journal entries.

Example:

``` text
Cash in Bank       Debit   500,000
Accounts Receivable Debit  200,000
Equipment          Debit   300,000
Retained Earnings  Credit  600,000
Accounts Payable   Credit  400,000
```

The opening balance journal must remain balanced.

------------------------------------------------------------------------

# 12. General Ledger

The General Ledger should provide transaction-level accounting history
for each account.

Users should be able to view:

-   Account Code
-   Account Name
-   Transaction Date
-   Journal Number
-   Description
-   Reference Number
-   Debit
-   Credit
-   Running Balance
-   Source Module
-   Posted By

### Example

  Date         Journal   Description            Debit   Credit   Balance
  ------------ --------- ------------------ --------- -------- ---------
  09/01/2026   JE-0001   Opening Balance      500,000        0   500,000
  09/05/2026   JE-0005   Office Supplies            0    5,000   495,000
  09/10/2026   JE-0010   Customer Payment      25,000        0   520,000

------------------------------------------------------------------------

# 13. Account Inquiry

Users should be able to select an account from the Chart of Accounts and
view:

-   Current balance
-   Beginning balance
-   Total debits
-   Total credits
-   Net movement
-   Transaction history
-   Current period activity
-   Prior period activity
-   Fiscal-year activity

Example:

``` text
Account: 1100 - Cash in Bank

Beginning Balance     500,000
Total Debits          250,000
Total Credits         175,000
------------------------------
Ending Balance        575,000
```

------------------------------------------------------------------------

# 14. Trial Balance Integration

The General Ledger should feed the Trial Balance.

The Trial Balance should show:

  Account                         Debit          Credit
  --------------------- --------------- ---------------
  Cash                          575,000               0
  Accounts Receivable           200,000               0
  Equipment                     300,000               0
  Accounts Payable                    0         400,000
  Retained Earnings                   0         675,000
  **Total**               **1,075,000**   **1,075,000**

The system should verify:

``` text
Total Debits = Total Credits
```

------------------------------------------------------------------------

# 15. Integration Architecture

All accounting modules should use the same General Ledger posting
engine.

Recommended architecture:

``` text
                    Chart of Accounts
                           │
                           ▼
                  Accounting Periods
                           │
                           ▼
                    Journal Entry
                           │
                           ▼
                     Validation
                           │
                           ▼
                      Approval
                           │
                           ▼
                       Posting
                           │
                           ▼
                  General Ledger
                     /         \
                    ▼           ▼
              Trial Balance   Account Inquiry
                    │
                    ▼
             Financial Reports
```

Operational modules should generate accounting transactions into the
same GL engine.

``` text
Sales ───────────────┐
Purchasing ──────────┤
Accounts Receivable ─┤
Accounts Payable ────┤
Cash & Bank ─────────┤
Inventory ───────────┤
Fixed Assets ────────┤
Payroll ─────────────┤
                     ▼
              Journal Entry Engine
                     │
                     ▼
              General Ledger
```

This avoids maintaining separate accounting logic in every module.

------------------------------------------------------------------------

# 16. Database Structure

A recommended starting structure is:

## AccountingPeriods

``` text
AccountingPeriodID PK
FiscalYear
PeriodNumber
PeriodName
StartDate
EndDate
Status
IsAdjustmentPeriod
CreatedBy
CreatedDate
ClosedBy
ClosedDate
```

## JournalEntries

``` text
JournalEntryID PK
JournalNumber
JournalDate
AccountingPeriodID FK
JournalTypeID FK
Description
ReferenceNumber
SourceModule
Status
CreatedBy
CreatedDate
SubmittedBy
SubmittedDate
ApprovedBy
ApprovedDate
PostedBy
PostedDate
ReversalOfJournalEntryID FK
ReversalReason
```

## JournalEntryDetails

``` text
JournalEntryDetailID PK
JournalEntryID FK
LineNumber
AccountID FK
Description
Debit
Credit
ReferenceNumber
DepartmentID FK
CostCenterID FK
ProjectID FK
```

## JournalTypes

``` text
JournalTypeID PK
JournalTypeCode
JournalTypeName
IsActive
```

## JournalAuditLogs

``` text
AuditLogID PK
JournalEntryID FK
Action
OldStatus
NewStatus
Reason
PerformedBy
PerformedDate
```

------------------------------------------------------------------------

# 17. Important Accounting Controls

The accounting system should enforce the following rules.

### Posted Transaction Immutability

Once posted:

``` text
EDIT = NOT ALLOWED
DELETE = NOT ALLOWED
```

Correction should use:

``` text
REVERSAL + CORRECTING JOURNAL
```

### Closed Period Protection

``` text
Period = CLOSED
        ↓
Posting = NOT ALLOWED
```

### Balanced Journal Requirement

``` text
Debit Total = Credit Total
```

### Account Validation

A journal should only use:

``` text
Active Account
+
Posting Account
```

### Audit Trail

Important events should be logged:

-   Create
-   Edit
-   Submit
-   Approve
-   Reject
-   Post
-   Reverse
-   Cancel
-   Period Close
-   Period Reopen

------------------------------------------------------------------------

# 18. User Interface Structure

Recommended Accounting menu:

``` text
Accounting
│
├── Chart of Accounts
│
├── Accounting Periods
│
├── General Ledger
│   ├── Journal Entries
│   ├── Journal Templates
│   ├── Recurring Journals
│   └── Account Inquiry
│
├── Trial Balance
│
├── Reports
│
└── Period Closing
```

### Journal Entry Screen

Recommended layout:

``` text
----------------------------------------------------
Journal Entry
----------------------------------------------------

Journal No:      JE-2026-000001
Journal Date:    09/18/2026
Period:          September 2026
Journal Type:    General Journal
Reference No:    INV-2026-1001

Description:
[ Purchase of office supplies ]

----------------------------------------------------
Account             Description       Debit     Credit
----------------------------------------------------
6100 Office Supply  Office supplies   5,000.00
1100 Cash in Bank   Payment                       5,000.00
----------------------------------------------------
                                    5,000.00     5,000.00

[Save Draft] [Submit] [Cancel]
----------------------------------------------------
```

For approved entries:

``` text
[Post]
```

For posted entries:

``` text
[View]
[Reverse]
```

------------------------------------------------------------------------

# 19. Recommended Development Order

For the Accounting System, the implementation sequence should be:

### Phase 1 --- Foundation

1.  Company Setup
2.  Fiscal Year
3.  Accounting Periods
4.  Chart of Accounts
5.  Users and Permissions

### Phase 2 --- General Ledger

6.  Journal Types
7.  Journal Entry
8.  Journal Entry Details
9.  Journal Validation
10. Journal Approval
11. Journal Posting
12. Journal Reversal
13. General Ledger
14. Account Inquiry
15. Opening Balances
16. Recurring Journals
17. Journal Templates

### Phase 3 --- Subledgers

18. Cash & Bank
19. Accounts Receivable
20. Accounts Payable
21. Sales
22. Purchasing
23. Inventory
24. Fixed Assets
25. Payroll Integration

### Phase 4 --- Reporting

26. Trial Balance
27. General Ledger Report
28. Balance Sheet
29. Income Statement
30. Cash Flow Statement
31. Account Analysis
32. Tax Reports

### Phase 5 --- Closing & Controls

33. Month-End Closing
34. Year-End Closing
35. Closing Entries
36. Retained Earnings Processing
37. Audit Trail
38. Audit Reports
39. Period Reopening Controls
40. System Audit / Security Reports

------------------------------------------------------------------------

# 20. Key Design Principle

The most important architectural principle is:

> **All financial transactions should ultimately flow through one
> controlled General Ledger posting engine.**

For example:

``` text
Sales Invoice
      ↓
Accounts Receivable
      ↓
Journal Entry
      ↓
General Ledger
```

``` text
Supplier Invoice
      ↓
Accounts Payable
      ↓
Journal Entry
      ↓
General Ledger
```

``` text
Cash Receipt
      ↓
Cash & Bank
      ↓
Journal Entry
      ↓
General Ledger
```

``` text
Payroll
      ↓
Payroll Accounting
      ↓
Journal Entry
      ↓
General Ledger
```

This architecture keeps the accounting records consistent and makes the
Trial Balance and Financial Statements reliable.

------------------------------------------------------------------------

# Recommended Next Development

If **Chart of Accounts is already completed**, the immediate next
modules should be:

### 1. Accounting Periods

This provides the period-control foundation.

### 2. General Ledger / Journal Entry

This becomes the central accounting transaction engine.

The relationship is:

``` text
Chart of Accounts
       ↓
Accounting Periods
       ↓
Journal Entry
       ↓
Approval
       ↓
Posting
       ↓
General Ledger
       ↓
Trial Balance
       ↓
Financial Statements
```

Once this core is working, the other modules such as AR, AP, Cash &
Bank, Sales, Purchasing, Inventory, and Fixed Assets can be connected to
it rather than implementing separate accounting logic.
