# asr-tolerance

**An agent skill for reading dictated prompts without quietly making things up.**

> *Quantity over quality of input — a paragraph of rambling beats a few precise sentences. That's what cranks the "Large" in an LLM.*

Voice input is leverage. Talking is 3–4× faster than typing, so dictation lets you pour far more context into a model than your thumbs ever could — and more context is more for the model to work with. The catch is in the pipe: **automatic speech recognition fails fluent-but-wrong, and the error is usually never surfaced.** `night gooey` becomes `NiceGUI`, `a sink` becomes `asyncio`, `the off slash fix dash timing branch` becomes a guess — and a capable model's own decode of a mangled term *feels certain even when it's wrong*, so it sails through unflagged.

This skill is what lets you crank the volume without feeding the model lies.

## The one insight

A capable agent already asks about *obvious* vagueness ("every so often" → "what interval?"). The failure that actually bites is **silent literal-decoding**: a mangled proper noun, number, branch name, or model id gets read as the nearest plausible thing and is never surfaced. So the rule is not "stop guessing" (that just nags) — it's:

- **Make the decode visible.** *"reading `night gooey`→`NiceGUI` — correct?"* A wrong decode you can see gets caught; a buried one ships.
- **Protect literals hardest.** Paths, flags, numbers, branch/package/model names, proper nouns degrade first and worst.
- **Don't trust smoothness.** Neither the ASR's confidence nor your own is evidence a decode is right.
- **Read back before anything irreversible** — a push, a deploy, a long autonomous run. A bad decode can burn a whole overnight job.
- **Context ranks readings; it never invents missing words** — that's how you avoid confabulating a fluent-but-fictional prompt.

## Install

Drop the skill folder into your agent's skills directory (e.g. for Claude Code, `~/.claude/skills/asr-tolerance/`). Most harnesses auto-discover it by the `description` in the frontmatter and engage it when a prompt shows signs of being dictated. The full discipline — when to engage, the read-back template, the confidence blind spot — lives in [`SKILL.md`](./SKILL.md).

## Why it's framed as "tolerance"

You don't fix ASR by demanding cleaner speech — that just moves the friction back onto the human, which defeats the point of dictating. You make the *reader* tolerant: decode what's supported, flag what isn't, and never let a confident-shaped guess pass for a fact. Rough-and-honest beats smooth-and-wrong.

## License

MIT — see [`LICENSE`](./LICENSE).
