---
name: vagabond
description: >
  Three-phase cross-domain synthesis engine. Phase 1: 8 domain lenses run as
  independent parallel Sonnet agents — no shared context — and convergence nodes
  are scored by independent agreement count (N/8). Phase 2: a Resonance Walk agent
  uses the scored, pre-filtered nodes as its frequency input, then traverses
  adjacent concept space with one irrational leap. Phase 3: Opus runs two Hegelian
  dialectical cycles on the walk's landing to produce paradigm-breaking insight.
  Triggers on /skill-vagabond, "vagabond this", "cross-domain synthesis",
  "find the hidden connections", "polymath analysis", "explore the latent space",
  "find bridges between X and Y", "map this across domains", "wander through this".
  Default trigger when no other vagabond sub-skill is specified.
disable-model-invocation: false
argument-hint: "[topic or question]"
---

# Vagabond — Cross-Domain Synthesis Engine

Chains three cognitive phases — parallel lattice analysis, resonance walk, shadow
dialectic — into a single pipeline. Input: any topic, question, or tension. Output:
a paradigm-breaking insight the user could not reach by staying inside any single
domain, written to `.tmp/vagabond_{slug}.md`.

**Why it works:** The lattice finds structural truths by requiring independent
agreement across 8 domains. The walk uses those truths as a pre-calibrated frequency
to traverse concept space efficiently. The shadow forces the walk's landing into
dialectical confrontation until it reaches a position that contains the original
frame as a special case — not just a restatement of it.

---

## Arguments

- `$0` — Topic, question, or conceptual tension to analyze (required)

Output: `.tmp/vagabond_{slugified_topic}.md`

---

## Architecture

```
                           [Topic: $0]
                                │
   ┌──────┬──────┬──────┬───────┴───────┬──────┬──────┬──────┐
   ▼      ▼      ▼      ▼               ▼      ▼      ▼      ▼
[L1]   [L2]   [L3]   [L4]            [L5]   [L6]   [L7]   [L8]
 Phy    Bio    Mat    Eco              Phi    His    Lan    Art
Sonnet Sonnet Sonnet Sonnet          Sonnet Sonnet Sonnet Sonnet
                  (8 agents, parallel, no shared context)     │
   └──────┴──────┴──────┴───────┬───────┴──────┴──────┴──────┘
                                │
                  [Convergence Aggregator]          ← in-context (you)
                  score each claim: N/8 lenses
                  ≥ 3/8 → signal node
                  ≥ 5/8 → structural truth (flagged)
                                │
                         [Walk Agent]               ← Sonnet ×1
                  frequency ← derived from scored nodes
                  4 resonant domains + 1 irrational leap
                  narrative journey → landing + thread
                                │
                       [Shadow Agent]               ← Opus ×1
                  cycle 1: thesis → shadow → synthesis₁
                  cycle 2: synthesis₁ → shadow → synthesis₂
                  → new ground + residual thread
                                │
                  [.tmp/vagabond_{slug}.md]
```

---

## Phase 1 — Parallel Lattice (8 × Sonnet)

Spawn all 8 lens agents in a **single message** using multiple Agent tool calls.
Every agent uses `model: "sonnet"` and `run_in_background: true`.

**Critical constraint:** Each agent receives ONLY its own lens context. Do not pass
other agents' outputs or any aggregation to any lens agent. Independence is the
mechanism — cross-contamination collapses convergence scoring to noise.

### Lens roster

| ID | Lens | Probe question |
|----|------|---------------|
| L1 | Physics / Thermodynamics | What are the conservation laws? What forces equilibrium? What is the cost of order? |
| L2 | Biology / Evolution | What survives here? What is the selection pressure? What is the symbiosis? |
| L3 | Mathematics / Topology | What is preserved under transformation? What are the invariants? What breaks continuity? |
| L4 | Economics / Game Theory | What are the incentives? Who defects? What is the equilibrium? What is priced out? |
| L5 | Philosophy / Epistemology | What does this assume is knowable? What is it refusing to question? What does it need to be true? |
| L6 | History / Archaeology | Has this pattern appeared before? What did it build? What did it destroy? Who was erased? |
| L7 | Language / Semiotics | What does the word root reveal? What metaphors does this concept live inside? What can't be said? |
| L8 | Art / Aesthetics | What is the texture of this? What would it feel like to inhabit? What does beauty reveal that argument cannot? |

