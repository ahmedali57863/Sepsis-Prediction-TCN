# Project Log — Sepsis-Prediction-TCN

**Purpose of this file:** every time we finish a real chunk of work, we add an
entry here explaining *what* we did and *why*, in plain language — not just
code comments. If a panel member points at anything in this repo and asks
"why is this here," the answer should be in this file, in your own words.

---

## Phase 0 — Project Setup & Compliance (COMPLETE)

### 0.1 Getting the data
MIMIC-IV is a real, de-identified database of ICU patient records from Beth
Israel Deaconess Medical Center (Boston), used worldwide in critical-care
machine learning research. It isn't public-download — you need to be
**credentialed**: complete an ethics training course (CITI), get a reference
to vouch for you, and sign PhysioNet's Data Use Agreement (DUA). This took
about two weeks.

**Important rule we found in the DUA:** each person who touches the data must
be *individually* credentialed. You can't hand your copy to a teammate —
that would break the agreement you personally signed. That's why we decided
Umama will not touch the raw data at all. She works on the model code,
training loop, explainability, dashboard, and paper writing — all of which
use *outputs* you produce (numbers, trained model files, charts), never the
raw patient records themselves.

### 0.2 Why we didn't just start coding against the downloaded files
The raw files are tens of gigabytes once uncompressed, and your laptop has
16GB RAM and a 4GB-VRAM GPU. Trying to load everything into memory at once
would crash. So before writing any model code, we needed a plan for how data
would flow: raw files → filtered down to only sepsis-relevant patients →
small time-series windows → only *then* into the model. That filtering step
is Phase 1 (below).

### 0.3 Environment: VS Code + a virtual environment (`sepsis_env`)
A **virtual environment** is an isolated folder of Python packages just for
this project, separate from anything else installed on your laptop. Without
it, installing one project's packages can silently break another project's.
We chose VS Code (with the Jupyter extension) over PyCharm because it lets
us both explore data interactively (like a notebook) and write proper
structured `.py` files in the same window — most ML/data-science work uses
this combo rather than a pure IDE.

### 0.4 `requirements.txt` — what's in it and why
This file lists every external package the project depends on, so anyone
(including future-you, or Umama, or an examiner trying to reproduce your
results) can install the exact same setup with one command
(`pip install -r requirements.txt`). What each group is for:

- **pandas, numpy, scikit-learn** — the basic tools for loading, cleaning,
  and manipulating tabular data.
- **torch, torchmetrics** — PyTorch, the deep learning library we'll build
  the TCN (Temporal Convolutional Network) in, plus a helper library for
  computing metrics like AUROC.
- **shap** — the explainability library. This is what turns the model's raw
  prediction into "here's *why* it thinks this patient is at risk," which is
  the whole point of the "Explainable" part of the project.
- **fastapi, uvicorn, pydantic** — for building the API later, the bridge
  between the trained model and the clinical dashboard.
- **matplotlib, seaborn** — for generating the actual charts that go in the
  research paper (replacing the placeholder AUROC graph in the template).
- **jupyter** — lets us open `.ipynb` notebooks in VS Code for quick,
  throwaway data exploration.

### 0.5 Why the folder structure looks the way it does
- `data/` stays **empty on GitHub on purpose** — it holds the actual patient
  data locally on your machine only, and is deliberately excluded from
  version control (see 0.6).
- `notebooks/` is for *looking* at data — quick, messy, exploratory. `src/`
  is for the *real* pipeline — clean scripts that actually produce the
  numbers and figures that go in the paper. Keeping these separate means
  your final results come from reviewable, reproducible code, not a
  notebook cell that got edited fifty times and forgotten.
- `configs/` will hold one file listing all your hyperparameters (window
  length, batch size, learning rate, etc.) in one place — so your paper's
  "Experimental Setup" section can just describe that file instead of you
  hunting through code to remember what you actually ran.

### 0.6 `.gitignore` — the single most important file in this repo
Git tracks every file you tell it to. `.gitignore` tells git "never track
these, even if I accidentally try." Without it, a simple `git add .` could
try to upload real patient data straight to GitHub. That's not just messy —
it would violate the Data Use Agreement you personally signed, which could
get your PhysioNet access revoked and reflect badly on your supervisor and
university, since the agreement is tied to Bahria University's name too.

What we excluded, and why:
- `data/`, `*.csv`, `*.parquet` — the actual patient data, raw or processed.
  Never committed, ever, at any stage.
- `checkpoints/`, `*.pt`, `*.pth` — saved model weights. These can get large
  (hundreds of MB) and GitHub isn't built to store big binary files well.
- `sepsis_env/` — your virtual environment folder. This is hundreds of MB of
  *installed copies* of the packages already listed in `requirements.txt` —
  redundant to upload since anyone can regenerate it from that file.
- `__pycache__/`, `.ipynb_checkpoints/` — Python's own temporary cache
  files. Not useful to anyone, just clutter.

