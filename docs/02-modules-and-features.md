# 02 — Modules & Features

## Module Map

```
VCAS
├── Module 01 — Student Account & Billing
├── Module 02 — Fee Structure Management
├── Module 03 — Payment Processing
├── Module 04 — Financial Aid & Scholarships
├── Module 05 — Accounts Receivable (AR)
├── Module 06 — Accounts Payable (AP)
├── Module 07 — General Ledger (GL)
├── Module 08 — Payroll Management
├── Module 09 — Budget Management
└── Module 10 — Financial Reporting & Analytics
```

---

## Module 01 — Student Account & Billing Management

### Purpose
Manages the complete financial lifecycle of a student from enrolment to graduation/separation, ensuring accurate billing and a clear statement of account at all times.

### Features

#### 1.1 Student Account Setup
- Automatic account creation triggered by SIS enrolment event (webhook/API poll)
- Manual account creation for walk-in or special enrolees
- Unique **Student Account Number** assigned per student
- Link student account to SIS `StudentId` for data synchronisation
- Store student demographic snapshot (name, programme, year level, contact) for billing purposes

#### 1.2 Statement of Account (SOA)
- Real-time SOA showing:
  - All assessed fees (tuition, miscellaneous, laboratory)
  - Applied scholarships/discounts
  - Payments made (date, amount, OR number)
  - Running balance (amount due)
- Filter SOA by Academic Year and Semester
- Print/export SOA to PDF
- Email SOA directly to student

#### 1.3 Assessment & Billing Generation
- Automatic fee assessment upon enrolment confirmation
- Billing generated per semester based on:
  - Programme of study
  - Units enrolled
  - Year level
  - Student type (regular, irregular, transferee, graduate)
- Manual billing adjustment with reason and approver tracking
- Credit memo issuance for overpayments or billing corrections

#### 1.4 Enrolment Hold / Clearance
- System flags accounts with outstanding balances above configurable threshold
- Clearance check before enrolment confirmation (integration point with SIS)
- Override capability for authorised roles (e.g., Accounting Manager)
- Clearance certificate generation upon full payment

#### 1.5 Instalment Plan Management
- Configure instalment schedules per semester (e.g., 3-instalment plan)
- Assign students to an instalment plan
- Track due dates and auto-compute balance per instalment
- Late payment penalty computation (configurable percentage or fixed amount)

---

## Module 02 — Fee Structure Management

### Purpose
Provides a centralised, configurable repository of all fees charged by the college, organised by academic year, semester, programme, and fee type.

### Features

#### 2.1 Chart of Fees (Fee Catalogue)
- Create, edit, and deactivate fee types:
  - **Tuition Fee** — per unit rate or fixed per programme
  - **Miscellaneous Fees** — registration, library, guidance, NSTP, medical, ID, etc.
  - **Laboratory Fees** — per subject requiring lab use
  - **Other Fees** — student council, yearbook, sports development, etc.
- Assign fee to GL account in the Chart of Accounts
- Fee effective date range (applies to specific AY/Semester)

#### 2.2 Programme-Based Fee Matrix
- Define fee matrix per:
  - College/Department
  - Programme (e.g., BS Accountancy, BS Computer Science)
  - Year Level
  - Student classification (New, Old, Transferee, Foreign)
- Bulk import fee matrix via Excel template

#### 2.3 Subject/Unit-Based Tuition Computation
- Configure per-unit rate per programme
- System computes tuition = enrolled units × per-unit rate
- Support fixed tuition (flat rate regardless of units) for certain programmes

#### 2.4 Fee Overrides & Waivers
- Authorised users can apply a one-time fee waiver or discount to a student account
- Waivers require a reason code and approval
- Full audit trail of all fee adjustments

#### 2.5 Academic Year / Semester Setup
- Create and manage Academic Years (e.g., 2025–2026)
- Define semesters within each AY (1st, 2nd, Summer)
- Lock prior semesters to prevent unauthorised changes
- Copy fee structure from previous AY as starting point

---

## Module 03 — Payment Processing

### Purpose
Records, validates, and posts all payments received from students and other payers, issuing official receipts and updating the student ledger in real time.

### Features

#### 3.1 Payment Collection (Counter/Cashier)
- Search student by name, student number, or account number
- Display outstanding balance before accepting payment
- Accept payment by:
  - **Cash** — cashier enters exact amount tendered; system computes change
  - **Cheque** — cheque number, bank, and date recorded; cleared on bank confirmation
  - **Bank Transfer / Online Payment** — reference number and bank recorded
  - **Credit/Debit Card** — terminal reference number recorded
- Partial payments allowed (apply to oldest due item or free allocation)
- Split payment across multiple payment modes in a single transaction

