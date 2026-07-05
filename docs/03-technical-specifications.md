# 03 — Technical Specifications

## 3.1 Technology Stack

| Tier | Technology | Version | Purpose |
|------|-----------|---------|---------|
| Runtime | .NET | 8.0 LTS | Application runtime |
| Web Framework | ASP.NET Core MVC | 8.0 | MVC web application |
| Language | C# | 12.0 | Primary programming language |
| ORM | Entity Framework Core | 8.x | Database access (Code-First) |
| Database | Microsoft SQL Server | 2019+ (Express / Standard / Enterprise) | Primary data store |
| Frontend CSS | Bootstrap | 5.3 | Responsive UI framework |
| Frontend JS | jQuery | 3.7 | DOM manipulation, AJAX |
| Data Tables | DataTables.js | 1.13 | Sortable, filterable tables |
| Charts | Chart.js | 4.x | Dashboard visualisations |
| Date Picker | Flatpickr | 4.6 | Date/time inputs |
| Rich Text | Summernote | 0.9 | Remarks / notes fields |
| Authentication | ASP.NET Core Identity | 8.x | User management, password hashing |
| Authorisation | Policy-based + Claims | — | RBAC enforcement |
| Reporting | FastReport.NET | Latest | RDLC-style report designer |
| PDF Export | iTextSharp / FastReport export | — | PDF generation |
| Excel Export | EPPlus | 7.x | Excel export (GPL / commercial) |
| Logging | Serilog | 3.x | Structured logging (file + DB sink) |
| Email | MailKit | 4.x | SMTP email (receipts, payslips, notifications) |
| Caching | IMemoryCache | built-in | COA, fee structure caching |
| Background Jobs | Hangfire | 1.8 | Scheduled reports, email dispatch |
| Testing (Unit) | xUnit | 2.x | Unit tests |
| Testing (Mock) | Moq | 4.x | Dependency mocking |
| Testing (Assert) | FluentAssertions | 6.x | Readable assertions |
| Source Control | Git | — | Version control |
| CI/CD | GitHub Actions / Azure DevOps | — | Build, test, deploy pipeline |

---

## 3.2 Solution Structure

```
ValenciaCollege.Accounting.sln
│
├── src/
│   ├── VCAS.Web/                        ← ASP.NET Core MVC Project (Presentation)
│   │   ├── Controllers/
│   │   │   ├── StudentAccountController.cs
│   │   │   ├── FeeStructureController.cs
│   │   │   ├── PaymentController.cs
│   │   │   ├── ScholarshipController.cs
│   │   │   ├── AccountsReceivableController.cs
│   │   │   ├── AccountsPayableController.cs
│   │   │   ├── GeneralLedgerController.cs
│   │   │   ├── PayrollController.cs
│   │   │   ├── BudgetController.cs
│   │   │   ├── ReportController.cs
│   │   │   └── AdminController.cs
│   │   ├── Views/
│   │   │   ├── Shared/
│   │   │   │   ├── _Layout.cshtml
│   │   │   │   ├── _NavigationSidebar.cshtml
│   │   │   │   ├── _Alerts.cshtml
│   │   │   │   └── Error.cshtml
│   │   │   ├── StudentAccount/
│   │   │   ├── FeeStructure/
│   │   │   ├── Payment/
│   │   │   ├── Scholarship/
│   │   │   ├── AccountsReceivable/
│   │   │   ├── AccountsPayable/
│   │   │   ├── GeneralLedger/
│   │   │   ├── Payroll/
│   │   │   ├── Budget/
│   │   │   ├── Report/
│   │   │   └── Dashboard/
│   │   ├── ViewModels/
│   │   ├── Filters/                     ← Action filters (audit, permission)
│   │   ├── Helpers/
│   │   ├── wwwroot/
│   │   │   ├── css/
│   │   │   ├── js/
│   │   │   ├── lib/                     ← Vendor JS/CSS
│   │   │   └── reports/                 ← RDLC report templates
│   │   ├── appsettings.json
│   │   ├── appsettings.Development.json
│   │   └── Program.cs
│   │
│   ├── VCAS.Application/                ← Service Layer (Business Logic)
│   │   ├── Interfaces/
│   │   │   ├── IStudentAccountService.cs
│   │   │   ├── IFeeService.cs
│   │   │   ├── IPaymentService.cs
│   │   │   ├── IScholarshipService.cs
│   │   │   ├── IArService.cs
│   │   │   ├── IApService.cs
│   │   │   ├── IGlService.cs
│   │   │   ├── IPayrollService.cs
│   │   │   ├── IBudgetService.cs
│   │   │   └── IReportService.cs
│   │   ├── Services/
│   │   │   └── (implementations of above interfaces)
│   │   ├── DTOs/                        ← Data Transfer Objects
│   │   ├── Validators/                  ← FluentValidation validators
│   │   ├── Mappings/                    ← AutoMapper profiles
│   │   └── Events/                      ← Domain events (e.g., PaymentPosted)
│   │
│   ├── VCAS.Domain/                     ← Domain Entities & Business Rules
│   │   ├── Entities/
│   │   │   ├── StudentAccount.cs
│   │   │   ├── FeeType.cs
│   │   │   ├── FeeMatrix.cs
│   │   │   ├── PaymentTransaction.cs
│   │   │   ├── OfficialReceipt.cs
│   │   │   ├── Scholarship.cs
│   │   │   ├── JournalEntry.cs
│   │   │   ├── JournalEntryLine.cs
│   │   │   ├── ChartOfAccount.cs
│   │   │   ├── Employee.cs
│   │   │   ├── PayrollRun.cs
│   │   │   ├── Budget.cs
│   │   │   ├── Vendor.cs
│   │   │   ├── PurchaseOrder.cs
│   │   │   └── DisbursementVoucher.cs
│   │   ├── Enums/
│   │   │   ├── PaymentMode.cs
│   │   │   ├── TransactionStatus.cs
│   │   │   ├── AccountType.cs
│   │   │   ├── EmploymentType.cs
│   │   │   └── ApprovalStatus.cs
│   │   └── ValueObjects/
│   │       ├── Money.cs
│   │       └── AcademicPeriod.cs
│   │
│   └── VCAS.Infrastructure/             ← Data Access & External Services
│       ├── Data/
│       │   ├── AppDbContext.cs
│       │   ├── Migrations/
│       │   └── Configurations/          ← EF Fluent API configurations
│       ├── Repositories/
│       │   ├── IRepository.cs           ← Generic repository interface
│       │   ├── Repository.cs            ← Generic repository implementation
│       └── Services/
│           ├── EmailService.cs
│           ├── ReportGeneratorService.cs
│           └── SisIntegrationService.cs
│
└── tests/
    ├── VCAS.UnitTests/
    │   ├── Services/
    │   └── Domain/
    └── VCAS.IntegrationTests/
        └── Controllers/
```

