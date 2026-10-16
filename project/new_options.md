# New Topic Options — Group 5 (Binay Kumar Sah & Kabir Tamari)

`options.md` was written before the other groups registered. Its top pick (anomaly detection) is now taken by five groups, so this file starts again from what is still open. **The topic deadline is 1 Oct.**

---

## 1. What's already taken

| # | Group | Their topic | Lane |
|---|-------|-------------|------|
| 1 | Sterner & Houssari | Model normal demand, flag days that deviate | Anomaly detection |
| 2 | Leite & Bavafa | Forecast casual + registered separately vs total directly | Regression / forecasting |
| 3 | Haider & Sayeed | Contextual anomalies: demand weather doesn't explain | Anomaly detection |
| 4 | Haraldsson & Simon | Cluster weekday profiles month by month, year-over-year growth | Clustering |
| 6 | Bauer & Vu | PCA + Isolation Forest, weather vs rentals, look up real events | Anomaly detection |
| 7 | Jensen & Johansson | Detect extreme-weather days, then their effect on demand | Anomaly detection |
| 8 | Hasan & Hussain | Same daily total, different hourly shapes | Clustering |
| 9 | Anand & Kumar | Peak-hour concentration + PCA + clustering of daily patterns | Clustering |
| 10 | Mishra | DBSCAN + hierarchical clustering of rider types | Clustering |
| 11 | Bleckert & Knutsson | Season and holiday anomalies, explained by known factors | Anomaly detection |

**Saturated:** anomaly detection as the research question (5 groups) and clustering of daily or hourly demand shapes (4 groups).
**Still open:** **classification**, **association rules**, **effects that carry over in time** (what happened in the last hours or yesterday) and **anything modern** (pretrained models, LLMs, prediction intervals).

> Every group still has to do anomaly detection, descriptive statistics and PCA as *required tasks*. Those won't make us stand out. The **research question** is the only place we can be different.

Each option below has the same parts: the idea in plain words, the research question, the steps, which lecture it comes from, the metric, the gain for the bike company, overlap with other groups, and the risk, effort and extra installs.

---

## 2. Classic options (course topics nobody has used)

### A. "The Weather Hangover" ⭐ safest unique pick

**Idea:** Everyone checks how weather affects demand *at the same time*. We check what happens **after the rain stops**. Do people stay off bikes for hours, and do commuters come back faster than casual riders?

**Research question:** *How long does bike demand stay down after bad weather ends, and do registered (commuter) and casual riders recover at different speeds?*

**We already have evidence** (quick check on `hour.csv`, dry daytime hours 7:00–21:00, demand as a fraction of the normal level for that hour, month and day type):

| Time since rain stopped | All riders | Casual | Registered | n hours |
|---|---|---|---|---|
| 1 h | **0.71** | **0.60** | 0.75 | 303 |
| 2–3 h | 0.87 | 0.73 | 0.90 | 463 |
| 4–6 h | 0.95 | 0.87 | 0.95 | 522 |
| 7–12 h | 1.00 | 0.98 | 1.00 | 861 |
| > 24 h | 1.01 | 1.03 | 1.01 | 6,265 |

The same pattern shows up in `day.csv`: a clear day after a rainy day reaches only 81% of normal demand, against 99.6% after a clear day. That's a real signal, and nobody else is looking at it.

**Steps:**
1. Engineer lag features: `hours_since_rain`, `rain_in_last_3h`, `rain_yesterday`, temperature change since yesterday. This also covers the required feature-engineering task.
2. **Association rules** (Apriori / FP-Growth) on discretized items, e.g. `{rain_last_1h, workingday, AM_peak} ⇒ {demand_low}`. Rank by **lift**, not confidence, because confidence looks high whenever the outcome is common anyway.
3. **Regression** with and without the lag features, to measure how much "weather memory" improves the forecast.
4. Fit recovery curves separately for casual and registered riders.

