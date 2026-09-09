# Guide: using rhizome-mcp from omp via direct MCP

Audience: future omp agent sessions working in any project that uses rhizome-mcp. Follow this runbook; do not re-derive the pattern. When working rhizome-mcp issues, claim the issue before engaging with its content in any way — see Order of operations.

**Every rhizome read and write goes through the MCP tools** (`xd://mcp__rhizome_mcp_*`): omp delivers the authoritative `structuredContent` payload. Use it directly, never parse ack text. Default responses are compact, bounded projections (free-text bodies omitted); pass `view: "full"` when you need them. Large collections paginate with `has_more`/`next_cursor` (default limit 20, max 100).

## Start: open the project

Call `open_project` with the absolute project root before anything else; it is the only projectless tool, and it returns the `project_ref` plus project metadata, limits, supported types/statuses/priorities, and guide links. Retain the `project_ref` and pass it to every subsequent project-scoped tool call; routing is stateless (omission works only against a server started with a configured default project). Use `get_project` for a metadata refresh (`next_issue_number`, supported values) — it is not a state snapshot. (`rhizome://guides/agent-workflow` is the companion workflow reference.)

## Order of operations: claim before any work

When you are asked to work an issue — or decide to — the server state moves first, and the issue's content is off-limits until the claim:

1. **State check only.** `get_issue` on ISSUE-N; read `status` and `blocked_reason`. Claimability fields (`is_claimable`) live in `list_issues` output, not in `get_issue` — the claim call itself is the authoritative gate. This state check answers exactly one question: can it be claimed now, and if not, what server transition (if any) is authorized. Do not read the description as a work order, do not analyze the problem, do not check anything the issue mentions, do not plan or draft. All of that is work, and work starts after the claim.
2. **Transitions only on explicit authorization.** A `blocked` issue is unblocked (`update_issue` with `changes: {"status": "ready", "blocked_reason": null}` and `expected_version` from the same `get_issue`) — and an `open` issue moved to `ready` — only when the user's instruction textually authorizes it: the words themselves must say to unblock or make ready ("unblock ISSUE-N", "make ISSUE-N ready"). "Let's work on ISSUE-N" does **not** authorize a transition on a non-claimable issue; it is a request that cannot be fulfilled yet. The protocol then **ends for this turn**: ask exactly one question — "ISSUE-N is blocked (`<blocked_reason>`). Unblock it?" — and stop. No description reading, no content thinking, no reachability checks, no lease reasoning until the user authorizes, the transition is applied, and the issue is claimable.
3. **Claim immediately once claimable.** `claim_issue` with an `idempotency_key` (e.g. `claim-ISSUE-N-<date>`). The result carries `attempt_id`, the attempt `kind` (`work`, or `review` for an issue bound to an active review request), `lease_expires_at`, and `lease_token` — keep all of them for the life of the attempt; the token is stored only hashed server-side and cannot be re-derived. Keyed retries of the same claim on a live attempt rotate the lease and return a fresh token; the same key after the attempt died returns `ATTEMPT_NOT_ACTIVE`; a different request reusing a key returns `IDEMPOTENCY_CONFLICT`. Work includes thinking about the issue's content: reading the description as a work order, feasibility and reachability checks, approach design, drafting instructions — none of it happens before the claim result is in hand. The claim is what marks the issue in-progress server-side: attribution, checkpoints, and attempt records exist only for claimed work, so pre-claim analysis is invisible to the process.
4. **Then work in small steps**, checkpointing findings and next steps with `save_attempt_note` (use `kind: "checkpoint"` for restartable state — `get_work_context` surfaces the latest checkpoint) so any interruption is restartable; `renew_attempt` per the lease-discipline rules below; `finish_attempt` when done. If you must stop mid-attempt — including waiting on the user — leave a `save_attempt_note` handoff and `finish_attempt` with `outcome: "interrupted"`, `interruption_reason_code: "handoff"`; the issue returns to `ready` for the resuming attempt. Choosing between renewing through a pause and handing off: renew — with `lease_seconds` sized to the expected pause (60-3600 s, default 900) — when you expect to resume shortly; hand off when the wait is long or indefinite.

