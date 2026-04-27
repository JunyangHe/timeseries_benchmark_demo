# Agent-Ready Time Series Benchmark Standards

Use this document as context when building a standardized time-series benchmarking repo for students’ final projects.

**How this maps to the course tutorials:** **Tutorial 1** (GitHub, **Section 3** repo layout, **Section 5** Colab) · **Tutorial 2** (dataset choice, `data/raw/`, **`metadata/ts_characteristics.json`**, sanity checks, preprocessing **without** train/validation/test splits—zero-shot inference later) · **Tutorial 3** (`experiment_config.json`, `benchmark_results.json`, `main.ipynb`, RMSE from past-only evaluation) · **Tutorial 4** (consistency checks, README replication text, **Findings/Discussion** for data and forecasts). Follow this file **and** those tutorials together.

## 1. Purpose

You are helping create a time-series benchmark project that should be reusable for future aggregation of benchmark results across datasets and models. The repo must produce structured, machine-readable metadata and outputs, plus a **README** that supports replication and—where relevant—**honest reporting** of data issues and evaluation outcomes (see **Section 2.5** and **Section 3.1**).

## 2. Required behavior

Before coding:

1. ask clarifying questions about the task
2. ask for data context when needed (see Section 2.1)
3. summarize the planned repo structure
4. explain the files and architecture to be created
5. then implement code step by step

Do not create one giant monolithic script unless explicitly requested.

### 2.1 Data context—ask during planning and clarifying

To write correct download and preprocessing code, you need visibility into the dataset. During the clarifying-questions stage, ask the user for:

**If the dataset is non-standard or custom:** Request a small sample (e.g., first 10–20 rows) or paste column names and dtypes so you can tailor the code.

**If using a different dataset than the tutorial example:** Request a description of the structure (column names, time column, target variable, file format) or a small sample before writing preprocessing code.

**If preprocessing fails later:** Ask the user to share the error message and, when useful, a small sample or `df.head()` output so you can fix the code.

For widely used public datasets with a stable, documented layout, you can proceed without a sample when that layout is clear. For anything else, insist on structure information or a sample before implementing data loading and preprocessing.

### 2.2 Model context—ask during planning and clarifying

To write correct model setup and inference code, ask the user for a link to the model's GitHub or Hugging Face page when available. This gives you the API, usage examples, and parameter documentation needed to generate accurate code. For widely documented foundation models you can proceed without a link when usage is standard; for custom or lesser-known models, request the link before implementing evaluation code.

### 2.3 Dataset zip committed to the repository

**You (the agent) should package the dataset the student will actually use for the benchmark and commit that artifact to the repo:**

