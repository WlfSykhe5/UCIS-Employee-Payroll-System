# UCIS — Employee & Payroll Management System

**ASP.NET Core Razor Pages | C# | Entity Framework Core | SQLite | Clerk | Bootstrap 5**

UCIS is an employee and payroll management web application developed as a collaborative project. It brings together employee management, monthly payroll preparation, review and approval, period management, and reporting.

The application incorporates business rules and internal controls to help maintain data accuracy, enforce appropriate access permissions, and protect finalized payroll records.

## Key Features

### Employee Management
- Create, view, update, and manage employee records.
- Maintain employee information and relationships with departments and job roles.
- Search employee records.
- Associate employees with effective-dated salary packages for payroll processing.

### Payroll Management
- Create and manage payroll records for monthly payroll periods.
- Automatically identify the applicable salary package based on the employee and payroll period.
- Calculate gross and net pay using basic salary, allowances, additional earnings, and deductions.
- Search payroll records by employee, payroll period, and processing status.
- View payroll statistics, including record counts by status, gross pay, deductions, and net pay.

### Payroll Workflow and Internal Controls
- **Preparation and approval:** Records move through Draft, Submitted, Approved, and Returned statuses, with corrections and resubmission supported.
- **Segregation of duties:** Users cannot approve payroll records they originally prepared.
- **Role-based authorization:** Payroll Specialists prepare and submit records, while Payroll Team Leads review, approve, and manage payroll periods.
- **Duplicate prevention:** Only one payroll record is permitted per employee and payroll period.
- **Period closure:** A payroll period can only be closed when every active employee has an approved payroll record.
- **Record protection:** Approved payroll records are read-only, and closed periods are protected against further payroll changes.

### Authentication and Security
- Authentication through Clerk using OpenID Connect.
- Application roles determined by matching authenticated users to employee records.
- Server-side authorization policies and protected application pages.
- Role-specific navigation and access to payroll functionality.
- Custom Access Denied handling.

## Technology Stack

| Component | Technology |
|---|---|
| Web application | ASP.NET Core Razor Pages (.NET 10) |
| Programming language | C# |
| Database | SQLite |
| Data access | Entity Framework Core |
| Authentication | Clerk / OpenID Connect |
| Authorization | ASP.NET Core authorization policies |
| User interface | Razor, HTML, CSS, Bootstrap 5 |
| Version control | Git and GitHub |

## My Contributions

My primary responsibility was developing the **payroll module**, including its data models, business rules, workflows, user interfaces, and reporting functionality.

My contributions included:

- **Payroll data modeling:** Developed the SalaryPackage, PayrollPeriod, and PayrollRecord models and their relationships with employee data.
- **Payroll preparation:** Implemented payroll record creation, editing, validation, automatic salary-package selection, and payroll calculations.
- **Review and approval workflow:** Developed record submission, approval, return-for-correction, and resubmission functionality.
- **Business controls:** Implemented restrictions on duplicate records, self-approval, unauthorized changes, and modifications to finalized payroll records.
- **Payroll period management:** Developed monthly payroll period creation and closure functionality, including completeness checks.
- **Search and statistics:** Built custom payroll search and statistics pages for record retrieval, monitoring, and reporting.
- **Testing and integration:** Performed end-to-end testing of payroll workflows, role-based permissions, period closure, and reporting. Also contributed to UI refinements and the final application demonstration.

An important part of this work was translating payroll business requirements into application functionality, rather than treating payroll as a simple data-entry process.

## Team Collaboration

UCIS was developed collaboratively by three team members.

| Team Member | Main Contributions |
|---|---|
| **Zarah Z.** | Payroll module, business rules and controls, payroll search and statistics, and testing |
| **Lila I.** | Employee management |
| **Deonta W.** | Base application setup, Clerk authentication, role-based authorization, GitHub collaboration and branch management, and UI refinements |

We used GitHub branches and pull requests to develop separate components, review changes, and integrate our work into the final application.

## Design Decisions

Several design decisions shaped the payroll module.

