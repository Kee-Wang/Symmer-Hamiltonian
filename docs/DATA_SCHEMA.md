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
  "data":        { ... }    // Metadata and computed energies
}
```

### `hamiltonian`

A dictionary mapping Pauli strings to their real-valued coefficients. Each key is a string of length `n_qubits` composed of characters `I`, `X`, `Y`, `Z`. The Hamiltonian is the sum over all terms:

$$H = \sum_i c_i \, P_i$$

where $c_i$ are real-valued coefficients and $P_i$ are Pauli strings.

Example (H2 / STO-3G / 4 qubits):

```json
{
  "IIZI": -0.22278593040418382,
  "IIIZ": -0.22278593040418382,
  "IIZZ":  0.17434844185575626,
  "YXXY":  0.045322202052874,
  "XYYX":  0.045322202052874
}
```

The identity term (`"IIII"`) represents the energy offset (nuclear repulsion + constant electronic terms).

### `data`

| Field | Type | Description |
|-------|------|-------------|
| `molecule_id` | string | Unique species identifier: `{formula}_{multiplicity}_{point_group}` |
| `qubit_encoding` | string | Fermion-to-qubit mapping, always `"JW"` (Jordan-Wigner) |
| `basis` | string | Gaussian basis set (e.g. `"sto-3g"`, `"cc-pvdz"`) |
| `charge` | int | Net molecular charge |
| `multiplicity` | int | Spin multiplicity ($2S+1$); always `1` for the singlet partition |
| `geometry` | string | XYZ coordinates in Angstroms, atoms separated by `\n` |
| `n_qubits` | int | Number of qubits in the Hamiltonian ($=$ Pauli string length) |
| `alpha` | float | Geometry scaling factor $\alpha$ relative to equilibrium ($1.0$ = equilibrium) |

#### `data.energies`

Total electronic energies in Hartrees for each method, computed from the same HF orbitals:

| Key | Method |
|-----|--------|
| `HF` | Restricted Hartree-Fock |
| `MP2` | Second-order M&oslash;ller–Plesset perturbation theory |
| `CISD` | Configuration interaction (singles and doubles) |
| `CCSD` | Coupled-cluster (singles and doubles) |
| `FCI` | Full configuration interaction (exact within basis) |

Values can be `null` when a method is physically inapplicable (e.g. MP2, CISD, and CCSD require at least two electrons, so single-electron species like H⁺ have `null` for these) or when computation produced a non-finite result.

#### `data.converged`

Boolean convergence flags for each method. Note that `MP2` always reports `false` because PySCF does not expose a convergence flag for MP2 (the energy is computed in a single step and is always valid).

#### `data.fci_spin_matches_target`

`true` if the FCI ground state has the target spin multiplicity. `false` indicates the lowest-energy FCI state has a different spin — the Hamiltonian is still valid but the FCI energy corresponds to a state of different multiplicity.

#### `data.fci_multiplicity`

The spin multiplicity of the FCI ground state (may differ from the target `multiplicity`).

#### `data.hf_method_fallback`

Which SCF convergence strategy succeeded:

| Value | Meaning |
|-------|---------|
| `"standard"` | Default DIIS solver |
| `"level_shift"` | Level-shifted SCF ($0.5\,E_h$ shift on virtual orbitals) |
| `"newton"` | Second-order Newton orbital optimisation |

#### `data.generation_time_s`

Wall-clock time in seconds for generating this single Hamiltonian.

#### `data.timestamp`

ISO 8601 timestamp of when this Hamiltonian was generated.

#### `data.fci_oscillatory_converged`

*(bool, optional)* — Present when FCI convergence was achieved via oscillation detection rather than standard convergence. `true` indicates the FCI energy was determined by detecting a stable oscillation pattern in the Davidson iterations.

#### `data.fci_oscillation_energy_change`

*(float|null, optional)* — Present alongside `fci_oscillatory_converged`. The energy change (in Hartrees) at the point where oscillatory convergence was detected. Smaller values indicate more reliable convergence.

#### Provenance

Per-species provenance (geometry source, charge source, multiplicity source) is stored in [`data/singlet/species_list.json`](../data/singlet/species_list.json), not in individual Hamiltonian files.

## Example: Loading a Hamiltonian

```python
import json

with open("data/singlet/hamiltonians/H2_singlet_Dooh/H2_singlet_Dooh_sto-3g_1.0.json") as f:
    entry = json.load(f)

pauli_dict = entry["hamiltonian"]   # {"IIZI": -0.222, "IIIZ": -0.222, ...}
n_qubits   = entry["data"]["n_qubits"]
fci_energy = entry["data"]["energies"]["FCI"]
```
