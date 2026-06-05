# Extradition Treaties and Homicide Rates: A Cross-Country Panel Analysis

## Overview

This project examines whether bilateral extradition treaties reduce homicide rates in signing countries and whether the expansion of extradition treaty networks displaces criminal activity to non-treaty "safe haven" countries. Using a staggered difference-in-differences design with country and year fixed effects, I find no statistically significant effect of extradition treaties on homicide rates — either through deterrence in treaty countries or displacement to non-treaty countries. This null result is robust across multiple specifications and suggests that extradition treaties alone may be insufficient to deter violent crime.

---

## Research Questions

1. Do countries that sign extradition treaties with the United States experience lower homicide rates in the years following the treaty?
2. Do non-treaty countries experience higher homicide rates as the U.S. extradition network expands, consistent with a criminal displacement or "safe haven" effect?

---

## Data

### Treaty Data
- **Source:** U.S. Department of State, *Treaties in Force* (2025)
- **Coverage:** 56 countries with U.S. extradition treaties, 36 of which entered into force after 1995
- **Construction:** Parsed programmatically from the State Department PDF, manually verified against primary source entries for all post-1995 treaties
- **Key variable:** Year of entry into force (used as treatment date)

### Homicide Data
- **Source:** United Nations Office on Drugs and Crime (UNODC), intentional homicide statistics
- **Coverage:** 208 countries, 1990–2022
- **Measure:** Intentional homicide rate per 100,000 population

### Control Variables
- **Source:** World Bank World Development Indicators (WDI), via the `WDI` R package
- **Variables:** Log GDP per capita, unemployment rate, urbanization rate, log population
- **Coverage:** 1990–2022

### Geographic Data
- **Source:** CEPII GeoDist dataset
- **Use:** Bilateral contiguity indicators for constructing neighbor-level treaty exposure variable

---

## Methodology

### Identification Strategy
I exploit the staggered timing of U.S. extradition treaty signings across countries as a source of quasi-experimental variation. Countries sign treaties at different points in time, generating a natural control group of never-treated countries for comparison.

The baseline specification is a two-way fixed effects (TWFE) difference-in-differences estimator:

```
homicide_rate_it = β(treaty_us_it) + γ_i + δ_t + X_it'π + ε_it
```

Where:
- `treaty_us_it` = 1 if country i has an active U.S. extradition treaty in year t
- `γ_i` = country fixed effects (absorb stable cross-country differences)
- `δ_t` = year fixed effects (absorb global time trends)
- `X_it` = time-varying controls (GDP per capita, unemployment, urbanization, population)
- Standard errors clustered at the country level

### Sample Construction
- **Treated group:** Countries whose extradition treaty with the U.S. entered into force after 1995 (36 countries)
- **Control group:** Never-treated countries — those with no U.S. extradition treaty in the sample period
- **Excluded:** Pre-1995 long-treated countries (treated well before the sample period, not clean controls) and protocol/supplement entries

### Event Study
I estimate an event study specification plotting leads and lags around the treaty entry into force date to test the parallel trends assumption and examine dynamic treatment effects. Endpoints are binned at −5 and +10 years.

### Safe Haven Analysis
To test the displacement hypothesis, I restrict the sample to never-treated countries and regress homicide rates on a neighbor-level treaty exposure variable — the share of each country's contiguous neighbors with an active U.S. treaty in a given year. This variable varies across countries within the same year, allowing identification with both country and year fixed effects.

---

## Results

### Baseline DiD
The coefficient on `treaty_us` is small and statistically insignificant (β = 0.37, p = 0.77), providing no evidence that extradition treaties reduce homicide rates in signing countries.

### Event Study
Pre-treatment coefficients (years −5 to −1) are close to zero and statistically indistinguishable from zero, supporting the parallel trends assumption. Post-treatment coefficients show no systematic pattern of decline, consistent with the null baseline result.

### Safe Haven Analysis
The neighbor treaty exposure variable is negative and insignificant (β = −0.99, p = 0.54), providing no evidence that non-treaty countries experience higher homicide rates as their neighbors join the U.S. extradition network.

### Regional Subsample — Americas
Restricting the sample to the Americas (45 countries), where most post-1995 U.S. treaty signings occurred and where criminal flight across borders is most common, yields a positive but insignificant coefficient (β = 2.47, p = 0.24). The high within-country variance in Latin American homicide rates (RMSE = 10.0) suggests that region-specific drivers of violence — gang activity, drug trafficking, institutional weakness — dominate any treaty effect.

---

## Interpretation

The consistent null findings across specifications suggest two possible interpretations:

1. **Weak deterrence mechanism:** Criminals may not sufficiently update their behavior in response to extradition treaty networks, either because they are unaware of treaty status, because enforcement is inconsistent, or because the probability of flight and capture is low regardless of treaty existence.

2. **Wrong outcome variable:** Homicide may not be the crime type most sensitive to extradition risk. Crimes involving premeditated flight — financial crime, organized drug trafficking, corruption — may show stronger treaty effects. This is a direction for future research.

---

## Limitations

- **U.S. treaties only:** A true safe haven analysis requires the full global bilateral treaty network. This project captures only U.S. bilateral treaties; countries may have treaties with other major sender nations (UK, EU, France, Germany) that this analysis does not account for.
- **Homicide as outcome:** As discussed above, homicide may not be the most theoretically appropriate outcome for testing extradition deterrence.
- **Gini coefficient excluded:** Income inequality data had 54% missing observations across the sample and was excluded from controls. Rule of law and institutional quality indicators were similarly unavailable through the WDI API for the full sample period.
- **Treaty enforcement:** Treaty existence does not guarantee enforcement. Some countries with active treaties have historically refused extradition requests. Treaty quality and enforcement intensity are not captured in this analysis.

---

## Repository Structure

```
├── data/
│   ├── us_extradition_treaties_clean.csv   # Verified US treaty dataset
│   ├── unodc_homicide_clean.csv            # UNODC homicide rates 1990-2022
│   ├── wb_controls.csv                     # World Bank control variables
│   └── panel_base.csv                      # Merged country-year panel
├── scripts/
│   ├── parse_treaties.R                    # Parse State Dept TIF PDF
│   ├── clean_treaties.R                    # Clean and verify treaty dataset
│   ├── build_panel.R                       # Merge datasets into panel
│   ├── did_baseline.R                      # Baseline DiD regression
│   ├── event_study.R                       # Event study specification
│   └── safehaven.R                         # Safe haven displacement analysis
├── figures/
│   └── event_study_us_baseline.png         # Event study plot
└── README.md
```

---

## Requirements

```r
install.packages(c("tidyverse", "pdftools", "countrycode", "WDI",
                   "readxl", "fixest", "did"))
```

---

## Citation

**Treaty data:** U.S. Department of State. *Treaties in Force: A List of Treaties and Other International Agreements of the United States in Force on January 1, 2025.* Washington, D.C.: U.S. Department of State, 2025.

**Homicide data:** United Nations Office on Drugs and Crime. *Intentional Homicide Victims.* UNODC Statistics, 2024. https://dataunodc.un.org/dp-intentional-homicide-victims

**World Bank data:** World Bank. *World Development Indicators.* Washington, D.C.: The World Bank, 2024. https://databank.worldbank.org/source/world-development-indicators

**Geographic data:** Mayer, T. & Zignago, S. (2011). *Notes on CEPII's distances measures: The GeoDist database.* CEPII Working Paper 2011-25.

---

## Author

Master of Arts in Economics, California State University Long Beach

*This project was developed as a portfolio piece demonstrating applied econometric methods in international economics and crime policy.*
