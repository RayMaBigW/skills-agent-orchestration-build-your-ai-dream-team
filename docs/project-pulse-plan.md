# Project Pulse Implementation Plan

## 1. Summary

Project Pulse is a small static dashboard for Mona's team. Contributors use it to see which projects are active, who owns each one, its status, recent activity, priority, and a short summary. It is built from four files:

| File | Owner | Purpose |
|------|-------|---------|
| `app/index.html` | **Coder** | Semantic page shell plus an inline script that fetches `project-data.json` and renders `.project-card` elements |
| `app/styles.css` | **Designer** | Polished, responsive, accessible styling: cards, status badges, priority treatment |
| `app/project-data.json` | **Coder** | Sample data with a top-level `projects` array |
| `.vscode/launch.json` | **Coder** | **Run Project Pulse Dashboard** launch config. It serves `app/` and opens `index.html` |

The Orchestrator coordinates and checks the result but writes no files. The Planner writes no code. No agent stages, commits, or pushes; the learner handles git through Copilot CLI.

Sections 4.1 and 4.2 define the data schema and the CSS class names before any work starts. That lets the Designer and the Coder work at the same time without editing each other's files.

## 2. Responsibilities

### Designer: owns `app/styles.css`
- Builds a polished dashboard frontend, not a plain page. Uses `border-radius`, `box-shadow`, clear typography, readable spacing and good contrast.
- Must include the selectors `.dashboard` and `.project-card`, plus every hook in the class contract (section 4.2).
- Status badges: a distinct color for each status in the schema, plus a neutral style for `unknown`. Badges always show text, so color is never the only signal.
- Priority treatment: `high` is clearly emphasized (for example an accent border or a bold badge), `medium` is moderate and `low` is subtle.
- Responsive grid: 1 column on phones, 2 on tablets, 3 or more on desktops (for example `grid-template-columns: repeat(auto-fill, minmax(280px, 1fr))`).
- Accessibility: text contrast of at least 4.5:1, visible `:focus-visible` outlines, and respect for `prefers-reduced-motion`.
- In Phase 2, reviews `app/index.html` without editing it and sends any markup or accessibility changes to the Coder through the Orchestrator.

### Coder: owns `app/index.html`, `app/project-data.json`, `.vscode/launch.json`
- `app/project-data.json`: valid JSON in the agreed schema, with 5–6 sample projects that cover every status and priority value.
- `app/index.html`:
  - `<title>` and `<h1>` both use the exact text **Project Pulse**.
  - Links `styles.css`.
  - Uses `fetch("project-data.json")`.
  - Renders one `.project-card` per project and shows `name`, `owner`, `status`, `recentActivity`, `priority` and `summary`.
  - The script is plain JavaScript inside the page: no build step, no external libraries, no separate JS file.
- `.vscode/launch.json`: strict JSON with no comments, `cwd` set to `${workspaceFolder}/app`, and it opens `http://localhost:%s/index.html`.
- Reports what changed, how it was checked, and any remaining risk.

### Orchestrator: owns no files
- Assigns file scopes, runs the phases below, passes Designer review notes to the Coder, and runs the final checks (section 7).

## 3. Ordered Implementation Steps

| # | Phase | Step | Agent | Files |
|---|-------|------|-------|-------|
| 0 | Contract | Confirm the data schema and class contract (section 4) as the source of truth for everyone | Orchestrator | none (this plan) |
| 1a | Build (parallel) | Create sample data that follows the schema | Coder | `app/project-data.json` |
| 1b | Build (parallel) | Create the launch configuration (section 4.3) | Coder | `.vscode/launch.json` |
| 1c | Build (parallel) | Build the page shell and rendering script using the class contract | Coder | `app/index.html` |
| 1d | Build (parallel) | Style every class-contract hook, the badges and the responsive layout | Designer | `app/styles.css` |
| 2 | Integration review | Designer reviews the rendered page and `app/index.html` markup for hierarchy and accessibility, then sends change requests | Designer (read-only) | none (review notes only) |
| 3 | Fixes | Coder applies markup changes; Designer adjusts styles. Each edits only their own files | Coder / Designer | `app/index.html` / `app/styles.css` |
| 4 | Validation | Run all checks in section 7 and report the result | Orchestrator (Coder runs commands if the Orchestrator has no execute tool) | none (no edits) |

## 4. Dependencies and Shared Contract

