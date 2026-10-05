# CIC Performance & M&E Management System — Eval360 demonstration

A demonstration of Eval360 configured for the **Construction Industry Council – Eswatini**:
strategy, programmes, contractors, construction projects, inspections, corrective actions,
training, budgets, risks and management actions in one institutional view.

**All data is illustrative.** People, contractors, projects and figures are fictional.

## Open it

Open `index.html` (or `cic_eval360_demo.html` directly) in a browser — straight from disk or from
any static web server. No install, no build. Sign in with any of the nine demo accounts.

For a presentation, start at **Overview → Demo Walkthrough**: the twelve-step sequence for senior
management, plus four live moments (verify a result, close a corrective action, record project
progress, see the read-only Council view).

## Check it

```bash
bash scripts/selftest.sh
```

Renders every screen as every role and asserts 53 data checks — that the brief's headline figures
fall out of the underlying records. Needs Chrome/Chromium and Python 3.

## Read before changing it

- **[CLAUDE.md](CLAUDE.md)** — conventions, the data-integrity rules, and where the build departs from the brief
- **[DESIGN_SYSTEM.md](DESIGN_SYSTEM.md)** — tokens, typography, components
- **[docs/DEPLOY.md](docs/DEPLOY.md)** — GitHub and server deployment
