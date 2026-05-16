---
name: stochastic-consensus
description: >
  Spawn N agents with the same prompt (slight framing variations) to independently
  analyze a problem, then aggregate results by consensus. Use for decision-making,
  ranking options, strategic analysis, or any problem where you want to filter
  hallucinations and surface high-variance ideas. Triggers on "consensus",
  "poll agents", "stochastic consensus", "spawn N agents to analyze",
  "multi-agent vote", or /stochastic-consensus.
disable-model-invocation: true
argument-hint: "[topic or question] (optional: [N agents, default 5])"
---

# Stochastic Multi-Agent Consensus

Spawn N agents (default 5) with identical context and near-identical prompts. Each independently analyzes and produces a structured response. Aggregate by finding consensus (mode), divergences (splits), and outliers (unique ideas).

**Why this works:** Exploits stochastic variation in LLM outputs. Like polling 5 experts instead of asking one. The mode filters out hallucinations and individual biases. Divergences reveal genuine judgment calls. Outliers surface creative ideas a single run would miss.

## Arguments

- `$0` — The topic, question, or problem to analyze (required)
- `$1` — Number of agents. Default: `5`. Use `3` for quick, `10` for high-stakes decisions.

Output saves to `.tmp/research_debate_<slugified_topic>.md`.

## Architecture

```
                        [Topic / Question]
                               |
         ┌─────┬─────┬────────┼────────┬─────┬─────┐
         v     v     v        v        v     v     v
       [A1]  [A2]  [A3]     [A4]     [A5]  ...  [AN]   ← Sonnet (parallel, independent)
         |     |     |        |        |     |     |
         └─────┴─────┴────────┼────────┴─────┴─────┘
                               |
                         [Synthesizer]                   ← Opus (finds consensus, splits, outliers)
                               |
                          [Output.md]
```

## Step 1: Spawn N Research Agents (Parallel)

Spawn ALL N agents in a **single message** using multiple Agent tool calls. Every agent uses `model: "sonnet"` and `run_in_background: true`.

Each agent gets the **same core prompt** but with a slight framing variation to induce stochastic diversity. The variations are NOT different roles or perspectives — they are minor phrasing differences that nudge the model's sampling distribution.

### Framing variations (cycle through these):

1. "Analyze this thoroughly and commit to clear positions."
2. "Think step by step. What does the evidence actually show?"
3. "Be contrarian. What's the non-obvious take here?"
4. "Prioritize the most recent and concrete data you can find."
5. "Consider second-order effects and downstream implications."
6. "Focus on what practitioners and insiders would say, not textbook answers."
7. "What would a skeptic say? What would they find convincing?"
8. "Think about this from first principles. Ignore conventional wisdom."
9. "What's the highest-leverage insight someone could act on?"
10. "Look for the thing everyone else is missing."

### Agent prompt template:

```
You are one of several independent research agents analyzing the same topic. Your analysis will be compared with others to find consensus, disagreements, and unique insights.

TOPIC: {topic}

FRAMING: {framing_variation}

INSTRUCTIONS:
1. Use WebSearch and WebFetch to research this topic thoroughly.
2. Commit to clear positions. Do NOT hedge everything — take stances.
3. Support claims with specific evidence: data, numbers, sources, dates.
4. Prioritize recent information (2024-2026).
5. Structure your output EXACTLY as:

## Analysis

### Key Claims
For each major claim, state it clearly and tag your confidence:
- [HIGH] Claim with strong evidence — [source]
- [MEDIUM] Claim with partial evidence — [source]
- [LOW] Informed speculation — [reasoning]

### Top 3 Insights
The three most important things someone needs to know. Rank them.
1. ...
2. ...
3. ...

### Surprising or Non-Obvious Findings
Anything you found that goes against conventional wisdom or that most people would miss.

### Open Questions
What you couldn't resolve or what needs deeper investigation.

### Sources
- [Source](URL) — what it contributed

Aim for 400-700 words. Be direct and specific.
```

**All N agents MUST launch in the SAME message block.** This is mandatory for parallelism.

## Step 2: Collect & Aggregate

Once all agents complete, collect all N outputs. Do NOT synthesize yet — first build a raw aggregation table.

For each major claim or finding across all agents, count:
- **Consensus (mode):** How many agents made the same or very similar claim? Claims supported by >60% of agents are high-consensus.
- **Splits:** Claims where agents roughly divide (e.g., 3 for, 2 against). These are genuinely contested points.
- **Outliers:** Claims made by only 1 agent. These are either hallucinations or creative insights — the synthesizer must judge which.

Build a mental (or literal) table like:

| Claim | Agents Supporting | Agents Contradicting | Classification |
|---|---|---|---|
| "Market is $X billion" | A1, A2, A4, A5 | A3 (says $Y) | Consensus (with one dissent) |
| "Regulation X will pass" | A1, A3 | A2, A4, A5 | Split |
| "Company Z is the dark horse" | A2 only | — | Outlier |

## Step 3: Synthesize with Opus

Spawn a SINGLE synthesizer agent with `model: "opus"`.

### Synthesizer prompt:

```
You are the synthesizer of a stochastic multi-agent research poll. {N} independent agents analyzed the same topic. Each had slight framing variations but the same core question. Your job is to find what's real (consensus), what's contested (splits), and what's interesting (outliers).

TOPIC: {topic}
NUMBER OF AGENTS: {N}

ALL AGENT OUTPUTS:
---
{Agent 1 output}
---
{Agent 2 output}
---
... (all N outputs)
---

YOUR TASK:
1. For each major claim, count how many agents independently arrived at the same conclusion.
2. Claims supported by >60% of agents → **Consensus** (high confidence, likely real).
3. Claims where agents roughly split → **Contested** (genuine uncertainty, present both sides).
4. Claims made by only 1 agent → **Outlier** (judge: hallucination or genuine insight?).
5. Look at the "Top 3 Insights" across all agents — which insights appeared most often? Which appeared only once but are compelling?

OUTPUT FORMAT:

# Research Brief: {topic}
*Generated: {date} | Method: Stochastic consensus, {N} agents*

## Executive Summary
3-5 sentences. Lead with the highest-consensus findings.

## Consensus Findings (>{60}% agent agreement)
These claims were independently reached by most agents — highest confidence.
- [Finding] — supported by {X}/{N} agents. [Key evidence]

## Contested Points (agents split)
Genuine disagreements that reveal real uncertainty.
- [Point] — {X} agents say A, {Y} agents say B. Evidence on each side: ...

## Outlier Insights (single-agent, worth flagging)
Claims only one agent made. For each, judge: hallucination or hidden gem?
- [Claim] — from Agent {X}. Assessment: [likely real because... / likely noise because...]

## Key Data & Numbers
Table of quantitative findings. Note how many agents cited each number.

| Metric | Value | Agents Citing | Confidence |
|---|---|---|---|
| ... | ... | X/N | High/Medium/Low |

## Open Questions (recurring across agents)
Questions that multiple agents flagged as unresolved.

## Sources
Deduplicated, annotated. Note how many agents independently cited each source.

Be rigorous. The whole point of this method is that consensus = signal and outliers need scrutiny. Don't flatten everything into a smooth narrative — preserve the variance.
```

## Step 4: Save & Report

1. Write the Opus synthesizer output to `.tmp/research_debate_<topic_slug>.md`
2. Report to user:
   - Topic
   - Number of agents used
   - Number of consensus findings vs contested vs outliers
   - Most interesting outlier (if any)
   - Output file path

## Design Notes

- **Why stochastic consensus:** LLM outputs vary run-to-run. Instead of fighting this variance, we exploit it. Agreement across independent runs is a strong signal. Disagreement reveals where the model is uncertain or where the question is genuinely hard.
- **Why slight framing variations (not identical prompts):** Identical prompts would produce highly correlated outputs, reducing the value of multiple runs. Minor framing nudges explore different parts of the output distribution without changing the core task.
- **Why NOT adversarial roles:** Unlike `/research` which assigns distinct angles, this method deliberately keeps agents on the same task. The diversity comes from stochastic sampling, not assigned perspectives. This avoids the sycophancy problem where role-assigned agents perform their role rather than seek truth.
- **Why Opus synthesizer:** The aggregation task (mode/split/outlier classification) requires judgment about which outliers are noise vs insight. This is the hardest part and benefits from the strongest model.
- **Cost:** N Sonnet calls + 1 Opus call. Default 5 agents ≈ 2.5x a single Sonnet deep research call, plus Opus synthesis. Roughly comparable to `/research` deep mode.
- **When to use more agents:** High-stakes decisions, topics where hallucination is costly, ranking/comparison tasks. 10 agents gives much stronger consensus signals.
- **When NOT to use:** Simple factual lookups, tasks where speed matters more than accuracy, topics so niche that all agents will return the same sparse results.
- **Known limitation (ICLR 2025):** Multi-agent methods underperform self-consistency baselines on simple benchmarks. This method shines on complex, multi-faceted questions where there are genuinely multiple defensible positions.

---

**Status:** deployed
**Deployed to:** `~/.claude/skills/stochastic-consensus/SKILL.md`
*Drafted: 2026-04-28*
*Promoted: 2026-04-29*
