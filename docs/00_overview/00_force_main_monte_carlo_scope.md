# Engineering Monte Carlo Project 01: Force-Main Reliability and Lifecycle Decisions

**Project type:** Applied civil engineering / probability / Monte Carlo simulation  
**Recommended effort:** 6–8 weeks, 3–5 hours per week (approximately 24–35 hours)  
**Tools:** Python, pandas, NumPy, matplotlib, Jupyter; optional SciPy  
**Data:** `force_main_segments_synthetic.csv` (240 segments, 8 force mains, 45 fields)  
**Data status:** Entirely **synthetic**. All physical characteristics, failure histories, and costs are invented for training. The dataset is NOT utility evidence, not an engineering design basis, and not calibrated to observed industry failure frequencies.

## 1. Decision question

Over a **20-year planning horizon beginning in 2026**, should a hypothetical utility (A) continue to maintain its existing force mains, (B) selectively rehabilitate high-risk segments, or (C) proactively replace selected or all segments? Compare the **distribution of discounted lifecycle costs**, the probability of exceeding an annual budget, and expected failures under uncertain conditions.

Do not use mean cost alone to choose a strategy. Report tradeoffs between expected cost, downside exposure, service continuity, and environmental consequences.

## 2. Learning objectives

1. Translate an engineering decision into explicit stochastic assumptions and cash flows.
2. Understand Poisson event counts, annual failure probabilities, and the limitations of independent-failure assumptions.
3. Simulate changing failure hazards with asset age, material, corrosion exposure, and maintenance strategy.
4. Model uncertain event costs, including emergency repair, disruption, and environmental response.
5. Apply discounting, summarize risk distributions, and perform sensitivity and scenario analysis.
6. Write and validate small Python functions independently before using AI for review.
7. Communicate defensible conclusions and limitations to a utility asset-management audience.

## 3. Synthetic system description

The CSV represents eight municipal wastewater force mains divided into **30 modeled segments each**. Segments differ in installation year, material, size, length, pressure, soil and groundwater exposure, crossing constraints, consequence severity, bypass availability, historical repair counts, and 2026 planning-level costs.

The segments are modeled decision units, **not** necessarily physically isolated hydraulic or operational assets. A segment failure can interrupt an entire force main; simply summing segment-level disruption costs may double count coincident system impacts. For this exercise, assume failures are sufficiently rare that this approximation is acceptable; identify it as a limitation.

### Data interpretation and cautions

- `historical_failure_count_2006_2025` is a simulated retrospective 20-year count. It is **not** a future target, a measured failure rate, or an observation that the segment has been under the same exposure for all 20 years. Assets installed after 2006 have shorter exposure; use `max(2006, installation_year)` to calculate years at risk through 2025.
- `last_failure_year`, `last_failure_cause`, `last_repair_year`, `repair_method`, and `repair_cost_last_usd_2026` are blank or `None` where no failure was simulated. These are **structural missing values**, not necessarily data-quality errors.
- `condition_score_1_best_5_worst` is a synthetic proxy score; it is not a measured physical condition grade. Its relation to failure must be assumed, tested, and qualified.
- Costs are **illustrative 2026 USD planning estimates**, not bids, quotes, or calibrated historical repair costs. Do not add them without checking whether a cost component is conditional on a failure and whether it overlaps another component.
- `estimated_service_disruption_cost_usd_2026` and `estimated_environmental_response_cost_usd_2026` are conditional **cost estimates per modeled failure event**, not annual expenditures. Environmental response is an exposure proxy, not a prediction of regulatory liability.
- `estimated_planned_rehab_cost_usd_2026` is a synthetic intervention estimate; **rehabilitation feasibility is not guaranteed** for every material, condition, crossing, or diameter. Infeasible segments must be excluded or flagged after engineering screening.
- Pressure fields are simplified. `design_pressure_psi` is an illustrative model attribute, **not proof of pressure class, surge adequacy, or code compliance**. `velocity_at_peak_fps` is a simplified nominal-diameter calculation.
- Rows marked `Review` are deliberately lower-confidence synthetic records. Do not silently discard them; compare results with and without them.
- The generated histories are not designed to perfectly reproduce the forward-looking hazard model. Treat that mismatch as an opportunity to study model uncertainty, not as a data error to conceal.

## 4. Model architecture

Build the following components as separate functions:

1. `load_and_validate_data(path)` — import CSV; validate unique IDs, numeric bounds, categorical values, null patterns, and logical constraints.
2. `years_at_risk(installation_year)` — calculate historical exposure years for each segment.
3. `annual_failure_intensity(segment, year, scenario)` — estimate expected annual event count (lambda) using a clearly documented **assumed** hazard model.
4. `simulate_annual_failures(lambda_, rng)` — draw annual failure count from a Poisson distribution.
5. `sample_failure_cost(segment, rng, scenario)` — sample conditional costs with positive-valued uncertainty distributions.
6. `apply_intervention(segment, strategy, year)` — change hazard and annual costs after planned rehabilitation or replacement.
7. `simulate_path(segments, strategy, rng)` — simulate one 20-year lifecycle path and discounted costs.
8. `summarize_simulations(results)` — expected value, median, P10/P90/P95, exceedance rates, and uncertainty intervals.

### Baseline annual failure intensity (illustrative, not calibrated)

Use a model of the form:

`lambda_i,t = lambda_ref * (length_i / 600) * material_factor_i * corrosion_factor_i * exp(age_coefficient * max(age_i,t - 15, 0))`

Starting assumptions for testing only:

| Parameter | Initial illustrative assumption | Sensitivity range |
|---|---:|---:|
| Reference intensity (`lambda_ref`) | 0.0045 failures/year per 600-ft reference segment | 0.002–0.012 |
| Age coefficient | 0.026 per year beyond age 15 | 0.01–0.05 |
| Material factor: DIP | 1.20 | 0.8–1.8 |
| Material factor: AC or steel | 1.70 | 1.1–2.5 |
| Material factor: PVC or HDPE | 0.62 | 0.35–1.0 |
| High-corrosivity multiplier (DIP/AC/steel) | 1.50 | 1.0–2.5 |
| Moderate/low corrosion multiplier | 1.00 | fixed initially |

These factors are **assumptions for an exercise**, not published performance coefficients. The retrospective data generator uses a related synthetic process, but a model should not be judged as validated merely because it recovers synthetic assumptions.

For each segment and year, `P(at least one failure) = 1 - exp(-lambda_i,t)`. Do **not** confuse this probability with the expected count `lambda_i,t`. Multiple failures within a year are possible in the Poisson model. At the beginning, assume conditional independence among segments and years; later test a shared storm/surge stress scenario to expose this limitation.

### Cost uncertainty

Use a lognormal distribution for each strictly positive failure-cost component, parameterized so the CSV estimate is its **mean** (not its median). Begin with illustrative coefficients of variation:

- Emergency repair: CV = 0.35
- Service disruption: CV = 0.60
- Environmental response: CV = 0.85
- Planned capital intervention: CV = 0.25

For lognormal mean `m` and CV `c`, use `sigma² = ln(1 + c²)` and `mu = ln(m) - sigma²/2`. Consider shared inflation and event-severity factors in an advanced version to avoid treating every cost as independent.

Use a **real discount rate of 3%** and constant 2026 dollars for the baseline. Sensitivity test 1%, 3%, and 5%. Do not mix nominal escalation with a real discount rate.

### Strategies

**A. Run to failure:** No planned capital intervention; annual maintenance continues and failures are repaired as simulated.

**B. Selective rehabilitation:** Screen eligible segments; rehabilitate those with condition score >=4 and either high consequence or elevated modeled risk. Initial assumption: 60% hazard reduction after rehab, subject to a 30%–80% sensitivity range. Spend planned rehabilitation cost in year 1; retain annual maintenance at 80% of baseline. Do not automatically rehab assets that are technically unsuitable.

**C. Selective replacement:** Replace segments with condition score >=4 and high consequence, or an explicitly defined economic trigger. Assume replacement resets effective age to zero and adopts a documented new-material hazard factor; spend replacement cost in year 1. Reduce annual maintenance to 50% of baseline for replaced segments. **Do not assume replacement eliminates all failures.**

Optional **D. System-wide replacement:** Replace all 240 segments as an intentionally extreme comparator; distinguish this from the economically targeted strategy.

A modeled failure may incur emergency repair + service disruption + environmental response. Explain whether environmental response should occur for every event or be drawn conditionally; for a more realistic second version, assign an explicit probability of environmental impact based on crossing type and exposure.

## 5. Work plan and deliverables

### Week 1 — Data ingestion and quality checks (3–4 hours)
- Read CSV; inspect dimensions, column types, nulls, uniqueness, ranges, and categories.
- Distinguish expected blank failure-history fields from genuine data gaps.
- Verify installed age, length, pressure, and cost relationships.
- **Deliverable:** `notebooks/01_ingestion.ipynb`, `reports/data_dictionary.md`.

