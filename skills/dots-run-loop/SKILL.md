---
name: dots-run-loop
description: Minimal run-based workflow for Dots implementation, screenshot review, drift recording, and iterative reruns.
---

# Skill: dots-run-loop

Use this skill when the user wants to run an implementation-review-revision loop backed by Dots and local `runs/` folders.

## Purpose

This skill defines a minimal run-based workflow:

- `runs/index.json` tracks the run chain
- each `runs/run_XXX/` folder is a standalone app workspace
- each run keeps a single `manifest.json`
- screenshots live in `runs/run_XXX/screenshots/`

The `manifest.json` file is the source of truth for that run. It includes:

- run metadata
- Dots-derived intent
- execution prompt
- implementation result summary
- screenshot references
- user feedback
- structured drift
- next run instructions

## Folder Layout

```text
project/
  dots.json
  runs/
    index.json
    run_001/
      manifest.json
      screenshots/
      package.json
      src/
      ...
    run_002/
      manifest.json
      screenshots/
      package.json
      src/
      ...
```

## Workflow

1. Read Dots and summarize the current intent.
2. Create a new `run_XXX/` folder.
3. Write `manifest.json` for that run using the examples in this skill.
4. Include an `execution_prompt` field inside `manifest.json`.
5. Implement inside that run folder.
6. Save screenshots under `screenshots/` and record them in `manifest.json.result.screenshots`.
7. When the user reports drift, append the raw feedback and summarize it into `review.drift`.
8. Convert the review into `review.next_run` instructions.
9. Create the next run from the prior run plus the new instructions.

## Rules

- Keep only one run index file: `runs/index.json`.
- Keep only one run state file per run: `runs/run_XXX/manifest.json`.
- Do not split review or drift into separate JSON files.
- Keep screenshots in a dedicated `screenshots/` folder only.
- Treat `intent` as the normalized view of Dots for this run.
- Treat `execution_prompt` as the exact prompt used for generation.
- Preserve user feedback text in `review.user_feedback`.
- Keep drift concise and implementation-oriented.

## Suggested File Naming

Screenshots:

```text
{screen-id}-{viewport}-{label}.png
```

Examples:

```text
orders-list-desktop-default.png
orders-list-mobile-default.png
orders-list-mobile-filter-open.png
```

## Run Status Values

- `generated`
- `review_pending`
- `reviewed`
- `accepted`

## Drift Types

- `missing`
- `incorrect`
- `overbuilt`
- `constraint_violation`

## Example Files

- `examples/runs-index.json`
- `examples/run-manifest.json`

Use these as templates when initializing a new project or a new run.
