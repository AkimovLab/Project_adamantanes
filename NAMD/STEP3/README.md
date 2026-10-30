# STEP3: Vibronic Hamiltonian Construction

This step builds the vibronic Hamiltonians and non-adiabatic couplings (NACs) needed for the NAMD simulations in STEP4. It processes the output of STEP2 (Kohn–Sham orbital energies, TDDFT excitations) and produces the per-snapshot vibronic Hamiltonian, state energies, and NACs consumed by the surface-hopping dynamics.

---

## Directory Structure

```
STEP3/
├── README.md
├── step3_0.tar.gz                              # packed per-structure output (n = 0, adamantane)
├── step3_1.tar.gz                              # packed per-structure output (n = 1, 1-methyl)
├── step3_2.tar.gz                              # packed per-structure output (n = 2, 1,3-dimethyl)
├── step3_3.tar.gz                              # packed per-structure output (n = 3, 1,3,5-trimethyl)
└── adamantane/                                 # reference structure (exploded)
    ├── adamantane_step3.ipynb                  # the analysis notebook
    ├── adamantane_results.tar.gz               # results bundle (Hvib / St / NACs)
    └── step33three_B3LYP.tar.gz                # packed per-snapshot vibronic data
```

> **Note.** Only `adamantane/` is delivered "exploded" (notebook + packed result bundles) — it is the reference structure. The three methylated structures ship as the top-level `step3_<n>.tar.gz` archives. Each `step3_<n>.tar.gz` **contains the same kind of content** that is inside `adamantane/step33three_B3LYP.tar.gz` for the reference structure. Unpack before running the notebook.

After unpacking, the canonical layout is:

```
STEP3/<structure>/
├── <structure>_step3.ipynb                     # only shipped for adamantane
├── <structure>_results.tar.gz                  # only shipped for adamantane
└── step33three_B3LYP/                          # exploded from step33three_B3LYP.tar.gz
    ├── Hvib_ci_*_re.npz                        # real part, CI basis
    ├── Hvib_ci_*_im.npz                        # imaginary part (NACs), CI basis
    ├── Hvib_sd_*_re.npz                        # real part, SD basis
    ├── Hvib_sd_*_im.npz                        # imaginary part (NACs), SD basis
    ├── St_ci_*_re.npz                          # CI-basis state energies
    ├── St_sd_*_re.npz                          # SD-basis state energies
    └── T_*.txt                                 # time axis and auxiliary time-dependent data
```

System labels (match `<n>` in `step3_<n>.tar.gz` and the downstream `PLOT/` layout):

| Structure | `<n>` | In repo as |
|-----------|-------|------------|
| Adamantane | 0 | `adamantane/` (exploded) + `step3_0.tar.gz` |
| 1-Methyl-adamantane | 1 | `step3_1.tar.gz` |
| 1,3-Dimethyl-adamantane | 2 | `step3_2.tar.gz` |
| 1,3,5-Trimethyl-adamantane | 3 | `step3_3.tar.gz` |

---

## Contents

| File / Folder | Description |
|---------------|-------------|
| `adamantane/adamantane_step3.ipynb` | Reference notebook: builds the vibronic Hamiltonian and NACs, runs the eight analyses listed under *Workflow*, and produces the `step33three_B3LYP/` bundle. Only shipped for `adamantane/`. |
| `adamantane/adamantane_results.tar.gz` | Packed results bundle for the reference structure (Hvib / St / NACs) — can be unpacked instead of re-running the notebook. |
| `adamantane/step33three_B3LYP.tar.gz` | Packed per-snapshot vibronic data for `adamantane` (explodes to the `step33three_B3LYP/` layout above). |
| `step3_<n>.tar.gz` | Packed per-snapshot vibronic data for structure `<n>` — the same content as `adamantane/step33three_B3LYP.tar.gz`, but for the methylated structures. |
| `README.md` | This file. |

---

## Workflow

The notebook `adamantane_step3.ipynb` performs the following steps.

1. **Active-space selection**
   Reads the Kohn–Sham orbital energies from STEP2 and selects a reduced active space around the HOMO/LUMO. For adamantane:
   - orbitals **18–46**,
   - HOMO = orbital **38**,
   - LUMO = orbital **39**.

   The same window is used for the three methylated structures, whose orbital ordering shifts only by the expected methyl-induced stabilisation.

2. **Data generation (`step3.run_step3_sd_nacs_libint`)**
   Computes the vibronic Hamiltonian (`Hvib_ci_*`) and the NACs from the selected active space, producing both the real and imaginary parts of the Hamiltonian on the CI and SD bases.

3. **Visualisation of NACs**
   Plots the average absolute value of the NAC matrix as a heatmap (state-pair × state-pair) to expose the coupling pattern.

4. **Influence spectrum**
   Computes the influence spectrum from the S₁–S₀ energy gap and quantifies the contribution of low-frequency modes to the gap fluctuation.

