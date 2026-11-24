# HHL Quantum Algorithm — Quantum Linear Systems Solver

A theoretical study and Qiskit implementation of the Harrow–Hassidim–Lloyd (HHL) algorithm for solving linear systems of equations using quantum computing.

The project explores the algorithm's mathematical foundations, circuit construction, and experimental behavior through local simulation and execution on real IBM Quantum hardware.

## Authors

- Angelica Porco
- Hanna Andrieieva

## Academic Context

Developed as a university project focused on Quantum Computing, combining theoretical analysis with practical implementation.

The project explores the potential and limitations of quantum algorithms for linear algebra problems, with particular attention to the gap between ideal quantum simulation and execution on real quantum hardware.

## Project Overview

The HHL algorithm is a quantum algorithm designed to solve linear systems of the form:

$$
A\mathbf{x}=\mathbf{b}
$$

where $A$ is a matrix, $\mathbf{x}$ is the unknown solution vector, and $\mathbf{b}$ is the input vector.

Rather than directly producing the classical solution vector, HHL prepares a quantum state proportional to:

$$
|x\rangle \propto A^{-1}|b\rangle
$$

Under specific assumptions concerning matrix sparsity, condition number, state preparation, and output extraction, the algorithm can offer a theoretical advantage over classical approaches for certain problems.

This project focuses on understanding and implementing the main stages of HHL, evaluating its performance on small systems, and comparing ideal simulation results with execution on real quantum hardware.

## Main Objectives

- Study the theoretical foundations of the HHL algorithm.
- Understand the role of Quantum Phase Estimation in encoding eigenvalues.
- Implement the complete quantum circuit using Qiskit.
- Test linear systems of dimensions $2\times2$ and $4\times4$.
- Compare quantum results with classical solutions.
- Evaluate circuit behavior on both local simulators and real IBM Quantum hardware.
- Investigate the impact of circuit complexity and hardware noise on solution fidelity.

## Algorithm Workflow

The implementation follows the main stages of the HHL algorithm.

### 1. State Preparation

The input vector $\mathbf{b}$ is normalized and encoded into the amplitudes of the system register, preparing the quantum state:

$$
|b\rangle=\sum_j \beta_j|u_j\rangle
$$

where $|u_j\rangle$ denotes an eigenvector of the matrix $A$, and $\beta_j$ represents the corresponding coefficient in the eigenbasis.

The matrix is assumed to be Hermitian and admits the spectral decomposition:

$$
A=\sum_j \lambda_j|u_j\rangle\langle u_j|
$$

where $\lambda_j$ are its eigenvalues.

### 2. Quantum Phase Estimation (QPE)

Quantum Phase Estimation is used to encode the eigenvalues of the matrix into the phase register.

The implementation constructs the unitary operator:

$$
U=e^{iAt}
$$

and applies controlled powers of this operator, followed by the Inverse Quantum Fourier Transform (IQFT).

For an eigenvector $|u_j\rangle$, the unitary evolution is:

$$
U|u_j\rangle=e^{i\lambda_jt}|u_j\rangle
$$

The resulting register contains an approximation of the eigenvalue phases.

### 3. Conditional Eigenvalue Inversion

A controlled rotation is applied to an auxiliary qubit to encode the inverse eigenvalue information.

The rotation angle is defined as:

$$
\theta_j=2\arcsin\left(\frac{C}{\tilde{\lambda}_j}\right)
$$

where $\tilde{\lambda}_j$ is the estimated eigenvalue and $C$ is a scaling constant chosen to ensure that the rotation is physically valid.

The corresponding rotation operation is:

$$
R_Y(\theta_j)=
\begin{pmatrix}
\cos(\theta_j/2) & -\sin(\theta_j/2)\\
\sin(\theta_j/2) & \cos(\theta_j/2)
\end{pmatrix}
$$

This operation encodes the inverse eigenvalue information into the amplitude of the auxiliary qubit.

### 4. Inverse QPE

The inverse Quantum Phase Estimation circuit is applied to uncompute the phase register, disentangling it from the system register.

This step removes the eigenvalue information from the control register while preserving the information encoded in the amplitudes of the system state.

### 5. Post-selection

Measurements are filtered according to the auxiliary qubit outcome.

When the auxiliary qubit is measured in state $|1\rangle$, the system register contains a state proportional to the solution:

$$
|x\rangle\propto A^{-1}|b\rangle
$$

