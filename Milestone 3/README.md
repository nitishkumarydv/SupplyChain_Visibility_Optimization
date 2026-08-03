# Milestone 3 — Supply Chain Visibility Optimization
## Power BI Dashboard: Supplier Performance & Transportation Analytics

This milestone delivers an interactive Power BI dashboard focused on two core areas of supply chain visibility: **supplier performance scorecarding** and **transportation cost & carrier analytics**. The goal is to give supply chain stakeholders a clear, data-driven view of supplier reliability and logistics efficiency to support better sourcing and shipping decisions.

---

## 1. Supplier Scorecard Calculation Methodology

The supplier scorecard aggregates supplier performance across multiple weighted dimensions to produce a single composite score (0–100) per supplier per period.

**Core metrics used:**
- **On-Time Delivery Rate (OTD)** – % of orders delivered on or before the promised date
- **Order Fill Rate / Accuracy** – % of order quantity delivered correctly against PO
- **Quality Rate** – % of received units passing quality inspection (1 − defect/return rate)
- **Lead Time Consistency** – variance between quoted and actual lead time
- **Cost Competitiveness** – supplier's average unit cost vs. category benchmark price

**Composite score formula:**
Supplier Score = (W1 × OTD) + (W2 × Fill Rate) + (W3 × Quality Rate)
+ (W4 × Lead Time Consistency) + (W5 × Cost Index)


Weights (W1–W5) are configurable and were set based on business priority (delivery and quality weighted highest, cost weighted lowest, reflecting a visibility-first rather than pure cost-cutting objective).

All metrics are normalized to a 0–100 scale before weighting so no single metric with a different unit of measure dominates the score.

---

## 2. Supplier Ranking and Benchmarking Approach

- Suppliers are **ranked within their category/commodity group**, not globally, since acceptable lead times and defect rates vary significantly by product type.
- A **tiering system** (A/B/C or Top/Mid/Bottom quartile) is applied based on the composite score, allowing quick visual identification of top-performing vs. at-risk suppliers.
- **Benchmarking** is done against:
  - Category average score (peer comparison)
  - Historical performance (trend over time, quarter-over-quarter)
  - Target/SLA thresholds defined in supplier contracts
- Suppliers consistently falling in the bottom tier over multiple periods are flagged for review/re-negotiation.

---

## 3. Transportation Cost Analysis Methodology

Transportation costs are broken down and analyzed across multiple dimensions to identify cost drivers and inefficiencies:

- **Cost per shipment / cost per unit / cost per mile (km)** — normalizes cost across shipments of different sizes and distances
- **Cost by mode** (road, rail, air, sea) — to compare mode efficiency
- **Cost by lane/route** — origin-destination pair analysis to spot expensive or inefficient lanes
- **Cost by carrier** — to compare carrier pricing for similar lanes/loads
- **Freight cost as % of goods value** — a standard logistics efficiency KPI
- **Trend analysis** — cost movement over time to catch seasonal spikes or emerging issues early

Data is pulled from shipment-level transaction records, aggregated at route, carrier, and time-period granularity using Power BI measures (DAX) for dynamic filtering.

---

## 4. Route and Carrier Performance Evaluation

Routes and carriers are evaluated using a combination of **cost, time, and reliability metrics**:

| Metric | Purpose |
|---|---|
| On-time delivery % by carrier/route | Reliability |
| Average transit time vs. planned transit time | Efficiency |
| Damage/loss incident rate | Service quality |
| Cost per mile/km by carrier | Cost efficiency |
| Volume/capacity utilization | Route optimization potential |

- **Carrier scorecards** rank carriers on a blended reliability + cost index, similar in structure to the supplier scorecard.
- **Route heat-mapping** highlights high-cost or high-delay lanes for renegotiation or mode-shift consideration (e.g., moving a lane from air to rail).
- Underperforming carrier-route combinations are flagged where cost is above benchmark **and** on-time performance is below SLA.

---

## 5. Key Insights and Business Recommendations

**Key Insights:**
- A small subset of suppliers/carriers disproportionately drives delivery delays and cost overruns (classic 80/20 pattern).
- Certain lanes show consistently higher cost-per-unit than category benchmarks, indicating renegotiation or mode-shift opportunities.
- Suppliers with strong quality scores but weak lead-time consistency present a risk that isn't visible from cost data alone — reinforcing the need for a multi-dimensional scorecard rather than cost-only supplier evaluation.

**Recommendations:**
1. **Prioritize renegotiation** with bottom-tier suppliers/carriers identified in the scorecards, or begin qualifying alternate sources for high-risk categories.
2. **Shift volume on underperforming lanes** to better-performing carriers or alternate transport modes where cost/time benchmarks are consistently missed.
3. **Set up ongoing monitoring** (this dashboard) as a recurring review tool in supplier/carrier business reviews, rather than a one-time analysis.
4. **Tighten SLAs** for suppliers/carriers with high variance in lead time or transit time, even if their average performance looks acceptable.

---

## 📊 Dashboard Screenshots

**Supplier Performance**
![Supplier Performance](Screenshots/Supplier_Performance.png)

**Transportation Analytics**
![Transportation Analytics](Screenshots/Transportation_Analytics.png)

---

## 🛠️ Tools Used
- Microsoft Power BI (data modeling, DAX, visualization)