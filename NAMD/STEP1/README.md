# STEP1: Structure Preparation and Adiabatic Molecular Dynamics

This step prepares the initial molecular structures of adamantane and its methylated
derivatives, aligns them to a common reference frame, and runs equilibrated
**adiabatic** molecular-dynamics (MD) simulations at 300 K with **CP2K** (B3LYP).
The resulting ground-state trajectories are the direct input to the TDDFT
calculations in STEP2.

---

## Directory Structure

```
STEP1/
├── README.md                                  # This file
├── align.py                                   # Structure-alignment script
├── adamantane/
│   ├── adamantane.inp                         # CP2K MD input
│   ├── 0_output.tar.gz                        # MD output (raw CP2K log + restart)
│   └── aligned.tar.gz                         # Aligned starting structures
├── 1_methyladamantane/
│   ├── 1_methyladamantane.inp                 # CP2K MD input
│   ├── 1_output.tar.gz                        # MD output (raw CP2K log + restart)
│   └── aligned.tar.gz                         # Aligned starting structures
├── 1_3_dimethyladamantane/
│   ├── 1_3_dimethyladamantane.inp             # CP2K MD input
│   ├── 2_output.tar.gz                        # MD output (raw CP2K log + restart)
│   └── aligned.tar.gz                         # Aligned starting structures
└── 1_3_5_trimethyladamantane/
    ├── 1_3_5_trimethyladamantane.inp          # CP2K MD input
    ├── 3_output.tar.gz                        # MD output (raw CP2K log + restart)
    └── aligned.tar.gz                         # Aligned starting structures
```

---

## Contents

| File / Folder | Description |
|---------------|-------------|
| `align.py` | Python script that aligns the raw crystal-derived geometries to a common reference frame and writes the `aligned.tar.gz` archives. |
| `<system>.inp` | CP2K input file for the 300 K adiabatic MD run (B3LYP, ground state). |
| `<N>_output.tar.gz` | Raw CP2K output for that system (log, trajectory, restart file). |
| `aligned.tar.gz` | The aligned starting geometries used as MD initial conditions. |
| `README.md` | This file. |

System labels used in this step:

| Directory | Chemical name |
|-----------|---------------|
| `adamantane` | Adamantane |
| `1_methyladamantane` | 1-Methyl-adamantane |
| `1_3_dimethyladamantane` | 1,3-Dimethyl-adamantane |
| `1_3_5_trimethyladamantane` | 1,3,5-Trimethyl-adamantane |

---

## Workflow

### 1. Structure Alignment

```bash
python align.py
```

The script reads the raw structural files, aligns them to a common reference
frame, and writes the aligned structures to the corresponding
`aligned.tar.gz` archives. Run this **before** submitting any MD job — STEP2
assumes the aligned starting geometries.

### 2. Adiabatic Molecular Dynamics (300 K)

Using the aligned structures and the per-system `.inp` file, run adiabatic
ground-state MD at 300 K with CP2K:

```bash
cp2k.popt -i adamantane.inp -o adamantane_md.out
```

Repeat for each of the four systems (or submit through your cluster's scheduler).
The B3LYP functional and the 300 K thermostat settings are set inside each
`<system>.inp`; no environment variables are required beyond a working CP2K
build.

**Objective.** Obtain equilibrated adiabatic trajectories at 300 K.

**Output.** Equilibrated ground-state MD trajectories (packed as
`<N>_output.tar.gz`) that provide the 300 K thermal snapshots consumed by
STEP2.

---

## Restarting an Interrupted CP2K MD Run

To continue a previously interrupted simulation, add the following block to the
system's `.inp` file **before** the `&FORCE_EVAL` section:

```
&EXT_RESTART
  RESTART_FILE_NAME adamantane_MD-1_5000.restart
  RESTART_DEFAULT T
&END EXT_RESTART
```

The restart file ships inside the corresponding `<N>_output.tar.gz`. Adjust
`RESTART_FILE_NAME` to the actual restart file in the archive (the numbers
encode the step counter at which the run was checkpointed).

---

## Purpose

Consistent alignment and equilibrated MD trajectories are essential for:

- **Reliable comparison across the methylated series** — every structure is
  aligned to the same reference frame, so differences in electronic properties
  (STEP2–STEP3) are not contaminated by arbitrary rigid-body rotations.
- **Stable MD trajectories in STEP2** — the 300 K snapshots fed to TDDFT are
  already equilibrated, so the vibronic Hamiltonian in STEP3 samples a
  physically meaningful region of configuration space.
- **Consistent active-space selection in STEP3** — the orbital window
  (orbitals 18–46 for adamantane, HOMO = 38, LUMO = 39) is defined relative to
  the aligned, equilibrated geometry produced here.

---

## Notes

- **Reference frame.** `align.py` aligns to a fixed, common reference so that
  the HOMO/LUMO ordering and the active-space window are transferable across
  the four structures. If you re-run the alignment, re-run STEP2 and STEP3.
- **File naming.** The `<N>_output.tar.gz` counter (`0`, `1`, `2`, `3`) is the
  system index used throughout the project (`adamantane = 0`,
  `1_methyl = 1`, `1_3_dimethyl = 2`, `1_3_5_trimethyl = 3`) and matches the
  `PLOT/` folder layout downstream.
- **Reproducibility.** The restart file is preserved inside each
  `<N>_output.tar.gz`, so an interrupted run can be reproduced bit-for-bit by
  replaying the restart block above.

---

## Contact

- Qingxin Zhang (qingxin@buffalo.edu)
- Hamid Zabihi Hashjin (h.zabihi@eng.uk.ac.ir)
- Alexey V. Akimov (alexeyak@buffalo.edu)