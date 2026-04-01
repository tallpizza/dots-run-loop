---
name: dots-run-loop
description: Minimal run-based workflow for Dots implementation, screenshot review, drift recording, and iterative reruns.
---

# Skill: dots-run-loop

Use this skill when the user wants to run an implementation-review-revision loop backed by Dots, a persistent project workspace, and local `runs/` artifact folders.

Assume the project root already contains `dots.json`.

## Purpose

This skill defines a minimal run-based workflow where Dots is the single source of truth for the current desired state.

The main goal of the loop is not just to generate code. The main goal is to refine and complete the current state in Dots so future runs produce better results with less drift.

Treat each run as a probe against the current Dots state:

- implementation output is a way to test how complete Dots is
- drift is evidence that Dots is missing clarity, constraints, or priorities
- the primary outcome of the loop is a better Dots state
- generated files and screenshots are secondary artifacts

- Dots stores the latest desired state only
- the main project workspace stores the current implementation
- local `runs/run_XXX/` folders store review artifacts only
- screenshots live in `runs/run_XXX/screenshots/`
- optional local notes can be stored in `runs/run_XXX/notes.json`

There is no local manifest file that competes with Dots. Use this split instead:

- Dots: source of truth for the current product intent
- main project workspace: current implementation that persists across runs
- `runs/run_XXX/`: local artifacts for one attempt
- `runs/run_XXX/notes.json`: optional local record of prompt, feedback, and applied changes

The agent should normalize whatever is currently in Dots into a working `intent` in memory. Do not require a fixed Dots node or edge model.

If the user wants history, keep it locally in the run folder. Do not treat local files as the source of truth.

When actually running an implementation attempt, do not use the current chat context as the generator. Always try to invoke `opencode` from Bash with an inline prompt built from the current Dots state.

The preferred validation unit is one of these:

- `page`
- `area`

Treat requests like `대시보드 페이지 구현해줘` as `page` requests.
Treat requests like `광고 페이지에 일별 광고 차트 영역 고도화해줘` as `area` requests.

## Folder Layout

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

## Workflow

1. Read Dots and summarize the current intent.
2. If this is the first run, ask the user the minimum setup questions listed below.
3. Interpret the user request as targeting either a `page` or an `area`.
4. Update Dots first so the new requirement, refinement, or drift is stored in the graph before implementation.
5. Create a new `run_XXX/` artifact folder.
6. Generate a fresh implementation prompt from Dots only.
7. Always try to execute prompt generation through an isolated path such as a dedicated prompt-generator subagent or `opencode` from Bash with an inline prompt assembled from Dots.
8. Do not treat the current assistant message context as the hidden source for prompt generation when this loop is running.
9. Show the generated prompt to the user and get confirmation before implementation.
10. Implement in the existing main project workspace after confirmation.
11. Save screenshots under `runs/run_XXX/screenshots/`.
12. If useful, write a small `notes.json` with the execution prompt, generated prompt, user feedback, reviewed areas, and applied changes.
13. When the user reports drift, first interpret that drift as a problem in the current Dots state unless it is clearly just an implementation bug.
14. Update Dots so it reflects the new latest desired state.
15. Start the next run from the updated Dots state and continue in the same codebase unless the user explicitly asks for an isolated experiment.

## Prompt Generation Validation

The primary validation question is not `can we regenerate the whole app?`.

The primary validation question is: `can the current Dots state generate a clear implementation prompt for the requested page or area?`

Before implementation, validate the generated prompt against these checks:

- target `page` or `area` is explicit
- relevant UI region is explicit
- relevant data, filters, KPIs, charts, or tables are included when needed
- important constraints are present
- expected visible outcome is clear
- unrelated pages or areas are not mixed in

If the prompt fails these checks, improve Dots and regenerate instead of implementing immediately.

## Drift Handling

When drift appears, prefer asking: `what should change in Dots so the next run is better?`

Before rerunning, classify the drift in one of these ways:

- Dots was ambiguous
- Dots missed an important constraint
- Dots missed an out-of-scope boundary
- Dots did not express the desired tone or priority clearly enough
- the implementation failed even though Dots was already clear

In most cases, improve Dots first, then rerun.

## Initial Setup

When the user says something like `dots loop 초기세팅 해줘`, do not jump straight to implementation. First create the minimal run structure and gather the missing setup inputs.

Create these files and folders first:

```text
runs/
  run_001/
    screenshots/
```

If the project folder is otherwise empty, ask the user the minimum questions needed to seed the first run:

1. What app/runtime should the main workspace use?
   Examples: `Next.js`, `React + Vite`, `static HTML`, `unknown`
2. Should the first run only create the run artifact folders, or also scaffold the main app workspace?
   Examples: `loop only`, `scaffold app too`
3. Which screens from Dots should be included in `run_001`?
   Examples: `all`, `home + orders`, `orders only`
4. What is the main goal sentence for this first run?
5. Are there any hard constraints or notes missing from Dots that should be added now?

If the user does not specify app/runtime, keep the project minimal and only create the run artifact folders. Do not invent a framework.

If the user asks for implementation after setup, keep the exact generation instruction in local notes if needed.

## Local Notes Shape

`notes.json` is optional. If you create it, keep it small and local-only.

Recommended fields:

- `run_id`
- `execution_prompt`
- `generated_prompt`
- `opencode_command`
- `feedback`
- `reviewed_areas`
- `applied_changes`
- `screenshots`

## Rules

- Dots is the only source of truth for the latest desired state.
- The main purpose of the loop is to improve Dots, not just to produce one good run.
- Do not create a local manifest that duplicates Dots state.
- Do not require a fixed Dots node or edge model.
- App source should stay in one persistent workspace by default.
- Do not copy or regenerate a fresh standalone app per run unless the user explicitly wants an isolated experiment, reset, or side-by-side variant.
- Always classify the active request as either a `page` or `area` target.
- Always update Dots before generating the implementation prompt for a new request or drift refinement.
- Always generate the implementation prompt from a fresh context that reads Dots, rather than relying on the current conversation memory.
- Always show the generated prompt to the user for confirmation before implementation.
- When running an implementation attempt, prefer `opencode` via Bash with an inline prompt assembled from Dots.
- Do not use the current assistant conversation as the direct generation mechanism for run execution unless `opencode` is unavailable.
- Keep screenshots in a dedicated `screenshots/` folder only.
- If drift is reported, prefer updating Dots before rerunning.
- Treat drift as a signal about missing or weak specification unless there is strong evidence of a pure implementation bug.
- Use local notes only for optional history or debugging.
- Keep local notes concise and implementation-oriented.

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

- `skills/dots-run-loop/examples/run-notes.json`
- `skills/dots-run-loop/prompt-generator-spec.md`

Use `run-notes.json` only when local history is useful. Otherwise the run folder can contain just screenshots or screenshots plus a small notes file.
