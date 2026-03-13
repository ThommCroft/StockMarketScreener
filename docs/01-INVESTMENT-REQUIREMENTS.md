# Investment Requirements Document

**Version:** 2.0 (Refined)  
**Date:** 2026-03-13  
**Author:** ThommCroft  
**Framework:** Warren Buffett & Charlie Munger Value Investing Principles

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Investment Philosophy](#investment-philosophy)
3. [Screening Criteria by Category](#screening-criteria-by-category)
4. [Priority Classification](#priority-classification)
5. [Metrics Mapping (Spreadsheet to Implementation)](#metrics-mapping)
6. [Data Requirements](#data-requirements)
7. [Implementation Notes](#implementation-notes)

---

## Executive Summary

This document defines **strict, quantifiable criteria** for identifying high-quality value stocks based on Warren Buffett and Charlie Munger's proven investment principles. The screener will evaluate companies across **all industries** using a multi-stage filtering approach:

- **Stage 1 (MUST HAVE)**: Financial Strength - Hard filters that eliminate weak businesses
- **Stage 2 (SHOULD HAVE)**: Quality Assessment - Deep analysis of business quality and earnings
- **Stage 3 (SHOULD HAVE)**: Valuation & Management - Identifies opportunities with safety margin

**Target Universe**: All US-listed equities with market cap > $300M

---

## Investment Philosophy

### Buffett/Munger Core Principles

#### 1. Circle of Competence & Business Understandability
- **Principle**: Only invest in businesses you can understand
- **Application**: Avoid complex financial structures, derivatives, and opaque business models
- **Metric**: Business Model Simplicity Score (qualitative assessment)

#### 2. Durable Competitive Advantage (Economic Moat)
- **Principle**: Sustainable competitive advantages create pricing power
- **Types of Moats**:
  - Brand strength (e.g., Coca-Cola, Apple)
  - Switching costs (e.g., enterprise software)
  - Network effects (e.g., payment processors)
  - Cost advantages (e.g., Costco, Walmart)
- **Application**: Favor companies with identifiable, sustainable moats
- **Metrics**: Market position, brand strength, recurring revenue %, margin stability

#### 3. Financial Fortress (Balance Sheet Strength)
- **Principle**: Strong balance sheets survive downturns and create optionality
- **Requirements**:
  - Low debt levels (D/E < 0.50)
  - Positive free cash flow
  - Adequate liquidity
- **Metrics**: Debt-to-Equity, Current Ratio, FCF > 0

#### 4. Consistent, Growing Earnings
- **Principle**: Real earnings (cash flow) matter more than accounting earnings
- **Requirements**:
  - Consistent profitability (low volatility)
  - Growing earnings over time
  - High-quality earnings (FCF > Net Income)
- **Metrics**: ROE stability, margin consistency, FCF conversion ratio

#### 5. Management Quality
- **Principle**: Competent management aligned with shareholders
- **Requirements**:
  - Significant insider ownership (skin in the game)
  - Honest, transparent communication
  - Conservative accounting practices
  - Smart capital allocation
- **Metrics**: Management ownership %, goodwill levels, one-time charge frequency

#### 6. Margin of Safety
- **Principle**: Buy at significant discount to intrinsic value
- **Requirements**:
  - Reasonable valuation multiples
  - Earnings yield > risk-free rate
  - 25-50% margin of safety preferred
- **Metrics**: P/E ratio, P/B ratio, earnings yield, discount to intrinsic value

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

**Types of Moats (Buffett Framework):**

1. **Brand Moat** (Examples: Coca-Cola, Apple, Nike)
   - Strong brand = pricing power
   - Indicator: Margins stable despite competition

2. **Switching Costs Moat** (Examples: Intuit, Salesforce, Microsoft)
   - High cost to switch suppliers
   - Indicator: Customer retention > 95%, high NPS

3. **Network Effects Moat** (Examples: Mastercard, Visa, Marketplace platforms)
   - Value increases with more users
   - Indicator: Growing user base, improving unit economics

4. **Cost Advantage Moat** (Examples: Costco, Walmart, LVMH)
   - Lower cost structure = sustainable pricing power
   - Indicator: Lower margins but market dominance, stable ROIC

**Implementation Notes:**
- Research company history, competitive position, industry structure
- Analyze 5-year stock price relative to peers (moat = outperformance)
- Review investor presentations, analyst reports
- Compare customer retention rates, NPS vs. peers
- Assess threat of disruption (e.g., retail vs. e-commerce)

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
