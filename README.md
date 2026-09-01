##  Workflow Overview

This project — *Nonadiabatic Molecular Dynamic in Pristine and Methylated Adamantanes* —  
In addition to the dynamic trajectory method, a single-optimization approach was employed.

---

###  **Static Optimization Structure**

Optimize the initial crystal structure using the **B3LYP** functional.

- **Input:** Optimized CIF files,ORCA and CP2K input files with the appropriate `energy_type` setting( 4 different systems)
- **Output:** Ground state and TDDFT output files and corresponding molden files. The TDDFT output files can be further analyzed with Multiwfn to generate UV–Vis spectra in `.txt` format. Different spectral line shapes and degrees of peak broadening can be obtained by adjusting the full width at half maximum (FWHM).

  Related scripts and examples are available in:

  `Static_Optimization_Structure/UV_Vis_spectra`

---

### **Nonadiabatic Molecular Dynamics Workflow**

Step 1: Adiabatic Molecular Dynamics

- **Objective:**  
Obtain equilibrated adiabatic trajectories at 300 K.

- **Output:**  
  Equilibrated ground-state molecular-dynamics trajectories for subsequent electronic-structure calculations.

- **Additional Note: Restarting a CP2K MD Simulation:**

  To continue a previously interrupted CP2K molecular-dynamics simulation, include the following restart settings in the input file:

  ```
  &EXT_RESTART
    RESTART_FILE_NAME adamantane_MD-1_5000.restart
    RESTART_DEFAULT T
  &END EXT_RESTART

---

Step 2: TDDFT Calculations

- **Input:**  
TDDFT Calculations Based on a 300 K Temperature Profile.

- **Output:**  
For visualizing electronic entropy and UV-Vis spectra.

---

Step 3: Vibronic Hamiltonian Construction

- **Objective:**  
Generate the time-dependent vibronic Hamiltonian using the Step 2 outputs and the selected active space.

- **Active space:**  
Take adamantane as an example, Orbitals 18–46, with orbital 38 as the HOMO and orbital 39 as the LUMO.

- **Output:**  
Time-dependent electronic energies, probability distributions and nonadiabatic couplings for the selected electronic states.

---

Step 4: Nonadiabatic Molecular Dynamics

- **Objective:**  
Perform nonadiabatic molecular-dynamics simulations using the Step 3 Hamiltonian data with six different surface-hopping methods: FSSH, FSSH2, GFSH, IDA, IDF, and mSDM.

- **Output:**  
Determine the timescale using relaxation dynamics.

---

[🔝 Back to Top](#top)
