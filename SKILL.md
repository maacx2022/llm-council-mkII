---
name: llm-council
description: "Run a high-stakes decision through a council of 3–5 AI advisors with task-defined analytical contracts (not persona theater), a blind rubric-based critique round, and a chairman who adjudicates rather than averages. Upgraded Nov 2025 with 2025 research: n_eff correlation limits, ColMAD critique framing, mechanical confidence calibration, forced devil's-advocate pass under unanimity, factual-vs-emphasis disagreement taxonomy. MANDATORY TRIGGERS: 'council this', 'run the council', 'war room this', 'pressure-test this', 'stress-test this', 'debate this'. STRONG TRIGGERS (only with a real decision + tradeoff + stakes): 'should I X or Y', 'which option', 'is this the right move', 'validate this', 'I'm torn between', 'get multiple perspectives'. HEURISTIC: if a single-model answer to this would start with 'it depends,' council it; if it would start with a fact, don't. DO NOT trigger on: factual lookups, drafting/creation tasks, reversible low-stakes choices, casual 'should I' without a meaningful tradeoff. DO trigger when the user presents a genuine decision with stakes, multiple options, and context that warrants adversarial pressure-testing."
---

# LLM Council v2

You ask one AI a question, you get one answer. That answer might be great. It might be mid. You have no way to tell because you only saw one perspective.

The council fixes this — but only for the right questions, and only if it's built to avoid the failure modes that plague naïve multi-agent systems: sycophancy cascades, majority anchoring, false consensus, and chairman-averaging.

This v2 is grounded in 2025 research (Karpathy's llm-council pattern, ColMAD, Council Mode, "Nine Judges Two Effective Votes," "Judging the Judges," Anthropic's multi-agent engineering lessons). The upgrades from v1 are structural, not cosmetic.

---

## TL;DR — The Flow

```
Trigger → Triage (council-worthy? stakes?) → Context enrichment + Decision Brief
       → Cast advisors (2 fixed structural + 1–3 dynamic domain)
       → Parallel isolated proposals (standard schema, committal)
       → Blind anonymized rubric critique (factual vs emphasis tagging)
       → [High-stakes only: targeted ColMAD resolution on factual disputes]
       → Chairman: forced CoT → devil's-advocate-under-unanimity → adjudicated verdict
       → Mechanical confidence + falsifiers + one next action
       → HTML report + MD transcript + decision journal entry
```

---

## What Changed From v1 (For Users Familiar With the Old Version)

- **No more 5 fixed personas** (Contrarian, First Principles, Expansionist, Outsider, Executor). Research since v1 (EMNLP 2024, "When A Helpful Assistant Is Not Really Helpful") shows personas add flavor, not decorrelation. Roles are now defined by **analytical task/deliverable**, not character.
- **Panel size is 3 by default, 5 only for high-stakes.** Correlated-error research (Kohli et al., "Nine Judges, Two Effective Votes") shows returns collapse fast.
- **Peer review is now rubric-scored**, not holistic "pick the best." Reviewers classify each disagreement as *factual* or *emphasis*.
- **Chairman is now an adjudicator**, forbidden from averaging, required to run a devil's-advocate pass on unanimous verdicts, and derives confidence mechanically from council structure — not from self-report.
- **New Triage phase** decides whether a council is warranted at all.
- **Decision Brief** is now a structured artifact, not just "the framed question."

---

## The Honest Caveat

Because this skill runs inside Claude with sub-agents, all advisors are Claude instances. That means we cannot get true cross-family model diversity (which the research clearly favors). We compensate with:
1. Different **task contracts** per advisor (proven to matter more than persona flavor).
2. **Interpretation diversity** — different framings of the same question (arXiv 2507.21168 shows this often equals model diversity for judgment tasks).
3. Temperature variation across sub-agents.
4. Explicit acknowledgment: **treat unanimity as a hypothesis, not a truth.** Claude-only councils are especially vulnerable to shared training biases.

