# reins-playbook: Rein In Your AI Dev Fleet

A three-step workflow to inventory, remediate, and continuously guard the standing permissions your AI coding agents accumulate. It chains the [Reins MCP server](https://github.com/<GITHUB_USERNAME>/reins) and the [Reins Audit skill](https://github.com/<GITHUB_USERNAME>/reins-skill) into one loop: **audit → remediate → enforce → watch**.

## Why a playbook and not just the tools

`reins_audit` tells you what's there. `reins_revoke` removes it. Neither one, by itself, is a workflow — you still have to decide what to remove, confirm it, generate guardrails so it doesn't quietly come back, and check again later. This playbook is that sequence, written down once so you don't have to reconstruct it each time.

## The chain

### 1. Audit — Reins MCP Server

Run `reins_audit` to inventory every standing permission grant across Claude Code's settings sources, classified into 12 risk categories. Run `reins_blast_radius` alongside it to see the same findings ranked by what a prompt-injected session could actually do with each one, worst first.

### 2. Remediate — Reins Audit Skill

Hand the findings to the Reins Audit skill (`/reins-audit`, or just ask "clean up my Claude Code permissions"). It walks `remove`-verdict findings first, shows you exactly what each one is before touching it, and executes confirmed revocations through `reins_revoke` — which re-verifies each rule is still present, backs up the file, and writes atomically. It reviews `review`-verdict findings with you rather than acting on them unasked.

### 3. Enforce and watch — Reins MCP Server

Once the standing grants are clean, run `reins_generate_policy` to produce `permissions.deny` rules and a `PreToolUse` hook script, so the same category of risk can't quietly reaccumulate unnoticed. Run `reins_snapshot` to save the clean state as a baseline. From then on, a periodic `reins_drift` re-check tells you exactly what's new since that baseline — with any newly added grant already fully classified and scored, not just flagged as "different."

## Walked end-to-end once, for real

This chain was run against a fixture settings file with 9 grants (3 `remove`-verdict, 2 `review`-verdict, 4 `keep`) using the real Reins MCP server, not a simulation:

1. `reins_audit` found the 3 `remove`-verdict grants (a `sudo` command, a credential-store read, and an unbounded `rm -rf *`) and 2 `review`-verdict grants (an unscoped package install, a broad cloud API wildcard).
2. `reins_blast_radius` ranked them: the `sudo` grant and the credential read both scored CRITICAL, the destructive and review-verdict grants scored HIGH.
3. `reins_revoke` removed the 3 `remove`-verdict grants, each with a timestamped backup, leaving the 2 `review`-verdict grants untouched for a human decision.
4. `reins_generate_policy` produced a `permissions.deny` block covering `sudo`, shell-escape forms, and the common credential-store paths.
5. `reins_snapshot` captured the resulting 6-grant clean state as a baseline.
6. A new grant (`Bash(curl *)`) was added to simulate a later "always allow" click. `reins_drift` caught it immediately — reported as one addition, already classified as category 6 (exfil-capable network), `remove`-verdict, CRITICAL blast radius.

See `PLAYBOOK.md` for the condensed step table.

## Prerequisites

- The [Reins MCP server](https://github.com/<GITHUB_USERNAME>/reins) connected to Claude Code.
- The [Reins Audit skill](https://github.com/<GITHUB_USERNAME>/reins-skill) installed (optional if you're comfortable driving `reins_revoke` yourself; the skill exists to make step 2 conversational rather than manual).

## What it outputs

A clean standing-permission baseline, a `permissions.deny` guardrail covering what was found, and a repeatable `reins_drift` check for whatever appears next.

## Known limitations

This is a workflow description, not new code — every claim about behavior here is a claim about the Reins MCP server and skill, and their limitations apply here too (pattern-based classification, point-in-time audits between drift checks, no ability to intercept a running agent mid-session). See the [Reins MCP server README](https://github.com/<GITHUB_USERNAME>/reins#known-limitations) for the full list.

## Security

Found a vulnerability in this workflow description itself? Please report it privately via [GitHub Security Advisories](https://docs.github.com/en/code-security/security-advisories/guidance-on-reporting-and-writing/privately-reporting-a-security-vulnerability). Vulnerabilities in the tools themselves belong in their own repos.

## License

MIT. See `LICENSE`.
