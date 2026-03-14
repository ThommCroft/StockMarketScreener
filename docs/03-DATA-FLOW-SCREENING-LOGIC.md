# Data Flow Logic Document - Stock Market Screener

**Version:** 1.0  
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

---

## Introduction

This document provides a detailed, technical description of how data flows through the Stock Market Screener system. It complements the Architecture Design Document by providing implementation-level details and decision logic at each stage.

**Target Audience:** Developers implementing each component  
**Scope:** Complete data transformation from company list to qualified results  
**Frequency:** Updated as logic changes

---

## Data Flow Overview

### Complete End-to-End Flow

```
Input: Industry List (from appsettings.json)
   │
   ▼
┌──────────────────────────────────────────────────┐
│ Stage 1: Data Ingestion & Reconciliation         │
│ ├─ Fetch companies from SEC EDGAR API            │
│ ├─ Fetch financial data (multiple sources)       │
│ ├─ Reconcile conflicting data                    │
│ └─ Output: Raw financial records (validated)     │
└────────────────────┬─────────────────────────────┘
                     │
                     ▼
┌──────────────────────────────────────────────────┐
│ Stage 2: Metric Calculation                      │
│ ├─ Parse financial statements                    │
│ ├─ Calculate 40+ financial metrics               │
│ ├─ Perform 10-year trend analysis                │
│ └─ Output: Metric-enriched company records       │
└────────────────────┬─────────────────────────────┘
                     │
                     ▼
┌──────────────────────────────────────────────────┐
│ Stage 3: 3-Stage Screening                       │
│ ├─ Stage 1: Financial Strength (hard filters)    │
│ ├─ Stage 2: Quality Assessment (scoring)         │
│ ├─ Stage 3: Valuation & Management               │
│ └─ Output: Pass/Fail + Composite Score (0-100)   │
└────────────────────┬─────────────────────────────┘
                     │
                     ▼
┌──────────────────────────────────────────────────┐
│ Stage 4: Results Processing                      │
│ ├─ Filter: Only keep PASS (score >= 75)          │
│ ├─ Compare with previous results                 │
│ ├─ Categorize: NEW, MAINTAINED, REMOVED          │
│ └─ Output: Screening results with status         │
└────────────────────┬─────────────────────────────┘
                     │
                     ▼
┌──────────────────────────────────────────────────┐
│ Stage 5: Notifications & Storage                 │
│ ├─ Save qualified companies to MySQL             │
│ ├─ Save screening run details (audit trail)      │
│ ├─ Generate email report                         │
│ ├─ Export CSV results                            │
│ └─ Output: Database + Email + Files exported     │
└──────────────────────────────────────────────────┘
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

**Data Structure:**

```csharp
public class CompanyBasicInfo
{
    public string Ticker { get; set; }              // "MSFT"
    public string CompanyName { get; set; }         // "Microsoft Corporation"
    public string Sector { get; set; }              // "Technology"
    public string Industry { get; set; }            // "Software"
    public DateTime AddedDate { get; set; }         // When added to system
    public string DataSourceFetched { get; set; }   // "SEC EDGAR"
}
```

---

### 1.2 Financial Data Fetching Strategy

**Goal:** Retrieve comprehensive financial statements for 10-year analysis

**Data Required:**

- Income Statement (Revenue, Net Income, Operating Income)
- Balance Sheet (Assets, Liabilities, Equity)
- Cash Flow Statement (Operating CF, Free CF, CapEx)
- Market Data (Stock Price, Market Cap, Dividends)

**Multi-Source Priority:**

```
For FINANCIAL STATEMENTS (Income, Balance Sheet, Cash Flow):
├─ Primary: SEC EDGAR (10-K annual, 10-Q quarterly)
│  ├─ Most authoritative source
│  ├─ Official company filings
│  └─ Timeout: 30 seconds per request
│
└─ Error: Log and skip company (required source)

For STOCK PRICES & MARKET DATA:
├─ Primary: Yahoo Finance API
│  ├─ Real-time stock prices
│  ├─ Market capitalization
│  └─ Timeout: 10 seconds per request
│
├─ Fallback #1: IEX Cloud
│  └─ If Yahoo fails or times out
│
├─ Fallback #2: Alpha Vantage
│  └─ Final fallback option
│
└─ Error (all fail): Use last known stock price from DB

