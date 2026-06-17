# Prompt — Improve the PyTorch Titanic baseline (`r_d_Lesson_2_Torch_Titanic.ipynb`)

Copy everything in the **Prompt** section below into the agent/chat together with the
notebook `r_d_Lesson_2_Torch_Titanic.ipynb`. It is written to be reproducible from the
baseline alone: it describes *what* to build and *why*, not a copy of any reference
solution.

---

## Prompt

You are an experienced ML engineer. Attached is a Jupyter notebook
`r_d_Lesson_2_Torch_Titanic.ipynb` that trains a small PyTorch MLP (`TitanicMLP`) to
predict survival on the seaborn Titanic dataset. Your job is the homework below.

### Goal

Understand the baseline code, then **redesign the model and the training procedure to
improve test-set quality by a relative +5–10%** over the original baseline. Then write a
project **`README.md`** documenting the work and explaining which changes helped and *why*.

### Hard constraints (make the comparison honest)

1. **Do not change the feature set or the preprocessing semantics.** Keep the same eight
   predictors (`pclass, age, sibsp, parch, fare, sex, embarked`, one-hot encoded,
   `dropna`). The improvement must come from **architecture + training procedure**, not
   from feature engineering, so the comparison is apples-to-apples.
2. **Keep the held-out test set identical** to the one the baseline is evaluated on
   (same `train_test_split(test_size=0.2, random_state=42, stratify=y)`). Never train,
   tune, early-stop, or select a threshold on the test set.
3. **Append your work to the existing notebook** — do not delete or overwrite the
   original baseline cells. The notebook must still run top-to-bottom and show both the
   original baseline result and your improved result.
4. **Reproducibility**: set `SEED = 42` on both `torch` and `numpy`, and re-seed before
   instantiating/training each model so runs are repeatable on CPU.
5. Production-quality code: PEP 8, full type annotations, NumPy-style docstrings,
   comments only for non-obvious intent. Prefer concise, idiomatic PyTorch.

### Step 1 — Diagnose the baseline's evaluation, then build a fair protocol

Critically read the baseline training/eval cells and identify why the reported number is
not a trustworthy model-selection signal. At minimum address:

- there is **no validation set** — the model trains for a hard-coded number of epochs and
  the **last-epoch** weights are used, with no way to detect over/under-fitting;
- the `StandardScaler` is fit on the **entire** training split (minor leakage relative to
  a proper train/val refit).

Build a reusable, fair training protocol and apply it identically to **every** model you
compare:

- Carve a **stratified validation set** (e.g. 20%) out of the *training* data only; leave
  the test set untouched.
- **Refit `StandardScaler` on the training fold only**; transform val and test with it.
- Write one `train_with_early_stopping(...)` helper that:
  - uses `AdamW` (`lr=1e-3`, `weight_decay=1e-4`) and `ReduceLROnPlateau` on the
    validation metric;
  - monitors **validation weighted F1** at the fixed `0.5` threshold each epoch;
  - applies **early stopping with `patience≈30`** and a generous `max_epochs` cap;
  - **restores the best (not last-epoch) weights** before returning;
  - returns per-epoch history (`train_loss`, `val_loss`, `val_f1`) plus `best_epoch`.
- Write one `evaluate(...)` helper returning `accuracy`, weighted `precision`/`recall`,
  `f1_weighted`, and `f1_macro` on a given set.

### Step 2 — Establish a fair baseline

Retrain the **original `TitanicMLP` architecture unchanged** under this new protocol
(call it `baseline_v2`) and evaluate it on the test set. This is the reference you must
beat, so that the final delta is attributable to the **architecture**, not the protocol.

### Step 3 — Design an improved architecture (`TitanicMLP2`)

Keep the same input/output contract (8 features in, single logit out, `BCEWithLogitsLoss`).
Design a stronger but still small tabular MLP and justify each choice. Use a combination
such as:

- **Learnable input normalisation** (`BatchNorm1d` on the raw input) so the network adapts
  to feature statistics rather than relying solely on the pre-fit scaler.
