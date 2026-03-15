## 1. Complete Quarterly Screening Flow (Sequence Diagram)
```
@startuml Complete_Quarterly_Screening

actor User
participant Program
participant ScreeningOrchestrator
participant ScreeningService
participant IDataFetchService
participant IMetricCalculationService
participant IValuationService
participant IReportingService
database MySQL

User -> Program: ExecuteQuarterlyScreeningAsync()
activate Program
    Program -> ScreeningOrchestrator: new ScreeningOrchestrator()
    activate ScreeningOrchestrator
        
        == STAGE 0: PRE-FILTER ==
        ScreeningOrchestrator -> ScreeningService: ExecuteStage0_PreFilterAsync()
        activate ScreeningService
            ScreeningService -> ScreeningService: Filter by MarketCap > $300M
            ScreeningService --> ScreeningOrchestrator: List<Company> (filtered)
        deactivate ScreeningService
        
        == STAGE 1: DATA INGESTION ==
        ScreeningOrchestrator -> IDataFetchService: FetchHistoricalFinancialDataAsync(ticker, years: 10)
        activate IDataFetchService
            IDataFetchService -> IDataFetchService: Try SEC EDGAR
            IDataFetchService -> IDataFetchService: If failed, try Yahoo Finance
            IDataFetchService -> IDataFetchService: If failed, try IEX Cloud
            IDataFetchService -> IDataFetchService: If failed, try Alpha Vantage
            IDataFetchService --> ScreeningOrchestrator: List<FinancialData>
        deactivate IDataFetchService
        
        == STAGES 2-3: METRICS & SCREENING ==
        ScreeningOrchestrator -> IMetricCalculationService: CalculateMetrics()
        activate IMetricCalculationService
            loop For each Company
                IMetricCalculationService -> IMetricCalculationService: Calculate 40+ metrics
                IMetricCalculationService -> IMetricCalculationService: Apply hard filters
                IMetricCalculationService -> IMetricCalculationService: Calculate Quality Score
            end
            IMetricCalculationService --> ScreeningOrchestrator: List<ScreeningResult>
        deactivate IMetricCalculationService
        
        == STAGE 4: COMPOSITE SCORING ==
        ScreeningOrchestrator -> ScreeningService: CalculateStage4Scores()
        activate ScreeningService
            loop For each ScreeningResult
                ScreeningService -> ScreeningService: Apply 40:35:25 weighting
                ScreeningService -> ScreeningService: Calculate Composite Score
            end
            ScreeningService --> ScreeningOrchestrator: (Quality, Valuation, Management, Composite)
        deactivate ScreeningService
        
        == STAGE 5: PASS/FAIL GATE ==
        ScreeningOrchestrator -> ScreeningService: ProcessStage5ResultsAsync()
        activate ScreeningService
            loop For each ScreeningResult
                alt CompositeScore >= 75
                    ScreeningService -> ScreeningService: PASS - advance to Stage 6
                else CompositeScore < 75
                    ScreeningService -> ScreeningService: FAIL - discard
                end
            end
            ScreeningService --> ScreeningOrchestrator: List<Qualified Companies>
        deactivate ScreeningService
        
        == STAGE 6: VALUATION ==
        ScreeningOrchestrator -> IValuationService: CalculateValuationsAsync()
        activate IValuationService
            loop For each Qualified Company
                IValuationService -> IValuationService: DCF Valuation (40%)
                IValuationService -> IValuationService: DDM Valuation (15%)
                IValuationService -> IValuationService: Multiples Valuation (20%)
                IValuationService -> IValuationService: Buffett Valuation (15%)
                IValuationService -> IValuationService: Munger Valuation (10%)
                IValuationService -> IValuationService: Calculate Consensus
                IValuationService -> IValuationService: Calculate Margin of Safety
            end
            IValuationService --> ScreeningOrchestrator: List<CompanyValuation>
        deactivate IValuationService
        
        == STAGE 7: REPORTING ==
        ScreeningOrchestrator -> IReportingService: GenerateAndSendAsync()
        activate IReportingService
            IReportingService -> IReportingService: Generate Email Report
            IReportingService -> IReportingService: Generate GitHub Summary
            IReportingService -> IReportingService: Generate CSV Export
            IReportingService -> MySQL: Store CompanyValuations
            activate MySQL
                MySQL --> IReportingService: Stored
            deactivate MySQL
            IReportingService --> ScreeningOrchestrator: Complete
        deactivate IReportingService
        
        ScreeningOrchestrator --> Program: ScreeningRunResult
    deactivate ScreeningOrchestrator
    
    Program --> User: "Screening completed successfully"
deactivate Program

@enduml
```
## 2. Stage 0: Market Cap Pre-Filter
```
@startuml Stage0_PreFilter

participant ScreeningOrchestrator
participant Stage0PreFilter
participant Company

ScreeningOrchestrator -> Stage0PreFilter: Filter(companies: List<Company>)
activate Stage0PreFilter
    
    Stage0PreFilter -> Stage0PreFilter: Initialize empty results list
    
    loop For each Company in input list
        Stage0PreFilter -> Company: GetMarketCapBillions()
        activate Company
            Company --> Stage0PreFilter: decimal marketCap
        deactivate Company
        
        alt marketCap > 0.3 (> $300M)
            Stage0PreFilter -> Stage0PreFilter: Add to results
            note over Stage0PreFilter
                ✓ PASS: Institutional quality
                   Market Cap: $300M+
            end note
        else marketCap <= 0.3
            Stage0PreFilter -> Stage0PreFilter: Skip (discard)
            note over Stage0PreFilter
                ✗ FAIL: Insufficient liquidity
                   Market Cap: < $300M
                   (saves API calls)
            end note
        end
    end
    
    Stage0PreFilter -> Stage0PreFilter: Count passed/failed
    Stage0PreFilter --> ScreeningOrchestrator: List<Company> (filtered)
    
    note over ScreeningOrchestrator
        Stage 0 Output:
        • ~85-90% of companies pass
        • ~10-15% rejected
        • Reduced universe for Stage 1
    end note

deactivate Stage0PreFilter

@enduml
```
## 3 . Stage 1: Data Ingestion & Reconciliation
```
@startuml Stage1_DataIngestion

participant ScreeningOrchestrator
participant DataFetchService
participant SecEdgarClient
participant YahooFinanceClient
participant IExCloudClient
participant ValidationHelper

ScreeningOrchestrator -> DataFetchService: FetchHistoricalFinancialDataAsync(ticker, years: 10)
activate DataFetchService
    
    DataFetchService -> SecEdgarClient: Get10K(ticker)
    activate SecEdgarClient
        SecEdgarClient -> SecEdgarClient: Query SEC EDGAR API
        alt Data found and valid
            SecEdgarClient --> DataFetchService: FinancialData (10 years)
            note over SecEdgarClient
                ✓ Primary source success
                Income Statement
                Balance Sheet
                Cash Flow Statement
            end note
        else No data or error
            SecEdgarClient --> DataFetchService: null
            note over SecEdgarClient
                ✗ Primary source failed
                Will try fallback
            end note
        end
    deactivate SecEdgarClient
    
    alt Data is valid from SEC EDGAR
        DataFetchService -> DataFetchService: Continue with SEC data
    else SEC EDGAR failed or incomplete
        DataFetchService -> YahooFinanceClient: GetHistoricalData(ticker)
        activate YahooFinanceClient
            YahooFinanceClient -> YahooFinanceClient: Query Yahoo Finance API
            alt Data found
                YahooFinanceClient --> DataFetchService: FinancialData
                note over YahooFinanceClient
                    ✓ Fallback #1 success
                end note
            else Still no data
                YahooFinanceClient --> DataFetchService: null
                DataFetchService -> IExCloudClient: GetHistoricalData(ticker)
                activate IExCloudClient
                    IExCloudClient -> IExCloudClient: Query IEX Cloud API
                    IExCloudClient --> DataFetchService: FinancialData
                    note over IExCloudClient
                        ✓ Fallback #2 success
                    end note
                deactivate IExCloudClient
            end
        deactivate YahooFinanceClient
    end
    
    DataFetchService -> DataFetchService: Fetch current stock price
    DataFetchService -> DataFetchService: Validate data quality
    activate ValidationHelper
        ValidationHelper -> ValidationHelper: Check data age < 1 year
        ValidationHelper -> ValidationHelper: Check minimum 5 years history
        ValidationHelper -> ValidationHelper: Validate all required fields
        alt Data passes validation
            ValidationHelper --> DataFetchService: valid = true
            note over ValidationHelper
                ✓ Data quality check PASSED
                • Age: < 1 year
                • History: >= 5 years
                • Completeness: 100%
            end note
        else Data fails validation
            ValidationHelper --> DataFetchService: valid = false
            note over ValidationHelper
                ✗ Data quality check FAILED
                Company will be rejected
            end note
        end
    deactivate ValidationHelper
    
    DataFetchService --> ScreeningOrchestrator: List<FinancialData> (validated)

deactivate DataFetchService

note over ScreeningOrchestrator
    Stage 1 Output:
    • Complete 10-year financial history
    • Current stock price
    • All data validated
    • Ready for metric calculation
end note

@enduml
```
## 4. Stages 2-3: Metric Calculation & Hard Filters
```
@startuml Stage2_3_MetricsCalculation

participant ScreeningOrchestrator
participant MetricCalculationService
participant ReturnOnCapitalCalculator
participant FinancialStrengthCalculator
participant ProfitabilityCalculator
participant CashFlowCalculator
participant GrowthCalculator
participant ValidationHelper

ScreeningOrchestrator -> MetricCalculationService: CalculateMetrics(company, financialData)
activate MetricCalculationService
    
    MetricCalculationService -> MetricCalculationService: Initialize InvestmentMetrics
    
    == CATEGORY 1: RETURN ON CAPITAL ==
    MetricCalculationService -> ReturnOnCapitalCalculator: CalculateROE(equity, netIncome)
    activate ReturnOnCapitalCalculator
        ReturnOnCapitalCalculator -> ReturnOnCapitalCalculator: ROE = NetIncome / Equity
        ReturnOnCapitalCalculator -> ReturnOnCapitalCalculator: Calculate 10-year CAGR
        ReturnOnCapitalCalculator --> MetricCalculationService: decimal ROE
    deactivate ReturnOnCapitalCalculator
    
    MetricCalculationService -> ReturnOnCapitalCalculator: CalculateROA(assets, netIncome)
    activate ReturnOnCapitalCalculator
        ReturnOnCapitalCalculator -> ReturnOnCapitalCalculator: ROA = NetIncome / Assets
        ReturnOnCapitalCalculator --> MetricCalculationService: decimal ROA
    deactivate ReturnOnCapitalCalculator
    
    MetricCalculationService -> ReturnOnCapitalCalculator: CalculateROIC(invested, nopat)
    activate ReturnOnCapitalCalculator
        ReturnOnCapitalCalculator -> ReturnOnCapitalCalculator: ROIC = NOPAT / Invested Capital
        ReturnOnCapitalCalculator --> MetricCalculationService: decimal ROIC
    deactivate ReturnOnCapitalCalculator
    
    == CATEGORY 2: FINANCIAL STRENGTH ==
    MetricCalculationService -> FinancialStrengthCalculator: CalculateDebtToEquity(debt, equity)
    activate FinancialStrengthCalculator
        FinancialStrengthCalculator -> FinancialStrengthCalculator: D/E = Total Debt / Equity
        FinancialStrengthCalculator --> MetricCalculationService: decimal D/E
        note over FinancialStrengthCalculator
            Hard Filter Check:
            ✓ If D/E < 1.5: PASS
            ✗ If D/E >= 1.5: FLAG
        end note
    deactivate FinancialStrengthCalculator
    
    MetricCalculationService -> FinancialStrengthCalculator: CalculateCurrentRatio(ca, cl)
    activate FinancialStrengthCalculator
        FinancialStrengthCalculator -> FinancialStrengthCalculator: CR = Current Assets / Current Liabilities
        FinancialStrengthCalculator --> MetricCalculationService: decimal CR
        note over FinancialStrengthCalculator
            Hard Filter Check:
            ✓ If CR > 1.0: PASS
            ✗ If CR <= 1.0: FLAG
        end note
    deactivate FinancialStrengthCalculator
    
    == CATEGORY 3: PROFITABILITY ==
    MetricCalculationService -> ProfitabilityCalculator: CalculateNetMargin(income, revenue)
    activate ProfitabilityCalculator
        ProfitabilityCalculator -> ProfitabilityCalculator: NM = Net Income / Revenue
        ProfitabilityCalculator -> ProfitabilityCalculator: Calculate stability (CoV)
        ProfitabilityCalculator --> MetricCalculationService: decimal NetMargin
    deactivate ProfitabilityCalculator
    
    MetricCalculationService -> ProfitabilityCalculator: CalculateOperatingMargin(op, revenue)
    activate ProfitabilityCalculator
        ProfitabilityCalculator -> ProfitabilityCalculator: OM = Operating Income / Revenue
        ProfitabilityCalculator --> MetricCalculationService: decimal OM
    deactivate ProfitabilityCalculator
    
    == CATEGORY 4: CASH FLOW QUALITY ==
    MetricCalculationService -> CashFlowCalculator: CalculateFreeCashFlow(ocf, capex)
    activate CashFlowCalculator
        CashFlowCalculator -> CashFlowCalculator: FCF = OCF - CapEx
        CashFlowCalculator -> CashFlowCalculator: Calculate 10-year trend
        CashFlowCalculator --> MetricCalculationService: decimal FCF
    deactivate CashFlowCalculator
    
    MetricCalculationService -> CashFlowCalculator: CalculateFCFToNIRatio(fcf, ni)
    activate CashFlowCalculator
        CashFlowCalculator -> CashFlowCalculator: FCF/NI = FCF / Net Income
        CashFlowCalculator --> MetricCalculationService: decimal FCF/NI
        note over CashFlowCalculator
            Hard Filter Check:
            ✓ If FCF/NI > 0.8: PASS (high quality)
            ✗ If FCF/NI < 0.5: FLAG (low quality)
        end note
    deactivate CashFlowCalculator
    
    == CATEGORY 5: GROWTH ==
    MetricCalculationService -> GrowthCalculator: CalculateRevenueGrowthCAGR(end, start, years)
    activate GrowthCalculator
        GrowthCalculator -> GrowthCalculator: CAGR = (End/Start)^(1/years) - 1
        GrowthCalculator --> MetricCalculationService: decimal Revenue CAGR
    deactivate GrowthCalculator
    
    MetricCalculationService -> GrowthCalculator: CalculateEPSGrowthCAGR(end, start, years)
    activate GrowthCalculator
        GrowthCalculator -> GrowthCalculator: CAGR = (End/Start)^(1/years) - 1
        GrowthCalculator --> MetricCalculationService: decimal EPS CAGR
    deactivate GrowthCalculator
    
    == VALIDATION ==
    MetricCalculationService -> ValidationHelper: ValidateMetrics(metrics)
    activate ValidationHelper
        alt All metrics calculated successfully
            ValidationHelper --> MetricCalculationService: valid = true
            note over ValidationHelper
                ✓ Metrics validation PASSED
                • All 40+ metrics calculated
                • No null values
                • Reasonable ranges
            end note
        else Missing or invalid metrics
            ValidationHelper --> MetricCalculationService: valid = false
            note over ValidationHelper
                ✗ Metrics validation FAILED
                Company rejected
            end note
        end
    deactivate ValidationHelper
    
    alt Metrics valid
        MetricCalculationService -> MetricCalculationService: Complete hard filter evaluation
        MetricCalculationService --> ScreeningOrchestrator: InvestmentMetrics (complete)
        note over MetricCalculationService
            Stage 2-3 Output:
            • 40+ metrics calculated
            • Hard filters evaluated
            • All data with complete context
            • Ready for quality scoring
        end note
    else Metrics invalid
        MetricCalculationService --> ScreeningOrchestrator: null (company rejected)
    end

deactivate MetricCalculationService

@enduml
```
## 5. Stage 4: Composite Scoring
```
@startuml Stage4_CompositeScoring

participant ScreeningOrchestrator
participant Stage3QualityScoring
participant Stage4CompositeScoring
participant ScreeningResult

ScreeningOrchestrator -> Stage3QualityScoring: CalculateQualityScore(metrics)
activate Stage3QualityScoring
    
    Stage3QualityScoring -> Stage3QualityScoring: ScorePillar1_ReturnOnCapital(metrics)
    note over Stage3QualityScoring
        Pillar 1: Return on Capital (max 30 points)
        • ROE consistency: 0-10 points
        • ROIC levels: 0-10 points
        • Trend direction: 0-10 points
    end note
    
    Stage3QualityScoring -> Stage3QualityScoring: ScorePillar2_Profitability(metrics)
    note over Stage3QualityScoring
        Pillar 2: Profitability (max 30 points)
        • Net margin: 0-10 points
        • Operating margin: 0-10 points
        • Margin stability: 0-10 points
    end note
    
    Stage3QualityScoring -> Stage3QualityScoring: ScorePillar3_CashFlowQuality(metrics)
    note over Stage3QualityScoring
        Pillar 3: Cash Flow Quality (max 20 points)
        • FCF/NI ratio: 0-10 points
        • OCF consistency: 0-10 points
    end note
    
    Stage3QualityScoring -> Stage3QualityScoring: ScorePillar4_BusinessQuality(metrics)
    note over Stage3QualityScoring
        Pillar 4: Business Quality (max 10 points)
        • Moat strength: 0-5 points
        • Competitive position: 0-5 points
    end note
    
    Stage3QualityScoring -> Stage3QualityScoring: Sum all pillars
    Stage3QualityScoring -> Stage3QualityScoring: QualityScore = P1 + P2 + P3 + P4
    Stage3QualityScoring --> ScreeningOrchestrator: decimal QualityScore (0-100)

deactivate Stage3QualityScoring

ScreeningOrchestrator -> Stage4CompositeScoring: CalculateCompositeScore(qualityScore, metrics)
activate Stage4CompositeScoring
    
    Stage4CompositeScoring -> Stage4CompositeScoring: CalculateValuationScore(metrics)
    note over Stage4CompositeScoring
        Valuation Score (0-100)
        Based on:
        • P/E ratio: 0-35 points
        • P/B ratio: 0-35 points
        • Free cash flow yield: 0-30 points
    end note
    
    Stage4CompositeScoring -> Stage4CompositeScoring: CalculateManagementScore(metrics)
    note over Stage4CompositeScoring
        Management Score (0-100)
        Based on:
        • Insider ownership: 0-25 points
        • Capital allocation quality: 0-25 points
        • Dividend/buyback consistency: 0-25 points
        • Goodwill ratio: 0-25 points
    end note
    
    Stage4CompositeScoring -> Stage4CompositeScoring: Apply weighted formula
    note over Stage4CompositeScoring
        COMPOSITE FORMULA:
        (Quality × 40%) + 
        (Valuation × 35%) + 
        (Management × 25%) 
        = Composite Score
    end note
    
    Stage4CompositeScoring -> Stage4CompositeScoring: Composite = (Q * 0.4) + (V * 0.35) + (M * 0.25)
    
    Stage4CompositeScoring -> ScreeningResult: Create ScreeningResult
    activate ScreeningResult
        ScreeningResult -> ScreeningResult: Set QualityScore
        ScreeningResult -> ScreeningResult: Set ValuationScore
        ScreeningResult -> ScreeningResult: Set ManagementScore
        ScreeningResult -> ScreeningResult: Set CompositeScore
        ScreeningResult --> Stage4CompositeScoring: ScreeningResult populated
    deactivate ScreeningResult
    
    Stage4CompositeScoring --> ScreeningOrchestrator: (QualityScore, ValuationScore, ManagementScore, CompositeScore)

deactivate Stage4CompositeScoring

note over ScreeningOrchestrator
    Stage 4 Output:
    • Quality Score (0-100)
    • Valuation Score (0-100)
    • Management Score (0-100)
    • Composite Score (0-100)
    Ready for Stage 5 Pass/Fail Gate
end note

@enduml
```
## 6. Stage 5: Pass/Fail Gate
```
@startuml Stage5_PassFailGate

participant ScreeningOrchestrator
participant Stage5ResultsProcessing
participant ScreeningResult

ScreeningOrchestrator -> Stage5ResultsProcessing: ProcessResults(results: List<ScreeningResult>)
activate Stage5ResultsProcessing
    
    Stage5ResultsProcessing -> Stage5ResultsProcessing: Initialize qualified list
    
    loop For each ScreeningResult in results
        
        Stage5ResultsProcessing -> ScreeningResult: GetCompositeScore()
        activate ScreeningResult
            ScreeningResult --> Stage5ResultsProcessing: decimal compositeScore
        deactivate ScreeningResult
        
        alt compositeScore >= 75
            Stage5ResultsProcessing -> ScreeningResult: SetPassedStage5(true)
            Stage5ResultsProcessing -> Stage5ResultsProcessing: Add to qualified list
            note over Stage5ResultsProcessing
                ✓ PASS: ADVANCE TO STAGE 6
                Composite Score >= 75
                Will receive full valuation
                (5 methods, margin of safety)
            end note
            
        else compositeScore < 75
            Stage5ResultsProcessing -> ScreeningResult: SetPassedStage5(false)
            Stage5ResultsProcessing -> Stage5ResultsProcessing: Add to failed list
            note over Stage5ResultsProcessing
                ✗ FAIL: DISCARD
                Composite Score < 75
                Not investment grade
                (does not proceed)
            end note
        end
        
    end
    
    Stage5ResultsProcessing -> Stage5ResultsProcessing: Compare with previous run
    note over Stage5ResultsProcessing
        Track Changes:
        • NEW: First time qualified
        • MAINTAINED: Already qualified
        • REMOVED: Lost qualification
        • IMPROVED: Score increased
        • DECLINED: Score decreased
    end note
    
    Stage5ResultsProcessing -> Stage5ResultsProcessing: Log statistics
    note over Stage5ResultsProcessing
        Statistics:
        • Total evaluated: X companies
        • Passed (Stage 5): Y companies
        • Failed: (X - Y) companies
        • Pass rate: Y/X %
    end note
    
    Stage5ResultsProcessing --> ScreeningOrchestrator: List<ScreeningResult> (qualified only)

deactivate Stage5ResultsProcessing

note over ScreeningOrchestrator
    Stage 5 Output:
    • Only passing companies (CompositeScore >= 75)
    • Ready for comprehensive valuation
    • Typical: 5-15% of universe qualifies
    • These receive Stage 6 valuation
end note

@enduml
```
## 7. Stage 6: Valuation Methods (Complete)
```
@startuml Stage6_Valuation_Methods

participant ScreeningOrchestrator
participant ValuationEngine
participant DCFValuation
participant DDMValuation
participant MultiplesValuation
participant BuffettValuation
participant MungerValuation
participant ConsensusValuationCalculator
participant MarginOfSafetyCalculator

ScreeningOrchestrator -> ValuationEngine: CalculateValuationsAsync(company, result)
activate ValuationEngine
    
    ValuationEngine -> ValuationEngine: Initialize CompanyValuation
    
    == METHOD 1: DCF VALUATION (40% WEIGHT) ==
    ValuationEngine -> DCFValuation: Calculate(company, historicalData)
    activate DCFValuation
        DCFValuation -> DCFValuation: Step 1: Gather 10-year FCF
        DCFValuation -> DCFValuation: Step 2: Calculate FCF CAGR
        DCFValuation -> DCFValuation: Step 3: Project next 10 years
        DCFValuation -> DCFValuation: Step 4: Calculate WACC (discount rate)
        DCFValuation -> DCFValuation: Step 5: Discount all cash flows
        DCFValuation -> DCFValuation: Step 6: Calculate terminal value
        DCFValuation -> DCFValuation: Step 7: Sum present values
        DCFValuation -> DCFValuation: Step 8: Divide by shares outstanding
        DCFValuation --> ValuationEngine: decimal DCFFairValue
        note over DCFValuation
            DCF Output:
            • Fair value per share: $X.XX
            • Terminal growth rate: Y%
            • Discount rate (WACC): Z%
        end note
    deactivate DCFValuation
    
    == METHOD 2: DDM VALUATION (15% WEIGHT) ==
    ValuationEngine -> DDMValuation: Calculate(company, historicalData)
    activate DDMValuation
        alt Company pays dividends
            DDMValuation -> DDMValuation: Step 1: Gather 10-year dividends
            DDMValuation -> DDMValuation: Step 2: Calculate dividend CAGR
            DDMValuation -> DDMValuation: Step 3: Check payout ratio
            DDMValuation -> DDMValuation: Step 4: Project future dividends
            DDMValuation -> DDMValuation: Step 5: Calculate terminal value
            DDMValuation -> DDMValuation: Step 6: Discount to present
            DDMValuation --> ValuationEngine: decimal DDMFairValue
            note over DDMValuation
                ✓ DDM APPLICABLE
                Dividend stock analysis
            end note
        else Company does not pay dividends
            DDMValuation -> DDMValuation: Calculate share buyback equivalent
            DDMValuation -> DDMValuation: Use as dividend substitute
            DDMValuation --> ValuationEngine: decimal AdjustedDDMValue
            note over DDMValuation
                ✓ BUYBACK-ADJUSTED DDM
                Used for non-dividend companies
            end note
        end
    deactivate DDMValuation
    
    == METHOD 3: MULTIPLES VALUATION (20% WEIGHT) ==
    ValuationEngine -> MultiplesValuation: Calculate(company, peerList)
    activate MultiplesValuation
        MultiplesValuation -> MultiplesValuation: Step 1: Identify 5 peer companies
        MultiplesValuation -> MultiplesValuation: Step 2: Calculate peer multiples
        note over MultiplesValuation
            Calculate for each peer:
            • P/E ratio (Price/Earnings)
            • P/B ratio (Price/Book)
            • P/S ratio (Price/Sales)
        end note
        MultiplesValuation -> MultiplesValuation: Step 3: Calculate average multiples
        MultiplesValuation -> MultiplesValuation: Step 4: Apply to target company
        alt Company has positive earnings
            MultiplesValuation -> MultiplesValuation: Use P/E multiple
            MultiplesValuation -> MultiplesValuation: FairValue = EPS × AvgPE
        else Company has negative earnings
            MultiplesValuation -> MultiplesValuation: Use P/S or P/B multiple
            MultiplesValuation -> MultiplesValuation: FairValue = Revenue × AvgPS
        end
        MultiplesValuation --> ValuationEngine: decimal MultiplesFairValue
    deactivate MultiplesValuation
    
    == METHOD 4: BUFFETT VALUATION (15% WEIGHT) ==
    ValuationEngine -> BuffettValuation: Calculate(company, historicalData)
    activate BuffettValuation
        BuffettValuation -> BuffettValuation: Technique 1: Owner's Earnings
        note over BuffettValuation
            Owner's Earnings Approach:
            1. Calculate normalized earnings
            2. Assign quality multiple (15-35x)
            3. Fair Value = Earnings × Multiple
        end note
        BuffettValuation -> BuffettValuation: Technique 2: Bond Yield Comparison
        note over BuffettValuation
            Earnings Yield Approach:
            1. Calculate earnings yield
            2. Compare to 10-year Treasury
            3. Apply risk premium
        end note
        BuffettValuation -> BuffettValuation: Apply margin of safety (25-50%)
        BuffettValuation --> ValuationEngine: decimal BuffettFairValue
    deactivate BuffettValuation
    
    == METHOD 5: MUNGER VALUATION (10% WEIGHT) ==
    ValuationEngine -> MungerValuation: Calculate(company, screeningResult)
    activate MungerValuation
        MungerValuation -> MungerValuation: Score business quality (1-10)
        note over MungerValuation
            Quality Factors:
            • Moat strength (1-10)
            • Business durability (1-10)
            • Management quality (1-10)
            • Earnings consistency (1-10)
            • Return on capital (1-10)
        end note
        MungerValuation -> MungerValuation: Assign quality-based multiple
        MungerValuation -> MungerValuation: Calculate base valuation
        MungerValuation -> MungerValuation: Apply Five Stock Test
        MungerValuation --> ValuationEngine: decimal MungerFairValue
    deactivate MungerValuation
    
    == CALCULATE CONSENSUS ==
    ValuationEngine -> ConsensusValuationCalculator: Calculate(dcf, ddm, mult, buff, mung)
    activate ConsensusValuationCalculator
        ConsensusValuationCalculator -> ConsensusValuationCalculator: Apply weights
        note over ConsensusValuationCalculator
            CONSENSUS FORMULA (weights):
            (DCF × 40%) +
            (Multiples × 20%) +
            (Buffett × 15%) +
            (DDM × 15%) +
            (Munger × 10%)
            = Consensus Fair Value
        end note
        ConsensusValuationCalculator --> ValuationEngine: decimal ConsensusFairValue
    deactivate ConsensusValuationCalculator
    
    == CALCULATE MARGIN OF SAFETY ==
    ValuationEngine -> MarginOfSafetyCalculator: Calculate(fairValue, currentPrice)
    activate MarginOfSafetyCalculator
        MarginOfSafetyCalculator -> MarginOfSafetyCalculator: MOS = (FV - CP) / FV × 100%
        MarginOfSafetyCalculator -> MarginOfSafetyCalculator: Determine MOS status
        alt MOS >= 30%
            MarginOfSafetyCalculator -> MarginOfSafetyCalculator: Status = EXCELLENT
        else MOS >= 20%
            MarginOfSafetyCalculator -> MarginOfSafetyCalculator: Status = GOOD
        else MOS >= 10%
            MarginOfSafetyCalculator -> MarginOfSafetyCalculator: Status = FAIR
        else MOS >= 0%
            MarginOfSafetyCalculator -> MarginOfSafetyCalculator: Status = MARGINAL
        else MOS < 0%
            MarginOfSafetyCalculator -> MarginOfSafetyCalculator: Status = OVERVALUED
        end
        MarginOfSafetyCalculator --> ValuationEngine: (decimal MOS, Status)
    deactivate MarginOfSafetyCalculator
    
    == ASSIGN RECOMMENDATIONS ==
    ValuationEngine -> ValuationEngine: AssignValuationGrade()
    note over ValuationEngine
        Grade based on:
        • Quality score
        • Margin of safety
        • Method convergence
        Grades: A+, A, B+, B, C, D, F
    end note
    
    ValuationEngine -> ValuationEngine: GetBuffettRecommendation()
    ValuationEngine -> ValuationEngine: GetMungerRecommendation()
    ValuationEngine -> ValuationEngine: GetConsensusRecommendation()
    note over ValuationEngine
        Recommendations:
        • BUY (strong buy signal)
        • CONSIDER (watchlist)
        • PASS (fairly valued)
        • AVOID (overvalued)
    end note
    
    ValuationEngine --> ScreeningOrchestrator: CompanyValuation (complete)

deactivate ValuationEngine

note over ScreeningOrchestrator
    Stage 6 Output:
    • Fair values from 5 methods
    • Consensus fair value
    • Margin of safety %
    • Valuation grade (A-F)
    • Three recommendations
    Ready for Stage 7 Reporting
end note

@enduml
```
## 8. Stage 7: Results Reporting
```
@startuml Stage7_ResultsReporting

participant ScreeningOrchestrator
participant ReportingService
participant ReportGroupingService
participant EmailReportGenerator
participant GitHubSummaryGenerator
participant CSVExportGenerator
participant IRepository
database MySQL

ScreeningOrchestrator -> ReportingService: GenerateAndSendAsync(valuations, screeningRunId)
activate ReportingService
    
    == STEP 1: GROUP RESULTS BY OPPORTUNITY ==
    ReportingService -> ReportGroupingService: GroupByOpportunity(valuations)
    activate ReportGroupingService
        
        loop For each CompanyValuation
            alt Quality >= 80 AND MOS >= 30%
                ReportGroupingService -> ReportGroupingService: Group 1: SLAM DUNK BUYS
                note over ReportGroupingService
                    Top Tier: Strongest buys
                    High quality + Large margin
                end note
            else Quality >= 75 AND MOS >= 20%
                ReportGroupingService -> ReportGroupingService: Group 2: STRONG BUYS
            else Quality >= 82 AND MOS >= 10%
                ReportGroupingService -> ReportGroupingService: Group 3: FAIR VALUE
            else Quality >= 80 AND MOS < 0%
                ReportGroupingService -> ReportGroupingService: Group 4: OVERVALUED
                note over ReportGroupingService
                    Watchlist: Currently too expensive
                    Add to watchlist for price drop
                end note
            end
        end
        
        ReportGroupingService --> ReportingService: Dictionary<ReportGroup, List<Valuations>>
        
    deactivate ReportGroupingService
    
    == STEP 2: GENERATE EMAIL REPORT ==
    ReportingService -> EmailReportGenerator: GenerateReport(groupedValuations)
    activate EmailReportGenerator
        
        EmailReportGenerator -> EmailReportGenerator: CreateHeader()
        note over EmailReportGenerator
            Header:
            • Date/Time
            • Screening period
            • Total companies screened
        end note
        
        EmailReportGenerator -> EmailReportGenerator: CreateExecutiveSummary()
        note over EmailReportGenerator
            Summary:
            • Total qualified: X
            • Best opportunity: Y
            • Average MOS: Z%
            • Stage-by-stage stats
        end note
        
        EmailReportGenerator -> EmailReportGenerator: CreateGroupedSections()
        note over EmailReportGenerator
            Sections:
            1. SLAM DUNK BUYS (3-5 stocks)
            2. STRONG BUYS (5-10 stocks)
            3. FAIR VALUE (10-20 stocks)
            4. OVERVALUED (watchlist)
            5. NOT QUALIFIED (stats)
        end note
        
        EmailReportGenerator -> EmailReportGenerator: FormatCompanyDetails()
        note over EmailReportGenerator
            For each stock:
            • Ticker, name, sector
            • Valuation scores
            • Fair value (5 methods)
            • Consensus price target
            • Margin of safety
            • Investment thesis
        end note
        
        EmailReportGenerator -> EmailReportGenerator: AttachCSV()
        EmailReportGenerator -> EmailReportGenerator: SendEmail()
        EmailReportGenerator --> ReportingService: Email sent successfully
        
    deactivate EmailReportGenerator
    
    == STEP 3: GENERATE GITHUB SUMMARY ==
    ReportingService -> GitHubSummaryGenerator: GenerateMarkdown(groupedValuations)
    activate GitHubSummaryGenerator
        
        GitHubSummaryGenerator -> GitHubSummaryGenerator: CreateSummaryTable()
        note over GitHubSummaryGenerator
            Table:
            | Group | Count | Avg MOS |
            | SLAM | X | Y% |
        end note
        
        GitHubSummaryGenerator -> GitHubSummaryGenerator: CreateTop5Table()
        note over GitHubSummaryGenerator
            Top 5 opportunities:
            • Ticker
            • Fair value
            • Current price
            • MOS %
            • Recommendation
        end note
        
        GitHubSummaryGenerator -> GitHubSummaryGenerator: CreateInsightsSection()
        GitHubSummaryGenerator -> GitHubSummaryGenerator: CreateComparisonWithPriorRun()
        GitHubSummaryGenerator -> GitHubSummaryGenerator: PostToGitHubActions()
        GitHubSummaryGenerator --> ReportingService: Summary posted
        
    deactivate GitHubSummaryGenerator
    
    == STEP 4: GENERATE CSV EXPORT ==
    ReportingService -> CSVExportGenerator: GenerateCSV(valuations)
    activate CSVExportGenerator
        
        CSVExportGenerator -> CSVExportGenerator: CreateHeader()
        note over CSVExportGenerator
            Columns:
            • Ticker, Company, Sector
            • Fair values (5 methods)
            • Consensus, MOS, Grade
            • Quality, Valuation, Management
            • Recommendation
            • (50+ columns total)
        end note
        
        CSVExportGenerator -> CSVExportGenerator: CreateRows()
        loop For each CompanyValuation
            CSVExportGenerator -> CSVExportGenerator: Format row
            CSVExportGenerator -> CSVExportGenerator: Add to output
        end
        
        CSVExportGenerator -> CSVExportGenerator: WriteToFile()
        CSVExportGenerator -> CSVExportGenerator: ValidateOutput()
        CSVExportGenerator --> ReportingService: CSV file generated
        
    deactivate CSVExportGenerator
    
    == STEP 5: STORE IN DATABASE ==
    ReportingService -> IRepository: SaveValuationsAsync(valuations)
    activate IRepository
        
        loop For each CompanyValuation
            IRepository -> MySQL: INSERT INTO CompanyValuations
            activate MySQL
                MySQL -> MySQL: Store complete valuation record
                MySQL --> IRepository: Stored (ID returned)
            deactivate MySQL
        end
        
        IRepository --> ReportingService: All records saved
        
    deactivate IRepository
    
    ReportingService --> ScreeningOrchestrator: Reporting complete
    note over ReportingService
        Stage 7 Output:
        ✓ Email sent to investor
        ✓ GitHub summary posted
        ✓ CSV file generated
        ✓ Database updated
        ✓ All results archived
    end note

deactivate ReportingService

note over ScreeningOrchestrator
    QUARTERLY SCREENING COMPLETE!
    
    Results Delivered:
    1. Email report (detailed analysis)
    2. GitHub summary (quick view)
    3. CSV export (spreadsheet analysis)
    4. Database storage (historical tracking)
    
    All stages executed successfully.
end note

@enduml
```
## 9. Utility: Individual Metric Calculation (ROE Example)
```
@startuml Utility_Metric_Calculation_ROE

participant MetricCalculationService
participant ReturnOnCapitalCalculator
participant CAGRCalculator
participant CoefficientOfVariationCalculator
participant ValidationHelper

MetricCalculationService -> ReturnOnCapitalCalculator: CalculateROE(company, financialDataList)
activate ReturnOnCapitalCalculator
    
    ReturnOnCapitalCalculator -> ReturnOnCapitalCalculator: Initialize ROE data structure
    
    == STEP 1: EXTRACT HISTORICAL EQUITY ==
    ReturnOnCapitalCalculator -> ReturnOnCapitalCalculator: Loop through 10 years of data
    note over ReturnOnCapitalCalculator
        For each year (2016-2025):
        Extract: Total Shareholders' Equity
        Store in list: [Eq2016, Eq2017, ..., Eq2025]
    end note
    
    == STEP 2: CALCULATE YEAR-BY-YEAR ROE ==
    ReturnOnCapitalCalculator -> ReturnOnCapitalCalculator: Loop through years
    loop For each year (i = 1 to 10)
        ReturnOnCapitalCalculator -> ReturnOnCapitalCalculator: ROE[i] = NetIncome[i] / Equity[i]
        note over ReturnOnCapitalCalculator
            Example (Year 2025):
            Net Income: $5B
            Equity: $20B
            ROE = 5/20 = 25%
        end note
    end
    
    == STEP 3: CALCULATE ROE CAGR ==
    ReturnOnCapitalCalculator -> CAGRCalculator: CalculateCAGR(endROE, startROE, years: 10)
    activate CAGRCalculator
        CAGRCalculator -> CAGRCalculator: CAGR = (End/Start)^(1/10) - 1
        note over CAGRCalculator
            Example:
            Start (2016): 18%
            End (2025): 25%
            CAGR = (25/18)^(1/10) - 1 = 3.3%
        end note
        CAGRCalculator --> ReturnOnCapitalCalculator: decimal roeCagr
    deactivate CAGRCalculator
    
    == STEP 4: CALCULATE ROE STABILITY ==
    ReturnOnCapitalCalculator -> CoefficientOfVariationCalculator: Calculate(roeList)
    activate CoefficientOfVariationCalculator
        CoefficientOfVariationCalculator -> CoefficientOfVariationCalculator: StdDev = sqrt(Variance)
        CoefficientOfVariationCalculator -> CoefficientOfVariationCalculator: Mean = Average(roeList)
        CoefficientOfVariationCalculator -> CoefficientOfVariationCalculator: CoV = StdDev / Mean
        note over CoefficientOfVariationCalculator
            Example:
            ROE values: [18%, 19%, 20%, 22%, 25%, 24%, 23%, 25%, 26%, 25%]
            Mean: 22.7%
            StdDev: 2.7%
            CoV: 2.7% / 22.7% = 0.119 (12%)
            
            Lower CoV = More stable earnings
        end note
        CoefficientOfVariationCalculator --> ReturnOnCapitalCalculator: decimal roeStability
    deactivate CoefficientOfVariationCalculator
    
    == STEP 5: VALIDATE CALCULATIONS ==
    ReturnOnCapitalCalculator -> ValidationHelper: ValidateROE(roe, cagr, stability)
    activate ValidationHelper
        alt ROE > 0 AND CAGR reasonable AND Stability valid
            ValidationHelper -> ValidationHelper: Passes all checks
            note over ValidationHelper
                ✓ ROE valid
                ✓ CAGR makes sense
                ✓ Stability reasonable
            end note
            ValidationHelper --> ReturnOnCapitalCalculator: valid = true
        else Any validation fails
            ValidationHelper -> ValidationHelper: Fails validation
            note over ValidationHelper
                ✗ Invalid ROE data
                Flag for investigation
            end note
            ValidationHelper --> ReturnOnCapitalCalculator: valid = false
        end
    deactivate ValidationHelper
    
    == STEP 6: RETURN RESULTS ==
    alt Data valid
        ReturnOnCapitalCalculator -> ReturnOnCapitalCalculator: Package results
        note over ReturnOnCapitalCalculator
            Result Structure:
            {
              Current ROE: 25% (2025)
              10-Year Avg: 22.7%
              10-Year CAGR: 3.3%
              Stability Score: 8.8/10
              Trend: INCREASING
              Quality Rating: A (stable, growing)
            }
        end note
        ReturnOnCapitalCalculator --> MetricCalculationService: Complete ROE data
    else Data invalid
        ReturnOnCapitalCalculator --> MetricCalculationService: null (flag company)
    end

deactivate ReturnOnCapitalCalculator

note over MetricCalculationService
    This same process repeats for ALL 40+ metrics:
    • 8 return-on-capital metrics
    • 8 financial-strength metrics
    • 7 profitability metrics
    • 6 cash-flow metrics
    • 5+ growth metrics
    • Plus valuation & management metrics
    
    ALL calculated before any decisions made.
end note

@enduml
```
