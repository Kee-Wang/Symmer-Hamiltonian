# Coulson-Fisher Point: Methodology and Reproducibility

## 1. Definition

The **Coulson-Fisher (CF) point** is the geometry scaling factor $\alpha_{\mathrm{CF}}$ at which the unrestricted Hartree-Fock (UHF) solution first diverges from the restricted Hartree-Fock (RHF) solution along a dissociation path. Below $\alpha_{\mathrm{CF}}$, the UHF wavefunction collapses to the RHF determinant (identical $\alpha$ and $\beta$ spatial orbitals). Above $\alpha_{\mathrm{CF}}$, the UHF solution spontaneously breaks spin symmetry, yielding $\langle \hat{S}^2 \rangle > 0$ and a lower energy than RHF.

Physically, the CF point marks the onset of **static (strong) correlation**: the single-determinant RHF picture becomes qualitatively inadequate, and at least two determinants are needed for a correct description. This makes $\alpha_{\mathrm{CF}}$ a useful metric for characterizing where along a dissociation curve multi-reference effects become important.

### Caveat: path dependence for polyatomics

For diatomic molecules, uniform geometry scaling is equivalent to stretching the single bond, and $\alpha_{\mathrm{CF}}$ is unambiguous. For **polyatomic** molecules, uniform scaling stretches *all* interatomic distances simultaneously. The CF point found this way is **path-dependent** and is generally *not* the same as the CF point along a single-bond dissociation coordinate (e.g., symmetric O-H stretch in $\mathrm{H_2O}$). The values reported here correspond specifically to the uniform-scaling dissociation path used throughout the Symmer-Hamiltonian dataset.

---

## 2. Geometry scaling

All geometries are derived from the equilibrium ($\alpha = 1.0$) structure stored in the Hamiltonian JSON files. Scaling is performed relative to the molecular centroid:

$$\mathbf{r}_i(\alpha) = \mathbf{r}_{\mathrm{centroid}} + \alpha \cdot (\mathbf{r}_i^{\mathrm{eq}} - \mathbf{r}_{\mathrm{centroid}})$$

where $\mathbf{r}_{\mathrm{centroid}} = \frac{1}{N_{\mathrm{atoms}}} \sum_i \mathbf{r}_i^{\mathrm{eq}}$.

This preserves the centroid position while uniformly scaling all atom-centroid distances by $\alpha$. For a diatomic, this gives bond length $R(\alpha) = \alpha \cdot R_{\mathrm{eq}}$.

Implementation: `symmerpyscf.scale_geometry()`. All coordinates are in Angstrom.

---

## 3. Alpha grid

| Parameter | Value |
|-----------|-------|
| Range | $[0.5,\; 3.0]$ |
| Step size | $0.025$ |
| Number of points | $101$ |

```python
ALPHAS_CF = np.round(np.arange(0.5, 3.0 + 1e-9, 0.025), 3)
```

The grid is uniform. Step size $0.025$ gives a CF bracket precision of $\pm 0.0125$ in $\alpha$ before interpolation. Sub-grid precision is obtained via linear interpolation (Section 6).

---

## 4. SCF computation

### 4.1 Common parameters

| Parameter | Value |
|-----------|-------|
| Convergence threshold | $10^{-9}\;\mathrm{Ha}$ (`conv_tol`) |
| Max SCF iterations | $200$ (`max_cycle`) |
| Spin | $0$ (singlet, $S_z = 0$) |
| Units | Angstrom |
| Software | PySCF 2.11.0 |

### 4.2 RHF: three-stage convergence cascade

RHF is computed with density-matrix (DM) propagation along an outward sweep from $\alpha = 1.0$. The converged DM at one $\alpha$ point seeds the initial guess for the next.

**Sweep order:**
1. Right sweep: $\alpha = 1.0 \to 1.025 \to 1.050 \to \cdots \to 3.0$
2. Left sweep: $\alpha = 0.975 \to 0.950 \to \cdots \to 0.5$ (seeded from the converged $\alpha = 1.0$ DM)

At each $\alpha$ point, a three-stage convergence cascade is applied:

