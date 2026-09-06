# `[properties]`

The `[properties]` section requests properties and selects state indices for
gradient-like workflows.

## Keywords

### `scf_prop`

| Field | Value |
| --- | --- |
| Type | comma-separated string list |
| Default | empty (no SCF properties) |
| Values | `el_mom`, `mulliken`, `lowdin`, `resp`, `nmr` |
| Used by | SCF property driver |

Requests SCF properties. Properties are **opt-in**: a property is computed only
when listed here, and only requested properties are written to the JSON
reference and regression-tested. Available analyses:

- `el_mom` — electric dipole (and moments); the dipole (a.u.) is exposed as
  `dipole` in `get_results()`.
- `mulliken` / `lowdin` — Mulliken / Löwdin atomic partial charges, exposed as
  `mulliken_charges` / `lowdin_charges`.
- `resp` — RESP/ESP-fitted atomic charges, exposed as `resp_charges`.
- `nmr` — isotropic NMR shielding (`nmr_shielding`).

```ini
[properties]
scf_prop=el_mom,mulliken,lowdin,resp
```

!!! note "Behavior change"
    `scf_prop` previously defaulted to `el_mom,mulliken`. It now defaults to
    empty so population/moment analysis runs only on request and properties are
    not added to every reference. Request them explicitly to restore the old
    output.

### `nmr_gauge`

| Field | Value |
| --- | --- |
| Type | string |
| Default | `cgo` |
| Values | `cgo`, `giao` |
| Used by | NMR shielding |

Selects the NMR gauge formulation. `cgo` is the common-gauge-origin formulation
and is limited to closed-shell RHF in the input checker. Use `giao` for
open-shell UHF/ROHF NMR shielding where supported.

### `nmr_origin`

| Field | Value |
| --- | --- |
| Type | string |
| Default | `com` |
| Values | `com`, `atom:<i>` |
| Used by | Ground-state and TDA excited-state CGO NMR shielding |

Selects the common gauge origin: `com` uses the centre of mass, and `atom:<i>`
uses nucleus `i` in the input geometry, counted from 1. For example, `atom:1`
selects the first nucleus. The origin is printed in Bohr. Nuclear-origin
selection requires `nmr_gauge=cgo`; an explicit `atom:<i>` with GIAO is rejected.
At finite basis size, CGO shieldings depend on this choice, so comparisons must
use the same origin and geometry.

The Python API exposes the keyword through
`job.settings.properties(nmr_origin="atom:1")`.

### `nmr_state`

| Field | Value |
| --- | --- |
| Type | integer |
| Default | `0` |
| Values | `0` for the SCF reference; positive TDA state index |
| Used by | NMR shielding |

A positive value selects a singlet TDA excited state with `method=tdhf`,
`[tdhf] type=tda`, a closed-shell RHF reference, and `nmr_gauge=cgo`.
The current implementation uses HF exchange; hybrid TDDFT and excited-state
GIAO extensions are under development. This option does not currently support
full TDDFT without the Tamm–Dancoff approximation. The excited-state tensors
are printed in ppm, with magnetic-field components as rows and nuclear magnetic
moment components as columns, separately for diamagnetic and paramagnetic terms.

```ini
[input]
method=tdhf

[scf]
type=rhf
multiplicity=1

[tdhf]
type=tda
nstate=3

[properties]
scf_prop=nmr
nmr_gauge=cgo
nmr_origin=atom:1
nmr_state=1
```

### `td_prop`

| Field | Value |
| --- | --- |
| Type | boolean |
| Default | `False` |
| Used by | response-state property placeholder |

Requests TD properties. The current input checker warns that this is not part of
the documented production surface.

### `grad`

| Field | Value |
| --- | --- |
| Type | integer list |
| Default | `0` |
| Used by | `runtype=grad` and state-specific gradients |

Selects gradient states. HF/DFT gradients use state `0`. Ordinary TDHF/TDDFT
state `1` is the first excited state. SF-TDDFT and MRSF-TDDFT state `1` is the
lowest spin-flip/MRSF target state, which can be the multiconfigurational ground
state. In Python, prefer `job.workflow.gradient(state=...)`; it maps to this
input-file keyword.

Example:

```ini
[input]
runtype=grad
method=tdhf

[properties]
grad=3
```

### `nac`

| Field | Value |
| --- | --- |
| Type | string |
| Default | empty |
| Used by | legacy NAC/property requests |

Legacy property-level NAC request. The current NAC and NACME workflows should
prefer the `[nac] states` keyword.

### `export`

| Field | Value |
| --- | --- |
| Type | boolean |
| Default | `False` |
| Used by | data export |

Requests export of selected computed data where supported.

### `title`

| Field | Value |
| --- | --- |
| Type | string |
| Default | empty |
| Used by | output labeling |

Optional output title.

### `back_door`

| Field | Value |
| --- | --- |
| Type | boolean |
| Default | `False` |
| Used by | developer/debug paths |

Developer/debug option. Leave false for production calculations.
