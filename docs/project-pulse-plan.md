# Project Pulse implementation plan

## Summary

Project Pulse is a lightweight static dashboard for Mona's team that surfaces a
quick read on active projects: owner, current status, recent activity, and
priority/risk. This plan builds the dashboard as a small three-file static app
under `app/` (`index.html`, `styles.css`, `project-data.json`) plus a
`.vscode/launch.json` entry called **Run Project Pulse Dashboard** that serves
from `app/` and opens `index.html` directly (never a directory listing). Work is
split between two specialists per the agent definitions in `.github/agents/`:
the **Designer** owns the visual layer and the **Coder** owns structure, data,
and launch configuration. The Orchestrator sequences the phases so file scopes
never overlap in the same phase.

## Ordered implementation steps

1. **Phase 1 — Data + structural skeleton (Coder).**
   Create `app/project-data.json` with a top-level `projects` array and create
   `app/index.html` with the semantic structure and deterministic CSS hooks the
   Designer will target (`.dashboard` root, one `.project-card` per project,
   plus badge/priority hooks). HTML should reference `styles.css` and render
   the seeded projects (either statically written to match the JSON or loaded
   from `project-data.json` via a small inline `fetch` script — see Open
   questions).
2. **Phase 2 — Visual design (Designer).**
   Create `app/styles.css` targeting the hooks defined in Phase 1: dashboard
   grid/layout, project card treatment, status badges, priority treatment,
   typography, spacing, responsive behavior, and accessible color contrast.
   Designer does not modify `index.html` markup; if a missing hook is needed,
   Designer requests it from the Orchestrator, who reassigns to Coder.
3. **Phase 3 — Launch configuration (Coder).**
   Create `.vscode/launch.json` with a configuration named
   **Run Project Pulse Dashboard** that sets `cwd` to
   `${workspaceFolder}/app` and opens `index.html` so the learner sees the
   dashboard, not a directory listing. Strict JSON, no comments.
4. **Phase 4 — Integration verification (Orchestrator + learner).**
   Launch the config, confirm the dashboard renders, cards appear, badges and
   priority styling are visible, JSON parses, and layout is responsive.

## File assignments

| File                       | Owner    | Phase | Notes                                                                 |
| -------------------------- | -------- | ----- | --------------------------------------------------------------------- |
| `app/project-data.json`    | Coder    | 1     | Top-level `projects` array; each item has `name`, `owner`, `status`, `recentActivity`, `priority`. |
| `app/index.html`           | Coder    | 1     | Semantic markup with `.dashboard` and `.project-card` hooks; links `styles.css`. |
| `app/styles.css`           | Designer | 2     | Owns all visual styling; must not require HTML changes.               |
| `.vscode/launch.json`      | Coder    | 3     | Strict JSON; `cwd = ${workspaceFolder}/app`; opens `index.html`.      |

No file has more than one owner. If a scope conflict arises (e.g., Designer needs
a new markup hook), the Orchestrator returns that file to the Coder in a follow-up
phase — Designer does not edit HTML.

## Designer responsibilities

- Own `app/styles.css` end-to-end.
- Deliver a polished dashboard look — not a bare HTML page.
- Target the deterministic hooks `.dashboard` and `.project-card`, plus
  status-badge and priority classes exposed by the Coder in `index.html`.
- Establish clear visual hierarchy: project name > status/priority > owner >
  recent activity.
- Use status badges with distinct, accessible colors (e.g., On Track / At Risk /
  Blocked / Complete) with sufficient contrast (WCAG AA).
- Provide priority treatment (High / Medium / Low) that is distinguishable
  without relying on color alone (icon, weight, or label).
- Provide readable spacing, rounded corners, subtle shadows, and clear
  typography.
- Provide a responsive layout (multi-column on wide screens, single column on
  narrow screens) using CSS grid or flexbox.
- Report design decisions and validation notes back to the Orchestrator.
- Stay within `app/styles.css`. Do not touch HTML, JSON, or launch config.
- Do not stage, commit, or push.

## Coder responsibilities

- Own `app/index.html`, `app/project-data.json`, and `.vscode/launch.json`.
- `project-data.json`: valid JSON, top-level `projects` array, each project
  object has `name`, `owner`, `status`, `recentActivity`, `priority`. Seed at
  least 4–6 realistic projects covering multiple statuses and priorities so the
  Designer's badge/priority treatments are all exercised.
- `index.html`: semantic HTML5 document, references `styles.css`, exposes the
  hooks `.dashboard` (root container) and `.project-card` (per project), plus
  class hooks for status (e.g., `.status-badge`, `.status-on-track`, etc.) and
  priority (e.g., `.priority-high`) so the Designer can style them
  deterministically. Include a dashboard header/title.
- `.vscode/launch.json`: strict JSON, no comments. Configuration name exactly
  **Run Project Pulse Dashboard**. `cwd` set to `${workspaceFolder}/app`.
  Opens `index.html` (not the directory).
- Keep control flow simple; make errors explicit; follow existing repo
  patterns (see `.vscode/tasks.json` for style).
- Validate: JSON parses, HTML loads locally, launch config opens the page.
- Stay within assigned files. Do not touch `styles.css`.
- Do not stage, commit, or push.

## Dependencies

