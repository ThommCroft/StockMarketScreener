# Stock Market Screener - Basic Stage Workflow

**Version:** 1.0  
**Date:** 2026-03-15  
**Author:** ThommCroft  
**Purpose:** Define the complete end-to-end pipeline stages for the stock screening system

---

## Table of Contents

1. [Overview](#overview)
2. [Stage-by-Stage Implementation](#stage-by-stage-implementation)
3. [Related Documents](#related-documents)

---

## Overview

This document outlines the basic workflow stages that make up the complete Stock Market Screener pipeline. The system is divided into 7 distinct stages, organized into two primary analysis phases:

- **Stages 0-5:** Financial Analysis (filtering and quality assessment)
- **Stage 6:** Stock Price Valuation (intrinsic value calculation)
- **Stage 7:** Results Reporting (output and storage)

Each stage has a clear purpose, defined inputs, and specific outputs that feed into the next stage.

---

## Stage-by-Stage Implementation

### Stage 0: Financial Analysis - Market Cap Pre-Filter

**Purpose:** Remove small-cap companies that lack institutional-quality liquidity

**Actions:**
- Eliminate companies < $300M market cap (saves API calls)
- ~10-15% of universe rejected at this step
- Focus remaining effort on institutional-quality companies

**Output:** Universe of companies > $300M market cap

---

### Stage 1: Financial Analysis - Data Ingestion & Metric Calculation

**Purpose:** Gather comprehensive financial data and calculate all quantitative metrics

**Actions:**
- Fetch financial data from SEC EDGAR (authoritative)
- Fetch market data from Yahoo Finance (with fallbacks: IEX Cloud, Alpha Vantage)
- Calculate ALL 40+ metrics for every company
- Validate data quality and completeness
- Create year-by-year metrics and 10-year averages

**Output:** Complete 40+ metric dataset for each company, stored in database

---

### Stage 2: Financial Analysis - Financial Strength Assessment

**Purpose:** Evaluate financial fortress status using hard filters on complete metric data

**Actions:**
- Evaluate hard filters using complete metric data
- Assess market cap trends and liquidity positions
- Evaluate leverage and financial flexibility
- Determine financial fortress status

**Output:** Companies flagged for financial strength; exceptions documented

---

### Stage 3: Financial Analysis - Quality Scoring (0-100)

**Purpose:** Assign comprehensive quality scores based on business fundamentals

**Actions:**
- Pillar 1: Return on Capital (max 30 points)
- Pillar 2: Profitability (max 30 points)
- Pillar 3: Cash Flow Quality (max 20 points)
- Pillar 4: Business Quality (max 10 points)

**Output:** Quality Score (0-100) for each company

---

### Stage 4: Financial Analysis - Composite Scoring & Management Assessment

**Purpose:** Evaluate management quality and calculate composite ranking score

**Actions:**
- Pillar 5: Composite Scoring (max 35 points)
- Pillar 6: Management Assessment (max 25 points)
- Composite: (Quality × 40%) + (Valuation × 35%) + (Management × 25%)

**Output:** Composite Score (0-100) for each company

---

### Stage 5: Financial Analysis - Results Processing

**Purpose:** Determine which companies advance to intrinsic value valuation

**Actions:**
- PASS: Composite score >= 75 (store in database)
- FAIL: Composite score < 75 (discard)
- Compare with previous qualified companies
- Identify new, maintained, and removed companies

**Output:** Qualified candidates list for Stage 6 valuation

**Threshold:** Composite Score ≥ 75

---

### Stage 6: Stock Price - Intrinsic Value Valuation

**Purpose:** Calculate intrinsic value using five independent valuation methods

**Description:**
All companies that have passed the above Financial Analysis (Score ≥ 75) will have their intrinsic value evaluated with the following methods:

**Valuation Methods:**
- Discounted Cash Flow (DCF)
- Dividend Discount Model (DDM)
- Multiples Valuation Model
- Warren Buffett Valuation Techniques
- Charlie Munger Valuation Techniques

**Output:** Fair value estimates, margin of safety, confidence levels, valuation grade

---

### Stage 7: Results Reporting

**Purpose:** Store all results and generate comprehensive reports in multiple formats

**Actions:**
- All Financial Data and Intrinsic Value Data will be stored in a MySQL Database
- All results will be sent to me via Email
- All results will be displayed on the GitHub run summary screen
- All results will be displayed in a downloadable CSV file which is also attached to the email

**Output Formats:**
- ✓ MySQL Database storage
- ✓ Email report
- ✓ GitHub summary
- ✓ CSV download

---

## Related Documents

| Document | Covers | Purpose |
|----------|--------|---------|
| [01-INVESTMENT-REQUIREMENTS.md](01-INVESTMENT-REQUIREMENTS.md) | Stages 0-5 | Detailed investment criteria, metrics, and quality scoring |
| [02-VALUATION-METHODOLOGY.md](02-VALUATION-METHODOLOGY.md) | Stage 6 | Complete valuation methodology for intrinsic value calculation |
| Stage 7 Document (TBD) | Stage 7 | Results reporting specifications and output formats |

---

## Quick Reference: Pipeline Flow

```
Stage 0: Pre-Filter by Market Cap
    ↓
Stage 1: Ingest Data & Calculate Metrics
    ↓
Stage 2: Assess Financial Strength
    ↓
Stage 3: Quality Scoring (0-100)
    ↓
Stage 4: Composite Score & Management (0-100)
    ↓
Stage 5: Pass/Fail Gate (≥75 threshold)
    ↓
    ├─→ FAIL: Composite < 75 (discard)
    └─→ PASS: Composite ≥ 75 (advance to Stage 6)
    ↓
Stage 6: Intrinsic Value Valuation (5 methods)
    ↓
Stage 7: Store Results & Generate Reports
```

---

**Document Version History**

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-03-15 | ThommCroft | Initial basic workflow document |
