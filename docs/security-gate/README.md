# Security gate: allow a new workflow that the linked issue names verbatim

## Why

`🛡️ PR Security Gate / 🚦 Security Gate Decision` rejects every PR that touches
`.github/` (`scripts/ci/issue_compliance.js`, the `forbiddenFiles` loop), even when the
linked sprint issue explicitly assigns that workflow as its **Primary Affected File**.

This makes DevOps issues unmergeable through a normal contributor PR — including
#160 ("automated release tagging and semantic changelog generator"), whose only
deliverable is `.github/workflows/release.yml`. The check is required, so no such PR
can ever land.

The gate is safe to run from `pull_request_target` with base-branch scripts, so the
contributor cannot carry this fix in their own PR; it has to land on `develop`.

## What the patch changes

It only widens one rule: a **newly added** file under `.github/workflows/` is accepted
when its exact path appears inside the body of an issue linked by the PR. Everything
else stays blocked:

- editing existing workflows (including `pr-security-gate.yml`) — blocked
- `scripts/ci/*` — blocked
- a new workflow whose path is not in the linked issue — blocked
- `.github` files that are not workflows (e.g. `CODEOWNERS`) — blocked
- assignment / closed-issue / max-4-issues checks — unchanged

## How to apply

```bash
cd soroban-anchor-gate-app
git checkout develop && git pull
git apply docs/security-gate/compliance-patch.diff   # or: patch -p1 < .../compliance-patch.diff
git commit -m "fix(ci): allow issue-assigned new workflows in issue compliance gate"
git push origin develop
```

Then re-run the failing check on the contributor PR (push any commit, or use
"Re-run jobs"). No contributor-side change is needed.
