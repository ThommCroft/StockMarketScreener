# Stage 5.4: Stock Price Valuation Analysis

**Version:** 1.0  
**Date:** 2026-03-14  
**Author:** ThommCroft  
**Purpose:** Comprehensive documentation for Stage 5.4 — Stock Price Valuation Analysis for companies that have passed all financial screening filters

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Stage Placement & Data Flow Integration](#stage-placement--data-flow-integration)
3. [Valuation Philosophy (Buffett/Munger Framework)](#valuation-philosophy-buffettmunger-framework)
4. [Input Data Requirements](#input-data-requirements)
5. [Method 1: Discounted Cash Flow (DCF)](#method-1-discounted-cash-flow-dcf)
6. [Method 2: Dividend Discount Model (DDM)](#method-2-dividend-discount-model-ddm)
7. [Method 3: Multiples Model (P/E, P/B, P/S)](#method-3-multiples-model-pe-pb-ps)
8. [Buffett Valuation Techniques](#buffett-valuation-techniques)
9. [Munger Valuation Techniques](#munger-valuation-techniques)
10. [Consensus Intrinsic Value Calculation](#consensus-intrinsic-value-calculation)
11. [Sensitivity Analysis](#sensitivity-analysis)
12. [Margin of Safety Analysis](#margin-of-safety-analysis)
13. [Company Grouping for Display](#company-grouping-for-display)
14. [Final Results Display](#final-results-display)
15. [Database Schema](#database-schema)
16. [Example Walkthrough](#example-walkthrough)
17. [Document History](#document-history)

---

## Executive Summary

**Stage 5.4** is the final valuation gate before any investment purchase decision. It receives companies that have already passed all hard financial filters AND achieved a composite screening score ≥ 75. At this stage, the question is no longer *"Is this a quality business?"* — that has already been confirmed — but rather: **"Is this quality business available at a fair or attractive price?"**

This stage applies three industry-standard valuation models combined with Buffett's and Munger's personal valuation techniques to calculate multiple intrinsic value estimates. All estimates are presented alongside the current market price, and margin of safety percentages are calculated at 20%, 30%, 40%, and 50% safety thresholds to give the investor clear, actionable buy targets.

**Key Outputs per Company:**
- Intrinsic value from 5 independent methods
- Consensus (average) intrinsic value
- Current margin of safety percentage
- Target buy prices at 4 safety margin levels
- Company grouping category (Slam Dunk Buy → Not Qualified)

**No scoring or pass/fail decision is made here.** Stage 5.4 is purely informational and supports manual investment decision-making.

---

## Stage Placement & Data Flow Integration

### Position in the Screening Pipeline

```
Input: Industry List (from appsettings.json)
   │
   ▼
┌──────────────────────────────────────────────────────────────┐
│ Stage 0: Company Pre-Filtering (Market Cap > $300M)          │
└────────────────┬────────────────────────────────────────────┘
                 ▼
┌──────────────────────────────────────────────────────────────┐
│ Stage 1: Data Ingestion & Reconciliation                     │
└────────────────┬────────────────────────────────────────────┘
                 ▼
┌──────────────────────────────────────────────────────────────┐
│ Merged Stages 2-3: Metric Calculation + Hard Filter          │
│ └─ ~100-150 companies pass hard filters                      │
└────────────────┬────────────────────────────────────────────┘
                 ▼
┌──────────────────────────────────────────────────────────────┐
│ Stage 4: Quality Assessment Scoring (0-100)                  │
└────────────────┬────────────────────────────────────────────┘
                 ▼
┌──────────────────────────────────────────────────────────────┐
│ Stage 5: Valuation & Management Ranking                      │
│ ├─ 5.1 Valuation Scoring (max 35 pts)                       │
│ ├─ 5.2 Management Quality Scoring (max 25 pts)              │
│ ├─ 5.3 Final Composite Score Calculation                     │
│ │  └─ Only companies with Composite Score ≥ 75 proceed       │
│ └─ ~45-60 companies qualify                                  │
└────────────────┬────────────────────────────────────────────┘
                 ▼
┌──────────────────────────────────────────────────────────────┐
│ Stage 5.4: STOCK PRICE VALUATION ANALYSIS ◄ YOU ARE HERE    │
│ ├─ Input: All qualified companies (score ≥ 75)              │
│ ├─ Calculate: DCF, DDM, Multiples, Buffett, Munger values   │
│ ├─ Calculate: Consensus intrinsic value                      │
│ ├─ Calculate: Margin of safety at 20/30/40/50% thresholds   │
│ └─ Output: Complete valuation cards per company              │
└────────────────┬────────────────────────────────────────────┘
                 ▼
┌──────────────────────────────────────────────────────────────┐
│ Stage 6: Results Processing & Storage                        │
│ ├─ Group companies by valuation category                     │
│ ├─ Store to MySQL (CompanyValuations + HistoricalTracking)   │
│ ├─ Generate email report (grouped, ranked by quality score)  │
│ ├─ Export CSV (all 40+ metrics + all valuations)             │
│ └─ Publish GitHub Summary Page (markdown)                    │
└──────────────────────────────────────────────────────────────┘
```

### Input from Stage 3.3

Stage 5.4 receives the following pre-calculated data from Stage 3.3 (Valuation & Management Ranking):

| Data Point | Source | Used For |
|------------|--------|----------|
| 10-year FCF history | Stage 1 ingestion | DCF calculation |
| 10-year EPS history | Stage 1 ingestion | Multiples, DDM |
| 10-year Dividend history | Stage 1 ingestion | DDM |
| 10-year Revenue history | Stage 1 ingestion | P/S Multiple |
| 10-year Book Value history | Stage 1 ingestion | P/B Multiple |
| Current stock price | Weekly update | Margin of safety |
| Shares outstanding (diluted) | SEC EDGAR | DCF per-share value |
| Total debt, Cash & equivalents | SEC EDGAR | WACC, DCF |
| Financial Quality Score (0-100) | Stage 4 output | Company grouping |
| Composite Score (0-100) | Stage 5.3 output | Company grouping |
| 10-year Treasury yield | Federal Reserve FRED | Discount rate |
| Share buyback history | SEC EDGAR | DDM adjustment |

---

## Valuation Philosophy (Buffett/Munger Framework)

### Core Principle

> *"Price is what you pay. Value is what you get."* — Warren Buffett

The purpose of Stage 5.4 is to determine whether the **price** the market is asking today is reasonable relative to the **value** the business will deliver over the long term. All valuation methods in this stage are grounded in fundamental analysis, avoiding speculative or momentum-based approaches.

### Buffett's Approach

Warren Buffett uses a small number of valuation techniques that focus on **owner's earnings** — the actual cash a business owner can extract — and compares the business to a **perpetual bond**. His key insight is that a stock is simply a claim on a future stream of cash flows, and the fair price is the present value of those cash flows.

**Key Buffett principles applied:**
- Use Free Cash Flow (or Owner's Earnings) as the primary value driver
- Discount at WACC to reflect the opportunity cost of capital
- Use a 10-year projection period aligned with available data
- Apply 2.5% terminal growth rate (conservative, below long-term GDP)
- Seek a margin of safety as protection against valuation errors

### Munger's Approach

Charlie Munger focuses on **business quality** as the foundation of value. While Buffett often starts with price, Munger starts with quality. His famous line: *"All I want to know is where I'm going to die, so I'll never go there."*

**Key Munger principles applied:**
- Quality assessment before any price discussion
- Quality-based multiples (better businesses deserve higher multiples)
- "Five Stock Test" — would you want to own 20% of this business forever?
- Business quality scoring on a 1–10 scale before assigning a valuation multiple

---

## Input Data Requirements

### Data Already Available (No New Sources Needed)

All data required for Stage 5.4 is collected during Stage 1 (Data Ingestion) and is already available for the 45–60 qualified companies:

```
Per Company:
├─ 10-year Free Cash Flow (FCF) history → DCF calculation
├─ 10-year Earnings Per Share (EPS) history → P/E Multiple, DDM
├─ 10-year Dividend Per Share history → DDM
├─ 10-year Revenue history → P/S Multiple
├─ 10-year Book Value Per Share history → P/B Multiple
├─ 10-year Share Buyback amounts → DDM buyback adjustment
├─ Current stock price (updated weekly) → Margin of safety
├─ Diluted shares outstanding → Per-share intrinsic value
├─ Total long-term debt → WACC calculation
├─ Cash & equivalents → Net debt, DCF terminal value
├─ Cost of debt (average interest rate on debt) → WACC
├─ Beta (5-year monthly) → CAPM equity cost
├─ 10-year US Treasury yield (risk-free rate) → WACC, DDM
├─ Financial Quality Score → Company grouping
└─ Composite Screening Score → Company grouping
```

### Data Handling for Incomplete History

When a company has fewer than 10 years of data available, the valuation calculations align with the available history:

| Available Years | Action |
|-----------------|--------|
| 7–9 years | Use available years; note limitation in output |
| 5–6 years | Use available years; apply higher discount rate (+1%) |
| < 5 years | Skip DCF; use Multiples and Munger methods only |

---

## Method 1: Discounted Cash Flow (DCF)

### Overview

DCF is the primary intrinsic value method. It projects the company's Free Cash Flow (FCF) forward for the available data period (up to 10 years), then calculates a terminal value for perpetual cash flows beyond the projection period. All future cash flows are discounted back to present value using WACC.

### Step 1: Calculate WACC (Weighted Average Cost of Capital)

```
WACC = (Weight of Equity × Cost of Equity) + (Weight of Debt × Cost of Debt × (1 - Tax Rate))

Where:
  Weight of Equity = Market Cap ÷ (Market Cap + Total Debt)
  Weight of Debt   = Total Debt ÷ (Market Cap + Total Debt)

  Cost of Equity (CAPM):
    = Risk-Free Rate + Beta × Equity Risk Premium
    = 10-Year Treasury Yield + Beta × 5.5%

  Cost of Debt:
    = Average Interest Rate on Long-Term Debt

  Tax Rate:
    = Effective Tax Rate (from income statement, typically ~21%)
```

**Example WACC Calculation:**
```
Market Cap:       $10B
Total Debt:       $2B
Total Capital:    $12B

Weight of Equity: $10B ÷ $12B = 83.3%
Weight of Debt:   $2B  ÷ $12B = 16.7%

10-Year Treasury: 4.2%
Beta:             0.85
Equity Risk Premium: 5.5%
Cost of Equity:   4.2% + (0.85 × 5.5%) = 4.2% + 4.7% = 8.9%

Cost of Debt:     5.1%
Tax Rate:         21%
After-tax Cost of Debt: 5.1% × (1 - 0.21) = 4.0%

WACC = (83.3% × 8.9%) + (16.7% × 4.0%)
     = 7.42% + 0.67%
     = 8.09% → Round to 8.1%
```

### Step 2: Determine FCF Growth Rate

```
Historical FCF CAGR (10-year):
  = (Latest Year FCF ÷ Earliest Year FCF)^(1 ÷ 9) - 1

Projected Growth Rate:
  = MIN(Historical FCF CAGR, 15%)   ← Cap at 15% (conservative)

For projection years beyond historical data:
  Year 1–5:  Use Projected Growth Rate
  Year 6–10: Apply decay toward terminal rate
             = Projected Growth Rate × (1 - 0.1 × (Year - 5))
             ← Each year beyond Year 5 reduces growth by 10%
```

**Growth Rate Decay Example:**
```
Projected Growth Rate: 12%

Year 1: 12.0%
Year 2: 12.0%
Year 3: 12.0%
Year 4: 12.0%
Year 5: 12.0%
Year 6: 12.0% × (1 - 0.10 × 1) = 10.8%
Year 7: 12.0% × (1 - 0.10 × 2) =  9.6%
Year 8: 12.0% × (1 - 0.10 × 3) =  8.4%
Year 9: 12.0% × (1 - 0.10 × 4) =  7.2%
Year 10:12.0% × (1 - 0.10 × 5) =  6.0%
Terminal: 2.5% (fixed perpetual growth rate)
```

### Step 3: Project FCF for Each Year

```
FCF(Year N) = FCF(Year 0) × (1 + Growth Rate for Year N)

Where:
  FCF(Year 0) = Latest available annual FCF
  Growth rates calculated per Step 2 above
```

### Step 4: Calculate Terminal Value

```
Terminal Value = FCF(Year 10) × (1 + Terminal Growth Rate)
                 ─────────────────────────────────────────
                        WACC - Terminal Growth Rate

Where:
  Terminal Growth Rate = 2.5% (fixed, perpetual)
  WACC = calculated in Step 1

Terminal Growth Rate must be < WACC (else formula breaks down).
If WACC ≤ 3%, use Terminal Growth Rate of 1.5% instead.
```

### Step 5: Discount All Cash Flows to Present Value

```
PV(FCF Year N) = FCF(Year N)  ÷  (1 + WACC)^N

PV(Terminal Value) = Terminal Value ÷ (1 + WACC)^10

DCF Enterprise Value = Σ PV(FCF Year 1..10) + PV(Terminal Value)
```

### Step 6: Calculate Equity Value Per Share

```
DCF Equity Value = DCF Enterprise Value - Total Debt + Cash & Equivalents

DCF Intrinsic Value Per Share = DCF Equity Value ÷ Diluted Shares Outstanding
```

### DCF Summary Formula

```
DCF Intrinsic Value Per Share =
  [ Σ(FCF(0) × ∏Growth_n ÷ (1+WACC)^n) for n=1..10
    + (FCF(10) × (1+g_t)) ÷ (WACC - g_t) ÷ (1+WACC)^10
    - Total Debt + Cash ]
  ÷ Diluted Shares

Where g_t = 2.5% terminal growth rate
```

---

## Method 2: Dividend Discount Model (DDM)

### Overview

The DDM values a company based on the present value of its future dividend stream. For companies that pay dividends, this captures the actual cash return to shareholders. For non-dividend paying companies, a **buyback-adjusted DDM** is used, which treats share buybacks as an equivalent return of capital to shareholders.

### Applicability Decision

```
IF Annual Dividend Yield ≥ 1%:
  → Use Standard Two-Stage DDM (dividends only)

ELSE IF Annual Share Buybacks > 0 (past 5 years):
  → Use Buyback-Adjusted DDM
     (treat buybacks as equivalent to dividends)

ELSE:
  → Skip DDM
  → Note: "DDM N/A — No dividends or buybacks" in output
  → Consensus value calculated from remaining methods only
```

### Standard Two-Stage DDM (Dividend-Paying Companies)

**Stage 1: High-Growth Phase (Years 1–10)**

```
Dividend Growth Rate (g1):
  = 10-Year Historical Dividend CAGR
  = (Latest DPS ÷ Earliest DPS)^(1 ÷ 9) - 1
  Capped at MIN(g1, 12%)

Payout Ratio Sustainability Check:
  = Current DPS ÷ Current EPS
  IF Payout Ratio > 80%: Reduce g1 by 2% (unsustainable pace)

Year N Dividend:
  = DPS(Year 0) × (1 + g1)^N

Discount Rate = Required Rate of Return (r):
  = WACC (same as DCF calculation)
```

**Stage 2: Terminal (Perpetual) Growth Phase**

```
Terminal Growth Rate (g2) = 2.5% (same as DCF)

Terminal DDM Value:
  = DPS(Year 10) × (1 + g2)  ÷  (r - g2)
```

**DDM Intrinsic Value**

```
DDM Value Per Share =
  Σ [ DPS(Year N) ÷ (1 + r)^N ] for N = 1..10
  + Terminal DDM Value ÷ (1 + r)^10
```

### Buyback-Adjusted DDM (Non-Dividend Companies)

For companies with no dividends or dividend yield < 1%, share buybacks represent an equivalent return of value. Reduced share count increases EPS and FCF per share, benefiting remaining shareholders.

**Adjusted DPS Calculation:**

```
Buyback Yield = Annual Share Buybacks ÷ Market Capitalisation

Effective DPS (Adjusted) = Current EPS × (Buyback Yield + Dividend Yield)

Apply same Two-Stage DDM formula above using Effective DPS instead of actual DPS
```

**Example:**
```
Company: Apple Inc. (AAPL)
Current EPS:         $6.43
Dividend Yield:      0.5% (minimal)
Buyback Yield:       2.8% ($90B buybacks ÷ $3.2T market cap)
Combined Yield:      3.3%

Effective DPS:       $6.43 × 3.3% = $0.21 per year (adjusted)

Note: This is a simplified view; actual buyback impact is captured
more precisely in the DCF through increased FCF per share over time.
```

---

## Method 3: Multiples Model (P/E, P/B, P/S)

### Overview

Multiples valuation uses historical average ratios to estimate fair value. Unlike DCF, it is not a projection model — it anchors valuation to the company's own historical trading range and compares against peers in the same industry. This method provides a market-grounded sanity check against the DCF and DDM estimates.

### Multiple 1: Price-to-Earnings (P/E) Model

**Calculation:**

```
10-Year Average P/E = Average of annual P/E ratios over 10 years

Peer Median P/E = Median P/E ratio across 3–5 industry peers
                  (using same SIC code classification)

Benchmark P/E = Smallest of:
  - 10-Year Average P/E
  - Peer Median P/E
  ← Uses the more conservative of the two

P/E Intrinsic Value = Benchmark P/E × Current EPS (TTM)
```

**Peer Selection Criteria:**
- Same GICS industry sub-sector
- Market cap within 50% of target company
- Must have passed Stage 3.1 hard filters (financial strength)
- Minimum 3 peers; maximum 5 peers (closest comparable companies)

**Example (from user's Multiples Model illustration):**

```
Symbol   Company Name              Current Price   EPS     P/E Ratio
KRYAY    Kerry Group plc           £79.84          £3.84   20.79
SJM      J. M. Smucker Company     £106.08         -£11.57 9.17
PPC      Pilgrim's Pride           £36.23          £4.54   7.98
BRRLY    Barry Callebaut AG        £17.55          £0.34   51.62
SFD      Smithfield Foods          £23.42          £0.00   0.00

                            Average P/E:    17.91
                            Median P/E:     9.17
                            Benchmark P/E:  9.17  ← Smallest (most conservative)

Target Company EPS:  £0.87
P/E Intrinsic Value: 9.17 × £0.87 = £7.98
```

*Note: Companies with negative EPS or zero P/E are excluded from the peer average/median calculation. Companies with P/E > 40 (outliers) are also excluded from the median to prevent distortion.*

### Multiple 2: Price-to-Book (P/B) Model

**Calculation:**

```
10-Year Average P/B = Average of annual P/B ratios over 10 years

Peer Median P/B = Median P/B ratio across 3–5 industry peers

Benchmark P/B = Smallest of:
  - 10-Year Average P/B
  - Peer Median P/B

P/B Intrinsic Value = Benchmark P/B × Current Book Value Per Share
```

### Multiple 3: Price-to-Sales (P/S) Model

**Calculation:**

```
10-Year Average P/S = Average of annual P/S ratios over 10 years

Peer Median P/S = Median P/S ratio across 3–5 industry peers

Benchmark P/S = Smallest of:
  - 10-Year Average P/S
  - Peer Median P/S

P/S Intrinsic Value = Benchmark P/S × Current Revenue Per Share
  = Benchmark P/S × (Annual Revenue ÷ Diluted Shares Outstanding)
```

### Multiple Peer Comparison Table (Per Company Output)

```
═══════════════════════════════════════════════════════════════════
MULTIPLES MODEL — [TICKER]: [COMPANY NAME]
═══════════════════════════════════════════════════════════════════

PEER COMPARISON TABLE:
Ticker   Company           Price    EPS     P/E   BV/Sh  P/B   Rev/Sh  P/S
──────   ───────────────   ──────   ──────  ────  ─────  ───   ──────  ───
PEER1    Peer Company 1    $xx.xx   $x.xx   x.xx  $x.xx  x.xx  $x.xx  x.xx
PEER2    Peer Company 2    $xx.xx   $x.xx   x.xx  $x.xx  x.xx  $x.xx  x.xx
PEER3    Peer Company 3    $xx.xx   $x.xx   x.xx  $x.xx  x.xx  $x.xx  x.xx
TARGET   [Company Name]    $xx.xx   $x.xx   x.xx  $x.xx  x.xx  $x.xx  x.xx

                           Avg P/E  x.xx   Avg P/B x.xx  Avg P/S x.xx
                           Med P/E  x.xx   Med P/B x.xx  Med P/S x.xx
                           10Y P/E  x.xx   10Y P/B x.xx  10Y P/S x.xx
                           Bench    x.xx   Bench   x.xx  Bench   x.xx ← Smallest

P/E Intrinsic Value:  $xx.xx  (Benchmark P/E × EPS)
P/B Intrinsic Value:  $xx.xx  (Benchmark P/B × Book Value/Share)
P/S Intrinsic Value:  $xx.xx  (Benchmark P/S × Revenue/Share)
═══════════════════════════════════════════════════════════════════
```

---

## Buffett Valuation Techniques

### Buffett Technique 1: Owner's Earnings Valuation

**Concept:** Buffett uses "Owner's Earnings" rather than reported FCF, which he defines as earnings that can be taken out of the business without impairing its competitive position. It adjusts FCF for maintenance capex vs. growth capex.

**Owner's Earnings Calculation:**

```
Owner's Earnings =
  Net Income
  + Depreciation & Amortisation
  - Maintenance Capital Expenditure
  +/- Changes in Working Capital (normalised)

Where:
  Maintenance Capex ≈ Depreciation × 0.75
  (Assumes 75% of depreciation is maintenance; 25% is growth)
  Adjust if company explicitly discloses maintenance capex split.
```

**Owner's Earnings Intrinsic Value:**

```
Owner's Earnings Growth Rate:
  = 10-Year OE CAGR, capped at 12%

Project Owner's Earnings for 10 Years:
  (same decay schedule as DCF FCF projection)

Discount at WACC (same as DCF)

Terminal Value at 2.5% perpetual growth (same as DCF)

OE Intrinsic Value Per Share:
  = [Σ PV(OE Year 1..10) + PV(Terminal Value) - Debt + Cash]
    ÷ Diluted Shares Outstanding
```

### Buffett Technique 2: Perpetual Bond Yield Comparison

**Concept:** Buffett famously compares stocks to perpetual bonds. The "bond yield" of a stock is its **Earnings Yield** (inverse of P/E ratio). A stock is attractive when its earnings yield significantly exceeds the 10-year Treasury yield.

**Earnings Yield Calculation:**

```
Earnings Yield = EPS (TTM) ÷ Current Stock Price
               = 1 ÷ Current P/E Ratio

Comparison:
  IF Earnings Yield > 10-Year Treasury Yield × 1.5:
     → ATTRACTIVE (stock offers premium over "risk-free" bond)
  IF Earnings Yield > 10-Year Treasury Yield:
     → ACCEPTABLE (stock offers some premium)
  IF Earnings Yield ≤ 10-Year Treasury Yield:
     → UNATTRACTIVE (bond pays as well or better)

Buffett Fair Value (Bond Yield Method):
  = EPS (TTM) ÷ (10-Year Treasury Yield × 1.5)
  ← Price at which earnings yield = 1.5× treasury yield
```

**Example:**
```
EPS:                  $5.20
Current Price:        $78.00
Earnings Yield:       5.20 ÷ 78.00 = 6.67%
10-Year Treasury:     4.20%
1.5× Treasury:        6.30%

Earnings Yield (6.67%) > 1.5× Treasury (6.30%) → ATTRACTIVE ✓

Buffett Bond Fair Value:
  = $5.20 ÷ (4.20% × 1.5)
  = $5.20 ÷ 6.30%
  = $82.54
```

---

## Munger Valuation Techniques

### Munger Technique 1: Quality-Based Business Assessment

Munger starts all valuations with a **quality checklist**. He believes that the value of a great business is almost impossible to overpay for in the long run, while the apparent cheapness of a mediocre business is a trap.

**Quality Checklist Scoring (1–10 Scale):**

| Quality Dimension | Description | Score (1–10) |
|-------------------|-------------|--------------|
| **Business Model Simplicity** | How easy is it to understand how the company makes money? | _/10 |
| **Competitive Moat Strength** | How durable and defensible is the competitive advantage? | _/10 |
| **Pricing Power** | Can the company raise prices without losing customers? | _/10 |
| **Revenue Predictability** | How consistent and recurring is the revenue stream? | _/10 |
| **Management Integrity** | How honest and shareholder-aligned is management? | _/10 |
| **Capital Allocation** | How wisely does management deploy retained earnings? | _/10 |
| **Customer Loyalty** | Are customers locked in or deeply habituated? | _/10 |
| **Margin Durability** | Are margins structurally protected, not cyclically dependent? | _/10 |
| **Scalability** | Can the business grow without proportional capex increases? | _/10 |
| **Regulatory Risk** | How low is exposure to adverse regulatory changes? | _/10 |

```
Munger Quality Score = Sum of all 10 dimensions ÷ 10
                     = Average score on 1–10 scale

Interpretation:
  9.0–10.0 → Exceptional business (Berkshire-quality)
  7.5–8.9  → Excellent business (strong buy candidate)
  6.0–7.4  → Good business (fair value or better required)
  4.0–5.9  → Average business (deep discount required)
  < 4.0    → Poor business (avoid regardless of price)
```

*Note: Since all Stage 5.4 companies have already passed hard filters and scored ≥ 75 composite, they are expected to score ≥ 6.0 on the Munger Quality Scale. Companies scoring below 6.0 after this assessment are flagged for review.*

### Munger Technique 2: Quality Multiple Assignment

Based on the Munger Quality Score, a **fair P/E multiple** is assigned. The key insight is that a higher-quality business deserves to be valued at a higher P/E multiple because its earnings are more reliable, durable, and growing.

**Quality Multiple Table:**

| Munger Quality Score | Assigned Fair P/E Multiple | Rationale |
|---------------------|--------------------------|-----------|
| 9.0–10.0 | 25–30× earnings | Exceptional quality commands premium |
| 8.0–8.9  | 20–25× earnings | Excellent business, fair premium |
| 7.0–7.9  | 15–20× earnings | Good business, market-rate multiple |
| 6.0–6.9  | 12–15× earnings | Above-average, discount warranted |
| 5.0–5.9  | 10–12× earnings | Average business, value pricing |

```
Munger Fair Multiple = Lower bound of range + (Quality Score - Lower Score Bound) × Step

Example:
  Quality Score: 7.6
  Range: 15–20× (for 7.0–7.9)
  Step: (20 - 15) ÷ (7.9 - 7.0) = 5.55 per point
  Multiple: 15 + (7.6 - 7.0) × 5.55 = 15 + 3.33 = 18.3×

Munger Intrinsic Value Per Share:
  = Munger Fair Multiple × Normalised EPS (10-Year Average EPS)
  ← Uses normalised EPS to avoid cyclical distortions
```

### Munger Technique 3: Five Stock Test

**Concept:** Munger's "concentration test" — if you could only hold 5 stocks forever, would you choose this one? This forces a high-bar qualitative assessment.

**Five Stock Test Questions:**

| # | Question | Pass Criteria |
|---|----------|--------------|
| 1 | Is this one of the 5 best businesses you know? | Business quality in top 10% of screened universe |
| 2 | Would you own it for 20+ years without monitoring? | Revenue and earnings predictable without active management |
| 3 | Is the competitive advantage growing or shrinking? | Moat has widened or held steady over past decade |
| 4 | Would a business school write a case study on this moat? | Moat is clear, definable, and defensible to an outsider |
| 5 | Would you own 20% of this entire business? | If you owned 20%, you'd be satisfied with the economics |

```
Five Stock Test Result:
  5/5 PASS → "Munger Conviction Buy" — high confidence valuation
  4/5 PASS → "Strong Candidate" — proceed with standard margin of safety
  3/5 PASS → "Consider" — require larger margin of safety (add 10% to threshold)
  ≤2/5 PASS → "Caution" — flag for review; may not belong in final results
```

### Munger Valuation Grade

The combination of Quality Score + Five Stock Test result produces a **Munger Valuation Grade**:

| Valuation Grade | Quality Score | Five Stock Test | Interpretation |
|----------------|--------------|-----------------|----------------|
| **A+** | ≥ 8.5 | 5/5 | Exceptional — buy confidently at margin of safety |
| **A** | ≥ 8.0 | 4–5/5 | Excellent — strong buy at margin of safety |
| **B+** | ≥ 7.5 | 4–5/5 | Very good — buy at standard margin of safety |
| **B** | ≥ 7.0 | 3–4/5 | Good — require 30%+ margin of safety |
| **B-** | ≥ 6.5 | 3/5 | Above average — require 40%+ margin of safety |
| **C+** | ≥ 6.0 | 3/5 | Acceptable — monitoring only unless deep discount |
| **F** | < 6.0 | ≤2/5 | Below standard — not recommended regardless of price |

---

## Consensus Intrinsic Value Calculation

### All Valuation Methods Summary

| Method | Code | When Available |
|--------|------|----------------|
| DCF (Free Cash Flow) | DCF | Always (if ≥ 5 years FCF data) |
| DDM Standard | DDM | Dividend yield ≥ 1% |
| DDM Buyback-Adjusted | DDM-B | Buybacks exist, no/minimal dividend |
| P/E Multiple | PE | Always (if positive EPS and peers available) |
| P/B Multiple | PB | Always (if positive book value and peers available) |
| P/S Multiple | PS | Always (if revenue data and peers available) |
| Buffett Owner's Earnings | BOE | Always (if ≥ 5 years data) |
| Buffett Bond Yield | BBY | Always (if positive EPS) |
| Munger Quality Multiple | MQM | Always (requires quality scoring) |

### Consensus Calculation

The consensus intrinsic value is the **straight average of all available estimates**. Each method receives equal weight so that no single method dominates:

```
Consensus Intrinsic Value =
  Sum of all available intrinsic value estimates
  ÷ Number of available methods

Example (all 9 methods available):
  DCF:   $125.40
  DDM:   $112.80
  PE:    $ 98.50
  PB:    $105.20
  PS:    $ 95.80
  BOE:   $130.10
  BBY:   $118.70
  MQM:   $108.40
  ──────────────
  Average: $124.88 (total ÷ 8 available methods)
  ← Note: DDM-B not used here as standard DDM was applicable

Consensus = $124.88
Current Price = $108.50
Margin of Safety = ($124.88 - $108.50) ÷ $124.88 = 13.1%
```

*Note: If fewer than 3 valuation methods are available, a warning is included in the output indicating limited valuation confidence.*

---

## Sensitivity Analysis

### Purpose

Valuation is inherently uncertain. Small changes in key assumptions (discount rate, growth rate, quality assessment) can produce meaningfully different intrinsic value estimates. The sensitivity analysis shows the **range of possible outcomes** and helps the investor understand which assumptions most significantly impact the valuation.

### Discount Rate (WACC) Sensitivity — DCF & DDM

For the DCF and DDM valuations, the intrinsic value is recalculated at five WACC levels: base case ±2% in 1% increments.

```
WACC Sensitivity Table (DCF):

WACC Applied    Intrinsic Value     vs. Base Case
─────────────   ───────────────     ─────────────
  WACC - 2%     $xxx.xx             +xx%  (higher value at lower discount)
  WACC - 1%     $xxx.xx             +xx%
  WACC (base)   $xxx.xx              —
  WACC + 1%     $xxx.xx             -xx%
  WACC + 2%     $xxx.xx             -xx%  (lower value at higher discount)

Example (base WACC = 8.1%):
  6.1%  →  $148.20  (+18.4%)
  7.1%  →  $133.50  (+6.7%)
  8.1%  →  $125.10   (base)
  9.1%  →  $113.80  (-9.0%)
 10.1%  →  $103.60  (-17.2%)
```

### Terminal Growth Rate Sensitivity — DCF & DDM

```
Terminal Growth Rate Sensitivity Table (DCF):

Terminal Rate    Intrinsic Value     vs. Base Case
─────────────   ───────────────     ─────────────
  1.5%           $xxx.xx             -xx%
  2.0%           $xxx.xx             -xx%
  2.5% (base)    $xxx.xx              —
  3.0%           $xxx.xx             +xx%
  3.5%           $xxx.xx             +xx%

Example (base g = 2.5%, WACC = 8.1%):
  1.5%  →  $112.40  (-10.1%)
  2.0%  →  $118.50  (-5.3%)
  2.5%  →  $125.10   (base)
  3.0%  →  $132.60  (+6.0%)
  3.5%  →  $141.30  (+12.9%)
```

### Quality Multiple Sensitivity — Munger Model

```
Quality Multiple Sensitivity Table:

Multiple Applied    Intrinsic Value     Scenario
────────────────    ───────────────     ─────────────────
  Base - 2×          $xxx.xx           Pessimistic (market re-rates lower)
  Base - 1×          $xxx.xx           Conservative
  Base Multiple       $xxx.xx          Base case
  Base + 1×          $xxx.xx           Optimistic
  Base + 2×          $xxx.xx           Bull case (market re-rates higher)

Example (base Munger multiple = 18.3×, normalised EPS = $4.80):
  16.3×  →  $78.24   (-20.4%)
  17.3×  →  $83.04   (-15.5%)
  18.3×  →  $87.84    (base)
  19.3×  →  $92.64   (+5.5%)
  20.3×  →  $97.44   (+10.9%)
```

### Sensitivity Summary Card

Each company's output includes a condensed sensitivity summary:

```
SENSITIVITY ANALYSIS — [TICKER]
══════════════════════════════════════════════════════
                     PESSIMISTIC    BASE     OPTIMISTIC
                     (WACC +2%,     (Base)   (WACC -2%,
                      g = 1.5%)             g = 3.5%)
─────────────────    ───────────    ──────   ──────────
DCF Intrinsic Value   $xxx.xx       $xxx.xx   $xxx.xx
DDM Intrinsic Value   $xxx.xx       $xxx.xx   $xxx.xx
Munger Quality Val    $xxx.xx       $xxx.xx   $xxx.xx
──────────────────    ───────────   ──────    ──────────
Consensus Range:      $xxx–$xxx     $xxx.xx   $xxx–$xxx

Current Price:        $xxx.xx
MOS at Pessimistic:    xx.x%
MOS at Base Case:      xx.x%
MOS at Optimistic:     xx.x%
══════════════════════════════════════════════════════
```

---

## Margin of Safety Analysis

### Overview

The margin of safety (MOS) is the percentage discount from intrinsic value at which the stock currently trades. A higher MOS means a larger cushion against valuation errors, economic downturns, or unexpected business deterioration.

```
Margin of Safety % =
  (Consensus Intrinsic Value - Current Market Price)
  ───────────────────────────────────────────────────  × 100
            Consensus Intrinsic Value

Positive MOS = Stock is BELOW intrinsic value (undervalued)
Negative MOS = Stock is ABOVE intrinsic value (overvalued)
```

### Target Buy Prices at Four Safety Levels

For each company, four target buy prices are calculated — one for each margin of safety threshold:

```
Target Buy Price (20% MOS) = Consensus Intrinsic Value × (1 - 0.20)
Target Buy Price (30% MOS) = Consensus Intrinsic Value × (1 - 0.30)
Target Buy Price (40% MOS) = Consensus Intrinsic Value × (1 - 0.40)
Target Buy Price (50% MOS) = Consensus Intrinsic Value × (1 - 0.50)
```

### Per-Method Margin of Safety

In addition to the consensus MOS, MOS is calculated separately against each valuation method's estimate:

```
MOS vs. DCF          = (DCF Value - Price) ÷ DCF Value × 100
MOS vs. DDM          = (DDM Value - Price) ÷ DDM Value × 100
MOS vs. P/E Multiple = (P/E Value - Price) ÷ P/E Value × 100
MOS vs. P/B Multiple = (P/B Value - Price) ÷ P/B Value × 100
MOS vs. P/S Multiple = (P/S Value - Price) ÷ P/S Value × 100
MOS vs. Buffett OE   = (BOE Value - Price) ÷ BOE Value × 100
MOS vs. Buffett Bond = (BBY Value - Price) ÷ BBY Value × 100
MOS vs. Munger QM    = (MQM Value - Price) ÷ MQM Value × 100
```

### Margin of Safety Display Block

```
MARGIN OF SAFETY ANALYSIS — [TICKER]: [COMPANY NAME]
══════════════════════════════════════════════════════════════════════
  Current Market Price:          $xxx.xx

  INTRINSIC VALUE ESTIMATES:
  ├─ DCF (Free Cash Flow):        $xxx.xx    MOS: xx.x%
  ├─ DDM (Dividend/Buyback):      $xxx.xx    MOS: xx.x%  [or N/A]
  ├─ P/E Multiple:                $xxx.xx    MOS: xx.x%
  ├─ P/B Multiple:                $xxx.xx    MOS: xx.x%
  ├─ P/S Multiple:                $xxx.xx    MOS: xx.x%
  ├─ Buffett Owner's Earnings:    $xxx.xx    MOS: xx.x%
  ├─ Buffett Bond Yield:          $xxx.xx    MOS: xx.x%
  └─ Munger Quality Multiple:     $xxx.xx    MOS: xx.x%

  ══════════════════════════════════════════════════════
  CONSENSUS INTRINSIC VALUE:      $xxx.xx
  CURRENT MARGIN OF SAFETY:        xx.x%
  ══════════════════════════════════════════════════════

  TARGET BUY PRICES:
  ├─ 20% Margin of Safety:        $xxx.xx  ← Minimum acceptable entry
  ├─ 30% Margin of Safety:        $xxx.xx  ← Good value entry
  ├─ 40% Margin of Safety:        $xxx.xx  ← Strong value entry
  └─ 50% Margin of Safety:        $xxx.xx  ← Deep value / distressed entry

  MUNGER VALUATION GRADE:  [A+ / A / B+ / B / B- / C+ / F]
  EARNINGS YIELD:           x.xx%  vs. Treasury x.xx%  → [ATTRACTIVE/ACCEPTABLE/UNATTRACTIVE]
══════════════════════════════════════════════════════════════════════
```

---

## Company Grouping for Display

After all valuations are calculated, companies are sorted into five groups based on their **Financial Quality Score** (from Stage 4, 0–100 scale) and **current Margin of Safety** (from consensus intrinsic value).

> **Score Clarification:** Two scores are used in grouping:
> - **Financial Quality Score** — Stage 4 output (0–100), measures business quality across Return on Capital, Profitability, Cash Flow, and Business Quality pillars.
> - **Composite Score** — Stage 5.3 output (weighted combination of Quality, Valuation, and Management scores). All companies entering Stage 5.4 have already passed the Composite Score ≥ 75 gate.
>
> Group thresholds below use the **Financial Quality Score** (Stage 4), not the Composite Score.

### Group Definitions

Groups are assigned using a **hierarchical (first-match) evaluation**. Each company is assessed in order from Group 1 to Group 5, and assigned to the **first group whose criteria it satisfies**. This prevents any overlap between groups.

| Priority | Group | Name | Financial Quality Score | MOS Threshold | Intent |
|----------|-------|------|------------------------|----------------|--------|
| 1st | **Group 1** | 🔥 SLAM DUNK BUYS | ≥ 80 | ≥ 30% | Best quality + deepest discount |
| 2nd | **Group 2** | ✅ STRONG BUYS | ≥ 75 | 20–29% | High quality at good price |
| 3rd | **Group 3** | 🟡 FAIR VALUE | ≥ 82 | 10–19% | Premium quality near fair value |
| 4th | **Group 4** | ⚠️ OVERVALUED | ≥ 80 | < 0% | Quality businesses priced above intrinsic value |
| 5th | **Group 5** | 📋 NOT QUALIFIED | Any | Any | Did not match Groups 1–4 |

**Hierarchical assignment logic:**
```
IF   Quality ≥ 80  AND  MOS ≥ 30%      → Group 1: SLAM DUNK BUY
ELIF Quality ≥ 75  AND  20% ≤ MOS < 30% → Group 2: STRONG BUY
ELIF Quality ≥ 82  AND  10% ≤ MOS < 20% → Group 3: FAIR VALUE
ELIF Quality ≥ 80  AND  MOS < 0%        → Group 4: OVERVALUED
ELSE                                     → Group 5: NOT QUALIFIED
```

*Note: Group 5 catches all companies that do not meet the specific quality+MOS combination of Groups 1–4. This includes companies with acceptable scores but MOS in the 0–9% range (not yet a buy), as well as companies with quality scores below the thresholds of Groups 1–4.*

### Within-Group Ranking

Within each group, companies are ranked by **Financial Quality Score (highest first)**. If two companies have the same score, they are ranked by **Current MOS% (highest first)**.

### Group Display Logic

```
DISPLAY ORDER:
  Group 1: All SLAM DUNK BUYS (sorted by Financial Quality Score DESC)
  Group 2: All STRONG BUYS (sorted by Financial Quality Score DESC)
  Group 3: All FAIR VALUE (sorted by Financial Quality Score DESC)
  Group 4: All OVERVALUED (sorted by Financial Quality Score DESC)
  Group 5: NOT QUALIFIED (sorted by Financial Quality Score DESC)

  Within ties in Financial Quality Score: Sort by MOS% DESC
```

---

## Final Results Display

### Email Report Format

The email report is designed to be readable on any device. Each company gets a **Brief Format Card**, with all groups presented in display order.

**Email Header (Summary Statistics):**

```
══════════════════════════════════════════════════════════════
📊 STOCK MARKET SCREENER — QUARTERLY VALUATION REPORT
  Run Date: [DATE]    Qualified Companies: [N]
══════════════════════════════════════════════════════════════
  🔥 SLAM DUNK BUYS:    [N] companies
  ✅ STRONG BUYS:       [N] companies
  🟡 FAIR VALUE:        [N] companies
  ⚠️  OVERVALUED:        [N] companies
  📋 NOT QUALIFIED:     [N] companies (monitoring)
══════════════════════════════════════════════════════════════
```

**Email Brief Format Card (per company):**

```
─────────────────────────────────────────────────────────────
[TICKER] — [COMPANY NAME]                    [GROUP LABEL]
─────────────────────────────────────────────────────────────
Price: $xxx.xx   Quality Score: xx/100   Munger Grade: [X]

KEY FINANCIALS:
  ROE: xx.x%    Net Margin: xx.x%    FCF Margin: xx.x%
  D/E: x.xx     Current Ratio: x.xx  Revenue CAGR: xx.x%

INTRINSIC VALUES:
  DCF: $xxx.xx  DDM: $xxx.xx  P/E: $xxx.xx
  BOE: $xxx.xx  MQM: $xxx.xx

CONSENSUS VALUE: $xxx.xx     CURRENT MOS: xx.x%

TARGET BUY PRICES:
  20% MOS: $xxx.xx  |  30% MOS: $xxx.xx
  40% MOS: $xxx.xx  |  50% MOS: $xxx.xx
─────────────────────────────────────────────────────────────
```

### GitHub Summary Page Format

The GitHub summary page uses markdown formatting and includes the **Detailed Format Card** with full percentages, all valuations, and all recommendations.

**GitHub Header:**

```markdown
# 📊 Stock Market Screener — Quarterly Valuation Report
**Run Date:** YYYY-MM-DD | **Qualified Companies:** N

## Summary
| Group | Count | Description |
|-------|-------|-------------|
| 🔥 Slam Dunk Buys | N | Quality ≥ 80, MOS ≥ 30% |
| ✅ Strong Buys | N | Quality 75–79, MOS 20–29% |
| 🟡 Fair Value | N | Quality ≥ 82, MOS 10–19% |
| ⚠️ Overvalued | N | Quality ≥ 80, MOS < 0% |
| 📋 Not Qualified | N | Score < 75, monitoring only |
```

**GitHub Detailed Format Card (per company):**

```markdown
## 🔥 [TICKER] — [COMPANY NAME]
**Sector:** [SECTOR] | **Industry:** [INDUSTRY]
**Quality Score:** xx/100 | **Munger Grade:** X | **Five Stock Test:** x/5

### 📈 Key Financial Metrics
| Metric | Value | 10Y Avg | Trend |
|--------|-------|---------|-------|
| ROE | xx.x% | xx.x% | ↑/↓/→ |
| Net Margin | xx.x% | xx.x% | ↑/↓/→ |
| Operating Margin | xx.x% | xx.x% | ↑/↓/→ |
| FCF Margin | xx.x% | xx.x% | ↑/↓/→ |
| Revenue CAGR (10Y) | xx.x% | — | — |
| EPS CAGR (10Y) | xx.x% | — | — |
| FCF CAGR (10Y) | xx.x% | — | — |
| D/E Ratio | x.xx | — | ↑/↓/→ |
| Current Ratio | x.xx | — | — |
| Insider Ownership | xx.x% | — | — |

### 💰 Valuation Analysis
| Method | Intrinsic Value | MOS vs. Current Price |
|--------|----------------|----------------------|
| DCF (Free Cash Flow) | $xxx.xx | xx.x% |
| DDM (Dividend/Buyback) | $xxx.xx | xx.x% |
| P/E Multiple | $xxx.xx | xx.x% |
| P/B Multiple | $xxx.xx | xx.x% |
| P/S Multiple | $xxx.xx | xx.x% |
| Buffett Owner's Earnings | $xxx.xx | xx.x% |
| Buffett Bond Yield | $xxx.xx | xx.x% |
| Munger Quality Multiple | $xxx.xx | xx.x% |
| **Consensus** | **$xxx.xx** | **xx.x%** |

**Current Price:** $xxx.xx

### 🎯 Target Buy Prices
| Margin of Safety | Target Buy Price |
|-----------------|-----------------|
| 20% MOS | $xxx.xx |
| 30% MOS | $xxx.xx |
| 40% MOS | $xxx.xx |
| 50% MOS | $xxx.xx |

### 📊 Sensitivity Analysis
| Scenario | DCF Value | DDM Value | Munger Value |
|----------|-----------|-----------|--------------|
| Pessimistic (WACC+2%, g=1.5%) | $xxx.xx | $xxx.xx | $xxx.xx |
| Base Case | $xxx.xx | $xxx.xx | $xxx.xx |
| Optimistic (WACC-2%, g=3.5%) | $xxx.xx | $xxx.xx | $xxx.xx |

### 🔍 Munger Quality Assessment
| Dimension | Score (1–10) |
|-----------|-------------|
| Business Model Simplicity | x |
| Competitive Moat Strength | x |
| Pricing Power | x |
| Revenue Predictability | x |
| Management Integrity | x |
| Capital Allocation | x |
| Customer Loyalty | x |
| Margin Durability | x |
| Scalability | x |
| Regulatory Risk | x |
| **Overall Quality Score** | **x.x / 10** |

**Five Stock Test:** x/5 PASS — [RESULT LABEL]
**Earnings Yield:** x.xx% vs. Treasury x.xx% → [ATTRACTIVE/ACCEPTABLE/UNATTRACTIVE]
**Recommendation:** [SLAM DUNK BUY / STRONG BUY / FAIR VALUE / OVERVALUED / MONITOR]
```

### CSV Export Format

The CSV export contains all 40+ screening metrics plus all valuation outputs for every qualified company. Columns are structured for easy analysis in Excel or Google Sheets.

**CSV Column Structure:**

```
GROUP 1 — IDENTIFICATION
  Ticker, Company Name, Sector, Industry, Market Cap, Run Date

GROUP 2 — FINANCIAL METRICS (40+ columns)
  ROE_10Y_Avg, ROE_Trend, Net_Margin_10Y_Avg, Op_Margin_10Y_Avg,
  Margin_Stability_CV, DE_Ratio, DE_Trend, Current_Ratio,
  OCF_Positive_Years, OCF_Growth_CAGR, FCF_Positive_Years,
  FCF_Growth_CAGR, Revenue_Growth_Avg, Business_Simplicity,
  ROA_10Y, ROIC_10Y, FCF_Margin, EPS_CAGR, BV_CAGR,
  Dividend_Yield, Buyback_Yield, Payout_Ratio, Insider_Ownership,
  Goodwill_Pct_Assets, Accounting_Quality_Flag,
  [all remaining 40+ metrics...]

GROUP 3 — SCREENING SCORES
  Quality_Score, Valuation_Score, Management_Score, Composite_Score

GROUP 4 — VALUATION RESULTS
  DCF_Intrinsic_Value, DDM_Intrinsic_Value, PE_Multiple_Value,
  PB_Multiple_Value, PS_Multiple_Value, Buffett_OE_Value,
  Buffett_Bond_Value, Munger_QM_Value, Consensus_Value,
  Current_Price, Current_MOS_Pct

GROUP 5 — TARGET BUY PRICES
  Target_20pct_MOS, Target_30pct_MOS, Target_40pct_MOS, Target_50pct_MOS

GROUP 6 — SENSITIVITY RANGES
  DCF_Pessimistic, DCF_Base, DCF_Optimistic,
  DDM_Pessimistic, DDM_Base, DDM_Optimistic,
  Munger_Pessimistic, Munger_Base, Munger_Optimistic

GROUP 7 — MUNGER ASSESSMENT
  Munger_Quality_Score, Munger_Grade, Five_Stock_Test_Score,
  Earnings_Yield, Bond_Yield_Comparison,
  Quality_Dim_1..10 (10 individual quality dimension scores)

GROUP 8 — CLASSIFICATION
  Display_Group, Display_Group_Label, Group_Rank
```

---

## Database Schema

### Table: CompanyValuations

Stores all valuation results for each qualified company per screening run.

```sql
CREATE TABLE CompanyValuations (
    Id                      INT PRIMARY KEY AUTO_INCREMENT,
    Ticker                  VARCHAR(10)    NOT NULL,
    CompanyName             VARCHAR(200)   NOT NULL,
    ScreeningRunId          INT            NOT NULL,  -- FK to ScreeningRuns
    RunDate                 DATE           NOT NULL,

    -- Current Market Data
    CurrentPrice            DECIMAL(10,2)  NOT NULL,
    MarketCap               BIGINT,
    DilutedShares           BIGINT,

    -- WACC Components
    WACC                    DECIMAL(6,4),
    CostOfEquity            DECIMAL(6,4),
    CostOfDebt              DECIMAL(6,4),
    TreasuryYield10Y        DECIMAL(6,4),
    Beta                    DECIMAL(6,4),

    -- DCF Valuation
    DCF_GrowthRate          DECIMAL(6,4),
    DCF_TerminalGrowthRate  DECIMAL(6,4)  DEFAULT 0.025,
    DCF_IntrinsicValue      DECIMAL(10,2),
    DCF_MOS_Pct             DECIMAL(6,4),

    -- DDM Valuation
    DDM_Applicable          BOOLEAN        DEFAULT FALSE,
    DDM_Type                VARCHAR(20),   -- 'STANDARD', 'BUYBACK_ADJUSTED', 'N/A'
    DDM_DividendGrowthRate  DECIMAL(6,4),
    DDM_IntrinsicValue      DECIMAL(10,2),
    DDM_MOS_Pct             DECIMAL(6,4),

    -- Multiples Valuation
    PE_BenchmarkMultiple    DECIMAL(6,2),
    PE_IntrinsicValue       DECIMAL(10,2),
    PE_MOS_Pct              DECIMAL(6,4),
    PB_BenchmarkMultiple    DECIMAL(6,2),
    PB_IntrinsicValue       DECIMAL(10,2),
    PB_MOS_Pct              DECIMAL(6,4),
    PS_BenchmarkMultiple    DECIMAL(6,2),
    PS_IntrinsicValue       DECIMAL(10,2),
    PS_MOS_Pct              DECIMAL(6,4),

    -- Buffett Valuations
    OE_GrowthRate           DECIMAL(6,4),
    OE_IntrinsicValue       DECIMAL(10,2),
    OE_MOS_Pct              DECIMAL(6,4),
    BondYield_IntrinsicValue DECIMAL(10,2),
    BondYield_MOS_Pct       DECIMAL(6,4),
    EarningsYield           DECIMAL(6,4),
    BondYieldComparison     VARCHAR(20),   -- 'ATTRACTIVE', 'ACCEPTABLE', 'UNATTRACTIVE'

    -- Munger Assessment
    MungerQualityScore      DECIMAL(4,2),
    MungerGrade             CHAR(2),       -- 'A+', 'A', 'B+', 'B', 'B-', 'C+', 'F'
    FiveStockTestScore      TINYINT,       -- 0 to 5
    MungerMultiple          DECIMAL(5,2),
    MungerIntrinsicValue    DECIMAL(10,2),
    MungerMOS_Pct           DECIMAL(6,4),
    QualityDim_1            TINYINT,       -- Business Model Simplicity (1-10)
    QualityDim_2            TINYINT,       -- Competitive Moat Strength
    QualityDim_3            TINYINT,       -- Pricing Power
    QualityDim_4            TINYINT,       -- Revenue Predictability
    QualityDim_5            TINYINT,       -- Management Integrity
    QualityDim_6            TINYINT,       -- Capital Allocation
    QualityDim_7            TINYINT,       -- Customer Loyalty
    QualityDim_8            TINYINT,       -- Margin Durability
    QualityDim_9            TINYINT,       -- Scalability
    QualityDim_10           TINYINT,       -- Regulatory Risk

    -- Consensus & Targets
    ConsensusIntrinsicValue DECIMAL(10,2),
    CurrentMOS_Pct          DECIMAL(6,4),
    Target_20pct_MOS        DECIMAL(10,2),
    Target_30pct_MOS        DECIMAL(10,2),
    Target_40pct_MOS        DECIMAL(10,2),
    Target_50pct_MOS        DECIMAL(10,2),

    -- Sensitivity Ranges
    DCF_Pessimistic         DECIMAL(10,2),
    DCF_Base                DECIMAL(10,2),
    DCF_Optimistic          DECIMAL(10,2),
    DDM_Pessimistic         DECIMAL(10,2),
    DDM_Base                DECIMAL(10,2),
    DDM_Optimistic          DECIMAL(10,2),
    Munger_Pessimistic      DECIMAL(10,2),
    Munger_Base             DECIMAL(10,2),
    Munger_Optimistic       DECIMAL(10,2),

    -- Display Grouping
    DisplayGroup            TINYINT,        -- 1='SLAM DUNK', 2='STRONG BUY', etc.
    DisplayGroupLabel       VARCHAR(30),
    GroupRank               INT,            -- Rank within group

    -- Audit
    CreatedAt               DATETIME       DEFAULT CURRENT_TIMESTAMP,
    UpdatedAt               DATETIME       ON UPDATE CURRENT_TIMESTAMP,

    -- Relationships
    FOREIGN KEY (ScreeningRunId) REFERENCES ScreeningRuns(Id),
    INDEX idx_ticker_date (Ticker, RunDate),
    INDEX idx_display_group (DisplayGroup, GroupRank)
);
```

### Table: HistoricalTracking

Stores point-in-time snapshots for trend analysis and performance tracking over time.

```sql
CREATE TABLE HistoricalTracking (
    Id                      INT PRIMARY KEY AUTO_INCREMENT,
    Ticker                  VARCHAR(10)    NOT NULL,
    CompanyName             VARCHAR(200)   NOT NULL,
    TrackingDate            DATE           NOT NULL,

    -- Price & Valuation Snapshot
    Price                   DECIMAL(10,2)  NOT NULL,
    ConsensusIntrinsicValue DECIMAL(10,2),
    CurrentMOS_Pct          DECIMAL(6,4),
    MungerGrade             CHAR(2),
    DisplayGroup            TINYINT,

    -- Performance Since First Qualification
    FirstQualifiedDate      DATE,
    PriceSinceQualification DECIMAL(6,4),  -- % price change since first qualified

    -- Screening Run Reference
    ScreeningRunId          INT,
    QualificationStatus     VARCHAR(20),   -- 'NEW', 'MAINTAINED', 'REMOVED'

    CreatedAt               DATETIME       DEFAULT CURRENT_TIMESTAMP,

    INDEX idx_ticker_date (Ticker, TrackingDate),
    FOREIGN KEY (ScreeningRunId) REFERENCES ScreeningRuns(Id)
);
```

### Relationship to QualifiedCompanies Table

```sql
-- QualifiedCompanies (existing table — stores screening results)
-- CompanyValuations links to this via Ticker + RunDate

-- To retrieve full company profile with valuation:
SELECT
    qc.*,
    cv.ConsensusIntrinsicValue,
    cv.CurrentMOS_Pct,
    cv.MungerGrade,
    cv.DisplayGroup,
    cv.Target_30pct_MOS
FROM QualifiedCompanies qc
JOIN CompanyValuations cv
    ON qc.Ticker = cv.Ticker
    AND qc.RunDate = cv.RunDate
ORDER BY cv.DisplayGroup ASC, cv.GroupRank ASC;
```

---

## Example Walkthrough

### Example Company: Consumer Staples — Food Industry

This example illustrates Stage 5.4 applied to a hypothetical qualified company.

```
Company: ABC Foods Inc. (ABCF)
Sector: Consumer Defensive
Industry: Packaged Foods
Current Price: $87.50
Quality Score (Stage 4): 83/100
Composite Score (Stage 5.3): 79.2
Treasury Yield (10Y): 4.2%
```

**Step 1: WACC Calculation**
```
Market Cap:     $8.2B
Total Debt:     $1.1B
Beta:           0.72
Cost of Equity: 4.2% + 0.72 × 5.5% = 8.16%
Cost of Debt:   4.8% → After-tax: 3.79%
Weight Equity:  88.2%, Weight Debt: 11.8%

WACC = (88.2% × 8.16%) + (11.8% × 3.79%)
     = 7.20% + 0.45%
     = 7.65%
```

**Step 2: DCF Intrinsic Value**
```
Latest FCF:         $420M
10-Year FCF CAGR:   9.2% → Capped at 9.2% (under 15% cap)

Year 1–5 Growth:    9.2%
Year 6–10 Growth:   Decaying from 9.2% to 4.6%

Projected FCF:
  Year 1:  $458.6M   PV: $425.9M
  Year 2:  $500.8M   PV: $432.4M
  Year 3:  $546.8M   PV: $438.8M
  Year 4:  $597.1M   PV: $445.3M
  Year 5:  $652.2M   PV: $452.0M
  Year 6:  $702.3M   PV: $451.8M
  Year 7:  $741.1M   PV: $442.5M
  Year 8:  $770.2M   PV: $427.3M
  Year 9:  $789.7M   PV: $407.3M
  Year 10: $799.5M   PV: $383.5M

Sum of PV(FCF):       $4,306.8M
Terminal Value:       $799.5M × 1.025 ÷ (0.0765 - 0.025) = $15,875.2M
PV(Terminal Value):   $7,618.9M

DCF Enterprise Value: $11,925.7M
Less: Debt:           ($1,100.0M)
Plus: Cash:           $350.0M
DCF Equity Value:     $11,175.7M
Diluted Shares:       97.2M
DCF Per Share:        $114.98 ≈ $115.00
```

**Step 3: DDM**
```
Annual DPS: $1.96  Dividend Yield: 2.24% → Standard DDM applicable
Dividend CAGR (10Y): 7.1%
Payout Ratio: 52% → Sustainable

Terminal DDM: $1.96 × 1.071^10 × 1.025 ÷ (0.0765 - 0.025)
            = $3.84 × 1.025 ÷ 0.0515
            = $76.49

DDM Per Share: $91.40
```

**Step 4: Multiples**
```
10-Year Avg P/E:     22.1×    Peer Median P/E: 18.3×  → Benchmark: 18.3×
10-Year Avg P/B:      4.2×    Peer Median P/B:  3.8×  → Benchmark:  3.8×
10-Year Avg P/S:      2.1×    Peer Median P/S:  1.9×  → Benchmark:  1.9×

Current EPS:         $4.77
Book Value/Share:    $24.10
Revenue/Share:       $48.50

P/E Value:  18.3 × $4.77  = $87.29
P/B Value:   3.8 × $24.10 = $91.58
P/S Value:   1.9 × $48.50 = $92.15
```

**Step 5: Buffett Methods**
```
Owner's Earnings:
  Net Income: $454M, D&A: $85M, Maintenance Capex: $63.75M (75% of D&A)
  OE: $454M + $85M - $63.75M = $475.25M
  Growth: 8.8% CAGR → Same decay projection
  OE Per Share: $118.20

Bond Yield Method:
  EPS: $4.77, Treasury: 4.2%
  Buffett Fair Value: $4.77 ÷ (4.2% × 1.5) = $4.77 ÷ 6.3% = $75.71
  Earnings Yield: $4.77 ÷ $87.50 = 5.45% > 4.2% → ACCEPTABLE
```

**Step 6: Munger Assessment**
```
Quality Dimensions (scored by system from financial metrics):
  Business Model Simplicity:   8   (packaged food — easy to understand)
  Competitive Moat Strength:   7   (strong brands, distribution network)
  Pricing Power:               7   (limited by commodity input costs)
  Revenue Predictability:      9   (consumer staples — highly defensive)
  Management Integrity:        8   (low goodwill, consistent accounting)
  Capital Allocation:          8   (good dividend growth + buybacks)
  Customer Loyalty:            8   (recurring household purchases)
  Margin Durability:           7   (margins can compress with inflation)
  Scalability:                 6   (physical distribution limits scale)
  Regulatory Risk:             8   (low regulatory exposure)

Munger Quality Score: (8+7+7+9+8+8+8+7+6+8) ÷ 10 = 7.6

Quality Multiple: 15 + (7.6 - 7.0) × 5.55 = 18.3×
Normalised EPS (10-Year Avg): $4.12
Munger Value: 18.3 × $4.12 = $75.40

Five Stock Test: 4/5 PASS (scalability question reduces to 4)
Munger Grade: B+ (Quality 7.5–7.9, Five Stock 4/5)
```

**Step 7: Consensus & Margin of Safety**
```
Method          Value
────────────    ───────
DCF:            $115.00
DDM:             $91.40
P/E Multiple:    $87.29
P/B Multiple:    $91.58
P/S Multiple:    $92.15
Buffett OE:     $118.20
Buffett Bond:    $75.71
Munger QM:       $75.40

Consensus:      Average of 8 = $93.34

Current Price:  $87.50
Margin of Safety: ($93.34 - $87.50) ÷ $93.34 = 6.3%

Target Buy Prices:
  20% MOS:  $93.34 × 0.80 = $74.67
  30% MOS:  $93.34 × 0.70 = $65.34
  40% MOS:  $93.34 × 0.60 = $56.00
  50% MOS:  $93.34 × 0.50 = $46.67

Display Group: Group 3 (FAIR VALUE)
  Quality Score 83 ≥ 82, MOS 6.3% is 10–19% range
```

**Final Card:**
```
══════════════════════════════════════════════════════════════════════
🟡 [ABCF] — ABC FOODS INC.                           FAIR VALUE
══════════════════════════════════════════════════════════════════════
Price: $87.50   |   Quality Score: 83/100   |   Munger Grade: B+
Sector: Consumer Defensive | Industry: Packaged Foods

KEY FINANCIALS:
  ROE: 18.4%   Net Margin: 11.2%   FCF Margin: 14.3%
  D/E: 0.27    Current Ratio: 2.1  Revenue CAGR: 7.8%

INTRINSIC VALUES:                 MOS vs. Price:
  DCF:                 $115.00    +24.4%
  DDM (Standard):       $91.40    + 4.3%
  P/E Multiple:         $87.29    - 0.2%
  P/B Multiple:         $91.58    + 4.7%
  P/S Multiple:         $92.15    + 5.3%
  Buffett OE:          $118.20    +26.0%
  Buffett Bond Yield:   $75.71    -15.5%
  Munger QM:            $75.40    -16.0%
  ─────────────────────────────────────
  CONSENSUS:            $93.34     CURRENT MOS: +6.3%

TARGET BUY PRICES:
  20% MOS: $74.67  |  30% MOS: $65.34
  40% MOS: $56.00  |  50% MOS: $46.67

SENSITIVITY (Consensus):
  Pessimistic: $81.20  |  Base: $93.34  |  Optimistic: $108.50

EARNINGS YIELD: 5.45% vs. Treasury 4.2% → ACCEPTABLE
FIVE STOCK TEST: 4/5 PASS
RECOMMENDATION: FAIR VALUE — Monitor for better entry; 
                buy at or below $74.67 (20% MOS target)
══════════════════════════════════════════════════════════════════════
```

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-03-14 | ThommCroft | Initial creation — Stage 5.4 Valuation Methodology |

---

*This document is part of the Stock Market Screener documentation suite. See also:*
- *`01-INVESTMENT-REQUIREMENTS.md` — Screening criteria and investment philosophy*
- *`02-ARCHITECTURE-DESIGN.md` — System architecture and component design*
- *`03-DATA-FLOW-SCREENING-LOGIC.md` — Complete data flow through all stages*
- *`04-PROJECT-ROADMAP.md` — Project phases and implementation timeline*
