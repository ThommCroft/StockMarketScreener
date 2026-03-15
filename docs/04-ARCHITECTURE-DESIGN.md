# Architecture Design Document - Stock Market Screener

**Version:** 4.0 
**Date:** 2026-03-15  
**Author:** ThommCroft
**Stack:** C# .NET 8.0, MySQL 8.0, GitHub Actions (Quarterly + Manual), Console Application  
**Deployment:** Local MySQL + GitHub Actions automation  
**Scope:** Complete 7-Stage Pipeline (Stages 0-7)

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [System Architecture Overview](#system-architecture-overview)
3. [Technology Stack](#technology-stack)
4. [Core Components](#core-components)
5. [Data Architecture (MySQL)](#data-architecture-mysql)
6. [Industry Configuration](#industry-configuration)
7. [Screening Workflow (Stages 0-7)](#screening-workflow-stages-0-7)
8. [Unified Metric Calculation & Hard Filter Evaluation](#unified-metric-calculation--hard-filter-evaluation)
9. [Data Reconciliation Layer](#data-reconciliation-layer)
10. [Valuation Engine (Stage 6)](#valuation-engine-stage-6)
11. [Results & Reporting (Stage 7)](#results--reporting-stage-7)
12. [GitHub Actions Automation](#github-actions-automation)
13. [Stock Price Update Job](#stock-price-update-job)
14. [Testing Strategy](#testing-strategy)
15. [Deployment & Operations](#deployment--operations)

---

## Executive Summary

The StockMarketScreener is a **console-based financial analysis engine** that implements a complete 7-stage investment screening pipeline based on Warren Buffett and Charlie Munger principles:

**Complete 7-Stage Pipeline:**
- **Stages 0-5:** Financial Analysis (filtering, quality assessment, composite scoring)
- **Stage 6:** Intrinsic Value Valuation (5 independent methods)
- **Stage 7:** Results Reporting (MySQL storage, CSV export, GitHub summary)

**Core Capabilities:**
- ✅ **Screens 6 default industries** (configurable to any industry)
- ✅ **Evaluates all companies** in selected industries quarterly/annually
- ✅ **Calculates ALL 40+ metrics** for every company BEFORE any filtering
- ✅ **Applies comprehensive hard filter evaluation** using complete metric data
- ✅ **Only stores passing companies** - complete financial metrics dataset
- ✅ **Automatically removes companies** that fall below thresholds
- ✅ **Calculates intrinsic value** using 5 independent valuation methods (DCF, DDM, Multiples, Buffett, Munger)
- ✅ **Determines margin of safety** and investment recommendations
- ✅ **Generates detailed results** in CSV format for analysis
- ✅ **Updates stock prices weekly** (separate background job, no email)
- ✅ **Runs automatically via GitHub Actions** (quarterly + manual on-demand)
- ✅ **Stores all data in MySQL** for historical tracking and analysis

**Key Design Principles:**
- ✅ Solo developer friendly
- ✅ Unified metric calculation + hard filter evaluation (merged approach)
- ✅ No early rejection based on incomplete analysis
- ✅ Complete financial picture before any decision
- ✅ Only stores passing companies (no clutter)
- ✅ Results-driven (CSV export for external tools)
- ✅ Minimal operational overhead
- ✅ Data reconciliation from multiple sources
- ✅ Comprehensive audit trail

---

## System Architecture Overview

### High-Level Data Flow (Complete 7-Stage Pipeline)


```
┌────────────────────────��────────────────────────────────────┐
│ GitHub Actions Trigger (Quarterly/Manual) │ │ OR Local Console Run
│ └────────────┬──────────────────────────────────────────────┘
│ ┌────────────▼────────────────────────────────────────────────┐
│StockMarketScreener Console Application │
├─────────────────────────────────────────────────────────────┤
│ │ │ ╔═══════════════════════════════════════════════════╗
││ ║ STAGE 0: Market Cap Pre-Filter ║
│ │ ║ Filter: Market Cap > $300M ║
│ │ ║ Output: Liquid, institutional-quality companies ║
│ │ ╚═════════────┬──────────────────────────────────────╝││
│ │ │ ┌────────────▼────────────────────────────────────────┐
│ │ │ STAGE 1: Data Ingestion & Reconciliation
│ │ │ │ ├─ Fetch from SEC EDGAR (10-K/10-Q) │ │ │ │ ├─ Fetch from Yahoo Finance API │ │ │ │ ├─ Fetch from IEX Cloud / Alpha Vantage │ │ │ │ └─ Reconcile & aggregate data from all sources │ │ │ └────────────┬────────────────────────────────────────┘ │ │ │ │ │ ┌────────────▼────────────────────────────────────────┐ │ │ │ STAGE 2-3: Calculate ALL 40+ Metrics │ │ │ │ ├─ Calculate ALL metrics for EVERY company │ │ │ │ ├─ 10-year analysis (prioritize, min 5 years) │ │ │ │ ├─ Year-by-year metrics & trend analysis │ │ │ │ │ │ │ │ │ └─ ⚠️ NO FILTERING - All metrics upfront │ │ │ └────────────┬────────────────────────────────────────┘ │ │ │ │ │ ┌────────────▼────────────────────────────────────────┐ │ │ │ STAGE 3: Hard Filter Evaluation (Unified) │ │ │ │ ├─ Evaluate hard filters using COMPLETE metrics │ │ │ │ ├─ Assess overall financial strength holistically │ │ │ │ └─ Companies below threshold → Skip │ │ │ └────────────┬────────────────────────────────────────┘ │ │ │ │ │ ┌────────────▼────────────────────────────────────────┐ │ │ │ STAGE 4: Quality Assessment Scoring (0-100) │ │ │ │ ├─ Score: Return on Capital (30 points max) │ │ │ │ ├─ Score: Profitability (30 points max) │ │ │ │ ├─ Score: Cash Flow (20 points max) │ │ │ │ └─ Score: Business Quality (10 points max) │ │ │ └────────────┬────────────────────────────────────────┘ │ │ │ │ │ ┌────────────▼────────────────────────────────────────┐ │ │ │ STAGE 5: Composite Scoring & Management Assessment │ │ │ │ ├─ Score: Valuation (max 35 points) │ │ │ │ ├─ Score: Management Quality (max 25 points) │ │ │ │ └─ Composite: (Qual×40% + Val×35% + Mgmt×25%) │ │ │ │ │ │ │ │ PASS: Composite >= 75 → STORE in database │ │ │ │ FAIL: Composite < 75 → SKIP (not stored) │ │ │ └────────────┬────────────────────────────────────────┘ │ │ │ │ │ ┌────────────▼────────────────────────────────────────┐ │ │ │ STAGE 6: Intrinsic Value Valuation (Per Company) │ │ │ │ ├─ Calculate DCF Fair Value │ │ │ │ ├─ Calculate DDM Fair Value (if applicable) │ │ │ │ ├─ Calculate Multiples Fair Value │ │ │ │ ├─ Calculate Buffett Fair Value │ │ │ │ ├─ Calculate Munger Fair Value │ │ │ │ ├─ Consensus Fair Value (weighted average) │ │ │ │ ├─ Margin of Safety Analysis │ │ │ │ └─ Valuation Grade (A+ to F) │ │ │ └────────────┬────────────────────────────────────────┘ │ │ │ │ │ ┌────────────▼────────────────────────────────────────┐ │ │ │ STAGE 7: Results Reporting & Storage │ │ │ │ ├─ Store all valuations in MySQL database │ │ │ │ ├─ Compare with previous results (new/maint/del) │ │ │ │ ├─ Export complete dataset to CSV │ │ │ │ ├─ Generate GitHub summary statistics │ │ │ │ └─ Create console output report │ │ │ └────────────┬────────────────────────────────────────┘ │ └─────────────────────────────────────────────────────────────┘ │ ┌────────┴─────────┬──────────────┐ │ │ │ ▼ ▼ ▼ MySQL CSV Export Console Database (Analysis) Output
```

### Component Architecture

```
StockMarketScreener.Console
├─ Program.cs (Entry Point, DI Configuration)
├─ Configuration
│ ├─ IndustryConfigService (6 default + custom)
│ ├─ ScreeningOptionsConfig (thresholds, settings)
│ └─ DataSourceConfig (API endpoints, keys)
│ ├─ Data Layer
│ ├─ ScreenerDbContext (EF Core + MySQL)
│ ├─ ICompanyRepository
│ ├─ IFinancialMetricsRepository
│ ├─ IQualifiedCompanyRepository
│ ├─ IRemovedCompanyRepository
│ └─ ICompanyValuationRepository (NEW - Stage 6 valuations)
│ ├─ External Data Services
│ ├─ SecEdgarService (10-K/10-Q fetching)
│ ├─ YahooFinanceService (Stock prices, dividends)
│ ├─ IexCloudService (Alternative market data)
│ ├─ AlphaVantageService (Fallback data source)
│ └─ DataReconciliationService (Aggregate & validate)
│ ├─ Metric Calculation (MERGED WITH SCREENING)
│ ├─ MetricCalculationService (40+ metrics for ALL companies)
│ ├─ TrendAnalysisService (10-year trends)
│ ├─ StabilityCalculationService (CV, volatility)
│ └─ ComprehensiveMetricsAnalyzer (All metrics upfront)
│ ├─ Screening Engine (USES CALCULATED METRICS)
│ ├─ HardFilterEvaluationService (Unified filter assessment)
│ ├─ Stage2ScoringService (Quality assessment)
│ ├─ Stage3RankingService (Valuation & Management)
│ └─ ScreeningOrchestrator (Unified orchestration)
│ ├─ Business Logic
│ ├─ CompanyStatusManager (Pass/fail tracking)
│ ├─ QualifiedCompanyService (Store passing companies)
│ ├─ RemovedCompanyService (Track removals)
│ └─ DataValidationService (Ensure data quality)
│ ├─ Valuation Engine (NEW - STAGE 6)
│ ├─ DCFValuationService
│ │ ├─ HistoricalFCFAnalyzer
│ │ ├─ WACCCalculationService
│ │ ├─ ProjectionService (growth projections)
│ │ └─ TerminalValueCalculator
│ ├─ DDMValuationService
│ │ ├─ DividendHistoryAnalyzer
│ │ └─ DividendProjectionService
│ ├─ MultiplesValuationService
│ │ ├─ PeerCompanySelector
│ │ └─ MultipleCalculationService
│ ├─ BuffettValuationService
│ │ ├─ OwnerEarningsCalculator
│ │ └─ QualityMultipleAssigner
│ ├─ MungerValuationService
│ │ ├─ BusinessQualityAssessor
│ │ └─ MoatStrengthEvaluator
│ └─ ConsensusValuationService
│ ├─ ValuationWeightingEngine
│ ├─ SensitivityAnalysisService
│ └─ MarginOfSafetyCalculator
│ ├─ Results & Reporting (NEW - STAGE 7)
│ ├─ ValuationResultsFormatter
│ ├─ CSVExportService
│ │ ├─ CompanyDataExporter
│ │ ├─ ValuationDataExporter
│ │ └─ MetricsDataExporter
│ ├─ GitHubSummaryGenerator
│ │ ├─ StatisticsCalculator
│ │ └─ SummaryTableBuilder
│ ├─ DatabaseValuationStorage
│ │ ├─ CompanyValuationRepository
│ │ └─ ResultsAuditTrail
│ └─ ConsoleReporter (Console output)
│ └─ Background Jobs
├─ ScreeningJob (Quarterly/Manual, Stages 0-7)
└─ StockPriceUpdateJob (Weekly, no email)
```

---

## Technology Stack

### Core Technologies

| Layer | Technology | Reason |
|-------|-----------|--------|
| **Runtime** | .NET 8.0 LTS | Latest, high performance, long support |
| **Language** | C# 12 | Strong typing, LINQ, financial calculations |
| **Database** | MySQL 8.0 | Familiar to you, easy local setup, free |
| **ORM** | Entity Framework Core 8 | MySQL support, migrations, type-safe |
| **Console Host** | .NET Generic Host | Built-in DI, logging, configuration |
| **HTTP Client** | HttpClientFactory + Polly | Retry policies, circuit breaker, pooling |
| **Logging** | Serilog | Structured logging, file + console |
| **CSV Export** | CsvHelper | Industry-standard CSV generation |
| **Configuration** | appsettings.json | Secrets, environment-specific config |
| **Testing** | NUnit + Moq | Industry standard for .NET |

### Financial Calculation Libraries

| Library | Purpose | Usage |
|---------|---------|-------|
| **LINQ** | Built-in | DCF projections, growth calculations, aggregations |
| **Math.NET Numerics** (Optional) | Advanced math | Matrix operations, statistical analysis |
| **Custom Financial Formulas** | Core logic | WACC, CAGR, margin analysis, valuation |

### External APIs & Data Sources

| Service | Purpose | Primary/Fallback |
|---------|---------|-----------------|
| **SEC EDGAR** | 10-K/10-Q filings | Primary (official) |
| **Yahoo Finance API** | Stock prices, dividends | Primary |
| **IEX Cloud** | Market data | Fallback #1 |
| **Alpha Vantage** | Market data | Fallback #2 |
| **Federal Reserve FRED** | Treasury yields | Primary |

### Local Development Environment

```
Development Machine
├─ Visual Studio 2022 Community (free)
├─ .NET 8.0 SDK
├─ MySQL 8.0 (Docker or native)
├─ MySQL Workbench (GUI client)
├─ Git for version control
└─ Postman (API testing - optional)
```

Quick Start (MySQL via Docker): docker run --name mysql -e MYSQL_ROOT_PASSWORD=root -d -p 3306:3306 mysql:8.0

---

## Core Components

### 1. Industry Configuration Service

**Purpose:** Manage which industries to screen

**Default Industries:**
- Consumer Defensive (beverages, food, household products)
- Consumer Cyclical (retail, apparel, restaurants)
- Technology (software, IT services, semiconductors)
- Communication Services (telecom, media, entertainment)
- Healthcare (pharma, biotech, medical devices)
- Defensive Industry (utilities, real estate, infrastructure)

**Configuration (appsettings.json):**

```
{
  "Screening": {
    "IndustriesToScreen": [
      "Consumer Defensive",
      "Consumer Cyclical",
      "Technology",
      "Communication Services",
      "Healthcare",
      "Defensive Industry"
    ],
    "QuarterlyRunDates": "15th of March, June, September, December",
    "MinimumYearsOfData": 5,
    "PreferredYearsOfData": 10,
    "PassingScoreThreshold": 75,
    "AutomaticRemovalOfFailedCompanies": true,
    "SendEmailOnCompletion": true
  },
  "Email": {
    "SmtpServer": "smtp.gmail.com",
    "SmtpPort": 587,
    "SenderEmail": "your-email@gmail.com",
    "SenderPassword": "your-app-password",
    "RecipientEmail": "your-email@gmail.com",
    "EnableSsl": true
  },
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Uid=root;Pwd=root;Database=stockscreener"
  }
}
```

**Customization:**
- Add/remove industries via `IndustriesToScreen` array
- Adjust score threshold in `PassingScoreThreshold`
- Control data retention with minimum/preferred years

---

### 2. Data Reconciliation Service

**Purpose:** Fetch data from multiple sources and reconcile into single, reliable dataset

**Data Sources Priority:**
1. **SEC EDGAR** (primary source for financials)
   - 10-K annual reports
   - 10-Q quarterly reports
   - Official, authoritative source

2. **Yahoo Finance** (primary for stock prices)
   - Current stock prices
   - Dividend history
   - Market cap

3. **IEX Cloud** (fallback if Yahoo fails)
   - Alternative market data
   - Real-time quotes

4. **Alpha Vantage** (final fallback)
   - When IEX unavailable
   - Historical data

**Reconciliation Logic:**

```
IF SEC EDGAR data available → Use it (authoritative)
IF Yahoo Finance data available → Use it (market data)
IF Yahoo fails → Try IEX Cloud
IF IEX fails → Try Alpha Vantage
IF all fail → Log error, skip company
IF multiple sources differ → Log discrepancy, use primary source
```

**Key Responsibility:**
- Ensures data quality BEFORE metrics calculation
- Prevents corrupted data from affecting analysis
- Creates audit trail of data sources used
- Validates that reconciled data is complete and consistent

---

### 3. Metric Calculation Service (MERGED COMPONENT)

**Purpose:** Calculate ALL 40+ financial metrics for EVERY company UPFRONT

**⚠️ CRITICAL:** This service calculates metrics for ALL companies, with NO filtering or rejection:

**Metric Categories (40+):**

1. **Return on Capital (4 metrics)**
   - ROE: Return on Equity
   - ROA: Return on Assets
   - ROIC: Return on Invested Capital
   - ROCE: Return on Capital Employed

2. **Financial Strength (6 metrics)**
   - Debt-to-Equity Ratio
   - Current Ratio
   - Interest Coverage Ratio
   - Total Liabilities-to-Equity
   - Equity Trend (10-year)
   - Debt Trend (10-year)

3. **Profitability (8 metrics)**
   - Net Profit Margin (with trend)
   - Operating Margin (with trend)
   - Gross Profit Margin
   - Depreciation Margin
   - Interest Expense Margin
   - Margin Stability (CV)
   - EBITDA Margin

4. **Cash Flow (6 metrics)**
   - Operating Cash Flow
   - Free Cash Flow
   - FCF-to-Net-Income Ratio
   - CapEx as % Revenue
   - CapEx as % OCF
   - OCF/FCF Growth trends

5. **Growth (7 metrics)**
   - Revenue Growth (YoY, CAGR)
   - Earnings Per Share (EPS)
   - EPS Growth
   - Retained Earnings
   - Growth Stability metrics

6. **Valuation (8 metrics)**
   - P/E Ratio
   - P/B Ratio
   - P/S Ratio
   - PEG Ratio
   - Earnings Yield
   - Dividend Yield
   - Payout Ratio
   - Market Cap

7. **Management Quality (6 metrics)**
   - Insider Ownership %
   - Goodwill Level
   - Goodwill as % of Assets
   - One-Time Charges Frequency
   - Accounting Quality
   - Management Tenure

**Data Prioritization:**
- **Prefer 10 years** of historical data (full analysis)
- **Accept 5-9 years** as minimum (flags as "partial")
- **Reject < 5 years** (insufficient data)
- **Rank companies** that have 10 years as higher priority

**Calculation Approach:**
- Year-by-year metrics for trend analysis
- 10-year averages for baseline assessment
- Coefficient of Variation (CV) for stability analysis
- CAGR (Compound Annual Growth Rate) for growth assessment

**NO REJECTION AT THIS STAGE:**
- All companies with valid data continue to next stage
- Metrics for all companies are complete and available
- Hard filter evaluation uses this complete dataset

---

### 4. Hard Filter Evaluation Service (MERGED WITH METRICS)

**Purpose:** Evaluate hard filters using the COMPLETE metric dataset just calculated

**CRITICAL CHANGE:** Hard filters are NOT standalone rejection criteria:

**Unified Evaluation Approach:**

```
public class HardFilterEvaluationService
{
    public EvaluationResult EvaluateCompany(CompanyWithAllMetrics company)
    {
        // All 40+ metrics are already calculated at this point
        // Use complete data to assess financial strength
        
        var roe10YAvg = company.Metrics.ReturnOnEquity_10YAvg;
        var netMargin10YAvg = company.Metrics.NetProfitMargin_10YAvg;
        var deRatio = company.Metrics.DebtToEquityRatio_Current;
        var ocfTrend = company.Metrics.OperatingCashFlowTrend_10Y;
        var fcfTrend = company.Metrics.FreeCashFlowTrend_10Y;
        
        // Evaluate hard filters within context of COMPLETE financial picture
        bool passesFinancialStrength = AssessCompleteFinancialStrength(company.Metrics);
        
        if (!passesFinancialStrength)
        {
            // Company fails hard filters, skip to next stage indicator
            return EvaluationResult.FailedHardFilters(company);
        }
        
        // Hard filters passed - continue to quality scoring
        return EvaluationResult.PassedHardFilters(company);
    }
    
    private bool AssessCompleteFinancialStrength(AllMetrics metrics)
    {
        // Assess financial fortress using complete metric dataset:
        // - ROE in context of margins, cash flow, growth
        // - D/E ratio in context of equity trends, coverage ratios
        // - Cash flow in context of profitability and reinvestment
        // - Complete holistic assessment, not checklist
        
        var financialScore = CalculateHolisticFinancialStrength(metrics);
        return financialScore >= RequiredThreshold;
    }
}
```

**Hard Filters Are Evaluated As:**
- ROE > 15% (10Y avg) - assessed with profitability context
- Net Margin > 10% (10Y avg) - assessed with operating metrics
- D/E < 0.50 - assessed with equity trends and coverage
- OCF Positive (all 10Y) - assessed with FCF conversion and growth
- FCF Positive (all 10Y) - assessed with OCF quality and sustainability

**Key Differences from Old Approach:**
- ✅ Hard filters use COMPLETE metric data, not isolated metrics
- ✅ Context from all metrics informs the assessment
- ✅ Weakness in one area assessed against strength in others
- ✅ No premature rejection based on incomplete data

---

### 5. Screening Orchestrator (UNIFIED APPROACH)

**Purpose:** Orchestrates the entire unified flow end-to-end

**Workflow:**
1. Load industry configuration
2. Fetch all companies in selected industries
3. **For each company:**
   - Reconcile financial data from multiple sources
   - **Calculate ALL 40+ metrics** (step 2-3 merged)
   - **Evaluate hard filters using complete metrics** (step 3)
   - Run quality scoring (Stage 2)
   - Run valuation/management ranking (Stage 3)
   - **ONLY store if passing** (composite score >= 75)
4. Compare with previously qualified companies
   - Identify newly qualified
   - Identify maintained
   - Identify removed (no longer passing)
5. Update database (save passers, remove failers)
6. Send email notification with detailed results
7. Export CSV for manual review

**Error Handling:**
- Continue processing if individual company fails
- Log all errors for debugging
- Partial success is acceptable (some companies fail, others pass)
- Robust fallback to secondary data sources

---

### 6. Qualified Company Service

**Purpose:** Manage storage of passing companies

**Responsibilities:**
- Save companies that pass screening (composite score >= 75)
- Update scores for previously qualified companies
- Remove companies that no longer qualify
- Track first qualified date and last qualified date
- Maintain complete financial metrics history

**Storage Strategy:**
- **Only store passing companies** - no waste on failures
- Store all 40+ metrics for each year analyzed
- Create audit trail of removals
- Enable historical trend analysis

---

### 7. Email Notification Service

**Purpose:** Send comprehensive results email

**Email Content:**
- Summary statistics (qualified count, new, maintained, removed)
- Detailed results table with all metrics
- Business quality assessment (MOAT, management)
- Valuation analysis with margin of safety
- List of removed companies with reasons

**Email Format:**
- HTML formatted for readability
- Color-coded scores (green = excellent, red = poor)
- Sortable/filterable tables
- Professional presentation

**Trigger:**
- Automatically after each screening run
- Can be disabled in config if desired

---

### 8. Stock Price Update Job

**Purpose:** Weekly update of stock prices (separate from screening)

**Execution:**
- Runs independently from screening
- Executes weekly (e.g., Monday evening)
- Updates only currently qualified companies
- **No email notifications** sent

**Data Updated:**
- Current stock price
- Market cap
- P/E Ratio
- Dividend yield

**Why Separate:**
- Screening runs quarterly (expensive data fetch)
- Stock prices change constantly
- Valuation metrics need updating
- No email spam - just updates data
- 
---

## Valuation Engine (Stage 6)

### Overview

Stage 6 calculates intrinsic value for each company passing Stage 5 (composite score >= 75) using five independent valuation methods. The engine produces:

1. **Individual fair values** from each method
2. **Consensus fair value** (weighted average)
3. **Margin of safety** analysis
4. **Valuation grade** (A+ to F)
5. **Investment recommendations** (BUY/CONSIDER/PASS/AVOID)

### Architecture Pattern

Each valuation method follows this pattern:

```
public interface IValuationMethod
{
    Task<ValuationResult> CalculateFairValueAsync(CompanyWithMetrics company);
}

public class ValuationResult
{
    public decimal FairValuePerShare { get; set; }
    public decimal ConservativeEstimate { get; set; }
    public decimal OptimisticEstimate { get; set; }
    public SensitivityMatrix SensitivityAnalysis { get; set; }
    public string MethodAssessment { get; set; }
}
```

1. DCF Valuation Service
Purpose: Calculate intrinsic value using Discounted Cash Flow methodology

**Key Components:**
Historical Free Cash Flow Analysis (10-year)
Growth Rate Projections (declining model)
WACC Calculation (discount rate)
Terminal Value Calculation
Sensitivity Analysis (discount rate × terminal growth rate)

**Inputs Required:**
10 years of historical free cash flow
Capital structure (debt, equity values)
Cost of equity (CAPM calculation)
Cost of debt
Revenue/earnings projections

**Outputs:**
Fair value per share
Conservative/base/optimistic estimates
Sensitivity matrix (2D: WACC × Terminal Growth)

**Formula Overview:**

DCF Fair Value = Σ(FCF_t / (1 + WACC)^t) + Terminal Value / (1 + WACC)^n

2. DDM (Dividend Discount Model) Valuation Service
Purpose: Calculate intrinsic value for dividend-paying companies

**Variants:**
Gordon Growth Model (single-stage): For stable dividend payers
Two-Stage Model (multi-stage): For companies with changing growth rates
Buyback-Adjusted: For companies returning cash via buybacks

**Key Components:**
Historical Dividend Per Share Analysis
Dividend Growth Rate Calculation
Payout Ratio Sustainability Check
Terminal Value (Gordon Growth)
Buyback adjustment for non-dividend companies

**Inputs Required:**
10 years of dividend history
Shares outstanding (historical)
Required rate of return
Payout ratio

**Outputs:**
Fair value per share
Sustainability assessment
Growth rate assumptions

**Formula Overview:**

DDM Fair Value = Σ(D_t / (1 + r)^t) + Terminal Value / (1 + r)^n
where D_t = Dividend in year t, r = required return

3. Multiples Valuation Service
Purpose: Calculate intrinsic value using industry comparable multiples

**Approach:**
Select 5 comparable peer companies
Calculate current multiples (P/E, P/B, P/S)
Apply peer average multiples to target company metrics
Average the three multiple-based valuations

**Key Components:**
Peer Company Selection (by industry/size/profitability)
Multiple Calculation (P/E, P/B, P/S)
Forward vs. Trailing multiples
Outlier trimming (remove high/low peers)

**Handling Special Cases:**
Negative Earnings: Use P/S (Price-to-Sales) multiple instead
Low/Zero Book Value: Use P/S or EV/EBITDA
Cyclical Businesses: Use normalized earnings

**Inputs Required:**
5 comparable peer companies
Peer market cap, earnings, book value, revenue
Target company financial metrics

**Outputs:**
P/E-based fair value
P/B-based fair value
P/S-based fair value
Average multiples valuation

**Formulas:**

P/E Based FV = EPS × Peer Average P/E
P/B Based FV = Book Value Per Share × Peer Average P/B
P/S Based FV = Revenue Per Share × Peer Average P/S

###4. Buffett Valuation Service

**Purpose:**
Apply Warren Buffett's investment methodology

**Two Techniques:**
Technique 1: Owner's Earnings Valuation
Concept: Calculate normalized "owner's earnings"
Formula: Net Income + D&A - CapEx (normalized for 10 years)
Valuation: Owner's Earnings × Quality Multiple (15-35x based on business quality)
Quality Multiple Assignment:
35x: Exceptional businesses (Apple, Microsoft)
25x: Very high quality (strong moat, stable growth)
15x: Good quality (reliable but no moat)
Technique 2: Perpetual Bond Yield Comparison
Concept: Earnings yield vs. risk-free rate
Formula: Earnings Yield = Net Income / Market Cap
Assessment: Must exceed 10-year Treasury yield + risk premium (4-5%)
Margin of Safety: 25-50% discount to intrinsic value

**Key Components:**
Normalized earnings calculation
Quality assessment (moat, management, consistency)
Risk premium determination
Margin of safety assignment

**Inputs Required:**
10 years of net income and owner earnings
Capital expenditure history
Business quality assessment
Current Treasury yield

**Outputs:**
Fair value per share (owner's earnings method)
Fair value per share (perpetual bond method)
Buffett assessment (BUY/WAIT/PASS)
5. Munger Valuation Service
Purpose: Apply Charlie Munger's investment methodology

**Business Quality Checklist (1-10 scale):**
Moat Strength: Economic competitive advantage
Durability: 20-year horizon sustainability
Management Quality: Honesty, competence, capital allocation
Earnings Consistency: Predictability and quality
Return on Capital: ROE/ROIC efficiency

**Two Techniques:**

Technique 1: Quality-Based Valuation

**Process:**
Score business quality (1-10 on each factor)
Assign quality multiple based on total score
Apply multiple to normalized earnings
Quality Multiple Assignment:
Score 45+: 35x multiple (exceptional)
Score 35-45: 25x multiple (very high quality)
Score 25-35: 15x multiple (good quality)
Score <25: 10x or less (below target)
Technique 2: Five Stock Test
Concept: Would you buy this as your only stock if limited to 5 holdings?

**Factors:**
Business understandability
Durable competitive advantage
Management quality
Sustainable profitability
Valuation reasonableness
Result: PASS / CONDITIONAL / FAIL

**Key Components:**
Business quality scoring (comprehensive)
Moat strength assessment
Management quality evaluation
Earnings consistency analysis
ROIC assessment

**Inputs Required:**
All 40+ financial metrics
Business model analysis
Management background research
Competitive positioning

**Outputs:**
Fair value per share (quality-based)
Five Stock Test result
Munger assessment (BUY/CONSIDER/PASS/AVOID)
Quality factors summary
6. Consensus Valuation Service
Purpose: Combine all five methods into single recommendation

**Weighting Strategy:**

Consensus Fair Value = 
  (DCF × 40%) +
  (DDM × 15%) +
  (Multiples × 20%) +
  (Buffett × 15%) +
  (Munger × 10%)

**Weighting Rationale:**
DCF (40%): Most theoretically sound for growth projection
Multiples (20%): Market reality check
Buffett (15%): Mechanical simplicity and proven track record
DDM (15%): Cash return validation
Munger (10%): Business quality judgment (less quantifiable)

**Margin of Safety Calculation:**

Margin of Safety % = (Fair Value - Current Price) / Fair Value × 100%

**Classification:**
- > 30%: SLAM DUNK (exceptional opportunity)
- 20-30%: STRONG BUY (good opportunity)
- 10-20%: FAIR VALUE (reasonable price)
- 0-10%: MARGINAL (limited upside)
- < 0%: OVERVALUED (avoid)

**Valuation Grade Assignment:**

Grade	Criteria	Action
A+	Quality 80+, MOS 30%+	Slam Dunk Buy
A	Quality 75-80, MOS 20-30%	Strong Buy
B+	Quality 82+, MOS 10-20%	Fair Value / Accumulate
B	Quality 75-82, MOS 0-10%	Hold / Monitor
C+	Quality 75+, MOS < 0%	Overvalued / Wait
C-F	Quality < 75	Below Threshold

**Investment Recommendation:**

IF Quality Score >= 80 AND MOS >= 20% → BUY (Slam Dunk or Strong Buy)
IF Quality Score >= 75 AND MOS >= 10% → CONSIDER (Fair Value)
IF Quality Score >= 75 AND MOS < 10% → PASS (Wait for better price)
IF Quality Score < 75 OR MOS < 0% → AVOID (Below criteria)

**Sensitivity Analysis:**
2D Matrix: Discount Rate (WACC) × Terminal Growth Rate
Show fair value range (conservative to optimistic)
Highlight key assumptions and risks

**Key Components:**
Valuation method integration
Weighting calculation
Margin of safety determination
Grade assignment
Recommendation logic

**Inputs Required:**
All 5 valuation method results
Current stock price
Quality assessment
Risk factors

**Outputs:**
Consensus fair value per share
Margin of safety percentage
Valuation grade (A+ to F)
Investment recommendation
Key assumptions summary

###Valuation Engine Data Flow

```
Input: Company passing Stage 5 (composite >= 75)
  │
  ├─→ DCF Service    → Fair Value + Sensitivity
  ├─→ DDM Service    → Fair Value (if dividend/buyback)
  ├─→ Multiples Svc  → Fair Value (peer-based)
  ├─→ Buffett Svc    → Fair Value + Assessment
  └─→ Munger Svc     → Fair Value + Quality Score
         │
         └─→ Consensus Service
              ├─ Weight all 5 valuations
              ├─ Calculate margin of safety
              ├─ Assign valuation grade
              ├─ Generate recommendation
              └─→ Output: Complete valuation package
                     │
                     ├─ Fair Value (per method + consensus)
                     ├─ Margin of Safety Analysis
                     ├─ Valuation Grade
                     ├─ Investment Recommendation
                     └─ Sensitivity Analysis
```

###Valuation Quality Checks
##Before storing valuation results:

**✓ Data Validation**
All required inputs present
Financial data within reasonable ranges
10+ years of history where applicable

**✓ Calculation Validation**
All formulas calculate without errors
Results within expected ranges
Growth rates reasonable (not > 15% perpetual)
Discount rates > 0%

**✓ Sanity Checks**
DCF fair value > P/B (shouldn't be below book)
Buffett > Munger (mechanical should be higher)
Fair values within reasonable bands (not 10x apart)
Margin of safety aligns with grade

**✓ Red Flags**
Valuation sensitive to single assumption
Dividend cut imminent (DDM risk)
Cyclical downturn (DCF risk)
Peer group unreliable (Multiples risk)

---

## Data Architecture (MySQL)

### Database Design

**Key Tables:**

1. **Companies** - Master list of all companies
2. **QualifiedCompanies** - Companies currently passing Stage 5 (the watchlist)
3. **FinancialMetrics** - Year-by-year metrics for qualified companies
4. **CompanyValuations** - Intrinsic value results from Stage 6 (NEW)
5. **ScreeningRuns** - Audit trail of each screening execution
6. **RemovedCompanies** - Companies that no longer qualify
7. **StockPriceHistory** - Weekly stock price snapshots

### Table Descriptions

**QualifiedCompanies** (Main table - Stage 5 results)
```
Ticker, CompanyName, Sector, Industry
CompositeScore (0-100) from Stage 5
QualityScore, ValuationScore, ManagementScore
CurrentStockPrice, MarketCap
PERatio, DividendYield
YearsOfDataAnalyzed (5-10)
DataSourcesUsed (SEC EDGAR, Yahoo Finance, etc.)
FirstQualifiedDate, LastQualifiedDate
IsActive (1 = currently qualified, 0 = removed)
```

**FinancialMetrics** (Historical data - Stage 1-3 results)
```
QualifiedCompanyId (foreign key)
FiscalYear (2016-2025 for 10-year analysis)
All 40+ metrics as decimal columns:
├─ Return on Capital (ROE, ROA, ROIC, ROCE)
├─ Financial Strength (D/E, Current Ratio, Coverage, etc.)
├─ Profitability (Margins, Stability)
├─ Cash Flow (OCF, FCF, ratios, trends)
├─ Growth (Revenue, EPS, stability)
├─ Valuation (P/E, P/B, P/S, PEG, Earnings Yield)
└─ Management (Insider ownership, Goodwill, etc.)
Enables trend analysis and historical review
```

**CompanyValuations** (NEW - Stage 6 results)
```
ValuationFields:

CompanyID (foreign key to Companies)
ScreeningRunID (foreign key to ScreeningRuns)
ValuationDate (when valuation was performed)
Market Data:

CurrentStockPrice
MarketCapBillions
SharesOutstandingBillions
DCF Valuation:

DCFProjectionYears
DCFHistoricalFCFCAGR
DCFProjectedGrowthRate
DCFTerminalGrowthRate
DCFDiscountRateWACC
DCFFairValuePerShare
DCFTerminalValue
DCFSensitivityLow, High
DCFAssessment (qualitative)
DDM Valuation (nullable if not applicable):

DDMApplicable (boolean)
DDMHistoricalDividendCAGR
DDMProjectedGrowthRate
DDMTerminalGrowthRate
DDMDiscountRate
DDMFairValuePerShare
DDMAdjustedForBuybacks
DDMPayoutRatioPercent
Multiples Valuation:

PEMultiplesFairValue
PBMultiplesFairValue
PSMultiplesFairValue
MultiplesFairValueAverage
ComparisonCompanies (varchar - list)
PEMultiplePeerAverage
PBMultiplePeerAverage
PSMultiplePeerAverage
Buffett Valuation:

BuffettOwnerEarningsBillions
BuffettQualityMultiple (int: 15-35x)
BuffettFairValuePerShare
BuffettEarningsYield
BuffettRequiredReturn
BuffettAssessment
Munger Valuation:

MungerBusinessQualityScore (1-50)
MungerMoatStrength (1-10)
MungerDurabilityScore (1-10)
MungerManagementQualityScore (1-10)
MungerEarningsConsistencyScore (1-10)
MungerROCScore (1-10)
MungerQualityMultiple (int)
MungerFairValuePerShare
MungerFiveStockTestResult (PASS/CONDITIONAL/FAIL)
Consensus Valuation:

ConsensusFairValuePerShare (weighted average)
ConsensusWeightingMethod (varchar description)
Margin of Safety:

BuffettMarginOfSafetyPercent
BuffettMarginStatus (SLAM_DUNK/STRONG_BUY/FAIR_VALUE/OVERVALUED)
MungerMarginOfSafetyPercent
MungerMarginStatus
ConsensusMarginOfSafetyPercent
ConsensusMarginStatus
Valuation Grade & Recommendations:

ValuationGrade (A+, A, B+, B, C+, C, etc.)
ValuationGradeReasoning (varchar 500)
BuffettRecommendation (BUY/CONSIDER/PASS/AVOID)
MungerRecommendation (BUY/CONSIDER/PASS/AVOID)
ConsensusRecommendation (BUY/CONSIDER/PASS/AVOID)
Supporting Data:

InvestmentThesis (varchar 1000)
Audit Trail:

CreatedAt (timestamp)
UpdatedAt (timestamp)
DataSourcesUsed (varchar 255)
CalculationNotes (varchar 500)
```

**ScreeningRuns (Audit trail)
```
RunID (primary key)
RunDate, RunType (Scheduled/Manual)
Status (Success/Failed/Partial)
Processing Statistics:

CompaniesProcessed
CompaniesMarketCapQualified
CompaniesDataValid
CompaniesHardFilterPassed
CompaniesQualified (composite >= 75)
CompaniesValuated (Stage 6 complete)
Results Summary:

NewlyQualified (count)
MaintainedQualified (count)
Removed (count)
ExecutionTimeSeconds
IndustriesScreened
Performance:

DataFetchTimeSeconds
MetricCalculationTimeSeconds
HardFilterTimeSeconds
ScoringTimeSeconds
ValuationTimeSeconds
ReportingTimeSeconds
```

**RemovedCompanies** (Track removal history)
```
Ticker, CompanyName, Sector, Industry
ReasonRemoved (which criteria it failed)
LastCompositeScore (from Stage 5)
LastValuationGrade (from Stage 6, if applicable)
DateRemoved
ScreeningRunIdRemoved
RemovalNotes (why it failed)
```

**StockPriceHistory** (Weekly updates)
```
QualifiedCompanyId
StockPrice, MarketCap, PERatio, DividendYield
UpdateDate, Source (Yahoo, IEX, etc.)
PreviousPrice (to calculate change)
ChangePercent (calculated)
UpdateSource (API used)
```


**Companies** (Master list)
```
CompanyID (primary key)
Ticker (unique)
CompanyName
Sector, Industry
ExchangeCode
FirstAnalyzedDate
IsActive (1 = currently tracking, 0 = archived)
```

### Database Schema Script (SQL DDL)

Key tables for Stage 6-7:

```
-- CompanyValuations table (Stage 6 results)
CREATE TABLE CompanyValuations (
    ValuationID INT PRIMARY KEY AUTO_INCREMENT,
    CompanyID INT NOT NULL,
    ScreeningRunID INT NOT NULL,
    ValuationDate DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    
    -- Current Market Data
    CurrentStockPrice DECIMAL(10, 2) NOT NULL,
    MarketCapBillions DECIMAL(15, 2),
    SharesOutstandingBillions DECIMAL(10, 2),
    
    -- DCF Valuation
    DCFProjectionYears INT,
    DCFHistoricalFCFCAGR DECIMAL(5, 2),
    DCFProjectedGrowthRate DECIMAL(5, 2),
    DCFTerminalGrowthRate DECIMAL(5, 2),
    DCFDiscountRateWACC DECIMAL(5, 2),
    DCFFairValuePerShare DECIMAL(10, 2),
    DCFTerminalValue DECIMAL(15, 2),
    DCFSensitivityLow DECIMAL(10, 2),
    DCFSensitivityHigh DECIMAL(10, 2),
    
    -- DDM Valuation
    DDMApplicable BOOLEAN,
    DDMHistoricalDividendCAGR DECIMAL(5, 2),
    DDMProjectedGrowthRate DECIMAL(5, 2),
    DDMTerminalGrowthRate DECIMAL(5, 2),
    DDMDiscountRate DECIMAL(5, 2),
    DDMFairValuePerShare DECIMAL(10, 2),
    DDMAdjustedForBuybacks BOOLEAN,
    DDMPayoutRatioPercent DECIMAL(5, 2),
    
    -- Multiples Valuation
    PEMultiplesFairValue DECIMAL(10, 2),
    PBMultiplesFairValue DECIMAL(10, 2),
    PSMultiplesFairValue DECIMAL(10, 2),
    MultiplesFairValueAverage DECIMAL(10, 2),
    ComparisonCompanies VARCHAR(255),
    PEMultiplePeerAverage DECIMAL(5, 2),
    PBMultiplePeerAverage DECIMAL(5, 2),
    PSMultiplePeerAverage DECIMAL(5, 2),
    
    -- Buffett Valuation
    BuffettOwnerEarningsBillions DECIMAL(15, 2),
    BuffettQualityMultiple INT,
    BuffettFairValuePerShare DECIMAL(10, 2),
    BuffettEarningsYield DECIMAL(5, 3),
    BuffettRequiredReturn DECIMAL(5, 2),
    BuffettAssessment VARCHAR(100),
    
    -- Munger Valuation
    MungerBusinessQualityScore INT,
    MungerMoatStrength INT,
    MungerDurabilityScore INT,
    MungerManagementQualityScore INT,
    MungerEarningsConsistencyScore INT,
    MungerROCScore INT,
    MungerQualityMultiple INT,
    MungerFairValuePerShare DECIMAL(10, 2),
    MungerFiveStockTestResult VARCHAR(50),
    
    -- Consensus Valuation
    ConsensusFairValuePerShare DECIMAL(10, 2),
    ConsensusWeightingMethod VARCHAR(255),
    
    -- Margin of Safety
    BuffettMarginOfSafetyPercent DECIMAL(5, 2),
    BuffettMarginStatus VARCHAR(50),
    MungerMarginOfSafetyPercent DECIMAL(5, 2),
    MungerMarginStatus VARCHAR(50),
    ConsensusMarginOfSafetyPercent DECIMAL(5, 2),
    ConsensusMarginStatus VARCHAR(50),
    
    -- Valuation Grade & Recommendations
    ValuationGrade VARCHAR(3),
    ValuationGradeReasoning VARCHAR(500),
    BuffettRecommendation VARCHAR(50),
    MungerRecommendation VARCHAR(50),
    ConsensusRecommendation VARCHAR(50),
    InvestmentThesis VARCHAR(1000),
    
    -- Audit Trail
    CreatedAt DATETIME DEFAULT CURRENT_TIMESTAMP,
    UpdatedAt DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    DataSourcesUsed VARCHAR(255),
    CalculationNotes VARCHAR(500),
    
    FOREIGN KEY (CompanyID) REFERENCES Companies(CompanyID),
    FOREIGN KEY (ScreeningRunID) REFERENCES ScreeningRuns(ScreeningRunID),
    INDEX idx_date (ValuationDate),
    INDEX idx_company (CompanyID),
    INDEX idx_grade (ValuationGrade),
    INDEX idx_recommendation (ConsensusRecommendation)
);

-- ScreeningRuns updated schema
ALTER TABLE ScreeningRuns ADD COLUMN CompaniesValuated INT DEFAULT 0;
ALTER TABLE ScreeningRuns ADD COLUMN ValuationTimeSeconds INT DEFAULT 0;
```

###Why MySQL for This Project
✅ Familiar to you - no learning curve 
✅ Easy local setup - simple, no cloud required 
✅ Free and open source - no licensing cost 
✅ Good EF Core support - Pomelo provider excellent 
✅ Sufficient performance - easily handles thousands of companies with valuations 
✅ Easy to query - can write custom SQL if needed ✅ Built-in indexing - fast lookups on company, date, grade 
✅ Relational integrity - foreign keys maintain data consistency

####Historical Data Retention

**Strategy:**
All CompanyValuations records retained permanently
Database acts as complete audit trail
Enables historical trend analysis (how valuations changed over time)
Supports "Valuation Quality Improvement" tracking
Quarterly snapshots allow comparing methodology refinements

**Backup Schedule:**
Daily incremental backups (retain 30 days)
Weekly full backups (retain 12 weeks)
Monthly snapshots (retain 12 months)
Quarterly archives (retain indefinitely for audit)

---

## Industry Configuration

### 6 Default Industries

```
public static readonly Dictionary<string, string> DefaultIndustries = new()
{
    ["Consumer Defensive"] = "Beverages, Food Products, Household Products, Personal Care",
    ["Consumer Cyclical"] = "Apparel, Retail, Restaurants, Hotels, Discretionary",
    ["Technology"] = "Software, IT Services, Semiconductors, Cloud Computing, Hardware",
    ["Communication Services"] = "Telecom, Media, Entertainment, Social Networks",
    ["Healthcare"] = "Pharmaceuticals, Medical Devices, Biotech, Healthcare Services",
    ["Defensive Industry"] = "Utilities, Real Estate, Infrastructure, Staples"
};
```

### Customization

To add custom industries:
```
{
  "Screening": {
    "IndustriesToScreen": [
      "Technology",
      "Healthcare",
      "Custom Energy Sector"
    ]
  }
}
```

---

## Screening Workflow (Stages 0-7)

### Quarterly Run Flow (Complete 7-Stage Pipeline)

```
Copilot said: ### **SECTION 8: Update Screening Workflow Section
SECTION 8: Update Screening Workflow Section (Lines 685-781)
Replace lines 685-781 with:

Markdown
## Screening Workflow (Stages 0-7)

### Quarterly Run Flow (Complete 7-Stage Pipeline)

📅 Quarterly Execution (e.g., March 15, 2026)
│ ├─ GitHub Actions triggered at 2 PM UTC
├─ OR Manual trigger via workflow_dispatch
│ ├─ Load Configuration
│ └─ Industries: [Consumer Defensive, Technology, Healthcare, ...]
│ ├─ Fetch Companies (by industry from SEC)
│ └─ 500+ companies across selected industries
│ ╔════════════════════════════════════════════════════════════╗
║ STAGE 0: Market Cap Pre-Filter ║
╚════════════════════════════════════════════════════════════╝
│ ├─ Filter: Market Cap > $300M
│ └─ ~400-450 companies pass, ~50-100 rejected (too small)
│ ├─ For Each Qualifying Company (400+ iterations)
│ │ ╔═══════════════════════════════════════════════════════╗
│ ║ STAGE 1: Data Ingestion & Reconciliation ║
│ ╚═══════════════════════════════════════════════════════╝
│ ├─ Fetch from SEC EDGAR (10-K/10-Q)
│ ├─ Fetch from Yahoo Finance (prices, dividends)
│ ├─ Fetch from IEX Cloud / Alpha Vantage (fallback)
│ └─ Reconcile & aggregate data from all sources
│ │ ╔═══════════════════════════════════════════════════════╗
│ ║ STAGE 2-3: Calculate ALL 40+ Metrics (Merged) ║
│ ╚═══════════════════════════════════════════════════════╝
│ ├─ Year-by-year calculation for each metric
│ ├─ 10-year averages and trends
│ ├─ Stability metrics (CV, volatility)
│ ├─ Growth rates (CAGR)
│ └─ ⚠️ NO FILTERING - All metrics calculated
│ │ ╔═══════════════════════════════════════════════════════╗
│ ║ STAGE 4: Hard Filter Evaluation (Unified) ║
│ ╚═══════════════════════════════════════════════════════╝
│ ├─ Evaluate hard filters using COMPLETE 40+ metrics
│ ├─ Assess overall financial strength holistically
│ ├─ Consider context from all metrics
│ └─ IF fails assessment → SKIP; IF passes → Continue
│ │ ╔═══════════════════════════════════════════════════════╗
│ ║ STAGE 5: Quality Assessment & Composite Scoring ║
│ ╚═══════════════════════════════════════════════════════╝
│ ├─ Score: Return on Capital (30 points max)
│ ├─ Score: Profitability (30 points max)
│ ├─ Score: Cash Flow (20 points max)
│ ├─ Score: Business Quality (10 points max)
│ │ → Quality Score (0-100)
│ │ │ ├─ Score: Valuation (P/E, P/B, etc.) - max 35 pts
│ ├─ Score: Management Quality - max 25 pts
│ │ │ ├─ Composite: (Quality×40% + Valuation×35% + Mgmt×25%)
│ │ │ └─ IF Composite >= 75 → Store & Continue; ELSE → Skip
│ │ ╔═══════════════════════════════════════════════════════╗
│ ║ STAGE 6: Intrinsic Value Valuation (Per Company) ║
│ ╚═══════════════════════════════════════════════════════╝
│ ├─ DCF Valuation
│ │ ├─ Analyze historical FCF (10 years)
│ │ ├─ Calculate growth rate (declining model)
│ │ ├─ Compute WACC (discount rate)
│ │ ├─ Calculate terminal value
│ │ └─ Fair Value per share + Sensitivity
│ │ │ ├─ DDM Valuation (if applicable)
│ │ ├─ Analyze dividend history
│ │ ├─ Project future dividends
│ │ ├─ Or adjust for buybacks (non-dividend companies)
│ │ └─ Fair Value per share
│ │ │ ├─ Multiples Valuation
│ │ ├─ Select 5 comparable peers
│ │ ├─ Calculate peer P/E, P/B, P/S multiples
│ │ ├─ Apply to target company metrics
│ │ └─ Fair Value per share (3 approaches averaged)
│ │ │ ├─ Buffett Valuation
│ │ ├─ Calculate normalized owner's earnings
│ │ ├─ Assign quality multiple (15-35x)
│ │ ├─ Earnings yield vs. risk-free rate assessment
│ │ └─ Fair Value per share
│ │ │ ├─ Munger Valuation
│ │ ├─ Score business quality (5 factors, 1-10 scale)
│ │ ├─ Assign quality multiple based on score
│ │ ├─ Five Stock Test (would you own this as your only stock?)
│ │ └─ Fair Value per share + Quality Assessment
│ │ │ └─ Consensus Analysis
│ ├─ Weight all 5 valuations
│ │ (DCF 40%, Multiples 20%, Buffett 15%, DDM 15%, Munger 10%)
│ ├─ Calculate consensus fair value
│ ├─ Determine margin of safety (Fair Value - Current Price)
│ ├─ Assign valuation grade (A+ to F)
│ └─ Generate recommendation (BUY/CONSIDER/PASS/AVOID)
│ └─ End company loop (all passing companies have valuations)

╔════════════════════════════════════════════════════════════╗
║ STAGE 7: Results Reporting & Storage ║
╚════════════════════════════════════════════════════════════╝
│ ├─ Update Database
│ ├─ INSERT/UPDATE CompanyValuations for all Stage 6 results
│ ├─ INSERT/UPDATE QualifiedCompanies with Stage 5 scores
│ ├─ UPDATE ScreeningRuns with execution summary
│ └─ MARK removed companies as inactive
│ ├─ Compare with Previous Results
│ ├─ Query prior screening run
│ ├─ Identify newly qualified companies
│ ├─ Identify maintained (still passing) companies
│ └─ Identify removed companies (now failing)
│ ├─ Generate Outputs
│ │ │ ├─ CSV Export
│ │ ├─ File: screening-results-{DATE}.csv
│ │ ├─ Columns:
│ │ │ ├─ Company identifiers (Ticker, Name, Sector)
│ │ │ ├─ Stage 5 scores (Quality, Valuation, Management, Composite)
│ │ │ ├─ Current stock metrics (Price, P/E, Market Cap)
│ │ │ ├─ Stage 6 valuations (DCF, DDM, Multiples, Buffett, Munger, Consensus)
│ │ │ ├─ Margin of safety (%, Status, Buy prices at 20/30/40/50%)
│ │ │ ├─ Valuation grade & recommendation
│ │ │ ├─ Key metrics summary (ROE, margins, D/E, FCF)
│ │ │ ├─ Moat/durability/management quality scores
│ │ │ └─ Status (NEW/MAINTAINED/REMOVED)
│ │ └─ Ready for import into Excel/Google Sheets
│ │ │ ├─ GitHub Summary (Actions Workflow)
│ │ ├─ Summary statistics table
│ │ │ ├─ Companies processed, qualified, valuated
│ │ │ ├─ Newly qualified, maintained, removed counts
│ │ │ ├─ Average quality/valuation/management scores
│ │ │ └─ Execution time breakdown
│ │ ├─ Top 10 opportunities table
│ │ │ ├─ Ranked by margin of safety
│ │ │ ├─ Shows fair value vs. current price
│ │ │ ├─ Quality scores and grade
│ │ │ └─ Recommendation status
│ │ ├─ Group statistics (Slam Dunks, Strong Buys, Fair Value, Overvalued)
│ │ └─ Comparison with prior run (improvement/decline trends)
│ │ │ └─ Console Output
│ ├─ Processing summary (companies per stage)
│ ├─ Screening results summary
│ ├─ Execution time by stage
│ ├─ Any errors or warnings
│ └─ File paths for outputs
│ └─ ✅ Complete
├─ CSV exported: output/screening-results-{DATE}.csv
├─ GitHub summary: Actions workflow summary
├─ Database updated: {N} companies stored
└─ Next screening: {DATE in 3 months}
```

### Manual Execution

```
User runs: dotnet run

Program.cs → ScreeningOrchestrator.RunScreening()
│ ├─ Interactive prompt (optional):
│ ├─ Select industries (or use default)
│ ├─ Confirm screening parameters
│ └─ Start screening (Stages 0-7)
│ └─ Results:
├─ Console output with summary
├─ CSV export generated
├─ Database updated with results
└─ Reports ready for analysis
```


### Stage Definitions Summary

| Stage | Name | Input | Processing | Output | Storage |
|-------|------|-------|------------|--------|---------|
| **0** | Market Cap Pre-Filter | All companies | Filter by MC > $300M | ~400 qualified | None |
| **1** | Data Ingestion | Qualified companies | Fetch from APIs | Complete financial data | In-memory |
| **2-3** | Metric Calculation | Financial data | Calculate 40+ metrics | Metrics for all | In-memory |
| **4** | Hard Filter Eval | All metrics | Holistic assessment | Pass/fail status | In-memory |
| **5** | Quality & Composite | Passing companies | Score & composite | Quality scores | QualifiedCompanies |
| **6** | Intrinsic Valuation | Passing companies | 5 methods + consensus | Fair values & grades | CompanyValuations |
| **7** | Results Reporting | All results | Format & export | CSV, DB, summary | MySQL + CSV |

### Performance Characteristics

**Timeline by Stage:**

Total Execution: ~50-75 minutes (first run)

Stage 0-1: Data Fetching ~30-40 min (API calls are slowest) 
Stage 2-3: Metric Calculation ~5-10 min 
Stage 4: Hard Filter Evaluation ~2-3 min 
Stage 5: Quality & Composite Scoring ~3-5 min 
Stage 6: Intrinsic Valuation ~5-10 min (5 methods per company) 
Stage 7: Results Processing & Export ~2-3 min

**Optimizations:**
Parallel company processing (per industry)
API response caching (don't re-fetch same data)
Batch database inserts (not one-at-a-time)
Stage 6 can be parallelized (valuation methods independent)
Stock price updates weekly (lightweight, <5 min)


### Error Handling Strategy

**Per-Company Level:**
- Try to process each company
- If any stage fails, log error with details
- Continue to next company (don't crash entire run)
- Report summary of failed companies at end

**API Failures:**
- Try primary source
- Fall back to secondary source
- If all fail, skip company and log
- Continue with remaining companies

**Database Issues:**
- Wrap inserts in transaction
- If insert fails, log and continue
- Save failed records to backup file
- Alert user to manual investigation

**Data Validation Issues:**
- Check data quality BEFORE calculations
- Skip company if data invalid
- Log which validation checks failed
- Don't let bad data corrupt calculations

**Result:** Screening run completes with partial results if individual companies fail (not a full failure)

### Resumption / Restart Capability

**Current Design:** Full re-run each time (no partial resumption)

**Rationale:**
- Quarterly runs (not continuous)
- ~50-75 minutes is acceptable
- Simplifies code and data integrity
- Avoids partial state management complexity

**Future Enhancement Option:**
- Could implement checkpoint system
- Save Stage 1-3 results to cache
- Resume from Stage 4 if needed
- Not required for initial implementation

---

## Unified Metric Calculation & Hard Filter Evaluation

### Why This Merged Approach?

**Prevents False Negatives:**
- A company with strong fundamentals might fail one metric in isolation
- Complete analysis reveals the metric is anomalous or contextually explained
- Example: Temporary debt increase to fund acquisition in strong market position

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

**Enables Better Decision Making:**
- Analyst can see why company passes/fails (complete data visible)
- All 40+ metrics available for manual review
- Scoring transparent and defensible
- Audit trail shows all data considered

### Implementation Details

**Metric Calculation Phase (Before Any Evaluation):**
```
var metricsResult = await metricCalculationService.CalculateAllMetricsAsync(company);

// Returns: Comprehensive metrics object with all 40+ metrics calculated
// - No filtering occurs here
// - All companies continue to hard filter evaluation
// - Complete financial picture now available
```

**Hard Filter Evaluation Phase (Using Complete Metrics):**
```csharp
var evaluationResult = await hardFilterEvaluationService.EvaluateCompanyAsync(
    company, 
    metricsResult  // Pass the complete metrics just calculated
);

// Hard filters are evaluated in context of:
// - Profitability trends alongside margins
// - Debt ratios alongside cash flow strength
// - Growth rates alongside capital efficiency
// - All metrics together paint complete picture
```

**Quality Scoring Phase (For Companies Passing Hard Filter Evaluation):**
```csharp
if (evaluationResult.PassedHardFilters)
{
    var qualityScore = await stage2ScoringService.ScoreQualityAsync(
        company, 
        metricsResult
    );
    
    var valuationScore = await stage3RankingService.ScoreValuationAsync(
        company, 
        metricsResult
    );
    
    var compositeScore = CalculateComposite(qualityScore, valuationScore);
    
    // Only companies with composite >= 75 are stored
}
```

---

## Data Reconciliation Layer

### Multi-Source Data Strategy

**Why Multiple Sources?**
- SEC EDGAR might be delayed or corrupted
- Single source creates single point of failure
- Different APIs provide different data
- Can cross-validate for accuracy

**Reconciliation Strategy:**
```
Financial Statements (Income, Balance Sheet, Cash Flow):
  1st choice: SEC EDGAR (official, required)

Stock Prices & Dividends:
  1st choice: Yahoo Finance API
  2nd choice: IEX Cloud (if Yahoo fails)
  3rd choice: Alpha Vantage (final fallback)

Treasury Yields:
  1st choice: Federal Reserve FRED API
  2nd choice: Hardcoded estimate if API unavailable

If multiple sources available:
  → Use primary source (authoritative)
  → Log discrepancies for manual review
  → Never mix sources for same metric
```

**Validation Before Use:**
- Check data completeness (all required fields present)
- Validate data ranges (e.g., ROE should be -100% to +200%)
- Ensure balance sheet balance (Assets = Liabilities + Equity)
- Verify historical consistency (no sudden 1000% swings)

**Error Recovery:**
```
Data Fetch Error:
  ├─ Log error with details
  ├─ Try next source in priority
  ├─ After all sources fail → Skip company
  └─ Continue to next company (don't crash)
```

---

## Results & Reporting (Stage 7)

### Overview

Stage 7 takes all valuation results from Stage 6 and produces comprehensive reports in multiple formats for analysis and investment decision-making. This stage handles:

1. **Database Storage** - Permanent record of all valuations
2. **CSV Export** - Spreadsheet-friendly data for analysis
3. **GitHub Actions Summary** - Generated in workflow (no code needed)
4. **Console Output** - Real-time feedback during execution

### Stage 7 Architecture

**Components:**

```
Stage 6 Valuations 
│ ├─→ ValuationResultsFormatter
│ └─ Prepares data structures for export
│ ├─→ CompanyValuationRepository (EF Core)
│ ├─ INSERT new valuations
│ ├─ UPDATE existing valuations
│ └─ Query historical data
│ ├─→ CSVExportService
│ ├─ Builds DataTable from valuations
│ ├─ Uses CsvHelper library
│ └─ Writes to file: output/screening-results-{DATE}.csv
│ ├─→ ConsoleReporter
│ ├─ Stage completion messages
│ ├─ Summary statistics
│ └─ Execution timing breakdown
│ └─→ GitHub Actions Summary (Markdown)
├─ Automated via workflow (no C# code)
├─ Uses workflow commands
└─ Displays in Actions tab
```

---

### 1. Database Storage

**Purpose:** Permanent storage of all valuation results from Stage 6

**Table:** `CompanyValuations` (created in Data Architecture section)

**Data Stored per Company:**
- All 5 valuation method results (DCF, DDM, Multiples, Buffett, Munger)
- Consensus fair value (weighted average)
- Margin of safety analysis
- Valuation grade (A+ to F)
- Investment recommendations from each method
- Sensitivity analysis details
- Complete audit trail

**Key Implementation:**

```
public interface ICompanyValuationRepository
{
    Task<int> InsertOrUpdateAsync(CompanyValuation valuation);
    Task<CompanyValuation> GetLatestByCompanyAsync(int companyId);
    Task<List<CompanyValuation>> GetByScreeningRunAsync(int screeningRunId);
    Task<List<CompanyValuation>> GetHistoryAsync(int companyId, int months = 12);
}

public class CompanyValuationRepository : ICompanyValuationRepository
{
    private readonly ScreenerDbContext _context;
    
    public async Task<int> InsertOrUpdateAsync(CompanyValuation valuation)
    {
        var existing = await _context.CompanyValuations
            .FirstOrDefaultAsync(v => 
                v.CompanyId == valuation.CompanyId && 
                v.ScreeningRunId == valuation.ScreeningRunId);
        
        if (existing != null)
        {
            // Update existing record
            _context.Entry(existing).CurrentValues.SetValues(valuation);
        }
        else
        {
            // Insert new record
            await _context.CompanyValuations.AddAsync(valuation);
        }
        
        return await _context.SaveChangesAsync();
    }
}
```

**Queries for Historical Analysis:**

```
-- Find all companies currently qualified with their latest valuations
SELECT c.Ticker, c.CompanyName, cv.ConsensusFairValuePerShare,
       cv.CurrentStockPrice, cv.ConsensusMarginOfSafetyPercent,
       cv.ValuationGrade, cv.ConsensusRecommendation
FROM CompanyValuations cv
JOIN Companies c ON cv.CompanyId = c.CompanyId
WHERE cv.ValuationDate = (SELECT MAX(ValuationDate) FROM CompanyValuations)
ORDER BY cv.ConsensusMarginOfSafetyPercent DESC;

-- Track valuation changes over time for one company
SELECT ValuationDate, ConsensusFairValuePerShare, CurrentStockPrice,
       ConsensusMarginOfSafetyPercent, ValuationGrade
FROM CompanyValuations
WHERE CompanyId = (SELECT CompanyId FROM Companies WHERE Ticker = 'AAPL')
ORDER BY ValuationDate DESC;

-- Identify methodology divergence (Buffett vs Munger disagreement)
SELECT Ticker, BuffettRecommendation, MungerRecommendation,
       BuffettFairValuePerShare, MungerFairValuePerShare,
       ABS(BuffettFairValuePerShare - MungerFairValuePerShare) as Divergence
FROM CompanyValuations cv
JOIN Companies c ON cv.CompanyId = c.CompanyId
WHERE cv.ValuationDate = (SELECT MAX(ValuationDate) FROM CompanyValuations)
  AND BuffettRecommendation != MungerRecommendation
ORDER BY Divergence DESC;
```

###2. CSV Export Service

**Purpose:**
Generate spreadsheet-friendly export for external analysis

**File:**
output/screening-results-{YYYY-MM-DD}.csv

**Column Structure:**

```
Company Identifiers:
├─ Ticker
├─ CompanyName
├─ Sector
├─ Industry

Stage 5 Scores (Financial Analysis):
├─ QualityScore (0-100)
├─ ValuationScore (0-100)
├─ ManagementScore (0-100)
├─ CompositeScore (0-100)
├─ Status (NEW / MAINTAINED / REMOVED)

Current Market Data:
├─ CurrentStockPrice
├─ MarketCapBillions
├─ PERatio
├─ DividendYield

Stage 6 Fair Values (5 Methods):
├─ DCFFairValue
├─ DDMFairValue (N/A if not applicable)
├─ MultiplesFairValue
├─ BuffettFairValue
├─ MungerFairValue
├─ ConsensusFairValue

Margin of Safety Analysis:
├─ ConsensusMarginOfSafetyPercent
├─ ConsensusMarginStatus (SLAM_DUNK / STRONG_BUY / FAIR_VALUE / OVERVALUED)
├─ BuyPrice_20PercentMOS
├─ BuyPrice_30PercentMOS
├─ BuyPrice_40PercentMOS
├─ BuyPrice_50PercentMOS

Valuation Recommendations:
├─ ValuationGrade (A+ to F)
├─ BuffettRecommendation (BUY / CONSIDER / PASS / AVOID)
├─ MungerRecommendation (BUY / CONSIDER / PASS / AVOID)
├─ ConsensusRecommendation (BUY / CONSIDER / PASS / AVOID)

Key Financial Metrics (Supporting):
├─ ROE_10YAvg (%)
├─ NetProfitMargin_10YAvg (%)
├─ OperatingMargin_10YAvg (%)
├─ DebtToEquity_Current
├─ FreeCashFlowTrend (positive/stable/declining)
├─ MarginStability_CV (coefficient of variation)

Business Quality Scores (Munger Framework):
├─ MoatStrength (1-10)
├─ BusinessDurability (1-10)
├─ ManagementQualityScore (1-10)
├─ EarningsConsistencyScore (1-10)

Tracking Dates:
├─ FirstQualifiedDate
├─ LastQualifiedDate
├─ ScreeningDate
└─ DataSourcesUsed
```

**CSV Generation Implementation:**

```
public class CSVExportService
{
    private readonly ICompanyValuationRepository _valuationRepository;
    private readonly ILogger<CSVExportService> _logger;
    
    public async Task ExportScreeningResultsAsync(
        int screeningRunId, 
        string outputDirectory)
    {
        var valuations = await _valuationRepository
            .GetByScreeningRunAsync(screeningRunId);
        
        var fileName = $"screening-results-{DateTime.UtcNow:yyyy-MM-dd}.csv";
        var filePath = Path.Combine(outputDirectory, fileName);
        
        using (var writer = new StreamWriter(filePath, Encoding.UTF8))
        using (var csv = new CsvWriter(writer, CultureInfo.InvariantCulture))
        {
            // Register class map for proper column ordering
            csv.Context.RegisterClassMap<CompanyValuationMap>();
            
            // Write header
            await csv.WriteHeaderAsync<CompanyValuationCSVRow>();
            await csv.NextRecordAsync();
            
            // Write data rows
            foreach (var valuation in valuations)
            {
                var row = MapValuationToCSV(valuation);
                await csv.WriteRecordAsync(row);
                await csv.NextRecordAsync();
            }
        }
        
        _logger.LogInformation(
            $"CSV export complete: {filePath} ({valuations.Count} companies)");
    }
    
    private CompanyValuationCSVRow MapValuationToCSV(CompanyValuation v)
    {
        return new CompanyValuationCSVRow
        {
            Ticker = v.Company.Ticker,
            CompanyName = v.Company.CompanyName,
            Sector = v.Company.Sector,
            Industry = v.Company.Industry,
            QualityScore = v.QualityScore,
            ValuationScore = v.ValuationScore,
            ManagementScore = v.ManagementScore,
            CompositeScore = v.CompositeScore,
            Status = DetermineStatus(v),
            CurrentStockPrice = v.CurrentStockPrice,
            MarketCapBillions = v.MarketCapBillions,
            PERatio = v.PERatio,
            DividendYield = v.DividendYield ?? 0,
            
            // Stage 6 Valuations
            DCFFairValue = v.DCFFairValuePerShare,
            DDMFairValue = v.DDMApplicable ? v.DDMFairValuePerShare : null,
            MultiplesFairValue = v.MultiplesFairValueAverage,
            BuffettFairValue = v.BuffettFairValuePerShare,
            MungerFairValue = v.MungerFairValuePerShare,
            ConsensusFairValue = v.ConsensusFairValuePerShare,
            
            // Margin of Safety
            ConsensusMarginOfSafetyPercent = v.ConsensusMarginOfSafetyPercent,
            ConsensusMarginStatus = v.ConsensusMarginStatus,
            BuyPrice_20MOS = CalculateBuyPrice(v.ConsensusFairValuePerShare, 0.20m),
            BuyPrice_30MOS = CalculateBuyPrice(v.ConsensusFairValuePerShare, 0.30m),
            BuyPrice_40MOS = CalculateBuyPrice(v.ConsensusFairValuePerShare, 0.40m),
            BuyPrice_50MOS = CalculateBuyPrice(v.ConsensusFairValuePerShare, 0.50m),
            
            // Recommendations
            ValuationGrade = v.ValuationGrade,
            BuffettRecommendation = v.BuffettRecommendation,
            MungerRecommendation = v.MungerRecommendation,
            ConsensusRecommendation = v.ConsensusRecommendation,
            
            // Financial Metrics
            ROE_10YAvg = v.FinancialMetrics?.FirstOrDefault()?.ReturnOnEquity_10YAvg,
            NetMargin_10YAvg = v.FinancialMetrics?.FirstOrDefault()?.NetProfitMargin_10YAvg,
            OperatingMargin_10YAvg = v.FinancialMetrics?.FirstOrDefault()?.OperatingMargin_10YAvg,
            DebtToEquity = v.FinancialMetrics?.FirstOrDefault()?.DebtToEquityRatio_Current,
            
            // Business Quality Scores
            MoatStrength = v.MungerMoatStrength,
            BusinessDurability = v.MungerDurabilityScore,
            ManagementQuality = v.MungerManagementQualityScore,
            EarningsConsistency = v.MungerEarningsConsistencyScore,
            
            // Dates
            ScreeningDate = v.ValuationDate,
            FirstQualifiedDate = v.FirstQualifiedDate,
            LastQualifiedDate = v.LastQualifiedDate
        };
    }
    
    private decimal CalculateBuyPrice(decimal fairValue, decimal mosPercent)
    {
        return fairValue * (1 - mosPercent);
    }
}
```

**Example CSV Output:**

```
Ticker,CompanyName,Sector,Industry,QualityScore,ValuationScore,ManagementScore,CompositeScore,Status,CurrentStockPrice,MarketCapBillions,PERatio,DividendYield,DCFFairValue,DDMFairValue,MultiplesFairValue,BuffettFairValue,MungerFairValue,ConsensusFairValue,ConsensusMarginOfSafetyPercent,ConsensusMarginStatus,BuyPrice_20MOS,BuyPrice_30MOS,BuyPrice_40MOS,BuyPrice_50MOS,ValuationGrade,BuffettRecommendation,MungerRecommendation,ConsensusRecommendation,ROE_10YAvg,NetMargin_10YAvg,OperatingMargin_10YAvg,DebtToEquity,MoatStrength,BusinessDurability,ManagementQuality,EarningsConsistency,ScreeningDate,FirstQualifiedDate,LastQualifiedDate

AAPL,Apple Inc.,Technology,Consumer Electronics,85,78,82,81.5,MAINTAINED,180.50,2850.0,25.8,0.45,285.50,0.00,187.20,265.00,280.00,252.00,39.7,SLAM_DUNK,201.60,176.40,151.20,126.00,A+,BUY,BUY,BUY,35.2,28.9,32.1,0.04,10,9,9,9,2026-03-15,2025-09-20,2026-03-15

MSFT,Microsoft Corporation,Technology,Software,92,82,88,87.5,MAINTAINED,420.50,3140.0,28.3,2.1,523.15,0.00,342.29,480.00,310.00,345.60,-21.7,OVERVALUED,C,PASS,PASS,CONSIDER,28.7,32.1,28.9,0.04,9,9,9,9,2026-03-15,2025-03-15,2026-03-15

JNJ,Johnson & Johnson,Healthcare,Pharmaceuticals,82,80,78,80.3,NEW,145.75,399.0,18.5,2.65,185.30,180.20,165.40,190.00,175.00,180.00,23.6,STRONG_BUY,144.00,126.00,108.00,90.00,A,BUY,CONSIDER,BUY,15.8,22.1,24.3,0.32,8,8,8,8,2026-03-15,2026-03-15,2026-03-15
```

**CSV Features:**
UTF-8 encoding for international characters
Comma-separated for Excel compatibility
Null values handled (DDM blank if not applicable)
All decimals to 2 places (currency standards)
Ready for pivot tables and conditional formatting
Can be imported to Python/R for analysis


###3. Console Output Report

**Purpose:**
Real-time feedback during Stage 7 execution

**Output Example:**

```
═══════════════════════════════════════════════════════════════════════════════
STAGE 7: RESULTS REPORTING & STORAGE
═══════════════════════════════════════════════════════════════════════════════

📊 Stage 7 Processing
  Companies with valuations: 45
  
  ✅ Database Storage
     ├─ Inserted 5 new CompanyValuation records
     ├─ Updated 38 existing records
     └─ Marked 2 removed companies as inactive

  ✅ Comparison with Prior Run (2025-12-15)
     ├─ Newly Qualified: 5 companies
     │  └─ NVDA, AVGO, ADBE, COST, NOW
     ├─ Maintained: 38 companies
     │  ├─ Quality improved: 12 (+avg 3.2 points)
     │  ├─ Quality declined: 8 (-avg 2.1 points)
     │  └─ Quality stable: 18 (±2 points)
     └─ Removed: 2 companies
        └─ IBM (score 72), GE (score 68)

  ✅ CSV Export Generated
     ├─ File: output/screening-results-2026-03-15.csv
     ├─ Rows: 45 companies
     ├─ Columns: 38 data fields
     └─ Size: 24 KB

═══════════════════════════════════════════════════════════════════════════════
SCREENING EXECUTION SUMMARY
═══════════════════════════════════════════════════════════════════════════════

Universe Statistics:
  ✅ Starting Universe: 520 companies
  ✅ Market Cap Qualified (>$300M): 480 (92%)
  ✅ Data Valid (5+ years): 450 (86%)
  ✅ Hard Filters Passed: 75 (14%)
  ✅ Composite Score >= 75: 45 (9%)
  ✅ Fully Valuated: 45 (100%)

Investment Opportunities:
  🎯 Slam Dunk Buys (A+, MOS 30%+): 3 companies
     ├─ AAPL: +39.7% margin
     ├─ JNJ: +23.6% margin
     └─ COST: +31.2% margin

  🎯 Strong Buys (A, MOS 20-30%): 8 companies
     └─ [Average 25.4% margin of safety]

  🎯 Fair Value (B+, MOS 10-20%): 12 companies
     └─ [Average 14.8% margin of safety]

  ⚠️  Overvalued (C, MOS <0%): 22 companies
     └─ [Watch for price improvements]

Execution Timeline:
  Stage 0 (Market Cap): 15s
  Stage 1-3 (Data & Metrics): 38m 45s
  Stage 4 (Hard Filters): 2m 03s
  Stage 5 (Quality Scoring): 4m 15s
  Stage 6 (Intrinsic Valuation): 8m 42s
  Stage 7 (Results Processing): 2m 18s
  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  TOTAL: 63m 15s ✅

═══════════════════════════════════════════════════════════════════════════════
✅ SCREENING COMPLETE - All 7 Stages Finished Successfully
════════════════════════════════════════════════════════════��══════════════════

📁 Output Files:
  ✅ CSV Export: output/screening-results-2026-03-15.csv
  ✅ Database: 45 valuation records stored in stockscreener
  ✅ Console Log: logs/screening-2026-03-15.log

📧 Email Notification:
  ✅ GitHub Actions will send comprehensive email with results
  
⏭️  Next Screening:
  📅 Scheduled for: 2026-06-15 (90 days)
  🔄 Manual trigger available anytime via GitHub Actions
```

###4. GitHub Actions Summary (Workflow-Generated)

**Note:**
This is generated by GitHub Actions workflow, not by C# code. The workflow uses:

```
- name: Generate Screening Summary
  run: |
    echo "## Stock Market Screener Results - $(date +%Y-%m-%d)" >> $GITHUB_STEP_SUMMARY
    echo "" >> $GITHUB_STEP_SUMMARY
    echo "| Metric | Value |" >> $GITHUB_STEP_SUMMARY
    echo "|--------|-------|" >> $GITHUB_STEP_SUMMARY
    echo "| Companies Analyzed | 520 |" >> $GITHUB_STEP_SUMMARY
    echo "| Market Cap Qualified | 480 |" >> $GITHUB_STEP_SUMMARY
    echo "| Composite Score >= 75 | 45 |" >> $GITHUB_STEP_SUMMARY
    ...
```

**Displays in:**
GitHub Actions tab → Workflow Run → Summary

###5. Comparison & Trend Analysis

**Methodology:**

```
public class ScreeningComparisonService
{
    public async Task<ScreeningComparison> CompareWithPriorRunAsync(
        int currentScreeningRunId,
        int priorScreeningRunId)
    {
        var currentValuations = await _repo.GetByScreeningRunAsync(currentScreeningRunId);
        var priorValuations = await _repo.GetByScreeningRunAsync(priorScreeningRunId);
        
        var comparison = new ScreeningComparison
        {
            NewlyQualified = currentValuations
                .Where(c => !priorValuations.Any(p => p.CompanyId == c.CompanyId))
                .ToList(),
            
            Maintained = currentValuations
                .Where(c => priorValuations.Any(p => p.CompanyId == c.CompanyId))
                .ToList(),
            
            Removed = priorValuations
                .Where(p => !currentValuations.Any(c => c.CompanyId == p.CompanyId))
                .ToList(),
            
            QualityImprovements = CalculateQualityChanges(currentValuations, priorValuations),
            ValuationChanges = CalculateValuationChanges(currentValuations, priorValuations)
        };
        
        return comparison;
    }
}
```

###6. Data Quality Assurance in Stage 7

**Validation Before Storage:**

```
public class ValuationQualityValidator
{
    public ValidationResult ValidateValuation(CompanyValuation valuation)
    {
        var errors = new List<string>();
        
        // Check valuations are within reasonable range
        var minFV = Math.Min(
            valuation.DCFFairValuePerShare,
            Math.Min(
                valuation.MultiplesFairValueAverage,
                valuation.BuffettFairValuePerShare));
        
        var maxFV = Math.Max(
            valuation.DCFFairValuePerShare,
            Math.Max(
                valuation.MultiplesFairValueAverage,
                valuation.MungerFairValuePerShare));
        
        if (maxFV / minFV > 10) // More than 10x difference
        {
            errors.Add($"Valuation range too wide: ${minFV} - ${maxFV}");
        }
        
        // Check growth rates are reasonable
        if (valuation.DCFTerminalGrowthRate > 15)
        {
            errors.Add($"Terminal growth rate unrealistic: {valuation.DCFTerminalGrowthRate}%");
        }
        
        // Check discount rates
        if (valuation.DCFDiscountRateWACC <= 0 || valuation.DCFDiscountRateWACC > 50)
        {
            errors.Add($"WACC out of bounds: {valuation.DCFDiscountRateWACC}%");
        }
        
        // Check margin of safety makes sense
        if (valuation.ConsensusMarginOfSafetyPercent < -100)
        {
            errors.Add("Margin of safety calculation error");
        }
        
        return new ValidationResult 
        { 
            IsValid = errors.Count == 0, 
            Errors = errors 
        };
    }
}
```

###7. Investment Decision Framework

**Using CSV + Database Results:**

**For Immediate Action (This Week):**
Filter CSV: ValuationGrade = 'A+' AND ConsensusRecommendation = 'BUY'
Sort by: ConsensusMarginOfSafetyPercent DESC
Action: Build initial positions in top 3

**For Medium-Term Accumulation (This Month):**
Filter CSV: ValuationGrade IN ('A', 'B+') AND ConsensusRecommendation IN ('BUY', 'CONSIDER')
Sort by: QualityScore DESC
Action: Dollar-cost average into these

**For Watchlist (Set Price Alerts):**
Filter CSV: ValuationGrade = 'C' AND QualityScore >= 75
Calculate trigger: ConsensusFairValue * 0.85 (15% discount)
Action: Buy limit orders at trigger price

**Skip (Do Not Buy):**
Filter CSV: CompositeScore < 75
Action: Don't buy (revisit next quarter)

---

## GitHub Actions Automation

### Overview

The Stock Market Screener runs completely automated via GitHub Actions on a quarterly schedule. Two workflows handle different responsibilities:

1. **Quarterly Screening Workflow** - Runs full 7-stage pipeline (Stages 0-7)
2. **Weekly Stock Price Update Workflow** - Updates valuation metrics (lightweight)

---

### Quarterly Screening Workflow

**File:** `.github/workflows/quarterly-screening.yml`

**Schedule:**
- Automatically triggers: 15th of March, June, September, December at 2:00 PM UTC
- Manual trigger available anytime via `workflow_dispatch`

**Trigger Configuration:**

```
name: Quarterly Stock Market Screening

on:
  schedule:
    # Run quarterly on the 15th at 2 PM UTC
    - cron: '0 14 15 3,6,9,12 *'
  
  workflow_dispatch:  # Allow manual trigger anytime
    inputs:
      industries:
        description: 'Industries to screen (comma-separated, or leave blank for default)'
        required: false
        default: 'Consumer Defensive,Technology,Healthcare,Communication Services,Consumer Cyclical,Defensive Industry'

env:
  DOTNET_VERSION: '8.0.x'
  MYSQL_VERSION: '8.0'
  CONFIGURATION: Release
```

**Workflow Steps:**

```
jobs:
  quarterly-screening:
    runs-on: ubuntu-latest
    timeout-minutes: 120  # 2-hour timeout for full pipeline
    
    services:
      mysql:
        image: mysql:8.0
        env:
          MYSQL_ROOT_PASSWORD: root
          MYSQL_DATABASE: stockscreener
        options: >-
          --health-cmd="mysqladmin ping"
          --health-interval=10s
          --health-timeout=5s
          --health-retries=3
        ports:
          - 3306:3306
    
    steps:
      # Step 1: Checkout code
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
      
      # Step 2: Setup .NET 8.0
      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: ${{ env.DOTNET_VERSION }}
      
      # Step 3: Display environment info
      - name: Display environment
        run: |
          echo "=== GitHub Actions Environment ==="
          dotnet --version
          echo "Runner OS: ${{ runner.os }}"
          echo "Workflow Trigger: ${{ github.event_name }}"
          echo "Scheduled: ${{ github.event.schedule }}"
      
      # Step 4: Restore NuGet packages
      - name: Restore NuGet packages
        run: dotnet restore src/StockMarketScreener.Console/StockMarketScreener.Console.csproj
      
      # Step 5: Build application
      - name: Build application
        run: |
          dotnet build src/StockMarketScreener.Console/StockMarketScreener.Console.csproj \
            --configuration ${{ env.CONFIGURATION }} \
            --no-restore
      
      # Step 6: Wait for MySQL to be ready
      - name: Wait for MySQL
        run: |
          until mysql -h 127.0.0.1 -u root -proot -e "SELECT 1"; do
            echo "Waiting for MySQL..."
            sleep 5
          done
      
      # Step 7: Run EF Core migrations
      - name: Run database migrations
        env:
          ConnectionStrings__DefaultConnection: "Server=127.0.0.1;Uid=root;Pwd=root;Database=stockscreener"
        run: |
          dotnet ef database update \
            --project src/StockMarketScreener.Console/StockMarketScreener.Console.csproj \
            --verbose
      
      # Step 8: Execute screening application (STAGES 0-7)
      - name: Execute screening application
        env:
          ConnectionStrings__DefaultConnection: "Server=127.0.0.1;Uid=root;Pwd=root;Database=stockscreener"
          Screening__IndustriesToScreen: ${{ github.event.inputs.industries || 'Consumer Defensive,Technology,Healthcare,Communication Services,Consumer Cyclical,Defensive Industry' }}
          Screening__PassingScoreThreshold: '75'
          Screening__AutomaticRemovalOfFailedCompanies: 'true'
        run: |
          echo "=== Starting Stock Market Screening Pipeline ==="
          echo "Industries: ${{ env.Screening__IndustriesToScreen }}"
          echo "Passing Threshold: ${{ env.Screening__PassingScoreThreshold }}"
          echo ""
          
          dotnet run \
            --project src/StockMarketScreener.Console/StockMarketScreener.Console.csproj \
            --configuration ${{ env.CONFIGURATION }} \
            --no-build
          
          SCREENING_EXIT_CODE=$?
          
          if [ $SCREENING_EXIT_CODE -eq 0 ]; then
            echo ""
            echo "✅ Screening completed successfully"
          else
            echo ""
            echo "❌ Screening failed with exit code: $SCREENING_EXIT_CODE"
            exit $SCREENING_EXIT_CODE
          fi
      
      # Step 9: Generate screening summary for GitHub Actions
      - name: Generate screening summary
        if: always()
        run: |
          echo "## 📊 Stock Market Screening Results" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "**Screening Date:** $(date '+%Y-%m-%d %H:%M UTC')" >> $GITHUB_STEP_SUMMARY
          echo "**Status:** ✅ Complete" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "### Results" >> $GITHUB_STEP_SUMMARY
          echo "- **CSV Export:** \`output/screening-results-$(date '+%Y-%m-%d').csv\`" >> $GITHUB_STEP_SUMMARY
          echo "- **Database:** Updated with valuation results" >> $GITHUB_STEP_SUMMARY
          echo "- **Next Run:** $(date -d '+90 days' '+%Y-%m-%d')" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "📁 Check **Artifacts** for CSV export file" >> $GITHUB_STEP_SUMMARY
      
      # Step 10: Upload results as GitHub artifact
      - name: Upload screening results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: screening-results-${{ github.run_id }}
          path: output/screening-results-*.csv
          retention-days: 90
      
      # Step 11: Upload logs as artifact
      - name: Upload logs
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: screening-logs-${{ github.run_id }}
          path: logs/
          retention-days: 30
      
      # Step 12: Optional - Commit results to repository
      - name: Commit results to repository (optional)
        if: success()
        run: |
          git config --local user.email "action@github.com"
          git config --local user.name "GitHub Action"
          
          RESULT_FILE="output/screening-results-$(date '+%Y-%m-%d').csv"
          
          if [ -f "$RESULT_FILE" ]; then
            git add "$RESULT_FILE"
            git commit -m "Screening results - $(date '+%Y-%m-%d')"
            git push
          fi
      
      # Step 13: Send notification email (via GitHub Secrets)
      - name: Send screening completion email
        if: always()
        uses: dawidd6/action-send-mail@v3
        with:
          server_address: ${{ secrets.SMTP_SERVER }}
          server_port: ${{ secrets.SMTP_PORT }}
          username: ${{ secrets.SENDER_EMAIL }}
          password: ${{ secrets.SENDER_PASSWORD }}
          subject: "Stock Market Screening Complete - ${{ job.status }}"
          to: ${{ secrets.RECIPIENT_EMAIL }}
          from: ${{ secrets.SENDER_EMAIL }}
          body: |
            Stock Market Screening Run Complete
            
            Status: ${{ job.status }}
            Date: $(date '+%Y-%m-%d %H:%M UTC')
            Workflow Run: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}
            
            Check the GitHub Actions run for detailed results and download the CSV export.
            
            Next quarterly screening: 3 months from now
```

**Environment Variables (Required in GitHub Secrets):**

```
SMTP_SERVER = smtp.gmail.com
SMTP_PORT = 587
SENDER_EMAIL = your-email@gmail.com
SENDER_PASSWORD = your-gmail-app-password (NOT regular password)
RECIPIENT_EMAIL = your-email@gmail.com
```

**To set GitHub Secrets:**
Go to: Repository → Settings → Secrets and variables → Actions
Click "New repository secret"
Add each secret from above

**Workflow Timing:**

```
Estimated Execution: ~60-75 minutes

- Setup & checkout: 30s
- .NET setup: 45s
- Restore packages: 2m
- Build: 3m
- MySQL wait: 1m
- Migrations: 30s
- ⏱️ SCREENING EXECUTION: 50-70m
  ├─ Stage 0-1 (Data fetching): 30-40m
  ├─ Stage 2-5 (Analysis): 10-15m
  ├─ Stage 6 (Valuations): 8-12m
  └─ Stage 7 (Results): 2-3m
- Post-processing: 3m
- Email: 30s
```

**Artifacts Generated:**

```
screening-results-{RUN_ID}/
├─ screening-results-{YYYY-MM-DD}.csv (Main output)
└─ screening-results-{YYYY-MM-DD}.csv (Duplicate in artifact)

screening-logs-{RUN_ID}/
├─ screening-{YYYY-MM-DD}.log
└─ error-{YYYY-MM-DD}.log (if any errors)
```

Retention: 90 days for CSV, 30 days for logs
Weekly Stock Price Update Workflow
File: .github/workflows/weekly-stock-update.yml

**Purpose:**
Update current stock prices for all qualified companies
Recalculate valuation metrics (P/E, margin of safety, etc.)
Lightweight operation (no full screening)
No email notifications

*Schedule:**
Every Monday at 8:00 PM UTC (after market close)
Manual trigger available

**Trigger Configuration:**

```
name: Weekly Stock Price Update

on:
  schedule:
    # Every Monday at 8 PM UTC
    - cron: '0 20 * * 1'
  
  workflow_dispatch:  # Allow manual trigger
```

**Workflow Steps:**

```
jobs:
  weekly-stock-update:
    runs-on: ubuntu-latest
    timeout-minutes: 30  # Much shorter than quarterly run
    
    services:
      mysql:
        image: mysql:8.0
        env:
          MYSQL_ROOT_PASSWORD: root
          MYSQL_DATABASE: stockscreener
        options: >-
          --health-cmd="mysqladmin ping"
          --health-interval=10s
          --health-timeout=5s
          --health-retries=3
        ports:
          - 3306:3306
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '8.0.x'
      
      - name: Restore and build
        run: |
          dotnet restore src/StockMarketScreener.Console/StockMarketScreener.Console.csproj
          dotnet build src/StockMarketScreener.Console/StockMarketScreener.Console.csproj \
            --configuration Release --no-restore
      
      - name: Wait for MySQL
        run: |
          until mysql -h 127.0.0.1 -u root -proot -e "SELECT 1"; do
            echo "Waiting for MySQL..."
            sleep 5
          done
      
      - name: Run migrations
        env:
          ConnectionStrings__DefaultConnection: "Server=127.0.0.1;Uid=root;Pwd=root;Database=stockscreener"
        run: |
          dotnet ef database update \
            --project src/StockMarketScreener.Console/StockMarketScreener.Console.csproj
      
      - name: Execute stock price update
        env:
          ConnectionStrings__DefaultConnection: "Server=127.0.0.1;Uid=root;Pwd=root;Database=stockscreener"
        run: |
          echo "=== Stock Price Update (Lightweight Job) ==="
          echo "Updating prices for all qualified companies..."
          
          dotnet run \
            --project src/StockMarketScreener.Console/StockMarketScreener.Console.csproj \
            --configuration Release \
            --no-build \
            -- --job StockPriceUpdate
      
      - name: Update GitHub status
        if: always()
        run: |
          echo "## 📈 Weekly Stock Price Update" >> $GITHUB_STEP_SUMMARY
          echo "**Status:** ✅ Complete" >> $GITHUB_STEP_SUMMARY
          echo "**Updated:** $(date '+%Y-%m-%d %H:%M UTC')" >> $GITHUB_STEP_SUMMARY
      
      - name: Upload logs
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: stock-update-logs-${{ github.run_id }}
          path: logs/
          retention-days: 7
```

**Execution Time:**

Setup: 2-3 minutes
Stock price fetch & update: 2-5 minutes
Total: ~5-8 minutes (very lightweight)
No Email Sent: This is a silent update. Errors logged but no notification.

###Manual Trigger Instructions

**Trigger Quarterly Screening Manually:**
Go to: GitHub → Actions → "Quarterly Stock Market Screening"
Click "Run workflow" button
Optional: Enter custom industries (or use defaults)
Click green "Run workflow" button
Monitor execution in real-time
Download CSV results from Artifacts when complete

**Trigger Stock Price Update Manually:**
Go to: GitHub → Actions → "Weekly Stock Price Update"
Click "Run workflow" button
Click green "Run workflow" button
Monitor for completion (~5 minutes)
Monitoring & Debugging

**View Workflow Logs:**
Go to: Repository → Actions
Click on the workflow run
Click on "quarterly-screening" or "weekly-stock-update"
View logs in real-time
Scroll to see:
Data fetching progress
Stage completion messages
Any errors or warnings
Final summary statistics

**Common Issues & Solutions:**

| Issue |	Cause |	Solution |
|-------|-------|----------|
|"Connection refused"|	MySQL not ready|	Wait for health check (automatic)|
|"API rate limit exceeded"|	Too many SEC/Yahoo calls|	Implement caching (future enhancement)|
|"Database migration failed"|	Schema mismatch|	Verify EF Core migrations are up to date|
|"Email not sent"|	Gmail SMTP blocked|	Enable "Less secure apps" or use app password|
|"Timeout exceeded"|	Screening taking too long|	Increase timeout in workflow YAML|

**Download Results:**
Workflow completes → View run summary
Scroll to "Artifacts" section
Click "screening-results-{RUN_ID}"
Download screening-results-{YYYY-MM-DD}.csv
Import into Excel/Sheets for analysis
Workflow Maintenance

**Update Schedule:**
Review workflows quarterly
Update .NET SDK version as needed
Monitor for GitHub Actions deprecations
Test manual triggers monthly

**Best Practices:**
Keep secrets secure (never commit to repo)
Use branch protection rules
Run tests before merging workflow changes
Monitor execution times for performance trends
Keep workflow YAML documented with comments

---

## Testing Strategy

### Overview

Comprehensive testing strategy covers all 7 stages with focus on:
- **Unit tests** for individual components (metrics, valuation methods, scoring)
- **Integration tests** for database operations and multi-stage flows
- **End-to-end tests** simulating complete 7-stage pipeline
- **Valuation accuracy tests** with known companies and expected results

---

### Unit Tests

**Testing Framework:** NUnit + Moq

**Test Project Structure:**

```
StockMarketScreener.Tests/
├─ Unit/
│ ├─ MetricCalculationTests.cs
│ ├─ HardFilterEvaluationTests.cs
│ ├─ ScoringTests.cs
│ ├─ ValuationTests/
│ │ ├─ DCFValuationServiceTests.cs
│ │ ├─ DDMValuationServiceTests.cs
│ │ ├─ MultiplesValuationServiceTests.cs
│ │ ├─ BuffettValuationServiceTests.cs
│ │ └─ MungerValuationServiceTests.cs
│ └─ ConsensusValuationTests.cs
│ ├─ Integration/
│ ├─ ScreeningPipelineTests.cs
│ ├─ DatabaseStorageTests.cs
│ └─ CSVExportTests.cs
│ └─ Data/
└─ TestDataBuilder.cs (10 companies with known results)
```

---

### 1. Metric Calculation Tests

**Focus:** Verify all 40+ metrics calculate correctly

**Test Coverage:**

```
[TestFixture]
public class MetricCalculationTests
{
    private MetricCalculationService _service;
    private TestDataBuilder _testData;
    
    [SetUp]
    public void Setup()
    {
        _service = new MetricCalculationService();
        _testData = new TestDataBuilder();
    }
    
    // Return on Capital Tests
    [Test]
    public void CalculateROE_WithKnownData_ReturnsCorrectValue()
    {
        // Arrange
        var company = _testData.BuildMicrosoftWithRealData();
        
        // Act
        var metrics = _service.CalculateAllMetrics(company);
        
        // Assert
        Assert.That(metrics.ReturnOnEquity_10YAvg, Is.GreaterThan(15));
        Assert.That(metrics.ReturnOnEquity_10YAvg, Is.LessThan(40));
    }
    
    [Test]
    public void CalculateROE_TrendAnalysis_ShowsGrowingOrStable()
    {
        // Arrange
        var company = _testData.BuildAppleWithRealData();
        
        // Act
        var metrics = _service.CalculateAllMetrics(company);
        var roeByYear = metrics.ReturnOnEquity_ByYear;
        
        // Assert - ROE should not decline significantly
        var recentROE = roeByYear.Skip(8).Average();
        var olderROE = roeByYear.Take(2).Average();
        Assert.That(recentROE, Is.GreaterThan(olderROE * 0.8)); // Not declined >20%
    }
    
    // Profitability Tests
    [Test]
    public void CalculateNetMargin_WithStrongBusiness_Above10Percent()
    {
        var company = _testData.BuildCocaColaWithRealData();
        var metrics = _service.CalculateAllMetrics(company);
        
        Assert.That(metrics.NetProfitMargin_10YAvg, Is.GreaterThan(10));
    }
    
    [Test]
    public void CalculateMarginStability_StableCompany_LowCoefficientOfVariation()
    {
        var company = _testData.BuildJNJWithRealData();
        var metrics = _service.CalculateAllMetrics(company);
        
        // Stable business should have CV < 15%
        Assert.That(metrics.MarginStability_CV, Is.LessThan(15));
    }
    
    // Cash Flow Tests
    [Test]
    public void CalculateFreeCAshFlow_QualityBusiness_PositiveEveryYear()
    {
        var company = _testData.BuildMicrosoftWithRealData();
        var metrics = _service.CalculateAllMetrics(company);
        
        // FCF should be positive every year
        Assert.That(metrics.FreeCashFlow_ByYear, Has.All.GreaterThan(0));
    }
    
    [Test]
    public void CalculateFCFToNetIncome_HighQualityEarnings_Above70Percent()
    {
        var company = _testData.BuildBerkshireHathawayWithRealData();
        var metrics = _service.CalculateAllMetrics(company);
        
        Assert.That(metrics.FCFToNetIncomeRatio_10YAvg, Is.GreaterThan(0.70));
    }
    
    // Growth Tests
    [Test]
    public void CalculateRevenuCAGR_GrowingBusiness_PositiveGrowth()
    {
        var company = _testData.BuildNvidiaWithRealData();
        var metrics = _service.CalculateAllMetrics(company);
        
        Assert.That(metrics.RevenueGrowth_10YCAGR, Is.GreaterThan(0));
    }
    
    // All Metrics Calculation
    [Test]
    public void CalculateAllMetrics_CompleteDataset_Returns40PlusMetrics()
    {
        var company = _testData.BuildMicrosoftWithRealData();
        var metrics = _service.CalculateAllMetrics(company);
        
        // Should have metrics for all categories
        Assert.That(metrics.ReturnOnEquity_10YAvg, Is.Not.Null);
        Assert.That(metrics.DebtToEquityRatio_Current, Is.Not.Null);
        Assert.That(metrics.NetProfitMargin_10YAvg, Is.Not.Null);
        Assert.That(metrics.FreeCashFlow_ByYear, Is.Not.Null);
        Assert.That(metrics.RevenueGrowth_10YCAGR, Is.Not.Null);
        Assert.That(metrics.PERatio_Current, Is.Not.Null);
        // ... many more assertions
    }
}
```

###2. Hard Filter Evaluation Tests

**Focus:**
Verify holistic assessment using complete metrics

```
[TestFixture]
public class HardFilterEvaluationTests
{
    private HardFilterEvaluationService _service;
    private TestDataBuilder _testData;
    
    [SetUp]
    public void Setup()
    {
        _service = new HardFilterEvaluationService();
        _testData = new TestDataBuilder();
    }
    
    [Test]
    public void EvaluateCompany_StrongBusiness_Passes()
    {
        // Arrange
        var company = _testData.BuildMicrosoftWithRealData();
        
        // Act
        var result = _service.EvaluateCompany(company);
        
        // Assert
        Assert.That(result.PassedHardFilters, Is.True);
    }
    
    [Test]
    public void EvaluateCompany_WeakBusinessWithHighDebt_Fails()
    {
        // Arrange
        var company = _testData.BuildWeakCompanyHighDebtScenario();
        
        // Act
        var result = _service.EvaluateCompany(company);
        
        // Assert
        Assert.That(result.PassedHardFilters, Is.False);
    }
    
    [Test]
    public void EvaluateCompany_GoodMetricsButNegativeFCF_Fails()
    {
        // Arrange
        var company = _testData.BuildGoodScoresButNegativeFCFScenario();
        
        // Act
        var result = _service.EvaluateCompany(company);
        
        // Assert - Should fail despite good appearance
        Assert.That(result.PassedHardFilters, Is.False);
        Assert.That(result.FailureReason, Contains.Substring("FCF"));
    }
    
    [Test]
    public void EvaluateCompany_ContextualAssessment_AccountsForMetricContext()
    {
        // Arrange - High D/E but growing equity and strong cash flow
        var company = _testData.BuildHighDEButHealthyScenario();
        
        // Act
        var result = _service.EvaluateCompany(company);
        
        // Assert - Should pass due to contextual assessment
        // (not just isolated D/E ratio)
        Assert.That(result.PassedHardFilters, Is.True);
    }
}
```

###3. Valuation Service Tests
 
**Focus:**
Verify each valuation method produces reasonable results

```
[TestFixture]
public class DCFValuationServiceTests
{
    private DCFValuationService _service;
    private TestDataBuilder _testData;
    
    [SetUp]
    public void Setup()
    {
        _service = new DCFValuationService();
        _testData = new TestDataBuilder();
    }
    
    [Test]
    public void CalculateDCF_MicrosoftRealData_FairValueWithinKnownRange()
    {
        // Arrange
        var company = _testData.BuildMicrosoftWithRealData();
        var currentPrice = 420.50m;
        
        // Act
        var result = _service.CalculateFairValue(company);
        
        // Assert - Fair value should be reasonable
        Assert.That(result.FairValuePerShare, Is.GreaterThan(200));
        Assert.That(result.FairValuePerShare, Is.LessThan(700));
        
        // Should have sensitivity analysis
        Assert.That(result.SensitivityAnalysis, Is.Not.Null);
        Assert.That(result.SensitivityAnalysis.Low, Is.LessThan(result.FairValuePerShare));
        Assert.That(result.SensitivityAnalysis.High, Is.GreaterThan(result.FairValuePerShare));
    }
    
    [Test]
    public void CalculateDCF_HighGrowthCompany_AccountsForGrowthRate()
    {
        var company = _testData.BuildNvidiaWithRealData();
        var result = _service.CalculateFairValue(company);
        
        // High growth company should have higher multiple
        Assert.That(result.FairValuePerShare, Is.GreaterThan(0));
    }
    
    [Test]
    public void CalculateDCF_SensitivityAnalysis_ShowsImpactOfAssumptions()
    {
        var company = _testData.BuildMicrosoftWithRealData();
        var result = _service.CalculateFairValue(company);
        
        var sensitivity = result.SensitivityAnalysis;
        
        // Sensitivity should show range of outcomes
        var range = sensitivity.High - sensitivity.Low;
        var midpoint = (sensitivity.High + sensitivity.Low) / 2;
        
        Assert.That(range / midpoint, Is.GreaterThan(0.3)); // At least 30% range
    }
}

[TestFixture]
public class BuffettValuationServiceTests
{
    private BuffettValuationService _service;
    private TestDataBuilder _testData;
    
    [SetUp]
    public void Setup()
    {
        _service = new BuffettValuationService();
        _testData = new TestDataBuilder();
    }
    
    [Test]
    public void CalculateOwnerEarnings_AppleRealData_ReasonableMultiple()
    {
        // Arrange
        var company = _testData.BuildAppleWithRealData();
        
        // Act
        var result = _service.CalculateOwnerEarnings(company);
        
        // Assert
        Assert.That(result.FairValuePerShare, Is.GreaterThan(0));
        Assert.That(result.QualityMultiple, Is.GreaterThanOrEqualTo(15));
        Assert.That(result.QualityMultiple, Is.LessThanOrEqualTo(35));
    }
    
    [Test]
    public void CalculateEarningsYield_StrongBusiness_ExceedsTreasuryRate()
    {
        var company = _testData.BuildMicrosoftWithRealData();
        var treasuryRate = 4.5m; // Assume 10Y Treasury at 4.5%
        
        var result = _service.CalculateEarningsYield(company);
        
        // Earnings yield should exceed risk-free rate
        Assert.That(result.EarningsYieldPercent, Is.GreaterThan(treasuryRate));
    }
}

[TestFixture]
public class MungerValuationServiceTests
{
    private MungerValuationService _service;
    private TestDataBuilder _testData;
    
    [SetUp]
    public void Setup()
    {
        _service = new MungerValuationService();
        _testData = new TestDataBuilder();
    }
    
    [Test]
    public void AssessBusinessQuality_AppleRealData_HighQualityScore()
    {
        var company = _testData.BuildAppleWithRealData();
        var result = _service.AssessBusinessQuality(company);
        
        // Apple should score high on all factors
        Assert.That(result.OverallQualityScore, Is.GreaterThan(40));
        Assert.That(result.MoatStrength, Is.GreaterThan(8));
    }
    
    [Test]
    public void AssessBusinessQuality_WeakBusiness_LowScore()
    {
        var company = _testData.BuildWeakBusinessScenario();
        var result = _service.AssessBusinessQuality(company);
        
        // Weak business should score low
        Assert.That(result.OverallQualityScore, Is.LessThan(25));
    }
    
    [Test]
    public void FiveStockTest_ExceptionalBusiness_Passes()
    {
        var company = _testData.BuildBerkshireHathawayWithRealData();
        var result = _service.RunFiveStockTest(company);
        
        Assert.That(result.TestPassed, Is.True);
    }
}

[TestFixture]
public class ConsensusValuationTests
{
    private ConsensusValuationService _service;
    
    [SetUp]
    public void Setup()
    {
        _service = new ConsensusValuationService();
    }
    
    [Test]
    public void CalculateConsensus_AllMethodsProvided_ProducesWeightedAverage()
    {
        // Arrange
        var valuations = new List<ValuationResult>
        {
            new ValuationResult { Method = "DCF", FairValue = 300m },
            new ValuationResult { Method = "DDM", FairValue = 250m },
            new ValuationResult { Method = "Multiples", FairValue = 280m },
            new ValuationResult { Method = "Buffett", FairValue = 320m },
            new ValuationResult { Method = "Munger", FairValue = 290m }
        };
        
        // Act
        var consensus = _service.CalculateConsensus(valuations);
        
        // Assert
        Assert.That(consensus.FairValuePerShare, Is.GreaterThan(250));
        Assert.That(consensus.FairValuePerShare, Is.LessThan(320));
    }
    
    [Test]
    public void CalculateMarginOfSafety_Overvalued_NegativeMOS()
    {
        var fairValue = 250m;
        var currentPrice = 300m;
        
        var result = _service.CalculateMarginOfSafety(fairValue, currentPrice);
        
        Assert.That(result.MarginOfSafetyPercent, Is.LessThan(0));
        Assert.That(result.Status, Is.EqualTo("OVERVALUED"));
    }
    
    [Test]
    public void AssignValuationGrade_SlamDunk_AssignsAPlusGrade()
    {
        var qualityScore = 85;
        var mosPercent = 35m;
        
        var grade = _service.AssignValuationGrade(qualityScore, mosPercent);
        
        Assert.That(grade, Is.EqualTo("A+"));
    }
}
```

###4. Database Storage Tests

**Focus:**
Verify CompanyValuations data persists correctly

```
[TestFixture]
public class CompanyValuationRepositoryTests
{
    private ICompanyValuationRepository _repository;
    private ScreenerDbContext _dbContext;
    
    [SetUp]
    public void Setup()
    {
        // Use in-memory SQLite for tests
        var options = new DbContextOptionsBuilder<ScreenerDbContext>()
            .UseSqlite("Data Source=:memory:")
            .Options;
        
        _dbContext = new ScreenerDbContext(options);
        _dbContext.Database.EnsureCreated();
        
        _repository = new CompanyValuationRepository(_dbContext);
    }
    
    [TearDown]
    public void Cleanup()
    {
        _dbContext?.Dispose();
    }
    
    [Test]
    public async Task InsertValuation_NewRecord_SavesSuccessfully()
    {
        // Arrange
        var valuation = new CompanyValuation
        {
            CompanyId = 1,
            ScreeningRunId = 1,
            ValuationDate = DateTime.UtcNow,
            CurrentStockPrice = 350m,
            DCFFairValuePerShare = 400m,
            ConsensusFairValuePerShare = 380m,
            ConsensusMarginOfSafetyPercent = 8.5m
        };
        
        // Act
        var result = await _repository.InsertOrUpdateAsync(valuation);
        
        // Assert
        Assert.That(result, Is.GreaterThan(0));
        
        var retrieved = await _repository.GetLatestByCompanyAsync(1);
        Assert.That(retrieved.ConsensusFairValuePerShare, Is.EqualTo(380m));
    }
    
    [Test]
    public async Task UpdateValuation_ExistingRecord_UpdatesSuccessfully()
    {
        // Arrange
        var valuation1 = new CompanyValuation
        {
            CompanyId = 1,
            ScreeningRunId = 1,
            ValuationDate = DateTime.UtcNow,
            ConsensusFairValuePerShare = 380m
        };
        await _repository.InsertOrUpdateAsync(valuation1);
        
        // Act - Insert updated record for same company/run
        var valuation2 = new CompanyValuation
        {
            CompanyId = 1,
            ScreeningRunId = 1,
            ValuationDate = DateTime.UtcNow,
            ConsensusFairValuePerShare = 390m // Updated
        };
        await _repository.InsertOrUpdateAsync(valuation2);
        
        // Assert
        var count = await _dbContext.CompanyValuations
            .Where(v => v.CompanyId == 1 && v.ScreeningRunId == 1)
            .CountAsync();
        
        Assert.That(count, Is.EqualTo(1)); // Still only 1 record
        
        var retrieved = await _repository.GetLatestByCompanyAsync(1);
        Assert.That(retrieved.ConsensusFairValuePerShare, Is.EqualTo(390m));
    }
    
    [Test]
    public async Task GetHistory_MultipleRuns_ReturnsAllRecords()
    {
        // Arrange - Insert valuations from multiple screening runs
        var val1 = new CompanyValuation { CompanyId = 1, ScreeningRunId = 1, ValuationDate = DateTime.UtcNow.AddMonths(-3) };
        var val2 = new CompanyValuation { CompanyId = 1, ScreeningRunId = 2, ValuationDate = DateTime.UtcNow };
        
        await _repository.InsertOrUpdateAsync(val1);
        await _repository.InsertOrUpdateAsync(val2);
        
        // Act
        var history = await _repository.GetHistoryAsync(companyId: 1, months: 12);
        
        // Assert
        Assert.That(history, Has.Count.EqualTo(2));
    }
}
```

###5. CSV Export Tests

**Focus:**
Verify CSV generation with correct data and formatting

```
[TestFixture]
public class CSVExportTests
{
    private CSVExportService _service;
    private ICompanyValuationRepository _repository;
    
    [SetUp]
    public void Setup()
    {
        _service = new CSVExportService();
        // Mock repository...
    }
    
    [Test]
    public async Task ExportResults_ValidData_GeneratesCSVFile()
    {
        // Arrange
        var valuations = new List<CompanyValuation>
        {
            new CompanyValuation
            {
                Company = new Company { Ticker = "AAPL", CompanyName = "Apple" },
                ConsensusFairValuePerShare = 250m,
                CurrentStockPrice = 180m
            }
        };
        
        var outputDir = Path.Combine(Path.GetTempPath(), "csv-test");
        Directory.CreateDirectory(outputDir);
        
        // Act
        await _service.ExportScreeningResultsAsync(valuations, outputDir);
        
        // Assert
        var files = Directory.GetFiles(outputDir, "screening-results-*.csv");
        Assert.That(files, Has.Length.EqualTo(1));
        
        var fileContent = File.ReadAllText(files[0]);
        Assert.That(fileContent, Contains.Substring("AAPL"));
        Assert.That(fileContent, Contains.Substring("Apple"));
        Assert.That(fileContent, Contains.Substring("250"));
    }
    
    [Test]
    public async Task ExportResults_MultipleCompanies_IncludesAllRows()
    {
        // Arrange
        var valuations = TestDataBuilder.BuildSampleValuations(45); // 45 companies
        
        // Act
        await _service.ExportScreeningResultsAsync(valuations, tempDir);
        
        // Assert
        var fileContent = File.ReadAllLines(csvFile);
        Assert.That(fileContent, Has.Length.EqualTo(46)); // Header + 45 data rows
    }
}
```

###6. End-to-End Pipeline Tests

**Focus:**
Verify complete 7-stage flow with real data

```
[TestFixture]
public class ScreeningPipelineTests
{
    private ScreeningOrchestrator _orchestrator;
    private ICompanyValuationRepository _valuationRepository;
    private ScreenerDbContext _dbContext;
    
    [SetUp]
    public void Setup()
    {
        // Setup in-memory database for testing
        var options = new DbContextOptionsBuilder<ScreenerDbContext>()
            .UseSqlite("Data Source=:memory:")
            .Options;
        
        _dbContext = new ScreenerDbContext(options);
        _dbContext.Database.EnsureCreated();
        
        // Setup orchestrator with mocked services
        _orchestrator = BuildTestOrchestrator();
        _valuationRepository = new CompanyValuationRepository(_dbContext);
    }
    
    [Test]
    public async Task RunFullPipeline_TestData_CompletesSuccessfully()
    {
        // Arrange
        var testCompanies = TestDataBuilder.BuildSampleUniverseWithRealData(10);
        
        // Act
        var result = await _orchestrator.RunScreeningAsync(testCompanies);
        
        // Assert
        Assert.That(result.ScreeningStatus, Is.EqualTo("Success"));
        Assert.That(result.CompaniesQualified, Is.GreaterThan(0));
        Assert.That(result.CompaniesValuated, Is.EqualTo(result.CompaniesQualified));
    }
    
    [Test]
    public async Task RunFullPipeline_TestData_StoresValuationsInDatabase()
    {
        // Arrange
        var testCompanies = TestDataBuilder.BuildSampleUniverseWithRealData(10);
        
        // Act
        await _orchestrator.RunScreeningAsync(testCompanies);
        
        // Assert
        var storedValuations = await _valuationRepository.GetByScreeningRunAsync(screeningRunId: 1);
        Assert.That(storedValuations, Is.Not.Empty);
        
        // Verify all stored records have complete data
        foreach (var valuation in storedValuations)
        {
            Assert.That(valuation.DCFFairValuePerShare, Is.GreaterThan(0));
            Assert.That(valuation.ConsensusFairValuePerShare, Is.GreaterThan(0));
            Assert.That(valuation.ValuationGrade, Is.Not.Null);
        }
    }
    
    [Test]
    public async Task RunFullPipeline_KnownCompanies_ProducesReasonableResults()
    {
        // Arrange
        var companies = TestDataBuilder.BuildKnownCompanies(
            "AAPL", "MSFT", "JNJ", "KO", "JPM");
        
        // Act
        var result = await _orchestrator.RunScreeningAsync(companies);
        
        // Assert - Apple and Microsoft should both qualify
        var stored = await _valuationRepository.GetByScreeningRunAsync(1);
        
        var aapl = stored.FirstOrDefault(v => v.Company.Ticker == "AAPL");
        var msft = stored.FirstOrDefault(v => v.Company.Ticker == "MSFT");
        
        Assert.That(aapl, Is.Not.Null);
        Assert.That(msft, Is.Not.Null);
        
        // Fair values should be reasonable
        Assert.That(aapl.ConsensusFairValuePerShare, Is.GreaterThan(50));
        Assert.That(msft.ConsensusFairValuePerShare, Is.GreaterThan(100));
    }
}
```

###7. Test Data Builder

**Helper class to generate test data:**

```
public class TestDataBuilder
{
    public static Company BuildMicrosoftWithRealData()
    {
        return new Company
        {
            Ticker = "MSFT",
            CompanyName = "Microsoft Corporation",
            Industry = "Software",
            FinancialData = new List<YearlyFinancials>
            {
                // 10 years of real data (2016-2025)
                new YearlyFinancials { FiscalYear = 2016, Revenue = 90154, NetIncome = 20988, /* ... */ },
                new YearlyFinancials { FiscalYear = 2017, Revenue = 89950, NetIncome = 21204, /* ... */ },
                // ... through 2025
                new YearlyFinancials { FiscalYear = 2025, Revenue = 245122, NetIncome = 88188, /* ... */ }
            }
        };
    }
    
    public static Company BuildAppleWithRealData()
    {
        return new Company
        {
            Ticker = "AAPL",
            CompanyName = "Apple Inc.",
            Industry = "Consumer Electronics",
            FinancialData = BuildRealFinancialData("AAPL")
        };
    }
    
    public static List<Company> BuildSampleUniverseWithRealData(int count)
    {
        return new List<Company>
        {
            BuildMicrosoftWithRealData(),
            BuildAppleWithRealData(),
            BuildCocaColaWithRealData(),
            // ... more companies
        }.Take(count).ToList();
    }
    
    public static List<CompanyValuation> BuildSampleValuations(int count)
    {
        var valuations = new List<CompanyValuation>();
        for (int i = 1; i <= count; i++)
        {
            valuations.Add(new CompanyValuation
            {
                CompanyId = i,
                DCFFairValuePerShare = 300m + (i * 5),
                ConsensusFairValuePerShare = 290m + (i * 5),
                CurrentStockPrice = 250m + (i * 3),
                ValuationGrade = i % 10 == 0 ? "A+" : i % 5 == 0 ? "A" : "B+"
            });
        }
        return valuations;
    }
}
```

###Test Execution

**Run all tests:**
```
dotnet test src/StockMarketScreener.Tests/StockMarketScreener.Tests.csproj
```
Run specific test class:
```
dotnet test --filter "ClassName=DCFValuationServiceTests"
```

Run with code coverage:
```
dotnet test /p:CollectCoverage=true /p:CoverageFormat=cobertura
```

**Target Coverage:**

Unit tests: 80%+ coverage
Critical paths (valuation, storage): 95%+ coverage
Integration tests: All major flows
Continuous Integration

**GitHub Actions test workflow:**

```
name: Run Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '8.0.x'
      
      - run: dotnet test src/StockMarketScreener.Tests/ --logger "console;verbosity=detailed"
```

Tests run on every push and pull request to ensure quality.

---

## Deployment & Operations

### Local Development Setup

**Prerequisites:**
- Windows 10+, macOS, or Linux
- Visual Studio 2022 Community (or VS Code)
- .NET 8.0 SDK
- MySQL 8.0
- Git

**Step-by-Step Setup:**

```
# 1. Clone repository
git clone https://github.com/ThommCroft/StockMarketScreener.git
cd StockMarketScreener

# 2. Start MySQL (Docker - recommended)
docker run --name stockscreener-mysql \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_DATABASE=stockscreener \
  -d -p 3306:3306 \
  mysql:8.0

# Wait for MySQL to be ready
sleep 10

# 3. OR Start MySQL (Native - if Docker not available)
# macOS: brew install mysql && mysql.server start
# Windows: Download & install MySQL Community Server
# Linux: sudo apt-get install mysql-server && sudo systemctl start mysql

# 4. Configure database connection
# Edit appsettings.json:
# "ConnectionStrings": {
#   "DefaultConnection": "Server=localhost;Uid=root;Pwd=root;Database=stockscreener"
# }

# 5. Restore dependencies
dotnet restore

# 6. Build project (Release configuration)
dotnet build --configuration Release

# 7. Create/update database schema (EF Core migrations)
dotnet ef database update \
  --project src/StockMarketScreener.Console/StockMarketScreener.Console.csproj

# 8. Verify setup with unit tests
dotnet test src/StockMarketScreener.Tests/

# 9. Run screening locally (full 7-stage pipeline)
dotnet run \
  --project src/StockMarketScreener.Console/StockMarketScreener.Console.csproj \
  --configuration Release

# 10. Check results
ls -la output/                           # CSV export
ls -la logs/                             # Execution logs
mysql -u root -p stockscreener          # Query database
```

**Verify Setup Success:**

✅ MySQL running and accessible
✅ Database schema created (Tables: Companies, QualifiedCompanies, CompanyValuations, ScreeningRuns, etc.)
✅ Application compiles without errors
✅ Unit tests pass
✅ Sample screening run completes (should take 30-45 minutes)
✅ CSV file generated in output/ directory
✅ Results stored in database

###GitHub Actions Deployment

No Special Deployment Needed!

The application runs entirely in GitHub Actions. No server to manage.

**How it works:**

1. Repository code exists in GitHub
2. Workflows run on GitHub's runners (ubuntu-latest)
3. MySQL runs in a container service (ephemeral)
4. Results saved as artifacts (CSV files)
5. Data stored in your local MySQL (via backup/sync)
6. Everything automated - no manual deployment needed

**Code Changes → Automatic Deployment:**
Make changes to code locally
Commit and push to GitHub
Tests run automatically (optional)
Workflow runs on schedule
Results available in Actions tab
CSV downloaded and analyzed
No infrastructure, no servers, no DevOps complexity.

###Database Management

**Local MySQL Backup Strategy:**

```
# Full backup
mysqldump -u root -p stockscreener > backup_$(date +%Y-%m-%d).sql

# Restore from backup
mysql -u root -p stockscreener < backup_2026-03-15.sql

# Backup with compression
mysqldump -u root -p stockscreener | gzip > backup_$(date +%Y-%m-%d).sql.gz

# Verify backup integrity
mysql -u root -p < backup_2026-03-15.sql --dry-run
```

**Backup Schedule (Recommended):**

**Manual Backups:**
- Before major code changes
- After each screening run
- Monthly archive (keep 12 months)

**Automated Backups (Optional):**
- Add cron job for daily backups
- Store in cloud storage (S3, Azure Blob, Google Cloud)
- Implement 30-day retention policy

**Database Queries for Operations:**

```
-- View all qualified companies (current state)
SELECT Ticker, CompanyName, CompositeScore, QualityScore,
       FirstQualifiedDate, LastQualifiedDate
FROM QualifiedCompanies
WHERE IsActive = 1
ORDER BY QualityScore DESC;

-- View all valuations from latest screening run
SELECT c.Ticker, c.CompanyName, cv.ConsensusFairValuePerShare,
       cv.CurrentStockPrice, cv.ConsensusMarginOfSafetyPercent,
       cv.ValuationGrade, cv.ConsensusRecommendation
FROM CompanyValuations cv
JOIN Companies c ON cv.CompanyId = c.CompanyId
WHERE cv.ValuationDate = (SELECT MAX(ValuationDate) FROM CompanyValuations)
ORDER BY cv.ConsensusMarginOfSafetyPercent DESC;

-- Check screening run history
SELECT RunDate, Status, CompaniesProcessed, CompaniesQualified,
       CompaniesValuated, ExecutionTimeSeconds,
       ROUND(ExecutionTimeSeconds / 60, 1) as ExecutionMinutes
FROM ScreeningRuns
ORDER BY RunDate DESC
LIMIT 10;

-- Find companies removed in last screening
SELECT Ticker, CompanyName, ReasonRemoved, LastCompositeScore,
       DateRemoved
FROM RemovedCompanies
WHERE DateRemoved >= DATE_SUB(NOW(), INTERVAL 3 MONTH)
ORDER BY DateRemoved DESC;

-- Compare valuations over time for one company
SELECT ValuationDate, ConsensusFairValuePerShare, CurrentStockPrice,
       ConsensusMarginOfSafetyPercent, ValuationGrade
FROM CompanyValuations
WHERE CompanyId = (SELECT CompanyId FROM Companies WHERE Ticker = 'AAPL')
ORDER BY ValuationDate DESC
LIMIT 8;

-- Identify methodology divergence (Buffett vs Munger)
SELECT c.Ticker, BuffettRecommendation, MungerRecommendation,
       ROUND(ABS(BuffettFairValuePerShare - MungerFairValuePerShare), 2) as Divergence
FROM CompanyValuations cv
JOIN Companies c ON cv.CompanyId = c.CompanyId
WHERE cv.ValuationDate = (SELECT MAX(ValuationDate) FROM CompanyValuations)
  AND BuffettRecommendation != MungerRecommendation
ORDER BY Divergence DESC;
```

###Configuration Management

**appsettings.json Structure:**

```
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Uid=root;Pwd=root;Database=stockscreener"
  },
  "Screening": {
    "IndustriesToScreen": [
      "Consumer Defensive",
      "Consumer Cyclical",
      "Technology",
      "Communication Services",
      "Healthcare",
      "Defensive Industry"
    ],
    "QuarterlyRunDates": "15th of March, June, September, December",
    "MinimumYearsOfData": 5,
    "PreferredYearsOfData": 10,
    "PassingScoreThreshold": 75,
    "AutomaticRemovalOfFailedCompanies": true
  },
  "DataFetching": {
    "SecEdgarTimeout": 30,
    "YahooFinanceTimeout": 30,
    "MaxRetries": 3,
    "RetryDelaySeconds": 5
  },
  "Valuation": {
    "DCFProjectionYears": 10,
    "DCFTerminalGrowthRate": 2.5,
    "BuffettOwnerEarningsYears": 10,
    "MungerQualityAssessmentEnabled": true
  },
  "Output": {
    "ExportCSV": true,
    "ExportPath": "output",
    "LogPath": "logs"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning"
    },
    "File": {
      "IncludeScopes": true,
      "MinimumLevel": "Information"
    }
  }
}
```

**Environment-Specific Configuration:**

```
# Development (local machine)
appsettings.json → appsettings.Development.json

# GitHub Actions (CI/CD)
appsettings.json → Environment variables in GitHub Secrets

# Production (if deployed)
appsettings.json → appsettings.Production.json
```

###Performance Tuning

**Stage 6 Valuation Optimization:**

The valuation stage (Stage 6) is the most time-consuming. 

**Optimizations:**

```
// 1. Parallel Processing (per valuation method)
var tasks = new List<Task<ValuationResult>>
{
    dcfService.CalculateAsync(company),
    ddmService.CalculateAsync(company),
    multiplesService.CalculateAsync(company),
    buffettService.CalculateAsync(company),
    mungerService.CalculateAsync(company)
};

await Task.WhenAll(tasks);  // All 5 methods run simultaneously

// 2. Caching financial data
var cachedData = _cache.GetOrCreate(company.Ticker, () => {
    return FetchFinancialData(company);
});

// 3. Batch database inserts (not one-at-a-time)
using (var transaction = _dbContext.Database.BeginTransaction())
{
    foreach (var valuation in valuations)
    {
        _dbContext.CompanyValuations.Add(valuation);
    }
    _dbContext.SaveChanges();
    transaction.Commit();
}

// 4. Use connection pooling (automatic with HttpClientFactory)
// 5. Implement API rate limiting awareness
```

**Current Performance:**

Data Fetching (Stage 0-1): 30-40 minutes
  - Bottleneck: API calls (not CPU-bound)
  - Optimization: Implement caching between runs

Metric Calculation (Stage 2-3): 5-10 minutes
  - Mostly CPU-bound (can parallelize)
  - Optimization: Process multiple companies in parallel

Valuation (Stage 6): 8-12 minutes
  - 5 methods × 45 companies = 225 valuations
  - Parallelizable: Each method runs independently
  - Optimization: Run all 5 methods simultaneously per company

Total: 50-75 minutes (acceptable for quarterly runs)

**Future Optimizations (Not in v1):**

- Parallel company processing (Stage 0-5)
- Local caching of financial data
- Incremental screening (only re-analyze changed companies)
- Database query optimization with proper indexes
- API response caching
- Distributed processing (if universe grows to 1000+ companies)
Monitoring & Alerting

**What to Monitor:**

**Quarterly Screening:**
✅ Workflow execution success/failure
✅ Execution time (alert if > 2 hours)
✅ Number of companies qualified (alert if < 20 or > 100)
✅ Database storage (check for corruption)
✅ Email delivery (verify email received)

**Weekly Stock Updates:**
✅ Execution success/failure
✅ All qualified companies have updated prices
✅ No stale data (> 7 days old)

**Database Health:**
✅ Backup completion
✅ Query performance
✅ Disk space
✅ Connection pool health

**Application Health:**
✅ Error logs (alert on exceptions)
✅ Warning count
✅ API failures (with retry counts)

*GitHub Actions Logs:**

**Errors and warnings are captured in GitHub Actions logs:**

Go to: Repository → Actions → Latest run

Click workflow step that failed
Expand logs to view details
Search for error, warning, failed

**Database Corruption Check:**

```
# Run MySQL check and repair
mysqlcheck -u root -p stockscreener --all-databases

# Verify table integrity
REPAIR TABLE CompanyValuations;
REPAIR TABLE QualifiedCompanies;
CHECK TABLE CompanyValuations;
```

###Troubleshooting Guide

**Issue:**
"Connection refused" on startup

**Cause:**
MySQL not running or not accessible

**Solution:**
  1. Check MySQL is running: mysql -u root -p -e "SELECT 1;"
  2. Verify connection string in appsettings.json
  3. If using Docker: docker ps (verify container running)
  4. Restart MySQL: docker restart stockscreener-mysql

**Issue:**
"Too many open connections"

**Cause:** 
Connection pool exhausted (API limits or slow responses)

**Solution:**
  1. Implement exponential backoff in retry logic
  2. Reduce MaxPoolSize in connection string if needed
  3. Check API rate limits (SEC EDGAR, Yahoo Finance)
  4. Consider implementing caching layer

**Issue:** 
"Valuation divergence too wide" (Buffett vs Munger)

**Cause:**
Methods using different assumptions about business quality

**Solution:**
  1. Manual review of the company (different perspectives valid)
  2. Check if one method has data quality issue
  3. Investigate what drives the disagreement
  4. Document as "requires further analysis"
 
**Issue:**
"Screening timeout (> 2 hours)"

**Cause:**
API calls taking too long or network issues

**Solution:**
  1. Check network connectivity
  2. Verify API services are up
  3. Review GitHub Actions logs for which stage timed out
  4. Increase timeout in workflow YAML if consistent
  5. Implement caching to skip re-fetching data

**Issue:**
"Database migration failed"

**Cause:**
Schema mismatch or previous failed migration

**Solution:**
  1. Verify database exists: mysql -u root -p -e "SHOW DATABASES;"
  2. Check for pending migrations: dotnet ef migrations list
  3. Remove database and start fresh:
     - DROP DATABASE stockscreener;
     - CREATE DATABASE stockscreener;
     - dotnet ef database update

**Issue:**
"CSV export empty or missing columns"

**Cause:**
Valuation data incomplete or null values

**Solution:**
  1. Verify screening run completed successfully
  2. Check database for CompanyValuations records
  3. Query: SELECT COUNT(*) FROM CompanyValuations;
  4. Ensure all required fields populated before export
  5. Check CSV mapping in CSVExportService

###Backup & Recovery

**Full System Recovery:**

```
# 1. Restore database from backup
mysql -u root -p < backup_2026-03-15.sql.gz

# 2. Verify data integrity
SELECT COUNT(*) FROM Companies;
SELECT COUNT(*) FROM QualifiedCompanies;
SELECT COUNT(*) FROM CompanyValuations;

# 3. Run tests to verify application integrity
dotnet test src/StockMarketScreener.Tests/

# 4. Check recent screening runs
SELECT * FROM ScreeningRuns ORDER BY RunDate DESC LIMIT 5;
```

**Disaster Recovery Plan:**

Scenario 1: Lost Recent Screening Results
  - Restore from backup
  - Re-run screening (Stage 0-7)
  - Restore from GitHub Actions artifacts (90-day retention)

Scenario 2: Corrupted Database
  - Restore from backup
  - Run table repair: REPAIR TABLE CompanyValuations;
  - Re-run data integrity checks

Scenario 3: Lost Code
  - Clone from GitHub: git clone https://github.com/ThommCroft/StockMarketScreener.git
  - Check out specific commit: git checkout COMMIT_SHA
  - Rebuild application and run

Scenario 4: GitHub Actions Workflow Broken
  - Revert workflow file to previous version
  - Fix issue locally, test, then push
  - Trigger manual workflow run

###Maintenance Schedule

**Weekly (Every Monday):**

Stock price update runs automatically
Verify no stale data (> 7 days old)
Check GitHub Actions success/failure

**Monthly:**
Backup database (full backup)
Review error logs for warnings
Check disk space and API limits
Update dependencies (dotnet packages)

**Quarterly (Before scheduled screening):**
Verify GitHub Secrets still valid
Test local screening run
Backup database before production run
Review configuration changes

**Annually:**
Review and archive old screening runs (>1 year)
Update documentation
Plan for future enhancements
Review cost/performance metrics
Scaling Considerations

**Current Setup Handles:**
500 companies per screening run
50-75 minute execution time
4 screening runs per year (quarterly)
~200 companies in active watchlist

**If Universe Grows to 1000+ Companies:**

**Performance Impact:**
  - Data fetching: 60-80 minutes
  - Metric calculation: 10-15 minutes
  - Valuation: 15-20 minutes
  - Total: 90-120 minutes

**Optimizations:**
  1. Parallel company processing (process 10 at a time)
  2. Implement caching layer (reduce API calls)
  3. Distribute across multiple runners (GitHub Actions matrix)
  4. Incremental screening (only changed companies)
  5. 
If Data Retention Grows (10+ Years of Data):

**Database Impact:**
  - Current: ~500 qualified companies × 8 screening runs = 4,000 records
  - After 10 years: ~40,000 records (manageable)

**Actions:**
  - Archive old screening runs (>2 years) to separate table
  - Implement partitioning by ValuationDate
  - Add indexes on frequently queried columns
  - Monitor query performance
Operational Success Criteria

**Screening Run Success Checklist:**

✅ Pipeline completes within 2 hours
✅ All 7 stages execute without fatal errors
✅ At least 20 companies qualify (>= 75 composite)
✅ All qualified companies have valuations
✅ CSV export contains all expected columns
✅ Database stores all results
✅ GitHub Actions workflow marked as success
✅ Email notification sent
✅ No API rate limits exceeded
✅ Error count < 5 total

**Valuation Quality Success Criteria:**

✅ All 5 methods provide fair value
✅ Buffett & Munger consensus within 50% (not divergent)
✅ Margin of safety calculations make sense
✅ Valuation grades consistent with methodology
✅ No extreme outliers (> 10x difference between methods)
✅ Sensitivity analysis shows reasonable range
✅ Recommendations align with margin of safety

**Database Health Success Criteria:**

✅ All tables have data
✅ Foreign key relationships intact
✅ No corrupted records
✅ Indexes performing well
✅ Query response times < 1 second
✅ Backup completes successfully
✅ Recovery test passes

---

## Success Criteria

This architecture is successful when:

### ✅ Unified Metric & Screening Approach

- All 40+ metrics calculated for every company BEFORE any filtering
- No premature rejection based on incomplete data
- Hard filters evaluated with complete financial picture
- Holistic assessment of company strength (not checklist)
- Clear audit trail showing why each company passed or failed

### ✅ Complete 7-Stage Pipeline

**Stage 0: Market Cap Pre-Filter**
- ✅ Filters 500+ companies down to ~400-450 qualified by market cap
- ✅ Saves API calls on illiquid companies
- ✅ Focus on institutional-quality universe

**Stages 1-3: Metric Calculation & Hard Filter Evaluation**
- ✅ Fetches financial data from multiple sources (SEC EDGAR, Yahoo Finance, fallbacks)
- ✅ Calculates ALL 40+ metrics for every company
- ✅ Data reconciliation handles API failures gracefully
- ✅ Hard filters evaluated using complete metric context
- ✅ ~75 companies qualify for quality scoring

**Stages 4-5: Quality Assessment & Composite Scoring**
- ✅ Quality Score (0-100): ROE, Profitability, Cash Flow, Business Quality
- ✅ Valuation Score (0-100): P/E, P/B, earnings yield, margin metrics
- ✅ Management Score (0-100): Insider ownership, accounting quality, capital allocation
- ✅ Composite Score: (Quality×40% + Valuation×35% + Management×25%)
- ✅ Only companies with Composite >= 75 advance to Stage 6
- ✅ ~45 companies typically qualify for valuation

**Stage 6: Intrinsic Value Valuation**
- ✅ DCF Method: 10-year projection, WACC discount, terminal value
- ✅ DDM Method: Dividend/buyback valuation (if applicable)
- ✅ Multiples Method: P/E, P/B, P/S based on peer comparables
- ✅ Buffett Method: Owner's earnings × quality multiple
- ✅ Munger Method: Quality-based multiple assignment
- ✅ Consensus Fair Value: Weighted average of all 5 methods
- ✅ Margin of Safety: Fair Value vs. Current Price
- ✅ Valuation Grade: A+ (Slam Dunk) to F (Below Threshold)
- ✅ Investment Recommendation: BUY / CONSIDER / PASS / AVOID

**Stage 7: Results Reporting & Storage**
- ✅ All valuations stored in MySQL database (audit trail)
- ✅ CSV export with all data (45 companies × 38 columns)
- ✅ GitHub Actions summary (workflow statistics)
- ✅ Console output (real-time feedback)
- ✅ Comparison with prior run (new/maintained/removed)

### ✅ Automation & Reliability

- ✅ Screening runs completely hands-off via GitHub Actions
- ✅ Quarterly trigger (15th of March, June, September, December)
- ✅ Manual trigger available anytime via workflow_dispatch
- ✅ No manual intervention needed
- ✅ Logs show exactly what happened (for debugging)
- ✅ Per-company error handling (don't crash on individual failure)
- ✅ Robust fallbacks for API failures
- ✅ Database integrity maintained across runs

### ✅ Data Quality

- ✅ All 40+ metrics calculate correctly for every company
- ✅ Data reconciliation from multiple sources works properly
- ✅ Metrics within realistic ranges (no outliers)
- ✅ 10-year historical data when available (prefer 5-year minimum)
- ✅ Results match financial reality (not theoretical exercises)
- ✅ Validation rules prevent bad data from corrupting analysis

### ✅ Company Filtering

- ✅ Only passing companies stored (no clutter in results)
- ✅ Complete metric context available for understanding decisions
- ✅ Proper removal when scores decline (automatic cleanup)
- ✅ Tracking of removed companies (why they failed)
- ✅ Comparison with prior runs (new/maintained/removed)
- ✅ ~40-50 qualified companies in typical watchlist

### ✅ Valuation Accuracy

- ✅ All 5 valuation methods produce reasonable fair values
- ✅ Buffett & Munger consensus align (not divergent > 50%)
- ✅ Margin of safety makes sense (not extreme)
- ✅ Valuation grades consistent with methodology
- ✅ Sensitivity analysis shows impact of assumptions
- ✅ Recommendations backed by rigorous analysis

### ✅ Results & Reporting

- ✅ Detailed CSV export (all metrics, valuations, recommendations)
- ✅ GitHub Actions summary (quick statistics and rankings)
- ✅ MySQL database (permanent historical record)
- ✅ Console output (real-time execution feedback)
- ✅ Clear identification (Slam Dunks, Strong Buys, Fair Value, Overvalued)
- ✅ Investment theses documented
- ✅ All data ready for further analysis (Excel, Python, etc.)

### ✅ Solo Developer Friendly

- ✅ Code is maintainable and well-documented
- ✅ Can run locally for testing
- ✅ Easy to modify screening criteria
- ✅ No complex infrastructure to manage
- ✅ No DevOps expertise required
- ✅ Minimal operational overhead
- ✅ Clear error messages for troubleshooting

### ✅ Performance

**Typical Execution Times:**
- **First Run (Full Data Fetch):** 50-75 minutes
  - Data fetching (Stages 0-1): 30-40 minutes
  - Metric calculation (Stages 2-3): 5-10 minutes
  - Hard filter evaluation (Stage 4): 2-3 minutes
  - Quality scoring (Stages 4-5): 4-5 minutes
  - Intrinsic valuation (Stage 6): 8-12 minutes
  - Results processing (Stage 7): 2-3 minutes

- **Stock Price Updates (Weekly):** < 5 minutes
- **Subsequent Runs (With Caching):** 20-30 minutes

- ✅ Acceptable for quarterly runs
- ✅ Weekly updates lightweight and fast
- ✅ Can scale to 1000+ companies with optimization

### ✅ Reliability

- ✅ Robust error handling (continue on individual failures)
- ✅ Multiple data sources prevent single point of failure
- ✅ Audit trail of all decisions and data sources
- ✅ Database integrity maintained
- ✅ Graceful degradation (use fallback sources)
- ✅ Comprehensive logging for debugging
- ✅ Test coverage on critical paths (95%+)

### ✅ Maintainability

- ✅ Clean code architecture (separation of concerns)
- ✅ Dependency injection for testability
- ✅ Well-documented (inline comments, architecture docs)
- ✅ Configuration-driven (easy to modify thresholds)
- ✅ No technical debt
- ✅ Easy to add new features or metrics

---

## Summary: Complete 7-Stage Architecture

The **Stock Market Screener** is a production-ready console application that implements a complete 7-stage investment analysis pipeline based on Warren Buffett and Charlie Munger principles.

### Design Highlights

**Unified Metric Calculation + Hard Filter Evaluation:**
- All 40+ metrics calculated upfront for every company
- Hard filters evaluated using complete metric data
- Context-aware assessment (not checklist-based)
- Prevents false negatives from incomplete analysis

**Multi-Stage Investment Analysis:**
- **Stages 0-5:** Financial Analysis (500 companies → 45 qualified)
- **Stage 6:** Intrinsic Value Valuation (5 independent methods)
- **Stage 7:** Results Reporting (CSV, database, summary)

**Five Valuation Methods:**
1. **DCF** - Discounted Cash Flow (theoretical value)
2. **DDM** - Dividend Discount Model (cash returns)
3. **Multiples** - Peer-based valuation (market reality check)
4. **Buffett** - Owner's earnings method (mechanical simplicity)
5. **Munger** - Quality-based valuation (business judgment)

**Consensus Valuation:**
- Weighted average: DCF 40%, Multiples 20%, Buffett 15%, DDM 15%, Munger 10%
- Produces single fair value with margin of safety analysis
- Grades investment opportunities (A+ to F)
- Recommends actions (BUY / CONSIDER / PASS / AVOID)

### Technology Stack

**Language & Runtime:** C# .NET 8.0 LTS
**Database:** MySQL 8.0 (local or cloud)
**Automation:** GitHub Actions (quarterly + manual)
**External Data:** SEC EDGAR, Yahoo Finance, IEX Cloud, Alpha Vantage

### Workflow

**Quarterly Execution (90 minutes):**

```
GitHub Actions Trigger (15th of quarter)
↓
Stage 0: Market Cap Filter (500 → 450 companies)
↓
Stage 1-3: Data Fetch & ALL Metrics Calculation (450 → 75 qualified)
↓
Stage 4: Hard Filter Evaluation (75 → 75 passed context check)
↓
Stage 5: Quality & Composite Scoring (75 → 45 qualified, ≥75 threshold)
↓
Stage 6: Intrinsic Valuation (45 companies × 5 methods)
↓
Stage 7: Results Reporting
├─ MySQL storage (audit trail)
├─ CSV export (45 companies, 38 columns)
├─ GitHub summary (statistics & rankings)
└─ Console output (execution log)
↓
Email notification → Investment decisions
```

### Key Metrics Tracked

**Comprehensive Financial Analysis (40+ Metrics):**

| Category | Metrics | Purpose |
|----------|---------|---------|
| **Return on Capital** | ROE, ROA, ROIC, ROCE | Management efficiency |
| **Financial Strength** | D/E, Current Ratio, Coverage | Balance sheet health |
| **Profitability** | Net/Gross/Operating Margins | Pricing power |
| **Cash Flow** | OCF, FCF, conversion ratios | Cash quality |
| **Growth** | Revenue CAGR, EPS growth | Business momentum |
| **Valuation** | P/E, P/B, P/S, PEG, Earnings Yield | Price reasonableness |
| **Management** | Insider ownership, Goodwill | Capital allocation |
| **Competitive Advantage** | Moat, Market position, Recurring % | Durability |

### Investment Opportunities

**Typical Results (45 Qualified Companies):**

```
Slam Dunk Buys (A+, MOS ≥ 30%): 3-5 companies → Exceptional businesses at attractive prices → Immediate purchase candidates

Strong Buys (A, MOS 20-30%): 8-12 companies → High quality at reasonable valuations → Build positions

Fair Value (B+, MOS 10-20%): 12-18 companies → Strong businesses, fairly priced → Accumulation opportunities

Overvalued (C, MOS < 0%): 18-25 companies → Good businesses, expensive prices → Watch for better entry points
```

### Success Metrics

✅ **Pipeline:** Completes in 60-75 minutes with all 7 stages
✅ **Accuracy:** Valuations reasonable, methodologies rigorous
✅ **Quality:** Only qualified companies stored (no clutter)
✅ **Automation:** Completely hands-off (no manual work)
✅ **Reliability:** Robust error handling, comprehensive logging
✅ **Reporting:** CSV, database, summary all generated automatically
✅ **Actionability:** Clear investment recommendations backed by analysis

### Ready for Production

This architecture is **complete and production-ready:**

- ✅ Full implementation specification (all 7 stages)
- ✅ Database schema defined
- ✅ Component architecture documented
- ✅ Testing strategy comprehensive
- ✅ Deployment automated (GitHub Actions)
- ✅ Operational procedures documented
- ✅ Troubleshooting guide included
- ✅ Performance characteristics analyzed
- ✅ Scaling considerations addressed

**Next Steps:**

1. **Implementation:** Build C# components in order (Stages 0-7)
2. **Testing:** Unit + integration tests for each stage
3. **Local Validation:** Run complete pipeline locally
4. **GitHub Setup:** Configure Actions workflows
5. **First Production Run:** Execute first quarterly screening
6. **Continuous Refinement:** Improve based on results

---

## Document References

**Cross-Document Architecture:**

| Document | Covers | Status |
|----------|--------|--------|
| **0-BASIC-STAGE-WORKFLOW.md** | Pipeline overview, 7 stages | Complete |
| **01-INVESTMENT-REQUIREMENTS.md** | Investment criteria, 40+ metrics | Complete |
| **02-VALUATION-METHODOLOGY-REQUIREMENTS.md** | Stage 6 valuation methods | Complete |
| **03-RESULTS-REPORTING-REQUIREMENTS.md** | Stage 7 reporting specifications | Complete |
| **04-ARCHITECTURE-DESIGN.md** | Technical architecture (THIS DOCUMENT) | v4.0 - Complete |

**All documents updated for:**
- ✅ Complete 7-stage pipeline integration
- ✅ Stage 6 valuation engine details
- ✅ Stage 7 results & reporting
- ✅ Unified metric + hard filter approach
- ✅ GitHub Actions workflows
- ✅ MySQL database schema
- ✅ Testing strategy
- ✅ Deployment & operations

---

## Document History

| Version | Date | Author | Major Changes |
|---------|------|--------|----------------|
| 1.0 | 2026-03-13 | ThommCroft | Initial generic architecture |
| 2.0 | 2026-03-13 | ThommCroft | Web-based multi-user version |
| 3.0 | 2026-03-13 | ThommCroft | Solo developer edition (MySQL + .NET 8.0 Console) |
| 3.1 | 2026-03-14 | ThommCroft | Merged approach (metrics before filtering) |
| 3.2 | 2026-03-14 | ThommCroft | Market cap pre-filter, FCF requirements |
| 3.3 | 2026-03-15 | ThommCroft | GitHub Actions, stock price updates |
| 4.0 | 2026-03-15 | ThommCroft | Complete 7-stage pipeline with Stage 6 valuation & Stage 7 reporting; comprehensive sections on GitHub Actions, testing, deployment, operations |

---

**End of Architecture Design Document**

---

## Quick Links to Key Sections

- [Valuation Engine (Stage 6)](#valuation-engine-stage-6) - 5 methods, consensus, margin of safety
- [Results & Reporting (Stage 7)](#results--reporting-stage-7) - Database storage, CSV export, GitHub summary
- [GitHub Actions Automation](#github-actions-automation) - Quarterly + weekly workflows
- [Testing Strategy](#testing-strategy) - Unit, integration, end-to-end tests
- [Deployment & Operations](#deployment--operations) - Local setup, GitHub Actions, monitoring
- [Success Criteria](#success-criteria) - How to verify everything works