5. **Time-dependent excitation energies**
   Extracts and plots the time-dependent excitation energies relative to the ground state (S₁–S₀, S₂–S₀, …) along the sampled MD trajectory.

6. **Kohn–Sham orbital-energy visualisation**
   Plots the time evolution of Kohn–Sham orbital energies for a short trajectory segment — useful for confirming that the active-space window stays intact over time.

7. **S₁–S₀ energy-gap distribution**
   Computes and plots the probability density of the S₁–S₀ gap, the quantity that drives the thermal activation of surface hops in STEP4.

8. **NAC distribution**
   Computes and plots the probability density of |NAC(S₀, S₁)| — the coupling that controls the hopping probability in FSSH-family methods.

9. **Distributions for all structures**
   Repeats steps 4 and 7–8 for the four structures and compares them in a single figure (S₁–S₀ gap PDF, NAC PDF, influence spectrum overlaid).

---

## Key Outputs

| Output | Description | Consumed by |
|--------|-------------|-------------|
| `Hvib_ci_*_re.npz` | Real part of the vibronic Hamiltonian, CI basis | STEP4 (`FSSH`, `FSSH2`, `IDA`, `MSDM`) |
| `Hvib_ci_*_im.npz` | Imaginary part = NACs, CI basis | STEP4 |
| `Hvib_sd_*_re.npz` | Real part, SD basis | STEP4 |
| `Hvib_sd_*_im.npz` | Imaginary part = NACs, SD basis | STEP4 |
| `St_ci_*_re.npz` | CI-basis state energies | STEP4 |
| `St_sd_*_re.npz` | SD-basis state energies | STEP4 |
| `T_*.txt` | Time axis and auxiliary time-dependent data | STEP4 |

These files are the direct input to the NAMD simulations in STEP4.

---

## How to Run

For the reference structure:

```bash
cd STEP3/adamantane
jupyter notebook adamantane_step3.ipynb
```

Run all cells sequentially. The notebook writes the `step33three_B3LYP/` directory next to itself; if `step33three_B3LYP/` already exists it is overwritten.

For a methylated structure, unpack the corresponding archive first:

```bash
cd STEP3
tar -xzf step3_1.tar.gz       # -> creates step3_1/ with the same step33three_B3LYP/ layout
cd step3_1
# copy the reference notebook, then point its BASE_DIR at this folder
cp ../adamantane/adamantane_step3.ipynb .
jupyter notebook adamantane_step3.ipynb
```

To analyse the already-packaged results **without** re-running the notebook:

```bash
cd STEP3
tar -xzf step3_<n>.tar.gz       # unpack the vibronic data
# or, for the reference structure:
cd adamantane
tar -xzf step33three_B3LYP.tar.gz
tar -xzf adamantane_results.tar.gz
```

The `step33three_B3LYP/` directory is then available to STEP4 directly — no recomputation needed.

---

## Purpose

The vibronic data produced here serve three downstream needs:

- **Surface-hopping dynamics (STEP4)** — `Hvib_ci_*` (real + imaginary) and `St_ci_*` are the exact inputs to the FSSH/FSSH2/IDA/MSDM recipes in `STEP4/recipes/`.
- **Active-space validation** — the Kohn–Sham orbital-energy time series (workflow item 6) confirms that orbitals 18–46 and the HOMO=38 / LUMO=39 window remain valid at every snapshot.
- **Physical diagnostics** — the S₁–S₀ gap PDF, NAC PDF, and influence spectrum (workflow items 4, 7, 8) characterise the non-adiabatic regime of each structure; the distributions for all structures (workflow item 9) make the methyl-substitution effect directly visible.

---

## Notes

- **Only `adamantane/` is exploded.** The methylated structures ship as top-level `step3_<n>.tar.gz` archives. Unpack before use.
- **System index.** The `<n>` in `step3_<n>.tar.gz` (`0, 1, 2, 3`) is the project-wide system index and matches the folder order in `STEP1`, `STEP2`, and `PLOT/` — do not renumber.
- **Notebook is single-source.** Only `adamantane/adamantane_step3.ipynb` is shipped; copy it into a methylated folder and adjust the `BASE_DIR` (or equivalent) path variable to analyse another structure.
- **Reproducibility.** Unpacking `step3_<n>.tar.gz` (or `adamantane/step33three_B3LYP.tar.gz`) gives the same `step33three_B3LYP/` layout that the notebook produces — so a downstream user can run STEP4 without ever re-running STEP3.
- **`adamantane_results.tar.gz`** is a second reference bundle for the same structure (final Hvib/St/NACs plus any auxiliary analysis output); use `step33three_B3LYP.tar.gz` for the canonical raw data.

---

## Contact

- Qingxin Zhang (qingxin@buffalo.edu)
- Hamid Zabihi Hashjin (h.zabihi@eng.uk.ac.ir)
- Alexey V. Akimov (alexeyak@buffalo.edu)