#### 3.2 Official Receipt (OR) Generation
- Auto-generate sequential OR numbers (series configurable per cashier station)
- Receipt includes: OR number, date, student name, student ID, breakdown of payment applied, amount, cashier name, and institution seal
- Print receipt to thermal printer or A4
- Re-print receipt capability (with REPRINT watermark)
- Void receipt (with reason, requires supervisor approval; voids GL posting)

#### 3.3 Online Payment Portal (Student-Facing)
- Students log in and view SOA
- Initiate payment via:
  - GCash / Maya (via payment gateway adapter)
  - Bank transfer (manual upload of proof)
- Payment request queued for cashier verification before OR issuance
- Email notification upon confirmed payment

#### 3.4 Cheque Management
- Received cheque register
- Track cheque status: Received → Deposited → Cleared / Bounced
- On bounce: auto-reverse payment, add penalty charge to student account, notify cashier

#### 3.5 Daily Cash Collection Summary
- Per-cashier daily collection report (cash, cheque, card, online)
- Fund count / cash turnover sheet for end-of-day cash surrender
- Cashier session open/close control (prevents posting outside session hours)

#### 3.6 Refund Processing
- Refund request form with reason and supporting documents
- Multi-level approval workflow (Cashier → Accounting Manager → VP Finance for amounts above threshold)
- Refund disbursement via cheque or bank transfer
- GL reversal entries posted upon approved refund

---

## Module 04 — Financial Aid & Scholarships

### Purpose
Manages all forms of tuition discounts, grants, scholarships, and financial assistance, ensuring correct application to student accounts and accurate GL recognition.

### Features

