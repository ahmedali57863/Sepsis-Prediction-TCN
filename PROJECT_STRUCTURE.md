# Project Structure — Sepsis-Prediction-TCN

```
Sepsis-Prediction-TCN/
├── data/                       # GITIGNORED — never committed
│   ├── raw/                    # untouched MIMIC-IV extract
│   ├── interim/                # after Sepsis-3 cohort filtering
│   └── processed/               # final windowed tensors ready for training
│
├── notebooks/                  # EDA only — throwaway exploration
│   └── 01_eda.ipynb
│
├── src/
│   ├── data/
│   │   ├── build_cohort.py     # runs mimic-code sepsis3 SQL, exports cohort
│   │   ├── windowing.py        # 24h sliding window construction
│   │   └── preprocessing.py    # imputation, normalization
│   ├── models/
│   │   └── tcn.py              # TCN architecture (residual blocks, dilations)
│   ├── training/
│   │   ├── train.py
│   │   └── class_weighting.py  # cost-sensitive loss setup
│   ├── explainability/
│   │   └── shap_utils.py
│   └── api/
│       └── main.py             # FastAPI serving layer (later phase)
│
├── configs/
│   └── default.yaml            # window length, batch size, lr, etc.
│
├── checkpoints/                # GITIGNORED — model weights
├── results/                    # GITIGNORED — metrics/figures generated
│                                # (export final approved figures manually
│                                #  into paper/figures/ when ready to cite)
├── paper/                      # your .tex source lives here or as submodule
│   └── figures/
├── docs/
│   └── DATA_GOVERNANCE.md
├── requirements.txt
├── .gitignore
└── README.md
```

## Why this shape

- **`data/` is never in git.** Everything under it is gitignored at every stage —
  raw, interim, and processed. If a reviewer or examiner ever clones this repo,
  it should contain zero patient rows, by construction, not by discipline.
- **`notebooks/` vs `src/`**: notebooks are for looking at the data, not for
  the pipeline that produces your paper's results. Anything that generates a
  number or figure that ends up in the thesis should be a script in `src/`,
  not a notebook cell — makes results reproducible and reviewable in PRs.
- **`configs/default.yaml`** holds every hyperparameter in one place, so your
  "Experimental Setup" paragraph in the paper can just describe this file
  instead of you hunting through code for what you actually ran.
- **`results/` is gitignored too** — figures get regenerated from code, not
  hand-edited. Only once a figure is final do you drop it into `paper/figures/`
  to be committed alongside the `.tex` source.