# The Weather Hangover: A Complete Explanation of the Bike-Sharing Data Mining Project

**Project title:** Does Bike Demand Stay Low After Bad Weather Ends?
**Course:** DT8013 Data Mining, Halmstad University
**Group:** Binay Kumar Sah and Kabir Tamari (Group 5)
**Dataset:** UCI Bike Sharing Dataset, Capital Bikeshare, Washington D.C., years 2011 and 2012 (Fanaee-T and Gama, 2013)
**Main notebook:** `bike_weather_recovery.ipynb`

---

## About this document

This is a study guide. It explains the whole project from start to finish: what the data is, every step of the analysis, every number that came out, and, most importantly, **why** each choice was made and which alternatives were rejected. It is written so that someone who has never seen the notebook can understand the project in depth and defend it in front of an examiner.

The project follows one storyline from beginning to end:

> **data → weather episodes → how long demand stays low → why it stays low → what the company should do about it.**

---

## Part 1. The project in a nutshell

Everybody knows that fewer people rent bikes when it rains. That is obvious, and most analyses of this dataset stop there. This project asks a different and less obvious question: **what happens after the rain stops?**

When the rain ends, does bike demand jump straight back to normal, or does it stay low for a while, like a "hangover"? And is the hangover the same for everyone? The dataset splits riders into two groups:

- **Registered riders** are annual or monthly members. They are mostly commuters who ride to work and back.
- **Casual riders** pay per trip or buy a day pass. They are mostly tourists and people riding for leisure.

The answer the project found is clear and holds up across three independent methods:

1. **Yes, there is a hangover.** At the first dry hour after rain or snow, registered demand is about 25 % below normal and casual demand is about 38 % below normal, even after controlling for the current weather.
2. **Commuters bounce back fast, and leisure riders do not.** Registered riders are back within 10 % of normal about **2 to 3 hours** after the rain stops. Casual riders take roughly **7 to 12 hours**, about half a day.
3. **The hangover costs real rides.** Of about 89,500 rides lost to bad weather each year, about **32,700 (37 %) are lost after the weather has already cleared.** For casual riders, more than half (55 %) of their weather losses happen after the rain has stopped.

Why does this matter to the bike company? A bike-sharing operator moves bikes around the city with trucks ("rebalancing") and runs promotions, and both are timed to the weather. If demand lags behind the weather, a forecast that only looks at the current weather will be too optimistic in the hours after rain. The company would send bikes to commuter stations too early, and it would miss the window where a promotion could bring the casual riders back.

---

## Part 2. The course assignment and why every part of the project exists

The course project has fixed requirements. Understanding them explains why the notebook contains some steps that are not directly about the research question.

The assignment asks every group to:

1. **Pre-process the data:** clean it, handle missing values, find anomalies with several *unsupervised* methods, and justify the anomalies with external information (for example, "low rentals because of a storm").
2. **Summarise the data** with descriptive statistics.
3. **Apply dimension reduction and visualisation** to show the important patterns.
4. **Ask a research question** and answer it with tools from the course. It should be clear and creative, and reveal hidden patterns.
5. **Engineer features** that could affect the outcome (time of year, holidays and so on).
6. Write a **report** of at most 3 pages (Calibri, size 11).
7. Give a **presentation** of at most 15 slides and 15 minutes, followed by 5 minutes of questions.

The grading criteria put particular weight on:

- how clear and convincing the data cleaning is;
- how relevant and creative the research question is, **especially the gain for the bike rental company**;
- whether the techniques are relevant and whether each choice is justified ("why and how?");
- creativity in the engineered features;
- using the **right evaluation metric**;
- clarity of the report and presentation;
- **citations to the course slides** (for example "Lecture 3 – Slide 87"). Missing citations can lead to a failing grade.

The deliverables are a ZIP file with the report, the slides, screenshots of every LLM prompt and output used (Supplementary A), the Jupyter notebook (Supplementary B) and a contribution split between the members (Supplementary C).

So the notebook has nine sections. Sections 1 to 5 cover the required tasks (cleaning, statistics, anomalies, features, dimension reduction). Sections 6 to 8 answer the research question with three different methods, and Section 9 turns the result into business value.

---

## Part 3. How the topic was chosen

The topic choice was strategic, not random.

The first option the group considered was **anomaly detection**: find unusual days and explain them with real events such as hurricanes. It is a natural fit, because the dataset's own Readme suggests it, and the dataset's original paper (Fanaee-T and Gama, 2013) is about event detection. But by the time topics were registered, **five other groups** had already chosen anomaly detection as their main question, and **four groups** had chosen clustering of daily demand shapes. Those two lanes were saturated.

The group's reasoning was this: every group has to do anomaly detection, descriptive statistics and PCA anyway, as required tasks. Those steps cannot make a project stand out. **The research question is the only place where a project can be different.** So the group looked for what was still open: association rules, classification, and **effects that carry over in time** (what happened in the past hours, not just right now).

Other options considered and rejected included:

- **Peak-overload early warning** (classify which hours will hit the top 10 % of demand);
- **"Conversion days"** (predict casual-dominated days for membership marketing);
- **Pretrained time-series transformers** (Chronos, TimesFM, TabPFN) against a trained model, which was more technical and riskier;
- **An LLM-built event calendar**, which was creative but overlapped with groups that look up events;
- **Conformal prediction intervals**;
- **Asking a chatbot to predict demand**, rejected because the business value was weak.

The **"Weather Hangover"** option won for four reasons:

1. **It was unique.** One other group studied extreme weather *on the day*; nobody studied the hours *after*.
2. **There was early evidence.** A quick first check of dry daytime hours showed that one hour after rain stops, total demand was only 71 % of normal (casual 60 %, registered 75 %), climbing back to 100 % only after 7 to 12 hours. In the daily file, a clear day after a rainy day reached only 81 % of normal demand. The signal was real.
3. **It uses course topics nobody else used:** association rules and regression.
4. **It has a strong business story:** timing of rebalancing and promotions.

The research question became:

> **After rain or snow stops, how many hours does it take for bike demand to return to normal, and do registered (commuter) and casual riders recover at different speeds?**

---

## Part 4. The dataset

The data comes from **Capital Bikeshare**, the public bike-sharing system of Washington D.C., for the two years **2011 and 2012**. Hadi Fanaee-T at the University of Porto aggregated the raw trip logs and added weather information from a weather website.

There are two files:

- **`day.csv`**: 731 rows, one per day.
- **`hour.csv`**: 17,379 rows, one per hour. This is the file the project mainly uses, because a recovery that takes a few hours is invisible in daily totals.

Each row contains:

- **Calendar fields:** date, season, year (0 = 2011, 1 = 2012), month, hour, holiday flag, weekday, and working-day flag (1 if the day is neither a weekend nor a holiday).
- **Weather situation (`weathersit`)**, a category from 1 to 4:
  - 1 = clear or partly cloudy;
  - 2 = mist and cloudy;
  - 3 = light rain or light snow, sometimes with thunderstorms;
  - 4 = heavy rain, ice pellets, thunderstorm, or snow with fog.
- **Weather measurements:** temperature, "feels-like" temperature (`atemp`), humidity and wind speed. All are scaled (normalised) between 0 and 1.
- **Demand:** `casual` (casual riders), `registered` (members), and `cnt`, the total.

In this project, **"bad weather" means `weathersit` = 3 (rain or snow)**. A **weather episode** is a run of consecutive bad-weather hours.

The dataset must be cited as: Fanaee-T, H. and Gama, J. (2013), "Event labeling combining ensemble detectors and background knowledge", *Progress in Artificial Intelligence*, Springer.

---

## Part 5. Step 1: Data audit and cleaning

### The philosophy

The group decided not to just call `dropna()` and move on. The grading rewards a cleaning process that is "clear and compelling", so **every problem is found with evidence, fixed with a stated decision, and justified with a reason.** The result is a nine-row **cleaning log** with the columns *issue → number of rows → evidence → decision → why*.

There was also a reason specific to the research question. The project measures "3 hours after the rain". That only works if the hourly table is a **complete, regular time grid**, where the next row is always the next hour. If hours are missing, "the next row" might be 5 hours later, and the recovery clock would be wrong.

### First impression: the data looks perfect

The first audit found no empty (NaN) cells and no duplicate rows. The total `cnt` always equals `casual + registered`, and the daily file matches the sum of the hourly file. At first glance, the data looks spotless.

But two years of hours should give 731 × 24 = **17,544 rows**, and the file has only **17,379**. So **165 hours are simply absent.** They are not marked as missing; they are just not there.

### Problem 1 and 2: The missing hours have two different causes

This is the most interesting part of the cleaning, and it works like a small detective story.

**Clue 1:** The smallest hourly count in the whole file is 1. There is not a single hour with zero rentals. But in a city at 4 a.m., there must be hours when nobody rents a bike. So **hours with zero rentals were never written to the file.**

**Clue 2:** The group grouped the missing hours into gaps of consecutive missing hours and looked at their lengths:

