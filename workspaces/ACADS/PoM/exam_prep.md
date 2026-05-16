# MGTS F211 — Exam Prep: 50 High-Probability Solved Questions

**Exam: 10 May 2026 · 3 hours · 35% weight · Close-book**
**Marking pattern (PYQ-confirmed): NAME the framework → LIST components verbatim → APPLY to the scenario.**

Questions are weighted by PYQ frequency. Priority blocks (computation-heavy, repeat across both PYQ papers): payoff-matrix decisions, financial ratios + fund flow, value chain, marketing mix, leadership theory application, JCM, BSC.

---

## Q1. [Decision Under Uncertainty — PYQ repeat] [6M]

A FMCG firm, **NoorChem Ltd.**, is launching a new detergent in 4 alternative price-pack formats (S1–S4) against 3 unknown competitor reactions (CA1–CA3). The marketing head wants you to recommend a strategy under each of the four psychological orientations to decision-making under uncertainty. Show every computation.

| Strategy | CA1 | CA2 | CA3 |
|---|---|---|---|
| S1 | 22 | 18 | 14 |
| S2 | 10 | 25 | 19 |
| S3 | 28 | 20 | 12 |
| S4 | 16 | 22 | 30 |

**Model Answer**

Under **uncertainty** (probabilities of competitor actions are unknown), four formal methods apply, each tied to a psychological orientation:

**1. MAXIMAX (Optimistic — "best of bests")**
Row maxima: S1=22, S2=25, S3=28, S4=30 → **Choose S4 (payoff = 30)**

**2. MAXIMIN / Wald (Pessimistic — "best of worsts")**
Row minima: S1=14, S2=10, S3=12, S4=16 → **Choose S4 (payoff = 16)**

**3. LAPLACE / Equal-likelihood (Neutral)**
Average each row (sum ÷ 3):
- S1 = (22+18+14)/3 = 18.00
- S2 = (10+25+19)/3 = 18.00
- S3 = (28+20+12)/3 = 20.00
- S4 = (16+22+30)/3 = **22.67**
**Choose S4**

**4. MINIMAX REGRET / Savage (Regret-minimizing)**
Column maxima: CA1=28, CA2=25, CA3=30. Build regret matrix (col-max − cell):

| | CA1 | CA2 | CA3 | Max regret |
|---|---|---|---|---|
| S1 | 6 | 7 | 16 | 16 |
| S2 | 18 | 0 | 11 | 18 |
| S3 | 0 | 5 | 18 | 18 |
| S4 | 12 | 3 | 0 | **12** |

Smallest of {16, 18, 18, 12} = 12 → **Choose S4**

**Recommendation:** All four orientations converge on **S4** — the dominant strategy.

---

## Q2. [Decision Under Uncertainty — PYQ repeat] [6M]

A defense supplier is evaluating 3 R&D project bids (P1–P3) against 4 plausible Ministry budget scenarios (B1–B4). Apply the four uncertainty methods. Indicate the strategy each orientation would select.

| Project | B1 | B2 | B3 | B4 |
|---|---|---|---|---|
| P1 | 50 | 70 | 60 | 40 |
| P2 | 30 | 90 | 80 | 20 |
| P3 | 60 | 65 | 55 | 70 |

**Model Answer**

**1. MAXIMAX:** Row maxima 70, 90, 70 → **P2 (90)** — Optimistic
**2. MAXIMIN:** Row minima 40, 20, 55 → **P3 (55)** — Pessimistic
**3. LAPLACE:** Averages = 220/4=55.0, 220/4=55.0, 250/4=**62.5** → **P3** — Neutral
**4. MINIMAX REGRET:** Col maxima 60, 90, 80, 70.

| | B1 | B2 | B3 | B4 | Max regret |
|---|---|---|---|---|---|
| P1 | 10 | 20 | 20 | 30 | 30 |
| P2 | 30 | 0 | 0 | 50 | 50 |
| P3 | 0 | 25 | 25 | 0 | **25** |

→ **P3 (25)** — Regret-minimizing

P3 wins under three of the four orientations; only an optimistic decision-maker would pick P2.

---

## Q3. [Mintzberg's Roles — Likely identification Q] [4M]

The CEO of **TataNova Ltd.** had this calendar today: (a) cut the ribbon at a new R&D centre, (b) read industry research on EV demand, (c) fired an underperforming VP, (d) approved the annual capital expenditure budget, (e) negotiated a long-term supply contract with a Japanese partner. Identify the Mintzberg role for each action and the role-category it belongs to.

**Model Answer**

Mintzberg classified managerial work into **3 categories** containing **10 roles**: Interpersonal (Figurehead, Leader, Liaison), Informational (Monitor, Disseminator, Spokesperson), Decisional (Entrepreneur, Disturbance handler, Resource allocator, Negotiator).

| Action | Role | Category |
|---|---|---|
| (a) Ribbon cutting | **Figurehead** | Interpersonal |
| (b) Reading industry research | **Monitor** | Informational |
| (c) Firing underperforming VP | **Disturbance handler** | Decisional |
| (d) Approving capex budget | **Resource allocator** | Decisional |
| (e) Supply contract negotiation | **Negotiator** | Decisional |

---

## Q4. [Katz's Skills — Likely short Q] [3M]

A first-line production supervisor at a textile mill has been promoted to plant manager. Using **Katz's three essential skills**, advise which skill she must now develop most and why.

**Model Answer**

Katz identified **3 essential managerial skills**:
1. **Technical** — job-specific knowledge and techniques. *Most critical for first-line managers.*
2. **Human (interpersonal)** — working effectively with individuals and groups. *Equally critical at all levels.*
3. **Conceptual** — abstract thinking, seeing the organization as a whole, integrating ideas. *Most critical for top managers.*

**Recommendation:** As she moves from first-line to middle management, **Conceptual skill** must be developed most — she must now see the plant as a system interacting with supply, finance, HR, and corporate strategy, rather than focus on machine-level efficiency. Human skill remains equally important; technical skill recedes in priority.

---

## Q5. [Taylor + Fayol — History likely 3M ID question] [3M]

Identify whether each statement reflects **Taylor's Scientific Management** or **Fayol's General Administrative Theory**, and name the specific principle.

(i) "Each worker is scientifically selected, trained, and developed for the specific task assigned."
(ii) "An employee should receive orders from one superior only."
(iii) "There must be equity — kindliness and justice — in management's dealing with employees."

**Model Answer**

**Taylor's 4 principles** (Scientific Management): (1) Develop a *science* for each element, (2) Scientifically *select* and train workers, (3) *Cooperate* with workers, (4) *Divide* work and responsibility between management and workers.

**Fayol's 14 principles** (Administrative Theory): Division of work, Authority, Discipline, Unity of command, Unity of direction, Subordination of individual interest, Remuneration, Centralization, Scalar chain, Order, Equity, Stability of tenure, Initiative, Esprit de corps.

(i) → **Taylor** — Principle 2: Scientific selection and training of workers.
(ii) → **Fayol** — Principle of **Unity of command**.
(iii) → **Fayol** — Principle of **Equity**.

---

## Q6. [Org Culture 7 Dimensions — PYQ-style application] [4M]

**Apple Inc.** is described as obsessive about precision in product design, willing to bet on radical product launches, places aggressive performance demands on engineers, and integrates work around small product teams rather than departments. Using the **7 dimensions of organizational culture** (Robbins), identify which dimensions Apple scores HIGH on, with one-line evidence each.

**Model Answer**

Robbins' **7 dimensions of organizational culture**:
1. Attention to detail
2. Outcome orientation
3. People orientation
4. Team orientation
5. Aggressiveness
6. Stability
7. Innovation and risk taking

**Apple's high-scoring dimensions:**
- **Attention to detail** — obsession with product design precision (typography, materials, micro-interactions).
- **Innovation and risk taking** — radical product bets (iPhone, Vision Pro) without prior market evidence.
- **Aggressiveness** — competitive culture; aggressive performance demands and direct internal critique.
- **Team orientation** — work integrated around small cross-functional product teams (Mac, iPhone, Services).

Apple scores LOW on **Stability** (constant disruption of own products) and is *moderate* on **Outcome vs. Process orientation** (both ship-quality and craft are emphasised).

