# sktime-llm-agent

#  LLM-based Forecasting Agent for sktime

> *LLM acts as a planner, sktime does the forecasting.*

---

##  Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Component Breakdown](#component-breakdown)
- [Key Principles](#key-principles)
- [Initial Scope (v1)](#initial-scope-v1)

---

##  Overview

The LLM-based Forecasting Agent for sktime is a modular, AI-powered time series
forecasting system. It leverages a Large Language Model (LLM) as an intelligent planner
to interpret natural language goals, select the right forecasting strategy, and construct
a pipeline — while sktime handles all actual forecasting execution in a deterministic and
reproducible manner.

This separation of concerns makes the system:

- **Flexible and user-friendly** — natural language interface
- **Deterministic and auditable** — sktime backend
- **Extensible and backend-agnostic** — swap any component independently

---

## Architecture

The system follows a 7-stage pipeline:

```
User Input
    ↓
LLM Planner (Reasoning & Planning)
    ↓
Sktime Tool / API Layer (Backend-agnostic)
    ↓
Sktime Execution Layer (Deterministic)
    ↓
Output
    ↓
Explanation (Optional)
    ↑
Feedback Loop (Iterative Improvement)
```

---

## 🔧 Component Breakdown

---

### 1. User Input

The system accepts:

| Input Type | Description | Example |
|---|---|---|
| **Natural Language Goal** | Plain-text description of the forecasting task | *"Forecast next 12 monthly sales with seasonality and trend."* |
| **Time Series Data** | Raw time series to be forecasted | Monthly sales figures |
| **Exogenous Variables** *(optional)* | External variables influencing the forecast | Marketing spend, holidays |

---

### 2. LLM Planner — Reasoning & Planning

The LLM acts as the **brain** of the system. It does not perform forecasting — it reasons
about the task and creates a structured plan across four sub-steps:

| Sub-step | What It Does |
|---|---|
| **Task Understanding** | Identifies task type, detects trend, seasonality, horizon, and metrics |
| **Model & Strategy Selection** | Selects model family (AutoARIMA, ETS, LightGBM, etc.) and relevant transformations |
| **Pipeline Planning** | Creates sktime pipeline steps and parameters, outputs structured JSON/YAML |
| **Plan Output** | Complete machine-readable specification passed to the execution layer |

**Example plan output:**

```json
{
  "model": "AutoARIMA",
  "transformations": ["log"],
  "fh": "1..12",
  "scoring": ["MASE", "RMSE"],
  "seasonality": 12
}
```

---

### 3. Sktime Tool / API Layer — Backend-agnostic

Provides a unified interface to sktime's capabilities:

| Module | Description |
|---|---|
| **Model Library** | All sktime forecasters: AutoARIMA, ETS, Theta, LightGBM, and more |
| **Transformations** | BoxCox, Log, Differencing, Scaling, Fourier, Detrending, etc. |
| **Detectors** | Seasonality, Trend, Change Point, Outlier, Stationarity detection |
| **Metrics** | MASE, RMSE, MAPE, sMAPE, CRPS, and more |
| **Data Utilities** | Split, CV, Horizon Handling, Alignment, Missing Values management |
| **Documentation Context** | sktime docs, tutorials, and examples for grounding the LLM |

---

### 4. Sktime Execution Layer — Deterministic

Once the LLM produces a plan, this layer executes it deterministically:

```
Build Pipeline → Fit → Predict → Evaluate
```

| Step | Description |
|---|---|
| **Build Pipeline** | Constructs the pipeline from the LLM's plan using sktime components |
| **Fit** | Fits the constructed pipeline on the training data |
| **Predict** | Generates forecasts for the specified prediction horizon |
| **Evaluate** | Computes metrics (MASE, RMSE, etc.) and diagnostics |

**Results object includes:** predictions (ŷ), metrics, metadata, and a fully
sktime-compatible results object.

---

### 5. Output

| Output | Description |
|---|---|
| **Forecast (ŷ)** | Predicted future values with confidence intervals |
| **Metrics** | Performance scores — MASE, RMSE, etc. |
| **Model Used** | Model and configuration selected by the LLM |
| **Transformations Applied** | All preprocessing steps applied to the data |
| **Diagnostics** | Residual plots, error analysis, and diagnostic information |

---

### 6. Explanation *(Optional)*

The LLM can generate a human-readable explanation covering:

- Why this specific model was chosen
- What transformations were applied and why
- Assumptions made during planning
- Known limitations of the selected approach
- Suggested next steps for improvement

---

### 7. Feedback Loop — Iterative Improvement

If performance is unsatisfactory or the user requests refinement:

```
Analyze Results (LLM)
        ↓
  Refine Plan (LLM)
        ↓
Adjust Model / Params / Transformations
        ↓
     Re-execute
```

This loop continues until the user is satisfied or a stopping criterion is met.

---

## Key Principles

| Principle | Description |
|---|---|
| **LLM = Planner (not predictor)** | The LLM reasons and plans — it never directly computes forecasts |
| **Sktime = Executor** | All actual forecasting is handled by sktime's deterministic algorithms |
| **Deterministic, reproducible results** | Given the same plan and data, results are always identical |
| **Backend-agnostic LLM interface** | Works with any LLM backend — OpenAI, Anthropic, local models, etc. |
| **Modular, extensible, sktime-native** | Each component can be swapped or extended independently |

---

##  Initial Scope (v1)

| Feature | Status |
|---|---|
| Forecasting only (no classification or regression) | ✅ Included |
| No complex pipelines — single-model focus | ✅ Included |
| Basic model selection | ✅ Included |
| Simple transformations | ✅ Included |
| Backend-agnostic LLM interface | ✅ Included |

---

> **Note:** This system is sktime-native — all outputs are fully compatible with the
> sktime ecosystem. The LLM's role is strictly limited to planning and explanation.
> No probabilistic or ML computation is performed by the LLM itself.

---