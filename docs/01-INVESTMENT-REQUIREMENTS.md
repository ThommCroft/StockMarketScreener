# Investment Requirements Document - Version 3.0

**Version:** 3.0 (Complete with All Metrics)  
**Date:** 2026-03-13  
**Author:** ThommCroft  
**Framework:** Warren Buffett & Charlie Munger Value Investing Principles + Complete Spreadsheet Metrics

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Investment Philosophy](#investment-philosophy)
3. [Complete Metrics Reference](#complete-metrics-reference)
4. [Screening Criteria by Category](#screening-criteria-by-category)
5. [Priority Classification](#priority-classification)
6. [Implementation Notes](#implementation-notes)
7. [Data Sources](#data-sources)

---

## Executive Summary

This document defines **strict, quantifiable criteria** for identifying high-quality value stocks based on Warren Buffett and Charlie Munger's proven investment principles, combined with **all metrics from the spreadsheet analysis tool**. The screener evaluates companies across **all industries** using a multi-stage filtering approach:

- **Stage 1 (MUST HAVE)**: Financial Strength - Hard filters eliminating weak businesses
- **Stage 2 (SHOULD HAVE)**: Quality Assessment - Deep analysis of business quality and earnings
- **Stage 3 (SHOULD HAVE)**: Valuation & Management - Identifies opportunities with safety margin

**Target Universe**: All US-listed equities with market cap > $300M

**Total Metrics Tracked**: 40+ quantitative and qualitative factors

---

## Investment Philosophy

### Buffett/Munger Core Principles

#### 1. Circle of Competence & Business Understandability
- **Principle**: Only invest in businesses you can understand
- **Application**: Avoid complex financial structures, derivatives, and opaque business models
- **Metrics**: Business Model Simplicity (qualitative), Market Position, Industry Classification
- **Why It Matters**: Buffett and Munger only invest in companies whose business models they can fully comprehend. Complex businesses with hard-to-understand economics are avoided, as they're prone to hidden risks and surprises.
- **Implementation**: Screen for companies with straightforward operations, transparent financial statements, and clear revenue models

#### 2. Durable Competitive Advantage (Economic Moat)
- **Principle**: Sustainable competitive advantages create pricing power and protect market share
- **Types of Moats**:
  - **Brand Strength** (e.g., Coca-Cola, Apple) - Customers choose the brand despite alternatives
  - **Switching Costs** (e.g., Intuit, Salesforce, Microsoft) - High cost for customers to switch
  - **Network Effects** (e.g., Mastercard, Visa) - More valuable as more people use it
  - **Cost Advantages** (e.g., Costco, Walmart) - Sustainable lower cost structure
- **Why It Matters**: A moat allows a company to maintain pricing power and defend market share against competition, leading to predictable long-term profits
- **Metrics**: Market Position, Recurring Revenue %, Margin Stability, Customer Concentration
- **Implementation**: Look for companies that have maintained or expanded market share over 10+ years despite competition

#### 3. Financial Fortress (Balance Sheet Strength)
- **Principle**: Strong balance sheets survive downturns and create optionality for opportunities
- **Requirements**:
  - Low debt levels (D/E < 0.50) - Financial flexibility
  - Positive free cash flow - Real cash available to shareholders
  - Adequate liquidity (Current Ratio 1.5-2.5x) - Can meet obligations
- **Why It Matters**: Financial strength protects companies during economic downturns and allows them to invest in growth or acquisitions when competitors struggle
- **Metrics**: Debt-to-Equity, D/E Trend, Current Ratio, Interest Coverage, Total Equity Trend
- **Implementation**: Reject companies with weak balance sheets; favor those with declining debt and strong liquidity

#### 4. Consistent, Growing Earnings
- **Principle**: Real earnings (cash flow) matter more than accounting earnings; consistency is key
- **Requirements**:
  - Consistent profitability over 10+ years (low volatility)
  - Growing earnings over time
  - High-quality earnings (FCF > Net Income indicates minimal accounting tricks)
- **Why It Matters**: Companies with erratic earnings are unpredictable and risky. Buffett wants to see companies that reliably generate more earnings year after year
- **Metrics**: ROE Stability, Margin Consistency, FCF Conversion Ratio, OCF/FCF Growth
- **Implementation**: Track 10-year trends; reject companies with erratic earnings or declining profitability

#### 5. Management Quality
- **Principle**: Competent management aligned with shareholders is crucial; you're investing in people
- **Requirements**:
  - Significant insider ownership (skin in the game)
  - Honest, transparent communication with shareholders
  - Conservative accounting practices (no aggressive or creative accounting)
  - Smart capital allocation (reinvestment, buybacks, dividends)
- **Why It Matters**: Management drives all business decisions. Poor management can destroy even good businesses. Insider ownership aligns management incentives with shareholder value
- **Metrics**: Management Insider Ownership %, Goodwill Levels, One-Time Charges Frequency, Accounting Quality
- **Red Flags**: CEO with minimal ownership, frequent restatements, aggressive accounting, high goodwill impairments
- **Implementation**: Research management background; look for long tenure and strong track records

#### 6. Margin of Safety
- **Principle**: Buy at significant discount to intrinsic value; provides protection against error
- **Requirements**:
  - Reasonable valuation multiples (P/E, P/B below market)
  - Earnings yield > risk-free rate (better than Treasury bonds)
  - 25-50% margin of safety is ideal
- **Why It Matters**: Even great businesses can be bad investments if overpaid. A margin of safety protects against errors in judgment and provides upside potential
- **Metrics**: P/E Ratio, P/B Ratio, P/S Ratio, PEG Ratio, Earnings Yield, Discount to Intrinsic Value
- **Implementation**: Calculate intrinsic value; only buy when price is significantly below fair value

---

## Complete Metrics Reference

### Quick Lookup - All 40+ Metrics by Category

| Metric | Threshold | Period | Priority | Category |
|--------|-----------|--------|----------|----------|
| Return on Equity (ROE) | > 15% | 10Y avg | **MUST** | Return on Capital |
| ROE Trend | Stable/Growing | 10Y | **MUST** | Return on Capital |
| Return on Assets (ROA) | > 10% | 10Y avg | SHOULD | Return on Capital |
| Return on Invested Capital (ROIC) | > 15% | 10Y avg | SHOULD | Return on Capital |
| Debt-to-Equity Ratio | < 0.50 | Current | **MUST** | Financial Strength |
| D/E Trend | Stable/Declining | 10Y | **MUST** | Financial Strength |
| Liabilities-to-Equity | < 0.75 | Current | SHOULD | Financial Strength |
| Current Ratio | 1.5-2.5x | Current | **MUST** | Financial Strength |
| Interest Coverage Ratio | > 5.0x | Latest FY | SHOULD | Financial Strength |
| Total Equity Trend | Growing/Stable | 10Y | SHOULD | Financial Strength |
| Net Profit Margin | > 10% | 10Y avg | **MUST** | Profitability |
| Net Margin Trend | Stable/Growing | 10Y | **MUST** | Profitability |
| Operating Margin | > 10% | 10Y avg | **MUST** | Profitability |
| Operating Margin Trend | Stable/Growing | 10Y | **MUST** | Profitability |
| Gross Profit Margin | > 30% | 10Y avg | SHOULD | Profitability |
| Margin Stability (CV) | < 15% | 10Y | **MUST** | Profitability |
| Depreciation Margin | Monitor | 10Y | SHOULD | Profitability |
| Interest Expense Margin | Monitor | 10Y | SHOULD | Profitability |
| Operating Cash Flow | Positive | Each year (10Y) | **MUST** | Cash Flow |
| OCF Growth | > 0% | 10Y CAGR | **MUST** | Cash Flow |
| Free Cash Flow | Positive | Each year (10Y) | **MUST** | Cash Flow |
| FCF Growth | > 0% | 10Y CAGR | **MUST** | Cash Flow |
| FCF-to-Net-Income Ratio | > 70% | 10Y avg | SHOULD | Cash Flow |
| OCF Stability (CV) | < 20% | 10Y | SHOULD | Cash Flow |
| Revenue Growth (CAGR) | 4-8% | 10Y CAGR | SHOULD | Growth |
| Revenue Growth Stability | < 20% CV | 10Y | SHOULD | Growth |
| EPS Growth | > Revenue Growth | 10Y CAGR | SHOULD | Growth |
| EPS Growth Stability | Low volatility | 10Y | SHOULD | Growth |
| CapEx as % Revenue | < 5-10% | 10Y avg | SHOULD | Capital Efficiency |
| CapEx as % OCF | < 50% | 10Y avg | SHOULD | Capital Efficiency |
| Retained Earnings Trend | Growing/Stable | 10Y | SHOULD | Reinvestment |
| P/E Ratio | < 25x (15-20x ideal) | Current | SHOULD | Valuation |
| P/B Ratio | < 3.0x (< 2.0x ideal) | Current | SHOULD | Valuation |
| P/S Ratio | < 2.5x | Current | SHOULD | Valuation |
| PEG Ratio | < 1.5x | Current | SHOULD | Valuation |
| Earnings Yield | > 10Y Treasury + 4% | Current | SHOULD | Valuation |
| Dividend Yield | 2-4% (if applicable) | Current | SHOULD | Valuation |
| Payout Ratio | 30-60% (if applicable) | Latest | SHOULD | Valuation |
| Management Insider Ownership | > 5% | Current | SHOULD | Management |
| Goodwill Level | < 20% of assets | Current | SHOULD | Management |
| Goodwill Impairments | Minimal/Declining | 10Y history | SHOULD | Management |
| One-Time Charges | < 1 per 3 years | 10Y history | SHOULD | Management |
| Accounting Quality | Conservative | Qualitative | SHOULD | Management |
| Management Tenure | 5+ years | Current | NICE | Management |
| Economic Moat | Identifiable | Qualitative | SHOULD | Competitive Advantage |
| Market Position | Leader or #2-3 | Qualitative | SHOULD | Competitive Advantage |
| Recurring Revenue % | > 50% (where applicable) | Business Model | SHOULD | Competitive Advantage |
| Brand Strength | Recognized/Valued | Qualitative | NICE | Competitive Advantage |
| Customer Concentration | No customer > 20% | Current | SHOULD | Competitive Advantage |
| Industry Cyclicality | Non-cyclical | Qualitative | SHOULD | Competitive Advantage |
| Business Model Simplicity | Easy to understand | Qualitative | **MUST** | Competitive Advantage |
| Market Capitalization | > $300M | Current | SHOULD | Size/Liquidity |

---

## Screening Criteria by Category

### Category 1: Return on Capital (Business Quality Foundation)

These metrics demonstrate whether management efficiently deploys shareholder capital.

| Metric | Threshold | Period | Priority | Notes |
|--------|-----------|--------|----------|-------|
| **Return on Equity (ROE)** | > 15% | 10-year average | **MUST HAVE** | Core Buffett metric; consistent high ROE = excellent management |
| **ROE Trend** | Stable or growing | Last 10 years | **MUST HAVE** | Declining ROE is red flag; quality deteriorating |
| **Return on Assets (ROA)** | > 10% | 10-year average | **SHOULD HAVE** | Complements ROE; shows asset efficiency |
| **Return on Invested Capital (ROIC)** | > 15% | 10-year average (calculated) | **SHOULD HAVE** | Better than ROE; evaluates all capital (debt + equity) |

**Implementation Notes:**
- Calculate as: Net Income ÷ Shareholders' Equity
- Use 10-year average to smooth business cycles
- Flag if declining trend evident
- For ROIC: NOPAT ÷ (Total Assets - Current Liabilities)

**Rejection Rule:**
- ROE < 10% = automatic rejection (Stage 1)
- ROE > 15% but declining trend = watch carefully

---

### Category 2: Financial Strength & Balance Sheet Health

Low debt and strong liquidity provide financial flexibility and reduce risk.

| Metric | Threshold | Period | Priority | Notes |
|--------|-----------|--------|----------|-------|
| **Debt-to-Equity Ratio** | < 0.50 | Current (latest filing) | **MUST HAVE** | Conservative leverage; healthy buffer for downturns |
| **D/E Trend** | Declining or stable | Last 10 years | **MUST HAVE** | Increasing debt is red flag |
| **Total Liabilities-to-Equity** | < 0.75 | Current | **SHOULD HAVE** | Total obligation assessment |
| **Current Ratio** | 1.5 - 2.5x | Current | **MUST HAVE** | Can meet short-term obligations comfortably |
| **Interest Coverage Ratio** | > 5.0x | Latest fiscal year | **SHOULD HAVE** | Can easily service debt (EBIT ÷ Interest Expense) |
| **Total Equity Trend** | Growing or stable | Last 10 years | **SHOULD HAVE** | Strengthening balance sheet |

**Implementation Notes:**
- D/E = Total Debt ÷ Total Equity
- Monitor year-over-year change
- Current Ratio = Current Assets ÷ Current Liabilities
- Interest Coverage = EBIT ÷ Interest Expense
- Companies with negative EBIT = automatic rejection

**Rejection Rules:**
- D/E > 0.75 = automatic rejection
- D/E < 0.50 but rapidly increasing = watch carefully
- Current Ratio < 1.2 or > 3.0 = investigate

---

### Category 3: Profitability & Operational Quality

Profit margins reveal pricing power, competitive position, and operational efficiency.

| Metric | Threshold | Period | Priority | Notes |
|--------|-----------|--------|----------|-------|
| **Net Profit Margin** | > 10% | 10-year average | **MUST HAVE** | High margins = strong competitive position |
| **Net Margin Trend** | Stable or growing | Last 10 years | **MUST HAVE** | Declining margins = competitive pressure |
| **Operating Margin** | > 10% | 10-year average | **MUST HAVE** | Core business profitability |
| **Operating Margin Trend** | Stable or growing | Last 10 years | **MUST HAVE** | Shows operational improvement or deterioration |
| **Gross Profit Margin** | > 30% | 10-year average | **SHOULD HAVE** | Indicates pricing power (lower for capital-intensive) |
| **Margin Stability** | Coefficient of Variation < 15% | Last 10 years | **MUST HAVE** | Predictable, consistent business |
| **Depreciation Margin** | Monitor trend | Last 10 years | **SHOULD HAVE** | Indicates capital intensity of business |
| **Interest Expense Margin** | Monitor trend | Last 10 years | **SHOULD HAVE** | Track debt burden trend |

**Implementation Notes:**
- Net Margin = Net Income ÷ Revenue
- Operating Margin = Operating Income ÷ Revenue
- Gross Margin = (Revenue - COGS) ÷ Revenue
- Calculate coefficient of variation: StdDev(Margins) ÷ Mean(Margins)
- Industry context matters (retail vs. software vs. utilities)

**Rejection Rules:**
- Declining margins for 3+ years = reject
- Negative operating margin = reject
- Margin volatility > 20% = reject (unpredictable)

**What Buffett/Munger REJECT:**
- ❌ EBITDA-centric analysis (ignores capital needs)
- ❌ Pro forma earnings (manipulated results)
- ❌ Adjusted earnings without justification

---

### Category 4: Cash Flow Quality (The Ultimate Test)

Cash flow is harder to manipulate than earnings. "Real earnings" = operating cash flow.

| Metric | Threshold | Period | Priority | Notes |
|--------|-----------|--------|----------|-------|
| **Operating Cash Flow (OCF)** | Positive | Latest 10 years (each year) | **MUST HAVE** | Real cash from operations, not accounting fiction |
| **OCF Growth** | Positive trend | 10-year average | **MUST HAVE** | Sustainable cash generation improving |
| **Free Cash Flow (FCF)** | Positive | Latest 10 years (each year) | **MUST HAVE** | Cash available to shareholders after capex |
| **FCF Growth** | Positive trend | 10-year average | **MUST HAVE** | Cash available for debt paydown, dividends, buybacks |
| **FCF-to-Net-Income Ratio** | > 70% | 10-year average | **SHOULD HAVE** | High-quality earnings (not accounting games) |
| **Operating Cash Flow Stability** | Coefficient of Variation < 20% | Last 10 years | **SHOULD HAVE** | Predictable cash generation |

**Implementation Notes:**
- OCF = Net Income + D&A - Changes in Working Capital (from cash flow statement)
- FCF = OCF - Capital Expenditures
- FCF/NI Ratio < 0.60 indicates earnings quality issues
- If OCF < 0 any year = automatic reject

**Rejection Rules:**
- Any year with negative OCF or FCF = reject
- FCF declining for 3+ years = reject
- FCF/NI < 0.50 = earnings quality concern

---

### Category 5: Growth & Capital Efficiency

Growth without profitability destroys value. Efficient capital deployment matters.

| Metric | Threshold | Period | Priority | Notes |
|--------|-----------|--------|----------|-------|
| **Revenue Growth** | 4-8% annually (sustainable) | 10-year CAGR | **SHOULD HAVE** | Healthy organic growth |
| **Revenue Growth Stability** | Coefficient of Variation < 20% | Last 10 years | **SHOULD HAVE** | Predictable, consistent growth |
| **Earnings Per Share (EPS) Growth** | Exceeds revenue growth | 10-year CAGR | **SHOULD HAVE** | Operating leverage, improving margins |
| **EPS Growth Stability** | Low volatility | Last 10 years | **SHOULD HAVE** | Predictable earnings growth |
| **CapEx as % of Revenue** | < 5-10% | 10-year average | **SHOULD HAVE** | Capital-light business model (lower is better) |
| **CapEx as % of Operating Cash Flow** | < 50% | 10-year average | **SHOULD HAVE** | Plenty of cash available after reinvestment |
| **Retained Earnings Trend** | Growing or stable | Last 10 years | **SHOULD HAVE** | Reinvestment of profits / shareholder value creation |

**Implementation Notes:**
- Revenue CAGR = (Revenue_End ÷ Revenue_Start)^(1/Years) - 1
- CapEx = Capital Expenditures from cash flow statement
- Monitor both absolute growth and stability
- If EPS declining despite revenue growth = margin compression (red flag)

**Rejection Rules:**
- Negative revenue growth for 2+ years = reject
- Revenue growing but earnings declining = reject (unsustainable)
- CapEx > 20% of revenue = capital-intensive (may still be acceptable)

---

### Category 6: Valuation & Margin of Safety

Price matters. Buffett won't buy even good companies at expensive valuations.

| Metric | Threshold | Type | Priority | Notes |
|--------|-----------|------|----------|-------|
| **Price-to-Earnings (P/E) Ratio** | < 25x (ideally 15-20x) | Current market | **SHOULD HAVE** | Reasonable valuation for quality business |
| **Price-to-Book (P/B) Ratio** | < 3.0x (ideally < 2.0x) | Current market | **SHOULD HAVE** | Not overvalued relative to book value |
| **Price-to-Sales (P/S) Ratio** | < 2.5x | Current market | **SHOULD HAVE** | Conservative valuation |
| **PEG Ratio** | < 1.5x | Current market | **SHOULD HAVE** | Growth-adjusted valuation (P/E ÷ Growth Rate %) |
| **Earnings Yield** | > (10-Year Treasury + 4-5%) | Current market | **SHOULD HAVE** | Better return than risk-free alternative |
| **Dividend Yield** | 2-4% (if applicable) | Current market | **SHOULD HAVE** | Income return to shareholders |
| **Payout Ratio** | 30-60% (if applicable) | Latest FY | **SHOULD HAVE** | Sustainable dividend policy |
| **Discount to Intrinsic Value** | 25-50% margin of safety | Intrinsic value calc | **NICE TO HAVE** | Safety buffer for error in judgment |

**Implementation Notes:**
- P/E = Stock Price ÷ EPS (trailing or forward 12 months)
- P/B = Market Cap ÷ Shareholders' Equity
- PEG = (P/E Ratio) ÷ (Expected EPS Growth Rate %)
- Earnings Yield = Net Income ÷ Market Cap
- 10-Year Treasury: Retrieved daily from Federal Reserve

**Valuation Context:**
- High-quality business (ROE > 20%, stable margins): Can justify P/E 20-25x
- Average quality (ROE 15-20%, moderate moat): Limit to P/E 15-20x
- Low quality (ROE 10-15%, weak moat): Want P/E < 12x

**Rejection Rules:**
- P/E > 30x without exceptional growth = reject
- Earnings Yield < 10-Year Treasury = reject
- PEG > 2.0 = expensive relative to growth

---

### Category 7: Management Quality & Capital Allocation

Buffett: "We look for managers with integrity, intelligence, and passion for the business."

| Metric | Threshold | Type | Priority | Notes |
|--------|-----------|------|----------|-------|
| **Management Insider Ownership** | > 5% | SEC filings | **SHOULD HAVE** | Skin in the game = aligned incentives |
| **Goodwill on Balance Sheet** | Minimal (< 20% of total assets) | Balance sheet | **SHOULD HAVE** | Shows if prior acquisitions at fair prices |
| **Goodwill Impairments** | Minimal or declining | Historical | **SHOULD HAVE** | Frequent impairments = prior overpayment |
| **One-Time Charges Frequency** | Rare (< 1 per 3 years) | 10-year history | **SHOULD HAVE** | "One-time" recurring = chronic problem |
| **Accounting Quality** | Conservative (no red flags) | 10-K analysis | **SHOULD HAVE** | Reliable earnings, no tricks |
| **Management Tenure** | 5+ years (CFO, CEO) | Company website | **NICE TO HAVE** | Stability and expertise |

**Implementation Notes:**
- Insider ownership: From SEC Schedule 13D, proxy statements
- Goodwill: Balance sheet item; should decline over time
- Impairments: Listed on income statement; watch for recurring
- One-Time Charges: Review 10-K for "adjusted earnings" claims
- Red Flags: Revenue recognition changes, frequent restatements, unusual related-party deals

**Red Flags (Automatic Rejection):**
- ❌ CEO with < 1% ownership
- ❌ Frequent accounting restatements
- ❌ Substantial goodwill with impairments
- ❌ Aggressive off-balance-sheet financing
- ❌ Management turnover (3+ changes in 5 years)

---

### Category 8: Business Quality & Competitive Advantage

Economic moats separate great businesses from commodities.

| Metric | Threshold | Type | Priority | Notes |
|--------|-----------|------|----------|-------|
| **Economic Moat** | Identifiable & sustainable | Qualitative | **SHOULD HAVE** | Protects pricing power, market share |
| **Market Position** | Leader or strong #2-3 | Industry ranking | **SHOULD HAVE** | Difficult to disrupt, pricing power |
| **Recurring Revenue %** | > 50% (where applicable) | Business model | **SHOULD HAVE** | Predictable, stable cash flows |
| **Brand Strength** | Recognized and valued | Qualitative | **NICE TO HAVE** | Customer loyalty, pricing power |
| **Customer Concentration** | No single customer > 20% | 10-K disclosure | **SHOULD HAVE** | Reduces business risk |
| **Industry Cyclicality** | Non-cyclical preferred | Industry analysis | **SHOULD HAVE** | Stable through economic cycles |
| **Business Model Simplicity** | Easy to understand | Qualitative | **MUST HAVE** | Within circle of competence |

**Types of Moats (Buffett Framework):**

| Moat Type | Examples | Indicator | How to Identify |
|-----------|----------|-----------|-----------------|
| **Brand Moat** | Coca-Cola, Apple, Nike | Margins stable despite competition | Brand recognition without heavy advertising |
| **Switching Costs** | Intuit, Salesforce, Microsoft | Customer retention > 95%, high NPS | High cost to switch suppliers |
| **Network Effects** | Mastercard, Visa, Facebook | Growing user base, improving unit economics | Value increases with more users |
| **Cost Advantage** | Costco, Walmart, LVMH | Market dominance despite lower margins | Sustainable lower cost structure |

**Red Flags (Weak/No Moat):**
- ❌ Easily replicated business model
- ❌ Commodity-like products (compete on price)
- ❌ Losing market share to competitors
- ❌ New disruptive technologies threatening business
- ❌ Customer concentration > 30%

---

## Priority Classification

### MUST HAVE (Deal Breakers)

ALL of these must be satisfied or company is rejected immediately:

| Metric | Threshold | Notes |
|--------|-----------|-------|
| **ROE** | > 15% (10-year average) | Core quality metric |
| **ROE Trend** | Stable or growing | Quality must not be declining |
| **Net Margin** | > 10% | High-quality earnings |
| **Operating Margin** | > 10% | Core business profitability |
| **Margin Stability** | CV < 15% | Predictable and consistent |
| **Debt-to-Equity** | < 0.50 | Financial strength |
| **D/E Trend** | Declining or stable | Not increasing leverage |
| **Current Ratio** | 1.5 - 2.5x | Can meet obligations |
| **Operating Cash Flow** | Positive (every year) | Real cash from operations |
| **OCF Growth** | > 0% (10-year CAGR) | Sustainable cash generation |
| **Free Cash Flow** | Positive (every year) | Cash available to shareholders |
| **FCF Growth** | > 0% (10-year CAGR) | Growing cash for shareholders |
| **Revenue Growth** | > 0% (10-year average) | Business not declining |
| **Business Model Simplicity** | Easy to understand | Within circle of competence |

**Stage 1 Screening Logic:**
