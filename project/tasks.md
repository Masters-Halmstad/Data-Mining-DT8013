# Project tasks: *Does Bike Demand Stay Low After Bad Weather Ends?*

**Group:** Binay Kumar Sah & Kabir Tamari · **Course:** DT8013 Data Mining (Halmstad)
**Notebook:** [bike_weather_recovery.ipynb](bike_weather_recovery.ipynb) (runs top to bottom in a few minutes) · **Figures:** [figures/](figures/)
**Run:** `cd project && ../.venv/bin/jupyter nbconvert --to notebook --execute --inplace bike_weather_recovery.ipynb`

**Research question.** After rain or snow stops, how many hours does it take for bike demand to return to normal, and do commuters (registered) and casual riders recover at different speeds?
**Value for the company.** Rebalancing and promotions are timed to the weather. If demand lags the weather, the company moves bikes too early and misses the hours when a promotion could bring casual riders back.

| Date | What |
|---|---|
| **Mon 12 Oct** | 10-min progress check with the professor (see §1) |
| 20–21 Oct | seminar paper presentations (little project time) |
| **Sun 25 Oct 23:59** | submit `GroupXX.zip` |
| 26 / 27 Oct 13:15 | project presentation, 15 min + 5 min questions (attendance mandatory, otherwise U) |

---

## 0. Status at a glance (11 Oct)

| Step (from criteria.md) | Status | Where |
|---|---|---|
| 1. Cleaning + unsupervised anomalies + external justification | ✅ first version done | notebook §1, §3 |
| 2. Descriptive statistics | ✅ done | §2 |
| 3. Dimension reduction + visualisation | ✅ done (PCA ×2) | §5 |
| 4. Research question | ✅ answered with 3 methods + sensitivity, humidity and day-off checks | §6, §7, §8 |
| 5. Feature engineering | ✅ core features done; 3 extra features planned | §4 |
| 6. Report (≤ 3 pages, Calibri 11) | ⬜ not started | n/a |
| 7. Slides (≤ 15) | ⬜ not started | n/a |
| Citations to lecture slides | 🟡 Anomaly lecture done; other lectures still `[Slide ?]` | §6 below |
| Supplements A (LLM screenshots), B (notebook), C (contribution %) | 🟡 B exists; A and C still to do | §4.8 |

**Headline results so far** (all printed by the notebook):
- Registered riders are back within 10 % of normal **~2 h** after the rain stops. Casual riders take **~7–12 h** (regression with HAC CIs, §7).
- Episode view: median recovery is **3 h vs 11 h**. Casual riders are slower on the same episodes (paired Wilcoxon p = 1.6 × 10⁻⁶; mean gap +3.0 h, 95 % CI 1.8–4.1). This holds in **all 9** threshold / episode-length settings (p ≤ 0.002).
- **37 % of rides lost to bad weather are lost *after* it has stopped** (≈ 32 700 of ≈ 89 500 per year). For casual riders the share is 55 %.
- Association rules give the same picture: `{0–1 h after rain} ⇒ registered = low` (lift 4.2) disappears after 3 h, while `⇒ casual = low` keeps lift > 1 up to 12 h. All 20 top rules mined on 2011 still hold in 2012.

---

## 1. Tomorrow's 10-minute check-in (Mon 12 Oct)

**Goal:** show we are on track and get the professor's view on 3–4 open choices. Talk to figures, not code.

| Min | Say | Show |
|---|---|---|
| 0–1 | RQ + why the company cares (one sentence each) | n/a |
| 1–3 | **Data audit, 3 surprises:** (1) 165 missing hours. Zero-ride hours are not written, and the long gaps are *system shutdowns* we confirmed in the news (Irene closed 18:00, gap starts 18:00; Sandy closed 01:00, gap starts 01:00). (2) The Readme's temperature formula is wrong (÷41 gives no frost in two D.C. winters); the UCI page formula is right. (3) The season labels in the Readme are wrong (1 = winter). | cleaning log table (§1) |
| 3–4 | Anomalies: 4 detectors from the lecture, 15 consensus days, **all 15 explained** (10 storms, 5 calendar days the holiday flag misses) | `fig03_anomalies.png` |
| 4–7 | **First answer:** recovery curves + regression | `fig06_recovery_curves.png`, `fig09_regression_recovery_effect.png` |
| 7–8 | Methods still to finish + metrics (post-rain MAE, lift, chronological split) | §3 table below |
| 8–10 | **Questions** (below) | n/a |

