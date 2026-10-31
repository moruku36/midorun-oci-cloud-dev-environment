# minorun-claude-code-study

English | [日本語](README.ja.md)

A study repository on how **Minorun** (minorun365 / Minoru Onda) — widely regarded as one of Japan's most skilled Claude Code practitioners — uses Claude Code, distilled from his own public materials: GitHub repositories, Speaker Deck talks, his book, and Qiita/note articles.

His setup is examined down to the actual configuration files, with an actionable checklist for adopting the ideas in your own development and daily work.

> This repository is a **summary and link collection** of public materials, not a reproduction of his slides or config files. Always consult the originals at the linked sources.

## Who is Minorun? (short version)

- Minoru Onda, Tech Evangelist at KDDI Agile Development Center (KAG) / Senior Expert at KDDI
- **AWS AI Hero** (2025, first in Japan), AWS Community Hero (2024), Qiita Top Contributor 2024 & 2025
- Author of 7 books, including the solo-authored *[Minorun-style] Claude Code Work Techniques for Business People* (Gijyutsu-Hyohron, Sep 24, 2026)
- Runs 800+ Claude sessions and ~3,500 interactions per quarter on a single work PC

## Documents

| # | Document | Contents |
|---|---|---|
| 01 | [Profile and book](docs/01-profile-and-book.md) | Career, awards, the book's aims and table of contents, review roundup |
| 02 | [Core philosophy](docs/02-philosophy.md) | Semi-automation, the monorepo and "compound interest of context", mini spec-driven development, voice input, approvals and safety |
| 03 | [Anatomy of his settings](docs/03-settings.md) | my-claude-code-settings: the CLAUDE.md / AGENTS.md split, settings.json, rules / hooks / agents / skills, the learning loop |
| 04 | [japanese-guard](docs/04-japanese-guard.md) | A Stop hook that mechanically sends English-drifted replies back for a Japanese rewrite |
| 05 | [Claude Code on EC2](docs/05-claude-code-on-ec2.md) | Always-on Claude Code on EC2 with smartphone Remote Control; architecture and security notes |
| 06 | [Slides and talks](docs/06-slides-and-talks.md) | Summaries of his key Speaker Deck decks and talks |
| 07 | [Articles](docs/07-articles.md) | Key points from his Qiita / note writing |
| 08 | [Adopting it in your own setup](docs/08-takeaways.md) | A prioritized, practical checklist |
| — | [Sources](docs/sources.md) | All links, with what was read in full vs. snippet-only |

## The three biggest lessons

1. **Semi-automation, not full automation**: let the AI run work end to end; the human only steps in at approval points.
2. **Accumulate context in one workplace (a monorepo)**: procedures, phrasing and preferences persist there and raise the quality of unrelated work — the "compound interest of context".
3. **Rules in natural language; hooks only where a machine can decide**: learnings are folded back into settings with a single "remember this", and nightly automation only *proposes* changes — a human adopts them.

## About this research

- Researched on 2026-11-01.
- Primary sources were read directly (the actual files in his GitHub repositories, the official gihyo.jp book page, Speaker Deck transcripts, Qiita/note article bodies) and summarized. Each document and the [source list](docs/sources.md) distinguish what was read in full from what is based on search snippets only.