The normalization factor is determined by the success probability of the controlled rotation and post-selection procedure.

## Implementation

The algorithm is implemented in Python using Qiskit.

The notebook supports different execution configurations through a set of global parameters.

| Parameter | Description |
|---|---|
| `DIMENSIONE_SISTEMA` | Selects a $2\times2$ or $4\times4$ linear system |
| `BACKEND_TIPO` | Selects local simulation or real IBM Quantum hardware |
| `M_QUBIT_CONTROLLO` | Sets the number of control qubits used by QPE |
| `NUM_SHOTS` | Defines the number of circuit executions |

The default configuration uses three control qubits and $16\,384$ shots.

The matrix exponential is computed using `scipy.linalg.expm`, allowing the construction of the unitary operator used in the QPE circuit.

## Experimental Results

Four configurations were evaluated:

- $2\times2$ system on a local simulator.
- $4\times4$ system on a local simulator.
- $2\times2$ system on real IBM Quantum hardware.
- $4\times4$ system on real IBM Quantum hardware.

### Simulation Results

| System | Fidelity | Success Probability | Theoretical Bound $1/\kappa^2$ |
|---|---:|---:|---:|
| $2\times2$ | 0.957 | 0.349 | 0.250 |
| $4\times4$ | 0.999 | 0.289 | 0.143 |

The $4\times4$ system achieves higher fidelity in simulation. This behavior is related to how its eigenvalue phases align with the phase discretization grid of QPE using three control qubits.

### Real IBM Quantum Hardware

| System | Fidelity | Fidelity Difference vs. Simulator | Two-Qubit Gates |
|---|---:|---:|---:|
| $2\times2$ | 0.934 | $-0.023$ | 70 |
| $4\times4$ | 0.706 | $-0.293$ | 242 |

The experimental results highlight the impact of hardware noise on circuit execution.

The $2\times2$ implementation maintains relatively high fidelity, with a limited reduction compared with ideal simulation.

The $4\times4$ implementation experiences a more significant degradation. Its larger circuit, requiring 242 two-qubit gates, is more exposed to accumulated noise and errors in the physical device.

These results illustrate the challenges of executing increasingly complex quantum algorithms on current Noisy Intermediate-Scale Quantum (NISQ) hardware.

## QPE Precision Analysis

The number of control qubits determines the phase estimation resolution:

$$
\Delta\phi=\frac{1}{2^m}
$$

where $m$ is the number of control qubits.

The implementation investigates different values of $m$.

| Control Qubits ($m$) | Phase Resolution ($\Delta\phi$) | Result |
|---:|---:|---|
| 2 | 0.250 | Insufficient |
| 3 | 0.125 | Verified |
| 4 | 0.0625 | Verified |
| $\geq 5$ | Halved with each additional qubit | Verified |

Increasing the number of control qubits improves the theoretical phase resolution, reducing the discretization error of the estimation.

The experiments indicate that two control qubits are insufficient for the tested systems, whereas three or more provide adequate resolution for the configurations considered.

## Limitations and Practical Considerations

Despite its theoretical potential, HHL has important practical limitations.

### Quantum State Preparation

Encoding a classical vector $\mathbf{b}$ into a quantum state can require $O(N)$ operations without specialized quantum memory, potentially reducing the overall advantage for classical input data.

### Hamiltonian Simulation

The implementation uses exact matrix exponentiation through `scipy.linalg.expm`, producing dense unitary operators.

This approach is suitable for small systems but does not scale efficiently to large matrices. More scalable implementations would require techniques such as Trotter–Suzuki decomposition or other Hamiltonian simulation methods.

### Condition Number

The condition number of a matrix $A$ is defined as:

$$
\kappa(A)=\|A\|\cdot\|A^{-1}\|
$$

The complexity of HHL depends strongly on the sparsity $s$, the condition number $\kappa$, and the desired precision $\epsilon$.

The algorithm is most promising for sparse, well-conditioned matrices. Its theoretical advantage is therefore dependent on the properties of the input problem.

### Output Extraction

HHL produces a quantum state rather than a classical solution vector.

Recovering all components of the classical solution requires repeated measurements and can eliminate the theoretical advantage. The algorithm is more suitable for applications where the objective is to estimate specific properties of the solution state, such as expectation values.

## Technologies

- **Python**
- **Qiskit**
- **Qiskit Aer**
- **IBM Quantum**
- **NumPy**
- **SciPy**
- **Jupyter Notebook**
