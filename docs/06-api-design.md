# 06 — API Design

This document covers both the **MVC Controller action contracts** (server-rendered pages) and the **internal JSON API endpoints** used by AJAX calls from Razor Views (DataTables, partial refreshes, and form submissions).

All AJAX API endpoints are prefixed with `/api/` and return JSON. They are consumed only by the same-origin Razor frontend — not intended as a public API.

---

## 6.1 General Conventions

### Response Envelope
All `/api/` endpoints return a consistent envelope:

```json
{
  "success": true,
  "data": { ... },
  "message": null,
  "errors": []
}
```

On failure:
```json
{
  "success": false,
  "data": null,
  "message": "A descriptive error message",
  "errors": ["Validation error 1", "Validation error 2"]
}
```

### HTTP Status Codes
| Status | Usage |
|--------|-------|
| `200 OK` | Successful GET or action |
| `201 Created` | Successful resource creation |
| `400 Bad Request` | Validation failure |
| `401 Unauthorized` | Not authenticated |
| `403 Forbidden` | Authenticated but insufficient permission |
| `404 Not Found` | Resource not found |
| `409 Conflict` | Business rule conflict (e.g., duplicate OR number, closed period) |
| `500 Internal Server Error` | Unhandled server exception |

### Pagination (for list endpoints)
```json
{
  "success": true,
  "data": {
    "items": [...],
    "totalCount": 500,
    "page": 1,
    "pageSize": 25,
    "totalPages": 20
  }
}
```

Query parameters: `?page=1&pageSize=25&search=&sortBy=&sortDesc=false`

---

## 6.2 Student Account Endpoints

### `GET /api/student-accounts`
Returns paginated list of student accounts.

**Query Parameters:** `page`, `pageSize`, `search` (name/student number), `programmeId`, `semesterId`

**Response `data`:**
```json
{
  "items": [
    {
      "studentAccountId": 1,
      "studentNumber": "2025-0001",
      "fullName": "Juan Dela Cruz",
      "programme": "BS Accountancy",
      "yearLevel": 2,
      "balance": 15000.00,
      "hasHold": false
    }
  ],
  "totalCount": 320,
  "page": 1,
  "pageSize": 25
}
```

---

### `GET /api/student-accounts/{id}/statement`
Returns the full Statement of Account for a student.

**Path Parameter:** `id` — StudentAccountId  
**Query Parameter:** `semesterId` (optional; defaults to current open semester)

**Response `data`:**
```json
{
  "studentNumber": "2025-0001",
  "fullName": "Juan Dela Cruz",
  "programme": "BS Accountancy",
  "yearLevel": 2,
  "semester": "1st Semester 2025-2026",
  "unitsEnrolled": 21,
  "totalAssessed": 42000.00,
  "totalDiscount": 5000.00,
  "totalPaid": 20000.00,
  "balance": 17000.00,
  "billingLines": [
    { "description": "Tuition Fee (21 units × ₱1,500)", "amount": 31500.00 },
    { "description": "Registration Fee", "amount": 500.00 },
    { "description": "Library Fee", "amount": 200.00 }
  ],
  "discounts": [
    { "description": "Dean's List Scholarship (50%)", "amount": -5000.00 }
  ],
  "payments": [
    {
      "orNumber": "OR-2025-00001",
      "date": "2025-08-01",
      "amount": 20000.00,
      "mode": "Cash"
    }
  ]
}
```

---

### `POST /api/student-accounts/{id}/billing-adjustment`
Applies a manual billing adjustment (requires `BillingClerk` role + approval).

**Request Body:**
```json
{
  "semesterId": 5,
  "feeTypeId": 3,
  "adjustmentAmount": -500.00,
  "reason": "Incorrect lab fee applied — not enrolled in Biology Lab",
  "supportingDocumentPath": "uploads/adj-2025-001.pdf"
}
```

**Response:** Standard envelope with updated billing summary.

---

## 6.3 Payment Endpoints

### `POST /api/payments`
Posts a new payment (Cashier role required).

**Request Body:**
```json
{
  "studentAccountId": 1,
  "semesterId": 5,
  "paymentMode": "Cash",
  "amountTendered": 20000.00,
  "referenceNumber": null,
  "bankName": null,
  "remarks": "Down payment — 1st instalment"
}
```

**Response `data`:**
```json
{
  "paymentTransactionId": 101,
  "orNumber": "OR-2025-00101",
  "amountApplied": 20000.00,
  "change": 0.00,
  "newBalance": 22000.00,
  "receiptPdfUrl": "/reports/receipts/OR-2025-00101.pdf"
}
```

---

### `POST /api/payments/{id}/void`
Voids a posted payment receipt.

**Request Body:**
```json
{
  "reason": "Wrong student account — cashier error"
}
```

