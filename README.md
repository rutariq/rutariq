# Atlas Quant Systems

**Microstructure research, options analytics and execution infrastructure for index futures,
built by one engineer with AI agents as the team.**

C# (NinjaTrader 8, .NET 9) · Python · TypeScript / Next.js · AWS · Bedrock · Schwab & IBKR APIs

> This page describes capabilities and the method families behind them. Source code, system
> design, model specifications, feature sets, parameters and trading results are private.

| Footprint | |
|---|---|
| Commits since April 2026 | 350+ |
| C# (indicators, strategies, infrastructure, replay engine) | ~32,000 lines |
| TypeScript (web platform) | ~57,000 lines |
| Custom NT8 indicators | 30+ |
| Offline research studies | 40+ |
| AWS Lambda services | 12+ |
| Research papers and audits | 20+ |

---

## Contents
1. [Platform overview](#1-platform-overview)
2. [Trading tools (NinjaTrader 8, C#)](#2-trading-tools-ninjatrader-8-c)
3. [Execution engineering](#3-execution-engineering)
4. [Research engine](#4-research-engine)
5. [Options analytics](#5-options-analytics)
6. [Quantitative methods](#6-quantitative-methods)
7. [AI agents and LLM systems](#7-ai-agents-and-llm-systems)
8. [Machine learning](#8-machine-learning)
9. [Cloud and data (AWS)](#9-cloud-and-data-aws)
10. [Web platform](#10-web-platform)
11. [Automation](#11-automation)
12. [Education](#12-education)
13. [Research integrity](#13-research-integrity)

---

## 1. Platform overview
Four layers, each built and operated by me:

| Layer | Stack | Role |
|---|---|---|
| **Trading runtime** | C# · NinjaTrader 8 · .NET 9 | Indicators, strategy engines, execution safety, telemetry |
| **Research** | C# · Python · DuckDB | Causal tick replay, statistical validation, studies |
| **Cloud** | AWS serverless · Bedrock | Market and options data, AI agents, reporting |
| **Product** | Next.js · TypeScript | Member dashboards, admin portal, education |

**Engineering standards**
- **Reproducibility:** decisions and releases are content-hashed (SHA-256), so any output can
  be re-derived and verified.
- **Causality:** research code cannot read data from the future; this is enforced in code, not
  by convention.
- **Parity:** offline research versions of live tools are tested against the live versions.
- **Resilience:** atomic writes, bounded retries, fail-closed components and heartbeat monitoring.

## 2. Trading tools (NinjaTrader 8, C#)
30+ custom indicators across these families:

- **Value and levels:** VWAP with deviation bands, market profile and value area
  (POC / VAH / VAL), session levels, multi-anchor level maps, support / resistance and trend rays.
- **Order flow:** cumulative volume delta, delta statistics, absorption detection and
  price-impact measures.
- **Filtering and momentum:** Kalman-filtered price and VWAP, adaptive prediction filters,
  impulse and MACD-family momentum, volatility bands.
- **Regime and volatility:** trend / chop classification, efficiency ratios, multi-timeframe
  regime ribbons, realized and historical volatility.
- **Context:** market breadth, economic-calendar events, regime dashboards.
- **Custom bar types:** Renko variants with their own renderer.
- **Shared data layer:** an in-process publish / subscribe bus so tools share live levels and
  state without recomputing them.

## 3. Execution engineering
- **Strategy engines** for ES and NQ futures, run in simulation and forward test.
- **Layered entry safety:** independent confirmation before any order, hard vetoes and kill
  switches that no signal can override, and safe exits if a stop has already been crossed.
- **Fill telemetry:** slippage, time to fill and excursion (MFE / MAE) recorded per fill.
- **State recovery** after restarts, and a CME trading-calendar model that survives contract rolls.
- **Versioned release kits** with hashed manifests and golden test fixtures.

## 4. Research engine
- **Tick-by-tick replay (C#):** tests ideas on every individual trade, not aggregated bars.
- **Exchange-style fill model:** FIFO matching with queue modelling; entries fill only on
  trades after the signal.
- **Fidelity tests** that pin offline research code to the live indicators.
- **40+ studies** covering entries, exits, trend persistence, false breakouts, market
  structure and order flow.
- **Ground-truth labelling** of historical market outcomes for evaluation and machine learning.
- **Results catalog** that records the provenance of every run (see section 13).

## 5. Options analytics
A dealer-positioning engine on live option chains (Schwab, Interactive Brokers):

- **Black–Scholes Greeks** with dividend yield, standard closed forms:
  $`d_1 = \frac{\ln(S/K) + (r - q + \sigma^2/2)T}{\sigma\sqrt{T}}`$,
  $`\Gamma = \frac{e^{-qT}\varphi(d_1)}{S\sigma\sqrt{T}}`$, plus delta, vanna and charm.
- **Implied volatility** by Newton–Raphson.
- **Dealer exposure:** gamma (GEX), delta (DEX), charm and vanna aggregated by strike, under
  the standard dealer-positioning convention
  $`\mathrm{GEX}_K = \Gamma_K \cdot OI_K \cdot 100 \cdot S^2 \cdot 0.01`$.
- **Key levels:** gamma walls and the zero-gamma level.
- **Views:** same-day (0DTE) and multi-expiry, put / call ratio, volatility indices
  (VIX, VVIX, VXN, SKEW).
- **Event studies** of dealer-positioning states.

## 6. Quantitative methods
Method families used across the research, with standard references:

| Area | Methods |
|---|---|
| **State estimation** | Kalman filtering, adaptive noise estimation, Interacting Multiple Model (IMM) filtering (Blom & Bar-Shalom 1988) |
| **Regime detection** | Hidden Markov models (Baum–Welch, forward filtering), Bayesian online change-point detection (Adams & MacKay 2007) |
| **Market microstructure** | Order-flow imbalance, cumulative delta, price impact (Kyle 1985), volume-clock sampling (Clark 1973; Easley, López de Prado & O'Hara 2012), point-process models |
| **Cross-asset** | Recursive least squares, rolling correlation, lead–lag analysis |
| **Validation** | Combinatorial purged cross-validation, Deflated Sharpe Ratio (Bailey & López de Prado 2014), probability of backtest overfitting (Bailey et al. 2017), walk-forward and leave-one-out ablation |
| **Null models** | Random-walk baselines, e.g. gambler's ruin for brick continuation, $`p_0 = r/(1+r)`$ |
| **Risk** | Volatility-scaled targets, stops as hypothesis invalidation, volatility-based position sizing |
| **Optimization** | CMA-ES with bounded parameters and trial accounting |

**Research papers (internal):** random-walk nulls, volatility scaling, regime switching,
chop detection, false-breakout microstructure, trend timing, stop-loss theory, overfitting
control, labelling for small samples, zero-lookahead backtest design, multi-agent trading
architectures, bounded LLM advisory, data lineage and LLM cost engineering.

## 7. AI agents and LLM systems
- **Multi-agent system on AWS Bedrock** with role-based model routing, model fallback,
  prompt caching and schema-validated outputs.
- **Shadow mode:** agent decisions are logged and scored, never executed directly.
- **Bounded advisory:** the AI tier can only restrict risk, never add it, and is compared
  against a rule-based baseline.
- **Local-first routing:** a local model handles routine decisions and escalates only when
  needed; the router fails closed.
- **Outbound data redaction** of secrets and personal data before anything leaves the machine.
- **Retrieval:** hybrid keyword + vector search with Reciprocal Rank Fusion,
  $`\mathrm{RRF}(d) = \sum_i 1/(k + \mathrm{rank}_i(d))`$, and a code symbol graph across C#,
  Python and TypeScript.
- **Context engineering:** zoned documentation with token budgets, MCP tool servers, local models.
- **LLM content generation:** daily market reports with technical and plain-language editions,
  narrated audio and automated quality feedback.
- **Agent Cortex:** one installable rulebook and toolkit for Claude Code, Google Antigravity,
  Cursor and VS Code.

## 8. Machine learning
- **Latent-state models:** hidden Markov models and change-point detection for market regimes.
- **Supervised classifiers:** gradient boosting, random forests and a local transformer
  (ModernBERT) classifier.
- **Calibration:** logistic calibration against random-walk baselines; strictly proper scoring
  rules and the Brier skill score $`\mathrm{BSS} = 1 - \mathrm{BS}/\mathrm{BS}_{\mathrm{ref}}`$.
- **Labelling:** causal ground-truth labelling, human-reviewed samples, triple-barrier and
  meta-labelling research.
- **Evaluation discipline:** purged cross-validation, trial ledgers and overfitting checks.
- **Deployment target:** native C# inference inside the trading runtime.

## 9. Cloud and data (AWS)
- **12+ Lambda services:** market-data fetchers, news ingestion, telemetry, heartbeat
  monitoring, documentation and archive APIs, data-lake sync, health checks.
- **Next.js on Lambda** with server-side rendering.
- **Infrastructure as code:** SAM / CloudFormation, API Gateway, IAM, CloudWatch alarms.
- **CI/CD:** CodeBuild pipelines for the site and the reporting system.
- **Services:** S3, CloudFront, Cognito, SES, DynamoDB, EventBridge, Bedrock, Polly, SNS,
  CloudWatch, EC2.
- **Data:** DynamoDB and S3 for live data and archives, a DuckDB research lake.

## 10. Web platform
- **Public site**, **member dashboard** and **admin portal** (Next.js, TypeScript).
- **Live dashboards:** futures, volatility and options positioning over WebSockets, with
  strike maps, heatmaps and an economic calendar.
- **Mission Control:** system readiness, engine heartbeat, decision journal and bounded controls.
- **Secure sign-in**, SEO (sitemap, structured data) and a full legal / compliance flow.

## 11. Automation
- **Daily AI research publishing:** data collection, regime analysis, AI drafting, charts,
  audio, archiving and community posting, end to end.
- **Self-healing:** automatic diagnosis and recovery, pre-publish checks and approval alerts.
- **Monitoring:** heartbeat checks on trading nodes with alerts.
- **Tooling:** background agents for code-graph and knowledge-base refresh, a Figma diagram
  plugin, and developer utilities.

## 12. Education
- **AQS-EDU course:** 10 modules from market basics and leverage to prop-firm trading and
  process frameworks.
- **Indicator education series**, guides on options positioning, market breadth and
  sentiment, a mobile-friendly reader and a Discord community.

## 13. Research integrity
- **No receipt, no result:** every result must carry its engine version, data fingerprint,
  bar-quality check, fill model, output hash, number of trials and validation state.
- **Results catalog** that labels unproven runs instead of hiding them.
- **Guardrails for AI coding tools** that block silent shortcuts (wrong engine, approximated
  data, look-ahead), with monitoring of the rules themselves.

---

🌐 [dev.atlasquantsystems.com](https://dev.atlasquantsystems.com)
