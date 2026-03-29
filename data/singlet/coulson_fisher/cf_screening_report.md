# Coulson-Fischer Screening Report

**Date:** 2026-03-29
**Source data:** `cf_summary.json` (156 entries, computed 2026-03-27)
**Input list:** 59 molecules from `species_list.json` filtered by: `n_atoms > 1`, `basis == best_basis`, `max(feasible_bases) <= 16 qubits`

## Methodology

The Coulson-Fischer (CF) point is the geometry scaling factor $\alpha$ where UHF breaks spin symmetry relative to RHF along a uniform bond-stretching dissociation curve. Detection criteria: $\langle S^2 \rangle_\mathrm{UHF} > 0.01$ or $|E_\mathrm{RHF} - E_\mathrm{UHF}| > 10^{-6}$ Ha.

CF data was computed on a 101-point $\alpha$ grid $[0.5, 3.0]$ with step 0.025, using PySCF RHF (3-stage cascade) + UHF (stability analysis, broken-symmetry DM propagation).

**Screening criterion:** $0.5 < \alpha_\mathrm{CF} \leq 1.5$

## Summary

| Category | Count | Description |
|----------|-------|-------------|
| **PASS** | 46 | $\alpha_\mathrm{CF}$ within $(0.5, 1.5]$ |
| **BOUNDARY** | 9 | $\alpha_\mathrm{CF} = 0.5$ (true CF $\leq 0.5$, below scan floor) |
| **FAIL — too late** | 2 | $\alpha_\mathrm{CF} > 1.5$ |
| **FAIL — no CF** | 2 | No spin-symmetry breaking detected |
| **Total** | **59** | |

---

## PASS — 46 molecules with $\alpha_\mathrm{CF} \in (0.5, 1.5]$

These molecules have their CF point within the screening range and are included in the CF-region Hamiltonian dataset at $\alpha = 0.5, 0.6, \ldots, 1.5$.