- **Phase 2 depends on Phase 1**: `styles.css` targets hooks and structure
  defined in `index.html`. Designer cannot start until the Coder has published
  the hook contract.
- **Phase 3 depends on Phase 1**: `launch.json` must open an `index.html` that
  actually exists at `app/index.html`.
- **Phase 3 is independent of Phase 2**: the launch config does not depend on
  styling.
- **Phase 4 depends on Phases 1, 2, and 3**.
- `project-data.json` and `index.html` are both owned by the Coder in Phase 1;
  the HTML depends on the JSON schema, so Coder should finalize the JSON shape
  first and then reflect it in the HTML.

## Parallel work decisions

- **Sequential:** Phase 1 → (Phase 2, Phase 3) → Phase 4.
- **Parallel (safe):** After Phase 1 completes, **Phase 2 (Designer,
  `styles.css`)** and **Phase 3 (Coder, `.vscode/launch.json`)** can run in
  parallel — they touch disjoint files and neither depends on the other.
- **Not parallelizable:** Phase 1's two Coder files (`project-data.json` and
  `index.html`) should stay in one phase because the HTML structure depends on
  the JSON schema and both are owned by the same agent; splitting them adds
  coordination cost without benefit.
- Designer and Coder must never share a phase that writes to the same file.
  Per the Orchestrator's delegation rules, overlapping file scopes are kept in
  separate phases.

## Validation expectations

The learner should verify all of the following:

1. **Launch config works.** In VS Code, Run and Debug → **Run Project Pulse
   Dashboard** opens the dashboard directly. The URL/preview shows the
   dashboard UI, not an `app/` directory listing.
2. **Working directory.** The launch configuration's `cwd` is
   `${workspaceFolder}/app`.
3. **JSON validity.** `app/project-data.json` parses (e.g.,
   `python -m json.tool app/project-data.json` or `jq . app/project-data.json`
   succeeds). Top-level key is `projects` (array). Every entry has `name`,
   `owner`, `status`, `recentActivity`, `priority`.
4. **`launch.json` validity.** File is strict JSON with no comments and no
   trailing commas.
5. **Cards render.** One `.project-card` visible per project in
   `project-data.json`. Every required field is visible on the card.
6. **Styling visible.** Page clearly looks like a designed dashboard: header,
   grid of cards, status badges with distinct colors, priority treatment,
   spacing, typography, shadows/rounded corners.
7. **Responsive.** Layout reflows to a single column on narrow viewports
   (~<600px) and multi-column on wide viewports.
8. **Accessibility smoke check.** Text has sufficient contrast; priority is
   not conveyed by color alone; headings form a sensible outline.
9. **No console errors** when the page loads.

## Edge cases to handle

- **Empty projects array.** HTML/CSS should degrade gracefully (empty state or
  at minimum no layout breakage). Recommended: Coder renders a simple "No
  projects yet" empty state hook that Designer can style.
- **Long project names or activity strings.** Cards must wrap or truncate
  cleanly without breaking the grid.
- **Unknown status or priority values.** Designer should provide a neutral
  fallback style so unrecognized values still render legibly rather than
  invisible.
- **Missing optional fields.** All five fields are required by the brief; Coder
  should treat missing fields as a data error, but the UI should not crash if a
  field is empty.
- **File:// vs http:// context.** If `index.html` is fetched via `file://`, a
  `fetch('project-data.json')` call may be blocked by the browser. Two safe
  options: (a) inline the seed data into the HTML at build time, or (b) ensure
  the launch config serves `app/` over HTTP so `fetch` works. Decision belongs
  to the Coder — see Open questions.
- **Directory listing risk.** If the launch config points at `app/` without
  specifying `index.html`, some servers show a directory listing. Coder must
  explicitly target `index.html`.
- **JSON comments.** `.vscode/launch.json` in this repo must be strict JSON per
  the Coder agent definition — no `//` comments, even though VS Code tolerates
  them elsewhere.
- **Existing `.vscode/` contents.** `tasks.json` already exists; Coder must
  create `launch.json` as a new sibling file and not modify `tasks.json`.

## Open questions

1. **Data loading strategy.** Should `index.html` load
   `project-data.json` at runtime via `fetch` (nicer separation, but requires
   an HTTP context), or should it inline the seeded projects into the HTML
   (works from `file://`, but duplicates data)? Recommendation: use `fetch` and
   have the launch config serve over HTTP via Live Preview / Simple Browser so
   the two data sources stay in sync.
2. **Launch mechanism.** The brief says "VS Code launch configuration." Should
   `.vscode/launch.json` use the Live Preview extension, `chrome`/`edge`
   `type: "chrome"` debug, or Simple Browser? The Coder should pick the
   option already available in the devcontainer; if unclear, ask the
   Orchestrator before Phase 3.
3. **Status vocabulary.** Confirm the canonical status set (e.g., On Track /
   At Risk / Blocked / Complete) and priority set (High / Medium / Low) so the
   Designer's badge palette matches the seeded data.
4. **Branding.** Any color palette, logo, or typographic direction from Mona's
   team, or is Designer free to choose within accessibility constraints?
5. **Interactivity scope.** Is Project Pulse strictly read-only for this
   iteration, or should cards be clickable / filterable? Current plan assumes
   read-only.
