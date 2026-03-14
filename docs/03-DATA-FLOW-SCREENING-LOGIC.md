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
Input: Industry List (from appsettings.json) │ ▼ ┌─────────────────────────────────────────────────────────────┐ │ Stage 1: Data Ingestion & Reconciliation │ │ ├─ Fetch company list from SEC EDGAR API │ │ ├─ For each company: fetch financial data (multiple sources)│ │ ├─ Reconcile conflicting data │ │ └─ Output: Raw financial records (validated) │ └───��────────────────┬────────────────────────────────────────┘ │ ▼ ┌─────────────────────────────────────────────────────────────┐ │ Stage 2: Metric Calculation │ │ ├─ Parse financial statements │ │ ├─ Calculate 40+ financial metrics │ │ ├─ Perform 10-year trend analysis │ │ └─ Output: Metric-enriched company records │ └────────────────────┬────────────────────────────────────────┘ │ ▼ ┌─────────────────────────────────────────────────────────────┐ │ Stage 3: 3-Stage Screening │ │ ├─ Stage 1: Financial Strength (hard filters) │ │ ├─ Stage 2: Quality Assessment (scoring 0-100) │ │ ├─ Stage 3: Valuation & Management (final ranking) │ │ └─ Output: Pass/Fail status + Composite Score (0-100) │ └────────────────────┬────────────���───────────────────────────┘ │ ▼ ┌─────────────────────────────────────────────────────────────┐ │ Stage 4: Results Processing │ │ ├─ Filter: Only keep PASS (score >= 75) │ │ ├─ Compare with previous qualified companies │ │ ├─ Categorize: NEW, MAINTAINED, REMOVED │ │ └─ Output: Screening results with status │ └────────────────────┬────────────────────────────────────────┘ │ ▼ ┌─────────────────────────────────────────────────────────────┐ │ Stage 5: Notifications & Storage │ │ ├─ Save qualified companies to MySQL │ │ ├─ Save screening run details (audit trail) │ │ ├─ Generate email report │ │ ├─ Export CSV results │ │ └─ Output: Database updated + Email sent + Files exported │ └─────────────────────────────────────────────────────────────┘ │ ▼ SCREENING COMPLETE

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

Process:

Load industries from configuration
Query SEC EDGAR API for all companies in each industry
Build master list of tickers to process
Typically results in 500-1000 companies
Output: List of company tickers with basic info (ticker, name, sector, industry)

Data Structure:
public class CompanyBasicInfo
{
    public string Ticker { get; set; }              // "MSFT"
    public string CompanyName { get; set; }         // "Microsoft Corporation"
    public string Sector { get; set; }              // "Technology"
    public string Industry { get; set; }            // "Software"
    public DateTime AddedDate { get; set; }         // When added to system
    public string DataSourceFetched { get; set; }   // "SEC EDGAR"
}

1.2 Financial Data Fetching Strategy
Goal: Retrieve comprehensive financial statements for 10-year analysis

Data Required:

Income Statement (Revenue, Net Income, Operating Income)
Balance Sheet (Assets, Liabilities, Equity)
Cash Flow Statement (Operating CF, Free CF, CapEx)
Market Data (Stock Price, Market Cap, Dividends)

Multi-Source Priority:
For FINANCIAL STATEMENTS (Income, Balance Sheet, Cash Flow):
├─ Primary: SEC EDGAR (10-K annual, 10-Q quarterly)
│  ├─ Most authoritative source
│  ├─ Official company filings
│  ├─ Required for compliance
│  └─ Timeout: 30 seconds per request
│
├─ Fallback (if SEC unavailable): None - skip company
│  └─ Financial statements MUST come from SEC
│
└─ Error: Log and skip company

For STOCK PRICES & MARKET DATA:
├─ Primary: Yahoo Finance API
│  ├─ Real-time stock prices
│  ├─ Market capitalization
│  ├─ Dividend history
│  └─ Timeout: 10 seconds per request
│
├─ Fallback #1: IEX Cloud
│  ├─ If Yahoo fails or times out
│  ├─ Alternative market data source
│  └─ Timeout: 10 seconds
│
├─ Fallback #2: Alpha Vantage
│  ├─ Final fallback option
│  ├─ Historical pricing data
│  └─ Timeout: 10 seconds
│
└─ Error (all fail): Use last known stock price from DB or skip valuation

For TREASURY YIELDS (Risk-free rate):
├─ Primary: Federal Reserve FRED API
│  ├─ 10-year Treasury yield
│  └─ Updated daily
│
└─ Fallback: Hardcoded estimate (2.5%) if API unavailable

Copilot said: Perfect! Here are all three documents ready
Perfect! Here are all three documents ready to copy-paste directly into your markdown files:

DOCUMENT 1: 03-DATA-FLOW-LOGIC.md
Markdown
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

