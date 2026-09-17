# Chart of Accounts — Complete User Stories

## 1. Module Overview

The **Chart of Accounts (COA)** is the master structure used by the Accounting System to classify financial transactions.

The COA should support:

- Account creation and maintenance
- Account hierarchy
- Account numbering
- Account classification
- Parent/child relationships
- Control and posting accounts
- Account activation/deactivation
- Beginning balances
- Account search and filtering
- Import/export
- Audit trail
- Period/year controls
- Integration with General Ledger
- Financial Statement mapping

### Recommended Account Hierarchy

```text
1000 ASSETS
 ├── 1100 CASH AND CASH EQUIVALENTS
 │    ├── 1110 Cash on Hand
 │    ├── 1120 Petty Cash
 │    └── 1130 Cash in Bank
 │         ├── 1131 BDO Checking Account
 │         └── 1132 BPI Savings Account
 │
 ├── 1200 ACCOUNTS RECEIVABLE
 │    ├── 1210 Trade Receivables
 │    └── 1220 Employee Receivables
 │
 └── 1300 INVENTORY

2000 LIABILITIES
 ├── 2100 Accounts Payable
 ├── 2200 Accrued Expenses
 └── 2300 Loans Payable

3000 EQUITY

4000 REVENUE

5000 COST OF SALES

6000 OPERATING EXPENSES
```

---

# Epic 1 — View Chart of Accounts

## US-COA-001 — View Chart of Accounts

**As an Accountant**,  
I want to view the complete Chart of Accounts,  
so that I can review the accounts available in the accounting system.

### Acceptance Criteria

1. The system displays all COA records.
2. Accounts are displayed using their account code and account name.
3. Accounts can be displayed hierarchically.
4. The system displays:
   - Account Code
   - Account Name
   - Account Type
   - Parent Account
   - Account Level
   - Normal Balance
   - Posting Allowed
   - Status
5. Active and inactive accounts are distinguishable.
6. Users can expand/collapse parent accounts.
7. Accounts are sorted by Account Code.
8. Users without permission cannot modify accounts.

---

# Epic 2 — Create Account

## US-COA-002 — Create Account

**As an Accountant**,  
I want to create a new account,  
so that new financial classifications can be recorded.

### Required Fields

```text
Account Code
Account Name
Account Type
Parent Account
Normal Balance
Posting Allowed
Status
Description
```

### Acceptance Criteria

1. Account Code is required.
2. Account Name is required.
3. Account Type is required.
4. Account Code must be unique.
5. Account Name may be duplicated only if business rules permit it.
6. Account Code must follow the configured numbering convention.
7. Parent Account must exist if specified.
8. A child account cannot have itself as its parent.
9. A parent account must belong to a compatible account type.
10. The system prevents duplicate account codes.
11. New accounts default to Active.
12. The system records:
    - Created By
    - Created Date/Time

### Example

```text
Code:            1133
Name:            UnionBank Current Account
Type:            Asset
Parent:          1130 Cash in Bank
Normal Balance:  Debit
Posting Allowed: Yes
Status:          Active
```

---

# Epic 3 — Edit Account

## US-COA-003 — Edit Account

**As an Accountant**,  
I want to edit an existing account,  
so that incorrect or outdated account information can be maintained.

### Acceptance Criteria

The user may modify permitted fields such as:

- Account Name
- Description
- Parent Account
- Posting configuration
- Financial statement mapping
- Other configurable attributes

The system must:

1. Validate the modified data.
2. Prevent duplicate Account Codes.
3. Record the user who modified the account.
4. Record modification date/time.
5. Maintain an audit trail.
6. Warn the user when changing an account that has existing transactions.

### Important Rule

For an account already used in posted transactions:

> **Account Code should normally not be editable.**

Instead, the system should allow the account to be renamed or deactivated subject to permissions.

---

# Epic 4 — Delete Account

## US-COA-004 — Delete Account

**As an Accountant**,  
I want to delete an unused account,  
so that unnecessary accounts do not clutter the Chart of Accounts.

### Acceptance Criteria

An account may only be deleted when:

- It has no posted transactions.
- It has no child accounts.
- It is not referenced by active configuration.
- It is not used by another accounting module.
- It is not mapped to a financial statement.
- It is not used as a default account.

If any condition fails:

```text
This account cannot be deleted because it is currently
being used by the accounting system.
```

### Recommended Design

For accounting systems, **Deactivate/Archive is preferable to physical deletion**.

---