---

## Q7. [CSR + Stakeholders — PYQ-confirmed Apple-style Q] [4M]

For **Apple Inc.**, list the **primary stakeholders** in two columns: (a) economic/commercial stakeholders, (b) social/community stakeholders. Then briefly state **one CSR concern** Apple owes to each.

**Model Answer**

A **stakeholder** is any group or individual affected by, or able to affect, the organization. Two views frame CSR: **Classical (Friedman)** — sole obligation is shareholder profit; **Socioeconomic** — obligation extends to society's welfare. Apple's stakeholder map (socioeconomic view):

| (a) Economic/Commercial | CSR concern |
|---|---|
| Shareholders | Sustained returns; transparent disclosures |
| Employees | Fair pay, safe workplace, growth |
| Customers | Product safety, data privacy, fair pricing |
| Suppliers (e.g. Foxconn, TSMC) | Fair contracts, prompt payment |
| Distributors / retailers | Margin sustainability, channel stability |

| (b) Social/Community | CSR concern |
|---|---|
| Local communities (factory regions) | Environmental impact, employment |
| Government / regulators | Tax compliance, antitrust |
| Environment | Carbon-neutral supply chain (Apple 2030 pledge) |
| NGOs / labour bodies | Conflict-mineral sourcing, supplier audits |
| Media | Transparent disclosure |

---

## Q8. [Hofstede's 5 Dimensions — Likely global-business Q] [4M]

An Indian software firm plans to expand operations into **Japan**. Using **Hofstede's 5 cultural dimensions**, predict three management challenges and recommend an adaptation for each.

**Model Answer**

Hofstede's **5 cultural dimensions**:
1. **Power distance** — acceptance of unequal power distribution.
2. **Individualism vs. collectivism** — loose vs. tight social fabric.
3. **Achievement vs. nurturing** (Masculinity vs. Femininity) — assertive vs. cooperative values.
4. **Uncertainty avoidance** — discomfort with ambiguity.
5. **Long-term vs. short-term orientation** — future vs. tradition.

**Japan's profile:** high power distance, high collectivism, very high uncertainty avoidance, high masculinity, very high long-term orientation.

| Challenge | Adaptation |
|---|---|
| **High uncertainty avoidance** — Japanese employees expect detailed SOPs, formal rules. | Introduce structured documentation, predictable career paths, written job specs. |
| **High collectivism** — group consensus (*nemawashi*) precedes decisions. | Avoid the Indian-style direct individual confrontation; build consensus before formal meetings. |
| **High long-term orientation** — relationships and patience matter. | Invest in long-cycle client trust building; avoid quarterly-only KPIs. |

---

## Q9. [Conflict Types — PYQ repeat] [3M]

List and briefly explain the **three types of conflict** that may occur in a work team. State which is functional and which is dysfunctional.

**Model Answer**

| Type | Nature | Effect |
|---|---|---|
| **Task conflict** | Disagreement over content, goals, or work decisions | **Functional** at low-to-moderate levels — improves decision quality |
| **Relationship conflict** | Interpersonal incompatibility, personality clashes | **Always dysfunctional** — reduces trust, derails teamwork |
| **Process conflict** | How work gets done — task allocation, methods, scheduling | **Functional** at low levels only — high process conflict erodes execution |

Managers should encourage moderate task conflict for richer decisions, monitor process conflict, and resolve relationship conflict immediately (e.g., via Thomas-Kilmann *Collaborating* or *Accommodating* styles).

---

## Q10. [Tuckman + Effective Teams — Likely Q] [4M]

A new cross-functional product team has just been formed at a fintech startup. Members are still uncertain of roles, leadership is contested, and they argue over who decides what. (a) Identify the current Tuckman stage. (b) List the **8 characteristics of effective teams** the leader should drive them toward.

**Model Answer**

**(a)** Tuckman's **5 stages of group development**: **Forming → Storming → Norming → Performing → Adjourning**.
The team shows role uncertainty + contested leadership + intragroup conflict → **Storming** stage. The leader's job is to surface conflict constructively and converge on norms.

**(b) 8 Characteristics of Effective Teams (verbatim):**
1. Clear goals
2. Relevant skills (technical + interpersonal)
3. Mutual trust
4. Unified commitment
5. Good communication
6. Negotiating skills
7. Appropriate leadership
8. Internal and external support

The leader should establish **clear goals** and **good communication** first — both directly accelerate movement from Storming to Norming.

---

## Q11. [Communication Channel Choice — PYQ repeat (Q14 paper 1)] [4M]

You are CEO. You must inform employees that two product lines are being shut down and 500 jobs lost. Identify **two communication methods** you will use and the **criteria** that justify each.

**Model Answer**

Methods of interpersonal communication are evaluated on criteria including: **feedback speed, complexity capacity, breadth potential, confidentiality, encoding ease, decoding ease, time-space constraint, cost, interpersonal warmth, formality, scanability**. Channel richness ranks: Face-to-face > Video > Phone > Email > Memo > Impersonal report.

**Method 1: Face-to-face town hall (richest channel)**
Justifying criteria: high **feedback speed** (immediate Q&A), high **interpersonal warmth** (manages emotional reaction to job loss), high **complexity capacity** (allows nuance on rationale and severance terms), nonverbal cues visible.

**Method 2: Formal written letter / official memo (lean, formal channel)**
Justifying criteria: **confidentiality** (signed individual notice), **scanability** (employee can re-read terms), **formality** required for legal/HR record, documented reference for severance, separation date, statutory benefits.

Together: town hall delivers the *message and meaning*; the formal letter delivers the *legal record*.

---

## Q12. [Communication Barriers — Likely Q] [3M]

A new manager reports: "When I send the weekly plan over email, my factory floor team either ignores it, misreads the priorities, or comes back with totally unrelated questions." Identify three likely **barriers to effective communication** and recommend a fix for each.

**Model Answer**

**6 barriers to effective communication:** Filtering, Emotions, Information overload, Defensiveness, Language, National culture.

| Barrier | Symptom in scenario | Fix |
|---|---|---|
| **Information overload** | Team ignores email | Reduce to a one-page priority list; bullet top 3 items |
| **Language** | Misread priorities | Avoid jargon; use plain factory-floor terms; translate to vernacular if needed |
| **Filtering / poor channel choice** | Unrelated follow-ups suggest the lean channel isn't carrying intent | Switch the weekly plan to a 10-min standup (face-to-face — richer channel) |

---

## Q13. [Big Five (OCEAN) — Personality application] [3M]

Tom is anxious, takes feedback poorly, dislikes uncertainty, and is reserved in meetings. He is being considered for a **regional sales head** role that requires aggressive client engagement and tolerance of revenue volatility. Use the **Big Five** to advise.

**Model Answer**

The **Big Five (OCEAN)** dimensions:
1. **O**penness to experience
2. **C**onscientiousness
3. **E**xtraversion
4. **A**greeableness
5. **N**euroticism

Tom's profile: **High Neuroticism** (anxious, defensive), **Low Extraversion** (reserved), likely **Low Openness** (dislikes uncertainty).

A regional sales head requires **High Extraversion** (client-facing assertiveness), **Low Neuroticism** (composure under volatility), and **High Conscientiousness** (territory discipline). Tom's profile is a poor fit.

**Recommendation:** Either (a) place him in an analytical/back-office role suited to his profile, or (b) sponsor coaching on **emotional self-regulation** (Goleman EI component) and a graduated client-exposure plan before the move.

---

## Q14. [Emotional Intelligence — Likely Q] [2M]

List Goleman's **5 components of Emotional Intelligence** and identify which one is most lacking in a manager who repeatedly snaps at his team during deadline weeks.

**Model Answer**

Goleman's **5 EI components**:
1. **Self-awareness** — recognizing own emotions and their impact.
2. **Self-management / self-regulation** — controlling disruptive impulses; pausing before acting.
3. **Self-motivation** — drive beyond money or status.
4. **Empathy** — understanding others' emotions.
5. **Social skills** — managing relationships, building rapport.

