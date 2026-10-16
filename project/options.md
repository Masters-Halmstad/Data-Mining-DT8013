# Project Topic Options — Analysis of Bike-Sharing Data

Dataset: [UCI Bike Sharing Dataset](https://archive.ics.uci.edu/dataset/275/bike+sharing+dataset) — 2 years (2011–2012) of Capital Bikeshare (Washington D.C.) rentals.

- `day.csv` — 731 daily records
- `hour.csv` — 17,379 hourly records
- Features: date, season, year, month, hour, holiday, weekday, workingday, weather situation (1–4), temp, atemp (feels-like), humidity, windspeed
- Targets: `casual`, `registered`, `cnt` (= casual + registered)

Course: **DT8013 Data Mining, Halmstad University.** The syllabus explicitly covers: **anomaly detection, association rule learning, clustering, classification, regression**, plus the full workflow (data acquisition, cleaning, representation change, evaluation). The criteria say the topic *must come from a lecture covered in the course* unless strongly justified — so every option below is anchored to a syllabus topic first, with modern methods positioned as justified extensions.

Regardless of the topic chosen, the mandatory tasks (preprocessing + unsupervised anomaly detection, descriptive statistics, dimension reduction + visualization, a research question, feature engineering) must all appear. The options differ in **which research question drives the story**.

---

## Option 1 — Anomaly Detection: "When the City Breaks Its Routine" ⭐ (recommended)

**Proposed title:** *Detecting and Explaining Anomalous Days in Washington D.C. Bike Sharing Using Unsupervised Methods and External Evidence*

**Research question:** *Can unsupervised anomaly detection on rental patterns identify real-world city events (storms, hurricanes, holidays, political events) — and can each detected anomaly be explained by external evidence, separating "weather-explainable" from "event-driven" anomalies?*

**Why this one:**
- **It IS the mandatory Task 1.** The criteria literally require "find anomalies via different unsupervised methods, and check if you can justify the anomalies using external information." Making this the *core topic* means the mandatory task and the research question reinforce each other instead of competing for your 3-page limit.
- **The dataset was built for it.** The dataset's own README lists "Event and Anomaly Detection" as an associated task and even gives a worked example: 2012-10-30 = **Hurricane Sandy**. The original paper (Fanaee-T & Gama 2013) is about event labeling — you can benchmark your detections against their labeled events.
- **Strong business framing** (the criteria weight "gain for the bike rental company" heavily): distinguishing *predictable* demand drops (weather) from *unpredictable* ones (events) tells the company when rebalancing/pricing algorithms should be overridden, and quantifies revenue lost to disruptions.
- **Directly on the syllabus** (anomaly detection is a named course topic), so lecture citations are easy.

**Methods (all justifiable from course lectures + Tan et al. Ch. on anomaly detection):**
1. Statistical baselines: z-score / IQR on residuals from a seasonal baseline (raw counts are trend/season-dominated — detect on *residuals*, a key methodological point to justify).
2. Density-based: Local Outlier Factor (LOF).
3. Isolation Forest (ensemble, handles mixed features).
4. Clustering-based: k-means / DBSCAN distance-to-centroid as anomaly score (ties into clustering lectures).
5. Compare the detectors: overlap of top-k anomalies, then manually verify each against news archives (Google "2012-10-30 washington dc") — this verification step is exactly what the criteria ask for.

**Feature engineering:** residual-from-expected demand, rolling 7-day mean/std, casual/registered ratio (event days shift this ratio dramatically), lag features, weather-change deltas (today vs yesterday).

**Evaluation metric:** precision@k against the published event labels from Fanaee-T & Gama; agreement between detectors (Jaccard); qualitative external validation table. (Using the *right* metric is an explicit grading criterion — for unlabeled anomaly detection, precision@k with external validation is the defensible choice.)

**Risk:** low. **Wow-factor:** high (real hurricanes, DC events). **Effort:** medium.

---

## Option 2 — Regression / Forecasting: "What Will Tomorrow's Demand Be?"

**Proposed title:** *Predicting Hourly Bike Rental Demand: From Linear Models to Ensembles — What Actually Drives Demand?*

**Research question:** *How accurately can hourly demand be predicted from weather and calendar features alone, which features carry the predictive power, and where (which hours/conditions) do models systematically fail?*

**Why:**
- Regression is a named syllabus topic; this is the dataset's canonical task, so literature and lecture support are abundant.
- Clear business value: demand forecasts drive bike rebalancing, staffing, and maintenance scheduling.
- Natural model ladder that shows methodological maturity: linear regression → decision tree → random forest / gradient boosting (XGBoost), with proper **time-based train/test split** (train on 2011, test on 2012 — justifying *why* random splits leak future information is exactly the "why and how" the graders want).

**Why be careful:** it's the **most common choice** — hundreds of Kaggle notebooks exist. Creativity is a heavily weighted criterion, so if you pick this, differentiate: (a) model `casual` and `registered` **separately** and show they behave like two different populations (commuters vs tourists), or (b) focus the story on **error analysis** — when does the model fail, and what does that teach the company?

**Feature engineering:** cyclical encoding of hour/month (sin/cos — easy to justify: hour 23 and hour 0 are neighbors), lag features (demand t-1, t-24, t-168), rolling weather aggregates, "comfort index" combining temp/humidity/wind, rush-hour × workingday interaction, holiday-adjacent days.

**Evaluation metric:** RMSE + MAE on chronological hold-out; RMSLE if you emphasize relative error; residual plots per hour-of-day. Compare against a naive seasonal baseline (same hour last week) — beating the baseline is the honest test.

**Risk:** low. **Wow-factor:** medium (unless the casual/registered split angle is played well). **Effort:** medium.

---

## Option 3 — Clustering: "The Rhythms of the City"

**Proposed title:** *Discovering Typical Days: Clustering Daily Demand Profiles to Segment Operating Regimes*

**Research question:** *Do days naturally group into distinct demand-profile archetypes (e.g., commuter weekday, leisure weekend, bad-weather day) — and do these data-driven archetypes match, or contradict, the calendar labels the company would naively use?*

**Why:**
- Clustering is a named syllabus topic, and you already have lab experience with it (Lab 1), so lecture citations and tooling are ready.
- Genuinely creative angle: represent **each day as a 24-dimensional vector** (hourly counts from `hour.csv`), cluster the *shapes* of days. The interesting finding is where clusters disagree with the calendar — e.g., a Friday that behaves like a Saturday, holidays that behave like workdays.
- Dimension reduction (mandatory Task 3) becomes integral, not decorative: PCA / t-SNE on the 24-d day-vectors to visualize the clusters.
- Business value: the company doesn't need 365 rebalancing plans — it needs one plan per archetype.

**Methods:** k-means (elbow + silhouette for k), hierarchical clustering (dendrogram shows regime structure), DBSCAN (its "noise" points double as anomalies → ties to mandatory Task 1). Normalize profiles (shape vs volume — a decision to justify explicitly).

**Feature engineering:** the day-as-vector representation itself *is* the engineered feature (course learning outcome: "change data representation according to the needs of the problem" — quote this), plus profile-shape features: peak hour, AM/PM peak ratio, peakedness.

**Evaluation metric:** silhouette score / Davies-Bouldin for internal validity; **adjusted Rand index** against calendar labels (workingday, season) for external validity — a metric choice that's easy to defend.

**Risk:** low-medium. **Wow-factor:** high if the calendar-mismatch story lands. **Effort:** medium.

---

## Option 4 — Classification: "Casual vs Registered — Two Cities, One System"

**Proposed title:** *Who Is Riding? Characterizing and Predicting the Casual-vs-Registered Demand Mix*

**Research question:** *Under what conditions does casual (tourist/leisure) ridership dominate over registered (commuter) ridership, and can the company predict "conversion-opportunity" days — high casual traffic days when membership marketing pays off most?*

**Why:**
- Classification is a named syllabus topic. Frame it as predicting a class you engineer: e.g., "casual-dominated day" (casual share above a threshold) or discretized demand levels (low/medium/high — discretization is classic Witten et al. material).
- The `casual`/`registered` split is the most under-used part of this dataset — most public analyses ignore it — so the creativity score is naturally high.
- Direct revenue framing: casual riders pay more per trip but convert to memberships; knowing *when* they show up tells marketing when to act.

**Methods:** decision trees (interpretable rules the company can read — justify choice via interpretability), random forest, k-NN, naive Bayes comparison; class-imbalance handling if the engineered class is skewed.

**Feature engineering:** casual share `casual/cnt`, weekend × weather interactions, tourist-season flags, discretized comfort bands.

**Evaluation metric:** F1 / balanced accuracy + confusion matrix (justify over plain accuracy due to imbalance); ROC-AUC for the ranking view.

**Risk:** medium (the target is engineered, so the framing must be airtight). **Wow-factor:** high. **Effort:** medium.

---

## Option 5 — Association Rule Mining: "Which Conditions Travel Together?"

**Proposed title:** *Association Rules Between Weather, Calendar, and Demand Regimes in Bike Sharing*

**Research question:** *Which combinations of conditions (weather situation, season, day-type, temperature band) reliably imply high or collapsed demand — expressed as human-readable rules the operations team can act on?*

**Why:**
- Association rule learning is a named syllabus topic that **almost nobody applies to this dataset** — instant differentiation.
- Requires discretizing continuous features into items (temp bands, demand tertiles) — a representation-change story the syllabus explicitly values.
- Output is inherently presentable: rules like `{weathersit=3, workingday=1} ⇒ {demand=low}` with support/confidence/lift fit beautifully on slides.

**Why be careful:** on 12 columns the rules risk being obvious ("cold ⇒ low demand"). Mitigate by mining on *engineered* items (lags, deltas: "temperature jumped ≥5° vs yesterday") and by reporting only high-**lift** rules. Works best as a **companion technique** inside Option 1 or 3 rather than standing alone.

**Methods:** Apriori / FP-Growth; prune with lift and conviction.

**Evaluation metric:** support, confidence, **lift** (justify lift over confidence — confidence is inflated by frequent consequents; this is a textbook Tan et al. point graders like).

**Risk:** medium (obviousness). **Wow-factor:** medium alone, high as a supplement. **Effort:** low-medium.

---

## Modern / Advanced Extensions

The criteria allow topics beyond the lectures **with strong justification and reputable citations**. Safest strategy: pick a syllabus-anchored core (Options 1–4) and add ONE of these as a clearly-labeled extension. This scores creativity points without risking the "topic not from the course" objection or the lecture-citation requirement.

### Option 6 — Transformer / Deep Time-Series Forecasting (extension of Option 2)

*Compare classical models against a modern architecture: LSTM/GRU, or a transformer (PatchTST / Temporal Fusion Transformer), or a **zero-shot foundation model** (Amazon Chronos, Google TimesFM — pretrained time-series transformers you just call, no training).*

- **Research angle:** "Does a pretrained time-series foundation model beat feature-engineered gradient boosting on 2 years of hourly data?" The honest, defensible finding is usually **no** — on small tabular datasets, well-engineered XGBoost typically wins, and *demonstrating that with a fair experiment* is a mature, examiner-pleasing conclusion.
- **Justification to write:** regression is the course anchor (cite the regression lecture); the transformer is the external extension (cite the PatchTST/TFT/Chronos papers — reputable, peer-reviewed).
- **Risk:** medium — deep models eat time, and 17k hourly points is small for transformers. Keep it to one notebook section, one comparison table.

### Option 7 — LLM-in-the-Loop Anomaly Explanation (extension of Option 1) ⭐ clever fit

*After detecting anomalous days statistically, feed each anomaly's date + context to an LLM ("What happened in Washington D.C. on 2012-10-30?") to retrieve candidate explanations, then verify against news sources.*

- **Why it's clever:** Deliverable Supplementary A **already requires screenshots of your LLM prompts/outputs**. This turns a compliance chore into a *methodological contribution*: a semi-automated pipeline (unsupervised detection → LLM hypothesis generation → human verification).
- Also usable for LLM-assisted feature-engineering brainstorming ("suggest features a bike company would care about") — documented in Supplementary A either way.
- **Justification:** the core method (anomaly detection) stays on-syllabus; the LLM is a documented *tool*, not the topic — minimal approval risk, maximal novelty.
- **Caveat to state in the report:** LLM outputs can hallucinate → every explanation is verified against a primary source. Saying this out loud earns rigor points.

### Option 8 — TabPFN / Modern AutoML Benchmark (extension of Option 2 or 4)

*Benchmark TabPFN (a pretrained transformer for small tabular problems, Nature 2025) against classical models on the daily dataset.*

- Fits the daily data well (731 rows is exactly TabPFN's regime, ≤10k samples); one `pip install`, near-zero training time.
- Strong "modern methods" citation (Hollmann et al., Nature 2025) — reputable by any standard.
- **Risk:** low effort but also a smaller story; best as a one-paragraph comparison row, not a headline.

### Option 9 — Embedding + Explainability Layer (cross-cutting garnish)

*Add UMAP alongside PCA/t-SNE for the visualization task, and SHAP values on the best regression/classification model for feature-importance storytelling.*

- Not a topic — a quality upgrade to whichever option you pick. SHAP plots ("temperature contributes +X rentals until 30 °C, then negative") are presentation gold and directly answer "which engineered features mattered," feeding the feature-engineering grading criterion.
- Cite: Lundberg & Lee (NeurIPS 2017) for SHAP; McInnes et al. for UMAP; anchor PCA to the course's dimension-reduction lecture.

---

## Comparison Matrix

| # | Option | Syllabus anchor | Creativity | Business value story | Risk | Effort |
|---|--------|----------------|------------|---------------------|------|--------|
| 1 | Anomaly detection + external evidence | ✅ named topic + mandatory task | High | Override forecasts on event days; quantify disruption losses | Low | Medium |
| 2 | Demand regression ladder | ✅ named topic | Low–Med (crowded) | Rebalancing & staffing forecasts | Low | Medium |
| 3 | Clustering day-profiles | ✅ named topic | High | One ops plan per day-archetype | Low–Med | Medium |
| 4 | Casual vs registered classification | ✅ named topic | High | Target membership marketing | Medium | Medium |
| 5 | Association rules | ✅ named topic | Medium | Readable ops rules | Medium | Low–Med |
| 6 | Transformers / foundation models | ⚠️ extension | High | Model-selection guidance | Medium | High |
| 7 | LLM anomaly explanation | ⚠️ extension (core stays on-syllabus) | Very high | Automated incident triage | Low | Low–Med |
| 8 | TabPFN benchmark | ⚠️ extension | Medium | Cheap strong baseline | Low | Low |
| 9 | UMAP + SHAP garnish | ⚠️ extension | Medium | Explainable drivers | Low | Low |

## Recommended Combination

**Core: Option 1 (anomaly detection) + Option 7 (LLM explanation loop), garnished with Option 9 (SHAP/UMAP), with Option 3's day-profile clustering as the anomaly-scoring representation.**

Reasoning against the grading criteria:
1. **Data cleaning & preparation** — anomaly detection *is* the required preprocessing task; no wasted pages.
2. **Relevance/creativity of the question** — "which anomalies are weather vs events, and can an LLM explain them automatically?" is creative and squarely about company gain (when to trust vs override demand models).
3. **Quality/justification** — every core method (LOF, Isolation Forest, DBSCAN, PCA, residual z-scores) maps to a named lecture topic → easy slide-number citations, which the criteria say are pass/fail.
4. **Feature engineering creativity** — residuals, day-shape vectors, casual/registered ratio, weather deltas are all non-obvious and defensible.
5. **Right evaluation metric** — precision@k vs the published event labels of the dataset's own authors (Fanaee-T & Gama 2013) is an unusually rigorous validation for an unsupervised task.
6. **Deliverables synergy** — the mandatory LLM-screenshot supplement becomes part of the method instead of an afterthought.

**Fallback if the examiner wants a "safer" topic at approval:** Option 2 (regression) with the casual/registered split angle + Option 8 (TabPFN) as the modern comparison.