# Epic 5 — Activate / Deactivate Account

## US-COA-005 — Deactivate Account

**As an Accountant**,  
I want to deactivate an account,  
so that it can no longer be used for new transactions while preserving historical records.

### Acceptance Criteria

When an account is deactivated:

- It cannot be selected for new journal entries.
- It cannot be selected in transaction entry.
- Existing historical transactions remain intact.
- Existing reports continue to display the account.
- The account remains searchable.
- The system records who deactivated it and when.

## US-COA-006 — Reactivate Account

An authorized user can reactivate an inactive account.

The system records:

```text
Reactivated By
Reactivated Date
```

---

# Epic 6 — Account Hierarchy

## US-COA-007 — Create Parent Account

**As an Accountant**,  
I want to create parent accounts,  
so that accounts can be organized hierarchically.

Example:

```text
1000 Assets
    1100 Cash
        1110 Cash on Hand
        1120 Petty Cash
```

### Acceptance Criteria

1. Parent accounts may contain child accounts.
2. Parent accounts can optionally be non-posting.
3. Parent accounts cannot be deleted if they contain children.
4. The system calculates hierarchy level automatically.
5. The system prevents circular relationships.

---

# Epic 7 — Posting Account

## US-COA-008 — Configure Posting Permission

**As an Accountant**,  
I want to specify whether an account is a posting account,  
so that transactions can only be posted to appropriate accounts.

Example:

```text
1000 Assets
Posting Allowed: NO

1100 Cash
Posting Allowed: NO

1110 Cash on Hand
Posting Allowed: YES
```

### Acceptance Criteria

If:

```text
Posting Allowed = No
```

the account cannot be selected in a Journal Entry.

If:

```text
Posting Allowed = Yes
```

the account may be selected for transaction posting.

---

# Epic 8 — Account Type

## US-COA-009 — Maintain Account Classification

The system shall support at least:

```text
Asset
Liability
Equity
Revenue
Cost of Sales
Expense
Other Income
Other Expense
```

The classification determines how the account behaves in financial statements.

### Example

```text
1000 Assets
2000 Liabilities
3000 Equity
4000 Revenue
5000 Cost of Sales
6000 Expenses
```

---

# Epic 9 — Normal Balance

## US-COA-010 — Configure Normal Balance

Each account must have a normal balance:

```text
Debit
Credit
```

Typical defaults:

| Account Type | Normal Balance |
|---|---|
| Asset | Debit |
| Expense | Debit |
| Cost of Sales | Debit |
| Liability | Credit |
| Equity | Credit |
| Revenue | Credit |

The system may automatically determine the normal balance from Account Type while allowing exceptions where accounting rules require them.

---

# Epic 10 — Account Code Validation

## US-COA-011 — Validate Account Code

The system shall validate account codes according to the configured COA structure.

Example:

```text
1xxx = Assets
2xxx = Liabilities
3xxx = Equity
4xxx = Revenue
5xxx = Cost of Sales
6xxx = Expenses
```

### Acceptance Criteria

The system must:

- Reject duplicate codes.
- Reject invalid formats.
- Validate parent/child relationships.
- Prevent conflicting account classifications.
- Support configurable code lengths.

---

# Epic 11 — Search and Filter

## US-COA-012 — Search Accounts

**As an Accountant**,  
I want to search for accounts,  
so that I can quickly locate an account.

### Search Criteria

```text
Account Code
Account Name
Account Type
Parent Account
Status
Posting Allowed
```

The system should support partial searches.

Example:

```text
Search: "cash"
```

Results:

```text
1110 Cash on Hand
1120 Petty Cash
1130 Cash in Bank
```

---

# Epic 12 — Account Detail

## US-COA-013 — View Account Details

The user can open an account and see:

```text
Account Code
Account Name
Account Type
Parent Account
Normal Balance
Posting Allowed
Status
Description

Created By
Created Date
Modified By
Modified Date
```

The system should also provide related information:

```text
Transaction Count
Current Balance
Last Transaction Date
Child Accounts
Linked Modules
```

---

# Epic 13 — Account Transaction Inquiry

## US-COA-014 — View Account Transactions

**As an Accountant**,  
I want to view transactions associated with an account,  
so that I can investigate its activity.

The system should show:

```text
Date
Journal Number
Reference
Description
Debit
Credit
Running Balance
Source Module
Posted By
```

Users can filter by:

```text
Date From
Date To
Transaction Type
Reference
```

---

# Epic 14 — Account Balance