### Lens agent prompt template

Fill `{topic}`, `{lens_name}`, `{probe_question}` for each of the 8 agents.

```
You are a domain specialist conducting an independent structural analysis.
You will NOT see analyses from any other domain. Work only from your lens.

TOPIC: {topic}

YOUR LENS: {lens_name}
PROBE QUESTION: {probe_question}

INSTRUCTIONS:
1. Do NOT summarize or describe the topic. Find the structural analog.
2. State what concept from your domain is isomorphic to this topic — not similar, but
   governed by the same underlying equation or principle.
3. State what your lens reveals that a direct description of the topic suppresses or
   cannot see.
4. Extract 3 structural claims — precise, falsifiable statements about the topic that
   only your lens makes visible. These are the raw material for cross-domain convergence.
5. Do not hedge. Commit to positions. A lens that says "it depends" has failed.

OUTPUT FORMAT (exact — do not deviate):

## {lens_name}

**Structural analog:** [The concept in your domain that is isomorphic to the topic.
One sentence. Name it precisely.]

**What this lens reveals:** [What the topic's direct framing suppresses or cannot
access. 2-3 sentences.]

**Structural claims:**
- [C1] [Precise claim about the topic that your lens makes visible. One sentence.]
- [C2] [Precise claim about the topic that your lens makes visible. One sentence.]
- [C3] [Precise claim about the topic that your lens makes visible. One sentence.]

**Residual:** [One thing your lens cannot reach or explain about this topic.
One sentence.]

250–350 words total. Dense. No prose filler.
```

---

## Aggregation — Convergence Scoring (in-context)

After all 8 lens agents complete, collect their outputs and build the convergence map.
Do this yourself (in-context) — do not spawn an agent for this step.

### Algorithm

1. **List all 24 structural claims** (3 per lens × 8 lenses) in a table.

2. **Cluster semantically**: Group claims that are asserting the same underlying
   structural truth in different domain vocabularies. A cluster is not "similar
   themes" — it is claims that would be explained by the same formal statement if
   translated into abstract terms.

3. **Score each cluster**: Score = number of lenses that independently contributed
   a claim to the cluster ÷ 8.
   - Score ≥ 3/8: **Signal node** — pass to Walk
   - Score ≥ 5/8: **Structural truth** — flag explicitly, these are the load-bearing beams

4. **Discard below threshold**: Claims cited by only 1–2 lenses may be domain
   artifacts, not structural truths. Do not pass them to Walk unless they are
   strikingly non-obvious (note them as low-signal outliers separately).

5. **Output format** (internal, for passing to Walk agent):

```
CONVERGENCE NODES — {topic}

STRUCTURAL TRUTHS (≥5/8 lenses):
- [Node A] ({X}/8 lenses) — [one-sentence statement of the structural truth]
- [Node B] ({X}/8 lenses) — [one-sentence statement of the structural truth]

SIGNAL NODES (3–4/8 lenses):
- [Node C] ({X}/8 lenses) — [one-sentence statement]
- [Node D] ({X}/8 lenses) — [one-sentence statement]

LOW-SIGNAL OUTLIERS (1–2/8, notable):
- [Node E] ({X}/8 lenses) — [one-sentence statement] ← candidate irrational leap

LENS RESIDUALS (what no lens could reach):
- [Compiled list of the 8 residuals from lens agents]
```

---

## Phase 2 — Resonance Walk (1 × Sonnet)

Spawn a single Walk agent with `model: "sonnet"`. Pass the full convergence node
output from the Aggregation step, not the raw lens outputs.

**Why scored nodes, not raw lens outputs:** The Walk agent's job is to extract the
frequency and traverse concept space. Raw lens outputs (8 × 350 words) would bury
the Walk in source material and force it to re-do the aggregation. The scored
convergence nodes are the pre-distilled frequency. The Walk can start from the
actual signal instead of mining for it.

### Walk agent prompt