---

## 3.3 Architectural Patterns

### 3.3.1 MVC Pattern
- **Models** — Entity Framework entities and ViewModels. Entities are in `VCAS.Domain`; ViewModels are in `VCAS.Web/ViewModels`.
- **Views** — Razor `.cshtml` files. Strongly typed to ViewModels.
- **Controllers** — Thin controllers. Delegate all business logic to Application Services.

### 3.3.2 Repository & Unit of Work
```csharp
// Generic repository interface
public interface IRepository<T> where T : class
{
    Task<T?> GetByIdAsync(int id);
    Task<IEnumerable<T>> GetAllAsync();
    Task<IEnumerable<T>> FindAsync(Expression<Func<T, bool>> predicate);
    Task AddAsync(T entity);
    void Update(T entity);
    void Remove(T entity);
}

// Unit of Work interface
public interface IUnitOfWork : IDisposable
{
    IRepository<StudentAccount> StudentAccounts { get; }
    IRepository<PaymentTransaction> Payments { get; }
    IRepository<JournalEntry> JournalEntries { get; }
    // ... one property per aggregate root
    Task<int> SaveChangesAsync();
}
```

### 3.3.3 Service Layer
Services encapsulate all business logic and orchestrate calls to repositories:
```csharp
public interface IPaymentService
{
    Task<ServiceResult<OfficialReceipt>> PostPaymentAsync(PostPaymentDto dto, string userId);
    Task<ServiceResult> VoidReceiptAsync(int receiptId, string reason, string userId);
    Task<DailyCollectionSummaryDto> GetDailyCollectionAsync(DateOnly date, string cashierId);
}
```

### 3.3.4 ViewModel Pattern
Controllers never pass domain entities directly to views. All data is mapped to ViewModels:
```csharp
public class StudentStatementViewModel
{
    public string StudentNumber { get; set; }
    public string FullName { get; set; }
    public string Programme { get; set; }
    public AcademicPeriodViewModel CurrentPeriod { get; set; }
    public decimal TotalAssessed { get; set; }
    public decimal TotalPaid { get; set; }
    public decimal Balance { get; set; }
    public List<BillingLineViewModel> BillingLines { get; set; }
    public List<PaymentHistoryViewModel> Payments { get; set; }
}
```

