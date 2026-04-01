---
title: Prompt generator
description: Build fresh implementation prompts from Dots
---

## 1. Purpose

Define a dedicated subagent or isolated execution path that turns a user request into an implementation prompt using Dots as the only source of product intent.

Use it after Dots has been updated and before any implementation starts.

## 2. Inputs

Accept one normalized request object plus runtime context.

```json
{
  "request_id": "req_001",
  "target_type": "page",
  "target_name": "dashboard",
  "user_request": "대시보드 페이지 구현해줘",
  "space_id": "default",
  "project_root": "/Users/tallpizza/project/dots-run-loop",
  "dots_source": "dots",
  "mode": "prompt_only"
}
```

Required fields:

- `target_type`: `page` or `area`
- `target_name`: canonical target name
- `user_request`: raw user text
- `space_id`: Dots scope to read from
- `project_root`: repo root
- `mode`: usually `prompt_only`

Optional fields:

- `page_name`: required when `target_type` is `area`
- `area_name`: explicit area label
- `changed_dot_ids`: recently updated nodes or edges
- `language`: prompt output language
- `format_version`: output schema version

## 3. Output contract

Return structured output that can be shown to the user without extra rewriting.

```json
{
  "status": "ok",
  "target_type": "page",
  "target_name": "dashboard",
  "prompt_markdown": "Implement the dashboard page ...",
  "summary": [
    "Targets the full dashboard page",
    "Includes KPIs, charts, and filter behavior",
    "Keeps unrelated pages out of scope"
  ],
  "evidence": {
    "dots_entities": ["page:dashboard", "area:kpi-row", "area:sales-chart"],
    "constraints": ["Preserve left nav layout", "Desktop first"],
    "gaps": []
  },
  "needs_confirmation": true
}
```

If generation is blocked, return `status: "needs_dots_update"` or `status: "error"` with concrete reasons.

## 4. Retrieval rules from Dots

Read Dots in a fresh context with no dependency on current chat history.

Retrieve in this order:

1. Exact target node for the requested `page` or `area`
2. Parent page when the target is an `area`
3. Direct children and closely related UI regions
4. Explicit constraints, priorities, and out-of-scope notes
5. Data requirements such as KPIs, charts, tables, filters, and states
6. Relevant interaction or visual behavior
7. Nearby entities only if they materially affect the target

Prefer exact and adjacent context over broad graph expansion.

Stop retrieval when the prompt can clearly describe one target without mixing unrelated surfaces.

## 5. Prompt composition rules

Compose one implementation prompt focused on a single validation unit.

The prompt should include:

- target type and canonical name
- clear implementation goal
- required UI elements
- required data or content
- required states and interactions
- constraints to preserve
- visible acceptance outcome
- out-of-scope boundary when helpful

When the target is an `area`, anchor it to its parent page and explain only the local region.

Do not include hidden reasoning, raw graph dumps, or unrelated backlog items.

Recommended prompt shape:

```md
Implement the `<page|area>` `<name>`.

Context:
- Parent page: ...
- Goal: ...

Include:
- ...
- ...

Preserve:
- ...
- ...

Out of scope:
- ...

Visible result:
- ...
```

## 6. Validation checklist

Before returning, verify all items below.

- Target is explicitly one `page` or one `area`
- Prompt can be understood without current chat history
- Parent page is named when target is an `area`
- Relevant UI, data, and interaction details are included
- Important constraints are present
- Unrelated pages or areas are excluded
- Prompt is specific enough to implement
- Missing Dots context is called out in `evidence.gaps`

If any of these fail, do not silently guess.

## 7. Failure modes and fallback behavior

If the target cannot be resolved, return `needs_dots_update` with the missing entity name.

If the target exists but key implementation details are missing, return a partial prompt plus a short gap list.

If retrieval pulls conflicting requirements, return `needs_dots_update` and list the conflict sources.

If Dots is unavailable, return `error` and do not fall back to chat-history-based prompt generation.

If isolation cannot be guaranteed, abort and report that prompt generation must run in a fresh context.

## 8. Example invocations for one page request and one area request

### Page request

Input:

```json
{
  "target_type": "page",
  "target_name": "dashboard",
  "user_request": "대시보드 페이지 구현해줘",
  "space_id": "default",
  "project_root": "/Users/tallpizza/project/dots-run-loop",
  "mode": "prompt_only"
}
```

Expected result:

```json
{
  "status": "ok",
  "target_type": "page",
  "target_name": "dashboard",
  "prompt_markdown": "Implement the dashboard page with the KPI row, trend chart, recent activity table, and top-level filters. Preserve the existing app shell and keep settings, billing, and admin pages out of scope.",
  "needs_confirmation": true
}
```

### Area request

Input:

```json
{
  "target_type": "area",
  "target_name": "daily-ad-chart",
  "page_name": "ads",
  "user_request": "광고 페이지에 일별 광고 차트 영역 고도화해줘",
  "space_id": "default",
  "project_root": "/Users/tallpizza/project/dots-run-loop",
  "mode": "prompt_only"
}
```

Expected result:

```json
{
  "status": "ok",
  "target_type": "area",
  "target_name": "daily-ad-chart",
  "prompt_markdown": "Implement the daily ad chart area inside the ads page. Show daily performance as a time-series chart with date range filtering, preserve the rest of the ads page layout, and keep campaign table changes out of scope.",
  "needs_confirmation": true
}
```

## 9. Recommended file name/location in this repo

Save this as `skills/dots-run-loop/prompt-generator-spec.md`.

Keep it next to `skills/dots-run-loop/SKILL.md` so the main skill and the subagent spec stay easy to find and evolve together.