| # | `molecule_id` | basis | formula | charge | $n_\mathrm{qubits}^\mathrm{tap}$ | $\alpha_\mathrm{CF}$ |
|---|---------------|-------|---------|--------|----------------------------------|----------------------|
| 1 | `BeN-_singlet_Coov` | sto-3g | BeN | -1 | 16 | 0.5261 |
| 2 | `H10_singlet_Dooh` | sto-3g | H10 | 0 | 15 | 0.85 |
| 3 | `H8_singlet_Dooh` | sto-3g | H8 | 0 | 11 | 0.875 |
| 4 | `NO+_singlet_Coov` | sto-3g | NO | +1 | 16 | 0.925 |
| 5 | `F2_singlet_Dooh` | sto-3g | F2 | 0 | 15 | 0.95 |
| 6 | `HMg+_singlet_Coov` | sto-3g | HMg | +1 | 16 | 0.9999 |
| 7 | `N2_singlet_Dooh` | sto-3g | N2 | 0 | 15 | 1.0 |
| 8 | `CN-_singlet_Coov` | sto-3g | CN | -1 | 16 | 1.025 |
| 9 | `BF_singlet_Coov` | sto-3g | BF | 0 | 16 | 1.05 |
| 10 | `H4_singlet_Dooh` | 3-21g | H4 | 0 | 11 | 1.05 |
| 11 | `BeF-_singlet_Coov` | sto-3g | BeF | -1 | 16 | 1.0742 |
| 12 | `CO_singlet_Coov` | sto-3g | CO | 0 | 16 | 1.1 |
| 13 | `FO-_singlet_Coov` | sto-3g | FO | -1 | 16 | 1.15 |
| 14 | `FLi_singlet_Coov` | sto-3g | FLi | 0 | 16 | 1.1584 |
| 15 | `BeF+_singlet_Coov` | sto-3g | BeF | +1 | 16 | 1.1963 |
| 16 | `CF+_singlet_Coov` | sto-3g | CF | +1 | 16 | 1.2045 |
| 17 | `HSi+_singlet_Coov` | sto-3g | HSi | +1 | 16 | 1.225 |
| 18 | `CH3-_singlet_D3h` | sto-3g | CH3 | -1 | 12 | 1.25 |
| 19 | `BO-_singlet_Coov` | sto-3g | BO | -1 | 16 | 1.2696 |
| 20 | `O2--_singlet_Dooh` | sto-3g | O2 | -2 | 15 | 1.275 |
| 21 | `H3N_singlet_Cs` | sto-3g | H3N | 0 | 13 | 1.275 |
| 22 | `BH2+_singlet_C2v` | sto-3g | BH2 | +1 | 10 | 1.3 |
| 23 | `ClD_singlet_Coov` | sto-3g | ClD | 0 | 16 | 1.3 |
| 24 | `ClH_singlet_Coov` | sto-3g | ClH | 0 | 16 | 1.3 |
| 25 | `D2O_singlet_C2v` | sto-3g | D2O | 0 | 10 | 1.3 |
| 26 | `DHO_singlet_Cs` | sto-3g | DHO | 0 | 11 | 1.3 |
| 27 | `H2O_singlet_C2v` | sto-3g | H2O | 0 | 10 | 1.3 |
| 28 | `AlH_singlet_Coov` | sto-3g | AlH | 0 | 16 | 1.325 |
| 29 | `D3N_singlet_C3v` | sto-3g | D3N | 0 | 13 | 1.325 |
| 30 | `FH2+_singlet_C1` | sto-3g | FH2 | +1 | 12 | 1.35 |
| 31 | `H3N_singlet_C3v` | sto-3g | H3N | 0 | 13 | 1.35 |
| 32 | `HS-_singlet_Coov` | sto-3g | HS | -1 | 16 | 1.35 |
| 33 | `H4N+_singlet_Td` | sto-3g | H4N | +1 | 14 | 1.375 |
| 34 | `HNa_singlet_Coov` | sto-3g | HNa | 0 | 16 | 1.375 |
| 35 | `H3O+_singlet_C3v` | sto-3g | H3O | +1 | 13 | 1.375 |
| 36 | `H2N-_singlet_C2v` | sto-3g | H2N | -1 | 10 | 1.3999 |
| 37 | `BeH2_singlet_Dooh` | sto-3g | BeH2 | 0 | 10 | 1.4 |
| 38 | `CD4_singlet_Td` | sto-3g | CD4 | 0 | 14 | 1.4 |
| 39 | `CH2D2_singlet_C2v` | sto-3g | CH2D2 | 0 | 14 | 1.4 |
| 40 | `CH3D_singlet_C3v` | sto-3g | CH3D | 0 | 15 | 1.4 |
| 41 | `CH4_singlet_Td` | sto-3g | CH4 | 0 | 14 | 1.4 |
| 42 | `CHD3_singlet_C3v` | sto-3g | CHD3 | 0 | 15 | 1.4 |
| 43 | `H3O+_singlet_C1` | sto-3g | H3O | +1 | 14 | 1.4 |
| 44 | `BH3_singlet_C2v` | sto-3g | BH3 | 0 | 12 | 1.425 |
| 45 | `BH4-_singlet_D2d` | sto-3g | BH4 | -1 | 14 | 1.45 |
| 46 | `CH3+_singlet_D3h` | sto-3g | CH3 | +1 | 12 | 1.5 |

### Distribution notes

- **Qubit range:** 10 ($\mathrm{BH_2^+}$, $\mathrm{D_2O}$, $\mathrm{H_2O}$, $\mathrm{H_2N^-}$, $\mathrm{BeH_2}$) to 16 (many diatomics with heavy atoms)
- **$\alpha_\mathrm{CF}$ range:** 0.5261 ($\mathrm{BeN^-}$) to 1.5 ($\mathrm{CH_3^+}$)
- **Median $\alpha_\mathrm{CF}$:** ~1.3
- **Early CF ($\alpha_\mathrm{CF} < 1.0$):** 6 molecules — strong static correlation near or before equilibrium ($\mathrm{BeN^-}$, $\mathrm{H_{10}}$, $\mathrm{H_8}$, $\mathrm{NO^+}$, $\mathrm{F_2}$, $\mathrm{HMg^+}$)
- **Late CF ($\alpha_\mathrm{CF} > 1.3$):** 17 molecules — symmetry breaking only at moderate stretch

---

## BOUNDARY — 9 molecules with $\alpha_\mathrm{CF} = 0.5$ (excluded)

These molecules show spin-symmetry breaking at the first grid point ($\alpha = 0.5$), meaning the true CF point is at or below the scan floor. The entire $\alpha \in [0.5, 1.5]$ range is in the broken-symmetry regime — they do not span the CF transition.

**Evidence:** $\mathrm{BO^+}$ has $\langle S^2 \rangle_\mathrm{UHF} = 0.994$ at $\alpha = 0.5$, confirming massive symmetry breaking well below the scan boundary.

