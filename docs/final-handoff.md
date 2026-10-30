# Project Pulse Final Handoff

Project Pulse is a static dashboard that shows Mona's team each project's owner, status, recent activity, priority and summary. The four agents in [agent-team.md](agent-team.md) built it from the plan in [project-pulse-plan.md](project-pulse-plan.md).

## What each agent did

| Agent | Contribution |
|-------|--------------|
| **Orchestrator** | Coordinated the phases and file scopes. Ran Phase 1 as parallel Coder + Designer work because their file scopes didn't overlap, ran Phase 2 as a Designer review, and ran Phase 4 validation. Wrote no files. |
| **Planner** | Wrote `docs/project-pulse-plan.md`, which covers the data schema, class contract, phases, edge cases, validation checklist and open questions. Wrote no code. |
| **Designer** | Owned `app/styles.css` and made the visual and accessibility decisions: tokens, status and priority badges, responsive grid, focus outlines, dark mode, reduced motion and forced colors. Reviewed the markup and asked for no HTML changes. |
| **Coder** | Built `app/index.html`, `app/project-data.json` and `.vscode/launch.json`. Ran the validation commands and, when the learner asked, the git commands. |

## Deliverables

- `app/index.html`: page shell plus an inline script. It fetches `project-data.json` and renders one `.project-card` per project using `textContent`, with no `innerHTML`. It also handles loading, empty, error and `file://` states.
- `app/styles.css`: polished, responsive and accessible styles for every hook in the class contract.
- `app/project-data.json`: 6 sample projects covering all 4 statuses and all 3 priorities.
- `.vscode/launch.json`: the **Run Project Pulse Dashboard** launch configuration.

## How to run

1. Open **Run and Debug** in VS Code.
2. Choose **Run Project Pulse Dashboard** and press Start.
3. The configuration runs `python3 -m http.server 5500` from `${workspaceFolder}/app` and opens `http://localhost:5500/index.html`, so you see the dashboard instead of a directory listing.

## Dashboard validation results

These checks were run on 2026-10-30 against the files in the working tree.

| Check | Command / method | Result |
|-------|------------------|--------|
| `app/project-data.json` is valid JSON | `python3 -m json.tool app/project-data.json` | PASS |
| `.vscode/launch.json` is valid JSON with no comments | `python3 -m json.tool .vscode/launch.json` plus a search for comments | PASS |
| Top-level `projects` key; each project has `name`, `owner`, `status`, `recentActivity`, `priority` | Python script | PASS (6/6) |
| Sample data covers every status and priority | Python script | PASS |
| Launch `name`, `command`, `cwd`, `serverReadyAction.uriFormat` | Python script | PASS (all 4) |
| `app/index.html` contains `<title>Project Pulse</title>`, `styles.css`, `project-data.json`, `project-card`, `status`, `recentActivity`, `priority` | String checks | PASS (7/7) |
| `app/styles.css` contains `.dashboard`, `.project-card`, `border-radius`, `box-shadow`, `@media` | String checks | PASS (5/5) |
| HTTP 200 for `index.html`, `styles.css`, `project-data.json` | `curl` against `python3 -m http.server 5500` | PASS (200 ×3) |
| Render: `.project-card` count equals project count | jsdom in `/tmp` | PASS (6 = 6) |
| Render: name, status badge, priority badge, activity text and `data-*` attributes on each card | jsdom | PASS (6/6) |
| Render: one `<h1>`, loading message hidden after load, status summary shown | jsdom | PASS |
| Cleanup: server stopped, temp files removed, port 5500 free | `kill <PID>`, `lsof` | PASS |

## Review notes: plan vs. implementation

- **Desktop column count:** at a 1280px viewport the `auto-fill`/`minmax(280px, 1fr)` grid fits **4** columns (1232px of content), but a comment in `app/styles.css` says "3 at ~1280px". The plan asks for "3 or more", so the layout passes; only the comment is wrong.
- **Summary placement:** `.dashboard__summary` sits inside the header. `app/styles.css` styles it for that position, so nothing breaks.
- **Hidden elements:** hidden elements use both the `hidden` attribute and an inline `display: none`. This repeats itself but does no harm.
- **Extra file:** `.vscode/tasks.json` is in the repo, but it came from the initial commit and none of the agents changed it.

## Learner handoff notes

**Manual checks still to do in a real browser**
- Responsive columns: check for 1 column at about 375px with no horizontal scroll, 2 at about 768px, and 3 or more at about 1280px.
- Focus outlines: press Tab through the page and confirm the `:focus-visible` outlines are visible.
- Contrast: confirm at least 4.5:1 in both light and dark mode, using DevTools or an a11y checker.
- `file://` error: open `app/index.html` directly and confirm the friendly error appears.
- Codespaces: confirm port 5500 is forwarded automatically and the browser opens `/index.html`.

**If port 5500 is busy**
```bash
lsof -nP -iTCP:5500 -sTCP:LISTEN   # find the PID
kill <PID>                          # then rerun the launch configuration
```

**Open questions from the plan that are still unresolved**
- Should there be fallback data embedded in the page for `file://`? The current answer is no; the page shows an error instead.
- Should `summary` and `lastUpdated` stay optional or become required?
- Are the status labels right (`on-track`, `at-risk`, `blocked`, `completed`), or is something like `paused` needed?
- Should there be a separate `risk` field alongside `priority`?
- Is port 5500 acceptable in Codespaces?
- Resolved in practice: the Coder owns `app/index.html`, and the Coder ran the Phase 4 commands.

**Git**
- You control all git operations through Copilot CLI prompts. No agent staged, committed or pushed this handoff.