**Response:** Standard envelope. Returns 403 if caller lacks `CanVoidReceipt` claim.

---

### `GET /api/payments/daily-summary`
Returns the daily collection summary for a cashier session.

**Query Parameters:** `date` (ISO date), `cashierId` (optional; defaults to current user)

**Response `data`:**
```json
{
  "date": "2025-08-15",
  "cashierName": "Maria Santos",
  "sessions": [
    {
      "sessionId": 12,
      "openedAt": "2025-08-15T08:00:00",
      "closedAt": "2025-08-15T17:00:00",
      "cash": 45000.00,
      "cheque": 10000.00,
      "bankTransfer": 20000.00,
      "card": 5000.00,
      "totalCollected": 80000.00,
      "transactionCount": 32
    }
  ]
}
```

---

## 6.4 Fee Structure Endpoints

### `GET /api/fee-matrices`
Returns fee matrix entries for a given semester and optional programme.

**Query Parameters:** `semesterId` (required), `programmeId`, `yearLevel`, `studentType`

### `POST /api/fee-matrices/assess`
Runs fee assessment for a student and returns the computed billing lines without saving.

**Request Body:**
```json
{
  "studentAccountId": 1,
  "semesterId": 5,
  "unitsEnrolled": 21
}
```

**Response `data`:** Array of `BillingLinePreviewDto` with computed amounts.

---

## 6.5 Scholarship Endpoints

### `GET /api/scholarships`
List all scholarship awards (paginated). Filter by: `semesterId`, `scholarshipTypeId`, `status`.

### `POST /api/scholarships`
Award a scholarship to a student.

**Request Body:**
```json
{
  "studentAccountId": 1,
  "scholarshipTypeId": 3,
  "awardedSemesterId": 5,
  "expiresOnSemesterId": 8,
  "remarks": "Governor's scholarship — 2025 awardee"
}
```

### `PUT /api/scholarships/{id}/revoke`
Revoke an active scholarship.

**Request Body:**
```json
{ "reason": "Failed to meet minimum GWA of 1.75" }
```

---

## 6.6 General Ledger Endpoints

### `GET /api/gl/trial-balance`
Returns the trial balance for a period.

**Query Parameters:** `semesterId` (required), `type` = `unadjusted | adjusted | post-closing`

**Response `data`:**
```json
{
  "semesterName": "1st Semester 2025-2026",
  "generatedAt": "2025-12-01T10:00:00",
  "accounts": [
    {
      "accountCode": "1010-001",
      "accountName": "Cash on Hand",
      "debit": 250000.00,
      "credit": 0.00
    }
  ],
  "totalDebit": 5000000.00,
  "totalCredit": 5000000.00,
  "isBalanced": true
}
```

---

### `POST /api/gl/journal-entries`
Creates a manual journal entry (Draft status).

**Request Body:**
```json
{
  "transactionDate": "2025-09-01",
  "description": "Accrual of electricity expense — August 2025",
  "semesterId": 5,
  "lines": [
    { "accountId": 510, "debit": 15000.00, "credit": 0.00, "departmentId": 2 },
    { "accountId": 201, "debit": 0.00, "credit": 15000.00, "departmentId": null }
  ]
}
```

**Validation:** `sum(debit)` must equal `sum(credit)`; all accounts must be postable and active; period must not be hard-closed.

---

### `POST /api/gl/journal-entries/{id}/submit`
Submits a Draft JE for review.

### `POST /api/gl/journal-entries/{id}/approve`
Posts a For-Review JE to the ledger (requires `AccountingManager` or `VPFinance` role).

### `POST /api/gl/journal-entries/{id}/reject`
Rejects a For-Review JE. **Request Body:** `{ "reason": "..." }`

---

## 6.7 Accounts Payable Endpoints

### `POST /api/ap/purchase-orders`
Creates a new Purchase Order.

### `PUT /api/ap/purchase-orders/{id}/approve`
Approves a PO (moves to Approved status).

### `POST /api/ap/disbursement-vouchers`
Creates a Disbursement Voucher from a vendor invoice.

**Request Body:**
```json
{
  "vendorId": 12,
  "purchaseOrderId": 45,
  "vendorInvoiceNumber": "INV-2025-00789",
  "invoiceDate": "2025-08-30",
  "grossAmount": 75000.00,
  "taxWithheld": 7500.00,
  "lines": [
    { "description": "Office Supplies", "amount": 75000.00, "accountId": 512 }
  ]
}
```

### `PUT /api/ap/disbursement-vouchers/{id}/approve`
Approves the DV (level-based; checks threshold).

### `PUT /api/ap/disbursement-vouchers/{id}/pay`
Records payment disbursement.

**Request Body:**
```json
{
  "paymentMode": "Cheque",
  "chequeNumber": "CHQ-0012345",
  "bankName": "Land Bank",
  "paymentDate": "2025-09-05"
}
```

