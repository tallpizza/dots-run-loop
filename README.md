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

See `skills/dots-run-loop/SKILL.md` and `skills/dots-run-loop/examples/` for the operating pattern and example files.

## First Run

For first-time setup, the expected minimum output is:

```text
runs/
  index.json
  run_001/
    manifest.json
    screenshots/
```

If the user has not specified the app/runtime yet, initialize only the loop files and ask follow-up questions before scaffolding an app.

## State Model

- project root `dots.json` is assumed to exist.
- `runs/index.json` is the only global file.
- `runs/run_XXX/manifest.json` is the only run-specific state file.
- There is no separate global manifest file.
- Dots remains the source of truth for long-lived requirements.
- `intent` inside each manifest is flexible and does not need a strict nested schema.

## Install

```bash
npx skills add tallpizza/dots-run-loop
```

For best compatibility, the skill lives at `skills/dots-run-loop/SKILL.md`.
