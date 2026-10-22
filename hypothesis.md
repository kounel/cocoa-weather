# Does West African weather predict cocoa prices?

Pre-registered hypothesis, method and results. Everything in sections 1–4 was fixed before running the regressions; section 5 reports what came out.

## 1. Question

Does rainfall in the main West African cocoa-growing regions predict changes in the global cocoa price, and if so, does the market react immediately or only when the harvest arrives?

## 2. Hypotheses

**Variable 1: signed rainfall anomaly (primary).** More rain than normal means a bigger crop, and a bigger crop means a lower price. Expected coefficient: **negative**.

**Variable 2: absolute rainfall anomaly (secondary).** More rain isn't always better. Too much damages the crop just as too little does, so the crop does best in a "goldilocks zone". The absolute anomaly measures how unusual the weather was in either direction: more unusual weather means a smaller crop and a higher price. Expected coefficient: **positive**. This variable exists because a linear model on the signed anomaly alone would miss a U-shaped relationship.

**Lag hypothesis A: efficient markets.** Traders already track rainfall in real time, so the price moves in the same month (lag 0) and shows nothing at the later, biological lag.

**Lag hypothesis B: slow information.** Weather data from remote growing regions is noisy, so market participants wait for actual harvest volumes, and the price moves several months later (lags 1–6).

**Prediction.** I expect hypothesis A to win: there may be a weak correlation, but nothing significant. Market makers and high-frequency traders price in rainfall the moment it happens. Only a small group of slower participants reacts to the harvested volume later, which moves the price a little but not most of the way.

Note: the biological lag (pods take 5–6 months to develop) is not the same thing as the price lag. A crop can depend on rain five months earlier while the price reacts immediately.

## 3. Data

| | |
|---|---|
| **Price** | FRED `PCOCOUSDM`: IMF global cocoa price, USD per metric ton, monthly. Available from January 1992. It is a monthly *average* of daily prices, not an end-of-month price. |
| **Price transformation** | Monthly log changes, because price levels trend and trending series produce spurious correlations. |
| **Rainfall** | NASA POWER daily point API, `PRECTOTCORR` (bias-corrected precipitation), from January 1981. It is the only POWER precipitation parameter going back that far. Daily mm/day values are summed to monthly totals. |
| **Points** | Eight points, four per country. Ivory Coast: Gagnoa, San-Pédro, Soubré, Daloa. Ghana: Sefwi Wiawso, Assin Fosu, Goaso, Bekwai. |
| **Point choices** | Kumasi was replaced by Bekwai, because Kumasi is a large city on the drier northern edge of the belt. Asankragwa was replaced by Assin Fosu, because it returned values identical to Sefwi Wiawso: they share a POWER grid cell, and the grid is roughly 50 km. |
| **Sample** | Price changes from February 1992 to February 2026, the last complete harvest season, giving 409 months. Rainfall from 1981 so the lags have history to draw on. |

## 4. Method

**Regional series.** Take the mean across the eight points (not the sum), giving one regional monthly rainfall series.

**Anomaly baseline.** Each month is compared with the long-run average for that calendar month. For example, the June 2010 anomaly is June 2010 rainfall minus the average of all Junes. The anomaly is standardised by dividing by that calendar month's standard deviation, so how unusual a month was is comparable across wet and dry months. This matters for Variable 2, which measures unusualness in both directions.

**Rainfall window.** The main harvest runs October–February and pods take about 5–6 months to develop, so the rainfall that determines crop size falls in April–August. This window defines the season-level test.

**Monthly regressions: 14, all pre-specified.**
`price_change(t) = a + b · anomaly(t − k)` for k = 0…6, run separately for the signed and the absolute anomaly. Lag 0 is the test of hypothesis A; lags 1–6 test hypothesis B. No lags are added after seeing results.

**Standard errors.** Newey–West (HAC), which corrects for autocorrelation and changing volatility without changing the coefficients. Lag length is 5, from the Newey–West rule of thumb `floor(4 · (T/100)^(2/9))` with T = 409.

**Season-level version.** One row per season. The predictor is the April–August rainfall total, standardised across years, plus its absolute value. The outcome is the log price change from September of the same year to February of the next (`ln P(Feb, y+1) − ln P(Sep, y)`). That gives about 34 seasons, so power is low by construction. Seasons don't overlap, so the standard errors only correct for uneven variance (HAC with 0 lags).

**Robustness: excluding 2023–24.** The 2023–24 drought and price spike is the single biggest event in the sample and could drive the result on its own. All regressions are rerun with every observation whose price change falls in calendar 2023 or 2024 removed. At season level, that means dropping the seasons 2022, 2023 and 2024.

**Reading 14 tests.** With 14 tests at the 5% level, about one "significant" result is expected by chance alone. Results are also read against a Bonferroni threshold of 0.05 / 14 ≈ 0.0036.

**Not doing (to avoid overfitting):**
- No other growing regions or countries.
- No phase weighting within April–August, although rainfall matters more in some growth phases than others.
- No dollar or inflation controls.
- No non-linear terms beyond the absolute anomaly.
- No bounding-box or regional grid averaging; points only.
- No FAOSTAT production data (possible extension if time allows).

## 5. Results

Monthly regressions: 409 months (Feb 1992 – Feb 2026), Newey–West standard errors with 5 lags. "Excl. 2023–24" drops every month whose price change falls in 2023 or 2024 (385 months).

### Monthly regressions: signed anomaly (expected sign: −)

