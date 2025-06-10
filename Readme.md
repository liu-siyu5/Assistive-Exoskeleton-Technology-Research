# Biomechanical Analysis of Ankle Torque in Human Gait

A Python toolbox for analysing human gait biomechanics by computing **ankle joint torque** from marker‐based kinematics and dual‐belt treadmill force–plate data. The code automatically discovers data files, aligns coordinate systems, calculates joint moments (τ = r × F), and generates ready‑to‑publish plots & CSV summaries.

![Pipeline schematic](docs/figs/pipeline_overview.png)

---

## Table of Contents

1. [Quick Start](#quick-start)
2. [Project Overview](#project-overview)
3. [Installation](#installation)
4. [Folder & File Structure](#folder--file-structure)
5. [Usage](#usage)

   * [Single‑file test](#single-file-test)
   * [Batch processing](#batch-processing)
   * [Acceleration ✕ Torque comparison](#acceleration--torque-comparison)
6. [Outputs](#outputs)
7. [Underlying Method](#underlying-method)
8. [Troubleshooting](#troubleshooting)
9. [Contributing](#contributing)

---

## Quick Start

```bash
# 1  Clone the repo
$ git clone https://github.com/<your‑org>/ankle‑torque‑analysis.git
$ cd ankle‑torque‑analysis

# 2  Create a virtual env (optional but recommended)
$ python -m venv .venv
$ source .venv/bin/activate   # Windows: .venv\Scripts\activate

# 3  Install dependencies
$ pip install -r requirements.txt

# 4  Run the example
$ python analyse.py --root /path/to/LAB/dephy
```

After running, browse the **`output_torque/`** folder for PNG plots and CSV stats.

---

## Project Overview

This repository couples a set of **Jupyter‑ready Python scripts** with a concise slide deck (see `docs/LAB_code.pptx`) to provide a turn‑key workflow:

```
Marker trajectories (.tsv)
    └──► Ankle position → rotated into global frame

Force‑plate files (.tsv)
    └──► Plate corners → rotation & origin
          └──► Transform & resample GRF & COP → global frame

Final combination:
    Ankle (global) + COP & Force (global)
        └──► τ = r × F
              └──► Plots ▪ Statistics ▪ CSV export
```

Key features ↴

* **Automatic file discovery** — finds every `Sxx_Txx.tsv` trajectory file & matches correct force file to each foot (handles special left/right swap in downhill trials T05 & T07).
* **Force‑plate helper utilities** — extracts four corner markers to build rotation matrices & origin offsets for each belt.
* **Coordinate unification** — marker space ➟ plate space ➟ lab global frame; optional slope‑angle correction (±5°, ±10°).
* **High‑quality plots** — full & zoomed force/torque curves, 3‑D ankle trajectories, side‑by‑side acceleration vs. torque panels.
* **CSV Augmentation** — inserts torque columns into existing sync‑aligned datasets.

See the slide deck for deeper visual explanation of each step.

---

## Installation

* **Python 3.8 +** (tested on 3.11)
* **Required packages** (also listed in `requirements.txt`):

  * `pandas`
  * `numpy`
  * `matplotlib`
  * `scipy`
  * `mpl_toolkits` (part of Matplotlib)

```bash
pip install pandas numpy matplotlib scipy
```

---

## Folder & File Structure

```
ankle‑torque‑analysis/
│
├── LAB_auto.ipynb            # Main entry point (script version of notebook)
├── notebook.ipynb            # Interactive walkthrough
├── output_torque/            # Auto‑created; all PNGs & CSVs saved here
├── docs/
│   ├── LAB_code.pptx         # Project presentation (methodology & diagrams)
│   └── figs/                 # PNG exports referenced in README
└── requirements.txt
```

> **Tip:** keep raw data outside the repo (e.g. `/Users/<you>/Downloads/LAB/dephy`). Pass the path via `--root` or edit the `root_dir` variable.

---

## Usage

### Single‑file test

Open the notebook & run **Cell 8**:

```python
# index of tsv pair to test
test_file_index = 1  # change as needed
```

The cell prints summary stats and interactive plots.

### Batch processing

Run **Cell 7** (or `python analyse.py`) — every matched pair is processed sequentially:

```bash
python analyse.py --root /Users/sophialiu/Downloads/LAB/dephy \
                  --out  output_torque
```

Console output shows progress; final summary lists mean ± range of Y‑axis torque for each trial.

### Acceleration ✕ Torque comparison

After torque CSVs are written, generate synced overlays:

```python
from analyse import plot_all_updated_csv_files
plot_all_updated_csv_files()
```

Saves two plots per file:

* `*_acceleration_torque_comparison.png` (separate panes)
* `*_acceleration_torque_overlay.png` (twin‑axis overlay)

---

## Outputs

* **PNG plots** (ankle position, force curves, torque curves, comparison figures)
* **Torque CSVs** `*_torque.csv`
* **Augmented CSVs** with **Torque\_X/Y/Z** columns inserted at first `sync == 0` row

Example torque stats (T04, LEFT):

```
Y‑axis Ankle Torque Statistics:
  Mean: 120.3 N·m
  Max : 298.6 N·m
  Min :  -3.2 N·m
```

---

## Underlying Method

1. **Corner extraction** — lines 10‑21 of each force‑plate TSV give the XYZ coordinates of four plate corners. Averaging yields plate centre; vectors build a *right‑handed* rotation matrix.
2. **Marker fusion** — lateral & medial ankle markers are averaged to estimate ankle centre → converted mm→m.
3. **Transform cascade**

   1. Translate to plate centre
   2. Rotate into plate frame
   3. Add global treadmill belt offset **±279.4 mm × +889 mm**
   4. Rotate by slope angle (trial T04→+5°, T05→−5°, T06→+10°, T07→−10°)
   5. Rotate ankle position into global frame
4. **Synchronise**

   * Resample Bertec GRF/COP 1200 Hz → 300 Hz (match Vicon trajectory rate)
   * Trim to equal length
5. **Joint moment** — for each valid (non‑zero) COP frame:
   $\tau = (\text{COP}_{global} - \text{Ankle}) \times \mathbf{F}$

Diagrams & derivations are in **slides 18‑24** of `docs/LAB_code.pptx`.

---

## Troubleshooting

| Symptom                     | Cause                                | Fix                                                     |
| --------------------------- | ------------------------------------ | ------------------------------------------------------- |
| `No rows found with sync=0` | CSV missing `sync` column            | Check acquisition script or add a dummy column of 1s ✔︎ |
| `File … not found`          | Force‑plate TSV mis‑named            | Confirm naming pattern `_f_6.tsv/_f_7.tsv`              |
| Torque columns all zeros    | COP rows are `[0,0,0]` (swing phase) | This is expected — only stance frames contribute        |

---

## Contributing

Pull requests are welcome! Please open an issue first to discuss major changes. Make sure to run `black` & `flake8` before committing.

---

## Acknowledgements

* **Bertec** for dual‑belt treadmill hardware.
* Initial project guidelines & figures adapted from *"Biomechanical Analysis of Ankle Torque in Human Movement"* (slide deck inside `docs/`).

---

> Made with ❤️ & 🦾 by Sophia Liu and contributors.
