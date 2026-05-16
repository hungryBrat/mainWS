---
name: model-chat
description: >
  Spawn 5+ Claude instances into a shared conversation room where they debate,
  disagree, and converge on solutions. Uses round-robin turns with parallel
  execution within each round. A synthesizer agent then merges the debate into
  a final answer. Triggers on "model chat", "multi-model debate",
  "agent debate", "spawn a chat room", or /model-chat. Pass a topic as the argument.
disable-model-invocation: false
argument-hint: "[topic or question] (optional: [agents, default 5] [rounds, default 5])"
---

# Model Chat

Spawn 5 Claude Sonnet instances into a shared conversation room. They debate a problem across 5 rounds using round-robin turns (all agents respond in parallel each round, see full history). A synthesizer agent then merges the debate into a final answer.

**Why this works:** Same model, slight framing variations = systematically different failure modes. Surfacing disagreements between instances is more valuable than any single instance's confident answer. Consensus across independent runs filters hallucinations; divergences reveal genuine judgment calls.

## Arguments

- `$0` — The topic, question, or problem to debate (required)
- `$1` — Number of agents. Default: `5`.
- `$2` — Number of rounds. Default: `5`.

Output saves to `.tmp/model_chat_<slugified_topic>.md`.

## Architecture

```
                         [Topic]
                            |
              ┌────┬────┬───┼───┬────┐
              v    v    v   v   v    v
            [S1] [S2] [S3] [S4] [S5]        ← Round 1: Independent first takes (parallel)
              |    |    |   |   |
              └────┴────┴───┼───┘
                   [shared transcript]
              ┌────┬────┬───┼───┬────┐
              v    v    v   v   v    v
            [S1] [S2] [S3] [S4] [S5]        ← Round 2: Respond to full history (parallel)
              |    |    |   |   |
                      ... x5 rounds ...
              └────┴────┴───┼───┘
                            v
                      [Synthesizer]          ← Opus: merges debate into final answer
                            |
                       [Output.md]
```

## Agent Identities

Each agent gets a short identity tag — NOT a deep persona, just enough to differentiate their voice in the transcript. Cycle through:

| Agent | Tag | Nudge |
|---|---|---|
| S1 | **Evidence-first** | "Ground every claim in data or sources." |
| S2 | **Devil's advocate** | "Challenge the emerging consensus. Find the weak points." |
| S3 | **Practitioner** | "Focus on what actually works in practice, not theory." |
| S4 | **Systems thinker** | "Look at second-order effects, feedback loops, and unintended consequences." |
| S5 | **Synthesizer-in-training** | "Try to find the common ground. Where do the others actually agree without realizing it?" |

For >5 agents, add: "First-principles reasoner", "Historian" (what does precedent say), "Contrarian optimist", "Risk analyst", "End-user perspective".

## Step 1: Round 1 — Independent First Takes (Parallel)

Spawn ALL agents in a **single message** using multiple Agent tool calls. Every agent uses `model: "sonnet"` and `run_in_background: true`.

### Round 1 prompt (each agent):

```
You are agent {AGENT_TAG} in a multi-agent debate room. {N} agents are debating the same topic. This is Round 1 — you are giving your independent first take. You have NOT seen anyone else's response yet.

TOPIC: {topic}

YOUR IDENTITY: {agent_tag} — {nudge}

INSTRUCTIONS:
1. If useful, use WebSearch/WebFetch to research the topic.
2. Give your honest, committed take. Do NOT hedge or give a "balanced view" — take a clear position.
3. Be concise and direct. 200-400 words.
4. Structure as:

### {AGENT_TAG} — Round 1

**Position:** [1-2 sentence clear stance]

**Reasoning:**
[Your argument with evidence]

**Key claim:** [The single most important thing you want the room to hear]
```

**All agents launch in ONE message block.**

## Step 2: Rounds 2-N — Debate with Full History (Parallel)

After each round completes, compile the **full transcript so far** and send it to every agent via `SendMessage`. All agents in each round launch in a **single message** for parallelism.

### Round 2+ prompt (via SendMessage to each agent):

```
ROUND {R} OF {TOTAL_ROUNDS}

FULL DEBATE TRANSCRIPT SO FAR:
---
{Complete transcript of all previous rounds, all agents}
---

YOUR TASK:
1. Read what ALL other agents said in previous rounds.
2. You MUST engage with specific points others made — quote or reference them directly.
3. You can:
   a. CHALLENGE a specific claim ("S2 said X, but this is wrong because...")
   b. BUILD ON someone's point ("S3's insight about Y connects to...")
   c. CONCEDE a point ("S1 convinced me that Z, updating my position")
   d. INTRODUCE new evidence not yet discussed
4. Do NOT repeat yourself. Do NOT just restate your Round 1 position.
5. If you're changing your mind, say so explicitly and say WHY.

Rules:
- You MUST reference at least 2 other agents by name.
- New evidence from WebSearch is encouraged but not required.
- 150-300 words. Be sharp.

### {AGENT_TAG} — Round {R}

**Responding to:** [which agents/points]

**Updated position:** [same / shifted / reversed]

[Your response]
```

