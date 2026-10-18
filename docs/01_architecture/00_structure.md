# Architecture

```text
force-main-risk-mc/
├── data/
│   └── raw/
│       └── force_main_segments_synthetic.csv
├── docs/
│   ├── 00_overview/
│   │   └── 00_force_main_monte_carlo_scope.md
│   ├── 01_architecture/
│   │   └── 00_structure.md
│   ├── 02_assumptions/
│   │   ├── 00_model_assumptions.md
│   │   └── 01_data_dictionary.md
├── notebooks/
│   ├── 01_ingestion.ipynb
│   ├── 02_eda.ipynb
│   ├── 03_monte_carlo.ipynb
│   ├── 04_strategy_comparison.ipynb
│   └── 05_sensitivity.ipynb
├── src/force_main_risk_mc/
│   ├── main
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