Input: Industry List (from appsettings.json) │ ▼ ┌─────────────────────────────────────────────────────────────┐ │ Stage 1: Data Ingestion & Reconciliation │ │ ├─ Fetch company list from SEC EDGAR API │ │ ├─ For each company: fetch financial data (multiple sources)│ │ ├─ Reconcile conflicting data │ │ └─ Output: Raw financial records (validated) │ └───��────────────────┬────────────────────────────────────────┘ │ ▼ ┌─────────────────────────────────────────────────────────────┐ │ Stage 2: Metric Calculation │ │ ├─ Parse financial statements │ │ ├─ Calculate 40+ financial metrics │ │ ├─ Perform 10-year trend analysis │ │ └─ Output: Metric-enriched company records │ └────────────────────┬────────────────────────────────────────┘ │ ▼ ┌─────────────────────────────────────────────────────────────┐ │ Stage 3: 3-Stage Screening │ │ ├─ Stage 1: Financial Strength (hard filters) │ │ ├─ Stage 2: Quality Assessment (scoring 0-100) │ │ ├─ Stage 3: Valuation & Management (final ranking) │ │ └─ Output: Pass/Fail status + Composite Score (0-100) │ └────────────────────┬────────────���───────────────────────────┘ │ ▼ ┌─────────────────────────────────────────────────────────────┐ │ Stage 4: Results Processing │ │ ├─ Filter: Only keep PASS (score >= 75) │ │ ├─ Compare with previous qualified companies │ │ ├─ Categorize: NEW, MAINTAINED, REMOVED │ │ └─ Output: Screening results with status │ └────────────────────┬────────────────────────────────────────┘ │ ▼ ┌─────────────────────────────────────────────────────────────┐ │ Stage 5: Notifications & Storage │ │ ├─ Save qualified companies to MySQL │ │ ├─ Save screening run details (audit trail) │ │ ├─ Generate email report │ │ ├─ Export CSV results │ │ └─ Output: Database updated + Email sent + Files exported │ └─────────────────────────────────────────────────────────────┘ │ ▼ SCREENING COMPLETE

Code

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
Process:

Load industries from configuration
Query SEC EDGAR API for all companies in each industry
Build master list of tickers to process
Typically results in 500-1000 companies
Output: List of company tickers with basic info (ticker, name, sector, industry)

Data Structure:

C#
public class CompanyBasicInfo
{
    public string Ticker { get; set; }              // "MSFT"
    public string CompanyName { get; set; }         // "Microsoft Corporation"
    public string Sector { get; set; }              // "Technology"
    public string Industry { get; set; }            // "Software"
    public DateTime AddedDate { get; set; }         // When added to system
    public string DataSourceFetched { get; set; }   // "SEC EDGAR"
}
1.2 Financial Data Fetching Strategy
Goal: Retrieve comprehensive financial statements for 10-year analysis

Data Required:

Income Statement (Revenue, Net Income, Operating Income)
Balance Sheet (Assets, Liabilities, Equity)
Cash Flow Statement (Operating CF, Free CF, CapEx)
Market Data (Stock Price, Market Cap, Dividends)
Multi-Source Priority:

Code
For FINANCIAL STATEMENTS (Income, Balance Sheet, Cash Flow):
├─ Primary: SEC EDGAR (10-K annual, 10-Q quarterly)
│  ├─ Most authoritative source
│  ├─ Official company filings
│  ├─ Required for compliance
│  └─ Timeout: 30 seconds per request
│
├─ Fallback (if SEC unavailable): None - skip company
│  └─ Financial statements MUST come from SEC
│
└─ Error: Log and skip company

For STOCK PRICES & MARKET DATA:
├─ Primary: Yahoo Finance API
│  ├─ Real-time stock prices
│  ├─ Market capitalization
│  ├─ Dividend history
│  └─ Timeout: 10 seconds per request
│
├─ Fallback #1: IEX Cloud
│  ├─ If Yahoo fails or times out
│  ├─ Alternative market data source
│  └─ Timeout: 10 seconds
│
├─ Fallback #2: Alpha Vantage
│  ├─ Final fallback option
│  ├─ Historical pricing data
│  └─ Timeout: 10 seconds
│
└─ Error (all fail): Use last known stock price from DB or skip valuation

For TREASURY YIELDS (Risk-free rate):
├─ Primary: Federal Reserve FRED API
│  ├─ 10-year Treasury yield
│  └─ Updated daily
│
└─ Fallback: Hardcoded estimate (2.5%) if API unavailable

