# Assignment 3: Multi-Dimensional Classification of Extremist Content

### Part A: Enhance your previous extremism detection system

Paper read: Violent Versus Nonviolent Actors: An Empirical Study of Different Types of Extremism (https://www.researchgate.net/publication/320574618_Violent_Versus_Nonviolent_Actors_An_Empirical_Study_of_Different_Types_of_Extremism )




## Overview
 
This assignment extends the Assignment 2 extremism detector into a multi-dimensional classification system. Rather than treating extremism as a binary label, content is analyzed across five dimensions:
 
1. Violent vs. non-violent extremism classification
2. Sub-type identification
3. Target group detection
4. Severity assessment
5. Counter-narrative generation
 
The work spans three parts and uses multiple LLMs via the **CMU AI Gateway**.
 
---
 
## Part A: Enhanced Extremism Detection
 
**Model:** `gpt-4.1-mini` via the CMU AI Gateway  
> The originally specified `gpt-4o-mini-2024-07-18` was no longer available on the platform; `gpt-4.1-mini` was used as the closest substitute.
 
### Q1: Key Differences and Similarities Between VE and NVE
 
Based on Knight, Woodward, and Lancaster (2017), who analyzed 40 in-depth case studies (20 violent, 20 nonviolent):
 
**Similarities:** All 40 individuals showed evidence of perceiving an external threat, seeking like-minded peers, and holding strong ideological convictions. Factors commonly associated with extremism (identity-seeking, sense of purpose, hostility toward outgroups) appeared at comparable rates in both groups and cannot reliably distinguish between VE and NVE on their own.
 
**Key Differences (behavioral and psychological):**
 
- **Personal responsibility to act** — the strongest differentiator. 87.5% of VEs felt a personal duty to translate ideology into action, compared to only 18.8% of NVEs.
- **Exposure to extreme violence** — VEs were significantly more likely to have been exposed to real-world violence (conflict zones, personal victimization, violent networks), potentially lowering their threshold for committing it.
- **Individual vulnerability factors** — VEs showed higher rates of low self-esteem, bullying history, and patterns of underachievement, creating a psychological need to prove oneself through dramatic action.
- **Operational preparation** — VEs were more likely to have attended training camps or traveled to conflict areas, indicating concrete escalation beyond rhetoric.
- NVEs remained within ideological expression: distributing propaganda, sending hate mail, or fundraising, without direct involvement in physical harm.
 
**Important Caveat:** The authors caution that some NVE individuals may be pre-violent — not yet violent, but potentially on a trajectory toward violence.
 
### Q2: How Radicalization and Extremism Are Defined
 
The authors adopt behavior-grounded operational definitions to address definitional inconsistency in the literature:
 
- **Extremism** is defined pragmatically as individuals who hold attitudes and beliefs falling well outside mainstream opinion on political or ideological issues AND who have been convicted of a criminal offense with evidence of extremist motivation.
- **Radicalization** draws on the UK Home Office definition: the process by which individuals come to support terrorism and extremism and, in some cases, participate in terrorist activity. Critically, radicalization does not inevitably lead to violence.
 
**Methodology:** Detailed case studies were constructed from open-source data (court records, media reports, official documents) for all 40 individuals, then coded against a framework of themes, subthemes, and variables. VEs and NVEs were compared statistically to identify which factors meaningfully separate the groups.
 
### Q3: VE and NVE Definitions for Prompt Design
 
| Category | Definition |
|----------|-----------|
| **Violent Extremism (VE)** | Any act (or clearly intended act) constituting murder, attempted murder, assault, or serious physical harm to persons or property in pursuit of an ideological goal. Includes individuals who conduct nonviolent behaviours that knowingly facilitate violence by others (financing attacks, providing weapons, operational planning). |
| **Non-Violent Extremism (NVE)** | The promotion, dissemination, or support of an extreme ideology through non-physical means, without direct involvement in or facilitation of violence (e.g., distributing extremist literature, sending hate mail, fundraising for extremist organisations). |
 
**Key insight for prompt design:** Ideology and grievance alone cannot separate VE from NVE. The classifier focuses on whether the text involves, plans, or enables physical harm (VE) versus whether it remains confined to ideological hostility without an actionable violence component (NVE).
 
### Prompt Design Choices
 
- **Conservative default:** Ambiguous posts default to NVE. Falsely flagging non-violent speech as violent extremism carries serious real-world consequences.
- **Facilitation clause:** Content that funds or logistically supports violence is classified as VE even if the author did not personally commit violence.
- **Confidence calibration:** Forces the model to express uncertainty (HIGH / MEDIUM / LOW) rather than defaulting to high confidence.
- **Text span requirement:** Every classification must cite the specific phrase that drove the decision.
- **Sub-type grounded in literature:** Ideological / Political / Religious categories informed by Berhoum et al. (2023) and Ismail et al. (2025).
- **Target group constrained categories:** A defined set of demographic categories (religious, ethnic/racial, political, gender, nationality, occupation) was provided to minimize vague "Other" labels.
 
### Classification Results
 
**Dataset Summary:** Working pool of 550 rows. Stopped early once 50 VE + 50 NVE confirmed. Final file: 100 balanced rows. Refusals logged separately (150 rows).
 
| Class | Count (%) |
|-------|-----------|
| Violent Extremism (VE) | 50 (50.0%) |
| Non-Violent Extremism (NVE) | 50 (50.0%) |
 
**Confidence Distribution:**
 
| Confidence | Count (%) |
|------------|-----------|
| HIGH | 78 (78.0%) |
| LOW | 20 (20.0%) |
| MEDIUM | 2 (2.0%) |
 
**VE Sub-Type Classification (50 VE posts):**
 
| Sub-Type | Count (%) |
|----------|-----------|
| IDEOLOGICAL | 16 (32.0%) |
| Refusal (model declined) | 16 (32.0%) |
| Unclear | 13 (26.0%) |
| POLITICAL | 3 (6.0%) |
| RELIGIOUS | 2 (4.0%) |
 
**Top Targeted Groups:** women (3), Muslims (2), migrant families (1), immigrants (1), illegal aliens (1), Giants fans (1), IAS officers (1), doctors (1), conservatives (1), political puppets (1)
 
### Analysis
 
- **Targeting patterns:** Women are the most frequently targeted group, followed by Muslims. Patterns reflect real-world extremist grievances: gender-based hostility, anti-Islamic sentiment, anti-immigration rhetoric, and political scapegoating.
- **Most common sub-type:** IDEOLOGICAL at 32% of VE posts, aligning with the dataset's social media context. A notable 32% resulted in model refusals for sub-type classification due to content filtering.
- **VE vs NVE balance:** The final dataset is balanced 50/50 by design, though the original Kaggle dataset skewed toward EXTREMIST posts.
 
### Interpretability
 
For every classification, the LLM was required to provide: specific text spans, an explanation justifying the classification, and a confidence level. This triple output is stored in `ve_nve_classifications.csv` for auditability.
 
---
 
## Part B: Prompt Engineering Comparison
 
**Model:** `gemini-2.5-flash`  
> Originally specified `llama3-2-11b-instruct`; `gemini-2.5-flash` used based on gateway availability.
 
### Severity Scale (from arXiv:2506.00973)
 
| Level | Definition |
|-------|-----------|
| 1 | General extremist interest or ideology with no direct threat |
| 2 | Propaganda, hate speech, or dehumanizing language targeting a group |
| 3 | Glorifying past violence or inciting non-violent illegal acts |
| 4 | Direct incitement to violence or specific threats against a person or group |
| 5 | Immediate threat, planning an attack, or tactical instructions for violence |
 
### Experimental Design (2x2)
 
100 examples (50 VE + 50 NVE) evaluated across four conditions:
 
|  | Zero-Shot | Chain-of-Thought |
|--|-----------|-----------------|
| **Default Temperature (0.7)** | Condition 1 | Condition 3 |
| **Temperature = 0.4** | Condition 2 | Condition 4 |
 
### Q1: Agreement Between Conditions
 
| Comparison | % Identical |
|------------|-------------|
| Zero-Shot 0.7 vs Zero-Shot 0.4 | 86.0% |
| CoT 0.7 vs CoT 0.4 | 90.0% |
| Zero-Shot 0.7 vs CoT 0.7 | 72.0% |
| Zero-Shot 0.4 vs CoT 0.4 | 79.0% |
 
**Key finding:** Prompting strategy (zero-shot vs CoT) matters more than temperature for severity assessment. Temperature changes yielded 86-90% agreement; prompting method changes yielded only 72-79%.
 
### Q2: Average Word Count per Run
 
| Condition | Avg Severity | Avg Word Count |
|-----------|-------------|----------------|
| Zero-Shot (0.7) | 2.97 | 20.5 |
| Zero-Shot (0.4) | 2.91 | 19.2 |
| CoT (0.7) | 2.87 | 11.8 |
| CoT (0.4) | 2.82 | 12.1 |
 
**Observation:** Zero-shot explanations are notably longer (~20 words) than CoT (~12 words). CoT structures reasoning into discrete steps, each individually concise, while producing more consistent outputs across temperature conditions (90% vs 86% agreement).
 
---
 
## Part C: Counter-Narrative Generation
 
**Setup:** 20 extremist posts sampled (random_state=42), temperature=0.7.
 
### Personas
 
| Persona | Approach |
|---------|----------|
| **Vanilla (Baseline)** | No specific persona, used as baseline |
| **Educator** | Objective educator correcting misinformation through calm, evidence-based teaching |
| **Compassionate NGO** | Humanitarian worker appealing to shared humanity |
| **Law Enforcement** | Professional officer warning about consequences and deterring harmful behavior |
 
### Model Comparison
 
| Task | Models Compared | Focus |
|------|----------------|-------|
| Task 1: Training Modes | `gpt-4.1-mini` (RLHF) vs `llama3-2-11b-instruct` (instruction fine-tuning) | Different training paradigms |
| Task 2: Cost vs Quality | `claude-sonnet-4` (higher tier) vs `claude-3-haiku` (cheaper) | Same provider, different tiers |
| Task 3: Best vs Best | Best from Task 1 vs Best from Task 2 | Cross-provider champion comparison |
 
### Verbosity Findings
 
- `claude-sonnet-4` produced the most verbose responses (larger model capacity, thorough outputs)
- `claude-3-haiku` produced the shortest (designed for speed and brevity)
- Law Enforcement and Educator personas tended toward shorter, directive text
- Compassionate NGO persona produced longer, more empathetic narratives
 
### Clarity and Readability
 
**Flesch Scoring (1-5 scale):**
 
| Flesch Reading Ease | Scale |
|--------------------|-------|
| >= 80 | 5 — Very easy |
| 70-79 | 4 — Easy |
| 60-69 | 3 — Standard |
| 50-59 | 2 — Somewhat difficult |
| < 50 | 1 — Difficult |
 
**LLM-Based Clarity** (GPT-4.1-mini evaluation, 1-5 rubric for general public audience at 8th-grade level)
 
### When Do Flesch and LLM Clarity Disagree?
 
The two metrics frequently disagree because they measure fundamentally different things. Flesch measures surface-level textual features (syllable count, sentence length), while LLM-based clarity evaluates semantic coherence, logical flow, and persuasive effectiveness.
 
| Example Type | Flesch / LLM | Why They Disagree |
|-------------|-------------|-------------------|
| Short, choppy Law Enforcement response | 5 / 2 | Abrupt, lacks context, feels threatening rather than persuasive |
| Long, empathetic NGO narrative | 2 / 5 | Logically coherent, emotionally resonant, clearly structured |
| Educator response with technical refs | 3 / 4 | Well-organized evidence-based argument despite some jargon |
| Brief vanilla response | 5 / 2 | Too vague, no substantive counter-argument |
| Sonnet response with rhetorical questions | 1 / 4 | Highly engaging and persuasive despite surface complexity |
 
**Conclusion:** The LLM-based clarity metric is more appropriate for counter-narratives. Counter-narratives require persuasive reasoning, emotional appeal, and logical argumentation — none of which Flesch formulas were designed to capture.
 
### Verbosity vs Readability
 
- More verbose responses do not necessarily score lower on LLM clarity
- Extremely brief responses tend to score low on LLM clarity despite high Flesch scores
- Optimal counter-narrative length appears to be **80-120 words**
- `claude-3-haiku`'s brevity sometimes sacrificed persuasive depth; `claude-sonnet-4`'s verbosity occasionally drifted into complexity
 
---
 
## Deliverables
 
| File | Description |
|------|-------------|
| `ve_nve_classifications.csv` | 50 VE + 50 NVE balanced dataset with labels, sub-types, target groups, text spans, explanations, confidence levels (100 rows) |
| `ve_nve_classifications_full_with_refusals.csv` | Full log including 50 refusals (150 rows) |
| `severity_comparison_results.csv` | Severity scores and explanations for all 4 conditions across 100 posts |
| `counter_narratives_all.csv` | All counter-narratives: 4 personas × 4 models × 20 posts |
| `counter_narratives_analyzed.csv` | Counter-narratives with verbosity, Flesch scores, and LLM clarity scores |
| `part_a_balanced_analysis.png` | Part A classification distribution charts |
| `part_b_severity_analysis.png` | Part B severity comparison visualizations |
| `part_c_analysis.png` | Part C analysis visualizations |
 
---
 
## Limitations and Future Work
 
- The CMU AI Gateway did not support `gpt-4o-mini-2024-07-18` or `llama3-2-11b-instruct` during implementation; substitutes were used.
- Content filtering by some models occasionally blocked classification or generation, leading to 50 refusals in Part A.
- Sample sizes (200 for Part A working pool, 100 for Part B, 20 for Part C) balance API cost with statistical coverage but may not capture rare sub-types.
- Flesch readability was not designed for persuasive or counter-narrative content and should be treated as a secondary metric.
- Future work could incorporate few-shot prompting, fine-tuned models on counter-narrative datasets, or human evaluation of counter-narrative effectiveness.
 
---
 
## References
 
1. Knight, R., Woodward, K., & Lancaster, G. (2017). Violent versus nonviolent actors: An empirical study of different types of extremism. *Journal of Threat Assessment and Management, 4*(4), 221-234.
2. Berhoum, A., et al. (2023). Sub-type classification — ideological, political, religious extremism. *ACM Digital Library*.
3. Ismail, et al. (2025). VE sub-type analysis. *Nature Scientific Reports*.
4. Severity Level Assignment for terrorism content (*arXiv:2506.00973*).
5. Readable.com — Flesch Reading Ease / Flesch-Kincaid Grade Level methodology.
 