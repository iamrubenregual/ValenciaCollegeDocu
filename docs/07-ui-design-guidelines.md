# 07 — UI Design Guidelines

## 7.1 Design Principles

| Principle | Description |
|-----------|-------------|
| **Clarity** | Financial data must be unambiguous. Use clear labels, consistent number formatting, and colour coding for positive/negative values. |
| **Efficiency** | Cashiers and clerks perform repetitive tasks. Keyboard navigation, auto-focus, and sensible tab order reduce friction. |
| **Hierarchy** | Critical actions (Post, Approve, Void) are visually distinct from secondary actions. Destructive actions require confirmation. |
| **Role Awareness** | The UI adapts to the logged-in user's role — irrelevant modules and actions are hidden, not merely disabled. |
| **Responsiveness** | Target minimum 1280×768 desktop resolution. Tablet support (1024px+) for read-only views; cashier station requires desktop. |

---

## 7.2 Layout Structure

### Master Layout (`_Layout.cshtml`)

```
┌─────────────────────────────────────────────────────────────┐
│  TOPBAR: Logo | Institution Name | User Menu | Notifications │
├──────────────┬──────────────────────────────────────────────┤
│              │  BREADCRUMB BAR                               │
│  SIDEBAR     ├──────────────────────────────────────────────┤
│  NAVIGATION  │                                               │
│              │  PAGE CONTENT AREA                            │
│  (collapsible│  @RenderBody()                                │
│   on < 1280) │                                               │
│              │                                               │
├──────────────┴──────────────────────────────────────────────┤
│  FOOTER: Version | © Valencia College | Support              │
└─────────────────────────────────────────────────────────────┘
```

### Sidebar Navigation Groups
```
📊  Dashboard
───────────────
👤  Student Accounts
    ├── Student List
    ├── Fee Structure
    └── Scholarships
───────────────
💳  Payments
    ├── Collect Payment
    ├── Daily Collection
    └── Cheque Registry
───────────────
📋  Accounts Receivable
    ├── AR Ledger
    ├── Invoices
    └── Ageing Report
───────────────
🧾  Accounts Payable
    ├── Vendors
    ├── Purchase Orders
    └── Disbursement Vouchers
───────────────
📒  General Ledger
    ├── Chart of Accounts
    ├── Journal Entries
    ├── Trial Balance
    └── Bank Reconciliation
───────────────
👔  Payroll
    ├── Employees
    ├── Payroll Runs
    └── Government Reports
───────────────
📦  Budget
    ├── Budget Plans
    └── Utilisation
───────────────
📈  Reports
───────────────
⚙️  Administration
    ├── Users & Roles
    ├── Audit Logs
    └── System Settings
```

---

## 7.3 Colour Palette

| Token | Hex | Usage |
|-------|-----|-------|
| `--color-primary` | `#1A3C6E` | Header, sidebar background, primary buttons |
| `--color-primary-light` | `#2A5BA8` | Active sidebar item, link hover |
| `--color-accent` | `#F5A623` | Notifications, highlight badges |
| `--color-success` | `#28A745` | Posted / Paid status, success alerts |
| `--color-warning` | `#FFC107` | Pending / For Review status, overdue warnings |
| `--color-danger` | `#DC3545` | Voided / Rejected status, balance due, destructive actions |
| `--color-info` | `#17A2B8` | Informational badges (Draft, Partial) |
| `--color-text` | `#212529` | Body text |
| `--color-muted` | `#6C757D` | Secondary/helper text |
| `--color-surface` | `#FFFFFF` | Card/panel backgrounds |
| `--color-bg` | `#F4F6F9` | Page background |

### Financial Value Colour Coding
- **Positive balance (amount owed):** `text-danger` (red) — the student owes money.
- **Zero balance:** `text-success` (green).
- **Over-payment (credit):** `text-info` (blue), shown as `(CR)` suffix.
- **Debit amounts in GL:** standard text.
- **Credit amounts in GL:** standard text.

---

## 7.4 Typography

| Element | Font | Size | Weight |
|---------|------|------|--------|
| Body | Inter / System UI | 14px | 400 |
| Headings H1 | Inter | 24px | 700 |
| Headings H2 | Inter | 20px | 600 |
| Headings H3 | Inter | 16px | 600 |
| Table data | Inter / monospace for amounts | 13px | 400 |
| Currency amounts | `font-variant-numeric: tabular-nums` | 13–14px | 500 |
| Labels | Inter | 13px | 500 |
| Badges | Inter | 11px | 600 |

---

## 7.5 Component Patterns

### Page Header
Every content page begins with a consistent header:

