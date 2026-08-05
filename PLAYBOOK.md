# Rein In Your AI Dev Fleet — step table

Narrative arc: **audit → remediate → enforce → watch**. Full walkthrough and a verified end-to-end run in `README.md`.

| Step | Agent | Tool calls | What it does |
|---|---|---|---|
| 1. Audit | Reins MCP Server | `reins_audit`, `reins_blast_radius` | Inventories every standing permission grant across Claude Code's settings sources, classifies each into one of 12 risk categories, and ranks findings by worst-case blast radius. |
| 2. Remediate | Reins Audit Skill | `reins_revoke` | Reviews `review`-verdict findings with the user, confirms `remove`-verdict revocations explicitly, then executes them — re-verifying each rule is still present, backing up the file, and writing atomically. |
| 3. Enforce | Reins MCP Server | `reins_generate_policy` | Generates `permissions.deny` rules and a `PreToolUse` hook script from the clean audit, so the same category of risk can't quietly reaccumulate unnoticed. |
| 3. Watch | Reins MCP Server | `reins_snapshot`, `reins_drift` (scheduled) | Snapshots the clean state as a baseline. Each later `reins_drift` run reports exactly what's new since that baseline, with new grants already classified and scored. |

## Expected end-state

- A standing-permission baseline with no `remove`-verdict grants left.
- A `permissions.deny` block and hook script in place, covering what the audit found plus the recommended universal baseline (credential paths, `sudo`, shell-escape forms).
- A saved snapshot at `~/.reins/baseline.json`.
- A repeatable `reins_drift` check, run on whatever cadence fits (after each work session, daily, or on a schedule), that reports new grants pre-classified, not just as an undifferentiated diff.

## Agents used

| Name | Role | Ref |
|---|---|---|
| Reins MCP Server | Inventory, classification, blast-radius ranking, guardrail generation, snapshot/drift | `mcp-servers/reins-mcp-server` |
| Reins Audit Skill | Interactive remediation: review findings with the user, confirm and execute revocations | `skills/reins-skill` |