- **70 short gaps** of 1 or 2 hours (64 one-hour gaps and 6 two-hour gaps, 76 hours in total). Almost all fall between 02:00 and 05:00 at night.
- **5 long gaps** of 6 hours or more (89 hours in total):
  - 29–30 October 2012: **36 hours**, the dates of **Hurricane Sandy**;
  - 26–27 January 2011: **22 hours**, a big **thundersnow storm**;
  - 27–28 August 2011: **13 hours**, **Hurricane Irene**;
  - 18 January 2011: 12 hours;
  - 22 February 2011: 6 hours.

**Clue 3:** The group checked news archives. Capital Bikeshare **shut the whole system down** during these storms. Irene: the news says the system closed at 18:00 on 27 August 2011, and the data gap starts at exactly 18:00. Sandy: the system closed from 01:00 on 29 October 2012 for about 36 hours, and the gap starts at 01:00 and lasts exactly 36 hours. The January 2011 snowstorm also caused a shutdown that night.

**The conclusion:** there are two kinds of missing hours, and they must be treated differently.

- **Short night gaps are real zero demand.** Nobody rode, so the hour was not recorded. Decision: **insert the hour with 0 rides.** Why: it keeps the grid regular, and zero is the true value.
- **Long gaps are system outages.** The bikes were locked away. Demand was *not observed*, and it is not zero demand. Decision: **keep the counts as missing (NaN), add an `outage` flag, and leave these hours out of every model.** Why: filling them in would invent rides that were impossible, and setting them to zero would wrongly suggest people did not want to ride.

The 6-hour threshold that separates the two groups is not fragile: **no gap is 3, 4 or 5 hours long**, so any threshold between 3 and 6 gives the same split. Two of the long gaps (18 January and 22 February 2011) could not be confirmed in the news. They are treated the same way because they look the same.

**Problem 3:** Inserting hours creates rows with no weather values. Decision: **forward-fill** the weather from the previous hour. Why: weather is persistent from hour to hour, and forward-filling avoids cutting a storm episode in two.

### Problem 4: The Readme's temperature formula is wrong

The `Readme.txt` that comes with the data says temperature was "divided by 41". The UCI website says something different: temperature was min-max scaled between −8 °C and +39 °C, so the real temperature is −8 + 47 × temp.

The two formulas cannot both be right, so the group tested which one is physically plausible:

| Check | Readme formula (÷41) | UCI formula |
|---|---|---|
| Coldest hour in two years | +0.8 °C | −7.1 °C |
| Hottest hour | 41.0 °C | 39.0 °C |
| During the 26 Jan 2011 snowstorm | 8.9 °C | 2.2 °C |

With the Readme formula, Washington D.C. would never have gone below freezing in two whole winters, and a snowstorm with about 5 inches of snow would have happened at 9 °C. Both are impossible. **Decision: use the UCI formula.** (For the feels-like temperature: −16 + 66 × atemp.) This matters for the project, because the rain-versus-snow proxy later is a temperature threshold.

### Problem 5: Humidity of 0 % on a rainy day

On **10 March 2011**, humidity is exactly 0 % for the whole day, and that day was rainy (weather situation 3 in 22 of the hours). Zero humidity in the rain is physically impossible, so it must be a sensor dropout. **Decision:** replace it with the median humidity for the same month and the same weather type. Why a *conditional* median: it keeps the natural link that rainy hours are humid. A plain overall median would have put average humidity into a rainy day.

### Problem 6: The feels-like temperature is stuck

On **17 August 2012**, the feels-like temperature has a single constant value all day, about 0 °C, while the air temperature was about 24 °C. Elsewhere, feels-like and air temperature are almost perfectly correlated (r = 0.99). **Decision:** predict the feels-like temperature from the air temperature with a quadratic fit on all the other hours.

A subtle detail: the group only flagged hours where feels-like is far **below** air temperature. Days where feels-like is far **above** air temperature, such as 21 July 2011, are a real heat wave (humidity makes it feel hotter), not an error.

### Problem 7: The season labels in the Readme are wrong

The Readme says season 1 = "springer" (spring). But the season code changes on 21 March, 21 June, 23 September and 21 December, and the year starts on 1 January with code 1. So **code 1 is winter**, then spring, summer and fall. **Decision:** relabel the seasons. Otherwise every seasonal chart would be mislabelled.

### Problem 8: Weather situation 4 is almost empty

Only **3 hours** in two years have weather situation 4 (heavy rain or snow). A category with 3 rows cannot be estimated by any model. **Decision:** merge it into category 3, "bad weather".

### Problem 9: Wind speed of zero

About 2,180 hours have wind speed exactly 0. Is that a missing value in disguise? The group checked: the next-smallest wind speed is 6 km/h, and nothing lies between 0 and 6. U.S. weather stations report wind below 3 knots (about 5.6 km/h) as "calm" and write it as zero. So **zero is a real measurement of calm air. Decision: keep it.**

### Why the cleaning section is strong

Three findings make the cleaning section stand out, and they are good material for the presentation:

1. The missing hours have **two causes**, separated with news evidence that matches the gaps to the hour.
2. The **official Readme's temperature formula is wrong**, proved with physical plausibility.
3. The **official Readme's season labels are wrong**, proved from the dates the code changes.

---

## Part 6. Step 2: Descriptive statistics

### Why this step

Before asking *how fast* demand recovers, three things need to be known: how large and how skewed demand is; whether casual and registered riders really behave differently (if not, half the research question would be pointless); and how big the weather effect is in the first place.

### The numbers

Per day, on average, the system had about **4,500 rides**: about 3,660 by registered riders and 850 by casual riders. Casual riders make up on average **17.6 %** of a day's rides, ranging from about 1 % to 51 %.

- **Casual demand is right-skewed** (skewness 1.27). Most days have modest casual numbers, but a few sunny weekend days have huge numbers.
- **Registered demand is roughly symmetric** (skewness close to 0).
- The daily mean temperature ranged from about −5 °C to +32 °C.

**Day type:**

| | Registered per day | Casual per day | Casual share |
|---|---|---|---|
| Working day | 3,978 | 607 | 12 % |
| Weekend or holiday | 2,959 | 1,371 | 29 % |

Registered riders ride more on working days, and casual riders more than double on days off.

**Hourly profiles** show the clearest difference between the two groups. On working days, **registered riders have two sharp peaks, at 08:00 and at 17:00–18:00**, the classic commuter shape. **Casual riders have a single gentle afternoon hump**, much larger at weekends. These really are two different populations with different reasons for riding. This is why the whole project models them **separately**.

**Weather effect (raw):** compared with clear hours, average rides per hour are:

| Weather | Registered | Casual |
|---|---|---|
| Clear | 163.7 (baseline) | 40.4 (baseline) |
| Mist | −11.5 % | −27 % |
| Rain or snow | **−42 %** | **−61 %** |

Casual riders are much more sensitive to weather than registered riders. But this raw comparison is **confounded**: rain might fall more often at certain hours or in certain seasons. Later sections control for hour and season properly.

**Trend:** the system grew strongly from 2011 to 2012. Total rentals rose by about 65 %. So any "normal demand" baseline must be computed **separately for each year and month**, or 2012 would look like one long anomaly compared with 2011.

### Figures

- **Figure 1 (daily time series):** daily rides over the two years, with a 7-day moving average for each rider type (registered in blue, casual in orange). It shows the seasonal wave, summer high and winter low, the growth from 2011 to 2012, and dotted lines on days with data outages.
- **Figure 2 (profiles and weather):** three panels. Hourly profiles on working days (registered two-peak shape), hourly profiles on weekends (casual afternoon hump), and mean rides per hour by weather (clear, mist, rain/snow), where both lines drop and the casual line drops proportionally more.

Throughout the project, **registered is always blue and casual is always orange**, so the audience never has to re-read a legend.

---

## Part 7. Step 3: Anomaly detection and external evidence

### The key methodological idea: detect on demand, explain with weather

Anomaly detection is a required task. The obvious approach would be to feed all the columns (counts, temperature, humidity, wind) into a detector. The group deliberately did **not** do that, for two reasons:

1. **Raw counts are dominated by trend and season.** A detector run on raw counts would simply flag winter days and 2011 days, because they are low. That is not interesting.
2. **If weather goes into the detector, it can no longer be used to explain the result.** The assignment asks to *justify* the anomalies with external information. To do that honestly, the explanation must be independent of the detection. So weather is kept **out** of the detector and saved as the **explanation**.

So each day's demand is compared with what is normal for that day. **"Normal" is the median demand for the same year, the same month and the same day type** (working day or not). The detector then looks at three features:

- the **log ratio** of actual to normal casual demand;
- the **log ratio** of actual to normal registered demand;
- the **casual share** of the day.

A log ratio of 0 means a perfectly normal day; −0.69 means half of normal; +0.69 means double. Using a log ratio treats "half" and "double" symmetrically.

### Four detectors, one from each family

