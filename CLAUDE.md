# CIC — Performance & M&E Management System · CLAUDE.md

Last updated: 2026-10-05 (demonstration build 1.0)

A demonstration of **Eval360 configured for the Construction Industry Council (CIC), Eswatini**.
It shows CIC executives how one system can connect strategy → programmes → projects and
contractors → indicators → evidence → compliance → corrective action → risk → management
accountability → executive reporting.

**All data is fictional.** People, contractors, projects and every figure are illustrative.
Only CIC's general organisational functions are real. Every screen carries the
"Demonstration Environment" strip; never remove it, and never present a figure as a CIC record.

The architecture and rules are carried over from the FPC demonstration (`../fpbc-eval360`),
which in turn inherited them from Eval360 (`../atbu`).

---

## What is in the repository

| Path | What it is |
|---|---|
| `cic_eval360_demo.html` | **The demonstration.** One self-contained file: CSS, shell, data, generators, calculations, 43 screens. This is the source of truth — edit it directly. |
| `index.html` | Landing page served at the folder root. |
| `scripts/selftest.sh` | Headless check: renders every screen as every role, runs the data checks. Run it after every change. |
| `scripts/deploy_pull.sh` | Server-side pull (same pattern as FPC). |
| `docs/DEPLOY.md` | GitHub and server deployment. |
| `DESIGN_SYSTEM.md` | Tokens, typography and component rules. |

There is no PHP, no build step, no package manager. Chart.js, Bootstrap Icons and Google Fonts
load from CDNs. Do not add a framework, a bundler or a dependency without saying so and why.

---

## Layout of the demo file

Sections are numbered in comments — search for `═══ N.`:

| § | Contents |
|---|---|
| 1–3 | Fixed clock (`TODAY_ISO = 2026-10-05`), seeded RNG (`mulberry32`, `quota()`), formatting |
| 4–6 | `appData` config, periods, regions, departments, staff; users, `ROLE_ACCESS`, `NAV`, `DETAIL_VIEWS` |
| 7–14 | Master data: objectives, programmes, interventions, 38 indicators (`IND_SPEC`/`IND_META`), named contractors and projects, checklist, risks, actions, training, report types |
| 15 | Seed generators — 741 contractors, 543 projects, milestones, 643 inspections, findings, 231 corrective actions, participants, results, evidence, log |
| 16 | Calculations — every derived figure (`programmeStats`, `indStatus`, `objectiveStats`, `overallScore`, `healthScore`, `complianceStats`, `caStats`, `notifications`…) |
| 17 | `runConsistencyChecks()` — asserts the brief's figures fall out of the records |
| 18 | UI helpers — `table()`, filters (`F`, `fSelect`…), `kpi()`, `tabs()`, charts, modal, toast |
| 20–28 | Screens, one block per area; each registers `VIEW_RENDERERS[key]` and its `ACTIONS`/`CHANGES`/`SUBMITS` |
| 30–37 | Router, event dispatch, sign-in, period selector, self-test, init |

---

## The rules

### Data integrity — the reason the demo is credible

- **Nothing on a screen is typed.** Every figure is computed from records by a function in §16.
  If the brief quotes a number, it is reached by calibrating the *records*, then asserted in
  `runConsistencyChecks()`. The sidebar chip shows the result; it must read "N data checks passed".
- **Quotas are exact.** Generators use `quota({...})` (a shuffled list with the exact count of each
  value) and calibrate sums with a residual on one record. Never rely on random draws landing near
  a target.
- **The generators are order-dependent.** `seedAll()` reseeds the RNG and runs in a fixed order.
  Adding a `rand()` call anywhere upstream changes everything downstream. After any change to §15,
  run the self-test — checks will tell you what moved.
- **Only verified results reach dashboards** (brief §32). `indValue()` returns the latest
  *Verified* result; anything else shows "Pending Verification". Keep it that way.
- **When you add a record that should appear somewhere, read the query that displays it** and match
  its fields exactly. A silently invisible record costs more than an error.

### Dates

- **All date arithmetic is UTC.** `addDays()` / `daysBetween()` use `T00:00:00Z`. Mixing local midnight
  with `toISOString()` shifts every date a day in UTC+2 (Eswatini) — it pushed inspections out of their
  quarter and froze the closure-date calibration in this build. Use the helpers; never `new Date(iso)` + `toISOString()`.
