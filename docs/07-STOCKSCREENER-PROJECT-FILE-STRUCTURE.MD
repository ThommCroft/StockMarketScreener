StockMarketScreener/
├── src/
│   ├── StockMarketScreener.Console/
│   │   ├── Program.cs
│   │   ├── appsettings.json
│   │   ├── StockMarketScreener.Console.csproj
│   │   ├── Jobs/
│   │   │   ├── QuarterlyScreeningJob.cs
│   │   │   ├── StockPriceUpdateJob.cs
│   │   │   └── IJob.cs (interface)
│   │   └── Startup.cs
│   │
│   ├── StockMarketScreener.Core/
│   │   ├── Models/
│   │   │   ├── Company.cs
│   │   │   ├── FinancialData.cs
│   │   │   ├── ScreeningResult.cs
│   │   │   ├── ValuationResult.cs
│   │   │   ├── InvestmentMetrics.cs
│   │   │   └── CompanyValuation.cs
│   │   │
│   │   ├── Services/
│   │   │   ├── Interfaces/
│   │   │   │   ├── IDataFetchService.cs
│   │   │   │   ├── IMetricCalculationService.cs
│   │   │   │   ├── IScreeningService.cs
│   │   │   │   ├── IValuationService.cs
│   │   │   │   ├── ICompanyService.cs
│   │   │   │   └── IReportingService.cs
│   │   │   │
│   │   │   ├── DataFetch/
│   │   │   │   ├── DataFetchService.cs
│   │   │   │   ├── SecEdgarClient.cs
│   │   │   │   ├── YahooFinanceClient.cs
│   │   │   │   ├── IExCloudClient.cs (fallback)
│   │   │   │   └── AlphaVantageClient.cs (fallback)
│   │   │   │
│   │   │   ├── Metrics/
│   │   │   │   ├── MetricCalculationService.cs
│   │   │   │   ├── ReturnOnCapitalCalculator.cs
│   │   │   │   ├── FinancialStrengthCalculator.cs
│   │   │   │   ├── ProfitabilityCalculator.cs
│   │   │   │   ├── CashFlowCalculator.cs
│   │   │   │   ├── GrowthCalculator.cs
│   │   │   │   ├── ValuationCalculator.cs
│   │   │   │   └── ManagementQualityCalculator.cs
│   │   │   │
│   │   │   ├── Screening/
│   │   │   │   ├── ScreeningOrchestrator.cs
│   │   │   │   ├── Stage0PreFilter.cs
│   │   │   │   ├── Stage1DataIngestion.cs
│   │   │   │   ├── Stage2FinancialStrength.cs
│   │   │   │   ├── Stage3QualityScoring.cs
│   │   │   │   ├── Stage4CompositeScoring.cs
│   │   │   │   ├── Stage5ResultsProcessing.cs
│   │   │   │   └── ScreeningResult.cs
│   │   │   │
│   │   │   ├── Valuation/
│   │   │   │   ├── ValuationEngine.cs
│   │   │   │   ├── DCFValuation.cs
│   │   │   │   ├── DDMValuation.cs
│   │   │   │   ├── MultiplesValuation.cs
│   │   │   │   ├── BuffettValuation.cs
│   │   │   │   ├── MungerValuation.cs
│   │   │   │   ├── ConsensusValuationCalculator.cs
│   │   │   │   └── MarginOfSafetyCalculator.cs
│   │   │   │
│   │   │   ├── Company/
│   │   │   │   ├── CompanyService.cs
│   │   │   │   └── IndustryConfigurationService.cs
│   │   │   │
│   │   │   └── Reporting/
│   │   │       ├── ReportingService.cs
│   │   │       ├── EmailReportGenerator.cs
│   │   │       ├── GitHubSummaryGenerator.cs
│   │   │       ├── CSVExportGenerator.cs
│   │   │       └── ReportGroupingService.cs
│   │   │
│   │   ├── Data/
│   │   │   ├── DbContext/
│   │   │   │   └── StockMarketScreenerDbContext.cs
│   │   │   │
│   │   │   ├── Repositories/
│   │   │   │   ├── Interfaces/
│   │   │   │   │   ├── IRepository.cs (generic)
│   │   │   │   │   ├── ICompanyRepository.cs
│   │   │   │   │   ├── IScreeningRunRepository.cs
│   │   │   │   │   └── ICompanyValuationRepository.cs
│   │   │   │   │
│   │   │   │   ├── CompanyRepository.cs
│   │   │   │   ├── ScreeningRunRepository.cs
│   │   │   │   ├── CompanyValuationRepository.cs
│   │   │   │   └── GenericRepository.cs
│   │   │   │
│   │   │   └── Migrations/
│   │   │       └── [EF Core migrations]
│   │   │
│   │   ├── Utilities/
│   │   │   ├── Extensions/
│   │   │   │   ├── DecimalExtensions.cs
│   │   │   │   ├── DateTimeExtensions.cs
│   │   │   │   └── EnumerableExtensions.cs
│   │   │   │
│   │   │   ├── Helpers/
│   │   │   │   ├── CAGRCalculator.cs
│   │   │   │   ├── CoefficientOfVariationCalculator.cs
│   │   │   │   ├── WACCCalculator.cs
│   │   │   │   ├── TreasuryYieldHelper.cs
│   │   │   │   └── ValidationHelper.cs
│   │   │   │
│   │   │   └── Logging/
│   │   │       ├── ScreeningLogger.cs
│   │   │       └── LoggerFactory.cs
│   │   │
│   │   └── StockMarketScreener.Core.csproj
│   │
│   ├── StockMarketScreener.Shared/
│   │   ├── Constants/
│   │   │   ├── MetricThresholds.cs
│   │   │   ├── ScreeningConstants.cs
│   │   │   ├── ValuationGrades.cs
│   │   │   └── InvestmentRecommendations.cs
│   │   │
│   │   ├── Enums/
│   │   │   ├── ReportGroup.cs
│   │   │   ├── Recommendation.cs
│   │   │   ├── MarginOfSafetyStatus.cs
│   │   │   └── Industry.cs
│   │   │
│   │   ├── Exceptions/
│   │   │   ├── DataFetchException.cs
│   │   │   ├── ScreeningException.cs
│   │   │   ├── ValuationException.cs
│   │   │   └── ConfigurationException.cs
│   │   │
│   │   └── StockMarketScreener.Shared.csproj
│   │
│   └── StockMarketScreener.Tests/
│       ├── Unit/
│       │   ├── Services/
│       │   │   ├── MetricCalculationServiceTests.cs
│       │   │   ├── ValuationEngineTests.cs
│       │   │   └── ScreeningOrchestratorTests.cs
│       │   │
│       │   └── Utilities/
│       │       ├── CAGRCalculatorTests.cs
│       │       └── WACCCalculatorTests.cs
│       │
│       ├── Integration/
│       │   ├── ScreeningIntegrationTests.cs
│       │   └── DatabaseIntegrationTests.cs
│       │
│       └── StockMarketScreener.Tests.csproj
│
├── .github/
│   └── workflows/
│       ├── quarterly-screening.yml
│       └── weekly-stock-update.yml
│
├── docs/
│   ├── 0-BASIC-STAGE-WORKFLOW.md
│   ├── 01-INVESTMENT-REQUIREMENTS.md
│   ├── 02-VALUATION-METHODOLOGY-REQUIREMENTS.md
│   ├── 03-RESULTS-REPORTING-REQUIREMENTS.md
│   ├── 04-ARCHITECTURE-DESIGN.md
│   ├── 05-DATA-FLOW-SCREENING-LOGIC.md
│   └── 07-PROJECT-ROADMAP.md
│
├── config/
│   └── appsettings.json
│
├── output/
│   ├── reports/
│   ├── csv/
│   └── logs/
│
├── StockMarketScreener.sln
└── README.md
