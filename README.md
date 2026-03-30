# dots-run-loop

Minimal run-based workflow for implementing from Dots, reviewing screenshots, recording drift, and iterating with new runs.

## Layout

```text
runs/
  run_001/
    screenshots/
    notes.json
```

## Core idea

- Dots holds the latest desired state
- each `run_XXX/` folder is a standalone app workspace
- screenshots are saved in `screenshots/`
- optional local notes can be saved in `notes.json`

See `skills/dots-run-loop/SKILL.md` and `skills/dots-run-loop/examples/` for the operating pattern and example files.

## First Run

For first-time setup, the expected minimum output is:

```text
runs/
  run_001/
    screenshots/
```

If the user has not specified the app/runtime yet, initialize only the loop files and ask follow-up questions before scaffolding an app.

## State Model

- project root `dots.json` is assumed to exist.
- Dots is the only source of truth for the latest desired state.
- `runs/run_XXX/` stores local artifacts only.
- `runs/run_XXX/notes.json` is optional and local-only.
- There is no local manifest or index file.

## Install

```bash
npx skills add tallpizza/dots-run-loop
```

For best compatibility, the skill lives at `skills/dots-run-loop/SKILL.md`.
