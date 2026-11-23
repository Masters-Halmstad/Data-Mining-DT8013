# Data Mining (DT8013) – Labs and Project

Halmstad University, Master's in Embedded and Intelligent Systems, 2026
Author: Binay Kumar Sah (project group partner: Kabir Tamari)

This repository has my five Data Mining labs and our course project, *"Does Bike Demand Stay Low after the Bad Weather Ends?"*. 

---

## Contents

| # | Lab / Project | Notebook | Dataset |
|---|---|---|---|
| 1 | Anomaly Detection 1 | `lab_anomaly_detection.ipynb` | `elderly_home.csv` |
| 2 | Anomaly Detection 2 (high-dimensional) | `lab_anomaly_detection_2.ipynb` | `organisms.csv` |
| 3 | Clustering | `Binay_Sah_Lab_Clustering.ipynb` | scikit-learn digits, iris |
| 4 | Time Series Analysis | `lab_timeseries_analysis.ipynb` | `air_quality.csv` |
| 5 | Interestingness and Association Rules | `lab_interestingness.ipynb` | clothing-store table, `BreadBasket_DMS.csv`, credit-card default data |
| P | Project: Weather and bike-demand recovery | `final.ipynb`, `Group5_Report.pdf` | `hour.csv` (UCI Bike Sharing) |

---

## Lab 1 – Anomaly Detection 1

### Intro
Finding unusual readings in sensor data from an elderly home (75,511 rows, 9 sensors: temperature, humidity, two CO2 sensors, MOX1–MOX4 and CO). The lab has no labels, so detection is unsupervised. The tasks: explore the data, guess which kinds of anomalies exist, run three detectors, pick a threshold, and study the top anomalies.

### Takeaway:
- **Types of anomalies:** univariate (one extreme value), multivariate (each value looks normal but the combination does not), and collective (a run of points that is unusual as a group).
- **HBOS:** builds one histogram per feature and adds `log(1/density)` over the features. It is very fast but assumes the features are independent, so it misses anomalies in combinations of features.
- **kNN:** the score is the distance to the k-th nearest neighbour, which finds points that are isolated from the rest of the data.
- **LOF:** compares a point's local density with the density of its neighbours. LOF ≈ 1 means normal and LOF ≫ 1 means anomaly. It handles clusters of different density.
- **The PyOD workflow:** drop duplicates, standardise, `fit`, then read `decision_scores_` and `labels_`.
- **Thresholds:** `contamination` only sets the cut-off and does not change the scores. Choosing the threshold (a percentile or the boxplot rule `Q3 + k·IQR` on the scores) is a judgement call that changes the result a lot.


## Lab 2 – Anomaly Detection 2 (High-Dimensional Data)

### Intro
Anomaly detection on codon-usage data from organisms (13,028 rows × 62 features). The lab shows why distance-based methods struggle in high dimensions and compares three ways to deal with that: PCA followed by a distance-based detector, an autoencoder, and Isolation Forest on random feature subspaces.

### Takeaway 
- **Curse of dimensionality (Task 1):** as the dimension d grows from 2 to 1000, the ratio of nearest to farthest distance approaches 1, so "near" and "far" stop meaning anything.
- **Preprocessing:** there were no missing values. I removed 40 duplicates, which left 12,988 rows. I standardised the features because their variances differed by about 25×.
- **PCA + kNN:** 29 components keep 90 % of the variance, even though the scree plot has its elbow at 3–4 components. kNN then runs on the reduced data, with the boxplot rule `Q3 + 1.5·IQR` on the scores.
- **Autoencoder (62→32→16→8→16→32→62):** the reconstruction error is the anomaly score. The error distribution is strongly right-skewed (median 0.16, max 8.70), and 747 organisms (5.75 %) were flagged.
- **Isolation Forest stability:** I ran it on random subspaces of size d = 5 … 62 and measured how much the top-100 lists overlap between runs. Stability rose from about 0.1 at d = 5 to about 0.55 at d = 62.
- **Method comparison (top 100 of each):** PCA+kNN and the autoencoder share 54 organisms, PCA+kNN and IForest share 24, the autoencoder and IForest share 29, and all three share 22.

