---
name: asr-tolerance
description: Use when an incoming prompt shows signs of being an automatic-speech-recognition / dictation transcript — missing or erratic punctuation, run-on phrasing, false starts or repetition, near-homophone substitutions, mangled technical jargon or proper nouns, or corruption around exact identifiers (paths, branch/package/model names, numbers, dates). Also when a user dictates prompts by voice and you must decode intent from lossy text without fabricating over the gaps.
---

# ASR-tolerance — decode lossy speech-to-text without confabulating

## Overview

ASR (automatic speech recognition) output is **fluent-but-wrong, and its errors are often not surfaced** to the agent or the user-facing interface. A dictated prompt arrives as clean-looking text that may contain silent substitutions. Worst of all, a *capable* model's own smooth decode of a mangled term **feels certain even when it's wrong** — so the error is never flagged.

**Core insight:** a capable agent already clarifies *obvious* vagueness ("every so often" → asks for an interval). The real trap is **silent literal-decoding** — the agent reads a mangled proper noun, number, or identifier as the nearest plausible thing and never flags it. This skill's job is to make that decode **visible** and to recognize dictation provenance **reliably**, not by luck. The market rewards confident-shaped output — a smooth wrong transcript feels finished while an honest "this part is uncertain" feels incomplete — so being the agent that marks the gap is the whole value.

## When to engage / when not

Engage when **2+ ASR signs** appear, or one severe sign obscures meaning. Signs include: erratic punctuation; run-on phrasing; false starts, self-corrections, repetition; odd profanity or word-boundary mis-segmentation; near-homophone substitutions; mangled jargon or proper nouns; suspicious corruption near exact identifiers. **Also engage when the user states, or context indicates, that the prompt was dictated — even if the transcript looks clean.**

Do **not** engage solely because of casual style or ordinary typos. Structured artifacts (code blocks, exact paths, flags, quoted strings) are evidence of typed input **only when they are internally consistent and not otherwise ASR-corrupted** — a dictated `dash dash force` or a misheard path still needs protection. Over-firing nags; under-firing buries a wrong decode. When unsure, a one-line *"reading this as dictated — confirming X"* is cheaper than either mistake.

## The discipline

1. **Decode only what the transcript supports; mark the rest** with coarse confidence (high/medium/low). Smooth-and-wrong is worse than rough-and-honest.
2. **Protect literals — and show your work.** Paths, commands, branch/package/model names, numbers, dates, and proper nouns are the highest-error zone. Cross-check against the project/context when available, then state the decode inline: *"reading `night gooey`→`NiceGUI`, `a sink`→`asyncio` — correct?"* A visible wrong decode gets caught; a buried one ships.
3. **Context ranks; it never invents.** Use context only to rank plausible readings of text *actually present*. Never fabricate missing content from what's merely plausible for the project — that produces fluent confabulations indistinguishable from a real reading. A context-suggested repair is an assumption unless the transcript strongly supports it.
4. **Ask one concise clarification** if the core objective, target, or action is uncertain — don't fan out five questions.
5. **Read back before commitment.** Irreversible or shared-impact actions, broad edits, long autonomous runs, destructive commands, external messages, deploys → read the decoded intent back first, even if technically reversible. A bad decode can waste a whole overnight run. Cheap/reversible work may proceed on a *stated* assumption.

## The confidence blind spot

Across most degradation, an ASR engine's self-reported uncertainty *tracks* its real error rate — it tends to know when it's struggling. But at **extreme degradation**, that inverts: the engine asserts garbage with near-zero self-doubt. The lesson generalizes to your own decoding: **do not treat the smoothness of a decode as evidence it's right** — not the ASR's confidence, not your own. Jargon and proper nouns degrade first and hardest; that is where to spend skepticism.

## Read-back template

> Reading your dictated message as: **\<one-line decoded intent\>**.
> Decoded literals: `transcript`→`decoded` (confidence), … · Unrecovered: "\<garble\>" [?].
> Reversible → I'll proceed on the assumption that \<X\>; say if I misheard.
> Irreversible/long-run → confirm before I start?
