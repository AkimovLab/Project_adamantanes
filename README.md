# Project Adamantanes

A computational study of the electronic structure and excited-state relaxation dynamics of pristine and methylated adamantanes.

---

## Overview

This project — *Nonadiabatic Molecular Dynamics in Pristine and Methylated Adamantanes* — combines static electronic-structure calculations with non-adiabatic molecular dynamics (NAMD) simulations to investigate the photophysical properties of adamantane and its methylated derivatives.

The repository contains the complete computational workflow for:

- Quantum-chemistry calculations of the electronic structure of adamantane and its methylated derivatives.
- Non-adiabatic molecular dynamics (NAMD) simulations of excited-state relaxation.
- Analysis of radiative and non-radiative recombination, quantum yields, and their dependence on molecular structure.
- Robust post-processing of NAMD trajectories: stretched-exponential (Kohlrausch) fitting, effective-lifetime extraction, photoluminescence quantum-yield (PLQY) evaluation, and Shannon-entropy analysis.

In addition to the dynamic trajectory method, a single-optimization approach was also employed.

---

## Repository Structure

```
Project_adamantanes/
├── LICENSE
├── README.md                          # This file
├── NAMD/                              # NAMD workflow (STEP1 → STEP4)
│   ├── STEP1/                         # Structure preparation and equilibration MD
│   ├── STEP2/                         # TDDFT calculations along MD trajectories
│   ├── STEP3/                         # Vibronic Hamiltonians and NACs
│   └── STEP4/                         # NAMD simulations (FSSH, FSSH2, IDA, MSDM)
├── PLOT/                              # Post-processing and figure generation
│   ├── adamantane/
│   │   ├── FSSH/                      # mem_data.hdf trajectories (icond 1 … 2901)
│   │   ├── FSSH2/
│   │   ├── IDA/
│   │   └── MSDM/
│   ├── 1methyl_adamantane/
│   │   ├── FSSH/ … MSDM/
│   ├── 2methyl_adamantane/
│   │   ├── FSSH/ … MSDM/
│   ├── 3methyl_adamantane/
│   │   ├── FSSH/ … MSDM/
│   ├── Kohlrausch_4x2_all_methods_results.csv
│   ├── QY_table.csv
│   ├── Nonradiative_lifetime_comparison.png
│   ├── Beta_comparison.png
│   ├── Radiative_quantum_yield_comparison.png
│   ├── *_SE_entropy_2x2.png            # per-method SE-entropy grids
│   └── *_SH_entropy_2x2.png            # per-method SH-entropy grids
└── Static_Optimization_Structure/     # Optimized geometries and IP/UV-Vis data
```

---

## Workflow Overview

### Static Optimization Structure

Optimize the initial crystal structure using the **B3LYP** functional.

- **Input:** Optimized CIF files, ORCA and CP2K input files with the appropriate `energy_type` setting (4 different systems).
- **Output:** Ground-state and TDDFT output files and corresponding Molden files. The TDDFT output files can be further analyzed with Multiwfn to generate UV–Vis spectra in `.txt` format. Different spectral line shapes and degrees of peak broadening can be obtained by adjusting the full width at half maximum (FWHM).

Related scripts and examples are available in:

```
Static_Optimization_Structure/UV_Vis_spectra
```

---

### Nonadiabatic Molecular Dynamics Workflow

#### STEP 1: Adiabatic Molecular Dynamics

- **Objective:** Obtain equilibrated adiabatic trajectories at 300 K.
- **Output:** Equilibrated ground-state molecular-dynamics trajectories for subsequent electronic-structure calculations.

**Additional Note: Restarting a CP2K MD Simulation:**

To continue a previously interrupted CP2K molecular-dynamics simulation, include the following restart settings in the input file:

```
&EXT_RESTART
  RESTART_FILE_NAME adamantane_MD-1_5000.restart
  RESTART_DEFAULT T
&END EXT_RESTART
```

---

#### STEP 2: TDDFT Calculations

- **Input:** TDDFT calculations based on a 300 K temperature profile.
- **Output:** Excited-state energies, oscillator strengths, and Kohn–Sham orbital energies for visualizing electronic entropy and UV–Vis spectra.

