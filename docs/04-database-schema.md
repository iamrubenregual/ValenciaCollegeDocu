# 04 — Database Schema

All tables include the following **audit columns** unless stated otherwise:

| Column | Type | Description |
|--------|------|-------------|
| `CreatedAt` | `datetime2` NOT NULL | Row creation timestamp |
| `CreatedBy` | `nvarchar(100)` NOT NULL | Username of creator |
| `UpdatedAt` | `datetime2` NULL | Last update timestamp |
| `UpdatedBy` | `nvarchar(100)` NULL | Username of last updater |
| `IsDeleted` | `bit` NOT NULL DEFAULT 0 | Soft-delete flag |
| `DeletedAt` | `datetime2` NULL | Deletion timestamp |
| `RowVersion` | `rowversion` | Optimistic concurrency token |

---

## 4.1 Entity Relationship Overview

```
AcademicYear ──────< Semester
Semester ──────────< FeeMatrix ──── FeeType ──── ChartOfAccount
Semester ──────────< StudentBilling ──────< BillingLine ──── FeeType
StudentAccount ────< StudentBilling
StudentAccount ────< PaymentTransaction ──────< PaymentReceipt
StudentAccount ────< ScholarshipAward ──── ScholarshipType
PaymentTransaction ─── ChartOfAccount (DR/CR via JournalEntry)

JournalEntry ──────< JournalEntryLine ──── ChartOfAccount

Employee ──────────< PayrollRun ──────────< PayslipLine
Employee ──────────< EmployeeDeduction

Vendor ─────────────< PurchaseOrder ──────< PurchaseOrderLine
PurchaseOrder ──────< ReceivingReport
ReceivingReport ────< DisbursementVoucher ──────< DvPaymentLine

Budget ─────────────< BudgetLine ──── ChartOfAccount
BudgetLine ─────────< BudgetObligation
```

---

## 4.2 Table Definitions

### Reference / Setup Tables

#### `AcademicYears`
| Column | Type | Notes |
|--------|------|-------|
| `AcademicYearId` | `int` PK IDENTITY | |
| `Code` | `nvarchar(20)` UNIQUE NOT NULL | e.g. `2025-2026` |
| `StartDate` | `date` NOT NULL | |
| `EndDate` | `date` NOT NULL | |
| `IsActive` | `bit` NOT NULL DEFAULT 1 | |

#### `Semesters`
| Column | Type | Notes |
|--------|------|-------|
| `SemesterId` | `int` PK IDENTITY | |
| `AcademicYearId` | `int` FK → AcademicYears | |
| `Name` | `nvarchar(50)` NOT NULL | e.g. `1st Semester` |
| `SemesterOrder` | `tinyint` NOT NULL | 1, 2, 3 (Summer) |
| `StartDate` | `date` NOT NULL | |
| `EndDate` | `date` NOT NULL | |
| `EnrollmentDeadline` | `date` NULL | |
| `IsOpen` | `bit` NOT NULL DEFAULT 0 | Controls posting access |
| `IsClosed` | `bit` NOT NULL DEFAULT 0 | Hard-closed flag |

#### `Departments`
| Column | Type | Notes |
|--------|------|-------|
| `DepartmentId` | `int` PK IDENTITY | |
| `Code` | `nvarchar(20)` UNIQUE NOT NULL | |
| `Name` | `nvarchar(200)` NOT NULL | |
| `ParentDepartmentId` | `int` NULL FK → Departments | For college/department hierarchy |

#### `Programmes`
| Column | Type | Notes |
|--------|------|-------|
| `ProgrammeId` | `int` PK IDENTITY | |
| `DepartmentId` | `int` FK → Departments | |
| `Code` | `nvarchar(20)` UNIQUE NOT NULL | e.g. `BSAC` |
| `Name` | `nvarchar(200)` NOT NULL | |
| `YearDuration` | `tinyint` NOT NULL | 4, 5, 2, etc. |

---

### Chart of Accounts