The anomaly detection lecture presents a taxonomy of methods (Anomaly Detection lecture 1, slide 20), and the comparison slides (116–117) show that different families define "anomaly" differently. So instead of trusting a single method, the group used **one detector from each family** and let them vote.

1. **Robust z-score (statistical, univariate).** This is the classic "3-sigma rule" (slide 27): how many standard deviations a day lies from the centre. The robust version uses the **median** and the **MAD** (median absolute deviation) instead of the mean and standard deviation. Why: big anomalies such as Hurricane Sandy would inflate the mean and standard deviation and hide themselves. The median and MAD are not pulled around by extreme values.

2. **HBOS, the Histogram-Based Outlier Score (slides 42–49).** It builds a histogram for each feature separately. A day that falls in a rare (low) bar of a histogram gets a high anomaly score, and the scores of the features are added together. It is very fast but assumes the features are independent.

3. **kNN distance (slides 53–64).** For each day, it measures the distance to its 10th nearest neighbour. A day far from all other days is anomalous. This is a *global* view of "being far from everything".

4. **LOF, the Local Outlier Factor (slides 65–90).** It compares how dense the neighbourhood around a day is with how dense the neighbourhoods of its neighbours are. A day in a sparse spot next to a dense cluster is anomalous. This is a *local* view, and it can find a day that is unusual for its own neighbourhood even if it is not globally extreme.

**How many anomalies?** Each detector outputs a ranking, and the lecture describes "top-N %" as a standard output (slide 19). Each detector flags its **top 2 %, which is 15 days**. A day flagged by **at least 2 of the 4 detectors** is a **consensus anomaly**.

A useful side result shows why a top-N cut was used instead of the classic threshold: with the plain "|z| > 3" rule, **73 days** would count as anomalies. That is 10 % of all days, far too many to be "rare". The demand data has heavy tails, so the 3-sigma rule, which assumes a normal distribution, over-flags.

### The detectors disagree, and that is the point

The overlap between detectors was measured with the **Jaccard index**: the number of days both flag, divided by the number of days either flags. A value of 1 means identical; 0 means no overlap.

- Robust z and kNN overlap the most (**0.58**). Both essentially look for days far from the centre.
- LOF overlaps the least (**0.15 to 0.25**). It looks for locally odd combinations, such as a normal total with an unusual casual share.
- Only **2 days** are flagged by all four detectors: **Hurricane Sandy (29 Oct 2012)** and the **26 December 2012 storm**.

Vote counts: 696 days got no votes, 20 got one vote, 7 got two, 6 got three and 2 got four. So **15 consensus days**.

Because agreement is low, any single detector would give a quite different list. The vote is more reliable than any one method.

### All 15 consensus anomalies are explained

All 15 consensus anomalies are **low-demand days**, and **every one of them can be explained**, by the weather in the data, by news archives, or by the calendar. The group used an LLM to *suggest* candidate events for each date and then **checked every suggestion against a primary source** (news articles, National Weather Service pages, NOAA storm reports) before accepting it. This is documented in Supplementary A.

**Ten are storms or weather days:**

- **29 Oct 2012 (Monday):** Hurricane Sandy. 22 rides against a normal of about 7,340. The system was closed from 01:00 for about 36 hours.
- **30 Oct 2012:** Sandy, day two; the system reopened in the afternoon.
- **27 Aug 2011:** Hurricane Irene; the system closed at 18:00.
- **27 Jan 2011:** the day after the 26 January thundersnow storm (about 5 inches of snow during the evening rush, about 300,000 people without power).
- **29 Oct 2011:** the 2011 Halloween nor'easter (a rain and snow mix, with a trace of snow in D.C.).
- **26 Dec 2012:** the late-December 2012 storm complex, the first wintry mix of the season.
- **22 Apr 2012:** a coastal storm with more than an inch of rain; a baseball game was postponed.
- **16 Apr 2011:** severe storms with four tornadoes in Maryland and Virginia, a 58 mph gust at Reagan airport and flooding at the Jefferson Memorial.
- **7 Dec 2011** and **6 Mar 2011:** all-day rain (21 and 9 hours of rain in the data). These two are explained by the in-data weather only; no external source was searched for them.

Four of these ten include a confirmed system shutdown.

**Five are calendar days that the dataset's `holiday` flag misses:**

- **Christmas Day 2011** fell on a Sunday, so the official holiday flag sits on the observed Monday, 26 December, not on the 25th itself.
- **Christmas Eve 2012**: federal offices were closed by executive order (announced 21 December 2012), but the dataset does not flag it as a holiday.
- **Christmas Day 2012** (flagged as a holiday, but still unusual).
- **The day after Thanksgiving in both 2011 and 2012.** These are not official holidays in the data, but the casual share was about 40 %, like a weekend. Many people take the day off.

### A spin-off finding: the holiday column is incomplete

The anomaly step found a real flaw in the dataset: some days behave exactly like holidays but are not marked as holidays. That pollutes every "normal demand" baseline. It suggests a new engineered feature, **`bridge_day`** (Christmas Eve, the day after Thanksgiving), which is planned for the next notebook update.

### The decision that follows: outage days are noise

The lecture distinguishes **noise** (data problems you remove) from **meaningful anomalies** (real phenomena you study) (slide 18). Days with a **system outage** are noise for this question: they reflect missing *supply* (bikes were locked away), not low *demand*. So they are **excluded** from the recovery analysis.

Every other low-demand anomaly is **kept**. Bad weather, and what happens after it, is exactly the phenomenon the project studies. Removing rainy anomalies would remove the signal.

### Figure 3 (anomalies)

A line chart of daily demand relative to normal (the log ratio) over two years, with the 15 consensus anomalies as orange dots and short labels: "Hurricane Sandy: system closed ~36 h", "Irene (closed 18:00)", "Halloween nor'easter", "tornado storms", "day after Thanksgiving" and so on. All dots lie below the zero line, and Sandy is by far the deepest drop.

---

## Part 8. Step 4: Feature engineering, building the "recovery clock"

### Why this is the heart of the project

The raw data only says **what the weather is right now**. The research question is about **what the weather was**. So the representation of the data has to change: from "a list of independent hours" to "time since an event". The course learning outcomes name this skill explicitly: changing the data representation to suit the problem. Without these features, the research question could not even be asked.

### The features

**1. `bad`:** 1 if the weather situation is 3 (rain or snow). This defines the event.

**2. `episode` and `dur_h`:** an episode is a maximal run of consecutive bad-weather hours. A four-hour shower is one event, not four separate rows. This turns 1,526 bad-weather hours into **492 episodes**. Most are short: the median episode is **2 hours**, the mean is 3.1 hours, and the longest is **51 hours**.

**3. `h_after`, "the recovery clock":** the number of hours since the last episode ended. The first dry hour is hour 0, the next is hour 1, and so on. **This is the single most important feature in the project.**

**4. `after_bin`:** the recovery clock grouped into bins: rain now, 0 h, 1 h, 2 h, 3 h, 4–6 h, 7–12 h, 13–24 h, and more than 24 h. Why bins instead of a straight number: a linear model given the raw number would be forced to assume demand recovers in a straight line. With bins, the model can learn **any recovery shape**, fast at first and then slow, for example. The bins are narrow early (hour by hour), where demand changes fast, and wider later, where it changes slowly. The ">24 h" bin is the reference: a normal dry hour.

**5. `prev_dur` and `prev_type`:** how long the last episode lasted, and whether it was rain or snow. This asks: does a long storm leave a longer hangover?

**6. The snow proxy:** weather situation 3 mixes light rain and light snow in one category. The group used the **mean temperature during the episode below 2 °C** as a proxy for snow or ice. This gives 470 rain episodes and 22 snow/ice episodes. It is clearly stated as a proxy, not a measurement, and it only works because the temperature was converted with the correct UCI formula.

**7. `bad_last6` and `bad_last24`:** the number of bad-weather hours in the previous 6 and 24 hours. This captures accumulated wetness: puddles and wet bike seats.

**8. `exp_*` and `ratio_*`, the expected demand and the demand ratio:** for every hour, the "normal" demand is the median for the **same year, month, day type and hour of the day**. The ratio is actual divided by normal (with +1 on both sides to avoid dividing by zero). A ratio of 1.0 means exactly normal; 0.7 means 30 % below normal.

  Why this matters: 8 a.m. on a working day normally has hundreds of rides, and 2 p.m. far fewer. Raw counts cannot be compared across hours, but ratios can. **The ratio puts every hour on the same scale.**

  An important detail: the "normal" median is computed **only from clean hours**: dry, more than 24 hours after any bad weather, not a holiday and not an outage. Otherwise, the baseline would itself be pulled down by post-rain hours, and the hangover would partly cancel itself out. For 48 hours, no clean reference existed, so a fallback median over all dry hours was used.

**9. `daypart`:** night, morning peak, midday, afternoon peak, evening. This captures commuter timing.

**10. `comfort_gap`:** feels-like temperature minus air temperature. It captures humid, sticky air after rain.

### Figure 4 (episodes)

