# Molecule Energy Estimator using VQE: H₂ on IBM Quantum Hardware

**Qiskit Fall Fest 2026 – Quantum Build Challenge (Track I5)**

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/oddeeksha/h2-vqe-ibm-quantum-qiskit/blob/main/H2_VQE_Project.ipynb)

## 1. Overview and Objectives

The challenge is to estimate the ground-state energy of a molecule using the Variational Quantum Eigensolver (VQE). This project implements the end-to-end VQE pipeline for molecular hydrogen ($\text{H}_2$) at its equilibrium bond length ($0.735\text{ Å}$) in the STO-3G basis.

**Objectives:**
- Construct the $\text{H}_2$ qubit Hamiltonian and compute the exact classical ground-state energy as a benchmark.
- Estimate the energy using VQE and evaluate performance across two classical optimizers: COBYLA and SPSA.
- Map the potential energy surface (PES) over interatomic bond distances from $0.3\text{ Å}$ to $2.5\text{ Å}$.
- Simulate realistic physical noise using live calibration data from an IBM Quantum backend (`ibm_kingston`).
- Execute the optimized circuit on real physical quantum hardware (`ibm_kingston`) and compare results against classical ground truth.

---

## 2. Environment Setup and Execution

**Files:** `H2_VQE_Project.ipynb` (contains full code and pre-rendered outputs for review).

**Run in Google Colab:**
1. Click the **Open in Colab** badge above or upload `H2_VQE_Project.ipynb` to Google Colab.
2. Generate an API token on the IBM Quantum Platform.
3. Open the **Secrets** (key icon) sidebar in Colab, add a secret named `IBM_QUANTUM_TOKEN` with your API key, and enable notebook access.
4. Execute cells sequentially. The setup cell installs all necessary packages:
   `qiskit qiskit-aer qiskit-algorithms qiskit-nature pyscf qiskit-ibm-runtime pylatexenc matplotlib`
5. If Colab prompts a runtime restart after installation, restart the session and continue from the next cell without rerunning the installation cell.

**Reviewer Notes:**
- **Local vs. Cloud Execution:** Steps 1 through 5 (molecule definition, Hamiltonian construction, exact classical benchmark, VQE optimization, and PES mapping) execute locally without requiring an IBM login. Authentication, Step 6 (noise simulation), and Step 7 (hardware execution and comparison) require an active IBM Quantum API token.
- **Hardware Queue:** Physical execution in Step 7a submits to the `ibm_kingston` queue, which may take time depending on traffic.
- **Backend Selection:** Hardware execution targets `ibm_kingston`. If `ibm_kingston` is offline or unavailable to your account, update the backend name in Steps 6 and 7a.
- **Reproducibility:** No global random seeds are fixed, so re-executing cells will yield numerical variations relative to the saved outputs and the results reported below.
- **Tested Versions:** Qiskit 2.5.2, Qiskit Aer 0.17.2, Qiskit Algorithms 0.4.0, Qiskit Nature 0.8.0, Qiskit IBM Runtime 0.50.0, PySCF 2.14.0.

---

## 3. Results & Execution Environment Benchmarks

Calculations for $\text{H}_2$ at $0.735\text{ Å}$ in the STO-3G basis. All values represent total energies (electronic energy + nuclear repulsion of $0.719969\text{ Ha}$):

| Environment | Total Energy (Ha) | Absolute Error vs. Exact (Ha) |
| :--- | :---: | :---: |
| **Exact Classical Baseline (NumPy)** | **-1.137306** | **0.000000** |
| **Ideal VQE (COBYLA Statevector)** | **-1.133817** | **0.003489** |
| **Ideal VQE (SPSA Statevector)** | **-1.114348** | **0.022958** |
| **Noisy VQE (Aer Simulator)** | **-1.102307** | **0.034999** |
| **Physical IBM Hardware (`ibm_kingston`)** | **-1.099869** | **0.037437** |