#### `AccountGroups`
| Column | Type | Notes |
|--------|------|-------|
| `AccountGroupId` | `int` PK IDENTITY | |
| `Code` | `nvarchar(10)` UNIQUE NOT NULL | e.g. `1`, `2`, `3` |
| `Name` | `nvarchar(100)` NOT NULL | Asset, Liability, Equity, Revenue, Expense |
| `NormalBalance` | `nvarchar(6)` NOT NULL | `Debit` or `Credit` |

#### `ChartOfAccounts`
| Column | Type | Notes |
|--------|------|-------|
| `AccountId` | `int` PK IDENTITY | |
| `AccountGroupId` | `int` FK → AccountGroups | |
| `ParentAccountId` | `int` NULL FK → ChartOfAccounts | Hierarchical COA |
| `Code` | `nvarchar(20)` UNIQUE NOT NULL | e.g. `1010-001` |
| `Name` | `nvarchar(200)` NOT NULL | |
| `AccountType` | `nvarchar(30)` NOT NULL | Asset / Liability / Equity / Revenue / Expense |
| `IsPostable` | `bit` NOT NULL DEFAULT 1 | False for summary/header accounts |
| `IsActive` | `bit` NOT NULL DEFAULT 1 | |
| `Description` | `nvarchar(500)` NULL | |

---

### Student Billing

#### `StudentAccounts`
| Column | Type | Notes |
|--------|------|-------|
| `StudentAccountId` | `int` PK IDENTITY | |
| `SisStudentId` | `nvarchar(50)` UNIQUE NOT NULL | Foreign key to SIS |
| `StudentNumber` | `nvarchar(20)` UNIQUE NOT NULL | |
| `FullName` | `nvarchar(200)` NOT NULL | Snapshot from SIS |
| `ProgrammeId` | `int` FK → Programmes | |
| `YearLevel` | `tinyint` NOT NULL | |
| `StudentType` | `nvarchar(30)` NOT NULL | Regular, Irregular, Transferee, Graduate |
| `Email` | `nvarchar(200)` NULL | |
| `IsActive` | `bit` NOT NULL DEFAULT 1 | |
| `HasHold` | `bit` NOT NULL DEFAULT 0 | Enrolment hold flag |

#### `FeeTypes`
| Column | Type | Notes |
|--------|------|-------|
| `FeeTypeId` | `int` PK IDENTITY | |
| `Code` | `nvarchar(20)` UNIQUE NOT NULL | |
| `Name` | `nvarchar(200)` NOT NULL | |
| `Category` | `nvarchar(50)` NOT NULL | Tuition, Miscellaneous, Laboratory, Other |
| `AccountId` | `int` FK → ChartOfAccounts | GL revenue account |
| `IsActive` | `bit` NOT NULL DEFAULT 1 | |

#### `FeeMatrices`
| Column | Type | Notes |
|--------|------|-------|
| `FeeMatrixId` | `int` PK IDENTITY | |
| `SemesterId` | `int` FK → Semesters | |
| `ProgrammeId` | `int` NULL FK → Programmes | NULL = applies to all |
| `YearLevel` | `tinyint` NULL | NULL = applies to all year levels |
| `StudentType` | `nvarchar(30)` NULL | NULL = applies to all types |
| `FeeTypeId` | `int` FK → FeeTypes | |
| `Amount` | `decimal(18,2)` NOT NULL | |
| `IsPerUnit` | `bit` NOT NULL DEFAULT 0 | If true, Amount = per-unit rate |

#### `StudentBillings`
| Column | Type | Notes |
|--------|------|-------|
| `StudentBillingId` | `int` PK IDENTITY | |
| `StudentAccountId` | `int` FK → StudentAccounts | |
| `SemesterId` | `int` FK → Semesters | |
| `UnitsEnrolled` | `decimal(5,2)` NOT NULL DEFAULT 0 | |
| `TotalAssessed` | `decimal(18,2)` NOT NULL | |
| `TotalDiscount` | `decimal(18,2)` NOT NULL DEFAULT 0 | |
| `TotalPaid` | `decimal(18,2)` NOT NULL DEFAULT 0 | |
| `Balance` | `decimal(18,2)` NOT NULL COMPUTED | `TotalAssessed - TotalDiscount - TotalPaid` |
| `Status` | `nvarchar(30)` NOT NULL | Open, FullyPaid, PartiallyPaid, OverPaid |
| `DueDateFirst` | `date` NULL | First instalment due date |
| `DueDateFinal` | `date` NULL | Final due date |

