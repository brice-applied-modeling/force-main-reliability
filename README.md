# Force-Main Reliability and Lifecycle Decisions

**Engineering Monte Carlo Project 01** | Applied civil engineering · Reliability modeling · Decision analytics

A Python-based Monte Carlo study exploring how uncertainty in wastewater force-main failures, repair costs, and asset deterioration affects long-term infrastructure investment decisions.

> **Educational simulation only.** The dataset and model coefficients are entirely synthetic and illustrative. Results are not calibrated to actual utility performance and must not be used as an engineering design basis, regulatory assessment, or real-world capital investment recommendation.

## Project overview

Municipal utilities must balance the immediate expense of rehabilitating or replacing aging force mains against the uncertain future costs of failures, service interruptions, and environmental response. This project evaluates those tradeoffs across a **20-year planning horizon beginning in 2026**, using stochastic failure events and lifecycle cost modeling rather than a single deterministic estimate.

### Central decision question

Should a hypothetical utility continue routine maintenance, selectively rehabilitate vulnerable segments, or replace high-consequence assets proactively?

The model compares not only expected lifecycle cost, but also tail risk, annual budget exposure, expected failures, and service/environmental consequences.

## System and dataset

The input file, `data/raw/force_main_segments_synthetic.csv`, describes **240 modeled segments across eight force mains**, with **45 fields** covering asset characteristics, exposure, simulated historical repairs, condition proxies, and planning-level costs.

| Category | Examples |
|---|---|
| Physical assets | Installation year, material, diameter, length, pressure |
| Exposure and constraints | Soil corrosivity, groundwater, crossings, bypass availability |
| Condition and history | Synthetic condition score, historical failures, repair history |
| Consequences | Emergency repair, disruption, environmental response costs |
| Interventions | Estimated rehabilitation and replacement costs |
| Quality | Records flagged `Review` for sensitivity checks |

Historical failure counts cover 2006–2025, but exposure time must reflect each segment's installation year. Missing repair-history fields can be structurally absent rather than erroneous. Segments are analytical units, not necessarily independent operational assets.

## Strategies evaluated

| Strategy | Description | Initial modeling assumptions |
|---|---|---|
| **A · Run to failure** | Continue maintenance; repair simulated failures | No planned capital intervention |
| **B · Selective rehabilitation** | Screen technically eligible high-condition-score, high-consequence or elevated-risk segments | 60% hazard reduction; 80% of baseline maintenance; rehabilitation in year 1 |
| **C · Selective replacement** | Replace qualifying high-consequence segments or those meeting a defined economic trigger | Reset effective age; assign new-material hazard; 50% maintenance for replaced segments; replacement in year 1 |
| **D · System-wide replacement (optional)** | Replace all segments as an extreme comparator | Explicitly distinguish from targeted replacement |

Rehabilitation feasibility must be screened, and replacement does **not** eliminate failure risk.

## Modeling approach

### Annual failure intensity

The baseline illustrative hazard model is:

\[
\lambda_{i,t}=\lambda_{ref}\left(\frac{L_i}{600}\right)M_iC_i\exp\left[k\max(A_{i,t}-15,0)\right]
\]

where `L` is segment length in feet, `M` is a material multiplier, `C` is a corrosion multiplier, `A` is asset age, and `k` is an age coefficient. The starting reference intensity is **0.0045 failures/year per 600-foot segment**, with `k = 0.026` after age 15. All coefficients are **assumed, not empirically calibrated**.

Annual failure counts are drawn from a Poisson distribution:

\[
N_{i,t}\sim\operatorname{Poisson}(\lambda_{i,t}),\qquad
P(N_{i,t}\geq1)=1-e^{-\lambda_{i,t}}
\]

The intensity is an expected event count, not itself a probability. Multiple failures per year are possible. The baseline assumes independence between segments and years, followed by a shared storm/surge stress test.

### Failure costs and discounting

Failure-event cost components include emergency repair, service disruption, and environmental response. Positive costs are sampled from **lognormal distributions** whose means equal the input estimates.

| Cost component | Initial coefficient of variation |
|---|---:|
| Emergency repair | 0.35 |
| Service disruption | 0.60 |
| Environmental response | 0.85 |
| Planned intervention | 0.25 |

The baseline uses **constant 2026 dollars** and a **3% real discount rate**, with 1% and 5% sensitivity cases. Environmental response can initially be applied to every failure, but a more realistic extension assigns conditional impact probabilities.