**Questions to ask the professor (decisions we want confirmed):**
1. Is "≥ 2 h of weathersit 3, then ≥ 6 dry hours" an acceptable definition of an *episode*? (It removes 1-hour flags, which are often single noisy readings.)
2. weathersit 3 mixes light rain and light snow. Is a **temperature proxy (< 2 °C) for snow** acceptable, or should we drop the rain/snow split? (Only 22 snow episodes.)
3. We **excluded shutdown days** from the recovery analysis (missing supply, not demand). Does the professor agree?
4. Is association rules + regression enough, or would they like a clustering angle (e.g. k-means on recovery curves)?
5. Gradient boosting does not improve with lag features because it reads recent rain from humidity and mist. Is it OK to report that honestly as a finding?

---

## 2. Milestones

| ID | Date | Done when | Owner (suggested) |
|---|---|---|---|
| M0 | Mon 12 Oct | check-in done; professor's feedback written into §5 Decision log | both |
| M1 | Wed 14 Oct | `bridge_day` + `ended_before_commute` features; notebook re-run with professor's feedback applied | Kabir |
| M2 | Fri 16 Oct | all lecture-slide citations filled; external-evidence table final; revenue translation (if a price source is found) | Binay |
| M3 | Sun 18 Oct | report outline + every figure final (titles, units, readable at slide size) | both |
| n/a | 20–21 Oct | seminar presentations (no project work planned) | n/a |
| M4 | Fri 23 Oct | report draft (3 pages) + slide draft (≤ 15); one full timed rehearsal | both |
| M5 | Sun 25 Oct 23:59 | `GroupXX.zip` submitted (report PDF, slides PDF, Supp. A/B/C) | both |
| M6 | 26/27 Oct | two more timed rehearsals (≤ 14:30 min); presentation | both |

---

## 3. How we cover each grading criterion

| Criterion | What we do | Where |
|---|---|---|
| **Data cleaning & preparation** (clear, compelling) | 9-item cleaning log, each with evidence → decision → why; outage vs zero-ride gaps confirmed by news; Readme errors found (temperature formula, season labels) | §1 |
| **Relevance/creativity of the question** (gain for the company) | the question is about *timing* (rebalancing, promotions), not just "how much"; it separates commuters from leisure riders; it quantifies lost rides after the weather clears | §6–9 |
| **Quality of performed task** (relevant tools, justified why/how) | 3 independent methods agree (episode curves, regression with HAC, association rules); every section opens with a Why/Source cell; sensitivity table (9 settings); robustness models C (humidity) and D (days off); honest negative results (gradient boosting, weekend hint) | §5–8 |
| **Creativity in engineered features** | the "recovery clock" (`h_after`, `after_bin`), weather *episodes*, snow proxy, demand ratio vs same-hour baseline, previous-episode length, bad hours in last 6/24 h; planned: `bridge_day`, `ended_before_commute` | §4 |
| **Right evaluation metric** | MAE/RMSE **on post-rain hours** (an all-hours metric hides the effect), chronological split + TimeSeriesSplit; lift (not confidence) for rules; paired Wilcoxon + bootstrap CI for recovery times; Jaccard agreement for detectors | §3, §6–8 |
| **Clarity of presentation & report** | one storyline: *data → episodes → how long → why → what to do*; consistent colours (registered blue, casual orange) | slides plan §4.7 |
| **Citation to course literature** | slide numbers for every method; textbook chapters (Tan et al. 2018; Witten et al. 2016) | §6 below |
| **Presentation mandatory** | both present; rehearse to ≤ 14:30 | M6 |