Three histograms: episode lengths (most are 1 to 3 hours, with a long tail), the hour of day at which episodes end, and the number of dry hours until the next episode.

### Planned extra features

- **`bridge_day`:** a flag for holiday-like days the dataset misses (Christmas Eve, the day after Thanksgiving).
- **`ended_before_commute`:** does rain that stops at 6 a.m. (before the morning commute) differ from rain that stops at 9 a.m. (after it)?
- **`temp_drop`** (optional): does a cold front behind the rain slow the recovery?

---

## Part 9. Step 5: Dimension reduction with PCA

The project applies **Principal Component Analysis (PCA)** twice, for two different purposes.

### What PCA does, in plain words

PCA takes data with many columns and finds a few new axes, called "principal components", that capture as much of the variation as possible. The first component is the single direction along which the data varies most, the second is the next-most-important direction at right angles to the first, and so on. If two components capture most of the variation, the data can be drawn on a flat 2-D chart without losing much. Each component's **loadings** tell you which original columns it is made of, which is how you interpret what the axis "means".

### PCA 1: The shape of a day

Each day is described by **24 numbers**, its rides in each hour. Before PCA, each day is **divided by its own total**, so every row sums to 1 and describes the **shape** of the day, not its volume.

**Why shape and not volume?** Volume is mostly season and growth: summer days are big and winter days are small. If volume were included, PCA's first axis would just say "summer versus winter", which is already known. Dividing by the total removes that and leaves the interesting part: *when* in the day people ride.

**Why not standardise the columns?** Standard practice before PCA is often to give every column a variance of 1. Here, that would be harmful. The night hours (2 to 5 a.m.) have almost no rides, so their tiny variation is mostly noise. Standardising would blow that noise up to the same importance as the rush hours.

**Results:**

- The first two components keep **85 %** of the variation (PC1 74 %, PC2 11 %).
- **PC1 is the commuter-versus-leisure axis.** Its loadings are strongly positive at 08:00 and 17:00–18:00 (the commute peaks) and negative across the middle of the day (10:00 to 15:00, the leisure hours). PC1 correlates **−0.74** with the casual share of the day. In the scatter plot, working days sit on one side and weekends and holidays on the other, as two clearly separate clouds.
- **PC2 contrasts the morning peak with the evening peak.** Its loadings are positive around 08:00 and strongly negative around 17:00–19:00.
- **Rainy days do not form their own cluster.** When days with 3 or more hours of daytime rain are highlighted, they stay inside their own day-type cloud, mostly at the edges. Rain distorts a day's shape a little, but **the day type still decides the shape. Rain mainly changes the volume.**

That last point drives a design decision: since rain does not create a distinct daily shape, the recovery must be measured **hour by hour against a same-hour baseline**, not with whole-day shapes. The two clearly separated populations (commuters versus leisure riders) also justify modelling casual and registered riders separately.

**Figure 5** has three panels: the PCA scatter coloured by day type (blue working days on the right, orange weekends on the left), the same scatter with rainy days highlighted in green, and the loadings of PC1 and PC2 across the 24 hours.

### PCA 2: The shape of a recovery

The second PCA is applied to **recovery curves**. For each of 65 episodes with complete data, the demand ratio is taken for the first 7 hours after the rain, for both registered and casual riders (14 numbers per episode).

- **PC1 (46 %)** loads mainly on the **casual** ratios. So episodes differ first in **how deep casual demand falls**.
- **PC2 (16 %)** contrasts the first hours with hours 5–6. So it captures **how fast** demand recovers.

In other words, recoveries differ mostly in **depth** and secondarily in **speed**. **Figure 8** shows this scatter, with episodes of 4 hours or more marked separately from short ones.

---

## Part 10. Research question, method A: Episode recovery curves

This is the most direct, intuitive answer to the question: follow each rain episode and watch what demand does afterwards.

### The design and why each choice was made

For each episode, the demand ratio (actual ÷ normal for the same year, month, day type and hour) is tracked for **12 hours after the first dry hour**.

**Which episodes are used:**

- **At least 2 hours long.** A one-hour flag is often a single noisy weather reading, not a real shower.
- **At least 6 dry hours afterwards.** If a new shower starts, the clock restarts, and the "recovery" would be contaminated by new rain.
- **No outage in the window.** Missing supply is not low demand.
- **Ending between 06:00 and 19:00.** At night, "normal" demand is close to zero rides, so ratios are pure noise. If normal is 2 rides and 1 ride happens, that is "50 % of normal", which means nothing.

This leaves **93 of the 492 episodes**: 87 rain and 6 snow/ice.

**Night hours (00:00–05:00) inside the window are skipped**, for the same reason, and because no operational decision is taken at night. If bad weather returns within the window, the curve is cut off at that point.

**When is demand "recovered"?** When the ratio is **at least 0.9 for 2 consecutive daytime hours.**

- Why 90 %, not 100 %: demand varies naturally from hour to hour by roughly 10 % even on normal days. Demanding exactly 100 % would mean many episodes never "recover" just by chance.
- Why 2 hours in a row: so that one lucky hour does not count as a recovery.

**What if it never recovers within 12 hours?** Then the recovery time is recorded as "more than 12 hours" (technically, **censored**). The group did not invent a recovery time it could not see. For the statistical test, these are treated as 13 hours.

### The results

At the **first dry hour**, the median registered demand is at **71 %** of normal and the median casual demand at **54 %** of normal.

| | Registered | Casual |
|---|---|---|
| Median ratio at hour 0 | 0.71 | 0.54 |
| Median ratio at hour 2 | 0.86 | 0.57 |
| **Median recovery time** | **3 hours** | **11 hours** |
| Recovered within 1 hour | 33 % of episodes | 13 % |
| Recovered within 3 hours | 53 % | 30 % |
| Recovered within 6 hours | 69 % | 43 % |
| Not recovered within 12 hours | 23 % | 48 % |

Almost half of the time, casual demand has not recovered even 12 hours after the rain stopped.

**Figure 6** shows the median recovery curves, with shaded bands from the 25th to the 75th percentile (the interquartile range). The blue registered line starts at about 0.71, jumps to about 0.86 after one hour, crosses the dashed 90 % "recovered" line at about hour 4, and is around 1.0 by hours 10 to 12. The orange casual line starts at about 0.54, wobbles around 0.6 for the first two hours, climbs slowly to about 0.8 by hour 6, and only reaches about 0.9 around hours 10 to 12. The casual line is below the registered line at every single hour.

### Is the difference statistically real? The paired Wilcoxon test

The group compared the recovery times of casual and registered riders **on the same episodes**.

**Why "paired"?** Episodes differ a lot from one another. A heavy autumn storm hits everybody harder than a light summer shower. Comparing casual and registered riders *within* the same episode removes those episode-to-episode differences and isolates the difference between the rider groups.

**Why Wilcoxon and not a t-test?** The t-test assumes roughly normally distributed differences. Recovery times are small whole numbers (0, 1, 2, ..., 13), skewed, and capped at 13. The **Wilcoxon signed-rank test** makes no such assumption: it ranks the differences and checks whether the positive ones outweigh the negative ones.

**Results:**

- On the same episodes, casual riders were **slower in 56 %**, **equal in 32 %** and **faster in only 12 %**.
- The paired Wilcoxon test gives **p = 1.6 × 10⁻⁶**, about 1.6 in a million. If there were really no difference, a result this lopsided would almost never happen by chance.
- The **mean gap is +3.0 hours** with a **95 % bootstrap confidence interval of 1.8 to 4.1 hours**. The median gap is 2 hours (95 % interval 0 to 3 hours).

**What is a bootstrap confidence interval?** The bootstrap resamples the 93 episode differences with replacement 5,000 times and recomputes the mean each time. The middle 95 % of those 5,000 means gives the interval. It needs no distributional assumption. The interval for the mean gap does not include zero, so casual riders are genuinely slower.

### Sensitivity analysis: does the answer depend on the arbitrary choices?

Two choices were judgement calls: the **90 % threshold** and the **2-hour minimum episode length**. A conclusion that flips when you nudge those choices would not be a real finding. So the group re-ran the whole analysis for every combination of threshold (85 %, 90 %, 95 %) and minimum length (1, 2, 3 hours), nine settings in total.

| Threshold | Min. episode | Episodes | Registered median | Casual median | Wilcoxon p |
|---|---|---|---|---|---|
| 85 % | 1 h | 160 | 1 h | 6 h | 4 × 10⁻¹³ |
| 85 % | 2 h | 93 | 2 h | 10 h | 6 × 10⁻⁹ |
| 85 % | 3 h | 53 | 2 h | 13 h | 2 × 10⁻⁶ |
| 90 % | 1 h | 160 | 2 h | 8 h | 8 × 10⁻¹¹ |
| **90 %** | **2 h** | **93** | **3 h** | **11 h** | **1.6 × 10⁻⁶** |
| 90 % | 3 h | 53 | 3 h | 13 h | 1 × 10⁻⁴ |
| 95 % | 1 h | 160 | 3.5 h | 12 h | 2 × 10⁻⁸ |
| 95 % | 2 h | 93 | 5 h | 13 h | 3 × 10⁻⁴ |
| 95 % | 3 h | 53 | 5 h | 13 h | 0.002 |

