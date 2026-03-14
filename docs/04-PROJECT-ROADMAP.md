# Project Roadmap - Stock Market Screener

**Version:** 1.1
**Date:** 2026-03-14  
**Author:** ThommCroft  
**Status:** Planning Phase  
**Timeline:** No fixed deadline - Quality over speed
**Last Updated:** 2026-03-14 (Cross-reference verification with other documents complete)

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Vision & Success Criteria](#vision--success-criteria)
3. [Project Phases](#project-phases)
4. [Phase 1: Foundation & Setup](#phase-1-foundation--setup)
5. [Phase 2: Core Screening Engine](#phase-2-core-screening-engine)
6. [Phase 3: Data Integration & Reconciliation](#phase-3-data-integration--reconciliation)
7. [Phase 4: Metric Calculation & Screening Logic](#phase-4-metric-calculation--screening-logic)
8. [Phase 5: Results & Notifications](#phase-5-results--notifications)
9. [Phase 6: GitHub Actions Automation](#phase-6-github-actions-automation)
10. [Phase 7: Testing & Quality Assurance](#phase-7-testing--quality-assurance)
11. [Phase 8: UI Development](#phase-8-ui-development)
12. [Post-Launch Planning](#post-launch-planning)
13. [Dependencies & Prerequisites](#dependencies--prerequisites)
14. [Risk Management](#risk-management)
15. [Success Metrics & Validation](#success-metrics--validation)

---

## Executive Summary

The Stock Market Screener project will be built incrementally in **8 focused phases**, culminating in a fully functional, production-ready application with comprehensive testing. The project prioritizes **quality and correctness over speed**, with no fixed deadline.

**Key Principles:**
- ✅ Build incrementally with validation at each phase
- ✅ Comprehensive testing after each phase
- ✅ Free, public, legal data sources only
- ✅ GitHub Actions automation included in core build
- ✅ UI and future features planned post-launch
- ✅ Solo developer friendly approach

---

## Vision & Success Criteria

### Project Vision

Create a **professional-grade, automated stock screening system** that:
1. Identifies high-quality, undervalued companies quarterly
2. Applies strict financial filtering and scoring
3. Delivers detailed results via email and dashboard
4. Maintains audit trail of all operations
5. Runs completely hands-off via GitHub Actions

### Success Definition

**The project is successful when:**

✅ **All companies screened correctly**
- Financial strength filters identify the right candidates
- Quality scores reflect business fundamentals accurately
- Valuation assessment matches market reality
- Results align with professional stock screening principles

✅ **Data integration working end-to-end**
- SEC EDGAR data fetching reliable
- Yahoo Finance prices accurate and current
- Fallback sources (IEX Cloud, Alpha Vantage) functioning
- Data reconciliation handles conflicts properly

✅ **Screening logic producing expected results**
- Stage 1 filters eliminating poor companies
- Stage 2 quality scoring differentiating between good/excellent
- Stage 3 valuation identifying undervalued opportunities
- Composite scores >= 75 correlate with quality companies

✅ **Automation running consistently**
- GitHub Actions triggers on schedule
- Quarterly screenings complete without manual intervention
- Email notifications sent with detailed results
- Database updates accurate and auditable

✅ **No critical bugs or data corruption**
- Edge cases handled gracefully
- Error handling prevents system crashes
- Data validation catches corrupted inputs
- Logging provides full audit trail

---

## Project Phases

### Overview

```
Phase 1: Foundation & Setup (1-2 weeks)
    ↓
Phase 2: Core Screening Engine (1 week)
    ↓
Phase 3: Data Integration (2-3 weeks)
    ↓
Phase 4: Metric Calculation (2-3 weeks)
    ↓
Phase 5: Results & Notifications (1-2 weeks)
    ↓
Phase 6: GitHub Actions Automation (1 week)
    ↓
Phase 7: Testing & Quality Assurance (2-3 weeks)
    ↓
Phase 8: UI Development (3-4 weeks)
    ↓
Launch & Ongoing Support
    ↓
Future Planning (Post-Launch)
```

**Estimated Total Timeline:** 12-19 weeks (3-5 months) for full implementation with comprehensive testing

---

## Phase 1: Foundation & Setup

### Duration: 1-2 weeks

### Objectives

- ✅ Set up development environment
- ✅ Initialize .NET 8.0 project structure
- ✅ Configure MySQL database locally
- ✅ Establish Git repository and workflows
- ✅ Document setup process for reproducibility

### Deliverables

1. **Project Structure**
   ```
   StockMarketScreener/
   ├── src/
   │   └── StockMarketScreener.Console/
   │       ├── Program.cs
   │       ├── Configuration/
   │       ├── Services/
   │       ├── Models/
   │       └── appsettings.json
   ├── tests/
   │   └── StockMarketScreener.Tests/
   │       ├── Unit/
   │       └── Integration/
   ├── docs/
   │   ├── 01-PROJECT-OVERVIEW.md
   │   ├── 02-ARCHITECTURE-DESIGN.md
   │   ├── 03-DATA-FLOW-SCREENING-LOGIC.md
   │   └── 04-PROJECT-ROADMAP.md (this file)
   ├── .github/workflows/
   │   ├── quarterly-screening.yml
   │   └── weekly-stock-update.yml
   ├── .gitignore
   ├── README.md
   └── StockMarketScreener.sln
   ```

2. **Development Environment**
   - Visual Studio 2022 Community Edition configured
   - .NET 8.0 SDK installed and verified
   - MySQL 8.0 set up locally (Docker recommended)
   - Git initialized with main branch protection

3. **Database Schema**
   - Companies table created
   - QualifiedCompanies table created
   - FinancialMetrics table created
   - ScreeningRuns table created
   - RemovedCompanies table created
   - StockPriceHistory table created
   - Entity Framework migrations created

4. **Configuration Files**
   - appsettings.json template created
   - appsettings.Development.json for local development
   - Environment variables documented
   - API key placeholders prepared

5. **Dependency Injection Setup**
   - Program.cs configured with .NET Generic Host
   - Service registration for all layers
   - Logging configured (Serilog)
   - Configuration provider set up

### Key Tasks

- [ ] Create Visual Studio solution
- [ ] Install NuGet packages (EF Core, Serilog, MailKit, HttpClientFactory, Polly)
- [ ] Create database locally using Docker
- [ ] Run EF Core migrations
- [ ] Configure appsettings.json
- [ ] Set up Git repository
- [ ] Create initial project documentation
- [ ] Verify project builds successfully
- [ ] Create development checklist for future reference

### Best Practices Applied

✅ **Containerized Database**: Use Docker for MySQL (reproducible, easy reset)  
✅ **Entity Framework**: Use Code-First approach with migrations  
✅ **Dependency Injection**: Register all services in Program.cs  
✅ **Configuration Management**: Use appsettings.json for environment-specific settings  
✅ **Git Workflow**: Create feature branches for each phase  

### Success Criteria

- ✅ Project builds without errors
- ✅ Database connection verified
- ✅ EF Core migrations run successfully
- ✅ Git repository initialized with clean commit history
- ✅ Documentation complete and accurate

---

## Phase 2: Core Screening Engine

### Duration: 1 week

### Objectives

- ✅ Implement screening orchestrator
- ✅ Create stage service interfaces
- ✅ Build Stage 1 financial strength filters
- ✅ Build Stage 2 quality scoring
- ✅ Build Stage 3 valuation & management ranking
- ✅ Implement composite score calculation

### Deliverables

1. **ScreeningOrchestrator**
   - Entry point for screening process
   - Orchestrates all 3 stages
   - Handles result collection
   - Manages company comparison

2. **Stage1ScreeningService**
   - Implements 5 hard filters
   - Returns PASS/FAIL status
   - Logs filter rejection reasons
   - All-or-nothing logic (all must pass)

3. **Stage2ScoringService**
   - Calculates 4 quality pillars
   - Enforces pillar maximum caps
   - Returns quality score (0-100)
   - Documents scoring breakdown

4. **Stage3RankingService**
   - Calculates valuation score (0-35)
   - Calculates management score (0-25)
   - Computes composite score
   - Enforces >= 75 threshold

5. **Models**
   ```
   FinancialData
   ├── Ticker
   ├── CompanyName
   ├── ProfitabilityMetrics
   ├── FinancialStrengthMetrics
   ├── CashFlowMetrics
   ├── GrowthMetrics
   ├── ValuationMetrics
   └── ManagementQualityMetrics
   
   ScreeningResult
   ├── Ticker
   ├── Stage1Pass
   ├── QualityScore
   ├── ValuationScore
   ├── ManagementScore
   ├── CompositeScore
   ├── Status (NEW/MAINTAINED/REMOVED)
   └── RecommendationLevel
   ```

### Key Tasks

- [ ] Create ScreeningOrchestrator class
- [ ] Implement Stage1ScreeningService with all 5 filters
- [ ] Implement Stage2ScoringService with 4 pillars and caps
- [ ] Implement Stage3RankingService with composite formula
- [ ] Create FinancialData model
- [ ] Create ScreeningResult model
- [ ] Create Result comparison logic (NEW/MAINTAINED/REMOVED)
- [ ] Implement logging at each stage
- [ ] Test with hardcoded sample data

### Best Practices Applied

✅ **Separation of Concerns**: Each stage as separate service  
✅ **Interface-based Design**: IScreeningService, IStage1Service, etc.  
✅ **Immutable Scoring**: Once pillar caps enforced, no overflow possible  
✅ **Clear Logging**: Every decision logged with reasoning  
✅ **Sample Data Testing**: Use known companies to validate logic  

### Success Criteria

- ✅ All 5 Stage 1 filters implemented and testable
- ✅ All 4 Stage 2 pillars calculate correctly
- ✅ Composite score >= 75 threshold enforced
- ✅ Sample company scores match expected values
- ✅ Logging shows clear decision trail

---

## Phase 3: Data Integration & Reconciliation

### Duration: 2-3 weeks

### Objectives

- ✅ Implement SEC EDGAR data fetching
- ✅ Implement Yahoo Finance data fetching
- ✅ Implement IEX Cloud fallback
- ✅ Implement Alpha Vantage fallback
- ✅ Build data reconciliation logic
- ✅ Create data validation pipeline
- ✅ Handle API errors gracefully

### Deliverables

1. **SecEdgarService**
   - Fetch 10-K annual reports
   - Extract financial statements
   - Parse Income Statement, Balance Sheet, Cash Flow
   - Handle SEC EDGAR API rate limits
   - Timeout: 30 seconds per request
   - Retry logic: 3 attempts with exponential backoff

2. **YahooFinanceService**
   - Fetch current stock prices
   - Fetch dividend history
   - Fetch market capitalization
   - Handle Yahoo Finance API
   - Timeout: 10 seconds per request
   - Retry logic: 3 attempts

3. **IexCloudService**
   - Fallback for market data
   - Alternative pricing source
   - Real-time quotes
   - Timeout: 10 seconds per request

4. **AlphaVantageService**
   - Final fallback option
   - Historical pricing data
   - OHLC data
   - Timeout: 10 seconds per request

5. **DataReconciliationService**
   ```
   Priority Order:
   1. Financial Statements: SEC EDGAR only (required)
   2. Stock Prices: Yahoo → IEX Cloud → Alpha Vantage
   3. Treasury Yields: Federal Reserve FRED → Hardcoded 2.5%
   
   Reconciliation Logic:
   - Use primary source if available
   - Fallback to secondary if primary fails
   - Log all discrepancies
   - Never mix sources for same metric
   - Create audit trail of sources used
   ```

6. **DataValidationService**
   ```
   Validation Rules:
   1. Completeness: All required fields present
   2. Range: ROE -100% to +200%, D/E 0 to 5.0, etc.
   3. Balance Sheet: Assets = Liabilities + Equity ±2%
   4. Consistency: YoY changes < 300%
   5. Data Age: Financials < 1 year, Prices < 1 day
   
   Outputs:
   - IsValid: boolean
   - Warnings: list of non-critical issues
   - Errors: list of critical issues
   - DataQualityScore: percentage (0-100%)
   ```

### API Keys Required

Create free accounts and obtain API keys:

| Service | Free Tier | Key Setup | URL |
|---------|-----------|-----------|-----|
| **SEC EDGAR** | Unlimited | No key needed | https://www.sec.gov/cgi-bin/ |
| **Yahoo Finance** | Limited* | Via unofficial API | https://finance.yahoo.com |
| **IEX Cloud** | Free tier available | https://iexcloud.io | Requires account |
| **Alpha Vantage** | Free: 5 calls/min | https://www.alphavantage.co | Requires API key |
| **Federal Reserve FRED** | Free & unlimited | https://fred.stlouisfed.org | Requires API key |

**Note:** Yahoo Finance doesn't have official API; use community wrapper (Python yfinance equivalent in C#)

### Key Tasks

- [ ] Create SEC EDGAR fetching with error handling
- [ ] Create Yahoo Finance wrapper service
- [ ] Implement IEX Cloud integration
- [ ] Implement Alpha Vantage integration
- [ ] Create data reconciliation logic
- [ ] Create data validation pipeline
- [ ] Implement retry logic with Polly
- [ ] Test each API independently
- [ ] Test fallback chain (primary → secondary → tertiary)
- [ ] Create API key configuration template

### Best Practices Applied

✅ **Polly Retry Policies**: Exponential backoff for rate limiting  
✅ **Timeout Management**: Different timeouts per service (30s SEC, 10s prices)  
✅ **Graceful Degradation**: Continue with fallback data if primary fails  
✅ **Audit Trail**: Log all data sources used  
✅ **Error Isolation**: Company error doesn't crash entire batch  
✅ **Circuit Breaker**: Stop calling API if it's down (Polly)  

### Success Criteria

- ✅ SEC EDGAR fetching works for 10+ companies
- ✅ Yahoo Finance price fetching works
- ✅ Fallback to IEX when Yahoo unavailable
- ✅ Data validation catches corrupted inputs
- ✅ All API errors logged appropriately
- ✅ Reconciliation prioritizes sources correctly
- ✅ No sensitive data in logs

---

## Phase 4: Metric Calculation & Screening Logic

### Duration: 2-3 weeks

### Objectives

- ✅ Implement all 40+ financial metrics
- ✅ Calculate year-by-year metrics (10 years)
- ✅ Calculate 10-year averages
- ✅ Implement stability analysis (CV)
- ✅ Implement trend analysis
- ✅ Verify calculations with known data

### Deliverables

1. **MetricCalculationService**
   - Profitability: 8 metrics (margins, ROE, ROA, ROIC, ROCE)
   - Financial Strength: 6 metrics (D/E, liquidity, coverage)
   - Cash Flow: 6 metrics (OCF, FCF, ratios, percentages)
   - Growth: 7 metrics (YoY, CAGR, EPS, retention)
   - Valuation: 8 metrics (P/E, P/B, P/S, PEG, yields, ratios)
   - Management: 6 metrics (insider ownership, goodwill, charges, retained, expense, turnover)

2. **TrendAnalysisService**
   - Regression analysis on 10-year data
   - Identify improving/stable/declining trends
   - Calculate trend slopes
   - Detect inflection points

3. **StabilityCalculationService**
   - Coefficient of Variation (CV) for each metric category
   - Interpretation: < 10% excellent, 10-20% good, 20-30% acceptable, > 30% volatile
   - Identify outlier years
   - Flag inconsistent companies

4. **Metric Models**
   ```
   CompanyMetricsRecord
   ├── Ticker
   ├── FiscalYear[]  // 2016-2025
   ├── ProfitabilityMetrics[]
   ├── FinancialStrengthMetrics[]
   ├── CashFlowMetrics[]
   ├── GrowthMetrics[]
   ├── ValuationMetrics[]
   ├── ManagementQualityMetrics[]
   ├── StabilityMetrics
   └── TrendAnalysis
   ```

5. **Calculation Verification**
   - Create test cases with known companies (MSFT, AAPL, JNJ, KO, JPM)
   - Verify calculations match financial databases
   - Document formula for each metric
   - Create calculation examples in code comments

### Key Tasks

- [ ] Implement all 8 profitability metrics with formulas
- [ ] Implement all 6 financial strength metrics
- [ ] Implement all 6 cash flow metrics
- [ ] Implement all 7 growth metrics with CAGR calculation
- [ ] Implement all 8 valuation metrics
- [ ] Implement all 6 management quality metrics
- [ ] Create 10-year averaging logic
- [ ] Implement Coefficient of Variation (CV) calculation
- [ ] Create trend analysis with regression
- [ ] Test against 5+ known companies
- [ ] Verify metrics against external sources (Yahoo Finance, etc.)
- [ ] Document all metric formulas

### Best Practices Applied

✅ **Formula Documentation**: Every metric has inline comments explaining calculation  
✅ **Defensive Coding**: Handle division by zero, null values gracefully  
✅ **Precision**: Use decimal type for financial calculations (not float)  
✅ **Immutable Results**: Metrics calculated once, never modified  
✅ **Verification**: Cross-check calculations against known sources  

### Success Criteria

- ✅ All 40+ metrics calculate without errors
- ✅ 10-year averaging works correctly
- ✅ Calculations match Yahoo Finance/SEC Edgar data
- ✅ Stability metrics identify volatile companies
- ✅ Trend analysis correctly identifies improving/declining trends
- ✅ Edge cases handled (missing years, negative values, etc.)

---

## Phase 5: Results & Notifications

### Duration: 1-2 weeks

### Objectives

- ✅ Implement database save logic
- ✅ Build email notification service
- ✅ Create CSV export functionality
- ✅ Implement console reporting
- ✅ Create HTML email templates
- ✅ Test end-to-end result delivery

### Deliverables

1. **QualifiedCompanyService**
   - Save newly qualified companies
   - Update scores for maintained companies
   - Mark removed companies as inactive
   - Track first/last qualified dates
   - Maintain complete audit trail

2. **EmailNotificationService**
   - Build HTML email template
   - Include summary statistics (qualified, new, maintained, removed)
   - Create detailed results table
   - Add business quality assessment
   - Include valuation analysis
   - List removed companies with reasons
   - Send via SMTP (Gmail recommended)

3. **CsvExportService**
   - Export all qualified companies
   - Include all 40+ metrics
   - Include composite scores
   - Include data source attribution
   - Generate filename with date

4. **ConsoleReporter**
   - Print screening progress (Company X/500)
   - Show pass/fail status for each stage
   - Print summary at completion
   - Display execution time
   - Show email delivery status

5. **Models**
   ```
   ScreeningReport
   ├── RunDate
   ├��─ IndustriesScreened[]
   ├── CompaniesProcessed
   ├── CompaniesQualified
   ├── NewlyQualified[]
   ├── Maintained[]
   ├── Removed[]
   ├── QualifiedCompanies[]
   │   ├── Ticker
   │   ├── CompanyName
   │   ├── Scores (Quality, Valuation, Management, Composite)
   │   ├── StockPrice
   │   ├── PERatio
   │   ├── DividendYield
   │   ├── Recommendation
   │   └── Status
   └── ExecutionTimeSeconds
   ```

### Key Tasks

- [ ] Implement QualifiedCompanyService with DB save logic
- [ ] Create EmailNotificationService
- [ ] Design HTML email template
- [ ] Implement SMTP configuration
- [ ] Create CSV export with all metrics
- [ ] Implement console progress reporting
- [ ] Test email delivery (use test email first)
- [ ] Test CSV export format and readability
- [ ] Verify database updates
- [ ] Create sample screening report

### Best Practices Applied

✅ **Template-Based Emails**: Use HTML templates for professional appearance  
✅ **Color-Coded Results**: Green/Yellow/Red for score ranges in email  
✅ **Comprehensive Data**: Include all metrics users might need  
✅ **Audit Trail**: Log all saves/updates to database  
✅ **Error Notifications**: Email failures logged, doesn't stop screening  
✅ **CSV Standards**: RFC 4180 compliant CSV format  

### Success Criteria

- ✅ Email sends successfully with professional formatting
- ✅ CSV exports with correct formatting and all columns
- ✅ Console output shows clear progress
- ✅ Database saves correctly with all metrics
- ✅ NEW/MAINTAINED/REMOVED categorization accurate
- ✅ Email includes all required information

---

## Phase 6: GitHub Actions Automation

### Duration: 1 week

### Objectives

- ✅ Create quarterly screening workflow
- ✅ Create weekly stock price update workflow
- ✅ Configure database in CI/CD environment
- ✅ Set up secrets management
- ✅ Test automation locally first
- ✅ Document workflow setup

### Deliverables

1. **quarterly-screening.yml**
   ```yaml
   name: Quarterly Screening Run
   
   on:
     schedule:
       # 15th of March, June, September, December at 2 PM UTC
       - cron: '0 14 15 3,6,9,12 *'
     workflow_dispatch:  # Manual trigger
   
   jobs:
     screening:
       runs-on: ubuntu-latest
       
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
       
       steps:
         - uses: actions/checkout@v3
         - uses: actions/setup-dotnet@v3
           with:
             dotnet-version: '8.0.x'
         
         - name: Restore dependencies
           run: dotnet restore
         
         - name: Build
           run: dotnet build --configuration Release
         
         - name: Run migrations
           env:
             ConnectionStrings__DefaultConnection: 'Server=mysql;Uid=root;Pwd=root;Database=stockscreener'
           run: dotnet ef database update
         
         - name: Run screening
           env:
             ConnectionStrings__DefaultConnection: 'Server=mysql;Uid=root;Pwd=root;Database=stockscreener'
             Email__SmtpServer: ${{ secrets.SMTP_SERVER }}
             Email__SenderEmail: ${{ secrets.SENDER_EMAIL }}
             Email__SenderPassword: ${{ secrets.SENDER_PASSWORD }}
             Email__RecipientEmail: ${{ secrets.RECIPIENT_EMAIL }}
           run: dotnet run --project src/StockMarketScreener.Console/StockMarketScreener.Console.csproj
         
         - name: Upload results
           if: always()
           uses: actions/upload-artifact@v3
           with:
             name: screening-results
             path: output/
             retention-days: 90
   ```

2. **weekly-stock-update.yml**
   ```yaml
   name: Weekly Stock Price Update
   
   on:
     schedule:
       # Every Monday at 8 PM UTC (after market close)
       - cron: '0 20 * * 1'
     workflow_dispatch:
   
   jobs:
     update:
       runs-on: ubuntu-latest
       
       services:
         mysql:
           image: mysql:8.0
           env:
             MYSQL_ROOT_PASSWORD: root
             MYSQL_DATABASE: stockscreener
       
       steps:
         - uses: actions/checkout@v3
         - uses: actions/setup-dotnet@v3
           with:
             dotnet-version: '8.0.x'
         
         - name: Restore & Build
           run: |
             dotnet restore
             dotnet build --configuration Release
         
         - name: Run stock price update
           env:
             ConnectionStrings__DefaultConnection: 'Server=mysql;Uid=root;Pwd=root;Database=stockscreener'
             Email__SmtpServer: ${{ secrets.SMTP_SERVER }}
             Email__SenderEmail: ${{ secrets.SENDER_EMAIL }}
             Email__SenderPassword: ${{ secrets.SENDER_PASSWORD }}
           run: dotnet run --project src/StockMarketScreener.Console/StockMarketScreener.Console.csproj -- --update-prices-only
   ```

3. **GitHub Secrets Setup**
   ```
   Required secrets to configure in GitHub repo:
   - SMTP_SERVER: smtp.gmail.com
   - SMTP_PORT: 587
   - SENDER_EMAIL: your-email@gmail.com
   - SENDER_PASSWORD: your-gmail-app-password
   - RECIPIENT_EMAIL: your-email@gmail.com
   ```

4. **Gmail App Password Setup**
   - Enable 2-Factor Authentication on Gmail
   - Generate app-specific password
   - Use this password in GitHub Secrets
   - Document steps for future reference

### Key Tasks

- [ ] Create .github/workflows directory
- [ ] Create quarterly-screening.yml workflow
- [ ] Create weekly-stock-update.yml workflow
- [ ] Test workflows locally with act (GitHub Actions local runner)
- [ ] Set up MySQL service in workflow
- [ ] Configure all GitHub Secrets
- [ ] Test quarterly workflow manually (workflow_dispatch)
- [ ] Test weekly workflow manually
- [ ] Verify email sends from GitHub Actions
- [ ] Create GitHub Actions setup documentation
- [ ] Set up workflow logging
- [ ] Create troubleshooting guide

### Best Practices Applied

✅ **Service Containers**: MySQL runs in container during CI  
✅ **Health Checks**: MySQL health check before running app  
✅ **Secrets Management**: Sensitive data via GitHub Secrets, never hardcoded  
✅ **Artifact Retention**: Results kept 90 days for audit trail  
✅ **Conditional Steps**: Upload artifacts even if screening fails  
✅ **Manual Trigger**: Workflow_dispatch for on-demand runs  
✅ **Clear Logging**: All steps logged for debugging  

### Success Criteria

- ✅ Quarterly workflow runs on schedule without errors
- ✅ Weekly workflow executes successfully
- ✅ Email sends from GitHub Actions
- ✅ Database updates correctly in CI environment
- ✅ Results artifacts uploaded and accessible
- ✅ Logs show clear execution path
- ✅ Manual triggers work perfectly

---

## Phase 7: Testing & Quality Assurance

### Duration: 2-3 weeks

### Objectives

- ✅ Implement comprehensive unit tests
- ✅ Implement integration tests
- ✅ Create test data and fixtures
- ✅ Achieve high code coverage
- ✅ Test edge cases and error scenarios
- ✅ Validate against real data

### Deliverables

1. **Unit Tests**
   - Metric calculations (100+ test cases)
   - Screening logic (50+ test cases)
   - Data validation (30+ test cases)
   - Score calculations (40+ test cases)
   - Utility functions

2. **Integration Tests**
   - End-to-end screening flow
   - Database operations
   - Data reconciliation pipeline
   - Email notification sending
   - CSV export functionality

3. **Test Fixtures**
   ```
   Test Data (10 Companies):
   1. MSFT (Microsoft) - Should PASS
   2. AAPL (Apple) - Should PASS
   3. JNJ (Johnson & Johnson) - Should PASS
   4. KO (Coca-Cola) - Should PASS
   5. JPM (JPMorgan Chase) - Should PASS
   6. TSLA (Tesla) - Should FAIL (high valuation)
   7. AMD (AMD) - Should FAIL (high debt)
   8. SNAP (Snapchat) - Should FAIL (low ROE)
   9. UBER (Uber) - Should FAIL (negative FCF)
   10. GOOG (Google) - Should PASS
   
   Each fixture includes:
   - 10 years of financial statements
   - Real historical data
   - Known expected outcomes
   - Documented edge cases
   ```

4. **Test Coverage**
   - Target: >= 80% code coverage
   - Critical paths: 100% coverage
   - Edge cases: All covered
   - Error paths: All tested

5. **Performance Tests**
   - 500 company screening: < 60 minutes
   - Individual metric calc: < 100ms
   - Database save: < 5 seconds
   - Email send: < 5 seconds

### Testing Strategy

```
Unit Tests (xUnit + Moq):
├─ MetricCalculationTests
│  ├─ TestROECalculation (with 5+ scenarios)
│  ├─ TestNetMarginCalculation
│  ├─ TestCAGRCalculation
│  ├─ TestCoefficientOfVariation
│  └─ ... 100+ total tests
│
├─ ScreeningLogicTests
│  ├─ TestStage1Filters (pass/fail scenarios)
│  ├─ TestStage2QualityScoring (pillar caps)
│  ├─ TestStage3CompositeScore (weighting)
│  ├─ TestCompanyComparison (NEW/MAINTAINED/REMOVED)
│  └─ ... 50+ total tests
│
├─ DataValidationTests
│  ├─ TestDataCompleteness
│  ├─ TestRangeValidation
│  ├─ TestBalanceSheetValidation
│  ├─ TestConsistencyCheck
│  └─ ... 30+ total tests
│
└─ EdgeCaseTests
   ├─ TestDivisionByZero
   ├─ TestNullValues
   ├─ TestMissingYears
   ├─ TestOutlierData
   └─ ... 20+ total tests

Integration Tests (xUnit + Testcontainers):
├─ EndToEndScreeningTests
│  ├─ TestFullScreeningFlow (5-10 companies)
│  ├─ TestResultsCorrectness
│  ├─ TestDatabaseSave
│  └─ TestComparisonLogic
│
├─ DataIntegrationTests
│  ├─ TestSecEdgarFetching (with mock data)
│  ├─ TestDataReconciliation
│  ├─ TestFallbackLogic
│  └─ TestDataValidation
│
├─ EmailNotificationTests
│  ├─ TestEmailFormatting
│  ├─ TestDataInclusion
│  └─ TestCSVExport
│
└─ GitHubActionsTests
   ├─ TestWorkflowTriggers
   ├─ TestEnvironmentVariables
   └─ TestArtifactGeneration
```

### Key Tasks

- [ ] Create Unit test project structure
- [ ] Write 100+ metric calculation tests
- [ ] Write 50+ screening logic tests
- [ ] Write 30+ data validation tests
- [ ] Write 10+ edge case tests
- [ ] Create test data fixtures (10 companies)
- [ ] Create integration test suite
- [ ] Set up Testcontainers for MySQL
- [ ] Achieve 80%+ code coverage
- [ ] Performance test with 500 companies
- [ ] Test error scenarios and recovery
- [ ] Document testing strategy
- [ ] Create test data generation scripts

### Best Practices Applied

✅ **AAA Pattern**: Arrange-Act-Assert structure  
✅ **Test Isolation**: Each test independent, no side effects  
✅ **Mock External Services**: Use Moq for API calls  
✅ **Real Database Tests**: Testcontainers for true integration tests  
✅ **Named Tests**: Descriptive test names explain what they test  
✅ **Test Data Builders**: Fluent builders for complex test objects  
✅ **Code Coverage Reports**: Generate and track coverage  

### Success Criteria

- ✅ >= 80% code coverage (critical paths 100%)
- ✅ All 200+ tests pass consistently
- ✅ Edge cases handled and tested
- ✅ Real data validation against Yahoo Finance
- ✅ Performance targets met (40-50 min for 500 companies)
- ✅ Error handling tested and working
- ✅ Test execution documented

---

## Phase 8: UI Development

### Duration: 3-4 weeks (Post-Launch)

### Status: Planned for after core functionality complete

### Objectives

- ✅ Create web dashboard
- ✅ Display qualified companies
- ✅ Show detailed metrics
- ✅ Enable filtering/sorting
- ✅ Display charts and trends
- ✅ Provide manual screening trigger

### Technology Stack

- **Framework**: ASP.NET Core 8.0 (Razor Pages or Blazor)
- **Frontend**: Bootstrap 5 (responsive, professional)
- **Database**: Use existing MySQL
- **Charting**: Chart.js (metrics visualization)
- **Hosting**: Free tier option (Azure App Service, Heroku, etc.)

### Key Features

1. **Dashboard**
   - Summary statistics (qualified count, new/removed this quarter)
   - Trending metrics chart
   - Latest screening date and results

2. **Company List**
   - All qualified companies with scores
   - Sortable/filterable table
   - Color-coded recommendation levels
   - Link to detailed view

3. **Company Detail**
   - All 40+ metrics displayed
   - 10-year trend charts
   - Financial statement breakdown
   - Screening breakdown (stage 1, 2, 3 results)

4. **History**
   - Previous screening results
   - Company removal history
   - Date and score progression

5. **Manual Trigger**
   - Initiate screening from UI
   - Monitor progress
   - View results as they complete

### Out of Scope (Phase 8)

- Real-time notifications
- Mobile app
- Advanced analytics
- User authentication/multi-user
- Export to external platforms

---

## Post-Launch Planning

### Status: After Phase 8 UI Launch

### Planned Features (Prioritized by Value)

1. **Web UI Enhancements (High Value)**
   - User authentication & multi-user support
   - Portfolio tracking
   - Custom screening criteria configuration
   - Alerts for qualifying companies

2. **Mobile App (Medium Value)**
   - React Native or Flutter
   - Push notifications
   - Company alerts
   - Portfolio management

3. **Advanced Analytics (Medium Value)**
   - Sector analysis
   - Performance tracking of recommendations
   - Backtesting screener criteria
   - Statistical analysis

4. **Integration Features (Low Value)**
   - Broker integration (TD Ameritrade, etc.)
   - Portfolio syncing
   - Trade execution
   - Tax reporting

5. **Additional Data Sources (Low Value)**
   - Earnings call transcripts
   - Analyst estimates
   - News sentiment analysis
   - Insider trading data

### Planning Process

When ready to plan future features:
1. Gather user feedback on core system
2. Define prioritization criteria
3. Create detailed specifications
4. Create new roadmap document
5. Begin Phase 9+ planning

---

## Dependencies & Prerequisites

### Required Before Starting

#### API Keys (All Free)

1. **SEC EDGAR** (No API key needed)
   - Accessible at https://www.sec.gov/cgi-bin/
   - Rate limit: Reasonable (no official limit, be respectful)
   - Setup time: < 5 minutes

2. **Yahoo Finance**
   - Community API via C# wrapper
   - Rate limit: 1-2 requests per second
   - Setup time: < 10 minutes (install NuGet package)

3. **IEX Cloud** (Recommended)
   - Free tier: 100 messages/month
   - https://iexcloud.io
   - Setup time: 15 minutes (register, get API key)

4. **Alpha Vantage**
   - Free tier: 5 calls/minute
   - https://www.alphavantage.co
   - Setup time: 15 minutes (register, get API key)

5. **Federal Reserve FRED** (Optional)
   - Free tier: Unlimited
   - https://fred.stlouisfed.org
   - Setup time: 15 minutes (register, get API key)

6. **Gmail SMTP** (For email notifications)
   - Free with Google account
   - Setup time: 20 minutes (2FA, app password)

#### Software Requirements

| Tool | Version | Purpose | Cost |
|------|---------|---------|------|
| **Visual Studio 2022** | Community | IDE | Free |
| **.NET SDK** | 8.0 | Runtime | Free |
| **MySQL** | 8.0 | Database | Free |
| **Docker Desktop** | Latest | Container runtime | Free |
| **Git** | Latest | Version control | Free |
| **GitHub** | Free plan | Repository | Free |

#### Knowledge Prerequisites

- **C# Fundamentals**: Classes, LINQ, async/await
- **Entity Framework Core**: Models, migrations, queries
- **SQL Basics**: SELECT, INSERT, UPDATE queries
- **REST APIs**: HTTP requests, authentication
- **GitHub Basics**: Clone, commit, push, pull requests
- **YAML**: GitHub Actions workflow syntax

### Setup Checklist

**Week 1 (Phase 1 - Foundation)**
- [ ] Install Visual Studio 2022 Community
- [ ] Install .NET 8.0 SDK
- [ ] Install Docker Desktop
- [ ] Set up local MySQL instance
- [ ] Create GitHub repository
- [ ] Create GitHub Projects board
- [ ] Document setup process
- [ ] Create initial project structure

**Week 2 (Before Phase 3)**
- [ ] Register IEX Cloud (free tier)
- [ ] Register Alpha Vantage
- [ ] Register Federal Reserve FRED
- [ ] Create Gmail app password
- [ ] Configure appsettings.json template
- [ ] Test API connectivity
- [ ] Document API rate limits

**Week 3+ (Before Phase 6)**
- [ ] Set up GitHub Secrets
- [ ] Test GitHub Actions locally (with act tool)
- [ ] Document workflow setup
- [ ] Create GitHub Actions troubleshooting guide

---

## Risk Management

### Identified Risks & Mitigation

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|-----------|
| **API Rate Limiting** | High | Medium | Use fallback sources, cache data, respect limits |
| **Data Quality Issues** | Medium | High | Comprehensive validation, alert on anomalies |
| **MySQL Setup Complexity** | Low | Medium | Use Docker, provide scripts, document thoroughly |
| **GitHub Actions Secrets** | Low | High | Document setup clearly, use security best practices |
| **Metric Calculation Errors** | Medium | High | Extensive unit tests, verify against known sources |
| **Email Delivery Failures** | Low | Low | Retry logic, log errors, continue screening |
| **Performance Degradation** | Medium | Medium | Parallel processing option, caching, monitoring |
| **Database Corruption** | Low | Critical | Regular backups, migrations, integrity checks |

### Mitigation Strategies

**API Rate Limiting**
- Implement Polly retry policies with exponential backoff
- Cache company lists and reference data
- Space out API calls
- Monitor rate limit headers
- Fall back to secondary sources

**Data Quality**
- Comprehensive validation at each stage
- Cross-check metrics against multiple sources
- Flag anomalies in logs and email
- Create data quality reports
- Manual review process for edge cases

**Performance**
- Parallel processing for API calls (5-10 concurrent)
- Local caching for reference data
- Database indexing on frequently queried columns
- Monitor execution time metrics
- Create performance baseline

---

## Success Metrics & Validation

### Phase Completion Criteria

Each phase is complete when:

1. **All deliverables created** ✓
2. **Code compiles without errors** ✓
3. **All objectives achieved** ✓
4. **Documentation complete** ✓
5. **Code reviewed** ✓
6. **Tests passing (where applicable)** ✓

### Project Success Criteria

**The project is fully successful when:**

✅ **Screening produces correct results**
- Financial strength filters working as designed
- Quality scores correlate with business quality
- Valuation assessment accurate
- Companies >= 75 score are genuinely quality investments
- External validation: Compare results with professional screeners

✅ **Automation running reliably**
- Quarterly screenings execute without manual intervention
- Email notifications arrive consistently
- GitHub Actions logs show clean execution
- No data corruption or inconsistencies

✅ **Data integration complete**
- SEC EDGAR data reliably fetched
- Yahoo Finance prices accurate
- Fallback sources working when needed
- All 40+ metrics calculating correctly

✅ **Quality assurance passing**
- >= 80% code coverage achieved
- All edge cases tested and handled
- Performance targets met (40-50 min for 500 companies)
- Error handling working as designed

✅ **Documentation complete**
- Architecture documented
- Setup process documented
- API integration guide complete
- Troubleshooting guide created
- Code well-commented

✅ **UI functional** (Phase 8)
- Dashboard displaying correctly
- Company list queryable and filterable
- Detail view showing all metrics
- Charts rendering properly
- Manual trigger working

### Validation Methods

1. **Manual Testing**
   - Run screening with test data
   - Verify results accuracy
   - Check email formatting
   - Test error scenarios

2. **Automated Testing**
   - Run unit tests (200+)
   - Run integration tests
   - Code coverage reports
   - Performance benchmarks

3. **Real-World Validation**
   - Compare results with Yahoo Finance
   - Compare results with other screeners
   - Interview professional investors
   - Backtesting performance

4. **Continuous Monitoring**
   - GitHub Actions logs
   - Error rate tracking
   - Performance metrics
   - Data quality scores

---

## Timeline Estimate

Based on best practices and realistic timelines:

| Phase | Duration | Est. Dates* | Notes |
|-------|----------|-----------|-------|
| **Phase 1: Foundation** | 1-2 weeks | Wk 1-2 | Sequential |
| **Phase 2: Screening Engine** | 1 week | Wk 3 | Core logic |
| **Phase 3: Data Integration** | 2-3 weeks | Wk 4-6 | API integration |
| **Phase 4: Metric Calculation** | 2-3 weeks | Wk 7-9 | Heaviest phase |
| **Phase 5: Results & Notifications** | 1-2 weeks | Wk 10-11 | Final touches |
| **Phase 6: GitHub Actions** | 1 week | Wk 12 | Automation |
| **Phase 7: Testing & QA** | 2-3 weeks | Wk 13-15 | Quality focus |
| **Phase 8: UI Development** | 3-4 weeks | Wk 16-19 | Post-launch |
| | | | |
| **TOTAL (Full Implementation)** | **12-19 weeks** | **3-5 months** | Quality over speed |

*Timeline estimates based on:
- Solo developer working part-time
- Quality over speed prioritization
- Comprehensive testing
- No fixed deadline pressure

### Realistic Expectations

**Minimum viable product** (screening + email): 6-8 weeks  
**Production ready** (with testing): 12-15 weeks  
**Including UI**: 16-19 weeks  

**This timeline assumes:**
- Part-time development (~20 hours/week)
- No major blockers or scope changes
- Comprehensive testing at each phase
- Full documentation completed

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-03-14 | ThommCroft | Initial project roadmap with 8 phases, best practices, dependencies, risk management, and success criteria |

---

## Next Steps

1. **Review this roadmap** with stakeholders
2. **Adjust timeline** if needed
3. **Prepare environment** (install software, create API keys)
4. **Create GitHub Projects board** to track progress
5. **Begin Phase 1** when ready

---

## Appendices

### A. Required API Keys & Setup

Create a checklist document with exact steps for each API key:
- Screenshots of registration process
- Rate limit documentation
- Example API calls
- Configuration in appsettings.json

### B. GitHub Actions Setup Guide

Step-by-step guide for:
- Creating secrets in GitHub
- Testing workflows locally
- Troubleshooting common issues
- Monitoring workflow runs

### C. MySQL Setup Script

Automated script to create:
- Database
- All tables
- Initial schema
- Sample data (optional)

### D. Development Environment Checklist

Comprehensive checklist for:
- Software installation
- Configuration
- API key setup
- GitHub setup
- Database initialization

### E. Testing Strategy Document

Detailed guide covering:
- Unit testing approach
- Integration testing approach
- Test data generation
- Coverage reporting
- Performance testing

---

**End of Project Roadmap Document**

This roadmap provides a clear, phased approach to building the Stock Market Screener. Follow each phase sequentially, validate at each step, and adjust timeline as needed based on progress.

---

## Document History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-03-14 | ThommCroft | Initial project roadmap with 8 phases, best practices, dependencies, risk management, and success criteria |
| 1.1 | 2026-03-14 | ThommCroft | **CURRENT** - Cross-reference verification completed with Investment Requirements, Architecture Design, and Data Flow documents; all core requirements confirmed aligned |