## Lab 3 – Clustering

### Intro
Grouping data without labels using three clustering paradigms: **k-means** (partition-based), **agglomerative hierarchical clustering**, and **DBSCAN** (density-based), with DBSCAN written from scratch. The data are the digits dataset (64 pixel features) and the iris dataset.

### Takeaway 
- **k-means:** clusters with and without PCA, the elbow curve on inertia for choosing k, and scoring against the true labels with accuracy (after matching clusters to labels) and **ARI**.
- **Hierarchical clustering:** single, complete, average and Ward linkage, dendrograms, and cutting the tree into clusters.
- **DBSCAN from scratch:** `neighborQuery` (the eps-neighbourhood), `grow_cluster` (density-reachability), and core, border and noise points. I checked my version against scikit-learn's DBSCAN.
- **Scaling and PCA** matter for every distance-based method.


## Lab 4 – Time Series Analysis

### Intro 
Analysing two air-quality signals, **PT08.S1(CO)** and **NOx(GT)**, over about 13 months. The tasks: preprocessing, decomposition, stationarity, forecasting, clustering of daily shapes, and motif/discord mining with the matrix profile.

### Takeaway 
- **Preprocessing:** convert dates to datetime, replace the `-200` sensor code with NaN (1,639 times, all in NOx), remove partial first and last days, and resample to **daily means**.
- **Decomposition** (`seasonal_decompose`, period = 7): NOx fits the multiplicative model and CO the additive one. CO's weekly seasonality turned out to be very small.
- **Stationarity (ADF):** both series were non-stationary at first (CO p = 0.974, NOx p = 0.357). First differencing for CO and log plus first differencing for NOx made both stationary (p < 0.001).
- **Forecasting (90 days, Holt-Winters vs SARIMA):** SARIMA was better for both signals (CO MAE 330 vs 1,151; NOx MAE 95 vs 171). Holt-Winters on NOx even lost to the seasonal-naive baseline, because its undamped trend over-extrapolated.
- **Clustering daily shapes** (first 6 readings per day, `TimeSeriesKMeans` with DTW and with Euclidean distance): k = 2 chosen by silhouette score. DTW and Euclidean gave almost the same clusters (ARI 0.981 for CO, 0.878 for NOx).
- **Matrix profile** (`stumpy`, m = 7): motifs (repeated patterns) and discords (unusual windows).


## Lab 5 – Interestingness and Association Rules

### Intro 
Market-basket analysis and measures of how interesting a pattern is. Part A computes support, confidence, lift and conviction by hand on a 7-transaction clothing-store table. Part B runs Apriori on **BreadBasket_DMS.csv** (9,684 transactions) and draws the rules as a graph. Part C finds concise, general and peculiar patterns in the **credit-card default** dataset (30,000 clients, 22.1 % default rate) and explains how **entropy** relates to itemset frequency.

### Takeaway 
- **Support** = P(X ∪ Y), **Confidence** = supp(X ∪ Y) / supp(X), **Lift** = supp(X ∪ Y) / (supp(X)·supp(Y)) (< 1, = 1, > 1), **Conviction** = (1 − supp(Y)) / (1 − conf).
- **The Apriori principle:** every subset of a frequent itemset is frequent. Candidates are built by a self-join and then pruned (e.g. PBM is pruned because BM is not frequent).
- The `efficient-apriori` library: rule objects with `lhs`, `rhs` and their measures. In Task 7 I kept the rules with `rhs = Bread` and drew them with networkx.
- **Interestingness criteria:** conciseness (few conditions), generality (large coverage) and peculiarity (a small group far from the rest). Each pattern is judged against the baseline default rate.
- **Entropy:** H = −Σ p·log₂ p. Itemsets whose items nearly always appear together have low entropy.