#### 4.1 Scholarship / Grant Type Setup
- Define scholarship types:
  - Full scholarship (100% tuition waiver)
  - Partial scholarship (configurable % or fixed amount)
  - Government grant (e.g., UniFAST / TES / CHED K-12 Transition)
  - Institutional discount (employee dependant, dean's list, athlete)
  - External corporate scholarship
- Define applicable fee categories (tuition only vs. tuition + miscellaneous)
- Assign to GL revenue/contra-revenue account

#### 4.2 Scholarship Application & Awarding
- Student applies (or HR/SIS pushes) scholarship data
- Scholarship officer reviews and approves/rejects with notes
- Approved scholarship linked to student account for specified AY/Semesters
- Auto-apply scholarship amount to student billing upon assessment

#### 4.3 Scholarship Renewal & Expiry
- Configure renewal requirements (minimum GWA, no failing grade, etc.)
- Scholarship expiry date or maximum semesters
- Automated notification to scholarship officer when renewal is due
- Renewal history tracking

#### 4.4 Discount & Waiver Management
- One-time or recurring discounts outside of formal scholarship programmes
- Examples: early bird discount, sibling discount, loyalty award
- Each discount requires approval and links to a GL account
- Cannot exceed total assessed fees (system validation)

#### 4.5 Scholarship Reporting
- List of scholarship recipients per semester
- Scholarship cost summary (total institutional commitment)
- Government grant utilisation report

---

## Module 05 — Accounts Receivable (AR)

### Purpose
Tracks all money owed to the college beyond student billing — including third-party payers, government grants receivable, and rental income.

### Features

#### 5.1 AR Ledger
- Separate AR accounts for:
  - Students (linked to student billing module)
  - Government agencies (grant receivables)
  - External tenants / facility rentals
  - Alumni pledges and donations
- AR balance computed from posted invoices minus posted receipts

#### 5.2 Invoice Management
- Create AR invoices for non-student debtors
- Invoice numbering (auto-sequential)
- Invoice line items with GL account mapping
- Send invoice via email (PDF attachment)
- Track invoice status: Draft → Sent → Partially Paid → Paid → Cancelled

#### 5.3 Ageing Analysis
- AR ageing report grouped by:
  - Current (0–30 days)
  - 31–60 days
  - 61–90 days
  - Over 90 days
- Per-debtor ageing detail
- Export to Excel / PDF

#### 5.4 Collection Follow-up
- Auto-generate collection letters (configurable templates per ageing bucket)
- Log collection calls and correspondence
- Assign AR accounts to collector/follow-up officer
- Promise-to-pay recording and tracking

#### 5.5 Bad Debt Management
- Write-off request workflow (AR Clerk → Accounting Manager → VP Finance)
- Write-off posts contra entry to Allowance for Doubtful Accounts
- Recovered bad debts re-posted with appropriate GL entries

---

## Module 06 — Accounts Payable (AP)

### Purpose
Manages all outgoing payments to vendors, suppliers, and service providers — from purchase order through payment.

### Features

#### 6.1 Vendor / Supplier Management
- Vendor master list (name, TIN, address, bank details, contact)
- Vendor categories (office supplies, utilities, professional services, etc.)
- Vendor accreditation status and supporting documents
- Blacklist capability with reason tracking

#### 6.2 Purchase Request & Purchase Order
- Department-originated Purchase Request (PR)
- PR approval workflow (Department Head → Budget Officer → VP Admin)
- Approved PR converted to Purchase Order (PO)
- PO numbering (auto-sequential)
- PO sent to vendor via email (PDF)
- Partial delivery / multiple delivery tracking against a single PO

#### 6.3 Receiving & Invoice Matching
- Receiving Report (RR) created upon goods/service receipt
- 3-way matching: PO ↔ RR ↔ Vendor Invoice
- Discrepancy flagging (quantity, price, or item mismatch)
- Manual override with approval for matched discrepancies

#### 6.4 AP Voucher & Payment Processing
- Disbursement Voucher (DV) created from matched vendor invoice
- DV approval workflow (AP Clerk → Accounting Manager → VP Finance for amounts above threshold)
- Payment modes: Cheque, Bank Transfer, Petty Cash (small amounts)
- Cheque preparation (auto-fill payee, amount in words)
- Payment scheduling (due date calendar view)

#### 6.5 AP Ageing & Reports
- AP ageing report (same bucket structure as AR)
- Payment forecast report (upcoming obligations for the next 30/60/90 days)
- Vendor payment history

---

## Module 07 — General Ledger (GL)

### Purpose
The financial backbone of the system. Every financial event posts a balanced journal entry to the GL, enabling accurate trial balance, financial statement generation, and period-end closing.

### Features

#### 7.1 Chart of Accounts (COA)
- Hierarchical COA (Account Group → Account Type → Account → Sub-Account)
- Standard account types: Assets, Liabilities, Equity, Revenue, Expenses
- Account code structure: configurable prefix scheme (e.g., 1xxx Assets, 2xxx Liabilities)
- Active/inactive flag per account
- Restrict posting to summary (parent) accounts
- Import/export COA via Excel

#### 7.2 Journal Entry Management
- **System-generated JEs** — auto-posted from sub-modules (billing, payment, payroll, AP)
- **Manual JEs** — created by accounting staff for adjustments, accruals, corrections
- JE fields: date, reference number, description, debit account, credit account, amount, department, cost centre
- Multi-line JE support (compound entries)
- Attachment support for supporting documents
- JE status workflow: Draft → For Review → Posted / Rejected

#### 7.3 Subsidiary Ledger Integration
- GL account balances always reconcile to subsidiary ledgers:
  - Accounts Receivable GL ↔ AR subsidiary
  - Accounts Payable GL ↔ AP subsidiary
  - Student Accounts GL ↔ Student billing ledger
  - Payroll GL ↔ Payroll register
- Reconciliation report flags any discrepancies

#### 7.4 Trial Balance
- Real-time unadjusted trial balance
- Adjusted trial balance (after adjustment JEs)
- Post-closing trial balance
- Export to Excel / PDF

#### 7.5 Period-End Closing
- Soft close: prevent new postings for a period without permanently locking
- Hard close (Period Lock): permanently lock a period after board approval
- Closing entries auto-generated: transfer income/expense balances to Retained Surplus
- Re-open period capability (super admin only, with approval)

#### 7.6 Bank Reconciliation
- Import bank statements (CSV/Excel)
- System auto-matches GL cash entries to bank statement entries
- Flag unmatched items (outstanding cheques, deposits in transit, bank charges)
- Reconciliation report: GL balance vs. bank statement balance vs. adjusted balance
- Lock reconciliation upon sign-off

---

## Module 08 — Payroll Management

### Purpose
Computes, validates, and disburses employee salaries with full compliance to Philippine statutory deductions (SSS, PhilHealth, Pag-IBIG, BIR withholding tax).

### Features

#### 8.1 Employee Master Setup
- Employee record: name, employee ID, department, designation, employment type (regular, contractual, part-time)
- Salary details: basic salary, pay frequency (semi-monthly, monthly), effective date
- Tax setup: TIN, tax status (S, S1, S2, ME, ME1…), withholding tax method
- Bank account details for direct transfer
- Government ID numbers (SSS, PhilHealth, Pag-IBIG)

#### 8.2 Payroll Period Management
- Create payroll cut-off periods
- Lock periods after processing to prevent edits
- Support multiple payroll groups (admin staff, faculty, casual)

#### 8.3 Earnings & Deduction Setup
- Configurable earning types: basic pay, overtime, night differential, holiday pay, allowances (transportation, rice, clothing), 13th month
- Configurable deduction types: SSS, PhilHealth, Pag-IBIG, withholding tax, loans, salary advance, cooperative contributions
- Formula-based computation rules (system maintains compliance tables for government contributions)

#### 8.4 Attendance & Leave Integration
- Import attendance data (CSV from biometric system or manual entry)
- Leave balance deduction from payroll
- Late/undertime deductions
- Overtime hours approval before payroll inclusion

#### 8.5 Payroll Computation & Review
- Batch compute payroll for a cut-off period
- Itemised computation sheet for review before posting
- Variance report vs. previous period (flag anomalies > configurable %)
- Approver reviews and approves/sends back for correction

#### 8.6 Payslip Generation
- Individual payslip: period, gross pay, itemised deductions, net pay
- Email payslip to employee (password-protected PDF with last 4 digits of employee ID)
- Print payslip batch

#### 8.7 Government Remittances
- Auto-generate SSS R3, PhilHealth RF-1, Pag-IBIG MCRF reports
- BIR Form 1601-C (monthly withholding tax) summary
- Alphalist data (BIR Form 1604-E/1604-CF)
- Remittance payment voucher linked to AP module

#### 8.8 Year-End Payroll
- 13th month pay computation (inclusive of all qualifying earnings)
- BIR Form 2316 (Certificate of Compensation Payment/Tax Withheld) per employee
- Annual income tax return data export (BIR format)

---

## Module 09 — Budget Management

### Purpose
Enables the college to plan, allocate, monitor, and control financial resources at the departmental and institutional level across academic years.

### Features

#### 9.1 Budget Preparation
- Annual budget creation per Academic Year
- Budget types: Operating, Capital Outlay, Special Fund
- Budget templates by department/cost centre
- Budget request submission by Department Heads
- Consolidation and review by Budget Officer
- Budget approval workflow (Budget Committee → VP Finance → Board Resolution)

#### 9.2 Budget Allocation & Distribution
- Approved budget distributed by:
  - Department / College
  - Cost Centre
  - Account Category (Personnel, MOOE, Capital Outlay)
- Monthly/quarterly distribution schedule
- Budget release mechanism (allotment release with approval)

#### 9.3 Budget vs. Actual Monitoring
- Real-time budget utilisation dashboard
- Budget balance = Approved Budget − Obligated − Actual Expenditure
- Drill-down from summary to individual transactions
- Alert when utilisation reaches configurable threshold (e.g., 80%, 90%)
- Over-budget warning; expenditures beyond budget require budget augmentation approval

#### 9.4 Budget Augmentation / Revision
- Request to increase or realign budget between line items
- Approval workflow with documented justification
- Revision history with before/after snapshots

#### 9.5 Budget Reports
- Budget Summary by Department
- Budget vs. Actual by Month
- Remaining Budget Utilisation Report
- Budget Obligation Report (committed but not yet paid)

---

## Module 10 — Financial Reporting & Analytics

### Purpose
Provides comprehensive financial reports for management decision-making, regulatory compliance, and audit purposes.

### Features

#### 10.1 Standard Financial Statements
| Report | Description |
|--------|-------------|
| Balance Sheet (Statement of Financial Position) | Assets, Liabilities, Equity at a point in time |
| Income Statement (Statement of Comprehensive Income) | Revenue and expenses for a period |
| Statement of Cash Flows | Operating, investing, financing activities |
| Statement of Changes in Equity | Movement in equity accounts |

All statements comply with the **Philippine Financial Reporting Standards (PFRS)** applicable to non-stock, non-profit educational institutions.

#### 10.2 Sub-Ledger Reports
- Student Billing Summary Report (per semester/per programme)
- Collection Report (per cashier, per day/month)
- AR Ageing Report
- AP Ageing Report
- Payroll Summary Report
- Scholarship Cost Report
- Bank Reconciliation Report

#### 10.3 Management Dashboard
- Key metrics widgets:
  - Total collection for the current period vs. target
  - Outstanding AR balance (with ageing breakdown)
  - Payables due in 30 days
  - Budget utilisation gauge per department
  - Cash position (per bank account)
- Interactive charts (bar, line, pie) using Chart.js
- Date range and department filters
- Role-sensitive: each role sees only relevant KPIs

#### 10.4 Drill-Down & Ad Hoc Reporting
- Drill from summary report to line-item detail
- Filter by: date range, department, account, student, vendor
- Export any report to:
  - PDF (for official printing)
  - Excel (for further analysis)
  - CSV (for system integration)

#### 10.5 Audit & Compliance Reports
- Transaction Audit Log (who posted what, when, from which IP)
- Voided Transactions Report
- Manual Adjustments Report (all manual JEs with approver)
- User Activity Log
- Period Closing History

#### 10.6 Custom Report Builder
- Drag-and-drop column selection from available data fields
- Group-by and subtotal options
- Save custom report templates for reuse
- Schedule reports for automatic email delivery (daily/weekly/monthly)