## US-COA-015 — View Account Balance

The system calculates the account balance from posted transactions.

Conceptually:

```text
Debit Balance  = Total Debit - Total Credit
Credit Balance = Total Credit - Total Debit
```

The system should distinguish:

```text
Opening Balance
Current Period Activity
Closing Balance
```

---

# Epic 15 — Beginning Balance

## US-COA-016 — Enter Beginning Balance

**As an Accountant**,  
I want to enter beginning balances for accounts,  
so that the system can start accounting from an existing financial position.

### Acceptance Criteria

1. Beginning balances are entered for a defined fiscal year/period.
2. Debit and credit amounts are validated.
3. Total debits must equal total credits.
4. Beginning balances become part of the accounting history.
5. Changes require appropriate authorization.
6. The system records an audit trail.

---

# Epic 16 — Bulk Import

## US-COA-017 — Import Chart of Accounts

**As an Accountant**,  
I want to import multiple accounts from Excel/CSV,  
so that I don't need to manually create hundreds of accounts.

Example:

```text
AccountCode,AccountName,AccountType,ParentCode,PostingAllowed
1000,Assets,Asset,,N
1100,Cash,Asset,1000,N
1110,Cash on Hand,Asset,1100,Y
```

### Acceptance Criteria

Before importing, the system validates:

- Duplicate codes
- Missing account names
- Invalid account types
- Invalid parent accounts
- Circular relationships
- Invalid posting configuration
- Invalid codes

The system provides an import preview:

```text
Valid Records:       245
Invalid Records:       7
Duplicate Records:     3
```

The user can review errors before committing the import.

---

# Epic 17 — Export

## US-COA-018 — Export Chart of Accounts

**As an Accountant**,  
I want to export the COA,  
so that I can use it for reporting, review, or external processing.

### Supported Formats

```text
Excel
CSV
PDF
```

### Export Filters

```text
Active only
Inactive only
Account Type
Account Level
Posting Accounts
Full hierarchy
```

---

# Epic 18 — Financial Statement Mapping

## US-COA-019 — Map Accounts to Financial Statements

**As an Accountant**,  
I want to map accounts to financial statement classifications,  
so that the system can automatically generate financial reports.

### Possible Mappings

```text
Balance Sheet
    Current Assets
    Non-Current Assets
    Current Liabilities
    Non-Current Liabilities
    Equity

Income Statement
    Revenue
    Cost of Sales
    Operating Expenses
    Other Income
    Other Expenses
```

Example:

```text
Account: 1130 Cash in Bank

Financial Statement:
Balance Sheet

Section:
Current Assets
```

---

# Epic 19 — Default Account Configuration

## US-COA-020 — Configure Default Accounts

**As an Accountant**,  
I want to configure default accounts for different modules,  
so that transactions can automatically use the appropriate GL accounts.

Examples:

```text
Accounts Receivable
Accounts Payable
Sales Revenue
Purchase Expense
Inventory
Cash
Bank
VAT Input
VAT Output
Payroll Expense
Salary Payable
Tax Payable
```

Example:

```text
Sales Invoice
       ↓
Accounts Receivable
       ↓
Sales Revenue
       ↓
Output VAT
```

---

# Epic 20 — Integration With General Ledger

## US-COA-021 — Use COA During Journal Entry

When a user creates a Journal Entry:

```text
Journal Entry
------------------------------
Account        Debit    Credit
1110 Cash      10,000
4100 Revenue             10,000
```

The system shall:

1. Display only active posting accounts.
2. Validate account existence.
3. Validate posting permission.
4. Validate account status.
5. Save account references using Account ID.
6. Prevent deleted account references.

---

# Epic 21 — Prevent Invalid Posting

## US-COA-022 — Prevent Posting to Non-Posting Account

If the user attempts to post to:

```text
1000 Assets
```

and:

```text
Posting Allowed = No
```

the system rejects the transaction.

Example message:

```text
Account 1000 - Assets is a control account and cannot
be used for transaction posting.
Please select a posting-level account.
```

---

# Epic 22 — Period Lock Protection

## US-COA-023 — Protect Accounts Used in Closed Periods

The system shall prevent changes that could compromise historical accounting data.

For example, if an account has transactions in a closed fiscal period:

- Account history cannot be destroyed.
- Historical transactions remain associated with the account.
- Account code changes should be restricted.
- Deletion is prohibited.
- Changes may require elevated authorization.

---

# Epic 23 — Audit Trail

## US-COA-024 — Record Account Changes

