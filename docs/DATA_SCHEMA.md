# Data Schema

Each Hamiltonian is stored as a single JSON file with two top-level keys: `hamiltonian` and `data`.

## File Naming Convention

```
{molecule_id}_{basis}_{alpha}.json
```

- **molecule_id** — `{formula}_{multiplicity}_{point_group}` (e.g. `H2_singlet_Dooh`)
- **basis** — Gaussian basis set in lowercase (e.g. `sto-3g`, `cc-pvdz`)
- **$\alpha$** — Geometry scaling factor with minimal decimal digits (e.g. `1.0`, `0.5`, `2.5`)

## JSON Structure

```jsonc
{
  "hamiltonian": { ... },   // Pauli operator representation
  "data":        { ... }    // Metadata, energies, and auxiliary operators
}
```

### `hamiltonian`

A dictionary mapping Pauli strings to their real-valued coefficients. Each key is a string of length `n_qubits` composed of characters `I`, `X`, `Y`, `Z`. The Hamiltonian is the sum over all terms:

$$H = \sum_i c_i \, P_i$$

where $c_i$ are real-valued coefficients and $P_i$ are Pauli strings.

Example (H2 / STO-3G / 4 qubits):

```json
{
  "IIZI": -0.22278593040418426,
  "IIIZ": -0.22278593040418432,
  "IIZZ":  0.17434844185575657,
  "YXXY":  0.04532220205287397,
  "XYYX":  0.04532220205287397
}
```

The identity term (`"IIII"`) represents the energy offset (nuclear repulsion + constant electronic terms).

### `data`

#### Core Metadata

| Field | Type | Description |
|-------|------|-------------|
| `molecule_id` | string | Unique species identifier: `{formula}_{multiplicity}_{point_group}` |
| `qubit_encoding` | string | Fermion-to-qubit mapping, always `"JW"` (Jordan-Wigner) |
| `unit` | string | Physical unit for geometry, always `"Angstrom"` |
| `basis` | string | Gaussian basis set (e.g. `"sto-3g"`, `"cc-pvdz"`) |
| `charge` | int | Net molecular charge |
| `multiplicity` | int | Spin multiplicity ($2S+1$) |
| `spin` | int | $2S$ (= `multiplicity - 1`); 0 for singlets |
| `geometry` | string | XYZ coordinates in Angstroms, atoms separated by `\n` |
| `n_qubits` | int | Number of qubits in the Hamiltonian ($=$ Pauli string length) |
| `n_particles` | dict | Electron count: `{"total": N, "alpha": N_a, "beta": N_b}` |
| `hf_array` | int[] | Hartree-Fock occupation vector (1 = occupied, 0 = virtual), length `n_qubits` |
| `hf_method` | string | PySCF SCF class (e.g. `"pyscf.scf.hf_symm.SymAdaptedRHF"`) |
| `convergence_threshold` | float | SCF convergence tolerance (typically `1e-6`) |
| `point_group` | dict | Molecular symmetry: `{"groupname": "...", "topgroup": "..."}` |
| `alpha` | float | Geometry scaling factor $\alpha$ relative to equilibrium ($1.0$ = equilibrium) |

#### `data.calculated_properties`

Total electronic energies and convergence status for each method, computed from the same HF orbitals:

```json
{
  "HF":   {"energy": -1.117, "converged": true},
  "MP2":  {"energy": -1.143, "converged": null},
  "CISD": {"energy": -1.148, "converged": true},
  "CCSD": {"energy": -1.149, "converged": true},
  "FCI":  {"energy": -1.151, "converged": true, "spin_matches_target": true, "multiplicity": 1.0,
           "oscillatory_converged": false, "oscillation_energy_change": null}
}
```

| Method | Description |
|--------|-------------|
| `HF` | Restricted Hartree-Fock |
| `MP2` | Second-order Møller–Plesset perturbation theory |
| `CISD` | Configuration interaction (singles and doubles) |
| `CCSD` | Coupled-cluster (singles and doubles) |
| `FCI` | Full configuration interaction (exact within basis) |

Energy values are `null` when a method is physically inapplicable (e.g. MP2/CISD/CCSD require $\geq 2$ electrons) or when computation produced a non-finite result.

**FCI-specific fields:**
- `spin_matches_target` — `true` if the FCI ground state has the target spin multiplicity
- `multiplicity` — spin multiplicity of the actual FCI ground state (may differ from target)
- `oscillatory_converged` — `true` when FCI convergence was achieved via oscillation detection; `false` for normal convergence
- `oscillation_energy_change` (float|null) — energy change at oscillatory convergence detection; `null` when `oscillatory_converged` is `false`

#### `data.auxiliary_operators`

Named auxiliary operators in Pauli form, each a dictionary mapping Pauli strings to coefficients (same format as `hamiltonian`). Present when the operators were successfully computed.

| Key | Description |
|-----|-------------|
| `number_operator` | Total particle number operator $\hat{N}$ |
| `N_alpha` | Alpha (spin-up) electron count operator |
| `N_beta` | Beta (spin-down) electron count operator |
| `S^2_operator` | Total spin-squared operator $\hat{S}^2$ |
| `UCCSD_operator` | UCCSD ansatz generator $-i(T - T^\dagger)$ (absent when CCSD did not converge or when the system has no correlated excitation space, e.g. single-spatial-orbital systems) |

**Note:** He/STO-3G has one spatial orbital (no virtual orbitals), so CCSD amplitudes are trivially zero and `UCCSD_operator` is absent despite CCSD converging. Consumers should use `.get("UCCSD_operator")` to handle its absence gracefully.

**Coefficient format:** `UCCSD_operator` is the Hermitian generator $-i(T - T^\dagger)$. Since Hermitian operators have real coefficients in the Pauli basis, all values are stored as plain floats (same format as the Hamiltonian).

#### Other Fields

| Field | Type | Description |
|-------|------|-------------|
| `hf_method_fallback` | string | SCF convergence strategy: `"standard"`, `"level_shift"`, or `"newton"` |
| `generation_time_s` | float | Wall-clock time in seconds for generating this Hamiltonian |
| `timestamp` | string | ISO 8601 generation timestamp |

#### Provenance

Per-species provenance (geometry source, charge source, multiplicity source) is stored in [`data/singlet/species_list.json`](../data/singlet/species_list.json), not in individual Hamiltonian files.

## Example: Loading a Hamiltonian

```python
import json

with open("data/singlet/hamiltonians/H2_singlet_Dooh/H2_singlet_Dooh_sto-3g_1.0.json") as f:
    entry = json.load(f)

pauli_dict = entry["hamiltonian"]   # {"IIZI": -0.222, "IIIZ": -0.222, ...}
n_qubits   = entry["data"]["n_qubits"]
fci_energy = entry["data"]["calculated_properties"]["FCI"]["energy"]
hf_array   = entry["data"]["hf_array"]   # [1, 1, 0, 0]

# Auxiliary operators (e.g., UCCSD ansatz)
aux = entry["data"].get("auxiliary_operators", {})
uccsd = aux.get("UCCSD_operator")  # may be None if CCSD didn't converge
```
