# Engineering Monte Carlo Project 01

## Notebook-by-Notebook Development Guide

**Project:** Force-Main Reliability and Lifecycle Decisions  
**Purpose:** Define the scope, tasks, and expected outputs of six Jupyter notebooks without providing implementation code.

The notebooks progress from data validation to engineering risk modeling, Monte Carlo simulation, financial evaluation, and sensitivity analysis.

| Notebook | Central question |
|---|---|
| `01_data_ingestion.ipynb` | Can I trust and use the data? |
| `02_eda.ipynb` | What patterns exist? |
| `03_failure_model.ipynb` | How likely are failures? |
| `04_monte_carlo.ipynb` | What outcomes are possible? |
| `05_lifecycle_cost.ipynb` | What are the financial consequences? |
| `06_sensitivity_analysis.ipynb` | How robust are the decisions? |

---

## 01_data_ingestion.ipynb

**Purpose:** Load, inspect, validate, and prepare the synthetic force-main dataset for analysis. This notebook is concerned with data integrity, not engineering conclusions.

### Tasks

- [ ] Import the required libraries.
- [ ] Load `force_main_segments_synthetic.csv`.
- [ ] Inspect the first and last records.
- [ ] Confirm the expected 240 segments, eight force mains, and 45 fields.
- [ ] Examine column names and data types.
- [ ] Identify missing values and calculate missing-value percentages.
- [ ] Distinguish structural missing values in historical repair records from actual data-quality issues.
- [ ] Check duplicate records and segment identifiers.
- [ ] Validate installation years, segment lengths, diameters, pressures, and cost fields.
- [ ] Inspect categorical values, including materials and data-quality flags.
- [ ] Identify inconsistent or suspicious records.
- [ ] Calculate historical years at risk, accounting for installation dates after 2006.
- [ ] Document all data-quality findings.

### Expected outputs

A validated dataset, a summary of data-quality findings, and a data dictionary.

**Boundary:** Descriptive checks such as `head`, `shape`, `info`, missing values, and uniqueness belong here. You are checking whether the dataset is usable, not interpreting failure behavior.

---

## 02_eda.ipynb

**Purpose:** Understand the engineering characteristics of the force-main network and explore relationships with historical failures.

### Tasks

- [ ] Examine the distribution of pipe ages.
- [ ] Compare segment counts and lengths by material.
- [ ] Examine diameter and segment-length distributions.
- [ ] Explore condition scores by material and age.
- [ ] Examine corrosion exposure and groundwater conditions.
- [ ] Analyze historical failure counts by material.
- [ ] Calculate failures relative to historical exposure years.
- [ ] Compare historical failures against pipe age, length, material, and corrosivity.
- [ ] Examine differences among the eight force mains.
- [ ] Investigate whether the historical data suggest overdispersion relative to a simple Poisson assumption.
- [ ] Compare the characteristics of records flagged `Review` against the remaining records.
- [ ] Document observed patterns and uncertainties.

### Suggested visualizations

1. Histogram of pipe ages.
2. Material distribution by total installed length.
3. Historical failure frequency versus age.
4. Failure frequency by material and corrosion exposure.
5. Condition score versus historical failure experience.

### Expected outputs

Three to five interpreted visualizations and a written summary of the most important patterns.

**Boundary:** EDA investigates what the synthetic historical data show. It does not establish the future failure model or prove causal relationships.

---

## 03_failure_model.ipynb

**Purpose:** Develop and evaluate a probabilistic model for annual force-main failures.

### Tasks

- [ ] Define the failure event being modeled.
- [ ] Explain why a Poisson distribution is being considered.
- [ ] Define annual failure intensity, λ, and its units.
- [ ] Distinguish expected annual failure count from the probability of at least one failure.
- [ ] Document the assumed baseline failure intensity.
- [ ] Incorporate segment length into the hazard model.
- [ ] Incorporate material-specific hazard multipliers.
- [ ] Incorporate corrosion exposure.
- [ ] Incorporate age-dependent deterioration.
- [ ] Calculate annual failure intensity for individual segments.
- [ ] Calculate the probability of at least one failure in a given year.
- [ ] Examine how annual failure intensity changes over the 20-year planning horizon.
- [ ] Test whether hazard increases appropriately with age and length.
- [ ] Verify that zero intensity produces zero failures.
- [ ] Compare theoretical Poisson properties against simulated event counts.
- [ ] Document independence assumptions and model limitations.

### Core mathematical concepts

The original scope establishes the illustrative hazard model:

$$
\lambda_{i,t} = \lambda_{\mathrm{ref}}
\left(\frac{L_i}{600}\right)
M_i C_i
\exp\left(\beta\max(A_{i,t}-15,0)\right)
$$

The probability of at least one failure in a year is:

$$
P(N_{i,t}\geq 1)=1-e^{-\lambda_{i,t}}
$$

**Reminder:** λ represents the expected number of events, not directly a probability. Model coefficients are illustrative assumptions, not calibrated industry failure rates.

### Expected outputs

A documented hazard model, example annual failure intensities, probability calculations, and validation checks.

Reusable functions should eventually live in `src/hazard.py`. The notebook should explain, demonstrate, and validate them rather than becoming the permanent home for all modeling logic.

---

## 04_monte_carlo.ipynb

**Purpose:** Use repeated random sampling to estimate the distribution of future failures across the force-main network.

### Tasks

