# Ask / Interview acceptance gates

Status: all behavior gates are **planned, not executed**. This is not a test-result report.

## State, targeting and delivery

- [ ] Request creation commits before UI presentation.
- [ ] Draft survives reload and is not delivered as a submitted answer.
- [ ] Request/answer ownership is checked on every read and write using verified identity.
- [ ] Cross-user and cross-conversation access is rejected.
- [ ] Switching chat surfaces cannot retarget a pending answer.
- [ ] Two-tab duplicate submission returns one receipt and causes at most one accepted application.
- [ ] Conflicting answers cannot silently overwrite each other.
- [ ] Changed question/option revision visibly rejects a stale submission.
- [ ] Restart before answering preserves the question and draft.
- [ ] Restart after acceptance preserves answer plus delivery intent without tool replay.
- [ ] Lost acknowledgement permits receipt retrieval; unknown effects are not blindly retried.
- [ ] Cancel/supersede invalidates stale callbacks.

## Scheduling

- [ ] Ask-before-action and action-before-Ask emission orders both gate dependent consequential work.
- [ ] No incremental streaming/eager-dispatch path starts a gated action prematurely.
- [ ] Blocking questions release execution resources instead of keeping a model stream/worker open.
- [ ] Deferred answers are applied at a safe boundary to the originating logical task.
- [ ] Cancellation distinguishes stopping work, cancelling request, and merely hiding a view.
- [ ] Unsupported nested/background invocation fails explicitly.
- [ ] Resumption does not duplicate prior tool effects or concurrent continuations.

## Content and consent

- [ ] Unknown/duplicate question or option IDs are rejected.
- [ ] Normal submit, draft, cancel and timeout paths enforce compatible bounded validation.
- [ ] Untouched suggestions, blank responses, timeout and dismissal do not grant consent.
- [ ] Evidence is inert and cannot execute scripts or submit answers.
- [ ] Handoff completion is clearly a user report until independently verified where needed.
- [ ] No arbitrary HTML or secret/account metadata enters public request content by default.

## Providers and UI

- [ ] OpenAI, Anthropic and Gemini retain human-answer provenance and valid message/tool ordering.
- [ ] Main chat, workspace chat and each supported Forge surface are tested independently.
- [ ] Keyboard navigation, focus handling and screen-reader labels are verified.
- [ ] Compact and expanded views show the same request, draft and receipt.
- [ ] 'Draft saved', 'Submitted', 'Delivery pending', 'Applied' and 'Unknown' are distinct.

## Evaluation

Measure verified task completion, user effort and interruption burden, lost/stale responses, duplicate continuations, latency and incorrect actions. Do not optimize question count or raw approval rate. Keep missing cost/usage unknown rather than reporting zero.

## Release boundary

Do not call this operational until the relevant implementation gates pass. Do not expand v1 to a generalized authorization engine, camera, notebook suspension, automatic option generation or autonomous promotion merely to increase feature coverage.