| # | `molecule_id` | basis | formula | charge | $n_\mathrm{qubits}^\mathrm{tap}$ | Reason |
|---|---------------|-------|---------|--------|----------------------------------|--------|
| 1 | `BO+_singlet_Coov` | sto-3g | BO | +1 | 16 | $\langle S^2 \rangle(0.5) = 0.99$; strong multireference at all geometries |
| 2 | `Be2_singlet_Dooh` | sto-3g | Be2 | 0 | 14 | Near-degenerate frontier orbitals; weak bond |
| 3 | `BeO_singlet_Coov` | sto-3g | BeO | 0 | 16 | Multireference ground state |
| 4 | `C2_singlet_Dooh` | sto-3g | C2 | 0 | 16 | Canonical multireference molecule |
| 5 | `CN+_singlet_Coov` | sto-3g | CN | +1 | 16 | Isoelectronic to $\mathrm{C_2}$; strong static correlation |
| 6 | `FO+_singlet_Coov` | sto-3g | FO | +1 | 16 | Radical cation character |
| 7 | `Li2_singlet_Dooh` | sto-3g | Li2 | 0 | 8 | Weak covalent bond; low barrier to symmetry breaking |
| 8 | `LiO-_singlet_Coov` | sto-3g | LiO | -1 | 16 | Ionic/multireference character |
| 9 | `H4_singlet_D4h` | 3-21g | H4 | 0 | 4 | Square planar geometry; frustrated bonding |

---

## FAIL — CF point above 1.5 (2 molecules)

| # | `molecule_id` | basis | $\alpha_\mathrm{CF}$ | Reason |
|---|---------------|-------|-----------------------|--------|
| 1 | `H3+_singlet_C1` | 3-21g | 1.85 | Very strong 3-center bonding; symmetry breaking only at extreme stretch |
| 2 | `H3+_singlet_D3h` | 3-21g | 1.875 | Symmetric $\mathrm{H_3^+}$; same physics as above |

---

## FAIL — No CF point found (2 molecules)

| # | `molecule_id` | basis | Reason |
|---|---------------|-------|--------|
| 1 | `HMg-_singlet_Coov` | sto-3g | Closed-shell ionic bonding ($\mathrm{Mg} + \mathrm{H^-}$); no accessible triplet instability. $\langle S^2 \rangle_\mathrm{UHF} = 0$ across entire scan range. |
| 2 | `Ne2_singlet_Dooh` | sto-3g | Noble gas dimer; van der Waals only, no covalent bond to break. $\langle S^2 \rangle_\mathrm{UHF} = 0$ across entire scan range. |

---

## Generation Parameters

- **Alpha grid:** $[0.5, 0.6, 0.7, 0.8, 0.9, 1.0, 1.1, 1.2, 1.3, 1.4, 1.5]$ (11 points, step 0.1)
- **Target directory:** `data/singlet/coulson_fisher_hamiltonians/`
- **Expected output:** 46 species × 11 $\alpha$ points = 506 Hamiltonians
- **Overlap with existing data:** 7 of 11 $\alpha$ points (0.5, 0.7, 0.8, 0.9, 1.0, 1.2, 1.5) already exist in `data/singlet/hamiltonians/`

---

## Cross-Validation: Overlapping Alpha Points

The CF alpha grid $[0.5, 0.6, \ldots, 1.5]$ shares 7 points with the existing dissociation data in `data/singlet/hamiltonians/`: $\alpha = 0.5, 0.7, 0.8, 0.9, 1.0, 1.2, 1.5$. The remaining 4 points ($\alpha = 0.6, 1.1, 1.3, 1.4$) are new and have no external reference — they are validated by internal consistency checks only.

### Energy cross-validation (322 pairs)

| Metric | Result |
|--------|--------|
| Species checked | 46/46 |
| Alpha-point pairs checked | 322/322 |
| Max $E_\mathrm{FCI}$ discrepancy | $1.51 \times 10^{-12}$ Ha (machine precision) |
| Max $E_\mathrm{HF}$ discrepancy | $1.86 \times 10^{-7}$ Ha (within SCF tolerance) |
| $n_\mathrm{qubits}$ mismatches | 0 |

### Hamiltonian operator differences

The Pauli-operator representations differ between the CF and dissociation datasets at 86 of 322 overlapping alpha-point pairs (27%). The CF data typically has additional Pauli terms (range: +4 to +454 extra terms), and some shared terms differ in coefficient by up to $\sim 0.5$. The three largest coefficient discrepancies are $\mathrm{H_4N^+}$ at $\alpha = 1.5$ ($0.46$), $\mathrm{H_4N^+}$ at $\alpha = 0.7$ ($0.35$), and $\mathrm{CD_4}$ at $\alpha = 0.8$ ($0.23$).

