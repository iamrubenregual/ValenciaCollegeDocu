# 01 — Project Overview

## 1.1 Vision

The **Valencia College Accounting System (VCAS)** is an enterprise-grade, web-based financial management platform that centralises all monetary transactions, billing operations, payroll, budgeting, and reporting functions for Valencia College. It eliminates manual ledger-keeping, reduces data-entry errors, enforces financial controls through approval workflows, and provides real-time financial visibility to authorised stakeholders.

---

## 1.2 Goals & Objectives

| # | Objective |
|---|-----------|
| 1 | Automate student billing from enrolment through payment and clearance |
| 2 | Maintain a real-time, accurate General Ledger with double-entry accounting |
| 3 | Enforce role-based access so each user interacts only with data they are authorised to see |
| 4 | Provide management-level dashboards and statutory financial reports on demand |
| 5 | Support audit compliance through immutable transaction logs |
| 6 | Integrate seamlessly with the existing Student Information System (SIS) |
| 7 | Process payroll with statutory deduction compliance |

---

## 1.3 Scope

### In-Scope

- Student account setup and lifecycle management
- Tuition, miscellaneous, and laboratory fee billing
- Multiple payment channels (cash, cheque, online transfer, card)
- Scholarship, grant, and waiver management
- Accounts Receivable (AR) and Accounts Payable (AP)
- Chart of Accounts and General Ledger (GL)
- Journal entry creation, review, and posting
- Employee payroll computation and payslip generation
- Departmental budget creation and monitoring
- Standard financial statements (Balance Sheet, Income Statement, Cash Flow)
- Role-based access control with multi-level approval
- Complete audit trail on all financial transactions

### Out-of-Scope (Phase 1)

- Mobile native application (Android/iOS)
- Integration with third-party e-commerce payment gateways beyond the defined adapters
- Fixed-asset depreciation module (planned Phase 2)
- Dormitory billing (planned Phase 2)

---

## 1.4 Stakeholders

| Stakeholder | Role in the System |
|-------------|-------------------|
| College President / VP-Finance | Executive Dashboard, Final Approver |
| Accounting Manager / Chief Accountant | GL management, Financial Reports, Payroll approval |
| Cashier / Billing Clerk | Payment collection, Invoice generation, Receipt printing |
| Scholarship Officer | Financial aid and scholarship management |
| Human Resources Officer | Employee records, Payroll inputs |
| Budget Officer | Budget creation and monitoring |
| Department Chair / Dean | Department budget viewing |
| Student | View statement of account, payment history |
| IT Administrator | User management, system configuration |
| External Auditor | Read-only audit reports |

---

## 1.5 High-Level System Architecture

```
┌───────────────────────────────────────────────────────────────────┐
│                        Client Browser                             │
│           Razor Views · Bootstrap 5 · jQuery · DataTables         │
└──────────────────────────┬────────────────────────────────────────┘
                           │ HTTPS
┌──────────────────────────▼────────────────────────────────────────┐
│                   ASP.NET Core 8 MVC                               │
│   Controllers → Services → Repositories → Entity Framework Core   │
│                 ASP.NET Core Identity (Auth)                       │
└──────────┬────────────────────────────────────┬───────────────────┘
           │                                    │
┌──────────▼──────────┐              ┌──────────▼──────────┐
│  Microsoft SQL       │              │  File Storage        │
│  Server 2019+        │              │  (Reports / Receipts)│
│  (Primary DB)        │              └─────────────────────┘
└─────────────────────┘
```

### Architectural Pattern

The system follows a **layered MVC architecture**:

```
Presentation Layer   →  Controllers + Razor Views (ViewModels)
Service Layer        →  Business Logic Services (Interfaces + Implementations)
Repository Layer     →  Data Access (Generic Repository + Unit of Work)
Data Layer           →  Entity Framework Core DbContext + SQL Server
```

---

## 1.6 Key Constraints

| Constraint | Detail |
|------------|--------|
| Platform | Windows Server, IIS |
| Browser Support | Chrome 110+, Edge 110+, Firefox 110+ |
| Language | English (with optional Filipino localisation in Phase 2) |
| Availability | 99.5% uptime during school days (6:00 AM – 10:00 PM) |
| Data Retention | Financial records retained for minimum 10 years per regulatory requirement |
| Security | OWASP Top 10 compliance; all sensitive data encrypted at rest and in transit |

---

## 1.7 Assumptions

1. A Student Information System (SIS) exists and exposes a REST API for student data.
2. The college uses a standard academic calendar (Semester-based: 1st Sem, 2nd Sem, Summer).
3. Currency is Philippine Peso (PHP) by default; the system supports multi-currency in configuration.
4. Government statutory deductions follow Philippine law (SSS, PhilHealth, Pag-IBIG, BIR Tax Table).
5. All accounting follows the **accrual basis** method as mandated for educational institutions.

---

## 1.8 Glossary

| Term | Definition |
|------|-----------|
| SOA | Statement of Account — itemised list of charges and payments for a student |
| GL | General Ledger — master record of all financial transactions |
| AR | Accounts Receivable — money owed to the college |
| AP | Accounts Payable — money the college owes to suppliers/vendors |
| COA | Chart of Accounts — structured list of all ledger accounts |
| JE | Journal Entry — a recorded financial transaction affecting two or more GL accounts |
| AY | Academic Year |
| SIS | Student Information System |
| RBAC | Role-Based Access Control |
