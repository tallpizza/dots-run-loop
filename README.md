# dots-run-loop

Minimal run-based workflow for iterating on a single app workspace from Dots, reviewing screenshots, recording drift, and improving the Dots state over time.

The loop is centered on whether Dots can generate a good implementation prompt for a requested `page` or `area`.

## Layout

```text
project/
  dots.json
  package.json
  src/
  public/
  runs/
    run_001/
      screenshots/
      notes.json
    run_002/
      screenshots/
      notes.json
```

## Core idea

- Dots holds the latest desired state.
- The app workspace is persistent and evolves across runs.
- Each `run_XXX/` folder records what happened in one attempt.
- Screenshots are saved in `screenshots/`.
- Optional local notes can be saved in `notes.json`.
- Implementation attempts should be run through `opencode` from Bash using an inline prompt built from Dots.
- Validation focuses on whether a clean prompt can be generated for a `page` or `area` from the current Dots state.

## Request Loop

When the user gives a request such as `대시보드 페이지 구현해줘` or `광고 페이지에 일별 광고 차트 영역 고도화해줘`, the expected loop is:

1. Interpret the request as targeting either a `page` or an `area`.
2. Update Dots first so the latest requirement, constraint, or drift is stored there.
3. Generate a fresh implementation prompt from Dots only.
4. Show that generated prompt to the user and get confirmation.
5. Implement in the main workspace after confirmation.
6. Record screenshots and small local notes under `runs/run_XXX/`.

The prompt generation step should happen from a fresh context. In practice, use a dedicated prompt-generator subagent or equivalent isolated execution path that reads Dots and does not rely on the current chat history as hidden context.

## Validation Units

- `page`: full screen or page-level implementation target
- `area`: a focused region inside a page such as a chart section, KPI block, or table area

The main validation question is: `Can the current Dots state generate a clear implementation prompt for this page or area?`

## Prompt Validation

A generated prompt is considered good when it includes:

- the exact target `page` or `area`
- the relevant data, charts, tables, filters, or KPI context
- the constraints that must be preserved
- the expected visible outcome
- any explicit out-of-scope boundary when relevant

If the prompt is weak, missing context, or mixes unrelated areas, improve Dots first and regenerate the prompt before implementing.

See `skills/dots-run-loop/SKILL.md` and `skills/dots-run-loop/examples/` for the operating pattern and example files.

## First Run

For first-time setup, the expected minimum output is:

```text
runs/
  run_001/
    screenshots/
```

If the user has not specified the app/runtime yet, initialize only the run artifact folders and ask follow-up questions before scaffolding the main app workspace.

## State Model

- Project root `dots.json` is assumed to exist.
- Dots is the only source of truth for the latest desired state.
- App source lives once in the main project workspace.
- `runs/run_XXX/` stores local artifacts only.
- `runs/run_XXX/notes.json` is optional and local-only.
- There is no local manifest or index file.

## Legacy Compatibility

- Older projects may still have `run_XXX/` folders that contain full app snapshots.
- Treat those folders as legacy historical artifacts.
- New runs should continue in the main workspace and keep only screenshots and notes under `runs/`.

## Install

```bash
npx skills add tallpizza/dots-run-loop
```

For best compatibility, the skill lives at `skills/dots-run-loop/SKILL.md`.