```
You are the Resonance Walk agent in a three-phase synthesis pipeline.
You have received pre-scored convergence nodes — structural truths that were
independently identified by multiple domain experts analyzing the same topic.
Your task: extract the core frequency these nodes share, then traverse concept
space using that frequency as your signal.

TOPIC: {topic}

CONVERGENCE NODES (pre-scored):
{full convergence node output from aggregation step}

INSTRUCTIONS:

STEP 1 — EXTRACT THE FREQUENCY
The convergence nodes reveal a shared underlying tension or paradox that
multiple domains independently circled. Do not describe the topic. Find the
irreducible question it embodies. State it as:
"The [{topic}] is fundamentally about [{tension}] — [{why this tension cannot
be resolved by staying inside any single domain}]."

STEP 2 — FIND 4 RESONANT DOMAINS
List 4 domains that vibrate at this exact frequency — not because they are
similar to the topic, but because the same underlying equation governs them.
At least one must be from art, music, or narrative.
At least one must be from mathematics or physics.
Do not repeat domains already in the lattice.
Format: "[Domain]: [how the frequency appears here. One sentence.]"

STEP 3 — THE IRRATIONAL LEAP
Scan the LOW-SIGNAL OUTLIERS in the convergence nodes. Choose the one that
seems most wrong or absurd as a connection to the topic. If none feel wrong
enough, invent one. Accept it. Force the connection. Describe in 2-3 sentences
what structural claim survives the forcing. If the claim survives, it is real —
the irrational leap is not decoration, it is the entropy injection that escapes
local optima.

STEP 4 — WALK THE PATH
Write the journey as connected narrative prose, moving through the 4 resonant
domains and the irrational leap in sequence. This is not a list.
Each domain's insight must CAUSE the move to the next — the reader should feel
concept space being traversed. 4–6 substantial paragraphs. Do not use headers
inside this section.

STEP 5 — THE LANDING
Where did the walk end? State the new position, tension, or question that did
not exist before the walk. This is what the Shadow agent will work with.
Format: "## Landing\n[2-3 sentences. Precise. This is passed to the next phase.]"

STEP 6 — THE UNRESOLVED THREAD
One thing the walk could not reach. One sentence. This is the handoff.
Format: "**Thread:** [sentence]"

OUTPUT: All 6 steps in order. Steps 1–3 as headed sections. Step 4 as flowing
prose with no internal headers. Steps 5–6 with the exact formatting above.
```

---

## Phase 3 — Shadow Dialectic (1 × Opus)

Spawn a single Shadow agent with `model: "opus"`.
Pass ONLY: (a) original topic, (b) the frequency statement from Step 1 of Walk,
(c) the Landing from Step 5 of Walk. Do not pass the full walk — Opus works on
the distilled output, not the journey.

**Why Opus for Shadow:** The dialectic's hardest operation is finding the shadow —
the thing the thesis *requires to be false* in order to function, not merely its
opposite. Weak shadow detection produces clever reframing, not paradigm shift.
This is the judgment call that benefits most from the strongest model.

### Shadow agent prompt

```
You are the Shadow Dialectic agent — the final phase of the Vagabond synthesis
pipeline. You receive a topic, its core frequency (extracted from cross-domain
convergence), and the landing point of a resonance walk across concept space.
Your task: run two full Hegelian dialectical cycles on the landing to produce
a position that the original topic cannot absorb.

ORIGINAL TOPIC: {topic}

CORE FREQUENCY: {frequency statement from Walk Step 1}

WALK LANDING: {landing from Walk Step 5}

INSTRUCTIONS — TWO DIALECTICAL CYCLES:

CYCLE 1:

Thesis: The Walk Landing is your thesis. Steelman it at its most confident.
What worldview does it require? What does it claim about the structure of reality?
Write this as if fully committed to it. (3–5 sentences)

Shadow (Antithesis): Find what the thesis REQUIRES TO BE FALSE in order to function.
Not its opposite — its structural dependency. Ask:
- What must this position assume that it cannot prove?
- What phenomenon would destroy it if taken seriously?
- Who or what does it render invisible in order to work?
- What cost does it exclude from its own ledger?
Steelman the antithesis as forcefully as the thesis. (3–5 sentences)

Synthesis₁: Neither wins. Find the frame large enough that both thesis and antithesis
are locally true within it — special cases of a more general principle. This is NOT
compromise. It is a move to a higher logical type. The synthesis must:
- Make the thesis look incomplete, not wrong
- Make the antithesis look incomplete, not wrong
- Introduce a claim neither side could reach alone
(4–6 sentences)

CYCLE 2 (more compressed):

Synthesis₁ becomes the new thesis. Find ITS shadow — what it now requires to be
false in order to function. (2–3 sentences for shadow)
Force the second synthesis. This is the position that contains the original topic,
the walk's landing, AND the first synthesis all as special cases. (3–4 sentences)

NEW GROUND:
After two cycles, describe the worldview that is now visible. Not a list of
conclusions — a description of a new position the user must now inhabit. What
can they see from here that they could not see from the original topic?
(1 substantial paragraph, 5–8 sentences)

WHAT THE SHADOW CANNOT SWALLOW:
Every synthesis has its own residual. Name the one thing this new position still
cannot account for. One sentence. This is not a failure — it is the beginning of
the next wandering.

OUTPUT FORMAT:
Dense prose throughout. No bullet lists. Use clear paragraph breaks to separate
the dialectical positions. Cycle 1 and Cycle 2 labeled. New Ground and Residual
as their own sections with those exact headers.

Warning: If the output is comfortable, the shadow was not found. Push harder.
```