**Course anchor:** association rule lecture, plus the regression lecture. **Metric:** support, confidence, lift for the rules; reduction in MAE when lag features are added.
**Company gain:** when to rebalance bikes after a storm, when to run "come back" promotions, and a warning that demand forecasts are too optimistic right after rain.
**Overlap:** small. Group 7 looks at extreme weather *on the day*; we look at the hours *after*.
**Risk:** low. **Effort:** medium. **Extra install:** `mlxtend` (Apriori).

---

### B. "Peak-Overload Early Warning"

**Idea:** The company doesn't need an exact count every hour. It needs to know **which hours will be overloaded**, so it can move bikes there beforehand.

**Research question:** *Can we predict a day ahead which hours will hit the top 10% of demand (above ≈451 rentals/hour)?*

**Steps:** label hours above the 90th percentile as "overload" → features: hour, day type, weather (actual weather standing in for a perfect forecast; we say this assumption out loud), same hour yesterday and last week → decision tree (rules people can read) vs random forest vs k-NN → **time-based split** (train on 2011, test on 2012).

**Course anchor:** classification lecture. **Metric:** recall, PR-AUC, and a **cost-weighted confusion matrix**, because a missed overload (empty stations, lost rides) costs more than a false alarm. "Use the right evaluation metric" is an explicit grading criterion, and this option gives a lot to say about it.
**Company gain:** a direct staffing and rebalancing tool.
**Overlap:** small. Group 2 forecasts counts; we classify overload risk.
**Risk:** low. **Effort:** medium. **Extra install:** none.

---

### C. "Conversion Days"

**Idea:** Casual riders pay per trip and could be converted to members. Which days are dominated by them?

**Research question:** *Which conditions produce casual-dominated days (casual share > 30% — 133 of 731 days), and can we predict them so marketing knows when to push memberships?*

**Steps:** engineer `casual_share` → binary label → decision tree, random forest, naive Bayes → handle class imbalance (only 18% positive).
**Course anchor:** classification lecture. **Metric:** F1 / balanced accuracy, plus ROC-AUC. Plain accuracy is misleading at 18% positives.
**Company gain:** marketing timing, revenue per ride.
**Overlap:** **medium.** Groups 2 and 10 also split casual vs registered, though with different methods.
**Risk:** medium. **Effort:** medium. **Extra install:** none.

---

## 3. Modern options (course topic as the base, modern method as the extension)

The rules require the topic to come from a lecture "unless you have a strong justification". The safe way to go modern: **the research question sits on a course topic** (regression or classification), and the modern method is a **clearly labelled comparison or tool**, cited from a peer-reviewed source.

### D. "Do You Even Need Your Own Model?" (pretrained time-series transformers)

**Idea:** Time-series **foundation models** are transformers pretrained on millions of other time series. They forecast new data **zero-shot**, with no training at all. Can one that has never seen D.C. bikes match a model trained on our two years of data?

**Research question:** *Can a pretrained time-series transformer forecast bike demand as well as a gradient-boosting model with engineered features — and where does each one fail?*

**Steps:**
1. Baselines: seasonal naive (same hour last week) and linear regression.
2. Trained model: gradient boosting with engineered features (lags, cyclic hour/month, weather).
3. Zero-shot models: **Chronos** (Ansari et al., TMLR 2024) and **TimesFM** (Das et al., ICML 2024) on the raw `cnt` series; **TabPFN** (Hollmann et al., *Nature* 2025) on `day.csv`, which is within its small-data range.
4. Rolling-origin backtest on 2012, then compare errors split by weather, holidays and peak hours.

**Course anchor:** regression lecture; the transformers are the cited extension. **Metric:** MAE, RMSE, **MASE** (error relative to the naive baseline, so it can be compared across models).
**Company gain:** can a small bike operator skip building its own model? The honest expected result is that trained boosting with good features still wins, and showing that with a fair test is a mature conclusion.
**Overlap:** small. Group 2 forecasts too, but their question is about forecasting the parts vs the total.
**Risk:** medium. **Effort:** high. **Extra install:** `torch`, `chronos-forecasting`, `timesfm`, `tabpfn`. ⚠️ Our venv is Python 3.14, so check early that torch installs. If it doesn't, use a Python 3.12 env or Google Colab (free GPU).