### 3.3.5 Double-Entry Accounting Engine
Every financial event calls the GL posting engine:
```csharp
public class GlPostingEngine
{
    // Each sub-module produces a PostingRequest; engine validates and posts JE
    Task<PostingResult> PostAsync(PostingRequest request);
}

public class PostingRequest
{
    public string SourceModule { get; init; }    // "Payment", "Payroll", "AP", etc.
    public string ReferenceNumber { get; init; }
    public DateTime TransactionDate { get; init; }
    public string Description { get; init; }
    public List<JournalLine> Lines { get; init; }  // Must balance (sum debits == sum credits)
}
```

---

## 3.4 Database Configuration

### Connection String (appsettings.json)
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=.;Database=ValenciaCollegeAccounting;Trusted_Connection=True;MultipleActiveResultSets=true;TrustServerCertificate=True"
  }
}
```

### EF Core DbContext
```csharp
public class AppDbContext : IdentityDbContext<ApplicationUser>
{
    public DbSet<StudentAccount> StudentAccounts { get; set; }
    public DbSet<FeeType> FeeTypes { get; set; }
    public DbSet<FeeMatrix> FeeMatrices { get; set; }
    public DbSet<PaymentTransaction> PaymentTransactions { get; set; }
    public DbSet<OfficialReceipt> OfficialReceipts { get; set; }
    public DbSet<Scholarship> Scholarships { get; set; }
    public DbSet<ChartOfAccount> ChartOfAccounts { get; set; }
    public DbSet<JournalEntry> JournalEntries { get; set; }
    public DbSet<JournalEntryLine> JournalEntryLines { get; set; }
    public DbSet<Employee> Employees { get; set; }
    public DbSet<PayrollRun> PayrollRuns { get; set; }
    public DbSet<Vendor> Vendors { get; set; }
    public DbSet<PurchaseOrder> PurchaseOrders { get; set; }
    public DbSet<DisbursementVoucher> DisbursementVouchers { get; set; }
    public DbSet<Budget> Budgets { get; set; }
    public DbSet<AuditLog> AuditLogs { get; set; }

    protected override void OnModelCreating(ModelBuilder builder)
    {
        base.OnModelCreating(builder);
        builder.ApplyConfigurationsFromAssembly(Assembly.GetExecutingAssembly());
    }
}
```

### EF Core Conventions
- All decimal/money columns use `decimal(18,2)` precision
- Soft delete via `IsDeleted` (bool) + `DeletedAt` (datetime) + global query filter
- Audit fields on every entity: `CreatedAt`, `CreatedBy`, `UpdatedAt`, `UpdatedBy`
- Optimistic concurrency via `RowVersion` (byte[]) on concurrency-sensitive entities

---

## 3.5 Dependency Injection Registration

```csharp
// Program.cs
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(connectionString));

builder.Services.AddIdentity<ApplicationUser, ApplicationRole>(options =>
{
    options.Password.RequiredLength = 8;
    options.Lockout.MaxFailedAccessAttempts = 5;
    options.Lockout.DefaultLockoutTimeSpan = TimeSpan.FromMinutes(15);
})
.AddEntityFrameworkStores<AppDbContext>()
.AddDefaultTokenProviders();

builder.Services.AddScoped<IUnitOfWork, UnitOfWork>();
builder.Services.AddScoped<IStudentAccountService, StudentAccountService>();
builder.Services.AddScoped<IFeeService, FeeService>();
builder.Services.AddScoped<IPaymentService, PaymentService>();
builder.Services.AddScoped<IScholarshipService, ScholarshipService>();
builder.Services.AddScoped<IGlService, GlService>();
builder.Services.AddScoped<IPayrollService, PayrollService>();
builder.Services.AddScoped<IBudgetService, BudgetService>();
builder.Services.AddScoped<IReportService, ReportService>();
builder.Services.AddScoped<GlPostingEngine>();

builder.Services.AddHangfire(config => config.UseSqlServerStorage(connectionString));
builder.Services.AddHangfireServer();

builder.Services.AddAutoMapper(typeof(MappingProfiles));
builder.Services.AddSingleton<IEmailService, SmtpEmailService>();
```

---

## 3.6 Configuration & App Settings

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "..."
  },
  "AppSettings": {
    "InstitutionName": "Valencia College",
    "InstitutionAddress": "...",
    "Currency": "PHP",
    "FiscalYearStartMonth": 6,
    "ReceiptSeriesPrefix": "OR",
    "MaxCashierSessionHours": 12
  },
  "EmailSettings": {
    "SmtpHost": "smtp.office365.com",
    "SmtpPort": 587,
    "SenderEmail": "accounting@valenciakollehiyo.edu.ph",
    "SenderName": "Valencia College Accounting"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.EntityFrameworkCore": "Warning"
    }
  },
  "Serilog": {
    "WriteTo": [
      { "Name": "Console" },
      { "Name": "File", "Args": { "path": "Logs/vcas-.log", "rollingInterval": "Day" } },
      { "Name": "MSSqlServer", "Args": { "connectionString": "...", "tableName": "AppLogs" } }
    ]
  }
}
```