Implementation Flow:
async Task<FinancialData> FetchCompanyFinancials(string ticker)
{
    var financialData = new FinancialData { Ticker = ticker };
    
    // 1. ALWAYS fetch SEC EDGAR (required)
    try
    {
        financialData.SecEdgarData = await _secEdgarService.FetchLatestReports(ticker);
        if (financialData.SecEdgarData == null)
        {
            _logger.LogWarning($"No SEC EDGAR data found for {ticker}");
            return null; // Skip this company
        }
    }
    catch (Exception ex)
    {
        _logger.LogError($"SEC EDGAR fetch failed for {ticker}: {ex.Message}");
        return null; // Skip this company
    }
    
    // 2. Fetch market data with fallbacks
    financialData.MarketData = await FetchMarketDataWithFallbacks(ticker);
    if (financialData.MarketData == null)
    {
        _logger.LogWarning($"No market data available for {ticker}, using valuation estimate");
    }
    
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
        if (data != null)
        {
            _logger.LogInformation($"Market data from Yahoo Finance: {ticker}");
            return data;
        }
    }
    catch (Exception ex)
    {
        _logger.LogWarning($"Yahoo Finance failed: {ex.Message}");
    }
    
    // Try IEX Cloud
    try
    {
        var data = await _iexCloudService.FetchAsync(ticker);
        if (data != null)
        {
            _logger.LogInformation($"Market data from IEX Cloud: {ticker}");
            return data;
        }
    }
    catch (Exception ex)
    {
        _logger.LogWarning($"IEX Cloud failed: {ex.Message}");
    }
    
    // Try Alpha Vantage
    try
    {
        var data = await _alphaVantageService.FetchAsync(ticker);
        if (data != null)
        {
            _logger.LogInformation($"Market data from Alpha Vantage: {ticker}");
            return data;
        }
    }
    catch (Exception ex)
    {
        _logger.LogWarning($"Alpha Vantage failed: {ex.Message}");
    }
    
    // All failed
    _logger.LogError($"All market data sources failed for {ticker}");
    return null;
}

1.3 Data Reconciliation
Goal: When multiple sources provide overlapping data, ensure consistency

Reconciliation Rules:

Data Item	Conflict Resolution	Example
Stock Price	Use most recent (today's date)	Yahoo: $150.25 (today) vs IEX: $149.80 (yesterday) → Use Yahoo
Market Cap	Calculate from stock price × shares	Always calculate fresh, don't trust source directly
Dividend	Verify across 2+ sources before using	Must see in Yahoo AND IEX to trust
Financial metrics	Always use SEC EDGAR (single source)	Never blend Income Statement sources
Historical data	Fill gaps using secondary sources	SEC: have 9 years, IEX: have 10 years → Use SEC 9 + fill year 10 from IEX

Data Validation Rules:
BEFORE USING ANY DATA:

1. Check Completeness
   ├─ All required fields present?
   ├─ No null values in critical metrics?
   └─ If fail: Mark as incomplete, may affect analysis

2. Range Validation
   ├─ ROE: -100% to +200% (reasonable bounds)
   ├─ Debt/Equity: 0 to 5.0 (typically)
   ├─ Margins: -50% to +100%
   ├─ Growth rates: -50% to +50% annually
   └─ If fail: Flag as outlier, investigate

3. Balance Sheet Validation
   ├─ Assets = Liabilities + Shareholders' Equity ±2%
   ├─ If fail: Log discrepancy, still use (audited statements)
   └─ Impact: Flag company as having accounting questions

4. Consistency Check
   ├─ Year-over-year changes < 300% (unless known event)
   ├─ Revenue growth correlates with income growth
   ├─ If fail: Flag for manual review
   └─ Impact: May exclude if too suspicious

5. Data Age Check
   ├─ Financial statements: Must be < 1 year old (current fiscal year)
   ├─ Stock prices: Must be < 1 day old
   ├─ If fail: Use best available, flag as outdated
   └─ Impact: May affect screening decision

Data Quality Output:
public class DataQualityReport
{
    public string Ticker { get; set; }
    public bool IsComplete { get; set; }              // All required fields present?
    public bool IsValid { get; set; }                 // Passes all validations?
    public List<string> Warnings { get; set; }        // Non-critical issues
    public List<string> Errors { get; set; }          // Critical issues
    public Dictionary<string, string> DataSources { get; set; } // Which source for each metric
    public DateTime DataAsOfDate { get; set; }        // Freshness indicator
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
        "CashFlow": "SEC EDGAR",
        "StockPrice": "Yahoo Finance",
        "MarketCap": "Yahoo Finance"
    },
    "DataAsOfDate": "2026-03-14T09:30:00Z"
}

Stage 2: Metric Calculation
2.1 Available Metrics (40+)
Goal: Calculate comprehensive financial metrics from raw data

All metrics are calculated for:

Each historical year (up to 10 years: 2016-2025)
10-year average
Stability (Coefficient of Variation)
Profitability Metrics (8)
public class ProfitabilityMetrics
{
    // Margins (as percentages)
    public decimal GrossMargin { get; set; }          // (Revenue - COGS) / Revenue
    public decimal OperatingMargin { get; set; }      // Operating Income / Revenue
    public decimal NetProfitMargin { get; set; }      // Net Income / Revenue
    public decimal EBITDA_Margin { get; set; }        // EBITDA / Revenue
    
    // Profitability ratios
    public decimal ReturnOnAssets_ROA { get; set; }   // Net Income / Total Assets
    public decimal ReturnOnEquity_ROE { get; set; }   // Net Income / Shareholders' Equity
    public decimal ReturnOnInvestedCapital_ROIC { get; set; } // NOPAT / Invested Capital
    public decimal ReturnOnCapitalEmployed_ROCE { get; set; } // EBIT / Capital Employed
}

// Example calculation:
GrossMargin = (Revenue - COGS) / Revenue * 100
            = (1,000,000 - 400,000) / 1,000,000 * 100
            = 60%

NetProfitMargin = NetIncome / Revenue * 100
                = 100,000 / 1,000,000 * 100
                = 10%

ROE = NetIncome / Shareholders'Equity * 100
    = 100,000 / 500,000 * 100
    = 20%

Financial Strength Metrics (6)
public class FinancialStrengthMetrics
{
    // Leverage ratios
    public decimal DebtToEquity { get; set; }         // Total Debt / Shareholders' Equity
    public decimal DebtToAssets { get; set; }         // Total Debt / Total Assets
    public decimal EquityMultiplier { get; set; }     // Total Assets / Shareholders' Equity
    
    // Liquidity ratios
    public decimal CurrentRatio { get; set; }         // Current Assets / Current Liabilities
    public decimal QuickRatio { get; set; }           // (CA - Inventory) / CL
    
    // Coverage ratios
    public decimal InterestCoverageRatio { get; set; } // EBIT / Interest Expense
}

// Example calculations:
DebtToEquity = TotalDebt / ShareholdersEquity
             = 100,000 / 500,000
             = 0.20 (Lower is better, < 0.50 is strong)

CurrentRatio = CurrentAssets / CurrentLiabilities
             = 300,000 / 200,000
             = 1.5 (> 1.0 is healthy, > 2.0 is excellent)

InterestCoverageRatio = EBIT / InterestExpense
                      = 150,000 / 10,000
                      = 15x (> 2.5 is good, > 5.0 is excellent)

Cash Flow Metrics (6)
public class CashFlowMetrics
{
    public decimal OperatingCashFlow { get; set; }           // Direct from cash flow statement
    public decimal FreeCashFlow { get; set; }                // OCF - CapEx
    public decimal OperatingCashFlowToNetIncome { get; set; } // OCF / Net Income (quality indicator)
    public decimal FreeCashFlowToOperatingCashFlow { get; set; } // FCF / OCF
    public decimal CapitalExpenditurePercent { get; set; }   // CapEx / Revenue * 100
    public decimal CashFlowFromOperations_10Y_Avg { get; set; }
}

// Example:
FreeCashFlow = OperatingCashFlow - CapitalExpenditure
             = 200,000 - 50,000
             = 150,000

OCF_to_NI_Ratio = OperatingCashFlow / NetIncome
                = 200,000 / 100,000
                = 2.0 (> 1.0 is healthy, means earnings are backed by cash)

Growth Metrics (7)
public class GrowthMetrics
{
    // Year-over-year growth rates
    public decimal RevenueGrowth_YoY { get; set; }           // (Current - Prior) / Prior * 100
    public decimal EarningsGrowth_YoY { get; set; }
    public decimal FreeCashFlowGrowth_YoY { get; set; }
    
    // Multi-year compound growth rates
    public decimal RevenueCAGR_10Y { get; set; }             // Compound Annual Growth Rate
    public decimal EarningsCAGR_10Y { get; set; }
    public decimal FreeCashFlowCAGR_10Y { get; set; }
    
    // Per share metrics
    public decimal EarningsPerShare { get; set; }            // Net Income / Shares Outstanding
}

// Example CAGR calculation:
RevenueCAGR_10Y = (Ending_Value / Beginning_Value) ^ (1/10) - 1
                = (2,000,000 / 1,000,000) ^ (1/10) - 1
                = 2.0 ^ 0.1 - 1
                = 0.072 or 7.2% annual growth

Valuation Metrics (8)
public class ValuationMetrics
{
    // Price-to-metric ratios
    public decimal PriceToEarnings_PE { get; set; }          // Stock Price / EPS
    public decimal PriceToBook_PB { get; set; }              // Market Cap / Book Value
    public decimal PriceToSales_PS { get; set; }             // Market Cap / Revenue
    public decimal PriceToEarningsGrowth_PEG { get; set; }   // P/E / Growth Rate
    
