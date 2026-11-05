# Drona SwarmSim-Lite

## Status
This is a **partially PS-compliant implementation**. The architecture, configuration, generic planner, preflight validator, RC-only controller boundary, headless PyBullet runner, synthetic pinhole/ArUco pipeline, safety rules, and automated unit tests are present. The physical calibration is not yet sufficient to claim the 10 cm tracking target, and a final four-page submission report/video must not be claimed complete.

## Architecture
`camera_sim.py → filtered estimates in run_swarm_sim.py → SafetySupervisor → CascadedPositionController → PlutoRCCommand → PlutoInterface → PyBullet`.

Only `pluto_interface.py` imports/uses PyBullet body operations. Ground truth is accessed solely for marker rendering and post-run metrics.

## Run on WSL Ubuntu 24.04
```bash
cd ~/projects/drona-swarm
source .venv/bin/activate
python -m pytest tests/test_ps_compliance.py -q
python preflight_validator.py --count 10
python run_swarm_sim.py --formation line
python run_swarm_sim.py --formation circle
```

## Fault drills
The runner schedules F1 (Drone 2 marker hidden 10.0–10.4 s) and F3 (a 1 m Drone 1 camera jump at 15 s). The timestamped `safety_events` field is written in each `logs/*_metrics.yaml` file. 

## Validator N=10
The validator uses a deterministic integer-indexed time grid and reports the first rejection with its measured value and remedy. Rejected validation exits non-zero; use --json for machine-readable reports.
~~~bash
python preflight_validator.py --count 10
python preflight_validator.py --count 10 --json
~~~

## Known gaps
- The low-level vehicle is a calibrated Bullet rigid-body surrogate, not an identified Pluto/Crazyflie URDF.
- Tracking gain/timing calibration is needed before reporting target compliance.
- Dashboard plots, phase/shape metrics, downwash model, F2/F4 methodology, final 4-page report and demo video are not finished.
- ROS 2 is intentionally excluded.

## Verified F1/F3 live drill
Run the real headless camera → safety → RC path (no ROS or downloads):

```bash
cd ~/projects/drona-swarm
./scripts/run_fault_drills.sh logs/evidence_fault_drills
```

The command injects F1 (Drone 2 marker hidden for 0.4 s) and F3 (Drone 1 camera jump at 15 s). It writes `line_metrics.yaml`, `line_run.csv`, and camera diagnostics under the evidence directory. F1 produces a timestamped `lost` event, F3 produces `outlier_rejected`, and any `LAND_AND_PAUSE` decision freezes mission time: unaffected drones receive controlled HOLD commands while the affected drone receives a controlled LAND command (`throttle=1000`, not disarm).
