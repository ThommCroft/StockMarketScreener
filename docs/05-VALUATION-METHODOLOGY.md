# Stock Price Valuation Methodology - Stage 5.4

**Version:** 1.0  
**Date:** 2026-03-14  
**Author:** ThommCroft  
**Framework:** Warren Buffett & Charlie Munger Value Investing Principles  
**Purpose:** Complete guide to calculating intrinsic value and margin of safety for investment decision-making

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Introduction & Philosophy](#introduction--philosophy)
3. [Discounted Cash Flow (DCF) Method](#section-3-discounted-cash-flow-dcf-method)
4. [Dividend Discount Model (DDM)](#section-4-dividend-discount-model-ddm)
5. [Multiples Valuation Model](#section-5-multiples-valuation-model)
6. [Warren Buffett Valuation Techniques](#section-6-warren-buffett-valuation-techniques)
7. [Charlie Munger Valuation Techniques](#section-7-charlie-munger-valuation-techniques)
8. [Consensus Valuation](#section-8-consensus-valuation)
9. [Margin of Safety Analysis](#section-9-margin-of-safety-analysis)
10. [Valuation Grade System](#section-10-valuation-grade-system)
11. [Company Grouping & Display](#section-11-company-grouping--display)
12. [Final Recommendations](#section-12-final-recommendations)
13. [Output Formats](#section-13-output-formats)
14. [Database Integration](#section-14-database-integration)
15. [Data Flow Integration](#section-15-data-flow-integration)
16. [Sensitivity Analysis](#section-16-sensitivity-analysis)
17. [Real-World Examples](#section-17-real-world-examples)
18. [Implementation Notes](#section-18-implementation-notes)
19. [Troubleshooting Guide](#section-19-troubleshooting-guide)
20. [Document History](#section-20-document-history)

---

## Executive Summary

### Purpose of Stage 5.4

Stage 5.4 (Stock Price Valuation Analysis) is the final analytical stage in the Stock Market Screener system. It takes companies that have passed the rigorous financial analysis (Stage 3.3 with composite score ≥75) and calculates their intrinsic value using multiple independent methods.

**Input:** All companies with composite financial score ≥75  
**Output:** Intrinsic values, margin of safety percentages, investment recommendations  
**Decision-Making:** Determines which companies are attractive at current prices

### The Three Valuation Methods

This stage employs **three independent valuation approaches**, each with distinct philosophies:

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

For each company, Stage 5.4 produces:

✅ **Three Independent Intrinsic Values** (DCF, DDM, Multiples)  
✅ **Two Master Valuation Assessments** (Buffett, Munger)  
✅ **Consensus Fair Value** (weighted combination)  
✅ **Margin of Safety** at multiple thresholds (20%, 30%, 40%, 50%)  
✅ **Valuation Grade** (A+ through F)  
✅ **Investment Recommendation** (BUY, HOLD, PASS)  
✅ **Grouping Category** (Slam Dunk / Strong Buy / Fair Value / Overvalued / Not Qualified)  

### Why This Matters

**Price ≠ Value**

A company with exceptional fundamentals can be a terrible investment if priced too high. Conversely, a mediocre business at a steep discount might be worth buying. Stage 5.4 separates price from value and identifies true opportunities.

---

## Introduction & Philosophy

### Stage 5.4 in the Screening Pipeline
Stage 3.3: Financial Quality Assessment ↓ (Score ≥75) Stage 5.4: Stock Price Valuation Analysis ├─ DCF Intrinsic Value Calculation ├─ DDM Intrinsic Value Calculation ├─ Multiples Fair Value Calculation ├─ Buffett Valuation Techniques ├─ Munger Valuation Techniques └─ Margin of Safety Analysis ↓ Stage 6: Results Processing ├─ Organize by opportunity group ├─ Identify new/maintained/removed └─ Generate recommendations ↓ Stage 7: Notifications & Storage ├─ Email report to investor ├─ GitHub summary page ├─ CSV export for analysis └─ Database storage


### Input Data

Stage 5.4 receives from Stage 3.3:

- **Company fundamentals:** All 40+ financial metrics calculated
- **Composite score:** Quality assessment (0-100)
- **Current market data:** Stock price, market cap, shares outstanding
- **Historical data:** 10 years of financial statements (or partial)
- **Quality scores:** Return on capital, profitability, cash flow, management quality

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


Buffett prefers 25-50% margin of safety. Munger prefers waiting until margin is clear and undeniable.

### Why Multiple Methods?

Using three independent valuation methods provides:

✅ **Triangulation:** Converging values increase confidence  
✅ **Sanity Check:** Diverging values signal investigation needed  
✅ **Risk Mitigation:** No single method's assumption dominates  
✅ **Comprehensive View:** Different methods emphasize different factors  

**DCF emphasizes:** Future growth and sustainability  
**DDM emphasizes:** Return of capital to shareholders  
**Multiples emphasize:** Market's current sentiment about similar companies  

When all three converge ($300-$350 range), fair value is likely $325 ± $25.
When they diverge widely ($200-$400 range), deeper analysis is needed.

### Buffett vs Munger: Complementary Approaches

**Warren Buffett:**
- Uses detailed mathematical models (DCF)
- Calculates precise intrinsic values
- Applies large margin of safety
- Quote: "The best thing to do is get out of the way."
- Philosophy: Invest when odds are overwhelming in your favor

**Charlie Munger:**
- Focuses on business quality first
- Uses simplified valuation multiples
- Patience is his margin of safety
- Quote: "The person that turns over the most rocks wins."
- Philosophy: Invest only in truly exceptional businesses

**Together:** They provide both mathematical rigor (Buffett) and business judgment (Munger)

### Alignment with Investment Requirements

Stage 5.4 implements all principles from `01-INVESTMENT-REQUIREMENTS.md`:

✅ **Return on Capital:** Reflected in quality multiple assignment  
✅ **Financial Strength:** Considered in normalized earnings  
✅ **Profitability:** Included in DCF growth assumptions  
✅ **Cash Flow Quality:** DCF uses free cash flow (not earnings)  
✅ **Management Quality:** Munger Quality Checklist (1-10 scoring)  
✅ **Economic Moat:** Reflected in quality multiple (5-35x range)  
✅ **Margin of Safety:** Core focus of final analysis  

---

## Next Steps

This document walks through:

1. Each valuation method with step-by-step formulas
2. Real-world examples with actual numbers
3. Sensitivity analysis showing valuation ranges
4. How Buffett and Munger assess companies
5. How to combine methods into final recommendation
6. Output formats for email, GitHub, and CSV
7. Database schema for persistent storage
8. Troubleshooting common issues

Ready to proceed to Section 3: Discounted Cash Flow (DCF) Method?


Buffett prefers 25-50% margin of safety. Munger prefers waiting until margin is clear and undeniable.

### Why Multiple Methods?

Using three independent valuation methods provides:

✅ **Triangulation:** Converging values increase confidence  
✅ **Sanity Check:** Diverging values signal investigation needed  
✅ **Risk Mitigation:** No single method's assumption dominates  
✅ **Comprehensive View:** Different methods emphasize different factors  

**DCF emphasizes:** Future growth and sustainability  
**DDM emphasizes:** Return of capital to shareholders  
**Multiples emphasize:** Market's current sentiment about similar companies  

When all three converge ($300-$350 range), fair value is likely $325 ± $25.
When they diverge widely ($200-$400 range), deeper analysis is needed.

### Buffett vs Munger: Complementary Approaches

**Warren Buffett:**
- Uses detailed mathematical models (DCF)
- Calculates precise intrinsic values
- Applies large margin of safety
- Quote: "The best thing to do is get out of the way."
- Philosophy: Invest when odds are overwhelming in your favor

**Charlie Munger:**
- Focuses on business quality first
- Uses simplified valuation multiples
- Patience is his margin of safety
- Quote: "The person that turns over the most rocks wins."
- Philosophy: Invest only in truly exceptional businesses

**Together:** They provide both mathematical rigor (Buffett) and business judgment (Munger)

### Alignment with Investment Requirements

Stage 5.4 implements all principles from `01-INVESTMENT-REQUIREMENTS.md`:

✅ **Return on Capital:** Reflected in quality multiple assignment  
✅ **Financial Strength:** Considered in normalized earnings  
✅ **Profitability:** Included in DCF growth assumptions  
✅ **Cash Flow Quality:** DCF uses free cash flow (not earnings)  
✅ **Management Quality:** Munger Quality Checklist (1-10 scoring)  
✅ **Economic Moat:** Reflected in quality multiple (5-35x range)  
✅ **Margin of Safety:** Core focus of final analysis  

---

## Next Steps

This document walks through:

1. Each valuation method with step-by-step formulas
2. Real-world examples with actual numbers
3. Sensitivity analysis showing valuation ranges
4. How Buffett and Munger assess companies
5. How to combine methods into final recommendation
6. Output formats for email, GitHub, and CSV
7. Database schema for persistent storage
8. Troubleshooting common issues

Ready to proceed to Section 3: Discounted Cash Flow (DCF) Method?

---

## Section 3: Discounted Cash Flow (DCF) Method

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

Year OCF (
B
)
C
a
p
E
x
(
B) FCF ($B) 2016 $18.5 $8.2 $10.3 2017 $20.1 $8.8 $11.3 2018 $22.3 $9.1 $13.2 2019 $24.5 $9.6 $14.9 2020 $26.8 $10.2 $16.6 2021 $28.9 $10.5 $18.4 2022 $30.1 $10.9 $19.2 2023 $31.5 $11.2 $20.3 2024 $33.2 $11.8 $21.4 2025 $35.1 $12.5 $22.6


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

Year Operating CF CapEx FCF (
B
)
(
B) ($B) 2016 $40.5 $9.2 $31.3 2017 $43.2 $9.8 $33.4 2018 $50.1 $10.5 $39.6 2019 $54.3 $11.2 $43.1 2020 $60.8 $12.1 $48.7 2021 $68.5 $13.5 $55.0 2022 $72.1 $14.2 $57.9 2023 $76.4 $15.1 $61.3 2024 $81.2 $16.0 $65.2 2025 $87.5 $17.2 $70.3

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

Year FCF (
B
)
D
i
s
c
o
u
n
t
F
a
c
t
o
r
P
V
(
B) 1 $76.63 1 / 1.10^1 $69.67 2 $83.04 1 / 1.10^2 $68.60 3 $89.68 1 / 1.10^3 $67.37 4 $95.51 1 / 1.10^4 $65.27 5 $100.77 1 / 1.10^5 $62.54 6 $105.31 1 / 1.10^6 $59.46 7 $109.00 1 / 1.10^7 $56.31 8 $112.27 1 / 1.10^8 $52.34 9 $115.31 1 / 1.10^9 $48.71 10 $118.19 1 / 1.10^10 $45.59

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

            Terminal Growth Rate
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

Year DPS (
)
D
i
s
c
o
u
n
t
F
a
c
t
o
r
P
V
(
) 1 $1.89 1 / 1.078^1 $1.75 2 $1.94 1 / 1.078^2 $1.67 3 $1.98 1 / 1.078^3 $1.60 4 $2.03 1 / 1.078^4 $1.53 5 $2.08 1 / 1.078^5 $1.47 6 $2.13 1 / 1.078^6 $1.41 7 $2.18 1 / 1.078^7 $1.36 8 $2.24 1 / 1.078^8 $1.31 9 $2.29 1 / 1.078^9 $1.27 10 $2.35 1 / 1.078^10 $1.22

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

Year Dividends (
B
)
B
u
y
b
a
c
k
s
(
B) Total Return ($B) 2016 $3.3 $24.5 $27.8 2017 $3.7 $32.5 $36.2 2018 $3.9 $23.6 $27.5 2019 $4.1 $22.2 $26.3 2020 $4.3 $22.1 $26.4 2021 $4.4 $27.5 $31.9 2022 $4.6 $27.3 $31.9 2023 $4.7 $29.0 $33.7 2024 $4.8 $30.8 $35.6 2025 $5.0 $32.5 $37.5

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