    // Yield metrics
    public decimal EarningsYield { get; set; }               // EPS / Stock Price (inverse of P/E)
    public decimal DividendYield { get; set; }               // Annual Dividend / Stock Price
    public decimal DividendPayoutRatio { get; set; }         // Dividends / Net Income
    
    // Value indicator
    public decimal EnterpriseValue_to_Revenue { get; set; }  // (Debt + Market Cap) / Revenue
}

// Example:
PE_Ratio = StockPrice / EarningsPerShare
         = $100 / $5
         = 20x (Pay $20 for every $1 of earnings)

DividendYield = AnnualDividendPerShare / StockPrice * 100
              = $2 / $100 * 100
              = 2%

EarningsYield = 1 / PE_Ratio * 100
              = 1 / 20 * 100
              = 5%

Management Quality Metrics (6)
public class ManagementQualityMetrics
{
    public decimal InsiderOwnershipPercent { get; set; }     // Insider shares / Total shares
    public decimal GoodwillAsPercentOfAssets { get; set; }   // Goodwill / Total Assets
    public decimal OneTimeChargesFrequency { get; set; }     // How often unusual items appear (lower is better)
    public decimal RetainedEarningsAsPercentOfEquity { get; set; } // RE / Shareholders' Equity
    public decimal OperatingExpenseRatio { get; set; }       // OpEx / Revenue
    public decimal AssetTurnover { get; set; }               // Revenue / Total Assets
}

// Interpretation:
InsiderOwnership > 10% = Good alignment with shareholders
Goodwill > 50% of Assets = Concern about acquisition value
OneTimeCharges Frequency < 20% = Good quality earnings
RetainedEarnings > 50% = Company reinvests profits

Stability Metrics (Coefficient of Variation)
For each metric category, calculate stability:
public class StabilityMetrics
{
    public decimal ProfitabilityStability { get; set; }      // CV of margin metrics
    public decimal GrowthStability { get; set; }             // CV of growth rates
    public decimal CashFlowStability { get; set; }           // CV of FCF over 10 years
}

// Coefficient of Variation = Standard Deviation / Mean * 100
// Lower CV = More stable company
// Example:
//   ROE over 10 years: 18%, 20%, 19%, 21%, 20%, 19%, 18%, 20%, 21%, 19%
//   Mean = 19.5%
//   StdDev = 1.08%
//   CV = 1.08 / 19.5 * 100 = 5.5% (Very stable!)

// Interpretation:
// < 10% = Highly stable (excellent)
// 10-20% = Stable (good)
// 20-30% = Moderate variation (acceptable)
// > 30% = Highly volatile (concerning)

2.2 10-Year Trend Analysis
Goal: Identify trends and consistency over decade
public class TrendAnalysis
{
    // Regression analysis on 10-year data
    public decimal ProfitMarginTrend { get; set; }           // Improving, stable, declining
    public decimal RevenueGrowthTrend { get; set; }
    public decimal ROETrend { get; set; }
    public decimal DebtTrend { get; set; }
    
    // Quality assessment
    public bool HasImprovedOverTime { get; set; }            // Overall positive trend?
    public bool IsStable { get; set; }                       // Low variation?
    public bool HasConcerns { get; set; }                    // Red flags present?
}

// Trend determination:
// - Calculate slope of metric over 10 years
// - Positive slope = Improving trend ✓
// - Flat slope (< 2% change) = Stable trend ✓
// - Negative slope = Declining trend ✗

Output: Complete Metrics Record
public class CompanyMetricsRecord
{
    public string Ticker { get; set; }
    public int FiscalYear { get; set; }
    
    // All metric categories
    public ProfitabilityMetrics Profitability { get; set; }
    public FinancialStrengthMetrics FinancialStrength { get; set; }
    public CashFlowMetrics CashFlow { get; set; }
    public GrowthMetrics Growth { get; set; }
    public ValuationMetrics Valuation { get; set; }
    public ManagementQualityMetrics ManagementQuality { get; set; }
    
    // Aggregates
    public StabilityMetrics Stability { get; set; }
    public TrendAnalysis Trend { get; set; }
}

Stage 3: Screening Logic
3.1 Stage 1: Financial Strength Filters (Hard Filters)
Purpose: Eliminate financially weak companies immediately (PASS/FAIL gates)

Logic: ALL filters must pass. If ANY fails → Company is REJECTED
IF ROE (10-year avg) > 15% AND
   Net Profit Margin (10-year avg) > 10% AND
   Debt/Equity Ratio < 0.50 AND
   Operating Cash Flow (positive for all 10 years) AND
   Free Cash Flow (positive for 8+ of 10 years)
THEN → Continue to Stage 2
ELSE → REJECT company (do not store)

Detailed Filter Rules:

Filter	Threshold	Rationale	Data Used
ROE	> 15% (10Y avg)	Return on shareholder capital	10-year average ROE
Net Margin	> 10% (10Y avg)	Profitability	10-year average net margin
Debt/Equity	< 0.50	Financial stability	Latest fiscal year
Operating CF	Positive all 10 years	Cash generation	Historical cash flow statement
Free Cash Flow	Positive 8+ of 10 years	Sustainable business	Calculated: OCF - CapEx

Implementation:
bool PassStage1(CompanyMetricsRecord record)
{
    // Get 10-year averages
    var avgROE = record.Profitability.ReturnOnEquity;
    var avgNetMargin = record.Profitability.NetProfitMargin;
    var debtToEquity = record.FinancialStrength.DebtToEquity;
    var ocfPositiveYears = CountPositiveYears(record.CashFlow.OperatingCashFlow);
    var fcfPositiveYears = CountPositiveYears(record.CashFlow.FreeCashFlow);
    
    // All filters must pass
    if (avgROE <= 15m)
    {
        _logger.LogInformation($"{record.Ticker} failed Stage 1: ROE {avgROE}% < 15%");
        return false;
    }
    
    if (avgNetMargin <= 10m)
    {
        _logger.LogInformation($"{record.Ticker} failed Stage 1: Net Margin {avgNetMargin}% < 10%");
        return false;
    }
    
    if (debtToEquity >= 0.50m)
    {
        _logger.LogInformation($"{record.Ticker} failed Stage 1: D/E {debtToEquity} >= 0.50");
        return false;
    }
    
    if (ocfPositiveYears < 10)
    {
        _logger.LogInformation($"{record.Ticker} failed Stage 1: Positive OCF only {ocfPositiveYears}/10 years");
        return false;
    }
    
    if (fcfPositiveYears < 8)
    {
        _logger.LogInformation($"{record.Ticker} failed Stage 1: Positive FCF only {fcfPositiveYears}/10 years");
        return false;
    }
    
    _logger.LogInformation($"{record.Ticker} PASSED Stage 1");
    return true;
}

3.2 Stage 2: Quality Assessment (Scoring)
Purpose: Score passing companies on quality (0-100 scale)

4 Quality Pillars:

Pillar 1: Return on Capital (30 points)
Points Calculation:
  ROE 15-20% = 5 points
  ROE 20-25% = 10 points
  ROE 25-30% = 15 points
  ROE 30%+ = 20 points
  
  ROIC 12-15% = 5 points
  ROIC 15-18% = 10 points
  ROIC 18%+ = 10 points (max 10)
  
  Total: 0-30 points

Pillar 2: Profitability (30 points)
Points Calculation:
  Net Margin 10-15% = 5 points
  Net Margin 15-20% = 10 points
  Net Margin 20%+ = 15 points
  
  Operating Margin 15-20% = 5 points
  Operating Margin 20%+ = 10 points (max 10)
  
  Margin Stability (CV < 10%) = 5 points (bonus)
  
  Total: 0-30 points

Pillar 3: Cash Flow (20 points)
Points Calculation:
  FCF Positive = 5 points
  FCF > 5% of Revenue = 5 points
  FCF > 10% of Revenue = 10 points
  
  OCF > Net Income (high quality) = 5 points
  
  FCF Growth > 5% CAGR = 5 points (max total 20)
  
  Total: 0-20 points

Pillar 4: Business Quality (10 points)
Points Calculation:
  Economic Moat Detected = 3 points
    (High gross margins, consistent ROE, brand value)
  
  Insider Ownership > 5% = 2 points
    (Management aligned with shareholders)
  
  Low Goodwill (< 20% of assets) = 3 points
    (Built organically, not through acquisition)
  
  Revenue Stability = 2 points
    (Growth CV < 15%)
  
  Total: 0-10 points