**Repeat for all rounds.** The transcript grows each round — every agent sees everything.

### Convergence early-exit (optional):

If after any round, ALL agents express the same position with no new arguments, you may skip remaining rounds. Note this in the report.

## Step 3: Synthesize with Opus

Once all rounds complete, spawn a SINGLE synthesizer agent with `model: "opus"`.

### Synthesizer prompt:

```
You are the synthesizer of a multi-agent debate. {N} Claude Sonnet instances debated a topic over {ROUNDS} rounds in a shared conversation room. Each agent had a different critical lens. Your job is to produce the definitive answer by analyzing the full debate.

TOPIC: {topic}

FULL DEBATE TRANSCRIPT (ALL ROUNDS):
{Complete transcript}

YOUR TASK:

1. **Track position evolution:** For each agent, note how their position changed across rounds. Who changed their mind? Who held firm? What evidence caused shifts?

2. **Find the consensus:** What did agents converge on? Weight these findings heavily — if 5 independent instances with different framings all arrive at the same conclusion, it's very likely correct.

3. **Preserve productive disagreements:** Where agents DIDN'T converge, present both sides fairly. These are genuinely hard calls.

4. **Evaluate argument quality:** Not all agents contributed equally. Note which agents made the strongest arguments (with evidence) vs. which capitulated or repeated themselves.

5. **Identify emergent insights:** Did the debate produce any insight that NO single agent had in Round 1? These multi-round emergent ideas are the unique value of this method.

OUTPUT FORMAT:

# Model Chat Brief: {topic}
*Generated: {date} | Method: {N}-agent debate, {ROUNDS} rounds + synthesis*

## Final Answer
The best answer to the question/topic, informed by the full debate. Be direct — this is the "if you read one paragraph" section. 3-5 sentences.

## How We Got Here
Brief narrative of how the debate evolved. Which rounds had the biggest shifts? What evidence was most persuasive? 1-2 paragraphs.

## Consensus Points
Claims that agents converged on. Weighted by how many agents and how strong the evidence.
- [Claim] — {X}/{N} agents converged. Key evidence: ...

## Open Disagreements
Where agents remained split even after {ROUNDS} rounds. Present both sides.
- [Point] — Position A (agents: ...) vs Position B (agents: ...). Why it didn't resolve: ...

## Emergent Insights
Ideas that only appeared through the debate process — not present in any agent's Round 1.
- [Insight] — first surfaced in Round {X} by {agent}, built on by {agents}

## Debate Quality Assessment
- Strongest contributor: {agent} — why
- Biggest position shift: {agent} — from X to Y, because...
- Most productive exchange: Round {X} between {agents} — about what

## Sources
Deduplicated list from all agents across all rounds.

Be rigorous. The value of debate over polling is the INTERACTION — highlight where agents built on or changed each other's thinking.
```

## Step 4: Save & Report

1. Write the Opus synthesizer output to `.tmp/model_chat_<topic_slug>.md`
2. Report to user:
   - Topic debated
   - Number of agents x rounds
   - Whether early consensus was reached (and at which round)
   - The final answer (executive summary)
   - Most interesting disagreement or emergent insight
   - Output file path

## Design Notes

- **Why round-robin with full history:** Unlike stochastic consensus (independent agents), the whole point of model-chat is that agents REACT to each other. Full transcript visibility creates genuine dialogue, not parallel monologues.
- **Why 5 rounds:** Enough for positions to evolve meaningfully. Du et al. (2023) showed gains up to 2-3 rounds on factual tasks; for open-ended debate, 5 rounds allows deeper exploration. Diminishing returns beyond 5.
- **Why 5 agents:** Sweet spot between diversity and coordination noise. 3 is too few for a real debate (tends to 2v1 deadlock). 5 gives enough voices for genuine convergence signals.
- **Anti-sycophancy:** The identity tags create structural incentive to disagree (especially "Devil's advocate"). The "MUST reference 2 other agents" rule forces engagement rather than parallel monologuing. The "Do NOT repeat yourself" rule prevents agents from just restating safe positions.
- **Cost:** 5 agents x 5 rounds = 25 Sonnet calls + 1 Opus synthesis. More expensive than `/stochastic-consensus` but produces richer, more nuanced output because of the interaction.
- **When to use vs other skills:**
  - `/research` — broad landscape mapping, factual surveys (fan-out, no interaction)
  - `/stochastic-consensus` — decision-making, filtering hallucinations (independent polling)
  - `/model-chat` — nuanced topics, contested questions, strategy, anything where the DEBATE itself produces insight
- **Known limitation:** Long transcripts in later rounds consume significant context. For very complex topics, consider reducing to 3 agents x 3 rounds to keep each agent's context manageable.

---

**Status:** deployed
**Deployed to:** `~/.claude/skills/model-chat/SKILL.md`
*Drafted: 2026-04-28*
*Promoted: 2026-04-29*
