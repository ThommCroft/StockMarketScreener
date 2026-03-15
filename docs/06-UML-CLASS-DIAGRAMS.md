@startuml

' Core Models
class Company {
    +String name
    +String ticker
}

class FinancialData {
    +Float revenue
    +Float netIncome
}

class InvestmentMetrics {
    +Float peRatio
    +Float dividendYield
}

class ScreeningResult {
    +Company company
    +Float score
}

class CompanyValuation {
    +Float valuation
}

Company --> FinancialData
FinancialData --> InvestmentMetrics
ScreeningResult --> Company
CompanyValuation --> Company


' Data Fetch Services
interface IDataFetchService {
    +List<FinancialData> fetchData(Company company)
}

class DataFetchService implements IDataFetchService {
    +List<FinancialData> fetchData(Company company)
}

class SecEdgarClient implements IDataFetchService {
    +List<FinancialData> fetchData(Company company)
}

class YahooFinanceClient implements IDataFetchService {
    +List<FinancialData> fetchData(Company company)
}

class IExCloudClient implements IDataFetchService {
    +List<FinancialData> fetchData(Company company)
}

class AlphaVantageClient implements IDataFetchService {
    +List<FinancialData> fetchData(Company company)
}



' Metric Calculation Services
interface IMetricCalculationService {
    +Float calculate(Company company)
}

class MetricCalculationService implements IMetricCalculationService {
    +Float calculate(Company company)
}

class ReturnOnCapitalCalculator implements IMetricCalculationService {
    +Float calculate(Company company)
}

class FinancialStrengthCalculator implements IMetricCalculationService {
    +Float calculate(Company company)
}

class ProfitabilityCalculator implements IMetricCalculationService {
    +Float calculate(Company company)
}

class CashFlowCalculator implements IMetricCalculationService {
    +Float calculate(Company company)
}

class GrowthCalculator implements IMetricCalculationService {
    +Float calculate(Company company)
}

class ValuationCalculator implements IMetricCalculationService {
    +Float calculate(CompanyValuation valuation)
}

class ManagementQualityCalculator implements IMetricCalculationService {
    +Float calculate(Company company)
}



' Screening Pipeline
class ScreeningOrchestrator {
    +List<ScreeningResult> runScreening()
}

class ScreeningService {
    +List<ScreeningResult> screenCompanies()
}

class Stage0PreFilter {
    +List<Company> preFilter()
}

class Stage1DataIngestion {
    +List<FinancialData> ingestData(List<Company> companies)
}

class Stage2FinancialStrength {
    +List<ScreeningResult> assessFinancialStrength(List<FinancialData> data)
}

class Stage3QualityScoring {
    +List<ScreeningResult> scoreQuality(List<Company> companies)
}

class Stage4CompositeScoring {
    +List<ScreeningResult> computeCompositeScores(List<ScreeningResult> results)
}

class Stage5ResultsProcessing {
    +void processResults(List<ScreeningResult> results)
}



' Valuation Engine
class ValuationEngine {
    +CompanyValuation valueCompany(Company company)
}

interface IDCFValuation {
    +Float performValuation(Company company)
}

interface IDDMValuation {
    +Float performValuation(Company company)
}

interface IMultiplesValuation {
    +Float performValuation(Company company)
}

interface IBuffettValuation {
    +Float performValuation(Company company)
}

interface IMungerValuation {
    +Float performValuation(Company company)
}

interface IMarginOfSafetyCalculator {
    +Float calculateMargin(CompanyValuation valuation)
}

class ConsensusValuationCalculator {
    +Float calculateConsensus(Company company)
}



' Reporting Services
interface IReportingService {
    +void generateReport()
}

class ReportingService implements IReportingService {
    +void generateReport()
}

class EmailReportGenerator implements IReportingService {
    +void generateReport()
}

class GitHubSummaryGenerator implements IReportingService {
    +void generateReport()
}

class CSVExportGenerator implements IReportingService {
    +void generateReport()
}

class ReportGroupingService {
    +void groupReports()
}



' Data Access Layer
interface IRepository {
    +void save()
}

class GenericRepository implements IRepository {
    +void save()
}

interface ICompanyRepository extends IRepository {
    +Company findById()
}

class CompanyRepository implements ICompanyRepository {
    +Company findById()
}

interface IScreeningRunRepository extends IRepository {
    +void recordScreeningRun()
}

interface ICompanyValuationRepository extends IRepository {
    +void saveValuation()
}

class StockMarketScreenerDbContext {
    +void contextMethod()
}



' Utility Classes
class CAGRCalculator {
    +Float calculate()
}

class CoefficientOfVariationCalculator {
    +Float calculate()
}

class WACCCalculator {
    +Float calculate()
}

class TreasuryYieldHelper {
    +Float getYield()
}

class ValidationHelper {
    +Boolean validate()
}

class DecimalExtensions {
    +Decimal extend()
}

class DateTimeExtensions {
    +DateTime extend()
}


' Shared Components
class Constants {
    +String APPLICATION_NAME
}

class Enums {
    +Enum TYPE
}

class Exceptions {
    +String message
}

@enduml