```html
<div class="page-header d-flex justify-content-between align-items-center mb-3">
    <div>
        <h1 class="h4 mb-0">Student Statement of Account</h1>
        <nav aria-label="breadcrumb">
            <ol class="breadcrumb small mb-0">
                <li class="breadcrumb-item"><a href="/">Dashboard</a></li>
                <li class="breadcrumb-item"><a href="/student-accounts">Student Accounts</a></li>
                <li class="breadcrumb-item active">Statement</li>
            </ol>
        </nav>
    </div>
    <div class="page-actions">
        <button class="btn btn-outline-secondary btn-sm"><i class="bi bi-printer"></i> Print</button>
        <button class="btn btn-primary btn-sm"><i class="bi bi-envelope"></i> Email SOA</button>
    </div>
</div>
```

### Status Badges
Consistent `<span class="badge ...">` usage:

| Status | Badge Class |
|--------|------------|
| Draft | `badge bg-secondary` |
| For Review / Pending | `badge bg-warning text-dark` |
| Posted / Active / Paid | `badge bg-success` |
| Approved | `badge bg-primary` |
| Rejected / Voided / Cancelled | `badge bg-danger` |
| Partial | `badge bg-info` |
| Expired | `badge bg-secondary` |

### Data Tables
All list pages use **DataTables.js** with server-side processing:

```html
<table id="studentTable" class="table table-hover table-bordered table-sm w-100">
    <thead class="table-dark">
        <tr>
            <th>Student No.</th>
            <th>Name</th>
            <th>Programme</th>
            <th class="text-end">Balance</th>
            <th>Status</th>
            <th>Actions</th>
        </tr>
    </thead>
</table>

<script>
$('#studentTable').DataTable({
    processing: true,
    serverSide: true,
    ajax: { url: '/api/student-accounts', type: 'GET' },
    columns: [
        { data: 'studentNumber' },
        { data: 'fullName' },
        { data: 'programme' },
        { data: 'balance', className: 'text-end',
          render: data => formatCurrency(data) },
        { data: 'status', render: data => renderStatusBadge(data) },
        { data: 'studentAccountId', render: renderActionButtons }
    ],
    pageLength: 25,
    language: { search: 'Search:' }
});
</script>
```

### Forms
- Labels always above the input (not floating labels — avoids confusion in financial forms).
- Required fields marked with a red asterisk `*`.
- Currency inputs: right-aligned, `step="0.01"`, formatted with thousands separator on blur.
- Date inputs: use Flatpickr with `dateFormat: "Y-m-d"` for consistent ISO format.
- All forms have `@Html.AntiForgeryToken()`.
- Disable the submit button after the first click to prevent double submission.

```html
<div class="mb-3">
    <label for="amountTendered" class="form-label fw-semibold">
        Amount Tendered <span class="text-danger">*</span>
    </label>
    <div class="input-group">
        <span class="input-group-text">₱</span>
        <input type="number" class="form-control text-end"
               id="amountTendered" name="AmountTendered"
               step="0.01" min="0" required
               asp-for="AmountTendered" />
    </div>
    <span asp-validation-for="AmountTendered" class="text-danger small"></span>
</div>
```

### Confirmation Modals
All destructive or irreversible actions use a Bootstrap modal confirmation:

```html
<!-- Void Receipt Button triggers modal -->
<button class="btn btn-danger btn-sm"
        data-bs-toggle="modal"
        data-bs-target="#voidModal"
        data-or-number="OR-2025-00101">
    Void Receipt
</button>

<!-- Void Modal -->
<div class="modal fade" id="voidModal">
    <div class="modal-dialog">
        <div class="modal-content">
            <div class="modal-header bg-danger text-white">
                <h5 class="modal-title">Confirm Void Receipt</h5>
            </div>
            <div class="modal-body">
                <p>You are about to void receipt <strong id="voidOrNumber"></strong>.
                   This action cannot be undone.</p>
                <div class="mb-3">
                    <label class="form-label fw-semibold">Reason <span class="text-danger">*</span></label>
                    <textarea id="voidReason" class="form-control" rows="3" required></textarea>
                </div>
            </div>
            <div class="modal-footer">
                <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Cancel</button>
                <button type="button" id="confirmVoidBtn" class="btn btn-danger">Void Receipt</button>
            </div>
        </div>
    </div>
</div>
```

### Alert / Toast Notifications
Use Bootstrap toast for non-blocking feedback after AJAX actions:

```html
<div class="toast-container position-fixed bottom-0 end-0 p-3">
    <div id="actionToast" class="toast" role="alert">
        <div class="toast-header">
            <strong class="me-auto" id="toastTitle">Success</strong>
            <button type="button" class="btn-close" data-bs-dismiss="toast"></button>
        </div>
        <div class="toast-body" id="toastMessage"></div>
    </div>
</div>
```

---

## 7.6 Key Page Layouts

### Dashboard
- Top row: 4 KPI cards (Total Collection Today, Outstanding AR, Payables Due in 30 Days, Budget Remaining %)
- Middle row: Bar chart (Monthly Collection vs. Target) + Pie chart (AR Ageing)
- Bottom row: Recent Transactions table (last 10 payments posted)
- All data loaded via AJAX on page load; refresh button triggers reload

