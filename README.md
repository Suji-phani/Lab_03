#  Automated Machine Learning Model Retraining with GitHub Actions

This repository demonstrates how to use **GitHub Actions** to automatically retrain, evaluate, and version a machine learning model whenever code is pushed to the main branch.  
Each workflow run generates a uniquely timestamped model and evaluation metrics — providing continuous integration and version control for your ML lifecycle.

---

##  Overview

Machine learning models evolve over time as new data becomes available.  
This project automates that process using **GitHub Actions**, ensuring:
- Continuous retraining and evaluation of the model  
- Versioning via timestamps for traceability  
- Automatic commit and push of trained models and metrics

---

##  Workflow Summary

**Workflow File:** `.github/workflows/model_retraining_on_push.yml`  
**Trigger:** Every push to the `main` branch  

| Step | Description |
|------|--------------|
|  **Checkout Repository** | Retrieves the latest code and data |
|  **Set up Python** | Installs Python 3.11 on the GitHub runner |
|  **Install Dependencies** | Installs libraries from `requirements.txt` |
|  **Train Model** | Runs `train_model.py` to train a model with timestamped versioning |
|  **Evaluate Model** | Runs `evaluate_model.py` to assess model performance |
|  **Commit Models & Metrics** | Saves model and metrics files and pushes them back to the repo |

---

##  Model Training (`src/train_model.py`)

The training script:
- Loads the dataset (`data/dataset.csv`) or generates synthetic data.
- Preprocesses numerical and categorical columns.
- Trains a **RandomForestRegressor** model.
- Saves:
  - The model pipeline (`.joblib`)
  - Model metadata (`.json`)
  - Evaluation metrics (`.json`)
- Each file is timestamped for clear version tracking.

---

## 📊 Model Evaluation (`src/evaluate_model.py`)

This script:
- Loads the trained model.
- Evaluates it on test data.
- Prints and saves performance metrics (e.g., **R²**, **MAE**, or accuracy**).  
- Outputs results to the `metrics/` directory.

---

##  GitHub Actions Workflow

Below is the YAML file that runs the retraining automation:

```yaml
name: Model Retraining on Push

on:
  push:
    branches:
      - main

permissions:
  contents: write

jobs:
  retrain:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      - name: Train model
        run: |
          TS=$(date +%Y%m%d_%H%M%S)
          echo "Using timestamp: $TS"
          python src/train_model.py --timestamp "$TS"

      - name: Evaluate model
        run: |
          TS=$(date +%Y%m%d_%H%M%S)
          echo "Using timestamp: $TS"
          python src/evaluate_model.py --timestamp "$TS" || echo "Skipping evaluation if not implemented"

      - name: Commit models & metrics
        run: |
          git config --global user.email "actions@github.com"
          git config --global user.name "github-actions[bot]"
          git add models metrics || true
          git commit -m "CI: add model and metrics" || echo "No changes to commit"
          git push