**In all 9 settings, casual riders are slower**, with registered medians of 1 to 5 hours, casual medians of 6 to 13 hours, and every p-value at or below 0.002. The finding is robust.

### Splits: when is the hangover longest?

**Figure 7** splits the recovery curves three ways:

- **By episode length.** After **long episodes (4 hours or more)**, casual demand starts much lower (41 % of normal versus 58 %) and recovers in **13 hours versus 6 hours**. Registered riders recover in 3 hours either way. The interpretation: **a commute has to happen, and a leisure trip can be dropped.** After a long storm, commuters still need to get home, but leisure riders simply decide not to go out today.
- **By day type.** On weekends, registered riders *seem* slower (median 10.5 hours). But this rests on only 20 episodes, and the regression check (Model D, below) did **not** confirm it. The group treats it as small-sample noise and does **not** claim it as a finding.
- **By rain versus snow.** Snow/ice episodes look slower for casual riders (13 hours), but with only 6 snow episodes this is only a hint.

### The limitation that motivates method B

The baseline here is a monthly median. But right after rain the air is still humid and often misty. Maybe people are not avoiding bikes *because it just rained*. Maybe they are avoiding them *because it is still humid and grey right now*. Method A cannot tell these apart. Method B can.

---

## Part 11. Research question, method B: Regression on lagged weather

### The question regression answers

Is the "hangover" real, **after controlling for the current weather?** The regression separates two explanations:

- **"The rain is still in effect":** people stay away because it rained recently (wet roads, wet seats, changed plans, habit).
- **"It is still unpleasant now":** people stay away because the air is still humid or misty right now.

If the recovery-clock effects survive after the model already knows the current humidity, mist, temperature and wind, then the hangover is real.

### The models

All models predict **log(1 + count)**, separately for registered and casual riders.

**Why the log?** Counts are right-skewed, and weather effects are **multiplicative**: rain cuts demand by a percentage, not by a fixed number of rides. Rain removes many more rides at 5 p.m. than at 3 a.m., but roughly the same *percentage*. On the log scale, a coefficient β translates directly to a percentage change of (e^β − 1) × 100 %. The "+1" handles hours with zero rides.

- **Model A (current weather only):** calendar terms (a separate level for each hour of the day on working days and on days off, plus month, year and holiday), plus current weather (mist, rain now, temperature and temperature squared, humidity, wind). Temperature squared allows "too hot" to reduce demand, not just "too cold".
- **Model B (Model A + recovery clock):** the same, plus the `after_bin` categories (0 h, 1 h, 2 h, 3 h, 4–6 h, 7–12 h, 13–24 h after the rain), measured against the reference, a normal dry hour more than 24 hours after any rain.
- **Model C (robustness: non-linear humidity):** Model B, but humidity is replaced by **7 humidity bands, each with its own temperature slope**, so that muggy heat and cold damp air can have different effects.
- **Model D (robustness: days off):** Model B plus an extra term for each recovery bin that is switched on only on non-working days. Does the recovery differ on weekends?

### HAC standard errors: why ordinary p-values would lie

Ordinary regression assumes each observation is independent. Hourly data is not: if this hour has more rides than the model predicts, the next hour probably does too. This is called **autocorrelation**. With autocorrelated errors, ordinary standard errors are **far too small**, and almost everything looks significant.

The fix is **HAC standard errors** (Heteroskedasticity and Autocorrelation Consistent, also called **Newey–West**, after Newey and West, 1987). They widen the confidence intervals to account for errors that are correlated over up to **24 hours** (one full day). Every p-value and confidence interval in the regression uses HAC.

### Results of Model B: the effect at each hour after rain

Percentage change in rides compared with a normal dry hour, **after controlling for current weather and calendar**:

| Time since rain stopped | Registered | Casual |
|---|---|---|
| Raining now | −44 % | −52 % |
| **0 h (first dry hour)** | **−25 %** | **−38 %** |
| 1 h | −15 % | −26 % |
| 2 h | **−9.5 %** | −20 % |
| 3 h | −7 % | −18 % |
| 4–6 h | −5 % | −11 % |
| 7–12 h | −1 % (not significant) | **−9 %** |
| 13–24 h | +1 % (not significant) | **−5 %** (p = 0.035) |

Reading this:

- **Registered riders** are 25 % below normal in the first dry hour, already within 10 % of normal at **2 hours**, and indistinguishable from normal **from 7 hours on**.
- **Casual riders** are 38 % below normal in the first dry hour, stay more than 10 % below normal through the 4–6 hour bin, are just within 10 % at **7–12 hours**, and are **still measurably below normal 13 to 24 hours later.**

So with the same "within 10 % of normal" yardstick, **registered riders recover in about 2 hours and casual riders in about 7 to 12 hours.**

The regression gives shorter recovery times than the episode curves (2 h versus 3 h for registered, 7–12 h versus 11 h for casual). That is expected: the regression removes the part of the drop that is due to lingering humidity, mist and season. **It is the more conservative estimate**, and the conclusion is the same.

**Model fit:** the models explain a lot of the variation (adjusted R² about 0.93 for registered and 0.85 for casual), and the **AIC** (a measure of fit that penalises extra complexity; lower is better) clearly prefers Model B over Model A for both groups: registered 16,067 → 15,773, casual 30,869 → 30,503. Adding the recovery clock genuinely improves the model.

**Figure 9** shows these effects as dots with 95 % HAC confidence-interval error bars, blue for registered and orange for casual, across the bins from "raining now" to 13–24 h. Both start deep below zero; the blue dots reach zero by 7–12 h; the orange dots are still below zero at the far right.

### Robustness check, Model C: is it just the humidity?

The weather in each recovery bin shows why this check is needed:

| | Rain now | 0 h | 1 h | 2 h | 3 h | 4–6 h | 7–12 h | 13–24 h | >24 h |
|---|---|---|---|---|---|---|---|---|---|
| Humidity % | 84 | 81 | 79 | 77 | 75 | 73 | 66 | 57 | 57 |
| Share of hours with mist | n/a | 79 % | 69 % | 62 % | 46 % | 40 % | 28 % | 20 % | 24 % |

Right after rain, humidity is high and the sky is usually misty, and both fade slowly. So the hangover *could* just be lingering humidity.

Model C lets humidity have a fully flexible, non-linear effect. If the hangover were just humidity, the recovery-clock effects would shrink towards zero. They barely change: **every effect moves by less than 2 percentage points** (for example, casual at 0 h goes from −37.8 % to −36.0 %, and registered at 2 h from −9.5 % to −8.9 %). **The hangover is not just the humidity.**

### Robustness check, Model D: is it about trip purpose?

The episode curves hinted that registered riders recover slowly on weekends. If true, that would suggest the hangover is about **why** people ride (commuting versus leisure), not **who** rides.

Model D adds a separate "extra effect on days off" for each recovery bin. Result: **no clear difference.**

- None of the registered day-off terms is significant (all p > 0.2).
- For casual riders, only one bin out of eight is significant (1 h after rain, −16 %, p = 0.02).
- Across 16 tests in total, getting one with p ≈ 0.02 is about what chance alone produces.

**Conclusion:** the weekend hint from the episode view was small-sample noise. The group does **not** claim a trip-purpose effect. The robust finding is the difference **between the rider groups**. This shows the habit of testing your own hints before presenting them, which makes good material for the "challenges" slide.

### Prediction: does the recovery clock improve forecasts?

Explaining the past is one thing. A company needs **forecasts**. So the group checked whether adding the recovery clock improves predictions on data the model has never seen.

**Chronological split.** The models are trained on January 2011 to June 2012 (13,075 hours) and tested on **July to December 2012** (4,380 hours). Why not a random split? With hourly data, a random split would put 3 p.m. in the training set and 4 p.m. on the same day in the test set. Because neighbouring hours are so similar, the model would effectively see the answer, and the test score would be falsely good. A chronological split mimics real forecasting: learn from the past, predict the future.

**The right metric: error on post-rain hours.** The models are scored with:

- **MAE** (mean absolute error): the average size of the error, in rides per hour. Easy to explain.
- **RMSE** (root mean squared error): like MAE, but punishes big misses more.
- **RMSLE** (root mean squared log error): error on the log scale, so relative errors on small counts count too.

Crucially, these are computed **both on all test hours and on the post-rain hours only** (0 to 6 hours after rain). Post-rain hours are only **596 of the 4,380 test hours, about 14 %**. An improvement on them gets diluted to almost nothing in an all-hours average. Measuring only on all hours would **hide** exactly the effect the project studies. This is the "right evaluation metric" choice the grading criteria ask for.

**Linear model results (registered riders):**

- **Post-rain MAE: 38.5 → 34.3 rides per hour (11 % better).**
- **Post-rain RMSE: 67.4 → 57.0 (15 % better).**
- All-hours MAE barely changes (40.15 → 40.27).