For TREASURY YIELDS (Risk-free rate):
├─ Primary: Federal Reserve FRED API
│  └─ 10-year Treasury yield (updated daily)
│
└─ Fallback: Hardcoded estimate (2.5%) if API unavailable
```

**Implementation Flow:**

```csharp
async Task<FinancialData> FetchCompanyFinancials(string ticker)
{
    var financialData = new FinancialData { Ticker = ticker };
    
    // 1. ALWAYS fetch SEC EDGAR (required)
    try
    {
        financialData.SecEdgarData = await _secEdgarService
            .FetchLatestReports(ticker);
        if (financialData.SecEdgarData == null)
        {
            _logger.LogWarning($"No SEC EDGAR data found for {ticker}");
            return null; // Skip this company
        }
    }
    catch (Exception ex)
    {
        _logger.LogError($"SEC EDGAR fetch failed: {ex.Message}");
        return null; // Skip this company
    }
    
    // 2. Fetch market data with fallbacks
    financialData.MarketData = await FetchMarketDataWithFallbacks(ticker);
    
    // 3. Fetch Treasury yield
    financialData.TreasuryYield = await FetchTreasuryYield();
    
    return financialData;
}

async Task<MarketData> FetchMarketDataWithFallbacks(string ticker)
{
    // Try Yahoo Finance
    try
    {
        var data = await _yahooFinanceService.FetchAsync(ticker);
        if (data != null) return data;
    }
    catch (Exception ex)
    {
        _logger.LogWarning($"Yahoo Finance failed: {ex.Message}");
    }
    
    // Try IEX Cloud
    try
    {
        var data = await _iexCloudService.FetchAsync(ticker);
        if (data != null) return data;
    }
    catch (Exception ex)
    {
        _logger.LogWarning($"IEX Cloud failed: {ex.Message}");
    }
    
    // Try Alpha Vantage
    try
    {
        var data = await _alphaVantageService.FetchAsync(ticker);
        if (data != null) return data;
    }
    catch (Exception ex)
    {
        _logger.LogWarning($"Alpha Vantage failed: {ex.Message}");
    }
    
    _logger.LogError($"All market data sources failed for {ticker}");
    return null;
}
```

---

### 1.3 Data Reconciliation

**Goal:** When multiple sources provide overlapping data, ensure consistency

**Reconciliation Rules:**

| Data Item | Conflict Resolution | Example |
|-----------|-------------------|---------|
| Stock Price | Use most recent (today's date) | Yahoo: $150.25 (today) vs IEX: $149.80 (yesterday) → Use Yahoo |
| Market Cap | Calculate from stock price × shares | Always calculate fresh |
| Dividend | Verify across 2+ sources before using | Must appear in Yahoo AND IEX |
| Financial metrics | Always use SEC EDGAR (single source) | Never blend sources |
| Historical data | Fill gaps using secondary sources | SEC: 9 years + IEX: fill year 10 |

**Data Validation Rules:**

```
BEFORE USING ANY DATA:

1. Check Completeness
   ├─ All required fields present?
   └─ No null values in critical metrics?

2. Range Validation
   ├─ ROE: -100% to +200%
   ├─ Debt/Equity: 0 to 5.0
   ├─ Margins: -50% to +100%
   └─ Growth rates: -50% to +50% annually

3. Balance Sheet Validation
   ├─ Assets = Liabilities + Shareholders' Equity ±2%
   └─ Flag discrepancies for review

4. Consistency Check
   ├─ Year-over-year changes < 300%
   └─ Revenue growth correlates with income growth

5. Data Age Check
   ├─ Financial statements: < 1 year old
   ├─ Stock prices: < 1 day old
   └─ Flag outdated data
```

**Data Quality Report:**

```csharp
public class DataQualityReport
{
    public string Ticker { get; set; }
    public bool IsComplete { get; set; }
    public bool IsValid { get; set; }
    public List<string> Warnings { get; set; }
    public List<string> Errors { get; set; }
    public Dictionary<string, string> DataSources { get; set; }
    public DateTime DataAsOfDate { get; set; }
}

