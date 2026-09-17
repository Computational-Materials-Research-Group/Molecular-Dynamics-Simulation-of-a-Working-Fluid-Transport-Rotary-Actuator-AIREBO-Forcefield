# Molecular Dynamics Simulation of a Working-Fluid Transport Rotary Actuator — AIREBO Forcefield

<p align="center">
  <img src="https://img.shields.io/badge/LAMMPS-MD%20Simulation-blue?style=for-the-badge&logo=gnu&logoColor=white"/>
  <img src="https://img.shields.io/badge/Process-Rotary%20Actuator%20(Nanoratchet)-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/AIREBO-CH.airebo%20(reactive%20C--H)-purple?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Structure-CNT%20Shaft%20%2B%20Graphene%2FFullerene%20Stack-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Property-Axial%20to%20Rotary%20Motion-yellow?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/OVITO-Visualization-green?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Python-Geometry%20Inspection-blue?style=for-the-badge&logo=python&logoColor=white"/>
</p>

<p align="center">
  A fully atomistic <b>molecular dynamics model of a nanoscale, axially-driven rotary
  actuator</b> -- two nominally similar, spatially separated carbon-nanotube "shaft"
  assemblies, each capped at one end by a multi-layer graphene/fullerene-derived sheet
  stack (the mobile <b>wheel</b>), threaded through fixed bearing collars
  (<code>anchor</code>), and driven by a rigid pusher collar (<code>negx</code>)
  translated at constant velocity along the shaft axis. The entire structure -- shaft,
  wheel stack, bearing collars, and pusher -- is a single homogeneous C/H hydrocarbon
  system, modeled throughout with the reactive <b>AIREBO</b> potential (no hybrid pair
  style needed). A two-stage workflow -- minimize-and-thermalize, then driven
  production -- brings the structure to 90 K before the pusher begins its 250 ps,
  250 &Aring; traverse along the shaft.
</p>

<p align="center">
  <b>The structural interpretation below (wheel = graphene/fullerene sheet stack,
  collars = fixed bearings, negx = driven piston) was reconstructed entirely from the
  atom coordinates and group definitions in the supplied files -- there is no bond
  topology or build script to check it against.</b> Read
  <a href="#known-open-issues">Known Open Issues</a> before treating any of it as
  confirmed design intent.
</p>

<img width="1600" height="1200" alt="simn1" src="https://github.com/user-attachments/assets/c4c6cd25-5683-493b-9d6e-9776620e378a" />


---

## Known Open Issues

**1. The physical identity of each part (shaft, wheel/stack, bearing collars, pusher)
was inferred purely from atom coordinates, not from any accompanying build script or
design notes.** `c60_ratchetOP.data` contains no `Bonds`, `Angles`, `Dihedrals`, or
`Impropers` section (confirmed by direct inspection of the file -- expected, since
AIREBO is a reactive bond-order potential and needs no predefined topology), so there
is no independent topology to check the geometric interpretation against. The
interpretation used here -- a multi-layer graphene/fullerene sheet stack fused to a
carbon-nanotube shaft, threaded through fixed bearing collars, driven by a rigid
pusher collar -- is geometrically self-consistent (confirmed by directly plotting the
atom positions in three orthographic projections plus one end-on view down the shaft
axis) but has **not** been checked against any original CAD or build source.

**2. The two apparently similar device assemblies in the box are not cleanly separable
by a single geometric threshold.** A simple `y > -20` split used to isolate one
assembly from the other returns very different atom counts (28,567 vs. 32,645) --
most likely because a floppy stack/sheet edge from one assembly extends across that
boundary, not because the two assemblies genuinely differ in size. Per-assembly
counts and collar locations quoted below should be read as **approximate**, not
authoritative, until confirmed with a connectivity- or distance-based clustering
instead of a flat y-cutoff.