- [ ] Define the 20-year simulation horizon beginning in 2026.
- [ ] Define one Monte Carlo iteration and what it represents.
- [ ] Establish a reproducible random seed.
- [ ] Calculate segment-level failure intensity for each simulated year.
- [ ] Generate random annual failure counts using the Poisson distribution.
- [ ] Allow multiple failures in a year where the model produces them.
- [ ] Simulate failures for all 240 segments.
- [ ] Aggregate failures by force main and by year.
- [ ] Track cumulative failures over the planning horizon.
- [ ] Run an initial 1,000 simulation trials.
- [ ] Increase to 10,000–20,000 trials after verifying the simulation.
- [ ] Examine convergence of estimated expected failures.
- [ ] Calculate mean, median, and percentile failure counts.
- [ ] Estimate probabilities of exceeding selected annual failure-count thresholds.
- [ ] Plot distributions of annual and 20-year failure counts.
- [ ] Document the limitations of independent segment and year assumptions.

### Suggested visualizations

- Histogram of total 20-year failures.
- Annual expected failures over time.
- Distribution of annual network-wide failure counts.
- Monte Carlo convergence plot.

### Expected outputs

A reproducible simulation engine and a distribution of possible failures over 20 years.

**Boundary:** Concentrate on physical failure outcomes, not lifecycle financial analysis. The question is: *How many failures might occur, and how uncertain is that estimate?*

---

## 05_lifecycle_cost.ipynb

**Purpose:** Translate simulated failures into financial consequences and compare alternative infrastructure management strategies.

### Tasks

- [ ] Identify the cost components associated with a force-main failure.
- [ ] Distinguish emergency repair, service disruption, and environmental response costs.
- [ ] Identify annual maintenance costs and planned capital costs.
- [ ] Check for overlapping cost components.
- [ ] Define the lognormal distributions for uncertain positive costs.
- [ ] Document the assumed coefficients of variation.
- [ ] Verify that sampled cost distributions have the intended means.
- [ ] Define the real discount rate and consistent 2026-dollar basis.
- [ ] Calculate the present value of annual expenditures.
- [ ] Establish the run-to-failure baseline strategy.
- [ ] Establish eligibility criteria for selective rehabilitation.
- [ ] Model rehabilitation costs and changes in future failure intensity.
- [ ] Establish eligibility criteria for selective replacement.
- [ ] Model replacement costs, effective age reset, and post-replacement maintenance.
- [ ] Run comparable simulations for each strategy.
- [ ] Compare total 20-year discounted lifecycle costs.
- [ ] Calculate mean, median, P90, and P95 lifecycle costs.
- [ ] Calculate annual budget exceedance probabilities using an explicitly stated budget threshold.
- [ ] Compare failure counts and service consequences alongside costs.
- [ ] Examine whether one strategy offers lower downside risk despite higher expected cost.

### Strategies to compare

| Strategy | Description |
|---|---|
| A | Run to failure |
| B | Selective rehabilitation |
| C | Selective replacement |
| D (optional) | System-wide replacement |

### Suggested visualizations

- Lifecycle cost distribution by strategy.
- Box plots comparing discounted lifecycle costs.
- Annual expected expenditure by strategy.
- Paired lifecycle cost differences between strategies.

### Expected outputs

A financial comparison of infrastructure management strategies, including cost uncertainty and service reliability consequences.

**Boundary:** Evaluate distributions of potential financial outcomes, not just average expenditures. For fair comparisons, use consistent assumptions and preferably common random numbers across alternatives.

---

## 06_sensitivity_analysis.ipynb

**Purpose:** Determine how strongly the conclusions depend on uncertain assumptions.

### Tasks

- [ ] Identify the most influential model assumptions.
- [ ] Vary the reference failure intensity.
- [ ] Vary the age deterioration coefficient.
- [ ] Vary material and corrosion multipliers.
- [ ] Vary rehabilitation effectiveness.
- [ ] Vary repair-cost uncertainty.
- [ ] Compare discount rates of 1%, 3%, and 5%.
- [ ] Introduce a correlated storm or surge stress scenario.
- [ ] Recalculate outcomes excluding lower-confidence `Review` records.
- [ ] Compare expected costs and downside exposure under each scenario.
- [ ] Identify assumptions that materially change strategy rankings.
- [ ] Develop a tornado chart or comparable sensitivity visualization.
- [ ] Distinguish parameter uncertainty from Monte Carlo sampling variability.
- [ ] Document limitations and potential sources of model error.
- [ ] Summarize which conclusions appear robust and which require better evidence.

### Questions to answer

- Which strategy appears most attractive under the baseline assumptions?
- Under what conditions does another strategy become preferable?
- Which uncertain inputs have the greatest influence?
- What additional engineering or utility data would improve confidence?

### Expected outputs

Sensitivity charts, scenario comparisons, and a written evaluation of model robustness.

**Boundary:** A simulation can generate precise-looking results from uncertain assumptions. This notebook tests whether those results are useful for decisions.

---

## How the notebooks work together

Each notebook should build on the previous stage without requiring earlier notebooks to be manually executed in a particular order.

| Notebook | Reusable module | Main responsibility |
|---|---|---|
| 01 | Data loading and validation module | Clean, validate, and document inputs |
| 02 | Primarily notebook analysis | Explore and interpret data |
| 03 | `src/hazard.py` | Calculate failure intensities |
| 04 | `src/simulation.py` | Simulate future failures |
| 05 | `src/costs.py` and simulation functions | Evaluate lifecycle strategies |
| 06 | Existing model modules | Test uncertainty and robustness |

The original project scope calls for independent functions, unit tests, and reproducibility. Preserve those principles throughout development.

## Definition of project success

The complete analytical reasoning chain should be explainable and defensible:

**Asset characteristics → Failure intensity → Random failures → Financial consequences → Strategy comparison → Sensitivity and decision confidence**

A successful project does more than run thousands of Monte Carlo iterations. It demonstrates the ability to translate a municipal infrastructure problem into a transparent analytical decision framework, while distinguishing synthetic assumptions from evidence that would be needed for real utility decisions.
