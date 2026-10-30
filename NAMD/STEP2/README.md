# STEP2: TDDFT Calculations

This step performs time-dependent density functional theory (TDDFT) calculations along the adiabatic MD trajectories from STEP1. The goal is to generate the Kohn–Sham orbital energies and excitation energies that will be used in STEP3 to build vibronic Hamiltonians and non-adiabatic couplings.

---

## Directory Structure

```
STEP2/
├── README.md
├── adamantane/                                  # reference system (exploded)
│   ├── AMS.inp                                  # TDDFT settings
│   ├── UV_vis_spectra_part_1_and_radiative_time.ipynb
│   ├── UV_vis_spectra_part_2.ipynb
│   ├── all_logfiles/                            # extracted per-step logs
│   │   ├── step_5901.log
│   │   ├── step_5902.log
│   │   ├── step_5903.log
│   │   ├── step_5904.log
│   │   └── step_5905.log
│   ├── all_pdosfiles/                           # extracted PDOS files
│   │   ├── Diag_libra-5901-adamantane-k1-1.pdos
│   │   ├── Diag_libra-5901-adamantane-k2-1.pdos
│   │   ├── Diag_libra-5902-adamantane-k1-1.pdos
│   │   ├── Diag_libra-5902-adamantane-k2-1.pdos
│   │   ├── Diag_libra-5903-adamantane-k1-1.pdos
│   │   ├── Diag_libra-5903-adamantane-k2-1.pdos
│   │   ├── Diag_libra-5904-adamantane-k1-1.pdos
│   │   ├── Diag_libra-5904-adamantane-k2-1.pdos
│   │   ├── Diag_libra-5905-adamantane-k1-1.pdos
│   │   └── Diag_libra-5905-adamantane-k2-1.pdos
│   ├── all_logfiles.tar.bz2                     # packed backup of all_logfiles/
│   ├── all_pdosfiles.tar.bz2                    # packed backup of all_pdosfiles/
│   ├── all_pdosfiles.tr.bz2                     # additional (transposed) PDOS archive
│   ├── distribute_jobs.py
│   ├── run.py
│   └── submit_0.slm
├── 1_methyladamantane/
│   ├── README.md                                # local copy of this file
│   ├── AMS.inp
│   ├── all_logfiles.tar.bz2
│   ├── all_pdosfiles.tar.bz2.partaa             # split PDOS archive (part 1/2)
│   ├── all_pdosfiles.tar.bz2.partab             # split PDOS archive (part 2/2)
│   ├── distribute_jobs.py
│   ├── run.py
│   └── submit_1.slm
├── 1_3_dimethyladamantane/
│   ├── AMS.inp
│   ├── all_logfiles.tar.bz2
│   ├── all_pdosfiles.tar.bz2
│   ├── distribute_jobs.py
│   ├── run.py
│   └── submit_2.slm
└── 1_3_5_trimethyladamantane/
    ├── AMS.inp
    ├── all_logfiles.tar.bz2
    ├── all_pdosfiles.tar.bz2
    ├── distribute_jobs.py
    ├── run.py
    └── submit_3.slm
```

> **Note.** Only `adamantane/` is delivered in "exploded" form (raw
> `all_logfiles/` and `all_pdosfiles/` directories plus the two UV-Vis
> notebooks) — it is the reference system. For the three methylated structures
> the same data are provided as `all_logfiles.tar.bz2` /
> `all_pdosfiles.tar.bz2` archives (in `1_methyladamantane/` the PDOS archive
> is split into `.partaa` / `.partab` because of its size). Unpack each archive
> before re-running the notebooks.

---

## Contents

| File / Folder | Description |
|---------------|-------------|
| `AMS.inp` | TDDFT settings for that system: B3LYP hybrid functional, 6-311++G** basis, 30 excited states, MD-trajectory sampling. |
| `distribute_jobs.py` | Splits the STEP1 MD trajectory into independent chunks and writes the per-chunk input folders consumed by `run.py`. |
| `run.py` | Main driver for a single chunk: runs the TDDFT calculation with AMS/CP2K and writes logs, PDOS, and the Kohn–Sham data consumed by STEP3. |
| `submit_<n>.slm` | SLURM batch script that launches `run.py` on the cluster (`n = 0, 1, 2, 3` matches the project-wide system index). |
| `all_logfiles/`, `all_logfiles.tar.bz2` | Per-frame AMS/CP2K logs (exploded only in `adamantane/`; packed elsewhere). |
| `all_pdosfiles/`, `all_pdosfiles.tar.bz2` | `Diag_libra-*-*.pdos` files — projected density of states, written by Libra's `pdos` module. |
| `all_pdosfiles.tr.bz2` | Additional (transposed) PDOS archive — only in `adamantane/`. |
| `UV_vis_spectra_part_1_and_radiative_time.ipynb` | Thermally-averaged UV-Vis spectra + radiative lifetimes from Einstein's formula (only in `adamantane/`). |
| `UV_vis_spectra_part_2.ipynb` | Additional spectral analysis and comparison with experiment (only in `adamantane/`). |
| `README.md` | This file (top level) and a local copy inside `1_methyladamantane/`. |

System labels (match `<n>` in `submit_<n>.slm` and the downstream `PLOT/` layout):