**Separate salary packages from employee records**

Compensation information is maintained in a separate entity with effective dates. This allows payroll processing to reference the applicable salary package without storing salary information directly in the employee record.

**Use a controlled payroll workflow**

Payroll records move through defined processing statuses, with separate permissions for preparation and approval. Returned records can be corrected and resubmitted.

**Enforce segregation of duties**

Payroll Specialists prepare and submit records, while Payroll Team Leads review and approve them. The application also prevents users from approving records they prepared themselves.

**Require completeness before period closure**

A payroll period cannot be finalized until every active employee has an approved payroll record.

**Protect finalized payroll information**

Approved payroll records and closed payroll periods are restricted from subsequent modifications.

These decisions demonstrate how relational database design, application logic, and authorization can work together to support business processes and internal controls.

## Current Scope and Future Enhancements

UCIS was developed as a functional course project.

The current implementation supports monthly payroll processing using preconfigured salary packages.

Potential enhancements include:

- Salary package maintenance and approval workflows.
- Prorated pay for mid-period salary changes and employee movements.
- Employee self-service payslips.
- More detailed audit history and change tracking.
- Additional payroll frequencies and processing options.

---

# Technical Documentation

## Authentication Architecture

Authentication is handled through Clerk using OpenID Connect.

The application uses:

- Cookie authentication for the local application session.
- OpenID Connect for Clerk authentication.
- Clerk `sub` claims to identify authenticated users.
- Clerk email claims to match authenticated users with employee records in the application database.

Application roles are obtained from each employee's associated job role.

The authentication and role-mapping process follows this structure:

```text
Clerk User
    |
    v
Email Claim
    |
    v
Employee.Email
    |
    v
Employee.JobRole
    |
    v
AppRole Claim
```

## Role-Based Access Control

The application supports the following roles:

| Role | Payroll Records | Statistics | Payroll Review | Payroll Periods | Create/Edit Payroll |
|---|---|---|---|---|---|
| Payroll Specialist | Yes | Yes | No | No | Yes |
| Payroll Team Lead | Yes | Yes | Yes | Yes | No |
| Non-Payroll User | No | No | No | No | No |

Payroll pages and actions are protected through authorization policies, including server-side checks.

Unauthorized users are directed to a custom Access Denied page.

Navigation visibility is used for convenience, while authorization restrictions are enforced on the server.

## Payroll Workflow

Payroll records follow a preparation, submission, review, and approval process.

```text
Draft
  |
  v
Submitted
  |
  +------> Returned
  |           |
  |           v
  |       Correction
  |           |
  |           v
  |       Resubmitted
  |           |
  |           v
  +------> Approved
```

### Payroll Specialist

Payroll Specialists can:

- Create payroll records.
- Edit Draft records.
- Edit Returned records.
- Delete Draft or Returned records.
- Submit Draft or Returned records for review.

### Payroll Team Lead

Payroll Team Leads can:

- View submitted payroll records.
- Approve payroll records.
- Return payroll records for correction.
- Manage payroll periods.
- Close completed payroll periods.

A Payroll Team Lead cannot approve a payroll record they originally prepared.

Once approved, payroll records become read-only.

## Payroll Period Rules

Payroll is processed on a monthly basis.

A payroll period cannot be closed until:

- A payroll record exists for every active employee.
- Every payroll record in the period has been approved.

Once a payroll period is closed:

- New payroll records cannot be created for that period.
- Existing payroll records cannot be edited or deleted.
- Records cannot be submitted for review.
- Records cannot be approved or returned.

These controls help preserve the completeness and integrity of finalized payroll periods.

## Database Design

The application uses Entity Framework Core with SQLite.

### Main Entities

- **Employee** — Employee information, including department and job role.
- **Department** — Organizational department information.
- **JobRole** — Employee job roles used for application role mapping.
- **SalaryPackage** — Basic salary, allowance, and applicable effective dates.
- **PayrollPeriod** — Payroll processing periods and closure status.
- **PayrollRecord** — Employee payroll transactions, calculated pay, status, and preparation/review details.

