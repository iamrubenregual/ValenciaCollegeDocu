# Valencia College Accounting System — Documentation

> **Stack:** ASP.NET Core MVC · Microsoft SQL Server · C# .NET

This repository contains the complete project documentation for the **Valencia College Accounting System** — a comprehensive, role-based financial management platform designed for school/college operations.

---

## Documentation Index

| # | Document | Description |
|---|----------|-------------|
| 1 | [Project Overview](docs/01-project-overview.md) | Vision, scope, stakeholders, and high-level architecture |
| 2 | [Modules & Features](docs/02-modules-and-features.md) | Detailed feature breakdown for all system modules |
| 3 | [Technical Specifications](docs/03-technical-specifications.md) | Stack, architecture patterns, project structure, and conventions |
| 4 | [Database Schema](docs/04-database-schema.md) | ERD narrative, table definitions, and relationships |
| 5 | [Security & User Roles](docs/05-security-and-roles.md) | RBAC matrix, authentication, authorization, and audit trail |
| 6 | [API Design](docs/06-api-design.md) | RESTful endpoint contracts, request/response models |
| 7 | [UI Design Guidelines](docs/07-ui-design-guidelines.md) | Layout conventions, component library, and UX standards |

---

## Quick Summary

The system is composed of **10 core modules**:

1. Student Account & Billing Management  
2. Fee Structure Management  
3. Payment Processing  
4. Financial Aid & Scholarships  
5. Accounts Receivable  
6. Accounts Payable  
7. General Ledger  
8. Payroll Management  
9. Budget Management  
10. Financial Reporting & Analytics  

---

## Technology Stack at a Glance

| Layer | Technology |
|-------|-----------|
| Frontend | Razor Views (cshtml), Bootstrap 5, jQuery, DataTables.js |
| Backend | ASP.NET Core 8 MVC, C# 12 |
| ORM | Entity Framework Core 8 (Code-First) |
| Database | Microsoft SQL Server 2019+ |
| Auth | ASP.NET Core Identity + JWT Bearer Tokens |
| Reporting | RDLC / FastReport.NET |
| Logging | Serilog (file + database sink) |
| Caching | IMemoryCache / Redis (optional) |
| Testing | xUnit, Moq, FluentAssertions |
