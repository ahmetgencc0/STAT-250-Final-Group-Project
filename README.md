# Political Violence & Protest Event Analysis in Turkey (2016–2023)

**An event-level statistical analysis of recorded political violence and protest activity using ACLED data.**  
*STAT250 Applied Statistics · Middle East Technical University (METU) · Group project, June 2026*

**Python · Data Cleaning · Exploratory Data Analysis · Statistical Inference · Regression Diagnostics**

> **Project focus:** How do recorded events differ by type, fatality status, reporting coverage, geography, and population context—and which characteristics are statistically associated with reported fatalities?

## At a glance

| Data preparation | Analysis scope | Statistical approach | Main regression result |
|:--|:--|:--|:--|
| **32,447** raw rows → **22,110** event-level observations | **2016–2023**, Turkey; **6** event types | **5** research questions; proportion tests, χ², Welch tests, and regression | Multiple linear regression **R² = 0.311**, with HC3 robust standard errors |

### Key findings

- **57.8% of recorded events were protests** (12,782 of 22,110; one-sample proportion test, *p* < 0.001).
- **43.18% of violent-category events had at least one reported fatality**, compared with **0.053% of non-violent-category events**. This is a descriptive association strongly influenced by how event categories are defined—not a causal effect.
- **Fatal events had a higher mean number of recorded reporting sources** (1.641 vs. 1.276; Welch's *t*-test, *p* < 0.001), although both groups had a median of one source.
- **Population context differed across event types** (Welch's ANOVA, *p* < 0.001), with results checked using multiplicity correction and a sensitivity analysis.
- **Violent-event classification was the most prominent predictor in the fitted regression**, while diagnostics identified heteroskedasticity and strong departures from residual normality.

> **Interpretation boundary:** These are associations in *recorded* events. The data are observational and subject to reporting coverage differences; the findings do **not** establish causality or measure all events that occurred.

---

## 1. Research objective

Political violence and protest datasets contain heterogeneous event types, uneven reporting, geographically varying patterns, and highly skewed fatality counts. Analyzing these records responsibly requires more than plotting event totals: the unit of observation must be defined, duplicated event records consolidated, statistical assumptions evaluated, and uncertainty communicated.

This project uses **Armed Conflict Location & Event Data (ACLED)** records for Turkey to investigate five questions:

1. **Event composition:** Do protests account for a majority of recorded events, and is fatality status associated with event type?
2. **Fatal event rates:** Do violent-category events have a higher fatal event rate than non-violent-category events?
3. **Reporting coverage:** Do events with reported fatalities have more recorded reporting sources?
4. **Population context:** Are event types associated with different estimated population contexts?
5. **Regression:** Which observed event characteristics are associated with `log(1 + fatalities)`?

The work also draws a **thematic connection to UN Sustainable Development Goal 16** (Peace, Justice and Strong Institutions). It does **not** compute an official SDG indicator or evaluate institutional effectiveness.

## 2. Data and preparation

**Source:** [ACLED — Armed Conflict Location & Event Data](https://acleddata.com/), with coding decisions informed by the [ACLED Codebook](https://acleddata.com/methodology/acled-codebook).

| Property | Value |
|:--|:--|
| Geographic scope | Turkey |
| Raw extract | 32,447 rows; 31 variables; covering 2016–2024 |
| Final analytical period | 2016–2023 |
| Final analytical data | 22,110 distinct recorded events; 65 variables |
| Unit of analysis | One event per row after restructuring |
| Excluded period | 2024, which contained only eight records in the available extract |

### Data processing decisions

**1. Establish an event-level analytical unit.** Some events appeared on multiple raw rows. Records were consolidated at the event level to avoid inflated counts and distorted statistical summaries, while preserving information on associated actors and reporting sources.

**2. Standardize data types and categories.** Event identifiers, dates, numeric variables, text fields, and event categories were cleaned and standardized for analysis.

**3. Handle missing values transparently.**

- `fatalities`: missing entries were set to **zero reported fatalities**, following the coding convention discussed in the report and ACLED Codebook. This must **not** be read as proof that no fatalities occurred.
- `source_scale`: missing categories were filled, when possible, using the most common category for the same reporting source; remaining missing values were filled with the overall mode.
- `timestamp`: missing entries were filled from `event_date` converted to Unix time. These values were **not** treated as authentic ACLED upload timestamps.

**4. Preserve and derive analytical features.** Repeated-record information was summarized into event-level features such as `actor_count` and `source_count`. Other derived fields made event comparisons and modeling possible:

| Variable | Meaning / derivation |
|:--|:--|
| `fatal_event` | 1 if reported fatalities ≥ 1; otherwise 0 |
| `violent_event` | 1 for Battles, Explosions/Remote violence, Violence against civilians, or Riots |
| `actor_count` | Number of recorded actors linked to an event |
| `source_count` | Number of distinct recorded reporting sources |
| `population_for_analysis` | Population proxy selected using a fallback sequence |
| `log_population` | Log-transformed population proxy |
| `log_fatalities` | `log(1 + fatalities)`, retaining zero-fatality observations |

**5. Construct a population proxy.** The population measure was selected in the order `population_best` → `population_5km` → `population_2km` → `population_1km`, with a median fallback for remaining missing values. The variable represents an **approximation of nearby population**, not the number of people affected by an event.

**6. Limit the time window.** The eight observations available for 2024 were excluded to avoid treating a clearly incomplete year as comparable to full years.

### Why this preparation matters

Without consistent event-level records, a single event represented multiple times could artificially increase event frequencies, reported-source counts, or statistical precision. The preprocessing choices therefore affect the **validity of the analysis**, not just dataset cleanliness.

## 3. Exploratory analysis

### Event composition over time

![Recorded event-type composition by year](assets/event_type_composition.png)

*Recorded event-type composition by year, 2016–2023. Protests account for the largest share of the final analytical dataset.*

### Geographic distribution

![Spatial distribution of recorded and fatal events in Turkey](assets/spatial_distribution.png)

*Locations of recorded events and events with at least one reported fatality. Recorded fatal events appear more concentrated in eastern and southeastern locations; the pattern is descriptive and may also reflect reporting coverage.*

### Fatality patterns by event type

![Fatal event status by event type](assets/fatality_rates_by_event_type.png)

*Fatal-event status varies markedly across ACLED event categories. Differences should be read together with the underlying event definitions.*

Additional EDA examined annual event counts and fatalities, province-level rankings, reporting-source counts, and a **Spearman** correlation matrix. Spearman correlation was preferred for exploratory pairwise associations because several numeric variables were discrete, skewed, or non-normally distributed.

## 4. Statistical analysis: five research questions

### RQ1 — Are protests the majority, and does fatality status vary by event type?

**Methods:** One-sample proportion *z*-test; chi-square test of independence; Cramér's V effect size.

- **Protests:** 12,782 / 22,110 events = **57.8%**; *z* = 23.229, **p < 0.001** for a one-sided test against 50%.
- **Association with fatality status:** χ²(5) = **9,292.705**, **p < 0.001**; **Cramér's V = 0.648**.
- **Selected fatal-event rates:** Battles **53.9%**, Explosions/Remote violence **38.0%**, and Violence against civilians **23.7%**.

**Interpretation:** Recorded event types differ substantially in their fatality patterns. Part of this association is expected from the **definitions of the categories themselves**, so a strong test result is not evidence that classification causes fatalities.

### RQ2 — Are fatal event rates higher in violent-category events?

**Method:** Two-proportion *z*-test with a confidence interval for the difference.

| Event grouping | Events | Events with ≥1 reported fatality | Fatal-event rate |
|:--|--:|--:|--:|
| Violent categories | 5,130 | 2,215 | **43.177%** |
| Non-violent categories | 16,980 | 9 | **0.053%** |

- **Estimated difference:** approximately **43.1 percentage points**.
- **95% confidence interval:** **41.8–44.5 percentage points**.
- **Test result:** *z* = **89.992**, **p < 0.001**.

**Statistical caution:** The non-violent group contained only **nine fatal events**, below a commonly used count-of-10 guideline for the normal approximation. The reported *z*-test should therefore be interpreted cautiously. The observed difference is large, but the classification is also inherently related to the outcome.

### RQ3 — Are fatal events reported by more sources?

**Methods:** Levene's variance test; Welch's two-sample *t*-test; Mann–Whitney U sensitivity check.

| Reporting metric | Fatal events | Non-fatal events |
|:--|--:|--:|
| Mean `source_count` | **1.641** | **1.276** |
| Median `source_count` | 1 | 1 |
| Share with multiple sources | 38.3% | 20.2% |

- **Mean difference:** **0.365 sources**; 95% CI **[0.323, 0.407]**.
- **Welch test:** *t* = **17.069**, **p < 0.001**.
- **Sensitivity check:** Mann–Whitney U test also returned **p < 0.001**.

**Interpretation:** Fatal events had somewhat broader **recorded** source coverage on average, but the shared median of one shows that most events in both groups were still associated with a single source. `source_count` is not a direct measure of media quality or total public attention.

![Mean recorded reporting-source counts by fatality status](assets/reporting_coverage.png)

### RQ4 — Does population context differ by event type?

**Methods:** Levene's test; **Welch's ANOVA** to allow unequal group variances; pairwise comparisons with **Holm correction**; sensitivity analysis.

- The mean `log_population` differed significantly across the six event categories (**Welch's ANOVA, p < 0.001**).
- Battles were associated with the lowest average population proxy; Strategic developments and Riots with the highest.
- Several pairwise differences remained statistically significant after **Holm adjustment**, but not all did.
- Excluding records where `log_population = 0` did not change the overall significance conclusion.

**Interpretation:** Event categories differ in their observed geographic/population context; this does not imply that population size causes a particular event type.

![Population proxy by recorded event type](assets/population_by_event_type.png)

### RQ5 — Which characteristics are associated with reported fatalities?

**Response:** `log_fatalities = log(1 + fatalities)` to reduce skewness while keeping zero-fatality events.

**Predictors:** `violent_event`, `actor_count`, `source_count`, and `log_population`.

```text
log(1 + fatalities) = β₀
                    + β₁ · violent_event
                    + β₂ · actor_count
                    + β₃ · source_count
                    + β₄ · log_population + ε
```

| Model / predictor | Reported result |
|:--|:--|
| Simple linear model: `violent_event` only | **R² = 0.293** |
| Multiple linear model: four predictors | **R² = 0.311** |
| `violent_event` coefficient | **+0.486** |
| `actor_count` coefficient | **+0.036** |
| `source_count` coefficient | **+0.069** |
| `log_population` coefficient | **−0.010** |

All four predictors were statistically significant at the 5% level in the fitted multiple model, using **HC3 heteroskedasticity-robust standard errors**. The violent-event indicator had the largest absolute *t*-statistic among the predictors; raw coefficients should **not** be ranked as effect strengths because the variables use different scales.

**Model comparison:** Adding actor count, source count, and population context raised R² from **0.293 to 0.311**. The report also found lower AIC and BIC for the multiple model, although the gain in explained variation was modest.

#### Regression diagnostics and methodological decisions

- **Heteroskedasticity:** Breusch–Pagan statistic = **3,783.589**, **p < 0.001**. Constant error variance was not supported, motivating **HC3 robust standard errors** for inference.
- **Residual normality:** Jarque–Bera statistic = **283,482.144**, **p < 0.001**; residual skewness **2.776**, kurtosis **19.640**. Residual normality was not supported.
- **Multicollinearity:** Predictor VIFs ranged from **1.011 to 1.408**, indicating no major multicollinearity concern in this specification.
- **Outcome distribution:** Even after `log(1 + fatalities)`, the response remained strongly **zero-heavy**.

![Residuals versus fitted values for multiple regression](assets/regression_residuals.png)

![Q-Q plot of multiple regression residuals](assets/regression_qq_plot.png)

**What the diagnostics mean:** HC3 improves the reliability of standard-error estimates under heteroskedasticity; it **does not** make residuals normal or solve the zero-heavy outcome. Accordingly, this model is presented as an **exploratory association model**, not a validated prediction system or a causal model.

## 5. Methodological takeaways

This project emphasizes decisions that matter in practical statistical work:

1. **Define the observational unit before analysis.** Consolidate repeated event records to avoid misleading counts.
2. **Document imputation rather than hiding it.** Distinguish “zero *reported* fatalities” from a known zero-fatality ground truth.
3. **Match tests to assumptions.** Prefer Welch procedures when equal variances cannot be assumed; use sensitivity checks when distributions are skewed.
4. **Control multiplicity.** Apply Holm correction to pairwise tests rather than reporting unadjusted significance alone.
5. **Report effect sizes and uncertainty.** Include rates, confidence intervals, and Cramér's V alongside p-values.
6. **Diagnose models explicitly.** Heteroskedasticity, non-normal residuals, and excess zeros are not hidden by a statistically significant regression.
7. **Avoid causal language.** Cross-sectional observational patterns and reporting processes can produce associations without identifying causal effects.

## 6. Limitations and responsible interpretation

- **Observational design:** Results describe associations, not mechanisms or causal effects.
- **Reporting and selection bias:** ACLED contains recorded events from available sources, not a complete census; coverage can differ by location and event type.
- **Population approximation:** The constructed population proxy describes estimated population near an event location, not people directly affected.
- **Very sparse fatal outcomes in some groups:** The two-proportion test for violent versus non-violent categories has a normal-approximation caveat.
- **Model specification:** The zero-heavy response and failed regression diagnostics limit the use of standard linear regression as a predictive model.
- **Scope:** Findings refer to the analyzed Turkey 2016–2023 extract and are not automatically generalizable elsewhere.

These limitations are part of the analytical results, not afterthoughts: **statistical significance alone does not establish substantive importance, causality, or model adequacy.**

## 7. Source material and reproducibility

The **analysis and numerical results in this README are drawn from the team's June 2026 STAT250 final project report**. The figures in `assets/` were extracted from that report, rather than recreated or modified with new data.

To reproduce the analysis independently, obtain the relevant ACLED data under its applicable access conditions and follow the transformations and test specifications described above. **This README package contains documentation and report figures, not the original event-level data or runnable analysis notebooks**; reproducibility would require the corresponding code, raw-data version, and software environment to be added to the repository.

If publishing notebooks or scripts alongside this README, ensure the published code matches the reported analysis and document any package versions or source-data access requirements.

## 8. Project team and academic context

**Group 11 — STAT250 Applied Statistics, Department of Statistics, Middle East Technical University (METU), June 2026**

- **Hasan Akkaya**
- **Aslı Ceren Aksu**
- **Ahmet Genç**

This was a **collaborative academic group project**. Results and methodological decisions described here are attributed to the team; this README does not assign unsupported individual ownership of specific analyses.

## 9. References

- [Armed Conflict Location & Event Data (ACLED)](https://acleddata.com/) — primary event-data source.
- [ACLED Codebook](https://acleddata.com/methodology/acled-codebook) — data definitions and coding conventions.
- [United Nations (2015), *Transforming our world: the 2030 Agenda for Sustainable Development*](https://sdgs.un.org/2030agenda) — SDG 16 context.
- [ACLED Turkey 2016 data release](https://acleddata.com/2016/09/01/acled-turkey-2016-data-release/) — additional background source cited in the project report.

---

*Academic project | Exploratory observational analysis | Findings are descriptive/associational, not causal.*
