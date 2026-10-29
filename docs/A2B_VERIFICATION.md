# A2B verification record

## Baseline — 29 October 2026

- Repository: `thaqif0709/a2b`, upstream main `134045abe40e3fd50f73fb24437c4ef4a2920509`.
- Node 24.22.1; npm 12.2.1; Windows/PowerShell; repository requires Node >=22.13.
- `git fetch upstream --prune`: exit 0. No reset/clean/overwrite of existing user branch.
- `npm test`: exit 0, **179 pass / 0 fail / 0 skip**; about 1.7 seconds on this machine.
- Existing unit suite needs only built-in Node modules; dependency installs use the committed lockfile
  for Wrangler and any added browser test tooling. No generated Overture download was needed.
- PR inventory: 40 total at audit, status/diffs/discussion checked. #14/#16 merged; #15 open and
  clean at `474f670b624bcfdee360aeb418060efc7d924896`. Historical 183-pass result is not this baseline.

## Evidence and limits

Each implementation PR must include its own `docs/change-notes/` entry with exact base/result commit,
commands, exit status, test counts, reproduction and known limits. Results from independent branches
must not be added together as though they were one integrated build.

| Area | Evidence at baseline | Outstanding |
|---|---|---|
| Account access | Synthetic unverified allowlisted signup reproduced privileged stats 200; new regression failed before fix. | Verify owner/tester ID provisioning with owner; no real IDs inferred. |
| Recovery | Code read and claim are separate; local concurrency and fault tests under construction. | D1 runtime batch validation, replacement-code browser journey. |
| GPS estimates | Current expiry recreates reckoner; estimated fix can latch arrival. | Deterministic actual app flow and real GPS acceptance tests. |
| Sensitive UI | Origin name uses innerHTML; password uses prompt. | DOM injection, masked forms, failure/pending/focus in Chromium and WebKit. |
| Full app UI | Current code inspected; no completed end-to-end baseline browser journey recorded here. | Actual map/search/drive/train/share/offline/reconnect and deep-link journeys. |
| Performance | Unit-suite elapsed time only; not an app performance metric. | Repeatable first-paint/latency/request/long-task/data measurements. |
| Production | No writes, security/load tests, migrations or deploys performed. | Maintainer review and own deployment process after approval. |

Browser component fixtures use real HTML/CSS/modules, synthetic accounts and local mocked requests;
they do not establish full map navigation correctness. Traces/screenshots must never capture actual
passwords, recovery codes, tokens, feedback content or personal locations. Physical iPhone/Android,
screen readers, real software keyboards and on-road GPS remain manual until explicitly recorded.

No claims of production readiness for all milestones, privacy-law compliance, permanent offline cache,
hard bandwidth cap or measured speed-up are supported by this baseline.

## M1 branch results

| Branch result | Commands / exit codes | Result / actual platform |
|---|---|---|
| Access `99ed819` | `npm test` 0; `git diff --check` 0 | 180 pass/0 fail; Chrome component smoke with real Worker + SQLite fixture |
| Recovery `f397fc2` (code `a366f8d`) | `npm test` 0; `git diff --check` 0; local D1 evidence harness 0 | 185 pass/0 fail; 1 winner and rollback/retry at 3 fault points on Wrangler local D1; synthetic browser recovery |
| GPS `8895c07` | `npm test` 0; `git diff --check` 0 | 189 pass/0 fail; actual app functions in mock GPS/DOM/clock fixtures; no on-road/browser map proof |
| UI `ecf1ba7` | `npm ci` 0; `npm test` 0; `npm run test:browser` 0; `node --check public/app.js` and `public/panels.js` 0; diff 0 | 179 pass/0 fail unit/API, 10 pass browser component scenarios across Chromium/WebKit; real 390px day/night renders, widths320–1280, landscape, text enlargement |

Each suite has zero skipped tests. Counts are per branch, not additive. M1-UI Playwright1.64.1 initial
launch failed because its browser executables were not installed (8 launch failures, no app assertions
executed); pinning1.62.1 to the available runtime produced the reported successful run. Initial4-case
per-browser run passed8, then review added close/late-failure coverage and the final run passed10.
Chromium screenshot capture reports151.0.7922.34. WebKit emulation is not physical iPhone evidence.

Local recovery D1 command, using the project's installed Wrangler4.151.3:

```sh
npx wrangler dev --local --ip 127.0.0.1 --port 8792 --persist-to .wrangler/recovery-test --log-level warn
```

The synthetic evidence harness invoked against this local server reported four PASS lines: concurrent
winner/loser/session state, then password-update, session-delete and session-insert fault rollback/retry.
The harness was a local audit artifact; persisted regression tests are `test/recovery.test.js` in #43.
No remote D1 binding was executed. One local SQLITE_BUSY from overlapping CLI access was retried with
a bound; this was test-harness contention, not a discarded application assertion.

UI screenshots and precise fixture scope are in #45 `docs/changes/SAFE_ACCOUNT_FORMS.md`.
The npm audit result has3 inherited Wrangler-chain advisories (2 moderate,1 high), also observed on
the recovery branch without Playwright. A dependency update belongs to a separate reviewed change.