#### `StudentBillingLines`
| Column | Type | Notes |
|--------|------|-------|
| `BillingLineId` | `int` PK IDENTITY | |
| `StudentBillingId` | `int` FK → StudentBillings | |
| `FeeTypeId` | `int` FK → FeeTypes | |
| `Description` | `nvarchar(200)` NOT NULL | |
| `Quantity` | `decimal(10,2)` NOT NULL DEFAULT 1 | Units for per-unit fees |
| `UnitAmount` | `decimal(18,2)` NOT NULL | |
| `Amount` | `decimal(18,2)` NOT NULL | |
| `IsWaived` | `bit` NOT NULL DEFAULT 0 | |
| `WaivedBy` | `nvarchar(100)` NULL | |
| `WaivedReason` | `nvarchar(500)` NULL | |

---

### Payment

#### `PaymentTransactions`
| Column | Type | Notes |
|--------|------|-------|
| `PaymentTransactionId` | `int` PK IDENTITY | |
| `StudentBillingId` | `int` FK → StudentBillings | |
| `StudentAccountId` | `int` FK → StudentAccounts | |
| `PaymentDate` | `datetime2` NOT NULL | |
| `PaymentMode` | `nvarchar(30)` NOT NULL | Cash, Cheque, BankTransfer, Card, Online |
| `AmountTendered` | `decimal(18,2)` NOT NULL | |
| `AmountApplied` | `decimal(18,2)` NOT NULL | |
| `Change` | `decimal(18,2)` NOT NULL DEFAULT 0 | For cash payments |
| `ReferenceNumber` | `nvarchar(100)` NULL | Cheque/transfer reference |
| `BankName` | `nvarchar(100)` NULL | |
| `Status` | `nvarchar(30)` NOT NULL | Posted, Voided, Pending (online) |
| `CashierId` | `nvarchar(100)` NOT NULL | FK → AspNetUsers.UserName |
| `CashierSessionId` | `int` FK → CashierSessions | |
| `JournalEntryId` | `int` NULL FK → JournalEntries | GL posting reference |
| `VoidedAt` | `datetime2` NULL | |
| `VoidedBy` | `nvarchar(100)` NULL | |
| `VoidReason` | `nvarchar(500)` NULL | |

#### `OfficialReceipts`
| Column | Type | Notes |
|--------|------|-------|
| `OfficialReceiptId` | `int` PK IDENTITY | |
| `PaymentTransactionId` | `int` FK → PaymentTransactions | |
| `ReceiptNumber` | `nvarchar(30)` UNIQUE NOT NULL | Sequential OR number |
| `IssuedAt` | `datetime2` NOT NULL | |
| `IsVoided` | `bit` NOT NULL DEFAULT 0 | |
| `PrintCount` | `int` NOT NULL DEFAULT 0 | Tracks reprints |

#### `CashierSessions`
| Column | Type | Notes |
|--------|------|-------|
| `CashierSessionId` | `int` PK IDENTITY | |
| `CashierId` | `nvarchar(100)` NOT NULL | |
| `OpenedAt` | `datetime2` NOT NULL | |
| `ClosedAt` | `datetime2` NULL | |
| `OpeningCash` | `decimal(18,2)` NOT NULL DEFAULT 0 | |
| `ClosingCash` | `decimal(18,2)` NULL | |
| `TotalCollected` | `decimal(18,2)` NULL | Computed on close |
| `Status` | `nvarchar(20)` NOT NULL | Open, Closed |

---

### Scholarships

