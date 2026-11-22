# Acceptance audit — 2026-11-22

Acceptance criteria come only from this repository's [ACCEPTANCE.md](../ACCEPTANCE.md). Final audit; current evidence is local, not a production-deployment claim.

| Item | Source rule | Evidence | Disposition |
|---|---|---|---|
| PR #1 translation lacks required closer | §8 | Historical body rejected by current checker; #2 passes | Not fixable after merge by a new branch. New PR includes a complete closer. |
| PR #1 has unchecked QA plan, no required mobile/interaction artifacts | §§2,4 | Original body and returned issue comments lack delivered 390×844 screenshots and interaction proof | Fixed prospectively: current Chromium screenshots of index and all seven module views, quiz selection and animated WebP walkthrough. Historical merge-time gap remains. |
| Missing product-status line | §5 | Generic deployment directions did not establish LIVE / BLOCKED / LOCAL ONLY plus next action | Fixed in README: LOCAL ONLY and review/hosting next human action. |

## Existing work

Baseline `6a4ff02e2d221a06b1a4ebc088bbe64b557821b8`; returned PRs #1 translation and #2 acceptance infrastructure were merged, with merge SHAs passing ancestry checks against origin/main. Reviewed README, briefs, base/footer fragments, seven modules, generated index, JS, CSS and build script. This is educational HTML, not validator software. Acceptance-only CI successes do not validate the course UI. GitHub deployments were empty and Pages returned HTTP 404; unknown external hosting remains possible.

## Current checks and artifacts

- `bash build.sh`: `Built index.html — open it in your browser.`
- `git diff --exit-code -- index.html`: exit 0, generated file unchanged. No module source edited.
- `node --check main.js`: exit 0. No product unit-test suite/linter is configured.
- Playwright Chromium 390×844: navigated all seven module dots, asserted the selected module enters the viewport, selected the first quiz option. Stills per module plus index and `interaction.webp` at `/home/ubuntu/audits/shots/solana-speed-demons/`, embedded in the PR body. Viewport stills are not full-page captures of every paragraph.
- Final PR body: `python scripts/check_acceptance.py <body>`; result recorded in the PR body.

## Coverage of all acceptance sections

- §1: minimal documentation-only changes following inspection; historical private plan/approval chronology is not assessable.
- §2: executed build/syntax/browser checks and external artifacts; no claim of exhaustive translation correctness. Tooltips, every definition and quiz scoring were not exhaustively tested.
- §3: no bug-fix episode established in translation/process PRs; no new product bug fix in this branch.
- §4: current seven-module navigation/stills/animation delivered; original merge proof cannot be supplied retroactively.
- §5: LOCAL ONLY with next human action; no deployment or merge of this branch claimed. Historical ancestry checked; private approval/duplicate-intent history unavailable.
- §6: README commands and repository acceptance pointers reviewed; no unrelated quality criteria imported.
- §7: no historical blocker/bypass episode established; deployment is not silently assumed.
- §8: #1 historical failure retained; #2 passes; new PR uses the required closer.

Not fixable after merge: historical PR #1's missing contemporaneous evidence and closer. Not checked: production hosting, external linked essay, exhaustive course-content accuracy. No new test framework or product feature introduced.