**Lease discipline while an attempt is active.** The remaining lease is visible in the claim result (`attempt.lease_expires_at`) and in every `renew_attempt` result (`lease_expires_at` alongside `server_time`); regardless, never estimate it — follow deterministic rules instead:

- **Renew on every resumption.** Whenever the attempt is active and you are about to work again after any pause — the user replied, a tool call was interrupted, unknown wall-clock time passed — call `renew_attempt` first, before anything else. It needs only `attempt_id` + `lease_token`, it pushes the expiry out, and it costs one call; deliberating about remaining time is always more expensive than renewing.
- **Renew before pausing too.** About to hand the turn to the user and expecting to resume this attempt? Renew immediately before pausing, sizing `lease_seconds` to the expected pause (60-3600 s, default 900).
- **On `LEASE_EXPIRED` or `ATTEMPT_NOT_ACTIVE` from any attempt call, the lease lapsed anyway.** Re-claim with a NEW idempotency key (the old key only replays the dead attempt with `ATTEMPT_NOT_ACTIVE`), take the fresh `attempt_id` + `lease_token` from the result, then `save_attempt_note` to re-anchor the checkpoint before continuing.

**Switching from one issue to another.** The same discipline governs the seam between issues. With current issue X and next issue Y:

1. Y is off-limits the moment a transition is on the table: no reading Y, no thinking about Y, no comparing approaches — even while X's wrap-up calls are in flight.
2. Wrap up X first, in the server: `save_attempt_note` if anything needs carrying forward, then `finish_attempt` exactly once with the truthful outcome — `completed` with `target_issue_status: "done"` when X's work is complete and accepted (the close a transition expects), or `outcome: "interrupted"` + `interruption_reason_code: "handoff"` when the remaining work genuinely moves elsewhere. Never leave the lease dangling, and never finish with a status the work has not earned.
3. The `finish_attempt` result is the boundary. Until it, you are still on X; after it, X gets no further thought or work — leftovers belong in X's record (a comment or note), not in Y's attempt.
4. Only then start Y under the steps above: state check only, authorized transition, claim, work.

## Write discipline (MCP tools)

- Every mutating call carries an `idempotency_key` (e.g. `<action>-<issue>-<date>`). Keyed retries replay the same server-side result instead of duplicating work; a key reused with a *different* request fails with `IDEMPOTENCY_CONFLICT`.
- `update_issue` requires `expected_version`. Read it from the `get_issue` result (`issue.version`) or from a `list_issues` compact item (also carries `version`). On `VERSION_CONFLICT`: refetch, reconcile, retry.
- `finish_attempt` always requires `result_summary`. Which other fields are required depends on the attempt `kind` and `outcome`: a completed **work** attempt requires `target_issue_status` (`done`/`review`/`ready`/`blocked`); a completed **review** attempt requires `review_outcome` (`approved`/`changes_requested`/`blocked`); `failure_reason_code` is required when `outcome: "failed"`; `interruption_reason_code` when `outcome: "interrupted"`. Each of these is forbidden in the other combinations.
- `save_attempt_note` retries need an `idempotency_key` too — bare repeats append duplicate notes.
- Status transitions via direct `update_issue`: `open` to `ready`, `ready` to `blocked` (include `blocked_reason`), `blocked` back to `ready`. `review` and `done` are reachable ONLY through `claim_issue` then `finish_attempt` (set `target_issue_status` there); a direct update to either fails with `INVALID_STATUS_TRANSITION` — for every issue type, epics included (an `epic` alone may be closed to `done` by direct patch from `open`/`ready`). `cancelled` is likewise a direct patch target (fails with `ACTIVE_ATTEMPT_EXISTS` while an attempt is active). Anything else — e.g. `open` to `blocked` — is not covered by this runbook; check the server docs before relying on it.
- Sub-issues require an `epic` parent: `INVALID_EPIC_PARENT` means convert the parent with `update_issue` (`changes: {"type": "epic"}`) first.
- New labels: pass `create_missing_labels: true` on `create_issue`/`update_issue`, or you get `LABEL_NOT_FOUND`.
- Only one active attempt per issue is allowed. Finish what you claim (`finish_attempt`); never leave a lease dangling. A lease runs 60-3600 s (`lease_seconds` on `claim_issue`/`renew_attempt`; default 900).