---

### E. "LLM-Built Event Calendar" ⭐ best modern pick

**Idea:** Weather and the calendar don't explain everything. City **events** do: festivals, marathons, storms, protests. The dataset has no events column. We ask an **LLM to build one**, then **check every event** against a real source.

**Research question:** *Can an LLM-generated event calendar explain bike demand that weather and calendar features cannot — and how reliable is the LLM as a data source?*

**Steps:**
1. Prompt an LLM: "List events in Washington D.C. 2011–2012 that could affect cycling", by type (festival, sports, storm, protest, holiday weekend). Real examples it should find: Cherry Blossom Festival, Marine Corps Marathon, July 4 on the Mall, Virginia earthquake (23 Aug 2011), Hurricane Irene (Aug 2011), June 2012 derecho, Hurricane Sandy (Oct 2012).
2. **Verify each event** against a news or official source. Record the LLM's **precision** (how many events were real and on the right date). That measure of how often it makes things up is a result in itself.
3. Turn the verified events into features: `event_type`, `days_to_event`, `multi_day_event`.
4. Regression or classification with and without event features. Measure the gain, split by event type and by casual vs registered riders.

**Course anchor:** regression lecture plus the feature-engineering part of the course. The LLM is a documented *tool*, not the topic, which keeps the approval risk low.
**Metric:** LLM precision on dates; improvement in R²/MAE from event features; event effect size by type.
**Company gain:** a cheap, repeatable way to build an event calendar for any city. It also shows which event types change demand and for which riders (festivals → casual, storms → everyone).
**Why it scores well:** the required **LLM-screenshot supplement** becomes part of the method instead of a chore. It uses the required "external information" step properly. The grading criteria for feature engineering literally mention "holidays, promotions", and events are the same idea.
**Overlap:** **medium.** Groups 6 and 11 look events up **after** detecting anomalies, to explain them. We use events as **inputs** and **measure** their effect. Say this difference clearly in the proposal.
**Risk:** low–medium. **Effort:** medium. **Extra install:** none (any chatbot works; screenshots go into Supplementary A).

---

### F. "Ask the Chatbot"

**Idea:** Describe a day in words ("Saturday, July 2012, 30 °C, clear, humid") and ask an LLM how many bikes will be rented. Compare with real models.

**Research question:** *Can an LLM predict bike demand from a text description as well as a regression model — or has it simply memorized this public dataset?*

**Steps:** turn rows into text → prompt the LLM (approach from LLMTime, Gruver et al., NeurIPS 2023) → compare with regression → **memorization test:** ask for exact counts on specific dates with no other information. If it is suspiciously accurate, it saw the UCI dataset during training.
**Course anchor:** regression lecture, weakly. **Metric:** MAE vs regression; accuracy on the memorization test.
**Company gain:** weak. This reads more like an ML research question than help for a bike company. That counts against it, since the company gain is weighted heavily in grading.
**Overlap:** none. **Risk:** medium–high. **Effort:** medium. **Extra:** LLM API access for hundreds of prompts, or a local model via Ollama.

---

### G. "How Many Bikes to Be 95% Sure?" (conformal prediction)

**Idea:** A forecast like "300 rentals" is only a single number. Operations needs a **range they can trust**: "between 220 and 410, with 95% certainty". **Conformal prediction** is a modern statistical method that gives intervals with a **guaranteed** coverage.

**Research question:** *Can we give the company demand ranges with a guaranteed 90–95% coverage — and on which days (rain, holidays, peaks) do the ranges have to be widest?*