Every important COA modification shall be recorded.

### Audit Information

```text
Audit ID
Account ID
Action
Old Value
New Value
Changed By
Changed Date/Time
Reason
IP Address / Session
```

### Actions

```text
CREATE
UPDATE
ACTIVATE
DEACTIVATE
DELETE
IMPORT
REASSIGN
```

Example:

```text
Account: 6100 Office Supplies

Action: UPDATE

Field:
Account Name

Old:
Office Supplies

New:
Office and Pantry Supplies

Changed By:
admin

Date:
2026-09-18 10:31:22
```

---

# Epic 24 — Authorization

## US-COA-025 — Control COA Permissions

The system shall support permissions such as:

```text
CanViewCOA
CanCreateCOA
CanEditCOA
CanDeleteCOA
CanActivateCOA
CanImportCOA
CanExportCOA
CanViewCOAAudit
CanChangeAccountMapping
CanChangeDefaultAccounts
```

### Example Role Structure

| Role | View | Create | Edit | Delete | Import |
|---|---:|---:|---:|---:|---:|
| Administrator | ✓ | ✓ | ✓ | ✓ | ✓ |
| Accounting Manager | ✓ | ✓ | ✓ | Limited | ✓ |
| Accountant | ✓ | ✓ | ✓ | No | ✓ |
| Accounting Staff | ✓ | Limited | No | No | No |
| Auditor | ✓ | No | No | No | No |

---

# Epic 25 — Account Merge / Replacement

## US-COA-026 — Replace Obsolete Account

Instead of deleting an account that has historical transactions, the system may allow it to be replaced by another account.

Example:

```text
Old Account:
6205 Telephone Expense

Replace With:
6210 Communication Expense
```

The old account becomes inactive.

Historical transactions remain:

```text
Historical Transaction
        ↓
6205 Telephone Expense
```

New transactions use:

```text
6210 Communication Expense
```

This is preferable to changing historical transaction references.

---

# Epic 26 — Reclassification

## US-COA-027 — Reclassify Account

An authorized accountant can change an account's classification when appropriate.

Example:

```text
Current:
6100 Operating Expense

New:
6200 Administrative Expense
```

The system should distinguish between:

- Changing the account's master-data classification.
- Creating an accounting reclassification journal entry.

**Important:** Changing the COA classification should not silently rewrite historical accounting transactions unless explicitly designed and authorized.

---

# Epic 27 — Multi-Company Support

## US-COA-028 — Maintain Company-Specific COA

If the Accounting System supports multiple companies:

```text
Company
    ↓
Chart of Accounts
    ↓
Account
```

Example:

```text
Company A
    1000 Assets

Company B
    1000 Assets
```

The same Account Code may exist in different companies if COAs are company-specific.

---

# Epic 28 — Fiscal Year Handling

## US-COA-029 — Carry COA Into New Fiscal Year

The system should allow the same COA to continue into a new fiscal year.

The system should distinguish:

```text
Account Master
        +
Fiscal Year Balances
```

The account itself should not normally be duplicated every year.

---

# Epic 29 — Year-End Processing

## US-COA-030 — Close Fiscal Year

During year-end:

### Balance Sheet Accounts

Balances carry forward:

```text
Assets
Liabilities
Equity
```

### Income Statement Accounts

Revenue and expense accounts are closed to the appropriate equity/retained earnings mechanism according to the accounting design.

The COA remains available for the next fiscal year.

---

# Epic 30 — Trial Balance Integration

## US-COA-031 — Generate Trial Balance From COA

The system shall generate:

```text
Account Code
Account Name
Debit
Credit
Balance
```

Only accounts with activity may optionally be displayed.

The Trial Balance must satisfy:

```text
Total Debits = Total Credits
```

---

# Epic 31 — General Ledger Integration

## US-COA-032 — Generate General Ledger

The system uses COA records to organize transactions into individual account ledgers.

Example:

```text
1130 Cash in Bank
-------------------------------
Date       Reference       Debit
09/01      JE-0001        50,000
09/05      OR-0012         5,000
09/10      AP-0021                  10,000
-------------------------------
Ending Balance            45,000
```

---

# Epic 32 — Financial Reports

## US-COA-033 — Generate Financial Statements

The COA shall drive financial reporting.

### Balance Sheet

```text
Assets
Liabilities
Equity
```

### Income Statement

```text
Revenue
Cost of Sales
Gross Profit
Operating Expenses
Net Income
```

### Trial Balance

