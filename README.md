# N-Body Gravitational Simulation

A self-contained Jupyter/Colab notebook (`N_body_gravitational_simulation.ipynb`) that simulates gravitating point masses with `numpy` and renders the result as animated GIFs with `matplotlib.animation`. The notebook has **5 cells**, meant to be run top to bottom in a single session.

## What's in the notebook

### Cell 1 — N-body physics engine
Defines the `NBodySystem` class, the shared engine used by both demos below.

- **Force law**: Newtonian gravity between every pair of bodies, vectorized with numpy broadcasting (no Python loops over pairs) via `accelerations()`.
- **Softening**: a `softening` length is added to the denominator of the force law so that close encounters don't produce divide-by-near-zero blow-ups.
- **Integrator**: `step(dt)` advances the system by one **leapfrog (kick-drift-kick)** step — a symplectic method, so total energy doesn't drift over long runs the way it does with plain Euler integration.
- **Diagnostics**: `total_energy()` returns kinetic + potential energy, useful for sanity-checking a run.
- **Other methods**: `run(dt, n_steps)` advances the system and returns the full position trajectory as a `(n_steps+1, N, 3)` array.

This cell must be run **before** either demo cell, since both demos construct an `NBodySystem` instance.

### Cell 2 — Solar system demo
Builds the Sun + 8 planets and animates ~1 full Neptune orbit (165 years).

- **Units**: AU (distance), years (time), solar masses (mass), with `G = 4·π²` — the standard trick that makes Kepler's third law (`P² = a³` for `M_sun = 1`) fall out automatically.
- **Initial conditions**: each planet is placed at its real semi-major axis with a starting angle, given a circular-orbit velocity `v = sqrt(G·M_sun / a)` perpendicular to its radius vector. The Sun is given a small recoil velocity so total momentum is exactly zero (keeps the system from drifting off-frame).
- **Animation**: 300 frames, each stepping the simulation forward with `dt = 0.005` yr, drawing all 9 bodies plus fading orbital trails.
- **Output**: saves `solar_system.gif` in the notebook's current working directory (a **relative path**, not an absolute one — see Notes below).

### Cell 3 — Galaxy collision demo
Builds two toy "galaxies" (a central mass + a disk of 180 lighter stars each) and sends them past each other on a gravitationally-focused flyby.

- **Units**: arbitrary "galactic units" with `G = 1`.
- **Encounter setup**: the two galactic centers are placed using a classic two-body scattering parametrization — separation `D`, impact parameter `b`, and asymptotic approach speed `v_inf` — so their mutual gravity bends the trajectory into a realistic flyby rather than a random collision course.
- **Disk construction** (`make_disk`): stars are placed at random radii/angles around each center with a circular velocity derived from the **softened** force law (matching the softening the integrator actually uses), then tilted out of the orbital plane and offset to the galaxy's position/velocity.
- **Camera**: the plotted view auto-frames itself every frame (recentering and rescaling to the current spread of all bodies), so both galaxies stay in view whether they're close together or flying apart.
- **Output**: saves `galaxy_collision.gif` in the notebook's current working directory.

### Cells 4–5 — Display the results
```python
from IPython.display import Image, display
display(Image(filename="solar_system.gif"))
```
```python
from IPython.display import Image, display
display(Image(filename="galaxy_collision.gif"))
```
These load the two GIFs written by cells 2 and 3 and render them inline in the notebook output. They must run **after** the corresponding demo cell has finished (and actually printed `Saved ...`), since that's when the file is written to disk.

## Requirements

- `numpy`
- `matplotlib` (with the `Agg` backend, set automatically inside the notebook — no GUI/display backend required)
- `Pillow` (used internally by `animation.PillowWriter` to write the `.gif` files — normally already installed alongside matplotlib)

No other files or modules are needed — everything (engine + both demos) lives inside this one notebook.

## How to run

1. Open the notebook in Jupyter, Colab, or any similar environment.
2. Run **Cell 1** first (defines `NBodySystem`).
3. Run **Cell 2** to generate `solar_system.gif`, then **Cell 4** to view it.
4. Run **Cell 3** to generate `galaxy_collision.gif`, then **Cell 5** to view it.
5. Re-running Cell 1 is only needed once per session — after that you can re-run Cells 2/3 as many times as you like (e.g. after tweaking parameters) without re-running Cell 1, since `NBodySystem` stays defined in memory.

Each `main()` call takes anywhere from several seconds to roughly a minute depending on your environment, since the animation writer renders every frame before saving the GIF.

## Notes / things to know

- **Output paths are relative**, e.g. `out_path = "solar_system.gif"`. The file is written to whatever directory the notebook kernel considers its current working directory — in Colab that's typically `/content`. If you want the GIFs somewhere specific, change `out_path` in Cells 2/3 to an absolute path that exists in your environment (create the folder first if needed).
- **Displaying the GIF doesn't happen automatically** — Colab/Jupyter won't show an animation just because it was saved to disk. Cells 4 and 5 exist specifically to load and render the saved file; make sure the matching demo cell has finished running first.
- **No `nbody_engine.py` import** — this notebook version defines `NBodySystem` directly in Cell 1 rather than importing it from a separate file, which avoids `ModuleNotFoundError: No module named 'nbody_engine'`. If you copy code between notebooks, copy the whole class definition, not just an import line.
- **Adjustable parameters** worth knowing about if you want to experiment:
  - Solar system: `total_years`, `dt`, `n_frames`, `trail_len` in Cell 2's `main()`.
  - Galaxy collision: `D`, `b`, `v_inf` in `build_colliding_galaxies()`, and `CENTRAL_MASS` / `STAR_MASS` / `SOFTENING` near the top of Cell 3, control how the flyby plays out. These were tuned so the encounter produces a visible tidal interaction without the bodies flying apart at unrealistic speeds — changing `CENTRAL_MASS` or `SOFTENING` substantially can make the encounter unstable (a very close, poorly-softened passage between two massive bodies can eject everything at high speed).
