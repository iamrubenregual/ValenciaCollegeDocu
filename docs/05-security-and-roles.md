# 05 — Security & User Roles

## 5.1 Authentication

### Mechanism
- **ASP.NET Core Identity** manages user accounts, password hashing (PBKDF2-SHA256), and account lockout.
- Users authenticate via a standard **Login** page (username + password).
- Failed login attempts are counted per account. After **5 consecutive failures**, the account is locked for **15 minutes**.
- Session maintained via encrypted **cookie** (sliding expiration: 8 hours; absolute expiration: 12 hours).
- All authenticated routes require `[Authorize]`; all pages are served over **HTTPS only**.

### Password Policy
| Rule | Requirement |
|------|------------|
| Minimum length | 8 characters |
| Uppercase | At least 1 |
| Lowercase | At least 1 |
| Digit | At least 1 |
| Special character | At least 1 |
| Password history | Last 5 passwords cannot be reused |
| Maximum age | 90 days (system prompts change on expiry) |

### Multi-Factor Authentication (MFA)
- **Phase 2**: TOTP-based MFA (Google Authenticator / Microsoft Authenticator) for privileged roles (Accounting Manager, VP Finance, IT Admin).
- **Phase 1**: Enabled optionally; enforced only for super admin.

---

## 5.2 Roles (RBAC)

All roles are defined in `AspNetRoles`. Each application user is assigned one or more roles via `AspNetUserRoles`.

| Role Name | Role Code | Description |
|-----------|-----------|-------------|
| Super Administrator | `SuperAdmin` | Full system access; manages users, roles, system config |
| IT Administrator | `ITAdmin` | User management, audit logs, system settings (no financial transactions) |
| VP Finance / President | `VPFinance` | Full read access; final approver for high-value transactions |
| Accounting Manager | `AccountingManager` | GL management, period close, report generation, payroll approval |
| Cashier | `Cashier` | Payment collection, OR issuance, daily collection reports |
| Billing Clerk | `BillingClerk` | Fee assessment, billing adjustments, SOA generation |
| Scholarship Officer | `ScholarshipOfficer` | Scholarship award, renewal, scholarship reports |
| AR Clerk | `ArClerk` | AR invoice management, collection follow-up, AR ageing |
| AP Clerk | `ApClerk` | Vendor management, PO creation, DV preparation |
| Budget Officer | `BudgetOfficer` | Budget preparation, monitoring, augmentation requests |
| HR / Payroll Officer | `PayrollOfficer` | Employee master, payroll computation, payslip generation |
| Department Head | `DeptHead` | View own department budget, approve PRs within threshold |
| Student | `Student` | View own SOA and payment history only |
| External Auditor | `Auditor` | Read-only access to GL, financial reports, audit logs |

---

## 5.3 Role-Permission Matrix

`✔` = Allowed  `★` = Allowed with approval  `✗` = Denied  `R` = Read-only

### Student Billing & Payments

| Permission | SuperAdmin | AcctMgr | Cashier | BillingClerk | ScholarOfficer | Student |
|------------|:---:|:---:|:---:|:---:|:---:|:---:|
| View SOA (own) | ✔ | ✔ | ✔ | ✔ | ✗ | ✔ |
| View SOA (all students) | ✔ | ✔ | ✔ | ✔ | ✗ | ✗ |
| Generate billing | ✔ | ✔ | ✗ | ✔ | ✗ | ✗ |
| Adjust billing line | ✔ | ✔ | ✗ | ★ | ✗ | ✗ |
| Post payment | ✔ | ✔ | ✔ | ✗ | ✗ | ✗ |
| Void receipt | ✔ | ✔ | ★ | ✗ | ✗ | ✗ |
| Issue refund | ✔ | ★ | ✗ | ✗ | ✗ | ✗ |
| Set enrolment hold | ✔ | ✔ | ✗ | ✔ | ✗ | ✗ |
| Override enrolment hold | ✔ | ✔ | ✗ | ✗ | ✗ | ✗ |

### General Ledger

| Permission | SuperAdmin | VPFinance | AcctMgr | Auditor |
|------------|:---:|:---:|:---:|:---:|
| View COA | ✔ | R | ✔ | R |
| Manage COA | ✔ | ✗ | ✔ | ✗ |
| Create manual JE | ✔ | ✗ | ✔ | ✗ |
| Approve/Post JE | ✔ | ✔ | ✔ | ✗ |
| Reject JE | ✔ | ✔ | ✔ | ✗ |
| View trial balance | ✔ | ✔ | ✔ | R |
| Soft close period | ✔ | ✗ | ✔ | ✗ |
| Hard close period | ✔ | ✔ | ✗ | ✗ |
| Re-open period | ✔ | ✗ | ✗ | ✗ |