Payroll records are related to:

- Employees
- Payroll periods
- Salary packages

The database also enforces a unique employee-and-period combination to prevent duplicate payroll records.

## Local Development Setup

### Prerequisites

- .NET 10 SDK
- A compatible development environment, such as Visual Studio
- A Clerk application configured for OpenID Connect
- Entity Framework Core CLI tools for database migrations

### 1. Clone the Repository

```bash
git clone https://github.com/WlfSykhe5/UCIS-Employee-Payroll-System.git
```

Navigate to the project directory containing the application project file.

### 2. Configure Clerk Authentication

Clerk credentials are stored using .NET User Secrets rather than being committed to the repository.

Initialize User Secrets, if not already configured:

```bash
dotnet user-secrets init
```

Add the Clerk settings:

```bash
dotnet user-secrets set "Clerk:ClientId" "YOUR_CLIENT_ID"
dotnet user-secrets set "Clerk:ClientSecret" "YOUR_CLIENT_SECRET"
dotnet user-secrets set "Clerk:Authority" "https://YOUR-CLERK-DOMAIN"
```

Verify the configuration:

```bash
dotnet user-secrets list
```

The application uses the following OpenID Connect package:

```bash
dotnet add package Microsoft.AspNetCore.Authentication.OpenIdConnect --version 10.0.0
```

The Clerk sign-up URL is configured in `appsettings.json`:

```json
{
  "Clerk": {
    "SignUpUrl": "https://YOUR-CLERK-DOMAIN/sign-up"
  }
}
```

The Clerk application must also be configured with redirect URLs that match the application's local HTTPS address.

### 3. Configure the Database

Install the Entity Framework Core CLI tools if needed:

```bash
dotnet tool install --global dotnet-ef
```

Apply the database migrations:

```bash
dotnet ef database update
```

### 4. Build and Run

Restore dependencies:

```bash
dotnet restore
```

Build the application:

```bash
dotnet build
```

Run using the HTTPS launch profile:

```bash
dotnet run --launch-profile https
```

The development application is configured to run at:

```text
https://localhost:7219
```

### 5. Test User Roles

Role assignment depends on matching the authenticated Clerk email address to an employee record.

The Clerk email must match an `Employee.Email` value in the application database.

Recommended test roles include:

- Payroll Specialist
- Payroll Team Lead
- Non-payroll employee

Use email addresses you control when testing Clerk verification.

## Project Structure

```text
IS7012-FinalProject
|
+-- Data
|   +-- ApplicationDbContext.cs
|
+-- Models
|   +-- Employee.cs
|   +-- Department.cs
|   +-- JobRole.cs
|   +-- SalaryPackage.cs
|   +-- PayrollPeriod.cs
|   +-- PayrollRecord.cs
|   +-- PayrollStatus.cs
|
+-- Pages
|   +-- Employees
|   +-- PayrollRecords
|   +-- PayrollReview
|   +-- PayrollPeriods
|   +-- PayrollStatistics
|   +-- AccessDenied.cshtml
|
+-- Services
|   +-- AppRoleClaimsTransformation.cs
|
+-- wwwroot
|   +-- css
|       +-- site.css
|
+-- Program.cs
+-- appsettings.json
```

## Security Controls

The application incorporates the following controls:

- Authenticated access to protected pages.
- Role-based authorization policies.
- Server-side authorization enforcement.
- Payroll Specialist-only preparation actions.
- Payroll Team Lead-only review and period management actions.
- Self-approval prevention.
- Duplicate payroll record prevention.
- Approved-record editing restrictions.
- Closed-period modification protection.
- Custom unauthorized-access handling.

These controls are intended to demonstrate how security and business requirements can be incorporated directly into application functionality.

---

**Project:** UCIS — Employee & Payroll Management System

**Development:** Collaborative academic project

**Repository:** [UCIS-Employee-Payroll-System](https://github.com/WlfSykhe5/UCIS-Employee-Payroll-System)
