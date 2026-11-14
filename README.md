# fly-sphero-hack: status board

A simulated fruit fly brain drives a Sphero BOLT robot ball in real time, with a 3D-printed
stick figure riding on a collar around the ball.

Rules, commands and architecture: [CLAUDE.md](CLAUDE.md). This file is status only.
Update it whenever something gets built, decided, or blocked.

## Event

- Hack Day x Hack_NCState & Thinking Machines, Sat Nov 14 2026, 9:00 AM–4:30 PM EST,
  Riddick Hall Room 301, 2401 Stinson Dr, Raleigh. Questions: hackncstate@ncsu.edu.
- Teams of 2–4. Food 11:45–12:15. Submissions 2:15–2:45. Demos 2:45–3:45.

## Today's plan

| Time | Who | What |
|---|---|---|
| 9:00–noon | everyone | brain sim running on the laptop; BOLT test script ready; body model exported |
| before noon | NCSU-ID teammate | finish the Makerspace online orientation |
| noon | NCSU-ID teammate | pick up a BOLT at Ask Us (Hunt opens noon; Hill: call 919-515-3364) |
| noon | NCSU-ID teammate | start the collar print at Hill Makerspace (opens noon Sat) |
| ~12:15 | anyone | `scripts/test_bolt.py` on the real ball before wiring it to the brain |
| by 2:15 | everyone | submit; `run_sim.py` is the backup demo |

## Library facts (checked Nov 14)

- BOLT kit: ball, charging base, power adapter; about 2 hours per charge; 1-week loan, no
  renewal. Pickup at Ask Us, Hunt or Hill. Count and availability not shown online.
- Borrowing and the Makerspace both need a valid NC State Campus ID / Wolfpack One Card.
- Makerspace: online orientation first (go.ncsu.edu/online-makerspace-orientation). Free,
  first-come, no reservations, no emailed files. Prusa Core One and Prusa Mini+.
- Saturday hours: Hill Makerspace noon–8pm, Hunt Ask Us noon–6pm, Hill building 9am–10pm,
  Hunt Maker Studio closed weekends.

## Built

- [x] CLAUDE.md, README.md, git repo
- [x] Environment (Nov 14): Python 3.13.18; numpy, scipy, pandas, pyarrow, spherov2 0.12.1,
  bleak, manifold3d, trimesh, pygame. All import; spherov2's BOLT class loads. PrusaSlicer
  2.9.7 installed.
- [ ] `data/`: connectome_loader, stl_saver
- [ ] `components/`: body_interface, bolt_body, sim_body, neuron_groups, brain_model,
  motor_map, collar_model
- [ ] `pages/`: sim_page (backup demo), bolt_page (real ball)
- [ ] `scripts/`: test_bolt, make_body
- [ ] `tests/`

## Open problems

1. Whether the organizers' notebook data is FlyWire, and what format it exports.
2. Whether spherov2 can send heading/speed fast enough over this laptop's Bluetooth.
3. Whether a team member has an NC State ID (needed for the BOLT and the printer).
4. BOLT ball diameter for the collar (about 73 mm; measure the real one).

## Who is editing what

Claim a file here before editing it; clear the line when it's pushed.

- (nobody yet)
