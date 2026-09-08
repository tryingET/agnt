# RFC: native Ask and Interview for Annie

Status: **DRAFT — design only, no runtime behavior implemented**.
Date: 2026-09-08.

## Problem

Chat prose can ask a question, but it does not reliably represent a pending decision across tabs, devices, restarts, and concurrent tool execution. A visual form alone does not ensure that work waits for an answer, that an answer reaches the correct execution, or that an unsubmitted default is not mistaken for consent.

## Proposal

One native human-input service with two views:

- **Ask**: a compact inline conversation card for clarification, review dispositions, and explicit human handoffs.
- **Interview**: an expanded multi-question view of the **same request**, not a second request system.

Working model-facing name: `ask_user`. Keep the schema discoverable and small. Use the existing database, conversation identity, tool execution, native chat components, and provider adapters. Do not install a Pi runtime, add a per-form HTTP server, introduce a second conversation store, or make this a general-purpose authorization engine.

A future pending-input inbox would be a projection of the same authoritative request store.

## Scope of the first implementation

- Single-select, multi-select, text, and informational questions.
- Stable question and option IDs; bounded labels, text, counts and evidence.
- Clarification, review, and handoff request kinds.
- Explicit `wait` and `defer` delivery, with handoffs requiring wait.
- Durable drafts distinct from accepted submissions.
- Authenticated owner/context binding, revision checks, idempotency and delivery receipts.
- Native accessible Ask/Interview UI, safe evidence rendering, refresh/restart recovery.
- Host-enforced scheduling: no dependent action before a gating answer.

### Not included

Camera, arbitrary HTML, AI-generated options, option-side conversations, bulk permissions, automatic recommendations-as-consent, autonomous memory promotion, a new fleet manager, or general tool-authorization policy. Attachments can follow after the core lifecycle is proven.

## Existing capabilities to reuse

A source-based audit identified useful native seams:

| Existing seam | Reuse | Do not assume |
|---|---|---|
| `UnifiedChatContainer.vue` / `MessageItem.vue` | Native conversation rendering | One tested surface proves all surfaces |
| Orchestrator steering and provider adapters | Correct user-answer provenance and valid history shaping | In-memory strings are a durable request store |
| Active run registry | Reconnect and explicit cancellation | Socket lifetime equals task lifetime |
| Run journal | Preserve work already produced | Restart recovery resumes generation or safely replays tools |
| Async tool execution | Existing context and event paths | An in-memory execution map is durable human waiting |
| Goal review | Reuse established task/goal ownership where appropriate | Goal acceptance equals universal per-tool permission |

These are source findings, not live capability guarantees. The running installation had local changes; this draft is based on a clean fork main at `535e136c2217a1e694c71634389baca71daf1a49`. Reconfirm integration seams on the implementation target instead of importing unrelated local changes.

## Contract sketch

`ask_user` input:

- `kind`: clarification / review / handoff
- `delivery`: wait / defer
- `presentation`: compact / interview
- `title` and bounded `questions`
- questions: stable `id`, type, text, requiredness, choices with stable IDs, optional other-text/comment and inert evidence references
- handoff: expected completion signal

Identity is host-derived: user, conversation, execution, originating tool call, surface, and optional task. The model must not nominate another owner's identity.

See [example-request.json](example-request.json) for an illustrative request, **not a production-validated schema**.

## Lifecycle and invariants

Keep request state and delivery state separate:

| Request state | Meaning |
|---|---|
| pending | Open for draft or submission |
| submitted | A valid human submission was durably accepted |
| dismissed | Human dismissed the request without inventing an answer |
| expired | Deadline passed; no implied permission |
| cancelled | Explicit cancellation |
| superseded | A later request/revision replaces this one |

Delivery: `not_queued`, `queued`, `applied`, `failed`, `unknown`.

1. Persist the immutable request definition and revision before announcing it.
2. Save drafts with optimistic revision checks. A draft does not advance execution.
3. Accept a submission atomically with a receipt and delivery intent. A retry of the same submission returns the same receipt. A conflicting second submission must not be silently accepted.
4. Deliver a correlated **user-input** event to the exact context at a safe boundary. Do not turn human responses, evidence, or subagent reports into developer policy.
5. Track acceptance separately from consumption. Never label uncertain delivery as completed.
6. Recover pending/accepted state after restart without replaying prior external effects.
7. Keep recommendation, explicit choice, dismissal, expiry, hiding UI, cancelling a request, and stopping work distinct.
8. A handoff completion is the user's report of completion, not automatic verification of an external condition.

### Wait without a held worker

Return a proper pending tool result, durably mark the logical task as waiting, and yield execution resources. Later, a correlated answer can start one guarded continuation. Do not retain a provider stream or worker promise for minutes while the user answers; do not leave malformed unmatched provider tool calls.