Dependencies:
- `app/index.html` rendering → depends on the **schema** of `app/project-data.json` (field names and allowed values), not on the file itself.
- `app/styles.css` → depends on the **class names and `data-*` attributes** that `app/index.html` outputs.
- `.vscode/launch.json` → depends only on `app/index.html` existing at its path. It is needed for running the app, not for writing it.
- Phase 2 → depends on all four files from Phase 1.
- Phase 4 → depends on Phase 3.

Both schema dependencies are covered by the contract below, so Phase 1 steps can run at the same time.

### 4.1 Data schema (`app/project-data.json`)
```json
{
  "lastUpdated": "2026-10-30",
  "projects": [
    {
      "id": "pulse-api",
      "name": "Pulse API",
      "owner": "Mona",
      "status": "on-track",
      "priority": "high",
      "recentActivity": "Merged PR #42: add caching layer (2 days ago)",
      "summary": "Backend service that aggregates project health signals."
    }
  ]
}
```
- Required on each project: `name`, `owner`, `status`, `recentActivity`, `priority` (all strings).
- Optional: `id`, `summary`, and the top-level `lastUpdated`.
- `status` allowed values: `on-track` | `at-risk` | `blocked` | `completed`.
- `priority` allowed values: `high` | `medium` | `low`.
- The top-level key must be exactly `projects`.

### 4.2 Markup / class contract (`app/index.html` → `app/styles.css`)
| Element | Class / attribute |
|---------|-------------------|
| `<main>` wrapper | `.dashboard` |
| Header containing `<h1>Project Pulse</h1>` and subtitle | `.dashboard__header`, `.dashboard__subtitle` |
| Status count summary (optional) | `.dashboard__summary` |
| Card list (`<ul>`) | `.project-grid` |
| Each card (`<article>` inside an `<li>`) | `.project-card` plus `data-status="…"` and `data-priority="…"` |
| Project name (`<h2>`) | `.project-card__name` |
| Owner | `.project-card__owner` |
| Status badge | `.status-badge` + `.status-badge--{on-track\|at-risk\|blocked\|completed\|unknown}` |
| Priority badge | `.priority-badge` + `.priority-badge--{high\|medium\|low\|unknown}` |
| Recent activity | `.project-card__activity` |
| Summary | `.project-card__summary` |
| Loading / empty / error message (`role="status"`, `aria-live="polite"`) | `.dashboard__message`, `.dashboard__message--error` |

### 4.3 Launch configuration (`.vscode/launch.json`)
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Run Project Pulse Dashboard",
      "type": "node-terminal",
      "request": "launch",
      "command": "python3 -m http.server 5500",
      "cwd": "${workspaceFolder}/app",
      "serverReadyAction": {
        "action": "openExternally",
        "pattern": "Serving HTTP on .* port ([0-9]+)",
        "uriFormat": "http://localhost:%s/index.html"
      }
    }
  ]
}
```
- This uses `node-terminal`, which is built into VS Code, because the devcontainer has no Python extension installed.
- `uriFormat` must open `index.html`, not `/`, so the learner sees the dashboard instead of a directory listing.

## 5. Parallel Work Decisions

**Run in parallel (Phase 1: 1a, 1b, 1c, 1d):**
- No two steps share a file. The Designer has only `app/styles.css`, and the Coder's three files are separate from each other.
- The only data dependencies, schema → HTML and HTML classes → CSS, are fixed by the contract in section 4. Nobody has to wait for another agent's output.
- If the Orchestrator runs only one Coder instance, do 1a → 1c → 1b in that order (1a first lets the Coder test rendering against real data). Run 1d alongside them either way.

**Run in order:**
- Phase 2 waits for Phase 1, because review needs the integrated page.
- Phase 3 waits for Phase 2, because fixes depend on the review notes. Coder and Designer fixes in Phase 3 can run in parallel because their files still don't overlap.
- Phase 4 waits for Phase 3, because validation must check the final files.

**Changing the contract:** if any agent needs to add or rename a class or field, it asks the Orchestrator. The Orchestrator updates both agents before either one changes its file. Agents must not change the contract on their own.

## 6. Edge Cases

| Case | Expected behavior | Owner |
|------|-------------------|-------|
| Page opened via `file://`, so `fetch` fails | `.dashboard__message--error` explains: "Open with the Run Project Pulse Dashboard launch configuration." The page must not stay blank or break | Coder |
| `fetch` returns a non-OK status (404/500) | Check `response.ok` and show an error message that includes the HTTP status | Coder |
| Invalid JSON | Catch the parse error and show a readable error message | Coder |
| `projects` is missing or not an array | Treat it as malformed data and show an error message | Coder |
| `projects` is an empty array | Show an empty-state message ("No projects yet") rather than an empty grid | Coder (text), Designer (style) |
| Unknown `status` or `priority` value | Show the raw text in the badge and use the `--unknown` modifier with neutral styling | Coder + Designer |
| A required field is missing or empty | Show a placeholder such as "Unassigned" or "No recent activity"; never print `undefined` | Coder |
| Data contains HTML or script | Insert values with `textContent` or `createElement`, never `innerHTML` built from data | Coder |
| Long names or activity text | Wrap with `overflow-wrap: anywhere`; cards must not overflow | Designer |
| Slow loading | Show an initial "Loading projects…" message in `.dashboard__message` | Coder |
| Port 5500 already in use | The launch fails with a visible terminal error. Document how to stop the old server; do not switch ports silently | Coder (note) |
| Codespaces port forwarding | `openExternally` with `localhost` is forwarded automatically; check this during validation | Orchestrator |
| Color-blind users or high-contrast mode | Badges show text labels and do not depend on color alone | Designer |