This is exactly the pattern expected: the recovery clock matters a lot in the hours it is about, and those hours are invisible in the overall average. For casual riders, counts per hour are small and the differences (±0.3 rides per hour) are within noise.

### An honest negative result: gradient boosting does not gain

The group also trained a **gradient boosting** model (a flexible machine-learning model made of many decision trees), with and without the lag features, to check that the result is not just an artefact of the linear model.

**Surprise:** gradient boosting did **not** improve with the lag features. Registered post-rain MAE actually got slightly worse (35.6 → 38.1), and across a 5-fold expanding-window time-series cross-validation it was worse by about 20 % on average (with high variation between folds).

Instead of hiding this, the group investigated **why**. They looked at the errors of the models that only see the current weather, split by recovery bin:

- The **linear model** without the clock **over-predicts every post-rain bin**, most strongly at the first dry hour (mean log residual about −0.37, against about −0.10 in normal hours).
- The **gradient boosting model** without the clock is **already almost unbiased from 1 hour after the rain onward**.

The explanation: **the trees figure out "it has just rained" from the current humidity and mist.** As the table above shows, in the first dry hour the sky is misty 79 % of the time (versus 24 % normally) and humidity is 81 % (versus 57 %). A flexible model can combine "high humidity + mist + this hour + this season" into a reliable "just rained" signal on its own. Explicit lag columns then add nothing new, only extra variance.

This does **not** contradict the hangover. It supports a practical conclusion: **a forecast must see the post-rain state somehow.** Either use a simple model with an explicit "hours since rain" feature, or a flexible model that can read the post-rain state from humidity and mist. "Hours since rain" is the version a dispatcher can actually read, and a weather forecast provides it directly.

---

## Part 12. Research question, method C: Association rules

### Why a third method

Regression gives an **average** effect per hour bin. It does not easily reveal **combinations**, for example "one hour after a long storm, at midday, on a cold weekend". Association rules find such combinations automatically and express them as readable **if-then rules** that a dispatcher can use directly, without the analyst guessing the interactions in advance.

### How association rule mining works, in plain words

Association rule mining was invented for supermarket baskets: "people who buy bread and butter also buy milk". Each shopping basket is a **transaction**, each product is an **item**, and a rule says "if these items, then that item".

In this project, **each transaction is one dry daytime hour** (07:00 to 21:00, no outage), and the items describe that hour:

- time since rain (0–1 h, 2–3 h, 4–6 h, 7–12 h, 13–24 h, more than 24 h);
- the previous episode's length (short or long, 4 hours or more) and whether it was snow/ice;
- part of the day (morning peak, midday, afternoon peak, evening);
- day type (working or off);
- temperature band (cold below 10 °C, mild 10–20, warm 20–28, hot above 28);
- high humidity (80 % or more);
- sky (mist or clear);
- season;
- and the outcome: **casual demand** and **registered demand**, each as **low (below 0.7 × normal)**, **normal**, or **high (above 1.3 × normal)**.

This gives **10,000 transactions** with **33 possible items**.

**Why fixed bands (0.7 and 1.3) instead of data-driven quantiles?** Because "30 % below normal" is a sentence a business person understands immediately. A quantile-based band such as "below the 23rd percentile" is not.

### Support, confidence, lift and conviction

Take a rule **A ⇒ B**, for example "{0–1 h after rain} ⇒ {registered = low}".

- **Support:** how often A and B happen together, as a share of all transactions. It measures how common the rule is.
- **Confidence:** of the hours where A is true, how many also have B. "When it has just stopped raining, how often is registered demand low?"
- **Lift:** confidence divided by how common B is anyway. Lift = 1 means A tells you nothing about B. Lift = 4 means B is four times more likely than usual when A holds.
- **Conviction:** how much more often the rule would be wrong if A and B were unrelated. Higher is stronger.

**Why rank by lift and not by confidence? This is a key methodological point.** The two outcomes have very different base rates:

- **"casual = low" is true in 24.5 % of all dry daytime hours** (casual demand is volatile).
- **"registered = low" is true in only 9.5 %** (commuters are steady).

So a casual rule with a confidence of 0.40 sounds strong but is only a lift of about 1.6, barely better than the base rate. A registered rule with the same confidence of 0.40 is a lift of 4.2, a very strong signal. **Ranking by confidence would systematically favour casual rules simply because casual demand is low more often in general.** Lift corrects for this.

### Algorithm settings

The algorithm is **Apriori**, from the course lecture (Agrawal and Srikant, 1994). Apriori finds all item combinations that appear often enough, using the fact that if a combination is rare, every larger combination containing it is also rare, which saves a huge amount of work.

- **Minimum support 0.003** (about 30 hours). Post-rain hours are rare by nature, so a higher threshold would hide every rule the project cares about.
- **At most 4 items per rule**, to keep rules readable.
- Only rules whose "then" side is a single demand outcome, whose "if" side contains no demand outcomes, and whose "if" side contains a recent time-since-rain item are kept.

### Results: the simplest rules mirror the regression

**One-item rules** of the form "{time since rain} ⇒ {low demand}":

| If | Then | Confidence | Lift |
|---|---|---|---|
| 0–1 h after rain | registered = low | 0.40 | **4.2** |
| 2–3 h after rain | registered = low | 0.18 | 1.9 |
| 4 h or more after rain | registered = low | no rule (lift ≤ 1 or too rare) | n/a |
| 0–1 h after rain | casual = low | 0.63 | **2.5** |
| 2–3 h after rain | casual = low | 0.49 | 2.0 |
| 4–6 h after rain | casual = low | 0.39 | 1.6 |
| 7–12 h after rain | casual = low | 0.29 | 1.2 |
| 13–24 h after rain | casual = low | 0.26 | 1.05 |

The registered rules **disappear after 3 hours**, and the casual rules **keep a lift above 1 all the way to 12 to 24 hours.** Two completely different methods, regression and rule mining, give the same picture: **commuters are back within about 3 hours; casual riders take about half a day.**

### The strongest combined rules

- **{0–1 h after rain, previous episode 4 h or more, midday} ⇒ casual = low**: confidence **0.90**, lift 3.7. Right after a long storm, at midday, casual demand is low 9 times out of 10.
- **{0–1 h after rain, high humidity, afternoon peak} ⇒ registered = low**: confidence 0.65, lift **6.9**. Even commuters stay away right after rain at the evening rush if it is still very humid.
- **{4–6 h after rain, day off, cold} ⇒ casual = low**: confidence 0.83, lift 3.4. On a cold weekend, the casual hangover lasts at least 6 hours.

### Stability: do the rules hold in a new year?

A rule that a company relies on must hold in the future, not just in the data it was found in. So the group **mined the rules on 2011 only** and then **re-measured the same rules on 2012**.

- **All 20 top rules from 2011 still have lift above 1.2 in 2012.**
- The median lift dropped from 6.6 to 5.7. Confidence fell somewhat, because the system grew and base rates changed, but **the direction holds for every rule.**

These rules can be used operationally.

---

## Part 13. Business translation: how many rides are lost after the weather clears?

### Why this step

The grading criteria put particular weight on "the gain for the bike rental company". Percentages and p-values are not something a manager can act on directly. Lost rides are.

### How it is computed: a counterfactual

A **counterfactual** asks "what would have happened if things had been different?" Here, every hour during and after bad weather is predicted **twice** with Model B:

1. **As it happened:** with its real recovery bin (for example, "1 hour after rain").
2. **As if it were a normal dry hour:** with the recovery bin set to "more than 24 hours", while keeping **the current temperature, humidity, mist and wind exactly the same.**

The difference between the two predictions is the number of rides lost **purely because bad weather happened recently**, not because of the current conditions. The same is done for the rainy hours themselves, for comparison. The totals over two years are divided by 2 to give rides per year.

### The result

Rides lost per **year** to bad weather:

| | Registered | Casual | Total |
|---|---|---|---|
| During rain or snow | 47,000 | 9,800 | 56,800 |
| After it stopped: 0–6 h | 21,600 | 7,400 | 29,000 |
| After it stopped: 7–24 h | about 0 | 4,400 | 3,800 |
| **Share lost after the weather cleared** | **31 %** | **55 %** | **37 %** |

So **about 32,700 of about 89,500 weather-related lost rides per year, 37 %, are lost after the rain or snow has stopped.** For casual riders, **more than half** of their weather losses happen after the weather has cleared. For context, the system had about 1.65 million rides per year on average, so the post-weather losses are around 2 % of all rides. That is not huge, but it is concentrated in specific, predictable hours, which is exactly what makes it actionable.

A forecast that looks only at the current weather **cannot see any of these lost rides.**

### Recommendations for Capital Bikeshare

1. **Forecast with a recovery clock.** Add "hours since rain or snow ended" to the demand forecast (or use a flexible model that sees humidity and mist). Without it, post-rain registered demand is over-predicted, and the linear model's post-rain error is 11 % higher.

