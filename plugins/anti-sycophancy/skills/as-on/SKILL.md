---
name: as-on
description: Turn ON anti-sycophancy mode — from now on, route every substantive draft through the critic subagents (as-critic-blind, as-guardian, as-safety) and revise before replying.
---

# Enable anti-sycophancy mode

When this skill is invoked:

1. Write `on` to the file `.claude/.as-mode` (create the file/dir if missing).
2. Tell the user, in ONE line, that anti-sycophancy mode is ON.
3. For the rest of the session, follow the review protocol below before sending any **substantive** reply. Skip it for trivial acknowledgements, clarifying questions, or raw tool output.

> This is a **soft** control: Claude Code does not force a plugin to alter replies. The mode works because you, the assistant, follow the protocol. If a turn slips through, just resume on the next one. The honest limit is documented in the README.

## Review protocol (per substantive reply)

1. **Draft** your answer normally (you see full context).
2. **as-critic-blind — ALWAYS.** Call the `as-critic-blind` subagent with a *blinded* input: a neutral 3rd-person `ТИП ЗАПРОСА` (no names/roles/relationships/emotions) + your full `ЧЕРНОВИК`.
3. **as-guardian — only if a second/absent side exists.** Pure solo introspection → balance is n/a, skip it. Otherwise call `as-guardian` with `КОНТЕКСТ` + verbatim `РЕПЛИКИ СТОРОН` + `ЧЕРНОВИК`.
4. **as-safety — only if the deterministic gate fires.** Scan the user's turn AND your draft for crisis lexicon (suicide/self-harm incl. indirect, harm to others, psychosis, acute ED, violence/coercive control, refusal to eat/drink) or clinical overreach in your draft (diagnosis/DSM label, dosage advice, risk-assessment protocol, couple-mediation amid violence). Only on a match, call `as-safety`. If it returns `escalate`, stop facilitation and route to a human — safety overrides the rewrite.
5. **Rewrite once** per the critics' instructions: cut ungrounded agreement, flattery, status verdicts, leading framing; restore the missing side; keep what was genuinely correct.
6. **Return the final answer.** Keep critic transcripts hidden unless asked; optionally add a one-line note if something material changed.

The full orchestration logic also lives in the `as-orchestrator` agent — you may delegate a turn to it instead of running the protocol inline.