---

## 4. Task list with thought process

Legend: ✅ done · ⬜ to do · 🟡 partly done. **Why** = our reasoning; **Source** = what we cite. `[Lecture X – Slide ?]` = slide number still to look up.

### 4.1 Step 1: Cleaning (notebook §1)

- ✅ **Audit the raw files.** There are no NaN cells and no duplicates, cnt = casual + registered on every row, and the daily file matches the hourly sums. But only 17 379 of 17 544 hours are present.
  - *Why:* a recovery clock ("3 h after rain") needs a **complete hourly grid**, otherwise "the next row" is not "the next hour".
- ✅ **Split the 165 missing hours into two causes.**
  - 76 hours in 1–2 h gaps at 02:00–05:00 → **filled with 0 rides**. The smallest count in the file is 1, so zero-ride hours were simply never written.
  - 89 hours in ≥ 6 h gaps → **outage** (NaN + flag, excluded from models). The news confirms that Capital Bikeshare was closed: Irene closed at 18:00 on 27 Aug 2011 and the gap starts at 18:00; Sandy closed at 01:00 on 29 Oct 2012 and the gap starts at 01:00 and lasts 36 h.
  - *Why:* imputing an outage would invent demand that was impossible. No gap is 3–5 h long, so the 6 h threshold is not delicate.
  - *Source:* noise vs meaningful anomaly [Anomaly Detection (1) – Slide 18]; data quality [Tan et al. 2018, Ch. 2 "Data", §2.2 Data quality].
- ✅ **Temperature scale.** The packaged `Readme.txt` says ÷41; the UCI page says (t + 8)/47. The Readme version gives min +0.8 °C over two D.C. winters and 9 °C during the 26-Jan-2011 snowstorm; the UCI version gives −7 °C and 2 °C.
  - *Decision:* use the UCI formula. *Why:* physical plausibility; the snow proxy depends on it.
- ✅ **Sensor errors.**
  - `hum = 0 %` all day on 2011-03-10, a rainy day, so it is impossible. Replaced by the median for the same month and weather type.
  - `atemp` stuck at one value on 2012-08-17 (≈ 0 °C at ≈ 24 °C air). Predicted from temp instead (r = 0.99). We only flag *negative* residuals: 2011-07-21 has a high feels-like temperature, which is a real heat wave.
- ✅ **Season labels.** The codes change on 21 Mar / 21 Jun / 23 Sep / 21 Dec, so code 1 = **winter**, not "springer" as the Readme says. Relabelled.
- ✅ **weathersit 4** appears in only 3 hours, so it is merged into 3. *Why:* a class with 3 rows cannot be estimated.
- ✅ **windspeed = 0** (2 189 h) is kept as real "calm". *Why:* there is no value between 0 and 6 km/h, and U.S. stations report wind < 3 kt (5.6 km/h) as calm. *Source:* AMS Glossary "Calm"; NWS ASOS wind sensor page.
- ⬜ **(Binay) Check the two unconfirmed outages** (18 Jan 2011 00–11 h; 22 Feb 2011 00–05 h) in news / NWS archives. If nothing is found, the report says "treated like the confirmed outages because they look the same".

### 4.2 Step 1b: Anomalies + external justification (notebook §3)

- ✅ **Detect on demand, explain with weather.** Features: log(actual/normal) for casual and registered, plus casual share. "Normal" = median for the same year, month and day type.
  - *Why:* raw counts are dominated by trend and season, so detecting on them would just flag winter. Weather is kept *out* of the detector so that it can be used as the *explanation*.