#### `ScholarshipTypes`
| Column | Type | Notes |
|--------|------|-------|
| `ScholarshipTypeId` | `int` PK IDENTITY | |
| `Code` | `nvarchar(30)` UNIQUE NOT NULL | |
| `Name` | `nvarchar(200)` NOT NULL | |
| `DiscountType` | `nvarchar(20)` NOT NULL | Percentage, FixedAmount |
| `DiscountValue` | `decimal(18,2)` NOT NULL | % (0–100) or fixed amount |
| `AppliesToTuition` | `bit` NOT NULL DEFAULT 1 | |
| `AppliesToMisc` | `bit` NOT NULL DEFAULT 0 | |
| `AccountId` | `int` FK → ChartOfAccounts | Contra-revenue account |
| `IsActive` | `bit` NOT NULL DEFAULT 1 | |

#### `ScholarshipAwards`
| Column | Type | Notes |
|--------|------|-------|
| `ScholarshipAwardId` | `int` PK IDENTITY | |
| `StudentAccountId` | `int` FK → StudentAccounts | |
| `ScholarshipTypeId` | `int` FK → ScholarshipTypes | |
| `AwardedSemesterId` | `int` FK → Semesters | Start semester |
| `ExpiresOnSemesterId` | `int` NULL FK → Semesters | End semester (NULL = until revoked) |
| `ApprovedBy` | `nvarchar(100)` NOT NULL | |
| `ApprovalDate` | `datetime2` NOT NULL | |
| `Status` | `nvarchar(20)` NOT NULL | Active, Expired, Revoked |
| `Remarks` | `nvarchar(1000)` NULL | |

---

### General Ledger

#### `JournalEntries`
| Column | Type | Notes |
|--------|------|-------|
| `JournalEntryId` | `int` PK IDENTITY | |
| `EntryNumber` | `nvarchar(30)` UNIQUE NOT NULL | Auto-sequential JE number |
| `TransactionDate` | `date` NOT NULL | |
| `Description` | `nvarchar(500)` NOT NULL | |
| `SourceModule` | `nvarchar(50)` NOT NULL | Payment, AP, Payroll, Manual, etc. |
| `SourceReferenceId` | `int` NULL | Source record ID |
| `SourceReferenceNumber` | `nvarchar(100)` NULL | OR number, DV number, etc. |
| `Status` | `nvarchar(20)` NOT NULL | Draft, ForReview, Posted, Rejected |
| `PostedAt` | `datetime2` NULL | |
| `PostedBy` | `nvarchar(100)` NULL | |
| `SemesterId` | `int` FK → Semesters | Period classification |
| `TotalDebit` | `decimal(18,2)` NOT NULL | Must equal TotalCredit |
| `TotalCredit` | `decimal(18,2)` NOT NULL | |

#### `JournalEntryLines`
| Column | Type | Notes |
|--------|------|-------|
| `JournalEntryLineId` | `int` PK IDENTITY | |
| `JournalEntryId` | `int` FK → JournalEntries | |
| `AccountId` | `int` FK → ChartOfAccounts | |
| `DepartmentId` | `int` NULL FK → Departments | Cost centre |
| `Debit` | `decimal(18,2)` NOT NULL DEFAULT 0 | |
| `Credit` | `decimal(18,2)` NOT NULL DEFAULT 0 | |
| `Remarks` | `nvarchar(300)` NULL | |

---

### Accounts Payable

#### `Vendors`
| Column | Type | Notes |
|--------|------|-------|
| `VendorId` | `int` PK IDENTITY | |
| `VendorCode` | `nvarchar(20)` UNIQUE NOT NULL | |
| `Name` | `nvarchar(200)` NOT NULL | |
| `TIN` | `nvarchar(20)` NULL | |
| `Category` | `nvarchar(100)` NOT NULL | |
| `Address` | `nvarchar(500)` NULL | |
| `ContactPerson` | `nvarchar(200)` NULL | |
| `ContactEmail` | `nvarchar(200)` NULL | |
| `BankName` | `nvarchar(100)` NULL | |
| `BankAccountNumber` | `nvarchar(50)` NULL | |
| `IsAccredited` | `bit` NOT NULL DEFAULT 0 | |
| `IsBlacklisted` | `bit` NOT NULL DEFAULT 0 | |
| `BlacklistReason` | `nvarchar(500)` NULL | |
| `IsActive` | `bit` NOT NULL DEFAULT 1 | |