| Stage | Method | Trigger | Details |
|-------|--------|---------|---------|
| 1 | Standard DIIS | Always | `scf.RHF(mol).kernel(dm0=dm_prev)` |
| 2 | Level shift | Stage 1 fails | Level shift $= 0.5\;\mathrm{E_h}$, re-run with best available DM |
| 3 | Newton second-order | Stage 2 fails | `scf.newton(mf).kernel(dm0=dm_stage2)` |

If all three stages fail, the point is marked `rhf_converged = false`.

### 4.3 UHF: inward sweep with stability analysis

UHF is computed via an **inward sweep from $\alpha = 3.0$ to $\alpha = 0.5$** (descending), propagating the broken-symmetry DM from larger $\alpha$ to smaller.

**Seeding strategy at $\alpha = 3.0$:**

- The first UHF calculation uses PySCF's default superposition-of-atomic-densities initial guess (`dm0=None`).
- After convergence, the UHF internal stability is checked. If the solution is internally unstable, the rotated MO coefficients from the stability analysis are used to reconverge UHF (up to `MAX_STABILITY_CYCLES = 5` iterations).

**Stability analysis loop** (applied at every $\alpha$ point):

```
for cycle in range(5):
    mo_internal, _, stable_internal, _ = mf.stability(return_status=True)
    if stable_internal:
        break
    dm = mf.make_rdm1(mo_internal, mf.mo_occ)
    mf.kernel(dm0=dm)
```

PySCF's `stability(return_status=True)` computes the orbital Hessian eigenvalues. For UHF, the *internal* stability check detects $\mathrm{UHF} \to \mathrm{UHF}$ instabilities — i.e., whether a lower-energy UHF solution exists with different orbital rotations. If unstable, the new MO coefficients `mo_internal` point along the descent direction.

**Collapse detection and early termination:**

A UHF solution is classified as *collapsed to RHF* when both:

$$\langle \hat{S}^2 \rangle_{\mathrm{UHF}} < 10^{-6} \quad \text{and} \quad |E_{\mathrm{UHF}} - E_{\mathrm{RHF}}| < 10^{-8}\;\mathrm{Ha}$$

When collapse is detected:

1. The collapsed DM is **not** propagated to the next point (to avoid anchoring the subsequent UHF to the RHF solution). Instead, the next point starts from a fresh default guess (`dm0 = None`).
2. After **3 consecutive** collapsed points, the sweep terminates early, and all remaining $\alpha$ points are filled with $E_{\mathrm{UHF}} = E_{\mathrm{RHF}}$, $\langle \hat{S}^2 \rangle = 0$.

**Edge case — no virtual orbitals:**

For species where the number of basis functions equals the number of occupied orbitals (e.g., $\mathrm{He_2}$/STO-3G: 2 basis functions, 4 electrons), the stability analysis is skipped because the orbital Hessian has dimension zero. In such cases, UHF trivially converges to RHF at all $\alpha$.

---

## 5. Spin expectation value

At each $\alpha$ point, the UHF spin expectation value $\langle \hat{S}^2 \rangle$ is computed via PySCF's `mf.spin_square()`. For a pure singlet ($S = 0$), $\langle \hat{S}^2 \rangle = 0$. Non-zero values indicate spin contamination from broken-symmetry mixing with higher-spin states.

$\langle \hat{S}^2 \rangle$ serves as the **primary diagnostic** for the CF point (Section 6), as it is a more direct indicator of symmetry breaking than the energy gap.

---

## 6. CF point detection algorithm

### 6.1 Divergence criteria

A grid point $\alpha_i$ is classified as *diverged* if both conditions hold simultaneously:

| Criterion | Threshold | Rationale |
|-----------|-----------|-----------|
| $\langle \hat{S}^2 \rangle_{\mathrm{UHF}} > 0.01$ | Primary | One decade above typical RHF numerical noise ($\sim 10^{-5}$) |
| $E_{\mathrm{RHF}} - E_{\mathrm{UHF}} > 10^{-6}\;\mathrm{Ha}$ | Secondary | Three orders of magnitude above `conv_tol` ($10^{-9}$) |