- Dates are ISO strings throughout; display with `fmtDate()` (`5 Oct 2026`).

### Screens and events

- A screen is `VIEW_RENDERERS[key] = meta => htmlString`. Renderers are pure string-builders; charts are
  queued with `afterRender(() => mountChart(id, cfg))` and mounted by the router.
- **No inline `onclick`.** Use `data-action="name"` and register `ACTIONS[name]`, `CHANGES[name]` or
  `SUBMITS[name]` (declared in §18 so screens can register before the router loads). The only exception
  is the print button inside an exported standalone document.
- **Drill-down:** `go(view, meta, text)`. For a list view, `meta` like `status=Overdue&region=Lubombo` is a
  filter preset, applied to `FILTERS[view]` before render. List views must honour every key other screens
  send; show keys without a control via `activeFilterChips()`.
- **Every write moves something visible and says so.** Mutate `appData`, `addLog()`, `redraw()`, and toast
  the before → after of the figure that moved.
- `esc()` every value that reaches HTML.

### Permissions

`access(key)` → `'full' | 'read' | 'none'` from `ROLE_ACCESS`, deny-by-default per role, admin screens
never inherited. The router enforces it; inside a screen show edit controls only when `canEdit(key)`.
Verification is `canVerify()` (Strategy & M&E, admin). This is a demonstration permission model, not
security — there is no real authentication.

### Design

Tokens in `:root` (`--brand`, `--brand-deep`, `--accent`, semantic `--ok/--warn/--bad/--info`). No colour
literals in components; charts read tokens via `chartTheme()`. Red/amber/green mean status, never decoration.
See `DESIGN_SYSTEM.md`. The palette is **provisional** — replace it when CIC's brand colours are known
(Settings → Branding does it live for a demo).

---

## Testing

```bash
bash scripts/selftest.sh          # every screen × every role + 53 data checks; exit 1 on any failure
```

Any screen opens directly with a hash: `cic_eval360_demo.html#project-detail/CIC-PRJ-2026-001/ceo`
(`view/meta/role`). Useful for screenshots:

```bash
google-chrome --headless=new --window-size=1440,1800 --screenshot=shot.png \
  "file://$PWD/cic_eval360_demo.html#dashboard//ceo"
```

---

## Where this build departs from the brief, and why

The brief's figures are not all mutually consistent. Where two could not both hold, the headline won:

| Brief | Built | Why |
|---|---|---|
| Objective weights 25/25/20/20/10 | **30/25/20/10/15** | With the brief's own objective scores (82/69/76/58/88), its weights give 73%, not the 76% headline. The new weights keep all five scores *and* 76%. Editable in Settings. |
| Compliance: 521 fully / 89 partial / 33 non-compliant | **~500 / ~107 / ~37** (derived) | Fully-compliant share must reproduce the quarterly trend (76% → 79% → 84%); 521 of 643 cannot. Total 643, average 84%, regional averages are exact. |
| Financial Management Training: 197 completions | **192** | Makes Contractor Development's budget-weighted performance the brief's 69% rather than 69.7%. |
| IND-COMP-003 trend 76 / 79 / 82 / 84 | **76 / 79 / 84** | §31 states Q2 = 84% (verified); Q3 has no results yet on 5 October. |
| Health score example 66/100 | **67/100** | The physical-progress component computes to 25/30, not 24/30, under a stated formula. |
| PRG-003 (76%) "On Track" | Needs Attention | The brief's own thresholds (green ≥ 80) make 76% amber. |

Everything else quoted in the brief — 76% overall, E22.3M at 69%, 29 of 38 indicators, 543 projects
(392/104/47, E9.6B, 61% / 64%, 38 more than 30 days late), 741 contractors, 643 inspections at 84%,
74 open / 19 overdue corrective actions at 16 days, the 48 management actions, 4 critical risks,
Sibaya's 81% and +19 points, the training cohort's 135/118/112, 48% → 79%, 81.7% — is derived and checked.

---

## Working agreement

1. Run `scripts/selftest.sh` before every commit. 53/53 and zero failures, or say why not.
2. Change a rule here when the code genuinely needs to differ — do not leave this file describing one
   thing while the demo does another.
3. If CIC confirms real names, figures or colours, they replace the illustrative ones — and the
   "Demonstration Environment" labels stay until CIC says the data is theirs.