### Statement of Account Page
```
┌─────────────────────────────────────────────────────────────┐
│  [Student Info Card: Name | Prog | Year | Student No.]       │
├─────────────────┬───────────────────────────────────────────┤
│  Semester:      │  [Semester Dropdown]                        │
├─────────────────┴───────────────────────────────────────────┤
│  BILLING SUMMARY CARD                                         │
│  Total Assessed: ₱42,000.00   Discount: (₱5,000.00)          │
│  Total Paid:     ₱20,000.00   Balance:  ₱17,000.00 [DANGER]  │
├─────────────────────────────────────────────────────────────┤
│  CHARGES TABLE                                                │
│  Tuition (21u × ₱1,500)  ₱31,500.00                          │
│  Registration Fee          ₱500.00                            │
│  ...                                                          │
├─────────────────────────────────────────────────────────────┤
│  DISCOUNTS TABLE                                              │
│  Dean's List Scholarship  (₱5,000.00)                         │
├─────────────────────────────────────────────────────────────┤
│  PAYMENT HISTORY TABLE                                        │
│  OR-2025-00001 | Aug 1, 2025 | Cash | ₱20,000.00             │
└─────────────────────────────────────────────────────────────┘
```

### Payment Collection Page
```
┌─────────────────────────────────────────────────────────────┐
│  STEP 1: Search Student  [Student No. / Name input]  [Search]│
├─────────────────────────────────────────────────────────────┤
│  STUDENT CARD: Juan Dela Cruz | 2025-0001 | Balance: ₱17,000 │
├─────────────────────────────────────────────────────────────┤
│  STEP 2: Payment Details                                      │
│  Payment Mode: [Cash ▼]                                       │
│  Amount Tendered: [₱ ________________]                        │
│  Change: ₱ 0.00  (auto-computed)                              │
├─────────────────────────────────────────────────────────────┤
│  [Cancel]                            [Post Payment →]         │
└─────────────────────────────────────────────────────────────┘
```

---

## 7.7 Currency Formatting

All monetary values in the system use a consistent format:

```javascript
// Shared helper function (vcas.js)
function formatCurrency(value) {
    if (value === null || value === undefined) return '—';
    return new Intl.NumberFormat('en-PH', {
        style: 'currency',
        currency: 'PHP',
        minimumFractionDigits: 2
    }).format(value);
    // Output: ₱17,000.00
}

// Negative values (credit memos, discounts) shown in parentheses
function formatCurrencyAccounting(value) {
    const formatted = Math.abs(value).toLocaleString('en-PH', {
        minimumFractionDigits: 2, maximumFractionDigits: 2
    });
    return value < 0 ? `(₱${formatted})` : `₱${formatted}`;
}
```

---

## 7.8 Print & PDF Templates

### Official Receipt Layout
```
┌─────────────────────────────────────────┐
│  VALENCIA COLLEGE                        │
│  [Address] | TIN: XXX-XXX-XXX           │
│  ────────────────────────────────────── │
│  OFFICIAL RECEIPT                        │
│  OR No.: OR-2025-00101     Date: Aug 1  │
│  ────────────────────────────────────── │
│  Received from: Juan Dela Cruz           │
│  Student No.:   2025-0001               │
│  ────────────────────────────────────── │
│  For:                           Amount  │
│  Tuition — 1st Sem 2025-2026  ₱20,000  │
│  ────────────────────────────────────── │
│  TOTAL:                        ₱20,000  │
│  ────────────────────────────────────── │
│  Payment Mode: CASH                      │
│  ────────────────────────────────────── │
│  Cashier: Maria Santos                  │
│  [Signature line]                        │
│                                          │
│  *** NOT VALID WITHOUT CASHIER STAMP *** │
└─────────────────────────────────────────┘
```

---

## 7.9 Accessibility

| Standard | Requirement |
|----------|-------------|
| WCAG 2.1 Level AA | Target compliance for all interactive elements |
| Keyboard navigation | All actions reachable via keyboard; visible focus ring on all focusable elements |
| ARIA labels | All icon-only buttons have `aria-label`; tables have `aria-describedby` |
| Colour contrast | Minimum 4.5:1 for normal text; 3:1 for large text |
| Error messages | Errors associated with inputs via `aria-describedby`; not communicated by colour alone |
| Screen readers | Status badges include hidden text (e.g., `<span class="visually-hidden">Status: </span>`) |

---

## 7.10 JavaScript Conventions

- Use `const` and `let`; avoid `var`.
- All AJAX calls via `fetch` API (or jQuery `$.ajax` for DataTables integration).
- Centralise CSRF token injection:
  ```javascript
  const antiForgeryToken = document.querySelector('input[name="__RequestVerificationToken"]').value;
  ```
- Disable form submit button on click; re-enable only on error response.
- Monetary input fields: auto-format on `blur`, strip formatting on `focus` for editing.
- Console logging disabled in production build (`rollup` / `gulp` strips `console.*`).