## 7. Validation Expectations

**JSON files**
- `python3 -m json.tool app/project-data.json` exits with code 0.
- `python3 -m json.tool .vscode/launch.json` exits with code 0. This also confirms the file has no comments.

**Data content**
- The top-level key is `projects`.
- Every project has `name`, `owner`, `status`, `recentActivity` and `priority`.
- The sample data covers at least one of each status and each priority.

**`.vscode/launch.json` fields**
- `name` is exactly `Run Project Pulse Dashboard`.
- `cwd` is `${workspaceFolder}/app`.
- `command` is `python3 -m http.server 5500`.
- `serverReadyAction.uriFormat` is `http://localhost:%s/index.html`.

**Text that the exercise's step 3 checker looks for in each file**
- `app/index.html` contains: `Project Pulse`, `styles.css`, `project-data.json`, `project-card`, `status`, `recentActivity`, `priority`.
- `app/styles.css` contains: `.dashboard`, `.project-card`, `border-radius`, `box-shadow`, and at least one `@media` query (or an `auto-fill` / `minmax` grid).

**Rendering**
- Run **Run Project Pulse Dashboard** from Run and Debug.
- The browser opens `/index.html`, not a directory listing.
- One card appears per project, each with a status badge, a priority badge and the activity text.
- `curl -s http://localhost:5500/index.html` returns the page.
- `curl -s http://localhost:5500/project-data.json` returns the data.

**Error paths**
- Opening `app/index.html` directly via `file://` shows the friendly error.
- Temporarily setting `projects: []` in a local copy of the data shows the empty state (no edits are committed).

**Responsive**
- At about 375px wide: one column and no horizontal scrolling.
- At about 768px: two columns.
- At about 1280px or wider: three or more columns.

**Accessibility basics**
- `<html lang="en">`, a viewport meta tag and `<meta charset>` are present.
- Exactly one `<h1>`, and card names use `<h2>`.
- Landmarks: `<header>` and `<main class="dashboard">`.
- The status message uses `aria-live`.
- Text contrast is at least 4.5:1.
- Focus outlines are visible, and badges include text labels.

**Scope and git**
- Only the four assigned files were created or changed.
- Nothing was staged, committed or pushed by an agent.

## 8. Open Questions

1. **Who owns `app/index.html`?** This plan gives it to the Coder (rendering logic) and makes the Designer a read-only reviewer. If the Orchestrator would rather have the Designer own the markup, the rendering script would need its own file (for example `app/app.js`), which is outside the four required files. Please confirm.
2. **Fallback data for `file://`:** should `index.html` include a copy of the data inside the page so it still renders when opened as a file? The trade-off is that the same data then lives in two places. The default here is no: show an error message only.
3. **`summary` and `lastUpdated`:** the brief asks for a "short contributor-friendly summary", but these fields are not in the required field list. They are planned as optional. Should they be required?
4. **Status values:** are `on-track`, `at-risk`, `blocked` and `completed` the right set, or does Mona's team use different labels (for example `paused`)?
5. **Priority vs. risk:** the brief says "priority or risk level". This plan uses one `priority` field. Is a separate `risk` field wanted?
6. **Port 5500:** is it acceptable in Codespaces, or is another fixed port preferred?
7. **Phase 4 commands:** the Orchestrator has no execute tool. Should the Coder run the validation commands and report the results?