### Scheduler barrier

A waiting form is not a scheduling barrier. Tool calls can arrive incrementally, and one may be dispatched before another announces a question. The implementation must prove that gating questions prevent dependent consequential calls in **either emission order**.

For the interactive path, stage consequential or unclassified calls until full-round gating is known. Only demonstrably independent safe reads may retain eager dispatch. Reject incompatible async/background execution for the interaction tool. Test hidden/nested invocation routes before enabling them; the experimental typed-tool path should not be expanded to waiting calls without a suspend/resume contract.

### Evidence and consent

Use native controls and inert evidence. No arbitrary executable HTML. Validate normal answers, partial drafts, cancellations and timeout payloads consistently; reject unknown or duplicate question IDs and option IDs.

Display recommendations as suggestions, never preselected consent. Changed wording/options require a new revision and visible stale-answer handling. Generic instructions such as 'use your judgment on timeout' must not authorize actions.

## UX outline

- Ask card lives in the originating conversation, with question, evidence expander, unselected choices, other-text, comment, explicit Submit and Dismiss controls.
- Open Interview expands the same request into a roomier view with question progress, evidence, drafts and keyboard navigation.
- Show precise statuses: Draft saved; Answer submitted; Delivery pending; Answer applied; Delivery uncertain.
- A waiting badge explains which work is blocked. Defer means independent work may continue, not broader permission.
- Avoid asking for facts inspectable from the environment; batch independent decisions to reduce interruption.

Any accompanying image is a concept, not implementation evidence. Raw user screenshots, private conversation text, local paths and provider account details must not be committed.

## Source-to-native mapping

| Reference | Adopt | Native owner / intentional difference |
|---|---|---|
| Howaboua `pi-ask` coordinator and pending reducer | wait/defer, serialized presentation, stale-session protection | Native human-input service with durable acceptance/delivery receipts |
| `pi-interview` | rich multi-question UI, context, comments, drafts | Shared Vue views; no separate browser server, arbitrary HTML, or inferred consent |
| Owned `pi-interaction` bridge | exact target and stale revision/digest rejection | Native request revision checks, not terminal focus/socket infrastructure |
| `pi-evidence-review` | bounded inert evidence | Separate evidence display from decision authority |
| `pi-peer-messaging` | correlated delivery and exact identity | Existing AGNT conversation/run events, not another broker |
| `pi-toolbox-discovery` | lightweight always-discoverable capability | Existing tool registry; activation is not user consent |
| `pi-activity-strip` | pending/blocked/freshness clarity | Existing chat/task state, not another desktop runtime |

Primary references:

- [Howaboua pi-ask, pinned source](https://github.com/IgorWarzocha/howaboua-pi-stuff/tree/5193fa5844a14d5ad7d1b077dfadf17eafb7e1bc/packages/pi-ask)
- [Pi Interview upstream](https://github.com/nicobailon/pi-interview-tool) — audit also used a local 0.11.0 candidate; local candidate behavior is not asserted as upstream behavior.
- [Owned Pi extensions](https://github.com/tryingET/pi-extensions/tree/26641308958144e9870655a6ac98944a8fd6d5f9/packages) — source reference, not a runtime dependency.

This draft adapts concepts and does not copy third-party implementation source. Future copied code needs an explicit license/attribution review.

## Implementation sequence

1. Agree on scope and contract, then recheck the base branch's actual seams.
2. Implement durable model/service and its transactional lifecycle tests.
3. Implement native UI plus draft and receipt handling against a test adapter.
4. Connect registry and scheduler yield/delivery, prove same-round barriers and recovery.
5. Prove provider history compatibility and each supported chat surface.
6. Only then consider goal/workflow extensions, attachments or other convenience features.

New model/service/routes/component/test files should own most implementation. Candidate existing integration points:

- database schema/migration registration
- orchestrator tool registration
- orchestrator scheduler and answer-delivery seam
- native realtime event and chat state handling
- shared message rendering

Review changes to identity, authorization, sessions or credentials with maintainers under [CONTRIBUTING.md](../../../CONTRIBUTING.md). This public draft does not modify restricted areas or describe a security vulnerability; security findings follow private reporting separately.

## Versioning and rollback

Record upstream source revisions, native owner, contract/schema version, intentional differences and behavior tests. Review upstream changes semantically; do not automatically mirror them. Avoid multiple owners for the same behavior.

A rollback disables creation of new requests while preserving readable pending requests, accepted answers and receipts. Never delete decision history or automatically replay unknown effects.

## Verification

See [acceptance.md](acceptance.md). This draft has no runtime implementation and claims no passing behavior tests. Documentation links, JSON syntax, patch cleanliness and publication scope can be checked now; scheduling, privacy, persistence and provider behavior require the later implementation.