---

#### STEP 3: Vibronic Hamiltonian Construction

- **Objective:** Generate the time-dependent vibronic Hamiltonian using the Step 2 outputs and the selected active space.
- **Active space:** Taking adamantane as an example, orbitals 18–46, with orbital 38 as the HOMO and orbital 39 as the LUMO.
- **Output:** Time-dependent electronic energies, probability distributions, and non-adiabatic couplings for the selected electronic states.

---

#### STEP 4: Nonadiabatic Molecular Dynamics

- **Objective:** Perform non-adiabatic molecular-dynamics simulations using the Step 3 Hamiltonian data with four different surface-hopping methods: **FSSH, FSSH2, IDA, and MSDM**.
- **Output:** `mem_data.hdf` files containing time-resolved adiabatic and surface-hopping populations (`se_pop_adi`, `sh_pop_adi`) and the simulation time axis. These are the direct inputs to the post-processing pipeline in `PLOT/`.

---

## Post-Processing and Analysis (`PLOT/`)

All downstream analysis is performed on the `mem_data.hdf` files stored in `PLOT/<structure>/<method>/`. The pipeline consists of four independent, re-runnable stages.

### 1. Stretched-Exponential (Kohlrausch) Fitting

**Script:** `Kohlrausch fits for 4 structures × 4 methods`

- Reads `sh_pop_adi/data` (S₀ population) and `time/data` from every trajectory.
- Fits the normalised S₀ rise with the stretched exponential

  \[
  P_{S_0}(t) = 1 - \exp\left[-\left(\frac{t}{\tau}\right)^{\beta}\right],
  \]

  using `scipy.optimize.curve_fit` (bounds `β ∈ [0.05, 10]`).
- Computes the effective lifetime \(\tau_{\rm eff} = \tau\,\Gamma(1+1/\beta)\).
- Applies a two-stage quality filter:
  - **Fixed R² thresholds:** `FSSH/FSSH2: R² > 0.80`; `IDA/MSDM: R² > 0.50`.
  - **Amplitude filter:** `ΔP ≥ 3.0×10⁻³` (FSSH/FSSH2) or `3.0×10⁻⁴` (IDA/MSDM).
- Rejects outliers with a **log-space MAD filter** (`k = 2.5`).
- **Outputs:**
  - `Kohlrausch_4x2_all_methods_results.csv` — one row per trajectory.
  - `Kohlrausch_4x2_FSSH_FSSH2_S0_rise.png` and `Kohlrausch_4x2_IDA_MSDM_S0_rise.png` — 4×2 comparison figures.

### 2. Quantum-Yield Table

**Script:** `Quantum yield table generation`

- Reads `Kohlrausch_4x2_all_methods_results.csv` (only rows with `accepted == 1`).
- Re-applies the same log-space MAD filter and computes the **mean ± 95 % CI** of \(\tau_{\rm eff}\) (Student-t, `scipy.special.stdtrit`).
- Evaluates the PLQY

  \[
  \mathrm{QY} = 100\,\frac{\tau_{\rm nr}}{\tau_{\rm nr} + \tau_r}\ [\%],
  \]

  with \(\tau_r\) taken from **Rander et al., JACS 2017, Table 3**.
- Propagates the uncertainty on \(\tau_{\rm nr}\) to the QY via the linearised first-order expression.
- **Output:** `QY_table.csv` and a Markdown-formatted Table 4 printed to the console.

### 3. Comparison Figures (τ_eff, β, PLQY)

**Script:** `Three comparison figures derived from Kohlrausch_4x2_all_methods_results.csv`

- Aggregates \(\tau_{\rm eff}\) and \(\beta\) per (structure, method) using the same log-space MAD filter, reporting **median ± half-IQR** for display.
- **Main-text figures (embedded data tables):**
  - `Nonradiative_lifetime_comparison.png` — \(\tau_{\rm eff}\) vs. structure, with experimental overlay.
  - `Radiative_quantum_yield_comparison.png` — PLQY vs. structure, with experimental overlay.