```text
Debit
Credit
```

### General Ledger

```text
Account-level transactions
```

---

# Epic 33 — Account Usage Inquiry

## US-COA-034 — View Where Account Is Used

**As an Accounting Manager**,  
I want to know where an account is configured,  
so that I can safely modify or deactivate it.

The system should identify references such as:

```text
Journal Entries
Sales
Purchases
Accounts Receivable
Accounts Payable
Inventory
Payroll
Tax Configuration
Bank Accounts
Cash Accounts
Fixed Assets
Default Accounts
Financial Statement Mapping
```

---

# Epic 34 — Duplicate Account Detection

## US-COA-035 — Detect Similar Accounts

The system may warn users when creating accounts with similar names.

Example:

```text
Existing:
6100 Office Supplies

New:
6120 Office Supply Expense
```

Warning:

```text
A similar account already exists.
Do you want to continue?
```

This should be a warning rather than necessarily blocking the transaction.

---

# Epic 35 — Account Notes

## US-COA-036 — Maintain Account Notes

Users with permission can maintain notes about an account.

Example:

```text
Account:
1131 BDO Checking

Notes:
Primary operating bank account.
Used for payroll and supplier payments.
```

Notes should not alter accounting transactions.

---

# Epic 36 — COA Dashboard

## US-COA-037 — View COA Dashboard

The COA dashboard can provide:

```text
Total Accounts
Active Accounts
Inactive Accounts
Posting Accounts
Non-Posting Accounts
Assets
Liabilities
Equity
Revenue
Expenses
```

Example:

```text
Chart of Accounts

Total Accounts       247
Active               231
Inactive              16
Posting Accounts     198
Control Accounts      33
```

---

# Epic 37 — COA Validation Utility

## US-COA-038 — Validate Chart of Accounts

An administrator/accounting manager can run a COA validation process.

The system checks:

```text
Duplicate Account Codes
Missing Parent Accounts
Invalid Account Types
Circular Hierarchies
Inactive Parent Accounts
Invalid Posting Accounts
Missing Financial Statement Mapping
Invalid Default Account References
Orphan Accounts
```

Example:

```text
COA Validation Result

✓ Duplicate Accounts       0
✓ Orphan Accounts          0
⚠ Missing Mapping          3
✓ Circular References      0
⚠ Inactive Parent          1
```

---

# Epic 38 — Import Validation and Rollback

## US-COA-039 — Transactional COA Import

The import process should be atomic where practical.

Example:

```text
500 records imported

Valid:      495
Invalid:      5
```

If configured as all-or-nothing:

```text
Import failed.

No records were committed.
Please correct the 5 errors.
```

This prevents partially corrupted COA structures.

---

# Epic 39 — Concurrency Control

## US-COA-040 — Prevent Conflicting Updates

If two users edit the same account:

```text
User A → opens account
User B → modifies account
User A → attempts save
```

The system detects that the record has changed and displays:

```text
This account was modified by another user.
Please reload the record before saving your changes.
```

---

# Epic 40 — Account Deletion Protection

## US-COA-041 — Check Dependencies Before Deletion

Before deleting an account, the system performs:

```text
Check Transactions
       ↓
Check Child Accounts
       ↓
Check Default Accounts
       ↓
Check Financial Statement Mapping
       ↓
Check Module References
       ↓
Allow / Reject Delete
```

This should be implemented at the **database/service layer**, not only through the UI.

---

# Epic 41 — REST/API Support

If your Accounting System is being developed using **C# MVC/.NET**, the COA functionality should also be service/API-ready.

Suggested endpoints:

```text
GET    /api/accounts
GET    /api/accounts/{id}

POST   /api/accounts
PUT    /api/accounts/{id}
DELETE /api/accounts/{id}

POST   /api/accounts/{id}/activate
POST   /api/accounts/{id}/deactivate

GET    /api/accounts/{id}/transactions
GET    /api/accounts/{id}/balance

POST   /api/accounts/import
GET    /api/accounts/export

GET    /api/accounts/{id}/dependencies
GET    /api/accounts/{id}/audit
```

---

# Epic 42 — Recommended Database Structure

For the Accounting System, separate the **account master** from transactions.

## `ChartOfAccounts`

```text
AccountID PK
CompanyID FK
AccountCode
AccountName
AccountTypeID FK
ParentAccountID FK
NormalBalance
IsPostingAccount
IsActive
Description
CreatedBy
CreatedDate
ModifiedBy
ModifiedDate
```