---

## Step 5 — Compile & Save

After Opus Shadow completes, compile the full output file:

```markdown
# Vagabond: {topic}
*{date} | 8-lens parallel lattice → resonance walk → shadow dialectic (2 cycles)*

## Convergence Map
{convergence node output from aggregation — structural truths, signal nodes, outliers}

## Frequency
{Walk Step 1 frequency statement}

## The Walk
{Walk Steps 2–4: resonant domains, irrational leap, narrative}

## Landing
{Walk Step 5}

## Shadow Dialectic
{Full Opus output: Cycle 1, Cycle 2, New Ground, Residual}

## Residual Threads
Walk thread: {Walk Step 6}
Shadow residual: {Shadow's final residual}
```

Write to: `.tmp/vagabond_{slug}.md` where `{slug}` is the topic lowercased with
spaces replaced by underscores, truncated to 40 characters.

---

## Report to User

After saving, report:
- Topic
- Number of structural truths found (≥5/8 lenses)
- Number of signal nodes (3–4/8)
- The frequency statement (one line)
- The landing (2–3 sentences)
- The new ground headline (first sentence of Opus output)
- File path

---

## Design Notes

**Why parallel lenses, not sequential:** Sequential lens analysis allows cross-lens
suggestion — lens 4 reads lens 3's output (implicitly, through shared context) and
echoes it, inflating apparent convergence. Independent parallel agents eliminate this
correlation bias. A structural truth that 5 agents independently find without
coordination is a signal about reality, not about the analysis process.

**Why scored nodes feed Walk, not raw lens outputs:** The Walk agent's task is
frequency extraction and traversal, not aggregation. Passing 2,800 words of raw
lens output forces the Walk to re-do the aggregation and biases it toward the most
verbose or vivid lens. The scored nodes are the pre-distilled signal — Walk starts
from the correct input space.

**Why the irrational leap is sourced from low-signal outliers:** Low-signal claims
(cited by 1–2 lenses) are either domain artifacts or genuinely surprising structural
claims that most domains cannot see. Using them as the irrational leap candidates
converts the pipeline's "noise floor" into the Walk's entropy injection — a direct
implementation of the Vagabond Protocol's use of stochastic error as fuel.

**Why Opus for Shadow only:** The lattice and walk are domain analysis and narrative
traversal — tasks where Sonnet's speed-quality ratio is excellent. Shadow dialectic
requires the judgment to distinguish "shallow shadow" (weak antithesis that is just
a weakened thesis) from "deep shadow" (what the thesis structurally depends on being
false). Getting this wrong collapses the output into clever reframing. Opus for the
judgment call; Sonnet for the domain work.

**Total cost:** 8 Sonnet calls (lattice) + 1 Sonnet call (walk) + 1 Opus call
(shadow). Wall-clock time dominated by Phase 1 parallelism. Comparable to
`/stochastic-consensus` at N=5 with Opus synthesis.

**Composing with other vagabond skills:** The individual skills (`skill-vagabond-walk`,
`skill-vagabond-lattice`, `skill-vagabond-shadow`) remain useful as standalone tools.
Use this unified skill for deep synthesis. Use the sub-skills for fast iteration or
when you already have output from a previous phase and want to re-run one stage.

**Status:** deployed
**Deployed to:** `~/.claude/skills/vagabond/SKILL.md`
*Drafted: 2026-04-28*
*Promoted: 2026-04-29*
