# Project Roadmap - Stock Market Screener

**Version:** 2.0 (Merged Metric Calculation + Unified Hard Filter Evaluation)  
**Date:** 2026-03-14  
**Author:** ThommCroft  
**Framework:** Warren Buffett Value Investing + Unified Comprehensive Analysis

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Vision & Success Criteria](#vision--success-criteria)
3. [Project Phases Overview](#project-phases-overview)
4. [Phase 1: Foundation & Setup](#phase-1-foundation--setup)
5. [Phase 2: Core Screening Engine](#phase-2-core-screening-engine)
6. [Phase 3: Data Integration & Reconciliation](#phase-3-data-integration--reconciliation)
7. [Phase 4: Merged Metric Calculation & Hard Filter Evaluation](#phase-4-merged-metric-calculation--hard-filter-evaluation)
8. [Phase 5: Quality Assessment & Composite Scoring](#phase-5-quality-assessment--composite-scoring)
9. [Phase 6: Results & Notifications](#phase-6-results--notifications)
10. [Phase 7: GitHub Actions Automation](#phase-7-github-actions-automation)
11. [Phase 8: Testing & Quality Assurance](#phase-8-testing--quality-assurance)
12. [Phase 9: UI Development (Post-Launch)](#phase-9-ui-development-post-launch)
13. [Dependencies & Prerequisites](#dependencies--prerequisites)
14. [Risk Management](#risk-management)
15. [Success Metrics & Validation](#success-metrics--validation)
16. [Timeline Estimate](#timeline-estimate)

---

## Executive Summary

**Stock Market Screener** is a comprehensive financial analysis engine that identifies high-quality, undervalued stocks using Warren Buffett and Charlie Munger principles.

**Key Architectural Feature (v2.0):**
- ✅ **MERGED APPROACH** - Unified metric calculation + hard filter evaluation
- ✅ **ALL 40+ metrics calculated upfront** for every qualifying company
- ✅ **NO cascading rejections** - complete financial picture before decisions
- ✅ **Holistic hard filter assessment** - context from all metrics informs evaluation
- ✅ **Comprehensive quality scoring** - pillars with maximum caps
- ✅ **Automated quarterly execution** via GitHub Actions
- ✅ **Solo developer friendly** - complete system for one person

**Target Implementation:** 9-11 weeks (solo developer)

**Project Outcome:** 
- Console-based screening application
- Quarterly automated screening via GitHub Actions
- Local MySQL database of qualified companies
- Email notifications with detailed analysis
- Weekly stock price updates
- CSV exports for manual review

---

## Vision & Success Criteria

### Project Vision

Build a **complete, automated stock screening system** that:
1. Screens 500+ US companies quarterly
2. Applies strict Buffett/Munger value criteria
3. Calculates ALL 40+ financial metrics comprehensively
4. Evaluates companies holistically (not checklist-based)
5. Stores only passing companies (> 75 composite score)
6. Sends detailed email reports with complete metrics
7. Requires zero manual intervention once deployed
8. Can be operated by one developer with minimal overhead

### Success Definition

**Technical Success:**
- ✅ System screens 500+ companies in 45-60 minutes
- ✅ Calculates all 40+ metrics for each company
- ✅ Hard filters evaluated with complete data
- ✅ Zero manual data entry required
- ✅ Zero crashes during screening runs
- ✅ Detailed audit logs of all decisions
- ✅ Complete test coverage (unit + integration)

**Operational Success:**
- ✅ GitHub Actions runs automatically on schedule
- ✅ Email notifications sent without errors
- ✅ Database updates correctly (new/maintained/removed)
- ✅ Results exportable to CSV
- ✅ Can modify criteria without code changes
- ✅ One-person operation possible

**Investment Success:**
- ✅ Identifies 40-60 high-quality stocks (9-12% pass rate)
- ✅ Only PASS = qualified companies stored
- ✅ Metrics align with Investment Requirements v3.0
- ✅ Hard filters prevent weak businesses
- ✅ Quality scoring identifies excellent companies
- ✅ Valuation assessment ensures margin of safety

---

## Project Phases Overview

| Phase | Name | Duration | Purpose |
|-------|------|----------|---------|
| 1 | Foundation & Setup | 1-2 weeks | Project structure, database, configuration |
| 2 | Core Screening Engine | 1 week | Orchestrator, company fetching, status tracking |
| 3 | Data Integration & Reconciliation | 2-3 weeks | SEC EDGAR, Yahoo Finance, data validation |
| 4 | Merged Metric Calculation + Hard Filters | 2-3 weeks | **NEW:** All 40+ metrics + unified evaluation |
| 5 | Quality Assessment & Composite Scoring | 1-2 weeks | Quality pillars, valuation, management scoring |
| 6 | Results & Notifications | 1-2 weeks | Email, CSV export, console output |
| 7 | GitHub Actions Automation | 1 week | Quarterly workflow, weekly updates |
| 8 | Testing & Quality Assurance | 2-3 weeks | Unit, integration, end-to-end tests |
| 9 | UI Development | 3-4 weeks | *Post-Launch:* Dashboard, analytics |

**Total Estimated Time:** 9-11 weeks (solo developer)

---

## Phase 1: Foundation & Setup

### Duration: 1-2 weeks

### Objectives
- Set up project structure and dependencies
- Configure MySQL database locally
- Create Entity Framework Core data models
- Implement dependency injection
- Configure logging and application settings
- Establish git workflow

### Deliverables
- Complete .NET 8.0 console application project
- EF Core DbContext with all data models
- appsettings.json with configuration
- Database schema created and tested
- Serilog logging configured
- GitHub repository set up

### Key Tasks

**1. Project Structure (Days 1-2)**
```
StockMarketScreener/
├── src/
│   ├── StockMarketScreener.Console/
│   │   ├── Program.cs
│   │   ├── Configuration/
│   │   ├── Data/
│   │   ├── Services/
│   │   └── Models/
│   └── StockMarketScreener.Tests/
├── docs/
├── .github/workflows/
├── appsettings.json
└── README.md
```

**2. Database Design (Days 2-3)**
- Create MySQL database: `stockscreener`
- Design tables: Companies, QualifiedCompanies, FinancialMetrics, ScreeningRuns, RemovedCompanies
- Create indexes and relationships
- EF Core migrations

**3. Dependency Injection Setup (Days 3-4)**
```csharp
var host = Host.CreateDefaultBuilder(args)
    .ConfigureServices((context, services) =>
    {
        // Register all services
        services.AddScoped<IScreenerDbContext, ScreenerDbContext>();
        services.AddScoped<ICompanyRepository, CompanyRepository>();
        services.AddScoped<IScreeningOrchestrator, ScreeningOrchestrator>();
        // ... more services
    })
    .UseSerilog()
    .Build();
```

**4. Configuration Management (Days 4-5)**
- appsettings.json with environment-specific settings
- Secrets management for API keys
- Industry configuration
- Screening thresholds
- Email settings

### Best Practices Applied
- ✅ SOLID principles (Single Responsibility, Dependency Injection)
- ✅ Repository pattern for data access
- ✅ Configuration as code (appsettings.json)
- ✅ Structured logging (Serilog)
- ✅ Entity Framework Core for ORM
- ✅ Git version control from start

### Success Criteria
- [x] Project compiles without errors
- [x] Database connection verified
- [x] All tables created with correct schema
- [x] DI container resolves all services
- [x] Logging works for all components
- [x] appsettings.json loads correctly
- [x] Can run empty console app without errors

---

## Phase 2: Core Screening Engine

### Duration: 1 week

### Objectives
- Create screening orchestrator that controls overall flow
- Implement company fetching from SEC EDGAR
- Create company status tracking system
- Implement pass/fail tracking
- Set up basic error handling

### Deliverables
- ScreeningOrchestrator service (orchestrates 7-stage flow)
- CompanyFetcher service (SEC EDGAR integration)
- CompanyStatusManager (pass/fail tracking)
- ScreeningRun logging system
- Error handling and recovery

### Key Tasks

**1. Screening Orchestrator (Days 1-2)**
```csharp
public class ScreeningOrchestrator
{
    public async Task RunScreeningAsync()
    {
        var companies = await FetchCompaniesAsync();
        
        foreach (var company in companies)
        {
            try
            {
                // Stage 0: Market Cap Pre-Filter
                if (!PassesMarketCapFilter(company)) continue;
                
                // Stage 1: Data Ingestion & Reconciliation
                var financialData = await IngestAndReconcileDataAsync(company);
                if (financialData == null) continue;
                
                // Stage 2-3: MERGED - Calculate ALL metrics + evaluate hard filters
                var metrics = await MergedMetricCalculationAndHardFilterAsync(company, financialData);
                if (!metrics.PassedHardFilters) continue;
                
                // Stages 4-5: Quality & Composite Scoring
                var scores = await ScoreCompanyAsync(company, metrics);
                if (scores.CompositeScore < 75) continue;
                
                // Store passing company
                await StoreQualifiedCompanyAsync(company, metrics, scores);
            }
            catch (Exception ex)
            {
                _logger.LogError($"Error processing {company.Ticker}: {ex.Message}");
                continue; // Continue to next company
            }
        }
    }
}
```

**2. Company Status Management (Days 2-3)**
- Fetch existing qualified companies from database
- Compare new results with previous run
- Categorize: NEW, MAINTAINED, REMOVED

**3. Error Handling (Days 3-4)**
- Try/catch at company level (don't crash entire run)
- Logging of all errors
- Recovery with fallback data
- Graceful degradation

### Best Practices Applied
- ✅ Orchestrator pattern for complex workflows
- ✅ Separation of concerns
- ✅ Robust error handling (continue on error)
- ✅ Detailed logging at each stage
- ✅ Async/await for performance

### Success Criteria
- [x] Orchestrator controls entire 7-stage flow
- [x] Can fetch 500+ companies from SEC
- [x] Pass/fail tracking works correctly
- [x] Error handling prevents crashes
- [x] Logs show what happened at each step
- [x] Can identify NEW/MAINTAINED/REMOVED companies

---

## Phase 3: Data Integration & Reconciliation

### Duration: 2-3 weeks

### Objectives
- Integrate SEC EDGAR API for financial data
- Integrate Yahoo Finance API for stock prices
- Implement fallback data sources (IEX Cloud, Alpha Vantage)
- Create data reconciliation logic
- Implement data validation rules
- Handle API rate limiting and errors

### Deliverables
- SecEdgarService (10-K/10-Q fetching)
- YahooFinanceService (stock price API)
- IexCloudService and AlphaVantageService (fallbacks)
- DataReconciliationService (merge + validate)
- Data validation logic
- Error recovery with fallbacks

### API Keys Required

**All FREE - No Cost**

1. **SEC EDGAR** - Free, no API key needed
   - https://www.sec.gov/edgar
   - Public data endpoint
   - Rate limit: Reasonable (1 request per second)

2. **Yahoo Finance** - Free via unofficial API
   - No API key needed
   - Rate limit: ~2000 requests/hour

3. **IEX Cloud** - Free tier available
   - Sign up: https://iexcloud.io
   - Free tier: 100 messages/month
   - Use for fallback only

4. **Alpha Vantage** - Free tier available
   - Sign up: https://www.alphavantage.co
   - Free tier: 5 requests/minute
   - Use as final fallback

5. **Federal Reserve FRED** - Free, no key needed
   - https://fred.stlouisfed.org
   - Treasury yield data
   - No rate limiting

### Key Tasks

**1. SEC EDGAR Integration (Days 1-5)**
```csharp
public class SecEdgarService
{
    public async Task<FinancialData> FetchCompanyFinancialsAsync(string ticker)
    {
        // Fetch 10-K annual reports
        var filings = await FetchFilingsAsync(ticker, "10-K");
        
        // Parse most recent 10 years of filings
        var financialData = new List<YearlyFinancials>();
        foreach (var filing in filings.Take(10))
        {
            var parsed = ParseFinancialStatements(filing);
            financialData.Add(parsed);
        }
        
        return financialData;
    }
}
```

**2. Multi-Source Data Fetching (Days 5-8)**
- Yahoo Finance: Primary for stock prices
- IEX Cloud: Fallback if Yahoo fails
- Alpha Vantage: Final fallback
- Federal Reserve: Treasury yields

**3. Data Reconciliation (Days 8-12)**
```csharp
public class DataReconciliationService
{
    public async Task<ReconcililedData> ReconcileDataAsync(string ticker)
    {
        // Try primary sources first
        var edgarData = await _edgarService.FetchAsync(ticker);
        var yahooData = await _yahooService.FetchAsync(ticker);
        
        if (yahooData == null)
        {
            yahooData = await _iexService.FetchAsync(ticker); // Fallback
        }
        if (yahooData == null)
        {
            yahooData = await _alphaService.FetchAsync(ticker); // Final fallback
        }
        
        // Validate and merge
        ValidateDataIntegrity(edgarData, yahooData);
        return new ReconcililedData { Financial = edgarData, Market = yahooData };
    }
}
```

**4. Data Validation (Days 12-14)**
- Balance sheet validation (A = L + E)
- Range validation (ROE, D/E ratios)
- Historical consistency checks
- Data age verification

### Best Practices Applied
- ✅ API abstraction (IService interfaces)
- ✅ Multi-source fallback strategy
- ✅ Comprehensive error handling
- ✅ Data validation before use
- ✅ Async HTTP requests with Polly
- ✅ Logging of all API calls

### Success Criteria
- [x] Can fetch financial data for 500+ companies
- [x] Stock prices from Yahoo Finance working
- [x] Fallback sources working correctly
- [x] Data reconciliation merges sources properly
- [x] Validation catches bad data
- [x] Rate limiting handled gracefully
- [x] Error recovery works (don't crash on API failure)

---

## Phase 4: Merged Metric Calculation + Hard Filter Evaluation

### Duration: 2-3 weeks

### ⚠️ CRITICAL PHASE - Key Architectural Change

**This phase implements the MERGED UNIFIED APPROACH:**
- ✅ ALL 40+ metrics calculated for EVERY company
- ✅ NO filtering during calculation phase
- ✅ Hard filters evaluated using COMPLETE dataset
- ✅ Holistic assessment prevents false negatives

### Objectives
- Calculate all 40+ financial metrics from reconciled data
- Perform 10-year trend analysis for each metric
- Implement stability calculations (Coefficient of Variation)
- Evaluate hard filters using complete metrics
- Assess financial strength holistically
- Create detailed audit trail

### Deliverables
- ComprehensiveMetricsCalculator (calculates ALL 40+ metrics)
- TrendAnalysisService (10-year trends)
- StabilityCalculationService (CV, volatility)
- UnifiedHardFilterEvaluationService (holistic assessment)
- MetricsRepository (store metrics in DB)
- Detailed logging of all assessments

### Key Tasks

**1. Metric Calculation Engine (Days 1-5)**

```csharp
public class ComprehensiveMetricsCalculator
{
    /// <summary>
    /// Calculate ALL 40+ metrics for a company
    /// NO FILTERING - all companies get complete metrics
    /// </summary>
    public ComprehensiveMetricsResult CalculateAllMetrics(
        ReconcililedData data,
        int yearsAvailable)
    {
        var result = new ComprehensiveMetricsResult();
        
        // PROFITABILITY METRICS (8 metrics)
        result.GrossMargin = (data.Revenue - data.COGS) / data.Revenue;
        result.OperatingMargin = data.OperatingIncome / data.Revenue;
        result.NetProfitMargin = data.NetIncome / data.Revenue;
        result.EbitdaMargin = data.EBITDA / data.Revenue;
        result.ROA = data.NetIncome / data.TotalAssets;
        result.ROE = data.NetIncome / data.Equity;
        result.ROIC = CalculateROIC(data);
        result.ROCE = CalculateROCE(data);
        
        // FINANCIAL STRENGTH METRICS (6 metrics)
        result.DebtToEquity = data.TotalDebt / data.Equity;
        result.DebtToAssets = data.TotalDebt / data.TotalAssets;
        result.EquityMultiplier = data.TotalAssets / data.Equity;
        result.CurrentRatio = data.CurrentAssets / data.CurrentLiabilities;
        result.QuickRatio = (data.CurrentAssets - data.Inventory) / data.CurrentLiabilities;
        result.InterestCoverage = data.EBIT / data.InterestExpense;
        
        // CASH FLOW METRICS (6 metrics)
        result.OperatingCashFlow = data.OCF;
        result.FreeCashFlow = data.OCF - data.CapEx;
        result.FCFToNIRatio = result.FreeCashFlow / data.NetIncome;
        result.FCFToOCFRatio = result.FreeCashFlow / data.OCF;
        result.CapExPercentRevenue = data.CapEx / data.Revenue;
        result.CapExPercentOCF = data.CapEx / data.OCF;
        
        // GROWTH METRICS (7 metrics)
        result.RevenueGrowthYoY = CalculateYoYGrowth(data.Revenue);
        result.EarningsGrowthYoY = CalculateYoYGrowth(data.NetIncome);
        result.FCFGrowthYoY = CalculateYoYGrowth(result.FreeCashFlow);
        result.RevenueCAGR_10Y = CalculateCAGR(data.RevenueHistory_10Y);
        result.EarningsCAGR_10Y = CalculateCAGR(data.EarningsHistory_10Y);
        result.FCFCAGR_10Y = CalculateCAGR(data.FCFHistory_10Y);
        result.EPS = data.NetIncome / data.SharesOutstanding;
        
        // VALUATION METRICS (8 metrics)
        result.PERatio = data.StockPrice / result.EPS;
        result.PBRatio = data.MarketCap / data.Equity;
        result.PSRatio = data.MarketCap / data.Revenue;
        result.PEGRatio = result.PERatio / (result.EarningsGrowthYoY * 100);
        result.EarningsYield = data.NetIncome / data.MarketCap;
        result.DividendYield = data.AnnualDividend / data.StockPrice;
        result.PayoutRatio = data.DividendsPaid / data.NetIncome;
        result.EVToRevenue = data.EnterpriseValue / data.Revenue;
        
        // MANAGEMENT QUALITY METRICS (6 metrics)
        result.InsiderOwnershipPercent = data.InsiderShares / data.SharesOutstanding;
        result.GoodwillPercentAssets = data.Goodwill / data.TotalAssets;
        result.OneTimeChargesFrequency = CountOneTimeCharges(data.History_10Y);
        result.RetainedEarningsPercent = data.RetainedEarnings / data.Equity;
        result.OperatingExpenseRatio = data.OperatingExpenses / data.Revenue;
        result.AssetTurnover = data.Revenue / data.TotalAssets;
        
        // NO FILTERING HERE - ALL METRICS CALCULATED FOR ALL COMPANIES
        // Even if metrics look bad, they're calculated
        
        return result;
    }
}
```

**2. Trend Analysis (Days 5-8)**
```csharp
public class TrendAnalysisService
{
    public TrendAnalysis Analyze10YearTrend(List<decimal> values)
    {
        return new TrendAnalysis
        {
            Average_10Y = values.Average(),
            Median_10Y = values.OrderBy(x => x).ElementAt(values.Count / 2),
            Trend = DetectTrend(values), // Improving/Stable/Declining
            StdDev = CalculateStandardDeviation(values),
            PercentChange = ((values.Last() - values.First()) / values.First()) * 100
        };
    }
}
```

**3. Stability Calculation (Days 8-10)**
```csharp
public class StabilityCalculationService
{
    public decimal CalculateCoeffientOfVariation(List<decimal> values)
    {
        var mean = values.Average();
        var variance = values.Average(x => (x - mean) * (x - mean));
        var stdDev = (decimal)Math.Sqrt((double)variance);
        
        return (stdDev / mean) * 100; // CV as percentage
    }
    
    // Results:
    // < 10% = Highly stable
    // 10-20% = Stable
    // 20-30% = Moderate volatility
    // > 30% = Highly volatile
}
```

**4. Unified Hard Filter Evaluation (Days 10-15)**

```csharp
public class UnifiedHardFilterEvaluationService
{
    /// <summary>
    /// Evaluate hard filters using COMPLETE metric dataset
    /// Assessment is holistic, not checklist-based
    /// </summary>
    public HardFilterAssessment EvaluateCompany(
        Company company,
        ComprehensiveMetricsResult allMetrics)
    {
        var assessment = new HardFilterAssessment { Company = company };
        
        // Assess each critical area using complete metrics context
        assessment.ReturnOnCapitalAssessment = AssessROE(
            allMetrics.ROE_10YAvg,
            allMetrics.OperatingMargin_Trend,
            allMetrics.RevenueGrowth_Trend,
            allMetrics
        );
        
        assessment.ProfitabilityAssessment = AssessProfitability(
            allMetrics.NetProfitMargin_10YAvg,
            allMetrics.OperatingMargin_10YAvg,
            allMetrics.MarginStability_CV,
            allMetrics
        );
        
        assessment.FinancialStabilityAssessment = AssessFinancialStability(
            allMetrics.DebtToEquity,
            allMetrics.DebtTrend_10Y,
            allMetrics.EquityTrend_10Y,
            allMetrics.InterestCoverage,
            allMetrics
        );
        
        assessment.CashFlowAssessment = AssessCashFlow(
            allMetrics.OCFTrend_10Y,
            allMetrics.FCFTrend_10Y,
            allMetrics.FCFToNI_Ratio,
            allMetrics.CapitalEfficiency,
            allMetrics
        );
        
        assessment.OverallScore = CalculateHolisticScore(
            assessment.ReturnOnCapitalAssessment,
            assessment.ProfitabilityAssessment,
            assessment.FinancialStabilityAssessment,
            assessment.CashFlowAssessment
        );
        
        assessment.PassesHardFilters = assessment.OverallScore >= RequiredThreshold;
        assessment.DetailedReasoning = GenerateDetailedExplanation(assessment, allMetrics);
        
        return assessment;
    }
}
```

### Best Practices Applied
- ✅ Separation of concerns (metrics, trends, stability)
- ✅ Comprehensive metric coverage (40+ metrics)
- ✅ Statistical analysis (CV, CAGR, trends)
- ✅ Holistic assessment (context-aware evaluation)
- ✅ Complete audit trail (all metrics logged)
- ✅ No premature rejections (all companies get full analysis)

### Success Criteria
- [x] All 40+ metrics calculate correctly
- [x] 10-year trend analysis works accurately
- [x] Stability (CV) calculations correct
- [x] Hard filters evaluate using complete data
- [x] Holistic assessment prevents false negatives
- [x] Detailed logs show reasoning
- [x] No companies rejected without full analysis

---

## Phase 5: Quality Assessment & Composite Scoring

### Duration: 1-2 weeks

### Objectives
- Implement 4 quality pillars (ROC, Profitability, Cash Flow, Business Quality)
- Create quality scoring with maximum caps per pillar
- Implement valuation scoring (P/E, P/B, PEG, etc.)
- Implement management quality scoring
- Calculate final composite score
- Create detailed scoring audit trail

### Deliverables
- QualityAssessmentService (4 pillars, 0-100 score)
- ValuationScoringService (max 35 points)
- ManagementQualityScoringService (max 25 points)
- CompositeScoreCalculator (final ranking)
- Scoring details logger

### Key Tasks

**1. Quality Pillars (Days 1-4)**

```csharp
public class QualityAssessmentService
{
    public QualityScoreResult CalculateQualityScore(ComprehensiveMetricsResult metrics)
    {
        var result = new QualityScoreResult();
        
        // PILLAR 1: Return on Capital (MAX 30 points)
        var roe = metrics.ROE_10YAvg;
        decimal pillar1 = 0;
        if (roe >= 30) pillar1 += 20; // ROE component
        else if (roe >= 25) pillar1 += 15;
        else if (roe >= 20) pillar1 += 10;
        else if (roe >= 15) pillar1 += 5;
        
        var roic = metrics.ROIC_10YAvg;
        if (roic >= 18) pillar1 += 10; // ROIC component
        else if (roic >= 15) pillar1 += 8;
        else if (roic >= 12) pillar1 += 5;
        
        pillar1 = Math.Min(pillar1, 30m); // CAP AT 30
        result.Pillar1_ReturnOnCapital = pillar1;
        
        // PILLAR 2: Profitability (MAX 30 points)
        decimal pillar2 = 0;
        var netMargin = metrics.NetProfitMargin_10YAvg;
        if (netMargin >= 20) pillar2 += 15;
        else if (netMargin >= 15) pillar2 += 10;
        else if (netMargin >= 10) pillar2 += 5;
        
        var opMargin = metrics.OperatingMargin_10YAvg;
        if (opMargin >= 20) pillar2 += 10;
        else if (opMargin >= 15) pillar2 += 5;
        
        if (metrics.MarginStability_CV < 10) pillar2 += 5; // Bonus
        
        pillar2 = Math.Min(pillar2, 30m); // CAP AT 30
        result.Pillar2_Profitability = pillar2;
        
        // PILLAR 3: Cash Flow (MAX 20 points)
        decimal pillar3 = 0;
        if (metrics.FreeCashFlow > 0) pillar3 += 5;
        
        var fcfPercent = metrics.FreeCashFlow / metrics.Revenue;
        if (fcfPercent >= 0.10m) pillar3 += 10;
        else if (fcfPercent >= 0.05m) pillar3 += 5;
        
        if (metrics.OCFToNI_Ratio > 1) pillar3 += 5;
        if (metrics.FCFCAGR_10Y > 5) pillar3 += 5;
        
        pillar3 = Math.Min(pillar3, 20m); // CAP AT 20
        result.Pillar3_CashFlow = pillar3;
        
        // PILLAR 4: Business Quality (MAX 10 points)
        decimal pillar4 = 0;
        if (HasEconomicMoat(metrics)) pillar4 += 3;
        if (metrics.InsiderOwnershipPercent > 5) pillar4 += 2;
        if (metrics.GoodwillPercentAssets < 20) pillar4 += 3;
        if (metrics.RevenueGrowthStability_CV < 15) pillar4 += 2;
        
        pillar4 = Math.Min(pillar4, 10m); // CAP AT 10
        result.Pillar4_BusinessQuality = pillar4;
        
        result.QualityScore = Math.Min(pillar1 + pillar2 + pillar3 + pillar4, 100m);
        
        return result;
    }
}
```

**2. Valuation Scoring (Days 4-6)**
```csharp
public class ValuationScoringService
{
    public ValuationScoreResult CalculateValuationScore(ComprehensiveMetricsResult metrics)
    {
        var result = new ValuationScoreResult();
        decimal score = 0;
        
        // P/E Component (max 12)
        if (metrics.PERatio < 15) score += 12;
        else if (metrics.PERatio < 20) score += 10;
        else if (metrics.PERatio < 25) score += 8;
        else if (metrics.PERatio < 35) score += 4;
        else score += 0;
        
        // PEG Component (max 10)
        if (metrics.PEGRatio < 1.0m) score += 10;
        else if (metrics.PEGRatio < 1.5m) score += 8;
        else if (metrics.PEGRatio < 2.0m) score += 4;
        else score += 0;
        
        // P/B Component (max 8)
        if (metrics.PBRatio < 1.5m) score += 8;
        else if (metrics.PBRatio < 2.5m) score += 6;
        else if (metrics.PBRatio < 4.0m) score += 2;
        else score += 0;
        
        // Dividend Component (max 5)
        if (metrics.DividendYield > 3) score += 5;
        else if (metrics.DividendYield > 2) score += 3;
        
        result.ValuationScore = Math.Min(score, 35m); // CAP AT 35
        return result;
    }
}
```

**3. Management Quality Scoring (Days 6-7)**
```csharp
public class ManagementQualityScoringService
{
    public ManagementScoreResult CalculateManagementScore(ComprehensiveMetricsResult metrics)
    {
        var result = new ManagementScoreResult();
        decimal score = 0;
        
        // Insider Ownership (max 10)
        if (metrics.InsiderOwnershipPercent > 15) score += 10;
        else if (metrics.InsiderOwnershipPercent > 10) score += 8;
        else if (metrics.InsiderOwnershipPercent > 5) score += 5;
        
        // Goodwill (max 8)
        if (metrics.GoodwillPercentAssets < 10) score += 8;
        else if (metrics.GoodwillPercentAssets < 20) score += 5;
        else if (metrics.GoodwillPercentAssets < 50) score += 2;
        
        // Accounting Quality (max 7)
        if (metrics.OneTimeChargesFrequency < 20) score += 7;
        else if (metrics.OneTimeChargesFrequency < 40) score += 4;
        
        result.ManagementScore = Math.Min(score, 25m); // CAP AT 25
        return result;
    }
}
```

**4. Composite Score (Days 7-8)**
```csharp
public class CompositeScoreCalculator
{
    public CompositeScoreResult CalculateCompositeScore(
        QualityScoreResult quality,
        ValuationScoreResult valuation,
        ManagementScoreResult management)
    {
        var compositeScore = 
            (quality.QualityScore * 0.40m) +
            (valuation.ValuationScore * 0.35m) +
            (management.ManagementScore * 0.25m);
        
        return new CompositeScoreResult
        {
            CompositeScore = compositeScore,
            Recommendation = compositeScore >= 75 ? "PASS" : "FAIL",
            QualityContribution = quality.QualityScore * 0.40m,
            ValuationContribution = valuation.ValuationScore * 0.35m,
            ManagementContribution = management.ManagementScore * 0.25m
        };
    }
}
```

### Best Practices Applied
- ✅ Pillar-based scoring with maximum caps
- ✅ Detailed component scoring
- ✅ Capping prevents pillar inflation
- ✅ Composite weighting (40/35/25)
- ✅ Clear passing threshold (75)
- ✅ Complete scoring transparency

### Success Criteria
- [x] All 4 quality pillars calculate correctly
- [x] Pillar caps enforced (30/30/20/10)
- [x] Valuation scoring (max 35) works
- [x] Management scoring (max 25) works
- [x] Composite score calculation accurate
- [x] Threshold logic works (>= 75 = PASS)
- [x] Detailed scoring logs created

---

## Phase 6: Results & Notifications

### Duration: 1-2 weeks

### Objectives
- Generate email reports with detailed metrics
- Create CSV exports for manual review
- Implement console output summaries
- Track newly qualified vs maintained vs removed companies
- Store all results in database

### Deliverables
- EmailNotificationService (HTML formatted)
- ScreeningResultsExporter (CSV)
- ConsoleReporter (summary output)
- ResultsProcessor (NEW/MAINTAINED/REMOVED tracking)
- Database storage logic

### Key Tasks

**1. Email Notification (Days 1-4)**
- HTML formatted table with all metrics
- Color-coded scores (green/yellow/red)
- Summary statistics
- Newly qualified highlighted
- Removed companies listed with reasons

**2. CSV Export (Days 4-6)**
- All 40+ metrics exported
- All companies (pass and fail) included as audit trail
- Sortable by score, ticker, industry
- Date stamped

**3. Database Storage (Days 6-8)**
- Insert newly qualified companies
- Update scores for maintained
- Mark removed as inactive
- Store all metrics
- Log screening run details

### Success Criteria
- [x] Email sends without errors
- [x] CSV contains all data
- [x] Database updates correctly
- [x] Console output shows summary
- [x] NEW/MAINTAINED/REMOVED tracked accurately

---

## Phase 7: GitHub Actions Automation

### Duration: 1 week

### Objectives
- Create quarterly screening workflow
- Create weekly stock price update workflow
- Set up environment secrets
- Configure schedule triggers
- Implement manual trigger option

### Deliverables
- `.github/workflows/quarterly-screening.yml`
- `.github/workflows/weekly-stock-update.yml`
- GitHub Secrets configured
- Documentation for manual triggers

### Key Tasks

**1. Quarterly Screening Workflow (Days 1-3)**
```yaml
name: Quarterly Stock Screening

on:
  schedule:
    - cron: '0 14 15 3,6,9,12 *'  # 15th of Mar/Jun/Sep/Dec at 2 PM UTC
  workflow_dispatch:

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
        ports:
          - 3306:3306

    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-dotnet@v3
        with:
          dotnet-version: 8.0.x
      
      - name: Restore dependencies
        run: dotnet restore

      - name: Build
        run: dotnet build --configuration Release

      - name: Run screening
        run: dotnet run --project src/StockMarketScreener.Console
        env:
          ConnectionStrings__DefaultConnection: ${{ secrets.DB_CONNECTION }}
          Email__SmtpServer: ${{ secrets.SMTP_SERVER }}
          Email__SenderEmail: ${{ secrets.SENDER_EMAIL }}
          Email__SenderPassword: ${{ secrets.SENDER_PASSWORD }}

      - name: Upload results
        uses: actions/upload-artifact@v3
        with:
          name: screening-results
          path: output/

      - name: Commit results
        run: |
          git config user.name "GitHub Actions"
          git config user.email "actions@github.com"
          git add output/
          git commit -m "Screening results: $(date)"
          git push
```

**2. Weekly Stock Price Update (Days 3-4)**
```yaml
name: Weekly Stock Price Update

on:
  schedule:
    - cron: '0 20 * * MON'  # Every Monday at 8 PM UTC
  workflow_dispatch:

jobs:
  update:
    runs-on: ubuntu-latest
    # Similar setup as quarterly, but runs stock price update job
```

**3. GitHub Secrets Setup (Days 4-5)**
- Configure SMTP credentials
- Configure database connection string
- Configure API keys
- Test connections

### Success Criteria
- [x] Quarterly workflow runs on schedule
- [x] Weekly workflow runs on schedule
- [x] Manual trigger works
- [x] Environment variables set correctly
- [x] Database connection works in Actions
- [x] Email sends from Actions
- [x] Results uploaded as artifacts

---

## Phase 8: Testing & Quality Assurance

### Duration: 2-3 weeks

### Objectives
- Create comprehensive unit tests
- Create integration tests with test database
- Create end-to-end screening test
- Achieve 80%+ code coverage
- Test error handling and recovery
- Performance validation

### Deliverables
- Unit test suite (40+ tests)
- Integration test suite (10+ tests)
- End-to-end test (1+ test)
- Code coverage report
- Performance test results

### Key Tasks

**1. Unit Tests (Days 1-7)**
- Metric calculation tests
- Scoring logic tests
- Hard filter evaluation tests
- Data validation tests
- Utility function tests

**2. Integration Tests (Days 7-12)**
- Database operations
- API integrations (mocked)
- End-to-end screening flow
- Email generation
- CSV export

**3. Test Data (Days 12-14)**
- 10 well-known companies
- Real historical data
- Companies that should PASS
- Companies that should FAIL
- Edge cases

### Success Criteria
- [x] 80%+ code coverage
- [x] All tests pass
- [x] Edge cases handled
- [x] Error scenarios tested
- [x] Performance acceptable (45-60 min for 500 companies)

---

## Phase 9: UI Development (Post-Launch)

### Duration: 3-4 weeks (AFTER Core Completion)

### Status: Planned for after core functionality proven

### Objectives (Future)
- Web dashboard for results viewing
- Interactive metrics explorer
- Historical trend analysis
- Company comparison tool
- Watchlist management

### Technology Stack (Future)
- **Frontend:** React or Blazor
- **Backend:** ASP.NET Core (extend current console)
- **Database:** Same MySQL (shared)
- **Deployment:** Azure App Service or AWS

### Out of Scope (Phase 9)
- Real-time data updates
- Machine learning analysis
- Social media integration
- Complex portfolio optimization

---

## Dependencies & Prerequisites

### Required Before Starting

#### API Keys (All Free - No Cost)

1. **SEC EDGAR** ✓ Free
   - No API key needed
   - Public endpoint
   - Rate: 1 req/sec

2. **Yahoo Finance** ✓ Free
   - No API key needed
   - Unofficial API
   - Rate: ~2000 reqs/hour

3. **IEX Cloud** (Optional - fallback)
   - Free tier: 100 msgs/month
   - Sign up: https://iexcloud.io

4. **Alpha Vantage** (Optional - fallback)
   - Free tier: 5 reqs/min
   - Sign up: https://www.alphavantage.co

5. **Federal Reserve FRED** ✓ Free
   - No API key needed
   - Treasury yields

#### Software Requirements

- ✅ .NET 8.0 SDK (free, Windows/Mac/Linux)
- ✅ Visual Studio 2022 Community (free)
- ✅ MySQL 8.0 (free, Docker recommended)
- ✅ Git (free)
- ✅ Docker (free Community Edition, optional)

#### Knowledge Prerequisites

- ✅ C# (.NET experience)
- ✅ SQL basics
- ✅ REST API integration
- ✅ Financial concepts (ratios, margins, cash flow)

### Setup Checklist

- [ ] Clone repository
- [ ] Install .NET 8.0 SDK
- [ ] Install MySQL 8.0
- [ ] Create database: `stockscreener`
- [ ] Run EF Core migrations
- [ ] Configure appsettings.json
- [ ] Test database connection
- [ ] Verify API endpoints (EDGAR, Yahoo, IEX, Alpha)
- [ ] Configure GitHub Secrets for Actions
- [ ] Run test suite
- [ ] Verify screening locally

---

## Risk Management

### Identified Risks & Mitigation

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|-----------|
| API rate limiting | Medium | High | Multi-source fallbacks, caching, retry logic |
| Data quality issues | Medium | High | Validation layer, reconciliation, logging |
| Calculation errors | Low | Critical | Unit tests, spot checks, detailed logging |
| Database connection loss | Low | High | Connection pooling, retry logic, error handling |
| Email delivery failure | Low | Medium | Logging, manual fallback, notification option |
| Merge conflicts (git) | Low | Medium | Feature branches, frequent commits |
| Performance degradation | Low | Medium | Profiling, parallel processing, caching |
| Market data discrepancies | Low | Medium | Multi-source validation, logging |

### Mitigation Strategies

**API Rate Limiting:**
- ✅ Implement exponential backoff
- ✅ Queue requests with delays
- ✅ Use multiple API sources
- ✅ Cache results locally
- ✅ Log all API calls

**Data Quality:**
- ✅ Validation on ingestion
- ✅ Balance sheet checking
- ✅ Range validation
- ✅ Consistency checks
- ✅ Manual review option

**Calculation Accuracy:**
- ✅ Comprehensive unit tests
- ✅ Spot checks with known data
- ✅ Detailed audit logs
- ✅ Code reviews
- ✅ Manual verification

---

## Success Metrics & Validation

### Phase Completion Criteria

| Phase | Completion Criteria |
|-------|---|
| 1 | Project structure, DB schema, DI setup complete |
| 2 | Orchestrator controls entire flow, status tracking works |
| 3 | Data fetching from all sources working, reconciliation complete |
| 4 | ALL 40+ metrics calculated, hard filters evaluated holistically |
| 5 | Quality/valuation/management scoring complete, composite score working |
| 6 | Email, CSV, database storage all working |
| 7 | GitHub Actions workflows running successfully |
| 8 | 80%+ test coverage, all tests passing |
| 9 | UI dashboard functional (post-launch) |

### Project Success Criteria

**Functional Success:**
- ✅ System screens 500+ companies in 45-60 minutes
- ✅ Qualifies 40-60 companies (9-12%)
- ✅ All 40+ metrics calculated for all companies
- ✅ Hard filters evaluated holistically
- ✅ Quality scores accurate
- ✅ Composite scores >= 75 = PASS
- ✅ Email reports detailed and correct
- ✅ Database updates accurately
- ✅ CSV exports all data

**Operational Success:**
- ✅ GitHub Actions runs automatically
- ✅ Zero crashes during screening
- ✅ Detailed audit logs
- ✅ Error recovery works
- ✅ Email notifications sent
- ✅ Results reproducible
- ✅ Performance acceptable

**Investment Success:**
- ✅ Qualified companies align with Buffett/Munger principles
- ✅ High ROE (> 15%)
- ✅ Strong profitability (margins > 10%)
- ✅ Solid balance sheets (D/E < 0.50)
- ✅ Positive cash flow (10 years)
- ✅ Identifiable competitive advantage
- ✅ Reasonable valuations

### Validation Methods

**Automated Validation:**
- Unit tests (80%+ coverage)
- Integration tests (database, APIs)
- Data validation rules
- Calculation verification
- Scoring logic tests

**Manual Validation:**
- Spot check 10 companies against original data
- Verify email formatting and data
- Review CSV exports
- Test error scenarios
- Performance profiling

**Investment Validation:**
- Compare qualified companies to known Buffett holdings
- Verify metrics match official financial data
- Check industry representation
- Validate scoring against selection criteria
- Historical performance tracking (future)

---

## Timeline Estimate

### Optimistic Scenario: 9 weeks
```
Week 1:   Phase 1 (Foundation)
Week 2:   Phase 2 (Core Engine)
Weeks 3-4: Phase 3 (Data Integration)
Weeks 5-6: Phase 4 (Metric Calculation - MERGED)
Week 7:   Phase 5 (Quality Scoring)
Week 8:   Phase 6 (Results) + Phase 7 (GitHub Actions)
Weeks 9+: Phase 8 (Testing)
```

### Realistic Scenario: 10-11 weeks
```
Weeks 1-2: Phase 1 (Foundation)
Week 3:    Phase 2 (Core Engine)
Weeks 4-5: Phase 3 (Data Integration)
Weeks 6-7: Phase 4 (Metric Calculation - MERGED)
Week 8:    Phase 5 (Quality Scoring)
Weeks 9:   Phase 6 (Results) + Phase 7 (GitHub Actions)
Weeks 10-11: Phase 8 (Testing & Polish)
```

### Buffer for Issues: 2-3 weeks
- API integration challenges
- Performance optimization
- Bug fixes and refinements
- Integration testing issues

**Total Realistic Timeline: 10-14 weeks (2.5-3.5 months)**

---

## Next Steps

### Immediate Actions (This Week)
1. [ ] Review all 4 documents (Requirements, Architecture, Data Flow, Roadmap)
2. [ ] Verify merged approach understanding
3. [ ] Set up local development environment
4. [ ] Clone GitHub repository
5. [ ] Create GitHub Project board

### Week 1 Actions (Phase 1)
1. [ ] Create project structure
2. [ ] Set up MySQL database
3. [ ] Configure Entity Framework Core
4. [ ] Implement appsettings.json
5. [ ] Test database connection

### Pre-Development Verification
1. [ ] Understand merged metric calculation approach
2. [ ] Review hard filter evaluation logic
3. [ ] Study quality pillar scoring system
4. [ ] Review composite score calculation
5. [ ] Understand valuation and management scoring

### API Setup (Before Phase 3)
1. [ ] Test SEC EDGAR endpoint
2. [ ] Test Yahoo Finance API
3. [ ] Sign up for IEX Cloud (fallback)
4. [ ] Sign up for Alpha Vantage (fallback)
5. [ ] Test Federal Reserve FRED API

### GitHub Setup (Before Phase 7)
1. [ ] Create GitHub Secrets
2. [ ] Configure SMTP credentials
3. [ ] Test email sending
4. [ ] Set up Actions workflows
5. [ ] Test manual triggers

---

## Appendices

### A. Required API Keys & Setup

**SEC EDGAR - No Key Needed**
```
Endpoint: https://www.sec.gov/cgi-bin/browse-edgar
Usage: Fetch 10-K and 10-Q filings
No authentication required
Rate limit: 1 request/second
```

**Yahoo Finance - No Key Needed**
```
Endpoint: https://query1.finance.yahoo.com/v10/finance/quoteSummary/
Usage: Stock prices, market cap, dividends
No authentication required
Rate limit: ~2000 requests/hour
```

**IEX Cloud - Free Tier**
```
Sign up: https://iexcloud.io
Free tier: 100 messages/month
API Key: Required in headers
Rate limit: 100 calls/5 minutes
Use as fallback if Yahoo fails
```

**Alpha Vantage - Free Tier**
```
Sign up: https://www.alphavantage.co
Free tier: 5 calls/minute
API Key: Required as parameter
Rate limit: 5 calls/minute
Use as final fallback
```

**Federal Reserve FRED - No Key Needed**
```
Endpoint: https://fred.stlouisfed.org/
Usage: 10-year treasury yield
No authentication required
No rate limiting
```

### B. GitHub Actions Setup Guide

**1. Create GitHub Secrets:**
```
Settings → Secrets and variables → Actions → New repository secret

Add:
- DB_CONNECTION: "Server=localhost;Uid=root;..."
- SMTP_SERVER: "smtp.gmail.com"
- SMTP_PORT: "587"
- SENDER_EMAIL: "your-email@gmail.com"
- SENDER_PASSWORD: "your-app-password"
- RECIPIENT_EMAIL: "your-email@gmail.com"
```

**2. Create Workflow Files:**
- `.github/workflows/quarterly-screening.yml`
- `.github/workflows/weekly-stock-update.yml`

**3. Test Locally:**
```bash
# Use GitHub CLI to test locally
gh workflow run quarterly-screening.yml
```

### C. MySQL Setup Script

```sql
-- Create database
CREATE DATABASE stockscreener;
USE stockscreener;

-- Create tables (EF Core migrations will handle this)
-- Just verify schema after running migrations

-- Check connection
SELECT 1;

-- Verify tables exist
SHOW TABLES;
```

### D. Development Environment Checklist

- [ ] .NET 8.0 SDK installed
- [ ] Visual Studio 2022 Community installed
- [ ] MySQL 8.0 running (Docker or native)
- [ ] Git installed and configured
- [ ] GitHub account set up
- [ ] Repository cloned locally
- [ ] appsettings.json configured
- [ ] Database connection verified
- [ ] Project compiles without errors
- [ ] Unit test project created
- [ ] First test runs successfully

### E. Testing Strategy Document

**Unit Tests (40+ tests)**
- Metric calculation (ROE, margins, cash flow, etc.)
- Scoring logic (pillars, composite)
- Hard filter evaluation
- Data validation
- Utility functions

**Integration Tests (10+ tests)**
- Database CRUD operations
- API mocking and fallbacks
- End-to-end screening (10 test companies)
- Email generation
- CSV export

**Test Data (10 Companies)**
1. MSFT - Should PASS
2. AAPL - Should PASS
3. JNJ - Should PASS
4. KO - Should PASS
5. JPM - Should PASS
6. TSLA - Should FAIL
7. AMD - Should FAIL
8. SNAP - Should FAIL
9. UBER - Should FAIL
10. GOOG - Should PASS

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-03-13 | ThommCroft | Initial project roadmap |
| 1.5 | 2026-03-14 | ThommCroft | Added Phase numbering, updated phase durations, added risk management |
| 2.0 | 2026-03-14 | ThommCroft | **CURRENT** - Integrated MERGED APPROACH throughout; separated Metric Calculation into Phase 4 with detailed merged explanation; added unified hard filter evaluation; updated Phase numbering to 9 phases total; Phase 4 emphasizes "ALL 40+ metrics before ANY filtering"; Phase 5 focuses on scoring of passing companies; updated timeline estimates for merged approach; emphasized holistic assessment prevents false negatives |
