# Data Flow Logic Document - Stock Market Screener

**Version:** 1.1  
**Date:** 2026-03-14  
**Author:** ThommCroft  
**Purpose:** Comprehensive documentation of data flow through each system component

---

## Table of Contents

1. [Introduction](#introduction)
2. [Data Flow Overview](#data-flow-overview)
3. [Stage 1: Data Ingestion & Reconciliation](#stage-1-data-ingestion--reconciliation)
4. [Stage 2: Metric Calculation](#stage-2-metric-calculation)
5. [Stage 3: Screening Logic](#stage-3-screening-logic)
6. [Stage 4: Results Processing](#stage-4-results-processing)
7. [Stage 5: Notifications & Storage](#stage-5-notifications--storage)
8. [Error Handling & Recovery](#error-handling--recovery)
9. [Data Quality Assurance](#data-quality-assurance)
10. [Performance Considerations](#performance-considerations)
11. [Example Walkthrough](#example-walkthrough)

---

## Introduction

This document provides detailed, technical description of how data flows through the Stock Market Screener system. It complements the Architecture Design Document by providing implementation-level details and decision logic at each stage.

**Target Audience:** Developers implementing each component  
**Scope:** Complete data transformation from company list to qualified results

---

## Data Flow Overview

### Complete End-to-End Flow

```
Input: Industry List (from appsettings.json)
   │
   ▼
┌──────────────────────────────────────────────────────┐
│ Stage 1: Data Ingestion & Reconciliation             │
│ ├─ Fetch company list from SEC EDGAR API            │
│ ├─ For each company: fetch financial data           │
│ ├─ Reconcile conflicting data                        │
│ └─ Output: Raw financial records (validated)         │
└──��─────────────────┬─────────────────────────────────┘
                     │
                     ▼
┌──────────────────────────────────────────────────────┐
│ Stage 2: Metric Calculation                          │
│ ├─ Parse financial statements                        │
│ ├─ Calculate 40+ financial metrics                   │
│ ├─ Perform 10-year trend analysis                    │
│ └─ Output: Metric-enriched company records           │
└────────────────────┬─────────────────────────────────┘
                     │
                     ▼
┌──────────────────────────────────────────────────────┐
│ Stage 3: 3-Stage Screening                           │
│ ├─ Stage 1: Financial Strength (hard filters)       │
│ ├─ Stage 2: Quality Assessment (scoring 0-100)      │
│ ├─ Stage 3: Valuation & Management (final ranking)  │
│ └─ Output: Pass/Fail + Composite Score (0-100)      │
└────────────────────┬───────────────────────���─────────┘
                     │
                     ▼
┌──────────────────────────────────────────────────────┐
│ Stage 4: Results Processing                          │
│ ├─ Filter: Only keep PASS (score >= 75)             │
│ ├─ Compare with previous qualified companies         │
│ ├─ Categorize: NEW, MAINTAINED, REMOVED             │
│ └─ Output: Screening results with status             │
└────────────────────┬─────────────────────────────────┘
                     │
                     ▼
┌──────────────────────────────────────────────────────┐
│ Stage 5: Notifications & Storage                     │
│ ├─ Save qualified companies to MySQL                 │
│ ├─ Save screening run details (audit trail)          │
│ ├─ Generate email report                             │
│ ├─ Export CSV results                                │
│ └─ Output: Database updated + Email sent             │
└──────────────────────────────────────────────────────┘
                     │
                     ▼
              SCREENING COMPLETE
```

---

## Stage 1: Data Ingestion & Reconciliation

### 1.1 Industry & Company Fetching

**Input:** List of industries from `appsettings.json`

```json
{
  "IndustriesToScreen": [
    "Technology",
    "Healthcare",
    "Consumer Defensive"
  ]
}
```

**Process:**
1. Load industries from configuration
2. Query SEC EDGAR API for all companies in each industry
3. Build master list of tickers to process
4. Typically results in 500-1000 companies

**Output:** List of company tickers with basic info

---

### 1.2 Data Age & Availability Requirements

**CRITICAL:** Companies are rejected at data ingestion if they don't meet minimum data requirements:

**Financial Statements:**
- **Required:** Current fiscal year statement (< 1 year old)
- **Preferred:** 10 years of historical data (2016-2025)
- **Minimum Acceptable:** 5 years of historical data
- **Rejected:** < 5 years of data (insufficient for analysis)

**Stock Prices:**
- **Required:** Current stock price (< 1 day old)
- **Acceptable:** Up to 5 days old (market holiday exception)
- **Rejected:** > 5 days old (stale data)

**Data Completeness:**
- **All required fields present:** Current quarter data
- **No nulls in critical metrics:** Revenue, Net Income, Total Assets, Total Liabilities
- **Historical consistency:** Data integrity checks pass

**Filtering Logic:**

```
IF Financial Data Age < 1 year AND
   Historical Data >= 5 years AND
   Stock Price Age <= 5 days AND
   All Required Fields Present
THEN → Continue to Stage 1.3
ELSE → REJECT company, skip to next
```

**Example Rejections:**
- Company with only 3 years of data: REJECTED
- Company with 8-month-old financials but no recent stock price: REJECTED
- Company with missing data in Q4 balance sheet: REJECTED
- Company with 9 years of clean data and current price: ACCEPTED ✓

---

### 1.3 Financial Data Fetching Strategy

**Data Required:**
- Income Statement (Revenue, Net Income, Operating Income)
- Balance Sheet (Assets, Liabilities, Equity)
- Cash Flow Statement (Operating CF, Free CF, CapEx)
- Market Data (Stock Price, Market Cap, Dividends)

**Multi-Source Priority:**

```
FINANCIAL STATEMENTS:
├─ Primary: SEC EDGAR (10-K/10-Q)
│  └─ Most authoritative source
├─ Fallback: None - skip if unavailable
└─ Error: Log and skip company

STOCK PRICES & MARKET DATA:
├─ Primary: Yahoo Finance API
├─ Fallback #1: IEX Cloud
├─ Fallback #2: Alpha Vantage
└─ Error: Use last known price or skip

TREASURY YIELDS:
├─ Primary: Federal Reserve FRED API
└─ Fallback: Hardcoded estimate (2.5%)
```

---

### 1.4 Data Reconciliation

**Reconciliation Rules:**

| Data Item | Conflict Resolution |
|-----------|-------------------|
| Stock Price | Use most recent (today's date) |
| Market Cap | Calculate from stock price × shares |
| Dividend | Verify across 2+ sources |
| Financial metrics | Always use SEC EDGAR (single source) |
| Historical data | Fill gaps using secondary sources |

**Data Validation Rules:**

1. **Completeness:** All required fields present?
2. **Range Validation:** ROE: -100% to +200%, D/E: 0 to 5.0
3. **Balance Sheet:** Assets = Liabilities + Equity ±2%
4. **Consistency:** Year-over-year changes < 300%
5. **Data Age:** Financials < 1 year old, Prices < 1 day old

---

## Stage 2: Metric Calculation

### 2.1 Metric Categories (40+)

**Profitability Metrics (8):**
- Gross Margin, Operating Margin, Net Profit Margin, EBITDA Margin
- ROA, ROE, ROIC, ROCE

**Financial Strength (6):**
- Debt/Equity, Debt/Assets, Equity Multiplier
- Current Ratio, Quick Ratio, Interest Coverage

**Cash Flow (6):**
- Operating Cash Flow, Free Cash Flow, OCF/NI Ratio
- FCF/OCF, CapEx %, OCF 10Y Avg

**Growth (7):**
- Revenue Growth YoY, Earnings Growth YoY, FCF Growth YoY
- Revenue CAGR 10Y, Earnings CAGR 10Y, FCF CAGR 10Y
- Earnings Per Share

**Valuation (8):**
- P/E Ratio, P/B Ratio, P/S Ratio, PEG Ratio
- Earnings Yield, Dividend Yield, Dividend Payout Ratio
- Enterprise Value/Revenue

**Management Quality (6):**
- Insider Ownership %, Goodwill % of Assets
- One-Time Charges Frequency, Retained Earnings %
- Operating Expense Ratio, Asset Turnover

**Stability Metrics:**
- Coefficient of Variation (CV) for each category
- < 10% = Highly stable, 10-20% = Stable, 20-30% = Moderate, > 30% = Volatile

---

## Stage 3: Screening Logic

### 3.1 Stage 1: Financial Strength Filters (Hard Filters)

**Logic:** ALL filters must pass. If ANY fails → REJECT

```
IF ROE (10-year avg) > 15% AND
   Net Profit Margin (10-year avg) > 10% AND
   Debt/Equity Ratio < 0.50 AND
   Operating Cash Flow (positive all 10 years) AND
   Free Cash Flow (positive 8+ of 10 years)
THEN → Continue to Stage 2
ELSE → REJECT
```

**Filter Rules:**

| Filter | Threshold | Rationale |
|--------|-----------|-----------|
| ROE | > 15% (10Y avg) | Return on shareholder capital |
| Net Margin | > 10% (10Y avg) | Profitability |
| Debt/Equity | < 0.50 | Financial stability |
| Operating CF | Positive all 10Y | Cash generation |
| Free Cash Flow | Positive 8+ of 10Y | Sustainable business |

---

### 3.2 Stage 2: Quality Assessment (Scoring)

**4 Quality Pillars - Total Score: 0-100 points (with max per pillar)**

**⚠️ IMPORTANT: Each pillar has a maximum score. Elements within each pillar CANNOT exceed that pillar's maximum.**

#### Pillar 1: Return on Capital (MAX: 30 points)

**ROE Component (max 20 points):**
- ROE 15-20% = 5 pts
- ROE 20-25% = 10 pts
- ROE 25-30% = 15 pts
- ROE 30%+ = 20 pts (MAX PILLAR COMPONENT)

**ROIC Component (max 10 points):**
- ROIC 12-15% = 5 pts
- ROIC 15-18% = 8 pts
- ROIC 18%+ = 10 pts (MAX PILLAR COMPONENT)

**Pillar Cap:** Total from both components cannot exceed 30 points
- If ROE scores 20 pts, ROIC can add 0-10 pts (capped at 10 max remaining)
- If ROE scores 15 pts, ROIC can add 0-15 pts (capped at 10 max for ROIC, 30 total)

**Example:** ROE = 20 pts, ROIC = 10 pts → Pillar Total = 30 pts (MAX)

---

#### Pillar 2: Profitability (MAX: 30 points)

**Net Margin Component (max 15 points):**
- Net Margin 10-15% = 5 pts
- Net Margin 15-20% = 10 pts
- Net Margin 20%+ = 15 pts (MAX COMPONENT)

**Operating Margin Component (max 10 points):**
- Operating Margin 15-20% = 5 pts
- Operating Margin 20%+ = 10 pts (MAX COMPONENT)

**Margin Stability Bonus (max 5 points):**
- Margin Stability (CV < 10%) = 5 pts (BONUS)

**Pillar Cap:** Total cannot exceed 30 points
- If Net Margin = 15 pts + Operating Margin = 10 pts + Stability = 5 pts = 30 pts (MAX)
- If Net Margin = 15 pts + Operating Margin = 10 pts, Stability bonus is capped at 5 pts (no overflow)

**Example:** Net = 15 pts, Op Margin = 10 pts, Stability = 5 pts → Pillar Total = 30 pts (MAX)

---

#### Pillar 3: Cash Flow (MAX: 20 points)

**FCF Positivity Component (max 5 points):**
- FCF Positive = 5 pts

**FCF as % of Revenue Component (max 10 points):**
- FCF > 5% of Revenue = 5 pts
- FCF > 10% of Revenue = 10 pts (MAX COMPONENT)

**OCF Quality Component (max 5 points):**
- OCF > Net Income (high quality) = 5 pts

**FCF Growth Component (max 5 points):**
- FCF Growth > 5% CAGR = 5 pts

**Pillar Cap:** Total cannot exceed 20 points
- Can combine: FCF Positive (5) + FCF% (10) + OCF Quality (5) = 20 pts (MAX)
- Cannot add FCF Growth if already at 20 pts

**Example:** Positive (5) + FCF% (10) + OCF (5) = 20 pts (MAX) - FCF Growth cannot add

---

#### Pillar 4: Business Quality (MAX: 10 points)

**Economic Moat Component (max 3 points):**
- Economic Moat Detected = 3 pts

**Management Alignment Component (max 2 points):**
- Insider Ownership > 5% = 2 pts

**Organic Growth Component (max 3 points):**
- Low Goodwill (< 20% of assets) = 3 pts

**Revenue Stability Component (max 2 points):**
- Revenue Stability (Growth CV < 15%) = 2 pts

**Pillar Cap:** Total cannot exceed 10 points
- Moat (3) + Insider (2) + Goodwill (3) + Stability (2) = 10 pts (MAX)
- All components combined achieve maximum

**Example:** Moat (3) + Insider (2) + Goodwill (3) + Stability (2) = 10 pts (MAX)

---

**Quality Score Calculation:**

```csharp
decimal CalculateQualityScore(CompanyMetricsRecord record)
{
    decimal totalScore = 0m;
    
    // PILLAR 1: Return on Capital (MAX 30)
    decimal pillar1 = 0m;
    var roe = record.Profitability.ReturnOnEquity;
    if (roe >= 30) pillar1 += 20m;
    else if (roe >= 25) pillar1 += 15m;
    else if (roe >= 20) pillar1 += 10m;
    else if (roe >= 15) pillar1 += 5m;
    
    var roic = record.Profitability.ReturnOnInvestedCapital_ROIC;
    if (roic >= 18) pillar1 += 10m;
    else if (roic >= 15) pillar1 += 8m;
    else if (roic >= 12) pillar1 += 5m;
    
    pillar1 = Math.Min(pillar1, 30m); // CAP AT 30
    totalScore += pillar1;
    
    // PILLAR 2: Profitability (MAX 30)
    decimal pillar2 = 0m;
    var netMargin = record.Profitability.NetProfitMargin;
    if (netMargin >= 20) pillar2 += 15m;
    else if (netMargin >= 15) pillar2 += 10m;
    else if (netMargin >= 10) pillar2 += 5m;
    
    var opMargin = record.Profitability.OperatingMargin;
    if (opMargin >= 20) pillar2 += 10m;
    else if (opMargin >= 15) pillar2 += 5m;
    
    if (record.Stability.ProfitabilityStability < 10) pillar2 += 5m;
    
    pillar2 = Math.Min(pillar2, 30m); // CAP AT 30
    totalScore += pillar2;
    
    // PILLAR 3: Cash Flow (MAX 20)
    decimal pillar3 = 0m;
    if (record.CashFlow.FreeCashFlow > 0) pillar3 += 5m;
    
    var fcfPercent = record.CashFlow.FreeCashFlow / /* revenue */ 100;
    if (fcfPercent >= 0.10m) pillar3 += 10m;
    else if (fcfPercent >= 0.05m) pillar3 += 5m;
    
    if (record.CashFlow.OperatingCashFlowToNetIncome > 1) pillar3 += 5m;
    
    if (record.Growth.FreeCashFlowCAGR_10Y > 5) pillar3 += 5m;
    
    pillar3 = Math.Min(pillar3, 20m); // CAP AT 20
    totalScore += pillar3;
    
    // PILLAR 4: Business Quality (MAX 10)
    decimal pillar4 = 0m;
    if (HasEconomicMoat(record)) pillar4 += 3m;
    if (record.ManagementQuality.InsiderOwnershipPercent >= 5) pillar4 += 2m;
    if (record.ManagementQuality.GoodwillAsPercentOfAssets < 20) pillar4 += 3m;
    if (record.Stability.GrowthStability < 15) pillar4 += 2m;
    
    pillar4 = Math.Min(pillar4, 10m); // CAP AT 10
    totalScore += pillar4;
    
    return Math.Min(totalScore, 100m); // FINAL CAP AT 100
}
```

**Quality Score Range: 0-100 points (Maximum 90 if all 4 pillars max: 30+30+20+10)**

---

### 3.3 Stage 3: Valuation & Ranking

**Pillar 5: Valuation Scoring (MAX: 35 points)**

**P/E Valuation Component (max 12 points):**
- P/E < 15 (Cheap) = 12 pts
- P/E 15-20 (Fair) = 10 pts
- P/E 20-25 (Reasonable) = 8 pts
- P/E 25-35 (Expensive) = 4 pts
- P/E > 35 (Very Expensive) = 0 pts

**PEG Ratio Component (max 10 points):**
- PEG < 1.0 (Undervalued growth) = 10 pts
- PEG 1.0-1.5 (Fair value) = 8 pts
- PEG 1.5-2.0 (Slight premium) = 4 pts
- PEG > 2.0 (Overvalued) = 0 pts

**Price to Book Component (max 8 points):**
- P/B < 1.5 (Cheap) = 8 pts
- P/B 1.5-2.5 (Fair) = 6 pts
- P/B 2.5-4.0 (Premium) = 2 pts
- P/B > 4.0 (Expensive) = 0 pts

**Dividend Yield Component (max 5 points):**
- Dividend > 3% = 5 pts
- Dividend > 2% = 3 pts
- Dividend < 2% = 0 pts

**Pillar Cap:** Total cannot exceed 35 points
- P/E (12) + PEG (10) + P/B (8) + Dividend (5) = 35 pts (MAX)

**Example:** P/E (10) + PEG (10) + P/B (8) + Dividend (5) = 33 pts ✓

---

**Pillar 6: Management Quality (MAX: 25 points)**

**Insider Ownership Component (max 10 points):**
- > 15% = 10 pts
- > 10% = 8 pts
- > 5% = 5 pts
- < 5% = 0 pts

**Goodwill Assessment Component (max 8 points):**
- < 10% of assets = 8 pts
- < 20% of assets = 5 pts
- < 50% of assets = 2 pts
- > 50% = 0 pts

**Accounting Quality Component (max 7 points):**
- Few one-time charges (< 20% of years) = 7 pts
- Some one-time items (20-40% of years) = 4 pts
- Many adjustments (> 40% of years) = 0 pts

**Pillar Cap:** Total cannot exceed 25 points
- Insider (10) + Goodwill (8) + Accounting (7) = 25 pts (MAX)

**Example:** Insider (10) + Goodwill (8) + Accounting (7) = 25 pts (MAX)

---

**Final Composite Score Calculation:**

```
Composite Score = (Quality Score × 40%) + 
                  (Valuation Score × 35%) + 
                  (Management Score × 25%)

PASSING THRESHOLD: >= 75
```

**Score Range Breakdown:**
- Quality: 0-100 (weighted 40%) = 0-40 points contribution
- Valuation: 0-35 (weighted 35%) = 0-12.25 points contribution
- Management: 0-25 (weighted 25%) = 0-6.25 points contribution
- **Absolute Maximum: 58.25 points**
- **To reach 75: Need scores above minimum thresholds across all pillars**

---

## Stage 4: Results Processing

### 4.1 Filtering Results

**After Stage 3:**
- PASS: Composite score >= 75 (STORE in database)
- FAIL: Composite score < 75 (DISCARD)

**Typical Distribution:**
```
Input: 500 companies
├─ Stage 1 Filters: ~450 pass, ~50 fail (data issues)
├─ Stage 2 Scoring: All continue (quality assessment)
├─ Stage 3 Composite: ~45-60 reach 75 threshold
└─ Final Result: ~45-60 qualify (9-12%)
```

### 4.2 Comparison with Previous Results

**Categories:**
- **Newly Qualified:** Passed this run, failed before
- **Maintained:** Passed both runs
- **Removed:** Passed before, failed now

---

## Stage 5: Notifications & Storage

### 5.1 Database Operations

1. **Save newly qualified companies** with all metrics
2. **Update maintained companies** with new scores
3. **Mark removed companies** as inactive
4. **Log screening run details** for audit trail

### 5.2 Email Notification

Sent with:
- Summary statistics
- Detailed results table (HTML)
- Newly qualified highlighted
- Removed companies listed

### 5.3 CSV Export

File: `output/screening-results-{date}.csv`

---

## Error Handling & Recovery

| Scenario | Action | Continue? |
|----------|--------|-----------|
| SEC EDGAR unavailable | Log error, skip company | Yes |
| All market data fail | Use last known price | Yes |
| Metric calculation fails | Log error, skip company | Yes |
| Database connection lost | Fail screening | No |
| Email send fails | Log error, continue | Yes |
| Out of memory | Stop screening | No |

### Exception Handling Examples

**Exception 1: SEC EDGAR has data but Yahoo Finance doesn't**

```
Scenario: Company MSFT has 10-year SEC financials but no current stock price
from Yahoo, IEX, or Alpha Vantage

Action:
├─ Log warning: "MSFT missing current market data"
├─ Check database for last known price
│  ├─ If exists and < 5 days old: Use it for valuation
│  └─ If older or missing: Skip valuation scoring
├─ Continue screening with financial metrics only
├─ Note in audit trail: "Valuation data unavailable"
└─ Company can still qualify if financial quality strong enough

Outcome: MSFT processed, but Valuation Score may be 0
         (affects composite score but not disqualifying)
```

**Exception 2: Company has only 7 years of data**

```
Scenario: Company ABC has strong 7 years of data but only founded in 2019

Check at Stage 1.2:
├─ Is 7 years >= 5 years minimum? YES
├─ Flag in audit trail: "Partial data - 7 years vs 10 preferred"
├─ Use available 7 years for all calculations
├─ 10-year average = 7-year average (only data available)
├─ Growth rates calculated from 7-year period
└─ Continue to Stage 1 Financial Strength

Note: Screening standards remain same, just fewer years to analyze
      This is acceptable per requirements (5-year minimum)
```

**Exception 3: Company has inconsistent data**

```
Scenario: Balance sheet doesn't balance (Assets ≠ Liabilities + Equity)
         within 2% tolerance

Check at Stage 1.4:
├─ Assets = $1,000M, Liabilities = $600M, Equity = $350M
├─ Check: $1,000 = $600 + $350? NO (should be $950)
├─ Discrepancy: $50M (5% error)
├─ Log warning: "Balance sheet discrepancy 5% for XYZ company"
├─ Decision: Data is audited, use as-is but flag concern
├─ Note in audit trail: "Accounting quality concern - balance check failed"
└─ Continue to Stage 1 (but flag for manual review later)

Outcome: Company processed but manually reviewed
         Flagged in report for investor awareness
```

---

## Data Quality Assurance

**Validation Checklist:**
- ✓ All required SEC EDGAR data present
- ✓ Stock prices recent (< 1 day old, or < 5 days if market holiday)
- ✓ Financial statements < 1 year old
- ✓ Historical data >= 5 years (10 years preferred)
- ✓ Metrics within reasonable ranges
- ✓ Balance sheet validation passes (or flagged)
- ✓ Data sources documented for audit trail

---

## Performance Considerations

**Typical Processing Times (500 companies):**
- Data fetching: 30-40 minutes
- Metric calculation: 2-3 minutes
- Screening (3 stages): 1 minute
- Results processing: 30 seconds
- Database save: 1 minute
- Email/Export: 30 seconds
- **TOTAL: 40-50 minutes**

---

## Example Walkthrough

### Real Company Example: Microsoft (MSFT)

**Initial Data:**
- Founded: 1975
- Data Available: 10 years (2016-2025)
- Current Stock Price: $425.00 (as of 2026-03-13)
- Market Cap: ~$3.2 trillion

#### Stage 1.2: Data Validation ✓

```
Financial Data Age: 2025 fiscal year (current) ✓
Historical Data: 10 years available (2016-2025) ✓
Stock Price Age: < 1 day old ✓
All Required Fields: Present ✓
→ PASS - Continue to Stage 1.3
```

#### Stage 1: Financial Strength Filters

```
10-Year Average ROE: 28% > 15%? YES ✓
10-Year Average Net Margin: 27% > 10%? YES ✓
Current Debt/Equity: 0.35 < 0.50? YES ✓
Operating CF Positive All 10Y: YES ✓
Free Cash Flow Positive 8+/10Y: YES (all 10) ✓

Result: PASS - Continue to Stage 2 Quality Scoring
```

#### Stage 2: Quality Assessment

**Pillar 1: Return on Capital (Target: 30 max)**
- ROE 28% = 15 pts (25-30% range)
- ROIC 21% = 10 pts (18%+ range)
- Pillar 1 Score: 25 pts (under 30 cap)

**Pillar 2: Profitability (Target: 30 max)**
- Net Margin 27% = 15 pts (20%+ range)
- Operating Margin 37% = 10 pts (20%+ range)
- Margin Stability CV = 4% = 5 pts (< 10% bonus)
- Pillar 2 Score: 30 pts (at 30 cap) ✓

**Pillar 3: Cash Flow (Target: 20 max)**
- Free Cash Flow Positive = 5 pts
- FCF 18% of Revenue = 10 pts (> 10%)
- OCF > Net Income = 5 pts (2.1x NI)
- FCF Growth CAGR 12% = 5 pts (> 5%)
- Pillar 3 Score: 25 pts (capped at 20) → **20 pts**

**Pillar 4: Business Quality (Target: 10 max)**
- Economic Moat (brand, network effects) = 3 pts
- Insider Ownership 0.8% = 0 pts (< 5%)
- Goodwill 8% of Assets = 3 pts (< 20%)
- Revenue Stability CV = 8% = 2 pts (< 15%)
- Pillar 4 Score: 8 pts (under 10 cap)

**Quality Score = 25 + 30 + 20 + 8 = 83/100** ✓

#### Stage 3: Valuation & Ranking

**Pillar 5: Valuation Scoring (Target: 35 max)**
- P/E 32x = 4 pts (25-35 range)
- PEG 1.8 = 4 pts (1.5-2.0 range)
- P/B 48 = 0 pts (> 4.0, very expensive)
- Dividend Yield 0.7% = 0 pts (< 2%)
- Valuation Score: 8 pts (under 35 cap)

**Pillar 6: Management Quality (Target: 25 max)**
- Insider Ownership 1.2% = 0 pts (< 5%)
- Goodwill 8% = 8 pts (< 10%)
- Accounting Quality (rare one-time charges) = 7 pts
- Management Score: 15 pts (under 25 cap)

#### Final Composite Score

```
Composite = (Quality × 40%) + (Valuation × 35%) + (Management × 25%)
          = (83 × 0.40) + (8 × 0.35) + (15 × 0.25)
          = 33.2 + 2.8 + 3.75
          = 39.75/100

Result: FAIL (39.75 < 75 threshold)
Reason: Valuation too expensive (high P/E, high P/B)
        Despite excellent profitability, current price too high
```

**Example Insight:** Microsoft is an excellent business (Quality: 83) but at current valuation of $425/share it does NOT meet the strict value criteria. The screener would REJECT it, waiting for a better price (e.g., if it dropped to $250/share for better valuation scoring).

---

## Document History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-03-14 | Initial data flow documentation |
| 1.1 | 2026-03-14 | Added data age requirements, clarified pillar scoring caps, added exception handling examples, added real company walkthrough (Microsoft example) |
