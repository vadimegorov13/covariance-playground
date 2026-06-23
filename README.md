# 19 × 7 Covariance — Classifier Playground

Re-run the motor-imagery classifier on the **precomputed 19-band × 7-window covariance
matrices**, without needing the raw EEG. Swap in any classifier and compare the *honest*
(band/window chosen on training data) vs *leaky* (chosen on the test data) evaluation.

## Repository layout

```
.
├── requirements.txt
├── src/
│   └── liu2024_covariance_classifier_playground.ipynb
└── covariance_cache/  # 19×7 covariances: one sub-XX_cov.npz per subject
```

Each `sub-XX_cov.npz` holds, for one subject, a covariance matrix **per trial** for every
band × time-window combination, plus the labels. The classifier is run on these directly —
the raw dataset is not required.

## Usage

### 1. Create a virtual environment

From the repository root:

```bash
python3.11 -m venv .venv
```

Activate the environment using the command for your shell:

```powershell
# PowerShell (Windows)
.\.venv\Scripts\Activate.ps1
```

```bat
:: Command Prompt (Windows)
.venv\Scripts\activate.bat
```

```bash
# Bash/Zsh (Linux/macOS)
source .venv/bin/activate
```

### 2. Install dependencies

```bash
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

### 3. Register the environment as a notebook kernel

```bash
python -m ipykernel install --user --name eeg-jepa --display-name "playground"
```

After that, select **playground** as the kernel in Jupyter or VS Code, if running through VS Code you might need to reload the window

### 4. Open the notebook and set the covariance path

Open `src/liu2024_covariance_classifier_playground.ipynb` and, in the first cell, point
`COV_DIR` at the covariance folder. The path is relative to the notebook in `src/`:

```python
COV_DIR = "../covariance_cache"
```

(An absolute path works too.) **This is the only edit required.** Optionally, pick a
different classifier in the "Choose your classifier" cell — `FgMDM` (the paper's DGFMDRM)
is set up by default, with TSLDA and the full TWFB+DGFMDRM listed as ready-to-use options.

### 5. Run the notebook

Run all cells (**Kernel → Restart & Run All**). The first run defaults to the first 5
subjects for a quick check; set `SUBJECTS_LIMIT = None` in the first cell to run all 50.

> **Runtime note.** With `N_REPEATS = 10` and all 50 subjects this is a long run (the
> classifier is refit for every band/window on every split). Keep `SUBJECTS_LIMIT` small
> while iterating; run the full 50 overnight. Setting `INNER_FOLDS = 3` speeds up the
> honest protocol with almost no change in the result.

## Reading the output

The run prints, per subject and as a group mean, two balanced-accuracy numbers (chance = 50%):

- **honest** — band/window selected using only the training trials. This is the fair number.
- **leaky** — band/window selected on the test trials (max-on-test), shown for comparison.