2. **Time the rebalancing to the commuters.** Commuter stations need their normal stock of bikes about **2 hours after the rain stops**, not the moment it stops. Leisure stations (the National Mall, the waterfront) stay under-used for **7 to 12 hours**. That window is free truck time for moving bikes and for maintenance.

3. **Promote where the hangover is longest.** The rules point to **weekends, cold days and the hours after long storms** (for example, "{0–1 h after, long episode, midday} ⇒ casual low" with confidence 0.90). That is the slot for an app notification such as "The sun is out: 24-hour pass discount". It targets the roughly 11,800 casual rides a year that are lost after the weather has already cleared.

4. **Fix the holiday calendar.** Christmas Eve 2012 and the days after Thanksgiving behave like holidays but are not flagged. A `bridge_day` feature would remove these false anomalies from every model.

The results are in **rides, not money**. Converting to revenue would need a reliable source for 2011–2012 prices. The group deliberately decided **not to guess prices**.

---

## Part 14. How the three methods fit together

The core strength of the project is **triangulation**: three independent methods, each with different assumptions, reach the same answer.

| | Method A: Episode curves | Method B: Regression | Method C: Association rules |
|---|---|---|---|
| Unit of analysis | 93 rain episodes | 17,455 hours | 10,000 dry daytime hours |
| What it controls for | year, month, day type, hour (baseline) | plus current temperature, humidity, mist, wind | combinations of conditions |
| Registered recovery | median 3 h | about 2 h (within 10 %) | rules vanish after 3 h |
| Casual recovery | median 11 h | about 7–12 h | lift above 1 up to 12–24 h |
| Statistical evidence | paired Wilcoxon p = 1.6 × 10⁻⁶; 9 of 9 sensitivity settings | HAC confidence intervals; AIC; Model C robust | 2011 rules hold in 2012 |
| Strength | intuitive, easy to show | separates the hangover from lingering humidity | readable rules for operations |

If only one method had been used, an examiner could say "that is an artefact of your method". Because three different lenses give the same answer, that objection does not hold.

---

## Part 15. The decision log: what was chosen, what was rejected, and why

| # | Decision | Alternative rejected | Why |
|---|---|---|---|
| D1 | Use hourly data | Daily data | Recovery happens within hours; daily totals hide it. |
| D2 | Short night gaps = 0 rides; long gaps (6 h or more) = outage, left out | Impute all gaps, or drop all | Shutdowns are confirmed by news; zero-ride hours are real demand. |
| D3 | UCI temperature formula | Readme formula (÷41) | The Readme formula gives no frost in two D.C. winters. |
| D4 | Detect anomalies on demand ratios, explain with weather | Detect on all variables | Otherwise the detector flags weather, and nothing independent is left to explain it. |
| D5 | Four detectors and a vote | A single detector | Agreement between detectors is low (Jaccard 0.15–0.58). |
| D6 | Episode = at least 2 h of bad weather, then at least 6 dry hours | Every bad hour | Single-hour flags are noisy; a new shower restarts the clock. |
| D7 | Baseline = same year × month × day type × hour, from clean hours only | An overall mean | The system grew; there is a strong hour and season pattern. |
| D8 | "Recovered" = at least 90 % for 2 consecutive hours | 100 %; or 1 hour | Normal hour-to-hour noise is about 10 %; one lucky hour is not a recovery. Holds for 85/90/95 % × 1/2/3 h. |
| D9 | Paired Wilcoxon test + bootstrap | t-test | Recovery times are skewed whole numbers; the same episodes are compared. |
| D10 | Separate models for casual and registered | One model on the total | Two different populations (profiles, PCA). |
| D11 | log(1 + count) target, OLS with HAC standard errors | Plain OLS standard errors; Poisson | Effects are multiplicative; errors are autocorrelated. |
| D12 | Chronological split + time-series cross-validation | Random K-fold | Random folds leak the future through autocorrelation. |
| D13 | Metric = MAE/RMSE on post-rain hours | All-hours MAE only | The effect is invisible in the all-hours average. |
| D14 | Rules ranked by lift; mined on 2011, checked on 2012 | Rank by confidence; no hold-out | Base rates differ by rider type; operational rules must hold next year. |
| D15 | Fixed demand bands (0.7 / 1.3 × normal) | Quantile bands | Business-readable: "30 % below normal". |
| D16 | Report the gradient-boosting non-gain | Hide it | It is honest, and it explains something real: the trees read the post-rain state from humidity and mist. |

---

## Part 16. Limitations and honest caveats

A good project states its own weaknesses before the examiner does.

1. **Weather situation 3 mixes light rain and light snow.** The rain-versus-snow split relies on a temperature proxy (below 2 °C), and there are only 22 snow episodes in total and 6 in the episode analysis. Snow results are called "a hint", not a finding.
2. **Few long episodes.** Only 34 of the 93 analysed episodes last 4 hours or more. The number of episodes is reported next to every split. The main claims rest on all 93 episodes and on the regression over 17,455 hours.
3. **The hangover could be lingering humidity.** Addressed by Model C: the effects change by less than 2 percentage points.
4. **Autocorrelation can make p-values too small.** Addressed with HAC standard errors, the paired episode test, and the 2011 → 2012 hold-out for the rules.
5. **The linear model is biased in the test period.** In July–December 2012, the current-weather-only linear model over-predicts even normal hours by about 10 %, probably because of the growth trend. The post-rain over-prediction is clearly larger than that general bias, so the conclusion holds, but the bins should be compared with each other, not with zero.
6. **Results are in rides, not revenue.** No reliable price source for 2011–2012 was found yet, and prices are not guessed.
7. **One city, two years.** The findings describe Washington D.C. in 2011–2012. Other cities, or the same city today, might differ.
8. **Two outages are unconfirmed.** The gaps on 18 January and 22 February 2011 could not be matched to news reports. They are treated like the confirmed outages because they look the same.
9. **The weekend effect is not claimed.** The episode view suggested registered riders recover slowly on weekends, but the regression did not confirm it, so it is reported as noise.

**Challenges faced** (for the presentation): the wrong temperature formula in the Readme; telling outages apart from zero-ride hours; rain and snow mixed in one weather category; autocorrelation in hourly data; and a promising-looking weekend effect that turned out to be noise.

---

## Part 17. Questions an examiner is likely to ask, with answers

**Q: Why not simply drop the missing hours?**
Because the research question counts hours since the rain stopped. Dropping rows would make "the next row" sometimes be several hours later, and the recovery clock would be wrong. Also, the missing hours are not all the same: night gaps are real zero demand, and storm gaps are system shutdowns confirmed in the news, matching to the hour.

**Q: How do you know the Readme's temperature formula is wrong?**
With it, the coldest hour in two Washington winters would be +0.8 °C, and a snowstorm would have happened at 9 °C. The UCI page formula gives −7 °C and 2 °C, which is plausible.

**Q: Why did you leave the weather out of the anomaly detector?**
So it could serve as an independent explanation. If weather goes into the detector, the detector flags unusual weather, and you cannot then use weather to "explain" the anomaly without going in a circle.

**Q: Why four detectors?**
The lecture's taxonomy shows that each family defines "anomaly" differently. Their agreement was low (Jaccard 0.15 to 0.58), so a vote of at least 2 out of 4 is more reliable than any single method.

**Q: Why use a ratio to normal instead of raw counts?**
Normal demand at 8 a.m. on a working day is many times higher than at 2 p.m. The ratio puts all hours on one scale, so "30 % below normal" means the same thing at any hour.

**Q: Why 90 % as the recovery threshold? Isn't that arbitrary?**
It is a judgement call, matched to normal hour-to-hour variation of about 10 %. That is exactly why the sensitivity analysis exists: at 85 %, 90 % and 95 %, combined with three minimum episode lengths, casual riders are slower in all nine settings.

**Q: Why the Wilcoxon test and not a t-test?**
Recovery times are skewed whole numbers capped at 13 hours, so the t-test's normality assumption does not hold. And the comparison is paired (the same episodes for both groups), which removes episode-to-episode differences.

**Q: Couldn't the slow recovery just be because it is still humid after rain?**
That is why Model C exists. It gives humidity a fully flexible effect, and the recovery effects change by less than 2 percentage points.

**Q: Why didn't gradient boosting improve with your features? Doesn't that undermine your finding?**
No. The trees already read "it has just rained" from the current humidity and mist, which stay high after rain. The residual analysis shows the current-weather-only boosting model is already almost unbiased from 1 hour after the rain on. The practical lesson is the same: a forecast must see the post-rain state, either through an explicit clock or through humidity and mist.

**Q: Why evaluate only on post-rain hours?**
Post-rain hours are only 14 % of the test set. An 11 % improvement on them becomes almost invisible in an all-hours average. Measuring on the hours the question is about is the right metric.

**Q: Why rank association rules by lift?**
Because "casual = low" has a base rate of 24.5 % and "registered = low" only 9.5 %. Confidence alone would favour casual rules just because casual demand is low more often in general. Lift corrects for the base rate.

