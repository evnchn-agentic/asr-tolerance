---
name: asr-tolerance
description: Use when an incoming prompt shows signs of being an automatic-speech-recognition / dictation transcript — missing or erratic punctuation, run-on phrasing, false starts or repetition, near-homophone substitutions, mangled technical jargon or proper nouns, or corruption around exact identifiers (paths, branch/package/model names, numbers, dates). Also when a user dictates prompts by voice and you must decode intent from lossy text without fabricating over the gaps. The literal-checking floor applies on every dictated turn, even clean-looking ones — a high-quality recognizer fails rarely but its misses are fluent enough to slip a garble-density gate.
---

# ASR-tolerance — decode lossy speech-to-text without confabulating

## Overview

ASR (automatic speech recognition) output is **fluent-but-wrong, and its errors are often not surfaced** to the agent or the user-facing interface. A dictated prompt arrives as clean-looking text that may contain silent substitutions. Worst of all, a *capable* model's own smooth decode of a mangled term **feels certain even when it's wrong** — so the error is never flagged.

**Core insight:** a capable agent already clarifies *obvious* vagueness ("every so often" → asks for an interval). The real trap is **silent literal-decoding** — the agent reads a mangled proper noun, number, or identifier as the nearest plausible thing and never flags it. This skill's job is to make that decode **visible**, with a clear threshold so it protects without nagging. The market rewards confident-shaped output — a smooth wrong transcript feels finished while an honest "this part is uncertain" feels incomplete — so being the agent that marks the gap is the whole value.

## The quality spectrum — a risk tendency, not a law

ASR sources vary widely: a coding-tuned in-CLI dictation mode primed with your project/branch names; a decent mobile dictation engine; a poor budget-phone recognizer; a lossy long-range voice relay. The naive response is "be careful with the bad ones, relax on the good ones." That instinct is dangerous, because of a **tendency** (not an absolute law):

> **Higher-quality ASR lowers the total error count, but its *remaining* errors are more likely to be fluent enough to escape a garble-density gate.**

| Source quality | Error rate | Errors *tend* to be… | Why it matters |
|---|---|---|---|
| **Poorer** (budget mobile, lossy relay) | Higher | more often **loud** — obvious garble that's hard to miss | you usually notice and clarify |
| **Higher** (coding-tuned dictation, good mobile) | Lower | more often **quiet** — a clean, plausible substitution | a single wrong literal slips through |

Only a tendency: a poor recognizer can *also* silently swap a common word, and a good one can fail *loudly* on a rare name, accent, crosstalk, or clipping. The actionable point is the asymmetry of *consequence*: **don't relax literal-checking just because the prose reads clean** — that is exactly when the rare, fluent miss hides. Hence the literal floor below is source-independent.

## Engagement: a floor + a dial, not a binary gate

Split the old "should I engage?" decision into two.

**1. The always-on literal floor (every dictated turn, any source, however clean).**
Watch the high-error zone — paths, commands, branch / package / model names, numbers, dates, proper nouns — but **surface a decode only when the literal is one of:**
- **action-critical** (about to enter a command, edit, target, or destructive op), or
- **ambiguous / newly-introduced** — you can't verify it against the project, files, or prior context, or
- **high-cost if wrong**, or
- **repaired from a non-literal transcript token** (you changed what was written).

You may **silently accept** a low-risk literal that **exact-matches** copied context, a quoted string, or a project lookup — surfacing those is the nagging failure mode. The floor does not wait for "2+ signs of garble"; it waits for *a literal that meets the bar above*.

Examples — **surface:** `clocked code`→`Claude Code` (proper-noun repair feeding a task); "delete branch `main`" (destructive + action-critical); "July 15" when June 15 is equally plausible and it drives scheduling; a package/model name with no exact match in the project.
**Don't surface:** ordinary common nouns; a path that appears verbatim in a pasted code block; a version number you can confirm exact against the lockfile; a date irrelevant to the action.

**2. The depth dial (clarification, read-back aggressiveness, re-grounding effort) scales with _observed_ error-sign density.** Signs include: erratic punctuation; run-on phrasing; false starts, self-corrections, repetition; odd profanity or word-boundary mis-segmentation; near-homophone substitutions; mangled jargon or proper nouns; suspicious corruption near exact identifiers. More signs → clarify more, read back more, re-ground harder. A clean transcript → lighter touch *on everything except the literal floor*.