## `AccountTypes`

```text
AccountTypeID PK
AccountTypeCode
AccountTypeName
NormalBalance
FinancialStatementType
IsActive
```

## `JournalEntries`

```text
JournalEntryID PK
JournalNumber
JournalDate
Description
Status
CreatedBy
PostedBy
PostedDate
```

## `JournalEntryDetails`

```text
JournalEntryDetailID PK
JournalEntryID FK
AccountID FK
Debit
Credit
Description
```

## `AccountAuditLogs`

```text
AuditID PK
AccountID FK
Action
FieldName
OldValue
NewValue
ChangedBy
ChangedDate
Reason
```

---

# Recommended End-to-End COA Workflow

```text
                    ┌──────────────────────┐
                    │   Chart of Accounts  │
                    └──────────┬───────────┘
                               │
                ┌──────────────┴──────────────┐
                │                             │
          Create Account                Import Accounts
                │                             │
                └──────────────┬──────────────┘
                               │
                         Validate COA
                               │
                    ┌──────────┴──────────┐
                    │                     │
                  Valid                 Invalid
                    │                     │
                    ▼                     ▼
              Save Account          Show Errors
                    │
                    ▼
             Account Hierarchy
                    │
                    ▼
          Financial Statement Mapping
                    │
                    ▼
             Default Account Setup
                    │
                    ▼
              Journal Entry
                    │
                    ▼
             General Ledger
                    │
          ┌─────────┼─────────┐
          ▼         ▼         ▼
      Trial       Balance   Income
      Balance     Sheet    Statement
```

---

# Suggested COA Screen Structure

A **master-detail layout** is recommended rather than having separate pages for every operation.

## Left / Main Panel

```text
Chart of Accounts

[ Search __________________ ] [Type ▼] [Status ▼]

+ Add Account    Import    Export    Validate

1000  ASSETS
  1100  CASH
    1110  Cash on Hand
    1120  Petty Cash
    1130  Cash in Bank
      1131 BDO Checking
      1132 BPI Savings

2000  LIABILITIES
  2100  Accounts Payable
  2200  Accrued Expenses

3000  EQUITY
4000  REVENUE
5000  COST OF SALES
6000  EXPENSES
```

## Right / Detail Panel

```text
Account Details

Account Code       [1131             ]
Account Name       [BDO Checking     ]

Account Type       [Asset ▼]
Parent Account     [1130 ▼]

Normal Balance     [Debit]
Posting Account    [✓]
Status             [Active ▼]

Description
[________________________________]

Financial Statement
[Balance Sheet ▼]

Section
[Current Assets ▼]

[Save] [Cancel]
```

---

# Overall User Story Breakdown

| Epic | Scope |
|---|---|
| COA-01 | View COA |
| COA-02 | Create Account |
| COA-03 | Edit Account |
| COA-04 | Delete Account |
| COA-05 | Activate/Deactivate |
| COA-06 | Account Hierarchy |
| COA-07 | Posting Controls |
| COA-08 | Account Classification |
| COA-09 | Account Validation |
| COA-10 | Search/Filter |
| COA-11 | Account Inquiry |
| COA-12 | Beginning Balances |
| COA-13 | Import/Export |
| COA-14 | Financial Statement Mapping |
| COA-15 | Default Account Mapping |
| COA-16 | General Ledger Integration |
| COA-17 | Trial Balance |
| COA-18 | Financial Statements |
| COA-19 | Audit Trail |
| COA-20 | Authorization |
| COA-21 | Reclassification |
| COA-22 | Account Replacement |
| COA-23 | Fiscal Year |
| COA-24 | COA Validation Utility |
| COA-25 | Dependency Checking |
| COA-26 | Concurrency |
| COA-27 | API/Integration |

---

# Key Design Principle

For an accounting system, **CRUD alone is not enough**.

The most important distinction is:

> **COA is master data, while Journal Entries and their details are financial transaction data.**

Once an account has been referenced by posted accounting transactions, the system should generally **preserve the historical reference rather than physically delete or rewrite it**.

This principle protects:

- General Ledger
- Trial Balance
- Balance Sheet
- Income Statement
- Audit Trail
- Year-End Processing
- Historical Financial Reports

The COA should therefore serve as the foundation for the General Ledger module and subsequent modules such as:

- Accounts Receivable
- Accounts Payable
- Cash/Bank
- Sales
- Purchases
- Inventory
- Fixed Assets
- Payroll
- Tax
- Financial Reporting
