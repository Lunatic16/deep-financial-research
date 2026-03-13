# deep-financial-research

A Claude skill for institutional-grade investment research. Connects to live market data via MCP servers to deliver company deep dives, due diligence reports, competitive landscape analysis, market sentiment, and DCF-based intrinsic value estimates — all from natural language prompts.

---

## Table of Contents

- [Overview](#overview)
- [Repository Structure](#repository-structure)
- [Workflows](#workflows)
- [DCF Valuation Subskill](#dcf-valuation-subskill)
- [Example Prompts](#example-prompts)
- [Output Format](#output-format)
- [MCP Server Setup](#mcp-server-setup)
- [Installation](#installation)
- [Error Handling](#error-handling)
- [Limitations](#limitations)
- [Disclaimer](#disclaimer)

---

## Overview

This skill gives Claude access to real-time financial data and neural web search, enabling research workflows that typically require Bloomberg terminals, financial databases, and hours of analyst time. It is structured as a **parent skill** with one **subskill**:

| Skill | File | Purpose |
|-------|------|---------|
| `deep-financial-research` | `SKILL.md` | Core research orchestration |
| `dcf-valuation` | `subskills/dcf-valuation/SKILL.md` | Intrinsic value estimation via DCF |

The two skills are designed to work independently or together. A prompt like *"Do a deep dive on AAPL with a DCF valuation"* will invoke both automatically.

---

## Repository Structure

```
deep-financial-research/
├── README.md
├── LICENSE.txt
├── SKILL.md                             # Main skill definition
└── subskills/
    └── dcf-valuation/
        └── SKILL.md                     # DCF valuation subskill
```

---

## Workflows

The main skill supports five research workflows, each triggered by natural language.

### 1. Company Deep Dive
**Trigger phrases:** *"deep dive on [ticker]"*, *"research [company]"*, *"investment overview of..."*

Pulls together a full investment-ready snapshot:
1. Real-time price and key market stats via Financial Datasets MCP
2. Recent news, press releases, and market narrative via Exa MCP
3. Core fundamentals — revenue, margins, P/E, EV/EBITDA, debt levels
4. Insider trading activity (net buys vs. sells, recent filings)
5. Analyst estimates, consensus ratings, and price targets
6. Synthesized bull case, bear case, and key risk summary

---

### 2. Due Diligence
**Trigger phrases:** *"due diligence on [company]"*, *"DD on [ticker]"*, *"red flags for..."*

Everything in the Company Deep Dive, plus:
- Litigation and regulatory search: SEC investigations, lawsuits, consent orders
- Accounting integrity checks: restatements, auditor resignations, material weaknesses
- Management background research: prior roles, track record, controversies
- Short interest and institutional ownership trends
- Structured risk and catalyst identification

---

### 3. Competitive Landscape
**Trigger phrases:** *"competitive landscape of [industry]"*, *"compare [company] to peers"*, *"who are the winners in [sector]"*

1. Industry mapping and market share analysis via Exa
2. Side-by-side fundamentals table across all major competitors (P/E, revenue growth, margins, ROE, net debt)
3. Recent M&A activity, new entrants, and disruptive developments
4. Positioning analysis — identifies leaders, laggards, and companies at inflection points

---

### 4. DCF Valuation
**Trigger phrases:** *"DCF on [ticker]"*, *"what is [stock] intrinsically worth"*, *"is [company] undervalued"*, *"fair value of..."*

Runs the integrated `dcf-valuation` subskill. See the [DCF Valuation Subskill](#dcf-valuation-subskill) section for full detail.

---

### 5. Market Sentiment
**Trigger phrases:** *"market sentiment on [ticker]"*, *"what is the street saying about [company]"*, *"analyst consensus for..."*

1. Mainstream and financial media search via Exa (filtered to bloomberg.com, reuters.com, wsj.com, ft.com, etc.)
2. Analyst estimate revisions and rating changes via Financial Datasets
3. Insider sentiment (aggregate net buying/selling pressure)
4. Structured bullish vs. bearish synthesis

---

## DCF Valuation Subskill

The `dcf-valuation` subskill performs a full discounted cash flow analysis to estimate intrinsic value per share. It runs as a standalone workflow or is automatically invoked as part of the Company Deep Dive when valuation language is detected.

### Methodology

| Step | Description |
|------|-------------|
| **1. Data Gathering** | 5-year FCF history, balance sheet, market cap, shares outstanding, sector |
| **2. Growth Rate** | FCF CAGR calculated, capped at 15%, cross-validated against revenue growth and analyst estimates |
| **3. WACC Estimation** | Built from risk-free rate (4.0%), equity risk premium (5.5%), cost of debt (5.5%), with sector adjustments |
| **4. FCF Projection** | 5-year forward projection with annual growth decay (5% step-down per year) |
| **5. Terminal Value** | Gordon Growth Model, terminal growth rate of 2.5% (GDP proxy) |
| **6. Present Value** | All cash flows discounted to today; equity value derived by subtracting net debt |
| **7. Sensitivity Analysis** | 3×3 matrix varying WACC (±1%) and terminal growth (2.0–3.0%) |
| **8. Validation** | Sanity-checked against reported EV, terminal value ratio (target: 50–80%), and implied P/FCF multiple |

### Sector WACC Adjustments

| Sector | Adjustment |
|--------|------------|
| Technology | +0.5% to +1.0% |
| Energy | +1.0% to +1.5% |
| Healthcare | +0.0% to +0.5% |
| Industrials | +0.0% to +0.5% |
| Consumer Staples | −0.5% |
| Utilities | −0.5% to −1.0% |
| Financials | Cost of equity only (no WACC) |

### DCF Output Example

```
### DCF Valuation: Apple Inc. (AAPL)

#### Valuation Summary
| Metric              | Value      |
|---------------------|------------|
| Current Price       | $189.42    |
| Fair Value          | $214.70    |
| Upside/(Downside)   | +13.3%     |
| Verdict             | Moderately Undervalued |

#### Sensitivity Analysis (Fair Value per Share)
| WACC \ Terminal  |  2.0%  |  2.5%  |  3.0%  |
|------------------|--------|--------|--------|
| 8.0%             | $231   | $244   | $259   |
| 9.0%             | $203   | $215   | $228   |
| 10.0%            | $180   | $190   | $201   |
```

Full output spec is defined in [`subskills/dcf-valuation/SKILL.md`](subskills/dcf-valuation/SKILL.md).

---

## Example Prompts

| Prompt | Workflow Triggered |
|--------|--------------------|
| `"Do a deep dive on NVDA"` | Company Deep Dive |
| `"Research Tesla for a long position"` | Due Diligence |
| `"What's MSFT worth based on DCF?"` | DCF Valuation |
| `"Analyze the AI chip competitive landscape"` | Competitive Landscape |
| `"What's the street sentiment on META?"` | Market Sentiment |
| `"Deep dive AAPL with DCF valuation"` | Deep Dive + DCF (combined) |
| `"Red flags on NKLA"` | Due Diligence (red flag focus) |
| `"Is AMZN undervalued right now?"` | DCF Valuation |

---

## Output Format

### Research Summary

Every Company Deep Dive and Due Diligence concludes with a structured summary:

```
### [Company] ([TICKER]) — Deep Research Summary

Current Price: $X.XX  |  Market Cap: $XB  |  As of: [timestamp]

Business Overview:
[1–2 sentences]

Key Metrics:
  Revenue       $XB    (+X% YoY)
  Gross Margin  X%
  P/E Ratio     X.x
  EV/EBITDA     X.x
  Net Debt      $XB

Recent Developments:
  • [Item] — Source (link)
  • [Item] — Source (link)

Bull Case:  [2–3 arguments]
Bear Case:  [2–3 arguments]
Key Risks:  [top risks]

Confidence: High / Medium / Low
```

---

## MCP Server Setup

This skill requires two MCP servers to be connected in your Claude environment.

### Required

| Server | Auth Method | Key Capabilities |
|--------|-------------|-----------------|
| [`financial-datasets`](https://www.financialdatasets.ai) | OAuth | Real-time prices, fundamentals, insider trades, analyst estimates, cash flow statements |
| [`exa`](https://exa.ai/) | API Key | Neural web search, domain-filtered news, citation-rich results |

### Optional

| Server | Auth Method | Key Capabilities |
|--------|-------------|-----------------|
| [`lightpanda`](https://github.com/lightpanda-io/browser) | Separate setup | Browser automation for paywalled or JS-rendered sources |

### Configuration Notes
- `financial-datasets` provides real-time data during market hours; brief delays possible depending on exchange
- `exa` results include timestamps — prefer results from the last 30–90 days for rapidly evolving situations
- Both servers are auto-discovered if registered in your MCP settings

---

## Installation

1. **Clone the repository**
   ```bash
   git clone https://github.com/your-username/deep-financial-research.git
   ```

2. **Add to your Claude skill configuration**
   - Point Claude to `SKILL.md` as the root skill file
   - The `dcf-valuation` subskill at `subskills/dcf-valuation/SKILL.md` is referenced automatically

3. **Connect MCP servers**
   - Configure `exa` with your API key in MCP settings
   - Connect `financial-datasets` via OAuth
   - Optionally configure `lightpanda` for browser automation

4. **Test the setup**
   ```
   "Do a quick deep dive on AAPL"
   ```
   A successful response will include real-time price data and recent news — confirming both MCP servers are live.

---

## Error Handling

| Scenario | Behavior |
|----------|----------|
| MCP server unavailable | Notifies user, falls back to Claude's training knowledge with explicit caveats |
| Tool timeout | Retries once; proceeds with available data and flags what's missing |
| Data discrepancy between sources | Notes the discrepancy and cites both values |
| Negative or volatile FCF (DCF) | Switches to analyst estimates or industry averages; documents the substitution |
| Calculated EV off by >30% from reported | Flags the divergence and revisits WACC/growth assumptions before presenting output |

---

## Limitations

- **Not investment advice.** This skill produces research analysis, not recommendations. All outputs should be treated as a starting point for further investigation.
- **DCF sensitivity.** Intrinsic value estimates are highly sensitive to WACC and growth rate assumptions. Small changes in inputs produce meaningfully different outputs — always review the sensitivity table.
- **Data freshness.** Financial Datasets provides real-time prices during market hours; fundamental data (balance sheets, cash flows) is updated on a reporting lag and reflects the most recent filing.
- **High-growth and financial companies.** DCF models are less reliable for pre-profitability companies, high-growth compounders, and financial institutions (which require DDM or residual income approaches instead).
- **Exa coverage.** Exa's neural search excels at English-language sources. Coverage of non-English or regional financial media may be limited.

---

## Disclaimer

This repository and the skills it contains are provided for informational and research purposes only. Nothing here constitutes investment advice, a solicitation, or a recommendation to buy or sell any security. Always conduct your own due diligence and consult a licensed financial advisor before making investment decisions.

See [LICENSE.txt](LICENSE.txt) for full terms.
