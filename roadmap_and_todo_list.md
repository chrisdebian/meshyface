# meshyface — Roadmap and To-Do List

**Project:** chrisdebian fork of jaronmcd/meshyface — external contribution tracker
**Purpose:** Track contribution work; document tech debt assessed during triage
**Last updated:** 2026-07-05

## External contribution rules

- **No unsolicited bulk issues/PRs**: raise only when we have a concrete fix or meaningful analysis, not every gap found.
- **Don't stack submissions**: wait for a response on an open PR before opening another.
- **Follow maintainer's lead**: jaronmcd is sole maintainer, ships fast, responsive and welcoming — match his pace but don't crowd the queue.

---

## PR History

No PRs currently open.

| PR | Title | Status |
|---|---|---|
| #40 | fix: mark non-cryptographic SHA1 ID generation as usedforsecurity=False | Merged 2026-07-05 (closed #38) |
| #41 | fix: subscribe to meshtastic.receive before opening the radio interface | Merged 2026-07-08 (closed #37) |

---

## Watching, not acting

- **Issue #36 — "any ideas for GUI layout improvements?"** *(reviewed 2026-07-05)* — an open-ended design conversation between jaronmcd and a user (lolife), not a call for outside contributions. Read the actual dashboard code (`meshdash/assets/dashboard.html.tmpl`, ~4345 lines, plus ~154 modular JS/CSS template fragments) to ground the assessment: the dashboard uses a single-view-at-a-time model (a `layout-view-menu` dropdown switches between Chat/Network/Console/Apps/Settings), so lolife's "everything on one page" ask is a real architectural change, not a small fix. jaronmcd's reply says he's already "playing with" expanding node details into a bigger view — confirmed there's already a `chat-node-details-drawer` with tabs (messages/details/telemetry/history/location/chat/links/notes), so he's actively building toward this himself. Decision: don't act — he hasn't asked for help here, he's mid-build on it, and we already have two unreviewed PRs (#40, #41) sitting with him. Revisit only if he explicitly invites contributions on this thread.

## To do

- [ ] **Add `pip-audit` to CI** *(found 2026-07-03, during security audit)* — `pip-audit` came back clean locally against the three pinned dependencies (`meshtastic`, `pypubsub`, `protobuf`), but it isn't wired into `.github/workflows/ci.yml` — only ruff, pytest, coverage, and a Docker smoke test run there today. Small, low-risk addition: a new step or job installing `pip-audit` and running it against `requirements.txt`. PRs #40 and #41 are both now merged (2026-07-05, 2026-07-08), so the earlier "don't stack a second unreviewed PR" hold no longer applies — free to raise when picked up.
- [ ] **Test coverage gaps** *(from 2026-06-23 evaluation — recheck before acting, several PRs have landed since; coverage numbers below may be stale)*:
  - `meshdash/api_system_restart.py` — was 23.3% coverage, 43 statements; biggest proportional gap
  - `meshdash/api_system_update.py` — was 74.0%, 141 missing statements; biggest absolute gap
  - `meshdash/history_raw_packets.py` — was 73.4%, 79 missing statements

---

## Technical Debt

Scan command (comment-only pattern, avoids Spanish false positives):
`grep -rn -E '^\s*#\s*(TODO|FIXME|HACK|WORKAROUND|XXX)' meshdash/ mesh_dashboard.py mesh_connection.py`

**2026-07-02 scan:** zero markers found anywhere in the Python source — clean.

### Security audit (2026-07-03)

Ran `pip-audit` (dependencies) and `bandit` (static analysis) against the full codebase.

- **`pip-audit`**: no known vulnerabilities in the three pinned dependencies.
- **`bandit`**: 156 low-severity (mostly benign `except: pass` in event-loop/callback code — too numerous and low-value to raise), 26 medium-severity (all `B608` SQL-injection warnings — checked several, all false positives: values are properly parameterised via `?` placeholders, only trusted literal clause fragments are f-string-interpolated) or `B104` binding-to-all-interfaces (intentional — this is a self-hosted LAN dashboard), 2 high-severity `B324` weak-SHA1 findings — **fixed via PR #40** (non-cryptographic ID generation, not a real vulnerability, but worth silencing correctly with `usedforsecurity=False`).
- **CI gap**: `pip-audit` not yet wired into CI — see To Do above.