- ✅ **Four detectors, one per family:** robust z-score (median/MAD version of the 3σ rule), HBOS, kNN, LOF. Each flags its top 2 %; consensus = flagged by ≥ 2.
  - *Why:* each family defines "anomaly" differently. Agreement is low (Jaccard 0.15–0.58), so a vote is more reliable than any single method.
  - *Source:* [Anomaly Detection (1) – Slide 20 (taxonomy), 27 (3σ), 42–49 (HBOS), 53–64 (kNN), 65–90 (LOF), 116–117 (comparison), 19 (top-N % output)]; Tan et al. 2018, Ch. 9.
- ✅ **External evidence for all 15 consensus days** (table in the notebook, links in §6.3): 10 storms or shutdowns, 5 calendar days.
- ⬜ **(Kabir) Add a `bridge_day` feature** (Christmas Eve, day after Thanksgiving) and re-run. Expect fewer "calendar" anomalies.
  - *Why:* the dataset's `holiday` flag misses days that behave like holidays (casual share ≈ 40 %), which pollutes every baseline.
  ```python
  BRIDGE = ['2011-11-25', '2012-11-23', '2012-12-24', '2011-12-25']   # the 4 days found in §3; extend only with verified dates
  df['bridge_day'] = df.dteday.isin(pd.to_datetime(BRIDGE)).astype(int)
  ```
- ⬜ **(Binay) Optional:** search news for 2011-12-07 and 2011-03-06 (currently in-data evidence only: 21 h / 9 h of rain).

### 4.3 Step 2: Descriptive statistics (notebook §2)

- ✅ Summary table with skewness: casual counts are right-skewed (skew 1.27), registered roughly symmetric.
- ✅ Hourly profiles: registered = two commuter peaks (08, 17–18); casual = one afternoon hump, much larger at weekends.
  - *Why it matters:* it justifies modelling the two groups **separately**.
- ✅ Weather effect (raw): in rain/snow hours registered demand is −42 % and casual −61 % vs clear hours.
  - *Caveat:* this is confounded by hour and season, so §6–7 control for both.
- ⬜ (Binay) Pick **2 figures** for the report and **3** for slides from §2 (suggestion: `fig02` left+middle, `fig01`).

### 4.4 Step 3: Dimension reduction (notebook §5)

- ✅ **PCA on 24-dim day shapes** (each day divided by its total): PC1 + PC2 = 85 %. PC1 (74 %) = commuter vs leisure shape (corr −0.74 with casual share).
  - *Why shapes and no standardising:* volume is just season and trend; standardising would blow up noise at night.
  - *Source:* [Lecture: Dimension reduction / PCA – Slide ?]; Tan et al. 2018, Appendix B; Witten et al. 2016, Ch. 8 (projections).
- ✅ **PCA on recovery curves** (65 complete episodes): PC1 (46 %) = depth of the casual dip, PC2 (16 %) = speed.
- ⬜ (optional, ask the professor) k-means on recovery curves → "fast / slow recovery" types, with silhouette to choose k. Only do this if the professor wants a clustering angle.

### 4.5 Step 4: Research question (notebook §6–9)

**RQ-A: episode recovery curves (§6)** ✅
- Ratio = actual ÷ normal (same year, month, day type, hour), for 12 h after the first dry hour. "Recovered" = ≥ 0.9 for 2 consecutive daytime hours.
- Paired Wilcoxon (same episodes), with a bootstrap CI for the mean difference.
- *Why these choices:* the ratio makes 8 a.m. and 2 p.m. comparable. 90 % ≈ normal hour-to-hour noise. Two hours in a row stops one lucky hour from counting. Recovery times are skewed integers (so no t-test), and pairing removes episode-to-episode differences.
- *Source:* [Lecture: Evaluation / hypothesis testing – Slide ?]; Tan et al. 2018, Ch. 10 (statistical testing, avoiding false discoveries).
- ✅ **Sensitivity table** (notebook §6): threshold 85/90/95 % × minimum episode 1/2/3 h. Casual riders are slower in **all 9 settings** (registered medians 1–5 h, casual 6–13 h, every p ≤ 0.002).
  - *Why:* the 90 % threshold and the 2 h minimum are our choices. A conclusion that flips with them would not be a finding.

