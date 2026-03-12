# Symmer-Hamiltonian

A curated database of molecular electronic Hamiltonians in [Symmer](https://github.com/UCL-CCS/symmer)-compatible qubit (Pauli operator) form, generated using the [symmer-pyscf](https://github.com/Kee-Wang/symmer-pyscf) pipeline.

![Dataset Overview](docs/figures/dataset_overview.png)

## Dataset at a Glance

| Metric | Value |
|--------|-------|
| Species | 135 unique (16 monoatomic, 50 diatomic, 69 polyatomic) |
| Hamiltonians | 1,594 across 190 (molecule, basis-set) combinations |
| Spin partition | Singlet ($2S+1 = 1$) |
| Qubit range (stored)  | 2–24 |
| Qubit range (tapered) | 1–20 |
| Encoding | Jordan-Wigner |
| Naming convention | `{formula}_{multiplicity}_{point_group}_{basis}_{alpha}.json`* |

\* $\alpha$ is the geometry scaling factor relative to equilibrium ($1.0$ = equilibrium). Multi-atom species are sampled at 10 $\alpha$ points along the dissociation coordinate; single-atom species have only $\alpha = 1.0$.

## Quick Start

```python
import json
from symmer import PauliwordOp

with open("data/singlet/hamiltonians/H2_singlet_Dooh/H2_singlet_Dooh_sto-3g_1.0.json") as f:
    entry = json.load(f)

H = PauliwordOp.from_dictionary(entry["hamiltonian"])

print(H.n_qubits)                             # 4
print(H.n_terms)                              # 15
print(entry["data"]["calculated_properties"]["FCI"]["energy"])  # -1.1372701746609026
```

## JSON Schema

Each Hamiltonian file has the following structure (H2 / STO-3G at equilibrium shown):

```jsonc
{
  "hamiltonian": {                          // Pauli operator → coefficient (float64)
    "IIZI": -0.22278593040418426,           //   string length = n_qubits
    "IIIZ": -0.22278593040418432,           //   characters: I, X, Y, Z
    "IIII": -0.09886396933545862,           //   identity term = energy offset
    "YXXY":  0.04532220205287397,
    ...
  },
  "data": {
    "molecule_id":    "H2_singlet_Dooh",    // {formula}_{multiplicity}_{point_group}
    "qubit_encoding": "JW",                 // Jordan-Wigner
    "unit":           "Angstrom",           // geometry unit
    "basis":          "sto-3g",             // Gaussian basis set
    "charge":         0,                    // net molecular charge
    "multiplicity":   1,                    // spin multiplicity (2S+1 = 1)
    "spin":           0,                    // 2S (= multiplicity - 1)
    "geometry":       "H 0.0 ... \nH ...",  // XYZ coordinates in Angstroms
    "n_qubits":       4,                    // number of qubits
    "n_particles":    {"total": 2, "alpha": 1, "beta": 1},
    "hf_array":       [1, 1, 0, 0],        // HF occupation vector
    "hf_method":      "pyscf.scf.hf_symm.SymAdaptedRHF",
    "convergence_threshold": 1e-6,          // SCF convergence tolerance
    "point_group":    {"groupname": "Dooh", "topgroup": "Dooh"},
    "alpha":          1.0,                  // geometry scaling (1.0 = equilibrium)

    "calculated_properties": {              // per-method energy + convergence
      "HF":   {"energy": -1.117, "converged": true},
      "MP2":  {"energy": -1.130, "converged": null},
      "CISD": {"energy": -1.137, "converged": true},
      "CCSD": {"energy": -1.137, "converged": true},
      "FCI":  {"energy": -1.137, "converged": true,
               "spin_matches_target": true, "multiplicity": 1.0,
               "oscillatory_converged": false, "oscillation_energy_change": null}
    },

    "auxiliary_operators": {                // Pauli-form operators (same format as hamiltonian)
      "number_operator": { ... },
      "N_alpha": { ... }, "N_beta": { ... },
      "S^2_operator": { ... },
      "UCCSD_operator": { ... }            // conditionally present (see notes below)
    },

    "hf_method_fallback":      "standard",  // SCF strategy: standard|level_shift|newton
    "generation_time_s":       0.024,       // wall-clock time (seconds)
    "timestamp":               "2026-02-25T23:56:22Z"
  }
}
```

**Notes:**
- **Energy values** in `calculated_properties` can be `null` when a method is physically inapplicable (e.g. MP2/CISD/CCSD for single-electron species like H⁺) or when computation produced a non-finite result.
- **`auxiliary_operators`** contains Pauli-form operators; `UCCSD_operator` is conditionally present (absent when CCSD did not converge or when the system has no correlated excitation space, e.g. single-spatial-orbital systems).
- FCI entries always include `oscillatory_converged` (bool) and `oscillation_energy_change` (float|null); these are `false`/`null` for normal convergence.
- **Provenance** (geometry/charge/multiplicity sources) is tracked per-species in [`data/singlet/species_list.json`](data/singlet/species_list.json), not in individual Hamiltonian files.

See [docs/DATA_SCHEMA.md](docs/DATA_SCHEMA.md) for the full field reference.

## Repository Structure

```
Symmer-Hamiltonian/
├── README.md
├── LICENSE                             CC-BY-4.0
│
├── docs/
│   ├── DATA_SCHEMA.md                  JSON format specification
│   ├── data_report.md                  Methodology, Hamiltonian index & data quality
│   └── figures/
│       ├── dataset_overview.png         Used in README.md and data_report.md
│       ├── overview_grid.png           Used in data_report.md
│       ├── plot_gallery.md             Gallery of multi-atom dissociation curves
│       └── dissociation/
│           ├── png/                    Dissociation curve images (web preview)
│           └── pdf/                    Dissociation curve images (publication quality)
│
└── data/
    └── singlet/
        ├── species_list.json           Molecule inventory & provenance
        └── hamiltonians/               1,594 JSON files across 135 species dirs
            ├── H2_singlet_Dooh/
            │   ├── H2_singlet_Dooh_sto-3g_0.5.json
            │   ├── H2_singlet_Dooh_sto-3g_1.0.json
            │   ├── ...
            │   ├── H2_singlet_Dooh_cc-pvdz_0.5.json
            │   ├── H2_singlet_Dooh_cc-pvdz_1.0.json
            │   └── ...
            └── ...
```

Each species directory contains Hamiltonians across **multiple basis sets** (e.g. `sto-3g`, `cc-pvdz`, `6-31g**`, ...) and $\alpha$ points.

## Generation Methodology

Each Hamiltonian is produced by a six-stage pipeline:

1. **Geometry sourcing** from [NIST CCCBDB](https://github.com/Kee-Wang/NIST-CCCBDB-database-mirror), PennyLane, or Symmer reference collections; species are included only if the tapered qubit count is ≤ 20
2. **Geometry scaling** — for multi-atom species, the equilibrium geometry is uniformly scaled by $\alpha \in \{0.5, 0.7, 0.8, 0.9, 1.0, 1.2, 1.5, 2.0, 2.5, 3.0\}$ to sample the dissociation coordinate; density-matrix propagation between $\alpha$ points ensures SCF continuity
3. **Hartree-Fock SCF** with a three-stage convergence cascade (DIIS $\rightarrow$ level-shift $\rightarrow$ Newton)
4. **Post-HF correlation** at MP2, CISD, CCSD, and FCI levels via PySCF
5. **Jordan-Wigner encoding** via Symmer
6. **Qubit tapering metadata** — $\mathbb{Z}_2$ symmetry analysis records achievable tapered qubit counts; stored Hamiltonians remain in the full untapered form

See [docs/data_report.md](docs/data_report.md) for the full methodology, Hamiltonian index, and data quality assessment.

## Documentation

| Document | Description |
|----------|-------------|
| [DATA_SCHEMA.md](docs/DATA_SCHEMA.md) | Complete JSON field reference |
| [data_report.md](docs/data_report.md) | Methodology, Hamiltonian index, data quality |
| [plot_gallery.md](docs/figures/plot_gallery.md) | Visual gallery of all dissociation curves |

## License

This dataset is released under the [CC-BY-4.0](LICENSE) license. You are free to share and adapt the data for any purpose, provided you give appropriate credit.