### 6.2 Two-consecutive-point requirement

To guard against isolated false positives from numerical noise or SCF convergence artifacts, the CF point is only declared when **two consecutive** $\alpha$ points satisfy both criteria. This eliminates single-point outliers.

### 6.3 Bracket identification

Scanning $\alpha$ from small to large, let $\alpha_i$ be the **first** grid point where both criteria hold (confirmed by $\alpha_{i+1}$ also satisfying them). Let $\alpha_j$ ($j = i - 1$) be the last grid point *below* both thresholds.

The CF bracket is $[\alpha_j,\; \alpha_i]$.

### 6.4 Linear interpolation

The CF point is estimated by linearly interpolating $\Delta E(\alpha) = E_{\mathrm{RHF}}(\alpha) - E_{\mathrm{UHF}}(\alpha)$ between the bracket endpoints:

$$\alpha_{\mathrm{CF}} = \alpha_j + \frac{0 - \Delta E_j}{\Delta E_i - \Delta E_j} \cdot (\alpha_i - \alpha_j)$$

When $\Delta E_j = 0$ (UHF $=$ RHF at $\alpha_j$), this simplifies to $\alpha_{\mathrm{CF}} = \alpha_j$, meaning the CF point coincides with the lower bracket boundary.

Spline interpolation was considered but rejected: the energy gap is not necessarily smooth near the CF transition (especially for polyatomics with multiple broken-symmetry channels), and cubic spline oscillations can produce spurious sub-grid estimates. Linear interpolation is conservative and numerically stable.

### 6.5 Classification

Each species receives a binary status:

- **`found`**: two consecutive above-threshold points identified; $\alpha_{\mathrm{CF}}$ reported with bracket.
- **`not_found`**: no such pair exists in $[0.5, 3.0]$. The RHF solution is UHF-stable across the entire scanned range.

No confidence levels are assigned. The bracket $[\alpha_j, \alpha_i]$ conveys the grid-limited precision.

### 6.6 Boundary cases

If $\alpha_i = \alpha_{\min}$ (no below-threshold reference exists), $\alpha_{\mathrm{CF}}$ is reported as $\alpha_i$ and the sanity-check system flags a `[WARN]`. This indicates the true CF point lies below $\alpha = 0.5$, outside the scanned range.

---

## 7. Sanity checks

Each species is assigned an `[OK]`, `[WARN]`, or `[FAIL]` status based on the following automated checks:

| Check | Level | Condition |
|-------|-------|-----------|
| Variational principle | `[FAIL]` | $E_{\mathrm{UHF}} > E_{\mathrm{RHF}} + 10^{-10}$ at any $\alpha$ |
| $\langle \hat{S}^2 \rangle$ bounds | `[FAIL]` | $\langle \hat{S}^2 \rangle < 0$ or $> \frac{N}{2}(\frac{N}{2} + 1)$ |
| Boundary CF | `[WARN]` | $\alpha_{\mathrm{CF}} \leq 0.525$ or $\geq 2.975$ |
| Convergence failures | `[WARN]` | $> 5$ unconverged RHF or UHF points |
| RHF cross-validation | `[WARN]` | Computed $E_{\mathrm{RHF}}$ *higher* than stored dataset value at matching $\alpha$ by $> 10^{-6}$ Ha |

The cross-validation is one-sided: a *lower* computed $E_{\mathrm{RHF}}$ is expected (finer DM propagation finds better SCF solutions at stretched geometries) and is not flagged.

---

## 8. Output data format

### Per-species JSON (`results/{molecule_id}_{basis}.json`)

