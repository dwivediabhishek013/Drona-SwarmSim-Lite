# Final Reproducibility Manifest

## Environment

```bash
cd ~/projects/drona-swarm
source .venv/bin/activate
PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 python -m pytest -q
```

## Deterministic commands

```bash
# N=10 evidence (expected non-zero status because plans are unsafe)
python preflight_validator.py --count 10 --json

# Normal mission artifacts
python - <<'PY'
from pathlib import Path
from run_swarm_sim import run
run('line', False, 36.2, Path('artifacts/final_candidate/line_normal'), 11, Path('artifacts/final_candidate/line_normal/diagnostics'))
run('circle', False, 36.2, Path('artifacts/final_candidate/circle_normal'), 11, Path('artifacts/final_candidate/circle_normal/diagnostics'))
PY

# F1/F3 line drill
python - <<'PY'
from pathlib import Path
from run_swarm_sim import run
run('line', True, 36.2, Path('artifacts/final_candidate/line_faults'), 11, Path('artifacts/final_candidate/line_faults/diagnostics'))
PY

# Metrics and plots
python metrics.py artifacts/final_candidate/circle_normal/circle_run.csv artifacts/final_candidate/circle_normal/plots
python metrics.py logs/line_run.csv artifacts/final_candidate/line_faults/plots
```

## Evidence currently generated

- `artifacts/final_candidate/circle_normal/`: complete 36.2 s circle run, diagnostics, metrics JSON, two plots.
- `artifacts/final_candidate/line_faults/`: line F1/F3 output, diagnostics, metrics JSON, two plots.
- `logs/recovery_20261105T083827Z/`: baseline snapshot and short failure/fix reproductions.

## Honest result status

- Line F1/F3 drill generated required stale/lost and one-metre-jump rejection evidence.
- Circle normal run completed without geofence/separation/lost events but its measured 3D tracking RMS is above 0.10 m.
- Therefore the provided PDF is an evidence/status report, not a claim that every performance target has passed.
