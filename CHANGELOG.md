# Changelog

All notable changes to this plugin are documented here.
Format follows [Keep a Changelog](https://keepachangelog.com/); this project uses [Semantic Versioning](https://semver.org/).

## [0.2.2] — 2026-06-24

Pre-release hardening pass closing the findings of an independent pre-production audit
(1 blocker + 4 major + minor/nit). The anti-sycophancy engine was unchanged and already
verified independently (behavioral 22/22, safety 12/12 escalate + 8/8 continue); this
release fixes packaging and the safety *wrapping* around it.

### Fixed
- **(blocker) Plugin metadata now loads.** Quoted the `description:` frontmatter in
  `agents/as-guardian.md`, `agents/as-solo-guardian.md`, `skills/as-couple/SKILL.md`,
  `skills/as-solo/SKILL.md`. An unquoted `:` had broken YAML parsing, so those files
  loaded with empty metadata — the two relational guardians lost their name, `model`,
  and `tools: []`, which silently disabled the relational critic in couple and solo
  modes. `claude plugin validate` now passes (exit 0).

### Added
- **Crisis resources.** A canonical, region-aware CRISIS RESOURCES block the escalate
  path emits verbatim (Russia 112 / МЧС +7 495 989-50-50 / Детский телефон доверия
  8-800-2000-122; US 988 and 741741; international findahelpline.com), with an honest
  fallback to look up a current local line if one is out of date.
- **Non-clinical disclaimer** in the README (before Install) and a one-line version
  surfaced by `/as-on`, `/as-couple`, `/as-solo`: not therapy, not diagnosis, not an
  emergency service; the safety gate is best-effort and must not be relied on for safety.
- **Deterministic safety floor.** On any crisis gate class (T1–T9) the orchestrator never
  leaves a turn with zero safety surface, even when the single-LLM safety critic returns
  `continue` — unless that `continue` is explicitly grounded as past narrative/metaphor.
- **Diagnosis-request trigger (C12-REQ).** The gate now fires when the *user* asks to be
  diagnosed (not only when the draft over-reaches) and routes to psychoeducation instead
  of naming a diagnosis.
- **AI disclosure** on escalation ("you're talking with an AI, not a clinician").
- Light post-rewrite self-check on flagged turns; optional per-turn **nonce** on the data
  fences passed to critics.
- `homepage` in `plugin.json`; `license` in the marketplace plugin entry;
  `LICENSE` mirrored into the plugin subtree; `CHANGELOG.md`.

### Changed
- README `Use` section clarified (the toggles are **skills**, invoked by name) and
  `Limitations` expanded (not a crisis detector; mode routing is model-judged; the gate
  lexicon is Russian-validated; single-pass with a light self-check).
- Documented the Russian→English verdict mapping on the metrics path.
- Honesty polish in README/EVALUATION (softened "genuinely useful" and "large net
  correction"; flagged that ELEPHANT and Cheng et al. are related work, not independent).
- `.gitignore` now lists the per-project runtime files defensively.
- Hardened the metrics `SubagentStop` hook: resolves its target directory once and writes
  only to an absolute path, so the log can't land in an unexpected folder.

## [0.2.1] — 2026-06-23
- Internal audit fixes: enforce critic blinding via `tools: []`, safety-gate T-codes,
  honesty and privacy passes, boolean metric fields, justified-asymmetry calibration.

## [0.2.0] — 2026-06-23
- Solo mode (`as-solo-guardian`, advocate for the absent) + couple/solo/auto routing;
  content-free metrics log + hook; `docs/EVALUATION.md` (before/after methodology).

## [0.1.0] — 2026-06-23
- Walking skeleton (B1): blinded textual critic, relational guardian, orchestrator,
  toggle skills, plugin + marketplace manifests.

[0.2.2]: https://github.com/mrdmitrydouble/anti-sycophancy/releases/tag/v0.2.2
[0.2.1]: https://github.com/mrdmitrydouble/anti-sycophancy/releases/tag/v0.2.1
[0.2.0]: https://github.com/mrdmitrydouble/anti-sycophancy/releases/tag/v0.2.0
[0.1.0]: https://github.com/mrdmitrydouble/anti-sycophancy/releases/tag/v0.1.0
