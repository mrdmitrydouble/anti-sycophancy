---
name: as-status
description: Show whether anti-sycophancy mode is currently ON or OFF.
---

# Anti-sycophancy mode status

When this skill is invoked:

1. Read the file `.claude/.as-mode`.
   - If it contains `on` → report: **anti-sycophancy mode is ON**.
   - If it contains `off`, is empty, or does not exist → report: **anti-sycophancy mode is OFF**.
2. Report the status in one line. If ON, briefly remind which critics are active (as-critic-blind always; as-guardian when a second side exists; as-safety on a gate hit).