1. After download or preprocessing, identify the **exact files or folder** the notebooks load (the student’s “working” dataset—possibly a subset of a larger public dataset).
2. **Create a single zip archive** (e.g. `data/dataset.zip` or `data/benchmark_dataset.zip`) containing those files, using a clear, stable filename.
3. **Commit the zip file to Git** as part of the project so graders and collaborators can reproduce runs without re-downloading from external hosts (unless licensing forbids redistribution—in that case, ask the student and document download-only steps in the README instead).
4. Ensure **`.gitignore` does not exclude** this zip if it is meant to be tracked. If the archive would exceed **GitHub’s file-size limits** (~100 MB per file hard limit; smaller is better), work with the student to use a **smaller subset**, [Git LFS](https://git-lfs.com/), or an instructor-approved hosting approach; do not commit multi-gigabyte blobs blindly.
5. In **README** (Data details / replication), state the zip path, how to unzip or load it in code (`zipfile` / `pandas.read_csv` from extracted paths), and checksum or size if useful.

Preprocessing code should read from the **extracted** layout under `data/raw/` or load directly from the zip if you implement that—be consistent everywhere.

### 2.4 Metadata and results files—create if missing, keep in sync

**`metadata/experiment_config.json`**, **`metadata/ts_characteristics.json`**, and **`results/benchmark_results.json`** must always exist in the repo once work begins, and must **reflect the current pipeline**. Follow this lifecycle:

1. **If a file does not exist**, **create it** immediately using the schemas in **Sections 3.2, 4, and 5**. For string/object fields use empty strings, empty objects, or `null` when unknown; for **`ts_characteristics.json`** integer counts use non-negative integers or `null` until counts are known (**Section 3.2**). Do not skip the file.
2. **Whenever** anything changes that belongs in those artifacts—e.g. lookback window, forecast horizon, model name or link, `model_parameters`, target variable, task type, **`input_mode`**, stream counts, instance count, **`data_modality`**, execution environment, dataset name, or evaluation setup—**update the affected file(s)** in the same change (or the same PR/session) as the code or notebook edits. Keep **`ts_characteristics.json`** counts consistent with the notebook’s actual columns and series layout. Do not leave metadata stale relative to the notebook or `src/`.
3. **After** a successful evaluation run, **update `benchmark_results.json`** with the computed **RMSE** and any final `model_parameters` / `notes` so results match what the code produced.
4. If the user refactors configs (moves paths, renames variables, switches model), **re-read the repo**, patch **`experiment_config.json`**, **`ts_characteristics.json`**, and **`benchmark_results.json`**, and confirm fields still match **Sections 3.2, 4–5** and **Section 7** (RMSE only).

Treat these files as **living metadata**, not one-time boilerplate.

### 2.5 Report anomalies in the data and in evaluation outcomes (with explanations)

You must **surface problems clearly** in chat **and** ensure the student **writes the explanations into `README.md`** (e.g. a **Findings**, **Discussion**, or **Data quality** section—**Tutorial 4, Section 3**). Chat alone is not enough: the README is the durable record for graders and collaborators. Use **`benchmark_results.json` → `notes`** only for short caveats that belong in machine-readable metadata; put the **full narrative** (what was wrong, why it matters, what you did or could not fix) in the **README**.

Do **not** ignore red flags.

**Dataset and preprocessing anomalies**

When you help with loading, sanity checks (**Tutorial 2**), or preprocessing, watch for issues such as: long gaps or irregular timestamps, heavy missingness, obvious outliers or impossible values, very short history, constant or near-constant targets, duplicate times, suspicious joins, or a subset that may not match the stated task. If you see them (from samples the user provides, from described structure, or from generated diagnostic output):

- **Say so explicitly** in your reply and briefly **explain why it matters** for forecasting or for comparing RMSE (e.g. non-stationarity, unreliable baselines, leakage if future values enter model inputs).
- Suggest concrete next steps (plots already in Tutorial 2, imputation vs drop, date filters, documenting limitations).
- **Populate `README.md`:** add or extend **Findings** / **Discussion** (or **Data quality**) with those explanations—not only a bullet in chat (**Tutorial 4, Section 3**).

**Final metrics and evaluation anomalies**

After or while implementing evaluation:

- **Sanity-check RMSE** against the scale of the target (order of magnitude, units). If RMSE is missing, `NaN`, absurdly large vs the series, or **inconsistent** with plots or printed errors, **report the discrepancy** and hypothesize **root causes tied to evidence** (wrong column, future leakage into inputs, shape mismatch, forgotten `inverse_transform`, evaluation mask bug, model failure)—not vague hand-waving.
- If forecasts are systematically poor, note **plausible drivers** linked to the setup (lookback vs seasonality, evaluation period spanning a regime change, model–data mismatch) and recommend what to verify next. **Add the same reasoning to `README.md`** (Findings/Discussion) so poor RMSE is interpreted in context, not left unexplained.

Use **`benchmark_results.json` → `notes`** for short, machine-readable caveats that affect cross-repo comparison (e.g. “evaluated on subset after dropping 30% missing dates”). **Always** mirror or expand the substantive explanation in **`README.md`**—students should not rely on `notes` alone for anomaly write-ups.

## 3. Required repo outputs

The repo should contain at least:

- `README.md` (replication + results; rubric sections in **Section 3.1** when the instructor uses them; **Tutorial 4, Section 3** for Findings on data and forecasts)
- `requirements.txt` or `environment.yml` (for environment reproducibility)
- **a committed zip of the student’s benchmark dataset** under `data/` (see Section 2.3), unless licensing or size makes that impossible—then document the alternative in the README
- `metadata/experiment_config.json` (create if missing; update whenever experiment config changes—**Section 2.4**)
- **`metadata/ts_characteristics.json`** (numeric stream/instance summary + **`data_modality`**; create if missing; update when layout or modality changes—**Section 2.4**, schema **Section 3.2**, **Tutorial 2 §3**)
- `results/benchmark_results.json` (create if missing; update whenever results or run parameters change—**Section 2.4**)
- **`notebooks/main.ipynb`** as the primary notebook for the full pipeline (load/preprocess, model run, evaluation, plotting) unless the course specifies otherwise; put reusable logic in `src/` modules
- saved plots under `results/figures/`

### 3.1 README: rubric sections and tutorial alignment

**Tutorial 4** asks for clear **run instructions**, **output locations**, and a **Findings/Discussion** area when data quality or forecast performance needs explanation. When the course provides a formal rubric, the table below applies; otherwise, still ensure replication steps and honest discussion of anomalies (**Section 2.5**) are not missing.

The README should include these sections when the course rubric requires them:

| Section | Required content |
|---------|------------------|
| Project title, introduction, and team members | Clear title, brief description, names of all team members |
| Problem Statement | What forecasting problem is being solved |
| Data details | Dataset description, variables, time range, preprocessing |
| Experiment process | Experiment setup, model choice, evaluation pipeline |
| Results | **RMSE** (primary evaluation metric); include figures |
| How to set the project environment and replicate the results | Step-by-step instructions (Python version, dependencies, Colab setup) |
| The necessary codes and configuration to run your project | Where to find code, how to run notebooks, config files |
| Link to your dataset | Direct URL or instructions to obtain the dataset |

### 3.2 `metadata/ts_characteristics.json` schema

**Create** `metadata/ts_characteristics.json` if it is missing; **update** it whenever stream counts, **`number_of_instances`**, or **`data_modality`** change (**Section 2.4**, **Tutorial 2 §3**). Use **exactly** this shape: non-negative **integers** for the `number_of_*` fields, and a string for **`data_modality`**. Use `null` for any count that is not yet known. Univariate vs multivariate belongs in **`experiment_config.json` → `input_mode`** only (must match the notebook).

```json
{
  "number_of_observed_streams": "",
  "number_of_target_streams": "",
  "number_of_exogenous_streams": "",
  "number_of_exogenous_static_variables": "",
  "number_of_instances": "",
  "data_modality": "numerical"
}
```

| Key | Meaning |
| --- | --- |
| **`number_of_observed_streams`** | Count of past input streams (features/columns) the model uses. |
| **`number_of_target_streams`** | Count of target streams being predicted. |
| **`number_of_exogenous_streams`** | Count of additional time-varying exogenous inputs (use **`0`** if none). |
| **`number_of_exogenous_static_variables`** | Count of time-invariant exogenous variables (use **`0`** if none). |
| **`number_of_instances`** | Count of benchmark instances (e.g. separate time series, locations, or entities—match how your notebook defines a unit of evaluation; document in README if non-obvious). |
| **`data_modality`** | e.g. **`"numerical"`**; use a short lowercase label consistent across repos. |

## 4. Experiment config schema

**Create** `metadata/experiment_config.json` if it is missing; **update** it whenever experiment configuration changes (**Section 2.4**). Use fields like:

```json
{
  "model_name": "",
  "model_link": "",
  "task_type": "",
  "target_variable": "",
  "input_mode": "",
  "lookback_window": null,
  "forecast_horizon": null,
  "evaluation_metric": "rmse",
  "execution_environment": "",
  "model_parameters": {}
}
```

- `evaluation_metric`: use **`"rmse"`** only for this course (matches **Tutorial 3**). Do not add other metrics unless the instructor explicitly allows it. If an older template used `evaluation_metrics` as an array, normalize to this single-metric field when updating files.
- `input_mode`: **`univariate`** or **`multivariate`**; must match the actual columns used in the notebook (**Tutorial 2 §3**, **Tutorial 3**).
- `model_link`: URL to the model's GitHub or Hugging Face page (optional but recommended; helps the agent generate correct inference code)

**Required model run parameters:** For each time-series foundation model evaluated, record the specific parameters used. At minimum, include:

- `lookback_window`: number of past time steps used as input (context length)
- `forecast_horizon`: number of future steps predicted
- `model_parameters`: any model-specific settings (e.g., temperature, top_p, max_length, device). Use an object so different models can have different keys.

## 5. Benchmark results schema

**Create** `results/benchmark_results.json` if it is missing; **update** it whenever run parameters or reported results change (**Section 2.4**). Use fields like:

```json
{
  "dataset_name": "",
  "model_name": "",
  "target_variable": "",
  "lookback_window": null,
  "forecast_horizon": null,
  "model_parameters": {},
  "rmse": null,
  "notes": ""
}
```

**Include run parameters with results:** Each benchmark run must record the exact parameters used (lookback window, forecast horizon, and any model-specific settings). This enables fair comparison and reproducibility when aggregating across student submissions. Add optional fields if useful, but keep the core schema stable.

## 6. Task-definition rules

When processing a time-series benchmark, explicitly identify:

- dataset name
- application domain
- target variable
- lookback window (context length used as input)
- forecast horizon
- model-specific run parameters (for each foundation model evaluated)
- whether forecasting is single-step or multi-step
- whether setup is univariate or multivariate
- whether exogenous variables are used
- that evaluation uses **RMSE only** (Section 7)
- where outputs are stored

If the model is multivariate-capable but the benchmark is framed around a single target, keep the target variable explicit and record supporting variables as observed or exogenous streams where appropriate.

## 7. Metric rules

**Evaluation uses RMSE only.** Do not compute, report, or store other metrics in `experiment_config.json`, `benchmark_results.json`, README results, or plotting captions unless the instructor explicitly asks for additional metrics.

### RMSE

```text
RMSE = sqrt(mean((y_true - y_pred)^2))
```

Report RMSE in the README **Results** section and in `benchmark_results.json`. Use the same definition everywhere (e.g. same **scoring timestamps** / evaluation rule and **units**).

## 8. Repo structure guidance

**Standard layout** (names of Python modules under `src/` may vary, but these directories and key files should match):

```text
project_root/
├── README.md                    # Must satisfy rubric (Section 3.1)
├── requirements.txt             # or environment.yml
├── data/
│   ├── dataset.zip              # committed zip of benchmark dataset when policy allows (Section 2.3)
│   ├── raw/                     # extracted or downloaded content as needed
│   └── processed/
├── metadata/
│   ├── experiment_config.json
│   └── ts_characteristics.json    # streams/modality (Tutorial 2 §3; Section 3.2)
├── results/
│   ├── benchmark_results.json
│   └── figures/
├── notebooks/
│   └── main.ipynb               # primary pipeline: load, preprocess, evaluate, plot
└── src/
    ├── data_utils.py            # or equivalent modular layout
    ├── eval_utils.py
    └── plotting_utils.py
```

Keep **`notebooks/main.ipynb`** and **`src/`** imports consistent with this tree (working directory, relative paths). This matches the **Tutorial 1, Section 3** diagram students use to verify their clone.

## 9. Coding workflow guidance

When asked to build the repo:

1. ask clarifying questions (including data context per Section 2.1 and model context per Section 2.2 when needed)
2. propose architecture and file list
3. **create or update** `metadata/experiment_config.json`, **`metadata/ts_characteristics.json`**, and `results/benchmark_results.json` as soon as paths exist—**create placeholders if missing** (Section 2.4); expand or revise them whenever configs change in later steps
4. implement data ingestion first
5. create the **dataset zip**, place it under `data/`, and ensure it is **committed** (Section 2.3)
6. implement preprocessing next (**Tutorial 2**: clean series for zero-shot use—**no** train/validation/test split; transforms only as the foundation model requires); **update `experiment_config.json`** and **`ts_characteristics.json`** if task fields, stream/instance counts (**Section 3.2**), or input setup change; **flag data anomalies** per **Section 2.5**
7. **create or update** `experiment_config.json` and **`ts_characteristics.json`** to match the evaluation setup (model, horizons, environment, metrics per Section 7; **`input_mode`** per **Section 4**; counts per **Section 3.2**)
8. implement model evaluation
9. compute RMSE; **validate plausibility** and **report anomalies** per **Section 2.5**
10. save plots
11. **update `benchmark_results.json`** with RMSE, parameters, and **`notes`** (caveats from **Section 2.5**); keep aligned with `experiment_config.json` and **`ts_characteristics.json`** (**Section 2.4**)
12. generate README: rubric sections when required + **Findings/Discussion** for data and metrics (**Section 3.1**, **Tutorial 4 §3**)
13. create requirements.txt or environment.yml
14. review repo consistency

## 10. Output style guidance

- keep **`notebooks/main.ipynb`** readable by calling **`src/`** helpers for heavy logic rather than one endless cell block
- use clear filenames
- write comments for beginners
- keep outputs reproducible in structure even if exact values differ
- leave uncertain fields as `null` rather than inventing values
- when anomalies exist, prefer explicit **README** + **`notes`** text over silence (**Section 2.5**)

## 11. Example task instantiation (illustrative)

One possible project (replace with the student’s actual dataset, domain, target, and model):

- dataset: e.g. a public time-series benchmark the course assigns
- domain: e.g. energy, finance, environment—whatever matches the data
- target: the series or column being forecast
- task: e.g. single-step forecasting
- model: the foundation or baseline model the student evaluates
- lookback_window / forecast_horizon: set to match the code
- execution: e.g. `google_colab` or `local`
- metrics: **RMSE only** (Section 7)

Example-shaped metadata (placeholders—not a required dataset or model):

```json
{
  "dataset_name": "example_dataset",
  "application_domain": "example_domain",
  "target_variable": "example_target",
  "task_type": "single_step_forecasting",
  "model_name": "example_model",
  "lookback_window": 64,
  "forecast_horizon": 1,
  "model_parameters": {},
  "evaluation_metric": "rmse"
}
```

## 12. Final repo review requirement

After implementation, perform a consistency check:

- is the **dataset zip** present under `data/`, documented in the README, and committed when policy allows (Section 2.3)?
- are `experiment_config.json`, **`metadata/ts_characteristics.json`**, and `benchmark_results.json` present? (If the user deleted them, **recreate** from Sections 3.2, 4–5—**Section 2.4**)
- were these files **updated** the last time experiment config, stream metadata, or results changed? (no stale fields—**Section 2.4**)
- are lookback window and forecast horizon recorded in experiment_config and benchmark_results?
- do these parameters match the values used in the evaluation code?
- are model-specific run parameters documented (for each foundation model)?
- do metric names match across README, code, and results?
- are outputs machine-readable?
- are plots saved?
- is the task definition explicit?
- **does the README satisfy the rubric (Section 3.1)** when the course uses it? Check: project title/intro/team, problem statement, data details, experiment process, results, environment setup, codes/config, dataset link.
- does the README include **Findings/Discussion** for notable **data limitations** or **weak/confusing metrics** (**Tutorial 4, Section 3**; **Section 2.5**)?
- are important caveats reflected in **`benchmark_results.json` → `notes`** where they affect interpretation?
- is requirements.txt or environment.yml present?

If something is missing, list it clearly—including any **unresolved anomalies** the student should address or document.