**RQ-B: regression on lagged weather (§7)** ✅
- log1p(count) OLS. Model A = calendar + current weather; Model B = A + recovery clock; Model C = B with non-linear humidity.
  - *Why:* this separates "still raining in effect" from "it is still humid now". We use HAC (Newey–West, 24 lags) because hourly errors are autocorrelated.
- Results:
  - Registered: −25 % (0 h), −9.5 % (2 h), not significant from 7 h.
  - Casual: −38 % (0 h), −9 % (7–12 h), −5 % (13–24 h).
  - Model C changes these by ≤ 4 percentage points.
- Prediction (train to Jun 2012, test Jul–Dec 2012):
  - Linear model: registered post-rain MAE −11 %, RMSE −15 %.
  - Gradient boosting: no gain, because current humidity and mist already tell it that rain just stopped (residual table in §7).
  - *Why the post-rain metric:* the error over all hours hides the effect, since post-rain hours are only 14 % of the test set.
  - *Source:* [Lecture: Regression – Slide ?]; [Lecture: Evaluation – Slide ?]; Witten et al. 2016, Ch. 4.6 (linear models) and Ch. 5 (evaluating numeric prediction: MAE/RMSE); Newey & West (1987).
- ✅ **Model D: is it the trip purpose?** Notebook §7: an extra dummy per recovery bin that is 1 on non-working days.
  - Result: **no clear difference**. All registered day-off terms have p > 0.2; for casual riders only the 1 h bin is significant (p = 0.02, 1 of 16 tests).
  - So the episode view's "registered riders are slow at weekends" (n = 20) was small-sample noise, and we **do not** claim a trip-purpose effect.
  - *Why we kept it:* it shows we test our own hints before presenting them. Good material for the challenges slide.
- ⬜ **(Kabir) `ended_before_commute` feature.** Does rain that stops at 06:00 vs 09:00 change the morning peak? Split the RQ-A curves by the end hour (06–08 vs 09–15 vs 16–19).
- ⬜ (optional) Long vs short episode interaction: `C(after_bin):I(prev_dur >= 4)`.

**RQ-C: association rules (§8)** ✅
- Apriori on 10 000 dry daytime hours, min_support 0.003, max_len 4; consequent = demand level (low < 0.7, high > 1.3 × normal); ranked by **lift**.
  - *Why lift:* the casual "low" base rate is 24.5 % vs 9.5 % for registered, so confidence alone favours casual rules.
- Mined on 2011 and re-checked on 2012: all 20 top rules keep lift > 1.2.
- *Source:* [Lecture: Association rules – Slide ?]; Tan et al. 2018, Ch. 5 (Apriori, §5.2–5.3; interest measures/lift, §5.7) and Ch. 6 (continuous attributes → discretisation); Witten et al. 2016, §4.5; Agrawal & Srikant (1994).
- ⬜ (Binay) Pick 3 rules for the slides: one per business action (rebalancing, promotion, forecasting).

**Business translation (§9)** ✅ first version
- Counterfactual with Model B: about 32 700 of about 89 500 weather-lost rides per year are lost *after* the weather has cleared.
- ⬜ **(Binay) Convert to money** only if we find a reputable price source for 2011–12 (Capital Bikeshare press release or archived pricing page). Otherwise keep it in rides and say so. **Do not guess prices.**

### 4.6 Step 5: Feature engineering (notebook §4)

All in §4, with a table of definition and reason for each feature.