**3. No completed production run has been reviewed for this README.** Everything
below is derived from (a) the initial, pre-equilibration geometry in
`c60_ratchetOP.data`, and (b) a single partial thermo log spanning steps
~51,700-52,700 (~5% of the intended 1,000,000-step production run), at which point
the driven pusher collar had not yet reached any fixed collar or the wheel --
consistent with that log showing an essentially flat potential energy over the
window. The step numbers at which contact is projected to occur (see
[What to Expect](#what-to-expect)) are a **kinematic projection** from the pusher's
fixed driving velocity, not an observed event.

**4. Per-atom energy/stress decomposition for AIREBO is an approximation.** Like any
many-body, bond-order potential, AIREBO does not have a physically unique way to
divide bonding energy/virial among individual atoms. `compute pe/atom` and
`compute stress/atom` (added to both input scripts -- see Development Notes) are fine
as a qualitative guide to where energy/stress concentrates, but should not be treated
as rigorous per-atom thermodynamic quantities.

**5. The originally supplied `c60_ratchet.input` had a missing `write_restart`
filename**, which would have caused the script to error out at the very end of the
(potentially very long) production run, after all 1,000,000 steps had already been
computed. Fixed -- see [Common Errors and Fixes](#common-errors-and-fixes).

## Development Notes

- **Confirmed there is no bond topology in the supplied data file** before relying on
  atom-position geometry alone to interpret the structure: grep-checked
  `c60_ratchetOP.data` for `Bonds`/`Angles`/`Dihedrals`/`Impropers` section headers --
  none present, only `Masses` and `Atoms #molecular` (see Known Open Issues #1).
- **The device layout was established by directly parsing and plotting the atom
  coordinates**, not by trusting the group/fix names in the input scripts at face
  value. Three orthographic projections (XY/XZ/YZ) of the whole system, a zoomed view
  of just the wheel/stack end, and an end-on view looking straight down the shaft axis
  were generated from `c60_ratchetOP.data`. The end-on view is what actually confirmed
  the fixed bearing collar as a ring of `anchor` atoms wrapped around the shaft's
  circular cross-section, and revealed that the "wheel" is a stack of roughly 15-20
  parallel flat sheets rather than a single hexagonal nut.
- **The step at which the driven pusher will reach each fixed collar was computed
  kinematically**, using the pusher's actual starting x-range from the data file
  (x &asymp; 220.4-227.4 &Aring; for one assembly), its constant driving velocity
  (`fix move_fix negx move linear -1.0 0.0 0.0`, i.e. 1.0 &Aring;/ps), and the
  timestep (0.00025 ps/step) -- also correctly accounting for the fact that the step
  counter carries over from Stage 1 (4000 steps) into Stage 2, since
  `c60_ratchet.input` never calls `reset_timestep`.
- **A real script bug was found at the very last line of `c60_ratchet.input`** --
  `write_restart` was supplied with no filename argument. Fixed by giving it an
  explicit output name (`c60ratchetOP.final.restart`).
- **Neither original input script gave OVITO anything to color by besides atom
  type/position.** Stage 1 used a fixed-format `atom` style dump; Stage 2 used
  `custom id type x y z` with no energy or stress fields. Added
  `compute peratom all pe/atom` and `compute stratom all stress/atom NULL` to both
  stages, and switched both dumps to `custom` style carrying
  `id type x y z c_peratom c_stratom[1..6]`, so OVITO can color by per-atom potential
  energy or by any stress component directly.

## Design Decisions and Why

| # | Decision | Why |
|---|----------|-----|
| 1 | Single homogeneous AIREBO potential (`pair_style airebo 3.0 1 0`, `CH.airebo`) for the entire system -- shaft, wheel/stack, bearing collars, and pusher all use the same C/H reactive potential | Every part of the device is carbon/hydrogen; a reactive bond-order potential lets bond breaking/formation at a collar/shaft or stack contact happen physically, with no separate contact-potential fit needed |
| 2 | Two-stage workflow -- minimize + thermalize to 90 K (`Bring_to_TEMP.input`), then driven production (`c60_ratchet.input`) via a `read_restart` handoff | Isolates the thermalization transient from the driven actuation, so the pusher's constant-velocity sweep starts from a properly equilibrated 90 K structure rather than the raw, unrelaxed input geometry |
| 3 | `anchor` and `negx` both fully frozen (`setforce 0.0 0.0 0.0`, velocity set to 0) during Stage 1 | Guarantees the fixed collars and the pusher start Stage 2 at exactly their original data-file positions, so the kinematic contact-step projections (see Development Notes) start from a known, unperturbed geometry |
| 4 | Pusher driven with `fix move linear` (constant velocity, fully kinematic) rather than a spring-based steered-MD approach | Prescribes an exact, closed-form position-vs-time relationship for the pusher -- what makes it possible to compute which step it reaches each fixed collar, rather than depending on a runtime spring force |
| 5 | Boundary switched from `s s s` (Stage 1, shrink-wrapped) to `p p p` (Stage 2, periodic) at the `read_restart` handoff | Box is large (288 x 174 x 85 &Aring;) relative to the AIREBO cutoff, so periodic imaging is unlikely to introduce spurious self-interaction -- flagged here for confirmation that this is intentional rather than left unstated |
| 6 | Two spatially separated, similar device assemblies included in the same box | Most likely intended to double sampling/statistics per production run without a second full run -- not confirmed against original design intent (see Known Open Issues #2) |
| 7 | `compute pe/atom` + `compute stress/atom`, `dump style custom` added to both stages | Neither originally supplied script gave OVITO any field to color by besides type/position; per-atom PE and stress let contact/wear events be visualized directly once the pusher reaches a collar |
| 8 | Explicit `write_restart c60ratchetOP.final.restart` filename added to Stage 2 | The originally supplied script had a bare `write_restart` with no filename, which errors out -- see Common Errors and Fixes |

## Simulation Overview

| Property | Value |
|----------|-------|
| Material | C, H (single hydrocarbon AIREBO system throughout -- shaft, wheel/stack, collars, pusher) |
| Forcefield | AIREBO (Adaptive Intermolecular Reactive Empirical Bond Order), `CH.airebo`, LJ on / torsion off (`pair_style airebo 3.0 1 0`) |
| Source structure | `c60_ratchetOP.data` -- no bond topology present (consistent with a reactive potential, see Known Open Issues #1) |
| Total atom count | 61,212 |
| Atom types | 1 = C (free: wheel/stack + shaft), 2 = H (free), 3 = C (anchor), 4 = H (anchor), 5 = C (negx pusher) |
| Atoms per type | 1: 45,439   2: 12,309   3: 2,000   4: 1,343   5: 121 |
| Device assemblies | 2, spatially separated by ~110 &Aring; in y (approximate split -- see Known Open Issues #2) |
| Simulation box | x [-43, 245], y [-117, 57], z [-56, 29] &Aring; (288 x 174 x 85 &Aring;) |
| Boundary conditions | Stage 1: `s s s` (shrink-wrapped); Stage 2: `p p p` (periodic) |
| Wheel/stack extent (per assembly, approx.) | x &asymp; -33 to 25 &Aring; (~15-20 stacked flat sheets, radial extent ~38-41 &Aring;) |
| Shaft (CNT) extent (per assembly, approx.) | x &asymp; 25 to 233 &Aring;, radius &asymp; 6.8 &Aring; |
| Fixed bearing (`anchor`) collar locations (approx., one assembly) | x &asymp; [-18.7, 24.8], [134.2, 135.5], [228.9, 232.3] &Aring; |
| Pusher (`negx`) starting position (approx., one assembly) | x &asymp; 220.4-227.4 &Aring; |
| Pusher driving velocity | -1.0 &Aring;/ps, axial (`fix move linear`) |
| Timestep | 0.00025 ps |
| Stage 1 -- minimize + thermalize | `minimize 1e-4 1e-6 1000 10000`, then 4,000 steps (1 ps) to 90 K, Berendsen thermostat on `free` |
| Stage 2 -- driven production | 1,000,000 steps (250 ps); pusher travels 250 &Aring; total (~ full shaft length) |

## System Geometry

```
x (shaft axis) ---------------------------------------------------------->

  WHEEL / STACK END                        SHAFT (CNT)                PUSHER START
  ~15-20 fused graphene/                   radius ~6.8 A                  |
  fullerene-derived sheets,                   |                           |
  radial extent ~38-41 A                      |                           |
      ((()))                                  |                           (||)
 <====((()))=================================||===========================(||)====>
      ((()))         ^                        |            ^              (||)
                      |                        |            |
                anchor collar             anchor collar   anchor collar (adjacent
               (root, fused at            (mid-shaft,      to pusher start,
                wheel/shaft junction,      x=134.2..135.5) x=228.9..232.3)
                x=-18.7..24.8)

  x = -33 ............... 25                                        220.4 .. 233 (approx.)

                                     <-- negx pusher driven at -1.0 A/ps -->

 (the above repeats once more, offset ~110 A in y, as a second, approximately
  similar device assembly -- see Known Open Issues #2)
```

## Simulation Phases

```
Initial Structure  (c60_ratchetOP.data: two hydrocarbon nanotube-shaft
  assemblies, no bond topology, AIREBO reactive potential throughout)
      |
      v
Group Definition  (free = types 1,2 -- wheel/stack + shaft;
  anchor = types 3,4 -- fixed bearing collars; negx = type 5 -- pusher)
      |
      v
Stage 1 -- Minimize + Thermalize  (Bring_to_TEMP.input: minimize energy;
  anchor & negx frozen; free thermostatted to 90 K via Berendsen, 4000 steps;
  per-atom PE/stress computed and dumped every 40 steps for OVITO;
  writes TEMP_c60ratchetOP.init.restart)
      |
      v
Stage 2 -- Driven Production  (c60_ratchet.input: read_restart; boundary
  switched to periodic; anchor stays frozen; free stays thermostatted to
  90 K; negx driven at constant -1.0 A/ps for 1,000,000 steps (250 ps);
  per-atom PE/stress dumped every 5000 steps; writes
  c60ratchetOP.final.restart)
      |
      v
Final Structure  ->  c60ratchetOP.final.restart + c60ratchetOP.lammpstrj
  (see What to Expect for the projected step at which the pusher reaches
  each fixed collar)
```

## Repository Structure

```
c60_ratchet_project/
|
├── Bring_to_TEMP.input       # Stage 1: minimize, thermalize to 90 K, write restart
|                              #   (adds per-atom PE/stress computes for OVITO)
├── c60_ratchet.input          # Stage 2: driven production run, 1,000,000 steps
|                              #   (adds per-atom PE/stress; fixes missing
|                              #   write_restart filename from the original script)
├── c60_ratchetOP.data          # Initial structure (61,212 atoms, no bond topology)
├── CH.airebo                    # AIREBO potential parameter file (Stuart/Brenner C-H)
├── README.md                     # This file
|
├── analysis/                      # Geometry renders produced while inspecting the
|   |                              #   structure (see Development Notes)
|   ├── geometry_overview.png          # Full system, 3 orthographic views, by group
|   ├── wheel_zoom.png                 # Zoom on the wheel/stack end of one shaft
|   └── wheel_endon.png                # End-on view down the shaft axis (bearing collar)
|
└── output/                        # Generated on run
    ├── bring_to_temp_c60rotor.lammpstrj   # Stage 1 trajectory, every 40 steps
    ├── c60ratchetOP.lammpstrj               # Stage 2 trajectory, every 5000 steps
    ├── TEMP_c60ratchetOP.init.restart       # Stage 1 -> Stage 2 handoff restart
    └── c60ratchetOP.final.restart            # Final restart after production run
```

## Requirements

- LAMMPS -- standard build with the MANYBODY package enabled (needed for
  `pair_style airebo`): https://www.lammps.org
- OVITO for visualization: https://www.ovito.org
- (Optional, only to reproduce the renders in `analysis/`) Python 3 with `pandas`,
  `numpy`, `matplotlib`

## Installation

```bash
# LAMMPS via conda-forge (make sure the MANYBODY package is included)
conda install -c conda-forge lammps

# Optional, only needed to reproduce the analysis/ renders
pip install pandas numpy matplotlib
```

## Running the Simulation

```bash
# 1. Bring the structure to 90 K
lmp -in Bring_to_TEMP.input -log log.bring2temp

# 2. Run the driven production stage (needs TEMP_c60ratchetOP.init.restart from step 1)
lmp -in c60_ratchet.input -log log.ratchet

# or, for speed:
mpirun -np 16 lmp -in Bring_to_TEMP.input -log log.bring2temp
mpirun -np 16 lmp -in c60_ratchet.input -log log.ratchet

# 3. Visualize c60ratchetOP.lammpstrj (and/or bring_to_temp_c60rotor.lammpstrj) in OVITO
```

Keep `c60_ratchetOP.data` and `CH.airebo` in the same directory as the two `.input`
scripts -- `TEMP_c60ratchetOP.init.restart` is produced automatically between the two
runs.

## Simulation Parameters

### Geometry (from `c60_ratchetOP.data`, approximate, one assembly)

| Variable | Meaning | Value |
|----------|---------|-------|
| Box | Simulation cell | x[-43,245], y[-117,57], z[-56,29] &Aring; |
| Wheel/stack x-extent | Multi-layer sheet stack at shaft end | ~ -33 to 25 &Aring; |
| Shaft radius | CNT cross-section, measured mid-span | ~ 6.8 &Aring; |
| Shaft x-extent | CNT length beyond the stack | ~ 25 to 233 &Aring; |
| Anchor collar 1 (root) | Fixed bearing at wheel/shaft junction | x ~ -18.7 to 24.8 &Aring; |
| Anchor collar 2 (mid) | Fixed bearing mid-shaft | x ~ 134.2 to 135.5 &Aring; |
| Anchor collar 3 (end) | Fixed bearing near pusher start | x ~ 228.9 to 232.3 &Aring; |
| Pusher (`negx`) start | Driven collar starting position | x ~ 220.4 to 227.4 &Aring; |

### Motion / Thermostat

| Variable | Meaning | Value |
|----------|---------|-------|
| Timestep | Integration step | 0.00025 ps |
| Target temperature | Berendsen thermostat on `free` | 90 K |
| Thermostat damping | Berendsen relaxation time | 0.25 ps |
| Stage 1 length | Minimize + MD to 90 K | &le;1000 minimizer iterations + 4000 MD steps (1 ps) |
| Stage 2 length | Driven production | 1,000,000 steps (250 ps) |
| Pusher velocity | `fix move linear -1.0 0.0 0.0` | -1.0 &Aring;/ps, axial |
| Total pusher travel | Velocity x Stage 2 duration | 250 &Aring; |
| Stage 1 dump interval | `bring_to_temp_c60rotor.lammpstrj` | every 40 steps |
| Stage 2 dump interval | `c60ratchetOP.lammpstrj` | every 5000 steps |

### Potential

| Parameter | Meaning | Value |
|-----------|---------|-------|
| Pair style | `airebo` | `pair_style airebo 3.0 1 0` (cutoff 3.0 &Aring;, LJ on, torsion off) |
| Pair coeff | Type -> element mapping | `pair_coeff * * CH.airebo C H C H C` (types 1-5 -> C,H,C,H,C) |
| Per-atom diagnostics | Added for OVITO (not in the original scripts) | `compute pe/atom`, `compute stress/atom NULL` |

## Visualization in OVITO

1. **File -> Load File** -- open `c60_ratchetOP.data` to inspect the starting geometry
   alone, or load `c60ratchetOP.lammpstrj` / `bring_to_temp_c60rotor.lammpstrj` as a
   trajectory.
2. **Color by "Particle Type" first**, before trusting any structural label in this
   README -- confirm the wheel/shaft/collar/pusher layout visually for yourself (see
   Known Open Issues #1).
3. **Add a "Create Bonds" modifier** (cutoff ~1.85-1.9 &Aring;) -- the data file has no
   bond topology, so OVITO shows bare, disconnected atoms until bonds are generated
   for rendering.
4. **Color by `c_peratom` or any `c_stratom[i]` component** (added to both dumps --
   see Development Notes) to see where potential energy/stress concentrates once the
   pusher makes contact with a collar.
5. **Use Displacement vectors** (referencing frame 0) on the `free` group to quantify
   any rotation/translation of the wheel/stack end relative to the fixed `anchor`
   collars.
6. **Watch specifically around the projected contact steps** (see What to Expect) --
   that is where the interesting mechanical events should appear.

## What to Expect

**Stage 1 (0-4000 steps):** minimization relaxes the raw geometry, then the `free`
group's temperature ramps toward 90 K; `anchor` and `negx` stay completely still
(frozen).

**Stage 2, early (up to ~step 344,000 absolute, i.e. ~340,000 production steps):** the
pusher slides through empty space along the shaft -- no contact yet. Potential energy
in the thermo log should stay essentially flat, matching what was observed through
step ~52,700 in the partial log reviewed for this README.

**Stage 2, mid-run (~step 344,000 absolute):** the pusher's leading edge is projected
to reach the mid-shaft anchor collar (x &asymp; 134-135 &Aring;) -- watch for a jump in
`PotEng` / `c_peratom` / `c_stratom` right around here.

**Stage 2, late (~step 787,000 absolute):** the pusher is projected to reach the
wheel/stack end and its root anchor collar -- the most mechanically significant part
of the run, where the stack should actually respond to the driven collar.

**Both of the above are kinematic projections** (see Development Notes), not observed
events -- confirm them once the full 1,000,000-step production log is available.

## Common Errors and Fixes

| Error / Symptom | Cause | Fix |
|------------------|-------|-----|
| Script errors on the very last line, after (potentially) all 1,000,000 steps have already run | `write_restart` in the original `c60_ratchet.input` had no filename argument | Added an explicit filename: `write_restart c60ratchetOP.final.restart` |
| OVITO shows bare, disconnected atoms with no bonds | `c60_ratchetOP.data` has no `Bonds` section (expected -- AIREBO is reactive and needs none) | Add a "Create Bonds" modifier in OVITO (cutoff ~1.85-1.9 &Aring;) for rendering only |
| Can't color by energy or stress in OVITO | Original dumps only wrote `id type x y z` (or a fixed `atom`-style dump) | Added `compute pe/atom`, `compute stress/atom NULL`, switched both dumps to `custom` style carrying those fields |
| A simple `y` threshold gives very different atom counts for the two "similar" assemblies | A floppy stack/sheet edge from one assembly crosses the flat cutoff used to split them | Treat per-assembly counts as approximate; use a connectivity- or distance-based clustering instead of a flat cutoff if exact per-assembly numbers are needed |
| Nothing visibly happens for the first several hundred thousand steps | The driven pusher starts ~85-195 &Aring; away from the nearest fixed collar/wheel; at 1.0 &Aring;/ps that takes tens to hundreds of ps (hundreds of thousands of steps at dt=0.00025 ps) | Expected -- see What to Expect for projected contact steps, or increase the pusher's driving velocity if the early empty-travel period isn't of interest |

## Extending the Simulation

| Extension | What to Change |
|-----------|-----------------|
| Confirm the geometric part mapping | Re-derive the wheel/stack, shaft, collar, and pusher boundaries from an independent source (original build script or CAD, if available) rather than from atom-position inspection alone -- see Known Open Issues #1 |
| Get a clean per-assembly split | Replace the flat `y > -20` threshold with a connectivity-based (graph) or distance-based clustering of the two assemblies -- see Known Open Issues #2 |
| Confirm the projected contact steps | Re-run through the projected steps (~344,000 and ~787,000) and check the thermo log / dumps directly, rather than relying on the kinematic projection in What to Expect |
| Speed up time-to-contact | Increase the magnitude of `fix move_fix negx move linear` (currently -1.0 &Aring;/ps), or reposition the pusher's starting location closer to a collar in `c60_ratchetOP.data` |
| Rigorous per-atom energy/stress | Replace the qualitative `pe/atom`/`stress/atom` approximation with a more careful many-body decomposition or per-atom volume (e.g. Voronoi) if quantitative per-atom thermodynamics are needed -- see Known Open Issues #4 |
| Other driven-actuation geometries | Same two-stage (equilibrate -> `fix move`-driven production) framework applies to other axially- or radially-driven nanomechanical devices -- only the geometry and driving fix need to change |

## Citation

If you use this simulation pipeline in your research, please cite:

```bibtex
@software{moore_mishra_rotary_actuator_airebo,
  author    = {Moore, T. and Mishra, Akshansh},
  title     = {Simulation of a Working-Fluid Transport Rotary Actuator with the AIREBO Forcefield},
  year      = {2026},
  publisher = {Zenodo},
  doi       = {10.5281/zenodo.22812110},
  url       = {https://doi.org/10.5281/zenodo.22812110}
}
```

Plain text citation:

> Moore, T., & Mishra, A. (2026). *Simulation of a Working-Fluid Transport Rotary
> Actuator with the AIREBO Forcefield* [Computer software]. Zenodo.
> https://doi.org/10.5281/zenodo.22812110

## References

- **AIREBO potential** -- S. J. Stuart, A. B. Tutein, J. A. Harrison, "A reactive
  potential for hydrocarbons with intermolecular interactions," *J. Chem. Phys.*
  **112**, 6472-6486 (2000). This is the standard reference for the AIREBO
  formulation implemented by LAMMPS's `pair_style airebo`; confirm this matches the
  specific version/parameterization of `CH.airebo` in this repository.
- **`c60_ratchetOP.data`** -- source/build script not available at write time. _Add
  the actual origin here (e.g. the CAD export or hand-built script that generated
  this structure) -- it is not included in this README because it wasn't available
  when this file was written; please don't leave a fabricated reference in its
  place._

## License

_Choose and confirm a license for this project before publishing it -- one has not
been selected on your behalf._ For reference, a common choice for shared MD pipelines
like this is a [Creative Commons Attribution-NonCommercial 4.0 International License
(CC BY-NC 4.0)](https://creativecommons.org/licenses/by-nc/4.0/), which permits
sharing and adapting for non-commercial purposes with attribution, while requiring
separate permission for commercial use.
