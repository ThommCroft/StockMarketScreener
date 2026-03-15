# Stock Market Screener - Comprehensive Sequence Diagrams

**Version:** 1.0  
**Date:** 2026-03-15  
**Author:** ThommCroft  
**Purpose:** Detailed sequence diagrams showing all major workflows

---

## Table of Contents

1. [Complete Quarterly Screening Flow](#complete-quarterly-screening-flow)
2. [Stage 0: Market Cap Pre-Filter](#stage-0-market-cap-pre-filter)
3. [Stage 1: Data Ingestion & Reconciliation](#stage-1-data-ingestion--reconciliation)
4. [Stages 2-3: Metric Calculation](#stages-2-3-metric-calculation)
5. [Stage 4: Composite Scoring](#stage-4-composite-scoring)
6. [Stage 5: Pass/Fail Gate](#stage-5-passfail-gate)
7. [Stage 6: Valuation Methods](#stage-6-valuation-methods)
8. [Stage 7: Results Reporting](#stage-7-results-reporting)

---

## Complete Quarterly Screening Flow

```mermaid
sequenceDiagram
    actor User
    participant Program
    participant ScreeningOrchestrator
    participant ScreeningService
    participant DataFetchService
    participant MetricCalcService
    participant ValuationService
    participant ReportingService
    participant MySQL

    User->>Program: ExecuteQuarterlyScreeningAsync()
    activate Program
    
    Program->>ScreeningOrchestrator: new ScreeningOrchestrator()
    activate ScreeningOrchestrator
    
    Note over ScreeningOrchestrator: STAGE 0: PRE-FILTER
    ScreeningOrchestrator->>ScreeningService: ExecuteStage0_PreFilterAsync()
    activate ScreeningService
    ScreeningService->>ScreeningService: Filter by MarketCap > $300M
    ScreeningService-->>ScreeningOrchestrator: List<Company> (filtered)
    deactivate ScreeningService
    
    Note over ScreeningOrchestrator: STAGE 1: DATA INGESTION
    ScreeningOrchestrator->>DataFetchService: FetchHistoricalFinancialDataAsync()
    activate DataFetchService
    DataFetchService->>DataFetchService: Try SEC EDGAR → Yahoo → IEX → Alpha Vantage
    DataFetchService-->>ScreeningOrchestrator: List<FinancialData>
    deactivate DataFetchService
    
    Note over ScreeningOrchestrator: STAGES 2-3: METRICS & FILTERS
    ScreeningOrchestrator->>MetricCalcService: CalculateMetrics()
    activate MetricCalcService
    loop For each Company
        MetricCalcService->>MetricCalcService: Calculate 40+ metrics
        MetricCalcService->>MetricCalcService: Apply hard filters
    end
    MetricCalcService-->>ScreeningOrchestrator: List<InvestmentMetrics>
    deactivate MetricCalcService
    
    Note over ScreeningOrchestrator: STAGE 4: COMPOSITE SCORING
    ScreeningOrchestrator->>ScreeningService: CalculateStage4Scores()
    activate ScreeningService
    ScreeningService->>ScreeningService: (Quality×40%) + (Valuation×35%) + (Management×25%)
    ScreeningService-->>ScreeningOrchestrator: List<ScreeningResult>
    deactivate ScreeningService
    
    Note over ScreeningOrchestrator: STAGE 5: PASS/FAIL GATE
    ScreeningOrchestrator->>ScreeningService: ProcessStage5ResultsAsync()
    activate ScreeningService
    loop For each Company
        alt CompositeScore >= 75
            ScreeningService->>ScreeningService: PASS → Stage 6
        else CompositeScore < 75
            ScreeningService->>ScreeningService: FAIL → Discard
        end
    end
    ScreeningService-->>ScreeningOrchestrator: List<Qualified>
    deactivate ScreeningService
    
    Note over ScreeningOrchestrator: STAGE 6: VALUATION (5 Methods)
    ScreeningOrchestrator->>ValuationService: CalculateValuationsAsync()
    activate ValuationService
    ValuationService->>ValuationService: DCF (40%), DDM (15%), Multiples (20%)
    ValuationService->>ValuationService: Buffett (15%), Munger (10%)
    ValuationService->>ValuationService: Calculate Consensus & MOS
    ValuationService-->>ScreeningOrchestrator: List<CompanyValuation>
    deactivate ValuationService
    
    Note over ScreeningOrchestrator: STAGE 7: REPORTING
    ScreeningOrchestrator->>ReportingService: GenerateAndSendAsync()
    activate ReportingService
    ReportingService->>ReportingService: Email + GitHub + CSV + Database
    ReportingService->>MySQL: Store CompanyValuations
    activate MySQL
    MySQL-->>ReportingService: Stored
    deactivate MySQL
    ReportingService-->>ScreeningOrchestrator: Complete
    deactivate ReportingService
    
    ScreeningOrchestrator-->>Program: ScreeningRunResult
    deactivate ScreeningOrchestrator
    
    Program-->>User: Screening Complete!
    deactivate Program
```

## 2. Stage 0: Market Cap Pre-Filter

```mermaid
sequenceDiagram
    participant ScreeningOrchestrator
    participant Stage0PreFilter
    participant Company

    ScreeningOrchestrator->>Stage0PreFilter: Filter(companies: List<Company>)
    activate Stage0PreFilter
    
    Stage0PreFilter->>Stage0PreFilter: Initialize results list
    
    loop For each Company
        Stage0PreFilter->>Company: GetMarketCapBillions()
        activate Company
        Company-->>Stage0PreFilter: decimal marketCap
        deactivate Company
        
        alt marketCap > 0.3 (> $300M)
            Stage0PreFilter->>Stage0PreFilter: ✓ PASS - Add to results
            Note over Stage0PreFilter: ~85-90% pass rate
        else marketCap <= 0.3
            Stage0PreFilter->>Stage0PreFilter: ✗ FAIL - Skip
            Note over Stage0PreFilter: ~10-15% rejected (saves API calls)
        end
    end
    
    Stage0PreFilter-->>ScreeningOrchestrator: List<Company> (filtered)
    Note over Stage0PreFilter: Ready for Stage 1: Data Ingestion
    deactivate Stage0PreFilter
```

## 3 . Stage 1: Data Ingestion & Reconciliation

```mermaid
sequenceDiagram
    participant ScreeningOrchestrator
    participant DataFetchService
    participant SecEdgarClient
    participant YahooFinanceClient
    participant IExCloudClient
    participant ValidationHelper

    ScreeningOrchestrator->>DataFetchService: FetchHistoricalFinancialDataAsync(ticker, 10)
    activate DataFetchService
    
    DataFetchService->>SecEdgarClient: Get10K(ticker)
    activate SecEdgarClient
    SecEdgarClient->>SecEdgarClient: Query SEC EDGAR API
    alt Data found
        SecEdgarClient-->>DataFetchService: FinancialData (10 years) ✓
        Note over SecEdgarClient: Primary source success
    else Data failed
        SecEdgarClient-->>DataFetchService: null
        Note over SecEdgarClient: Try fallback #1
    end
    deactivate SecEdgarClient
    
    alt SEC EDGAR success
        DataFetchService->>DataFetchService: Use SEC data
    else Try Yahoo Finance
        DataFetchService->>YahooFinanceClient: GetHistoricalData(ticker)
        activate YahooFinanceClient
        YahooFinanceClient-->>DataFetchService: FinancialData ✓
        deactivate YahooFinanceClient
    end
    
    alt Still no data
        DataFetchService->>IExCloudClient: GetHistoricalData(ticker)
        activate IExCloudClient
        IExCloudClient-->>DataFetchService: FinancialData ✓
        deactivate IExCloudClient
    end
    
    DataFetchService->>DataFetchService: Fetch current stock price
    
    DataFetchService->>ValidationHelper: ValidateData()
    activate ValidationHelper
    alt Data valid
        ValidationHelper-->>DataFetchService: valid = true ✓
        Note over ValidationHelper: Age < 1 year, History >= 5 years, Complete
    else Data invalid
        ValidationHelper-->>DataFetchService: valid = false ✗
        Note over ValidationHelper: Fails validation, company rejected
    end
    deactivate ValidationHelper
    
    DataFetchService-->>ScreeningOrchestrator: List<FinancialData> (validated)
    Note over DataFetchService: 10-year history + current price ready
    deactivate DataFetchService
```

## 4. Stages 2-3: Metric Calculation & Hard Filters

```mermaid
sequenceDiagram
    participant ScreeningOrchestrator
    participant MetricCalculationService
    participant ReturnOnCapitalCalculator
    participant FinancialStrengthCalculator
    participant ProfitabilityCalculator
    participant CashFlowCalculator
    participant ValidationHelper

    ScreeningOrchestrator->>MetricCalculationService: CalculateMetrics(company, data)
    activate MetricCalculationService
    
    Note over MetricCalculationService: CATEGORY 1: RETURN ON CAPITAL
    MetricCalculationService->>ReturnOnCapitalCalculator: CalculateROE()
    activate ReturnOnCapitalCalculator
    ReturnOnCapitalCalculator->>ReturnOnCapitalCalculator: ROE = NI / Equity
    ReturnOnCapitalCalculator-->>MetricCalculationService: decimal ROE ✓
    deactivate ReturnOnCapitalCalculator
    
    Note over MetricCalculationService: CATEGORY 2: FINANCIAL STRENGTH
    MetricCalculationService->>FinancialStrengthCalculator: CalculateDebtToEquity()
    activate FinancialStrengthCalculator
    FinancialStrengthCalculator->>FinancialStrengthCalculator: D/E = Debt / Equity
    alt D/E < 1.5
        FinancialStrengthCalculator-->>MetricCalculationService: PASS ✓
    else D/E >= 1.5
        FinancialStrengthCalculator-->>MetricCalculationService: FLAG ⚠
        Note over FinancialStrengthCalculator: Hard Filter: Excessive leverage
    end
    deactivate FinancialStrengthCalculator
    
    Note over MetricCalculationService: CATEGORY 3: PROFITABILITY
    MetricCalculationService->>ProfitabilityCalculator: CalculateNetMargin()
    activate ProfitabilityCalculator
    ProfitabilityCalculator->>ProfitabilityCalculator: NM = NI / Revenue
    ProfitabilityCalculator-->>MetricCalculationService: decimal NetMargin ✓
    deactivate ProfitabilityCalculator
    
    Note over MetricCalculationService: CATEGORY 4: CASH FLOW QUALITY
    MetricCalculationService->>CashFlowCalculator: CalculateFreeCashFlow()
    activate CashFlowCalculator
    CashFlowCalculator->>CashFlowCalculator: FCF = OCF - CapEx
    CashFlowCalculator-->>MetricCalculationService: decimal FCF ✓
    deactivate CashFlowCalculator
    
    Note over MetricCalculationService: VALIDATION
    MetricCalculationService->>ValidationHelper: ValidateMetrics()
    activate ValidationHelper
    alt All 40+ metrics valid
        ValidationHelper-->>MetricCalculationService: valid = true ✓
        Note over ValidationHelper: All metrics calculated, no nulls, reasonable ranges
    else Metrics invalid
        ValidationHelper-->>MetricCalculationService: valid = false ✗
        Note over ValidationHelper: Company rejected
    end
    deactivate ValidationHelper
    
    MetricCalculationService-->>ScreeningOrchestrator: InvestmentMetrics (40+ fields)
    Note over MetricCalculationService: Ready for Stage 4: Quality Scoring
    deactivate MetricCalculationService
```

## 5. Stage 4: Composite Scoring

```mermaid
sequenceDiagram
    participant ScreeningOrchestrator
    participant Stage3QualityScoring
    participant Stage4CompositeScoring
    participant ScreeningResult

    ScreeningOrchestrator->>Stage3QualityScoring: CalculateQualityScore(metrics)
    activate Stage3QualityScoring
    
    Stage3QualityScoring->>Stage3QualityScoring: Pillar 1: Return on Capital (0-30 pts)
    Stage3QualityScoring->>Stage3QualityScoring: Pillar 2: Profitability (0-30 pts)
    Stage3QualityScoring->>Stage3QualityScoring: Pillar 3: Cash Flow Quality (0-20 pts)
    Stage3QualityScoring->>Stage3QualityScoring: Pillar 4: Business Quality (0-10 pts)
    Stage3QualityScoring->>Stage3QualityScoring: QualityScore = P1+P2+P3+P4
    
    Stage3QualityScoring-->>ScreeningOrchestrator: decimal QualityScore (0-100)
    deactivate Stage3QualityScoring
    
    ScreeningOrchestrator->>Stage4CompositeScoring: CalculateCompositeScore(quality)
    activate Stage4CompositeScoring
    
    Stage4CompositeScoring->>Stage4CompositeScoring: CalculateValuationScore() (0-100)
    Stage4CompositeScoring->>Stage4CompositeScoring: CalculateManagementScore() (0-100)
    
    Note over Stage4CompositeScoring: COMPOSITE FORMULA:<br/>(Quality × 40%) + (Valuation × 35%) + (Management × 25%)
    
    Stage4CompositeScoring->>Stage4CompositeScoring: Composite = (Q*0.4) + (V*0.35) + (M*0.25)
    
    Stage4CompositeScoring->>ScreeningResult: Create ScreeningResult
    activate ScreeningResult
    ScreeningResult->>ScreeningResult: Set QualityScore
    ScreeningResult->>ScreeningResult: Set ValuationScore
    ScreeningResult->>ScreeningResult: Set ManagementScore
    ScreeningResult->>ScreeningResult: Set CompositeScore
    ScreeningResult-->>Stage4CompositeScoring: ScreeningResult populated ✓
    deactivate ScreeningResult
    
    Stage4CompositeScoring-->>ScreeningOrchestrator: (Quality, Valuation, Management, Composite)
    Note over Stage4CompositeScoring: Ready for Stage 5: Pass/Fail Gate
    deactivate Stage4CompositeScoring
```

## 6. Stage 5: Pass/Fail Gate

```mermaid
sequenceDiagram
    participant ScreeningOrchestrator
    participant Stage5ResultsProcessing
    participant ScreeningResult

    ScreeningOrchestrator->>Stage5ResultsProcessing: ProcessResults(results: List<Result>)
    activate Stage5ResultsProcessing
    
    Stage5ResultsProcessing->>Stage5ResultsProcessing: Initialize qualified list
    
    loop For each ScreeningResult
        Stage5ResultsProcessing->>ScreeningResult: GetCompositeScore()
        activate ScreeningResult
        ScreeningResult-->>Stage5ResultsProcessing: decimal compositeScore
        deactivate ScreeningResult
        
        alt compositeScore >= 75
            Stage5ResultsProcessing->>ScreeningResult: SetPassedStage5(true)
            Stage5ResultsProcessing->>Stage5ResultsProcessing: Add to qualified list
            Note over Stage5ResultsProcessing: ✓ PASS: Advance to Stage 6 Valuation
        else compositeScore < 75
            Stage5ResultsProcessing->>ScreeningResult: SetPassedStage5(false)
            Stage5ResultsProcessing->>Stage5ResultsProcessing: Add to failed list
            Note over Stage5ResultsProcessing: ✗ FAIL: Score too low, discard
        end
    end
    
    Stage5ResultsProcessing->>Stage5ResultsProcessing: Compare with previous run
    Note over Stage5ResultsProcessing: Track: NEW, MAINTAINED, REMOVED, IMPROVED, DECLINED
    
    Stage5ResultsProcessing->>Stage5ResultsProcessing: Log statistics
    Note over Stage5ResultsProcessing: Total evaluated, Passed, Failed, Pass rate %
    
    Stage5ResultsProcessing-->>ScreeningOrchestrator: List<Qualified Companies>
    Note over Stage5ResultsProcessing: Typically 5-15% of universe qualifies
    Note over Stage5ResultsProcessing: Ready for Stage 6: Comprehensive Valuation
    deactivate Stage5ResultsProcessing
```

## 7. Stage 6: Valuation Methods (Complete)

```mermaid
sequenceDiagram
    participant ScreeningOrchestrator
    participant ValuationEngine
    participant DCFValuation
    participant DDMValuation
    participant MultiplesValuation
    participant BuffettValuation
    participant MungerValuation
    participant ConsensusCalculator
    participant MOSCalculator

    ScreeningOrchestrator->>ValuationEngine: CalculateValuationsAsync(company)
    activate ValuationEngine
    
    ValuationEngine->>ValuationEngine: Initialize CompanyValuation
    
    Note over ValuationEngine: METHOD 1: DCF (40% weight)
    ValuationEngine->>DCFValuation: Calculate(company, data)
    activate DCFValuation
    DCFValuation->>DCFValuation: 10-year FCF projection
    DCFValuation->>DCFValuation: Calculate WACC (discount rate)
    DCFValuation->>DCFValuation: Calculate terminal value
    DCFValuation->>DCFValuation: Sum discounted values
    DCFValuation-->>ValuationEngine: decimal DCFFairValue
    deactivate DCFValuation
    
    Note over ValuationEngine: METHOD 2: DDM (15% weight)
    ValuationEngine->>DDMValuation: Calculate(company, data)
    activate DDMValuation
    alt Pays dividends
        DDMValuation->>DDMValuation: Project future dividends
        DDMValuation-->>ValuationEngine: decimal DDMFairValue ✓
    else No dividends
        DDMValuation->>DDMValuation: Use buyback-adjusted approach
        DDMValuation-->>ValuationEngine: decimal AdjustedDDMValue ✓
    end
    deactivate DDMValuation
    
    Note over ValuationEngine: METHOD 3: MULTIPLES (20% weight)
    ValuationEngine->>MultiplesValuation: Calculate(company, peers)
    activate MultiplesValuation
    MultiplesValuation->>MultiplesValuation: Identify 5 peer companies
    MultiplesValuation->>MultiplesValuation: Calculate P/E, P/B, P/S multiples
    MultiplesValuation->>MultiplesValuation: Apply to target company
    MultiplesValuation-->>ValuationEngine: decimal MultiplesFairValue
    deactivate MultiplesValuation
    
    Note over ValuationEngine: METHOD 4: BUFFETT (15% weight)
    ValuationEngine->>BuffettValuation: Calculate(company, data)
    activate BuffettValuation
    BuffettValuation->>BuffettValuation: Owner's Earnings approach
    BuffettValuation->>BuffettValuation: Bond Yield comparison
    BuffettValuation->>BuffettValuation: Apply 25-50% margin of safety
    BuffettValuation-->>ValuationEngine: decimal BuffettFairValue
    deactivate BuffettValuation
    
    Note over ValuationEngine: METHOD 5: MUNGER (10% weight)
    ValuationEngine->>MungerValuation: Calculate(company, result)
    activate MungerValuation
    MungerValuation->>MungerValuation: Score business quality (1-10)
    MungerValuation->>MungerValuation: Assign quality-based multiple
    MungerValuation->>MungerValuation: Apply Five Stock Test
    MungerValuation-->>ValuationEngine: decimal MungerFairValue
    deactivate MungerValuation
    
    Note over ValuationEngine: CALCULATE CONSENSUS
    ValuationEngine->>ConsensusCalculator: Calculate(dcf, ddm, mult, buff, mung)
    activate ConsensusCalculator
    ConsensusCalculator->>ConsensusCalculator: (DCF×40%) + (Multiples×20%) + (Buffett×15%) + (DDM×15%) + (Munger×10%)
    ConsensusCalculator-->>ValuationEngine: decimal ConsensusFairValue
    deactivate ConsensusCalculator
    
    Note over ValuationEngine: CALCULATE MARGIN OF SAFETY
    ValuationEngine->>MOSCalculator: Calculate(fairValue, currentPrice)
    activate MOSCalculator
    MOSCalculator->>MOSCalculator: MOS = (FV - CP) / FV × 100%
    alt MOS >= 30%
        MOSCalculator->>MOSCalculator: Status = EXCELLENT
    else MOS >= 20%
        MOSCalculator->>MOSCalculator: Status = GOOD
    else MOS >= 10%
        MOSCalculator->>MOSCalculator: Status = FAIR
    else MOS >= 0%
        MOSCalculator->>MOSCalculator: Status = MARGINAL
    else MOS < 0%
        MOSCalculator->>MOSCalculator: Status = OVERVALUED
    end
    MOSCalculator-->>ValuationEngine: (decimal MOS, Status)
    deactivate MOSCalculator
    
    ValuationEngine->>ValuationEngine: AssignValuationGrade (A+ to F)
    ValuationEngine->>ValuationEngine: GetBuffettRecommendation()
    ValuationEngine->>ValuationEngine: GetMungerRecommendation()
    ValuationEngine->>ValuationEngine: GetConsensusRecommendation()
    
    ValuationEngine-->>ScreeningOrchestrator: CompanyValuation (complete)
    Note over ValuationEngine: Ready for Stage 7: Results Reporting
    deactivate ValuationEngine
```

## 8. Stage 7: Results Reporting

```mermaid
sequenceDiagram
    participant ScreeningOrchestrator
    participant ReportingService
    participant ReportGroupingService
    participant EmailReportGenerator
    participant GitHubSummaryGenerator
    participant CSVExportGenerator
    participant IRepository
    database MySQL

    ScreeningOrchestrator->>ReportingService: GenerateAndSendAsync(valuations)
    activate ReportingService
    
    Note over ReportingService: STEP 1: GROUP BY OPPORTUNITY
    ReportingService->>ReportGroupingService: GroupByOpportunity(valuations)
    activate ReportGroupingService
    
    loop For each CompanyValuation
        alt Quality >= 80 AND MOS >= 30%
            ReportGroupingService->>ReportGroupingService: Group 1: SLAM DUNK BUYS
        else Quality >= 75 AND MOS >= 20%
            ReportGroupingService->>ReportGroupingService: Group 2: STRONG BUYS
        else Quality >= 82 AND MOS >= 10%
            ReportGroupingService->>ReportGroupingService: Group 3: FAIR VALUE
        else Quality >= 80 AND MOS < 0%
            ReportGroupingService->>ReportGroupingService: Group 4: OVERVALUED (Watchlist)
        end
    end
    
    ReportGroupingService-->>ReportingService: Dictionary<ReportGroup, List<Companies>>
    deactivate ReportGroupingService
    
    Note over ReportingService: STEP 2: GENERATE EMAIL REPORT
    ReportingService->>EmailReportGenerator: GenerateReport(grouped)
    activate EmailReportGenerator
    EmailReportGenerator->>EmailReportGenerator: Create header & executive summary
    EmailReportGenerator->>EmailReportGenerator: Create grouped sections
    EmailReportGenerator->>EmailReportGenerator: Format company details
    EmailReportGenerator->>EmailReportGenerator: Attach CSV
    EmailReportGenerator->>EmailReportGenerator: SendEmail()
    EmailReportGenerator-->>ReportingService: Email sent ✓
    deactivate EmailReportGenerator
    
    Note over ReportingService: STEP 3: GENERATE GITHUB SUMMARY
    ReportingService->>GitHubSummaryGenerator: GenerateMarkdown(grouped)
    activate GitHubSummaryGenerator
    GitHubSummaryGenerator->>GitHubSummaryGenerator: Create summary tables
    GitHubSummaryGenerator->>GitHubSummaryGenerator: Create top 5 opportunities
    GitHubSummaryGenerator->>GitHubSummaryGenerator: Create insights section
    GitHubSummaryGenerator->>GitHubSummaryGenerator: PostToGitHubActions()
    GitHubSummaryGenerator-->>ReportingService: Summary posted ✓
    deactivate GitHubSummaryGenerator
    
    Note over ReportingService: STEP 4: GENERATE CSV EXPORT
    ReportingService->>CSVExportGenerator: GenerateCSV(valuations)
    activate CSVExportGenerator
    CSVExportGenerator->>CSVExportGenerator: Create header (50+ columns)
    CSVExportGenerator->>CSVExportGenerator: Create data rows
    CSVExportGenerator->>CSVExportGenerator: WriteToFile()
    CSVExportGenerator->>CSVExportGenerator: ValidateOutput()
    CSVExportGenerator-->>ReportingService: CSV generated ✓
    deactivate CSVExportGenerator
    
    Note over ReportingService: STEP 5: STORE IN DATABASE
    ReportingService->>IRepository: SaveValuationsAsync(valuations)
    activate IRepository
    
    loop For each CompanyValuation
        IRepository->>MySQL: INSERT INTO CompanyValuations
        activate MySQL
        MySQL->>MySQL: Store complete valuation record
        MySQL-->>IRepository: Stored (ID returned)
        deactivate MySQL
    end
    
    IRepository-->>ReportingService: All records saved ✓
    deactivate IRepository
    
    ReportingService-->>ScreeningOrchestrator: Reporting complete
    Note over ReportingService: ✓ Email sent<br/>✓ GitHub summary posted<br/>✓ CSV generated<br/>✓ Database updated
    deactivate ReportingService
    
    ScreeningOrchestrator-->>ScreeningOrchestrator: QUARTERLY SCREENING COMPLETE!
    Note over ScreeningOrchestrator: Results Delivered:<br/>1. Email report (detailed)<br/>2. GitHub summary (quick view)<br/>3. CSV export (spreadsheet)<br/>4. Database storage (historical)
```

## 9. Utility: Individual Metric Calculation (ROE Example)

```mermaid
sequenceDiagram
    participant MetricCalculationService
    participant ReturnOnCapitalCalculator
    participant CAGRCalculator
    participant CoefficientOfVariationCalculator
    participant ValidationHelper

    MetricCalculationService->>ReturnOnCapitalCalculator: CalculateROE(company, financialDataList)
    activate ReturnOnCapitalCalculator
    
    Note over ReturnOnCapitalCalculator: STEP 1: EXTRACT HISTORICAL EQUITY
    ReturnOnCapitalCalculator->>ReturnOnCapitalCalculator: Loop through 10 years (2016-2025)
    ReturnOnCapitalCalculator->>ReturnOnCapitalCalculator: Extract Total Shareholders' Equity
    Note over ReturnOnCapitalCalculator: Store: [Eq2016, Eq2017, ..., Eq2025]
    
    Note over ReturnOnCapitalCalculator: STEP 2: CALCULATE YEAR-BY-YEAR ROE
    loop For each year (i = 1 to 10)
        ReturnOnCapitalCalculator->>ReturnOnCapitalCalculator: ROE[i] = NetIncome[i] / Equity[i]
        Note over ReturnOnCapitalCalculator: Example Year 2025:<br/>NI: $5B / Equity: $20B = 25%
    end
    
    Note over ReturnOnCapitalCalculator: STEP 3: CALCULATE ROE CAGR
    ReturnOnCapitalCalculator->>CAGRCalculator: CalculateCAGR(endROE, startROE, years: 10)
    activate CAGRCalculator
    CAGRCalculator->>CAGRCalculator: CAGR = (End/Start)^(1/10) - 1
    Note over CAGRCalculator: Example:<br/>Start 2016: 18%<br/>End 2025: 25%<br/>CAGR = (25/18)^(1/10) - 1 = 3.3%
    CAGRCalculator-->>ReturnOnCapitalCalculator: decimal roeCagr ✓
    deactivate CAGRCalculator
    
    Note over ReturnOnCapitalCalculator: STEP 4: CALCULATE ROE STABILITY
    ReturnOnCapitalCalculator->>CoefficientOfVariationCalculator: Calculate(roeList)
    activate CoefficientOfVariationCalculator
    CoefficientOfVariationCalculator->>CoefficientOfVariationCalculator: StdDev = sqrt(Variance)
    CoefficientOfVariationCalculator->>CoefficientOfVariationCalculator: Mean = Average(roeList)
    CoefficientOfVariationCalculator->>CoefficientOfVariationCalculator: CoV = StdDev / Mean
    Note over CoefficientOfVariationCalculator: Example:<br/>Values: [18%, 19%, 20%, 22%, 25%, 24%, 23%, 25%, 26%, 25%]<br/>Mean: 22.7%, StdDev: 2.7%<br/>CoV: 12% (Lower = More Stable)
    CoefficientOfVariationCalculator-->>ReturnOnCapitalCalculator: decimal roeStability ✓
    deactivate CoefficientOfVariationCalculator
    
    Note over ReturnOnCapitalCalculator: STEP 5: VALIDATE CALCULATIONS
    ReturnOnCapitalCalculator->>ValidationHelper: ValidateROE(roe, cagr, stability)
    activate ValidationHelper
    alt All checks pass
        ValidationHelper->>ValidationHelper: ✓ ROE > 0
        ValidationHelper->>ValidationHelper: ✓ CAGR reasonable
        ValidationHelper->>ValidationHelper: ✓ Stability valid
        ValidationHelper-->>ReturnOnCapitalCalculator: valid = true ✓
    else Any check fails
        ValidationHelper->>ValidationHelper: ✗ Invalid ROE data
        ValidationHelper-->>ReturnOnCapitalCalculator: valid = false ✗
        Note over ValidationHelper: Flag for investigation
    end
    deactivate ValidationHelper
    
    Note over ReturnOnCapitalCalculator: STEP 6: PACKAGE RESULTS
    alt Data valid
        ReturnOnCapitalCalculator->>ReturnOnCapitalCalculator: Package ROE result
        Note over ReturnOnCapitalCalculator: Result Structure:<br/>Current ROE: 25% (2025)<br/>10-Year Avg: 22.7%<br/>10-Year CAGR: 3.3%<br/>Stability Score: 8.8/10<br/>Trend: INCREASING<br/>Quality: A (stable, growing)
        ReturnOnCapitalCalculator-->>MetricCalculationService: Complete ROE data ✓
    else Data invalid
        ReturnOnCapitalCalculator-->>MetricCalculationService: null (flag company) ✗
    end
    
    deactivate ReturnOnCapitalCalculator
    
    Note over MetricCalculationService: REPEATS FOR ALL 40+ METRICS:<br/>• 8 return-on-capital metrics<br/>• 8 financial-strength metrics<br/>• 7 profitability metrics<br/>• 6 cash-flow metrics<br/>• 5+ growth metrics<br/>• Plus valuation & management metrics<br/><br/>ALL calculated BEFORE any decisions made
```

---