**Steps:** quantile gradient boosting → **conformalized quantile regression** (Romano, Patterson & Candès, NeurIPS 2019; Python library `MAPIE`) → check coverage on 2012 → split coverage and width by conditions.
**Course anchor:** regression lecture plus the evaluation lecture. **Metric:** actual coverage vs target (does 95% really cover 95%?), mean interval width, coverage split by weather.
**Company gain:** stock bikes for the 95th percentile instead of the average, which means fewer empty stations.
**Overlap:** small. Group 2 gives single-number forecasts. **Risk:** low–medium. **Effort:** medium. **Extra install:** `mapie`.
Can be combined with **D** (intervals around the pretrained models' forecasts).

---

## 4. Wild cards

- **H. Cross-city transfer:** does a model trained on D.C. work on the **UCI Seoul Bike Sharing** dataset (hourly, with weather)? Very creative, but it brings in a second dataset, and examiners may say the project is no longer about our dataset. Risky.
- **I. Weather-adjusted growth:** rentals grew **+64.9%** from 2011 to 2012. But 2012 also had better weather (6 bad-weather days vs 15, slightly warmer). How much growth was real adoption, and how much was weather? Good question for management. **Medium overlap with group 4** (year-over-year growth).

---

## 5. Comparison matrix

| | Option | Course anchor | Overlap | Creativity | Company gain | Risk | Effort | Extra |
|---|---|---|---|---|---|---|---|---|
| A | Weather hangover | ✅ Assoc. rules + regression | Small (7) | High | High | Low | Med | `mlxtend` |
| B | Peak-overload warning | ✅ Classification | Small (2) | Med | High | Low | Med | — |
| C | Conversion days | ✅ Classification | Med (2, 10) | Med | Med–High | Med | Med | — |
| D | Pretrained transformers vs local model | ✅ Regression + ext. | Small (2) | High | Med | Med | High | torch, Py 3.12/Colab |
| E | LLM event calendar | ✅ Regression + ext. | Med (6, 11) | Very high | High | Low–Med | Med | — |
| F | LLM as forecaster | ⚠️ Weak | None | High | Low | Med–High | Med | LLM API |
| G | Conformal intervals | ✅ Regression + eval. | Small (2) | High | High | Low–Med | Med | `mapie` |
| H | Cross-city transfer | ⚠️ Weak | None | High | Med | High | High | 2nd dataset |
| I | Weather-adjusted growth | ✅ Regression | Med (4) | Med | Med | Med | Med | — |

---

## 6. Recommendation — shortlist of 3, ready to register

**If unsure, pick A.** It has the strongest evidence already, no other group uses its method, and it carries little risk. **Pick E if you want the modern / LLM angle.** A and E also combine well: weather memory plus events as two new feature sets in one demand model.

**A — Safest unique**
> **Title:** The Weather Hangover: Does Bad Weather Keep Bike Demand Down After It Ends?
> **Description:** This project measures how long bike rental demand stays lower after rain stops, and whether registered (commuter) and casual riders recover at different speeds. Using association rule mining on time-lagged weather conditions and regression with lag features, we show the company when to rebalance bikes and run promotions after bad weather.

**E — Best modern**
> **Title:** Can an LLM Explain What Weather Can't? Event-Aware Bike Demand Analysis
> **Description:** We use an LLM to build a calendar of Washington D.C. events (2011–2012), verify each event against real sources, and add them as features to a demand model. We measure how much demand variation events explain beyond weather and calendar, which event types affect casual vs registered riders, and how reliable the LLM is as a data source.

**D (+G) — Most technical modern**
> **Title:** Do Bike-Sharing Operators Need Their Own Model? Pretrained Time-Series Transformers vs Engineered Regression
> **Description:** We compare zero-shot time-series foundation models (Chronos, TimesFM) with a gradient-boosting model trained on two years of hourly data, measuring accuracy and how reliable the prediction intervals are, to find out whether a bike-sharing company can forecast demand without building its own model, and where each approach fails.

---

### Reminders that apply to every option
- Every claim in the report needs a course citation like **"Lecture 3 - Slide 87"**. Missing citations can mean an F. Modern methods need the peer-reviewed paper cited instead.
- The required tasks don't change: cleaning, anomalies via several unsupervised methods (+ external justification), descriptive stats, dimension reduction and visualization, feature engineering, a 3-page report and 15 slides.
- Dataset citation: Fanaee-T & Gama (2013), *Event labeling combining ensemble detectors and background knowledge*, Progress in Artificial Intelligence.