// Example:
{
    "Ticker": "MSFT",
    "IsComplete": true,
    "IsValid": true,
    "Warnings": ["Stock price > 24h old"],
    "Errors": [],
    "DataSources": {
        "IncomeStatement": "SEC EDGAR",
        "BalanceSheet": "SEC EDGAR",
        "StockPrice": "Yahoo Finance"
    },
    "DataAsOfDate": "2026-03-14T09:30:00Z"
}
```

---

## Stage 2: Metric Calculation

### 2.1 Metric Categories (40+)

**Goal:** Calculate comprehensive financial metrics from raw data

All metrics are calculated for:
- Each historical year (up to 10 years: 2016-2025)
- 10-year average
- Stability (Coefficient of Variation)

#### Profitability Metrics (8)

```csharp
public class ProfitabilityMetrics
{
    // Margins (as percentages)
    public decimal GrossMargin { get; set; }
    public decimal OperatingMargin { get; set; }
    public decimal NetProfitMargin { get; set; }
    public decimal EBITDA_Margin { get; set; }
    
    // Ratios
    public decimal ReturnOnAssets_ROA { get; set; }
    public decimal ReturnOnEquity_ROE { get; set; }
    public decimal ReturnOnInvestedCapital_ROIC { get; set; }
    public decimal ReturnOnCapitalEmployed_ROCE { get; set; }
}

// Example Calculations:
// GrossMargin = (Revenue - COGS) / Revenue * 100
// NetProfitMargin = NetIncome / Revenue * 100
// ROE = NetIncome / Shareholders'Equity * 100
```

#### Financial Strength Metrics (6)

```csharp
public class FinancialStrengthMetrics
{
    // Leverage ratios
    public decimal DebtToEquity { get; set; }
    public decimal DebtToAssets { get; set; }
    public decimal EquityMultiplier { get; set; }
    
    // Liquidity ratios
    public decimal CurrentRatio { get; set; }
    public decimal QuickRatio { get; set; }
    
    // Coverage ratios
    public decimal InterestCoverageRatio { get; set; }
}

// Interpretation:
// DebtToEquity < 0.50 = Strong
// CurrentRatio > 1.0 = Healthy
// InterestCoverageRatio > 2.5 = Good
```

#### Cash Flow Metrics (6)

```csharp
public class CashFlowMetrics
{
    public decimal OperatingCashFlow { get; set; }
    public decimal FreeCashFlow { get; set; }
    public decimal OperatingCashFlowToNetIncome { get; set; }
    public decimal FreeCashFlowToOperatingCashFlow { get; set; }
    public decimal CapitalExpenditurePercent { get; set; }
    public decimal CashFlowFromOperations_10Y_Avg { get; set; }
}

// Example:
// FreeCashFlow = OperatingCashFlow - CapitalExpenditure
// OCF_to_NI_Ratio > 1.0 = High quality earnings
```

#### Growth Metrics (7)

```csharp
public class GrowthMetrics
{
    // Year-over-year
    public decimal RevenueGrowth_YoY { get; set; }
    public decimal EarningsGrowth_YoY { get; set; }
    public decimal FreeCashFlowGrowth_YoY { get; set; }
    
    // Compound annual growth rates
    public decimal RevenueCAGR_10Y { get; set; }
    public decimal EarningsCAGR_10Y { get; set; }
    public decimal FreeCashFlowCAGR_10Y { get; set; }
    
    // Per share
    public decimal EarningsPerShare { get; set; }
}

// Example CAGR:
// CAGR = (Ending_Value / Beginning_Value) ^ (1/10) - 1
// 2M / 1M = 2.0, 2.0^0.1 - 1 = 7.2% annual growth
```

#### Valuation Metrics (8)

```csharp
public class ValuationMetrics
{
    // Price-to-metric ratios
    public decimal PriceToEarnings_PE { get; set; }
    public decimal PriceToBook_PB { get; set; }
    public decimal PriceToSales_PS { get; set; }
    public decimal PriceToEarningsGrowth_PEG { get; set; }
    
    // Yield metrics
    public decimal EarningsYield { get; set; }
    public decimal DividendYield { get; set; }
    public decimal DividendPayoutRatio { get; set; }
    
    // Enterprise value
    public decimal EnterpriseValue_to_Revenue { get; set; }
}

// Example:
// PE_Ratio = StockPrice / EPS
// DividendYield = AnnualDividend / StockPrice * 100
// EarningsYield = 1 / PE_Ratio * 100
```

#### Management Quality Metrics (6)

```csharp
public class ManagementQualityMetrics
{
    public decimal InsiderOwnershipPercent { get; set; }
    public decimal GoodwillAsPercentOfAssets { get; set; }
    public decimal OneTimeChargesFrequency { get; set; }
    public decimal RetainedEarningsAsPercentOfEquity { get; set; }
    public decimal OperatingExpenseRatio { get; set; }
    public decimal AssetTurnover { get; set; }
}

