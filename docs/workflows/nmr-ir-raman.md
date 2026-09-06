# NMR, IR, and Raman

## NMR Shielding

Request NMR shielding with the `nmr` modifier.

`.oqp`:

```text
hf/sto-3g
energy
nmr(gauge=cgo)
geom="h2o.xyz"
```

Python:

```python
from oqp.openqp import OpenQP

job = OpenQP("h2o_nmr", silent=1)
job.molecule(geometry="water", charge=0, multiplicity=1)
job.theory.hf(basis="sto-3g")
job.workflow.nmr(gauge="cgo")

mol = job.run()
```

Legacy `.inp`:

```ini
[input]
runtype=energy
method=hf
basis=sto-3g

[scf]
type=rhf
multiplicity=1

[properties]
scf_prop=nmr
nmr_gauge=cgo
```

Runnable `.oqp`:
[`examples/NMR/H2O_RHF-NMR.oqp`](https://github.com/Open-Quantum-Platform/openqp/blob/main/examples/NMR/H2O_RHF-NMR.oqp).
The same-stem `.inp` file is retained for legacy use.

`nmr_gauge` accepts:

| Value | Meaning |
| --- | --- |
| `cgo` | Common-gauge-origin shielding. |
| `giao` | Gauge-including atomic orbital shielding where supported. |

`job.workflow.nmr(...)` requires an HF/DFT reference-SCF theory. CGO NMR is
limited to closed-shell RHF; use `gauge="giao"` for open-shell UHF/ROHF
references. The helper also blocks range-separated and meta-GGA functionals for
NMR because those paths are not implemented.

### TDA excited-state shielding

The legacy input supports a singlet TDA excited state of a closed-shell RHF or
RKS reference. For example, use these sections with a molecular geometry and
basis under `[input]`:

```ini
[input]
method=tdhf
functional=pbe0

[scf]
type=rhf
multiplicity=1

[tdhf]
type=tda
nstate=3

[properties]
scf_prop=nmr
nmr_gauge=giao
nmr_state=1
```

The log prints all nine diamagnetic and paramagnetic tensor components for
each nucleus in ppm. Add the two tensors to obtain total shielding. GIAO
includes the London overlap, integral, and XC response terms; its total tensor
is invariant under a rigid translation of the molecule. CGO is also supported;
choose `nmr_gauge=cgo` and optionally `nmr_origin=atom:1` to place the common
gauge origin at the first nucleus.

LDA/GGA global hybrids are supported, including PBE0 and BHHLYP. Full TDDFT
without TDA, MRSF, range-separated exchange, meta-GGA, and ECP magnetic
derivatives are not included in this excited-state implementation. See
[`nmr_state`](../keywords/properties.md#nmr_state) for the input requirements.

## IR and Raman

IR and Raman intensities are produced from supported Hessian/frequency
workflows. See [Hessian and Frequencies](hessian.md) for the main Hessian
workflow page.

`.oqp`:

```text
dft/bhhlyp/6-31g*
hess(S0,type=analytical)
ir
raman
geom="h2o.xyz"
```

Python:

```python
from oqp.openqp import OpenQP

job = OpenQP("h2o_freq", silent=1)
job.molecule(geometry="water", charge=0, multiplicity=1)
job.theory.dft(functional="bhhlyp", basis="6-31g*")
job.workflow.hessian(type="analytical", state=0)

mol = job.run()
```

Legacy `.inp`:

```ini
[input]
runtype=hess
method=hf
functional=bhhlyp
basis=6-31g*

[hess]
type=analytical
state=0
```

Runnable `.oqp` inputs:

- [`examples/HESS/H2O_RHF-DFT_ANA_HESS.oqp`](https://github.com/Open-Quantum-Platform/openqp/blob/main/examples/HESS/H2O_RHF-DFT_ANA_HESS.oqp)
- [`examples/HESS/H2O_RHF-DFT_NUM_HESS.oqp`](https://github.com/Open-Quantum-Platform/openqp/blob/main/examples/HESS/H2O_RHF-DFT_NUM_HESS.oqp)

Each has a same-stem legacy `.inp` companion.