#### `PurchaseOrders`
| Column | Type | Notes |
|--------|------|-------|
| `PurchaseOrderId` | `int` PK IDENTITY | |
| `PoNumber` | `nvarchar(30)` UNIQUE NOT NULL | |
| `VendorId` | `int` FK → Vendors | |
| `DepartmentId` | `int` FK → Departments | Requesting department |
| `PoDate` | `date` NOT NULL | |
| `DeliveryDate` | `date` NULL | Expected delivery |
| `TotalAmount` | `decimal(18,2)` NOT NULL | |
| `Status` | `nvarchar(30)` NOT NULL | Draft, Approved, PartiallyReceived, Received, Cancelled |
| `ApprovedBy` | `nvarchar(100)` NULL | |
| `ApprovedAt` | `datetime2` NULL | |
| `Remarks` | `nvarchar(1000)` NULL | |

#### `PurchaseOrderLines`
| Column | Type | Notes |
|--------|------|-------|
| `PoLineId` | `int` PK IDENTITY | |
| `PurchaseOrderId` | `int` FK → PurchaseOrders | |
| `ItemDescription` | `nvarchar(300)` NOT NULL | |
| `Quantity` | `decimal(10,2)` NOT NULL | |
| `UnitOfMeasure` | `nvarchar(30)` NULL | |
| `UnitPrice` | `decimal(18,2)` NOT NULL | |
| `TotalPrice` | `decimal(18,2)` NOT NULL | |
| `AccountId` | `int` FK → ChartOfAccounts | Expense account |
| `ReceivedQuantity` | `decimal(10,2)` NOT NULL DEFAULT 0 | |

#### `DisbursementVouchers`
| Column | Type | Notes |
|--------|------|-------|
| `DisbursementVoucherId` | `int` PK IDENTITY | |
| `DvNumber` | `nvarchar(30)` UNIQUE NOT NULL | |
| `VendorId` | `int` FK → Vendors | |
| `PurchaseOrderId` | `int` NULL FK → PurchaseOrders | |
| `VendorInvoiceNumber` | `nvarchar(100)` NULL | |
| `InvoiceDate` | `date` NULL | |
| `DvDate` | `date` NOT NULL | |
| `GrossAmount` | `decimal(18,2)` NOT NULL | |
| `TaxWithheld` | `decimal(18,2)` NOT NULL DEFAULT 0 | EWT amount |
| `NetAmount` | `decimal(18,2)` NOT NULL | GrossAmount − TaxWithheld |
| `Status` | `nvarchar(30)` NOT NULL | Draft, ForApproval, Approved, Paid, Cancelled |
| `PaymentMode` | `nvarchar(30)` NULL | Cheque, BankTransfer, PettyCash |
| `ChequeNumber` | `nvarchar(50)` NULL | |
| `PaymentDate` | `date` NULL | |
| `JournalEntryId` | `int` NULL FK → JournalEntries | |

---

### Payroll

#### `Employees`
| Column | Type | Notes |
|--------|------|-------|
| `EmployeeId` | `int` PK IDENTITY | |
| `EmployeeNumber` | `nvarchar(20)` UNIQUE NOT NULL | |
| `FullName` | `nvarchar(200)` NOT NULL | |
| `DepartmentId` | `int` FK → Departments | |
| `Designation` | `nvarchar(100)` NOT NULL | |
| `EmploymentType` | `nvarchar(30)` NOT NULL | Regular, Contractual, PartTime |
| `BasicSalary` | `decimal(18,2)` NOT NULL | |
| `PayFrequency` | `nvarchar(20)` NOT NULL | SemiMonthly, Monthly |
| `TIN` | `nvarchar(20)` NULL | |
| `TaxStatus` | `nvarchar(10)` NOT NULL | S, S1, ME, ME1, etc. |
| `SssNumber` | `nvarchar(20)` NULL | |
| `PhilhealthNumber` | `nvarchar(20)` NULL | |
| `PagibigNumber` | `nvarchar(20)` NULL | |
| `BankName` | `nvarchar(100)` NULL | |
| `BankAccountNumber` | `nvarchar(50)` NULL | |
| `HireDate` | `date` NOT NULL | |
| `SeparationDate` | `date` NULL | |
| `IsActive` | `bit` NOT NULL DEFAULT 1 | |
| `UserId` | `nvarchar(450)` NULL FK → AspNetUsers | Optional portal access |

