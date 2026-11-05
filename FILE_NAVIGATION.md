# File Navigation Guide

| File | Role |
|---|---|
| `config.yaml` | Fixed PS values, controller gains, vehicle limits, camera and fault settings. |
| `run_swarm_sim.py` | Main headless mission runner and integration path. |
| `simulator.py` | PyBullet backend, physical marker planes, truth firewall, drift and battery sag. |
| `camera_sim.py` | Hardware-rendered frame detection, PnP localization, noise/drop/latency queue. |
| `vision_localization/filters.py` | Alpha-beta camera-state smoothing and velocity estimate. |
| `safety.py` | Freshness, geofence, separation and outlier safety actions. |
| `controller.py` | Cascaded camera-state PID to structured Pluto RC commands. |
| `pluto_interface.py` | Arm/disarm, RC clamping, delayed RC application and PyBullet translation. |
| `formation_planner.py` | Formation slots, assignment, quintic mission phases and wave targets. |
| `preflight_validator.py` | Complete-plan 0.02 s validator and N=10 remedies. |
| `metrics.py` | Post-run metrics JSON and trajectory/tracking plots. |
| `tests/` | Regression tests. |
| `artifacts/final_candidate/` | Generated run evidence. |
| `MATHEMATICS_METHODOLOGY.md` | Equations, model values and methodology. |
| `REPRODUCIBILITY_MANIFEST.md` | Commands, inputs and evidence locations. |

## Data boundary

```text
PyBullet truth -> renderer -> ArUco/PnP -> filter -> safety -> controller
-> Pluto RC -> PlutoInterface -> PyBullet

PyBullet truth -> CSV/plots/metrics only
```

The runner bootstraps only after delayed physical-camera observations; it does not seed controller/safety positions from PyBullet truth.
