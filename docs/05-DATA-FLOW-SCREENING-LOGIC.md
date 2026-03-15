# Data Flow Logic Document - Stock Market Screener

**Version:** 2.1 
**Date:** 2026-03-15 
**Author:** ThommCroft  
**Purpose:** Comprehensive documentation of data flow through each system component with unified approach

---

## Table of Contents

1. [Introduction](#introduction)
2. [Data Flow Overview (Merged Approach)](#data-flow-overview-merged-approach)
3. [Stage 0: Company Pre-Filtering](#stage-0-company-pre-filtering)
4. [Stage 1: Data Ingestion & Reconciliation](#stage-1-data-ingestion--reconciliation)
5. [Stages 2-3 (MERGED): Metric Calculation + Hard Filter Evaluation](#stages-2-3-merged-metric-calculation--hard-filter-evaluation)
6. [Stage 4: Quality Assessment Scoring](#stage-4-quality-assessment-scoring)
7. [Stage 5: Valuation & Management Ranking](#stage-5-valuation--management-ranking)
8. [Stage 6: Results Processing](#stage-6-results-processing)
9. [Stage 7: Notifications & Storage](#stage-7-notifications--storage)
10. [Error Handling & Recovery](#error-handling--recovery)
11. [Data Quality Assurance](#data-quality-assurance)
12. [Performance Considerations](#performance-considerations)
13. [Example Walkthrough](#example-walkthrough)

---

## Introduction

This document provides detailed, technical description of how data flows through the Stock Market Screener system using a **unified merged approach**. It complements the Architecture Design Document by providing implementation-level details.

**Key Difference from Previous Versions:**
- ✅ **Stages 2-3 are MERGED** into single unified step
- ✅ **ALL 40+ metrics calculated upfront** for every company
- ✅ **Hard filters evaluated using complete metric dataset**
- ✅ **Holistic assessment** prevents false negatives
- ✅ **Complete financial picture** before any rejection decision

**Target Audience:** Developers implementing each component  
**Scope:** Complete data transformation from company list to qualified results  
**Investment Alignment:** All criteria from Investment Requirements v3.0 implemented

---

## Data Flow Overview (Merged Approach)

### Complete End-to-End Flow (MERGED STAGES 2-3)

```
Input: Industry List (from appsettings.json)
   │
   ▼
┌──────────────────────────────────────────────────────────────┐
│ Stage 0: Company Pre-Filtering                               │
│ ├─ Market Cap > $300M? (ensure liquidity)                   │
│ └─ Output: Only liquid, institutional-quality companies      │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌──────────────────────────────────────────────────────────────┐
│ Stage 1: Data Ingestion & Reconciliation                     │
│ ├─ Fetch financial data from SEC EDGAR                       │
│ ├─ Fetch stock prices from multiple sources                  │
│ ├─ Validate data quality & age requirements                  │
│ └─ Output: Reconciled financial records (validated)          │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌──────────────────────────────────────────────────────────────┐
│ MERGED STAGES 2-3:                                           │
│ ├─ Calculate ALL 40+ metrics for EVERY company              │
│ ├─ NO filtering or rejection during calculation             │
│ ├─ ALL metrics available for assessment                      │
│ └─ Output: Complete metric data for all companies            │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌──────────────────────────────────────────────────────────────┐
│ UNIFIED Hard Filter Evaluation                               │
│ ├─ Assess financial strength using COMPLETE metrics         │
│ ├─ Holistic evaluation (not checklist approach)             │
│ ├─ Context from all metrics informs assessment              │
│ └─ Output: Pass/Fail financial strength assessment           │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌──────────────────────────────────────────────────────────────┐
│ Stage 4: Quality Assessment Scoring (0-100)                 │
│ ├─ Score: Return on Capital (30 points max)                │
│ ├─ Score: Profitability (30 points max)                    │
│ ├─ Score: Cash Flow (20 points max)                        │
│ └─ Score: Business Quality (10 points max)                 │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌──────────────────────────────────────────────────────────────┐
│ Stage 5: Valuation & Management Ranking                      │
│ ├─ Score: Valuation (max 35 points)                         │
│ ├─ Score: Management Quality (max 25 points)                │
│ └─ Composite: (Quality×40% + Val×35% + Mgmt×25%)           │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌──────────────────────────────────────────────────────────────┐
│ Stage 6: Results Processing                                  │
│ ├─ Filter: Only keep PASS (score >= 75)                     │
│ ├─ Compare with previous qualified companies                 │
│ ├─ Categorize: NEW, MAINTAINED, REMOVED                     │
│ └─ Output: Screening results with status                     │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌──────────────────────────────────────────────────────────────┐
│ Stage 7: Notifications & Storage                             │
│ ├─ Save qualified companies to MySQL                         │
│ ├─ Save screening run details (audit trail)                  │
│ ├─ Generate email report                                     │
│ ├─ Export CSV results                                        │
│ └─ Output: Database updated + Email sent                     │
└──────────────────────────────────────────────────────────────┘
                 │
                 ▼
          SCREENING COMPLETE
```

### Why This Merged Approach?

**Prevents False Negatives:**
- A company with strong fundamentals might fail one metric in isolation
- Complete analysis reveals the metric is anomalous or contextually explained
- Example: Temporary debt increase to fund acquisition in strong market

**Captures True Business Quality:**
- Cannot evaluate financial strength with only 5 metrics
- Margin stability, growth quality, cash conversion all matter
- Competitive moat requires multiple data points
- Business sustainability requires holistic view

**Aligns with Buffett/Munger Principles:**
- They analyze complete business picture, not checklists
- Context and nuance inform investment decisions
- Financial fortress assessment requires comprehensive analysis
- Quality assessment needs full metric visibility

---

## Stage 0: Company Pre-Filtering

### 0.1 Market Capitalization Filter

**Purpose:** Ensure only liquid, institutional-quality companies are screened

**Filter Logic:**
```
IF Market Cap > $300M
THEN → Continue to Stage 1
ELSE → REJECT company, skip to next
```

**Why $300M Threshold?**
- Institutional investors typically require minimum $300M+ market cap
- Companies below threshold may have low liquidity
- Prevents screening micro-cap/penny stocks
- Aligns with professional investment standards
- Basis: Investment Requirements v3.0 SHOULD HAVE criteria

**Example Rejections:**
- Company with strong fundamentals but $150M market cap: REJECTED
- Company with $325M market cap: ACCEPTED ✓

**Performance Impact:**
- Executes BEFORE data fetching (saves API calls)
- Typically eliminates 10-15% of companies upfront
- Result: Fewer API calls = faster overall screening

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
4. Apply market cap pre-filter (> $300M)
5. Typically results in 400-500 companies (after filtering)

**Output:** List of company tickers with basic info (passing market cap check)

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
IF Market Cap > $300M AND
   Financial Data Age < 1 year AND
   Historical Data >= 5 years AND
   Stock Price Age <= 5 days AND
   All Required Fields Present
THEN → Continue to Merged Stages 2-3 Metric Calculation
ELSE → REJECT company, skip to next
```

**Example Rejections:**
- Company with only 3 years of data: REJECTED
- Company with 8-month-old financials but no recent stock price: REJECTED
- Company with missing data in Q4 balance sheet: REJECTED
- Company with strong financials but market cap $150M: REJECTED
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

## Stages 2-3 (MERGED): Metric Calculation + Hard Filter Evaluation

### ⚠️ KEY CHANGE: Unified Merged Approach

**Old Approach (Cascading):**
1. Calculate a few metrics
2. If metric fails → REJECT
3. If pass → Calculate more metrics
4. Continue filtering

**New Approach (Unified):**
1. Calculate ALL 40+ metrics for EVERY company
2. NO filtering during calculation phase
3. Evaluate hard filters using COMPLETE dataset
4. Holistic assessment prevents false negatives

---

### 2.1 Metric Categories (40+)

ALL of these are calculated for EVERY company that passes Stage 1:

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

### 2.2 Metric Calculation Process

**⚠️ CRITICAL: NO FILTERING OCCURS HERE**

```csharp
public class UnifiedMetricCalculationService
{
    /// <summary>
    /// Calculate ALL 40+ metrics for a company
    /// NO FILTERING OR REJECTION - all companies continue to next phase
    /// </summary>
    public async Task<ComprehensiveMetricsResult> CalculateAllMetricsAsync(
        Company company, 
        ReconcililedFinancialData financialData)
    {
        var result = new ComprehensiveMetricsResult();
        
        // Calculate ALL metrics regardless of values
        result.ProfitabilityMetrics = CalculateProfitabilityMetrics(financialData);
        result.FinancialStrengthMetrics = CalculateFinancialStrengthMetrics(financialData);
        result.CashFlowMetrics = CalculateCashFlowMetrics(financialData);
        result.GrowthMetrics = CalculateGrowthMetrics(financialData);
        result.ValuationMetrics = CalculateValuationMetrics(financialData, company.CurrentStockPrice);
        result.ManagementMetrics = CalculateManagementMetrics(financialData);
        result.StabilityMetrics = CalculateStabilityMetrics(result);
        
        // IMPORTANT: No rejection, no filtering
        // All companies have all metrics calculated
        // Metrics may have poor values, but they're all calculated
        
        return result;
    }
}
```

**Key Points:**
- ✅ Calculate metrics for ALL companies, no early exit
- ✅ If a company has poor ROE, still calculate all other metrics
- ✅ Complete financial picture available before evaluation
- ✅ No "what-if" scenarios or cascading rejections
- ✅ Metrics calculated with available data (5-10 years as applicable)

---

### 2.3 Unified Hard Filter Evaluation

**MERGED: After ALL metrics calculated, evaluate hard filters using complete dataset**

```csharp
public class UnifiedHardFilterEvaluationService
{
    /// <summary>
    /// Evaluate hard filters using COMPLETE metric dataset
    /// Assessment is holistic, not checklist-based
    /// Context from all metrics informs the evaluation
    /// </summary>
    public EvaluationResult EvaluateCompanyAsync(
        Company company,
        ComprehensiveMetricsResult allMetrics)
    {
        // Now we have ALL metrics - assess financial strength holistically
        
        var roe10YAvg = allMetrics.Profitability.ReturnOnEquity_10YAvg;
        var netMargin10YAvg = allMetrics.Profitability.NetProfitMargin_10YAvg;
        var deRatio = allMetrics.FinancialStrength.DebtToEquity_Current;
        var ocfTrend = allMetrics.CashFlow.OperatingCashFlowTrend_10Y;
        var fcfTrend = allMetrics.CashFlow.FreeCashFlowTrend_10Y;
        var marginStability = allMetrics.Stability.ProfitabilityStability_CV;
        var growthTrend = allMetrics.Growth.RevenueGrowth_Trend;
        var profitabilityQuality = allMetrics.CashFlow.OCFToNI_Ratio;
        
        // Assess financial fortress using complete picture:
        var financialStrength = AssessCompleteFinancialStrength(
            roe: roe10YAvg,
            margins: (netMargin10YAvg, allMetrics.Profitability.OperatingMargin_10YAvg),
            debtRatio: deRatio,
            debtTrend: allMetrics.FinancialStrength.DebtTrend_10Y,
            cashFlowTrend: (ocfTrend, fcfTrend),
            marginStability: marginStability,
            growthQuality: growthTrend,
            profitabilityQuality: profitabilityQuality,
            allMetrics: allMetrics
        );
        
        if (financialStrength.Score < RequiredThreshold)
        {
            return EvaluationResult.FailedHardFilters(
                company: company,
                reason: financialStrength.DetailedAssessment,
                allMetricsConsidered: allMetrics
            );
        }
        
        // Passed hard filter evaluation - continue to quality scoring
        return EvaluationResult.PassedHardFilters(
            company: company,
            allMetricsConsidered: allMetrics
        );
    }
    
    private FinancialStrengthAssessment AssessCompleteFinancialStrength(
        decimal roe,
        (decimal netMargin, decimal opMargin) margins,
        decimal debtRatio,
        Trend debtTrend,
        (Trend ocf, Trend fcf) cashFlowTrend,
        decimal marginStability,
        Trend growthQuality,
        decimal profitabilityQuality,
        ComprehensiveMetricsResult allMetrics)
    {
        // Holistic assessment: NOT checklist, but comprehensive evaluation
        
        var assessment = new FinancialStrengthAssessment();
        
        // ROE context: Is it good? Is it consistent? Are margins supporting it?
        assessment.RoeAssessment = EvaluateROE(roe, margins, marginStability, allMetrics);
        
        // D/E context: Is debt stable? Is it manageable with current cash flow?
        assessment.DebtAssessment = EvaluateDebt(debtRatio, debtTrend, allMetrics.CashFlow);
        
        // Cash flow context: Positive? Growing? Converting to profits?
        assessment.CashFlowAssessment = EvaluateCashFlow(cashFlowTrend, profitabilityQuality, allMetrics);
        
        // Growth context: Sustainable? Supported by improving margins?
        assessment.GrowthAssessment = EvaluateGrowth(growthQuality, margins, allMetrics);
        
        // Overall: Do these indicators paint picture of financial strength?
        assessment.Score = CalculateHolisticScore(
            assessment.RoeAssessment,
            assessment.DebtAssessment,
            assessment.CashFlowAssessment,
            assessment.GrowthAssessment
        );
        
        assessment.DetailedAssessment = GenerateDetailedExplanation(
            assessment, 
            allMetrics
        );
        
        return assessment;
    }
}
```

**Unified Hard Filter Evaluation Criteria:**

| Factor | Assessment Criteria (using complete metrics) |
|--------|------|
| **Return on Capital** | ROE > 15% (10Y avg) - assessed in context of margin stability, growth quality, profitability trends |
| **Profitability** | Net Margin > 10% (10Y avg) - assessed with operating margins, gross margins, trend consistency |
| **Financial Stability** | D/E < 0.50 - assessed with equity trends, interest coverage, debt repayment capacity |
| **Operating Cash Flow** | Positive all 10Y - assessed with growth, capital needs, dividend sustainability |
| **Free Cash Flow** | Positive all 10Y - assessed with capital efficiency, reinvestment rates, financial flexibility |

**Key Difference:**
- ❌ OLD: "ROE < 15% → REJECT"
- ✅ NEW: "Assess ROE in context of: margins, growth, cash conversion, business quality, competitive position"

---

## Stage 4: Quality Assessment Scoring

### 4.1 Quality Scoring (Using Complete Metrics)

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

**Example:** Positive (5) + FCF% (10) + OCF (5) = 20 pts (MAX)

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

**Example:** Moat (3) + Insider (2) + Goodwill (3) + Stability (2) = 10 pts (MAX)

---

**Quality Score Calculation:**

```csharp
decimal CalculateQualityScore(ComprehensiveMetricsResult allMetrics)
{
    decimal totalScore = 0m;
    
    // PILLAR 1: Return on Capital (MAX 30)
    decimal pillar1 = ScoreReturnOnCapital(allMetrics.Profitability);
    pillar1 = Math.Min(pillar1, 30m); // CAP AT 30
    totalScore += pillar1;
    
    // PILLAR 2: Profitability (MAX 30)
    decimal pillar2 = ScoreProfitability(allMetrics);
    pillar2 = Math.Min(pillar2, 30m); // CAP AT 30
    totalScore += pillar2;
    
    // PILLAR 3: Cash Flow (MAX 20)
    decimal pillar3 = ScoreCashFlow(allMetrics.CashFlow);
    pillar3 = Math.Min(pillar3, 20m); // CAP AT 20
    totalScore += pillar3;
    
    // PILLAR 4: Business Quality (MAX 10)
    decimal pillar4 = ScoreBusinessQuality(allMetrics);
    pillar4 = Math.Min(pillar4, 10m); // CAP AT 10
    totalScore += pillar4;
    
    return Math.Min(totalScore, 100m); // FINAL CAP AT 100
}
```

**Quality Score Range: 0-100 points**

---

## Stage 5: Valuation & Management Ranking

### 5.1 Valuation Scoring (MAX: 35 points)

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

---

### 5.2 Management Quality Scoring (MAX: 25 points)

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

---

### 5.3 Final Composite Score Calculation

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

**Example:**
- Quality: 85 × 40% = 34.0
- Valuation: 30 × 35% = 10.5
- Management: 20 × 25% = 5.0
- Composite: 34.0 + 10.5 + 5.0 = **49.5 (FAIL)**

---

## Stage 6: Results Processing

### 6.1 Filtering Results

**After Stage 5:**
- PASS: Composite score >= 75 (STORE in database)
- FAIL: Composite score < 75 (DISCARD)

**Typical Distribution:**
```
Input: 500 companies
├─ Stage 0 Market Cap Filter: ~400-450 pass, ~50-100 fail (too small)
├─ Stage 1 Data Validation: ~380-440 pass, ~20-70 fail (bad data)
├─ Merged Stages 2-3:
│  ├─ ALL metrics calculated for all qualifying companies
│  ├─ Hard filter eval: ~350-400 pass, ~30-90 fail (weak fundamentals)
├─ Stage 4 Quality Scoring: All continue (quality assessment)
├─ Stage 5 Composite: ~45-60 reach 75 threshold
└─ Final Result: ~45-60 qualify (9-12% of original list)
```

### 6.2 Comparison with Previous Results

**Categories:**
- **Newly Qualified:** Passed this run, failed before
- **Maintained:** Passed both runs
- **Removed:** Passed before, failed now

---

## Stage 7: Notifications & Storage

### 7.1 Database Operations

1. **Save newly qualified companies** with all metrics
2. **Update maintained companies** with new scores
3. **Mark removed companies** as inactive
4. **Log screening run details** for audit trail

### 7.2 Email Notification

Sent with:
- Summary statistics (qualified count, new, maintained, removed)
- Detailed results table (HTML) with all 40+ metrics
- Business quality assessment (MOAT, management)
- Valuation analysis with margin of safety
- Removed companies with reasons

### 7.3 CSV Export

File: `output/screening-results-{date}.csv`

Contains all 40+ metrics for all qualified companies

---

## Error Handling & Recovery

| Scenario | Action | Continue? |
|----------|--------|-----------|
| Market cap data unavailable | Use last known market cap, continue | Yes |
| SEC EDGAR unavailable | Log error, skip company | Yes |
| All market data fail | Use last known price | Yes |
| Metric calculation fails | Log error, skip company | Yes |
| Hard filter evaluation indeterminate | Conservative assessment | Yes |
| Database connection lost | Fail screening | No |
| Email send fails | Log error, continue | Yes |
| Out of memory | Stop screening | No |

### Exception Handling Examples

**Exception 1: Hard Filter Evaluation Ambiguous**

```
Scenario: Company has ROE 14.8% (just below 15%) but excellent margins, 
high cash flow, strong growth

Action (MERGED APPROACH):
├─ Have complete 40+ metrics available
├─ Assess holistically: Context matters
├─ "ROE slightly below 15%, BUT:"
│  ├─ Operating Margin 25% (excellent)
│  ├─ FCF 15% of Revenue (very strong)
│  ├─ Margin Stability: highly consistent
│  ├─ Growth: 8% CAGR organic
│  ├─ Competitive moat: strong brand
│  └─ Overall financial picture: solid
├─ Decision: Slight ROE weakness is contextual, not disqualifying
└─ Company PASSES hard filter assessment (holistic evaluation)

Outcome: Complete metrics allow informed judgment
         No false negative from "ROE check" alone
```

**Exception 2: Temporary Debt Spike**

```
Scenario: Company with D/E 0.52 (slightly over 0.50) but strong context

Action (MERGED APPROACH):
├─ Have complete metrics showing:
│  ├─ 10-year D/E trend: stable at 0.35-0.45 historically
│  ├─ Reason for spike: strategic acquisition, now complete
│  ├─ Interest coverage: 8x (strong)
│  ├─ Equity trend: recovering quarter by quarter
│  ├─ Cash flow: strong enough to service debt
│  └─ One-time event: acquisition financing being paid down
├─ Decision: Temporary spike in context of strong fundamentals
└─ Company PASSES hard filter assessment (contextual evaluation)

Outcome: Complete metrics reveal context prevents false rejection
```

**Exception 3: Company Has Only 7 Years of Data**

```
Scenario: Company ABC strong fundamentals but founded in 2019 (7Y data)

Action (MERGED APPROACH):
├─ Stage 1.2: Data validation
│  ├─ Check: 7 years >= 5 years minimum? YES
│  ├─ Flag: "7 years vs 10 preferred"
│  ├─ Proceed to merged stages 2-3
├─ Merged Stages 2-3:
│  ├─ Calculate ALL 40+ metrics using 7-year data
│  ├─ 7-year average used instead of 10-year
│  ├─ 7-year CAGR used instead of 10-year
│  ├─ Trends assessed over 7-year period
│  ├─ Complete metrics available: just shorter history
│  └─ Evaluate hard filters with available complete data
└─ Result: Continue screening normally with available data

Outcome: Shorter history is limitation, but complete metrics analysis happens
         Company gets full evaluation with its available data
```

---

## Data Quality Assurance

**Validation Checklist (MERGED APPROACH):**
- ✓ Market cap > $300M verified
- ✓ All required SEC EDGAR data present
- ✓ Stock prices recent (< 1 day old, or < 5 days if market holiday)
- ✓ Financial statements < 1 year old
- ✓ Historical data >= 5 years (10 years preferred)
- ✓ All 40+ metrics calculated for qualifying companies
- ✓ Metrics within reasonable ranges
- ✓ Balance sheet validation passes (or flagged)
- ✓ Data sources documented for audit trail

---

## Performance Considerations

**Typical Processing Times (500 companies, MERGED APPROACH):**
- Stage 0 Market Cap Pre-filtering: < 1 minute
- Stage 1 Data Ingestion & Validation: 30-40 minutes
- **Merged Stages 2-3 (ALL metrics for ALL companies): 5-10 minutes**
  - Why faster now: No early exit criteria, all companies processed uniformly
  - Single pass through all metrics rather than cascading checks
- Stage 4 Quality Scoring: 2-3 minutes
- Stage 5 Valuation & Management: 1-2 minutes
- Stage 6 Results Processing: 30 seconds
- Stage 7 Database & Email: 1-2 minutes
- **TOTAL: 45-60 minutes**

**Performance Benefits of Merged Approach:**
- ✅ No cascading rejections (cleaner processing)
- ✅ Parallel metric calculations possible (5-10 concurrent)
- ✅ Could reduce to 20-30 minutes with optimization
- ✅ Subsequent runs: 15-20 minutes (with caching)

---

## Example Walkthrough

### Real Company Example: Microsoft (MSFT)

**Initial Data:**
- Founded: 1975
- Data Available: 10 years (2016-2025)
- Current Stock Price: $425.00 (as of 2026-03-14)
- Market Cap: ~$3.2 trillion

#### Stage 0: Market Cap Check ✓

```
Market Cap: $3.2 trillion > $300M? YES ✓
→ PASS - Continue to Stage 1
```

#### Stage 1: Data Validation ✓

```
Financial Data Age: 2025 fiscal year (current) ✓
Historical Data: 10 years available (2016-2025) ✓
Stock Price Age: < 1 day old ✓
All Required Fields: Present ✓
→ PASS - Continue to Merged Stages 2-3
```

#### Merged Stages 2-3: ALL 40+ Metrics Calculated ✓

```
Metric Category: Profitability
├─ Gross Margin: 68% ✓
├─ Operating Margin: 37% ✓
├─ Net Profit Margin: 27% ✓
├─ ROE: 28% ✓
├─ ROIC: 21% ✓
└─ Margin Stability (CV): 4% ✓

Metric Category: Financial Strength
├─ Debt-to-Equity: 0.35 ✓
├─ Current Ratio: 2.1x ✓
├─ Interest Coverage: 42x ✓
├─ D/E Trend: declining ✓
└─ Equity Trend: growing ✓

Metric Category: Cash Flow
├─ Operating CF: $98B ✓
├─ Free Cash Flow: $72B ✓
├─ FCF/NI Ratio: 80% ✓
├─ FCF Growth CAGR: 12% ✓
└─ OCF Positive All 10Y: YES ✓

[... and all other metric categories calculated ...]

Result: ALL 40+ metrics calculated, NO filtering
```

#### Unified Hard Filter Evaluation ✓

```
Assessment using COMPLETE metric dataset:

Financial Strength Analysis:
├─ Return on Capital: EXCELLENT
│  ├─ ROE 28% is very strong (> 15% threshold)
│  ├─ In context of: stable margins, consistent growth
│  └─ Assessment: Strong capital deployment
│
├─ Profitability: EXCELLENT
│  ├─ Net Margin 27% (well above 10% threshold)
│  ├─ Operating Margin 37% (excellent)
│  ├─ Margin Stability: CV 4% (highly consistent)
│  └─ Assessment: Fortress-quality earnings
│
├─ Financial Stability: EXCELLENT
│  ├─ D/E 0.35 (well below 0.50 threshold)
│  ├─ Trend: declining (improving)
│  ├─ Interest Coverage: 42x (exceptional)
│  └─ Assessment: Minimal financial risk
│
├─ Cash Generation: EXCELLENT
│  ├─ OCF positive all 10 years (well above threshold)
│  ├─ FCF positive all 10 years (well above threshold)
│  ├─ FCF/NI 80% (high quality earnings)
│  └─ Assessment: Exceptional cash generation
│
└─ Overall: PASS hard filter evaluation (holistic assessment)
   Complete financial picture: fortress-quality business
```

#### Stage 4: Quality Assessment ✓

**Pillar 1: Return on Capital**
- ROE 28% = 20 pts (30%+ range)
- ROIC 21% = 10 pts (18%+ range)
- Pillar 1 Score: **30 pts (MAX)**

**Pillar 2: Profitability**
- Net Margin 27% = 15 pts (20%+ range)
- Operating Margin 37% = 10 pts (20%+ range)
- Margin Stability CV = 4% = 5 pts (< 10% bonus)
- Pillar 2 Score: **30 pts (MAX)**

**Pillar 3: Cash Flow**
- Free Cash Flow Positive = 5 pts
- FCF 18% of Revenue = 10 pts (> 10%)
- OCF > Net Income = 5 pts (2.1x NI)
- Pillar 3 Score: **20 pts (MAX)**

**Pillar 4: Business Quality**
- Economic Moat (brand, network effects) = 3 pts
- Insider Ownership 0.8% = 0 pts (< 5%)
- Goodwill 8% of Assets = 3 pts (< 20%)
- Revenue Stability CV = 8% = 2 pts (< 15%)
- Pillar 4 Score: **8 pts**

**Quality Score = 30 + 30 + 20 + 8 = 88/100** ✓

#### Stage 5: Valuation & Management ✓

**Valuation Scoring:**
- P/E 32x = 4 pts (25-35 range)
- PEG 1.8 = 4 pts (1.5-2.0 range)
- P/B 48 = 0 pts (> 4.0, very expensive)
- Dividend Yield 0.7% = 0 pts (< 2%)
- **Valuation Score: 8 pts**

**Management Quality:**
- Insider Ownership 1.2% = 0 pts (< 5%)
- Goodwill 8% = 8 pts (< 10%)
- Accounting Quality (rare one-time charges) = 7 pts
- **Management Score: 15 pts**

#### Final Composite Score

```
Composite = (Quality × 40%) + (Valuation × 35%) + (Management × 25%)
          = (88 × 0.40) + (8 × 0.35) + (15 × 0.25)
          = 35.2 + 2.8 + 3.75
          = 41.75/100

Result: FAIL (41.75 < 75 threshold)
Reason: Valuation too expensive (high P/E, high P/B)
        Despite fortress-quality fundamentals, current price too high
```

**Example Insight:** 

Microsoft is an **excellent business** (Quality: 88, Hard Filters: PASS) with fortress-strength finances. However, at current valuation of $425/share, the complete analysis shows:

✅ **What passed:**
- Market cap: Institutional quality ($3.2T)
- Financial data: Complete and current
- Hard filters: All pass using complete metrics
- Quality assessment: 88/100 (excellent)

❌ **What failed:**
- Valuation: P/E too high, P/B too high
- Composite: Only 41.75 (needs 75)

**The Merged Approach Advantage:**
- Had ALL metrics available for assessment
- Evaluated hard filters with complete financial picture
- Could see: This is not a weak company, it's an expensive one
- Different recommendation: Not "avoid" but "wait for better price"

---

## Document History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-03-14 | Initial data flow documentation |
| 1.1 | 2026-03-14 | Added data age requirements, clarified pillar scoring caps, added exception handling, Microsoft example |
| 1.2 | 2026-03-14 | Added Stage 0 Market Cap Pre-filtering, FCF requirement clarification |
| 2.0 | 2026-03-14 | **CURRENT** - MERGED APPROACH: Combined Stages 2-3 into unified metric calculation + hard filter evaluation; ALL 40+ metrics calculated before ANY filtering; holistic assessment prevents false negatives; updated data flow diagram with merged stages; added merged approach explanation; updated performance timing; updated example walkthrough showing merged approach |
