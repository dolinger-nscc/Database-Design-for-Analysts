# Changelog — *Database Design for Analysts*

## Revision 1.1 — November 2026

This revision corrects every item in the November 2026 errata, plus related issues found while applying the corrections. Changes apply to the chapter READMEs and to `Database_Design_for_Analysts.pdf`, which was regenerated from the corrected chapters.

Every SQL block that changed was checked by parsing it as T-SQL. The only blocks that don't parse are ones that didn't parse before either, because they are deliberate fragments (a lone `CONSTRAINT` clause, for example) or use `DBCC`, which the parser doesn't support.

### README

- **Series links:** the links to *SQL for Analysts* and *ETL for Business Intelligence* pointed to branches that don't exist. The titles are now plain text until those books are published.

### Chapter 1 — The Relational Model Revisited

- **§7.4 Entity map:** `Countries`, `StateProvinces`, and `Cities` moved from the Sales domain to the Application domain, where they live in the database, with a note on how `Sales.Customers` references them.

### Chapter 2 — Entity-Relationship Diagrams

- **§10.3:** `LoyaltyTransaction` no longer references the data mart's `fact.Sales`. It now has `RewardID` (for redeem transactions) and `InvoiceID` referencing `Sales.Invoices` (for earn transactions), matching Chapter 3.
- **§10.4–10.5:** Customer ↔ LoyaltyAccount is now *optional on the Customer side, mandatory on the LoyaltyAccount side*, matching the Chapter 3 design. Existing customers have no account until they join.
- **Review Question 3:** reworded so it no longer claims §10 identified `LoyaltyTransaction` as a weak entity.

### Chapter 3 — From ERD to Tables, and Chapter 9

- **Constraint names follow §11.3.** Every constraint and index in the two complete scripts students build — the Loyalty Programme (Chapter 3, §12) and `CabotTrailExecutive` (Chapter 9, §9) — now uses the full table and column names. Short illustrative fragments elsewhere in the book are unchanged.

| Was | Now |
|---|---|
| `FK_LoyaltyAccounts_Cust` | `FK_LoyaltyAccounts_Customers` |
| `FK_LoyaltyAccounts_Tier` | `FK_LoyaltyAccounts_LoyaltyTiers` |
| `UQ_LoyaltyAccounts_Cust` | `UQ_LoyaltyAccounts_CustomerID` |
| `CK_LoyaltyAccounts_Bal` | `CK_LoyaltyAccounts_PointBalance` |
| `FK_LoyaltyTxn_Account` | `FK_LoyaltyTransactions_LoyaltyAccounts` |
| `FK_LoyaltyTxn_Reward` | `FK_LoyaltyTransactions_LoyaltyRewards` |
| `FK_LoyaltyTxn_Invoice` | `FK_LoyaltyTransactions_Invoices` |
| `CK_LoyaltyTxn_Type` | `CK_LoyaltyTransactions_TransactionType` |
| `CK_LoyaltyTxn_EarnPos` (Ch 3) and `CK_LoyaltyTxn_PointsDirection` (Ch 6) | `CK_LoyaltyTransactions_PointsDirection` (both chapters) |
| `UQ_LoyaltyTiers_Name` | `UQ_LoyaltyTiers_TierName` |
| `CK_LoyaltyTiers_MinPts` | `CK_LoyaltyTiers_MinimumPoints` |
| `CK_LoyaltyRewards_Cost` | `CK_LoyaltyRewards_PointCost` |
| `UQ_Period_CalYearMonth` | `UQ_Period_CalendarYearMonth` |
| `CK_Period_MonthNum` | `CK_Period_MonthNumber` |
| `CK_Period_CalYear` | `CK_Period_CalendarYear` |
| `CK_Period_FiscalQtr` | `CK_Period_FiscalQuarterNumber` |
| `UQ_ProductCategory_Name` | `UQ_ProductCategory_CategoryName` |
| `CK_ProductCategory_Margin` | `CK_ProductCategory_MarginTier` |
| `FK_ExecSummary_Period` | `FK_ExecutiveSummary_Period` |
| `FK_ExecSummary_Category` | `FK_ExecutiveSummary_ProductCategory` |
| `CK_ExecSummary_Revenue` | `CK_ExecutiveSummary_TotalRevenue` |
| `CK_ExecSummary_InvoiceCount`, `_CustomerCount`, `_SalesLineCount` | `CK_ExecutiveSummary_InvoiceCount`, `_CustomerCount`, `_SalesLineCount` |
| `IX_ExecSummary_Period`, `IX_ExecSummary_Territory` | `IX_ExecutiveSummary_Period`, `IX_ExecutiveSummary_Territory` |

The earn/redeem rule had two different names in Chapters 3 and 6; both now use `CK_LoyaltyTransactions_PointsDirection`, the name Chapter 6 holds up as self-documenting.

### Chapter 6 — Constraints and Data Integrity

- **§8.3:** the comment "An employee has at most one loyalty account" now reads "A customer has at most one loyalty account."

### Chapter 8 — OLTP vs OLAP

- **§6.2:** added a note on checking the `Dimension.DimDate` column names (see "Still open," below).
- **§12.1:** `CabotTrailExecutive` is now "3 tables + 1 view," not "4 tables."

### Chapter 9 — Designing CabotTrailExecutive

- **§6.2 Data types:** the table now matches the DDL — `MonthName NVARCHAR(10)`, `YearMonthLabel NVARCHAR(10)`, `QuarterLabel NVARCHAR(12)`, `CategoryName NVARCHAR(60)`.
- **§7 Constraints:** explains why there is deliberately no CHECK on `TotalGrossProfit` (it can legitimately be negative) or `IsActive` (`BIT` already limits it to 0 and 1).
- **Load window (§2, §5.2, §9, §10, §11):** every load and the reconciliation now share one window, *January 2022 through the last complete month*, defined by `@FirstDateKey` and `@LastDateKey`. This replaces the hard-coded year 2026 and the mismatch between the period load (through the current year) and the fact load (before 2026).
  - *This differs from the errata's suggested code,* which loaded complete calendar years only. Loading through the last complete month matches the requirements in §2 — history "through the current month" with a monthly refresh — so each monthly load adds the month just finished.
  - The filters use `DateKey`, which every chapter agrees on.
  - `IsCurrentMonth` now flags the most recent complete month in the window; under the old logic it would never have matched a loaded month.
  - §11.1's expected row counts now depend on the load date (for example, 58 periods when loaded on 1 November 2026).
- **§10.1:** the opening comment said the periods load "from the DW fact table"; it loads from the DW date dimension.
- **§11.2 Reconciliation:** uses the same window as the load, followed by a new paragraph explaining that a reconciliation is only valid when its filters and joins match the load.
- **§12.1–12.2 Monthly load:** the load no longer truncates the dimension tables, which fails because the fact table references them. It now truncates the fact table, then deletes from both dimensions and reseeds their IDENTITY values, with an explanation and the script.

### PDF

`Database_Design_for_Analysts.pdf` was regenerated with WeasyPrint from the corrected chapters, in the original layout (207 pages). Lists that previously printed with literal hyphens now render as bulleted lists, matching how GitHub displays the chapters.

### Still open

- **`DimDate` year column:** Chapter 8, §6.2 lists it as `CalendarYear`; the Chapter 9 scripts use `YearNumber`. Both chapters now tell readers how to check their own warehouse. A future revision will settle on the name used in `CabotTrailOutdoorDW`.