### Key Results Summary
* **Ideal VQE Accuracy:** COBYLA achieved an error of $0.0035\text{ Ha}$ ($\approx 2\times$ the chemical accuracy threshold of $0.0016\text{ Ha}$). SPSA achieved $0.0230\text{ Ha}$ ($\approx 14\times$ the threshold).
* **Hardware Noise Agreement:** The Aer noise simulation ($0.0350\text{ Ha}$ error) agreed with physical hardware execution ($0.0374\text{ Ha}$ error) to within $0.0024\text{ Ha}$ in this single run.
* **PES Scan:** Mapped across 12 distances ($0.3\text{ Å}$ to $2.5\text{ Å}$). The VQE curve tracks exact diagonalization closely at the extremes, but deviates by roughly $0.1\text{ Ha}$ near $1.5\text{ Å}$ due to iteration limits and local minima.

---

## 4. Technical Implementation Details

* **Hamiltonian & Mapping:** Second-quantized electronic Hamiltonian constructed via PySCF in Qiskit Nature and mapped to 4 qubits (15 Pauli terms) using the Jordan-Wigner transformation. The exact energy comes from diagonalizing the qubit Hamiltonian matrix with NumPy.
* **Ansatz Circuit:** `TwoLocal` circuit on 4 qubits using $R_y$ single-qubit rotation layers and linear $CZ$ entanglement blocks across 2 repetitions (12 parameters total). Initialized from the default zero state $\vert{}0000\rangle$.
* **Optimizer Dynamics:**
  * **COBYLA (100 iterations):** Gradient-free local optimizer. Converged smoothly within ~70 evaluations to the lowest energy in this run.
  * **SPSA (100 iterations):** Stochastic optimizer. Showed high variance during the initial ~100 function evaluations.
  * Each optimizer was run once from random starting points, so this comparison is not conclusive (the ranking was reversed in an earlier run of this notebook).
* **Noise Modeling:** `AerSimulator.from_backend(ibm_kingston)` constructed a noise profile incorporating gate error rates, readout errors, and thermal relaxation based on the device's calibration data. VQE was run under this model with SPSA.
* **Hardware Execution:** The optimal COBYLA parameters were bound to the ansatz, transpiled with `optimization_level=3` for `ibm_kingston`, and evaluated via `qiskit-ibm-runtime` (`executor_estimator.Estimator`) in job mode without error mitigation.

---

## 5. Evidence of Physical Hardware Execution

| Item | Detail |
| :--- | :--- |
| **Backend Name** | `ibm_kingston` |
| **Active Job ID** | `db3i3bsvf2bc73ct776g` |
| **Hardware Measured Total Energy** | **-1.099869 Ha** |
| **Hardware Error vs. Exact** | **0.037437 Ha** |
| **Bound Parameters Source** | Optimal COBYLA parameters from ideal simulation |
| **Historical Job Executions (earlier parameter set)** | `db3hft4lf4us73c1ab90` (-1.0820 Ha), `db3hlu4lf4us73c1an20` (-1.0769 Ha) |

The submitted Job IDs can be verified on the IBM Quantum Platform jobs page of the account that submitted them. The two historical jobs ran the same circuit with the same parameters and differed by about $0.005\text{ Ha}$, which shows the run-to-run variation of a single hardware evaluation.

---

## 6. Analytical Insights & Limitations

* **Hardware Noise Degradation:** Simulated noise added $0.0315\text{ Ha}$ and physical hardware added $0.0339\text{ Ha}$ of error above the ideal simulation baseline.
* **Methodology Difference:** The Aer simulation re-optimized parameters using SPSA under noise, while the hardware execution evaluated a single-point expectation value using pre-optimized COBYLA parameters. The two are therefore not identical experiments.
* **Hartree-Fock Reference State:** Under the Jordan-Wigner transformation used here, the Hartree-Fock reference state for $\text{H}_2$ is $\vert{}0101\rangle$. Prepending a `HartreeFock` initial state to the ansatz, rather than starting from $\vert{}0000\rangle$, together with more optimizer iterations, would be expected to improve optimization and bring the ideal error closer to chemical accuracy ($0.0016\text{ Ha}$).
* **PES Scan Limits:** The PES scan uses a short optimizer budget (100 iterations) and random restarts at each distance, which causes the deviation near $1.5\text{ Å}$.
* **Single Hardware Evaluation:** The hardware result is one evaluation with no error mitigation.