If a user has OpenRouter Fusion or similar cross-provider access, real model heterogeneity is superior. This skill approximates the pattern within Claude's constraints.

---

## When to Run the Council

The council is for questions where being wrong is expensive AND the answer is genuinely contested.

**Good council questions:**
- "Should I launch a $97 workshop or a $497 course?"
- "Which of these 3 positioning angles is strongest?"
- "I'm thinking of pivoting from X to Y. Am I crazy?"
- "Here's my landing page copy. What's weak?"
- "Should I hire a VA or build an automation first?"

**Bad council questions:**
- Factual lookups ("what's the capital of France")
- Creation tasks ("write me a tweet")
- Processing tasks ("summarize this article")
- Reversible low-stakes choices ("should I use markdown here")

**The heuristic:** If a single-model answer would start with *"it depends,"* council it. If it would start with a fact, don't.

**Cost warning:** A council run is 8–15 sub-agent calls. Roughly 5–15× the tokens of a normal answer. Only worth it when the decision matters.

---

## Session Flow

### Phase 0 — Triage

Before doing anything else, silently classify:

1. **Council-worthy?** Genuinely contested + consequential + multi-dimensional + hard-to-reverse? If not → answer directly (optionally with one self-critique pass) and skip the rest of this skill.
2. **Stakes level?**
   - **Standard**: 3 advisors, one critique round, no resolution round.
   - **High-stakes**: 5 advisors, one critique round, targeted ColMAD resolution round on factual disagreements, devil's-advocate steelman under unanimity is mandatory.
3. **Missing critical context?** If yes, ask ONE clarifying question before proceeding. Just one. If the user's original message was rich, skip this.

### Phase 1 — Context Enrichment & Decision Brief

**A. Workspace scan (≤30 seconds).** Use `Glob` and `Read` to find:
- `CLAUDE.md` / `claude.md` in project root
- Any `memory/` folder (audience, voice, business specifics, past decisions)
- Files the user explicitly referenced
- Recent council transcripts in this folder (to spot re-councils and prior verdicts)

**B. Construct the Decision Brief.** This is a structured artifact every advisor receives verbatim:

```
DECISION BRIEF
==============
Decision statement: [what is actually being decided, in one sentence]
Options on the table: [explicit list — if user didn't specify, extract from message]
Hard constraints: [must-haves, non-negotiables — from brief + workspace]
Success criteria: [what would make this decision "right" 6 months from now]
Reversibility: [reversible / expensive to reverse / one-way door]
Time horizon: [when does this need to be decided; when do outcomes materialize]
Known facts and numbers: [from user + workspace — be specific, cite sources]
Unknowns / assumptions in play: [what's not verified, what's assumed]
Stakes: [why this matters — cost of being wrong]
```

If a critical field is unknown, either extract-with-assumption (label it) or ask the user (only if it would change the verdict).

**C. Cast the advisors.** Hybrid: 2 fixed + 1–3 dynamic.

**Fixed structural roles (always present):**
- **Red Team / Pre-mortem Analyst** — "It is 6 months from now and this decision has failed. List the 3 most likely failure paths, each with a probability, an early warning sign, and the assumption that turned out false."
- **Evidence & Base Rates Analyst** — "What are the base rates for decisions like this? Which 2 facts in the brief are most load-bearing for the recommendation, and how could each be cheaply verified before committing?"

**Dynamic domain roles (pick 1–3 based on the decision):**
- Financial decisions → "Unit Economics Modeler" ("model the cash flow under three plausible scenarios; identify the break-even sensitivity")
- Product/positioning → "Customer Voice Analyst" ("channel the target audience — what would they actually notice, want, resist")
- Strategic → "Counterfactual Analyst" ("what would happen if we did nothing / the opposite; what are we assuming about the alternative")
- Execution → "Path-to-Monday Analyst" ("what are the first 3 concrete steps and what would block each")
- Ethical/stakeholder → "Second-Order Effects Analyst"
- Timing → "Cost of Waiting Analyst"

Choose contracts that create genuine reasoning-trace divergence, not personality variety.