### 0.7 `docs/DATA_GOVERNANCE.md`
A short written policy stating exactly how we're staying compliant (each
person individually credentialed, no data in git, approved cloud path only).
Two reasons this exists: (1) so anyone looking at the repo — including you
in six months — can see the rules at a glance instead of re-deriving them,
and (2) this text becomes the compliance/ethics note in the paper's Dataset
section almost word-for-word.

### 0.8 Git & GitHub — what we actually did, step by step
Plain-English version of the commands you ran:

- **`git add <files>`** — "stage" these files, meaning: mark them as ready
  to be included in the next save-point. Like putting items in a cart
  before checkout — nothing is final yet.
- **`git commit -m "..."`** — actually create the save-point (a "commit"),
  with a message describing what changed. This save-point lives on your
  laptop only at this stage.
- **`git push`** — upload that save-point to GitHub, so it's backed up
  online and visible to teammates/supervisors.
- **The "no upstream branch" error** — the first time you ever push a new
  branch, git doesn't automatically know which place on GitHub it should
  link to. `git push --set-upstream origin master` told it, once, "link my
  local `master` branch to the `master` branch on GitHub (`origin`)." Every
  push after that just works with plain `git push`.
- **`git status`** — shows what's changed since the last commit, and what's
  staged vs. not. We used this *before* every commit specifically to check
  that `data/` and `sepsis_env/` never appeared in the list — our proof the
  `.gitignore` was actually working.

**What's on GitHub right now:** `.gitignore`, `README.md`,
`PROJECT_STRUCTURE.md`, `requirements.txt`, `test_setup.py`, and
`docs/DATA_GOVERNANCE.md`. Zero patient data. That's the whole point.

### How Phase 0 connects to the paper
Nothing here becomes its own paragraph, except: the Data Governance content
becomes 2–3 sentences in **Methodology → Dataset and Preprocessing** (a
compliance note most reviewers expect to see when a paper uses controlled
health data), and the tools list becomes the basis of the already-drafted
**Experimental Setup** paragraph in `Experiments and Results`.

---

## Phase 1 — Sepsis-3 Cohort Definition (STARTING NOW)

**Goal:** out of hundreds of thousands of ICU patients, figure out exactly
*which* ones had sepsis, and *when* it started. This becomes the label our
model learns to predict.

**Why we're not writing this logic ourselves:** "Sepsis" isn't a single
number in the database — it's officially defined (Sepsis-3 clinical
criteria) as a suspected infection *plus* a jump in a patient's SOFA score
(a measure of organ failure, itself built from ~6 different lab/vital
values). Correctly computing this from raw tables is genuinely complex and
easy to get subtly wrong. Instead, we're using SQL logic from the
`mimic-code` GitHub repository — maintained by the same MIT group that
publishes MIMIC-IV, and used by most published papers in this field
(including our base paper). Using it isn't "cheating" — it's the accepted
standard, and it means a reviewer trusts our cohort the same way they'd
trust anyone else's.

**Why BigQuery instead of a local database:** MIT has already run this exact
cohort-building query and stored the result, ready to fetch, in a table
called `mimic_derived.sepsis3` on Google BigQuery. Rather than installing
and managing a full database engine on your laptop just to re-run a query
someone else already validated, we fetch the precomputed answer directly.

**What's left before we can run anything:**
1. Verify a Gmail address on your PhysioNet profile.
2. Link that Google account to PhysioNet via the "Cloud" page.
3. Request cloud access to MIMIC-IV for that linked account.
4. Create a (free) Google Cloud project and enable the BigQuery API.

*(This section will be rewritten once Phase 1 is actually done, with the
real query we ran and what the cohort numbers turned out to be.)*

---

## Glossary (plain English, for panel Q&A)

- **DUA (Data Use Agreement):** the legal contract you sign before touching
  MIMIC-IV, stating how you're allowed to use and protect the data.
- **Credentialed access:** verified permission to use PhysioNet's restricted
  datasets, granted per-person after training + a signed DUA.
- **Virtual environment (venv):** an isolated set of installed Python
  packages just for one project.
- **Git:** a tool that tracks changes to your code over time, so you can
  save versions and undo mistakes.
- **GitHub:** a website that hosts a backed-up, shareable copy of your git
  project.
- **Commit:** one saved snapshot of your code at a point in time.
- **Repository ("repo"):** the project folder that git is tracking.
- **`.gitignore`:** a list of files/folders git should never track.
- **SQL:** the query language used to ask a database questions like "give
  me every patient who meets condition X."
- **Cohort:** the specific group of patients selected for a study (here:
  ICU patients who did or didn't develop sepsis).
- **Sepsis-3 criteria:** the current official clinical definition of sepsis
  (suspected infection + a significant rise in SOFA score).
- **SOFA score:** Sequential Organ Failure Assessment — a numeric score
  built from several vital/lab measurements, used to track how badly a
  patient's organs are functioning.
- **BigQuery:** Google's cloud tool for running SQL queries against very
  large datasets without hosting the database yourself.
- **TCN (Temporal Convolutional Network):** the type of neural network
  we're using — designed to find patterns across time in sequences of data
  (here, hour-by-hour vital signs).
- **SHAP:** a method for explaining *why* a model made a specific
  prediction, by scoring how much each input feature contributed.