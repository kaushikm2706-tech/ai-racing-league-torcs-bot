# AI Racing League — TORCS Autonomous Driver

Capstone project for the **IBM SkillsBuild "AI Racing League: Build with AI, go full speed with IBM Bob"**
program (delivered by SkillUp). An autonomous driving client that connects to the TORCS
(The Open Racing Car Simulator) `scr_server` interface over UDP and drives a full lap of the
**Corkscrew** track with no human input.

## What this is

TORCS by itself doesn't drive — when the `scr_server` robot is selected as the driver, the game
sits and waits for an external client to connect on port 3001 and send it steering/throttle/brake
commands every simulation step. The script in this repo (`snakeoil3_gym.py`) is that client: it
reads the car's live sensor state (track range-finders, speed, RPM, wheel spin, etc.) and decides
what to do with the pedals and the wheel, 20 times a second, in real time.

## How the driver works

The driving logic (function `drive_example()` in `snakeoil3_gym.py`) is a sensor-based controller
adapted from the published TORCS Simulated Car Racing (SCR) Championship "SimpleDriver" baseline
(Loiacono, Cardamone & Lanzi), tuned specifically for Corkscrew:

- **Look-ahead braking** — uses the car's forward range-finder sensor (how far it can "see" down
  the track before the track edge curves away) to judge how much room is left before a corner,
  the same way a driver sights a braking point.
- **Trail braking** — throttle and brake are blended smoothly with an exponential curve rather than
  a hard on/off switch, so the car eases off the power on the approach and brakes progressively
  harder the closer it gets, instead of slamming the brakes or ploughing straight into the wall.
- **Graded corner speeds** — target speed is banded by how tight the upcoming corner is (hairpin,
  tight, medium, sweeper, straight) so the car carries realistic cornering speed instead of
  crawling through every bend.
- **RPM-based automatic gearbox** — shifts up/down off real RPM thresholds, like an actual gearbox,
  instead of a fixed speed table.
- **Automatic stuck-recovery** — if the car ever gets pinned against a wall (large heading angle,
  near-zero speed for a sustained number of steps), it automatically reverses with opposite
  steering lock to free itself.
- **Traction control** — cuts throttle slightly when the rear wheels spin faster than the fronts
  (wheelspin on corner exit).

## Result

Completed a full lap of Corkscrew with **0 damage, 0 pit stops, top speed 160 km/h**, finishing
1st (of 1).

## Running it

1. Launch TORCS (`wtorcs.exe`), start a **Quick Race**, set the only driver to **`scr_server 1`**
   (Configure Players → delete any human player → Select Drivers → keep only `scr_server 1`),
   and accept through to the track-loading screen. TORCS will sit waiting — that's expected, it's
   listening for the client on port 3001.
2. From this folder, run:
   ```
   python snakeoil3_gym.py
   ```
3. The client connects automatically and drives the lap.

## Files

| File | Purpose |
|---|---|
| `snakeoil3_gym.py` | The driving client — connects to `scr_server` and contains the driving logic described above. **This is the main file for this project.** |
| `gym_torcs.py` | OpenAI Gym-style environment wrapper around TORCS (from the upstream `gym_torcs` project; not used for this submission's rule-based driver, kept for reference). |
| `example_experiment.py`, `sample_agent.py` | Example scaffolding from the upstream project for a reinforcement-learning agent loop. |
| `jmcncarai.py`, `snakeoil3_jm2.py`, `torcs_jm_par.py` | Earlier driver variants kept for reference/comparison. |
| `practice.xml` | TORCS practice-mode race configuration. |
| `THIRD_PARTY_README.md`, `LICENSE-upstream` | Original README and MIT license from the upstream `gym_torcs` project this builds on. |

## Credits

Built on top of [`gym_torcs`](https://github.com/ugo-nama-kun/gym_torcs) (MIT License, © 2016 Naoto
Yoshida) and the TORCS SCR client protocol. The `drive_example()` driving logic was written and
tuned for this project.