---

## 3.7 Coding Standards & Conventions

### Naming
| Element | Convention | Example |
|---------|-----------|---------|
| Classes | PascalCase | `StudentAccountService` |
| Interfaces | `I` prefix + PascalCase | `IStudentAccountService` |
| Methods | PascalCase | `PostPaymentAsync` |
| Private fields | `_` prefix + camelCase | `_unitOfWork` |
| Local variables | camelCase | `paymentDto` |
| Constants | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` |
| DB tables | PascalCase plural | `PaymentTransactions` |
| DB columns | PascalCase | `StudentAccountId` |

### Async/Await
- All I/O-bound operations (DB, email, file) must be `async Task` 
- No `.Result` or `.Wait()` (deadlock risk)
- Use `ConfigureAwait(false)` in library code

### Error Handling
- Use a `ServiceResult<T>` pattern to return success/failure without exceptions for expected business errors
- Use `try/catch` only at controller boundary or for unexpected infrastructure failures
- Global exception handler middleware catches unhandled exceptions and returns appropriate HTTP responses

```csharp
public class ServiceResult<T>
{
    public bool IsSuccess { get; init; }
    public T? Data { get; init; }
    public string? ErrorMessage { get; init; }
    public List<string> ValidationErrors { get; init; } = new();

    public static ServiceResult<T> Success(T data) => new() { IsSuccess = true, Data = data };
    public static ServiceResult<T> Failure(string error) => new() { IsSuccess = false, ErrorMessage = error };
}
```

### Validation
- Server-side validation using **FluentValidation** for all DTOs
- Client-side validation via jQuery Validation + `data-val-*` attributes generated from DataAnnotations
- Never trust client-side input alone — always validate server-side

---

## 3.8 Deployment Architecture

```
┌─────────────────────────────────────────────────────┐
│                  Windows Server 2022                 │
│                                                      │
│   ┌──────────────────────┐  ┌──────────────────┐    │
│   │   IIS 10             │  │  SQL Server 2019  │    │
│   │   Application Pool   │  │  (Named Instance) │    │
│   │   .NET 8 Hosting     │  │                  │    │
│   │   Bundle             │  │  DB: VCAS_Prod   │    │
│   └──────────────────────┘  └──────────────────┘    │
│                                                      │
│   ┌──────────────────────┐                          │
│   │  Hangfire Dashboard  │                          │
│   │  (Background Jobs)   │                          │
│   └──────────────────────┘                          │
└─────────────────────────────────────────────────────┘
```

### Deployment Checklist
1. Install .NET 8 Hosting Bundle on IIS server
2. Create Application Pool (No Managed Code, set to .NET CLR v4 for IIS compatibility)
3. Publish project: `dotnet publish -c Release -o ./publish`
4. Configure IIS site pointing to publish output
5. Run EF migrations: `dotnet ef database update --project VCAS.Infrastructure`
6. Seed initial data (COA, roles, admin user)
7. Configure environment variables for connection string and secrets (do NOT store in `appsettings.json` in production)
8. Enable HTTPS (Let's Encrypt or institutional certificate)
9. Configure SQL Server backup schedule (nightly full, hourly differential)

---

## 3.9 Performance Considerations

| Area | Strategy |
|------|---------|
| COA and Fee Structure | Cache in `IMemoryCache` with sliding expiration (5 min), invalidated on update |
| Large Report Queries | Use raw SQL or compiled EF queries; paginate results |
| DataTables | Use server-side processing for tables > 1,000 rows |
| Payroll Batch | Process in background job via Hangfire to avoid HTTP timeout |
| File Exports | Stream response directly; do not buffer entire file in memory |
| DB Indexes | Index on frequently filtered columns (StudentId, AcademicYearId, TransactionDate, Status) |

---

## 3.10 Testing Strategy

| Type | Tool | Coverage Target |
|------|------|----------------|
| Unit Tests | xUnit + Moq + FluentAssertions | All Service layer methods; GL posting engine |
| Integration Tests | xUnit + EF Core InMemory / TestContainers | Controller → Service → DB round trips |
| UI Tests | Playwright (optional Phase 2) | Critical payment and payroll flows |

### Key Unit Test Areas
- `PaymentService.PostPaymentAsync` — validates balance update, OR generation, GL posting
- `GlPostingEngine.PostAsync` — validates balanced entries, period lock enforcement
- `PayrollService.ComputeAsync` — statutory deduction accuracy (SSS, PhilHealth, BIR)
- `BudgetService.CheckBudgetAvailability` — over-budget alert logic
- `FeeService.AssessStudent` — fee matrix lookup and billing generation
