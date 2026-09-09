# 483553A — Data Analyst: Securitisation AI Agent
### Zetheta WorkBridge Platform | Project 1A | 15-Day Analytics Project

---

## Project Overview

This repository contains all deliverables for the **Securitisation AI Agent** project, a 15-day enterprise-scale financial analytics project that builds sophisticated risk assessment dashboards for a securitised auto loan portfolio using advanced DAX in Microsoft Power BI.

The project transforms raw loan-level data from an assumed SAP/ERP system into actionable risk intelligence covering:

- IFRS 9 Expected Credit Loss (ECL) modelling
- Delinquency and roll rate (transition matrix) analysis
- Vintage / static pool cumulative loss curve analysis
- Cash flow waterfall and structural trigger monitoring
- Investor-grade monthly servicer reporting
- Concentration and geographic risk analysis

**Pool in scope:** Auto Loan ABS Pool `ZAAUTO2024-1` | Cutoff: 2024-10-31 | 500 loans | ₹31.8 crore outstanding

---

## Repository Structure

```
483553A_YourName/
│
├── README.md                          ← This file
│
├── data/
│   ├── raw/                           ← Original source files (as received)
│   │   ├── auto_loan_securitisation_data.csv
│   │   ├── dpd_snapshot_history.csv
│   │   ├── dynamic_loss_monthly.csv
│   │   └── static_pool_vintage_data.csv
│   │
│   └── cleaned/                       ← Power BI-ready cleaned tables
│       ├── DimDate.csv
│       ├── DimLoan.csv
│       ├── FactLoanMaster.csv
│       ├── FactLoanSnapshot.csv
│       ├── FactDynamicLoss.csv
│       └── FactStaticPool.csv
│
├── notebooks/
│   └── Securitisation_Data_Cleaning_Analytics.ipynb   ← Full data cleaning + DAX crosscheck
│
├── powerbi/
│   └── 483553A_YourName_PowerBIModel.pbix             ← 8-page Power BI dashboard
│
├── excel/
│   ├── DAX_Measure_Dictionary.xlsx                    ← All 40+ DAX measures documented
│   └── Excel_Crosscheck_Workbook.xlsx                 ← ECL + waterfall validation
│
├── report/
│   └── 483553A_YourName_MainReport.pdf                ← Full project report (Parts A–F)
│
└── presentation/
    └── 483553A_YourName_Presentation.pdf              ← 10-slide final presentation
```

---

## Dashboard Pages

| Page | Title | Key Visuals |
|------|-------|-------------|
| 1 | Pool Overview | KPI cards (balance, pool factor, WAC, WALA), delinquency status bar, region donut |
| 2 | Delinquency / DPD | 30+/60+/90+ DPD KPIs, 12-month trend line, roll rate heatmap matrix |
| 3 | Concentration | Top 10 borrower bar, HHI index, geographic state bar, vehicle make breakdown |
| 4 | Vintage Analysis | Cumulative loss curves (15 vintages), seasoning comparison table, pool factor chart |
| 5 | IFRS 9 ECL | Stage 1/2/3 balances, ECL provision, coverage ratio, crosscheck table |
| 6 | Waterfall & Triggers | Monthly cash flow waterfall, structural trigger status (PASS/FAIL) |
| 7 | Investor Reporting | Monthly servicer report table, CDR/CPR/loss rate trend lines |
| 8 | Executive Summary | Headline KPIs, business insight narrative, page navigation |

---

## Data Model — Star Schema

```
DimDate ──────────────────┐
                           ├──▶ FactLoanSnapshot (6,000 rows — 500 loans × 12 months)
DimLoan ──────────────────┤
  │                        └──▶ FactLoanMaster   (500 rows — cutoff snapshot)
  │
  └── VintageID ──────────────▶ FactStaticPool   (375 rows — 15 vintages × MOB)

DimDate ──────────────────────▶ FactDynamicLoss  (12 rows — monthly pool performance)
```

