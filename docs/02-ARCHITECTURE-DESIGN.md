# Architecture Design Document - Stock Market Screener

**Version:** 3.1 (Solo Developer - .NET 8.0 + MySQL)  
**Date:** 2026-03-14  
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
8. [Data Reconciliation Layer](#data-reconciliation-layer)
9. [Results & Notifications](#results--notifications)
10. [GitHub Actions Automation](#github-actions-automation)
11. [Stock Price Update Job](#stock-price-update-job)
12. [Testing Strategy](#testing-strategy)
13. [Deployment & Operations](#deployment--operations)

---

## Executive Summary

The StockMarketScreener is a **console-based financial analysis engine** that:

- **Screens 6 default industries** (configurable to any industry)
- **Evaluates all companies** in selected industries quarterly/annually
- **Applies strict 3-stage filtering** to identify high-quality, undervalued stocks
- **Only stores passing companies** - complete financial metrics dataset
- **Automatically removes companies** that fall below thresholds
- **Sends comprehensive email reports** with detailed scoring
- **Updates stock prices weekly** (separate background job, no email)
- **Runs automatically via GitHub Actions** (quarterly + manual on-demand)
- **Stores data in local MySQL** for easy access and analysis

**Key Design Principles:**
- ✅ Solo developer friendly
- ✅ Batch processing (quarterly/annual)
- ✅ Only stores passing companies (no clutter)
- ✅ Email-first notifications
- ✅ Minimal operational overhead
- ✅ Data reconciliation from multiple sources
- ✅ Comprehensive audit trail

---

## System Architecture Overview

### High-Level Data Flow

```
┌─────────────────────────────────────────────────────────────┐
│  GitHub Actions Trigger (Quarterly/Manual)                  │
│  OR Local Console Run                                       │
└────────────┬──────────────────────────────────────���─────────┘
             │
┌────────────▼────────────────────────────────────────────────┐
│  StockMarketScreener Console Application                    │
├─────────────────────────────────────────────────────────────┤
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Step 1: Data Ingestion & Reconciliation              │  │
│  │ ├─ Fetch from SEC EDGAR (10-K/10-Q)                │  │
│  │ ├─ Fetch from Yahoo Finance API                    │  │
│  │ ├─ Fetch from IEX Cloud / Alpha Vantage           │  │
│  │ └─ Reconcile & aggregate data from all sources     │  │
│  └──────────────────────────────────────────────────────┘  │
│                          │                                   │
│  ┌──────────────────────▼──────────────────────────────┐  │
│  │ Step 2: Metric Calculation Engine                   │  │
│  │ ├─ Calculate 40+ financial metrics                 │  │
│  │ ├─ 10-year analysis (prioritize, min 5 years)    │  │
│  │ └─ Assess business quality                         │  │
│  └──────────────────────┬───────────────────────────────┘  │
│                         │                                   │
│  ┌──────────────────────▼──────────────────────────────┐  │
│  │ Step 3: 3-Stage Screening                           │  │
│  │ ├─ Stage 1: Financial Strength (hard filters)      │  │
│  │ ├─ Stage 2: Quality Assessment (scoring)           │  │
│  │ └─ Stage 3: Valuation & Management                 │  │
│  └──────────────────────┬───────────────────────────────┘  │
│                         │                                   │
│  ┌──────────────────────▼──────────────────────────────┐  │
│  │ Step 4: Results Processing                          │  │
│  │ ├─ Only store companies that PASS (score >= 75)   │  │
│  │ ├─ Compare with previously stored companies       │  │
│  │ ├─ Remove companies that now FAIL                 │  │
│  │ └─ Prepare comprehensive results table             │  │
│  └──────────────────────┬───────────────────────────────┘  │
│                         │                                   │
│  ┌──────────────────────▼──────────────────────────────┐  │
│  │ Step 5: Notifications & Output                      │  │
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
├─ Metric Calculation
│  ├─ MetricCalculationService (40+ metrics)
│  ├─ TrendAnalysisService (10-year trends)
│  └─ StabilityCalculationService (CV, volatility)
│
├─ Screening Engine
│  ├─ Stage1ScreeningService (MUST HAVE filters)
│  ├─ Stage2ScoringService (Quality assessment)
│  ├─ Stage3RankingService (Valuation & Management)
│  └─ ScreeningOrchestrator (3-stage orchestration)
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
| **Testing** | xUnit + Moq | Industry standard for .NET |

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
- Ensures data quality before metrics calculation
- Prevents corrupted data from affecting analysis
- Creates audit trail of data sources used
- Validates that reconciled data is complete and consistent

---

### 3. Metric Calculation Service

**Purpose:** Calculate all 40+ financial metrics from reconciled financial data

**Metric Categories (40+):**

1. **Return on Capital (4 metrics)**
   - ROE: Return on Equity
   - ROA: Return on Assets
   - ROIC: Return on Invested Capital

2. **Financial Strength (6 metrics)**
   - Debt-to-Equity Ratio
   - Current Ratio
   - Interest Coverage Ratio
   - Total Liabilities-to-Equity

3. **Profitability (8 metrics)**
   - Net Profit Margin
   - Operating Margin
   - Gross Profit Margin
   - Depreciation Margin
   - Interest Expense Margin

4. **Cash Flow (6 metrics)**
   - Operating Cash Flow
   - Free Cash Flow
   - FCF-to-Net-Income Ratio
   - CapEx as % Revenue
   - CapEx as % OCF

5. **Growth (7 metrics)**
   - Revenue Growth (YoY, CAGR)
   - Earnings Per Share (EPS)
   - Retained Earnings

6. **Valuation (8 metrics)**
   - P/E Ratio
   - P/B Ratio
   - P/S Ratio
   - PEG Ratio
   - Earnings Yield
   - Dividend Yield
   - Payout Ratio

7. **Management Quality (6 metrics)**
   - Insider Ownership %
   - Goodwill Level
   - Goodwill as % of Assets
   - One-Time Charges Frequency

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

---

### 4. Screening Orchestrator

**Purpose:** Orchestrates the entire screening flow end-to-end

**Workflow:**
1. Load industry configuration
2. Fetch all companies in selected industries
3. For each company:
   - Reconcile financial data from multiple sources
   - Calculate 40+ metrics
   - Run Stage 1 (hard filters)
   - Run Stage 2 (quality scoring)
   - Run Stage 3 (final ranking)
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

### 5. Qualified Company Service

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

### 6. Email Notification Service

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

### 7. Stock Price Update Job

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

### Quarterly Run Flow

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
├─ For Each Company (500+ iterations)
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
│  ├─ Calculate 40+ Metrics
│  │  ├─ Year-by-year calculation
│  │  ├─ 10-year averages and trends
│  │  ├─ Stability metrics (CV, volatility)
│  │  └─ Growth rates (CAGR)
│  │
│  ├─ Stage 1: Financial Strength (Hard Filters)
│  │  ├─ ROE > 15%?
│  │  ├─ Net Margin > 10%?
│  │  ├─ D/E < 0.50?
│  │  ├─ Operating Cash Flow positive (all 10 years)?
│  │  ├─ Free Cash Flow positive (all 10 years)?
│  │  └─ IF ANY FAIL → SKIP company (not stored)
│  │  **Note:** These 5 core decision filters are backed by additional 
│  │  MUST HAVE prerequisites (Market Cap > $300M, 5+ years of data, etc.)
│  │  defined in Investment Requirements document.
│  │
│  ├─ Stage 2: Quality Assessment (Scoring)
│  │  ├─ Score: Return on Capital (30 points)
│  │  ├─ Score: Profitability (30 points)
│  │  ├─ Score: Cash Flow (20 points)
│  │  ├─ Score: Business Quality (10 points)
│  │  └─ Total: Quality Score (0-100)
│  │
│  ├─ Stage 3: Valuation & Ranking
│  │  ├─ Score: Valuation (P/E, P/B, etc.)
│  │  ├─ Score: Management Quality
│  │  ├─ Composite: (Quality×40% + Valuation×35% + Management×25%)
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
└─ Same flow as scheduled run
   └─ Results: Email + Console Output + Database update
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
    ✅ Stage 1 PASSED
    ✅ Stage 2 Quality Score: 92/100
    ✅ Stage 3 Composite: 87.5/100 (BUY)
    ✅ QUALIFIED & STORED
  
  Company 2/500: AAPL
    ✅ Stage 1 PASSED
    ✅ Stage 2 Quality Score: 89/100
    ✅ Stage 3 Composite: 84.2/100 (BUY)
    ✅ QUALIFIED & STORED
  
  Company 3/500: XYZ
    ❌ Stage 1 FAILED: D/E ratio > 0.50
    ❌ SKIPPED (not stored)
  
  ...

═══════════════════════════════════════════════════════════
  SCREENING COMPLETED
═══════════════════════════════════════════════════════════

📊 Summary
  Companies Screened: 500
  Companies Qualified: 45
  Newly Qualified: 8
  Maintained: 37
  Removed: 2
  Execution Time: 4m 23s

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
7. Execute screening application
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

**Why Separate from Screening:**
- Screening runs quarterly (expensive data fetch)
- Stock prices change constantly
- Valuation metrics need updating
- No email spam - just updates data
- Lightweight, quick execution

---

## Testing Strategy

### Unit Tests

**Testing Framework:** xUnit + Moq

**Test Coverage:**

1. **Metric Calculations**
   - Test each metric formula with known data
   - Verify 10-year averaging
   - Test stability (CV) calculations
   - Test CAGR calculations

2. **Screening Logic**
   - Test Stage 1 filters with passing/failing data
   - Test quality score with various metrics
   - Test composite score calculation
   - Verify threshold logic

3. **Data Reconciliation**
   - Test data merging from multiple sources
   - Test fallback logic (primary → secondary → tertiary)
   - Test validation logic
   - Mock API responses

4. **Repository Operations**
   - Test save/update/delete operations
   - Verify database constraints
   - Test filtering and querying

### Integration Tests

**Testing Framework:** xUnit + Testcontainers (MySQL in Docker)

**Test Data:**
- 10 well-known companies with real historical data
- Companies that should PASS screening
- Companies that should FAIL screening
- Edge cases (low margins, high debt, etc.)

**Test Scenarios:**
1. End-to-end screening of test data
2. Verify correct companies pass/fail
3. Verify storage and retrieval
4. Verify removal of previously qualified companies
5. Test email generation (no actual send)

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
1. MSFT (Microsoft) - Should PASS
2. AAPL (Apple) - Should PASS
3. JNJ (Johnson & Johnson) - Should PASS
4. KO (Coca-Cola) - Should PASS
5. JPM (JPMorgan Chase) - Should PASS
6. TSLA (Tesla) - Should FAIL (high valuation)
7. AMD (Advanced Micro Devices) - Should FAIL (high debt)
8. SNAP (Snapchat) - Should FAIL (low ROE)
9. UBER (Uber) - Should FAIL (negative FCF)
10. GOOG (Google) - Should PASS

Test Data Characteristics:
- 10 years of historical data
- Mix of pass/fail scenarios
- Real financial metrics
- Used for all test cases
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

# 7. Run screening locally
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

✅ **Automation**
- Screening runs completely hands-off via GitHub Actions
- No manual intervention needed
- Logs show exactly what happened

✅ **Data Quality**
- All 40+ metrics calculate correctly
- Data reconciliation handles API failures
- Results match financial reality

✅ **Company Filtering**
- Only passing companies stored (no clutter)
- Companies with poor metrics filtered out
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
- First run: 40-50 minutes (full data fetch from APIs with sequential calls)
  - Data fetching from SEC EDGAR, Yahoo Finance, IEX, Alpha Vantage: 30-40 minutes
  - Metric calculations: 2-3 minutes
  - Screening and ranking: 1 minute
  - Results processing and email: 2 minutes
- Subsequent runs: 10-15 minutes (with caching and local data)
- With parallel processing (5-10 concurrent API calls): Could reduce to 10-15 minutes
- Data reconciliation handles API rate limits gracefully
- Stock price updates (weekly job) complete in < 5 minutes
- No timeout errors with proper fallback strategy

✅ **Reliability**
- Robust error handling (continue if 1 company fails)
- Multiple data sources prevent single-point-of-failure
- Audit trail of all screening runs
- Database integrity maintained

---

## Summary

The StockMarketScreener architecture is designed specifically for a solo developer with 6 years of C# experience. It prioritizes:

1. **Simplicity** - No complex infrastructure
2. **Automation** - GitHub Actions handles scheduling
3. **Quality** - Multi-source data reconciliation
4. **Efficiency** - Only stores passing companies
5. **Maintainability** - Clean code, well-documented
6. **Visibility** - Detailed email reports and logs

This is a complete, production-ready architecture that can be built incrementally, tested thoroughly, and operated with minimal effort.

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-03-13 | ThommCroft | Initial generic architecture |
| 2.0 | 2026-03-13 | ThommCroft | Web-based multi-user version |
| 3.0 | 2026-03-13 | ThommCroft | Solo developer edition (MySQL + .NET 8.0 Console), GitHub Actions automation, data reconciliation, email notifications, stock price updates, comprehensive testing strategy |
| 3.1 | 2026-03-14 | ThommCroft | **CURRENT** - Minor clarifications on Stage 1 hard filters, Market Cap pre-filter integration, FCF requirement confirmation |
