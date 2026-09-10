# `[nac]`

The `[nac]` section controls NAC and NACME state-pair workflows. NAC and NACME
currently require `[input] method=tdhf` and `[tdhf] type=mrsf`.

## Keywords

### `type`

| Field | Value |
| --- | --- |
| Type | string |
| Default | `numerical` |
| Values | `numerical`, `analytical` |
| Used by | NAC dispatch |

Selects finite-difference or analytic NAC vectors for static `nac` calculations
and branching-plane analysis (`bp`). `analytical` uses the MRSF Lagrangian
response, including the state-pair Z-vector equation. It requires:

- `[input] method=tdhf` and `[tdhf] type=mrsf`;
- an ROHF/ROKS triplet reference (`[scf] type=rohf`, `multiplicity=3`);
- singlet target states (`[tdhf] multiplicity=1`) and `nstate >= 2`;
- finite, positive `[scf] conv` and `[tdhf] conv` values no larger than `1e-8`.
  Use `1e-10` near a crossing; the default `1e-6` thresholds are too loose.

Unsupported settings fail validation; there is no automatic substitution of
numerical NAC. MRSF-TDHF is also supported by omitting the DFT functional.
Triplet targets, UMRSF, and MRSF-TDDFTB are outside this analytic implementation.
`nacme` evaluates overlap time-derivative couplings and does not accept
`type=analytical`.

```text
mrsf(nstate=2)/bhhlyp/6-31g nac(S0,S1,type=analytical)
geom="h2o.xyz"
scf(conv=1e-10,maxit=200)
tdhf(conv=1e-10,maxit=100)
```

Use `bp(S0,S1,type=analytical)` for branching-plane analysis with the same
electronic settings. A small input is provided in the OpenQP source at
`examples/other/h2o_nac_analytical_mrsf.oqp`.

The corresponding Python API setup is:

```python
from oqp.openqp import OpenQP

job = OpenQP("h2o_analytic_nac").molecule(geometry="water")
job.theory.mrsf(functional="bhhlyp", basis="6-31g", nstate=2)
job.settings.scf(conv=1e-10)
job.settings.tdhf(conv=1e-10)
job.workflow.nac(type="analytical", states="1 2")
mol = job.run()
```

In the log, **NAC Vector (h_ij)** is the gap-scaled vector
`h_ij = (E_j - E_i) d_ij`, in Hartree/bohr; **DC Vector (d_ij)** is the derivative
coupling, in inverse bohr.

### `states`

| Field | Value |
| --- | --- |
| Type | state-pair list |
| Default | `1 2` |
| Used by | NAC and NACME state pairs |

Lists pairs of response roots. Each state index must be at least `1` and no
larger than `[tdhf] nstate`. For MRSF singlets, `1 2` means physical `S0/S1`,
and `2 3` means `S1/S2`. Analytic NAC requires two distinct states.

The static analytic driver currently evaluates **all retained state pairs**;
`states` does not restrict its analytic calculation to one pair. After a
converged MRSF energy calculation with the settings above, the lower-level
`analytic_nac(mol, pair=(1, 2))` function in `oqp.library.nac_analytic` evaluates
only that pair and returns `(h, d)`. Both arrays retain the full
`[I, J, atom, xyz]` layout; uncomputed pairs are zero and mean **not evaluated**.
The `pair` argument is one-based; NumPy array indexing is zero-based.

Example:

```ini
[nac]
states=1 2,2 3
```

### `dt`

| Field | Value |
| --- | --- |
| Type | float |
| Default | `1` |
| Used by | time-derivative coupling style inputs |

Time step for workflows that use time-separated geometries.

### `dx`

| Field | Value |
| --- | --- |
| Type | float |
| Default | `0.0001` |
| Used by | finite-difference NAC |

Finite-difference displacement.

### `bp`

| Field | Value |
| --- | --- |
| Type | boolean |
| Default | `False` |
| Used by | branching-plane style workflows |

Enables branching-plane behavior where wired.

### `nproc`

| Field | Value |
| --- | --- |
| Type | integer |
| Default | `1` |
| Used by | NAC workers |

Worker count. Must be at least `1`.

### `restart`

| Field | Value |
| --- | --- |
| Type | boolean |
| Default | `False` |
| Used by | NAC restart |

Continues a NAC workflow where supported.

### `clean`

| Field | Value |
| --- | --- |
| Type | boolean |
| Default | `False` |
| Used by | temporary NAC files |

Removes temporary NAC files where supported.

### `align`

| Field | Value |
| --- | --- |
| Type | string |
| Default | `reorder` |
| Values | `reorder`, `no` |
| Used by | NACME alignment |

Controls alignment of states/geometries in the Python layer. The input checker
warns for values other than `reorder` and `no`.