- **More capacity**: wider hidden width (e.g. 64 → 128) and added depth via **pre-activation
  residual blocks** (skip connections `x + f(x)`) to model non-linear interactions
  (`sex × pclass`, `age × fare`, …) while keeping gradients healthy.
- **`GELU` instead of `ReLU`** — smoother gradients, typically better for small tabular MLPs.
- A lightweight **per-feature gating** mechanism (e.g. squeeze-and-excitation:
  `Linear → GELU → Linear → Sigmoid`, multiply) to up/down-weight features.
- A **dedicated classification head** (`BN → GELU → Dropout → Linear → … → Linear(→1)`).
- **Uniform, milder dropout** (e.g. 0.25) since BatchNorm already regularises.

Train `TitanicMLP2` with the **same** `train_with_early_stopping` protocol and seed.

### Step 4 — Compare rigorously

- Print `classification_report` for `baseline_v2` and `TitanicMLP2` on the **same** test set.
- Build a comparison table of all metrics with **absolute** deltas
  (`delta = m_new - m_old`) and **relative** deltas
  (`delta_% = 100 * delta / m_old`).
- Plot: (a) validation weighted-F1 curves for both models with their `best_epoch` marked,
  and (b) a grouped bar chart of test metrics.
- Explicitly check and print whether the **relative weighted-F1 uplift ≥ 5%** (the target).
  Treat **weighted F1** as the primary metric (classes are imbalanced).

### Step 5 — Conclusions deliverable

Produce a concise write-up (markdown in the notebook is fine, plus a separate short
doc/slides outline) covering:

- the protocol bug in the original evaluation and how the fair protocol fixes it;
- each architectural change and the mechanism by which it should help on this dataset;
- the final results table (absolute + relative deltas) and whether the +5–10% goal was met;
- honest caveats (tiny 143-sample test set, CPU vs CUDA non-determinism).

### Acceptance criteria

- Notebook runs top-to-bottom without errors; original baseline cells are intact.
- A fair `baseline_v2` and an improved `TitanicMLP2` are both trained under one identical
  protocol and evaluated on the identical test set.
- Relative weighted-F1 improvement of `TitanicMLP2` over `baseline_v2` is **≥ 5%**.
- Code is typed, documented (NumPy-style docstrings), and reproducible with `SEED = 42`.
- A short conclusions document explains *what changed and why it helped*.

### Step 6 — Write the project `README.md`

After the notebook runs and the final numbers are known, write a `README.md` at the repo
root that documents the result. Use the **actual numbers produced by your run** (do not
invent them) and follow this structure and tone:

- **Title** comparing the two models, e.g. `# Titanic — TitanicMLP vs TitanicMLP2`.
- **One-paragraph summary** stating the headline result as a relative weighted-F1 uplift
  (e.g. "+X% relative weighted F1 vs. the baseline retrained under the same fair protocol")
  and a link to the notebook.
- **Quick start** — `uv sync`, `source .venv/bin/activate`, and an `nbconvert --execute`
  command to reproduce the notebook end-to-end.
- **Dataset** — the eight predictors and a note that the feature set is identical to the
  baseline, so the comparison reflects *architecture + training procedure*.
- **Results** — a markdown table on the held-out test set (threshold = 0.5) with one row
  per metric (accuracy, weighted precision/recall, weighted F1, macro F1) and columns for
  the baseline, `TitanicMLP2`, absolute delta, and relative delta; bold the weighted-F1 row.
- **Architectural improvements (`TitanicMLP2`)** — show the baseline and new architecture
  as fenced code blocks (layer stack + the `ResBlock` formula), then a numbered list of
  each change and *why it helps on this dataset*.
- **Training-procedure improvements** — bullet list of the fair-protocol changes that apply
  equally to both models (stratified train/val split, scaler refit on train fold only,
  `AdamW`, `ReduceLROnPlateau`, early stopping with best-weights restoration, seeding),
  and a sentence contrasting them with the original baseline-evaluation cells.
- **Reproducibility caveats** — CPU vs CUDA non-determinism, tiny 143-sample test set.
- **File layout** — a tree of the key files.

Keep the writing concise, technical, and formatted like a polished open-source README
(short paragraphs, fenced code blocks, tables, inline `code` for identifiers).