| Lag | Coef | Std err | p | R² | Coef, excl. 2023–24 | p, excl. 2023–24 | R², excl. 2023–24 |
|---|---|---|---|---|---|---|---|
| 0 | −0.0038 | 0.0048 | 0.423 | 0.003 | −0.0078 | 0.093 | 0.013 |
| 1 | −0.0048 | 0.0053 | 0.368 | 0.004 | **−0.0122** | **0.005** | 0.031 |
| 2 | −0.0002 | 0.0049 | 0.975 | 0.000 | −0.0030 | 0.473 | 0.002 |
| 3 | 0.0051 | 0.0052 | 0.321 | 0.005 | −0.0004 | 0.924 | 0.000 |
| 4 | −0.0020 | 0.0041 | 0.630 | 0.001 | −0.0031 | 0.345 | 0.002 |
| 5 | 0.0088 | 0.0058 | 0.131 | 0.013 | 0.0057 | 0.231 | 0.006 |
| 6 | 0.0037 | 0.0043 | 0.381 | 0.002 | 0.0054 | 0.176 | 0.006 |

### Monthly regressions: absolute anomaly (expected sign: +)

| Lag | Coef | Std err | p | R² | Coef, excl. 2023–24 | p, excl. 2023–24 | R², excl. 2023–24 |
|---|---|---|---|---|---|---|---|
| 0 | 0.0071 | 0.0081 | 0.385 | 0.004 | 0.0000 | 0.9995 | 0.000 |
| 1 | 0.0153 | 0.0092 | 0.096 | 0.016 | 0.0047 | 0.532 | 0.002 |
| 2 | 0.0045 | 0.0074 | 0.543 | 0.001 | 0.0014 | 0.835 | 0.000 |
| 3 | −0.0035 | 0.0075 | 0.638 | 0.001 | −0.0078 | 0.286 | 0.005 |
| 4 | −0.0052 | 0.0058 | 0.366 | 0.002 | −0.0092 | 0.103 | 0.006 |
| 5 | −0.0042 | 0.0071 | 0.552 | 0.001 | −0.0130 | 0.086 | 0.011 |
| 6 | −0.0112 | 0.0076 | 0.139 | 0.009 | −0.0059 | 0.402 | 0.002 |

- Full sample: **0 of 14** regressions have p < 0.05.
- Excluding 2023–24: **1 of 14** (signed anomaly, lag 1, p = 0.005).
- None pass the Bonferroni threshold (0.0036) in either sample.

### Season-level (April–August rain vs. log price change Sep → Feb)

| Variable | Coef | Std err | p | R² | n | Coef, excl. 2023–24 | p, excl. 2023–24 | n, excl. |
|---|---|---|---|---|---|---|---|---|
| signed | 0.0441 | 0.0459 | 0.338 | 0.034 | 34 | 0.0142 | 0.715 | 31 |
| absolute | 0.0828 | 0.0734 | 0.259 | 0.044 | 34 | −0.0356 | 0.601 | 31 |

### What the robustness check changed

On the full sample nothing is significant. With 2023–24 removed, a signed effect at lag 1 appears: a month 1 standard deviation wetter than normal is followed by a price change about 1.2% lower the next month (p = 0.005, R² 3%), with the expected sign. Lag 0 points the same way (−0.8%, p = 0.09).

The relationship is weaker with 2023–24 included, not stronger. In that period prices moved by 20–30% a month for reasons that one month's rainfall anomaly cannot explain: a multi-season supply deficit, crop disease, and speculative and liquidity effects in futures markets. That extra volatility swamps the small, ordinary rainfall signal. The weak absolute-anomaly result at lag 1 in the full sample (p = 0.096) goes the other way: it disappears without 2023–24, so it was driven by that period.

The season level tells the same story. Nothing is significant in either sample, which is expected with 31–34 seasons. The signed coefficient has the wrong sign (positive) both times. The absolute coefficient has the expected positive sign on the full sample (+8% per std, p = 0.26) but turns negative once the 2022–24 seasons are dropped. So the only "unusual weather raises prices" pattern in the data comes from the recent drought seasons.

### Verdict

- **Hypothesis A vs B.** There is no support for B: lags 4–6, where a harvest-timed reaction would appear, show nothing in either sample. The only effect found is at lag 1, which fits a fast reaction, because PCOCOUSDM is a monthly average of daily prices. Rain news arriving late in month t−1 is mostly reflected in month t's average price, so an immediate market reaction shows up at lag 0–1.
- **Absolute anomaly.** No evidence that unusual weather in either direction raises prices. Most coefficients have the wrong sign.
- **Was my prediction right?** Mostly. The pre-specified full-sample test found nothing significant, as predicted, and the market shows no delayed reaction. The pre-specified robustness check found a modest effect in normal years that I did not expect. It does not survive a multiple-testing correction, so it is suggestive, not established.

## 6. Limitations

- **Price construction.** PCOCOUSDM is a monthly average of daily prices, which mechanically creates autocorrelation between consecutive months and blurs exactly when the market reacted.
- **No dollar or inflation controls.** Cocoa is priced in USD, so dollar moves and general inflation are part of the price change being explained.
- **Point-based sampling.** Eight points on a ~50 km grid stand in for the whole belt. Local rainfall between points is missed.
- **No phase weighting.** All April–August months count equally, although some growth phases are more sensitive to rain.
- **Regions not covered.** Nigeria, Cameroon, Latin America and Asia are excluded, as are the drier northern parts of the Ivorian and Ghanaian belts.
- **Monthly anomalies include dry months.** The monthly regressions use all twelve months. Standardised anomalies in the December–February dry season are based on very little rain and can be noisy.
- **Small sample at season level.** About 34 seasons, so a real but modest effect could easily go undetected.
- **Rainfall only.** Temperature, disease (black pod, swollen shoot), farm-gate pricing policy in Ivory Coast and Ghana, and speculative flows are not modelled.
