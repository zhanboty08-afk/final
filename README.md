# BudgetPipeline

**Course:** Introduction to Functional Programming  
**Team:** Zhanbolat Tulepbergen, Narges Jowkar, Almadi Askhat  
**Group:** IT-2513  
**Stack:** C# / .NET 8 and native C  
**Supported platform:** Windows x64

BudgetPipeline reads transaction and expense-request CSV files, checks requests against category budgets, and produces spending reports. It demonstrates a functional core with immutable state, explicit failures, configurable rules, LINQ reports, lazy processing, and a native C calculation.

## Project documents

- [Group project report](Group_Project_Report.md) — scope, architecture, implementation, tests, roles, limitations, and submission details.
- [Individual contribution report — Zhanbolat Tulepbergen](Individual_Contribution_Report_Zhanbolat_Tulepbergen.md)

## Main workflows

- Summarise transactions with approved-only and approved-plus-pending policies.
- Validate expense requests and approve them sequentially against the current ledger.
- Process requests through `Result<T>` so a failure stops later dependent work.
- Produce a category spending report and export it to CSV.
- Produce a scored transaction report using two configured scoring functions.
- Calculate category budget utilisation with the native C function.

The bundled transaction data produces totals of **15.50** for approved transactions and **40.50** for approved plus pending. The category report contains Food (**2 transactions, 17.50**) and Transport (**1 transaction, 3.00**). The native report shows Food at **62.50%**, Transport at **30.00%**, and Books at **0.00%** of their limits. Gifts has a zero limit and returns a handled validation failure.

## Source map

| Area | Files and entry points |
|---|---|
| Budget pipeline | `BudgetWorkflow.cs`: `SelectTransactions`, `CalculateCategoryTotals`, `CreateSummary`, `Run` |
| Expense validation and operations | `ExpenseRules.cs`, `ExpenseModels.cs`, `ExpenseWorkflow.cs`, `ExpenseOperations.cs` |
| Explicit results and lookups | `Result.cs`, `Maybe.cs`, `Error.cs` |
| Extensions and reports | `TransactionExtensions.cs`, `Reports.cs` |
| Configured scoring and lazy stream | `Scoring.cs`, `TransactionStream.cs` |
| CSV boundary and CLI | `DataLoader.cs`, `TransactionParser.cs`, `ExpenseRequestParser.cs`; `Program.cs` loads inputs, while `BudgetPipelineConsole.cs` coordinates the demos |
| Console demos | `ReportingDemo.cs`, `BudgetUtilizationDemo.cs`, `ExpenseWorkflowDemo.cs`, `ResultPipelineDemo.cs`, `LookupDemo.cs`, `FileFailureDemo.cs` |
| Native C integration | `native/budget_native.c`, `NativeBudgetAdapter.cs` |

## Requirements implemented

The solution includes pure staged transformations, policy delegates, immutable domain values, validated construction, explicit state cases, `Result`/`Maybe`, two custom extension methods, two LINQ reports, deterministic tie-breakers, two partially applied scoring functions, a deferred `yield return` stream, CSV loading/export, and native C integration. The test runner covers domain and workflow cases, file failures, reports, lazy processing, resource release, and native success/failure. See the [group report](Group_Project_Report.md) for requirements traceability and test outcomes.

## Build, run, and test

### Prerequisites

- Windows x64
- .NET 8 SDK
- Visual Studio 2022 with the **Desktop development with C++** workload

From PowerShell in the folder containing `BudgetPipeline.sln`:

```powershell
.\build-native.ps1
dotnet build BudgetPipeline.sln -c Release
dotnet run -c Release --project BudgetPipeline.Tests
dotnet run -c Release --project BudgetPipeline
```

The native library is built from `BudgetPipeline/native/budget_native.c` and placed in `BudgetPipeline/runtimes/win-x64/native`. The project copies it beside the application and tests. Bundled CSV paths are resolved from the application directory.

To supply custom paths:

```powershell
dotnet run -c Release --project BudgetPipeline -- BudgetPipeline/data/transactions.csv BudgetPipeline/data/expense_requests.csv BudgetPipeline/data/category_spending_report.csv
```

To demonstrate missing and malformed input:

```powershell
dotnet run -c Release --project BudgetPipeline -- BudgetPipeline/data/transactions.csv missing.csv
dotnet run -c Release --project BudgetPipeline -- BudgetPipeline/data/transactions.csv BudgetPipeline/data/expense_requests_malformed.csv
```

The expected test result is `Tests: 50 passed, 0 failed.`

## Limitations

- The CSV parser does not support quoted values containing commas.
- Budget limits are configured in `Program.cs`.
- Native integration currently supports Windows x64 only.
- The application exports the category report but does not persist updated transaction data.

## Repository and submitted version

- **Repository URL:** <https://github.com/zhanbolat-tulepbergen/BudgetPipeline>
- **Branch:** `main`

The submission archive should include the solution, C# and C sources, native DLL/build script, sample CSV data, tests, this README, `Group_Project_Report.md`, and the submitting student's individual report. Exclude `bin/`, `obj/`, `.native-build/`, IDE caches, and credentials.
