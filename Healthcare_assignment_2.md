# Assignment 2: Social Media Extremism Detection Challenge
## Healthcare Group — README / Analysis Report

**Team:** Healthcare Group (Edtech — Nthabiseng)  
**Course:** Machine Learning, CMU Africa  
**Date:** February 2026  
**Dataset:** [Digital Extremism Detection Curated Dataset](https://www.kaggle.com/datasets/adityasureshgithub/digital-extremism-detection-curated-dataset/)

---

## A. Quantitative Analysis

### Dataset Overview

The dataset contains **2,777 social media posts** with two columns: `Original_Message` (text) and `Extremism_Label` (binary classification). The class distribution is moderately balanced with 1,454 NON_EXTREMIST posts (52.4%) and 1,323 EXTREMIST posts (47.6%), giving a class ratio of 0.91. No duplicate rows were found. One missing value existed in `Original_Message`, which was imputed using an empty string — this was chosen over dropping (to preserve the labeled row) and over placeholder text (to avoid introducing fake tokens into the TF-IDF vocabulary).

### Text Characteristics

Three columns were added: `text_length`, `word_count`, and `avg_word_length`. EXTREMIST posts are 12.8% longer in character count and 10.5% longer in word count on average compared to NON_EXTREMIST posts. Average word length is nearly identical between classes (~5.4 characters). The high overlap in distributions across both classes indicates that text length features alone are not strong discriminators for this task.

### Linguistic Analysis

The Top 20 most frequent words (after removing NLTK English stopwords) reveal a critical distinction between classes:

**EXTREMIST** — The two most prominent words are **"kill"** (425 occurrences) and **"attack"** (154). The class is dominated by violence-oriented language including "murder", "genocide", "shoot", and "die", reflecting direct calls to violence and dehumanization of targeted groups.

**NON_EXTREMIST** — The two most prominent words are **"bitch"** (730 occurrences) and **"fucking"** (229). This class is dominated by profanity and insults used as emotional expression rather than calls to violence.

**Key Insight:** Both classes contain highly offensive language, but the distinction lies in intent. Offensive language does not equal extremist content — EXTREMIST posts call for violence or dehumanize specific groups, while NON_EXTREMIST posts use profanity without advocating harm. This overlap is precisely why simple keyword filtering would fail and why this classification task is challenging.

---

## B. Qualitative Analysis

### Annotation Process

30 examples were randomly sampled (15 per class, `random_state=42`) and independently annotated by all 5 team members. The `original_label` column was hidden during annotation. Each annotator classified posts as EXTREMIST or NON_EXTREMIST without discussing examples, viewing others' annotations, or using LLMs.

### Inter-Rater Reliability (IRR)

**Overall Krippendorff's Alpha: 0.214** — well below the 0.667 threshold for tentative conclusions, indicating low inter-rater agreement. This reflects the inherent subjectivity of extremism detection.

Key findings from pairwise analysis:

- **Strongest agreement:** Annotator 3 vs Annotator 4 (α = 0.596, 83.3% agreement) — these two shared the most consistent interpretation of extremism.
- **Weakest agreement:** Annotator 1 vs Annotator 2 (α = −0.062, 63.3%) — negative alpha indicates agreement worse than random chance, suggesting fundamentally different annotation criteria.
- **Annotator 2 as outlier:** Annotator 2 labeled nearly all examples (28/30) as EXTREMIST, resulting in negative alpha against original labels (α = −0.372). This suggests a very broad definition of extremism that included any offensive or vulgar language, rather than restricting it to content calling for violence or dehumanizing specific groups.
- **Low agreement with original labels across the board:** Only Annotator 5 (63.3%) and Annotator 1 (60.0%) exceeded chance agreement with the original dataset labels. Annotators 2 (43.3%), 3 (43.3%), and 4 (46.7%) disagreed with the original labels more often than they agreed.

### Disagreement Analysis

**Inter-human disagreement rate: 53.3%** (16 out of 30 examples had at least one annotator disagree).

**Majority vote vs original labels:** The annotator majority agreed with the original labels only **46.7% of the time** — meaning our team disagreed with the original annotations more often than it agreed.

Notably, several posts originally labeled NON_EXTREMIST were **unanimously** labeled EXTREMIST by all 5 annotators. For example, posts containing dehumanizing language targeting racial groups (e.g., calling non-whites "scum of the world" and "senseless disgusting people") were labeled NON_EXTREMIST in the original dataset. This strongly suggests the original dataset contains labeling errors, particularly for posts that dehumanize groups without making explicit calls to violence.

**What this tells us about annotation quality:**

1. Extremism detection is inherently subjective — reasonable people disagree on where the line falls between offensive and extremist content.
2. The original dataset labels should not be treated as absolute ground truth — they represent one annotator's judgment with a potentially narrow definition of extremism.
3. Some of our ML model's "errors" (Part D) may actually be cases where the model learned the flawed original labels rather than the true concept of extremism.
4. Effective annotation requires clear guidelines with concrete examples, multiple annotators per example, and regular calibration sessions.

---

## C. ML Baseline Model

### Setup

The cleaned dataset (2,777 posts) was split into training (1,943 / 70%), validation (417 / 15%), and test (417 / 15%) sets using stratified splitting (`random_state=42`) to maintain the ~52/48% class balance across all sets.

Text was converted to numerical features using `TfidfVectorizer` with default parameters (unigrams only), producing a vocabulary of 6,067 unique terms. The vectorizer was fit on training data only to prevent data leakage.

### Baseline Results

A `LogisticRegression` model with default parameters achieved:

| Metric | Training | Validation |
|--------|----------|------------|
| Accuracy | 92.6% | 80.8% |

The ~12% gap between training and validation accuracy suggests some overfitting — the model memorized training patterns that don't fully generalize. This gap indicates room for improvement through regularization or feature engineering.

Validation set performance by class:

| Class | Precision | Recall | F1-Score |
|-------|-----------|--------|----------|
| EXTREMIST | 0.80 | 0.79 | 0.80 |
| NON_EXTREMIST | 0.81 | 0.82 | 0.82 |

The model performs roughly equally on both classes, with 39 False Positives (non-extremist flagged as extremist) and 41 False Negatives (extremist content missed).

---

## D. Error Analysis

### Systematic Review

Out of 417 validation examples, the model misclassified **80 posts** (19.2% error rate):

- **False Positives: 39** — Model predicted EXTREMIST but the post was NON_EXTREMIST (over-sensitive)
- **False Negatives: 41** — Model predicted NON_EXTREMIST but the post was EXTREMIST (missed harmful content)

In content moderation, False Negatives are typically more critical — missed extremist content can cause real harm.

### Error Patterns (from manual review of 10 examples)

**Pattern 1 — Profanity without violence (False Positives):** Posts containing heavy profanity and insults but no calls to violence. The model over-relies on offensive keywords like "bitch", "fuck", and "scum" which appear frequently in both classes.

**Pattern 2 — Implicit threats (False Negatives):** Posts using indirect or coded language without explicit violence words like "kill" or "attack". TF-IDF captures surface word frequency but not semantic meaning or context, so subtle extremism is missed.

**Pattern 3 — Short ambiguous texts (both FP and FN):** Very short posts (under 10 words) produce sparse TF-IDF vectors with few features, making predictions unreliable.

**Pattern 4 — Political anger vs extremism (False Positives):** Posts expressing strong political frustration without targeting specific groups or calling for violence. The model confuses emotional intensity with extremism.

### Why the Model Fails

The baseline TF-IDF + Logistic Regression model is limited by:

- **Bag-of-words assumption:** TF-IDF treats each word independently, losing word order and context. "Kill all Muslims" and "Don't kill Muslims" would have similar TF-IDF representations.
- **No semantic understanding:** Cannot distinguish between using a word as a threat vs. quoting, condemning, or discussing extremism.
- **Shared vocabulary problem:** Both classes contain overlapping offensive language, making keyword-based features insufficient.

### Proposed Improvements and Results

**Improvement 1 — Bigrams in TF-IDF (`ngram_range=(1,2)`):**
Captures two-word phrases like "burn ground" or "kill all" which are more indicative of threats than individual words. This expands the feature space to capture some contextual patterns.

**Improvement 2 — Balanced class weights (`class_weight='balanced'`):**
Adjusts the loss function to give more weight to the minority class (EXTREMIST), potentially reducing False Negatives.

**Improvement 3 — Combined (Bigrams + Balanced weights):**

| Model | Val Accuracy | FP | FN |
|-------|-------------|-----|-----|
| Baseline (unigram + default LR) | 80.8% | 39 | 41 |
| Bigrams + Balanced weights | 81.5% | 34 | 43 |

The combined model achieved a modest improvement in overall accuracy (80.8% → 81.5%). False Positives dropped from 39 to 34, meaning the model improved at recognizing that offensive language alone does not constitute extremism — bigrams help because phrases like "fucking stupid" (insult) differ from "kill all" (threat). However, False Negatives increased slightly from 41 to 43, indicating a trade-off: the model became more conservative about labeling posts as EXTREMIST. The right balance between these error types depends on the platform's priorities — minimizing censorship of legitimate speech vs. minimizing missed harmful content.



---