**Q: Why train on the past and test on the future instead of a random split?**
Neighbouring hours are very similar. A random split would let the model "see" the hours around each test hour, which is information leakage, and would give a falsely good score. A chronological split mimics real forecasting.

**Q: What is the single most important feature you engineered?**
The recovery clock: hours since the bad weather ended. It changes the data from "independent hours" to "time since an event", and the research question cannot be asked without it.

**Q: What should the company actually do differently?**
Add "hours since rain" to the forecast, restock commuter stations about 2 hours after rain stops rather than immediately, use the 7–12-hour lull at leisure stations for truck moves and maintenance, and push casual promotions in the hours after long storms, especially on cold weekends.

---

## Part 18. Glossary

- **Capital Bikeshare:** the public bike-sharing system of Washington D.C.
- **Registered rider:** a member with an annual or monthly subscription; mostly commuters.
- **Casual rider:** a pay-per-trip or day-pass user; mostly tourists and leisure riders.
- **`weathersit`:** the weather situation category, from 1 (clear) to 4 (heavy rain or snow). "Bad weather" in this project means category 3 (with 4 merged into it).
- **Episode:** a run of consecutive bad-weather hours.
- **Recovery clock (`h_after`):** hours since the last episode ended; 0 is the first dry hour.
- **Demand ratio:** actual demand divided by normal demand for the same year, month, day type and hour.
- **Outage:** a period when the bike system was shut down, so demand was not observed.
- **Robust z-score:** the 3-sigma idea using the median and MAD instead of the mean and standard deviation.
- **HBOS:** Histogram-Based Outlier Score; flags values in rare histogram bins.
- **kNN anomaly score:** distance to the k-th nearest neighbour.
- **LOF:** Local Outlier Factor; compares a point's local density with its neighbours' density.
- **Jaccard index:** overlap between two sets: size of the intersection divided by size of the union.
- **PCA:** Principal Component Analysis; finds a few axes that capture most of the variation.
- **Loading:** how much each original column contributes to a principal component.
- **Censored observation:** a value only known to be above some limit (here, "not recovered within 12 hours").
- **Paired Wilcoxon signed-rank test:** a non-parametric test of whether paired differences lean in one direction.
- **Bootstrap:** resampling the data with replacement many times to estimate uncertainty without distributional assumptions.
- **OLS:** ordinary least squares linear regression.
- **log1p:** log(1 + x); handles zeros and turns multiplicative effects into additive ones.
- **Autocorrelation:** errors in neighbouring hours are related to each other.
- **HAC / Newey–West standard errors:** standard errors that stay valid under autocorrelation.
- **AIC:** Akaike Information Criterion; model fit penalised for complexity; lower is better.
- **Gradient boosting:** a model built from many small decision trees, each correcting the errors of the previous ones.
- **MAE / RMSE / RMSLE:** mean absolute error; root mean squared error; root mean squared log error.
- **Chronological split / TimeSeriesSplit:** train on earlier data, test on later data, to avoid leaking the future.
- **Association rule (A ⇒ B):** "if A, then B", found from co-occurrence in transactions.
- **Support, confidence, lift, conviction:** how common a rule is; how often it is right; how much better than chance it is; how much more often it would fail if A and B were unrelated.
- **Apriori:** the classic algorithm for finding frequent item combinations.
- **Counterfactual:** a model prediction of what would have happened under a different condition.
- **Bridge day:** a day that behaves like a holiday but is not an official holiday (for example, the day after Thanksgiving).

---

## Part 19. Guide to the figures

| Figure | File | What it shows |
|---|---|---|
| 1 | `fig01_daily_timeseries.png` | Daily rides 2011–2012 for both rider types with 7-day averages; seasonality, growth and outage days. |
| 2 | `fig02_profiles_weather.png` | Hourly profiles on working days and weekends; mean rides by weather type. |
| 3 | `fig03_anomalies.png` | Daily demand relative to normal with the 15 labelled consensus anomalies. |
| 4 | `fig04_episodes.png` | Histograms of episode length, the hour episodes end, and dry hours until the next episode. |
| 5 | `fig05_pca_day_shapes.png` | PCA of 24-hour day shapes: working days versus weekends, rainy days highlighted, loadings. |
| 6 | `fig06_recovery_curves.png` | Median recovery curves after rain for both rider types, with interquartile bands and the 90 % line. |
| 7 | `fig07_recovery_splits.png` | Recovery curves split by episode length, day type and rain versus snow. |
| 8 | `fig08_pca_recovery_curves.png` | PCA of recovery curves: depth (PC1) versus speed (PC2). |
| 9 | `fig09_regression_recovery_effect.png` | Regression effect per hour after rain, with 95 % HAC confidence intervals. |

The suggested presentation (15 slides): title; question and why it matters; data and three cleaning surprises; anomalies (figure 3); from weather rows to episodes (figure 4); PCA showing two rider populations (figure 5); recovery curves (figure 6); paired test and splits (figure 7); regression effect (figure 9); "is it just humidity?" (Model C and the gradient-boosting finding); three rules and their stability; lost rides; recommendations; limitations and challenges; questions.

---

## Part 20. Where the project stands (as of 11 October 2026)

**Done:** cleaning with a nine-item log; descriptive statistics; four-detector anomaly detection with all 15 consensus anomalies explained; the core engineered features; both PCAs; all three research-question methods, with the sensitivity table and robustness Models C and D; and the business translation.

**Still to do:**

- add the `bridge_day` and `ended_before_commute` features and re-run;
- fill in the slide-number citations for the lectures on preprocessing, PCA, regression, association rules and evaluation (only the anomaly-detection lecture citations are complete, and missing citations can mean a failing grade);
- confirm the two unconfirmed outages in news archives;
- write the 3-page report and the 15-slide presentation;
- prepare Supplementary A (LLM screenshots) and Supplementary C (contribution split);
- rehearse the presentation to under 14½ minutes.

**Key dates:** progress check with the professor on Monday 12 October; submission by Sunday 25 October at 23:59; presentations on 26 and 27 October.

---

## Part 21. References

**Course literature**

- Tan, P.-N., Steinbach, M., Karpatne, A. and Kumar, V. (2018). *Introduction to Data Mining*, 2nd edition, Pearson. Chapter 2 (data quality and preprocessing), Chapters 5–6 (association analysis), Chapter 9 (anomaly detection), Chapter 10 (avoiding false discoveries), Appendix B (dimensionality reduction), Appendix D (regression).
- Witten, I. H., Frank, E., Hall, M. A. and Pal, C. J. (2016). *Data Mining: Practical Machine Learning Tools and Techniques*, 4th edition, Morgan Kaufmann. Section 4.5 (association rules), Section 4.6 (linear models), Chapter 5 (evaluation), Chapter 8 (data transformations).
- Fanaee-T, H., *Anomaly Detection (1)*, course lecture slides: definition (slide 3), anomaly versus noise (18), output as a ranking or top-N % (19), method taxonomy (20), Gaussian 3-sigma (27), HBOS (42–49), kNN (53–64), LOF (65–90), comparison of methods (116–117).

**Papers**

- Fanaee-T, H. and Gama, J. (2013). Event labeling combining ensemble detectors and background knowledge. *Progress in Artificial Intelligence*, Springer. (The dataset citation.)
- Agrawal, R. and Srikant, R. (1994). Fast algorithms for mining association rules. *VLDB*.
- Breunig, M. et al. (2000). LOF: identifying density-based local outliers. *SIGMOD*.
- Goldstein, M. and Dengel, A. (2012). Histogram-based outlier score (HBOS). *KI-2012*.
- Ramaswamy, S., Rastogi, R. and Shim, K. (2000). Efficient algorithms for mining outliers from large data sets. *SIGMOD*.
- Newey, W. and West, K. (1987). A simple, positive semi-definite, heteroskedasticity and autocorrelation consistent covariance matrix. *Econometrica* 55(3).
- Wilcoxon, F. (1945). Individual comparisons by ranking methods. *Biometrics Bulletin* 1(6).

**External evidence used to explain anomalies and outages**

- Hurricane Sandy shutdown (October 2012): ARLnow, WTOP.
- Hurricane Irene shutdown (August 2011): DCist, Washington Times.
- The 26 January 2011 thundersnow storm: NOAA Weather Prediction Center storm review, NPR.
- The 2011 Halloween nor'easter; the late-December 2012 storm complex; the 22 April 2012 coastal storm (NWS monthly summary); the 16 April 2011 severe storms (NWS Sterling event page).
- Christmas Eve 2012 federal closure: GovExec, FedSmith.
- Definition of "calm" wind: American Meteorological Society glossary; National Weather Service ASOS wind sensor page.
- Temperature formula: the UCI Machine Learning Repository page for the Bike Sharing Dataset.

Explain this data mining project to a university student who has to present and defend it. Focus on the reasoning behind each decision, why alternatives were rejected, and how the three methods support each other. Explain statistical terms simply, with the project's own numbers as examples. End with the likely examiner questions.