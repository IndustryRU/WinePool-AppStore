# WinePool agent operating rules

These rules apply to every agent session in this repository.

## Efficiency and budget contract

- Preserve full implementation autonomy inside the user's stated scope. Do not
  turn efficiency constraints into a refusal to perform multi-step work.
- Target at least 50% more completed, verified work per unit of agent budget
  than the 2026-08-29 session baseline, while targeting no more than 50% of the
  agent-budget cost for an equivalent task.
- Treat rate-limit budget as a scarce project resource. Prefer shipping a
  complete scoped result over exhaustive narration or repeated confirmation of
  already established facts.
- Start from the current Git state, the latest handoff/status section, and the
  user's newest acceptance evidence. Do not re-audit settled history or reread
  unrelated specifications.
- Use one focused discovery pass, one implementation pass, and one consolidated
  verification pass whenever practical. Batch independent reads and tests.
- Do not repeat builds, browser checks, remote reads, deployments, or hashes
  unless code changed, the prior result failed, or the repetition answers a new
  material question.
- Reuse successful user acceptance checks as evidence. Add automated or remote
  verification only where it closes a real risk not already covered.
- Keep commentary short and milestone-based. Do not spend budget narrating
  routine tool use.
- For production work, make the required scoped backup, apply only the exact
  migration or artifact in scope, and perform one bounded smoke/reconciliation
  pass. Avoid broad production audits unless explicitly required.
- Commit completed scoped work when requested. Never push without the user's
  separate permission.
- Before an unusually expensive detour, reconsider the cheapest reliable path.
  If the detour is not necessary for the requested outcome, skip it and record
  it as follow-up rather than consuming the current session.

## Token-economy profile

- Default to concise answers and milestone-only progress updates. Do not repeat
  the request, settled context, command output, or unchanged findings.
- Keep discovery bounded: inspect only files directly relevant to the current
  task, prefer targeted searches, and stop investigating once enough evidence
  exists to implement and verify the scoped result.
- Do not spawn sub-agents unless the user explicitly requests delegation or
  parallel agent work.
- Prefer one batched tool call over many small calls. Cap command output to the
  minimum useful amount and avoid dumping whole logs or large files.
- Use the lowest reasoning effort that can reliably complete the task. Escalate
  reasoning only after a concrete failure or for genuinely high-risk work.
- Do not browse, open UI automation, or load optional skills/plugins unless the
  request requires them or higher-priority instructions mandate them.
- For implementation tasks, make the smallest coherent change and run the
  narrowest meaningful verification. Do not perform unrelated cleanup.
- Ask for clarification only when different answers would materially change the
  implementation; otherwise state a reasonable assumption and proceed.