## Outputs and decision measures

The analysis is designed to produce:

- Mean and median discounted 20-year lifecycle costs, with P10, P90, and P95 outcomes.
- Simulated failure counts and service/environmental consequence comparisons.
- Probability of exceeding a clearly specified annual budget threshold.
- Cost distribution plots and paired strategy comparisons.
- Sensitivity results for failure intensity, deterioration, rehabilitation effectiveness, cost variation, discount rate, shared stress events, and lower-confidence records.
- A concise decision memo explaining tradeoffs, model uncertainty, and limitations.

**No strategy is designated optimal in advance.** Rankings must be determined by simulation and interpreted in light of engineering feasibility and risk tolerance.

## Repository layout

See the [Structure](docs/01_architecture/00_structure.md) for a complete layout of the intended repository structure. 

The structure above is **planned**; individual files will be added as project milestones are completed. The project scope also calls for `reports/data_dictionary.md`; this README uses `docs/data_dictionary.md` to keep documentation together. Use one location consistently.

## Getting started

**Requirements:** Python, Jupyter, NumPy, pandas, matplotlib; SciPy is optional. The project scope does not specify pinned package versions.

```bash
# From the repository root
uv init
# Create a virtual environment
uv venv
# Activate the virtual environment
source .venv/bin/activate       # Linux/macOS
# .venv\Scripts\Activate.ps1    # Windows PowerShell
# Install project dependencies
uv add numpy pandas matplotlib seaborn scipy jupyterlab pytest
```

Place the supplied synthetic CSV at `data/raw/force_main_segments_synthetic.csv`. Work through the numbered notebooks in order. Once the hazard tests are implemented, run:

```bash
python -m pytest tests/
```

The commands describe the intended environment and workflow; they do not imply that the notebooks, tests, or simulation engine are already implemented. Record the tested environment and random seed as development progresses.

## Development roadmap

| Phase | Focus | Primary deliverables |
|---|---|---|
| Week 1 | Data ingestion and validation | `01_ingestion.ipynb`, data dictionary |
| Week 2 | Exploratory analysis and historical exposure | `02_eda.ipynb`, 3–5 interpreted figures |
| Week 3 | Hazard equations and unit tests | `hazard.py`, `test_hazard.py`, model assumptions |
| Week 4 | Monte Carlo lifecycle engine | `simulation.py`, `03_monte_carlo.ipynb` |
| Week 5 | Strategy comparison | `04_strategy_comparison.ipynb` |
| Week 6 | Sensitivity and model criticism | `05_sensitivity.ipynb`, validation notes |
| Weeks 7–8 (optional) | Decision memo and portfolio polish | Decision memo, reproducibility documentation |

**Estimated effort:** 6–8 weeks at 3–5 hours per week (approximately 24–35 hours). Start with **1,000 simulation trials** for verification, then expand to **10,000–20,000 trials**. Use a fixed seed and common random numbers or appropriately managed independent streams to make comparisons reproducible and fair.

## Validation criteria

The project is complete when:

- Input assumptions and units are documented, and data checks cover identifiers, bounds, categories, missingness, and logical consistency.
- Historical exposure accounts for assets installed after 2006; no future information leaks into baseline 2026 risk.
- Tests cover zero intensity, hazard monotonicity, and agreement between simulated Poisson means and theoretical intensity.
- Cash-flow timing and real-dollar discounting are consistent.
- Reported outputs include mean, median, P90, P95, and annual budget exceedance probabilities.
- Sensitivity analysis examines whether and when the preferred strategy changes.
- The final report separates synthetic demonstration, assumed engineering behavior, and real-world calibration requirements.

## Important limitations

This study deliberately simplifies several features of real wastewater systems. Failures are initially treated as independent, although storm, surge, operational, and geographic stresses may correlate events. Segment-level disruption estimates can overlap when a failure interrupts an entire force main. The synthetic historical records are not calibrated field evidence, and condition scores are proxies rather than inspected physical grades. Intervention eligibility, regulatory exposure, hydraulic adequacy, and construction feasibility require independent engineering review.

## Learning philosophy

The objective is to develop defensible engineering and programming judgment, not merely produce a visually convincing Monte Carlo plot. Model assumptions should be written before coding; functions should be attempted independently, checked with meaningful tests, and reviewed critically, including when AI assistance is used.

**A successful model is one whose failure logic, uncertainty, cost assumptions, and limitations can be explained and defended.**