| Feature | Status | Why it is creative / needed |
|---|---|---|
| `episode` (run of bad hours), `dur_h` | ✅ | turns 1 526 noisy rows into 492 events |
| `h_after`, `after_bin` (the recovery clock) | ✅ | the core representation change; the RQ cannot be asked without it |
| `prev_dur`, `prev_type` (snow proxy < 2 °C) | ✅ | "does a long storm leave a longer hangover?" (yes, for casual riders: 13 h vs 6 h) |
| `bad_last6`, `bad_last24` | ✅ | accumulated wetness |
| `exp_*`, `ratio_*` (same-hour baseline) | ✅ | puts every hour on one scale |
| `daypart`, `comfort_gap` | ✅ | commuter timing; muggy air |
| `bridge_day` | ⬜ | fixes missing holidays (found by the anomaly step) |
| `ended_before_commute` | ⬜ | rain stopping at 06:00 vs 09:00 |
| `temp_drop` over the episode | ⬜ optional | does a cold front behind the rain slow the recovery? |

*Source:* the learning outcome "change data representation to suit a particular problem" (course syllabus), quoted directly in the report; Witten et al. 2016, Ch. 8 (data transformations).

### 4.7 Steps 6–7: Report and slides

- ⬜ **Report (≤ 3 pages, Calibri Body 11)**. Word budget:
  - Intro + RQ + company value: ~150 words
  - Data & cleaning (cleaning log as a compact table): ~300 words
  - Anomalies: ~150 words + 1 figure
  - Features & method (3 methods, why each): ~350 words
  - Results: ~350 words + 2 figures (fig06, fig09) + 1 small table (rules)
  - Recommendations + limitations: ~200 words
  - References (slides + books + external)

  Every method sentence carries a slide citation.
- ⬜ **Slides (≤ 15, 15 min)**:
  1. Title
  2. Question & why it matters
  3. Data & 3 cleaning surprises
  4. Anomalies (fig03)
  5. From weather rows to *episodes* (fig04 + feature table)
  6. PCA: two rider populations (fig05)
  7. Recovery curves (fig06)
  8. Paired test + splits (fig07)
  9. Regression recovery effect (fig09)
  10. Is it just humidity? (Model C + gradient-boosting finding)
  11. Rules (3 rules + stability)
  12. Lost rides
  13. Recommendations
  14. Limitations & challenges
  15. Thank you / questions
- ⬜ **Challenges slide** (the criteria ask for "challenges faced"): wrong Readme formula, outages vs zero rides, weathersit mixes rain and snow, autocorrelation.

### 4.8 Deliverables & admin

- ⬜ **Supplementary A: LLM screenshots** (prompt + output) for *every* use: this Claude Code session (planning, code, event look-ups) and any ChatGPT use. Keep a list as we go. Say in the report that every LLM-suggested event was checked against a primary source.
- ⬜ **Supplementary B:** the notebook. Before zipping: *Restart & Run All* and confirm there are no errors.
- ⬜ **Supplementary C:** contribution table (who did what, in %). Agree on it with Binay by M4.
- ⬜ `GroupXX.zip` with the correct group number; test-open the ZIP before submitting.

---

## 5. Decision log (what we chose, what we rejected, why)