#### `PayrollRuns`
| Column | Type | Notes |
|--------|------|-------|
| `PayrollRunId` | `int` PK IDENTITY | |
| `RunNumber` | `nvarchar(30)` UNIQUE NOT NULL | |
| `PayrollGroup` | `nvarchar(50)` NOT NULL | Admin, Faculty, Casual |
| `PeriodFrom` | `date` NOT NULL | |
| `PeriodTo` | `date` NOT NULL | |
| `PayDate` | `date` NOT NULL | |
| `TotalGross` | `decimal(18,2)` NOT NULL | |
| `TotalDeductions` | `decimal(18,2)` NOT NULL | |
| `TotalNet` | `decimal(18,2)` NOT NULL | |
| `Status` | `nvarchar(20)` NOT NULL | Draft, Computed, Approved, Posted, Paid |
| `ApprovedBy` | `nvarchar(100)` NULL | |
| `JournalEntryId` | `int` NULL FK → JournalEntries | |

#### `PayrollRunDetails`
| Column | Type | Notes |
|--------|------|-------|
| `PayrollRunDetailId` | `int` PK IDENTITY | |
| `PayrollRunId` | `int` FK → PayrollRuns | |
| `EmployeeId` | `int` FK → Employees | |
| `BasicPay` | `decimal(18,2)` NOT NULL | |
| `Overtime` | `decimal(18,2)` NOT NULL DEFAULT 0 | |
| `Allowances` | `decimal(18,2)` NOT NULL DEFAULT 0 | |
| `GrossPay` | `decimal(18,2)` NOT NULL | |
| `SssContribution` | `decimal(18,2)` NOT NULL DEFAULT 0 | |
| `PhilhealthContribution` | `decimal(18,2)` NOT NULL DEFAULT 0 | |
| `PagibigContribution` | `decimal(18,2)` NOT NULL DEFAULT 0 | |
| `WithholdingTax` | `decimal(18,2)` NOT NULL DEFAULT 0 | |
| `OtherDeductions` | `decimal(18,2)` NOT NULL DEFAULT 0 | Loans, advances |
| `TotalDeductions` | `decimal(18,2)` NOT NULL | |
| `NetPay` | `decimal(18,2)` NOT NULL | |

---

### Budget

#### `Budgets`
| Column | Type | Notes |
|--------|------|-------|
| `BudgetId` | `int` PK IDENTITY | |
| `AcademicYearId` | `int` FK → AcademicYears | |
| `BudgetType` | `nvarchar(30)` NOT NULL | Operating, CapitalOutlay, SpecialFund |
| `Title` | `nvarchar(200)` NOT NULL | |
| `TotalAmount` | `decimal(18,2)` NOT NULL | |
| `Status` | `nvarchar(20)` NOT NULL | Draft, Submitted, Approved, Locked |
| `ApprovedBy` | `nvarchar(100)` NULL | |
| `ApprovedAt` | `datetime2` NULL | |

#### `BudgetLines`
| Column | Type | Notes |
|--------|------|-------|
| `BudgetLineId` | `int` PK IDENTITY | |
| `BudgetId` | `int` FK → Budgets | |
| `DepartmentId` | `int` FK → Departments | |
| `AccountId` | `int` FK → ChartOfAccounts | |
| `Description` | `nvarchar(300)` NOT NULL | |
| `ApprovedAmount` | `decimal(18,2)` NOT NULL | |
| `ObligatedAmount` | `decimal(18,2)` NOT NULL DEFAULT 0 | Committed via approved POs |
| `ActualAmount` | `decimal(18,2)` NOT NULL DEFAULT 0 | Actual expenditure posted |
| `BalanceAmount` | `decimal(18,2)` NOT NULL COMPUTED | `ApprovedAmount - ObligatedAmount - ActualAmount` |

---

### Security & Audit

