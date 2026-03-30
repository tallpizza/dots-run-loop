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

There is no global manifest file. Use this split instead:

- `runs/index.json`: global run index only
- `runs/run_XXX/manifest.json`: per-run snapshot and state
- Dots: source of truth for long-lived product intent

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
2. If this is the first run, ask the user the minimum setup questions listed below.
3. Create a new `run_XXX/` folder.
4. Write `manifest.json` for that run using the example in this skill.
5. Include an `execution_prompt` field inside `manifest.json`.
6. Implement inside that run folder.
7. Save screenshots under `screenshots/` and record them in `manifest.json.result.screenshots`.
8. When the user reports drift, append the raw feedback and summarize it into `review.drift`.
9. Convert the review into `review.next_run` instructions.
10. Create the next run from the prior run plus the new instructions.

## Initial Setup

When the user says something like `dots loop 초기세팅 해줘`, do not jump straight to implementation. First create the minimal run structure and gather the missing setup inputs.

Create these files and folders first:

```text
runs/
  index.json
  run_001/
    manifest.json
    screenshots/
```

For the first run, `run_001/manifest.json` must start from an empty review state:

- `run.based_on_run_id = null`
- `review.user_feedback = []`
- `review.drift = []`
- `review.decision.accepted = false`
- `review.decision.reason = ""`
- `review.next_run.instructions = []`
- `review.next_run.manifest_additions = []`

If the project folder is otherwise empty, ask the user the minimum questions needed to seed the first run:

1. What app/runtime should the run workspace use?
   Examples: `Next.js`, `React + Vite`, `static HTML`, `unknown`
2. Should the first run only create the run loop files, or also scaffold the app?
   Examples: `loop only`, `scaffold app too`
3. Which screens from Dots should be included in `run_001`?
   Examples: `all`, `home + orders`, `orders only`
4. What is the main goal sentence for this first run?
5. Are there any hard constraints missing from Dots that must be inserted into `intent.constraints` now?

If the user does not specify app/runtime, keep the run workspace minimal and only create the loop files. Do not invent a framework.

If the user asks for implementation after setup, use `execution_prompt` to record the exact generation instruction.

## Execution Prompt Rules

For `run_001`, the `execution_prompt.inputs` should only reference the current intent:

- `intent.goal`
- `intent.screens`
- `intent.constraints`
- `intent.acceptance`
- `intent.out_of_scope`

For `run_002+`, you may additionally include:

- `review.next_run.instructions`
- `review.next_run.manifest_additions`

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

- `skills/dots-run-loop/examples/runs-index.json`
- `skills/dots-run-loop/examples/run-manifest.json`

Use `run-manifest.json` as the default template for `run_001`. For later runs, keep the same shape and fill `review` and `result` as the loop progresses.