**Relationships:**
- `DimLoan[LoanID]` → `FactLoanMaster[LoanID]` (1:1, active)
- `DimLoan[LoanID]` → `FactLoanSnapshot[LoanID]` (1:many, active)
- `DimDate[Date]` → `FactLoanSnapshot[SnapshotDate]` (1:many, active)
- `DimDate[Date]` → `FactDynamicLoss[ReportingDate]` (1:many, active)
- `DimDate[Date]` → `FactLoanMaster[CutoffDate]` (1:many, **inactive** — used via USERELATIONSHIP)
- `DimLoan[VintageID]` → `FactStaticPool[VintageID]` (1:many, active)

---

## Key DAX Measures

A full measure dictionary with formulas and business purposes is in `excel/DAX_Measure_Dictionary.xlsx`. Selected highlights:

```dax
-- Pool Factor
Pool Factor =
DIVIDE ( SUM ( FactLoanMaster[CurrentBalance] ), SUM ( FactLoanMaster[OriginalLoanAmount] ) )

-- Roll Rate Matrix (transition probability)
Roll Rate % =
VAR RowBucket = SELECTEDVALUE ( FactLoanSnapshot[DPD_Bucket_Prior] )
VAR RowTotal =
    SUMX (
        FILTER ( ALL ( FactLoanSnapshot ), FactLoanSnapshot[DPD_Bucket_Prior] = RowBucket ),
        FactLoanSnapshot[CurrentBalance]
    )
RETURN DIVIDE ( SUM ( FactLoanSnapshot[CurrentBalance] ), RowTotal, 0 )

-- IFRS 9 ECL Crosscheck
Computed ECL (PD x LGD x EAD) =
SUMX ( FactLoanMaster, FactLoanMaster[PD_Estimate] * FactLoanMaster[LGD_Estimate] * FactLoanMaster[EAD] )

-- HHI Index (Borrower Concentration)
HHI Index (Borrower) =
VAR TotalBal = CALCULATE ( SUM ( FactLoanMaster[CurrentBalance] ), ALL ( FactLoanMaster ), ALL ( DimLoan ) )
VAR Shares =
    ADDCOLUMNS (
        SUMMARIZE ( FactLoanMaster, DimLoan[BorrowerID] ),
        "Share", DIVIDE ( CALCULATE ( SUM ( FactLoanMaster[CurrentBalance] ) ), TotalBal )
    )
RETURN SUMX ( Shares, [Share] ^ 2 )
```

---

## Key Findings

| Metric | Value | Assessment |
|--------|-------|------------|
| Total Outstanding Balance | ₹31.8 crore | — |
| Pool Factor | ~70% | Pool well-seasoned |
| 30+ DPD Rate | **14.11%** | ⚠ Exceeds 5% trigger threshold |
| 60+ DPD Rate | **10.59%** | ⚠ Serious delinquency elevated |
| NPA Rate (90+ DPD) | **5.57%** | ⚠ Above typical ABS benchmark |
| HHI Index (Borrower) | 0.0033 | ✅ Highly diversified pool |
| Top 10 Borrower Concentration | 7.37% | ✅ No single-name risk |
| ECL Coverage Ratio | ~X% | Computed vs provided: reconciled |
| Delinquency Structural Trigger | ≤ 5.0% | ❌ **BREACH — 14.11% current** |

**Key analytical insight:** The delinquency rate of 14.11% materially exceeds the structural trigger threshold of 5.0%, which under standard securitisation transaction documents would activate sequential pay-down acceleration — diverting collections from mezzanine and equity tranches to accelerate senior tranche repayment. The roll rate matrix confirms the deterioration is structural rather than transient: the 60-89 DPD → 90-119 DPD roll-to-default rate of 32.76% indicates ongoing credit deterioration requiring close monitoring.

---

## Data Quality Notes