// Interpretation:
// InsiderOwnership > 10% = Good alignment
// Goodwill > 50% of Assets = Concern
// OneTimeCharges < 20% = Good quality
// RetainedEarnings > 50% = Strong reinvestment
```

#### Stability Metrics (Coefficient of Variation)

```csharp
public class StabilityMetrics
{
    public decimal ProfitabilityStability { get; set; }
    public decimal GrowthStability { get; set; }
    public decimal CashFlowStability { get; set; }
}

// CV = (Standard Deviation / Mean) * 100
// < 10% = Highly stable ✓
// 10-20% = Stable ✓
// 20-30% = Moderate
// > 30% = Volatile ✗
```

---

### 2.2 10-Year Trend Analysis

**Goal:** Identify trends and consistency over decade

```csharp
public class TrendAnalysis
{
    public decimal ProfitMarginTrend { get; set; }
    public decimal RevenueGrowthTrend { get; set; }
    public decimal ROETrend { get; set; }
    public decimal DebtTrend { get; set; }
    
    public bool HasImprovedOverTime { get; set; }
    public bool IsStable { get; set; }
    public bool HasConcerns { get; set; }
}

// Trend Determination:
// Positive slope = Improving trend ✓
// Flat slope (< 2% change) = Stable trend ✓
// Negative slope = Declining trend ✗
```

---

## Stage 3: Screening Logic

### 3.1 Stage 1: Financial Strength Filters (Hard Filters)

**Purpose:** Eliminate financially weak companies immediately

**Logic:** ALL filters must pass. If ANY fails → Company is REJECTED

```
IF  ROE (10-year avg) > 15% AND
    Net Profit Margin (10-year avg) > 10% AND
    Debt/Equity Ratio < 0.50 AND
    Operating Cash Flow (positive all 10 years) AND
    Free Cash Flow (positive 8+ of 10 years)
THEN → Continue to Stage 2
ELSE → REJECT company
```

**Detailed Filter Rules:**

| Filter | Threshold | Rationale | Data Used |
|--------|-----------|-----------|-----------|
| **ROE** | > 15% (10Y avg) | Return on capital | 10-year average |
| **Net Margin** | > 10% (10Y avg) | Profitability | 10-year average |
| **Debt/Equity** | < 0.50 | Financial stability | Latest fiscal year |
| **Operating CF** | Positive all 10 years | Cash generation | Historical |
| **Free Cash Flow** | Positive 8+ of 10 years | Sustainable business | Calculated |

**Implementation:**

```csharp
bool PassStage1(CompanyMetricsRecord record)
{
    var avgROE = record.Profitability.ReturnOnEquity;
    var avgNetMargin = record.Profitability.NetProfitMargin;
    var debtToEquity = record.FinancialStrength.DebtToEquity;
    var ocfPositiveYears = CountPositiveYears(record.CashFlow.OperatingCashFlow);
    var fcfPositiveYears = CountPositiveYears(record.CashFlow.FreeCashFlow);
    
    if (avgROE <= 15m) return false;
    if (avgNetMargin <= 10m) return false;
    if (debtToEquity >= 0.50m) return false;
    if (ocfPositiveYears < 10) return false;
    if (fcfPositiveYears < 8) return false;
    
    return true;
}
```

---

### 3.2 Stage 2: Quality Assessment Scoring (0-100)

**Purpose:** Score passing companies on quality

**Pillar 1: Return on Capital (30 points)**

```
ROE 15-20%       = 5 points
ROE 20-25%       = 10 points
ROE 25-30%       = 15 points
ROE 30%+         = 20 points

ROIC 12-15%      = 5 points
ROIC 15-18%      = 10 points
ROIC 18%+        = 10 points

Total: 0-30 points
```

**Pillar 2: Profitability (30 points)**

```
Net Margin 10-15%   = 5 points
Net Margin 15-20%   = 10 points
Net Margin 20%+     = 15 points

Operating Margin 15-20% = 5 points
Operating Margin 20%+   = 10 points

Margin Stability (CV < 10%) = 5 points (bonus)

Total: 0-30 points
```

**Pillar 3: Cash Flow (20 points)**

```
FCF Positive           = 5 points
FCF > 5% of Revenue    = 5 points
FCF > 10% of Revenue   = 10 points

OCF > Net Income       = 5 points

FCF Growth > 5% CAGR   = 5 points