## Project – Does Bike Demand Stay Low after the Bad Weather Ends?

**Group 5:** Binay Kumar Sah, Kabir Tamari
**Files:** `final.ipynb` (analysis), `Group5_Report.pdf` (report), `hour.csv` (data)

### Research questions
1. Does bad weather affect bike demand, and how much?
2. When rain or snow stops, how long does demand take to return to normal?
3. Do registered riders (commuters) and casual riders recover at the same speed?

Only methods from the labs are used: anomaly detection (kNN, LOF), time-series ideas (trend and seasonality removal, baselines, forecast evaluation) and interestingness (lift as an observed ÷ expected ratio).

### 1. Data summary
- **Source:** UCI Bike Sharing Dataset, Capital Bikeshare, Washington D.C., 2011–2012.
- **File:** `hour.csv`, **17,379 hourly rows**. Hourly data was chosen over daily because recovery happens within hours.
- **Main columns:** date/time (`dteday`, `yr`, `mnth`, `hr`, `weekday`, `workingday`, `holiday`), weather (`weathersit` 1–4, `temp`, `atemp`, `hum`, `windspeed`), and demand (`casual`, `registered`, `cnt = casual + registered`).
- **Quality check:** no NaN cells, no duplicate rows, and `cnt = casual + registered` holds everywhere. But two full years should have 731 × 24 = **17,544 hours**, so **165 hours are missing**.
- **Demand profile:** an average day has **4,504 rides, 82 % registered and 18 % casual** (the casual share ranges from 1 % to 51 %). Registered riders peak at 08:00 and 17:00 on working days, while casual riders only rise in the afternoon.

### 2. Approach

**Step 1 – Gap analysis and filling the timeline.** The 165 missing hours form **75 gaps**:
- **70 short gaps (1–2 h)**, almost all between 02:00 and 05:00. These are hours with no rentals, which also explains why the minimum `cnt` is 1 and never 0. They were filled with 0 rides and flagged `zero_fill` (76 h).
- **5 long gaps (6–36 h)**, which match documented storms that shut the service down (Hurricane Sandy, Hurricane Irene, a snowstorm, freezing rain, snow and sleet). Rides were left as **NaN** with an `outage` flag and `weathersit = 4` (89 h). Demand during a shutdown is unknown, not zero, so filling it with 0 would teach a model that a normal day can have no riders.
- Weather in the added rows was forward-filled from the previous hour.

**Step 2 – Cleaning and correction.** Every change follows a missing-value approach from Lecture 2 and gets a True/False flag column:

| Problem | Hours | Action |
|---|---|---|
| Short night gaps | 76 | 0 rides, flag `zero_fill` |
| Storm shutdowns | 89 | rides left NaN, flag `outage` |
| No weather in missing hours | 165 | forward fill |
| README temperature formula gives impossible values (never below 0 °C, 9 °C in heavy snow) | all | original UCI formula `t = −8 + 47·temp` (594 hours below 0 °C) |
| Season codes shifted | all | relabelled (1 = winter, Dec–Mar, mean 6.0 °C) |
| Humidity 0 % for a whole day (found by kNN/LOF) | 24 | monthly median for that weather type, flag `hum_filled` |
| `atemp` stuck for 2 days (found by kNN/LOF) | 48 | predicted from `temp` with a fitted line, flag `atemp_filled` |

**Step 3 – Anomaly detection (kNN + LOF, unsupervised).**
- Two passes with feature subsets chosen by meaning (feature bagging): **Pass A** uses only weather features (to find storms and broken sensors), and **Pass B** uses only rider counts (to find unusual demand). Fewer, relevant columns stop a faulty day from looking ordinary (curse of dimensionality).
- The unit is **days, not hours**, because neighbouring hours would hide each other (masking). Settings: k = 15, standardised features, detectors combined by **average rank**, and a day is trusted when both detectors rank it high.
- In Pass B, each day's rides were divided by the median day of the same year, month and day type (removing trend and seasonality), then log-transformed.