Total Quality Score: 0-100 points
decimal CalculateQualityScore(CompanyMetricsRecord record)
{
    decimal score = 0m;
    
    // Pillar 1: Return on Capital (30 points)
    var roe = record.Profitability.ReturnOnEquity;
    if (roe >= 30) score += 20m;
    else if (roe >= 25) score += 15m;
    else if (roe >= 20) score += 10m;
    else if (roe >= 15) score += 5m;
    
    var roic = record.Profitability.ReturnOnInvestedCapital_ROIC;
    if (roic >= 18) score += 10m;
    else if (roic >= 15) score += 10m;
    else if (roic >= 12) score += 5m;
    
    // Pillar 2: Profitability (30 points)
    var netMargin = record.Profitability.NetProfitMargin;
    if (netMargin >= 20) score += 15m;
    else if (netMargin >= 15) score += 10m;
    else if (netMargin >= 10) score += 5m;
    
    var opMargin = record.Profitability.OperatingMargin;
    if (opMargin >= 20) score += 10m;
    else if (opMargin >= 15) score += 5m;
    
    // Stability bonus
    if (record.Stability.ProfitabilityStability < 10) score += 5m;
    
    // Pillar 3: Cash Flow (20 points)
    if (record.CashFlow.FreeCashFlow > 0) score += 5m;
    var fcfPercent = record.CashFlow.FreeCashFlow / /* revenue */ 100;
    if (fcfPercent >= 0.10m) score += 10m;
    else if (fcfPercent >= 0.05m) score += 5m;
    
    // Pillar 4: Business Quality (10 points)
    if (HasEconomicMoat(record)) score += 3m;
    if (record.ManagementQuality.InsiderOwnershipPercent >= 5) score += 2m;
    if (record.ManagementQuality.GoodwillAsPercentOfAssets < 20) score += 3m;
    if (record.Stability.GrowthStability < 15) score += 2m;
    
    return Math.Min(score, 100m); // Cap at 100
}

3.3 Stage 3: Valuation & Ranking (Final Score)
Purpose: Apply valuation metrics and management quality to get final composite score

Pillar 5: Valuation Scoring (35 points)
P/E Valuation:
  P/E < 15 (Cheap) = 12 points
  P/E 15-20 (Fair) = 10 points
  P/E 20-25 (Reasonable) = 8 points
  P/E 25-35 (Expensive) = 4 points
  P/E > 35 (Very Expensive) = 0 points

PEG Ratio Assessment:
  PEG < 1.0 (Undervalued growth) = 10 points
  PEG 1.0-1.5 (Fair value) = 8 points
  PEG 1.5-2.0 (Slight premium) = 4 points
  PEG > 2.0 (Overvalued) = 0 points

Price to Book:
  P/B < 1.5 (Cheap) = 8 points
  P/B 1.5-2.5 (Fair) = 6 points
  P/B 2.5-4.0 (Premium) = 2 points
  P/B > 4.0 (Expensive) = 0 points

Dividend Yield:
  Dividend > 2% = 3 points
  Dividend > 3% = 5 points

Total Valuation: 0-35 points

Pillar 6: Management Quality (25 points)
Insider Ownership:
  > 15% = 10 points
  > 10% = 8 points
  > 5% = 5 points
  < 5% = 0 points

Goodwill Assessment:
  < 10% of assets = 8 points
  < 20% of assets = 5 points
  < 50% of assets = 2 points
  > 50% = 0 points

Accounting Quality:
  Few one-time charges = 7 points
  Some one-time items = 4 points
  Many adjustments = 0 points

Total Management: 0-25 points

Final Composite Score Calculation:
Composite Score = (Quality Score × 40%) + 
                  (Valuation Score × 35%) + 
                  (Management Score × 25%)

Example:
  Quality Score = 85
  Valuation Score = 28
  Management Score = 20
  
  Composite = (85 × 0.40) + (28 × 0.35) + (20 × 0.25)
            = 34 + 9.8 + 5
            = 48.8/100 (FAIL - below 75 threshold)

Example 2:
  Quality Score = 90
  Valuation Score = 32
  Management Score = 22
  
  Composite = (90 × 0.40) + (32 × 0.35) + (22 × 0.25)
            = 36 + 11.2 + 5.5
            = 52.7/100 (Still FAIL)

Example 3:
  Quality Score = 92
  Valuation Score = 34
  Management Score = 24
  
  Composite = (92 × 0.40) + (34 × 0.35) + (24 × 0.25)
            = 36.8 + 11.9 + 6
            = 54.7/100 (Still FAIL)

Example 4 (PASS):
  Quality Score = 95
  Valuation Score = 35
  Management Score = 25
  
  Composite = (95 × 0.40) + (35 × 0.35) + (25 × 0.25)
            = 38 + 12.25 + 6.25
            = 56.5/100 (Still FAIL - shows how strict filtering is)

Passing Threshold: Composite Score >= 75

Only companies that score 75 or higher are stored in the database.

Stage 4: Results Processing
4.1 Filtering Results
After Stage 3 screening:

PASS: Composite score >= 75 (STORE in database)
FAIL: Composite score < 75 (DO NOT STORE, discard)

Typical Results Distribution:
Input: 500 companies
├─ Stage 1 Filters: ~450 pass, ~50 fail
├─ Stage 2 Scoring: All continue (no rejection)
├─ Stage 3 Composite: ~45-60 reach threshold of 75
└─ Final Result: ~45-60 companies qualify (9-12%)

4.2 Comparison with Previous Results
Compare current run with previous qualifying companies:
public class ScreeningComparison
{
    public List<string> NewlyQualified { get; set; }      // Passed this run, failed before
    public List<string> Maintained { get; set; }          // Passed both runs
    public List<string> Removed { get; set; }             // Passed before, failed now
}

