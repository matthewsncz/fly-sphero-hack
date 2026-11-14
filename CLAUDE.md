# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Hackathon project (Hack_NCState x Thinking Machines Hack Day, Sat Nov 14 2026, prize "Best
Use of a Fruit Fly's Brain"): a simulated fruit fly brain drives a real Sphero BOLT robot
ball in real time. A 3D-printed stick figure rides on a collar around the ball, so the ball
is the fly's "legs" and the connectome is its "brain".

This file holds what is stable: rules, architecture, workflows, hard constraints. Status
(what's built, today's plan, open problems) lives in README.md.

## Rules

Keep the app organized so a beginner can read it.
- **Screens go in `pages/`.** Anything that opens a window is a page.
- **Shared pieces go in `components/`.** Anything used by more than one page or script.
- **Saving and loading goes in `data/`.** Reading the connectome, writing STL files, logs.
- **One job per file.** The file name says the job (`motor_map.py`, not `utils.py`).
- **No file over 300 lines** (aim for 200). Split by job before it gets there, and add the
  new file to the layout table below.
- **Before writing anything new, check if it already exists.** Look at the layout table
  and search the code. Reuse or extend it instead of making a duplicate.

## Start of every session

1. `git pull`, then read README.md (status board).
2. If the session touches code, run the tests first.
3. Before ending, update README.md and save progress with git.

Teammates share this repo. Claim a file in README.md before editing it, so two people
never change the same file at once.

## Commands (your words → what to run, from this folder)

| When you say | Run |
|---|---|
| "set up" / "rebuild the environment" | `uv sync` |
| "run the tests" / "check everything" | `uv run python -m tests.run_tests` |
| "test the <part>" (brain, motor, sim, body) | `uv run python -m tests.run_tests brain` |
| "is the ball connected" / "test the ball" | `uv run python -m scripts.test_bolt` |
| "run the demo" | `uv run python -m pages.bolt_page` |
| "run the backup" / "run the sim" | `uv run python -m pages.sim_page` |
| "make the body" / "export the print" | `uv run python -m scripts.make_body` (writes `data/prints/*.stl`) |
| "open the print" / "how long will it print" | `& "C:\Program Files\Prusa3D\PrusaSlicer\prusa-slicer.exe" data/prints/collar.stl` (pick the Prusa MINI or CORE One profile, Slice, read the time) |
| "save my progress" | `git add -A` then `git commit -m "<what changed>"` |
| "share it" / "push" | `git pull --rebase` then `git push` |
| "where are we" | read README.md; no command |

Always run from the project folder with `python -m folder.file`, so `pages/` and `scripts/`
can import `components/` and `data/`. Always `uv run`; never bare `python` or `pip`.
When a page or script is added, add its row here. If a command's file doesn't exist yet,
say so instead of improvising a different one.

## Architecture (data flow)

```
data/raw/  (connectome export from the organizers' Colab notebook)
  → data/connectome_loader.py      load_connectome() → sparse weight matrix + neuron table
  → components/neuron_groups.py    pick_groups() → sensory, left-motor, right-motor indexes
  → components/brain_model.py      Brain.step(inputs) → firing rates, ~5–10 steps a second
  → components/motor_map.py        motor_to_drive(rates) → (heading, speed)
  → Body.drive(heading, speed)     components/bolt_body.py (real ball) or sim_body.py (screen)
  ← Body.read_sensors()            collision, tilt, light → next step's inputs
components/collar_model.py → data/stl_saver.py → data/prints/*.stl → PrusaSlicer
```

- The loop runs on the laptop. Colab runs in the cloud and cannot reach the laptop's
  Bluetooth, so the notebook is only for downloading data.
- `components/body_interface.py` defines `Body`: `drive(heading, speed)`, `read_sensors()`,
  `stop()`. `BoltBody` and `SimBody` both follow it, so the brain never knows which it drives.
- `pages/bolt_page.py` and `pages/sim_page.py` run the same loop with a different `Body`.

## Layout

| Path | Job |
|---|---|
| `pages/bolt_page.py` | screen: live brain view while it drives the real ball |
| `pages/sim_page.py` | screen: 2D robot on screen (backup demo, no hardware) |
| `components/body_interface.py` | the `Body` rules every body follows |
| `components/bolt_body.py` | `BoltBody`: the only file that calls spherov2 |
| `components/sim_body.py` | `SimBody`: on-screen robot physics |
| `components/neuron_groups.py` | which neurons are sensory, left-motor, right-motor |
| `components/brain_model.py` | `Brain`: steps the simulation |
| `components/motor_map.py` | motor activity → heading and speed, with the speed cap |
| `components/collar_model.py` | collar + stick figure shape (manifold3d) |
| `data/connectome_loader.py` | read the downloaded connectome files |
| `data/stl_saver.py` | write a shape to `data/prints/` as STL |
| `data/raw/` | downloaded connectome files (read-only, not in git) |
| `data/prints/` | generated STL files (in git, so a teammate can grab them) |
| `scripts/test_bolt.py` | command-line check that the ball connects and moves |
| `scripts/make_body.py` | command-line export of the print files |
| `tests/run_tests.py` | runs every test file; plain asserts, stops at the first failure |

`scripts/` is for command-line tools that open no window. `tests/` holds one test file per part.

## Stack

| Layer | Choice | Why |
|---|---|---|
| Language | Python 3.13.18, uv-managed (`uv.lock`) | uv installs the exact same Python for every teammate |
| Robot link | spherov2 0.12.1 + bleak | only Python library with a BOLT class that runs on Windows; unofficial, BOLT "In Progress" |
| Math | numpy + scipy.sparse | whole-brain matrix fits in memory |
| Screens | pygame | simple window loop |
| 3D model | manifold3d (solids) + trimesh (STL export) | no OpenSCAD on this PC; stays inside `uv run` |
| Slicer | PrusaSlicer 2.9.7 | same slicer as the Makerspace's Prusa printers |
| Version control | git, branch `main`, public GitHub repo | push only when asked |

Keep `.python-version` pinned to `3.13.18` and `python-preference = "only-managed"` in
`pyproject.toml`; uv downloads that exact Python for every teammate. To upgrade:
`uv python install 3.13.N`, `uv python pin 3.13.N`, rerun tests.

## Hard constraints

**Always go through these helpers. Never re-implement them inline.**
- Connectome data only from `data/connectome_loader.py: load_connectome()`.
- Neuron groups only from `components/neuron_groups.py: pick_groups()`. Never hard-code
  neuron IDs anywhere else.
- Robot motion only through a `Body`. Never call spherov2 outside `components/bolt_body.py`.
- Heading and speed only from `components/motor_map.py: motor_to_drive()`, which clamps
  speed to `MAX_SPEED` (set in that file, well under the BOLT's 255).
- STL files only from `data/stl_saver.py`.
- Every page or script that drives the ball makes sure `Body.stop()` runs on exit, crash,
  or Ctrl+C.

**Do not touch**
- `data/raw/`: downloaded files are read-only. Never edit, rename, or commit them.
- `data/prints/`: generated files. Change `collar_model.py` and regenerate; never hand-edit.

**Boundaries**
- Never attach anything to the ball. The BOLT's whole shell spins and it has no internal
  magnet. The figure rides on a collar that rings the ball near the floor on 3 skids.
- Test the BOLT link before building anything that depends on it.
- The demo must work with no robot (`pages/sim_page.py`).
- Ask before downloading data or adding a package (`uv add <name>`, then rerun the tests).
