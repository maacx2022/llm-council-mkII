# LLM Council v2

> Run a high-stakes decision through a council of 3–5 AI advisors with task-defined analytical contracts, blind rubric-based peer critique, and a chairman who **adjudicates rather than averages**.

## Genealogy & Lineage

```
Andrej Karpathy (Original LLM Council Methodology)
    ↓
Bruno Okamoto (okjpg/llm-council - First Implementation)
    ↓
LLM Council v2 (2025 Multi-Agent Architecture Rebuild)
    ↓
[OpenRouter Fusion Council - v2 Update]
    └─ Council Members (via Fusion):
       • Claude 3.7 Sonnet (Red Team / Pre-mortem)
       • Fable 5 (Evidence & Base Rates)
       • GPT 5.6 Sol (Customer Voice / Strategic)
       • Gemini 3.1 Pro (Unit Economics / Path-to-Monday)
       • DeepSeek v4 Flash 0731 (Second-Order Effects)
       └─ Chairman: Claude 4 (Opus 4.7) - Adjudication & Confidence Calibration
```

This version (`maacx2022/llm-council-mkII`) represents a structural rebuild grounded in 2025 multi-agent research, evaluated and refined through an OpenRouter Fusion council run with the strongest contemporary models.

---

Originally inspired by [Andrej Karpathy's LLM Council methodology](https://x.com/karpathy). v2 is a structural rebuild grounded in 2025 multi-agent research: **ColMAD**, Council Mode, "Nine Judges, Two Effective Votes," "Judging the Judges," and Anthropic's multi-agent research.

## What's New in v2

v1 was a faithful implementation of Karpathy's original pattern. Since then, the multi-agent literature has moved fast — and several v1 design choices are now demonstrably suboptimal. v2 keeps the same *intuition* but overhauls structure, role design, critique mechanics, and confidence calibration.

| Layer | v1 | v2 |
|---|---|---|
| **Roles** | 5 fixed thinking-style personas (Contrarian, First Principles, Expansionist, Outsider, Executor) | Hybrid casting: 2 fixed **task-defined contracts** (Red Team, Evidence & Base Rates) + 1–3 dynamic domain roles |
| **Panel size** | Always 5 | 3 by default, 5 only for high-stakes (correlated-error research shows returns collapse fast) |
| **Peer review** | Holistic "which is best?" ranking | Rubric scoring (0–3 across 5 criteria) + **factual vs emphasis** disagreement tagging |
| **Chairman** | Synthesizes agreements/clashes/blind spots | **Adjudicates**: forced CoT, ban on compromise verdicts, mandatory devil's-advocate pass under unanimity, **mechanical confidence calibration** |
| **Triage** | None — always runs when triggered | Pre-flight classification: is this even council-worthy? Standard or high-stakes? |
| **Framing** | "Framed question" string | Structured **Decision Brief** artifact (options, constraints, success criteria, reversibility, tripwires) |
| **Output** | Verdict + agreements + clashes + blind spots + recommendation + next step | Verdict + confidence (HIGH/MODERATE/LOW) + why + **steelmanned dissent** + falsifiers + one next action + counter-narrative |
| **Artifacts** | HTML report + MD transcript | HTML report + MD transcript + **YAML decision journal** (for targeted re-runs when facts change) |
| **High-stakes extras** | — | ColMAD-framed targeted resolution round on factual disagreements only |

## Why the Changes

- **Personas are theater, not decorrelation.** EMNLP 2024 ("When A Helpful Assistant Is Not Really Helpful") shows persona prompts on the same model add stylistic flavor but don't decorrelate errors. Analytical *contracts* (falsifiable constraints on reasoning) do.
- **Bigger panels don't help.** Kohli et al. ("Nine Judges, Two Effective Votes") — 9 frontier models yield only ~2–2.5 effective independent votes. Panel almost never beats its best member. 3–5 is empirically optimal.
- **Naïve debate can *degrade* accuracy.** "Talk Isn't Always Cheap" (2025) and "When and Why Does Multi-Agent Debate Fail" (2025) document sycophancy cascades and premature consensus. v2 keeps critics anonymous, randomizes order, and rewards dissent.
- **Chairmen tend to average.** v2 explicitly bans compromise verdicts, forces adjudication, allows majority override with justification, and requires a steelman-against-consensus whenever advisors are split.
- **Self-reported confidence is poorly calibrated.** v2 derives confidence *mechanically* from structural signals: unanimity + steelman survival + fact verification → HIGH; unresolved factual clashes → LOW.
- **Format bias is the biggest judge bias.** "Judging the Judges" (2025) — judges prefer markdown 73–97% of the time vs 57% for humans. v2 lightly normalizes response formatting before critique.

## The Honest Caveat

This skill runs inside Claude with sub-agents. That means all advisors are Claude instances — we cannot get true cross-family model diversity, which the research clearly favors. v2 compensates via:

1. Different **task contracts** per advisor (proven to matter more than persona flavor).
2. **Interpretation diversity** — different question framings (arXiv 2507.21168).
3. Temperature variation across sub-agents.
4. Explicit rule: **treat unanimity as a hypothesis, not proof.** Same-family councils are especially vulnerable to shared training biases — hence the mandatory devil's-advocate pass.

If you have OpenRouter Fusion or another cross-provider setup, true model heterogeneity is superior. This skill approximates the pattern within Claude's constraints.

## How It Works

1. **Triage** — Is this council-worthy? What stakes? Missing critical context?
2. **Enrich & Brief** — Scan workspace (`CLAUDE.md`, `memory/`, prior transcripts) → build structured Decision Brief
3. **Cast** — 2 fixed structural roles + 1–3 dynamic domain roles chosen for the specific decision
4. **Propose** — All advisors work in parallel, isolated, with a strict output schema (committal recommendation + load-bearing reasons + constraint check + falsifier + per-claim confidence)
5. **Critique** — Anonymized, order-randomized, format-normalized rubric scoring; each disagreement tagged factual or emphasis; reviewers explicitly rewarded for finding flaws
6. **Resolve** *(high-stakes only)* — Short ColMAD-framed round on factual disputes: contribute new evidence, concede correct points, no rhetoric
7. **Adjudicate** — Chairman runs forced CoT → detects unanimity → generates steelman if unanimous → derives confidence mechanically → produces verdict (adjudicated, not averaged)
8. **Report** — Visual HTML + MD transcript + YAML decision journal

## Panel Composition

**Fixed structural roles (always present):**

| Role | Analytical Contract |
|---|---|
| **Red Team / Pre-mortem** | "It's 6 months from now and this failed. List the 3 most likely failure paths — each with probability, early warning sign, and the assumption that turned out false." |
| **Evidence & Base Rates** | "What are the base rates for decisions like this? Which 2 facts in the brief are most load-bearing, and how could each be cheaply verified before committing?" |

**Dynamic domain roles (1–3, chosen per decision):**

| Decision Type | Role |
|---|---|
| Financial | Unit Economics Modeler |
| Product / positioning | Customer Voice Analyst |
| Strategic | Counterfactual Analyst |
| Execution | Path-to-Monday Analyst |
| Ethical / stakeholder | Second-Order Effects Analyst |
| Timing | Cost of Waiting Analyst |

## Triggers

**Mandatory:** `council this`, `run the council`, `war room this`, `pressure-test this`, `stress-test this`, `debate this`

**Strong (with real decision + tradeoff + stakes):** `should I X or Y`, `which option`, `is this the right move`, `validate this`, `I'm torn between`, `get multiple perspectives`

**Heuristic:** *If a single-model answer to this question would start with "it depends," council it. If it would start with a fact, don't.*

## When to Use

✅ Decisions with real stakes and multiple viable options
✅ Contested judgment calls (reasonable people would disagree)
✅ Hard-to-reverse choices where being wrong is expensive
✅ Multi-dimensional tradeoffs across incommensurable criteria

❌ Factual lookups (councils don't fix facts — use tools/verification)
❌ Creation tasks (write me a tweet)
❌ Processing tasks (summarize this)
❌ Reversible low-stakes choices

**Cost warning:** A council run is 8–15 sub-agent calls — roughly 5–15× the tokens of a normal answer. Only worth it when the decision matters.

## Install

Copy `SKILL.md` to your Claude Code skills folder:

```bash
# Project-level
cp SKILL.md .claude/skills/llm-council/SKILL.md

# Global (all projects)
cp SKILL.md ~/.claude/skills/llm-council/SKILL.md
```

Or for [OpenClaw](https://openclaw.ai) users, place in your workspace `skills/` directory.

## Output

Every high-stakes council session produces three artifacts:

- `council-report-[timestamp].html` — Visual briefing: verdict banner (with color-coded HIGH/MODERATE/LOW confidence badge), one thing to do first, why, **strongest case against** (equal visual weighting), steelman on dissent, call-to-action with tripwires.
- `council-transcript-[timestamp].md` — Full transcript with de-anonymized advisor responses, all peer reviews with rubric scores, resolution outcomes (if any), and the chairman's synthesis.
- `council-journal-[timestamp].yaml` — Compact decision record (verdict, confidence, key assumptions, tripwires, review date) enabling later targeted re-runs when facts change.

## Post-Council Re-runs

When you tell the council *"we assumed X, but actually Y"* or *"here's new info"*, v2 doesn't re-run the whole thing. It loads the previous transcript + journal, identifies which advisor conclusions depend on the changed fact, and re-runs only those advisors + the chairman.

## Failure Modes v2 Actively Guards Against

- **Sycophancy cascade** — parallel blind critique; "rewarded for finding flaws" instruction
- **Majority anchoring** — chairman sees individual positions with scores, not vote tallies
- **False consensus** (biggest risk in same-family councils) — mandatory devil's-advocate steelman under unanimity
- **Chairman averaging** — explicit ban on compromise verdicts; forced adjudication
- **Format/verbosity/position bias in critique** — light format normalization + rubric scoring + per-reviewer randomization
- **Self-preference bias** — advisors don't review themselves
- **Overconfident synthesis** — mechanical confidence derivation from council structure
- **Context collapse** — rubric criterion "Uses provided context" + explicit constraint block in brief
- **Genericness cascade** — advisors must commit with a falsifier; "it depends" without a variable is banned
- **Persona theater** — roles are analytical contracts, not characters

## Credits

- **Original methodology:** [Andrej Karpathy](https://x.com/karpathy)
- **First implementation:** [Bruno Okamoto](https://x.com/obrunookamoto) (`okjpg/llm-council`)
- **v2 architecture & research grounding:** 2025 multi-agent studies (Kohli et al., ColMAD, Council Mode, "Judging the Judges," "Talk Isn't Always Cheap," Anthropic's multi-agent research system)
- **v2 evaluation & refinement:** OpenRouter Fusion council (Fable 5, GPT 5.6 Sol, Gemini 3.1 Pro, DeepSeek v4 Flash 0731, with Claude 4 Opus 4.7 as chairman)

## License

MIT