Snapping under deadline pressure shows poor **Self-management/self-regulation** — and likely weak **Self-awareness** (he isn't catching the trigger).

---

## Q15. [Maslow vs. Herzberg — Motivation contrast] [4M]

A factory worker has just received a 30% pay hike and a more spacious cafeteria, but reports being "still bored and uninspired." Use **Maslow** and **Herzberg** to explain why money + cafeteria didn't motivate.

**Model Answer**

**Maslow's hierarchy (lower → higher):** Physiological → Safety → Social → Esteem → Self-actualization. Once a need level is satisfied it ceases to motivate; the next level becomes the active motivator.

**Herzberg's two-factor theory:**
- **Hygiene factors** (cause dissatisfaction if absent, but don't motivate when present): Supervision, Company policy, Relationship with supervisor, Working conditions, **Salary**, Relationship with peers, Personal life, Status, Security.
- **Motivators** (cause satisfaction): Achievement, Recognition, Work itself, Responsibility, Advancement, Growth.

**Application:**
- The **pay hike** addresses **Salary** — a *hygiene* factor (Herzberg) and lower-level *Physiological/Safety* need (Maslow). It removes dissatisfaction but does **not** create motivation.
- The **cafeteria** improves **Working conditions** — also hygiene.
- The worker is "bored" → he needs activation of higher-level motivators: **Recognition, Work itself, Responsibility, Achievement** (Herzberg) and **Esteem / Self-actualization** (Maslow). Suggest job enrichment, public recognition, and more challenging work.

---

## Q16. [JCM + MPS — PYQ repeat (Q13 paper 1)] [4M]

The Job Characteristics Model (JCM) provides a framework for designing motivating jobs. List the **5 core job dimensions**, state the formula for **Motivating Potential Score (MPS)**, and compute MPS for a job rated: SV=6, TI=7, TS=8, A=4, F=3 (each on 1–10 scale).

**Model Answer**

**5 Core Job Dimensions** (Hackman & Oldham):
1. **Skill variety (SV)** — uses different skills and talents.
2. **Task identity (TI)** — completing a whole, identifiable piece of work.
3. **Task significance (TS)** — substantial impact on others' lives or work.
4. **Autonomy (A)** — freedom in scheduling and procedure.
5. **Feedback (F)** — direct, clear information about performance.

Dimensions 1–3 → *experienced meaningfulness*; Autonomy → *experienced responsibility*; Feedback → *knowledge of results*.

**MPS Formula:**
> **MPS = [(SV + TI + TS) / 3] × Autonomy × Feedback**

**Computation:**
MPS = [(6 + 7 + 8)/3] × 4 × 3 = (21/3) × 12 = 7 × 12 = **84**

(Maximum possible MPS on 1–10 scale = 1000; this job is moderately motivating but autonomy and feedback are the weak links — redesign should target those.)

---

## Q17. [Expectancy Theory — Diagnostic Q] [4M]

Vidya, a software engineer, recently said: "I work hard, but my manager doesn't notice; even if she did, the year-end bonus is just ₹5,000 anyway." Apply **Vroom's Expectancy Theory** to diagnose her motivation, and recommend the managerial fix.

**Model Answer**

**Vroom's Expectancy Theory:**
> **Motivation = Expectancy × Instrumentality × Valence**

- **Expectancy (E→P):** belief that effort leads to performance.
- **Instrumentality (P→O):** belief that performance leads to reward.
- **Valence:** value the individual places on the reward.

If **any one of the three is zero, motivation = zero** (multiplicative).

**Diagnosis:**
- *"I work hard"* → Expectancy is intact.
- *"manager doesn't notice"* → **Instrumentality is broken** (no perceived link from performance to reward).
- *"bonus is just ₹5,000"* → **Valence is low** (reward is not valued).

**Managerial fix:**
1. Fix **Instrumentality** — install a transparent performance-tracking system and visible link to ratings.
2. Fix **Valence** — survey what Vidya actually values (career growth, learning budget, sabbatical) and align rewards.

---

## Q18. [Equity Theory — Likely Q] [3M]

Two analysts at the same level were given identical bonuses (₹50,000 each) despite Analyst A delivering 30% higher project margins than Analyst B. Both feel demotivated. Explain using **Adams' Equity Theory** and distinguish **equality vs. equity**.

**Model Answer**

Adams' **Equity Theory**: employees compare their **Outcomes/Inputs (O/I)** ratio with that of a *referent* (peer, self past, market). Three states:
- O_self/I_self **<** O_other/I_other → underrewarded inequity (anger, lower effort).
- O_self/I_self **=** O_other/I_other → equity (motivated).
- O_self/I_self **>** O_other/I_other → overrewarded inequity (guilt — temporary).

**PYQ trap distinction:**
- **Equality** = same reward to all employees regardless of contribution.
- **Equity** = reward proportional to each employee's input/contribution.

**Application:** The firm gave **equality** (₹50,000 each), but Adams' theory says employees compare ratios. Analyst A: high inputs / equal outcome → underrewarded → demotivated. Analyst B: low inputs / equal outcome → overrewarded → guilt or complacency. Both feel inequity. **Fix:** move to *equity-based* differentiated bonuses tied to delivered margin.

Also relevant: **Distributive justice** (fairness of *amount* — broken here) and **Procedural justice** (fairness of *process*).

---

## Q19. [McClelland's 3 Needs — Quick ID] [2M]

Identify the dominant McClelland need for each:
(a) Anu volunteers for the toughest target every quarter and keeps a personal scoreboard.
(b) Bharat campaigns to head every committee and likes influencing peers.
(c) Chitra avoids assignments that pit her against colleagues and prizes team lunches.

**Model Answer**

McClelland's **3 acquired needs**:
- **nAch (Need for Achievement)** — drive to excel, beat standards.
- **nPow (Need for Power)** — drive to influence others.
- **nAff (Need for Affiliation)** — desire for warm relationships.

(a) → **nAch** (b) → **nPow** (c) → **nAff**

---

## Q20. [Fiedler's Contingency Model — PYQ repeat (Q5 paper 1)] [4M]

Riya is the new manager of a software team. She has **strong position power**, the **task is highly structured** (well-defined sprints), and her **leader-member relations are good** (the team trusts her). Identify the situational favourableness and the recommended Fiedler leadership style.

**Model Answer**

Fiedler's **Contingency Model** matches a leader's style (measured by **LPC — Least-Preferred Coworker** score) to **situational favourableness**, defined by 3 contingency variables:

1. **Leader-member relations** — Good / Poor
2. **Task structure** — High / Low
3. **Position power** — Strong / Weak

These yield **8 situational categories (I–VIII)**:
- I (Good / High / Strong) → Highly favourable → **Task-oriented (Low LPC)** is most effective.
- II–III → Favourable → Task-oriented.
- IV–VI → Moderate → **Relationship-oriented (High LPC)** most effective.
- VII–VIII → Highly unfavourable → Task-oriented.

**Application:** Good relations + High structure + Strong position power → **Category I — Highly favourable** → Riya should adopt a **Task-oriented (Low LPC)** style: focus on deliverables, sprint discipline, output. Relationship maintenance is already good and does not need leader effort.

---

## Q21. [Hersey-Blanchard SLT — PYQ repeat (Q1 model)] [4M]

Anjali leads a team of 5. Her direct reports are **capable but currently unwilling** to implement an upcoming restructuring. Using **Hersey-Blanchard's Situational Leadership Theory**, identify the readiness level and the recommended style.

**Model Answer**

**Hersey-Blanchard SLT** focuses on **follower readiness** = ability + willingness for the task. Four readiness levels and matching styles:

| Readiness | Description | Style | Behavior |
|---|---|---|---|
| **R1** | Unable + Unwilling | **Telling** | High task, Low relationship — define what/how/when |
| **R2** | Unable + Willing | **Selling** | High task + High relationship — direct + support |
| **R3** | Able + Unwilling | **Participating** | Low task + High relationship — share decisions, build buy-in |
| **R4** | Able + Willing | **Delegating** | Low task + Low relationship — hand over |

**Application:** "capable but unwilling" → **R3** → **Participating** style. Anjali should consult her team in restructuring decisions, surface concerns, share authority over implementation choices — willingness returns when followers feel ownership.

---

## Q22. [Path-Goal Theory — Likely application] [4M]

Saurav leads a team of new graduates working on an unstructured AI research project. They are eager but lack clear direction. Using **House's Path-Goal Theory**, recommend the leadership behavior and justify.

**Model Answer**

**Path-Goal Theory (House):** the leader's job is to clear the path to followers' goals by providing direction and support contingent on the situation. **4 leadership behaviors:**

1. **Directive** — schedules work, gives specific guidance.
2. **Supportive** — shows concern for followers' welfare.
3. **Participative** — consults followers, uses suggestions.
4. **Achievement-oriented** — sets challenging goals, expects high performance.

Two contingency factor sets:
- **Environmental:** task structure, formal authority, work group.
- **Subordinate:** locus of control, experience, perceived ability.

**Application:** Task is **unstructured** + subordinates are **inexperienced** → **Directive** behavior is most appropriate (provides the structure followers can't yet supply themselves). Once the project gains structure and members gain experience, Saurav should shift to **Achievement-oriented** behavior to leverage their drive.

---

## Q23. [Transformational vs. Transactional + Power — Likely Q] [4M]

Anjali "inspires employees beyond job expectations" and articulates a vision of company renewal during downsizing. (a) Identify her leadership style. (b) Distinguish transactional vs. transformational leadership. (c) List the **5 sources of power** she could draw upon.

**Model Answer**

**(a)** Anjali's style = **Transformational Leadership** — leaders who stimulate and inspire followers to achieve extraordinary outcomes; charismatic + visionary + developmental + intellectually stimulating.

**(b)** Distinction:
- **Transactional leaders** lead through *exchanges* — clarify role/task requirements and provide rewards contingent on performance. Operate within the existing system.
- **Transformational leaders** *transform* followers — appeal to higher ideals, create vision, encourage challenging the status quo. Build on but go beyond transactional.

**(c) 5 Sources of Power (French & Raven):**
1. **Legitimate power** — from formal position.
2. **Coercive power** — to punish/control.
3. **Reward power** — to give positive rewards.
4. **Expert power** — from expertise/knowledge.
5. **Referent power** — from personal traits, charisma, admiration.

Transformational leaders rely heavily on **Referent** and **Expert** power, less on Coercive.

---

## Q24. [Leadership Traits + Trust] [3M]

Name the **8 traits associated with leadership** and the **5 dimensions of trust** a leader must build.

**Model Answer**

**8 Leadership Traits:**
1. Drive
2. Desire to lead
3. Honesty and integrity
4. Self-confidence
5. Intelligence
6. Job-relevant knowledge
7. Extraversion
8. Proneness to guilt

**5 Dimensions of Trust:**
**Integrity · Competence · Consistency · Loyalty · Openness**

Of these, **Integrity** is the most critical — research shows employees rate it #1 in any leader-trust survey.

---

## Q25. [Org Design 6 Elements — PYQ-style restructuring case] [4M]

**Flipkart** is shrinking from 22 office floors to 5 to enable hybrid work. Identify the impact on each of the **6 key elements of organizational design** and state whether the firm is moving towards mechanistic or organic structure.

**Model Answer**

**6 Key elements of organizational design:**
1. **Work specialization** — degree of task subdivision.
2. **Departmentalization** — basis of grouping (functional, product, geographic, process, customer).
3. **Chain of command** — line of authority.
4. **Span of control** — direct reports per manager.
5. **Centralization vs. decentralization** — where decisions are made.
6. **Formalization** — degree of rule-bound standardization.

**Impact of the 22-to-5-floor shrink:**

| Element | Direction of change |
|---|---|
| Work specialization | Likely consolidates — fewer narrow roles |
| Departmentalization | Shifts towards cross-functional / project-based |
| Chain of command | Flattens — fewer hierarchical layers |
| **Span of control** | **Widens** — fewer managers per employee |
| **Centralization** | **Decreases** — hybrid empowers local decisions |
| **Formalization** | **Decreases** — fewer rigid in-office rules |

Net direction → **Organic structure** (collaborative, adaptable, few rules, decentralized, flat) — appropriate for Flipkart's dynamic e-commerce environment.

---

## Q26. [Mechanistic vs. Organic — Quick contrast] [3M]

Distinguish **mechanistic vs. organic** organizational structure on six attributes, and indicate which suits a stable mass-production firm and which suits a dynamic biotech R&D firm.

**Model Answer**

| Attribute | Mechanistic | Organic |
|---|---|---|
| Hierarchy | Rigid | Collaborative (vertical + horizontal) |
| Duties | Fixed | Adaptable |
| Rules | Many | Few |
| Communication | Formal channels | Informal |
| Decision authority | Centralized | Decentralized |
| Shape | Tall | Flat |

**Fit:** Mass-production (stable, high-volume, efficiency-driven) → **Mechanistic**. Biotech R&D (dynamic, innovation-driven, ambiguous tasks) → **Organic**. (Woodward + Lawrence/Lorsch contingency findings.)

---

## Q27. [Decision-Making Biases — Likely Q] [3M]

Identify the **decision-making bias** in each scenario:
(a) A manager keeps funding a failing project because "we've already spent ₹2 crore."
(b) A CEO recalls a recent plane crash and cancels all corporate flights.
(c) A founder is sure the new product will succeed because the prototype demo went well.

**Model Answer**

The 12 common biases include: Overconfidence, Immediate gratification, Anchoring, Selective perception, Confirmation, Framing, Availability, Representation, Randomness, Sunk costs, Self-serving, Hindsight.

(a) → **Sunk cost bias** — past unrecoverable spend wrongly influences forward decision.
(b) → **Availability bias** — recent vivid event distorts probability estimate.
(c) → **Overconfidence bias** — excess belief in one's own judgement / forecast.

---

## Q28. [BSC Design — PYQ repeat (Q14 paper 2 / Q6 model)] [4M]

**ZeroHunger Foods**, a green-foods startup, asks you to design a **Balanced Scorecard**. Build a 4-perspective table with one strategic objective, one KPI, and one initiative for each.

**Model Answer**

The **Balanced Scorecard (Kaplan & Norton)** measures performance across **4 perspectives**, linked by a strategy-map cause-and-effect chain: **Learning & Growth → Internal Process → Customer → Financial**.

| Perspective | Strategic objective | KPI | Initiative |
|---|---|---|---|
| **Financial** | Achieve sustainable revenue growth | Revenue growth (% YoY) | Direct-to-consumer subscription rollout |
| **Customer** | Build trust as the green-foods brand | Net Promoter Score (NPS) | Transparent farm-to-pack traceability portal |
| **Internal Business Process** | Reduce supply-chain waste | Spoilage rate (%) | Cold-chain digitization |
| **Learning & Growth** | Build sustainability competency | Training hrs/employee on ESG | Partnership with ag-tech accelerator |

Cause-and-effect: ESG-trained workforce → cold-chain efficiency → on-shelf product trust → revenue growth.

---

## Q29. [Control Types — Likely Q] [3M]

Distinguish **feedforward, concurrent, and feedback** control with one example from a manufacturing firm.

**Model Answer**

Controlling = monitoring, comparing, correcting performance against goals.

| Type | When | Example |
|---|---|---|
| **Feedforward control** | *Before* the activity — anticipates problems | Inspecting incoming raw-material steel for defects before the press line runs |
| **Concurrent control** | *During* the activity — real-time correction | Management by walking around (MBWA) on the assembly line; statistical process control alerts |
| **Feedback control** | *After* the activity — corrects after occurrence | Monthly defect-rate report; financial close report comparing actual vs. budgeted COGS |

The most cost-effective is **Feedforward** — preventing problems is cheaper than correcting them.

---

## Q30. [Value Chain — PYQ repeat (Q16 paper 1, Maruti Suzuki)] [4M]

Using **Porter's Value Chain framework**, draw the value chain for **Maruti Suzuki India Ltd.** (car manufacturer).

**Model Answer**

Porter's **Value Chain** decomposes a firm into **primary** and **support** activities; the difference between value created and total cost = **margin**, the source of competitive advantage.

**PRIMARY ACTIVITIES:**

1. **Inbound Logistics** — receiving steel coils, components, electronics from 200+ Tier-1 suppliers (e.g., Bharat Forge, Bosch India); inventory management at Manesar / Gurugram / Kharkhoda plants.
2. **Operations** — stamping, welding, painting, trim, assembly of Swift, Baleno, Brezza, Dzire, etc.
3. **Outbound Logistics** — dispatch via rail/road to ~4,000 dealerships nationwide; Nexa premium channel, Arena mass channel.
4. **Marketing & Sales** — TV/digital campaigns, festive offers, exchange schemes, Nexa brand for premium segment.
5. **Service** — 3,600+ Maruti Authorized Service Stations (MASS), Maruti Genuine Parts, AMCs, Maruti True Value (used cars).

**SUPPORT ACTIVITIES:**

1. **Firm Infrastructure** — Suzuki Motor Corp Japan as parent; finance, legal, planning, governance.
2. **HRM** — Maruti Training Academy, ITI partnerships, technician upskilling.
3. **Technology Development** — Rohtak R&D centre; Suzuki tech transfers; CNG/hybrid/EV development.
4. **Procurement** — vendor development; long-term supply contracts; cost-down programs.

**Margin** = customer-perceived value − total cost across the chain → enables Maruti's cost leadership.

---

## Q31. [Value Chain (Service) — Likely Q] [4M]

Draw the **value chain for an airline** such as IndiGo.

**Model Answer**

For service firms, value chain primary activities are reframed but structurally identical (Porter):

**PRIMARY ACTIVITIES:**
1. **Inbound Logistics** — fuel procurement (HPCL/IOC), aircraft leasing/purchase from Airbus, catering supplies, spares.
2. **Operations** — flight scheduling, ground handling, on-time turnaround at hubs (Delhi, Bengaluru, Hyderabad).
3. **Outbound Logistics** — passenger boarding, baggage handling, on-time arrival at destination.
4. **Marketing & Sales** — IndiGo website, mobile app, OTA partners (MMT, Yatra), corporate accounts.
5. **Service** — call centre, self-service rebooking, baggage tracking, frequent-flyer (BluChip).

**SUPPORT ACTIVITIES:**
1. **Firm Infrastructure** — InterGlobe Aviation finance, regulatory affairs (DGCA), safety management.
2. **HRM** — pilot/cabin-crew training academy, FlyAhead programmes.
3. **Technology Development** — Navitaire reservation, fuel-efficiency analytics, A320neo NEO engines.
4. **Procurement** — fuel contracts, lessor negotiation, ground-handling outsourcing.

**Margin** = on-time, low-cost, predictable service at a price point above unit cost. IndiGo's competitive advantage = cost leadership through rapid turnaround + single-fleet-type operations.

---

## Q32. [Marketing Mix 7Ps — PYQ repeat (Q17 paper 1, Rommy salon)] [4M]

**Rommy** is opening a hair salon in South Delhi for high-end customers seeking premium services. List the **7 Ps** Rommy must plan for, with one sentence each.

**Model Answer**

For a **service business**, the marketing mix extends the 4Ps to **7Ps** (Booms & Bitner):

1. **Product** — premium hair-care services (cuts, colour, keratin, scalp therapy) using imported salon-grade brands (e.g., L'Oréal Professionnel, Kérastase).
2. **Price** — *premium / skimming* pricing to signal exclusivity (₹3,500–₹15,000 per service).
3. **Place** — upscale retail location (e.g., Khan Market / DLF Emporio); appointment-only walk-in policy.
4. **Promotion** — Instagram influencer collaborations, referral rewards, exclusive launches; *not* mass-media.
5. **People** — internationally-trained stylists in branded uniform; concierge-style reception.
6. **Process** — pre-booking via app, consultation protocol, digital style-history record per client.
7. **Physical evidence** — boutique interior, signature scent, branded robes, espresso bar, Insta-worthy mirrors.

---

## Q33. [Marketing Mix 4Ps — Likely Q] [4M]

A consumer-electronics startup is launching a ₹15,000 **bone-conduction headphone for runners**. Plan the **4Ps**.

**Model Answer**

The **4 Ps of the Marketing Mix** for a *product*:

1. **Product** — bone-conduction wireless headphone; IP67 sweat-proof; 8-hr battery; 5 levels — Core benefit (audio while staying ambient-aware) → Basic (pair of bone-conduction transducers) → Expected (Bluetooth, charging cable) → Augmented (companion app, 1-yr warranty, free replacement of ear-hooks) → Potential (firmware-upgradeable noise profiles).
2. **Price** — *value-based* pricing at ₹15,000, mid-premium vs. Shokz (₹18,000) and budget Chinese imports (₹4,000) — positioning above the budget tier without crossing the import-premium ceiling.
3. **Place** — Amazon + Flipkart D2C; brand website with virtual fit-tool; Decathlon and 3 specialty running stores in Bengaluru/Pune/Mumbai.
4. **Promotion** — partnerships with marathon series (TCS World 10K, Mumbai Marathon); content-led influencer campaigns with running coaches; QR-code in-store demo zones.

Underlying STP: **Segmentation** (urban distance runners 25–45) → **Targeting** (mid-premium fitness-tech buyers) → **Positioning** ("audio + safety on the road").

---

## Q34. [BCG Matrix — PYQ-style portfolio Q] [4M]

**ITC Ltd.** has these business lines: (a) Cigarettes, (b) Hotels/Hospitality, (c) Agri-business / e-Choupal, (d) FMCG branded foods (Aashirvaad, Sunfeast, Bingo), (e) Education & Stationery (Classmate). Classify each into a **BCG quadrant** with one-line justification, and recommend the strategic action.

**Model Answer**

**BCG Matrix:** Y-axis = Industry growth rate; X-axis = Relative market share. Four quadrants:

| | High Market Share | Low Market Share |
|---|---|---|
| **High Growth** | **Stars** — invest to maintain | **Question Marks** — invest or divest |
| **Low Growth** | **Cash Cows** — milk for cash | **Dogs** — divest/liquidate |

**ITC portfolio classification:**

| Business | Quadrant | Justification | Action |
|---|---|---|---|
| Cigarettes | **Cash Cow** | Dominant share, mature/declining category | Milk; reinvest cash in Stars and QMs |
| Hotels | **Question Mark** | High-growth tourism, low ITC share vs. Taj/Oberoi | Invest selectively or divest non-core properties |
| Agri-business / e-Choupal | **Star** | High-growth, ITC has dominant rural-procurement network | Continue investing |
| FMCG branded foods | **Star** (some lines Question Mark) | High-growth segment, ITC building share against HUL/Nestlé | Aggressive marketing investment |
| Education & Stationery | **Question Mark / Dog** | Slower-growth, mid-to-low share | Decide: build to Star or divest |

The cigarette **Cash Cow** funds the **Stars** (Agri, Foods) — classic BCG cash-flow logic.

---

## Q35. [Porter's Generic Strategies — Likely Q] [3M]

Identify the **3 generic competitive strategies** of Porter and classify the following Indian firms: (a) Maruti Suzuki, (b) Apple India, (c) Patanjali Ayurved.

**Model Answer**

Porter's **3 Generic Strategies:**

1. **Cost Leadership** — lowest costs in the broad industry (not necessarily lowest *price*); broad market; e.g., scale, scope economies.
2. **Differentiation** — unique product/brand widely valued, allowing premium price; broad market.
3. **Focus** — *cost focus* or *differentiation focus* in a narrow niche/segment.

**Stuck in the middle** = no clear strategy → below-average performance.
**Blue Ocean** = combine cost + differentiation through value innovation.

**Application:**
(a) **Maruti Suzuki** → **Cost Leadership** (broad mass market; cost-down vendor ecosystem).
(b) **Apple India** → **Differentiation** (premium brand, ecosystem lock-in, charges premium).
(c) **Patanjali** → **Focus / Differentiation Focus** (Ayurveda-natural niche; later expanded toward broad differentiation).

---

## Q36. [Porter's 5 Forces — Likely Q] [5M]

Apply **Porter's 5 Forces** to the **Indian quick-commerce industry** (Blinkit, Zepto, Instamart). Rate each force as High / Medium / Low.

**Model Answer**

Porter's **5 Forces** model assesses industry attractiveness; *high force = lower profitability*. Strategy: position where forces are weakest.

| Force | Indicators | Q-commerce in India | Rating |
|---|---|---|---|
| **Threat of new entrants** | Capital, scale, brand, govt policy, switching costs | Heavy capex (dark stores, fleet); Amazon/Flipkart entering; FDI-funded competition | **High** |
| **Bargaining power of suppliers** | Few suppliers, unique inputs, switching costs | Many FMCG suppliers; quick-commerce buys at distributor terms; suppliers have moderate-to-low power | **Low–Medium** |
| **Bargaining power of buyers** | Few large buyers, standardized product, switching costs | Customers can switch apps in seconds; near-zero switching cost | **High** |
| **Threat of substitutes** | Substitute products at lower cost | Kirana stores, modern trade, scheduled grocery delivery (BB, Amazon Fresh) | **High** |
| **Rivalry among existing competitors** | Number, growth, fixed cost, exit barriers | 3–4 well-funded rivals burning cash; price/discount wars; high fixed-cost dark stores | **Very High** |

**Conclusion:** Industry attractiveness is **structurally low** today — high rivalry, high threat of substitutes, weak buyer lock-in. Profit will accrue only to the eventual scale leader; strategy = win the market-share race or exit.

---

## Q37. [SWOT — Likely Q] [4M]

Conduct a **SWOT analysis** for **Tata Motors' EV division** (Nexon EV, Tiago EV, Punch EV).

**Model Answer**

**SWOT Framework:**
- **Strengths** — internal positives.
- **Weaknesses** — internal negatives.
- **Opportunities** — external positives.
- **Threats** — external negatives.

| | Internal | External |
|---|---|---|
| Positive | **Strengths** | **Opportunities** |
| Negative | **Weaknesses** | **Threats** |

**Tata Motors EV SWOT:**

**S — Strengths:**
- ~70% market share in Indian passenger EV.
- Existing dealer network (~1,000+ outlets) for sales and service.
- In-house battery and powertrain capability (Tata Group: Tata Chemicals battery JV, Tata Power charging).

**W — Weaknesses:**
- Limited model range in higher-priced EV segment.
- Range anxiety vs. global rivals (BYD, MG).
- Quality perception gap vs. legacy ICE leaders.

**O — Opportunities:**
- Government FAME-II / PLI subsidies; state-level EV incentives.
- Rising fuel prices; rising EV awareness.
- Group synergy: Tata Power charging stations, Tata Capital financing.

**T — Threats:**
- Aggressive entry by **MG, BYD, Hyundai, Mahindra** in mass-market EV.
- Volatile lithium prices and battery supply-chain risk.
- Policy reversal risk (subsidies expiring).

**Strategic implication:** Use *Strengths* (network) and *Opportunities* (subsidies) to defend the SO quadrant — model expansion + faster charging-network rollout — before MG/BYD scale.

---

## Q38. [Financial Ratios — PYQ repeat (Q11 paper 1)] [5M]

The balance sheet of **MEGA Ltd.** as on 31-Mar-2024 (₹):

| ASSETS | ₹ | LIABILITIES & EQUITY | ₹ |
|---|---|---|---|
| Cash | 50,000 | Accounts Payable | 15,000 |
| Accounts Receivable | 30,000 | Short-term Loans | 5,000 |
| Inventory | 20,000 | Accrued Expenses | 5,000 |
| **Total Current Assets** | **1,00,000** | **Total Current Liabilities** | **25,000** |
| Land | 1,00,000 | Long-term Loans | 80,000 |
| Equipment | 50,000 | **Total Liabilities** | **1,05,000** |
| | | Common Stock | 50,000 |
| | | Retained Earnings | 95,000 |
| **Total Assets** | **2,50,000** | **Total Liab. & Equity** | **2,50,000** |

Compute and interpret: (a) Quick Ratio (b) Debt-to-Equity Ratio.

**Model Answer**

**(a) Quick Ratio (Acid-test):**
> QR = (Current Assets − Inventory) / Current Liabilities

= (1,00,000 − 20,000) / 25,000 = 80,000 / 25,000 = **3.2**

*Interpretation:* QR of 3.2 is well above the benchmark of 1.0 → strong liquidity. For every ₹1 of short-term obligations, MEGA holds ₹3.2 of *liquid* assets (excluding inventory). The company is comfortably solvent in the short term — possibly *over-liquid* (idle cash drag on returns).

**(b) Debt-to-Equity Ratio:**
> D/E = Total Debt / Total Shareholders' Equity

Total Equity = 50,000 + 95,000 = 1,45,000
Using Total Liabilities (1,05,000) for Debt:

D/E = 1,05,000 / 1,45,000 = **0.72**

*Interpretation:* D/E of 0.72 (<1) indicates **conservative financing** — equity exceeds debt. Low financial leverage, low solvency risk. The firm has substantial unused borrowing capacity for future expansion. Trade-off: foregoes the tax shield benefits of higher leverage.

---

## Q39. [Financial Ratios — Different format] [4M]

ABC Pvt. Ltd. reports: Net Sales = ₹40,00,000; Net Income = ₹4,00,000; Total Assets = ₹25,00,000; Shareholders' Equity = ₹15,00,000; COGS = ₹24,00,000; Average Inventory = ₹4,00,000. Compute (a) Net Profit Margin, (b) ROA, (c) ROE, (d) Inventory Turnover. Interpret each.

**Model Answer**

**(a) Net Profit Margin** = Net Income / Net Sales × 100 = 4,00,000 / 40,00,000 × 100 = **10%**
*Interpretation:* For every ₹100 of sales, ₹10 becomes profit — healthy for most industries.

**(b) Return on Assets (ROA)** = Net Income / Total Assets × 100 = 4,00,000 / 25,00,000 × 100 = **16%**
*Interpretation:* Each ₹1 of assets generates ₹0.16 of profit — strong asset productivity.

**(c) Return on Equity (ROE)** = Net Income / Equity × 100 = 4,00,000 / 15,00,000 × 100 = **26.67%**
*Interpretation:* Shareholders earn ₹26.67 on every ₹100 of their equity. ROE > ROA confirms positive financial leverage (debt is amplifying shareholder returns).

**(d) Inventory Turnover** = COGS / Average Inventory = 24,00,000 / 4,00,000 = **6 times/year**
*Interpretation:* Inventory cycles every ~60 days. Whether good depends on industry — fast-moving FMCG would expect higher; durables lower.

---

## Q40. [Fund Flow Statement — PYQ repeat (Q16 paper 2)] [4M]

ABC Pvt. Ltd. is a mid-sized firm. Income statement shows 10% YoY revenue growth, but net income is *declining*; balance sheet shows substantial increase in **fixed assets (machinery, factory buildings)** alongside a rise in **long-term borrowings**. The Fund Flow Statement (FY 2024-25) is below.

**Sources of Funds (₹ lakhs):**
| | |
|---|---|
| Long-term Borrowings (new loan) | 500 |
| Retained Earnings (from net profit) | 120 |
| Sale of Old Equipment | 30 |
| **Total Sources** | **650** |

**Application of Funds (₹ lakhs):**
| | |
|---|---|
| Purchase of Fixed Assets (Machinery, Plant) | 450 |
| Construction of New Factory Building | 100 |
| Increase in Working Capital (Net) | 70 |
| Repayment of Short-term Loans | 30 |
| **Total Applications** | **650** |

State **four key observations** about the firm's financial situation.

**Model Answer**

A **Fund Flow Statement** explains *changes between two balance sheets*: Sources (↑ liability/equity, ↓ asset) vs. Applications (↓ liability/equity, ↑ asset). For ABC Ltd.:

**Observation 1 — Heavy capacity expansion in progress.**
₹450L (machinery) + ₹100L (building) = ₹550L (~85% of total fund deployment) into long-term productive assets. This explains the rise in fixed assets noted on the balance sheet.

**Observation 2 — Capex is debt-funded, not earnings-funded.**
Long-term borrowings of ₹500L finance most of the ₹550L capex; retained earnings only ₹120L. Future debt-servicing obligation rises → could be the cause of *declining net income* (interest expense up).

**Observation 3 — Improved short-term debt profile.**
₹30L of short-term loans repaid → refinanced into long-term debt, improving liquidity profile and reducing rollover risk.

**Observation 4 — Working capital rising in line with operations.**
₹70L additional working capital is consistent with 10% revenue growth — supports higher receivables/inventory at the expanded scale.

**Diagnosis:** ABC is in a *capacity-expansion + leverage-up* phase. Falling net income is structural (depreciation + interest) and should be acceptable *if* the new capacity drives revenue and operating profit growth in future years. Risk: if revenue growth stalls, the firm could face debt-servicing strain.

---

## Q41. [Capital Structure vs. Capital Budgeting — PYQ repeat (Q10 paper 1)] [3M]

Distinguish **Capital Structure** and **Capital Budgeting** with one example each from a manufacturing firm.

**Model Answer**

**Capital Structure** = the firm's mix of long-term financing — **debt vs. equity** — used to finance its operations and growth. Goal: minimise **WACC** (weighted average cost of capital) while keeping financial risk acceptable.
- *Example:* Tata Motors finances 40% of long-term capital through debt (bonds, term loans) and 60% through equity (shareholders' funds + retained earnings). Debt is cheaper due to the **interest tax shield**; too much debt raises bankruptcy risk.

**Capital Budgeting** = process of evaluating and selecting **long-term investment projects** that exceed cost of capital. Tools: Payback period, NPV, IRR, Profitability Index.
- *Example:* Tata Motors evaluates a new ₹2,000 cr EV assembly line. Forecasts cash inflows over 10 years, discounts at WACC, computes NPV; accepts if **NPV > 0**, IRR > cost of capital.

**Distinction:** Capital structure is about **how to finance** (right side of balance sheet); capital budgeting is about **what to invest in** (left side). Both interact via WACC.

---

## Q42. [HRM Decruitment — PYQ repeat (Q14 paper 1)] [3M]

A CEO has decided to close two product lines — ~500 employees affected. List the **7 decruitment options**, and recommend two methods most appropriate here, with justification.

**Model Answer**

**7 Decruitment Options (Robbins, Exhibit 10-5):**
1. **Firing** — permanent involuntary termination
2. **Layoffs** — temporary involuntary termination
3. **Attrition** — not filling voluntary departures
4. **Transfers** — lateral or downward movement
5. **Reduced workweeks** — fewer hours / part-time
6. **Early retirement** — incentivising older employees to retire
7. **Job sharing** — two people share one position

**Recommended for the 500-job scenario:**

**(i) Voluntary Early Retirement** — least adversarial; protects institutional knowledge departure (older workers exit with dignity); reduces legal exposure; signals goodwill to remaining workforce.

**(ii) Transfers** — redeploy capable employees to growing parts of the business (different product lines, geographies). Preserves human capital, reduces severance cost, maintains employer brand.

If the gap remains: **Attrition** (slower, no public layoff event) before resorting to **Layoffs / Firing**.

---

## Q43. [HRM Workforce Diversity — PYQ repeat (Q3 paper 1)] [4M]

A modular kitchen firm wants to fill 5 newly created **HRM positions**, ensuring workforce diversity. Identify the nature of HRM and the development steps required to source talent and ensure diversity.

**Model Answer**

**HRM** = the management function concerned with attracting, developing, and retaining a competent and committed workforce. **Process flow (Exhibit 10-2):**

**HR Planning → Recruitment / Decruitment → Selection → Orientation → Training → Performance Mgmt → Compensation & Benefits → Career Development**

**Step plan to fill the 5 positions with diversity:**

1. **HR Planning** — job analysis: prepare job descriptions and job specifications for the 5 roles; identify diversity gaps in current workforce.
2. **Recruitment** — multi-channel sourcing: campus placements (women-led B-schools), Naukri, LinkedIn, employee referrals, return-to-work programs for women on career break.
3. **Selection** — structured interviews + behavioural tests (high *validity* and *reliability*); diverse interview panels to reduce bias.
4. **Orientation** — work-unit + organization orientation including diversity-and-inclusion module.
5. **Training** — on-the-job mentoring; off-the-job D&I sensitization for managers.
6. **Performance Management** — BARS or 360-degree appraisal; bias-audit annual review.
7. **Compensation** — equitable pay band; pay-equity audit.
8. **Career Development** — sponsored growth paths, ERGs (Employee Resource Groups).

**Diversity benefits:** broader talent pool, richer decision-making, customer-base reflection, lower groupthink.

---

## Q44. [Performance Appraisal Methods — Likely Q] [3M]

Name **5 performance appraisal methods**, and recommend the most suitable for a **call-centre tele-sales agent** vs. a **senior R&D scientist**.

**Model Answer**

**Performance appraisal methods:**
1. **Written essays** — narrative description of strengths/weaknesses.
2. **Critical incidents** — focus on key behaviours (effective vs. ineffective).
3. **Graphic rating scales** — rating along a list of factors.
4. **BARS (Behaviourally Anchored Rating Scales)** — rating scales with specific behavioural examples per anchor.
5. **Multi-person comparisons** — relative ranking among peers.
6. **MBO (Management by Objectives)** — performance vs. mutually-agreed goals.
7. **360-degree feedback** — peers + subordinates + supervisor + self + customer.

**Tele-sales agent** → **Graphic rating scales** + **Multi-person comparisons** — high-volume, comparable task, easy to score on calls/conversion.

**Senior R&D scientist** → **MBO** + **360-degree feedback** — output is non-routine, multi-year; goals must be mutually-agreed; peer/cross-functional input critical for evaluating research quality.

---

## Q45. [MBO and SMART Goals — Likely Q] [3M]

Define **MBO** and list its process steps. State the **SMART** criteria for goal-setting.

**Model Answer**

**MBO (Management by Objectives)** — a process by which superiors and subordinates jointly identify common goals, define each individual's responsibilities in terms of expected results, and use those goals as the basis for evaluating performance.

**MBO Process:**
1. **Set organizational objectives** (top management).
2. **Cascade to departmental goals**.
3. **Translate into individual goals** (joint manager-subordinate negotiation — *mutual agreement*).
4. **Develop action plans**.
5. **Periodic review** of progress.
6. **Performance appraisal** linked to goal achievement.

**SMART** goals are:
- **S**pecific
- **M**easurable
- **A**ttainable
- **R**elevant
- **T**ime-bound

Together: MBO provides the *process*; SMART provides the *quality criterion* for goals within that process.

---

## Q46. [Scenario Planning — PYQ repeat (Q18 paper 2)] [3M]

Why is **scenario planning** essential in resource-allocation decisions, and how can it help operations managers prepare for uncertainty? Answer in concise bullet points.

**Model Answer**

**Scenario planning** = preparing for *multiple plausible futures* (best-case, base-case, worst-case) rather than a single forecast.

**4-step process:**
1. Identify key drivers of change (demand, raw-material price, regulation, technology).
2. Develop 3–4 distinct, internally consistent scenarios.
3. Analyze impact of each scenario on operations (capacity, sourcing, headcount).
4. Prepare contingent strategies and trigger points for each.

**Why essential for resource allocation:**
- Forecasts have high error under uncertainty; scenario planning **stress-tests** the resource plan against multiple futures.
- Identifies **leading indicators** (early warning signals) that should trigger reallocation.
- Builds **strategic flexibility** — pre-committed responses, not reactive scrambling.
- Reduces **sunk-cost lock-in**: capacity, hiring, contracts can be modular/staged rather than all-or-nothing.

---

## Q47. [Value Chain Contribution — PYQ repeat (Q17 paper 2)] [4M]

How does **value chain management** contribute to improving operational efficiency and customer value in a business? Answer in **four concise bullet points**.

**Model Answer**

**Value Chain Management** = the process of managing the entire sequence of integrated activities and information about product flows, from raw material through end customer.

1. **Eliminates non-value-adding activities** — by mapping primary (Inbound → Operations → Outbound → Marketing & Sales → Service) and support (Infrastructure, HRM, TechDev, Procurement) activities, managers identify and remove waste, reducing cost without reducing customer value.
2. **Coordinates suppliers and partners end-to-end** — a managed chain shares demand signals, shrinks lead times, lowers inventory and stock-outs (e.g., Maruti's tier-1 vendor cluster around Manesar).
3. **Drives customer value through alignment of activities** — every activity is evaluated against "does this add value the customer pays for?" — the **Margin** = perceived value − total cost is maximised.
4. **Builds competitive advantage** — sustained either through cost leadership (efficient chain) or differentiation (chain that produces unique customer-valued features) — the basis of Porter's Generic Strategies.

---

## Q48. [Conflict Resolution + Strategy — PYQ repeat (Q12 paper 1)] [4M]

Iyana works in a 4-team-member project group. Members disagree on **sales targets** (task disagreement) and a colleague Bhuvan has refused to talk to her after a personal argument (interpersonal). (a) Identify the two conflict types. (b) Recommend the **Thomas-Kilmann conflict-handling style** for each.

**Model Answer**

**(a) Conflict types:**

| Disagreement | Type | Functional? |
|---|---|---|
| Sales targets | **Task conflict** | Functional at low–moderate level (improves decisions) |
| Bhuvan refusing to speak | **Relationship conflict** | Always **dysfunctional** |

**(b) Thomas-Kilmann's 5 conflict-handling styles:**
- **Competing/Forcing** (high assertive, low cooperative) — win-lose
- **Collaborating** (high assertive, high cooperative) — win-win
- **Compromising** (moderate, moderate) — split
- **Avoiding** (low, low) — withdraw
- **Accommodating** (low assertive, high cooperative) — yield

**Recommendations:**
- **Sales-target task conflict** → **Collaborating** — bring data, surface assumptions, reach a jointly owned target. Allowing structured task disagreement improves quality.
- **Personal/relationship conflict with Bhuvan** → **Accommodating** initially (de-escalate; rebuild rapport) followed by **Collaborating** in a structured 1-on-1 once emotion subsides. **Avoiding** is wrong because relationship conflict festers if ignored.

---

## Q49. [Leadership + Motivation Integrated Case — PYQ-style integrated Q] [5M]

Ram, a startup founder, leads a team of fresh graduates. They are **enthusiastic but lack skills** (R2). The compensation is below market. Ram says he wants to "**inspire** the team to do extraordinary work." Diagnose using:
(a) Hersey-Blanchard SLT — recommended style.
(b) Vroom's Expectancy Theory — what's at risk.
(c) Bass — Ram's leadership style label.
(d) Herzberg — likely effect of compensation.

**Model Answer**

**(a) Hersey-Blanchard SLT:** R2 = unable + willing → **Selling style** (high task + high relationship). Ram should provide both clear instruction (task structure) and supportive coaching (relationship). Pure delegation will fail; pure directing will demotivate.

**(b) Vroom's Expectancy Theory:** Motivation = E × I × V.
- *Expectancy* (effort → performance) is at risk because skills are inadequate → no matter the effort, performance lags. Fix with training.
- *Instrumentality* depends on whether Ram links performance to reward — risky in cash-short startups.
- *Valence* — below-market cash will have low valence unless replaced with non-cash (equity, learning, career trajectory) that the team values.

**(c) Bass's classification:** Ram's stated intent "inspire to extraordinary work" maps to **Transformational Leadership** — vision-led, charismatic, developmental — versus Transactional leadership of routine reward-for-performance.

**(d) Herzberg:** Compensation is a **hygiene factor**. Below-market pay creates **dissatisfaction**; raising it to market clears dissatisfaction but does **not** create satisfaction. Lasting motivation must come from **motivators** — Achievement, Recognition, Work itself, Responsibility, Advancement, Growth — which Ram's transformational, autonomy-rich startup environment can provide.

**Integrated recommendation:** Ram should (1) adopt Selling SLT style, (2) invest in skill training to repair Expectancy, (3) supplement cash with equity + visible recognition + ownership (motivators) to compensate for hygiene gap.

---

## Q50. [Strategic Mgmt Process + BSC + Value Chain Integration — PYQ-style integrated final Q] [6M]

You are a strategy consultant hired by **GreenWheels Ltd.**, an Indian electric two-wheeler startup. (a) List the **6 steps of the Strategic Management Process**. (b) For step 4 (SWOT), identify two each of S, W, O, T. (c) Based on the SWOT, recommend a **generic strategy** (Porter) and one **growth strategy**. (d) Build a 4-perspective **BSC** with one KPI per perspective.

**Model Answer**

**(a) Strategic Management Process — 6 steps:**
1. Identify the organization's current mission, goals, strategies.
2. **External analysis** — opportunities and threats.
3. **Internal analysis** — strengths and weaknesses.
4. **SWOT** — combine 2 + 3.
5. **Formulate strategies** — corporate, competitive, functional.
6. **Implement and evaluate** strategies.

**(b) GreenWheels SWOT:**

| | Internal | External |
|---|---|---|
| Positive | **S:** EV-only design from ground up; lean local supply chain | **O:** FAME-II + state EV subsidies; rising fuel prices |
| Negative | **W:** Limited capital; under-developed service network | **T:** Heavyweight rivals (Bajaj Chetak, TVS iQube, Ola Electric); battery-supply volatility |

**(c) Strategy recommendations:**
- **Generic strategy** → **Differentiation Focus** — niche on a specific segment (e.g., urban delivery riders) with EV-tailored features (swap-battery, telematics) rather than competing head-on in mass market against Ola/TVS.
- **Growth strategy** → **Concentration** initially (deepen the chosen niche before diversifying); later move to **Vertical Integration backward** into batteries to control supply.

**(d) BSC for GreenWheels:**

| Perspective | KPI | Target |
|---|---|---|
| **Financial** | Revenue per vehicle sold | ₹85,000 |
| **Customer** | Customer Satisfaction (NPS) | > 60 |
| **Internal Business Process** | Production cycle time | < 18 hrs |
| **Learning & Growth** | Engineer training hrs (EV systems) | 60 hrs/employee/yr |

Strategy-map causal chain: trained EV engineers (L&G) → faster, defect-free production (IBP) → satisfied delivery-rider customers (Customer) → revenue and margin growth (Financial).

---

# APPENDIX — LAST-MILE FRAMEWORK CHECKLIST

Before writing any answer, ask: **(1) Which framework? (2) Can I list its components verbatim? (3) Have I applied it to the specific case in the question?** All three steps must be visible on the page for full marks.

| If question mentions… | Reach for… |
|---|---|
| "Payoff matrix" | Maximax / Maximin / Laplace / Minimax Regret |
| "Balance sheet" | Quick / Current / D-E / ROA / ROE / Inventory Turnover |
| "Sources and Applications of funds" | Fund Flow Statement interpretation |
| "Car/airline/firm — value created" | Porter Value Chain (5 primary + 4 support) |
| "Marketing strategy / new product or service" | 4Ps (product) or 7Ps (service) |
| "Where to invest among SBUs" | BCG Matrix |
| "Industry analysis" | Porter's 5 Forces |
| "Competitive positioning" | Porter's 3 Generic Strategies |
| "Followers willing/unwilling, able/unable" | Hersey-Blanchard SLT |
| "Leader-member relations / task structure / position power" | Fiedler |
| "Inspires beyond exchange" | Transformational Leadership |
| "Pay raise didn't motivate" | Herzberg (salary = hygiene) |
| "Effort-Performance-Reward broken" | Vroom Expectancy |
| "Identical bonus despite different output" | Adams Equity (Equality vs Equity trap) |
| "Job redesign" | JCM 5 dimensions + MPS = (SV+TI+TS)/3 × A × F |
| "Performance dashboard" | Balanced Scorecard 4 perspectives |
| "Restructuring / shrinking layers" | 6 Org Design elements + Mechanistic↔Organic |
| "Layoffs / downsizing" | 7 Decruitment options |
| "Conflict in team" | Task / Relationship / Process + Thomas-Kilmann |
| "Communication channel" | Channel Richness hierarchy + selection criteria |
| "Cultural differences across countries" | Hofstede 5 |
| "Manager's day" | Mintzberg 10 roles |
| "Skill needed for promotion" | Katz 3 skills |
