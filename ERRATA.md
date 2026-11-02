# Errata — *Database Design for Analysts*

### Corrections and clarifications

This page lists known errors and inconsistencies in *Database Design for Analysts*, with the correction for each. Unless noted otherwise, each entry applies to both the chapter `README.md` files and `Database_Design_for_Analysts.pdf`.

If you find an error that is not listed here, please open an [issue](https://github.com/dolinger-nscc/Database-Design-for-Analysts/issues) or email Patrick.Dolinger@nscc.ca.

**Last updated:** November 2026

---

## Summary

| # | Location | Type | Issue |
|---|---|---|---|
| 1 | README — series links | Broken link | Links to the other two books do not resolve |
| 2 | Ch 1, §7.4 | Clarification | Geography tables are in the `Application` schema, not `Sales` |
| 3 | Ch 2, §10.3 | Error | `LoyaltyTransaction` references a data mart fact table and is missing `RewardID` |
| 4 | Ch 2, §10.4–10.5 | Inconsistency | Customer ↔ LoyaltyAccount participation disagrees with Ch 3, §12.2 |
| 5 | Ch 2, Review Question 3 | Error | The question's premise is not stated in §10 |
| 6 | Ch 3, §11.3 and later DDL | Inconsistency | Book DDL does not follow the book's own FK naming convention |
| 7 | Ch 8, §6.2 / Ch 9, §10 | Inconsistency | `DimDate` year column named two different ways |
| 8 | Ch 8, §12.1 | Error | `CabotTrailExecutive` has three tables, not four |
| 9 | Ch 9, §6.2 | Error | Data type table does not match the DDL in §9 |
| 10 | Ch 9, §7 | Inconsistency | Two listed constraints are not in the §9 DDL |
| 11 | Ch 9, §10–11 | Error | Period load and expected row counts disagree; year is hard-coded |
| 12 | Ch 9, §11.2 | Clarification | Reconciliation filters do not fully match the load filters |
| 13 | Ch 9, §12.1–12.2 | Error | `TRUNCATE` on the dimension tables fails |

---

## README

### 1. Links to the other books in the series do not resolve

**Location:** `README.md`, the list at the end ("This is the second book in a three-part series").

**Issue:** The links use relative paths (`../sql-for-analysts/README.md` and `../etl-for-business-intelligence/README.md`). On GitHub these resolve to branches named `sql-for-analysts` and `etl-for-business-intelligence`, which do not exist in this repository, so both links return a 404.

**Correction:** Replace each relative path with the full URL of that book's repository once it is published. Until then, show the titles as plain text without links.

---

## Chapter 1 — The Relational Model Revisited

### 2. Geography tables belong to the `Application` schema

**Location:** §7.4, "The CabotTrail OLTP Entity Map."

**Issue:** The entity map draws `Countries`, `StateProvinces`, and `Cities` under the **Sales domain**. In `CabotTrailOutdoor` these tables are in the **`Application`** schema, as the queries in Chapter 8, §4.1 show (`Application.Cities`, `Application.StateProvinces`, `Application.Countries`). Only `Sales.Customers.DeliveryCityID` is in the `Sales` schema.

**Correction:** Move `Countries`, `StateProvinces`, and `Cities` to the Application domain in the entity map, and show the link from `Application.Cities` to `Sales.Customers`. The domain grouping is a reading aid; the schema qualification is what you will use in queries.

---

## Chapter 2 — Entity-Relationship Diagrams

### 3. `LoyaltyTransaction` references a data mart fact table and is missing `RewardID`

**Location:** §10.3, the `LOYALTY_TRANSACTION` attribute list.

**Issue:** Two problems:

- The last attribute is `SalesLineID (FK → fact.Sales — nullable, for earn transactions)`. `fact.Sales` is in the data mart, not the OLTP. An operational table should not depend on an analytical table that is rebuilt by the ETL process, and SQL Server does not support foreign keys across databases. Chapter 3, §12.3 correctly uses `InvoiceID` referencing `Sales.Invoices` instead.
- §10.4 says a transaction *may redeem one Reward*, but the attribute list has no `RewardID` to implement that relationship.

**Correction:** Replace the last line of the `LOYALTY_TRANSACTION` attribute list with these two lines:

```
├── RewardID (FK → Reward — nullable, for redeem transactions)
└── InvoiceID (FK → Sales.Invoices — nullable, for earn transactions)
```

### 4. Customer ↔ LoyaltyAccount participation disagrees with Chapter 3

**Location:** §10.4 and §10.5, compared with Chapter 3, §12.2.

**Issue:** Chapter 2 says each customer has *exactly one* loyalty account and concludes "1:1, mandatory both sides." Chapter 3's DDL comment says the opposite: "FK on the optional side (not every customer has an account — yet)." The Chapter 3 design is the realistic one: when the programme launches, existing customers do not yet have accounts. It is also the design that the Chapter 3, §7 mapping rule (FK on the optional side, plus UNIQUE) produces.

**Correction:**

- §10.4: change "(1:1 — each customer has exactly one account)" to "(1:1 — each customer has at most one account)."
- §10.5: change the first question's answer and the result to:
  - "Can a customer exist without a loyalty account?" — yes: existing customers have no account until they join.
  - Result: **1:1, optional on the Customer side, mandatory on the LoyaltyAccount side.**

### 5. Review Question 3 relies on a statement §10 does not make

**Location:** §13, Review Question 3.

**Issue:** The question begins, "In the Loyalty Programme ERD from section 10, `LoyaltyTransaction` was identified as a weak entity." Section 10 never identifies it as weak. The weak-entity treatment first appears in Chapter 3, §12.3 ("Rule 4: Weak entity").

**Correction:** Replace the question with:

> 3. Is `LoyaltyTransaction` in the Loyalty Programme ERD from section 10 a weak entity? Explain your reasoning, and describe what its composite primary key would look like in the physical model if it were implemented without a surrogate key.

---

## Chapter 3 — From ERD to Tables

### 6. The book's DDL does not follow its own constraint naming convention

**Location:** §11.3 (the convention), compared with §12 and Chapter 9, §9 (the DDL).

**Issue:** §11.3 sets the foreign key pattern `FK_TableName_ReferencedTable`, and §11.1 asks for no abbreviations. Several constraints in the book's own scripts abbreviate the table names instead. Students following the convention will produce different names from the book.

**Correction:** In this book, the convention in §11.3 is the standard to follow. Where the scripts differ, use the names in the right-hand column:

| Name in the book | Name following §11.3 |
|---|---|
| `FK_LoyaltyAccounts_Cust` | `FK_LoyaltyAccounts_Customers` |
| `FK_LoyaltyAccounts_Tier` | `FK_LoyaltyAccounts_LoyaltyTiers` |
| `UQ_LoyaltyAccounts_Cust` | `UQ_LoyaltyAccounts_CustomerID` |
| `CK_LoyaltyAccounts_Bal` | `CK_LoyaltyAccounts_PointBalance` |
| `FK_LoyaltyTxn_Account` | `FK_LoyaltyTransactions_LoyaltyAccounts` |
| `FK_LoyaltyTxn_Reward` | `FK_LoyaltyTransactions_LoyaltyRewards` |
| `FK_LoyaltyTxn_Invoice` | `FK_LoyaltyTransactions_Invoices` |
| `CK_LoyaltyTxn_Type` | `CK_LoyaltyTransactions_TransactionType` |
| `CK_LoyaltyTxn_EarnPos` | `CK_LoyaltyTransactions_PointsAmount` |
| `FK_ExecSummary_Period` | `FK_ExecutiveSummary_Period` |
| `FK_ExecSummary_Category` | `FK_ExecutiveSummary_ProductCategory` |
| `CK_ExecSummary_*` | `CK_ExecutiveSummary_*` |

Abbreviated names still work in SQL Server. The issue is consistency, which is the whole point of a naming convention.

---

## Chapter 8 — OLTP vs OLAP

### 7. The `DimDate` year column is named two different ways

**Location:** Chapter 8, §6.2, compared with Chapter 9, §10.1 and §10.3.

**Issue:** Chapter 8, §6.2 lists the year column of `CabotTrailOutdoorDW.Dimension.DimDate` as `CalendarYear`. Chapter 9's population scripts use `d.YearNumber` for the same column. Only one of these can match the database.

**Correction:** Before running the Chapter 9 scripts, check the actual column names in your copy of the data warehouse:

```sql
SELECT COLUMN_NAME, DATA_TYPE
FROM   CabotTrailOutdoorDW.INFORMATION_SCHEMA.COLUMNS
WHERE  TABLE_SCHEMA = 'Dimension' AND TABLE_NAME = 'DimDate'
ORDER BY ORDINAL_POSITION;
```

Use the name the query returns in every script that reads `DimDate`. Both chapters will be updated to use the same name in the next revision.

### 8. `CabotTrailExecutive` has three tables, not four

**Location:** §12.1, the architecture diagram.

**Issue:** The diagram lists "`CabotTrailExecutive` — 4 tables." Chapter 9 designs three tables (`dim.Period`, `dim.ProductCategory`, `fact.ExecutiveSummary`) and one view (`fact.vw_ExecutiveSummaryYoY`).

**Correction:** Change the line to: "`CabotTrailExecutive` — 3 tables + 1 view — Executive summary (Chapter 9)."

---

## Chapter 9 — Designing CabotTrailExecutive

### 9. The data type table does not match the DDL

**Location:** §6.2, "Data Type Decisions," compared with §9.

**Issue:** §6.2 gives `NVARCHAR(20)` for `MonthName`, `QuarterLabel`, and `CategoryName`. The §9 DDL uses `NVARCHAR(10)`, `NVARCHAR(12)`, and `NVARCHAR(60)`, and adds `YearMonthLabel NVARCHAR(10)`, which §6.2 omits. The DDL is correct: several category names, such as 'Climbing and Rappelling', are longer than 20 characters and would fail to load into `NVARCHAR(20)`.

**Correction:** Replace the `MonthName, QuarterLabel, CategoryName` row of the §6.2 table with these rows:

| Column | Type | Rationale |
|---|---|---|
| MonthName | `NVARCHAR(10) NOT NULL` | Longest value, 'September', is 9 characters |
| YearMonthLabel | `NVARCHAR(10) NOT NULL` | Format 'Mar 2024' — 8 characters |
| QuarterLabel | `NVARCHAR(12) NOT NULL` | Format 'Q1 2024/25' — 10 characters |
| CategoryName | `NVARCHAR(60) NOT NULL` | Matches the length of category names in the DW |

### 10. Two constraints listed in §7 are not in the DDL

**Location:** §7, "Constraint Design," compared with §9.

**Issue:** §7 lists `CHECK: TotalGrossProfit >= 0` for `fact.ExecutiveSummary` and `CHECK: IsActive IN (0, 1)` for `dim.ProductCategory`. Neither appears in the §9 DDL.

**Correction:** Both omissions are deliberate design decisions; §7 should say so. Replace the two lines with:

- `TotalGrossProfit` — **no CHECK constraint.** Gross profit can legitimately be negative for a territory and category in a month (returns exceeding sales, or sales below cost), so a `>= 0` rule would reject valid data.
- `IsActive` — **no CHECK constraint.** The `BIT` data type already restricts the column to 0 and 1 (or NULL, which `NOT NULL` prevents), so a CHECK would add nothing.

### 11. The period load and the expected row counts disagree, and the year is hard-coded

**Location:** §10.1, §10.3, §11.1, §11.2, and §5.2.

**Issue:**

- §10.1 loads `dim.Period` for `YearNumber BETWEEN 2022 AND YEAR(GETDATE())`. Run in 2026, that loads five years, or 60 periods.
- §10.3 loads facts only for `YearNumber < 2026`, so the 12 periods for 2026 have no summary rows.
- §11.1 expects 48 periods ("12 months × 4 years"), and §5.2 refers to "a reference table with 48 rows." The expected count is only right if the scripts run during 2025.
- The year `2026` is hard-coded in §10.3 and §11.2. Because the database is refreshed monthly, the scripts produce different results, with no warning, once the calendar year changes.

**Correction:** Use one cutoff value for every load and for the reconciliation, so the period dimension and the fact table always cover the same range. Add this at the top of the population script:

```sql
-- Load complete calendar years only: everything before the current year
DECLARE @FirstYear SMALLINT = 2022;
DECLARE @LastYear  SMALLINT = YEAR(GETDATE()) - 1;
```

Then change:

- §10.1: `WHERE d.YearNumber BETWEEN @FirstYear AND @LastYear`
- §10.3: `AND d.YearNumber BETWEEN @FirstYear AND @LastYear` (replacing `d.YearNumber < 2026`)
- §11.2: filter the source total on the same years (replacing `OrderDateKey / 10000 < 2026`)
- §11.1: expected `dim.Period` rows = 12 × (`@LastYear` − `@FirstYear` + 1); maximum `fact.ExecutiveSummary` rows = periods × 13 × 10
- §5.2: "Acceptable for a small reference table (12 rows per year of history)."

All variables must be declared in the same batch as the statements that use them, so do not put a `GO` between the declarations and the loads.

### 12. The reconciliation filters do not fully match the load filters

**Location:** §11.2, compared with §10.3.

**Issue:** The load in §10.3 excludes rows through its joins: products whose category is the unknown member (`dpc.ProductCategoryID <> 0`) and customers that are not the current version (`cust.IsCurrent = 1`). The reconciliation query in §11.2 filters only on `CustomerKey`, `ProductKey`, and the year, so it can include sales the load correctly left out. When that happens, the totals differ even though the load is correct — yet the text says both values "must be equal."

**Correction:** Add this paragraph after the §11.2 query:

> A reconciliation is only valid if the source query applies **exactly the same filters and joins** as the load. If the totals differ, first check whether the difference is made up of rows the load excluded on purpose — for example, products with an unknown category. Review Question 4 asks you to diagnose exactly this kind of mismatch.

### 13. `TRUNCATE` on the dimension tables fails

**Location:** §12.1 (step 1, "Truncate all three tables") and §12.2 (the SSIS package outline).

**Issue:** SQL Server does not allow `TRUNCATE TABLE` on a table that is referenced by a foreign key, even when the referencing table is empty. Because `fact.ExecutiveSummary` has foreign keys to both dimension tables, these statements fail with error 4712 ("Cannot truncate table … because it is being referenced by a FOREIGN KEY constraint"):

```sql
TRUNCATE TABLE dim.Period;
TRUNCATE TABLE dim.ProductCategory;
```

**Correction:** Clear the fact table with `TRUNCATE`, then clear the dimensions with `DELETE` (children before parents) and reset their IDENTITY values:

```sql
TRUNCATE TABLE fact.ExecutiveSummary;    -- allowed: no table references the fact table

DELETE FROM dim.Period;
DELETE FROM dim.ProductCategory;

DBCC CHECKIDENT ('dim.Period', RESEED, 0);           -- next PeriodKey will be 1
DBCC CHECKIDENT ('dim.ProductCategory', RESEED, 0);  -- next CategoryKey will be 1
```

Update the three `TRUNCATE` steps in §12.2 the same way: `TRUNCATE fact.ExecutiveSummary` first, then `DELETE` from `dim.Period` and `dim.ProductCategory`.

---

*Database Design for Analysts* © Patrick Dolinger, NSCC Institute of Technology. Licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).