### Accounts Payable

| Permission | SuperAdmin | VPFinance | AcctMgr | ApClerk | DeptHead |
|------------|:---:|:---:|:---:|:---:|:---:|
| Manage vendors | ✔ | ✗ | ✔ | ✔ | ✗ |
| Create PR | ✔ | ✗ | ✔ | ✔ | ✔ |
| Approve PR | ✔ | ✔ | ✔ | ✗ | ★ |
| Create PO | ✔ | ✗ | ✔ | ✔ | ✗ |
| Create DV | ✔ | ✗ | ✔ | ✔ | ✗ |
| Approve DV (≤ ₱50,000) | ✔ | ✗ | ✔ | ✗ | ✗ |
| Approve DV (> ₱50,000) | ✔ | ✔ | ✗ | ✗ | ✗ |
| Release payment | ✔ | ✔ | ✔ | ✗ | ✗ |

### Payroll

| Permission | SuperAdmin | VPFinance | AcctMgr | PayrollOfficer |
|------------|:---:|:---:|:---:|:---:|
| Manage employee master | ✔ | ✗ | ✗ | ✔ |
| Create payroll run | ✔ | ✗ | ✗ | ✔ |
| Compute payroll | ✔ | ✗ | ✗ | ✔ |
| Approve payroll | ✔ | ✔ | ✔ | ✗ |
| Post payroll to GL | ✔ | ✗ | ✔ | ✗ |
| View all payslips | ✔ | ✔ | ✔ | ✔ |
| View own payslip | ✔ | ✔ | ✔ | ✔ |

### Budget

| Permission | SuperAdmin | VPFinance | AcctMgr | BudgetOfficer | DeptHead |
|------------|:---:|:---:|:---:|:---:|:---:|
| Create budget | ✔ | ✗ | ✗ | ✔ | ✗ |
| Submit budget | ✔ | ✗ | ✗ | ✔ | ✗ |
| Approve budget | ✔ | ✔ | ✗ | ✗ | ✗ |
| View own dept budget | ✔ | ✔ | ✔ | ✔ | ✔ |
| View all budgets | ✔ | ✔ | ✔ | ✔ | ✗ |
| Request augmentation | ✔ | ✗ | ✗ | ✔ | ✔ |
| Approve augmentation | ✔ | ✔ | ✗ | ✗ | ✗ |

---

## 5.4 Claims-Based Authorization

Beyond roles, fine-grained permissions are implemented as **claims**:

```csharp
// Claim types defined as constants
public static class ClaimTypes
{
    public const string CanVoidReceipt      = "permission.receipt.void";
    public const string CanOverrideBillingHold = "permission.billing.override_hold";
    public const string CanApproveHighValueDv  = "permission.dv.approve_high_value";
    public const string CanClosePeriod         = "permission.gl.close_period";
    public const string CanViewPayslipAll      = "permission.payroll.view_all_payslips";
}

// Policy registration
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("CanVoidReceipt", policy =>
        policy.RequireClaim(ClaimTypes.CanVoidReceipt, "true"));

    options.AddPolicy("CanApproveHighValueDv", policy =>
        policy.RequireClaim(ClaimTypes.CanApproveHighValueDv, "true"));
});

// Controller usage
[Authorize(Policy = "CanVoidReceipt")]
public async Task<IActionResult> VoidReceipt(int id, string reason) { ... }
```

---

## 5.5 Data-Level Security

### Student Data Isolation
- Students authenticated with the `Student` role can only query their own `StudentAccountId`.
- Controller enforces: `studentAccountId == currentUser.StudentAccountId` before returning data.

### Department Scope
- `DeptHead` role queries are filtered to their assigned `DepartmentId`.
- The `IDepartmentScopedService` interface enforces department filtering at the service layer.

### Cashier Session Scope
- Cashiers can only access transactions within their **open session**.
- Reports for other cashiers are restricted to the `AccountingManager` role.

---

## 5.6 Input Validation & Injection Prevention

| Threat | Mitigation |
|--------|-----------|
| SQL Injection | EF Core parameterised queries; no raw string SQL concatenation |
| XSS (Cross-Site Scripting) | Razor auto-encodes all output; `[ValidateAntiForgeryToken]` on all POST actions |
| CSRF | Anti-forgery token on all forms via `@Html.AntiForgeryToken()` or `[AutoValidateAntiforgeryToken]` globally |
| Mass Assignment | Explicit ViewModels with only allowed properties; never bind domain entities directly |
| Path Traversal | File upload paths sanitised; only allowed extensions (PDF, PNG, JPG, XLSX) |
| Open Redirect | Validate return URLs against allow-list before redirect |
| Sensitive Data Exposure | Bank account numbers and TINs masked in UI (show last 4 digits only) |