| # | Decision | Alternatives rejected | Why | Source |
|---|---|---|---|---|
| D1 | Hourly data, not daily | daily file | recovery happens within hours; daily totals hide it | RQ |
| D2 | Zero-ride hours = 0; ≥ 6 h gaps = outage (NaN) | impute all gaps; drop all | shutdowns confirmed by news; zero hours are real demand | Slide 18 (noise); Tan Ch. 2.2 |
| D3 | UCI temperature formula | Readme ÷41 | ÷41 gives no frost in D.C. winters | UCI page; NOAA WPC Jan-2011 storm |
| D4 | Anomalies detected on *demand ratios*, explained by weather | detect on all variables | otherwise the detector flags weather, not demand, and nothing is left to explain | Slides 18, 20 |
| D5 | 4 detectors + vote | 1 detector | low agreement (Jaccard 0.15–0.58) | Slides 116–117 |
| D6 | Episode = ≥ 2 h bad, ≥ 6 h dry after | every bad hour | single-hour flags are noisy; a new shower restarts the clock | ask the professor (Q1) |
| D7 | Baseline = same year × month × day type × hour, "normal" hours only | overall mean | the system grew 2011→12; strong hour/season pattern | n/a |
| D8 | Recovered = ≥ 90 % for 2 h | 100 %; 1 h | normal noise; one lucky hour ≠ recovery; result holds for 85/90/95 % × 1/2/3 h | sensitivity table (§6) |
| D9 | Paired Wilcoxon + bootstrap | t-test | skewed integer times; same episodes | Tan Ch. 10 |
| D10 | Separate models for casual / registered | one model on cnt | different populations (PCA, profiles) | §2, §5 |
| D11 | log1p target, OLS + HAC SE | plain OLS SE; Poisson | multiplicative effects; autocorrelation | Newey & West 1987 |
| D12 | Chronological split + TimeSeriesSplit | random K-fold | random folds leak the future through autocorrelation | Witten Ch. 5 |
| D13 | Metric = MAE/RMSE **on post-rain hours** | all-hours MAE | the effect is invisible on all hours | n/a |
| D14 | Rules ranked by lift; mined on 2011, checked on 2012 | confidence; no hold-out | base rates differ by rider type; operational rules must hold next year | Tan Ch. 5.7 |
| D15 | Fixed demand bands (0.7 / 1.3) | quantiles | business-readable ("30 % below normal") | n/a |
| D16 | Report the gradient-boosting non-gain | hide it | honest: trees read recency from humidity/mist; this supports "forecast must see the post-rain state" | n/a |

---

## 6. Sources

### 6.1 Course literature (syllabus)
- Tan, P.-N., Steinbach, M., Karpatne, A., Kumar, V. (2018). *Introduction to Data Mining*, 2nd ed., Pearson.
  - Ch. 2 Data (quality, preprocessing); Ch. 5–6 Association analysis; Ch. 9 Anomaly detection; Ch. 10 Avoiding false discoveries; App. B Dimensionality reduction; App. D Regression.
- Witten, I. H., Frank, E., Hall, M. A., Pal, C. J. (2016). *Data Mining: Practical Machine Learning Tools and Techniques*, 4th ed., Morgan Kaufmann.
  - §4.5 Association rules; §4.6 Linear models; Ch. 5 Credibility/evaluation; Ch. 8 Data transformations (discretisation, PCA).
- ⬜ **Check every chapter/section number against the actual books** before the report goes out.

### 6.2 Lecture slides
**Known:** *Anomaly Detection (1)*, Fanaee ([lab-2/AnomalyDetection1.pdf](../lab-2/AnomalyDetection1.pdf)):

| Topic | Slides |
|---|---|
| definition | 3 |
| anomaly vs noise | 18 |
| output (rank / top-N %) | 19 |
| method taxonomy | 20 |
| boxplot | 22 |
| histogram | 24 |
| Gaussian 3σ | 27 |
| GMM | 32–34 |
| multivariate outliers | 37–40 |
| HBOS | 42–49 |
| kNN | 53–64 |
| LOF | 65–90 |
| COF | 91–115 |
| comparison | 116–117 |
| OCSVM | 119–124 |
| clustering-based / DBSCAN | 126–129 |

⬜ **(Binay) Download the remaining lecture PDFs into `project/lectures/` and fill every `[Lecture: … – Slide ?]`** in the notebook and report:
- data preprocessing / exploration
- PCA / dimension reduction
- regression
- association rules
- evaluation

A missing slide citation can mean an F.

### 6.3 Papers
- Fanaee-T, H. & Gama, J. (2013). Event labeling combining ensemble detectors and background knowledge. *Progress in Artificial Intelligence*, pp. 1–15, Springer. doi:10.1007/s13748-013-0040-3. **(mandatory dataset citation, as given in the dataset Readme)**
- Agrawal, R. & Srikant, R. (1994). Fast algorithms for mining association rules. *VLDB*.
- Breunig, M. et al. (2000). LOF: identifying density-based local outliers. *SIGMOD*.
- Goldstein, M. & Dengel, A. (2012). Histogram-based outlier score (HBOS). *KI-2012 Poster and Demo Track*.
- Ramaswamy, S., Rastogi, R., Shim, K. (2000). Efficient algorithms for mining outliers from large data sets. *SIGMOD*.
- Newey, W. & West, K. (1987). A simple, positive semi-definite, heteroskedasticity and autocorrelation consistent covariance matrix. *Econometrica* 55(3).
- Wilcoxon, F. (1945). Individual comparisons by ranking methods. *Biometrics Bulletin* 1(6).

