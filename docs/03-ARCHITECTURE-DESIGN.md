# Architecture Design Document - Stock Market Screener

**Version:** 3.3 
**Date:** 2026-03-15  
**Author:** ThommCroft
**Stack:** C# .NET 8.0, MySQL 8.0, GitHub Actions (Quarterly + Manual), Console Application  
**Deployment:** Local MySQL + GitHub Actions automation  

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [System Architecture Overview](#system-architecture-overview)
3. [Technology Stack](#technology-stack)
4. [Core Components](#core-components)
5. [Data Architecture (MySQL)](#data-architecture-mysql)
6. [Industry Configuration](#industry-configuration)
7. [Screening Workflow](#screening-workflow)
8. [Unified Metric Calculation & Hard Filter Evaluation](#unified-metric-calculation--hard-filter-evaluation)
9. [Data Reconciliation Layer](#data-reconciliation-layer)
10. [Results & Notifications](#results--notifications)
11. [GitHub Actions Automation](#github-actions-automation)
12. [Stock Price Update Job](#stock-price-update-job)
13. [Testing Strategy](#testing-strategy)
14. [Deployment & Operations](#deployment--operations)

---

## Executive Summary

The StockMarketScreener is a **console-based financial analysis engine** that:

- **Screens 6 default industries** (configurable to any industry)
- **Evaluates all companies** in selected industries quarterly/annually
- **Calculates ALL 40+ metrics** for every company BEFORE any filtering
- **Applies comprehensive hard filter evaluation** using complete metric data
- **Only stores passing companies** - complete financial metrics dataset
- **Automatically removes companies** that fall below thresholds
- **Sends comprehensive email reports** with detailed scoring
- **Updates stock prices weekly** (separate background job, no email)
- **Runs automatically via GitHub Actions** (quarterly + manual on-demand)
- **Stores data in local MySQL** for easy access and analysis

**Key Design Principles:**
- ✅ Solo developer friendly
- ✅ Unified metric calculation + hard filter evaluation (merged approach)
- ✅ No early rejection based on incomplete analysis
- ✅ Complete financial picture before any decision
- ✅ Only stores passing companies (no clutter)
- ✅ Email-first notifications
- ✅ Minimal operational overhead
- ✅ Data reconciliation from multiple sources
- ✅ Comprehensive audit trail

---

## System Architecture Overview

### High-Level Data Flow (Merged Approach)

```
┌─────────────────────────────────────────────────────────────┐
│  GitHub Actions Trigger (Quarterly/Manual)                  │
│  OR Local Console Run                                       │
└────────────┬──────────────────────────────────────────────┘
             │
┌────────────▼────────────────────────────────────────────────┐
│  StockMarketScreener Console Application                    │
├─────────────────────────────────────────────────────────────┤
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Step 0: Market Cap Pre-Filter                        │  │
│  │ ├─ Filter: Market Cap > $300M                        │  │
│  │ └─ Output: Liquid, institutional-quality companies   │  │
│  └──────────────────────┬───────────────────────────────┘  │
│                         │                                   │
│  ┌──────────────────────▼──────────────────────────────┐  │
│  │ Step 1: Data Ingestion & Reconciliation              │  │
│  │ ├─ Fetch from SEC EDGAR (10-K/10-Q)                │  │
│  │ ├─ Fetch from Yahoo Finance API                    │  │
│  │ ├─ Fetch from IEX Cloud / Alpha Vantage           │  │
│  │ └─ Reconcile & aggregate data from all sources     │  │
│  └──────────────────────┬───────────────────────────────┘  │
│                         │                                   │
│  ┌──────────────────────▼──────────────────────────────┐  │
│  │ Step 2: ALL 40+ METRICS CALCULATION                │  │
│  │ ├─ Calculate ALL metrics for EVERY company         │  │
│  │ ├─ 10-year analysis (prioritize, min 5 years)     │  │
│  │ └─ Year-by-year metrics & trend analysis           │  │
│  │                                                    │  │
│  │ ⚠️ NO FILTERING OR REJECTION AT THIS STAGE          │  │
│  │ ✓ All companies continue to next step              │  │
│  └──────────────────────┬───────────────────────────────┘  │
│                         │                                   │
│  ┌──────────────────────▼──────────────────────────────┐  │
│  │ Step 3: UNIFIED Hard Filter + Quality Evaluation    │  │
│  │ ├─ Evaluate hard filters using COMPLETE metrics    │  │
│  │ ├─ Assess overall financial strength               │  │
│  │ ├─ Companies below threshold → Skip                │  │
│  │ └─ Companies passing → Continue to scoring          │  │
│  └──────────────────────┬───────────────────────────────┘  │
│                         │                                   │
│  ┌──────────────────────▼──────────────────────────────┐  │
│  │ Step 4: Quality Assessment Scoring (0-100)         │  │
│  │ ├─ Score: Return on Capital (30 points)            │  │
│  │ ├─ Score: Profitability (30 points)                │  │
│  │ ├─ Score: Cash Flow (20 points)                    │  │
│  │ └─ Score: Business Quality (10 points)             │  │
│  └──────────────────────┬───────────────────────────────┘  │
│                         │                                   │
│  ┌──────────────────────▼──────────────────────────────┐  │
│  │ Step 5: Valuation & Management Ranking             │  │
│  │ ├─ Score: Valuation (max 35 points)                │  │
│  │ ├─ Score: Management Quality (max 25 points)       │  │
│  │ └─ Composite: (Quality×40% + Val×35% + Mgmt×25%)  │  │
│  └──────────────────────┬───────────────────────────────┘  │
│                         │                                   │
│  ┌──────────────────────▼──────────────────────────────┐  │
│  │ Step 6: Results Processing                          │  │
│  │ ├─ Only store companies that PASS (score >= 75)   │  │
│  │ ├─ Compare with previously stored companies       │  │
│  │ ├─ Remove companies that now FAIL                 │  │
│  │ └─ Prepare comprehensive results table             │  │
│  └──────────────────────┬──��────────────────────────────┘  │
│                         │                                   │
│  ┌──────────────────────▼──────────────────────────────┐  │
│  │ Step 7: Notifications & Output                      │  │
│  │ ├─ Send detailed email with HTML table             │  │
│  │ ├─ Console output with summary                     │  │
│  │ └─ CSV export for manual review                    │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
             │
    ┌────────┴────────┬──────────────┐
    │                 │              │
    ▼                 ▼              ▼
  MySQL           Email          Console
  Database        Notification     Output
```

### Component Architecture

```
StockMarketScreener.Console
├─ Program.cs (Entry Point, DI Configuration)
├─ Configuration
│  ├─ IndustryConfigService (6 default + custom)
│  ├─ ScreeningOptionsConfig (thresholds, settings)
│  └─ DataSourceConfig (API endpoints, keys)
│
├─ Data Layer
│  ├─ ScreenerDbContext (EF Core + MySQL)
│  ├─ ICompanyRepository
│  ├─ IFinancialMetricsRepository
│  ├─ IQualifiedCompanyRepository
│  └─ IRemovedCompanyRepository
│
├─ External Data Services
│  ├─ SecEdgarService (10-K/10-Q fetching)
│  ├─ YahooFinanceService (Stock prices, dividends)
│  ├─ IexCloudService (Alternative market data)
│  ├─ AlphaVantageService (Fallback data source)
│  └─ DataReconciliationService (Aggregate & validate)
│
├─ Metric Calculation (MERGED WITH SCREENING)
│  ├─ MetricCalculationService (40+ metrics for ALL companies)
│  ├─ TrendAnalysisService (10-year trends)
│  ├─ StabilityCalculationService (CV, volatility)
│  └─ ComprehensiveMetricsAnalyzer (All metrics upfront)
│
├─ Screening Engine (USES CALCULATED METRICS)
│  ├─ HardFilterEvaluationService (Unified filter assessment)
│  ├─ Stage2ScoringService (Quality assessment)
│  ├─ Stage3RankingService (Valuation & Management)
│  └─ ScreeningOrchestrator (Unified orchestration)
│
├─ Business Logic
│  ├─ CompanyStatusManager (Pass/fail tracking)
│  ├─ QualifiedCompanyService (Store passing companies)
│  ├─ RemovedCompanyService (Track removals)
│  └─ DataValidationService (Ensure data quality)
│
├─ Results & Reporting
│  ├─ ScreeningResultsFormatter (HTML table)
│  ├─ EmailNotificationService (Send results)
│  ├─ ScreeningResultsExporter (CSV export)
│  └─ ConsoleReporter (Console output)
│
└─ Background Jobs
   ├─ StockPriceUpdateJob (Weekly, no email)
   └─ ScreeningJob (Quarterly/Manual)
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
| **Email** | MailKit | SMTP integration, HTML support |
| **Configuration** | appsettings.json | Secrets, environment-specific config |
| **Testing** | NUnit + Moq | Industry standard for .NET |

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

Quick Start (MySQL via Docker):
  docker run --name mysql -e MYSQL_ROOT_PASSWORD=root -d -p 3306:3306 mysql:8.0
```

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

```json
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

```csharp
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

---

## Data Architecture (MySQL)

### Database Design

**Key Tables:**

1. **Companies** - Master list of all companies
2. **QualifiedCompanies** - Companies currently passing (the watchlist)
3. **FinancialMetrics** - Year-by-year metrics for qualified companies
4. **ScreeningRuns** - Audit trail of each screening execution
5. **RemovedCompanies** - Companies that no longer qualify
6. **StockPriceHistory** - Weekly stock price snapshots

### Table Descriptions

**QualifiedCompanies** (Main table)
```
- Ticker, CompanyName, Sector, Industry
- CompositeScore (0-100)
- QualityScore, ValuationScore, ManagementScore
- Recommendation (BUY, HOLD, NEUTRAL)
- StockPrice, MarketCap, PERatio, DividendYield
- YearsOfDataAnalyzed (5-10)
- DataSourcesUsed (SEC EDGAR, Yahoo Finance, etc.)
- FirstQualifiedDate, LastQualifiedDate
- IsActive (1 = currently qualified, 0 = removed)
```

**FinancialMetrics** (Historical data)
```
- QualifiedCompanyId (foreign key)
- FiscalYear (2016-2025 for 10-year analysis)
- All 40+ metrics as decimal columns
- Enables trend analysis and historical review
```

**ScreeningRuns** (Audit trail)
```
- RunDate, RunType (Scheduled/Manual)
- Status (Success/Failed/Partial)
- CompaniesProcessed, CompaniesQualified
- NewlyQualified, MaintainedQualified, Removed
- ExecutionTimeSeconds
- IndustriesScreened
```

**RemovedCompanies** (Track removal history)
```
- Ticker, CompanyName, Sector, Industry
- ReasonRemoved (which filters it failed)
- LastCompositeScore
- DateRemoved
- ScreeningRunIdRemoved
```

**StockPriceHistory** (Weekly updates)
```
- QualifiedCompanyId
- StockPrice, MarketCap, PERatio, DividendYield
- UpdateDate, Source (Yahoo, IEX, etc.)
```

### Why MySQL for This Project

✅ **Familiar to you** - no learning curve
✅ **Easy local setup** - simple, no cloud required
✅ **Free and open source** - no licensing cost
✅ **Good EF Core support** - Pomelo provider excellent
✅ **Sufficient performance** - easily handles thousands of companies
✅ **Easy to query** - can write custom SQL if needed

---

## Industry Configuration

### 6 Default Industries

```csharp
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
```json
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

## Screening Workflow

### Quarterly Run Flow (MERGED APPROACH)

```
📅 Quarterly Execution (e.g., March 15, 2026)
│
├─ GitHub Actions triggered at 2 PM UTC
├─ OR Manual trigger via workflow_dispatch
│
├─ Load Configuration
│  └─ Industries: [Consumer Defensive, Technology, Healthcare, ...]
│
├─ Fetch Companies (by industry from SEC)
│  └─ 500+ companies across selected industries
│
├─ Stage 0: Market Cap Pre-Filter
│  ├─ Filter: Market Cap > $300M
│  └─ ~400-450 companies pass, ~50-100 rejected (too small)
│
├─ For Each Qualifying Company (400+ iterations)
│  ├─ Fetch Financial Data
│  │  ├─ SEC EDGAR (10-K/10-Q) - authoritative
│  │  ├─ Yahoo Finance (prices, dividends) - primary
│  │  ├─ IEX Cloud (fallback if Yahoo fails)
│  │  └─ Alpha Vantage (final fallback)
│  │
│  ├─ Validate Data Quality
│  │  ├─ Check: 5+ years available (10 preferred)
│  │  ├─ Check: Data integrity (balance sheet balances)
│  │  └─ Flag: 10-year data vs 5-9 year (prioritize 10)
│  │
│  ├─ MERGED STEPS 2-3: Calculate ALL 40+ Metrics
│  │  ├─ Year-by-year calculation for each metric
│  │  ├─ 10-year averages and trends
│  │  ├─ Stability metrics (CV, volatility)
│  │  ├─ Growth rates (CAGR)
│  │  └─ ⚠️ NO FILTERING HERE - All metrics calculated
│  │
│  ├─ UNIFIED Hard Filter Evaluation
│  │  ├─ Evaluate hard filters using COMPLETE 40+ metrics
│  │  ├─ Assess overall financial strength holistically
│  │  ├─ Consider context from all metrics
│  │  └─ IF fails assessment → SKIP; IF passes → Continue
│  │
│  ├─ Stage 2: Quality Assessment (Scoring)
│  │  ├─ Score: Return on Capital (30 points max)
│  │  ├─ Score: Profitability (30 points max)
│  │  ├─ Score: Cash Flow (20 points max)
│  │  ├─ Score: Business Quality (10 points max)
│  │  └─ Total: Quality Score (0-100)
│  │
│  ├─ Stage 3: Valuation & Management Ranking
│  │  ├─ Score: Valuation (P/E, P/B, etc.) - max 35 pts
│  │  ├─ Score: Management Quality - max 25 pts
│  │  ├─ Composite: (Quality×40% + Valuation×35% + Mgmt×25%)
│  │  └─ IF Composite >= 75 → STORE; ELSE SKIP
│  │
│  └─ Results
│     ├─ PASS & STORE (composite >= 75)
│     │  └─ Save all metrics to MySQL
│     └─ FAIL & SKIP (composite < 75)
│        └─ No storage, not listed
│
├─ Compare with Previous Results
│  ├─ Newly Qualified (passed this run, didn't before)
│  ├─ Maintained (passed both runs)
│  └─ Removed (passed before, failed this run - DELETE from storage)
│
├─ Update Database
│  ├─ INSERT newly qualified companies
│  ├─ UPDATE scores for maintained companies
│  ├─ DELETE removed companies (mark inactive)
│  └─ UPDATE ScreeningRuns with summary
│
├─ Send Email Notification
│  ├─ Subject: Screening Results - {Date}
│  ├─ Summary: X qualified, Y new, Z maintained, W removed
│  ├─ Detailed results table with all metrics
│  ├─ Business quality assessment
│  ├─ Valuation analysis
│  └─ Removed companies list
│
└─ ✅ Complete
   └─ Logs available, CSV exported, database updated
```

### Manual Execution

```
User runs: dotnet run

Program.cs → ScreeningOrchestrator.RunScreening()
│
└─ Same flow as scheduled run (merged approach)
   └─ Results: Email + Console Output + Database update
```

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
```csharp
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

## Results & Notifications

### Email Report Format

**Email Components:**

1. **Header Summary**
   - Run date and time
   - Industries screened
   - Total companies processed

2. **Quick Stats**
   - X companies qualified
   - Y newly qualified (green highlight)
   - Z maintained (yellow highlight)
   - W removed (red highlight)

3. **Detailed Results Table**
   - Ticker, Company, Industry
   - Composite Score (0-100)
   - Quality Score, Valuation Score, Management Score
   - Current Stock Price
   - P/E Ratio
   - Recommendation (BUY/HOLD/NEUTRAL)
   - Status (NEW/MAINTAINED)

4. **Metrics Breakdown Table**
   - ROE (10Y Avg) with color coding
   - Net Margin with color coding
   - Operating Margin with color coding
   - Debt-to-Equity with color coding
   - Free Cash Flow with color coding
   - P/E Ratio with color coding
   - All color-coded: Green (excellent) → Yellow (acceptable) → Red (poor)

5. **Business Quality Section**
   - Economic Moat (yes/no, type)
   - Moat Assessment (brand, switching costs, network, cost advantage)
   - Management Quality (insider ownership %, goodwill assessment)
   - Margin Stability (CV metric)
   - Trend Analysis (improving/stable/declining)

6. **Valuation & Margin of Safety**
   - Stock Price and Market Cap
   - Valuation Ratios (P/E, P/B, P/S, PEG)
   - Dividend Yield (if applicable)
   - Earnings Yield vs Treasury
   - Estimated Margin of Safety

7. **Removed Companies Section**
   - Ticker, Company, Industry
   - Reason Removed (which filter failed)
   - Last Composite Score
   - When Removed
   - Encouragement: "Monitor for requalification"

8. **Footer**
   - Timestamp
   - Data sources used
   - Note about manual review
   - Link to repository (for future UI)

### Console Output Format

```
═══════════════════════════════════════════════════════════
  STOCK MARKET SCREENING RUN
═══════════════════════════════════════════════════════════

📊 Configuration
  Industries: Consumer Defensive, Technology, Healthcare, ...
  Minimum Data: 5 years (prefer 10)
  Passing Threshold: 75

📈 Processing...
  Company 1/500: MSFT
    ✅ Market Cap Check: PASS ($3.2T)
    ✅ Data Validation: PASS (10 years)
    ✅ Metrics Calculated: ALL 40+ metrics complete
    ✅ Hard Filter Evaluation: PASS
    ✅ Quality Score: 92/100
    ✅ Composite: 87.5/100 (BUY)
    ✅ QUALIFIED & STORED
  
  Company 2/500: AAPL
    ✅ Market Cap Check: PASS ($3.0T)
    ✅ Data Validation: PASS (10 years)
    ✅ Metrics Calculated: ALL 40+ metrics complete
    ✅ Hard Filter Evaluation: PASS
    ✅ Quality Score: 89/100
    ✅ Composite: 84.2/100 (BUY)
    ✅ QUALIFIED & STORED
  
  Company 3/500: XYZ
    ✅ Market Cap Check: PASS ($400M)
    ✅ Data Validation: PASS (7 years)
    ✅ Metrics Calculated: ALL 40+ metrics complete
    ❌ Hard Filter Evaluation: FAILED
    ❌ Complete assessment: Financial weakness across metrics
    ❌ SKIPPED (not stored)
  
  ...

═══════════════════════════════════════════════════════════
  SCREENING COMPLETED
═══════════════════════════════════════════════════════════

📊 Summary
  Companies Processed: 500
  Market Cap Qualified: 480
  Data Valid: 450
  Hard Filters Passed: 75
  Composite Score >= 75: 45
  
  Newly Qualified: 8
  Maintained: 37
  Removed: 2
  Execution Time: 45m 30s

📧 Email Sent: your-email@gmail.com
💾 Database Updated: 45 companies
📁 CSV Exported: output/results-2026-03-15.csv

✅ SCREENING SUCCESSFUL
```

### CSV Export Format

```csv
Ticker,Company,Industry,CompositeScore,QualityScore,ValuationScore,ManagementScore,Recommendation,StockPrice,PERatio,Status,YearsOfData,DataSources,FirstQualifiedDate,LastQualifiedDate

MSFT,Microsoft Corporation,Technology,87.5,92,82,88,BUY,350.25,28.3,NEW,10,SEC EDGAR; Yahoo Finance,2026-03-15,2026-03-15
AAPL,Apple Inc.,Technology,84.2,89,78,82,BUY,180.50,25.8,MAINTAINED,10,SEC EDGAR; Yahoo Finance,2025-09-20,2026-03-15
JNJ,Johnson & Johnson,Healthcare,82.1,85,80,78,HOLD,145.75,18.5,MAINTAINED,10,SEC EDGAR; Yahoo Finance,2025-12-10,2026-03-15
...
```

---

## GitHub Actions Automation

### Quarterly Screening Workflow

**File:** `.github/workflows/quarterly-screening.yml`

**Trigger:**
- Schedule: 15th of March, June, September, December at 2 PM UTC
- Manual: workflow_dispatch (anytime)

**Steps:**
1. Checkout code
2. Setup .NET 8.0
3. Restore NuGet packages
4. Build application (Release configuration)
5. Setup MySQL database (Docker service)
6. Run EF Core migrations
7. Execute screening application (merged approach)
8. Upload results as GitHub artifact (90-day retention)
9. Commit results to repository (optional)
10. Send email notification (via MailKit)

**Environment Variables:**
```
ConnectionStrings__DefaultConnection: MySQL connection
Email__SmtpServer: smtp.gmail.com
Email__SenderEmail: your-email@gmail.com
Email__SenderPassword: app-specific-password
```

**GitHub Secrets Required:**
```
SMTP_SERVER = smtp.gmail.com
SMTP_PORT = 587
SENDER_EMAIL = your-email@gmail.com
SENDER_PASSWORD = your-gmail-app-password
RECIPIENT_EMAIL = your-email@gmail.com
```

### Weekly Stock Price Update Workflow

**File:** `.github/workflows/weekly-stock-update.yml`

**Trigger:**
- Schedule: Every Monday at 8 PM UTC (after market close)
- Manual: workflow_dispatch

**Steps:**
1. Checkout code
2. Setup .NET 8.0
3. Restore packages
4. Build application
5. Setup MySQL
6. Run stock price update job
7. No email sent (silent update)
8. Log results

**Purpose:**
- Update current stock prices for all qualified companies
- Update P/E ratios, market caps, dividend yields
- Enable live valuation tracking
- No email spam - just data update

---

## Stock Price Update Job

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
- Lightweight, quick execution

---

## Testing Strategy

### Unit Tests

**Testing Framework:** NUnit + Moq

**Test Coverage:**

1. **Metric Calculations**
   - Test each metric formula with known data
   - Verify 10-year averaging
   - Test stability (CV) calculations
   - Test CAGR calculations
   - **NEW:** Test that all 40+ metrics calculate for every company

2. **Hard Filter Evaluation (MERGED)**
   - Test hard filter assessment using complete metrics
   - Test contextual evaluation (e.g., D/E in context of trends)
   - Test holistic financial strength assessment
   - Verify no premature rejection

3. **Screening Logic**
   - Test quality score with various metrics
   - Test composite score calculation
   - Verify threshold logic (>= 75)

4. **Data Reconciliation**
   - Test data merging from multiple sources
   - Test fallback logic (primary → secondary → tertiary)
   - Test validation logic
   - Mock API responses

5. **Repository Operations**
   - Test save/update/delete operations
   - Verify database constraints
   - Test filtering and querying

### Integration Tests

**Testing Framework:** NUnit + Testcontainers (MySQL in Docker)

**Test Data:**
- 10 well-known companies with real historical data
- Mix of pass/fail scenarios
- Edge cases (low margins, high debt, etc.)

**Test Scenarios:**
1. **Complete flow with merged approach**
   - Verify all metrics calculated
   - Verify hard filter evaluation uses complete metrics
   - Verify correct companies pass/fail

2. **Hard filter evaluation accuracy**
   - Company with good overall metrics but one weak area
   - Verify holistic assessment works
   - Verify context prevents false rejection

3. **Storage and retrieval**
   - Verify storage of passing companies
   - Verify removal of previously qualified companies
   - Verify all metrics stored

4. **Email generation**
   - Test email generation (no actual send)
   - Verify all metrics included
   - Verify formatting correct

### Mock Tests

**Mocking Strategy:**

```csharp
// Mock SecEdgarService
var mockEdgar = new Mock<ISecEdgarService>();
mockEdgar.Setup(s => s.FetchLatestAnnualReport("MSFT"))
    .ReturnsAsync(testFinancialData);

// Mock YahooFinanceService
var mockYahoo = new Mock<IYahooFinanceService>();
mockYahoo.Setup(s => s.FetchCurrentData("MSFT"))
    .ReturnsAsync(testStockPrice);

// Mock EmailService
var mockEmail = new Mock<IEmailNotificationService>();
mockEmail.Setup(s => s.SendScreeningResultsAsync(It.IsAny<string>(), It.IsAny<List<ScreeningResult>>()))
    .Returns(Task.CompletedTask);

// Mock MetricCalculationService
var mockMetrics = new Mock<IMetricCalculationService>();
mockMetrics.Setup(s => s.CalculateAllMetricsAsync(It.IsAny<Company>()))
    .ReturnsAsync(completeMetricsObject);
```

### Test Database

**Setup:**
```csharp
// Use in-memory SQLite for fast unit tests
// Use Testcontainers for integration tests with real MySQL

[Collection("Integration Tests")]
public class ScreeningIntegrationTests : IAsyncLifetime
{
    private readonly TestcontainersContainer _container;
    
    public async Task InitializeAsync()
    {
        _container = new MySqlTestcontainersBuilder()
            .WithImage("mysql:8.0")
            .WithDatabase("stockscreener")
            .WithUsername("root")
            .WithPassword("root")
            .Build();
            
        await _container.StartAsync();
    }
    
    public async Task DisposeAsync()
    {
        await _container.StopAsync();
    }
}
```

### Test Data (10 Companies)

```
Well-Known Test Companies:
1. MSFT (Microsoft) - Should PASS (strong across all metrics)
2. AAPL (Apple) - Should PASS (strong overall)
3. JNJ (Johnson & Johnson) - Should PASS (quality fundamentals)
4. KO (Coca-Cola) - Should PASS (stable, strong moat)
5. JPM (JPMorgan Chase) - Should PASS (financial strength)
6. TSLA (Tesla) - Should FAIL (high valuation despite metrics)
7. AMD (Advanced Micro Devices) - Should FAIL (higher debt)
8. SNAP (Snapchat) - Should FAIL (low ROE overall)
9. UBER (Uber) - Should FAIL (negative FCF)
10. GOOG (Google) - Should PASS (strong fundamentals)

Test Data Characteristics:
- 10 years of historical data (2016-2025)
- Mix of pass/fail scenarios
- Real financial metrics
- Used for all test cases
- Verify merged approach works correctly
```

---

## Deployment & Operations

### Local Development Setup

```bash
# 1. Clone repository
git clone https://github.com/ThommCroft/StockMarketScreener.git
cd StockMarketScreener

# 2. Start MySQL (Docker)
docker run --name mysql -e MYSQL_ROOT_PASSWORD=root -d -p 3306:3306 mysql:8.0

# 3. Configure database connection
# Edit appsettings.json:
#   ConnectionStrings.DefaultConnection = "Server=localhost;Uid=root;Pwd=root;Database=stockscreener"

# 4. Restore dependencies
dotnet restore

# 5. Build project
dotnet build

# 6. Create/update database schema
dotnet ef database update

# 7. Run screening locally (with merged approach)
dotnet run --project src/StockMarketScreener.Console/StockMarketScreener.Console.csproj

# 8. Check results
ls -la output/  # Results CSV and HTML
```

### GitHub Actions Deployment

```
No special deployment needed!

1. Code pushed to GitHub
2. GitHub Actions runs on schedule (or manual trigger)
3. Everything runs in GitHub's cloud
4. Results stored as artifacts
5. Email sent upon completion
6. MySQL database updated
```

### Backup Strategy

**Local Database Backup:**
```bash
# Backup MySQL
mysqldump -u root -p stockscreener > backup_$(date +%Y-%m-%d).sql

# Restore from backup
mysql -u root -p stockscreener < backup_2026-03-15.sql
```

**Automated Backup (optional):**
- Add Docker volumes for persistence
- Use cloud-hosted MySQL (Azure Database for MySQL, AWS RDS)
- Automated backups with retention

---

## Success Criteria

This architecture is successful when:

✅ **Unified Approach**
- All 40+ metrics calculated before any filtering
- No premature rejection based on incomplete data
- Hard filters evaluated with complete financial picture
- Holistic assessment of company strength

✅ **Automation**
- Screening runs completely hands-off via GitHub Actions
- No manual intervention needed
- Logs show exactly what happened

✅ **Data Quality**
- All 40+ metrics calculate correctly for every company
- Data reconciliation handles API failures
- Results match financial reality

✅ **Company Filtering**
- Only passing companies stored (no clutter)
- Complete metric context available for understanding decisions
- Proper removal when scores decline

✅ **Results & Notifications**
- Detailed email with complete metrics breakdown
- HTML tables with color-coded scores
- CSV export for spreadsheet analysis
- Clear identification of new/maintained/removed companies

✅ **Solo Developer Friendly**
- Code is maintainable and well-documented
- Can run locally for testing
- Easy to modify screening criteria
- Minimal operational overhead

✅ **Performance**
- First run: 45-60 minutes (full data fetch + ALL metric calculation)
  - Data fetching: 30-40 minutes
  - ALL metric calculations: 5-10 minutes
  - Hard filter evaluation: 2-3 minutes
  - Screening and ranking: 2-3 minutes
  - Results processing and email: 2 minutes
- Subsequent runs: 15-20 minutes (with local caching)
- With parallel processing: Could reduce to 20-30 minutes
- Data reconciliation handles API rate limits gracefully
- Stock price updates complete in < 5 minutes

✅ **Reliability**
- Robust error handling (continue if 1 company fails)
- Multiple data sources prevent single-point-of-failure
- Audit trail of all screening runs
- Database integrity maintained

---

## Summary

The StockMarketScreener architecture employs a **unified merged approach** for metric calculation and hard filter evaluation. This design:

1. **Calculates all 40+ metrics** for every company upfront
2. **Evaluates hard filters** using the complete metric dataset
3. **Prevents false negatives** from incomplete data analysis
4. **Provides holistic assessment** of financial strength
5. **Maintains audit trail** of all decisions

Built specifically for a solo developer with 6 years of C# experience:
- **Simplicity** - No complex infrastructure
- **Automation** - GitHub Actions handles scheduling
- **Quality** - Comprehensive metric analysis
- **Efficiency** - Only stores passing companies
- **Maintainability** - Clean code, well-documented
- **Visibility** - Detailed reports and logs

This is a complete, production-ready architecture that can be built incrementally, tested thoroughly, and operated with minimal effort.

---

## Cross-Document References

This document is part of a 4-document architecture specification:

1. **01-INVESTMENT-REQUIREMENTS.md** - Warren Buffett principles and financial criteria (UPDATED)
2. **02-ARCHITECTURE-DESIGN.md** - System architecture and technical design (THIS DOCUMENT - UPDATED)
3. **03-DATA-FLOW-SCREENING-LOGIC.md** - Detailed screening workflow and logic (NEXT)
4. **04-PROJECT-ROADMAP.md** - Implementation phases and timeline (NEXT)

All documents are aligned and internally consistent as of 2026-03-14.

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-03-13 | ThommCroft | Initial generic architecture |
| 2.0 | 2026-03-13 | ThommCroft | Web-based multi-user version |
| 3.0 | 2026-03-13 | ThommCroft | Solo developer edition (MySQL + .NET 8.0 Console), GitHub Actions automation, data reconciliation, email notifications, stock price updates |
| 3.1 | 2026-03-14 | ThommCroft | Minor clarifications on Stage 1 hard filters, Market Cap pre-filter, FCF requirement confirmation |
| 3.2 | 2026-03-14 | ThommCroft | **CURRENT** - MERGED APPROACH: All 40+ metrics calculated BEFORE hard filter evaluation; unified comprehensive analysis replacing cascading filters; updated data flow diagram; updated component architecture; updated screening workflow; added "Unified Metric Calculation & Hard Filter Evaluation" section |

