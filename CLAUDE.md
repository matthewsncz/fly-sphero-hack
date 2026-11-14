# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Hackathon project (Hack_NCState x Thinking Machines Hack Day, Sat Nov 14 2026): a simulated
fruit fly brain is the brain, and a Sphero BOLT robot ball is its body. People talk to it
through the laptop. Their voice becomes input to the fly's real sense neurons, the fly's
wiring decides the response, and the ball moves and lights up its 8x8 LED screen.

This file holds what is stable: mission, rules, architecture, hard constraints. Status
(what's built, today's plan, open problems) lives in README.md.

## Mission (the prize we are building for)

**"Best Use of a Fruit Fly's Brain."** The organizers' description:
- *Data-Driven Insights:* extract neural graphs, synaptic pathways and structural brain data
  using their starter Colab notebook.
- *Creative Applications:* apply the data to anything: an agent, simulation, game, or
  full project.
- *Rapid Prototyping:* move quickly from raw connectome diagrams to a working prototype.

**Mission check (run it on every new idea, and say plainly if one fails):**
1. **The random-number test.** If the fly brain were swapped for random numbers, would the
   feature look the same? If yes, the fly isn't doing anything, and the feature is off-mission.
2. **The AI never decides.** The AI may *listen* (turn speech into sense inputs) and *narrate*
   (describe what the brain did). It never picks the movement, the light, or the reply.
3. **Show the pathway.** Every reaction should be traceable to which sense neurons fired and
   which descending neurons answered. That is the "Data-Driven Insights" part.

## Rules

Keep the app organized so a beginner can read it.
- **Screens go in `pages/`.** Anything that opens a window is a page.
- **Shared pieces go in `components/`.** Anything used by more than one page or script.
- **Saving and loading goes in `data/`.** Reading the connectome, writing run logs.
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
| "test the <part>" (brain, motor, senses, screen, sim) | `uv run python -m tests.run_tests brain` |
| "is the ball connected" / "test the ball" | `uv run python -m scripts.test_bolt` |
| "test the screen" | `uv run python -m scripts.test_matrix` |
| "test the mic" | `uv run python -m scripts.test_mic` |
| "run the demo" | `uv run python -m pages.bolt_page` |
| "run the backup" / "run the sim" | `uv run python -m pages.sim_page` |
| "save my progress" | `git add -A` then `git commit -m "<what changed>"` |
| "share it" / "push" | `git pull --rebase` then `git push` |
| "where are we" | read README.md; no command |

Always run from the project folder with `python -m folder.file`, so `pages/` and `scripts/`
can import `components/` and `data/`. Always `uv run`; never bare `python` or `pip`.
When a page or script is added, add its row here. If a command's file doesn't exist yet,
say so instead of improvising a different one.

## Architecture (data flow)

```
laptop mic → components/mic_listener.py     loudness, pitch, recognized words
  → components/sense_encoder.py             → input current for sense neurons (hearing,
                                              taste, vision) — the AI's only job going in
data/raw/ → data/connectome_loader.py       load_connectome() → sparse weights + neuron table
  → components/neuron_groups.py             pick_groups() → sense and descending-neuron indexes
  → components/brain_model.py               Brain.step(inputs) → firing rates, ~5–10 steps/s
  → components/motor_map.py                 left vs right descending activity → heading,
                                              total → speed, backward neurons → reverse
  → Body.drive(heading, speed)              bolt_body.py (real ball) or sim_body.py (screen)
  → components/matrix_display.py            brain state → 8x8 LED picture or short word
  → components/narrator.py                  what fired → one plain sentence, spoken on laptop
  → data/run_logger.py                      every step saved for replay and the demo
```

- The loop runs on the laptop. Colab runs in the cloud and cannot reach the laptop's
  Bluetooth, so the notebook is only for downloading data.
- `components/body_interface.py` defines `Body`: `drive(heading, speed)`, `show(frame)`,
  `read_sensors()`, `stop()`. `BoltBody` and `SimBody` both follow it, so the brain never
  knows which one it drives. `SimBody` draws the 8x8 screen in the window too.
- Stimulate **senses**, read **descending neurons** (brain → legs). Never stimulate the
  output neurons directly; that skips the brain.

## What the BOLT can and can't do (from spherov2 0.12.1 source)

