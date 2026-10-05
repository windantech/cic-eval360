# CIC Eval360 — GitHub and deployment

Static files only. Proposed address: **https://mel.eval360.tech/cic/**, alongside the FPC build at
`/fpc` — same document root, same pattern:

```
/home/vantage/domains/eval360.tech/public_html/mel/
├── atbu/
├── fpc/
└── cic/     → mel.eval360.tech/cic   (this project)
```

## 1 · Create the GitHub repository

Create `windantech/cic-eval360` on GitHub first (private is fine), then from this machine:

```bash
cd ~/cic-eval360
git remote add origin https://github.com/windantech/cic-eval360.git
git push -u origin main
```

## 2 · First deployment on the server

```bash
cd /home/vantage/domains/eval360.tech/public_html/mel
git clone https://USERNAME:TOKEN@github.com/windantech/cic-eval360.git cic
cd cic
bash scripts/deploy_pull.sh
```

The token is stored in the clone's `.git/config`; treat it as a secret.

## 3 · Every deployment after that

```bash
bash scripts/selftest.sh            # locally — must pass
git add -A && git commit -m "…" && git push
```

On the server:

```bash
cd /home/vantage/domains/eval360.tech/public_html/mel/cic
bash scripts/deploy_pull.sh
```

The script fetches, resets to `origin/main`, prints what changed and fixes file modes. It never runs
`git clean`.

## Notes

- Everything is relative, so the same build works at `/cic/`, at another path, or opened from disk.
- The page is marked `noindex`. It is a demonstration; do not link it publicly without CIC's agreement.