- **Supporting-Information figure:**
  - `Beta_comparison.png` — stretch exponent \(\beta\) vs. structure (no experimental β available).

### 4. Shannon-Entropy Analysis

**Script:** `Shannon entropy analysis`

- Reads `se_pop_adi/data`, `sh_pop_adi/data`, and `time/data` from every trajectory.
- Re-normalises populations to Σp = 1 and evaluates

  \[
  S(t) = -\sum_i p_i(t)\,\ln p_i(t)
  \]

  for both the SE (adiabatic) and SH (surface-hopping) populations.
- Averages across trajectories and draws a \(\pm\sigma\) band (`ddof=1`).
- **Outputs:** 8 figures — `FSSH_SE_entropy_2x2.png`, `FSSH_SH_entropy_2x2.png`, …, `MSDM_SH_entropy_2x2.png` — each a 2×2 structure grid with shared axes.

---

## Workflow Summary Table

| Step | Purpose | Key Outputs |
|------|---------|-------------|
| Static Optimization | B3LYP geometry optimization and UV-Vis | CIF, Molden, TDDFT output, spectra |
| STEP1 | Prepare and align molecular structures, run MD at 300 K | `aligned.tar.gz`, MD trajectories |
| STEP2 | TDDFT calculations along MD trajectories | Kohn–Sham energies, UV-Vis spectra |
| STEP3 | Build vibronic Hamiltonians and non-adiabatic couplings | `Hvib_ci_*.npz`, `St_ci_*.npz` |
| STEP4 | Run NAMD simulations and analyze kinetics | `mem_data.hdf`, `analysis_results.pkl` |
| **PLOT / Kohlrausch** | Fit S₀ rise, extract \(\tau_{\rm eff}\) and \(\beta\) | `Kohlrausch_4x2_all_methods_results.csv`, 4×2 figures |
| **PLOT / QY table** | Aggregate \(\tau_{\rm eff}\), compute PLQY with error propagation | `QY_table.csv`, Markdown Table 4 |
| **PLOT / Comparison** | τ_eff, β, PLQY vs. structure (main text + SI) | `Nonradiative_lifetime_comparison.png`, `Beta_comparison.png`, `Radiative_quantum_yield_comparison.png` |
| **PLOT / Entropy** | Shannon entropy of SE and SH populations | 8 `*_entropy_2x2.png` figures |

---

## Structures Studied

- Adamantane
- 1-Methyl-adamantane
- 1,3-Dimethyl-adamantane
- 1,3,5-Trimethyl-adamantane

---

## Surface Hopping Methods

The NAMD simulations in STEP4 employ four different trajectory surface-hopping (TSH) methods:

| Method | Description |
|--------|-------------|
| FSSH | Fewest Switches Surface Hopping (Tully, 1990) |
| FSSH2 | Alternative FSSH formulation with robust switching |
| IDA | Instantaneous Decoherence at Attempted hops |
| MSDM | Simplified Decay of Mixing |

---

## Dependencies

- Python 3.10+
- **Core libraries:** `numpy`, `scipy`, `h5py`, `matplotlib`
- **Libra:** `libra_py` (Libra library)
- **Electronic-structure codes (STEP2 / static calculations):** CP2K, ORCA, Multiwfn

The post-processing scripts additionally use:

- `scipy.optimize.curve_fit` (Kohlrausch fitting)
- `scipy.special.gamma`, `scipy.special.stdtrit` (effective lifetime, Student-t CI)
- `matplotlib.pyplot`, `matplotlib.ticker` (figures and table embedding)

---

## Data Availability

All trajectory data required to reproduce the analysis are stored as `mem_data.hdf` files under `PLOT/<structure>/<method>/<METHOD>0_icond_<icond>/`. The derived CSV files (`Kohlrausch_4x2_all_methods_results.csv`, `QY_table.csv`) and all final figures are included in the same directory for direct inspection.

---

## Contact

- Qingxin Zhang (qingxin@buffalo.edu)
- Hamid Zabihi Hashjin (h.zabihi@eng.uk.ac.ir)
- Alexey V. Akimov (alexeyak@buffalo.edu)