Total: 0-20 points
```

**Pillar 4: Business Quality (10 points)**

```
Economic Moat Detected        = 3 points
Insider Ownership > 5%        = 2 points
Low Goodwill (< 20% assets)   = 3 points
Revenue Stability (CV < 15%)  = 2 points

Total: 0-10 points
```

**Total Quality Score: 0-100 points**

```csharp
decimal CalculateQualityScore(CompanyMetricsRecord record)
{
    decimal score = 0m;
    
    // Pillar 1: Return on Capital (30 points)
    var roe = record.Profitability.ReturnOnEquity;
    if (roe >= 30) score += 20m;
    else if (roe >= 25) score += 15m;
    else if (roe >= 20) score += 10m;
    else if (roe >= 15) score += 5m;
    
    // Pillar 2: Profitability (30 points)
    var netMargin = record.Profitability.NetProfitMargin;
    if (netMargin >= 20) score += 15m;
    else if (netMargin >= 15) score += 10m;
    else if (netMargin >= 10) score += 5m;
    
    // Additional pillars...
    
    return Math.Min(score, 100m);
}
```

---

### 3.3 Stage 3: Valuation & Final Score

**Purpose:** Apply valuation and management metrics for final composite score

**Pillar 5: Valuation Scoring (35 points)**

```
P/E Valuation:
  P/E < 15           = 12 points
  P/E 15-20          = 10 points
  P/E 20-25          = 8 points
  P/E 25-35          = 4 points
  P/E > 35           = 0 points

PEG Ratio:
  PEG < 1.0          = 10 points
  PEG 1.0-1.5        = 8 points
  PEG 1.5-2.0        = 4 points
  PEG > 2.0          = 0 points

Price to Book:
  P/B < 1.5          = 8 points
  P/B 1.5-2.5        = 6 points
  P/B 2.5-4.0        = 2 points
  P/B > 4.0          = 0 points

Dividend Yield:
  Dividend > 2%      = 3 points
  Dividend > 3%      = 5 points

Total Valuation: 0-35 points
```

**Pillar 6: Management Quality (25 points)**

```
Insider Ownership:
  > 15%              = 10 points
  > 10%              = 8 points
  > 5%               = 5 points
  < 5%               = 0 points

Goodwill Assessment:
  < 10% of assets    = 8 points
  < 20% of assets    = 5 points
  < 50% of assets    = 2 points
  > 50%              = 0 points

Accounting Quality:
  Few one-time charges   = 7 points
  Some one-time items    = 4 points
  Many adjustments       = 0 points

Total Management: 0-25 points
```

**Final Composite Score Calculation:**

```
Composite Score = (Quality Score × 40%) +
                  (Valuation Score × 35%) +
                  (Management Score × 25%)

PASSING THRESHOLD: >= 75
```

**Score Examples:**

```
Example 1 (FAIL):
  Quality = 85, Valuation = 28, Management = 20
  Composite = (85×0.40) + (28×0.35) + (20×0.25) = 48.8

Example 2 (FAIL):
  Quality = 90, Valuation = 32, Management = 22
  Composite = (90×0.40) + (32×0.35) + (22×0.25) = 52.7

Example 3 (PASS):
  Quality = 95, Valuation = 35, Management = 25
  Composite = (95×0.40) + (35×0.35) + (25×0.25) = 56.5

Note: Achieving 75+ requires excellent scores across all pillars
```

---

## Stage 4: Results Processing

### 4.1 Filtering Results

After Stage 3 screening:

- **PASS:** Composite score >= 75 (STORE in database)
- **FAIL:** Composite score < 75 (DO NOT STORE)

**Typical Results Distribution:**

```
Input: 500 companies
├─ Stage 1 Filters: ~450 pass, ~50 fail
├─ Stage 2 Scoring: All continue
├─ Stage 3 Composite: ~45-60 reach 75 threshold
└─ Final Result: ~45-60 qualify (9-12%)
```

---

### 4.2 Comparison with Previous Results

Compare current run with previous qualifying companies:

```csharp
public class ScreeningComparison
{
    public List<string> NewlyQualified { get; set; }
    public List<string> Maintained { get; set; }
    public List<string> Removed { get; set; }
}

// Example logic:
var current = currentResults
    .Where(r => r.PassedScreening)
    .Select(r => r.Ticker)
    .ToList();

var previous = await _db.QualifiedCompanies
    .Where(c => c.IsActive)
    .Select(c => c.Ticker)
    .ToListAsync();