---

## 6.8 Payroll Endpoints

### `POST /api/payroll/runs`
Creates a new payroll run.

**Request Body:**
```json
{
  "payrollGroup": "Admin",
  "periodFrom": "2025-08-16",
  "periodTo": "2025-08-31",
  "payDate": "2025-09-05"
}
```

### `POST /api/payroll/runs/{id}/compute`
Triggers batch payroll computation (runs as a background Hangfire job).

**Response `data`:**
```json
{ "jobId": "hangfire-job-abc123", "statusUrl": "/api/payroll/runs/42/status" }
```

### `GET /api/payroll/runs/{id}/status`
Returns computation status for polling.

### `POST /api/payroll/runs/{id}/approve`
Approves the computed payroll run.

### `POST /api/payroll/runs/{id}/post`
Posts approved payroll to GL.

---

## 6.9 Budget Endpoints

### `GET /api/budgets/{id}/utilisation`
Returns real-time budget utilisation for a budget.

**Response `data`:**
```json
{
  "budgetTitle": "FY 2025-2026 Operating Budget",
  "totalApproved": 10000000.00,
  "totalObligated": 3500000.00,
  "totalActual": 2800000.00,
  "totalBalance": 3700000.00,
  "utilisationPercent": 63.0,
  "lines": [
    {
      "department": "Finance Office",
      "account": "5010 — Salaries",
      "approved": 3000000.00,
      "obligated": 1500000.00,
      "actual": 1500000.00,
      "balance": 0.00
    }
  ]
}
```

---

## 6.10 Reporting Endpoints

### `GET /api/reports/financial-statements`
**Query Parameters:** `type` = `balance-sheet | income-statement | cash-flow`, `asOfDate`, `semesterId`

**Response:** Redirect to PDF stream or JSON data structure (depending on `format` param: `json | pdf | excel`).

### `GET /api/reports/ar-ageing`
**Query Parameters:** `asOfDate`

### `GET /api/reports/ap-ageing`
**Query Parameters:** `asOfDate`

### `GET /api/reports/collection`
**Query Parameters:** `fromDate`, `toDate`, `cashierId`

### `POST /api/reports/custom`
Executes a saved custom report template.

**Request Body:**
```json
{
  "templateId": 5,
  "parameters": {
    "fromDate": "2025-06-01",
    "toDate": "2025-12-31",
    "departmentId": 3
  },
  "format": "excel"
}
```

---

## 6.11 MVC Controller Action Summary

| Controller | Action | HTTP | Route | Description |
|------------|--------|------|-------|-------------|
| `AccountController` | `Login` | GET/POST | `/account/login` | Login page |
| `AccountController` | `Logout` | POST | `/account/logout` | Logout |
| `DashboardController` | `Index` | GET | `/` | Main dashboard |
| `StudentAccountController` | `Index` | GET | `/student-accounts` | Student list |
| `StudentAccountController` | `Statement` | GET | `/student-accounts/{id}/statement` | SOA page |
| `PaymentController` | `Collect` | GET/POST | `/payments/collect` | Payment collection form |
| `PaymentController` | `Receipt` | GET | `/payments/receipt/{orNumber}` | Receipt view/print |
| `FeeStructureController` | `Index` | GET | `/fee-structure` | Fee matrix list |
| `FeeStructureController` | `Create` | GET/POST | `/fee-structure/create` | New fee entry |
| `ScholarshipController` | `Index` | GET | `/scholarships` | Scholarship list |
| `ScholarshipController` | `Award` | GET/POST | `/scholarships/award` | Award scholarship |
| `GLController` | `JournalEntries` | GET | `/gl/journal-entries` | JE list |
| `GLController` | `Create` | GET/POST | `/gl/journal-entries/create` | New JE form |
| `GLController` | `TrialBalance` | GET | `/gl/trial-balance` | Trial balance view |
| `APController` | `PurchaseOrders` | GET | `/ap/purchase-orders` | PO list |
| `APController` | `Vouchers` | GET | `/ap/vouchers` | DV list |
| `PayrollController` | `Index` | GET | `/payroll` | Payroll run list |
| `PayrollController` | `Compute` | POST | `/payroll/{id}/compute` | Trigger computation |
| `BudgetController` | `Index` | GET | `/budget` | Budget list |
| `BudgetController` | `Utilisation` | GET | `/budget/{id}/utilisation` | Utilisation dashboard |
| `ReportController` | `Index` | GET | `/reports` | Report catalogue |
| `ReportController` | `Generate` | POST | `/reports/generate` | Generate/download report |
| `AdminController` | `Users` | GET | `/admin/users` | User management |
| `AdminController` | `AuditLogs` | GET | `/admin/audit-logs` | Audit log viewer |