| Issue | Detail |
|-------|--------|
| Duplicate row in `dynamic_loss_monthly.csv` | Two rows share `ReportingDate = 2024-05-30`. De-duplicated in Power Query (first occurrence retained). |
| No tranche/waterfall data provided | Senior/Mezzanine/Equity structure is **assumed** per project brief's allowance for "assumed bank data." Assumptions documented in `Excel_Crosscheck_Workbook.xlsx`. |
| No prior-month IFRS9_Stage column | Stage migration measures use `Times30/60/90DPD_Last12M` as a proxy indicator. Documented as an approximation. |
| Recent vintage CNL at 24M blanks | 2023-Q4 and 2024-Qx vintages are less than 24 months old at cutoff — blank 24M data points are correct behavior, not missing data. |

---

## Technology Stack

| Tool | Purpose |
|------|---------|
| Microsoft Power BI Desktop | Dashboard development, DAX measure authoring |
| Python (Jupyter Notebook) | Data cleaning, validation, DAX crosscheck |
| pandas / numpy / matplotlib | Data manipulation and chart generation |
| DAX Studio | Query profiling, performance testing |
| Microsoft Excel | ECL crosscheck, waterfall validation, measure dictionary |
| GitHub (PRIVATE) | Version control, submission |

---

## ECL Crosscheck Validation

IFRS 9 ECL was independently computed as `PD × LGD × EAD` in both Python (Jupyter Notebook, Section 10) and DAX (`Computed ECL` measure), then compared against the provided `ECL_Provision` column.

**Result:** Aggregate variance < 0.01% — **RECONCILED.**

Full loan-level crosscheck is in `excel/Excel_Crosscheck_Workbook.xlsx` → sheet `ECL_Crosscheck`.

---

## Roll Rate Matrix — Python Reference

Independently computed in Jupyter Notebook (Section 8) and validated against Power BI DAX output:

| From ↓ / To → | Current | 1-29 DPD | 30-59 DPD | 60-89 DPD | 90-119 DPD | 120+ DPD | Default | Repossessed |
|---|---|---|---|---|---|---|---|---|
| **Current** | 93.41% | 5.60% | 0.57% | 0.39% | 0.03% | — | — | — |
| **1-29 DPD** | 54.29% | 42.13% | 3.58% | — | — | — | — | — |
| **30-59 DPD** | — | — | 51.68% | 48.32% | — | — | — | — |
| **60-89 DPD** | — | — | — | 58.51% | 32.76% | 8.73% | — | — |
| **90-119 DPD** | — | — | — | — | 78.18% | 9.87% | 9.23% | 2.72% |
| **120+ DPD** | — | — | — | — | — | 83.82% | 5.81% | 10.37% |
| **Default** | — | — | — | — | — | — | 100.00% | — |

---

## Regulatory & Methodology References

- RBI Master Direction on Securitisation of Standard Assets (RBI/2021-22/85, September 2021)
- RBI IRAC Norms — Income Recognition, Asset Classification, and Provisioning
- RBI SMA Classification Guidelines (November 2021 revised framework)
- SEBI (Issue and Listing of Securitised Debt Instruments) Regulations, 2008
- S&P Global LEVELS Model — Auto ABS Criteria
- Moody's Idealised Default Rate Tables
- CRISIL Indian Securitisation Rating Criteria
- IFRS 9 / Ind AS 109 — Financial Instruments: Expected Credit Loss

---

## Submission Details

| Item | Detail |
|------|--------|
| Project Code | 483553A |
| Platform | Zetheta WorkBridge |
| Submission deadline | Day 15–30 |
| Repository transfer | Settings → Transfer → @ZethetaIntern |
| File naming convention | `483553A_YourName_DeliverableName` |

---

## Final Commit Message Format

```
Final submission – Securitisation AI Agent v1.0 — [Your Email] — [Date]
```

```bash
git tag -a v1.0 -m "Securitisation AI Agent v1.0 Final Submission"
```

---

*Strictly Private and Confidential — Submitted to Zetheta Algorithms Private Limited*
*All IP arising from this project is the exclusive property of Zetheta Algorithms Pvt. Ltd.*