| Can | Notes |
|---|---|
| Roll any heading 0–359°, speed 0–255 | `set_heading`, `set_speed`; raw left/right motors too |
| Single pixels, colored boxes | `set_matrix_pixel`, `set_matrix_fill` |
| One character | `set_matrix_character` |
| Scrolling text | `scroll_matrix_text`: max 25 characters, 1–30 fps, about 1 letter visible at a time |
| Stored animations | `register_matrix_animation` (frames of 8x8, 16-color palette) then `play_matrix_animation` |
| Main, front and back LEDs | `set_main_led`, `set_front_led`, `set_back_led` |
| Sensors | acceleration, gyro, orientation, location, velocity, ambient light, collisions |

| Can't | So instead |
|---|---|
| Play sound or speak (no `Audio` in the BOLT class) | speak through the laptop speaker |
| Hear (no microphone) | use the laptop mic |
| Hold anything on the ball (whole shell spins) | no add-ons; the LED screen is the "face" |

Every pixel change is one Bluetooth message. For moving pictures, register an animation
once and play it, or use `set_matrix_fill` boxes; don't redraw 64 pixels a frame.

## Stack

| Layer | Choice | Why |
|---|---|---|
| Language | Python 3.13.18, uv-managed (`uv.lock`) | uv installs the exact same Python for every teammate |
| Robot link | spherov2 0.12.1 + bleak | only Python library with a BOLT class that runs on Windows; unofficial, BOLT "In Progress" |
| Math | numpy + scipy.sparse | whole-brain matrix fits in memory |
| Screens | pygame | simple window loop |
| Mic, speech-to-text, voice | not chosen yet (see README open problems) | ask before adding |
| Version control | git, branch `main`, public GitHub repo | push only when asked |

Keep `.python-version` pinned to `3.13.18` and `python-preference = "only-managed"` in
`pyproject.toml`. To upgrade: `uv python install 3.13.N`, `uv python pin 3.13.N`, rerun tests.

## Hard constraints

**Always go through these helpers. Never re-implement them inline.**
- Connectome data only from `data/connectome_loader.py: load_connectome()`.
- Neuron groups only from `components/neuron_groups.py: pick_groups()`. Never hard-code
  neuron IDs anywhere else.
- Sense input only from `components/sense_encoder.py`. Speech never reaches the brain any
  other way.
- Robot motion and the LED screen only through a `Body`. Never call spherov2 outside
  `components/bolt_body.py`.
- Heading and speed only from `components/motor_map.py: motor_to_drive()`, which clamps
  speed to `MAX_SPEED` (set in that file, well under 255).
- LED pictures only from `components/matrix_display.py`. It may only show things computed
  from brain activity (activity bars, the chosen direction, a word picked by what fired).
- Never use spherov2's `set_matrix_line`: its source computes y from x, so non-diagonal lines
  come out wrong. Use `set_matrix_fill` or pixels.
- Every page or script that drives the ball makes sure `Body.stop()` runs on exit, crash,
  or Ctrl+C.

**Do not touch**
- `data/raw/`: downloaded files are read-only. Never edit, rename, or commit them.

**Boundaries**
- Test the BOLT link before building anything that depends on it.
- The demo must work with no robot (`pages/sim_page.py`).
- Ask before downloading data or adding a package (`uv add <name>`, then rerun the tests).

## Layout

| Path | Job |
|---|---|
| `pages/bolt_page.py` | screen: live brain view, mic on, driving the real ball |
| `pages/sim_page.py` | screen: same loop with an on-screen ball and 8x8 screen (backup demo) |
| `components/body_interface.py` | the `Body` rules every body follows |
| `components/bolt_body.py` | `BoltBody`: the only file that calls spherov2 |
| `components/sim_body.py` | `SimBody`: on-screen ball and LED screen |
| `components/mic_listener.py` | laptop mic → loudness, pitch, recognized words |
| `components/sense_encoder.py` | those features → input current for sense neurons |
| `components/neuron_groups.py` | which neurons are senses and which are descending outputs |
| `components/brain_model.py` | `Brain`: steps the simulation |
| `components/motor_map.py` | descending activity → heading and speed, with the speed cap |
| `components/matrix_display.py` | brain state → 8x8 frame or short word |
| `components/narrator.py` | what fired → one plain sentence, spoken on the laptop |
| `data/connectome_loader.py` | read the downloaded connectome files |
| `data/run_logger.py` | save each run's brain activity for replay |
| `data/raw/` | downloaded connectome files (read-only, not in git) |
| `scripts/test_bolt.py` | check the ball connects and rolls |
| `scripts/test_matrix.py` | check the LED screen: pixel, box, character, scrolling word |
| `scripts/test_mic.py` | check the mic: prints loudness and recognized words |
| `tests/run_tests.py` | runs every test file; plain asserts, stops at the first failure |

`scripts/` is for command-line tools that open no window. `tests/` holds one test file per part.
