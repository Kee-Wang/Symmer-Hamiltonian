# Symmer-Hamiltonian

A curated database of molecular electronic Hamiltonians in [Symmer](https://github.com/UCL-CCS/symmer)-compatible qubit (Pauli operator) form, generated using the [symmer-pyscf](https://github.com/Kee-Wang/symmer-pyscf) pipeline.

![Dataset Overview](docs/figures/dataset_overview.png)

## Dataset at a Glance

| Metric | Value |
|--------|-------|
| Species | 190 (34 single-atom, 82 diatomic, 74 polyatomic) |
| Hamiltonians | 1,594 |
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
print(entry["data"]["energies"]["FCI"])        # -1.137270174660904
```

## JSON Schema

Each Hamiltonian file has the following structure (H2 / STO-3G at equilibrium shown):

```jsonc
{
  "hamiltonian": {                          // Pauli operator → coefficient (float64)
    "IIZI": -0.22278593040418382,           //   string length = n_qubits
    "IIIZ": -0.22278593040418382,           //   characters: I, X, Y, Z
    "IIII": -0.09886396933546007,           //   identity term = energy offset
    "YXXY":  0.045322202052874,
    ...
  },
  "data": {
    "molecule_id":    "H2_singlet_Dooh",    // {formula}_{multiplicity}_{point_group}
    "qubit_encoding": "JW",                 // Jordan-Wigner
    "basis":          "sto-3g",             // Gaussian basis set
    "charge":         0,                    // net molecular charge
    "multiplicity":   1,                    // spin multiplicity (2S+1 = 1)
    "geometry":       "H 0.0 ... \nH ...",  // XYZ coordinates in Angstroms
    "n_qubits":       4,                    // number of qubits
    "alpha":          1.0,                  // geometry scaling (1.0 = equilibrium)

    "energies": {                           // total electronic energies (Hartrees, float64)
      "HF":   -1.1166843870853413,
      "MP2":  -1.1298551535553103,
      "CISD": -1.1372701746609035,
      "CCSD": -1.1372703406409206,
      "FCI":  -1.137270174660904            // exact within basis set
    },
    "converged": {                          // convergence flags per method
      "HF": true, "MP2": false,
      "CISD": true, "CCSD": true, "FCI": true
    },

    "fci_spin_matches_target": true,        // FCI ground state has target spin?
    "fci_multiplicity":        1.0,         // actual FCI ground-state multiplicity
    "hf_method_fallback":      "standard",  // SCF strategy: standard|level_shift|newton
    "generation_time_s":       0.024,       // wall-clock time (seconds)
    "timestamp":               "2026-02-25T23:56:22Z"
  }
}
```

**Notes:**
- **Energy values** can be `null` when a method is physically inapplicable (e.g. MP2/CISD/CCSD for single-electron species like H⁺) or when computation produced a non-finite result.
- Some files include optional fields `fci_oscillatory_converged` (bool) and `fci_oscillation_energy_change` (float) when FCI convergence was achieved via oscillation detection rather than standard convergence.
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
        ├── generation_summary.json     Pipeline run metadata
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

Each Hamiltonian is produced by a four-stage pipeline:

1. **Geometry sourcing** from [NIST CCCBDB](https://github.com/Kee-Wang/NIST-CCCBDB-database-mirror), PennyLane, or Symmer reference collections; species are included only if the tapered qubit count is ≤ 20
2. **Hartree-Fock SCF** with a three-stage convergence cascade (DIIS $\rightarrow$ level-shift $\rightarrow$ Newton)
3. **Post-HF correlation** at MP2, CISD, CCSD, and FCI levels via PySCF
4. **Jordan-Wigner encoding** via Symmer; $\mathbb{Z}_2$ symmetry analysis records achievable tapered qubit counts (stored Hamiltonians are untapered)

For multi-atom species, the equilibrium geometry is uniformly scaled by a factor $\alpha$ to sample the dissociation coordinate. Density-matrix propagation between $\alpha$ points ensures SCF continuity.

See [docs/data_report.md](docs/data_report.md) for the full methodology, Hamiltonian index, and data quality assessment.

## Documentation

| Document | Description |
|----------|-------------|
| [DATA_SCHEMA.md](docs/DATA_SCHEMA.md) | Complete JSON field reference |
| [data_report.md](docs/data_report.md) | Methodology, Hamiltonian index, data quality |
| [plot_gallery.md](docs/figures/plot_gallery.md) | Visual gallery of all dissociation curves |

## License

This dataset is released under the [CC-BY-4.0](LICENSE) license. You are free to share and adapt the data for any purpose, provided you give appropriate credit.
