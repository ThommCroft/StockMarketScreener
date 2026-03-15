# Stock Market Screener - Valuation Methodology Requirements

**Version:** 1.0  
**Date:** 2026-03-15  
**Author:** ThommCroft  
**Framework:** Warren Buffett & Charlie Munger Value Investing Principles + Multi-Method Valuation  
**Stage:** Stage 6 - Intrinsic Value Valuation

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Introduction & Stage Context](#introduction--stage-context)
3. [Input Requirements](#input-requirements)
4. [Discounted Cash Flow (DCF) Method](#section-3-discounted-cash-flow-dcf-method)
5. [Dividend Discount Model (DDM)](#section-4-dividend-discount-model-ddm)
6. [Multiples Valuation Model](#section-5-multiples-valuation-model)
7. [Warren Buffett Valuation Techniques](#section-6-warren-buffett-valuation-techniques)
8. [Charlie Munger Valuation Techniques](#section-7-charlie-munger-valuation-techniques)
9. [Consensus Valuation & Final Assessment](#consensus-valuation--final-assessment)
10. [Output Definitions](#output-definitions)
11. [Real-World Examples](#real-world-examples)
12. [Implementation Notes](#implementation-notes)
13. [Troubleshooting Guide](#troubleshooting-guide)
14. [Related Documents](#related-documents)
15. [Document History](#document-history)

---

## Executive Summary

### Purpose of Stage 6: Intrinsic Value Valuation

Stage 6 (Intrinsic Value Valuation) is the final analytical stage in the Stock Market Screener system. It takes companies that have passed the rigorous financial analysis and calculates their intrinsic value using multiple independent methods.

### Stage 6 in the Pipeline

**Input:** Companies that passed Stage 5 (Financial Analysis Results Processing) with Composite Score ≥ 75

**Process:** Calculate intrinsic value using five independent valuation methods:
- Discounted Cash Flow (DCF)
- Dividend Discount Model (DDM)
- Multiples Valuation Model
- Warren Buffett Valuation Techniques
- Charlie Munger Valuation Techniques

**Output:** Fair value estimates, margin of safety, confidence levels, valuation grade, investment recommendation

**Decision-Making:** Determines which companies are attractive at current prices

**Next Stage:** Stage 7 - Results Reporting (store data, generate reports, send notifications)

### The Five Valuation Methods

This stage employs **five independent valuation approaches**, each with distinct philosophies:

1. **Discounted Cash Flow (DCF)** - Present value of all future free cash flows
2. **Dividend Discount Model (DDM)** - Present value of future dividends (or buybacks)
3. **Multiples Valuation** - Relative valuation based on peer comparison

Each method produces its own intrinsic value estimate. When they converge, confidence increases. When they diverge, deeper investigation is warranted.

### Buffett & Munger Investment Techniques

Beyond the three methods, we apply both Warren Buffett and Charlie Munger's proven valuation frameworks:

**Buffett Approach:**
- Owner's Earnings Valuation (normalized cash earnings × quality multiple)
- Perpetual Bond Yield Comparison (earnings yield vs risk-free rate)
- Emphasis on margin of safety (25-50%)

**Munger Approach:**
- Quality-Based Business Assessment (1-10 scoring on 5 dimensions)
- Quality Multiple Assignment based on business strength
- Five Stock Test (would you own this forever?)
- Built-in conservatism through quality emphasis

### Key Outcomes

For each company, Stage 6 produces:

✅ **Three Independent Intrinsic Values** (DCF, DDM, Multiples)  
✅ **Two Master Valuation Assessments** (Buffett, Munger)  
✅ **Consensus Fair Value** (weighted combination)  
✅ **Margin of Safety** at multiple thresholds (20%, 30%, 40%, 50%)  
✅ **Valuation Grade** (A+ through F)  
✅ **Investment Recommendation** (BUY, HOLD, PASS)  
✅ **Grouping Category** (Slam Dunk / Strong Buy / Fair Value / Overvalued / Not Qualified)  

### Why This Matters

---

## Philosophy & Investment Framework

### Core Philosophy: Price ≠ Value

**Price ≠ Value**

A company with exceptional fundamentals can be a terrible investment if priced too high. Conversely, a mediocre business at a steep discount might be worth buying. Stage 6 separates price from value and identifies true opportunities.

---

## Introduction & Stage Context

### How Stage 6 Fits in the Overall Pipeline

Stage 6 (Stock Price - Intrinsic Value Valuation) is the second major phase of the stock screener, following Stage 5 (Financial Analysis - Results Processing).

**Pipeline Overview:**

Stages 0-5: Financial Analysis 
↓ 
Stage 5: Results Processing (Composite Score ≥ 75 threshold) 
↓ 
Stage 6: Intrinsic Value Valuation ← YOU ARE HERE 
↓ 
Stage 7: Results Reporting & Output

**Key Relationship:**
- **Input Source:** Document 01-INVESTMENT-REQUIREMENTS.md (Stages 0-5)
- **Output Destination:** Document 03-RESULTS-REPORTING-REQUIREMENTS.md (Stage 7)
- **Workflow Overview:** Document 0-BASIC-STAGE-WORKFLOW.md

### What Happens Before Stage 6

Companies reaching Stage 6 have already:
- ✅ Passed market cap filter (> $300M)
- ✅ Had all 40+ metrics calculated
- ✅ Passed financial strength assessment
- ✅ Received Quality Score (Pillars 1-4)
- ✅ Received Composite Score (Pillars 1-6, weighted)
- ✅ Met threshold: Composite Score ≥ 75

**Only these qualified companies** proceed to Stage 6 valuation analysis.

### What Happens After Stage 6

Stage 6 outputs are used in Stage 7 to:
- Store all valuation results in MySQL database
- Generate email reports with findings
- Create GitHub summary pages
- Produce downloadable CSV files
- Identify investment opportunities

---

## Input Requirements

### Required Input Data from Stage 5

For each company reaching Stage 6, the following data must be available:

**Financial Metrics (from Stage 1 & onward):**
- ✅ 10 years of historical financial statements
- ✅ Revenue, Operating Income, Net Income
- ✅ Free Cash Flow (OCF - CapEx)
- ✅ Debt, Equity, Interest Expense
- ✅ Shares Outstanding, Dividends Per Share (if applicable)
- ✅ Return on Equity (ROE)
- ✅ Return on Invested Capital (ROIC)

**Scoring Data (from Stage 3-5):**
- ✅ Quality Score (0-100)
- ✅ Valuation Score (0-100)
- ✅ Management Score (0-100)
- ✅ Composite Score (0-100)
- ✅ Composite Score ≥ 75 (qualification criterion)

**Market Data:**
- ✅ Current stock price
- ✅ Shares outstanding
- ✅ Market capitalization
- ✅ Dividend yield (if applicable)

**Risk-Free Rate:**
- ✅ 10-year Treasury yield (for discount rate calculation)

### Data Quality Assurance

All input data must meet these requirements:
- ✓ Complete (no missing critical values)
- ✓ Validated (balance sheet balanced, ratios reasonable)
- ✓ Recent (stock price < 1 day old, financials < 1 year old)
- ✓ Consistent (year-over-year changes within acceptable ranges)

---

### Processing Steps

For each company:

1. **Calculate three intrinsic values** using different methods
2. **Apply Buffett valuation techniques** (two approaches)
3. **Apply Munger valuation techniques** (two approaches)
4. **Calculate consensus fair value** (weighted average)
5. **Analyze margin of safety** (current vs fair value)
6. **Assign valuation grade** (A+ to F)
7. **Generate recommendations** (Buffett, Munger, Joint)
8. **Assign grouping category** (5 opportunity groups)
9. **Create investment thesis** (one-paragraph summary)
10. **Store in database** with full audit trail

---

### Output Data

Stage 5.4 produces:

**Per-Company Results:**
- DCF fair value (+ sensitivity range)
- DDM fair value
- Multiples fair value (P/E, P/B, P/S averages)
- Buffett Owner's Earnings valuation
- Buffett Perpetual Bond Yield assessment
- Munger Quality Checklist scores (1-10 for 5 dimensions)
- Munger Quality-Based fair value
- Munger Five Stock Test result
- Consensus fair value
- Margin of safety (vs each method, for each safety level)
- Valuation grade (A+ to F)
- Buffett recommendation (BUY/HOLD/PASS)
- Munger recommendation (BUY/HOLD/PASS)
- Joint recommendation
- Grouping category (1-5)
- Investment thesis

**Aggregate Results:**
- Summary statistics (average fair value, average margin, etc.)
- Distribution across grouping categories
- Comparison with prior screening run
- Performance metrics for previously recommended companies

### Core Philosophy: Price ≠ Value

The fundamental principle underlying this entire stage:

> **Price is what you pay. Value is what you get.**
> — Warren Buffett

A company trading at $100/share might have:
- Intrinsic value of $150 (UNDERVALUED, buying opportunity)
- Intrinsic value of $100 (FAIRLY VALUED, hold if owned)
- Intrinsic value of $50 (OVERVALUED, avoid or sell)

Stage 5.4 determines which scenario applies.

### Investment Decision Framework

The valuation results drive investment decisions:
IF Intrinsic Value >> Current Price (>30% margin) → BUY (compelling opportunity)

ELSE IF Intrinsic Value > Current Price (10-30% margin) → ACCUMULATE (fair opportunity, wait for better entry)

ELSE IF Intrinsic Value ≈ Current Price (<10% margin) → HOLD (if owned, likely no reason to buy more)

ELSE (Current Price > Intrinsic Value) → PASS or SELL (overvalued, wait for decline)

---

## Section 3: Discounted Cash Flow (DCF) Method

**Method 1 of 5 - Weight in Consensus: 40%**

### 3.1 DCF Philosophy & Foundational Concepts

#### What is DCF?

Discounted Cash Flow (DCF) valuation is based on a simple principle:

> **The intrinsic value of any investment is the present value of all cash it will generate in the future.**

Rather than focusing on current earnings or stock price, DCF asks: "What is all the cash this business will produce over its lifetime worth in today's dollars?"

#### Why Buffett Uses DCF

Warren Buffett relies on DCF as his primary valuation method because:

1. **It focuses on cash, not accounting profits** - Real cash available to shareholders matters more than reported earnings
2. **It accommodates different growth rates** - Young companies with high growth are valued differently from mature companies
3. **It uses conservative assumptions** - Building in margin of safety through discount rates
4. **It's mathematically rigorous** - Removes emotion from valuation
5. **It works for any business** - Whether dividend-paying or not, cyclical or stable

Buffett has said: *"The basic principle is that an asset's value is the present value of the cash flows that the asset will generate in the future."*

#### Key Concepts in DCF

**Free Cash Flow (FCF):** The cash available to all investors (debt and equity holders) after the company has made investments to maintain and grow the asset base.

FCF = Operating Cash Flow - Capital Expenditures

Why FCF matters: It's harder to manipulate than earnings. A company can boost reported earnings through accounting tricks, but FCF reflects real cash.

**Discount Rate (WACC):** The rate at which future cash flows are discounted to present value. It represents the weighted average cost of debt and equity capital.

WACC = (E/V × Re) + (D/V × Rd × (1-Tc))

Lower WACC → Higher intrinsic value (cash is worth more today)  
Higher WACC → Lower intrinsic value (cash is worth less today)

**Terminal Value:** The value of all cash flows beyond the projection period (Year 11 onwards, in perpetuity).

Terminal Value = FCFn × (1 + g) / (WACC - g)

Where g is the perpetual growth rate (typically 2.5% = GDP growth estimate)

**Perpetuity Growth Rate:** We assume the company will grow at a steady rate forever after the projection period ends. 2.5% is conservative (roughly GDP growth) and is Buffett's typical assumption.

---

### 3.2 DCF Complete Formula

#### Main DCF Formula
Intrinsic Value = Σ(FCFt / (1+WACC)^t) + Terminal Value / (1+WACC)^n

Where: FCFt = Free Cash Flow in year t WACC = Weighted Average Cost of Capital (discount rate) t = Year (1 through 10) n = Final projection year (Year 10) Terminal Value = FCF10 × (1+g) / (WACC - g) g = Perpetual growth rate (2.5%)

#### Breaking Down the Formula

**Present Value of Projected Cash Flows (Years 1-10):**

Each year's free cash flow is discounted back to present value:

PV(Year 1) = FCF1 / (1 + WACC)^1 PV(Year 2) = FCF2 / (1 + WACC)^2 ... PV(Year 10) = FCF10 / (1 + WACC)^10

**Total Intrinsic Value:**

Intrinsic Value = Sum(PV of Years 1-10) + PV(Terminal Value) Per-Share Value = Intrinsic Value / Diluted Shares Outstanding

---

### 3.3 DCF Step-by-Step Calculation

Follow these 8 steps to calculate DCF valuation for any company:

#### **Step 1: Gather Historical Free Cash Flow (Last 10 Years)**

Source: SEC 10-K cash flow statement

Free Cash Flow = Operating Cash Flow - Capital Expenditures

From SEC filings, find:
- **Operating Cash Flow** (cash from operations, Item 4.1.1)
- **Capital Expenditures** (property, plant, equipment, Item 4.1.2)

Subtract: FCF = OCF - CapEx

Create a 10-year historical table:

Year OCF (B)CapEx(B) FCF ($B) 2016 $18.5 $8.2 $10.3 2017 $20.1 $8.8 $11.3 2018 $22.3 $9.1 $13.2 2019 $24.5 $9.6 $14.9 2020 $26.8 $10.2 $16.6 2021 $28.9 $10.5 $18.4 2022 $30.1 $10.9 $19.2 2023 $31.5 $11.2 $20.3 2024 $33.2 $11.8 $21.4 2025 $35.1 $12.5 $22.6

**Quality Check:** Do OCF and FCF both trend upward? If FCF is declining despite growing OCF, company is overspending on capital.

#### **Step 2: Calculate Historical FCF Growth Rate (CAGR)**

Historical FCF growth will inform your forward projections.

FCF CAGR = (FCF2025 / FCF2016)^(1/9) - 1 = ($22.6B / $10.3B)^(1/9) - 1 = (2.195)^(0.1111) - 1 = 1.0897 - 1 = 8.97% ≈ 9.0%

**Interpretation:** This company has grown free cash flow at ~9% annually over the past 10 years.

**What this tells us:** 
- Fast growth (>10%) suggests strong business
- Moderate growth (5-10%) suggests mature but improving
- Slow growth (<5%) suggests mature/stable business
- Negative growth suggests problems

#### **Step 3: Project Future Free Cash Flow (Years 1-10)**

You must make assumptions about future growth. This is where DCF becomes an art, not just math.

**Common approach: Declining growth rates**

Start with historical growth, then gradually decay toward 2.5% terminal rate:

Years 1-3: Use historical growth rate or slightly conservative Years 4-7: Gradually decline growth rate Years 8-10: Approach terminal growth rate (2.5%)

Example for 9% historical growth: Year 1: 9.0% growth Year 2: 8.5% growth Year 3: 8.0% growth Year 4: 6.5% growth Year 5: 5.5% growth Year 6: 4.5% growth Year 7: 3.5% growth Year 8: 3.0% growth Year 9: 2.7% growth Year 10: 2.5% growth (terminal)

**Alternative approach: Conservative flat rate**

For mature companies or when uncertain:

Use conservative fixed growth (3-5%) for all 10 years

**Project forward:**

Year 1 FCF = Prior Year FCF × (1 + Growth Rate) = $22.6B × 1.090 = $24.63B

Year 2 FCF = $24.63B × 1.085 = $26.72B

...continuing through Year 10...

Year 10 FCF = $32.18B (growing at 2.5%)

**Full 10-year projection:**

Year Growth % FCF ($B) Notes 1 9.0% $24.63 Historical momentum 2 8.5% $26.72 Slight deceleration 3 8.0% $28.86 Expected as company matures 4 6.5% $30.73 Transition to slower growth 5 5.5% $32.41 Moderate growth phase 6 4.5% $33.86 Approaching mature rate 7 3.5% $35.02 Near terminal growth 8 3.0% $36.07 Converging to terminal 9 2.7% $37.04 Close to terminal 10 2.5% $37.96 Terminal growth rate

**Critical assumption:** These projections must be defensible. 
- If projecting 10% growth, why? Markets growing 5%, so company gaining share?
- If industry is mature (restaurants, retail), 2-3% is more realistic than 8%
- Beware: Most companies can't sustain above-market growth forever

#### **Step 4: Calculate WACC (Discount Rate)**

WACC is the rate at which future cash flows are discounted. Higher WACC = lower valuation.

WACC = (E/V × Re) + (D/V × Rd × (1-Tc))

Where: E = Market value of equity (Stock Price × Shares Outstanding) D = Market value of debt (from balance sheet) V = E + D (total firm value) Re = Cost of Equity (using CAPM) Rd = Cost of Debt Tc = Corporate tax rate

**Calculating Cost of Equity (Re) using CAPM:**
Re = Rf + β × (Rm - Rf)

Where: Rf = Risk-free rate (10-year Treasury yield) β = Beta (stock volatility vs market) Rm = Market return (typically assumed 10%) (Rm - Rf) = Equity risk premium

**Example calculation:**
Current inputs: 10-Year Treasury (Rf): 4.2% Company Beta (β): 1.1 (slightly more volatile than market) Market Return (Rm): 10% (standard assumption)

Cost of Equity = 4.2% + 1.1 × (10% - 4.2%) = 4.2% + 1.1 × 5.8% = 4.2% + 6.38% = 10.58% ≈ 10.6%

**Calculating Cost of Debt (Rd):**
Rd = Interest Expense / Total Debt

From financial statements: Annual Interest Expense: $2.5B Total Debt Outstanding: $45B

Cost of Debt = $2.5B / $45B = 5.56% ≈ 5.6%

**Calculate Weights (E/V and D/V):**

Market Cap (E) = $120/share × 2.4B shares = $288B Total Debt (D) = $45B Total Value (V) = $288B + $45B = $333B

E/V = $288B / $333B = 86.5% D/V = $45B / $333B = 13.5%

**Tax rate:**

Tc = Income Tax / Earnings Before Tax = $8.5B / $28.5B = 29.8% ≈ 30%

**Calculate WACC:**

WACC = (0.865 × 0.106) + (0.135 × 0.056 × (1-0.30)) = (0.0917) + (0.135 × 0.056 × 0.70) = 0.0917 + 0.00529 = 0.0970 = 9.7%

**WACC of 9.7%** means future cash flows are discounted at this rate.

**Interpretation:**
- WACC 6-8% = Low risk company (utilities, consumer staples, Microsoft)
- WACC 8-11% = Moderate risk company (most large-cap tech/industrial)
- WACC 11-15% = Higher risk company (small-cap, cyclical)
- WACC >15% = Distressed/speculative company

#### **Step 5: Discount All Projected Cash Flows to Present Value**

Take each year's projected FCF and discount it back to today's value.

Formula: ```PV = FCF / (1 + WACC)^Year```

Year 1: $24.63B / (1.097)^1 = $24.63B / 1.097 = $22.46B Year 2: $26.72B / (1.097)^2 = $26.72B / 1.204 = $22.19B Year 3: $28.86B / (1.097)^3 = $28.86B / 1.320 = $21.86B Year 4: $30.73B / (1.097)^4 = $30.73B / 1.448 = $21.24B Year 5: $32.41B / (1.097)^5 = $32.41B / 1.588 = $20.41B Year 6: $33.86B / (1.097)^6 = $33.86B / 1.742 = $19.44B Year 7: $35.02B / (1.097)^7 = $35.02B / 1.910 = $18.33B Year 8: $36.07B / (1.097)^8 = $36.07B / 2.096 = $17.20B Year 9: $37.04B / (1.097)^9 = $37.04B / 2.299 = $16.10B Year 10: $37.96B / (1.097)^10 = $37.96B / 2.522 = $15.05B

**Sum of PV (Years 1-10) = $194.28B**

**What this means:** All the cash flows from Years 1-10 are worth $194.28B in today's dollars.

#### **Step 6: Calculate Terminal Value (Years 11 Onwards)**

The company is assumed to continue in perpetuity after Year 10 at the terminal growth rate (2.5%).

Terminal Value = FCF10 × (1 + g) / (WACC - g) = $37.96B × (1 + 0.025) / (0.097 - 0.025) = $37.96B × 1.025 / 0.072 = $38.91B / 0.072 = $540.15B


**This represents:** The value of all cash flows from Year 11 to infinity.

**Discount Terminal Value to Present:**

PV(Terminal Value) = $540.15B / (1.097)^10 = $540.15B / 2.522 = $214.11B

**Why discount?** Because this value occurs 10 years from now, it's worth less in today's dollars.

#### **Step 7: Sum All Discounted Cash Flows**

DCF Intrinsic Value = Sum(PV Years 1-10) + PV(Terminal Value) = $194.28B + $214.11B = $408.39B

**This is the enterprise value** - the total value of the company to all investors (equity + debt holders).

#### **Step 8: Calculate Per-Share Intrinsic Value**

Enterprise Value: $408.39B Less: Net Debt (Total Debt - Cash) Total Debt: $45B Cash: $12B Net Debt: $33B

Equity Value = $408.39B - $33B = $375.39B

Diluted Shares Outstanding: 2.4B

Per-Share Intrinsic Value = $375.39B / 2.4B = $156.41/share

**This is your DCF fair value: $156.41 per share**

---

### 3.4 DCF Example Calculation - Microsoft (Real Numbers)

Let's walk through a complete DCF valuation for Microsoft Corporation:

#### Company Data (As of Q4 2025)

Current Stock Price: $420.50 Shares Outstanding (diluted): 2.4B Market Cap: $3.14 trillion

#### Step 1: Historical FCF (2016-2025)

From SEC filings:

Year Operating CF CapEx FCF (B)(B) ($B) 2016 $40.5 $9.2 $31.3 2017 $43.2 $9.8 $33.4 2018 $50.1 $10.5 $39.6 2019 $54.3 $11.2 $43.1 2020 $60.8 $12.1 $48.7 2021 $68.5 $13.5 $55.0 2022 $72.1 $14.2 $57.9 2023 $76.4 $15.1 $61.3 2024 $81.2 $16.0 $65.2 2025 $87.5 $17.2 $70.3

#### Step 2: FCF Growth Rate

CAGR = ($70.3B / $31.3B)^(1/9) - 1 = (2.246)^(0.1111) - 1 = 9.2% annual growth

**Interpretation:** Microsoft has grown FCF at 9.2% annually - strong, sustainable growth for a mature company.

#### Step 3: Project Future FCF (Declining Growth Model)

Year Growth % Calculation FCF ($B) 1 9.0% $70.3 × 1.090 $76.63 2 8.5% $76.63 × 1.085 $83.04 3 8.0% $83.04 × 1.080 $89.68 4 6.5% $89.68 × 1.065 $95.51 5 5.5% $95.51 × 1.055 $100.77 6 4.5% $100.77 × 1.045 $105.31 7 3.5% $105.31 × 1.035 $109.00 8 3.0% $109.00 × 1.030 $112.27 9 2.7% $112.27 × 1.027 $115.31 10 2.5% $115.31 × 1.025 $118.19

#### Step 4: Calculate WACC

**Cost of Equity (CAPM):**

Risk-free rate: 4.2% (10-year Treasury) Beta: 1.05 (Microsoft is slightly more volatile than market) Market return: 10%

Re = 4.2% + 1.05 × (10% - 4.2%) = 4.2% + 1.05 × 5.8% = 4.2% + 6.09% = 10.29%

**Cost of Debt:**

Interest Expense: $2.1B Total Debt: $45B

Rd = $2.1B / $45B = 4.67%

**Weights:**

Market Cap: $420.50 × 2.4B = $1,009.2B Total Debt: $45B Total Value: $1,054.2B

E/V = $1,009.2B / $1,054.2B = 95.7% D/V = $45B / $1,054.2B = 4.3%

**Tax Rate:**

Income Tax / EBT = $8.2B / $28.5B = 28.8%

**WACC:**

WACC = (0.957 × 0.1029) + (0.043 × 0.0467 × (1-0.288)) = 0.0985 + 0.00146 = 0.0999 = 9.99% ≈ 10.0%

#### Step 5: Discount Projected FCF to Present Value

Year FCF (B)DiscountFactorPV(B) 1 $76.63 1 / 1.10^1 $69.67 2 $83.04 1 / 1.10^2 $68.60 3 $89.68 1 / 1.10^3 $67.37 4 $95.51 1 / 1.10^4 $65.27 5 $100.77 1 / 1.10^5 $62.54 6 $105.31 1 / 1.10^6 $59.46 7 $109.00 1 / 1.10^7 $56.31 8 $112.27 1 / 1.10^8 $52.34 9 $115.31 1 / 1.10^9 $48.71 10 $118.19 1 / 1.10^10 $45.59

Sum of PV (Years 1-10) = $595.86B

#### Step 6: Calculate Terminal Value

Terminal Value = $118.19B × (1.025) / (0.10 - 0.025) = $121.14B / 0.075 = $1,614.53B

PV(Terminal Value) = $1,614.53B / (1.10)^10 = $1,614.53B / 2.5937 = $622.70B

#### Step 7: Sum Discounted Cash Flows

Enterprise Value = $595.86B + $622.70B = $1,218.56B

#### Step 8: Calculate Per-Share Value

Enterprise Value: $1,218.56B Less: Net Debt Total Debt: $45B Cash: $82B Net Cash: ($37B) [negative = net cash position]

Equity Value = $1,218.56B - (-$37B) = $1,255.56B

Per-Share Intrinsic Value = $1,255.56B / 2.4B shares = $523.15/share

#### DCF Fair Value Result

DCF Fair Value: $523.15 per share Current Price: $420.50 per share Upside: ($523.15 - $420.50) / $420.50 = 24.4% Status: UNDERVALUED (according to DCF)

---

### 3.5 DCF Sensitivity Analysis

DCF valuations are highly sensitive to two key assumptions: discount rate (WACC) and terminal growth rate.

#### Discount Rate Sensitivity

What if WACC is different than our 10% estimate?

Terminal Value = $121.14B / (WACC - 0.025) PV(TV at Year 10) = Terminal Value / (1 + WACC)^10 Sum of PV (Yrs 1-10) = Recalculate with different WACC

Testing different WACC values:

WACC Terminal Value PV(TV) Sum PV(1-10) Enterprise Value Per Share 8.0% $1,513.50B $703.42B $635.28B $1,338.70B $558.63 8.5% $1,338.00B $618.86B $615.71B $1,234.57B $514.40 9.0% $1,212.59B $555.33B $607.82B $1,163.15B $484.65 9.5% $1,113.33B $505.95B $601.44B $1,107.39B $461.41 10.0% $1,032.29B $465.92B $595.86B $1,061.78B $442.41 ← Our estimate 10.5% $963.73B $432.65B $590.79B $1,023.44B $426.44 11.0% $904.33B $404.41B $585.99B $990.40B $412.67 11.5% $852.35B $380.29B $581.40B $961.69B $400.70 12.0% $806.19B $359.55B $576.98B $936.53B $390.22

**Sensitivity interpretation:**

- At 8% WACC (lower discount rate): Fair value = $558.63
- At 12% WACC (higher discount rate): Fair value = $390.22
- Range: $168.41 difference (43% spread)

This shows that getting the discount rate right is critical. If Microsoft's true cost of capital is 8%, it's more undervalued than our 10% estimate suggests. If it's 12%, it's less undervalued.

#### Terminal Growth Rate Sensitivity

What if perpetual growth is different than 2.5%?

Testing different terminal growth rates (holding WACC at 10%):

Terminal Terminal Value PV(TV) Enterprise Value Per Share Growth % Calculation at Year 10 2.0% $97.45B / 0.080 $448.30B $1,044.16B $435.07 2.3% $103.25B / 0.077 $473.76B $1,069.62B $445.68 2.5% $107.94B / 0.075 $496.06B $1,091.92B $454.97 ← Our estimate 2.7% $112.64B / 0.073 $518.37B $1,114.23B $464.30 3.0% $119.44B / 0.070 $549.81B $1,145.67B $477.36 3.5% $130.04B / 0.065 $600.43B $1,196.29B $498.45

**Sensitivity interpretation:**

- At 2.0% terminal growth: Fair value = $435.07
- At 3.5% terminal growth: Fair value = $498.45
- Range: $63.38 difference (14.5% spread)

Terminal growth is less sensitive than WACC, but still matters. Assuming 3.5% perpetual growth when realistic is 2.5% would overvalue the company by 9.6%.

#### 2D Sensitivity Matrix (WACC × Terminal Growth)

Show impact of BOTH variables changing simultaneously:

**Terminal Growth Rate**
WACC 2.0% 2.5% 3.0% 3.5% 8.0% $601.45 $625.50 $651.25 $679.15 8.5% $557.42 $579.48 $603.35 $629.48 9.0% $518.75 $539.65 $562.15 $586.48 9.5% $484.30 $504.05 $525.22 $548.48 10.0% $453.41 $471.82 $491.63 $514.40 ← Our base case 10.5% $426.40 $442.95 $460.97 $482.21 11.0% $401.72 $416.73 $433.43 $452.90 11.5% $379.42 $392.99 $408.24 $426.21 12.0% $359.22 $371.60 $385.57 $402.27

**Key findings:**

- **Best case** (8% WACC, 3.5% growth): $679.15/share (+53% vs current $420.50)
- **Base case** (10% WACC, 2.5% growth): $471.82/share (+12.2%)
- **Worst case** (12% WACC, 2.0% growth): $359.22/share (-14.6% vs current price)

The sensitivity matrix shows that even under moderately adverse assumptions, Microsoft remains fairly valued or undervalued.

---

### 3.6 DCF Limitations & When to Use

#### Strengths of DCF

✅ **Mathematically rigorous** - Forces clear thinking about business fundamentals  
✅ **Works for all companies** - Dividend-paying or not, mature or growing  
✅ **Conservative** - Built-in margin of safety through discount rate  
✅ **Focus on cash** - Harder to manipulate than accounting earnings  
✅ **Time-value aware** - Accounts for money being worth more today than tomorrow  

#### Limitations of DCF

❌ **Highly sensitive to assumptions** - Small changes in growth or WACC create large valuation swings  
❌ **Difficult to project 10 years** - Impossible to predict accurately that far ahead  
❌ **Garbage in, garbage out** - Incorrect assumptions lead to meaningless results  
❌ **Requires financial expertise** - Calculating WACC correctly isn't trivial  
❌ **Terminal value dominates** - Often 50-70% of total value comes from years 11+, which are most uncertain  
❌ **Industry-dependent** - DCF works better for stable businesses than disruptive industries  

#### When DCF Works Best

✅ Mature, stable companies (Microsoft, Coca-Cola, utilities)  
✅ Companies with predictable cash flows  
✅ Companies with visible competitive advantages (moats)  
✅ Companies not facing disruptive threats  
✅ When you have 10+ years of historical data  

#### When DCF Works Poorly

❌ Startups and early-stage companies (highly uncertain cash flows)  
❌ Companies in disrupted/changing industries (assumptions invalid)  
❌ Companies with volatile cash flows  
❌ Companies with <5 years of history  
❌ When terminal growth rate is unclear  

#### DCF Sanity Checks

Before accepting a DCF valuation, ask:

1. **Does the assumed growth rate make sense?**
   - If projecting 15% growth, is there a reason to believe this company will gain market share?
   - Is the industry growing that fast?
   - Can management execute on this growth?

2. **Is the WACC reasonable?**
   - Compare to industry peers
   - Does it reflect the company's actual risk level?
   - Has the cost of debt/equity changed recently?

3. **Is terminal growth realistic?**
   - 2.5% = GDP growth (reasonable long-term)
   - >3% = Predicting company will grow faster than economy forever (rare)
   - <2% = Predicting recession or decline (needs justification)

4. **Does the result pass the "smell test"?**
   - If DCF says $50/share but peers trade at 15x earnings ($75/share), investigate why
   - If DCF says $200/share but company is barely profitable, be skeptical
   - Does the valuation align with business quality?

5. **How sensitive is the result?**
   - If DCF value ranges from $300-$600 with small assumption changes, there's too much uncertainty
   - If DCF value stays in $450-$500 range despite assumption changes, more confidence

---

### Summary: DCF Method

DCF Intrinsic Value = Present Value of All Future Free Cash Flows

Process:

Gather 10 years historical FCF
Calculate historical FCF growth rate (CAGR)
Project future FCF (10 years) with declining growth rates
Calculate WACC (discount rate)
Discount all projected FCF to present value
Calculate terminal value (perpetual growth after Year 10)
Sum all discounted cash flows
Divide by diluted shares for per-share value
Result: Fair value per share

Buffett's preference: Conservative assumptions, large margin of safety

Key insight: A company's value is what it will earn in the future, not what the market currently pays for it.

---

## Section 4: Dividend Discount Model (DDM)

**Method 2 of 5 - Weight in Consensus: 15%**

### 4.1 DDM Philosophy & Foundational Concepts

#### What is DDM?

The Dividend Discount Model values a stock based on the present value of all future dividends it will pay to shareholders.

> **The value of a stock is the present value of all cash dividends it will distribute to shareholders in perpetuity.**

#### Why Use DDM?

While DCF focuses on all free cash flow, DDM specifically values what's **returned to equity holders** through dividends or buybacks. This approach is particularly valuable for:

1. **Dividend-paying companies** - Direct cash return to shareholders
2. **Companies with buyback programs** - Repurchasing shares is equivalent to dividends
3. **Mature, stable companies** - Predictable dividend payments
4. **Investors seeking income** - Dividend streams are tangible and reliable

#### DDM vs DCF: When to Use Each

| Aspect | DCF | DDM |
|--------|-----|-----|
| **Best For** | All companies | Dividend/buyback stable companies |
| **Cash Focus** | All free cash flow | Only returned to shareholders |
| **Growth Assumption** | Sustainable long-term | Sustainable dividend growth |
| **Data Required** | 10Y cash flow statements | 10Y dividend history |
| **Complexity** | Higher (WACC, projections) | Lower (simpler assumptions) |
| **Result Usage** | Baseline valuation | Validation/cross-check |

**Recommended approach:** Calculate DCF as primary valuation, then use DDM as validation for dividend/buyback companies.

#### Key Concepts in DDM

**Dividend Per Share (DPS):** Annual cash paid to each share.

DPS = Total Dividends Paid / Shares Outstanding

**Payout Ratio:** Percentage of earnings returned to shareholders.

Payout Ratio = Total Dividends / Net Income

**Sustainable payout ratio:** 30-60% (leaves money for growth and downturns)

**Dividend Growth Rate:** Year-over-year growth in dividends (historically and projected).

Dividend CAGR = (DPS_current / DPS_10years_ago)^(1/10) - 1

**Buyback-Adjusted Dividend:** For non-dividend companies, equivalent dividend calculated from share buybacks.

Buyback-Adjusted DPS = Share Repurchases / Weighted Avg Shares Total Return = Cash Dividends + Buyback-Adjusted Dividends

**Required Return (Discount Rate):** Similar to WACC in DCF, but sometimes simplified for DDM.

r = Risk-Free Rate + Risk Premium

For dividend stocks: Risk Premium = 3-5% (lower risk than general equity)

---

### 4.2 DDM Formulas

#### Formula 1: Gordon Growth Model (Simple DDM)

The simplest form of DDM assumes constant dividend growth forever:

Intrinsic Value = D₁ / (r - g)

Where: D₁ = Expected dividend next year r = Required rate of return (discount rate) g = Perpetual dividend growth rate

Per-Share Value = Intrinsic Value / Shares Outstanding

**When to use:** Mature companies with stable, predictable dividend growth.

**Example:**

Current DPS: $2.50 Expected growth: 5% annually (perpetual) Required return: 10%

Per-Share Value = $2.50 × (1.05) / (0.10 - 0.05) = $2.625 / 0.05 = $52.50/share

**Critical constraint:** r must be > g (otherwise formula breaks down)

#### Formula 2: Two-Stage DDM (Complex)

More realistic model that assumes high initial growth, then declining to terminal growth:

Intrinsic Value = Σ(Dt / (1+r)^t) [Years 1-N] + (DN+1 / (r-g)) / (1+r)^N

Where: Dt = Dividend in year t N = Transition year (typically 5-10) r = Discount rate g = Terminal growth rate (perpetual)

**When to use:** When company has clearly different growth phases (high-growth years, then stable).

**Example structure:**

Years 1-5: High dividend growth (10-12% annually) Years 6-10: Moderate growth (6-8% annually) Year 11+: Terminal growth (2-3% perpetually)

Each phase is discounted separately and summed.

#### Formula 3: Buyback-Adjusted Dividend for Non-Dividend Companies

For companies like Microsoft, Apple, Google that buyback instead of paying dividends:

Effective DPS = (Cash Dividends + Share Repurchases) / Weighted Avg Shares

This creates an "equivalent dividend" that can be valued using DDM.

**Example:**

Cash Dividends Paid: $12.2B Share Repurchases: $27.5B Total Return to Shareholders: $39.7B Weighted Average Shares: 2.4B

Effective DPS = $39.7B / 2.4B = $16.54/share

---

### 4.3 DDM Step-by-Step Calculation

Follow these 9 steps to calculate DDM valuation:

#### **Step 1: Gather Historical Dividend Per Share (Last 10 Years)**

From company investor relations or SEC filings:

Year Dividend Per Share ($) Annual Growth % 2016 $1.44 — 2017 $1.52 5.6% 2018 $1.68 10.5% 2019 $1.82 8.3% 2020 $2.04 12.1% 2021 $2.27 11.3% 2022 $2.48 9.3% 2023 $2.71 9.3% 2024 $2.92 7.7% 2025 $3.15 7.9%

**Quality Check:**
- Do dividends grow consistently?
- Are there any cuts or stalls? (red flag if yes)
- Is growth rate sustainable?

#### **Step 2: Calculate Dividend Growth Rate (CAGR)**

Historical dividend CAGR shows trend:

Dividend CAGR = ($3.15 / $1.44)^(1/9) - 1 = (2.188)^(0.1111) - 1 = 8.9% annual growth

**Interpretation:**
- >10% = High-growth dividend company (excellent)
- 5-10% = Moderate growth (common for mature dividend stocks)
- <5% = Low/stable dividend (mature, defensive)
- Negative = Declining dividend (major red flag)

#### **Step 3: Check Payout Ratio Sustainability**

Verify that dividends are sustainable (not paying out too much of earnings):

Latest Year Data: Net Income: $72.8B Total Dividends Paid: $12.2B

Payout Ratio = $12.2B / $72.8B = 16.7%

**Payout Ratio Assessment:**

Payout Ratio Range Assessment <25% Very conservative, room for growth 25-50% Healthy, sustainable level 50-70% Aggressive, less room for growth

70% Risky, vulnerable to earnings decline 100% Unsustainable, must cut dividend

**In this example:** 16.7% payout ratio is very conservative, indicating dividends can easily grow without strain.

**Alternative check - Dividend Coverage Ratio:**

Dividend Coverage = Operating Cash Flow / Total Dividends = $87.5B / $12.2B = 7.17x

Rule of thumb: >1.5x is comfortable, <1.0x is warning sign

#### **Step 4: Calculate Expected Next Year Dividend (D₁)**

Based on historical growth, estimate next year's dividend:

Conservative approach: Use historical CAGR D₁ = Current DPS × (1 + Historical CAGR) = $3.15 × (1 + 0.089) = $3.15 × 1.089 = $3.43/share

Alternative approach: Use management guidance If company guides 6% growth: D₁ = $3.15 × 1.06 = $3.34

Conservative choice: Use lower of the two = $3.34

#### **Step 5: Establish Required Rate of Return (Discount Rate)**

For dividend stocks, this is typically lower than general equity (lower risk):

Method 1: Simplified approach Required Return = Risk-Free Rate + Dividend Stock Risk Premium

Current 10-Year Treasury: 4.2% Dividend Stock Risk Premium: 4.0% (lower than average 5.8%)

Required Return = 4.2% + 4.0% = 8.2%

Method 2: CAPM approach (more precise) Required Return = Rf + β × (Rm - Rf)

Risk-free rate: 4.2% Beta for dividend stock: 0.85 (less volatile than market) Market return: 10%

Required Return = 4.2% + 0.85 × (10% - 4.2%) = 4.2% + 0.85 × 5.8% = 4.2% + 4.93% = 9.13% ≈ 9.1%

**Use the more conservative estimate: 9.1%**

#### **Step 6: Project Future Dividends (10 Years)**

Use declining growth rate model (similar to DCF):

Years 1-3: Historical growth (8.9%) or slightly conservative Years 4-7: Gradual decline toward terminal Years 8-10: Approach terminal growth (2.5%)

Projected Dividends:

Year Growth % Calculation DPS ($) 1 8.9% $3.34 × 1.089 $3.64 2 8.5% $3.64 × 1.085 $3.95 3 8.0% $3.95 × 1.080 $4.27 4 6.5% $4.27 × 1.065 $4.55 5 5.5% $4.55 × 1.055 $4.80 6 4.5% $4.80 × 1.045 $5.02 7 3.5% $5.02 × 1.035 $5.20 8 3.0% $5.20 × 1.030 $5.36 9 2.7% $5.36 × 1.027 $5.50 10 2.5% $5.50 × 1.025 $5.64

#### **Step 7: Calculate Present Value of Projected Dividends (Years 1-10)**

Discount each year's dividend to present value:

PV = Dividend / (1 + Discount Rate)^Year

Year 1: $3.64 / (1.091)^1 = $3.64 / 1.091 = $3.34 Year 2: $3.95 / (1.091)^2 = $3.95 / 1.191 = $3.32 Year 3: $4.27 / (1.091)^3 = $4.27 / 1.299 = $3.29 Year 4: $4.55 / (1.091)^4 = $4.55 / 1.418 = $3.21 Year 5: $4.80 / (1.091)^5 = $4.80 / 1.548 = $3.10 Year 6: $5.02 / (1.091)^6 = $5.02 / 1.689 = $2.97 Year 7: $5.20 / (1.091)^7 = $5.20 / 1.843 = $2.82 Year 8: $5.36 / (1.091)^8 = $5.36 / 2.011 = $2.67 Year 9: $5.50 / (1.091)^9 = $5.50 / 2.194 = $2.51 Year 10: $5.64 / (1.091)^10 = $5.64 / 2.394 = $2.36

Sum of PV (Years 1-10) = $29.59

#### **Step 8: Calculate Terminal Value (Years 11+)**

Assumes dividends continue in perpetuity at 2.5% growth:

Terminal Value = D₁₀ × (1 + g) / (r - g) = $5.64 × (1 + 0.025) / (0.091 - 0.025) = $5.78 / 0.066 = $87.58

This value occurs in Year 10, so discount to present:

PV(Terminal Value) = $87.58 / (1.091)^10 = $87.58 / 2.394 = $36.58

#### **Step 9: Sum and Calculate Per-Share Value**

Intrinsic Value = Sum(PV Years 1-10) + PV(Terminal Value) = $29.59 + $36.58 = $66.17/share

**This is your DDM fair value: $66.17 per share**

---

### 4.4 DDM Example - Coca-Cola (Dividend-Paying Company)

Complete DDM valuation for Coca-Cola, a classic dividend stock:

#### Company Data

Current Stock Price: $61.50 Current DPS: $1.84 Shares Outstanding: 2.8B

#### Step 1: Historical Dividends (10 Years)

Year DPS ($) Growth % 2016 $1.40 — 2017 $1.48 5.7% 2018 $1.56 5.4% 2019 $1.64 5.1% 2020 $1.72 4.9% 2021 $1.76 2.3% (pandemic impact) 2022 $1.80 2.3% 2023 $1.82 1.1% 2024 $1.84 1.1% 2025 $1.84 0.0% (flat)

**Observation:** Coca-Cola's dividend growth has slowed dramatically (was 5%+, now near 0%)

This is typical for mature dividend aristocrats - sustainable but slow growth.

#### Step 2: Dividend Growth Rate

Recent CAGR (2016-2025): ($1.84 / $1.40)^(1/9) - 1 = 3.5% Recent 5-year growth (2020-2025): ($1.84 / $1.72)^(1/5) - 1 = 1.7%

**Interpretation:** Dividend growth has decelerated. Plan conservatively at 2.5% going forward.

#### Step 3: Payout Ratio Check

Latest Year (2025): Net Income: $11.2B Dividends Paid: $5.1B

Payout Ratio = $5.1B / $11.2B = 45.5%

**Assessment:** 45.5% is moderate and sustainable. Room for modest growth.

#### Step 4: Expected Next Year Dividend

Conservative approach: Assume 2.5% growth D₁ = $1.84 × 1.025 = $1.89/share

#### Step 5: Required Rate of Return

10-Year Treasury: 4.2% Coca-Cola Beta: 0.62 (defensive, less volatile than market) Market return: 10%

Required Return = 4.2% + 0.62 × (10% - 4.2%) = 4.2% + 0.62 × 5.8% = 4.2% + 3.60% = 7.8%

**Lower discount rate reflects Coca-Cola's lower risk profile.**

#### Step 6: Project Future Dividends

Given the company's mature status, use conservative flat growth (2.5%):

Year DPS ($) Calculation 1 $1.89 $1.84 × 1.025 2 $1.94 $1.89 × 1.025 3 $1.98 $1.94 × 1.025 4 $2.03 $1.98 × 1.025 5 $2.08 $2.03 × 1.025 6 $2.13 $2.08 × 1.025 7 $2.18 $2.13 × 1.025 8 $2.24 $2.18 × 1.025 9 $2.29 $2.24 × 1.025 10 $2.35 $2.29 × 1.025

#### Step 7: Discount to Present Value

Year DPS ()DiscountFactorPV() 1 $1.89 1 / 1.078^1 $1.75 2 $1.94 1 / 1.078^2 $1.67 3 $1.98 1 / 1.078^3 $1.60 4 $2.03 1 / 1.078^4 $1.53 5 $2.08 1 / 1.078^5 $1.47 6 $2.13 1 / 1.078^6 $1.41 7 $2.18 1 / 1.078^7 $1.36 8 $2.24 1 / 1.078^8 $1.31 9 $2.29 1 / 1.078^9 $1.27 10 $2.35 1 / 1.078^10 $1.22

Sum PV (Years 1-10) = $14.59

#### Step 8: Terminal Value

Terminal Value = $2.35 × (1.025) / (0.078 - 0.025) = $2.41 / 0.053 = $45.47

PV(Terminal Value) = $45.47 / (1.078)^10 = $45.47 / 2.089 = $21.75

#### Step 9: Calculate Per-Share Value

DDM Fair Value = $14.59 + $21.75 = $36.34/share

Current Price: $61.50 Status: OVERVALUED (according to DDM) Discount: ($36.34 - $61.50) / $61.50 = -40.9%

**Interpretation:** Using conservative 2.5% perpetual growth, Coca-Cola appears overvalued at $61.50. This suggests either:

1. Market expects faster dividend growth than 2.5%
2. Investors value the dividend stability differently
3. This is the correct valuation and stock is overpriced

This is where **multiple valuation methods matter** - one method's result should validate against others.

---

### 4.5 Handling Non-Dividend Companies: Buyback-Adjusted DDM

Not all companies pay dividends. Some, like Microsoft and Apple, return cash through **share buybacks** instead.

#### Buyback as Equivalent Dividend

A buyback reduces share count, which is economically equivalent to a dividend:

Dividend: $10 dividend on 100 shares = $1,000 total Buyback: Repurchase $1,000 of stock (reduces shares to ~99)

Result: Both return $1,000 to shareholders

#### Calculating Buyback-Adjusted Dividend

Total Return = Cash Dividends + Share Repurchases Effective DPS = Total Return / Weighted Average Shares

Example - Microsoft:

Cash Dividends Paid: $12.2B Share Repurchases: $27.5B Total Return: $39.7B

Weighted Average Shares: 2.4B Effective DPS = $39.7B / 2.4B = $16.54/share

This can now be valued using DDM formulas.

#### Full Example: Apple (Buyback-Heavy Company)

Year Dividends (B)Buybacks(B) Total Return ($B) 2016 $3.3 $24.5 $27.8 2017 $3.7 $32.5 $36.2 2018 $3.9 $23.6 $27.5 2019 $4.1 $22.2 $26.3 2020 $4.3 $22.1 $26.4 2021 $4.4 $27.5 $31.9 2022 $4.6 $27.3 $31.9 2023 $4.7 $29.0 $33.7 2024 $4.8 $30.8 $35.6 2025 $5.0 $32.5 $37.5

**Calculate Total Return Growth:**

Total Return CAGR = ($37.5B / $27.8B)^(1/9) - 1 = (1.348)^(0.1111) - 1 = 3.6% growth

**Calculate per-share total return:**

Year Total Return Shares Outstanding Return Per Share 2025 $37.5B 16.3B $2.30

Now value using DDM with $2.30 per share as the "dividend."

---

### 4.6 DDM Limitations & When to Use

#### Strengths of DDM

✅ **Simple and intuitive** - Easy to understand what you're valuing  
✅ **Based on actual cash** - Dividends and buybacks are real cash returned  
✅ **Conservative** - Only values cash actually returned, not all FCF  
✅ **Good for mature companies** - Predictable, stable dividend streams  
✅ **Works with partial data** - 5-year dividend history often sufficient  

#### Limitations of DDM

❌ **Only values dividends/buybacks** - Ignores cash retained for growth  
❌ **Dividend cuts are disasters** - One cut can invalidate entire model  
❌ **Assumes stable growth forever** - Unrealistic assumption  
❌ **Less useful for growth companies** - Companies investing heavily may not pay dividends  
❌ **Sensitive to discount rate** - Small changes create large valuation swings  
❌ **Terminal value dominates** - Usually 60-80% of value is from Year 11+  

#### When DDM Works Best

✅ Mature dividend-paying companies (utilities, consumer staples, financials)  
✅ Companies with 5+ years of consistent dividend growth  
✅ Companies with sustainable payout ratios (30-60%)  
✅ Companies returning consistent cash to shareholders  
✅ When combined with DCF (validation purpose)  

#### When DDM Works Poorly

❌ Growth companies (low/no dividends)  
❌ Companies with inconsistent dividend history  
❌ Companies with unsustainable high payout ratios  
❌ Young companies (limited dividend history)  
❌ Companies in uncertain competitive positions  

#### DDM Sanity Checks

Before accepting a DDM valuation:

1. **Is payout ratio sustainable?**
   - Should be 30-60% of earnings
   - Should be covered by operating cash flow
   - Check if it's been stable historically

2. **Is dividend growth realistic?**
   - Can't grow faster than earnings
   - Can't grow faster than business growth
   - Historical trend should inform expectations

3. **Is the discount rate reasonable?**
   - Compare to company's cost of equity
   - Lower risk = lower discount rate
   - Check vs industry peers

4. **Does result align with business quality?**
   - High-quality company should have higher valuation
   - Compare to P/E multiple implied by DDM
   - Does it make sense?

---

### 4.7 Combining DCF and DDM

#### Why Use Both?

DCF and DDM answer different questions:

- **DCF:** "What is all the cash this business will ever generate worth?"
- **DDM:** "What is the cash returned to shareholders worth?"

DCF value should be **higher** than DDM value (since DCF includes all FCF, not just dividends).

#### Typical Relationship

Company Type DCF Value DDM Value Ratio High-growth tech $500 $300 1.67x (DCF > DDM) Mature dividend stock $75 $72 1.04x (similar) Non-dividend growth $300 $50 6.0x (DCF >> DDM)

#### Using Both Methods Together

Step 1: Calculate DCF (primary valuation) Result: $156/share

Step 2: Calculate DDM (validation) Result: $145/share

Step 3: Compare Difference: $11 (7% variance)

Conclusion: Both methods converge on ~$150/share High confidence in valuation Stock at $120 = clear undervaluation

When methods diverge significantly, investigate why:

If DCF = $100 but DDM = $50: → Check: Is company spending too much on growth? → Check: Are dividends being cut? → Check: Is business deteriorating? → Action: Deeper analysis needed

---

### Summary: DDM Method

DDM Intrinsic Value = Present Value of All Future Dividends (or Buybacks)

Process:

Gather 10 years historical dividend per share
Calculate historical dividend growth rate (CAGR)
Check payout ratio sustainability (30-60%)
Project future dividends (10 years) with declining growth
Establish required rate of return (discount rate)
Discount all projected dividends to present value
Calculate terminal value (perpetual growth after Year 10)
Sum all discounted dividends
Result: Fair value per share
For non-dividend companies: Add buybacks to dividends to create "effective dividend" Value using same process

Result: Fair value per share (typically lower than DCF)

Best use: Validation of DCF, or primary method for dividend stocks

Key insight: What matters is cash returned to shareholders, not cash generated by the business.
---

## **Section 5: Multiples Valuation Model**

### **5.1 Multiples Philosophy & Foundational Concepts**

#### **What is Multiples Valuation?**

Multiples valuation is based on a simple concept:

> **The value of a company is what similar companies are worth, adjusted for size.**

Rather than calculating intrinsic value from first principles (like DCF), multiples ask: "What are comparable companies trading at, and what should this company trade at given its size and profitability?"

#### **Why Use Multiples?**

Multiples provide several advantages:

1. **Market-based reality** - Reflects what investors actually pay, not theoretical models
2. **Simple and intuitive** - Easy to understand and calculate
3. **Quick sanity check** - Compare your DCF result against market sentiment
4. **Works for any company** - Even unprofitable companies can be valued with P/S
5. **Industry comparisons** - See if company is cheap or expensive vs peers

#### **Key Concept: The Three Main Multiples**

| Multiple | Formula | What It Measures | Best For |
|----------|---------|------------------|----------|
| **P/E Ratio** | Stock Price / Earnings Per Share | Earnings multiples | Profitable companies |
| **P/B Ratio** | Market Cap / Book Value | Asset-based value | Asset-heavy companies |
| **P/S Ratio** | Market Cap / Revenue | Revenue multiples | All companies (no earnings needed) |

#### **How Multiples Work**

Step 1: Identify 5 peer companies (similar business, size, industry) Step 2: Calculate average P/E multiple across peers (e.g., 18x) Step 3: Get target company's EPS (e.g., $5.00) Step 4: Calculate fair value = $5.00 × 18 = $90/share

**Key insight:** If peers trade at 18x earnings and this company trades at 15x, it's cheaper. If it trades at 25x, it's more expensive.

#### **Multiples vs DCF: Different Questions**

| Question | Method | Answer |
|----------|--------|--------|
| "What is this company WORTH?" | DCF | Based on future cash flows |
| "What is market WILLING TO PAY?" | Multiples | Based on comparable companies |
| "Is it cheap or expensive NOW?" | Multiples | Compared to peers |

**Best practice:** Use DCF for intrinsic value, multiples to validate if price is reasonable vs peers.

---

## Section 5: Multiples Valuation Model

**Method 3 of 5 - Weight in Consensus: 20%**

#### **Formula 1: P/E Multiple Valuation**

Fair Value = Peer Average P/E × Target Company EPS

Fair Value Per Share = Peer Avg P/E × EPS

**Example:**
Peer Average P/E: 18x Target Company EPS: $5.00

Fair Value = 18 × $5.00 = $90/share

**When to use:** For profitable companies with consistent earnings

**When NOT to use:** 
- Negative earnings (P/E is undefined)
- Cyclical earnings (use normalized EPS)
- One-time charges (adjust earnings)

#### **Formula 2: P/B Multiple Valuation**

Fair Value = Peer Average P/B × Target Company Book Value Per Share

Book Value Per Share = Total Equity / Shares Outstanding

**Example:**

Peer Average P/B: 2.5x Target Company Book Value/Share: $30

Fair Value = 2.5 × $30 = $75/share
**When to use:** For profitable companies with consistent earnings

**When NOT to use:** 
- Negative earnings (P/E is undefined)
- Cyclical earnings (use normalized EPS)
- One-time charges (adjust earnings)

#### **Formula 2: P/B Multiple Valuation**

Fair Value = Peer Average P/B × Target Company Book Value Per Share

Book Value Per Share = Total Equity / Shares Outstanding

**Example:**

Peer Average P/B: 2.5x Target Company Book Value/Share: $30

Fair Value = 2.5 × $30 = $75/share

**When to use:** 
- Asset-heavy companies (banks, insurance)
- Companies with negative earnings
- Real estate/tangible asset companies

**When NOT to use:**
- Service companies (book value not meaningful)
- Companies with substantial intangible assets
- High-growth tech companies

#### **Formula 3: P/S Multiple Valuation**

Fair Value = Peer Average P/S × Target Company Revenue Per Share

Revenue Per Share = Total Revenue / Shares Outstanding

**Example:**

Peer Average P/S: 3.5x Target Company Revenue/Share: $25

Fair Value = 3.5 × $25 = $87.50/share

**When to use:**
- Any company (profitable or not)
- Early-stage companies with losses
- When earnings quality is questionable

**When NOT to use:**
- Very low-margin businesses
- Companies with one-time large expenses
- When profit margins vary widely

---

### **5.3 Multiples Step-by-Step Calculation**

Follow these 6 steps to calculate multiples valuation:

#### **Step 1: Identify 5 Comparable Peer Companies**

Select companies with:
- Similar business models
- Similar size (within 50-200% of target)
- Same industry/sector
- Comparable growth rates
- Comparable profitability

**Example - For Microsoft:**

Peer companies:
1. Apple (AAPL) - Large-cap tech
2. Alphabet/Google (GOOGL) - Large-cap software/services
3. Adobe (ADBE) - Software/cloud services
4. Salesforce (CRM) - Cloud software
5. ServiceNow (NOW) - Enterprise software

**Why these peers:**
- All large-cap tech/software companies
- Similar business models (software/cloud services)
- Similar growth rates (5-15% annually)
- Similar profit margins (20%+ net margins)

#### **Step 2: Gather Current Market Data for Peers**

For each peer, collect:
- Current stock price
- Shares outstanding
- Market capitalization
- Latest earnings per share (TTM)
- Latest revenue per share (TTM)
- Book value per share
- Total equity

**Example data:**

| Company | Price | EPS | Revenue/Share | Book Value/Share |
|---------|-------|-----|---|---|
| AAPL | $230 | $6.05 | $35.50 | $39.00 |
| GOOGL | $180 | $6.40 | $60.25 | $101.00 |
| ADBE | $625 | $13.20 | $97.50 | $82.00 |
| CRM | $290 | $1.90 | $40.75 | $10.25 |
| NOW | $745 | $2.85 | $82.50 | $15.00 |

#### **Step 3: Calculate Individual Multiples for Each Peer**

**P/E Ratio = Stock Price / EPS**

AAPL: $230 / $6.05 = 38.0x GOOGL: $180 / $6.40 = 28.1x ADBE: $625 / $13.20 = 47.3x CRM: $290 / $1.90 = 152.6x (Note: High due to lower earnings) NOW: $745 / $2.85 = 261.4x (Note: Very high due to low earnings)

**P/B Ratio = Stock Price / Book Value Per Share**

AAPL: $230 / $39.00 = 5.9x GOOGL: $180 / $101.00 = 1.8x ADBE: $625 / $82.00 = 7.6x CRM: $290 / $10.25 = 28.3x NOW: $745 / $15.00 = 49.7x

**P/S Ratio = Stock Price / Revenue Per Share**

AAPL: $230 / $35.50 = 6.5x GOOGL: $180 / $60.25 = 3.0x ADBE: $625 / $97.50 = 6.4x CRM: $290 / $40.75 = 7.1x NOW: $745 / $82.50 = 9.0x

#### **Step 4: Calculate Average (or Median) Multiples**

**For P/E:** Average the five peer P/E ratios

Individual ratios: 38.0x, 28.1x, 47.3x, 152.6x, 261.4x Average: (38.0 + 28.1 + 47.3 + 152.6 + 261.4) / 5 = 105.5x

Problem: Two outliers (CRM, NOW) skew average upward! Better approach: Use MEDIAN instead

Sorted: 28.1x, 38.0x, 47.3x, 152.6x, 261.4x Median (middle value): 47.3x

Alternative: Remove outliers Use: (38.0 + 28.1 + 47.3) / 3 = 37.8x (more reasonable)

**Use Median or trimmed average when outliers exist!**

**For P/B:** Average the peer P/B ratios

Individual ratios: 5.9x, 1.8x, 7.6x, 28.3x, 49.7x Median: 7.6x Trimmed (exclude outliers): (5.9 + 1.8 + 7.6) / 3 = 5.1x

**For P/S:** Average the peer P/S ratios

Individual ratios: 6.5x, 3.0x, 6.4x, 7.1x, 9.0x Median: 6.5x Average: (6.5 + 3.0 + 6.4 + 7.1 + 9.0) / 5 = 6.4x

**Conservative approach:** Use median or trimmed average, not simple average.

#### **Step 5: Get Target Company's Financial Metrics**

For Microsoft (our target company):

Current Stock Price: $420.50 Trailing Twelve Months (TTM) Data: Earnings Per Share (EPS): $12.15 Revenue Per Share: $60.75 Book Value Per Share: $35.00 Shares Outstanding: 2.4B

#### **Step 6: Calculate Fair Values Using Each Multiple**

**P/E Fair Value:**

Fair Value = Peer Median P/E × Microsoft EPS = 37.8x × $12.15 = $459.57/share

**P/B Fair Value:**

Fair Value = Peer Median P/B × Microsoft Book Value/Share = 5.1x × $35.00 = $178.50/share

**P/S Fair Value:**

Fair Value = Peer Median P/S × Microsoft Revenue/Share = 6.4x × $60.75 = $388.80/share

**Multiples Valuation Result:**

P/E Fair Value: $459.57 P/B Fair Value: $178.50 P/S Fair Value: $388.80

Average: ($459.57 + $178.50 + $388.80) / 3 = $342.29/share

Current Price: $420.50 Status: OVERVALUED vs multiples (11% above average fair value)

---

### **5.4 Handling Negative Earnings: Critical Issue**

#### **The P/E Problem**

P/E ratio is **undefined** when earnings are negative:

P/E = Stock Price / EPS = $100 / (-$5) = -20x (meaningless!)

This is a critical problem because:
- Many growth companies are unprofitable
- P/E cannot be used for loss-making companies
- Using P/E would exclude valuable companies

#### **Solutions for Negative Earnings**

**Option 1: Use Normalized Earnings**

If company had one-time charges, normalize earnings:

Reported EPS: -$2.00 (due to $500M one-time charge) One-time charge per share: $2.50 Normalized EPS: -$2.00 + $2.50 = $0.50

Now P/E = $100 / $0.50 = 200x (still high, but usable)

**Option 2: Use Forward Earnings**

Project next year's earnings if profitable:

Current year: Loss of $2.00 per share Management guidance: Next year EPS $3.00 Use forward P/E with next year earnings

Fair Value = 25x × $3.00 = $75/share

**Option 3: Switch to P/B Multiple**

Use Price-to-Book instead (always works):

Current Price: $100 Book Value/Share: $50 P/B = 100/50 = 2.0x

If peers trade at 1.5x P/B: Fair Value = 1.5x × $50 = $75/share

**Option 4: Use P/S Multiple**

Use Price-to-Sales (works regardless of profitability):

Current Price: $100 Revenue/Share: $40 P/S = 100/40 = 2.5x

If peers trade at 2.0x P/S: Fair Value = 2.0x × $40 = $80/share

#### **Decision Tree for Negative Earnings**

Company has negative earnings? │ ├─ YES, but one-time charges? │ └─ Normalize earnings, use P/E with adjusted EPS │ ├─ YES, but expect profitability next year? │ └─ Use forward P/E with projected earnings │ ├─ YES, asset-heavy company? │ └─ Use P/B multiple │ └─ YES, ongoing losses? └─ Use P/S multiple (doesn't require profitability)

#### **Real Example: Company with Negative Earnings**

**Company: Early-stage biotech (BIOTECH)**

Current Stock Price: $50 Annual Revenue: $10M Shares Outstanding: 5M

Revenue Per Share: $10M / 5M = $2.00/share P/S Ratio: $50 / $2.00 = 25x

Peer biotech companies (profitable ones) trade at 8x P/S Fair Value = 8x × $2.00 = $16/share

Conclusion: Stock appears overvalued at $50 vs peers at comparable stage

---

### **5.5 Multiples Valuation Example - Kerry Group**

Complete multiples valuation for Kerry Group (multinational food/beverage company):

#### **Step 1: Peer Selection**

Kerry Group operates in food ingredients and flavors. Comparable peers:

1. Givaudan (GIVN) - Flavors and fragrances
2. IFF (International Flavors & Fragrances) - Flavors/ingredients
3. Ingredion (INGR) - Food ingredients
4. Archer Daniels (ADM) - Agricultural commodities/ingredients
5. Tate & Lyle (TATE) - Food ingredients/sweeteners

#### **Step 2: Peer Financial Data**

| Company | Price | EPS | Rev/Share | Book Value/Share |
|---------|-------|-----|-----------|------------------|
| GIVN | $2,800 | $16.50 | $520 | $180 |
| IFF | $95 | $2.20 | $18.50 | $8.00 |
| INGR | $102 | $4.80 | $45.50 | $15.00 |
| ADM | $68 | $4.15 | $28.75 | $12.00 |
| TATE | $890 | $7.20 | $95.00 | $35.00 |

#### **Step 3: Calculate Peer Multiples**

**P/E Ratios:**
GIVN: 2,800 / 16.50 = 169.7x IFF: 95 / 2.20 = 43.2x INGR: 102 / 4.80 = 21.3x ADM: 68 / 4.15 = 16.4x TATE: 890 / 7.20 = 123.6x

Median P/E: 43.2x Trimmed (exclude outliers): (43.2 + 21.3 + 16.4) / 3 = 27.0x

**P/B Ratios:**
GIVN: 2,800 / 180 = 15.6x IFF: 95 / 8.00 = 11.9x INGR: 102 / 15.00 = 6.8x ADM: 68 / 12.00 = 5.7x TATE: 890 / 35.00 = 25.4x

Median P/B: 11.9x Trimmed: (11.9 + 6.8 + 5.7) / 3 = 8.1x

**P/S Ratios:**
GIVN: 2,800 / 520 = 5.4x IFF: 95 / 18.50 = 5.1x INGR: 102 / 45.50 = 2.2x ADM: 68 / 28.75 = 2.4x TATE: 890 / 95.00 = 9.4x

Median P/S: 5.1x Average: (5.4 + 5.1 + 2.2 + 2.4 + 9.4) / 5 = 4.9x

#### **Step 4: Kerry Group Metrics**

Current Stock Price: €110 Trailing Twelve Months: EPS: €2.85 Revenue Per Share: €38.50 Book Value Per Share: €12.00

#### **Step 5: Calculate Fair Values**

**P/E Fair Value:**
Fair Value = 27.0x × €2.85 = €76.95/share

**P/B Fair Value:**
Fair Value = 8.1x × €12.00 = €97.20/share

**P/S Fair Value:**
Fair Value = 4.9x × €38.50 = €188.65/share

**Multiples Valuation Result:**

P/E Fair Value: €76.95 P/B Fair Value: €97.20 P/S Fair Value: €188.65

Average: (76.95 + 97.20 + 188.65) / 3 = €120.93 Median: €97.20

Current Price: €110 Status: FAIRLY VALUED (close to median fair value)

Note: P/S is much higher, suggesting market prices in growth expectations

---

### **5.6 Multiples Limitations & When to Use**

#### **Strengths of Multiples**

✅ **Quick and intuitive** - Easy to calculate and understand
✅ **Market-based** - Reflects actual investor sentiment
✅ **Works for any company** - Even unprofitable ones (use P/S)
✅ **Good validation tool** - Cross-checks DCF results
✅ **Industry context** - See if company is cheap vs peers
✅ **Data readily available** - Stock price and earnings are public

#### **Limitations of Multiples**

❌ **Doesn't consider quality** - Peers might all be overvalued
❌ **Market sentiment dependent** - Affected by bull/bear markets
❌ **Requires good peer selection** - Wrong peers = wrong answer
❌ **Ignores business quality** - High-quality company might deserve higher multiple
❌ **Cyclical distortions** - Peak earnings years might show artificially low multiples
❌ **Sector-dependent** - Tech multiples are 3x higher than utilities
❌ **One-time items ignored** - Extraordinary charges distort multiples

#### **When Multiples Work Best**

✅ **Large, mature companies** - Plenty of comparable data
✅ **Stable industries** - Predictable multiples (utilities, consumer staples)
✅ **Profitable companies** - P/E works best when earnings are clear
✅ **Validation tool** - Cross-checking DCF valuations
✅ **Market sentiment check** - Is market being rational?
✅ **Quick estimates** - When deep analysis not needed

#### **When Multiples Work Poorly**

❌ **Startup/early-stage** - Few or no comparables
❌ **Unprofitable companies** - Can't use P/E
❌ **Highly cyclical** - Earnings vary wildly
❌ **Companies with anomalies** - One-time charges distort multiples
❌ **Asset write-downs** - Distort book value multiples
❌ **Merger/acquisition targets** - Comparison not relevant
❌ **Distressed industries** - Peers also overvalued/undervalued

#### **Multiples Sanity Checks**

Before accepting a multiples valuation:

1. **Are the peers truly comparable?**
   - Same business model?
   - Similar size and growth?
   - Same profitability level?
   - If not, valuation is questionable

2. **Are multiples reasonable?**
   - If P/E is 50x for a 3% growth company, peers are overvalued
   - If stock trades at 0.5x P/B vs peers at 2.0x, why? (quality? book value flawed?)
   - Does multiple match business quality?

3. **Do multiple methods converge?**
   - P/E says $100, P/B says $50, P/S says $200?
   - Divergence suggests issue with one method
   - Investigate which multiple is most reliable

4. **Has anything changed?**
   - Have peers changed business strategy?
   - New competition entering the space?
   - Industry consolidation affecting multiples?
   - Economic cycle changed (expansion → recession)?

5. **Compare to historical multiples:**
   - Is company trading at its historical average multiple?
   - Above average = expensive, below = cheap
   - But context matters (growth rate changed, profitability improved, etc.)

---

### **Summary: Multiples Valuation Method**

**Multiples Fair Value = Peer Average Multiple × Target Company Metric**

**Process:**

1. Identify 5 comparable peer companies
2. Gather current market data (price, earnings, revenue, book value)
3. Calculate multiples for each peer (P/E, P/B, P/S)
4. Calculate average (or median, or trimmed average) multiples
5. Get target company's financial metrics
6. Calculate fair values using each multiple
7. Average the results for final valuation

**For negative earnings:**
- Use normalized earnings, forward earnings, P/B, or P/S
- Never use P/E with negative earnings (meaningless)

**Result:** Fair value per share (market-based, not theoretical)

**Best use:** Validation of DCF, market sentiment check, quick estimates

**Key insight:** What the market pays for comparable companies tells you what this company should trade at.

**Critical reminder:** Multiples are only as good as your peer selection. Bad peers = bad valuation.

---

## Section 6: Warren Buffett Valuation Techniques

**Method 4 of 5 - Weight in Consensus: 15%**

### **6.1 Buffett's Philosophy & Investment Approach**

#### **Who is Warren Buffett?**

Warren Buffett is the world's most successful investor. His investment philosophy has generated over 20% annual returns for 60+ years—far exceeding the market average of 10%.

**Key quote:**

> **"Price is what you pay. Value is what you get."**

This single sentence captures Buffett's entire philosophy: the goal is to pay less than something is worth.

#### **Buffett's Core Beliefs**

1. **Focus on value, not price** - Buy when price is below intrinsic value
2. **Margin of safety is critical** - Only buy with 25-50% discount to fair value
3. **Long-term holding** - Think in terms of 10+ year ownership
4. **Business quality matters most** - Prefer excellent businesses at fair prices over mediocre businesses at cheap prices
5. **Cash flow is king** - Real earnings matter, not accounting tricks
6. **Think like a business owner** - You're buying pieces of businesses, not trading stocks

#### **Why Two Valuation Techniques?**

Buffett uses two complementary techniques:

1. **Owner's Earnings Valuation** - Calculates intrinsic value based on normalized cash earnings
2. **Perpetual Bond Yield Comparison** - Validates valuation by comparing earnings yield to risk-free rate

Together, they provide both mathematical rigor and a reality check.

---

### **6.2 Buffett Technique #1: Owner's Earnings Valuation**

#### **6.2.1 Owner's Earnings Concept**

**Owner's Earnings** is Buffett's term for the cash available to the owner of a business.

**Formula:**

Owner's Earnings = Net Income + Depreciation & Amortization - Capital Expenditures - Change in Working Capital

**What each component means:**

| Component | Meaning | Example |
|-----------|---------|---------|
| **Net Income** | Accounting profit | $100M profit reported |
| **+ D&A** | Non-cash charges added back | $20M depreciation |
| **- CapEx** | Cash spent on equipment/infrastructure | $30M spent on new factories |
| **- Δ Working Capital** | Change in cash tied up in operations | $5M more cash needed for inventory |
| **= Owner's Earnings** | Real cash owner can take out | $85M available to shareholders |

#### **Why Owner's Earnings Matters**

**Example: Two companies with identical net income**

**Company A (Tech Software):**
- Net Income: $100M
- D&A: $5M (low - software doesn't depreciate much)
- CapEx: $10M (low - cloud infrastructure reuses assets)
- Δ Working Capital: $0 (no physical inventory)
- **Owner's Earnings: $95M** (95% of net income converts to cash)

**Company B (Manufacturing):**
- Net Income: $100M
- D&A: $20M (high - factories depreciate)
- CapEx: $50M (high - must replace aging equipment)
- Δ Working Capital: $15M (cash tied up in inventory/receivables)
- **Owner's Earnings: $55M** (only 55% of net income converts to cash)

**Conclusion:** Company A is much better despite identical net income. Owner's Earnings reveals the truth.

#### **Why Buffett Prefers Owner's Earnings**

✅ **Harder to manipulate** - Can't fake depreciation or capital spending
✅ **Focuses on cash** - Real money, not accounting fiction
✅ **Accounts for reinvestment** - Shows cost of staying in business
✅ **Long-term sustainability** - Reveals if business is real or unsustainable

#### **6.2.2 Calculating 10-Year Normalized Owner's Earnings**

**Step 1: Gather 10 years of data**

For each year from SEC filings, collect:
- Net Income (from income statement)
- Depreciation & Amortization (from cash flow statement)
- Capital Expenditures (from cash flow statement, or calculated from balance sheet changes)
- Change in Working Capital (from cash flow statement)

**Step 2: Calculate year-by-year Owner's Earnings**

Year Net Income D&A CapEx Δ WC Owner's Earnings (M)(M) (M)(M) ($M) 2016 $8,500 $1,200 $2,100 $300 $7,300 2017 $9,200 $1,350 $2,200 $250 $8,100 2018 $10,100 $1,400 $2,300 $200 $9,000 2019 $11,200 $1,500 $2,500 $350 $9,850 2020 $12,500 $1,600 $2,700 $400 $10,700 2021 $13,800 $1,800 $2,900 $300 $11,800 2022 $14,200 $1,900 $3,100 $250 $12,250 2023 $15,500 $2,000 $3,300 $400 $13,800 2024 $16,800 $2,100 $3,500 $350 $15,050 2025 $18,200 $2,200 $3,700 $300 $16,400

**Step 3: Calculate 10-year average**

10-Year Average Owner's Earnings = ($7,300 + $8,100 + $9,000 + ... + $16,400) / 10 = $114,150 / 10 = $11,415M

**Step 4: Assess normalization (adjust for anomalies)**

Check if any years had unusual items:

2020: COVID impact - revenues down 5%, but that's over now 2022: Large capital expenditure for new factory - one-time event 2025: Restructuring charge included in net income? Remove it.

Conservative approach: Use 10-year average as-is OR: Remove extreme years and use 8-year average if outliers exist

Decision: Use $11,415M as our normalized Owner's Earnings

#### **6.2.3 Assigning Quality Multiple (15-35x)**

The quality multiple reflects how durable and excellent the business is.

**Quality Multiple Assessment Table:**

| Business Quality Level | Characteristics | Multiple | Examples |
|------------------------|-----------------|----------|----------|
| **Exceptional** | Unassailable moat, 20%+ ROE, very consistent earnings, 20%+ growth sustainable | **28-35x** | Coca-Cola, Apple, Microsoft (high-quality moats) |
| **Strong** | Strong moat, 15-20% ROE, consistent earnings, 8-12% growth | **22-27x** | Mastercard, Visa, PayPal (payment networks) |
| **Good** | Identifiable moat, 12-15% ROE, mostly consistent, 5-8% growth | **16-21x** | Starbucks, Adobe, Chipotle (brand/switching costs) |
| **Moderate** | Modest competitive advantage, 10-12% ROE, average growth 3-5% | **12-15x** | Many industrials, retailers (average quality) |
| **Weak** | Commodity business, <10% ROE, erratic earnings, 0-3% growth | **6-11x** | Cyclical businesses, weak moats (low quality) |

**How to assign quality multiple:**

Ask these questions about the target company:

1. **Economic Moat Strength** (0-10 points)
   - Unassailable moat (brand, switching costs, network): 9-10
   - Strong moat with durable competitive advantage: 7-8
   - Identifiable but not ironclad moat: 5-6
   - Weak or questionable moat: 1-4

2. **Return on Equity** (0-10 points)
   - >20% ROE: 9-10
   - 15-20% ROE: 7-8
   - 12-15% ROE: 5-6
   - 10-12% ROE: 3-4
   - <10% ROE: 1-2

3. **Earnings Consistency** (0-10 points)
   - Very smooth, predictable earnings (CV <10%): 9-10
   - Mostly consistent (CV 10-15%): 7-8
   - Somewhat variable (CV 15-25%): 5-6
   - Erratic (CV >25%): 1-4

4. **Growth Sustainability** (0-10 points)
   - 10%+ organic growth, sustainable: 8-10
   - 5-10% sustainable growth: 6-7
   - 3-5% sustainable growth: 4-5
   - <3% or uncertain growth: 1-3

**Total Quality Score: Sum of all four = 0-40 points**

**Convert to multiple:**
Quality Score Multiple 36-40 (Exceptional) 30-35x 30-35 (Strong) 23-29x 24-29 (Good) 17-22x 18-23 (Moderate) 12-16x <18 (Weak) 6-11x

#### **6.2.4 Example: Apple Inc.**

**Step 1: Normalized Owner's Earnings**

From Apple's financial statements:

10-Year Average (2016-2025): Average Net Income: $60.2B Average D&A: $11.8B Average CapEx: $10.2B Average Δ Working Capital: $1.2B

Normalized Owner's Earnings = $60.2B + $11.8B - $10.2B - $1.2B = $60.6B per year

**Step 2: Quality Assessment**

Moat Strength: 10/10 (iconic brand, ecosystem lock-in, pricing power) ROE: 10/10 (>100% ROE - capital-light model, returns to shareholders) Earnings Consistency: 9/10 (very smooth, high-margin business) Growth Sustainability: 8/10 (5-7% organic growth, sustainable)

Total Quality Score: 37/40 (Exceptional) Assigned Multiple: 32x (top tier)

**Step 3: Calculate Intrinsic Value**

Owner's Earnings: $60.6B Quality Multiple: 32x Enterprise Value = $60.6B × 32 = $1,939.2B

Less: Net Debt Total Debt: $110B Cash: $180B Net Cash: ($70B)

Equity Value = $1,939.2B - (-$70B) = $2,009.2B

Per-Share Value = $2,009.2B / 16.5B shares = $121.76/share

**Result: Apple's Owner's Earnings fair value = $121.76/share**

---

### **6.3 Buffett Technique #2: Perpetual Bond Yield Comparison**

#### **6.3.1 Philosophy: Earnings Yield vs Risk-Free Rate**

This technique asks: **"What return would you get from owning this stock vs owning Treasury bonds?"**

**Key insight:**

- Treasury bonds offer safe, guaranteed return (4.2% in 2026)
- Stocks offer uncertain, higher return (earnings yield)
- You only buy stocks if earnings yield exceeds bond yield + risk premium

#### **6.3.2 The Formulas**

**Earnings Yield:**

Earnings Yield = Owner's Earnings / Market Capitalization × 100%

Example: Owner's Earnings: $60.6B Market Cap: $3,000B Earnings Yield = ($60.6B / $3,000B) × 100% = 2.02%

**Required Return (using CAPM):**

Required Return = Risk-Free Rate + Risk Premium

Where: Risk-Free Rate = 10-Year Treasury Yield (e.g., 4.2%) Risk Premium = Extra return demanded for stock risk (typically 3-5% for quality stocks)

Example: Risk-Free Rate: 4.2% Risk Premium for quality stock: 3.5% Required Return: 4.2% + 3.5% = 7.7%

**Assessment:**

IF Earnings Yield > Required Return THEN Stock is UNDERVALUED (attractive)

ELSE IF Earnings Yield ≈ Required Return THEN Stock is FAIRLY VALUED

ELSE (Earnings Yield < Required Return) THEN Stock is OVERVALUED (avoid or sell)

#### **6.3.3 Step-by-Step Calculation**

**Step 1: Calculate Owner's Earnings**

(From previous section - already calculated as $60.6B for Apple)

**Step 2: Get Current Market Cap**

Stock Price: $180/share Shares Outstanding: 16.5B Market Cap = $180 × 16.5B = $2,970B

**Step 3: Calculate Earnings Yield**

Earnings Yield = ($60.6B / $2,970B) × 100% = 0.0204 = 2.04%

**Step 4: Get Risk-Free Rate**

Check current 10-Year Treasury Yield:

Current 10-Year Treasury: 4.2% (as of 2026-03-14)

**Step 5: Assess Risk Premium**

For Apple (high-quality business, low-risk):

Quality Assessment:

Brand moat: Exceptional
Earnings stability: Very high
Financial strength: Fortress balance sheet
Growth: Sustainable
Risk Premium for this quality: 3.0-3.5% Use: 3.2% (conservative estimate)

**Step 6: Calculate Required Return**

Required Return = 4.2% + 3.2% = 7.4%

**Step 7: Compare and Assess**

Apple's Earnings Yield: 2.04% Required Return: 7.4% Shortfall: 7.4% - 2.04% = 5.36%

Assessment: OVERVALUED

Interpretation: Apple's stock returns only 2.04% in earnings yield You could get 4.2% (risk-free) from Treasury bonds You'd actually LOSE 2.16% by owning Apple instead of bonds!

For Apple to be attractive, earnings yield must exceed 7.4% That would require stock price of approximately $82-$90 (At $82: EY = $60.6B / $1,353B = 4.48%, still below 7.4%) (At $65: EY = $60.6B / $1,073B = 5.64%, still below 7.4%) (At $45: EY = $60.6B / $742.5B = 8.15%, now attractive!)

Current price $180 requires stock to fall to ~$45 for attractiveness That's a 75% decline!

#### **6.3.4 Real Example: Coca-Cola**

**Coca-Cola Perpetual Bond Yield Analysis:**

Owner's Earnings: $10.2B (normalized) Stock Price: $61.50 Shares Outstanding: 2.8B Market Cap: $61.50 × 2.8B = $172.2B

Earnings Yield = ($10.2B / $172.2B) × 100% = 5.92%

Risk-Free Rate: 4.2% Risk Premium for Coca-Cola (stable, predictable): 2.5% Required Return: 4.2% + 2.5% = 6.7%

Comparison: Coca-Cola Earnings Yield: 5.92% Required Return: 6.7% Shortfall: 0.78%

Assessment: SLIGHTLY OVERVALUED (but close to fair)

Why acceptable despite shortfall:

Shortfall is small (less than 1%)
Coca-Cola is exceptionally safe and predictable
Margin of error is acceptable given business quality

#### **6.3.5 Risk Premium Assessment Table**

How to determine appropriate risk premium:

| Business Quality | Risk Characteristics | Risk Premium |
|------------------|---------------------|--------------|
| **Exceptional** | Fortress balance sheet, strong moat, very predictable | **2.5-3.5%** |
| **Strong** | Solid balance sheet, identifiable moat, mostly predictable | **3.5-4.5%** |
| **Good** | Good balance sheet, modest moat, some variability | **4.5-5.5%** |
| **Moderate** | Average balance sheet, weak moat, moderate variability | **5.5-6.5%** |
| **Weak** | Weak balance sheet, no moat, high variability | **6.5-8.0%** |

**Decision rule:**
- Add 0.5% premium for each red flag (high debt, weak moat, etc.)
- Subtract 0.5% premium for each strength (fortress balance, strong moat, etc.)

---

### **6.4 Buffett Margin of Safety**

#### **Definition**

**Margin of Safety** is the gap between intrinsic value and current price.

Margin of Safety = (Fair Value - Current Price) / Fair Value × 100%

Positive margin = Stock is undervalued (buy opportunity) Negative margin = Stock is overvalued (avoid or sell)

#### **Buffett's Preference: 25-50% Margin**

Buffett rarely buys without substantial discount:

If fair value is $100: Will buy at: $50-75 (25-50% margin) Won't buy at: $80-100 (0-20% margin) Avoid at: $100+ (negative margin)

**Why such large margins?**

1. **Valuation is uncertain** - Fair value might be $80-120, not exactly $100
2. **Business quality can deteriorate** - Moat might weaken
3. **Market can surprise** - Economic recession, industry disruption
4. **Provides upside** - Even if wrong, you still make money

#### **Example: 30% Margin of Safety**

Fair Value (Owner's Earnings method): $100/share Desired Margin: 30%

Target Buy Price = $100 × (1 - 0.30) = $100 × 0.70 = $70/share

**Interpretation:**

At $70, you're buying $100 of value for $70 (30% discount)
If valuation is slightly wrong ($90 instead of $100), you still break even
If business deteriorates 10%, you still break even
If you're right, you have $30 upside immediately

#### **Buffett's Practical Approach**

Rather than calculating precise intrinsic value, Buffett thinks in ranges:

I think this company is worth $80-120 (wide range due to uncertainty) Average = $100

I'll only buy if price is $70 or below (below the range) This gives me margin of safety even if I'm wrong about value

---

### **6.5 Buffett's Final Valuation Assessment**

#### **Combining Owner's Earnings and Perpetual Bond Yield**

**Owner's Earnings method** says: Fair value is $121.76/share (based on quality multiple)

**Perpetual Bond Yield method** says: Stock should yield 7.4%, requiring price around $45-50/share to be attractive

**These diverge significantly!**

**Why the discrepancy?**

The perpetual bond yield is more conservative. It's asking: "Should I buy this stock instead of bonds?"

The Owner's Earnings method is asking: "What is the business intrinsically worth?"

**Buffett reconciles by:**

1. **Using Owner's Earnings as baseline** ($121.76/share)
2. **Applying large margin of safety** (25-50%)
3. **Result: Buy price of $60-90/share** (substantial discount from $121.76)
4. **Validating with perpetual bond yield** - Does the earnings yield make sense?

#### **Final Valuation Output**

Company: Apple Inc. Current Price: $180/share

Valuation Method 1 (Owner's Earnings): Fair Value: $121.76/share Current Price: $180/share Discount/(Premium): (47.8%) [OVERVALUED by 48%]

Valuation Method 2 (Perpetual Bond Yield): Required Return: 7.4% Current Earnings Yield: 2.04% Assessment: OVERVALUED (earnings yield too low) Fair Price for 7.4% yield: ~$45-50/share

Buffett's Assessment: "Apple is an exceptional business with an unassailable moat. However, at current price of $180, it is expensive. I would consider buying below $85 (30% margin of safety). Current price does not provide adequate margin of safety. RECOMMENDATION: PASS (or wait for better price)"

Quality: ⭐⭐⭐⭐⭐ (5/5) Valuation: ❌ (Overvalued) Overall: 🟡 HOLD/WAIT

---

### **Summary: Buffett Valuation Techniques**

**Technique #1: Owner's Earnings Valuation**

Process:

Calculate 10-year normalized owner's earnings OE = Net Income + D&A - CapEx - Δ Working Capital

Assess business quality (moat, ROE, consistency, growth)

Assign quality multiple (15-35x based on quality)

Calculate fair value = OE × Quality Multiple

Result: Intrinsic value per share

**Technique #2: Perpetual Bond Yield Comparison**

Process:

Calculate earnings yield = Owner's Earnings / Market Cap

Determine required return = Risk-Free Rate + Risk Premium

Compare: Is EY > Required Return?

If yes = undervalued, if no = overvalued

Result: Validation and market-based reality check

**Buffett's Final Decision Framework:**

IF Owner's Earnings Fair Value significantly > Current Price AND Perpetual Bond Yield is attractive AND Margin of Safety is 25-50% THEN: BUY (compelling opportunity)

ELSE IF Fair Value ≈ Current Price AND Earnings Yield reasonable THEN: HOLD (if owned)

ELSE (Fair Value < Current Price OR Earnings Yield weak) THEN: PASS (wait for better price)

**Key principles:**

✅ Focus on cash earnings, not accounting earnings
✅ Apply 25-50% margin of safety
✅ Only invest when odds overwhelmingly favor you
✅ Patience is your advantage (willing to wait)
✅ Quality business at reasonable price beats mediocre at steep discount

---

## Section 7: Charlie Munger Valuation Techniques

**Method 5 of 5 - Weight in Consensus: 10%**

### **7.1 Charlie Munger's Philosophy & Investment Approach**

#### **Who is Charlie Munger?**

Charlie Munger is Warren Buffett's business partner and vice chairman of Berkshire Hathaway. At 100+ years old, he remains one of the sharpest investment minds in the world.

While Buffett focuses on mathematical DCF calculations, Munger emphasizes **business quality and competitive advantages**.

**Key quote:**

> **"The person that turns over the most rocks wins. We're looking for a few great businesses and competitive advantages. That's the game."**

#### **Munger vs Buffett: Key Differences**

| Aspect | Buffett | Munger |
|--------|---------|--------|
| **Valuation Approach** | Precise mathematical (DCF) | Business quality first |
| **Decision Process** | Calculate fair value, apply margin | Assess quality, then value |
| **Key Focus** | Cash flow projections | Durable competitive advantages |
| **Risk Management** | Large margin of safety (50%) | Only buys exceptional businesses |
| **Patience** | Will wait indefinitely | Will wait indefinitely |
| **Philosophy** | "Price is what you pay, value is what you get" | "Only invest in truly exceptional businesses" |

#### **Munger's Core Principles**

1. **Business quality first** - If business is mediocre, don't buy at any price
2. **Competitive advantages matter most** - Look for unassailable moats
3. **Management quality is essential** - Honest, capable capital allocators
4. **Durability and longevity** - Will the business thrive in 20 years?
5. **Simple, understandable businesses** - Avoid complexity and hidden risks
6. **Patience is a superpower** - Only act when odds are overwhelming
7. **Learn from mistakes** - Reverse engineering business failures teaches more than success

#### **Munger's Famous Quote**

> **"If you're not willing to think deeply about a stock, don't buy it."**

This encapsulates his approach: thorough business analysis before any financial calculations.

---

### **7.2 Munger's Business Quality Checklist (1-10 Scale)**

Before calculating any valuation, Munger asks: **"Is this an extraordinary business?"**

He assesses business quality across five dimensions, each scored 1-10:

#### **7.2.1 Moat Strength (Economic Competitive Advantage)**

**What is a moat?**

A moat is a durable competitive advantage that protects the business from competition.

**Scoring:**

Score Assessment Examples 9-10 Unassailable moat Coca-Cola (brand), Apple (ecosystem), Visa (network effects) 7-8 Strong moat Mastercard, Costco (switching costs), Starbucks (brand) 5-6 Identifiable moat Chipotle (brand), Adobe (switching costs) 3-4 Weak moat Regional bank, local retailer 1-2 No moat Commodity business, intense competition

**How to assess moat:**

**Brand moat:** Can customers easily switch to a competitor?
Coca-Cola: NO - iconic brand, emotional attachment, distribution Pepsi: Maybe - similar product, but Coke is preferred Store brand cola: YES - easily switches for 30% discount Score: Coca-Cola = 9, Pepsi = 6, Store brand = 2

**Switching costs:** How expensive/difficult to switch?
Microsoft Office: HIGH - thousands of documents, macros, training Google Docs: LOWER - easier to switch, but not seamless Apple ecosystem: VERY HIGH - switching from iPhone loses iCloud, iMessage, etc. Score: Microsoft = 8, Google = 5, Apple = 9

**Network effects:** Does the product get better as more people use it?
Visa: STRONG - more merchants accept it, more value Facebook: STRONG - more friends = more value Twitter: WEAK - works with few followers Score: Visa = 9, Facebook = 8, Twitter = 4

**Cost advantages:** Can you produce cheaper than competitors?
Amazon: STRONG - scale, logistics, data centers Walmart: STRONG - scale, supply chain efficiency Small retailer: WEAK - can't match buying power Score: Amazon = 9, Walmart = 8, Small retailer = 3

**Munger's guidance:** "If you can't identify the moat in 10 minutes, it doesn't exist."

#### **7.2.2 Business Durability (20-Year Horizon)**

**Question:** Will this business still be relevant and thriving 20 years from now?

**Scoring:**

Score Assessment Examples 9-10 Very durable, essential Coca-Cola, water utilities, business Microsoft (cloud services) 7-8 Durable, unlikely disruption Banks, insurance companies, Pharmaceutical companies 5-6 Reasonably durable, E-commerce, discount retail some disruption risk 3-4 Vulnerable to disruption Traditional retail, DVD rental 1-2 Likely disrupted Kodak (film), Blockbuster (video)

**Assessment questions:**

1. **Is the business threatened by disruption?**
   - Kodak made cameras. Digital disrupted photography.
   - Netflix made streaming. Physical rental disrupted.
   - Zoom makes video calls. Telecommunications disrupted.
   - Score Zoom low (9-10) for durability? Actually moderate (6-7) due to competition

2. **Does the business depend on a specific technology that might become obsolete?**
   - Coca-Cola: NO - beverages eternal
   - Intel: MAYBE - semicondutor technology evolving
   - Electric utilities: NO - electricity always needed
   - Score: Coca-Cola = 10, Intel = 6, Utilities = 9

3. **Are customer preferences stable or constantly changing?**
   - Fast fashion: CHANGING - trends shift constantly (score = 4)
   - Basic clothing: STABLE - jeans never go out of style (score = 8)
   - Restaurant: CHANGING - always new concepts (score = 5)

4. **Is regulation likely to hurt the business?**
   - Tobacco: YES - increasing regulation (score = 4)
   - Healthcare: MAYBE - regulation increases (score = 6)
   - Utilities: MAYBE - price controls, renewable requirements (score = 7)

#### **7.2.3 Management Quality (Honesty, Competence, Capital Allocation)**

**Munger says:** "We look for managers who are honest and able."

**Scoring:**

Score Management Quality Red Flags 9-10 Exceptional (Berkshire-like) Insider ownership >20%, track record, conservative accounting, shareholder-friendly 7-8 Strong and competent Insider ownership >10%, good track record, transparent communication 5-6 Adequate Some insider ownership, mixed track record 3-4 Weak No insider ownership, poor track record, accounting games 1-2 Untrustworthy Insider selling, accounting restatements, compensation abuse

**How to assess management:**

**1. Insider Ownership**

Does management have significant personal wealth in the company?

Buffett at Berkshire: Owns ~15% of company (personally wealthy from it) CEO with 50,000 shares and $1M salary: WEAK signal (small ownership) CEO with $100M in personal holdings: STRONG signal (skin in the game)

**Rule of thumb:** >10% insider ownership = skin in the game, alignment with shareholders

**2. Track Record**

Has management executed well historically?

Tim Cook at Apple: STRONG - 15+ years, proven execution Unknown new CEO: WEAK - unproven in this role CEO with history of failed initiatives: WEAK - poor track record

**3. Capital Allocation**

Does management deploy capital wisely?

Berkshire under Buffett: EXCEPTIONAL

Acquisitions create value
Share buybacks when undervalued
Cash on hand for opportunities
No wasteful spending
Technology company burning cash on bad acquisitions: WEAK

CEO buying companies at peak prices
Overpaying for talent
Destroying shareholder value

**4. Honest Communication**

Does management communicate transparently?

Buffett: EXCEPTIONAL

Annual letters acknowledge mistakes
Honest about challenges
Admits when wrong
Doesn't manage earnings
CEO that guides and beats every quarter: SUSPICIOUS

Too perfect suggests conservative guidance or accounting tricks
Real businesses have quarters that miss
Over-optimistic outlook signals dishonesty

**5. Compensation Structure**

Is executive pay reasonable and aligned with shareholders?

CEO compensation = $500K salary + stock options tied to 5-year performance → GOOD alignment (if stock undervalued relative to value creation)

CEO compensation = $10M salary + guaranteed bonus + golden parachute → BAD alignment (getting paid regardless of performance)

**Munger's guidance:** "If management is running the company for themselves rather than shareholders, run away."

#### **7.2.4 Earnings Consistency (Quality & Predictability)**

**Question:** Are earnings smooth, predictable, and backed by real cash?

**Scoring:**

Score Earnings Character Examples 9-10 Very consistent, high-quality Coca-Cola, Costco, Microsoft real cash earnings (smooth, predictable revenue) 7-8 Mostly consistent, good Banks (predictable), utilities quality, some variability
5-6 Uneven, some accounting Cyclical industrials, retailers games possible
3-4 Very volatile, quality Airlines, casinos, auto makers questionable
1-2 Erratic, possibly fraudulent Startups with no profits, companies with accounting issues

**How to assess earnings consistency:**

**1. Earnings volatility**

Calculate coefficient of variation (CV) of earnings over 10 years:

High-quality company (Coca-Cola): Earnings: $5.0B, $5.2B, $5.5B, $5.8B, $6.1B, $6.4B, $6.8B, $7.1B, $7.5B, $7.9B Trend: Smooth, consistent growth CV: ~8% (low volatility) Score: 9

Cyclical company (Airline): Earnings: $2B, -$0.5B, $3B, $1B, -$1B, $4B, -$0.5B, $2.5B, $1B, $3B Trend: Wildly erratic CV: >100% (very high volatility) Score: 2

**2. Earnings quality (Cash vs Accounting)**

Does reported earnings match operating cash flow?

Good quality: Reported earnings: $1B Operating cash flow: $0.95B (95% conversion to cash)

Red flag: Reported earnings: $1B Operating cash flow: $0.2B (only 20% conversion to cash) → Likely accounting tricks, not real earnings

**3. Earnings persistence**

Are high earnings from core business or one-time items?

Apple earnings: $12/share

iPhone revenue: consistent year-over-year
Services revenue: growing double-digit
One-time items: none significant Score: 9 (durable, core earnings)
Mining company earnings: $10/share

Commodity prices up 50% this year
One-time gain on mine sale: $3/share
Adjusted core earnings: $7/share Score: 4 (dependent on commodity cycle, inflated by one-time gains)

#### **7.2.5 Return on Capital (ROE/ROIC)**

**Question:** Does management generate high returns on shareholder capital?

**Scoring:**

Score ROE Level ROIC Level 9-10 >20% ROE >20% ROIC (exceptional) 7-8 15-20% ROE 15-20% ROIC (strong) 5-6 12-15% ROE 12-15% ROIC (good) 3-4 10-12% ROE 10-12% ROIC (average) 1-2 <10% ROE <10% ROIC (weak)

**How to calculate and interpret:**

**ROE (Return on Equity):**

ROE = Net Income / Average Shareholders' Equity × 100%

Example: Net Income: $10B Shareholders' Equity: $60B ROE = $10B / $60B = 16.7%

Score: 7-8 (strong ROE)

**ROIC (Return on Invested Capital):**

ROIC = NOPAT / Invested Capital × 100%

Where: NOPAT = Net Operating Profit After Tax Invested Capital = Total Debt + Shareholders' Equity - Cash

More complex, but better for businesses with significant debt.

**What high ROE tells you:**

Microsoft ROE: 35%+ (exceptional) → Earning $0.35 per dollar of shareholder equity → Can grow capital without destroying shareholder value

Bank ROE: 12-15% (good for banking) → Earning $0.12-0.15 per dollar → Similar to cost of capital, so not exceptional

Utility ROE: 10-12% (acceptable for utilities) → Regulated industry, so high ROE would be unfair → 10-12% is appropriate and adequate

**Munger's guidance:** "We want businesses that earn high returns on incremental capital. That's how you build shareholder value."

#### **7.2.6 Combined Quality Score**

**How to score overall business quality:**

Step 1: Score each dimension (1-10) Moat Strength: 8 Durability: 9 Management Quality: 8 Earnings Consistency: 9 Return on Capital: 7

Step 2: Sum total = 41 points out of 50

Step 3: Interpret 45-50: Exceptional business (rare) 40-44: Excellent business (Munger buys here) 35-39: Very good business (interesting) 30-34: Good business (needs right price) 25-29: Acceptable business (only at steep discount) <25: Poor quality (avoid entirely)

Interpretation for 41 points: "Excellent business with strong moat, durable competitive position, good management, consistent earnings, and solid returns on capital. This is investment-grade quality."

**Munger's guideline:** "If the quality score is <35, don't even bother calculating valuation. Move on to the next opportunity."

---

### **7.3 Munger Technique #1: Quality-Based Valuation**

#### **7.3.1 Quality Multiple Assignment (Linking Quality to Valuation Multiple)**

Once you've assessed business quality (0-50 scale), assign a valuation multiple based on that quality.

**Quality to Multiple Conversion:**

Quality Score Quality Multiple P/E Equivalent Interpretation 45-50 28-35x 22-28x P/E Exceptional quality 40-44 23-27x 18-22x P/E Excellent quality 35-39 18-22x 14-18x P/E Very good quality 30-34 14-17x 11-14x P/E Good quality 25-29 10-13x 8-11x P/E Acceptable quality <25 6-9x 5-8x P/E Poor quality (avoid)

**Why link quality to multiple?**

The higher quality the business, the more you should pay for each dollar of earnings.

Company A: Coca-Cola (quality 9/10) Earnings: $5.00/share Quality multiple: 30x Fair value: $150/share

You pay $30 for each $1 of earnings because:

Moat is unassailable
Business will grow for 50+ years
Management is excellent
Earnings very consistent
ROE is exceptional
Company B: Generic cola maker (quality 4/10) Earnings: $5.00/share Quality multiple: 10x Fair value: $50/share

You pay only $10 for each $1 of earnings because:

No sustainable moat
Could be disrupted anytime
Management average
Earnings erratic
ROE is weak
Same $5 earnings but Coca-Cola worth 3x more!

#### **7.3.2 Calculating Quality-Based Fair Value**

**Step 1: Complete business quality assessment**

(See Section 7.2 above - score each dimension 1-10)

**Step 2: Sum quality dimensions**

Moat: 9 Durability: 9 Management: 8 Earnings Consistency: 9 ROE: 8 Total: 43 (Excellent quality)

**Step 3: Assign quality multiple from table**

Quality Score 43 → Quality Multiple = 25x (Between 40-44 range → 23-27x range, use 25x)

**Step 4: Get normalized earnings**

Use 10-year average to smooth out cyclical fluctuations:

10-year average net income: $72.5B Shares outstanding: 2.4B Normalized EPS: $72.5B / 2.4B = $30.21/share

**Step 5: Calculate fair value**

Fair Value = Quality Multiple × Normalized EPS = 25x × $30.21 = $755.25/share

**Step 6: Compare to current price**

Current Price: $420.50 Fair Value: $755.25 Upside/(Downside): 79.6% upside Status: UNDERVALUED

#### **7.3.3 Real Example: Apple Inc. (Quality-Based Valuation)**

**Step 1: Apple Quality Assessment**

Moat Strength: 10/10

Brand is globally iconic
Ecosystem lock-in (iPhone, iPad, Mac, Watch, AirPods)
Customer loyalty is exceptional
Switching costs are high (Photos, Messages, iCloud, etc.) Score: 10 (unassailable)
Durability: 9/10

Consumer electronics will exist forever
Likely to be dominant 20+ years out
Some risk from new technologies (AR/VR, AI)
But brand so strong it could adapt Score: 9 (very durable)
Management Quality: 9/10

Tim Cook: 15+ years proven track record
Execution excellence on products/services
Aggressive buybacks when stock undervalued
Transparent communication
Some concerns: over-reliance on iPhone (70% of revenue) Score: 9 (exceptional)
Earnings Consistency: 9/10

iPhone revenue stable and predictable
Services revenue growing consistently
Gross margins very stable (38-42%)
Few one-time items or accounting gimmicks Score: 9 (very consistent)
Return on Capital: 9/10

ROE: 100%+ (returns earnings to shareholders via buybacks)
ROIC: >30% (cash generation exceptional)
Returns exceed cost of capital by 20%+ Score: 9 (exceptional)
TOTAL QUALITY SCORE: 46/50 (EXCEPTIONAL)

**Step 2: Assign quality multiple**

Quality 46 → Multiple 30-33x (top tier exceptional quality) Use: 31x (middle of range)

**Step 3: Apple's normalized earnings**

10-year average net income: $60.6B Shares outstanding (diluted): 16.5B Normalized EPS: $60.6B / 16.5B = $3.67/share

Wait - that seems low. Let me recalculate with recent data: Latest 12-month net income: $99.8B Shares outstanding: 15.6B TTM EPS: $99.8B / 15.6B = $6.40/share

More realistic: Use $6.40/share

**Step 4: Calculate fair value**

Fair Value = 31x × $6.40 = $198.40/share

**Step 5: Compare to current price**

Current Price: $180/share Fair Value: $198.40/share Upside: 10.2% Status: SLIGHTLY UNDERVALUED

Munger Assessment: "Apple is an exceptional business with quality score of 46/50. At current price, it offers modest upside but no compelling margin of safety. I'd prefer waiting for a better entry price (closer to $140-150)."

---

### **7.4 Munger Technique #2: The Five Stock Test**

#### **7.4.1 Philosophy: The Ultimate Quality Filter**

Munger has famously said:

> **"If you could only own 5 stocks for the rest of your life, would you own this one?"**

This single question forces you to think like a true owner, not a trader.

**Why this test matters:**

It separates truly exceptional businesses from merely good ones. You only make this list if:
1. Business quality is unquestionable
2. You have supreme confidence in durability
3. Management is trustworthy
4. Valuation is fair (not necessarily cheap)

#### **7.4.2 The Five Stock Test Decision Framework**

**Question 1: Is this business extraordinary?**

IF NO → REJECT immediately "This is a mediocre business. Even at $5, it's not worth owning forever. Move on to look for better opportunities."

IF YES → Continue to Question 2

Extraordinary means:
- Exceptional moat (9-10 on moat scale)
- Very durable (8-10 on durability scale)
- Excellent management (8-10 on management scale)
- Combined quality score 42+

**Question 2: Am I willing to own this forever?**

Even if stock price never moved, would I be happy owning this company for 50 years, through multiple economic cycles?

IF NO → REJECT "Too much risk of disruption. Don't trust long-term durability."

IF YES → Continue to Question 3

This filters out:
- Commodity businesses
- Businesses in fast-changing industries
- Companies dependent on single product
- Businesses threatened by regulation

**Question 3: Is valuation reasonable for permanent holding?**

Fair Value (quality multiple method): $200/share Current Price: $180/share

Is fair value realistic and based on conservative assumptions? Is current price below fair value? Is there some margin of safety (15-25%)?

IF current price is MUCH higher than fair value (>30% premium) AND you can't justify the premium → WAIT for better price → "The valuation doesn't work for a permanent holding"

IF fair value seems reasonable and price provides some margin → CONSIDER BUYING

#### **7.4.3 The Decision Tree**


                Is business extraordinary?
                      /         \
                    NO           YES
                    |             |
                  REJECT      Will you own forever?
                                /        \
                              NO         YES
                              |           |
                            REJECT    Is valuation reasonable?
                                        /           \
                                      NO             YES
                                      |               |
                                    WAIT           BUY or
                                 for better     ACCUMULATE
                                   price


#### **7.4.4 Real Examples: Five Stock Test**

**Example 1: Coca-Cola**

Question 1: Is this business extraordinary? Moat: 10/10 (brand unassailable) Durability: 10/10 (beverages forever) Management: 9/10 (excellent) Quality score: 44/50 (exceptional) ANSWER: YES ✅

Question 2: Would I own it forever?

Coke will exist in 50 years? Absolutely
Still dominant in beverages? Probably
Risk of disruption? Low
Risk of business model changing? Low ANSWER: YES ✅
Question 3: Is valuation reasonable? Fair Value (quality multiple): $95/share Current Price: $61.50/share Margin of Safety: 35% (excellent)

Is $61.50 a fair entry for 50-year holding? YES Is there margin of safety? YES ANSWER: YES ✅

FINAL DECISION: ✅ YES, WOULD INCLUDE IN FIVE STOCK PORTFOLIO "Coca-Cola passes all tests. It's a world-class business at a fair price. I would be happy to own this forever."

**Example 2: Tesla (Electric Vehicles)**

Question 1: Is this business extraordinary? Moat: 6/10 (brand strong, but competition increasing) Durability: 6/10 (EV market likely permanent, but Tesla dominance uncertain) Management: 7/10 (visionary, but unconventional) Quality score: 35/50 (very good, not exceptional)

Note: Quality is good, but not "extraordinary" ANSWER: NO (borderline) ❌

FINAL DECISION: ❌ NO, WOULD NOT INCLUDE IN FIVE STOCK PORTFOLIO "Tesla is a good business with strong growth, but I'm not confident it's 'extraordinary' enough for a 50-year permanent holding. EV competition is intensifying. I'd need to see more durable moat. Pass."

**Example 3: Regional Bank**

Question 1: Is this business extraordinary? Moat: 4/10 (weak - lots of competition, regulated) Durability: 5/10 (banking eternal, but model threatened by fintech) Management: 5/10 (average - many capable managers) Quality score: 28/50 (acceptable, not good) ANSWER: NO ❌

FINAL DECISION: ❌ NO, WOULD NOT INCLUDE IN FIVE STOCK PORTFOLIO "This is a competent bank, but not extraordinary. There are better businesses to own forever. Pass."

**Example 4: Microsoft**

Question 1: Is this business extraordinary? Moat: 9/10 (enterprise software switching costs, Office ecosystem) Durability: 9/10 (software eternal, cloud essential, AI positioning) Management: 9/10 (Nadella excellent, capital allocation smart) Quality score: 43/50 (excellent) ANSWER: YES ✅

Question 2: Would I own it forever?

Software exists forever? Yes
Microsoft likely still dominant? Probably (80% market share Office)
Cloud (Azure) becoming essential? Yes, growing 25%+ annually
Risk of disruption? Low-moderate (AI could shift, but MSFT leading there too) ANSWER: YES ✅
Question 3: Is valuation reasonable? Fair Value (quality multiple 27x): $173/share Current Price: $420/share Problem: Current price is 2.4x fair value Margin of safety: NEGATIVE (massive premium)

Is $420 a fair entry for 50-year holding? NO Do I have margin of safety? NO ANSWER: NO ❌

FINAL DECISION: ✅ YES to business quality, ❌ NO to current valuation "Microsoft is an extraordinary business I'd own forever - it passes questions 1 and 2 easily. However, at $420/share it's priced for perfection. I would WAIT for a better price (around $280-320) before buying for a permanent holding. Current price is too expensive even for an exceptional business."

#### **7.4.5 Why the Five Stock Test Works**

This test is powerful because:

1. **Forces ownership mentality** - You're buying a business, not trading a stock
2. **Filters for quality** - Only truly exceptional businesses make the cut
3. **Recognizes timing** - A great business at the wrong price isn't a great investment
4. **Builds patience** - You'll hold through market volatility
5. **Aligns incentives** - You profit only if business succeeds long-term

**Munger's wisdom:** "If you understand the business deeply and have high conviction, you don't need a 50% margin of safety. A 20-25% margin is sufficient. But the business quality must be exceptional."

---

### **7.5 Munger's Quality-Based Decision Matrix**

**Combining all Munger techniques:**


                Quality Score
       Exceptional    Good    Acceptable    Poor
       (45-50)      (35-44)   (25-34)     (<25)
Valuation: Undervalued BUY! BUY MAYBE NO Fair BUY CONSIDER PASS NO Expensive WAIT PASS PASS NO Very Expensive PASS PASS PASS NO

**Using this matrix:**

Company A: Quality 48, Price at fair value Matrix says: BUY Munger thinks: "Exceptional business at fair price. Buy now."

Company B: Quality 38, Price 25% below fair value Matrix says: BUY Munger thinks: "Good business at attractive price. Worth buying."

Company C: Quality 43, Price at 2x fair value Matrix says: WAIT Munger thinks: "Great business but outrageously expensive. Wait for decline. If I can't find better opportunities, this is 2nd choice."

Company D: Quality 32, Price at fair value Matrix says: PASS Munger thinks: "Acceptable quality at fair price. Not good enough. Plenty of better opportunities exist."

---

### **7.6 Munger Valuation Summary & Output**

#### **Final Munger Valuation Assessment**

**For the same company, Munger's output would look like:**

COMPANY: Microsoft Corporation

BUSINESS QUALITY ASSESSMENT:

Moat Strength: 9/10 → Enterprise software switching costs → Office ecosystem dominance (80% market share) → Cloud infrastructure lock-in (Azure)

Durability: 9/10 → Software industry permanent → Cloud computing essential forever → AI leadership being built → Some risk from competition (Google, Amazon)

Management Quality: 9/10 → Satya Nadella proven track record → Smart capital allocation → Aggressive buybacks when stock undervalued → Transparent with investors

Earnings Consistency: 9/10 → Software revenue very recurring → Margins stable at 35%+ → Few one-time items → FCF generation excellent

Return on Capital: 8/10 → ROE: 35%+ (excellent) → ROIC: >25% (excellent) → Returns greatly exceed cost of capital

TOTAL QUALITY SCORE: 44/50 (EXCELLENT)

QUALITY-BASED VALUATION:

Quality Multiple (44 = 23-27x range): 25x Normalized EPS (trailing 12M): $12.15 Fair Value = 25x × $12.15 = $303.75/share

FIVE STOCK TEST:

Question 1 - Extraordinary business? ✅ YES (Quality 44/50, excellent moat and durability)

Question 2 - Own forever? ✅ YES (Software/cloud essential, Microsoft likely dominant)

Question 3 - Reasonable valuation? ❌ NO (Current $420 is 1.38x fair value - expensive)

FIVE STOCK TEST RESULT: Qualified but overvalued

FINAL MUNGER ASSESSMENT:

"Microsoft is an excellent business with a durable competitive advantage in enterprise software and cloud services. Management is exceptional, earnings are consistent, and returns on capital are excellent. I would be delighted to own this company forever.

However, at $420/share, the stock is expensive. Fair value is approximately $304/share based on quality multiples. The current price provides no margin of safety for a permanent holding.

I would be willing to buy at $240-270/share (20-25% margin below fair value). Until then, I will PASS and wait for a better entry point. There are other equally good businesses at better prices right now."

RECOMMENDATION: 🟡 PASS / WAIT for better price RATING: Quality A+ / Valuation C (too expensive) DECISION: Hold if owned, do not buy at current price

---

### **Summary: Munger Valuation Techniques**

**Technique #1: Quality-Based Business Assessment**

Process:

Score business across 5 dimensions (1-10 each)

Moat Strength
Business Durability
Management Quality
Earnings Consistency
Return on Capital
Sum quality dimensions (total /50)

Assign quality multiple based on score 45-50 → 28-35x 40-44 → 23-27x 35-39 → 18-22x etc.

Calculate fair value = Multiple × Normalized EPS

Result: Quality-based intrinsic value

**Technique #2: Five Stock Test**

Process:

Is business extraordinary? (Quality score 42+?)
Would you own it forever? (Durability & moat)
Is valuation reasonable? (Margin of safety 15-25%)
Result: YES to all three → Consider buying NO to any → Pass or wait

Decision framework: Quality first, valuation second

**Munger's Philosophy:**

✅ Only invest in truly exceptional businesses
✅ Business quality matters more than valuation
✅ Patience provides margin of safety
✅ A great business at a fair price beats mediocre at steep discount
✅ If you can't understand the business deeply, don't buy it

**Key insight:** Munger's approach is more about business judgment than mathematical precision. He'd rather own a 50-year business at 20x earnings than a mediocre business at 8x earnings.

---

## Section 8: Consensus Valuation & Final Assessment

### 8.1 Consensus Valuation Formula

After calculating fair value using all five methods, combine them using weighted average:

**Consensus Fair Value = (DCF × 40%) + (Multiples × 20%) + (Buffett × 15%) + (DDM × 15%) + (Munger × 10%)**

**Why These Weights?**
- **DCF (40%):** Most theoretically sound, captures long-term value
- **Multiples (20%):** Market-based reality check
- **Buffett (15%):** Owner's earnings perspective
- **DDM (15%):** Cash returned to shareholders
- **Munger (10%):** Quality-based valuation as final filter

### 8.2 Confidence Level Determination

**Calculate the spread of all five methods:**

Spread % = ((Max Method Value - Min Method Value) / Min Method Value) × 100%

**Confidence Level Assignment:**
- **HIGH Confidence:** Spread ≤ 20% (methods agree closely)
- **MEDIUM Confidence:** Spread 20-50% (methods reasonably aligned)
- **LOW Confidence:** Spread > 50% (methods diverge significantly)

**Interpretation:**
- High spread suggests uncertain valuation; caution warranted
- Methods agreeing closely increases confidence in fair value estimate
- Consider business quality and competitive position when confidence is low

### 8.3 Valuation Grade Assignment

Assign a grade (A+ to F) based on valuation certainty and business quality:

| Grade | Criteria | Interpretation |
|-------|----------|-----------------|
| **A+** | Consensus high, Spread ≤ 10%, Strong moat, ROE > 20% | Exceptional clarity on valuation |
| **A** | Consensus high, Spread ≤ 20%, Strong quality metrics | Clear valuation, high confidence |
| **B+** | Consensus medium, Spread 20-35%, Good quality | Good confidence, minor uncertainty |
| **B** | Consensus medium, Spread 35-50%, Adequate quality | Moderate confidence |
| **C+** | Consensus low, Spread 50-75%, Adequate quality | Low confidence, proceed cautiously |
| **C** | Wide divergence, Spread > 75%, Weak quality | Significant uncertainty |
| **F** | Cannot value (negative earnings, insufficient data) | Insufficient data for valuation |

### 8.4 Margin of Safety Calculation

**Margin of Safety % = ((Consensus Fair Value - Current Price) / Consensus Fair Value) × 100%**

**Buffett's Preferred Margins:**
- **Excellent:** ≥ 50% (stock trading at ≤ 50% of fair value)
- **Good:** 25-50% (attractive risk/reward)
- **Acceptable:** 15-25% (reasonable cushion)
- **Caution:** < 15% (limited margin for error)
- **Avoid:** Negative (stock overvalued)

### 8.5 Investment Recommendation Assignment

Based on Margin of Safety and Valuation Grade:

| MOS | Grade A+ | Grade A | Grade B+ | Grade B | Grade C+ | Grade C |
|-----|----------|---------|----------|---------|----------|---------|
| **≥50%** | **BUY** | **BUY** | **BUY** | CONSIDER | CONSIDER | PASS |
| **25-50%** | **BUY** | **BUY** | CONSIDER | CONSIDER | PASS | PASS |
| **15-25%** | **BUY** | CONSIDER | CONSIDER | PASS | PASS | PASS |
| **<15%** | CONSIDER | CONSIDER | PASS | PASS | PASS | AVOID |
| **Negative** | AVOID | AVOID | AVOID | AVOID | AVOID | AVOID |

**Recommendation Definitions:**
- **BUY:** Strong opportunity with favorable risk/reward and clear valuation
- **CONSIDER:** Interesting opportunity worth deeper analysis; reasonable valuation
- **PASS:** Fair price or minor concerns; wait for better entry
- **AVOID:** Overvalued or significant concerns; do not purchase

### 8.6 Output Summary Format

The consensus valuation output includes:

| Metric | Definition |
|--------|-----------|
| **Consensus Fair Value** | Weighted average of all five methods |
| **Valuation Spread** | Range from lowest to highest method |
| **Confidence Level** | HIGH / MEDIUM / LOW |
| **Valuation Grade** | A+ to F based on clarity and quality |
| **Current Price** | Market price at analysis date |
| **Margin of Safety** | % discount to fair value |
| **Recommendation** | BUY / CONSIDER / PASS / AVOID |

---

## Section 9: Output Definitions

### 9.1 Stage 6 Output Package

Each company evaluated in Stage 6 produces the following outputs:

**Primary Outputs:**
1. Consensus Fair Value (per share)
2. Fair Value from each method (DCF, DDM, Multiples, Buffett, Munger)
3. Margin of Safety (%)
4. Confidence Level (HIGH/MEDIUM/LOW)
5. Valuation Grade (A+ to F)
6. Investment Recommendation (BUY/CONSIDER/PASS/AVOID)
7. Valuation Spread (%)

**Supporting Outputs:**
- Sensitivity analysis (WACC, growth rate, multiples)
- Calculation details for each method
- Assumptions used in projections
- Risk factors identified

### 9.2 Data Fields for Database Storage

The following fields are stored in MySQL database from Stage 6:

**Company Identifiers:**
- Ticker, Company Name, Industry, Market Cap

**Fair Value Estimates:**
- dcf_fair_value, ddm_fair_value, multiples_fair_value
- buffett_fair_value, munger_fair_value
- consensus_fair_value

**Valuation Assessment:**
- valuation_spread_percent, confidence_level
- valuation_grade, current_price
- margin_of_safety_percent

**Recommendation:**
- investment_recommendation, analysis_date
- analyst_notes, sensitivity_summary

### 9.3 Output Format Notes

**Format Details:**
- All prices in USD per share
- Percentages shown as decimals (0.25 = 25%)
- Dates in ISO format (YYYY-MM-DD)
- All calculations rounded to 2 decimal places for currency

### 9.4 Output Quality Checks

Before data is stored, validate:
- ✓ All five methods produced values
- ✓ Consensus fair value is reasonable (not extreme outlier)
- ✓ Margin of safety calculation is correct
- ✓ Confidence level matches spread % range
- ✓ Recommendation aligns with MOS and grade
- ✓ No missing required fields

---

## Section 10: Format Notes for Stage 7 Integration

### 10.1 Note: Output Format Details

Email report formats, GitHub summaries, and CSV export specifications are defined in Stage 7 (Results Reporting Requirements). This document focuses on the valuation calculations for Stage 6.

For report format specifications, see: [03-RESULTS-REPORTING-REQUIREMENTS.md](03-RESULTS-REPORTING-REQUIREMENTS.md)

---

##Section 15: Data Flow Integration
###**15.1 Stage 5.4 in Complete Pipeline**

┌─────────────────────────────────────────────────────────────┐
│                    INPUT (from Stage 3.3)                    │
├─────────────────────────────────────────────────────────────┤
│  All companies with composite score ≥75 (typically 45-60)   │
│  All 40+ financial metrics calculated                        │
│  Quality scores (ROE, margins, cash flow, etc.)             │
│  Current stock price and market data                         │
│  10-year historical financial statements                     │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│              STAGE 5.4: VALUATION ANALYSIS                   │
├─────────────────────────────────────────────────────────────┤
│ ✅ Calculate DCF valuation (10-year projection)              │
│ ✅ Calculate DDM valuation (dividend/buyback analysis)       │
│ ✅ Calculate Multiples valuation (peer comparison)           │
│ ✅ Apply Buffett techniques (Owner's Earnings + yield)       │
│ ✅ Apply Munger techniques (Quality + Five Stock Test)       │
│ ✅ Calculate consensus fair value (weighted average)         │
│ ✅ Analyze margin of safety (multiple thresholds)            │
│ ✅ Assign valuation grade (A+ to F)                          │
│ ✅ Create investment recommendations                         │
│ ✅ Group companies by opportunity (5 groups)                 │
│ ✅ Store results in database                                 │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│              OUTPUT (to Stage 6 Processing)                  │
├─────────────────────────────────────────────────────────────┤
│ • Intrinsic values (5 methods)                              │
│ • Margin of safety percentages                              │
│ • Investment recommendations (Buffett, Munger, Joint)       │
│ • Valuation grades and grouping categories                  │
│ • Investment theses                                         │
│ • Database records with audit trail                         │
│ • Sensitivity analysis ranges                               │
│ • All supporting calculations                               │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│         STAGE 6: RESULTS PROCESSING                          │
├─────────────────────────────────────────────────────────────┤
│ • Organize by grouping category (Groups 1-5)                │
│ • Identify new/maintained/removed vs prior run              │
│ • Calculate aggregate statistics                            │
│ • Prepare formatted reports                                 │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│         STAGE 7: NOTIFICATIONS & STORAGE                     │
├─────────────────────────────────────────────────────────────┤
│ • Email report to investor (detailed)                       │
│ • GitHub summary page (quick view)                          │
│ • CSV export (spreadsheet data)                             │
│ • Database storage (complete audit trail)                   │
└─────────────────────────────────────────────────────────────┘
15.2 Data Requirements for Stage 5.4
From Stage 3.3 (Financial Analysis):

Required Fields:
  ✅ Company ID & Ticker
  ✅ Company Name & Sector
  ✅ Composite Financial Score (0-100)
  ✅ 10-year financial statements (SEC EDGAR)
    - Income Statement (Net Income, Tax Rate)
    - Cash Flow Statement (OCF, CapEx, D&A, FCF)
    - Balance Sheet (Equity, Debt, Cash, Book Value)
  ✅ Current market data (stock price, shares outstanding)
  ✅ Quality scores (ROE, margins, cash flow consistency)
  ✅ Management quality assessment
  ✅ Moat strength evaluation

From External Sources:
  ✅ Current stock price (daily update from Yahoo Finance)
  ✅ Risk-free rate (10-year Treasury from Fed FRED)
  ✅ Peer company data (multiples from Yahoo Finance)
  ✅ Dividend history (10 years from company filings)
  ✅ Share buyback history (from SEC filings)
15.3 Quality Checks Before Proceeding
Before running Stage 5.4 valuation, verify:

Data Completeness:
  ✅ 10 years of financial data available (or note if less)
  ✅ Current stock price available and recent
  ✅ All required balance sheet items present
  ✅ FCF can be calculated (OCF and CapEx data)

Financial Reasonableness:
  ✅ Net Income matches operating cash flow (within 20%)
  ✅ FCF is positive or has clear path to positive
  ✅ No accounting red flags (restated earnings, etc.)
  ✅ Debt levels reasonable relative to industry

Market Data:
  ✅ Stock price reasonable (not stale data)
  ✅ Shares outstanding reflects current dilution
  ✅ Market cap matches stock price × shares
  ✅ Peer companies are truly comparable

Quality Assessment:
  ✅ Moat strength scored based on real competitive advantages
  ✅ Management quality assessment based on track record
  ✅ Durability assessment realistic (not overly optimistic)
  ✅ Earnings consistency score reflects actual volatility
Section 16: Sensitivity Analysis
16.1 Purpose of Sensitivity Analysis
Sensitivity analysis tests how valuation changes when key assumptions vary:

####**Why it matters:**

DCF says fair value is $300/share
But is that true if:
  - WACC is 9% instead of 10%? ($340 instead of $300)
  - Terminal growth is 3% instead of 2.5%? ($330 instead of $300)
  - Both assumptions wrong? (could be $400 or $220)

Sensitivity analysis shows the range and which assumptions matter most.
16.2 DCF Discount Rate Sensitivity
Impact of WACC variations on DCF fair value:

WACC Range Analysis (holding terminal growth constant at 2.5%):

WACC %    Fair Value/Share    vs Base Case    Interpretation
8.0%      $558.63             +18.6%          Very optimistic (low risk)
8.5%      $514.40             +9.1%           Optimistic (lower risk)
9.0%      $484.65             +2.8%           Slightly optimistic
9.5%      $461.41             -1.8%           Slightly pessimistic
10.0%     $442.41             -6.1%           Base case (moderate)
10.5%     $426.44             -9.4%           Pessimistic (higher risk)
11.0%     $412.67             -12.1%          Very pessimistic
11.5%     $400.70             -14.8%          Conservative (much higher risk)
12.0%     $390.22             -16.9%          Very conservative
####**How to interpret:**

If you believe WACC is:
  - 8-9%: Fair value in $485-560 range (optimistic scenario)
  - 9-11%: Fair value in $410-485 range (realistic range)
  - 11-12%: Fair value in $390-410 range (conservative scenario)

Your best estimate (10%): Fair value = $442/share
Your confidence range: $390-490/share (±11%)
16.3 Terminal Growth Rate Sensitivity
Impact of terminal growth variations (holding WACC constant at 10%):

Terminal Growth %    Fair Value/Share    vs Base Case
1.5%                 $435.07             -1.6%
2.0%                 $445.68             +0.8%
2.3%                 $454.97             +2.9%
2.5%                 $463.30             +4.8%  (Base case)
2.7%                 $474.30             +7.1%
3.0%                 $486.48             +10.0%
3.5%                 $514.40             +16.3%

####**How to interpret:**

Using conservative 2.5% perpetual growth: $463/share
If market expects 3.0% (reasonable): $487/share (+5%)
If market expects 3.5% (too optimistic): $514/share (+11%)

####**Conclusion: Terminal growth is less sensitive than WACC**
(±1% growth rate = ±5% valuation impact)
(±1% WACC = ±10% valuation impact)
16.4 2D Sensitivity Matrix (WACC × Terminal Growth)
Shows impact when BOTH assumptions vary:

Terminal Growth Rate
           1.5%     2.0%     2.5%     3.0%     3.5%
WACC
8.0%      $601.45  $625.50  $651.25  $679.15  $709.80
8.5%      $557.42  $579.48  $603.35  $629.48  $658.15
9.0%      $518.75  $539.65  $562.15  $586.48  $612.88
9.5%      $484.30  $504.05  $525.22  $548.48  $573.92
10.0%     $453.41  $471.82  $491.63  $514.40  $539.68  ← Base case
10.5%     $426.40  $442.95  $460.97  $482.21  $506.51
11.0%     $401.72  $416.73  $433.43  $452.90  $475.19
11.5%     $379.42  $392.99  $408.24  $426.21  $446.54
12.0%     $359.22  $371.60  $385.57  $402.27  $421.27

####**Key findings:**

Best case (8% WACC, 3.5% growth): $710/share
Base case (10% WACC, 2.5% growth): $492/share
Worst case (12% WACC, 1.5% growth): $359/share

Range: $359 to $710 (98% spread)
Most likely range: $450-530 (±20% from base)

####**Conclusion: Fair value is roughly $490/share with $400-580 confidence range**
16.5 Quality Multiple Sensitivity (Munger Method)
Impact of quality multiple variations on Munger fair value:

Quality Assessment    Quality Multiple    Fair Value (× $12.15 EPS)
High estimate        27x                  $328.05
Base estimate        25x                  $303.75
Conservative estimate 23x                 $279.45

Buffett equivalent:
High estimate (28x)                       $340.20
Base estimate (25x)                       $303.75
Conservative (22x)                        $267.30
How to interpret:

Quality score of 43/50 suggests multiple of 25x
But confidence range: 23-27x quality multiple
This gives fair value of $279-328/share
Central estimate: $304/share
Range: $279-328 (±8%)

####**Conclusion: Munger valuation more stable than DCF**
(Depends more on quality assessment than growth assumptions)
16.6 Interpretation & Use in Decision-Making
How to use sensitivity analysis:

Step 1: Calculate base case valuation
  Result: Fair value = $300/share

Step 2: Run sensitivity analysis
  WACC sensitivity: $270-350/share range
  Growth sensitivity: $280-320/share range
  Multiple sensitivity: $275-330/share range

Step 3: Identify realistic range
  Conservative estimate: $270/share
  Base estimate: $300/share
  Optimistic estimate: $330/share
  Working range: $270-330/share

Step 4: Compare to current price
  Current price: $250/share
  Assessment: Trading below even conservative estimate
  Conclusion: Stock is undervalued with high confidence

Step 5: Decide on action
  Margin of safety = ($300 - $250) / $300 = 17%
  Even if valuation pessimistic, still undervalued
  Recommendation: BUY at current price
16.7 Sensitivity Analysis Output Example
For final report, include summary:

###**SENSITIVITY ANALYSIS SUMMARY**

BASE CASE FAIR VALUE: $304/share

Discount Rate Sensitivity (WACC):
  Conservative (12% WACC): $275/share
  Base Case (10% WACC): $304/share
  Optimistic (8% WACC): $348/share
  Range: $275-348 (±13%)

Terminal Growth Sensitivity:
  Conservative (1.5% growth): $289/share
  Base Case (2.5% growth): $304/share
  Optimistic (3.5% growth): $322/share
  Range: $289-322 (±6%)

Quality Multiple Sensitivity (Munger):
  Conservative (23x): $280/share
  Base Case (25x): $304/share
  Optimistic (27x): $329/share
  Range: $280-329 (±8%)

CONFIDENCE ASSESSMENT:
  Most likely range: $280-330/share
  Central estimate: $304/share
  Conservative case: $275/share

Current Price: $250/share
Margin vs conservative: 9%
Margin vs base: 18%
Margin vs optimistic: 24%

CONCLUSION: Stock undervalued across all reasonable scenarios.
Recommend BUY with confidence.
Summary: Sensitivity Analysis
Key takeaways:

✅ Valuation is uncertain, not fixed ✅ Sensitivity shows realistic range ✅ Most reasonable outcomes converge ($280-330) ✅ Outliers (very optimistic/pessimistic) excluded ✅ Margin of safety should cover worst reasonable case ✅ Stock at $250 attractive vs even conservative estimate

Use sensitivity to:

Test valuation robustness
Identify key assumptions
Build confidence in recommendation
Communicate valuation uncertainty
Set realistic expectations

---




## Section 11: Real-World Examples

**Complete walkthroughs of three company types demonstrating all valuation methods**

### **11.1 Complete Example #1: Technology Company (Microsoft - High Growth, No Dividend)**

#### **Company Overview**

Ticker: MSFT Company: Microsoft Corporation Sector: Technology/Software/Cloud Current Stock Price: $420.50 Shares Outstanding: 2.4B Market Cap: $3.14 trillion Date of Analysis: 2026-03-14

#### **Step 1: Financial Data Gathering**

####**10-Year Historical Financials:**

Year Net Income D&A CapEx FCF EPS (B)(B) (B)(B) ($) 2016 $20.5 $5.2 $4.8 $20.9 $2.40 2017 $21.2 $5.5 $5.0 $21.7 $2.51 2018 $26.8 $5.8 $5.3 $27.3 $3.18 2019 $39.2 $6.2 $6.5 $38.9 $4.68 2020 $44.3 $6.8 $7.2 $43.9 $5.49 2021 $69.9 $7.5 $8.1 $69.3 $8.84 2022 $72.8 $8.2 $9.0 $72.0 $9.43 2023 $88.2 $8.9 $10.2 $86.9 $11.52 2024 $99.8 $9.5 $11.5 $97.8 $12.97 2025 $109.5 $10.2 $12.8 $106.9 $14.35

#### **Step 2: DCF Valuation**

####**Historical Analysis:**

FCF CAGR (2016-2025): = ($106.9B / $20.9B)^(1/9) - 1 = 18.8% annual growth

Interpretation: Exceptional growth rate

Reflects cloud (Azure) strong adoption
Reflects AI momentum
Reflects enterprise software demand

####**Projection (10-year forward):**

Declining growth model (growth decays toward 2.5% terminal):

Year Growth % Projected FCF Discount Factor PV 1 18.0% $126.2B 0.9099 $114.8B 2 15.0% $145.1B 0.8279 $120.2B 3 12.0% $162.5B 0.7539 $122.5B 4 10.0% $178.7B 0.6855 $122.6B 5 8.0% $193.0B 0.6237 $120.4B 6 6.0% $204.6B 0.5674 $116.0B 7 4.5% $213.8B 0.5161 $110.4B 8 3.5% $221.2B 0.4697 $103.9B 9 3.0% $227.8B 0.4274 $97.4B 10 2.5% $233.4B 0.3888 $90.8B

Sum of PV (Years 1-10) = $1,118.8B

####**Terminal Value:**

Terminal Value = FCF₁₀ × (1+g) / (WACC - g) = $233.4B × 1.025 / (0.10 - 0.025) = $239.2B / 0.075 = $3,189.3B

PV(Terminal Value) = $3,189.3B / (1.10)^10 = $3,189.3B / 2.594 = $1,230.0B

####**DCF Fair Value:**

Enterprise Value = $1,118.8B + $1,230.0B = $2,348.8B

Less: Net Debt Total Debt: $45B Cash: $82B Net Cash: -$37B (negative = net cash position)

Equity Value = $2,348.8B - (-$37B) = $2,385.8B

Per-Share = $2,385.8B / 2.4B shares = $994.08/share

DCF FAIR VALUE: $994.08/share Current Price: $420.50 Upside: 136% (extremely undervalued by DCF)

####**DCF Sensitivity:**

WACC × Terminal Growth Sensitivity Matrix:

####**Terminal Growth:**
WACC 2.0% 2.5% 3.0% 3.5% 8% $1,245 $1,385 $1,568 $1,815 9% $1,025 $1,127 $1,260 $1,437 10% $855 $994 $1,115 $1,261 ← Base case 11% $724 $835 $932 $1,063 12% $618 $709 $790 $901

Range: $618-1,815/share Most likely: $835-1,115/share

#### **Step 3: DDM Valuation (Buyback-Adjusted)**

####**Dividend Analysis:**

Cash Dividends (TTM): $12.2B Share Buybacks (TTM): $27.5B Total Return: $39.7B

Weighted Average Shares: 2.4B Effective DPS: $39.7B / 2.4B = $16.54/share

Recent Buyback Trend: 2023: $26.0B 2024: $28.5B 2025: $27.5B (includes AI investments)

Estimated Annual Buyback: $27.5B (stable)

####**Projected Dividends + Buybacks (10 years):**

Year Growth % Adjusted DPS PV 1 8.0% $17.86 $16.37 2 7.5% $19.19 $15.98 3 7.0% $20.53 $15.52 4 6.0% $21.76 $14.92 5 5.0% $22.85 $14.19 6 4.0% $23.76 $13.39 7 3.0% $24.47 $12.47 8 2.8% $25.15 $11.55 9 2.5% $25.78 $10.68 10 2.5% $26.43 $9.80

Sum PV (Years 1-10) = $134.87

####**Terminal Value (DDM):**

TV = D₁₀ × (1+g) / (r-g) = $26.43 × 1.025 / (0.095 - 0.025) = $27.09 / 0.070 = $387.0

PV(TV) = $387.0 / (1.095)^10 = $163.4

####**DDM Fair Value:**

Total = $134.87 + $163.4 = $298.3/share

DDM FAIR VALUE: $298.30/share Current Price: $420.50 Downside: -41% (overvalued by DDM)

####**Why the divergence?**

DCF projects $994, DDM projects $298. Why?
- DCF assumes all FCF goes to growth/investment
- DDM only values cash returned (dividends + buybacks)
- Gap = cash retained for growth investments
- Microsoft reinvesting ~60% of FCF into:
  - AI research and development
  - Cloud infrastructure expansion
  - Strategic acquisitions

####**Interpretation:** Gap is normal and healthy for growth company.

#### **Step 4: Multiples Valuation**

####**Peer Selection:**

Comparable technology companies:

Apple (AAPL)
Alphabet/Google (GOOGL)
Adobe (ADBE)
Salesforce (CRM)
ServiceNow (NOW)

####**Peer Multiples:**

Company Price EPS P/E Book Value P/B Revenue/Sh P/S AAPL $195 $6.40 30.5x $39.00 5.0x $35.50 5.5x GOOGL $180 $6.80 26.5x $101.00 1.8x $60.25 3.0x ADBE $625 $13.20 47.3x $82.00 7.6x $97.50 6.4x CRM $290 $1.90 152.6x $10.25 28.3x $40.75 7.1x NOW $745 $2.85 261.4x $15.00 49.7x $82.50 9.0x

Average P/E: 93.7x (skewed by CRM/NOW) Median P/E: 47.3x (more reliable) Trimmed (exclude outliers): (30.5 + 26.5 + 47.3) / 3 = 34.8x

####**Microsoft Metrics:**

TTM EPS: $12.97 Book Value/Share: $35.00 Revenue/Share: $60.75

####**Multiples Fair Values:**

P/E Based: 34.8x × $12.97 = $451.36 P/B Based: 6.8x × $35.00 = $238.00 P/S Based: 6.1x × $60.75 = $370.58

Average: ($451.36 + $238.00 + $370.58) / 3 = $353.31

MULTIPLES FAIR VALUE: $353.31/share Current Price: $420.50 Assessment: Slightly overvalued vs peers

#### **Step 5: Buffett Valuation (Owner's Earnings)**

####**Normalized Owner's Earnings:**

10-Year Average: Net Income: $62.1B D&A: $7.6B CapEx: $8.3B Δ Working Capital: $0.5B

Owner's Earnings = $62.1B + $7.6B - $8.3B - $0.5B = $60.9B

Alternative: Use recent normalized (exclude one-time items) Last 3-year average: $65.8B (more current)

Use conservative: $60.9B

####**Quality Assessment:**

Moat Strength: 9/10

Enterprise software ecosystem (Office, Teams, Azure)
Massive switching costs (billions in documents, training)
Network effects (Microsoft ecosystem)
Durability: 9/10

Software essential forever
Cloud computing core infrastructure
AI leadership emerging
Management: 9/10

Satya Nadella proven (20+ years execution)
Strategic vision (cloud pivot, AI)
Capital allocation excellent
Earnings Consistency: 9/10

Software revenue very predictable
Subscription model (recurring)
Margins stable 35%+
ROE: 8/10

35%+ ROE (excellent)
ROIC >25% (excellent)
Returns exceed cost of capital
TOTAL QUALITY SCORE: 44/50 (Excellent) Quality Multiple: 25x (23-27x range for 40-44 score)

####**Buffett Fair Value:**

FV = Owner's Earnings × Quality Multiple = $60.9B × 25 = $1,522.5B

Per-Share = $1,522.5B / 2.4B = $634.38/share

BUFFETT FAIR VALUE: $634.38/share Current Price: $420.50 Upside: 50.8%

####**Perpetual Bond Yield Check:**

Market Cap: $420.50 × 2.4B = $1,009.2B Earnings Yield: $60.9B / $1,009.2B = 6.03%

Risk-Free Rate: 4.2% Risk Premium: 3.2% (moderate, Microsoft is quality) Required Return: 7.4%

Comparison: EY: 6.03% RR: 7.4% Shortfall: -1.37%

Assessment: Slightly expensive by earnings yield metric But moderate shortfall acceptable for quality business

#### **Step 6: Munger Valuation (Quality-Based)**

####**Business Quality Checklist:**

Moat Strength: 10/10 Unassailable switching costs, ecosystem dominance

Durability: 9/10 Software/cloud permanent, slight AI disruption risk

Management Quality: 9/10 Exceptional execution, insider aligned

Earnings Consistency: 9/10 Very smooth, predictable, high-quality

Return on Capital: 9/10 ROE 35%+, ROIC 25%+, exceptional

TOTAL: 46/50 (Exceptional) Quality Multiple: 31x (top tier for 45-50 score)

####**Munger Fair Value:**

Normalized EPS (TTM): $12.97 Quality Multiple: 31x

FV = 31x × $12.97 = $402.07/share

MUNGER FAIR VALUE: $402.07/share Current Price: $420.50 Assessment: Fairly valued (slight premium)

####**Five Stock Test:**

Q1: Is this business extraordinary? ✅ YES - Quality 46/50, exceptional moat, durable

Q2: Would I own it forever? ✅ YES - Software/cloud essential, Microsoft likely dominant

Q3: Is valuation reasonable? ⚠️ MAYBE - Fair value $402, current price $420

Small 4.5% premium
Munger willing to pay small premium for exceptional quality
But would prefer $360-380 for better margin
RESULT: ✅ YES, but would prefer lower price

#### **Step 7: Consensus Valuation**

####**Summary of All Methods:**

DCF: $994/share (optimistic - assumes perfect execution) DDM: $298/share (conservative - only dividends/buybacks) Multiples: $353/share (market-based validation) Buffett: $634/share (owner's earnings method) Munger: $402/share (quality-based multiple)

WEIGHTED CONSENSUS (DCF 40%, Multiples 20%, Buffett 15%, Munger 25%): = ($994 × 0.40) + ($353 × 0.20) + ($634 × 0.15) + ($402 × 0.25) = $397.6 + $70.6 + $95.1 + $100.5 = $663.8/share

MEDIAN (removes outliers): $402/share

CONSERVATIVE RANGE: $300-400/share CENTRAL ESTIMATE: $403/share OPTIMISTIC RANGE: $634-994/share

####**Fair Value Assessment:**

Most realistic range: $350-500/share Central estimate: $400/share Current price: $420.50/share

#### **Step 8: Margin of Safety Analysis**

####**Against Central Estimate ($400):**

Current Price: $420.50 Fair Value: $400 MOS: ($400 - $420.50) / $400 = -5.1% (OVERVALUED)

Target Buy Prices: ├─ 20% MOS: $400 × 0.80 = $320 (need 24% decline) ├─ 30% MOS: $400 × 0.70 = $280 (need 33% decline) ├─ 40% MOS: $400 × 0.60 = $240 (need 43% decline) └─ 50% MOS: $400 × 0.50 = $200 (need 52% decline)

####**Against Conservative Estimate ($350):**

Current Price: $420.50 Fair Value: $350 (worst reasonable case) MOS: ($350 - $420.50) / $350 = -20.1% (significantly overvalued)

####**Against Optimistic Estimate ($634):**

Current Price: $420.50 Fair Value: $634 (best reasonable case) MOS: ($634 - $420.50) / $634 = +33.7% (undervalued)

####**Conclusion:**

Price range $350-634 gives wide spread. Central estimate $400 suggests stock is fairly valued (no margin). Conservative case suggests overvalued. Optimistic case suggests undervalued.

Buffett's threshold: Needs 25%+ margin Current: -5% margin vs central (doesn't meet threshold)

Recommendation: PASS (waiting for better price)

#### **Step 9: Valuation Grade**

Financial Quality Score: 82/100 (Excellent) Valuation Grade: B (Fair Value)

Reasoning: ✅ Exceptional business quality (score 82) ✅ Excellent growth prospects ✅ Strong competitive position ❌ Current price offers insufficient margin of safety ❌ Price close to or above fair value estimate

GRADE: B (Fair Value, Hold, not buy at current price)

#### **Step 10: Final Recommendation**

BUFFETT RECOMMENDATION: 🟡 HOLD

Fair value $634 vs current $420 suggests upside
BUT declining growth assumptions suggest $400 fair value
Margin of safety insufficient (<25% threshold)
Would buy below $300/share (25% margin)
MUNGER RECOMMENDATION: 🟡 WAIT

Exceptional business (would own forever)
Current price $420 vs $402 fair value = small premium
Munger prefers clear margin, so would wait for $360-380
Put on watchlist, monitor quarterly
CONSENSUS RECOMMENDATION: 🟡 HOLD/PASS

Both recommend pass or wait at current price
Good business, but not compelling valuation
Not urgent to buy
INVESTMENT THESIS:

Microsoft is an exceptional technology business with an unassailable competitive moat built through enterprise software ecosystem lock-in, cloud infrastructure (Azure) dominance, and emerging AI leadership. Financial quality is excellent (score 82/100) with consistent double-digit growth, exceptional returns on capital (35%+ ROE), and fortress balance sheet. However, at $420.50/share, the stock is fairly valued to slightly expensive. Consensus fair value is approximately $400/share, providing insufficient margin of safety by Buffett's standards (requires 25%+ margin). We recommend HOLD if owned and WAIT for better entry point near $280-300/share (25-30% margin). Excellent quality business deserves to be in portfolio, but not at current valuation premium.

GROUPING: GROUP 4 (Overvalued)

Quality: 82/100
MOS: -5% (negative)
Recommendation: HOLD/WAIT for better price

#### **Step 11: Sensitivity Summary**

Fair Value Range (All Scenarios):

Conservative Case ($350):

WACC 12%, Terminal Growth 1.5%, Quality Multiple 20x
Stock trading at 20% premium
NOT ATTRACTIVE
Base Case ($400):

WACC 10%, Terminal Growth 2.5%, Quality Multiple 25x
Stock trading fairly valued (small premium)
ACCEPTABLE but no hurry
Optimistic Case ($634):

WACC 8%, Terminal Growth 3.5%, Quality Multiple 28x
Stock trading at 20% discount
ATTRACTIVE (but assumes perfect execution)
REALISTIC RANGE: $350-500/share MOST PROBABLE: $380-420/share CURRENT PRICE: $420.50/share

Conclusion: Stock within reasonable valuation range but no margin of safety.

---

### **11.2 Complete Example #2: Dividend Stock (Coca-Cola - Mature, Stable)**

#### **Company Overview**

Ticker: KO Company: The Coca-Cola Company Sector: Consumer Staples/Beverages Current Stock Price: $61.50 Shares Outstanding: 2.8B Market Cap: $172.2B Date of Analysis: 2026-03-14

#### **Quick Analysis (Abbreviated)**

####**10-Year Dividend History:**

Year DPS Growth 2016 $1.40 — 2017 $1.48 5.7% 2018 $1.56 5.4% 2019 $1.64 5.1% 2020 $1.72 4.9% 2021 $1.76 2.3% 2022 $1.80 2.3% 2023 $1.82 1.1% 2024 $1.84 1.1% 2025 $1.84 0.0%

Trend: Growth decelerating (5%+ to flat) Assessment: Mature company, dividend growth plateauing

####**DCF Valuation:**

Historical FCF CAGR: 2.1% (very slow, mature company) Projection: 2.5% growth perpetual (no acceleration expected) WACC: 7.8% (lower risk than Microsoft)

DCF Fair Value: $68.50/share Current Price: $61.50 Upside: 11.4%

####**DDM Valuation:**

Expected Dividend (next year): $1.89 (2.7% growth assumed) Required Return: 7.8% Terminal Growth: 2.5%

Using Gordon Growth Model: FV = D₁ / (r - g) = $1.89 / (0.078 - 0.025) = $1.89 / 0.053 = $35.66/share

Wait - that's too low. Let me recalculate with two-stage:

Years 1-5: 2.5% growth Years 6+: 2.5% terminal

Projected Dividends: Year 1: $1.89 PV: $1.75 Year 2: $1.94 PV: $1.67 Year 3: $1.98 PV: $1.60 Year 4: $2.03 PV: $1.53 Year 5: $2.08 PV: $1.47 ...continuing 5-10 years... Sum PV Years 1-10: $14.59

Terminal Value: $45.47 PV(TV): $21.75

DDM Fair Value: $36.34/share

Current Price: $61.50 Assessment: OVERVALUED by 69% (significant overvaluation!)

####**Why the disconnect?**

DCF says $68.50 (undervalued) DDM says $36.34 (significantly overvalued) Current price: $61.50 (in between)

Why? Coca-Cola retains earnings for other purposes:

Increased dividends for other regions
Share buybacks
Strategic acquisitions
Capital investments
If we adjust DDM for buybacks: Cash Returned (Div + Buybacks): $14.2B Effective DPS: $14.2B / 2.8B = $5.07/share

Using buyback-adjusted DDM: FV = $5.07 / (0.078 - 0.025) = $95.66/share

That's still higher than current price! So adjusted DDM says undervalued.

####**Multiples Valuation:**

Peer P/E multiples (beverage companies): PepsiCo: 26x P/E Monster Energy: 35x P/E Red Bull (private): N/A Average: 30x P/E

Coca-Cola EPS: $2.10 Fair Value = 30x × $2.10 = $63/share

Current Price: $61.50 (slightly undervalued)

####**Buffett Valuation:**

Owner's Earnings: $13.2B (normalized) Quality Multiple: 32x (exceptional moat - brand)

FV = $13.2B × 32 = $422.4B / 2.8B = $150.86/share

Earnings Yield: $13.2B / $172.2B = 7.67% Required Return: 6.7% (4.2% + 2.5% premium)

EY > RR: YES (attractive)

####**Munger Valuation:**

Quality Score: 45/50 (Exceptional) Quality Multiple: 30x

FV = 30x × $2.10 = $63/share

Five Stock Test: ✅ YES

####**Consensus Summary:**

DCF: $68.50 DDM (unadjusted): $36.34 DDM (buyback-adjusted): $95.66 Multiples: $63.00 Buffett: $150.86 (seems high, may be conservative) Munger: $63.00

Range: $36-$151 (huge spread!) Most likely: $60-95 (more realistic) Central: $73.00/share

####**Margin of Safety:**

Current Price: $61.50 Fair Value (conservative): $60 Fair Value (central): $73 Fair Value (optimistic): $95

Assessment: vs conservative: -2% (slightly overvalued) vs central: -16% (moderately overvalued) vs optimistic: -35% (significantly undervalued)

Interpretation: Coca-Cola is priced for optimistic case ($95) More realistic central case ($73) suggests 19% downside Conservative case ($60) suggests 2% downside

Stock trading at upper end of reasonable range.

####**Final Recommendation:**

BUFFETT: 🟢 BUY (strong positive assessment) MUNGER: 🟡 WAIT (good business, not compelling price)

CONSENSUS: 🟡 HOLD (fairly valued, good business, no hurry)

INVESTMENT THESIS:

Coca-Cola is one of the world's greatest businesses with an unmatched brand moat, 100+ years of proven durability, and predictable cash generation. The company has consistently returned cash to shareholders through increasing dividends and buybacks. At $61.50/share, the stock is fairly valued to slightly expensive depending on assumptions. Conservative fair value estimates ($60-65) suggest little margin of safety, while optimistic scenarios ($90-95) suggest undervaluation. For a quality holding, Coca-Cola deserves a place in any long-term portfolio, but current price offers limited upside. Recommend HOLD if owned; for new purchases, wait for price dip to $55/share (10% below current) for better entry.

GROUPING: GROUP 3 (Fair Value)

Quality: Exceptional (dividend aristocrat)
MOS: ~5% (minimal)
Recommendation: HOLD/ACCUMULATE on dips

---

### **11.3 Complete Example #3: Unprofitable Company (Biotech with Losses)**

#### **Company Overview**

Ticker: BIOTECH (fictional) Company: Biotech Innovation Corp Sector: Biotechnology/Pharmaceuticals Current Stock Price: $75.00 Market Cap: $6.2B Revenue (TTM): $520M EPS (TTM): -$2.15 (losses)

#### **Problem: Negative Earnings**

Company is losing money - cannot use P/E multiple

Options:

Project when profitable (forward earnings)
Use P/S multiple (price-to-sales)
Use P/B multiple if applicable
Skip valuation if too uncertain

#### **Approach: Use P/S Multiple**

**Peer Biotech Multiples (profitable peers):**

Company Price Revenue/Share P/S Regeneron $850 $150 5.7x Amgen $330 $95 3.5x Moderna $180 $35 5.1x Average P/S 4.8x

**Biotech Innovation Corp:**

Revenue: $520M Shares: 82.7M Revenue/Share: $6.28

Fair Value = 4.8x × $6.28 = $30.14/share Current Price: $75.00

**Assessment:**

Stock trading at 2.5x what peers pay for same revenue Explanation:

Market betting on future drug approval
Pipeline has promising phase 3 trials
Price reflects hope for profitability
Valuation Grade: F (Overvalued)

Company unprofitable
Trading at premium to peers
High risk, high price = poor risk/reward
Recommend: AVOID or PASS
Potential: Could be worth much more if drugs approved But: Current price already prices in success Risk/Reward: Unfavorable (large downside if approvals fail)

---

## Section 12: Implementation Notes

### **12.1 Data Sources & Integration**

**SEC EDGAR (10-K/10-Q):**

URL: https://www.sec.gov/cgi-bin/browse-edgar Use: Download 10-year historical financials Frequency: Quarterly (10-Q) and Annual (10-K) Key items:

Consolidated Statements of Income
Consolidated Statements of Cash Flows
Consolidated Balance Sheets

**Yahoo Finance API:**

URL: https://finance.yahoo.com Use: Current stock prices, dividends, shares outstanding Frequency: Daily Data points:

Current stock price
52-week range
Dividend history
Stock splits, share count


**Federal Reserve FRED:**

URL: https://fred.stlouisfed.org Use: Risk-free rate (10-year Treasury yield) Frequency: Daily Symbol: GS10 (10-Year Treasury Constant Maturity Rate)

**Company Investor Relations:**

Source: Company websites Use: Press releases, shareholder letters, presentations Frequency: Quarterly/Annual Information:

Dividend announcements
Share buyback authorization
Guidance on future performance

---

### **12.2 Calculation Considerations**

**Rounding & Precision:**

Financial statements: Use billions for large numbers Per-share values: Round to 2 decimal places Percentages: Round to 1 decimal place Multiples: Round to 1 decimal place

Example: $1,234,567,890 = $1.23B (3 significant figures) Fair Value: $123.45/share Growth Rate: 8.3% P/E Multiple: 22.5x

**Handling Missing Data:**

If <10 years available:

Use what's available (note limitation)
Example: Only 7 years historical
Calculate CAGR on 7 years
Add note: "Limited to 7 years of data available"
If quarter incomplete:

Use last complete quarter
Note: "Data as of Q3 2025"
Mark as preliminary if very recent
If single year of losses:

Use normalized earnings (average profitable years)
Example: Negative in 2023, normalize to 2022/2021 average
Note methodology in assumptions

**Edge Cases:**

Negative earnings: ✅ Use P/B, P/S multiples ✅ Project forward earnings ❌ Never use P/E with negative earnings

Negative book value (more liabilities than assets): ✅ Use P/S multiple ✅ Use DCF (if company has positive cash flow) ❌ Don't use P/B multiple

Negative cash flow (company burning cash): ✅ Use P/S multiple ✅ Project to profitability ❌ May not be investable if no path to profitability

Zero dividends + substantial buybacks: ✅ Use buyback-adjusted dividend in DDM ✅ Works same as dividend equivalent ⚠️ Monitor buyback pace (may not be sustainable)

---

### **12.3 Quality Checkpoints**

**Before accepting valuation results:**

**Data Quality:**

☑️ Financial statements match across periods ☑️ Net income consistent with operating cash flow (±20%) ☑️ Balance sheet balances (Assets = Liabilities + Equity) ☑️ Stock price current (not stale) ☑️ Share count reflects dilution (stock options, warrants)

**Calculation Accuracy:**

☑️ FCF = Operating CF - CapEx (positive or improving) ☑️ ROE = Net Income / Avg Equity (reasonable level) ☑️ P/E = Stock Price / EPS (peer comparison makes sense) ☑️ WACC components (cost of debt, equity) reasonable ☑️ DCF terminal value <50% of total value (or flag)

**Valuation Sanity Checks:**

☑️ Fair value in reasonable range ($100-$500 for large-cap) ☑️ Fair value > current price OR has margin of safety ☑️ Quality multiple matches business quality (not 50x for weak moat) ☑️ Growth assumptions supported by historical data ☑️ Terminal growth rate ≤ GDP growth rate (2.5% conservative) ☑️ DCF and Multiples within reasonable range of each other ☑️ Buffett and Munger recommendations make sense together

**Red Flags (Stop & Investigate):**

🚩 Fair value <current price with no margin → Re-check calculations, may be overvalued

🚩 DCF >> Multiples >> Buffett (wide divergence) → One method has bad assumptions → Investigate which is unreliable

🚩 Fair value changes >30% quarter-over-quarter → Either stock price moved or assumptions changed significantly → Document what changed

🚩 Quality multiple assigned is 35x+ for weak moat company → Over-optimistic assessment → Review quality scoring

🚩 Terminal growth rate >3% → Unrealistic (outpacing economy forever) → Reset to 2.5%

🚩 WACC is <5% or >15% → Likely wrong (too extreme) �� Review components

---

### **12.4 Performance Optimization**

**Valuation Update Frequency:**

Daily: Stock prices (automatic via API) Weekly: Reviews/monitoring (manual) Monthly: Full revaluation if major price move (>10%) Quarterly: Full revaluation at earnings season Annual: Deep reanalysis with fresh assumptions

**When to Revalue:**

Trigger: Stock price moves >20% from fair value

Examples: Fair value: $100 Previous price: $90 (10% discount) New price: $70 (30% discount) Action: Revalue immediately (crossed threshold)

Fair value: $100 Previous price: $90 (10% discount) New price: $85 (15% discount) Action: Monitor, no urgent revaluation needed

**Batch Processing:**

For screening 500 companies quarterly: Stage 1 (Filters): ~2-3 hours Stage 2-3 (Financial Analysis): ~4-5 hours Stage 5.4 (Valuation): ~3-4 hours Stage 6-7 (Reporting): ~2 hours

Total time: ~12-14 hours per quarter Frequency: Once per quarter (every 90 days)

---

## Section 13: Troubleshooting Guide

### **13.1 Common Issues & Solutions**

**Problem: DCF valuation is unrealistic ($500 for $50 stock)**

Likely causes:

Terminal growth rate too high (>3%)
WACC too low (<5%)
FCF growth assumptions too aggressive
Terminal value >70% of total
Solutions: ✅ Check terminal growth (reset to 2.5%) ✅ Verify WACC calculation (compare to peer WACC) ✅ Review growth assumptions (vs industry growth) ✅ If terminal value >70%, use more conservative growth

If still high: → Company may be in hyper-growth (acceptable if realistic) → Or fair value truly much higher than current price → Document assumptions clearly

**Problem: Multiples valuation much different from DCF**

Example: DCF = $300, Multiples = $500

Possible reasons:

Peers are also overvalued/undervalued
Company growth different from peers
Margins different from peers
Quality different from peers
Investigation: ✅ Check if peers trading at premiums (bull market) ✅ Compare growth rates (is this company faster/slower?) ✅ Compare margins (profit quality) ✅ Compare moats (competitive position)

Resolution: → If DCF is conservative, Multiples more realistic → If Multiples inflated by market sentiment, DCF more reliable → Likely truth is between them → Use weighted average, not just one method

**Problem: Dividend history inconsistent (cuts, freezes)**

Company: 2021: $2.00 dividend 2022: $1.80 (cut 10%) 2023: $1.80 (frozen) 2024: $1.82 (resume 1%) 2025: $1.84 (slow growth)

Assessment: ✅ Company had financial stress in 2022 ✅ Now recovering ✅ Dividend sustainable going forward?

Action: → Use most recent trend (2023-2025) → Not full historical CAGR (includes cut period) → More conservative growth assumption (2%) → Note in documentation: "Dividend cut in 2022, now stabilizing"

**Problem: Negative book value (balance sheet distressed)**

Company: $100B market cap, -$5B equity This means: Liabilities > Assets Company technically insolvent on balance sheet

DCF: Still valid if cash flows positive P/E: Still valid if profitable P/B: NOT valid (negative book value) P/S: Still valid

Action: ✅ Skip P/B multiple ✅ Use P/S multiple as primary ✅ Investigate why book value negative ✅ Is company distressed or restructuring? ✅ If distressed, may fail (avoid) ✅ If restructuring, opportunity exists (risky)

**Problem: Company paid special dividend (skewed results)**

Dividend history: 2022: $1.50 regular + $0.50 special = $2.00 2023: $1.55 regular 2024: $1.60 regular 2025: $1.65 regular

Issue: If use $2.00 average, over-estimates regular dividend

Solution: ✅ Separate regular from special ✅ Use $1.50-1.65 for projections (regular only) ✅ Special dividend is one-time, not recurring ✅ Project only regular dividend growth

**Problem: Company just changed business model**

Scenario: Company was manufacturing, now shifting to services Historical metrics not relevant to future

Example: IBM: Shifted from hardware → software/services GE: Multiple business exits/restructuring

Solution: ✅ Use only recent years reflecting new business ✅ If transition not complete, be conservative ✅ May need to split valuation: - Old business (declining, uses FCF) - New business (growing, needs separate DCF) ✅ Document the transition ✅ Add extra margin of safety due to uncertainty

---

## Section 14: Related Documents

| Document | Covers | Purpose |
|----------|--------|---------|
| [0-BASIC-STAGE-WORKFLOW.md](0-BASIC-STAGE-WORKFLOW.md) | All Stages | Overall pipeline and workflow definitions |
| [01-INVESTMENT-REQUIREMENTS.md](01-INVESTMENT-REQUIREMENTS.md) | Stages 0-5 | Financial analysis that feeds into Stage 6 |
| [02-VALUATION-METHODOLOGY.md](02-VALUATION-METHODOLOGY.md) | Stage 6 | Intrinsic value calculation (this document) |
| [03-RESULTS-REPORTING-REQUIREMENTS.md](03-RESULTS-REPORTING-REQUIREMENTS.md) | Stage 7 | Results reporting and output specifications |

---

## Section 15: Document History

| 1.0 | 2026-03-15 | ThommCroft | **CURRENT** - Initial comprehensive valuation methodology document; aligned with 0-BASIC-STAGE-WORKFLOW.md as Stage 6 requirements |

---

**End of Document**

For questions or clarifications on valuation methodology, refer to the five detailed method sections (DCF, DDM, Multiples, Buffett, Munger) and the real-world examples provided in Section 11.
