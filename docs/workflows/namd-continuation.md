# Smaller-step NAMD continuation

Use a supported local-continuation build to investigate a short interval where
a fixed nuclear time step no longer resolves the electronic force. The method
continues the complete saved electronic and nuclear state into separate output
files. It does not repair an unconverged SCF solution or compensate energy errors.

## Prepare the source

Retain the last accepted `.npz` checkpoint and its matching `.trj` file. Their
committed trajectory prefix must agree. Do not extract a failed final point and
label it a valid restart. Start from a copy of the source calculation input and
keep its geometry definition, electronic method, SCF/response controls, state
count, hopping settings, seed, and RNG stream unchanged. The geometry definition
is part of the system identity; the checkpoint supplies the continued coordinates.

## Select a smaller time step

In the copied canonical `namd(...)` call, set both
`continuation_checkpoint="/absolute/source.npz"` and
`continuation_trajectory="/absolute/source.trj"`, choose a smaller `dt`, and
set distinct `trajectory_file` and `restart_file` outputs. Use a new job/log name.
Do not set `restart=true` for this first continuation. All source/output path
aliases and existing continuation sidecars are rejected.

For a source saved at step 7 and time 0.7 fs with `dt=0.1` fs, these trials reach
the same final time, 0.8 fs:

| New dt (fs) | nstep | Additional full NAMD steps |
| --- | --- | --- |
| 0.05 | 9 | 2 |
| 0.025 | 11 | 4 |
| 0.0125 | 15 | 8 |

Each small step performs nuclear motion, electronic propagation, and hopping.
This differs from `disc_substeps`, which subdivides nuclear motion within an
outer electronic interval. The generated restart manifest resumes the new
trajectory at its new fixed dt and restores its saved physical-time offset.

The schema also exposes the two paths through `job.settings.md(...)` in the
Python API. Retain the same source-model settings when preparing that job.

## Evaluate the result

Compare energy drift, coordinates, velocities, and electronic populations at
common physical times as dt decreases. Completion alone is insufficient: the
short-time results should approach a stable limit. Preserve actual SCF and
response convergence checks and disable numerical energy compensation when
testing integration accuracy.

The saved seed, stream, and next RNG counter are retained, but smaller steps
change the times and number of hopping decisions. An identical stochastic path
is not guaranteed. Likewise, agreement with a previous trajectory segment that
received a large kinetic-energy correction is not an acceptance criterion.

If smaller steps fail to stabilize the result, examine reference continuity,
orbital-response conditioning, and electronic-state identity. Force clipping or
response damping changes the derivative and is not a substitute for this check.