**Source inference from transcript text is weak and must not control safety behavior.** A near-clean dictation-mode turn and a clean mobile turn are largely indistinguishable in text; weak signals exist (dictated-punctuation words like "comma"/"period", autocapitalization quirks, platform substitutions) but aren't reliable enough to branch behavior on. Use the source profiles below only as *priors for where errors tend to land* — never as a detector. Allow a stronger prior **only** when the user explicitly names the source or the transcript carries unmistakable artifacts.

Do **not** engage the *dial* solely because of casual style or ordinary typos. Structured artifacts (code blocks, exact paths, flags, quoted strings) are strong evidence of typed input **only when they are internally consistent and not otherwise ASR-corrupted** — a dictated `dash dash force` or a misheard path still needs protection. Over-firing nags; under-firing buries a wrong decode. When unsure, a one-line *"reading this as dictated — confirming X"* is cheaper than either mistake.

### Source profiles — calibration priors, NOT a detector

- **Coding-tuned in-CLI dictation** (streaming recognizer primed with project/branch names + dev vocabulary): strong on common dev terms and your project's own nouns; **blind spots = novel proper nouns and product/person names outside the hint set.** Fails rarely; the misses are quiet.
- **Mainstream mobile dictation**: solid on everyday language; weaker on technical jargon and acronyms; occasional near-homophone slips.
- **Budget / older mobile, or a lossy voice relay**: high garble, mostly loud — mangled words, dropped clauses, repetition.

Treat these as "where to spend skepticism," then verify against the actual text — never assert a source.

## The discipline

1. **Decode only what the transcript supports; mark the rest** with coarse confidence (high/medium/low). Smooth-and-wrong is worse than rough-and-honest.
2. **Protect literals — and show your work (within the floor's threshold).** Cross-check against the project/context when available, then state any surfaced decode inline: *"reading `night gooey`→`NiceGUI`, `a sink`→`asyncio` — correct?"* A visible wrong decode gets caught; a buried one ships.
   - **How to avoid *inventing* a decode (the hard case — a plausible literal that might already be correct):**
     1. **Exact-match first.** If the token exactly matches a project / context / quoted value, accept it; don't "repair" it.
     2. **No exact match → propose at most one likely repair, with confidence.** Don't silently normalize to the nearest project term.
     3. **Multiple plausible repairs, or a high-impact action → ask**, don't pick.
     4. **Never rewrite a literal inside an actual command** without confirmation, unless the action is reversible *and* you state the assumption.
3. **Context ranks; it never invents.** Use context only to rank plausible readings of text *actually present*. Never fabricate missing content from what's merely plausible for the project — that produces fluent confabulations indistinguishable from a real reading. A context-suggested repair is an assumption unless the transcript strongly supports it.
4. **Ask one concise clarification** if the core objective, target, or action is uncertain — don't fan out five questions.
5. **Read back before commitment.** Irreversible or shared-impact actions, broad edits, long autonomous runs, destructive commands, external messages, deploys → read the decoded intent back first, even if technically reversible. A bad decode can waste a whole overnight run. Cheap/reversible work may proceed on a *stated* assumption.

## The confidence blind spot

Across most degradation, an ASR engine's self-reported uncertainty *tracks* its real error rate — it tends to know when it's struggling. But it can be **confidently wrong with near-zero self-doubt** at both extremes: heavy degradation (it asserts garbage), and a high-quality recognizer's rare miss (a clean substitution amid clean prose). The lesson generalizes to your own decoding: **do not treat the smoothness of a decode as evidence it's right** — not the ASR's confidence, not your own. Empirically, the missed tokens are overwhelmingly **proper nouns and near-homophones**; that is where to spend skepticism first.

## Read-back template

> Reading your dictated message as: **\<one-line decoded intent\>**.
> Decoded literals: `transcript`→`decoded` (confidence), … · Unrecovered: "\<garble\>" [?].
> Reversible → I'll proceed on the assumption that \<X\>; say if I misheard.
> Irreversible/long-run → confirm before I start?

---

*The quality-spectrum / floor-and-dial model is grounded in real dictation samples observed across recognizer tiers.*