### Phase 2 — Parallel Isolated Proposals

Spawn all advisors simultaneously as sub-agents. Each is isolated (no visibility into other advisors). Vary temperature slightly across advisors (e.g., 0.5, 0.7, 0.9) to encourage divergent framings.

**Advisor prompt template:**

```
You are the [ROLE NAME] on an LLM Council.

Your analytical contract:
[the specific task-defined contract for this role — e.g., "Identify the 3 most likely failure paths in this decision within 6 months. For each: probability estimate (low/med/high), an early warning sign, and the specific assumption that would need to be false."]

Decision Brief:
---
[full Decision Brief from Phase 1]
---

Produce your response in this exact schema:

RECOMMENDATION: [Your committal answer to the decision. No "it depends" without specifying on what. If your analytical contract doesn't produce a recommendation directly, state what your findings imply for the decision.]

LOAD-BEARING REASONS: [2–3 bullets. The specific reasons that carry your recommendation. Reference facts from the brief.]

KEY ASSUMPTIONS: [What you're assuming that isn't in the brief. Be specific.]

CONSTRAINT CHECK: [Address each hard constraint from the brief. Does your recommendation satisfy it?]

FALSIFIER: [Complete: "I would change my recommendation if ___"]

CONFIDENCE PER CLAIM: [For each load-bearing reason, mark: verified / plausible-but-untested / speculative]

Rules:
- Do not hedge or try to be balanced. Lean fully into your analytical contract.
- Use the specific facts and numbers from the brief. Generic advice is a failure mode.
- If the brief lacks information you need, state the assumption you're making and continue.
- 200–350 words total.
- No preamble. Start with RECOMMENDATION.
```

### Phase 3 — Blind Structured Critique

Collect all advisor responses. Prepare them for review:

1. **Anonymize:** Label as Answer A, B, C, D, E. Strip role names.
2. **Randomize order per reviewer** (each reviewer sees a different randomization).
3. **Light format normalization** (equalize markdown usage, bullet vs paragraph — this counters the biggest documented judge bias: format preference).
4. **Never let a reviewer review their own answer.**

Spawn one reviewer sub-agent per advisor.

**Reviewer prompt template:**

```
You are reviewing peer analyses on an LLM Council. Five (or three) independent advisors answered the same decision. Their responses have been anonymized and randomized.

Decision Brief:
---
[full Decision Brief]
---

Anonymized responses:

**Answer A:** [response]
**Answer B:** [response]
**Answer C:** [response]
[...]

For EACH answer you see, produce this rubric score (0–3 each, with a 1-sentence justification):

- Constraint adherence: does it actually address the hard constraints from the brief?
- Reasoning soundness: are the load-bearing reasons logically sound and evidence-backed?
- Addresses strongest counterargument: does it engage with what might be wrong with its own view?
- Actionability: could a decision-maker act on this Monday morning?
- Uses provided context: does it use the specific facts/numbers from the brief, or could it have been written without them?

Then, list every meaningful DISAGREEMENT you see between the answers. For each, classify it:
- **FACTUAL disagreement** — advisors make incompatible claims about facts, base rates, or verifiable outcomes.
- **EMPHASIS disagreement** — advisors weigh the same facts differently or emphasize different aspects.

Finally, answer: What did ALL responses miss that the council should consider?

Rules:
- You are rewarded for finding flaws, not for agreeing. Politeness is a failure mode here.
- Reference answers by letter.
- Be specific. Generic critiques are useless.
- Keep total review under 350 words.
```

### Phase 3.5 — Targeted Resolution Round (HIGH-STAKES ONLY)

Skip this phase for standard-stakes councils.

For each flagged **factual** disagreement (emphasis disagreements are surfaced but not re-litigated), spawn a short resolution sub-agent using ColMAD framing:

```
Two advisors on an LLM Council made incompatible factual claims:

Claim from Advisor [X]: [claim + reasoning]
Claim from Advisor [Y]: [claim + reasoning]

Decision Brief context: [relevant portion]

Your job (collaborative, not competitive):
1. Which claim can be supported with more verifiable evidence?
2. What NEW information or reasoning bears on this that neither advisor cited?
3. Concede explicitly what is correct in each position.
4. State the resolution: which claim survives, with what qualifications?

No rhetoric. Only evidence and verifiable reasoning. 150 words max.
```

Collect resolutions. Feed to chairman.

### Phase 4 — Chairman Synthesis

The chairman receives EVERYTHING: brief, de-anonymized raw advisor responses, all peer reviews with scores, disagreement map, resolution outcomes (if any).

**Chairman prompt template:**

```
You are the Chairman of an LLM Council. Your job is to ADJUDICATE — not to average, not to summarize, not to hedge.

Decision Brief:
---
[full brief]
---

ADVISOR RESPONSES (de-anonymized):
[each advisor's full response, labeled by role]

PEER REVIEWS (with rubric scores and disagreement classification):
[all reviews]

[If high-stakes: RESOLUTION OUTCOMES on factual disagreements]

STEP 1 — Forced Analysis (do this fully before writing the verdict):
For each advisor's recommendation, evaluate against:
- Does it address the hard constraints?
- Are its load-bearing reasons factually supported (per peer review)?
- Did it survive critique, or did reviewers find fatal flaws?
- Is its falsifier realistic — could that condition actually be tested?

STEP 2 — Detect Unanimity:
Are all advisors converging on the same recommendation? If YES, you MUST generate the strongest steelman AGAINST the consensus and address it before publishing high confidence. False consensus among Claude instances is a known failure mode.

STEP 3 — Determine Confidence Mechanically:
- HIGH: unanimity + steelman found no load-bearing objection + key facts are verified (not assumed) + dissent (if any) was decisively answered
- MODERATE: majority agreement with substantive dissent that was addressed; OR unanimity where key facts remain unverified
- LOW: unresolved factual disagreements; OR verdict hinges on assumptions nobody could verify; OR reviewers found significant flaws in the leading recommendation

Do NOT self-report confidence. Derive it from the council's structural signals above.

STEP 4 — Produce the Verdict:

Use this exact structure. Do not deviate.

## VERDICT
[One sentence. Committal. No "consider" / "it depends" / "weigh both." If the answer is genuinely conditional, state the condition: "Do X if Y, otherwise Z." A real answer.]

## CONFIDENCE: [HIGH / MODERATE / LOW]
[One sentence explaining WHY this confidence level, referencing the specific structural signals from Step 3.]

## WHY
[3 load-bearing bullets. Each references specific evidence, brief facts, or peer-review findings. Attribute to advisor reasoning where relevant.]

## STRONGEST CASE AGAINST
[The steelmanned dissent. If unanimity, this is the devil's-advocate you generated in Step 2. This section is mandatory — a verdict that can't articulate its best counterargument shouldn't ship.]

## WHAT WOULD CHANGE THE ANSWER
[2–3 falsifiers / tripwires. Concrete conditions that, if observed, should trigger revisiting.]

## THE ONE THING TO DO FIRST
[A single concrete next step. Not a list. Preferably reversible or information-generating so it de-risks the larger decision.]

## COUNCIL MAP
- **Where advisors converged:** [points of genuine agreement — treat as hypothesis, not proof]
- **Where advisors clashed (factual):** [resolved via resolution round if high-stakes; otherwise flagged]
- **Where advisors clashed (emphasis):** [values/weighting disagreements — these often can't be "resolved," only surfaced]
- **Unique findings:** [important points only one advisor raised that survived critique]

RULES:
- You may override the majority. If you do, name specifically which advisor's reasoning you found stronger and why.
- You may NOT produce a compromise verdict. Adjudicate. If you find yourself writing "consider both," rewrite.
- No hedging. No "it depends" without specifying on what.
- The user came to the council for clarity they can't get from a single answer. Deliver it.
```

### Phase 5 — Generate Artifacts

Produce three artifacts:

**A. `council-report-[timestamp].html`** — visual report.

