# STEP4: Nonadiabatic Molecular Dynamics (NAMD)

This step performs Nonadiabatic Molecular Dynamics (NAMD) simulations using the vibronic Hamiltonians and non-adiabatic couplings generated in STEP3, and provides the analysis notebook that reads the resulting `mem_data.hdf` files. The simulations are carried out with four surface-hopping methods: **FSSH**, **FSSH2**, **IDA**, and **MSDM**.

The full post-NAMD analysis pipeline — stretched-exponential (Kohlrausch) fitting, photoluminescence quantum yield (PLQY), structure/method comparison figures, and Shannon-entropy analysis — is applied to the STEP4 output from a separate analysis layer that reads the `mem_data.hdf` files. The scripts for that layer are described under [Post-NAMD Analysis](#post-namd-analysis-external).

---

## Directory Structure

```
STEP4/
├── NAMD_MB.py                         # Main NAMD driver
├── NAMD_MB.py:Zone.Identifier         # Windows metadata (exclude via .gitignore)
├── PLOT_NAMD.ipynb                    # Analysis notebook (reads mem_data.hdf)
├── PLOT_NAMD.ipynb:Zone.Identifier    # Windows metadata (exclude via .gitignore)
├── README.md
├── README.md:Zone.Identifier          # Windows metadata (exclude via .gitignore)
└── recipes/
    ├── fssh_nbra.py                   # FSSH
    ├── fssh2_nbra.py                  # FSSH2
    ├── ida_nbra.py                    # IDA
    └── msdm_nbra.py                   # MSDM
```

**Not committed** (generated locally — see [Notes](#notes)):
`mem_data.hdf`, `_dyn_params.txt`, `_model_params.txt`, and the analysis output
folder `analysis_results_batch_average/`.

---

## Simulation Workflow

### 1. Main driver — `NAMD_MB.py`

Runs the NAMD simulation for a given method, structure, and initial condition. It
reads the vibronic Hamiltonian and non-adiabatic couplings from STEP3 and
propagates each trajectory with one of the four surface-hopping recipes in
`recipes/`.

Trajectory output (per `(structure, method, icond)`) is written to the external
working directory used by the analysis scripts:

```
<BASE_DIR>/<structure>/<method>/<METHOD>0_icond_<icond>/
```

with `<BASE_DIR> = /home/hamid/A/PLOT` by convention (the analysis scripts read
from this root; override it inside each script if your output goes elsewhere).
The `<METHOD>0_` prefix is the project-wide convention used by the post-processing
scripts (`FSSH0`, `FSSH20`, `IDA0`, `MSDM0`).

> **Note.** `STEP4/` contains only the driver and recipes — the trajectory data are
> generated externally, not stored in the repository. Confirm `NAMD_MB.py --help`
> for the exact argument names of your installed driver version.

### 2. Surface-hopping recipes — `recipes/`

Four surface-hopping methods are implemented, each a drop-in recipe consumed by
`NAMD_MB.py`:

| Recipe | Method | Key feature |
|--------|--------|-------------|
| `fssh_nbra.py` | FSSH | Standard Tully FSSH |
| `fssh2_nbra.py` | FSSH2 | Alternative FSSH with robust switching |
| `ida_nbra.py` | IDA | Instantaneous Decoherence at Attempted hops |
| `msdm_nbra.py` | MSDM | Simplified Decay of Mixing |

Each recipe defines:

- the hopping probability,
- the decoherence correction (if applicable),
- the acceptance / rejection rules for proposed hops.

### 3. Output files (generated locally)

Per trajectory, `NAMD_MB.py` writes:

| File | Contents |
|------|----------|
| `mem_data.hdf` | HDF5 with the population time series — the primary input to every analysis script. |
| `_dyn_params.txt` | Dynamics parameters for that run. |
| `_model_params.txt` | Model parameters for that run. |

The two HDF5 datasets consumed by the analysis layer are:

| Dataset | Description |
|---------|-------------|
| `time/data` | Time array (atomic units; converted to fs via `units.au2fs`). |
| `sh_pop_adi/data` | Surface-hopping adiabatic populations — column `0` is the S₀ population, all columns are the full set of excited states. |
| `se_pop_adi/data` | SE (adiabatic-reference) populations — used by the Shannon-entropy analysis. |

---

## Data Analysis (in-repo) — `PLOT_NAMD.ipynb`

`PLOT_NAMD.ipynb` is the in-repo analysis notebook for the trajectories produced by
`NAMD_MB.py`. It reads the `mem_data.hdf` files from the external working
directory, extracts the S₀ population (`sh_pop_adi/data[:, 0]`), fits it with the
normalised stretched exponential

$$
P_{S_0}(t) = 1 - \exp\!\left[-\left(\frac{t}{\tau}\right)^{\beta}\right],
$$

applies an R² threshold to filter poor fits, builds an ensemble from the accepted
trajectories, and writes a result folder `analysis_results_batch_average/` in the
local working directory (not committed — see [Notes](#notes)).

The `analysis_results_batch_average/` folder contains:

| File | Contents |
|------|----------|
| `analysis_results.pkl` | Nested result dictionary keyed by `(structure, method)` — `accepted_batches`, `rejected_batches`, `ensemble`, `final_fit`, `bootstrap`, `plqy`; plus global metadata and experimental reference values. |
| `batch_curves.csv` | Per-trajectory S₀ populations, fit parameters (`tau_ps`, `beta`, `r2`), and accept/reject status. |
| `ensemble_curves.csv` | Ensemble mean and std of S₀, fitted curve, and bootstrap 95% CI band. |
| `comparison_data.csv` | Summary `(structure, method)` table: `tau_ps ± CI`, `beta ± CI`, `plqy_percent ± CI`, experimental `τ`, experimental PLQY, radiative lifetimes, `n_accepted`, `n_rejected`. |
| `summary_results.txt` | Human-readable per-panel summary. |

---

## Post-NAMD Analysis (external)

The published comparison figures, PLQY table, and entropy figures are produced by
scripts that read the **same** `mem_data.hdf` files (or the CSVs produced by
`PLOT_NAMD.ipynb`) from the external analysis root `/home/hamid/A/PLOT`. They are
independent of `STEP4/` in the sense that they can be re-run without re-simulating.

| Stage | Script | Inputs | Outputs |
|-------|--------|--------|---------|
| 1 | `Kohlrausch fits for 4 structures × 4 methods` | `mem_data.hdf` | `Kohlrausch_4x2_all_methods_results.csv`, `Kohlrausch_4x2_FSSH_FSSH2_S0_rise.png`, `Kohlrausch_4x2_IDA_MSDM_S0_rise.png` |
| 2 | `Quantum yield table generation` | `Kohlrausch_4x2_all_methods_results.csv` | `QY_table.csv`, Markdown Table 4 |
| 3 | `Three comparison figures …` | `Kohlrausch_4x2_all_methods_results.csv` | `Nonradiative_lifetime_comparison.png`, `Beta_comparison.png`, `Radiative_quantum_yield_comparison.png` |
| 4 | `Shannon entropy analysis` | `mem_data.hdf` (`se_pop_adi/data`, `sh_pop_adi/data`, `time/data`) | 8 × `*_SE_entropy_2x2.png`, `*_SH_entropy_2x2.png` |

Stage 3 uses the **median ± half-IQR** estimator for display (main text τ_eff and
PLQY figures; β is delegated to Supporting Information). Stages 1–2 use the
**mean ± 95 % t-CI** estimator for the reported numbers (Table 4). Both estimators
apply the same **log-space MAD filter** (`k = 2.5`) and the same fixed R²
thresholds (`FSSH/FSSH2 > 0.80`, `IDA/MSDM > 0.50`), so the two estimators agree
in trend and the reported tables are internally consistent.

---

## How to Run

### 1. Run a single NAMD trajectory

```bash
python NAMD_MB.py --method FSSH --structure adamantane --icond 1
```

Repeat over the four methods (`FSSH`, `FSSH2`, `IDA`, `MSDM`), the four structures
(`adamantane`, `1methyl_adamantane`, `2methyl_adamantane`, `3methyl_adamantane`),
and the 30 initial conditions per panel (`icond = 1, 101, 201, …, 2901`).
Argument names may differ between driver versions — verify with
`python NAMD_MB.py --help`.

### 2. Run the in-repo analysis notebook

```bash
jupyter notebook PLOT_NAMD.ipynb
```

Run cells in order. The first cell performs the full analysis and writes
`analysis_results_batch_average/`; the subsequent cells generate the summary
figures described above.

### 3. Run the post-NAMD analysis (external)

```bash
# from /home/hamid/A/PLOT (or wherever the mem_data.hdf tree lives)
python kohlrausch_4x2.py     # -> Kohlrausch_4x2_all_methods_results.csv + 2 figures
python qy_table.py           # -> QY_table.csv
python three_comparison_figures.py   # -> 3 comparison figures
python shannon_entropy.py    # -> 8 entropy grids
```

Script names are placeholders — use the actual filenames in the external analysis
directory.

---

## Dependencies

- Python 3.10+
- `numpy`, `scipy`, `h5py`, `matplotlib`
- `pickle`, `csv` (standard library)
- `libra_py` (Libra library — used by the recipes and by `NAMD_MB.py`)

---

## Notes

- **Trajectory output is external.** `mem_data.hdf`, `_dyn_params.txt`, and
  `_model_params.txt` are written by `NAMD_MB.py` to the external working
  directory, **not** to `STEP4/`. See [Simulation Workflow](#1-main-driver--namd_mbpy).
- **Analysis output is external.** `analysis_results_batch_average/` is written by
  the first cell of `PLOT_NAMD.ipynb` in the local working directory, and is
  **not** committed.
- **`Zone.Identifier` files.** The `*:Zone.Identifier` files are Windows
  metadata artifacts (they record the file's provenance when downloaded). They
  are not part of the computational content and should be excluded via
  `.gitignore`:
  ```
  *:Zone.Identifier
  ```
- **No entropy notebook in `STEP4/`.** The Shannon-entropy analysis is not
  shipped inside `STEP4/`; it lives in the external post-processing directory
  and reads `se_pop_adi/data` / `sh_pop_adi/data` directly from `mem_data.hdf`.
- **`NAMD/PLOT - Copy.ipynb`.** A copy of the plotting notebook exists one level
  up (`NAMD/PLOT - Copy.ipynb`); the canonical version for this step is
  `STEP4/PLOT_NAMD.ipynb`.

---

## Contact

- Qingxin Zhang (qingxin@buffalo.edu)
- Hamid Zabihi Hashjin (h.zabihi@eng.uk.ac.ir)
- Alexey V. Akimov (alexeyak@buffalo.edu)