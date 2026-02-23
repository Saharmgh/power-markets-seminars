# Setup Guide

This guide helps you set up your environment to run the Power Markets seminar notebooks.

## Option 1: Google Colab (Easiest)

1. Go to [Google Colab](https://colab.research.google.com/)
2. Click **File > Open notebook > GitHub**
3. Enter the repository URL and select the notebook you want to run
4. Each notebook has a `pip install` cell at the top -- run it first to install dependencies
5. You're ready to go!

**Advantages:** No local setup needed, free GPU/CPU, works on any device with a browser.

## Option 2: Local Setup

### Step 1: Install Python

Download and install Python 3.9+ from [python.org](https://www.python.org/downloads/).

Verify your installation:
```bash
python --version  # Should show 3.9 or later
```

### Step 2: Clone the Repository

```bash
git clone https://github.com/Saharmgh/power-markets-seminars.git
cd power-markets-seminars
```

### Step 3: Create a Virtual Environment

```bash
python -m venv .venv

# Activate it:
source .venv/bin/activate      # macOS / Linux
# .venv\Scripts\activate       # Windows (Command Prompt)
# .venv\Scripts\Activate.ps1   # Windows (PowerShell)
```

### Step 4: Install Dependencies

```bash
pip install -r requirements.txt
pip install jupyter
```

### Step 5: Launch Jupyter

```bash
jupyter notebook
```

Navigate to the `seminars/` folder and open the notebook you want to run.

## Troubleshooting

### "No module named 'pyomo'"
Make sure you activated your virtual environment and ran `pip install -r requirements.txt`.

### "appsi_highs is not available"
The HiGHS solver interface requires `highspy`. Install it with:
```bash
pip install highspy
```

### Solver runs but returns infeasible
Check your input data and constraints. Some exercises intentionally create infeasible problems to demonstrate concepts -- read the instructions carefully.

### Plotly charts don't render in Jupyter
Install the Jupyter Plotly extension:
```bash
pip install jupyter nbformat
jupyter nbextension enable --py plotly
```
Alternatively, use JupyterLab which has built-in Plotly support:
```bash
pip install jupyterlab
jupyter lab
```

## Package Versions

The notebooks have been tested with:
- Python 3.10+
- Pyomo 6.7+
- highspy 1.5+
- pandas 2.0+
- plotly 5.18+
- matplotlib 3.7+
