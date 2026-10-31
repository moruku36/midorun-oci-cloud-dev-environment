# 02 Core philosophy

English | [日本語](02-philosophy.ja.md)

The core ideas of "Minorun-style", drawn from his talks, book and flagship Qiita article.

## 1. Semi-automation (don't aim for full automation)

- Claude Code is not just for coding: expense reports, attendance, travel arrangements, project work (agendas, meeting notes, research, documents) and talk preparation are delegated **end to end, not piecemeal**.
- But full automation is not the goal. In a human → AI → human flow, **the human only handles instructions and approvals at key points** — he works with 3 sessions running in parallel behind a meeting.
- Measured reality: about 60% of tasks land right on the first shot (his own 3-month measurement); the rest are finished through iteration; 1 in 20 fails. The AI sometimes reports falsehoods confidently, so always open and check the deliverable yourself.

## 2. The metaphor: three excellent mid-career hires — on their first day

Claude is "like having three excellent ex-consulting mid-career hires assigned to you full-time — except it's their first day and they know nothing about the company." So **onboarding comes first**: hand over your role, your task list, where the documents live, and who the people are.

### The 3 setup steps

1. **Create a daily-work monorepo** (gather your work into one repository)
2. **Introduce yourself exhaustively by voice input** (give tool URLs and document paths too; let the AI organize the folders)
3. **Then just ask**

His recommended starting point is the Claude Desktop app plus the $20/month Pro plan.

## 3. The monorepo and the "compound interest of context"

- When procedures, phrasing preferences and past interactions accumulate in one workplace (the monorepo), they **raise the quality of unrelated work too** — the "compound interest of context". Things get easier from the second time onward.
- Example: for expense reports, it starts investigating the situation from past interactions without being told the procedure.
- Over 80% of the files he has Claude edit are Japanese documents — non-coding work is the main battlefield.

## 4. Mini spec-driven development

Before any code, create four documents in `/docs` (from his flagship article and DevSumi talk):

| File | Role |
|---|---|
| PLAN.md | The plan, dumped by voice in ~10 minutes (he uses Aqua Voice) |
| SPEC.md | The spec, aligned through dialogue to remove misunderstandings |
| TODO.md | Progress tracking |
| KNOWLEDGE.md | Accumulated learnings |

## 5. How to ask

- Instructions in **plain spoken Japanese are fine**; no prompt "incantations" needed.
- Make the AI ask *you* questions when something is unclear.
- Use **voice input** (Aqua Voice is his staple; OS dictation works too) to maximize the amount of context you hand over.
- Consult on the delegation itself: "how should I hand this task to you to make it easy?"
- To grow it, just say **"remember this"** and it appends to the settings; when settings get messy, ask for a "big cleanup".
- A 22:00 nightly routine reviews all sessions and drafts a learnings report — direct edits are forbidden; he reviews and applies them the next morning.

## 6. Approvals and safety

- Recommends the standard "auto mode", with high-impact operations screened by a separate AI behind the scenes.
- **Never wave through an approval you don't understand** — reject it and ask back.
- Write guardrails in natural language and let the AI itself write them into the settings. Example: "the human presses the final button for orders, sends and deletions."
- Paradoxically, *narrowing* confirmations makes things safer.
- Security is "control between prohibition and laissez-faire" — offensive security. AI adds three surfaces: the instruction inlet (instructions hidden in external text), the execution of operations (it acts with your privileges), and the data outlet (an LLM is outside the company).
- His own incident: a Slack reply was sent without confirmation during a request-handling session. After concretizing the rule to "operations that reach other people require confirmation", it never recurred.
- Win over the security department by writing down concretely "which data may be operated on, and how far".

## 7. Fallbacks when nothing connects

- If a corporate system can't be reached via MCP, use browser operation instead. **If even that fails, have it produce a copy-paste-ready Markdown table** — a 30-minute job still becomes 5 minutes.
- Rule: "login" and "confirm/send" are always performed by the human.

## Sources

- Speaker Deck, "Making Claude Code an AI secretary that grows the more you use it" (Sep 17, 2026, 74 slides) → [06](06-slides-and-talks.md)
- Qiita, "Speed up ALL your daily work with Claude Code!" → [07](07-articles.md)
- The book → [01](01-profile-and-book.md)