| Field | Type | Description |
|-------|------|-------------|
| `alpha_grid` | `float[101]` | Alpha values |
| `e_rhf` | `(float\|null)[101]` | RHF energies (Ha); `null` if SCF failed |
| `e_uhf` | `(float\|null)[101]` | UHF energies (Ha) |
| `s2_uhf` | `(float\|null)[101]` | UHF $\langle \hat{S}^2 \rangle$ |
| `rhf_converged` | `bool[101]` | RHF convergence flags |
| `uhf_converged` | `bool[101]` | UHF convergence flags |
| `coulson_fisher.alpha_cf` | `float\|null` | Interpolated CF point |
| `coulson_fisher.alpha_bracket` | `[float, float]\|null` | Grid bracket $[\alpha_j, \alpha_i]$ |
| `coulson_fisher.status` | `str` | `"found"` or `"not_found"` |
| `timing.rhf_sweep` | `float` | RHF wall time (seconds) |
| `timing.uhf_sweep` | `float` | UHF wall time (seconds) |
| `timing.total` | `float` | Total wall time (seconds) |

### Summary JSON (`cf_summary.json`)

Aggregates `alpha_cf` and `status` for all species, with total counts.

---

## 9. Scope

| Parameter | Value |
|-----------|-------|
| Partition | Singlet ($2S+1 = 1$) |
| Species filter | Multi-atom only ($N_{\mathrm{atoms}} > 1$) |
| Basis sets | All feasible bases per molecule (matching the Hamiltonian dataset) |
| Total species | 156 (molecule, basis) pairs |

---

## 10. Reproducibility

### Software environment

| Component | Version / Path |
|-----------|---------------|
| Python | 3.x via `/opt/anaconda3/envs/symmer-pyscf/bin/python` |
| PySCF | 2.11.0 |
| symmerpyscf | Development install from `libs/1-symmer-pyscf/` |
| NumPy | (PySCF dependency) |
| Matplotlib | (plotting only) |

### Recomputation

The computation script `compute_cf.py` lives in the companion code repository (`Symmer-Hamiltonian-Code/toolchain/`). All commands below assume the working directory is the `coulson_fisher/` data directory.

To regenerate all results from scratch:

```bash
# Delete existing results
rm -rf results/ figures/png/*.png

# Full recomputation (from the code repo's toolchain)
/opt/anaconda3/envs/symmer-pyscf/bin/python -u ../../Symmer-Hamiltonian-Code/toolchain/compute_cf.py
```

To regenerate plots and gallery only (without recomputing SCF):

```bash
/opt/anaconda3/envs/symmer-pyscf/bin/python -u ../../Symmer-Hamiltonian-Code/toolchain/compute_cf.py --report-only
```

To recompute a single species:

```bash
rm results/H2_singlet_Dooh_sto-3g.json
/opt/anaconda3/envs/symmer-pyscf/bin/python -u ../../Symmer-Hamiltonian-Code/toolchain/compute_cf.py --molecule H2_singlet_Dooh --basis sto-3g
```

### Determinism

SCF is deterministic for a given initial guess. Results should be exactly reproducible on the same platform with the same PySCF version. Cross-platform variations may arise from LAPACK/BLAS differences affecting eigenvalue ordering near degeneracies, but CF points should agree to within the grid bracket.

---

## 11. Summary statistics (from the production run)

| Metric | Value |
|--------|------:|
| Species scanned | 156 |
| CF point found | 138 |
| No CF point | 18 |
| Failed | 0 |
| Mean $\alpha_{\mathrm{CF}}$ | 1.121 |
| Median $\alpha_{\mathrm{CF}}$ | 1.217 |
| Min $\alpha_{\mathrm{CF}}$ | 0.500 (boundary) |
| Max $\alpha_{\mathrm{CF}}$ | 1.875 |
| Std $\alpha_{\mathrm{CF}}$ | 0.372 |
| Total wall time | 307 s |

Species with no CF point are predominantly noble gas dimers and compounds ($\mathrm{He_2}$, $\mathrm{Ne_2}$, $\mathrm{HHe^+}$, $\mathrm{HNe^+}$, $\mathrm{ArH^+}$, $\mathrm{HeLi^+}$, $\mathrm{LiNe^+}$) and $\mathrm{HMg^-}$, all of which lack covalent bonds whose breaking would induce spin-symmetry breaking.

Species with $\alpha_{\mathrm{CF}} = 0.500$ (boundary) include $\mathrm{C_2}$, BH, $\mathrm{B_2H_2}$, and other strongly correlated species where the RHF solution is unstable even at compressed geometries. For these, the true CF point lies below the scanned range.
