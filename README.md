# fly-sphero-hack: status board

Real sense → descending-neuron pathways, extracted from the fruit fly connectome with the
organizers' Colab notebook, drive a Sphero BOLT robot ball. The laptop shows the pathway
lighting up, cutting a neuron in the data changes how the robot behaves, and people can
talk to it to pick which sense gets stimulated.

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
- Nov 14: the extracted pathways are the core of the project, not a black box behind the
  ball. Pathway diagram and lesion mode cover the prize's "Data-Driven Insights" and
  "Rapid Prototyping" bullets; voice is now the input picker.

## Build order (stop wherever the clock runs out)

1. **Notebook → pathway → sim ball.** Open the organizers' notebook first and see what it
   exports. Export one pathway (sugar or sound → descending neurons), run
   `list_pathways.py`, and drive the sim ball with it. Without this there is no project.
2. **Pathway diagram + lesion mode.** The live circuit on screen and click-to-cut. This is
   the "insights" pitch, not an extra.
3. **Real BOLT + LED screen.** Swap the sim for the ball; the screen shows the brain's choice
   (arrow, activity, or a word picked by what fired).
4. **Mic picks the sense.** "food" → taste, loud noise → hearing/escape, "danger" → looming.
5. **Narrator.** One spoken sentence about which neurons fired and what the fly did.

## Today's plan

| Time | Who | What |
|---|---|---|
| 10:30–11:00 | everyone | open the notebook; decide which pathways to export |
| 11:00–noon | everyone | steps 1–2 in the sim; `test_bolt.py` and `test_matrix.py` ready |
| noon | NCSU-ID teammate | pick up a BOLT at Ask Us (Hunt opens noon; Hill: call 919-515-3364) |
| ~12:15 | anyone | `test_bolt.py` then `test_matrix.py` on the real ball |
| 12:30–2:00 | everyone | steps 3–5 |
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
- [ ] `components/`: body_interface, bolt_body, sim_body, neuron_groups, pathway_finder,
  lesion, pathway_view, brain_model, motor_map, matrix_display, mic_listener,
  sense_encoder, narrator
- [ ] `pages/`: sim_page (backup demo), bolt_page (real ball)
- [ ] `scripts/`: test_bolt, test_matrix, test_mic, list_pathways
- [ ] `tests/`

## Open problems

1. **Check first:** what the organizers' notebook exports. Full connectome, or only query
   helpers that need an online account or token? Is it FlyWire? Does it label cell types,
   sense neurons, descending neurons, and hemisphere? The whole plan depends on this.
2. Whether the pathways are strong enough. Some senses may barely reach steering neurons
   within a few hops. Try several, keep the clearest left/right difference, and say in the
   demo which ones were tried. Sugar → feeding is the safest start (a 2024 whole-brain model
   predicted it). Check descending-neuron names in the data, not from memory.
3. Whether lesion mode visibly changes behavior for the chosen pathway.
4. Whether spherov2 can send heading/speed fast enough over this laptop's Bluetooth.
5. Whether a team member has an NC State ID (needed to borrow the BOLT).
6. Mic and speech: which packages (live mic, speech-to-text, laptop voice). Not chosen,
   not installed. Speech-to-text either needs an internet service or a downloaded model.
7. An LLM for the narrator needs an API key; without one, the narrator uses fill-in-the-blank
   sentences.

## Who is editing what

Claim a file here before editing it; clear the line when it's pushed.

- (nobody yet)
