# fly-sphero-hack: status board

A simulated fruit fly brain is the brain and a Sphero BOLT robot ball is its body. People
talk to it through the laptop; their voice drives the fly's real sense neurons, the fly's
wiring decides, and the ball moves and lights up its 8x8 LED screen.

Repo: https://github.com/matthewsncz/fly-sphero-hack. To join: `git clone` it, then `uv sync`.

Rules, mission, commands and architecture: [CLAUDE.md](CLAUDE.md). This file is status only.
Update it whenever something gets built, decided, or blocked.

## Event

- Hack Day x Hack_NCState & Thinking Machines, Sat Nov 14 2026, 9:00 AM–4:30 PM EST,
  Riddick Hall Room 301, 2401 Stinson Dr, Raleigh. Questions: hackncstate@ncsu.edu.
- Teams of 2–4. Food 11:45–12:15. Submissions 2:15–2:45. Demos 2:45–3:45.

## Decisions

- Nov 14: dropped the 3D-printed collar. Nothing can ride on the spinning ball, and the print
  cost a teammate two hours for decoration. The BOLT's LED screen is the "face" instead.
- Nov 14: the AI only listens (speech → sense inputs) and narrates (what fired). The fly's
  wiring picks every movement and light. See the mission check in CLAUDE.md.

## Build order (stop wherever the clock runs out)

1. **Brain moves the ball:** load the connectome, stimulate sugar-taste neurons, read the
   descending neurons, drive the sim, then the real BOLT. Without this there is no project.
2. **LED screen shows the brain:** activity bars or the chosen direction, plus a word picked
   by what fired.
3. **Laptop mic drives the senses:** loudness and pitch → hearing neurons; keywords like
   "food" → taste, "danger" → looming vision.
4. **Narrator:** one spoken sentence about which neurons fired and what the fly did.

## Today's plan

| Time | Who | What |
|---|---|---|
| 9:00–noon | everyone | build steps 1–2 in the sim; `test_bolt.py` and `test_matrix.py` ready |
| noon | NCSU-ID teammate | pick up a BOLT at Ask Us (Hunt opens noon; Hill: call 919-515-3364) |
| ~12:15 | anyone | `test_bolt.py` then `test_matrix.py` on the real ball |
| 12:30–2:00 | everyone | steps 3–4, swap the sim for the real ball |
| by 2:15 | everyone | submit; `pages/sim_page.py` is the backup demo |

## Library facts (checked Nov 14)

- BOLT kit: ball, charging base, power adapter; about 2 hours per charge; 1-week loan, no
  renewal. Pickup at Ask Us, Hunt or Hill. Count and availability not shown online.
- Borrowing needs a valid NC State Campus ID / Wolfpack One Card.
- Saturday hours: Hunt Ask Us noon–6pm, Hill building 9am–10pm (Hill Ask Us not confirmed).

## Built

- [x] CLAUDE.md, README.md, public git repo
- [x] Environment (Nov 14): Python 3.13.18; numpy, scipy, pandas, pyarrow, spherov2 0.12.1,
  bleak, pygame. All import; spherov2's BOLT class loads. (manifold3d and trimesh are
  installed from the collar plan and now unused.)
- [ ] `data/`: connectome_loader, run_logger
- [ ] `components/`: body_interface, bolt_body, sim_body, neuron_groups, brain_model,
  motor_map, matrix_display, mic_listener, sense_encoder, narrator
- [ ] `pages/`: sim_page (backup demo), bolt_page (real ball)
- [ ] `scripts/`: test_bolt, test_matrix, test_mic
- [ ] `tests/`

## Open problems

1. Whether the organizers' notebook data is FlyWire, what format it exports, and whether it
   labels sense and descending neurons.
2. Whether a connectome-only model gives a clear left/right signal. Try sugar → feeding first
   (a 2024 whole-brain model predicted that pathway); steering from vision is harder.
3. Whether spherov2 can send heading/speed fast enough over this laptop's Bluetooth.
4. Whether a team member has an NC State ID (needed to borrow the BOLT).
5. Mic and speech: which packages (live mic, speech-to-text, laptop voice). Not chosen,
   not installed. Speech-to-text either needs an internet service or a downloaded model.
6. An LLM for the narrator needs an API key; without one, the narrator uses fill-in-the-blank
   sentences.

## Who is editing what

Claim a file here before editing it; clear the line when it's pushed.

- (nobody yet)
