# Local execution and reproducibility

## 1. Reference environment

Validation used **macOS ARM64, Python 3.11.14 and Homebrew libomp 23.1.0**.
The Python reference versions are pinned in [requirements.txt](../requirements.txt).
This concise specification pins the project dependencies and selected runtime
dependencies; it is not a complete lock of all transitive packages.
Linux and Windows execution have not been validated.

## 2. Project directory

The commands below assume the repository has been cloned and the terminal is at
its root, containing `requirements.txt`, `data/` and `notebooks/`.

## 3. Create a virtual environment

Use a `python3.11` interpreter reporting **3.11.14** before creating the environment:

```sh
python3.11 --version
python3.11 -m venv .venv
```

Activation is optional:

```sh
source .venv/bin/activate
```

The commands below explicitly use `.venv/bin/python`, so activation is unnecessary
and interpreter selection remains unambiguous.

## 4. Install Python dependencies

```sh
.venv/bin/python -m pip install -r requirements.txt
```

These pins represent the validated Python reference environment. Keep them unchanged
when replaying it. If installation fails, record the error before changing versions.

## 5. macOS ARM64 / XGBoost prerequisite

On the validated macOS ARM64 setup, XGBoost required Homebrew's OpenMP runtime,
`libomp`. If that runtime is missing on this platform and Homebrew is available:

```sh
brew install libomp
```

Validation used **libomp 23.1.0**; this command does not pin the Homebrew formula to
that version. Record the installed version when reproducing the environment.
No additional compiler flags were needed. This prerequisite is specific to the
validated setup, not a requirement to use Homebrew on every platform. As a native
system dependency, `libomp` is intentionally absent from `requirements.txt`.

Check that XGBoost can load:

```sh
.venv/bin/python -c "import xgboost; print(xgboost.__version__)"
```

Expected: `3.2.0`. If import fails, retain the error and resolve the environment
before running the notebook.

## 6. Verify the environment

```sh
.venv/bin/python --version
.venv/bin/python -m pip check
.venv/bin/python -c "import platform, numpy, pandas, sklearn; print(platform.machine()); print(numpy.__version__, pandas.__version__, sklearn.__version__)"
```

Expected: Python `3.11.14`, no broken requirements, architecture `arm64`, and
NumPy `2.4.6`, pandas `3.0.5`, scikit-learn `1.9.0`.

## 7. Dataset location and identity

The notebook expects the tracked dataset at `data/dados.csv`: **1,470 rows and
35 columns**. Its existing loading logic supports a working directory of either
the repository root or `notebooks/`.

Run this identity check from the repository root; it prints no employee-level rows:

```sh
.venv/bin/python - <<'PY'
from pathlib import Path
import hashlib
import pandas as pd

path = Path("data/dados.csv")
expected = "2bb6903956c66f339b45224cb20251af7ad5874798f70b6c7089ffd194395622"
digest = hashlib.sha256(path.read_bytes()).hexdigest()
shape = pd.read_csv(path).shape
print("SHA-256:", digest)
print("Dimensions:", shape)
assert digest == expected, "Dataset checksum mismatch"
assert shape == (1470, 35), "Dataset dimensions mismatch"
PY
```

## 8. Run the portfolio notebook

Open `notebooks/employee_attrition_v2.ipynb` in VS Code or another
Jupyter-compatible frontend:

1. Select the Python kernel belonging to this project's `.venv`.
2. Ensure the working directory is the repository root or `notebooks/`.
3. Restart the kernel, then run all cells from top to bottom without skipping cells.

`ipykernel` supplies the Python kernel; these instructions do not require JupyterLab.
The archived `notebooks/employee_attrition_original.ipynb` is the historical project
version, not the notebook for the current portfolio workflow.

The validation run used an isolated copy with the same `data/` and `notebooks/`
structure. To preserve the committed outputs during an audit, use such a copy and
save executed outputs there rather than overwriting the tracked notebook.

## 9. Expected reproducibility evidence

During validation, all **47 code cells** executed in order from a fresh kernel,
without hidden prior state. The run reproduced historical results at their reported
precision; the final configuration and confusion matrix were unchanged.

Frozen configuration: **Logistic Regression**, `C=10`, `class_weight=None`,
`solver="lbfgs"`, threshold **0.34**.

| Holdout metric | Reference value |
|---|---:|
| Accuracy | 0.8478 |
| Precision Yes | 0.5273 |
| Recall Yes | 0.4915 |
| F1 Yes | 0.5088 |
| ROC-AUC | 0.8098 |
| Average Precision | 0.5783 |

Confusion matrix counts: **TN=283, FP=26, FN=30, TP=29**.
These are reference values for checking a replay, not optimization targets.

## 10. Scope and limitations

Reference validation applies to the recorded macOS ARM64 environment. Universal
compatibility or exact bit-for-bit equality across operating systems and native
numerical libraries is not claimed. Historical metrics were stored rounded, so the
R3 audit verified agreement at their reported precision.

Record discrepancies before investigating them. Holdout results must never be used
to retune the model, change its threshold or revise the frozen selection.
