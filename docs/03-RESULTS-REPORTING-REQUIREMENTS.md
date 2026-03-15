# Stock Market Screener - Results Reporting Requirements

**Version:** 1.0  
**Date:** 2026-03-15  
**Author:** ThommCroft  
**Framework:** Warren Buffett & Charlie Munger Value Investing Principles  
**Stage:** Stage 7 - Financial Analysis and Intrinsic Value Results Reporting

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Introduction & Stage Context](#introduction--stage-context)
3. [Input Requirements](#input-requirements)
4. [Output Formats & Specifications](#output-formats--specifications)
   - 4.1 [Email Report Format](#email-report-format)
   - 4.2 [GitHub Summary Format](#github-summary-format)
   - 4.3 [CSV Export Format](#csv-export-format)
5. [Database Storage & Integration](#database-storage--integration)
   - 5.1 [CompanyValuations Table Schema](#company-valuations-table-schema)
   - 5.2 [Sample JSON Record](#sample-json-record)
   - 5.3 [Query Examples](#query-examples)
   - 5.4 [Data Retention Policy](#data-retention-policy)
6. [Comparison Reporting](#comparison-reporting)
7. [Implementation Notes](#implementation-notes)
8. [Quality Assurance & Validation](#quality-assurance--validation)
9. [Troubleshooting Guide](#troubleshooting-guide)
10. [Related Documents](#related-documents)
11. [Document History](#document-history)

---

## Executive Summary

### Purpose of Stage 7: Results Reporting

Stage 7 (Financial Analysis and Intrinsic Value Results Reporting) is the final stage of the stock screener pipeline. It takes all valuation results from Stage 6 and produces comprehensive reports in multiple formats for investment decision-making.

### Stage 7 in the Pipeline

**Input:** Companies evaluated in Stage 6 with complete valuation data:
- Fair values from 5 valuation methods (DCF, DDM, Multiples, Buffett, Munger)
- Consensus fair values and margins of safety
- Valuation grades and confidence levels
- Investment recommendations

**Process:** 
- Store all valuation results in MySQL database
- Generate email report with detailed analysis
- Create GitHub summary page
- Produce downloadable CSV file
- Attach CSV to email notification

**Output Formats:**
- ✅ MySQL Database storage (persistent historical record)
- ✅ Email report (detailed investment analysis)
- ✅ GitHub summary (quick statistics and rankings)
- ✅ CSV download (spreadsheet for further analysis)

**Next Step:** None - Stage 7 is the final stage

### Key Deliverables

1. **Email Report** - Comprehensive analysis grouped by opportunity level
2. **GitHub Summary** - Quick overview and top opportunities
3. **CSV Export** - Complete dataset for spreadsheet analysis
4. **Database Records** - Permanent historical record for tracking

### Why This Matters

Results reporting is critical because:
- ✅ Centralizes all findings in one place
- ✅ Makes investment decisions easier (ranked by priority)
- ✅ Provides historical tracking across screening runs
- ✅ Enables comparison with prior analyses
- ✅ Documents all assumptions and calculations
- ✅ Allows further analysis in spreadsheets

---

## Introduction & Stage Context

### How Stage 7 Fits in the Overall Pipeline

Stage 7 (Results Reporting) is the final phase of the stock screener, following Stage 6 (Intrinsic Value Valuation).

**Complete Pipeline:**

\`\`\`
Stages 0-5: Financial Analysis
    ↓
Stage 6: Intrinsic Value Valuation
    ↓
Stage 7: Results Reporting ← YOU ARE HERE (Final Stage)
\`\`\`

### What Happens Before Stage 7

All companies reaching Stage 7 have:
- ✅ Passed market cap filter (> $300M)
- ✅ Had all 40+ financial metrics calculated
- ✅ Passed financial strength assessment
- ✅ Received quality, valuation, and management scores
- ✅ Met composite score threshold (≥ 75)
- ✅ Been evaluated with 5 valuation methods
- ✅ Received consensus fair value and margin of safety
- ✅ Been assigned valuation grade and recommendation

### What Stage 7 Does

Stage 7 takes all this analysis and:
1. **Stores** all results in persistent database
2. **Groups** companies by opportunity level (Slam Dunks → Overvalued)
3. **Generates** detailed email reports for investment decisions
4. **Creates** GitHub summary for quick review
5. **Exports** CSV data for spreadsheet analysis
6. **Compares** results with prior screening runs
7. **Tracks** changes (new, maintained, removed companies)

### Output Audience

**Primary:** You (investor making decisions)

**Secondary:**
- GitHub repository (documentation/tracking)
- Database (historical analysis)
- CSV exports (deeper research)

---

## Input Requirements

### Required Input Data from Stage 6

For Stage 7 reporting, all data from Stage 6 must be available:

**Company Identifiers:**
- Ticker symbol
- Company name
- Sector/Industry
- Market capitalization
- Current stock price
- Shares outstanding

**Quality Scores (from Stages 3-5):**
- Quality Score (0-100)
- Valuation Score (0-100)
- Management Score (0-100)
- Composite Score (0-100)
- Pass/Fail status (Composite ≥ 75)

**Valuation Results (from Stage 6):**
- DCF Fair Value per share
- DDM Fair Value per share
- Multiples Fair Value per share
- Buffett Fair Value per share
- Munger Fair Value per share
- Consensus Fair Value per share

**Margin of Safety:**
- Margin of Safety % (Buffett)
- Margin of Safety % (Munger)
- Margin of Safety % (Consensus)
- Margin status (SLAM DUNK / STRONG BUY / FAIR VALUE / OVERVALUED)

**Recommendations:**
- Buffett recommendation (BUY/CONSIDER/PASS/AVOID)
- Munger recommendation (BUY/CONSIDER/PASS/AVOID)
- Consensus recommendation (BUY/CONSIDER/PASS/AVOID)
- Valuation grade (A+ to F)
- Investment thesis (brief narrative)

**Supporting Data:**
- Return on Equity (%)
- Net Profit Margin (%)
- Debt-to-Equity Ratio
- Free Cash Flow Score
- Moat Strength (1-10)
- Management Quality (1-10)
- Earnings Consistency (1-10)

### Data Quality Standards

All input data must meet these requirements:
- ✓ Complete (no missing critical values)
- ✓ Consistent (no contradictions)
- ✓ Recent (valuations current as of analysis date)
- ✓ Documented (source noted for each data point)

### Data Flow from Stage 6 to Stage 7

\`\`\`
Stage 6 Calculations
    ↓
Valuation Results Database
    ↓
Stage 7 Processing
    ├─→ Email Report Generation
    ├─→ GitHub Summary Creation
    ├─→ CSV Export
    └─→ Database Storage
    ↓
Outputs Delivered
\`\`\`

---

## Output Formats & Specifications

### 4.1 Email Report Format

The email report is the **PRIMARY OUTPUT** for investment decision-making.

#### Email Structure

TO: [Your Email] SUBJECT: Stock Market Screener Results - [Date] | [# Qualified] Companies

#### Email Header

📊 STOCK MARKET SCREENER RESULTS

Screening Date: [Date] Analysis Period: [Quarter/Year] Status: ✅ COMPLETE

#### Section 1: Executive Summary

**Screening Overview:**
- Companies Analyzed: [#]
- Passed Market Cap Filter (>$300M): [#]
- Passed Data Validation: [#]
- Passed Financial Strength Filters: [#]
- Passed Financial Analysis (Stage 5): [#] companies
- Complete Valuation Analysis: [#] companies

**Key Statistics:**
- Average Financial Quality Score: [#]/100
- Average Margin of Safety: [#]%
- Consensus Average Fair Value: $[#]
- Consensus Average Current Price: $[#]

**Distribution Across Groups:**
- GROUP 1 (Slam Dunks): [#] companies
- GROUP 2 (Strong Buys): [#] companies
- GROUP 3 (Fair Value): [#] companies
- GROUP 4 (Overvalued): [#] companies
- GROUP 5 (Not Qualified): [#] companies

**Comparison with Prior Run ([Prior Date]):**
- Newly Qualified: [#] companies
- Maintained Qualification: [#] companies
- Removed from Prior List: [#] companies

#### Section 2: GROUP 1 - SLAM DUNK BUYS

**Criteria:** Quality Score 80+, Margin of Safety 30%+

These are the best opportunities - exceptional businesses at attractive prices.

**For each company in Group 1, include:**

[TICKER] - [Company Name]
Snapshot: ├─ Current Price: $XXX.XX 
├─ Market Cap: $XXB 
├─ Sector: [Sector] 
├─ Quality Score: [#]/100 ⭐⭐⭐⭐ 
└─ Margin of Safety: [+#]%

Financial Quality Highlights: 
├─ Return on Equity: [#]% 
├─ Net Profit Margin: [#]% 
├─ Operating Margin: [#]% 
├─ Debt-to-Equity: [#] 
├─ Free Cash Flow: Positive (10/10 years) ✅ 
└─ Margin Stability: [#]% (CV < 15%) ✅

Valuation Summary:

DCF Method: 
├─ Fair Value: $XXX per share 
├─ Sensitivity Range: $X - $X 
└─ Base Case Growth: [#]% annually

DDM Method: [or N/A if non-dividend] 
├─ Fair Value: $XXX per share 
├─ Dividend Growth: [#]% historical 
└─ Payout Ratio: [#]% (sustainable)

Multiples Method: 
├─ P/E Fair Value: $XXX (vs peer avg XX.X) 
├─ P/B Fair Value: $XXX 
├─ P/S Fair Value: $XXX 
└─ Average Multiples Value: $XXX

Buffett Valuation: 
├─ Owner's Earnings: $X.XXB normalized 
├─ Quality Multiple: [#]x 
├─ Fair Value: $XXX per share 
└─ Assessment: [ASSESSMENT]

Munger Valuation: 
├─ Business Quality Score: [#]/10 
├─ Moat Strength: [#]/10 ([Type]) 
├─ Management Quality: [#]/10 
├─ Fair Value: $XXX per share 
└─ Five Stock Test: [YES/NO] ✅

Consensus Fair Value: $XXX per share 
├─ Conservative Estimate: $XXX (Buffett) 
├─ Base Case: $XXX (Weighted) 
└─ Optimistic Estimate: $XXX (DCF)

Margin of Safety Analysis: 
├─ Fair Value (Consensus): $XXX 
├─ Current Price: $XXX 
├─ Current Margin: +[#]% ✅ 
└─ Target Buy Prices: 
    ├─ 20% Safety Margin: $XXX/share 
    ├─ 30% Safety Margin: $XXX/share 
    ├─ 40% Safety Margin: $XXX/share 
    └─ 50% Safety Margin: $XXX/share

Valuation Grade: A+ (Slam Dunk Buy)

Buffett Assessment: 🟢 BUY 
└─ Intrinsic value significantly exceeds current price Margin of safety adequate (>25%) Owner's Earnings method supports purchase

Munger Assessment: 🟢 BUY 
└─ Exceptional business quality (8.5/10) Would include in 5-stock permanent portfolio Passes all quality tests, fair valuation

Consensus Recommendation: 🟢 STRONG BUY 
├─ Both Buffett and Munger recommend purchase 
├─ Build meaningful position at current price 
└─ Highest priority for capital allocation

Investment Thesis: 
[Company] is an exceptional business with a durable competitive advantage built through [moat type: brand/switching costs/network effects]. 
The company has demonstrated consistent earnings growth of [#]% annually... [2-3 paragraph summary of investment case]

#### Section 3: GROUP 2 - STRONG BUYS

**Criteria:** Quality Score 75-80, Margin of Safety 20-30%

Good to excellent businesses with fair valuations and adequate margins.

[Same detailed format as Group 1, but slightly condensed - focus on key metrics]

#### Section 4: GROUP 3 - FAIR VALUE

**Criteria:** Quality Score 82+, Margin of Safety 10-20%

Strong/excellent businesses at reasonable prices - good for accumulation.

[Format: Brief summary with key numbers only - less detail than Groups 1 & 2]

#### Section 5: GROUP 4 - OVERVALUED

**Criteria:** Quality Score 80+, Margin of Safety <0%

Excellent businesses but too expensive - watch for better prices.

**Note:** These are high-quality companies. The issue is valuation, not business quality. Add to watchlist and revisit when price declines.

[Format: Brief summary focused on valuation vs quality disconnect]

#### Section 6: GROUP 5 - NOT QUALIFIED

**Criteria:** Financial Score <75, Kept for Monitoring

These companies did not meet financial quality thresholds. Monitor for improvement:

[Format: Simple list with scores and reason for non-qualification]

#### Section 7: Summary Statistics & Insights

**Quality Score Distribution:**
- Average Quality Score: [#]
- Median Quality Score: [#]
- Highest Score: [#] ([Company])
- Lowest Qualified Score: [#] ([Company])

**Valuation Status:**
- Undervalued (MOS >20%): [#] companies
- Fairly Valued (MOS 0-20%): [#] companies
- Overvalued (MOS <0%): [#] companies

**Buffett vs Munger Agreement:**
- Both Recommend BUY: [#] companies
- Buffett BUY, Munger WAIT: [#] companies
- Buffett WAIT, Munger BUY: [#] companies
- Both Recommend PASS: [#] companies

**Investment Opportunity Score:**
- Slam Dunk Buys (A+): [#] companies - Highest priority
- Strong Buys (A): [#] companies - High priority
- Fair Value (B+/B): [#] companies - Medium priority
- Overvalued (C/D): [#] companies - Watchlist

#### Section 8: How to Use This Report

**For Immediate Investment Decisions:**

**Focus on Group 1 & 2 (Slam Dunks + Strong Buys)**
- 11 companies with best opportunities
- Prioritize by quality score (highest first)
- Build positions according to position sizing rules

**Monitor Group 3 (Fair Value)**
- Watch for small price dips
- Accumulate during weakness
- Good for dollar-cost averaging

**Watchlist Group 4 (Overvalued)**
- Set price alerts (25-30% below current)
- Review next quarter
- Buy when margin improves

**Review Group 5 (Not Qualified)**
- Check next quarter for improvements
- Some may requalify with better fundamentals
- Don't buy yet, even if other factors attractive

**For Deeper Research:**
- Compare Buffett vs Munger assessments (learn why they differ)
- Study investment theses to understand key value drivers
- Use sensitivity analysis to test valuation assumptions
- Review margin of safety calculations

**For Portfolio Management:**
- Current positions: Check if still in Groups 1-3 (still buy-worthy?)
- New allocation: Focus on Group 1 first, then Group 2
- Rebalancing: Consider trimming Group 4 positions
- Tax-loss harvesting: Review Group 5 candidates

#### Section 9: Detailed Data & Supporting Files

**Complete Data Available:**

- **CSV Export:** screening-results-[DATE].csv
  - All [#] companies with complete metrics
  - All valuations and margins of safety
  - Suitable for spreadsheet analysis

- **GitHub Summary:** GitHub Actions Run Summary
  - Quick statistics and charts
  - Previous run comparisons
  - Performance tracking

- **Database Records:** Stored in CompanyValuations table
  - Full audit trail of calculations
  - Historical tracking across quarters
  - Sensitivity analysis details

#### Section 10: Technical Notes

**Valuation Methodology:**
- DCF: 10-year projection, 2.5% terminal growth, WACC discount rate
- DDM: Gordon Growth Model with buyback adjustment where applicable
- Multiples: 5-peer average (trimmed for outliers), P/E, P/B, P/S
- Buffett: Owner's Earnings × Quality Multiple (15-35x)
- Munger: Quality-based multiple assignment (quality score method)

**Consensus Weighting:**
- DCF: 40% (growth projection focus)
- DDM: 15% (dividend/buyback validation, only if applicable)
- Multiples: 20% (market-based check)
- Buffett: 15% (mechanical simplicity)
- Munger: 10% (business judgment)

**Data Sources:**
- SEC EDGAR (10-K/10-Q financial statements)
- Yahoo Finance (stock prices, dividends, shares outstanding)
- Federal Reserve FRED (Treasury yields)
- Company investor relations (dividends, buybacks)

**Screening Date:** [Date]  
**Next Screening:** [Projected Date]

---

### 4.2 GitHub Summary Format

Brief summary posted to GitHub Actions run:

#### Page Title & Header

Stock Market Screener Results - [Date]

#### Summary Statistics Table

| Metric | Value |
|--------|-------|
| Companies Analyzed | [#] |
| Passed Financial Filters | [#] |
| Average Quality Score | [#] |
| Average Fair Value | $[#] |
| Average Current Price | $[#] |
| Average MOS | [+#]% |

#### Opportunities by Group Table

| Group | Count | Quality | MOS | Action |
|-------|-------|---------|-----|--------|
| Slam Dunk Buys | [#] | 85+ | 30%+ | Buy Aggressively |
| Strong Buys | [#] | 75-80 | 20-30% | Buy Confidently |
| Fair Value | [#] | 82+ | 10-20% | Accumulate |
| Overvalued | [#] | 80+ | <0% | Wait |
| Not Qualified | [#] | <75 | — | Monitor |

#### Top 5 Best Opportunities Table

| Rank | Ticker | Company | Quality | MOS | Price |
|------|--------|---------|---------|-----|-------|
| 1 | [TICK] | [Name] | [#] | [+#]% | $[#] |
| 2 | [TICK] | [Name] | [#] | [+#]% | $[#] |
| 3 | [TICK] | [Name] | [#] | [+#]% | $[#] |
| 4 | [TICK] | [Name] | [#] | [+#]% | $[#] |
| 5 | [TICK] | [Name] | [#] | [+#]% | $[#] |

#### Key Insights Section

- **Best Opportunity:** [Company] with [X%] margin of safety
- **Highest Quality:** [Company] with quality score [#]
- **Most Undervalued:** [Company] with [X%] margin of safety
- **Buffett/Munger Agreement:** [X] companies both recommend buy
- **New Qualifiers:** [X] companies newly qualified this run

#### Comparison with Prior Run

- Maintained: [#] companies
- New: [#] companies
- Removed: [#] companies
- Average quality improvement: [+#] points

#### Files & Links Section

- [View Full Report (Email Format)](link)
- [Download CSV Data](link)
- [View Detailed Company Analyses](link)
- [Methodology Documentation](link)

#### Footer

**Generated:** [Date] | **Next Run:** [Date]

---

### 4.3 CSV Export Format

All data in spreadsheet-friendly format:

#### CSV Column Definitions

| Column | Data Type | Description |
|--------|-----------|-------------|
| Ticker | String | Stock ticker symbol |
| Company Name | String | Full company name |
| Sector | String | Industry sector |
| Current Price | Decimal | Stock price at analysis date |
| Market Cap ($B) | Decimal | Market capitalization in billions |
| Quality Score | Integer | Quality score 0-100 from Stage 3 |
| DCF Fair Value | Decimal | Fair value per share from DCF method |
| DDM Fair Value | Decimal | Fair value per share from DDM method |
| Multiples Fair Value | Decimal | Fair value per share from Multiples method |
| Buffett Fair Value | Decimal | Fair value per share from Buffett method |
| Munger Fair Value | Decimal | Fair value per share from Munger method |
| Consensus Fair Value | Decimal | Weighted average of all methods |
| Current MOS % | Decimal | Margin of safety percentage |
| Valuation Grade | String | Grade A+ to F |
| Buffett Recommendation | String | BUY/CONSIDER/PASS/AVOID |
| Munger Recommendation | String | BUY/CONSIDER/PASS/AVOID |
| Consensus Recommendation | String | BUY/CONSIDER/PASS/AVOID |
| Group | String | GROUP 1-5 classification |
| ROE % | Decimal | Return on Equity percentage |
| Net Margin % | Decimal | Net profit margin percentage |
| D/E Ratio | Decimal | Debt-to-Equity ratio |
| FCF Score | Integer | Free cash flow quality score |
| Margin Stability | Decimal | Margin stability coefficient of variation |
| Moat Strength /10 | Integer | Economic moat strength 1-10 |
| Durability /10 | Integer | Business durability 1-10 |
| Management /10 | Integer | Management quality 1-10 |
| Earnings Consistency /10 | Integer | Earnings consistency 1-10 |
| ROIC /10 | Integer | Return on invested capital 1-10 |
| Investment Thesis | String | 1-2 paragraph investment thesis |

#### CSV Export Example

Ticker,Company Name,Sector,Current Price,Market Cap ($B),Quality Score,DCF Fair Value,DDM Fair Value,Multiples Fair Value,Buffett Fair Value,Munger Fair Value,Consensus Fair Value,Current MOS %,Valuation Grade,Buffett Recommendation,Munger Recommendation,Consensus Recommendation,Group,ROE %,Net Margin %,D/E Ratio,FCF Score,Margin Stability,Moat Strength /10,Durability /10,Management /10,Earnings Consistency /10,ROIC /10,Investment Thesis

MSFT,Microsoft Corporation,Technology,420.50,3140.0,82,523.15,0.00,342.29,122.00,304.00,345.60,-21.7,C,PASS,WAIT,PASS,4,35,33.2,0.04,10,6.2,9,9,9,9,8,"Microsoft is exceptional but expensive at $420. Quality score 82/100 shows strong fundamentals. However consensus fair value is $345.60 creating negative margin of safety of -21.7%. Watch for better entry point."

AAPL,Apple Inc.,Technology,180.00,2850.0,85,450.25,0.00,380.50,198.40,310.00,365.00,+50.7,A,BUY,BUY,STRONG BUY,1,100,29.8,0.15,10,5.1,10,9,9,9,9,"Exceptional business with unassailable moat trading at excellent value. Quality score 85/100 reflects strong fundamentals. Margin of safety +50.7% provides excellent downside protection. Strong buy at current price."

[... more rows ...]

---

## Database Storage & Integration

### 5.1 CompanyValuations Table Schema

**Store all valuation results for historical tracking and analysis:**

**SQL Table Definition**

```
CREATE TABLE CompanyValuations (
    ValuationID INT PRIMARY KEY AUTO_INCREMENT,
    CompanyID INT NOT NULL,
    ScreeningRunID INT NOT NULL,
    ScreeningDate DATETIME NOT NULL,
    
    -- Current Market Data
    CurrentStockPrice DECIMAL(10, 2) NOT NULL,
    MarketCapBillions DECIMAL(15, 2),
    SharesOutstandingBillions DECIMAL(10, 2),
    
    -- VALUATION METHOD 1: DCF
    DCFProjectionYears INT,
    DCFHistoricalFCFCAGR DECIMAL(5, 2),
    DCFProjectedGrowthRate DECIMAL(5, 2),
    DCFTerminalGrowthRate DECIMAL(5, 2),
    DCFDiscountRateWACC DECIMAL(5, 2),
    DCFFairValuePerShare DECIMAL(10, 2),
    DCFTerminalValue DECIMAL(15, 2),
    DCFSensitivityLow DECIMAL(10, 2),
    DCFSensitivityHigh DECIMAL(10, 2),
    
    -- VALUATION METHOD 2: DDM
    DDMApplicable BOOLEAN,
    DDMHistoricalDividendCAGR DECIMAL(5, 2),
    DDMProjectedGrowthRate DECIMAL(5, 2),
    DDMTerminalGrowthRate DECIMAL(5, 2),
    DDMDiscountRate DECIMAL(5, 2),
    DDMFairValuePerShare DECIMAL(10, 2),
    DDMAdjustedForBuybacks BOOLEAN,
    DDMPayoutRatioPercent DECIMAL(5, 2),
    
    -- VALUATION METHOD 3: MULTIPLES
    PEMultiplesFairValue DECIMAL(10, 2),
    PBMultiplesFairValue DECIMAL(10, 2),
    PSMultiplesFairValue DECIMAL(10, 2),
    MultiplesFairValueAverage DECIMAL(10, 2),
    ComparisonCompanies VARCHAR(255),
    PEMultiplePeerAverage DECIMAL(5, 2),
    PBMultiplePeerAverage DECIMAL(5, 2),
    PSMultiplePeerAverage DECIMAL(5, 2),
    
    -- BUFFETT VALUATION TECHNIQUE
    BuffettOwnerEarningsBillions DECIMAL(15, 2),
    BuffettQualityMultiple INT,
    BuffettFairValuePerShare DECIMAL(10, 2),
    BuffettEarningsYield DECIMAL(5, 3),
    BuffettRequiredReturn DECIMAL(5, 2),
    BuffettEarningsYieldAssessment VARCHAR(50),
    
    -- MUNGER VALUATION TECHNIQUE
    MungerBusinessQualityScore INT,
    MungerMoatStrength INT,
    MungerDurabilityScore INT,
    MungerManagementQualityScore INT,
    MungerEarningsConsistencyScore INT,
    MungerROCScore INT,
    MungerQualityMultiple INT,
    MungerFairValuePerShare DECIMAL(10, 2),
    MungerFiveStockTestResult VARCHAR(50),
    MungerOverallAssessment VARCHAR(255),
    
    -- CONSENSUS VALUATION
    ConsensusMethod VARCHAR(50),
    ConsensusFairValuePerShare DECIMAL(10, 2),
    ConsensusWeightedFairValue DECIMAL(10, 2),
    
    -- MARGIN OF SAFETY
    BuffettMarginOfSafetyPercent DECIMAL(5, 2),
    BuffettMarginStatus VARCHAR(50),
    BuffettBuyPrice20MOS DECIMAL(10, 2),
    BuffettBuyPrice30MOS DECIMAL(10, 2),
    BuffettBuyPrice40MOS DECIMAL(10, 2),
    BuffettBuyPrice50MOS DECIMAL(10, 2),
    
    MungerMarginOfSafetyPercent DECIMAL(5, 2),
    MungerMarginStatus VARCHAR(50),
    MungerBuyPrice20MOS DECIMAL(10, 2),
    MungerBuyPrice30MOS DECIMAL(10, 2),
    MungerBuyPrice40MOS DECIMAL(10, 2),
    MungerBuyPrice50MOS DECIMAL(10, 2),
    
    ConsensusMarginOfSafetyPercent DECIMAL(5, 2),
    ConsensusMarginStatus VARCHAR(50),
    ConsensusBuyPrice20MOS DECIMAL(10, 2),
    ConsensusBuyPrice30MOS DECIMAL(10, 2),
    ConsensusBuyPrice40MOS DECIMAL(10, 2),
    ConsensusBuyPrice50MOS DECIMAL(10, 2),
    
    -- VALUATION GRADE
    ValuationGrade VARCHAR(3),
    ValuationGradeReasoning VARCHAR(500),
    
    -- RECOMMENDATIONS
    BuffettRecommendation VARCHAR(50),
    MungerRecommendation VARCHAR(50),
    ConsensusRecommendation VARCHAR(50),
    GroupingCategory VARCHAR(50),
    InvestmentThesis VARCHAR(1000),
    
    -- AUDIT TRAIL
    CreatedAt DATETIME DEFAULT CURRENT_TIMESTAMP,
    UpdatedAt DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    DataSourcesUsed VARCHAR(255),
    CalculationNotes VARCHAR(500),
    
    FOREIGN KEY (CompanyID) REFERENCES Companies(CompanyID),
    FOREIGN KEY (ScreeningRunID) REFERENCES ScreeningRuns(ScreeningRunID),
    INDEX idx_date (ScreeningDate),
    INDEX idx_company (CompanyID),
    INDEX idx_grade (ValuationGrade)
);
```

---

### 5.2 Sample JSON Record

**Example of how a single company's valuation is stored:**

```
{
  "ValuationID": 12547,
  "CompanyID": 145,
  "ScreeningRunID": 89,
  "ScreeningDate": "2026-03-14",
  "CurrentStockPrice": 420.50,
  "MarketCapBillions": 3140.0,
  "DCFFairValuePerShare": 523.15,
  "DDMFairValuePerShare": null,
  "MultiplesFairValueAverage": 342.29,
  "BuffettFairValuePerShare": 122.00,
  "MungerFairValuePerShare": 304.00,
  "ConsensusFairValuePerShare": 345.60,
  "ConsensusMarginOfSafetyPercent": -21.7,
  "ConsensusMarginStatus": "OVERVALUED",
  "ValuationGrade": "C",
  "BuffettRecommendation": "PASS",
  "MungerRecommendation": "WAIT",
  "ConsensusRecommendation": "PASS",
  "GroupingCategory": "OVERVALUED",
  "InvestmentThesis": "Exceptional business but too expensive at current valuation..."
}
```

---

### 5.3 Query Examples

**Common queries for analyzing historical data:**

**Query 1: Best Opportunities Across All Runs**

```
SELECT 
    Ticker,
    CompanyName,
    ScreeningDate,
    ConsensusFairValuePerShare,
    CurrentStockPrice,
    ConsensusMarginOfSafetyPercent,
    ValuationGrade,
    ConsensusRecommendation
FROM CompanyValuations cv
JOIN Companies c ON cv.CompanyID = c.CompanyID
WHERE ConsensusMarginOfSafetyPercent > 25
  AND ValuationGrade IN ('A+', 'A')
ORDER BY ScreeningDate DESC, ConsensusMarginOfSafetyPercent DESC;
```

**Query 2: Track Quality Improvement Over Time**

```
SELECT 
    Ticker,
    CompanyName,
    ScreeningDate,
    QualityScore,
    ConsensusRecommendation,
    ConsensusMarginOfSafetyPercent
FROM CompanyValuations cv
JOIN Companies c ON cv.CompanyID = c.CompanyID
WHERE CompanyID = [SPECIFIC_COMPANY_ID]
ORDER BY ScreeningDate DESC;
```

**Query 3: New Qualifiers This Run**

```
SELECT 
    c.Ticker,
    c.CompanyName,
    cv.ScreeningDate,
    cv.ConsensusRecommendation,
    cv.ValuationGrade
FROM CompanyValuations cv
JOIN Companies c ON cv.CompanyID = c.CompanyID
WHERE ScreeningRunID = [CURRENT_RUN_ID]
  AND CompanyID NOT IN (
    SELECT DISTINCT CompanyID 
    FROM CompanyValuations 
    WHERE ScreeningRunID = [PRIOR_RUN_ID]
  );
```

---

###5.4 Data Retention Policy

**Historical Data:**
✅ All data retained permanently
✅ Database acts as audit trail
✅ Enables historical trend analysis
✅ Tracks quality improvements/deterioration

**Backup Schedule:**
Daily incremental backups (retain 30 days)
Weekly full backups (retain 12 weeks)
Monthly snapshots (retain 12 months)
Quarterly archives (retain indefinitely)

**Data Access:**
Historical queries always available
Can compare any two screening runs
Can identify long-term trends
Supports investment decision-making over time
Comparison Reporting
Change Detection Logic

**New Companies:**
In current run ✅
NOT in prior run ❌
Action: Highlight as new opportunity

**Maintained Companies:**
In current run ✅
In prior run ✅
Action: Track if quality changed

**Removed Companies:**
In current run ❌
In prior run ✅
Action: Note reason for removal
Metrics Tracked in Comparison
For maintained companies, note changes in:

Quality Score change (up/down)
Margin of Safety change
Recommendation change (upgraded/downgraded)
Fair value change
Prior Run Comparison Section in Email

```
Comparison with Prior Run ([Prior Date]):

Newly Qualified: [#] companies
├─ Example: [TICKER] - [Reason for qualification]
├─ Example: [TICKER] - [Reason for qualification]
└─ Example: [TICKER] - [Reason for qualification]

Maintained Qualification: [#] companies
├─ Quality improved: [#] companies
├─ Quality declined: [#] companies
├─ Quality stable: [#] companies
└─ Average quality change: [+#] points

Removed from Prior List: [#] companies
├─ Example: [TICKER] - Composite score dropped below 75
├─ Example: [TICKER] - Financial deterioration
└─ Example: [TICKER] - Valuation no longer attractive
```

---

###**Implementation Notes**
Email Delivery Specifications

**Email Service:**
Service: AWS SES (Simple Email Service) or SendGrid
From: [noreply@yourdomain.com]
To: [Your email address]
Frequency: [Weekly/Monthly/As needed]

**Email Construction:**
Format: HTML with styled tables
Attachments: CSV file (screening-results-[DATE].csv)
Size limit: Keep under 25MB (CSV usually <5MB)
Encoding: UTF-8

**Timing:**
Send time: [Specify desired time of day]
Timezone: [Your timezone]
Retry logic: Retry 3 times if delivery fails
GitHub Actions Integration

**Workflow Trigger:**
Event: Scheduled (e.g., every Sunday at 8 AM)
Or: Manual trigger via GitHub Actions
Or: Webhook trigger from data pipeline

**GitHub Output:**
Create/Update GitHub Actions summary
Post to repository README
Create issue for each new opportunity (optional)
Add to project board (optional)

**GitHub Files Created:**

```
/reports/screening-results-[DATE].md
/data/screening-results-[DATE].csv
/reports/archive/[PREVIOUS_RESULTS]
```

---

###CSV Generation Process

**File Location:**
Path: /data/screening-results-[DATE].csv
Filename: screening-results-YYYY-MM-DD.csv
Format: UTF-8 encoding, comma-delimited

**File Contents:**
Header row with column names
One row per qualified company
All data fields populated
Numbers formatted to 2 decimal places

**File Attachment:**
Attached to email
Also available in GitHub repository
Suitable for import into Excel/Google Sheets
Database Connection Details

**Connection String:**

```
Server=[DB_HOST]
Database=[DB_NAME]
User ID=[DB_USER]
Password=[DB_PASSWORD]
Port=3306
```

**Connection Pool:**
Min connections: 5
Max connections: 20
Connection timeout: 30 seconds
Command timeout: 60 seconds

**Transaction Handling:**
Insert all valuation records in single transaction
If any record fails, rollback all
Log transaction details to audit table
Quality Assurance & Validation
Output Checks Before Delivery

**Email Report Validation:**
✓ All groups have correct companies
✓ All financial metrics present
✓ No missing valuation data
✓ Recommendations consistent with margins
✓ Formatting is clean and readable
✓ Mathematical calculations correct

**GitHub Summary Validation:**
✓ Tables render correctly
✓ Statistics match email report
✓ Links are functional
✓ Rankings are correct

**CSV Export Validation:**
✓ All columns populated
✓ No empty required fields
✓ Data types correct
✓ Numbers within reasonable ranges
✓ File can be opened in Excel
✓ Encoding correct (UTF-8)

**Database Validation:**
✓ All records inserted successfully
✓ Foreign keys valid
✓ No duplicates
✓ Timestamps correct
✓ Indexes created

---

###Error Handling

**If Email Fails:**
Log error with timestamp
Retry up to 3 times with exponential backoff
Send alternate notification (Slack, Teams)
Alert user to manual download

**If Database Write Fails:**
Rollback entire transaction
Log detailed error
Save data to backup JSON files
Alert user to investigate

**If GitHub Actions Fails:**
Log error to action logs
Create GitHub issue for manual review
Email user with error details
Suggest remediation steps
Fallback Procedures

**Email Delivery:**
Try AWS SES
Fall back to SendGrid
Fall back to SMTP server
As last resort: save to file for manual sending

**GitHub:**
Try GitHub API
Fall back to commit-based upload
Fall back to manual file creation

**Database:**
Try primary database
Fall back to read replica
Fall back to file-based backup
Troubleshooting Guide
Common Issues & Solutions
Issue #1: Email Not Received

**Possible Causes:**
Email service credentials invalid
Domain SPF/DKIM not configured
Email in spam folder
Email size too large

**Solutions:**
Verify email service credentials
Check domain SPF/DKIM records
Check spam/junk folder
Check email attachment size
Try removing attachments first
Issue #2: CSV File Has Corrupt Data

**Possible Causes:**
Unicode/encoding issues
Division by zero in calculations
Missing company data from Stage 6
NULL values in database

**Solutions:**
Check file encoding (should be UTF-8)
Verify no division by zero in calculations
Check all Stage 6 data is complete
Handle NULL values properly (use 0 or N/A)
Issue #3: GitHub Summary Won't Render

**Possible Causes:**
Markdown syntax errors
Links pointing to wrong locations
Table formatting incorrect
Special characters not escaped

**Solutions:**
Validate markdown syntax
Test all links before posting
Check table column alignment
Escape special characters
Issue #4: Database Query Timeout

**Possible Causes:**
Too many records to process
Missing indexes
Network connectivity issues
Database server overloaded

**Solutions:**
Add appropriate indexes
Use pagination for large result sets
Check network connectivity
Check database server resources
Increase query timeout if needed
Issue #5: Recommendations Don't Match Margins

**Possible Causes:**
Margin of safety calculation error
Recommendation logic incorrect
Data not synced from Stage 6
Grade/recommendation mismatch

**Solutions:**
Verify margin of safety formula
Check recommendation decision tree
Validate data from Stage 6
Ensure grade matches recommendation

---

###Related Documents

| Document | Covers | Purpose|
| 0-BASIC-STAGE-WORKFLOW.md | All Stages | Overall pipeline and workflow definitions |
|--------|-------|
| 01-INVESTMENT-REQUIREMENTS.md | Stages 0-5 | Financial analysis that feeds into Stages 6-7 |
|--------|-------|
| 02-VALUATION-METHODOLOGY.md | Stage 6 | Intrinsic value calculation that feeds into Stage 7 |
|--------|-------|
| 03-RESULTS-REPORTING-REQUIREMENTS.md | Stage 7 | Results reporting and output specifications (this document) |

---

###Document History

| Version | Date | Author | Changes |
|--------|-------|
| 1.0 | 2026-03-15 | ThommCroft | CURRENT - Initial comprehensive results reporting requirements document; aligned with 0-BASIC-STAGE-WORKFLOW.md as Stage 7; reorganized and expanded from original content |

---

**End of Document**

For questions on output formats, database integration, or reporting procedures, refer to the relevant sections above or the supporting Stage 6 documentation.