var newly = current.Except(previous).ToList();       // [MSFT, AAPL]
var maintained = current.Intersect(previous).ToList(); // [JNJ, KO, JPM]
var removed = previous.Except(current).ToList();     // [OLD1, OLD2]
```

---

## Stage 5: Notifications & Storage

### 5.1 Database Operations

```csharp
// 1. Save newly qualified companies
foreach (var ticker in newly)
{
    var company = new QualifiedCompany
    {
        Ticker = ticker,
        CompanyName = result.CompanyName,
        CompositeScore = result.CompositeScore,
        QualityScore = result.QualityScore,
        ValuationScore = result.ValuationScore,
        ManagementScore = result.ManagementScore,
        FirstQualifiedDate = DateTime.UtcNow,
        LastQualifiedDate = DateTime.UtcNow,
        IsActive = true
    };
    
    _db.QualifiedCompanies.Add(company);
    
    // Save metrics for each year
    for (int year = 2016; year <= 2025; year++)
    {
        var metrics = new FinancialMetrics { /* ... */ };
        _db.FinancialMetrics.Add(metrics);
    }
}

// 2. Update maintained companies
foreach (var ticker in maintained)
{
    var company = await _db.QualifiedCompanies
        .FirstAsync(c => c.Ticker == ticker);
    company.CompositeScore = result.CompositeScore;
    company.LastQualifiedDate = DateTime.UtcNow;
}

// 3. Mark removed companies as inactive
foreach (var ticker in removed)
{
    var company = await _db.QualifiedCompanies
        .FirstAsync(c => c.Ticker == ticker);
    company.IsActive = false;
    company.RemovedDate = DateTime.UtcNow;
    
    var removedRecord = new RemovedCompany
    {
        Ticker = ticker,
        ReasonRemoved = "Failed latest screening",
        LastCompositeScore = company.CompositeScore,
        DateRemoved = DateTime.UtcNow
    };
    _db.RemovedCompanies.Add(removedRecord);
}

// 4. Log screening run
var runRecord = new ScreeningRun
{
    RunDate = DateTime.UtcNow,
    RunType = "Scheduled",
    Status = "Success",
    CompaniesProcessed = totalCompanies,
    CompaniesQualified = current.Count,
    NewlyQualified = newly.Count,
    MaintainedQualified = maintained.Count,
    Removed = removed.Count,
    ExecutionTimeSeconds = stopwatch.ElapsedMilliseconds / 1000,
    IndustriesScreened = "Technology, Healthcare, Consumer Defensive"
};
_db.ScreeningRuns.Add(runRecord);

await _db.SaveChangesAsync();
```

### 5.2 Email Notification

Sent to configured recipient with:
- Summary statistics
- Detailed results table (HTML)
- Newly qualified companies highlighted
- Removed companies listed
- CSV attachment

### 5.3 CSV Export

File: `output/screening-results-{date}.csv`

Contains all qualified companies with their metrics.

---

## Error Handling & Recovery

### Error Scenarios:

| Scenario | Action | Continue? |
|----------|--------|-----------|
| SEC EDGAR unavailable | Log error, skip company | Yes |
| All market data sources fail | Use last known price | Yes |
| Metric calculation fails | Log error, skip company | Yes |
| Database connection lost | Fail screening, alert admin | No |
| Email send fails | Log error, continue | Yes |
| Out of memory | Stop screening, log error | No |

---

## Data Quality Assurance

### Validation Checklist:

- ✓ All required SEC EDGAR data present
- ✓ Stock prices recent (< 1 day old)
- ✓ Financial statements not null
- ✓ Metrics within reasonable ranges
- ✓ 10-year historical data available (8+ years acceptable)
- ✓ Data sources documented for audit trail

---

## Performance Considerations

### Typical Processing Times:

```
500 companies processed:
├─ Data fetching: ~30-40 minutes (API rate limits)
├─ Metric calculation: ~2-3 minutes
├─ Screening (3 stages): ~1 minute
├─ Results processing: ~30 seconds
├─ Database save: ~1 minute
└─ Email/Export: ~30 seconds

TOTAL: ~40-50 minutes per screening run
```

### Optimization Tips:

1. Use connection pooling for HTTP clients
2. Cache industry lists
3. Parallel process companies (5-10 concurrent)
4. Batch database inserts (500 at a time)
5. Compress CSV before email

---

## Document History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-03-14 | Initial data flow documentation |