| Directory | Chemical name | `<n>` |
|-----------|---------------|-------|
| `adamantane` | Adamantane | 0 |
| `1_methyladamantane` | 1-Methyl-adamantane | 1 |
| `1_3_dimethyladamantane` | 1,3-Dimethyl-adamantane | 2 |
| `1_3_5_trimethyladamantane` | 1,3,5-Trimethyl-adamantane | 3 |

---

## Workflow

### 1. TDDFT Settings (`AMS.inp`)

Defines the AMS/CP2K calculation for a single MD snapshot:

- B3LYP hybrid functional,
- 6-311++G** basis,
- 30 excited states,
- the MD-trajectory sampling window.

### 2. Job Distribution (`distribute_jobs.py`)

Splits the STEP1 trajectory into independent chunks (one per SLURM job) so that the TDDFT calculations run in parallel on the cluster. Each chunk corresponds to a contiguous block of MD steps and produces its own `step_<n>.log` and `.pdos` files.

### 3. Main Driver (`run.py`)

Runs the TDDFT calculation for one chunk and produces:

- `step_<n>.log` — AMS/CP2K log for that frame,
- `Diag_libra-<n>-<system>-k?-1.pdos` — PDOS files (one per spin channel `k1`, `k2`),
- Kohn–Sham orbital data consumed by STEP3.

### 4. SLURM Submission (`submit_<n>.slm`)

Batch scripts for the cluster. Launch with:

```bash
cd STEP2/<structure>
python distribute_jobs.py            # generate per-chunk input folders
sbatch submit_<n>.slm                # submit the whole sweep
```

### 5. Post-Processing Notebooks (only in `adamantane/`)

- `UV_vis_spectra_part_1_and_radiative_time.ipynb` — Boltzmann-averaged UV-Vis spectrum over the sampled geometries and the radiative lifetime `τ_r` from Einstein's A coefficient. This produces the τ_r values used downstream in `PLOT/QY_table.csv`.
- `UV_vis_spectra_part_2.ipynb` — spectral-feature analysis and comparison with the experimental reference.

The notebooks are the **reference implementation**; copy them into any other structure folder and adjust the input paths to analyse that structure.

---

## Key Outputs

| Output | Description | Used in |
|--------|-------------|---------|
| `all_logfiles/step_*.log` | Per-frame AMS/CP2K logs | QC / debugging |
| `all_pdosfiles/Diag_libra-*.pdos` | Projected density of states per frame | UV-Vis analysis |
| Kohn–Sham data from `run.py` | Orbital energies at each snapshot | STEP3 |
| `UV_vis_spectra_part_1_and_radiative_time.ipynb` | τ_r values | `PLOT/QY_table.csv` |

> **Note on Kohn–Sham `E_ks_*.npz` files.** `run.py` writes Kohn–Sham data in the format consumed by STEP3, but the resulting `.npz` files are **not committed** to the repository (they are large and are regenerated deterministically by re-running `run.py`). If you need them, unpack the corresponding `all_logfiles.tar.bz2` / `all_pdosfiles.tar.bz2` and re-run `run.py` — the per-snapshot energies are written next to the logs.

---

## How to Run

For each structure:

```bash
cd STEP2/<structure>
python distribute_jobs.py            # generate input chunks
sbatch submit_<n>.slm                # launch jobs on the cluster

# after the jobs finish:
tar -xjf all_logfiles.tar.bz2
tar -xjf all_pdosfiles.tar.bz2
```

For `1_methyladamantane/` the PDOS archive is split — concatenate first:

```bash
cd STEP2/1_methyladamantane
cat all_pdosfiles.tar.bz2.partaa all_pdosfiles.tar.bz2.partab > all_pdosfiles.tar.bz2
tar -xjf all_pdosfiles.tar.bz2
```

Then open the UV-Vis notebooks (only shipped under `adamantane/`, but they can be pointed at any structure by editing the `BASE_DIR` path inside the notebook).

---

## Purpose

The TDDFT data produced here serve three downstream needs:

- **Vibronic Hamiltonian (STEP3)** — the per-snapshot Kohn–Sham orbital energies and excitation energies are the raw material for the time-dependent vibronic Hamiltonian and non-adiabatic couplings.
- **Radiative lifetime (`PLOT/QY_table.csv`)** — the thermally-averaged UV-Vis spectrum and Einstein A coefficient give the `τ_r` used in the PLQY formula.
- **Active-space validation** — the PDOS files let you confirm that the orbitals 18–46 window (HOMO = 38, LUMO = 39) still contains the physically relevant states at every snapshot.

---

## Notes

- **Only `adamantane/` is exploded.** The other three structures ship as compressed archives to keep the repository small. Unpack before use.
- **Split archive.** `1_methyladamantane/all_pdosfiles.tar.bz2` is split into `.partaa` / `.partab`. Concatenate before decompressing (see `How to Run`).
- **System index.** The `<n>` in `submit_<n>.slm` (`0, 1, 2, 3`) is the project-wide system index and matches the folder order in `PLOT/` — do not renumber.
- **Only `adamantane/` ships the UV-Vis notebooks.** Copy them into another structure folder and adjust the input paths to analyse that structure.
- **`1_methyladamantane/README.md`** is a local copy of this file kept inside that structure's folder for convenience; the canonical version is this top-level one.

---

## Contact

- Qingxin Zhang (qingxin@buffalo.edu)
- Hamid Zabihi Hashjin (h.zabihi@eng.uk.ac.ir)
- Alexey V. Akimov (alexeyak@buffalo.edu)