### 6.4 Data documentation and external evidence (checked 11 Oct 2026)

**Data documentation**
- UCI dataset page (temperature formula): https://archive.ics.uci.edu/dataset/275/bike+sharing+dataset

**Storms and system shutdowns**
- Hurricane Sandy, system closed ~36 h from 01:00 on 29 Oct 2012:
  - https://www.arlnow.com/?p=43145
  - https://wtop.com/news/2012/10/sandy-affects-metro-marc-and-vre-service/
  - https://batesareacivicassociation.org/2012/10/28/no-dc-circulator-and-capital-bikeshare-service-on-monday/
- Hurricane Irene, system closed 18:00 on 27 Aug 2011:
  - https://dcist.com/story/11/08/27/capital-bikeshare-closing-at-600-pm
  - https://m.washingtontimes.com/news/2011/aug/27/gray-proclaims-dc-ready-hurricane-irene/
- 26 Jan 2011 thundersnow, bike share shut:
  - https://www.wpc.ncep.noaa.gov/storm_summaries/event_reviews/2011/Mid-Atlantic_Northeast_WinterStorm_Jan2011.pdf
  - https://www.npr.org/sections/itsallpolitics/2011/01/27/133266936/u-s-capital-digs-out-from-wet-snow-blanket
  - https://oobrien.com/2011/01/washington-dc-storm-bike-share-shutdown/
- 2011 Halloween nor'easter:
  - https://en.wikipedia.org/wiki/2011_Halloween_nor%27easter
  - https://phys.org/news/2011-11-halloween-weekend-ghostly-picture-northeast.html
- Late-Dec 2012 storm complex: https://en.wikipedia.org/wiki/Late_December_2012_North_American_storm_complex
- 22 Apr 2012 coastal storm:
  - https://www.washingtontimes.com/news/2012/apr/22/heavy-rain-east-and-april-snowflakes-some/
  - https://www.weather.gov/media/publications/presto/2012may.pdf
- 16 Apr 2011 severe storms: https://www.weather.gov/lwx/events_20110416

**Calendar**
- Christmas Eve 2012, federal offices closed:
  - https://govexec.com/pay-benefits/2012/12/obama-gives-feds-christmas-eve/60325
  - https://fedsmith.com/2012/12/21/president-gives-feds-extra-paid-holiday-for-december-24th

**Wind**
- "Calm" = wind < 3 kt:
  - https://glossary.ametsoc.org/wiki/Calm
  - https://www.weather.gov/asos/WindSensor.html

---

## 7. Risks and how we handle them

| Risk | Mitigation |
|---|---|
| Few long and few snow episodes (34 episodes ≥ 4 h; 6 snow in RQ-A) | report n next to every split; call snow results "a hint"; main claims rest on all 93 episodes + the regression on 17 455 hours |
| weathersit 3 mixes rain and snow | temperature proxy, stated as a proxy |
| "Hangover" could be lingering humidity | Model C (non-linear humidity) → effects change ≤ 4 pp |
| Autocorrelation makes p-values too small | HAC standard errors; paired episode test; 2011 → 2012 rule hold-out |
| Report must fit 3 pages | cleaning log as a table, 3 figures max, details in the notebook |
| Running out of the 15 minutes | 13 content slides, 3 timed rehearsals, cut slide 10 first if long |
| Missing slide citations (F risk) | M2 deadline; every Why/Source cell must have a slide number before M4 |
