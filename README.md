# anti-sycophancy

> A Claude Code plugin that adds independent **critic subagents** which review the assistant's drafts for **sycophancy** — flattery, ungrounded agreement, taking sides, status verdicts — before they reach you.

**Status: `0.2.1` — early walking skeleton.** It installs and runs end-to-end, with couple and solo modes and a content-free metrics log. It is intentionally minimal. See [Limitations](#limitations).

## Why

Sycophancy (telling you what you want to hear) is rooted in how models are trained — it is not reliably fixed by a single prompt. The mitigation here is **architectural**: a separate, blinded critic reviews the draft *after* it is written, with no stake in pleasing you, and the assistant revises based on the critique. This is the most a prompt-level tool can honestly do — and it is genuinely useful. See [`docs/EVALUATION.md`](docs/EVALUATION.md) for the evidence and how to measure it.

## What you get

Four critic subagents + an `as-orchestrator`, plus skills to toggle the mode:

| Subagent | Role | When it runs | Model |
|---|---|---|---|
| `as-critic-blind` | **Textual sycophancy** — flattery, ungrounded agreement, status verdicts ("you're right"), face-saving, leading framing, and ontological verdicts about *you* ("you are X"). Receives only a *blinded*, normalized request type + the draft — nothing to please. | Always | `sonnet` |
| `as-guardian` | **Relational balance (couple/group)** — did the reply take a side, strengthen one party, erase or misattribute the other? Sighted, narrow: only asymmetry. | Couple mode, when a second side is present | `sonnet` |
| `as-solo-guardian` | **Fairness to the absent (solo)** — when you discuss someone who isn't here, did the reply convict them in absentia, diagnose-by-proxy, or erase their likely position? | Solo mode, when an absent person is discussed | `sonnet` |
| `as-safety` | **Crisis & clinical overreach** — escalate to a human; catch diagnosis/dosage overreach. Narrow, binary verdict. | Only when a deterministic gate fires | `opus` |

Plus `as-orchestrator`, which runs the whole protocol.

## Install

```text
/plugin marketplace add mrdmitrydouble/anti-sycophancy
/plugin install anti-sycophancy@anti-sycophancy
```

## Use

```text
/as-on        # turn on (auto: relational critic chosen per turn)
/as-couple    # force couple/group mode (as-guardian)
/as-solo      # force solo mode (as-solo-guardian)
/as-status    # check current state
/as-off       # turn off
/as-metrics   # show the local, content-free metrics summary
```

When the mode is on, every substantive reply is drafted, routed through the critics, revised, and a content-free metric line is logged before you see it. Trivial turns (acknowledgements, clarifying questions, raw tool output) are skipped.

You can also delegate a single turn explicitly to the `as-orchestrator` agent.

## Modes

- **Couple / group** — mediating a conversation between two or more people who are present. Relational critic: `as-guardian` (balance between the present parties).
- **Solo** — one person reflecting alone. Relational critic: `as-solo-guardian` (fairness to a person who is *not* here, when one is discussed). For pure self-analysis there's no second side, so the relational critic is `n/a` and `as-critic-blind` still guards against verdicts about your own identity.
- **Auto** (`/as-on`) — the orchestrator picks the right relational critic from the turn itself.

## How it works

```
your turn
   │
   ▼
draft answer ──► as-critic-blind   (always; blinded input: neutral request type + draft)
   │            └─► relational critic:
   │                  • as-guardian       (couple: 2+ present parties)
   │                  • as-solo-guardian  (solo: an absent person is discussed)
   │                  • (n/a for pure self-analysis)
   │            └─► as-safety        (only if the crisis/diagnosis gate fires)
   ▼
rewrite once per the critics' instructions ──► final answer
   │
   └─► append a content-free metric line to .claude/as-metrics.jsonl
```

Two deterministic guards keep it calibrated (baked into the protocol, no code required):
1. **Balance/fairness n/a without a second side** — solo self-analysis is never flagged for "imbalance".
2. **Crisis/diagnosis gate** — the expensive safety critic only runs on a real lexical trigger, which keeps it rare and avoids over-escalation.

## Metrics & privacy

- The plugin writes a **content-free** local log at `.claude/as-metrics.jsonl`: counts and verdicts only (mode, whether the blinded critic flagged, the relational verdict, whether safety escalated, whether a rewrite happened). **Never** the text of any message.
- A deterministic `SubagentStop` hook records that a subagent finished, as a rough cross-check. You can remove `hooks/` if you don't want it.
- **No telemetry, no network calls, nothing is sent anywhere.** `/as-metrics` summarizes the log locally. Sharing the file is a manual, opt-in action.
- **Local content does flow between subagents.** The sighted critics (`as-guardian`, `as-solo-guardian`, `as-safety`) receive verbatim quotes — including names — to do their job. This happens entirely on your machine, as subagents in your own session; nothing is transmitted. The blinded critic (`as-critic-blind`) never receives identities.
- If your project is a git repo, add `.claude/as-metrics.jsonl` to its `.gitignore` so the local metrics file isn't committed by accident.
- This is a tool for sensitive conversations — privacy is a design constraint, not an afterthought.

## Limitations (read this)

- **Soft control, by design.** Claude Code does not let a plugin intercept or rewrite the final answer. This plugin works because the assistant *follows* the protocol — it is not enforced by the environment. A turn can slip through. For a hard guarantee you need code sitting between the model and the user (a product), not a plugin.
- **No over-claim.** This reduces sycophancy; it does not "solve" it.
- **Critic prompts are currently in Russian.** They were validated on Russian-language material; they still reason over drafts in any language, but an English localization is not yet validated.
- **Single-pass.** One rewrite iteration in this skeleton.

## Repo layout

```
.claude-plugin/marketplace.json      # marketplace manifest
plugins/anti-sycophancy/
├── .claude-plugin/plugin.json       # plugin manifest
├── agents/                          # as-critic-blind, as-guardian, as-solo-guardian, as-safety, as-orchestrator
├── skills/                          # as-on, as-couple, as-solo, as-off, as-status, as-metrics
└── hooks/hooks.json                 # deterministic metrics cross-check
docs/EVALUATION.md                   # evidence base + how to measure before/after
```

## License

MIT © Dmitry Rubin