---

## 5.7 Audit Trail

All create, update, delete, post, void, approve, and reject operations are recorded in `AuditLogs` via an **EF Core SaveChanges interceptor**:

```csharp
public class AuditInterceptor : SaveChangesInterceptor
{
    public override InterceptionResult<int> SavingChanges(
        DbContextEventData eventData, InterceptionResult<int> result)
    {
        var context = eventData.Context!;
        var entries = context.ChangeTracker.Entries()
            .Where(e => e.State is EntityState.Added or EntityState.Modified or EntityState.Deleted);

        foreach (var entry in entries)
        {
            var log = new AuditLog
            {
                Timestamp  = DateTime.UtcNow,
                UserId     = _currentUserService.UserId,
                UserName   = _currentUserService.UserName,
                Action     = entry.State.ToString(),
                EntityName = entry.Entity.GetType().Name,
                EntityId   = entry.Properties.FirstOrDefault(p => p.Metadata.IsPrimaryKey())
                                  ?.CurrentValue?.ToString(),
                OldValues  = entry.State == EntityState.Modified
                                  ? JsonSerializer.Serialize(entry.OriginalValues.ToObject())
                                  : null,
                NewValues  = entry.State != EntityState.Deleted
                                  ? JsonSerializer.Serialize(entry.CurrentValues.ToObject())
                                  : null,
                IpAddress  = _httpContextAccessor.HttpContext?.Connection.RemoteIpAddress?.ToString()
            };
            context.AuditLogs.Add(log);
        }
        return base.SavingChanges(eventData, result);
    }
}
```

### Immutability
- Audit log records are **insert-only** — no UPDATE or DELETE permissions are granted to the application DB user on `AuditLogs`.
- The `AuditLogs` table is on a separate filegroup with read-only file after rotation (optional advanced config).

---

## 5.8 Multi-Level Approval Workflows

### Approval Thresholds

| Transaction Type | Level 1 (AP Clerk / Initiator) | Level 2 (Acctg Manager) | Level 3 (VP Finance) |
|-----------------|-------------------------------|------------------------|---------------------|
| Disbursement Voucher | Create / prepare | Approve if ≤ ₱50,000 | Approve if > ₱50,000 |
| Manual Journal Entry | Create | Approve / Post | — |
| Refund | Request | Approve if ≤ ₱5,000 | Approve if > ₱5,000 |
| Budget Augmentation | Request | Recommend | Approve |
| Receipt Void | Request | Approve | — |
| Bad Debt Write-off | AR Clerk request | Accounting Manager recommend | VP Finance approve |

### Approval Notifications
- On submission: email notification sent to the next approver in the chain.
- On approval/rejection: email notification sent to the initiator.
- Pending approvals visible in each user's **My Approvals** dashboard widget.

---

## 5.9 Session & Cookie Security

```csharp
builder.Services.ConfigureApplicationCookie(options =>
{
    options.Cookie.HttpOnly = true;
    options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
    options.Cookie.SameSite = SameSiteMode.Strict;
    options.SlidingExpiration = true;
    options.ExpireTimeSpan = TimeSpan.FromHours(8);
    options.LoginPath = "/Account/Login";
    options.LogoutPath = "/Account/Logout";
    options.AccessDeniedPath = "/Account/AccessDenied";
});
```

---

## 5.10 Security Headers

Applied via middleware:

```csharp
app.Use(async (context, next) =>
{
    context.Response.Headers.Add("X-Content-Type-Options", "nosniff");
    context.Response.Headers.Add("X-Frame-Options", "DENY");
    context.Response.Headers.Add("X-XSS-Protection", "1; mode=block");
    context.Response.Headers.Add("Referrer-Policy", "strict-origin-when-cross-origin");
    context.Response.Headers.Add("Content-Security-Policy",
        "default-src 'self'; script-src 'self' 'nonce-{NONCE}'; style-src 'self' 'unsafe-inline';");
    await next();
});
```

---

## 5.11 User Management (IT Admin Functions)

- Create / deactivate user accounts
- Assign / revoke roles and claims
- Force password reset
- Unlock locked accounts
- View login history and active sessions
- Export user-role assignment matrix for auditing