## Reads (MCP tools, full JSON)

All of these return the authoritative payload directly:

| Tool | Use |
|---|---|
| `get_issue` | One issue; standard view has `status`/`version`/`blocked_reason` but no bodies — `view: "full"` adds description and acceptance criteria |
| `list_issues` | Filtered lists with effective status, blockers, and `is_claimable`; compact items carry `version` for list-then-mutate flows |
| `get_planning_graph` | Dependency-aware entry points and blocking nodes |
| `search` | Full-text search with cursor pagination |
| `list_decisions` | Project-wide or issue-scoped decision records |
| `get_issue_graph` | Bounded relation/hierarchy graph around one issue |
| `get_issue_activity` | Unified newest-first timeline for one issue |
| `get_work_context` | Bounded resumption context for an issue; surfaces the latest checkpoint note (optional sections: comments, attempt history, relations, decisions) |

Health check remains CLI-only (no doctor MCP tool): `rhizome-mcp doctor --format json` from the project root should report `healthy: true` — including wal journal mode, schema_version current, `one_active_attempt_per_issue ok`, and `quick_check ok`. The CLI's issue commands are read-only (`issue list`, `issue show`); do not look for CLI issue mutations.

## Quick reference

| Task | How |
|---|---|
| Session start | `open_project` (absolute root); retain `project_ref` for every later call |
| Read issues, decisions, search | MCP read tools (table above) |
| Create/update/close issues, comments, relations, labels | MCP write tools with `idempotency_key` |
| Start work on a ready issue | `claim_issue` — claim FIRST, before any analysis of the issue; token comes back in the result |
| Switch from issue X to issue Y | wrap X in the server first (note + `finish_attempt`, truthful outcome), then claim-before-work for Y — no Y engagement during X's wrap-up |
| Finish work | `finish_attempt` with `attempt_id` + `lease_token` + `outcome` (`completed`/`failed`/`interrupted`) + `result_summary`; completed work attempts add `target_issue_status`, completed review attempts `review_outcome`, failures `failure_reason_code`, interruptions `interruption_reason_code`; artifacts use `{type, uri}` |
| Note progress during an attempt | `save_attempt_note` with `attempt_id` + `lease_token`; `kind: "checkpoint"` for restartable state |
| Renew the lease | `renew_attempt` with `attempt_id` + `lease_token` — renew on every resumption and before pausing; never estimate remaining time |
| Check system health | `rhizome-mcp doctor --format json` (CLI) |
| Recover a stuck lease | `rhizome-mcp maintenance release-attempt ATTEMPT-ID` (last resort; loses outcome) |

## Pitfalls

- Retain and pass `project_ref` on every project-scoped call; routing is stateless.
- The claim-before-work rule covers *thinking* about an issue, not just touching it: no description reads, no approach design, no reachability checks before the claim result is in hand.
- Never fabricate a `lease_token` or reuse a dead attempt's; after `LEASE_EXPIRED`, re-claim with a new idempotency key.
- A claim can also fail with `WORKFLOW_GATE_UNSATISFIED` (an unmet workflow-policy requirement) before any attempt is created — resolve the gate before claiming.
- If the issue changed during your attempt, `finish_attempt` may require `acknowledged_changes` (`issue_version` + `latest_event_id` from fresh reads) to proceed.
- Gitignored paths are refused by the mcp-workspace server (it enforces `.gitignore`); use built-in file tools there and say so when you do. Check with `git check-ignore -v <path>`.