Single self-contained HTML file with inline CSS. Structure:
1. Header: the decision statement + timestamp + stakes level
2. **VERDICT banner** (prominent, top of page): verdict + confidence badge + one-line confidence rationale
3. **The one thing to do first** (also prominent)
4. **Why** (3 bullets)
5. **Strongest case against** (equal visual weight to Why — this is what makes it trustworthy)
6. **What would change the answer** (falsifiers)
7. **Agreement/disagreement visual**: simple table or grid showing which advisors converged, where the factual clashes were (and resolutions if any), where the emphasis clashes were
8. **Collapsible sections** for each advisor's full response
9. **Collapsible section** for peer reviews with rubric scores
10. Footer: brief metadata + note about council limitations (same-family models)

Design: clean white background, system-font stack, subtle color for the confidence badge (green/amber/red for high/moderate/low), soft borders. Professional briefing document — not a dashboard.

Open the HTML after generation.

**B. `council-transcript-[timestamp].md`** — full transcript for reference.

Contains: original question, full Decision Brief, advisor casting rationale, all raw advisor responses (de-anonymized), anonymization mapping revealed, all peer reviews with scores, resolution outcomes if any, chairman's full synthesis, timestamp.

**C. `council-journal-[timestamp].yaml`** — decision journal entry (small).

```yaml
timestamp: [ISO 8601]
decision: [one-line decision statement]
verdict: [one-line verdict]
confidence: HIGH / MODERATE / LOW
stakes: standard / high
key_assumptions:
  - [assumption 1]
  - [assumption 2]
tripwires:
  - [falsifier 1]
  - [falsifier 2]
review_after: [suggested review date or milestone]
transcript_file: council-transcript-[timestamp].md
```

This enables future targeted re-runs and outcome tracking.

### Phase 6 — Post-Council Support

If the user later says something like "the council assumed X, but actually Y" or "here's new info":

- Do NOT re-run the whole council.
- Load the previous transcript + journal.
- Identify which advisor conclusions depended on the changed fact.
- Re-run only affected advisors (usually 1–2) with the updated brief.
- Have chairman produce a diff verdict: "Previously: X. Now: Y. What changed: Z."
- Update the decision journal.

---

## Failure Modes This Skill Actively Guards Against

- **Sycophancy cascade** → critique is parallel and blind; reviewers never see revisions of others; "rewarded for finding flaws" instruction.
- **Majority anchoring** → chairman sees individual positions with scores, not raw vote tallies.
- **False consensus** (biggest risk on same-family council) → mandatory devil's-advocate pass under unanimity; unanimity is not automatically HIGH confidence.
- **Chairman averaging** → explicit ban on compromise verdicts; forced adjudication; override permission with justification.
- **Verbosity/format bias in critique** → light format normalization before critique; rubric scoring (not holistic).
- **Position bias in critique** → per-reviewer randomization.
- **Self-preference bias** → advisors don't review themselves.
- **Overconfident synthesis** → mechanical confidence derivation from council structure, not self-report.
- **Context collapse** (advisors ignoring provided facts) → rubric criterion "Uses provided context"; brief has explicit constraint block that each advisor must address.
- **Genericness cascade** → advisors must commit to a recommendation with falsifier; "it depends" without variable is banned.
- **Persona theater** → roles are analytical contracts, not characters.

---

## Important Operational Notes

- **Always spawn advisors in parallel.** Sequential = anchoring.
- **Always anonymize for critique.** Free bias reduction.
- **Chairman may override the majority** — and often should when the minority caught a load-bearing flaw.
- **Never council trivial questions.** The Triage phase exists to prevent this. If the answer is factual or the decision is reversible and cheap, just answer directly.
- **Treat unanimity as a hypothesis, not proof.** Especially in a same-family council.
- **The HTML report is what most users will read.** Verdict banner + strongest-case-against are the two most important elements.
- **The decision journal turns one-shot answers into a decision-support relationship over time.** Encourage users to revisit tripwires.
