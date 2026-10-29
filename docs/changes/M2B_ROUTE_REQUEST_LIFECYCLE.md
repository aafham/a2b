# Route request lifecycle

Based on upstream `134045a`, refreshed with the open PR list on 29 October 2026. This branch changes the existing website; it does not include M1 account changes or PR #15.

## Problem and changes

- Route previews previously checked only destination object and travel mode. Two requests for the same destination/mode but different origins or preferences could arrive out of order. Driving reroutes checked only whether any drive was active, so a result from drive A could replace drive B.
- Each preview now has its own request identity and copied inputs: start, destination, travel mode, routing preferences, heading and stops where relevant. Replacing or closing a preview cancels the old request and invalidates its success/error UI.
- Driving and train trips have a generation that changes at start/end. Reroutes/replans own their busy state, result and error. An older request's `finally` cannot unlock a newer request. A forced reroute (for example adding a stop or changing route preferences) supersedes the previous one.
- Train preview and Leave later requests have the same ownership checks. Editing a departure time invalidates the old request immediately during debounce; closing the panel cancels its timer/request. Arrive-by retries use a snapshot and share one total deadline.
- API calls can opt into an AbortSignal and timeout. Route/transit calls use 30 seconds, including response-body reading. Cancellation and timeout are distinct from offline errors; callers settle even if a transport ignores abort. Existing API callers have no new default timeout.
- Changing routing preferences or units refreshes the current preview, or requests a new route during a drive, using the updated settings.
- A preview replacement immediately stops its auto-start countdown. Each timer tick belongs to both its preview and its timer; a queued old tick cannot start a drive or stop a newer countdown. Closure fallback and unknown closure status still require an explicit start, including when resuming a saved drive.
- The service worker caches the new lifecycle module and advances its shell version to `a2b-v39`.

## Validation

- `npm test`: **188 passed**, including nine new request-lifecycle tests. `git diff --check` and JavaScript syntax checks passed.
- Deferred transport tests execute the production functions from `app.js` with DOM, map and transport stubs. The fake transport deliberately ignores abort. They cover B resolving before A for the same destination with changed origin/preferences; stale transit errors; closing previews; real `endDrive`/`startDrive` calls between old/new reroutes; cancelled/timeout ownership of locks; train replans; closing Leave later; and queued countdown callbacks.
- API tests cover a stalled response body, external cancellation and an already-aborted signal. Timeout remains distinguishable from an offline error.
- Browser check in Edge against a local synthetic API: requested slow Drive then fast Walk; the server returned Walk before Drive, and the Walk preview remained selected after the delayed Drive response. Closure warning remained visible with explicit Go now. No routing provider was called by this fixture; no road test was performed. Browser error log was empty.

## Boundaries and integration

- This bounds the browser wait and cancels the request; it does **not** guarantee that the server/provider stops work after a disconnect. The server's optional route enrichers currently mutate route arrays during concurrent/sequential work. A separate change should collect immutable enrichment results under a shared deadline before applying them, with explicit fallbacks and closure/construction regression tests. That backend deadline is deferred rather than claimed complete here.
- Upstream's offline shell also omits existing `route-preview.js` and `probe.js`. Those are handled by the separate M3 shell/import-graph change, not silently included here. Apply both before offline acceptance, preserving the new lifecycle module in the combined shell.
- PR #11 also proposes API AbortSignal support; its final integration should retain this timeout/error contract. Open route PRs such as #27 (entrances) and #29 (faster routes) touch route callers and must adapt to the snapshot-based `requestRoutes(body, signal)` signature when rebased. No code from those PRs is included.
- Browser cancellation uses the standard [AbortController API](https://developer.mozilla.org/en-US/docs/Web/API/AbortController/abort). Ownership checks remain necessary because cancelling transport alone cannot retract an already-completed result.
- No framework, dependencies, production configuration, deployment or CLA changes.
