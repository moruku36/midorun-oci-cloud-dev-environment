# 08 Adopting it in your own setup

English | [日本語](08-takeaways.ja.md)

Minorun-style, translated into a practical checklist for my own environment (moruku's): multiple AI agents in parallel (Claude Code / Codex / Antigravity / Muse), technical writing, and personal development projects. In priority order.

## Quick wins (configuration)

- [ ] **Keep one canonical rulebook**: if several agents share the same rules, make AGENTS.md the source of truth and keep only tool-specific deltas in each tool's config (the CLAUDE.md = `@AGENTS.md` + deltas pattern from [03](03-settings.md)).
- [ ] **Cap the context budget of the skill listing** (`skillListingBudgetFraction`-style) and measure listing bloat as skills accumulate.
- [ ] **Sync settings via symlinks into a dotfiles repo** — delete the copy-sync step entirely.
- [ ] **Isolate research into subagents** to protect the parent context, with a fixed report format (conclusion → problem → evidence → recommended action).

## Run the learning loop

- [ ] **Make "remember this" the wake phrase**: any turn containing a correction, dissatisfaction, a discovered trap, or a settled preference gets persisted on the spot, routed by destination (shared → AGENTS.md / project-specific → that repo / procedures → skills).
- [ ] **Persist only facts you verified once yourself.** Never rewrite settings as a substitute for an apology.
- [ ] **Nightly automation goes up to "proposal" only**: review the day's sessions and draft a learnings report; a human adopts it next morning (the dreaming pattern).
- [ ] **Budget the always-loaded volume**: AGENTS.md holds only judgments needed at the entrance of every turn; overflow moves into skills.

## Stop what must be stopped, with hooks

- [ ] **Hook only what a machine can decide**: wrong destinations (the check-outbound pattern), language drift (the japanese-guard pattern). Do not mechanically inject natural-language learning reminders every turn.
- [ ] Design hook send-backs fully: at most once per turn, rewrite only the flagged passages, no excuses.
- [ ] **Exactly one confirmation for external sends** (double approvals decay into rubber stamps). Your own repos pass; other people's destinations stop.

## How to work

- [ ] **Default to semi-automation**: delegate whole jobs end to end and approve at key points, instead of spot-outsourcing small tasks.
- [ ] **Mini spec-driven development**: before code — PLAN.md (10-minute voice dump), SPEC.md, TODO.md, KNOWLEDGE.md.
- [ ] **Accumulate context in one workplace (monorepo)**: keep procedures, phrasing and preferences there and let the "compound interest of context" work. Accept that things get easier from the second time.
- [ ] **Never wave through an approval you don't understand** — reject and ask back. Write guardrails in natural language and let the AI write them into the settings.
- [ ] When a system can't connect, degrade in order: **MCP → browser operation → copy-paste Markdown**. The human always presses "log in" and "confirm/send".

## Always-on environment (moving development to the cloud)

- [ ] Minorun's [EC2 setup](05-claude-code-on-ec2.md) costs ~$52–55/month. For the same pattern at ¥0, **OCI Always Free (Ampere A1 — currently ~2 OCPU / 12 GB)** is the candidate: inbound fully closed, SSM-style admin access, tmux auto-recovery, and outbound-only Remote Control all transfer as-is.
- [ ] Detach development from local-Windows asset-fetch failures and reboots; turn the local machines into thin clients.

## Writing and presenting

- [ ] **Distill know-how into skills** (his talk-deck skill, [07](07-articles.md)) — including mechanical checks like real-size figure inspection — to make it reusable.
- [ ] For technical writing, never claim "it works" until you have run the procedure end to end yourself; keep a source-trust ranking and treat even your own knowledge base as secondary information (the tech-writing-verification pattern).
