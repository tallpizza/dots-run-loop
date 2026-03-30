# dots-run-loop

Minimal run-based workflow for implementing from Dots, reviewing screenshots, recording drift, and iterating with new runs.

## Layout

```text
runs/
  index.json
  run_001/
    manifest.json
    screenshots/
```

## Core idea

- `runs/index.json` tracks the run chain
- each `run_XXX/` folder is a standalone app workspace
- each run stores all run state in `manifest.json`
- screenshots are saved in `screenshots/`

See `SKILL.md` and `examples/` for the operating pattern and example files.
