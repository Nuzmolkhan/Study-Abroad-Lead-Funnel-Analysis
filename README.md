# Study Abroad Lead Funnel Analysis

Lead acquisition, conversion, and counselor performance analysis across **1,200 study-abroad leads** (Jan 2025 – Jul 2026), built as an end-to-end analytics project: data cleaning → exploratory analysis → composite channel scoring → an interactive 6-tab dashboard.

**[View the live interactive dashboard →](https://app.hex.tech/019ff178-c7f3-757b-b506-c32f40e9b1b1/app/Study-Abroad-Lead-Funnel-Analysis-0345MtN10MD65gvJe2ohj6/latest)**
*(GitHub can only render the static notebook; the interactive version lives in Hex.)*



---

## Headline finding

> **Volume and quality run in opposite directions.**
> Facebook Ads supplies 30% of all leads and converts worst. Referral converts 3x better and needs one-third the follow-up effort. And the gap between two counselors *inside the same branch* is larger than the gap between any two branches.

---

## Business problem

A study abroad consultancy with five branches needs to know which **channels, branches, and counselors** actually turn inquiries into enrolments — so that limited marketing budget and counselor capacity can be reallocated on evidence rather than on lead volume.

Three questions drive the analysis:

1. Which acquisition channels are worth the spend, after accounting for the effort they consume?
2. Where does the funnel leak, and how big is each leak?
3. Is performance variation a branch problem or a people problem?

---

## Dataset

| | |
|---|---|
| Rows | 1,200 leads |
| Period | 1 Jan 2025 – 31 Jul 2026 |
| Grain | One row per lead |
| Source | Synthetic/anonymised CRM-style export (`data/study_abroad_leads_dataset.csv`) |

**Key fields:** `lead_source` (channel), `branch`, `counselor`, `status`, `country_interested`, `service_type`, `budget_range`, `created_date`, `last_update_date`, `cycle_days`, `follow_up_count`

**Status values (pipeline stages):** New · Contacted · Consultation Done · Application Submitted · Visa Approved · Enrolled · Lost

---

## Method

### 1. Cleaning
- Standardised text casing and stripped whitespace across categorical fields
- De-duplicated exact duplicate lead records
- `budget_range` nulls → explicit `"Not provided"` category (treated as signal, not missingness)
- Derived `month`, `year`, `quarter`, `day` from `created_date`
- Derived `channel_details`: grouped the six channels into **Paid** (Facebook Ads, Google Ads) vs **Organic** (Referral, Website Form, Walk-In, Seminar/Event)

### 2. Outlier handling
Z-score screening (|z| > 3) on `cycle_days` and `follow_up_count` flagged **19 of 1,200 rows (1.6%)**:
- 13 long `cycle_days` — all belonging to `Enrolled` or `Visa Approved` leads, i.e. genuine long-running successful cases
- 6 high `follow_up_count`

**Decision: no rows removed.** These are legitimate business outcomes, not data errors. Deleting them would systematically remove successes and inflate the apparent loss rate.

### 3. Metric definitions

Raw conversion rate is misleading here, because a large share of leads are still in-flight (Germany 62% on-going, Canada 55%). So two metrics are used side by side:

```
Conversion rate  = Enrolled / All leads
Win rate         = Enrolled / (Enrolled + Lost)          # resolved leads only
Effort           = Total follow-ups / Enrolments         # follow-ups per enrolment
```

### 4. Composite channel score

To rank channels on more than one axis, each metric was z-standardised across the six channels and combined:

```
Score = 0.50·z(win rate) + 0.20·z(resolution rate) − 0.30·z(effort per enrolment)
```

Weights are a stated judgement, not a fitted result — quality dominates, speed of resolution helps, effort is a penalty.

---

## Results

### Overall

| Metric | Value |
|---|---|
| Total leads | 1,200 |
| Enrolled | 179 |
| Conversion rate | **14.92%** |
| Lost rate | **30.92%** |
| Avg. cycle time | ~31 days |

**Status distribution:** Lost 371 · Consultation Done 186 · Contacted 183 · Enrolled 179 · New 150 · Application Submitted 76 · Visa Approved 55

### Channel performance

| Channel | Leads | Win rate | Follow-ups / enrolment | Composite score |
|---|---:|---:|---:|---:|
| **Referral** | — | **48.3%** | **9.6** | **+1.91** |
| Seminar / Event | — | 32.1% | — | 0.00 |
| Walk-In | — | 31.1% | — | −0.12 |
| Google Ads | — | 29.5% | — | −0.44 |
| Website Form | — | 25.3% | — | −0.50 |
| **Facebook Ads** | **361 (30%)** | 26.1% | **28.7** | **−0.84** |

- Referral is not marginally better — it is a **different tier**. It is the only channel more than one standard deviation above the mean, and it needs **one-third the follow-up effort** per enrolment that Facebook Ads does.
- Facebook Ads contributes **30% of leads but only 20.7% of enrolments (37)**.
- Aggregated: **Organic 624 leads @ 18.59% conversion vs Paid 576 leads @ 10.94%**.

### Funnel leakage

Excluding lost leads, the cumulative funnel:

```
New            829
Contacted      679   (−18%)
Consultation   496   (−27%)
Application    310   (−37%)   ← largest single leak
Visa Approved  234   (−25%)
Enrolled       179   (−24%)
```

**The biggest drop is Consultation Done → Application Submitted: 63% of leads that complete a consultation never submit an application.** That is a post-consultation follow-up problem, not an acquisition problem — and it is the highest-leverage fix in the whole funnel.

### Branch performance

| Branch | Conversion | Lost rate |
|---|---:|---:|
| Chattogram | 16.92% | — |
| Kalabagan | 16.01% | **33.18%** (highest) |
| Uttara | 13.99% | — |
| Dhanmondi | 13.60% | — |
| Sylhet | 11.65% | — |

Average cycle time is **flat across all branches (27.7 – 34.1 days)** — cycle time carries almost no explanatory signal at branch level.

### Counselor performance — the real story

Branch-level averages hide the actual variance. Within a single branch:

- **Chattogram:** Mim **34.48%** vs Shuvo **8.0%** — a 4x spread inside one office
- **Priya** ranks *bottom* in Uttara (6.67%, 45 leads) and *bottom* in Sylhet (5.56%), but *top* in Dhanmondi (21.21%)
- Weak performers frequently hold the **largest lead books**

**Within-branch variation dwarfs between-branch variation.** Any intervention targeted at "underperforming branches" would be aimed at the wrong unit of analysis.

> Caveat: Priya's inconsistency across branches, combined with small per-branch sample sizes (some under 50 leads), means these are **hypotheses to investigate**, not verdicts on individuals. Lead-mix differences are not yet controlled for.

### Markets

- **UK:** 354 leads (largest market) but only ~14% conversion
- **Australia:** best conversion at ~17%
- **Malaysia:** worst — ~13% conversion with a 37% loss rate
- **Germany / Canada:** raw conversion is depressed by very large in-flight backlogs (62% / 55% still on-going), so win rate is the fairer read

### Budget bands

5–10 Lakh converts best (16.67%) and 20+ Lakh worst (11.73%) — **but the 5–10 Lakh strength is almost entirely driven by Referral leads, not paid ads.** The budget effect is largely a channel effect in disguise.

---

## Recommendations

1. **Cap Facebook Ads spend and fund a referral incentive programme.** Referral wins at 48.3% with 9.6 follow-ups per enrolment; Facebook Ads wins at 26.1% with 28.7. The reallocation pays for itself in counselor hours alone.
2. **Fix the consultation → application handoff.** A 63% drop-off at this single stage is worth more than any acquisition change. Suggested first step: a mandatory 72-hour post-consultation follow-up with a documented outcome.
3. **Run counselor development at the individual level, not the branch level.** Pair the 34% converters with the 8% converters inside the same branch, and rebalance lead books away from the weakest converters holding the largest volumes.
4. **Report win rate alongside conversion rate.** With 55–62% of leads still in flight for some markets, raw conversion systematically penalises newer pipelines.

---

## Limitations

Stated explicitly because they affect how far these results can be pushed:

- **Small samples.** Several counselor and branch×channel cells fall below 50 leads; differences within a few percentage points are not distinguishable from noise.
- **Z-scores over 6 channels and 5 branches.** Standardising across so few groups makes the composite score a *ranking* device, not a statistical test.
- **In-flight leads.** Around 40% of leads are neither Enrolled nor Lost. Conversion rate is therefore a lower bound and is biased against recently acquired cohorts.
- **No mix adjustment.** Counselor comparisons do not yet control for the channel/country/budget mix of leads assigned to each person. A mix-adjusted lift (`Expected = Σ nₛ·pₛ`, `Lift = Actual / Expected`) is the natural next step.
- **Data quality note.** 60 rows have `cycle_days ≠ last_update_date − created_date`. The discrepancy is documented and left unmodified rather than silently patched.
- **Truncated final months.** Data ends 31 Jul 2026, so Aug–Dec in the monthly trend contains 2025 only. Year-over-year comparison is valid Jan–Jul only.
- **Correlation, not causation.** Referral leads may convert better because referred prospects arrive with higher intent — the channel may be selecting good leads rather than creating them.

---

## Dashboard

Six tabs, built in Hex:

| Tab | Contents |
|---|---|
| **Overview** | 5 headline KPIs, 2025-vs-2026 monthly trend, channel volume, monthly table |
| **Pipeline** | Stage-by-stage KPIs and funnel table |
| **Channels** | Conversion by channel, conversion-vs-volume combo, composite score, paid-vs-organic split |
| **Branches** | Conversion, loss rate, cycle time, branch × channel matrix |
| **Counselors** | Ranked counselor conversion, best-vs-weakest per branch, top/bottom tables |
| **Markets** | Country demand, service type, conversion-vs-loss, country × service pivot |

---

## Repository structure

```
study-abroad-lead-funnel-analysis/
├── README.md
├── data/
│   └── study_abroad_leads_dataset.csv     # raw input
├── notebooks/
│   └── Study Abroad Lead Funnel Analysis.ipynb         # full analysis, exported from Hex
├── hex/
│   └── Study Abroad Lead Funnel Analysis.yaml                       # Hex project export
└── Dashboard overview screenshots/
    ├── overall.png
    ├── status stages pipeline keys.png
    ├── channel based conversion rate analysis.png
    └── best channel & (paid-organic) channels performance analysis.png
    ├── branch's conversion vs loss rate.png
    ├── branch's average cycle time analysis.png
    ├── counselor's performance analysis for each branch.png
    └── top & bottom performed counselors.png
    ├── country & service demand analysis.png
    └── country based conversion and loss rate analysis.png


```

---

## Tech stack

**Python**
- `pandas` — data cleaning, de-duplication, categorical standardisation, derived date fields
- `numpy` — z-score outlier screening and metric standardisation for the composite channel score

**SQL (DuckDB, in-notebook)**
- Aggregations and conversion/win-rate calculations at channel, branch, counselor, and country grain
- Window functions with `QUALIFY` for top/bottom-performer-per-branch ranking
- Cross-tab and pivot-style queries for branch × channel and country × service matrices
- Chained queries — each cell's output reused as the input table for the next

**Hex**
- Notebook environment for the Python + SQL analysis chain
- Native chart, pivot, and KPI cells
- Published 6-tab interactive app with drill-down

**Version control**
- Git / GitHub for the notebook export, dataset, and documentation

---

## Reproducing this analysis

```bash
git clone https://github.com/<your-username>/study-abroad-lead-funnel-analysis.git
cd study-abroad-lead-funnel-analysis
pip install pandas numpy duckdb jupyter
jupyter notebook notebooks/lead_funnel_analysis.ipynb
```

---

## Author

**Nuzmol Islam Khan** — [LinkedIn](# Study Abroad Lead Funnel Analysis

Lead acquisition, conversion, and counselor performance analysis across **1,200 study-abroad leads** (Jan 2025 – Jul 2026), built as an end-to-end analytics project: data cleaning → exploratory analysis → composite channel scoring → an interactive 6-tab dashboard.

**[View the live interactive dashboard →](https://app.hex.tech/019ff178-c7f3-757b-b506-c32f40e9b1b1/app/Study-Abroad-Lead-Funnel-Analysis-0345MtN10MD65gvJe2ohj6/latest)**
*(GitHub can only render the static notebook; the interactive version lives in Hex.)*



---

## Headline finding

> **Volume and quality run in opposite directions.**
> Facebook Ads supplies 30% of all leads and converts worst. Referral converts 3x better and needs one-third the follow-up effort. And the gap between two counselors *inside the same branch* is larger than the gap between any two branches.

---

## Business problem

A study abroad consultancy with five branches needs to know which **channels, branches, and counselors** actually turn inquiries into enrolments — so that limited marketing budget and counselor capacity can be reallocated on evidence rather than on lead volume.

Three questions drive the analysis:

1. Which acquisition channels are worth the spend, after accounting for the effort they consume?
2. Where does the funnel leak, and how big is each leak?
3. Is performance variation a branch problem or a people problem?

---

## Dataset

| | |
|---|---|
| Rows | 1,200 leads |
| Period | 1 Jan 2025 – 31 Jul 2026 |
| Grain | One row per lead |
| Source | Synthetic/anonymised CRM-style export (`data/study_abroad_leads_dataset.csv`) |

**Key fields:** `lead_source` (channel), `branch`, `counselor`, `status`, `country_interested`, `service_type`, `budget_range`, `created_date`, `last_update_date`, `cycle_days`, `follow_up_count`

**Status values (pipeline stages):** New · Contacted · Consultation Done · Application Submitted · Visa Approved · Enrolled · Lost

---

## Method

### 1. Cleaning
- Standardised text casing and stripped whitespace across categorical fields
- De-duplicated exact duplicate lead records
- `budget_range` nulls → explicit `"Not provided"` category (treated as signal, not missingness)
- Derived `month`, `year`, `quarter`, `day` from `created_date`
- Derived `channel_details`: grouped the six channels into **Paid** (Facebook Ads, Google Ads) vs **Organic** (Referral, Website Form, Walk-In, Seminar/Event)

### 2. Outlier handling
Z-score screening (|z| > 3) on `cycle_days` and `follow_up_count` flagged **19 of 1,200 rows (1.6%)**:
- 13 long `cycle_days` — all belonging to `Enrolled` or `Visa Approved` leads, i.e. genuine long-running successful cases
- 6 high `follow_up_count`

**Decision: no rows removed.** These are legitimate business outcomes, not data errors. Deleting them would systematically remove successes and inflate the apparent loss rate.

### 3. Metric definitions

Raw conversion rate is misleading here, because a large share of leads are still in-flight (Germany 62% on-going, Canada 55%). So two metrics are used side by side:

```
Conversion rate  = Enrolled / All leads
Win rate         = Enrolled / (Enrolled + Lost)          # resolved leads only
Effort           = Total follow-ups / Enrolments         # follow-ups per enrolment
```

### 4. Composite channel score

To rank channels on more than one axis, each metric was z-standardised across the six channels and combined:

```
Score = 0.50·z(win rate) + 0.20·z(resolution rate) − 0.30·z(effort per enrolment)
```

Weights are a stated judgement, not a fitted result — quality dominates, speed of resolution helps, effort is a penalty.

---

## Results

### Overall

| Metric | Value |
|---|---|
| Total leads | 1,200 |
| Enrolled | 179 |
| Conversion rate | **14.92%** |
| Lost rate | **30.92%** |
| Avg. cycle time | ~31 days |

**Status distribution:** Lost 371 · Consultation Done 186 · Contacted 183 · Enrolled 179 · New 150 · Application Submitted 76 · Visa Approved 55

### Channel performance

| Channel | Leads | Win rate | Follow-ups / enrolment | Composite score |
|---|---:|---:|---:|---:|
| **Referral** | — | **48.3%** | **9.6** | **+1.91** |
| Seminar / Event | — | 32.1% | — | 0.00 |
| Walk-In | — | 31.1% | — | −0.12 |
| Google Ads | — | 29.5% | — | −0.44 |
| Website Form | — | 25.3% | — | −0.50 |
| **Facebook Ads** | **361 (30%)** | 26.1% | **28.7** | **−0.84** |

- Referral is not marginally better — it is a **different tier**. It is the only channel more than one standard deviation above the mean, and it needs **one-third the follow-up effort** per enrolment that Facebook Ads does.
- Facebook Ads contributes **30% of leads but only 20.7% of enrolments (37)**.
- Aggregated: **Organic 624 leads @ 18.59% conversion vs Paid 576 leads @ 10.94%**.

### Funnel leakage

Excluding lost leads, the cumulative funnel:

```
New            829
Contacted      679   (−18%)
Consultation   496   (−27%)
Application    310   (−37%)   ← largest single leak
Visa Approved  234   (−25%)
Enrolled       179   (−24%)
```

**The biggest drop is Consultation Done → Application Submitted: 63% of leads that complete a consultation never submit an application.** That is a post-consultation follow-up problem, not an acquisition problem — and it is the highest-leverage fix in the whole funnel.

### Branch performance

| Branch | Conversion | Lost rate |
|---|---:|---:|
| Chattogram | 16.92% | — |
| Kalabagan | 16.01% | **33.18%** (highest) |
| Uttara | 13.99% | — |
| Dhanmondi | 13.60% | — |
| Sylhet | 11.65% | — |

Average cycle time is **flat across all branches (27.7 – 34.1 days)** — cycle time carries almost no explanatory signal at branch level.

### Counselor performance — the real story

Branch-level averages hide the actual variance. Within a single branch:

- **Chattogram:** Mim **34.48%** vs Shuvo **8.0%** — a 4x spread inside one office
- **Priya** ranks *bottom* in Uttara (6.67%, 45 leads) and *bottom* in Sylhet (5.56%), but *top* in Dhanmondi (21.21%)
- Weak performers frequently hold the **largest lead books**

**Within-branch variation dwarfs between-branch variation.** Any intervention targeted at "underperforming branches" would be aimed at the wrong unit of analysis.

> Caveat: Priya's inconsistency across branches, combined with small per-branch sample sizes (some under 50 leads), means these are **hypotheses to investigate**, not verdicts on individuals. Lead-mix differences are not yet controlled for.

### Markets

- **UK:** 354 leads (largest market) but only ~14% conversion
- **Australia:** best conversion at ~17%
- **Malaysia:** worst — ~13% conversion with a 37% loss rate
- **Germany / Canada:** raw conversion is depressed by very large in-flight backlogs (62% / 55% still on-going), so win rate is the fairer read

### Budget bands

5–10 Lakh converts best (16.67%) and 20+ Lakh worst (11.73%) — **but the 5–10 Lakh strength is almost entirely driven by Referral leads, not paid ads.** The budget effect is largely a channel effect in disguise.

---

## Recommendations

1. **Cap Facebook Ads spend and fund a referral incentive programme.** Referral wins at 48.3% with 9.6 follow-ups per enrolment; Facebook Ads wins at 26.1% with 28.7. The reallocation pays for itself in counselor hours alone.
2. **Fix the consultation → application handoff.** A 63% drop-off at this single stage is worth more than any acquisition change. Suggested first step: a mandatory 72-hour post-consultation follow-up with a documented outcome.
3. **Run counselor development at the individual level, not the branch level.** Pair the 34% converters with the 8% converters inside the same branch, and rebalance lead books away from the weakest converters holding the largest volumes.
4. **Report win rate alongside conversion rate.** With 55–62% of leads still in flight for some markets, raw conversion systematically penalises newer pipelines.

---

## Limitations

Stated explicitly because they affect how far these results can be pushed:

- **Small samples.** Several counselor and branch×channel cells fall below 50 leads; differences within a few percentage points are not distinguishable from noise.
- **Z-scores over 6 channels and 5 branches.** Standardising across so few groups makes the composite score a *ranking* device, not a statistical test.
- **In-flight leads.** Around 40% of leads are neither Enrolled nor Lost. Conversion rate is therefore a lower bound and is biased against recently acquired cohorts.
- **No mix adjustment.** Counselor comparisons do not yet control for the channel/country/budget mix of leads assigned to each person. A mix-adjusted lift (`Expected = Σ nₛ·pₛ`, `Lift = Actual / Expected`) is the natural next step.
- **Data quality note.** 60 rows have `cycle_days ≠ last_update_date − created_date`. The discrepancy is documented and left unmodified rather than silently patched.
- **Truncated final months.** Data ends 31 Jul 2026, so Aug–Dec in the monthly trend contains 2025 only. Year-over-year comparison is valid Jan–Jul only.
- **Correlation, not causation.** Referral leads may convert better because referred prospects arrive with higher intent — the channel may be selecting good leads rather than creating them.

---

## Dashboard

Six tabs, built in Hex:

| Tab | Contents |
|---|---|
| **Overview** | 5 headline KPIs, 2025-vs-2026 monthly trend, channel volume, monthly table |
| **Pipeline** | Stage-by-stage KPIs and funnel table |
| **Channels** | Conversion by channel, conversion-vs-volume combo, composite score, paid-vs-organic split |
| **Branches** | Conversion, loss rate, cycle time, branch × channel matrix |
| **Counselors** | Ranked counselor conversion, best-vs-weakest per branch, top/bottom tables |
| **Markets** | Country demand, service type, conversion-vs-loss, country × service pivot |

---

## Repository structure

```
study-abroad-lead-funnel-analysis/
├── README.md
├── data/
│   └── study_abroad_leads_dataset.csv     # raw input
├── notebooks/
│   └── Study Abroad Lead Funnel Analysis.ipynb         # full analysis, exported from Hex
├── hex/
│   └── Study Abroad Lead Funnel Analysis.yaml                       # Hex project export
└── Dashboard overview screenshots/
    ├── overall.png
    ├── status stages pipeline keys.png
    ├── channel based conversion rate analysis.png
    └── best channel & (paid-organic) channels performance analysis.png
    ├── branch's conversion vs loss rate.png
    ├── branch's average cycle time analysis.png
    ├── counselor's performance analysis for each branch.png
    └── top & bottom performed counselors.png
    ├── country & service demand analysis.png
    └── country based conversion and loss rate analysis.png


```

---

## Tech stack

**Python**
- `pandas` — data cleaning, de-duplication, categorical standardisation, derived date fields
- `numpy` — z-score outlier screening and metric standardisation for the composite channel score

**SQL (DuckDB, in-notebook)**
- Aggregations and conversion/win-rate calculations at channel, branch, counselor, and country grain
- Window functions with `QUALIFY` for top/bottom-performer-per-branch ranking
- Cross-tab and pivot-style queries for branch × channel and country × service matrices
- Chained queries — each cell's output reused as the input table for the next

**Hex**
- Notebook environment for the Python + SQL analysis chain
- Native chart, pivot, and KPI cells
- Published 6-tab interactive app with drill-down

**Version control**
- Git / GitHub for the notebook export, dataset, and documentation

---

## Reproducing this analysis

```bash
git clone https://github.com/<your-username>/study-abroad-lead-funnel-analysis.git
cd study-abroad-lead-funnel-analysis
pip install pandas numpy duckdb jupyter
jupyter notebook notebooks/lead_funnel_analysis.ipynb
```

---

## Author

**Nuzmol Islam Khan** — [LinkedIn](https://www.linkedin.com/in/mdnuzmol/) · [GitHub](https://github.com/Nuzmolkhan))