**Step 4 – Descriptive statistics and PCA.** We compared mean demand by weather type. PCA of the daily ride shapes was used as a supporting check of the commuter vs leisure difference.

**Step 5 – Baseline ("normal" demand) and recovery clock.**
- The baseline was trained only on **clean hours**: no rain, not within 24 h after rain, not an outage (**9,906 hours**). This keeps the slow return after rain out of "normal".
- **Primary baseline – kNN regression:** for each hour, the mean of the 6 nearest clean days in time with the same hour and day type. Nearness in time handles trend and season, and same hour and day type handle the daily and weekly cycle.
- **Robustness baseline – median table:** the median for the same year, month, day type and hour.
- **Validation:** we hid random 2-day blocks (80 days, 8 repetitions), predicted them from the remaining clean days, and scored daytime hours (07–21) with MAE as a % of mean demand, plus the bias.
- **Recovery ratio** for each hour after rain = **sum of actual ÷ sum of normal** (not the mean of hourly ratios, which quiet hours would distort). This observed ÷ expected ratio is the same idea as **lift**.

### 3. Finding and Conclusion

**Anomaly detection**
- **Pass A:** the scores flatten after about rank 15. Only **3 of the top 15 days are sensor faults**: `atemp` stuck on 2012-08-17 and 2012-12-29, and humidity 0 on 2011-03-10. These were repaired and flagged. The other 12 are real weather (storms, rain, dry wind, big swings), which are meaningful anomalies, not noise.
- **Pass B:** all top-15 days are explained by storms (Sandy, the 2011 Halloween nor'easter, the April and December 2012 storms) or calendar effects (Christmas Eve, New Year, the day after Thanksgiving). None is a data error, so they stay in the data.
- **No day appears in both top-15 lists**, which supports splitting the features.

**Effect of weather (RQ1)**
- Compared with clear hours, **mist** cuts registered rides by 12 % and casual rides by 27 %. **Light rain/snow** cuts them by **42 % and 61 %**.
- PCA: PC1 holds 74 % and PC2 11 % of the variance. Working days and weekends separate along PC1, which correlates −0.74 with the casual share. This confirms the commuter vs leisure difference.

**Baseline accuracy (mean of 8 draws)**

| Baseline | Registered error / bias | Casual error / bias |
|---|---|---|
| kNN | 14.7 % / +0.1 % | 28.6 % / +4.8 % |
| Median table | 15.4 % / +0.1 % | 28.7 % / −0.9 % |

Both baselines are equally accurate. Casual demand is harder to predict, and kNN runs about 5 % too high for casual riders (which would make their recovery look slower), so the median baseline is used as a check.

**Recovery (RQ2 and RQ3)**
- Demand does **not** return when the rain stops. Registered and casual riders are at **56 % and 42 % of normal while it rains**, and still at **71 % and 54 % in the first dry hour**.
- **Registered riders recover in about 3 h** (with both baselines). **Casual riders need 7–9 h.**
- Bad weather costs about **101,000 daytime rides per year**, and **39 % of that loss happens after the rain**. For registered riders the after-rain loss is mostly in the first hours. For casual riders it continues 5–24 h later.

**Recommendations**
1. Add `hours_after_rain` as a feature in demand forecasts.
2. Restock commuter stations about 3 h after the rain. Leisure stations stay quiet for 7–9 h, which leaves time to rebalance.
3. A "sun is out" day-pass offer could win back casual riders.

**Limitations:** we count rides, not revenue; "rain" mixes light rain, snow and storms; and the data covers only one city over two years.

**Conclusion:** bike demand stays low after bad weather ends. Commuters recover in about 3 hours, leisure riders in 7–9 hours, and 39 % of the weather loss happens after the rain.

### References
1. H. Fanaee-T, *Bike Sharing Dataset*, UCI Machine Learning Repository, 2013. doi:10.24432/C5W894
2. The full reference list (storm and holiday sources) is in `Project_Report.pdf`.