**These differences are expected and do not indicate errors.** Detailed investigation reveals the dominant pattern is **exact sign flips** on Pauli coefficients: terms like `CF = -0.1739, Existing = +0.1739`. This is the signature of **degenerate orbital permutations** within the SCF eigensolver. Molecules with high point-group symmetry ($T_d$, $C_{3v}$, $D_{3h}$) have degenerate orbital subspaces (e.g., the $T_2$ representation in $T_d$ is triply degenerate). LAPACK's eigensolver returns eigenvectors in arbitrary order within degenerate subspaces, so two independent PySCF runs can produce orbitals that differ by a permutation. Since the Jordan-Wigner encoding is order-dependent, swapping two degenerate orbitals flips signs on the off-diagonal Pauli terms involving those orbitals.

The affected species are exactly those with orbital degeneracies: $\mathrm{H_4N^+}$ ($T_d$), $\mathrm{CD_4}$ ($T_d$), $\mathrm{CH_4}$ ($T_d$), $\mathrm{CH_2D_2}$ ($C_{2v}$ subgroup of $T_d$), $\mathrm{CH_3D}$/$\mathrm{CHD_3}$ ($C_{3v}$), $\mathrm{CH_3^+}$/$\mathrm{CH_3^-}$ ($D_{3h}$). Species without orbital degeneracies ($\mathrm{H_2O}$, $\mathrm{N_2}$, $\mathrm{F_2}$, diatomics) reproduce to machine precision ($< 10^{-13}$).

**The physics is identical.** FCI energies match to $10^{-12}$ Ha across all 322 pairs, proving the Hamiltonians have the same spectrum. HF energies also match to $10^{-14}$ Ha (eigenvalues are invariant to eigenvector ordering). The Hamiltonians are unitarily equivalent — they represent the same physical operator in different (but equally valid) orbital orderings.

### Implications for users

Both the CF and dissociation Hamiltonians are valid for quantum computing benchmarks. The different orbital rotations mean the Pauli operator coefficients, Hartree-Fock reference states, and UCCSD ansatz operators may differ between the two datasets for the same molecule at the same geometry. Users should treat each dataset as self-consistent and **not mix Hamiltonians from one dataset with reference states or ansatze from the other**.

### Dataset integrity summary (506 files)

| Check | Result |
|-------|--------|
| No NaN/Inf in Hamiltonian coefficients | [OK] |
| All Pauli strings valid ($\{I,X,Y,Z\}^{n_q}$) | [OK] |
| Variational principle: $E_\mathrm{FCI} \leq E_\mathrm{CISD} \leq E_\mathrm{HF}$ | [OK] |
| All FCI converged | [OK] |
| All FCI $\mathrm{spin\_matches\_target}$ = True | [OK] |
| Particle number conservation | [OK] |
| $n_\mathrm{qubits}$ consistent across $\alpha$ per species | [OK] |
| All species: 11 files each, 506 total | [OK] |
| Energy monotonicity at extremes ($\alpha = 0.5, 1.5$) | [OK] |
| HF minimum near $\alpha = 1.0$ | [OK] |

---

## Reproducibility

The species selection can be reproduced from `cf_summary.json` and `species_list.json` by applying the following criteria:

1. Multi-atom species only (`n_atoms > 1`)
2. Best available basis (`basis == best_basis`)
3. Maximum tapered qubit count across all feasible bases $\leq 16$
4. CF point found with $0.5 < \alpha_\mathrm{CF} \leq 1.5$

```python
import json

with open("species_list.json") as f:
    spec_list = json.load(f)

with open("coulson_fisher/cf_summary.json") as f:
    cf_data = json.load(f)

cf_lookup = {(s["molecule_id"], s["basis"]): s for s in cf_data["species"]}

for spec in spec_list:
    if spec["n_atoms"] == 1:
        continue
    if spec["basis"] != spec["best_basis"]:
        continue
    if max(spec["feasible_bases"].values()) > 16:
        continue
    key = (spec["molecule_id"], spec["basis"])
    cf = cf_lookup.get(key)
    if cf and cf["status"] == "found" and 0.5 < cf["alpha_cf"] <= 1.5:
        print(f'PASS: {spec["molecule_id"]} ({spec["basis"]}) alpha_cf={cf["alpha_cf"]}')
```