### Week 2 — EDA and historical exposure (3–4 hours)
- Plot age/material/condition distributions; compare failures per historical exposure-year, not just raw counts.
- Examine failure counts versus material, age, length, and corrosivity.
- Assess whether a simple Poisson assumption seems reasonable; note sparse-data uncertainty.
- **Deliverable:** `notebooks/02_eda.ipynb` and 3–5 interpreted figures.

### Week 3 — Mathematical model and unit tests (4–5 hours)
- Write the intensity formula and units on paper before coding.
- Implement `annual_failure_intensity()` and `simulate_annual_failures()`.
- Test `lambda=0`, monotonicity with length and age, and empirical simulated mean versus lambda.
- **Deliverable:** `src/hazard.py`, `tests/test_hazard.py`, `docs/model_assumptions.md`.

### Week 4 — Monte Carlo engine (4–5 hours)
- Implement yearly events, failure costs, maintenance, and discounting.
- Run 1,000 trials initially, then 10,000–20,000 trials after verification.
- Fix a random seed for reproducibility and use independent RNG streams or common random numbers for fair strategy comparisons.
- **Deliverable:** `src/simulation.py`, `notebooks/03_monte_carlo.ipynb`.

### Week 5 — Intervention comparison (3–5 hours)
- Add A/B/C strategies and document eligibility and timing.
- Compare total 20-year present value, failure counts, P90 costs, and annual budget exceedance.
- Plot cost distributions and paired strategy differences.
- **Deliverable:** `notebooks/04_strategy_comparison.ipynb`.

### Week 6 — Sensitivity and model criticism (4–5 hours)
- Vary baseline hazard, deterioration, rehab effectiveness, repair-cost variability, and discount rate.
- Include a correlated surge/storm stress test and data-quality exclusion test.
- Identify cases where the preferred strategy changes.
- **Deliverable:** `notebooks/05_sensitivity.ipynb`, `reports/validation_notes.md`.

### Weeks 7–8 (optional) — Executive decision memo and polish (3–7 hours)
- Write a two-page memo for a hypothetical utility director, with assumptions, 3 alternatives, cost distributions, uncertainty, and limitations.
- Refactor and test reusable functions. Add a README and reproducible environment instructions.
- **Deliverable:** `reports/decision_memo.md`, `README.md`, portfolio-ready repository.

## 6. Acceptance criteria

- Every model input has a source or is explicitly labeled an assumption.
- Historical years at risk correctly account for installation after 2006.
- No future information is used to calculate baseline 2026 risk.
- Monte Carlo outputs are reproducible for a fixed seed.
- Basic probability checks pass (`P>=1 = 1 - exp(-lambda)` and simulated event counts converge toward lambda).
- All cost streams are discounted consistently and capital expenditure timing is clear.
- Model results include mean, median, P90, P95, and annual budget exceedance probabilities.
- Sensitivity analysis shows at least one scenario in which the strategy ranking might change, or explicitly reports that it did not.
- Report clearly separates **synthetic demonstration**, **engineering assumptions**, and **real-world decisions requiring calibration**.

## 7. Suggested repository structure

```text
force-main-risk-mc/
├── data/
│   └── raw/
│       └── force_main_segments_synthetic.csv
├── docs/
│   ├── model_assumptions.md
│   └── data_dictionary.md
├── notebooks/
│   ├── 01_ingestion.ipynb
│   ├── 02_eda.ipynb
│   ├── 03_monte_carlo.ipynb
│   ├── 04_strategy_comparison.ipynb
│   └── 05_sensitivity.ipynb
├── src/
│   ├── hazard.py
│   ├── costs.py
│   └── simulation.py
├── tests/
│   └── test_hazard.py
├── reports/
│   ├── validation_notes.md
│   └── decision_memo.md
└── README.md
```

## 8. AI-use protocol

1. Write a short pseudocode plan and the engineering assumptions yourself.
2. Attempt one small function independently for 15–20 minutes, with documentation permitted.
3. Ask AI for **hints or review first**, rather than an entire finished solution.
4. After using AI, rewrite or modify the function without copying it verbatim.
5. Write one test that would fail if the function were conceptually wrong.
6. Record in a learning log what you could do independently and what required assistance.

**Success is not a model with the lowest simulated cost. Success is a model whose assumptions, units, failure logic, uncertainty, and limitations you can defend.**