#### `AuditLogs`
| Column | Type | Notes |
|--------|------|-------|
| `AuditLogId` | `bigint` PK IDENTITY | |
| `Timestamp` | `datetime2` NOT NULL | |
| `UserId` | `nvarchar(450)` NOT NULL | |
| `UserName` | `nvarchar(100)` NOT NULL | |
| `Action` | `nvarchar(50)` NOT NULL | Create, Update, Delete, Post, Void, Approve, Reject, Login, Logout |
| `EntityName` | `nvarchar(100)` NOT NULL | Table/entity affected |
| `EntityId` | `nvarchar(100)` NULL | Primary key value |
| `OldValues` | `nvarchar(max)` NULL | JSON snapshot before change |
| `NewValues` | `nvarchar(max)` NULL | JSON snapshot after change |
| `IpAddress` | `nvarchar(50)` NULL | |
| `UserAgent` | `nvarchar(500)` NULL | |

#### `ApprovalRequests`
| Column | Type | Notes |
|--------|------|-------|
| `ApprovalRequestId` | `int` PK IDENTITY | |
| `EntityType` | `nvarchar(50)` NOT NULL | JournalEntry, DisbursementVoucher, Refund, etc. |
| `EntityId` | `int` NOT NULL | |
| `Level` | `tinyint` NOT NULL | Approval level (1, 2, 3) |
| `Status` | `nvarchar(20)` NOT NULL | Pending, Approved, Rejected |
| `AssignedTo` | `nvarchar(100)` NOT NULL | Approver UserId |
| `ActedAt` | `datetime2` NULL | |
| `ActionBy` | `nvarchar(100)` NULL | |
| `Remarks` | `nvarchar(1000)` NULL | |

---

## 4.3 Key SQL Server Objects

### Stored Procedures
| Procedure | Purpose |
|-----------|---------|
| `usp_GetStudentStatementOfAccount` | Returns full SOA for a student for a given semester |
| `usp_GetArAgingReport` | Returns AR ageing buckets as of a given date |
| `usp_GetApAgingReport` | Returns AP ageing buckets as of a given date |
| `usp_GetTrialBalance` | Returns trial balance for a period |
| `usp_GetBudgetVsActual` | Returns budget utilisation per department |
| `usp_PostPayroll` | Atomic payroll posting: updates `PayrollRuns` and inserts `JournalEntry` |

### Views
| View | Purpose |
|------|---------|
| `vw_StudentBalances` | Current balance per student per semester |
| `vw_GlAccountBalances` | Current GL account balances (debit/credit net) |
| `vw_DailyCollection` | Daily collection summary per cashier |
| `vw_OutstandingPayables` | AP invoices not yet fully paid |

### Indexes
```sql
-- Frequently filtered columns
CREATE INDEX IX_PaymentTransactions_StudentAccountId_PaymentDate
    ON PaymentTransactions (StudentAccountId, PaymentDate);

CREATE INDEX IX_JournalEntryLines_AccountId
    ON JournalEntryLines (AccountId);

CREATE INDEX IX_StudentBillings_SemesterId_Status
    ON StudentBillings (SemesterId, Status);

CREATE INDEX IX_AuditLogs_Timestamp
    ON AuditLogs (Timestamp DESC);

CREATE INDEX IX_PayrollRunDetails_PayrollRunId
    ON PayrollRunDetails (PayrollRunId);
```

---

## 4.4 Database Naming Conventions

| Object | Convention | Example |
|--------|-----------|---------|
| Tables | PascalCase, plural | `StudentAccounts` |
| Columns | PascalCase | `StudentAccountId` |
| Primary Keys | `[Table singular]Id` | `StudentAccountId` |
| Foreign Keys | `[Referenced Table singular]Id` | `SemesterId` |
| Stored Procedures | `usp_` prefix | `usp_GetTrialBalance` |
| Views | `vw_` prefix | `vw_GlAccountBalances` |
| Indexes | `IX_[Table]_[Columns]` | `IX_Payments_StudentId` |
| Triggers | `trg_[Table]_[Action]` | `trg_JournalEntries_ValidateBalance` |
| Constraints (check) | `CK_[Table]_[Rule]` | `CK_JournalEntryLines_DebitOrCredit` |