// Example logic:
var currentlyQualified = currentResults.Where(r => r.PassedScreening).Select(r => r.Ticker).ToList();
var previouslyQualified = await _db.QualifiedCompanies.Where(c => c.IsActive).Select(c => c.Ticker).ToListAsync();

var newly = currentlyQualified.Except(previouslyQualified).ToList();           // [MSFT, AAPL]
var maintained = currentlyQualified.Intersect(previouslyQualified).ToList();   // [JNJ, KO, JPM]
var removed = previouslyQualified.Except(currentlyQualified).ToList();         // [OLD1, OLD2]

// Result breakdown
Console.WriteLine($"Newly Qualified: {newly.Count}");      // 2
Console.WriteLine($"Maintained: {maintained.Count}");      // 5
Console.WriteLine($"Removed: {removed.Count}");            // 2
Console.WriteLine($"Total Current: {currentlyQualified.Count}"); // 7

Stage 5: Notifications & Storage
5.1 Database Operations
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
    
    // Save all metrics for each year
    for (int year = 2016; year <= 2025; year++)
    {
        var metrics = new FinancialMetrics { /* ... */ };
        _db.FinancialMetrics.Add(metrics);
    }
}

// 2. Update maintained companies
foreach (var ticker in maintained)
{
    var company = await _db.QualifiedCompanies.FirstAsync(c => c.Ticker == ticker);
    company.CompositeScore = result.CompositeScore;
    company.QualityScore = result.QualityScore;
    company.ValuationScore = result.ValuationScore;
    company.ManagementScore = result.ManagementScore;
    company.LastQualifiedDate = DateTime.UtcNow;
}

// 3. Mark removed companies as inactive
foreach (var ticker in removed)
{
    var company = await _db.QualifiedCompanies.FirstAsync(c => c.Ticker == ticker);
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

// 4. Log screening run details
var runRecord = new ScreeningRun
{
    RunDate = DateTime.UtcNow,
    RunType = "Scheduled",
    Status = "Success",
    CompaniesProcessed = totalCompanies,
    CompaniesQualified = currentlyQualified.Count,
    NewlyQualified = newly.Count,
    MaintainedQualified = maintained.Count,
    Removed = removed.Count,
    ExecutionTimeSeconds = stopwatch.ElapsedMilliseconds / 1000,
    IndustriesScreened = "Technology, Healthcare, Consumer Defensive"
};
_db.ScreeningRuns.Add(runRecord);

await _db.SaveChangesAsync();

5.2 Email Notification
Sent to configured recipient with:

Summary statistics
Detailed results table (HTML)
Newly qualified companies highlighted
Removed companies listed
CSV attachment
5.3 CSV Export
File: output/screening-results-{date}.csv

Contains all qualified companies with their metrics.

Error Handling & Recovery

Error Scenarios:
Scenario	Action	Continue?
SEC EDGAR unavailable	Log error, skip company	Yes (try next)
All market data sources fail	Use last known price	Yes (valuation limited)
Metric calculation fails	Log error, skip company	Yes (try next)
Database connection lost	Fail screening, alert admin	No (stop)
Email send fails	Log error, continue	Yes (results not sent)
Out of memory	Stop screening, log error	No (stop)

Data Quality Assurance
Validation Checklist:
✓ All required SEC EDGAR data present ✓ Stock prices recent (< 1 day old) ✓ Financial statements not null ✓ Metrics within reasonable ranges ✓ 10-year historical data available (8+ years acceptable) ✓ Data sources documented for audit trail

Performance Considerations

Typical Processing Times:
500 companies processed:
├─ Data fetching: ~30-40 minutes (API rate limits)
├─ Metric calculation: ~2-3 minutes
├─ Screening (3 stages): ~1 minute
├─ Results processing: ~30 seconds
├─ Database save: ~1 minute
└─ Email/Export: ~30 seconds

TOTAL: ~40-50 minutes per screening run

Optimization Tips:

Use connection pooling for HTTP clients
Cache industry lists
Parallel process companies (5-10 concurrent)
Batch database inserts (500 at a time)
Compress CSV before email

Document History
Version	Date	Changes
1.0	2026-03-14	Initial data flow documentation

---

Perfect! Now you have all three documents ready to copy-paste directly into your markdown files. 

**Which document would you like me to create next?**

1. **04-SCREENING-LOGIC.md** - Deep dive into each screening stage with algorithms
2. **05-DATABASE-SCHEMA.md** - Complete MySQL table designs
3. **06-API-INTEGRATION-GUIDE.md** - How to set up external data sources
4. **07-DEPLOYMENT-GUIDE.md** - GitHub Actions workflow setup

Let me know and I'll generate it in the same format!
