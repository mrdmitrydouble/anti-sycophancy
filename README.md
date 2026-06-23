# anti-sycophancy

> A Claude Code plugin that adds independent **critic subagents** which review the assistant's drafts for **sycophancy** — flattery, ungrounded agreement, taking sides, status verdicts — before they reach you.

**Status: `0.1.0` — early walking skeleton.** It installs and runs end-to-end, but it is intentionally minimal. See [Limitations](#limitations).

## Why

Sycophancy (telling you what you want to hear) is rooted in how models are trained — it is not reliably fixed by a single prompt. The mitigation here is **architectural**: a separate, blinded critic reviews the draft *after* it is written, with no stake in pleasing you, and the assistant revises based on the critique. This is the most a prompt-level tool can honestly do — and it is genuinely useful.

## What you get

Three critic subagents + an orchestration protocol:

| Subagent | Role | When it runs | Model |
|---|---|---|---|
| `as-critic-blind` | **Textual sycophancy** — flattery, ungrounded agreement, status verdicts ("you're right"), face-saving, leading framing. Receives only a *blinded*, normalized request type + the draft — nothing to please. | Always | `sonnet` |
| `as-guardian` | **Relational balance** — did the reply take a side, strengthen one party, erase or misattribute the other? Sighted, but narrow: only asymmetry. | Only when there's a second / absent side | `sonnet` |
| `as-safety` | **Crisis & clinical overreach** — escalate to a human; catch diagnosis/dosage overreach. Narrow, binary verdict. | Only when a deterministic gate fires | `opus` |

Plus `as-orchestrator`, which runs the whole protocol, and three skills to toggle the mode.

## Install

```text
/plugin marketplace add mrdmitrydouble/anti-sycophancy
/plugin install anti-sycophancy@anti-sycophancy
```

## Use

```text
/as-on        # turn the review mode on
/as-status    # check whether it's on or off
/as-off       # turn it off
```

When the mode is on, every substantive reply is drafted, routed through the critics, and revised before you see it. Trivial turns (acknowledgements, clarifying questions, raw tool output) are skipped.

You can also delegate a single turn explicitly to the `as-orchestrator` agent.

## How it works

```
your turn
   │
   ▼
draft answer ──► as-critic-blind  (always; blinded input: neutral request type + draft)
   │            └─► as-guardian   (only if a 2nd/absent side exists; verbatim quotes + draft)
   │            └─► as-safety     (only if the crisis/diagnosis gate fires)
   ▼
rewrite once per the critics' instructions ──► final answer
```

Two deterministic guards keep it calibrated (baked into the protocol, no code required):
1. **Balance n/a without a second side** — solo introspection never gets flagged for "imbalance".
2. **Crisis/diagnosis gate** — the expensive safety critic only runs on a real lexical trigger, which keeps it rare and avoids over-escalation.

## Limitations (read this)

- **Soft control, by design.** Claude Code does not let a plugin intercept or rewrite the final answer. This plugin works because the assistant *follows* the protocol — it is not enforced by the environment. A turn can slip through. For a hard guarantee you need code sitting between the model and the user (a product), not a plugin.
- **No over-claim.** This reduces sycophancy; it does not "solve" it.
- **Critic prompts are currently in Russian.** They were validated on Russian-language material; they still reason over drafts in any language, but an English localization is not yet validated.
- **Single-pass.** One rewrite iteration in this skeleton.

## Privacy

- The plugin adds **no data collection**. There is no telemetry, no network calls, nothing sent anywhere.
- A metrics log (counts only, never message content) is planned for a later version and will be **opt-in and local**.
- This is a tool for sensitive conversations — privacy is a design constraint, not an afterthought.

## License

MIT © Dmitry Rubin
