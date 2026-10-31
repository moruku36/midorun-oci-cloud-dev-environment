# 04 claude-code-japanese-guard: a Stop hook that stops English drift

English | [日本語](04-japanese-guard.ja.md)

Repository: https://github.com/minorun365/claude-code-japanese-guard (★75, Apache-2.0, created Sep 25, 2026)
The identical file also ships inside [my-claude-code-settings](03-settings.md) (blob hash verified).

## Motivation

"Writing 'reply in Japanese' in CLAUDE.md did not stop the mid-task drift into English. So I built a hook that mechanically inspects the text at the end of the turn and, if it's English, sends it back for a rewrite." (from the README)

This is the flagship example of the design philosophy in [03](03-settings.md): **what natural-language instructions can't stop, stop with a mechanical check.**

## Layout

```
hooks/japanese-guard.py      (11.0 KB, Python 3 stdlib only)
tests/test_japanese_guard.py (8.6 KB)
README.md / LICENSE (Apache-2.0) / NOTICE
```

Installation is designed to be "give Claude Code the repository URL and say 'set this up for me'" — the README contains a setup procedure addressed to Claude Code itself (hook placement, the two settings.json registrations, JSON validation, a smoke check).

## How it works (from reading the source)

1. **Input**: reads the JSON Claude Code passes to the hook on stdin at turn end. If `stop_hook_active` is set, it exits immediately — **at most one send-back per turn**, so even a still-English rewrite ends there and can't loop forever.
2. **Getting the final reply (two paths)**: (a) judge `last_assistant_message` from the hook input immediately; (b) parse the transcript JSONL and take "the text after the last tool call, following the last genuine user message" as the final reply. Because a Stop hook can fire before the transcript is flushed, it polls up to 3 seconds for the write — unless path (a) is already English, in which case it sends back without waiting.
3. **The English test, `is_english()`**: strip code blocks, inline code, URLs, email addresses and Markdown links, then count Latin letters vs. Japanese characters (hiragana/katakana/kanji). It judges English-dominant when **Latin ≥ 25 characters AND Latin > Japanese × 3**. Short replies like "OK", and legitimate cases (showing an English draft or a command), are excluded by design.
4. **Three inspection targets**: (1) the final reply; (2) interim progress reports between tool calls (only multi-line ones with ≥60 Latin chars, or single-line paragraphs with ≥250 — one-line preambles are exempt); (3) the desktop app's "next step" box (the wrap_up tool's recap/ask/button texts).
5. **The send-back**: if English is found, it outputs `{"decision": "block", "reason": ...}` and stops the turn from ending. The reason quotes the detected passages (max 5, first 80 chars of each line) and carries a fixed instruction: "rewrite only the English passages above into Japanese; do not repeat the parts already delivered in Japanese; do not write excuses or explanations of the cause."
6. **Also registered as PreToolUse**: the same script on the wrap_up tool stops an English "next step" box with `permissionDecision: "deny"` before it is ever displayed. The README even documents the matcher regex pitfall (a bare `wrap_up` matcher is treated as an exact match and never fires).
7. **Tuning and audit**: all thresholds are environment variables. A `--check <transcript.jsonl>` mode audits past session transcripts and shows which turns were English-dominant.

## What to learn from it

- Problems a machine can decide (language drift) are solved with a **hook + tests**, not with prompts.
- The send-back itself is designed: "only once", "rewrite only the flagged passages", "no excuses".
- Careful in-scope/out-of-scope lines (short replies, legitimately shown English, one-line preambles) keep false positives from blocking work.
