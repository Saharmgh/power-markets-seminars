# Power Markets and Regulations - Seminar Notebooks

| Notebook | Open in Colab |
|----------|:-------------:|
| Economic Dispatch (Basic) | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Saharmgh/power-markets-seminars/blob/main/seminars/01_economic_dispatch/Economic_Dispatch_Basic.ipynb) |
| Economic Dispatch (Enhanced) | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Saharmgh/power-markets-seminars/blob/main/seminars/01_economic_dispatch/Economic_Dispatch_Enhanced.ipynb) |
| Unit Commitment | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Saharmgh/power-markets-seminars/blob/main/seminars/02_unit_commitment/Unit_Commitment.ipynb) |
| Optimal Power Flow | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Saharmgh/power-markets-seminars/blob/main/seminars/03_optimal_power_flow/Optimal_Power_Flow.ipynb) |

**Course:** Power Markets and Regulations
**TA:** Sahar Moghimian

---

## Overview

This repository contains hands-on seminar notebooks for the **Power Markets and Regulations** course. The seminars progressively build up from basic economic dispatch to unit commitment and optimal power flow, covering the core optimization problems that underpin electricity market operations.

All notebooks use **Python** with [Pyomo](https://www.pyomo.org/) for optimization modeling and [HiGHS](https://highs.dev/) as the open-source solver.

## Seminars

| # | Topic | Description |
|---|-------|-------------|
| 1 | [Economic Dispatch](seminars/01_economic_dispatch/) | Merit order, supply curves, single/multi-period dispatch, ramp constraints, duck curve sensitivity |
| 2 | [Unit Commitment](seminars/02_unit_commitment/) | Binary commitment decisions, startup/shutdown costs, minimum up/down times, auxiliary variables for ramps, spinning reserves |
| 3 | [Optimal Power Flow](seminars/03_optimal_power_flow/) | DC-OPF formulation, phase angles, susceptance, 3-bus and IEEE 14-bus test systems, locational marginal prices |

### Seminar 1: Economic Dispatch
- **Basic version:** Direct translation of the original Julia/JuMP seminar. Covers single-period ED, multi-period ED, ramp constraints, and the California duck curve.
- **Enhanced version:** Adds LMP extraction, real CAISO data exploration

### Seminar 2: Unit Commitment
Three progressive stages:
1. **Simple UC** - Binary commitment variables (COMMIT, START, SHUT), minimum up/down time constraints
2. **UC with ramps** - Auxiliary variable GENAUX for handling ramps during startup/shutdown
3. **Full UC with reserves** - Spinning reserve requirements (RESUP, RESDN), contingency reserves

### Seminar 3: Optimal Power Flow
- **3-bus system** - Two generators, one load bus, three transmission lines. Demonstrates how power flows split across parallel paths.
- **IEEE 14-bus system** - Standard test case with 2 generators, 11 loads, 20 lines. Includes a student exercise template.
- **LMP analysis** - Shows how congestion creates price separation across buses (including the $150/MWh Bus 3 example).

## Getting Started

### Option 1: Google Colab (recommended for students)

Click the Colab badge above or open any notebook directly in Colab. Each notebook includes a `pip install` cell that installs all dependencies automatically.

### Option 2: Local installation

```bash
# Clone the repository
git clone https://github.com/Saharmgh/power-markets-seminars.git
cd power-markets-seminars

# Create a virtual environment (recommended)
python -m venv .venv
source .venv/bin/activate  # macOS/Linux
# .venv\Scripts\activate   # Windows

# Install dependencies
pip install -r requirements.txt

# Launch Jupyter
jupyter notebook
```

### Prerequisites

- Python 3.9 or later
- No commercial solver licenses needed -- all notebooks use the open-source HiGHS solver

## Data Sources

All data is loaded directly from GitHub at runtime. No local data files are required. The datasets come from:

- **Economic Dispatch & Unit Commitment:** SDG&E-based system via [PowerGenome](https://github.com/gschivley/PowerGenome), including 25-33 generators, hourly demand, and variable generation capacity factors.
- **Optimal Power Flow:** Modified 3-bus test case and [IEEE 14-bus test system](https://icseg.iti.illinois.edu/ieee-14-bus-system/) in MATPOWER format.

## Acknowledgments

The seminar materials are adapted from the **Power Systems Optimization** course developed by:

- **Prof. Michael R. Davidson** (UC San Diego) - MAE 243: Electric Power Systems Modeling
- **Prof. Jesse D. Jenkins** (Princeton) - MAE/ENE 539: Optimization Methods for Energy Systems Engineering

Original Julia/JuMP course materials: [Power-Systems-Optimization-Course](https://github.com/Power-Systems-Optimization-Course/power-systems-optimization)

The Python/Pyomo adaptation was created to make the material more accessible to students already familiar with Python.

## References

1. Davidson, M.R. & Jenkins, J.D. (2022). *Power Systems Optimization*. GitHub repository.
2. Kirschen, D.S. & Strbac, G. (2018). *Fundamentals of Power System Economics*. 2nd edition. Wiley.
3. Morales-Espana, G., Latorre, J.M., & Ramos, A. (2013). Tight and Compact MILP Formulation of Start-Up and Shut-Down Ramping in Unit Commitment. *IEEE Transactions on Power Systems*, 28(2), 1288-1296.
4. Knueven, B., Ostrowski, J., & Watson, J.-P. (2019). On Mixed Integer Programming Formulations for the Unit Commitment Problem. *Optimization Online*.
5. Zimmerman, R.D., Murillo-Sanchez, C.E., & Thomas, R.J. (2011). MATPOWER: Steady-State Operations, Planning, and Analysis Tools for Power Systems Research and Education. *IEEE Transactions on Power Systems*, 26(1), 12-19.

## License

MIT License. See [LICENSE](LICENSE) for details.
