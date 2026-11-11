# Data and calculation methodology

This document describes the **uploaded `WB_ESG.csv` snapshot**, not the continuously updated live Data360 dataset. Dataset audit: 461,719 observation rows, 71 indicators, 239 geographical area codes, years 1960 to 2023. Matching the file’s geographic codes to the ISO 3166-1 alpha-3 code list yields 193 ISO-coded country/economy/territory entries and 46 other codes (principally regional, income-group, and institutional aggregates).

## Core indicator dictionary

| DAX indicator code | Source label | Unit | Last year in extract |
|---|---|---|---:|
| `WB_ESG_NY_GDP_MKTP_KD_ZG` | GDP growth (annual %) | Annual percentage change | 2023 |
| `WB_ESG_EG_USE_COMM_FO_ZS` | Fossil fuel energy consumption (% of total) | Percentage of total energy consumption | 2023 |
| `WB_ESG_EG_ELC_COAL_ZS` | Electricity production from coal sources (% of total) | Percentage of electricity generation | 2023 |
| `WB_ESG_EG_ELC_RNEW_ZS` | Renewable electricity output (% of total electricity output) | Percentage of electricity generation | 2021 |
| `WB_ESG_EG_FEC_RNEW_ZS` | Renewable energy consumption (% of total final energy consumption) | Percentage of final energy use | 2022 |

`GDP growth (annual %)` means GDP level growth, **not GDP per capita growth**. `Renewable electricity output` and `renewable energy consumption` differ in their denominator. `Coal electricity` and `renewable electricity` measure shares of total output, so they are compatible as electricity mixes but do not sum to 100% because of other sources.

## Selected calculations

### 1. Filtering one indicator at the selected year and area

I used the verified indicator code and preserved the year and area filters. Example:

```dax
Renewable Electricity (%) =
CALCULATE(
    SELECTEDVALUE('WB_ESG'[OBS_VALUE]),
    REMOVEFILTERS('WB_ESG'[INDICATOR], 'WB_ESG'[INDICATOR_LABEL]),
    'WB_ESG'[INDICATOR] = "WB_ESG_EG_ELC_RNEW_ZS"
)
```

A country, year, and indicator should resolve to one `OBS_VALUE` in this extract. `SELECTEDVALUE` deliberately returns blank if more than one distinct observation survives filtering.

### 2. Comparing 2010 and 2020 for a country

```dax
Renewable Electricity 2010 (%) =
CALCULATE(
    [Renewable Electricity (%)],
    REMOVEFILTERS('WB_ESG'[TIME_PERIOD]),
    'WB_ESG'[TIME_PERIOD] = 2010
)

Renewable Electricity 2020 (%) =
CALCULATE(
    [Renewable Electricity (%)],
    REMOVEFILTERS('WB_ESG'[TIME_PERIOD]),
    'WB_ESG'[TIME_PERIOD] = 2020
)

Renewable Change 2010-2020 (pp) =
VAR StartValue = [Renewable Electricity 2010 (%)]
VAR EndValue = [Renewable Electricity 2020 (%)]
RETURN
IF(
    NOT ISBLANK(StartValue) && NOT ISBLANK(EndValue),
    EndValue - StartValue
)
```

The calculation returns **percentage points**. A change from 10% to 30% is +20 pp, not +20% relative growth. It requires nonblank values for both years.

### 3. UK validation observations

| Indicator | 1990 | 2010 | 2020 | 2021 | 2023 |
|---|---:|---:|---:|---:|---:|
| UK renewable electricity % | 1.8174 | 7.6632 | 43.8948 | 40.3081 | Not observed |
| UK coal electricity % | 64.5649 | 28.4043 | 2.0997 | 2.4239 | 1.5658 |

Values come directly from the provided CSV.

## Reproducing the annual scatter comparisons

For each year, I selected GDP growth (`WB_ESG_NY_GDP_MKTP_KD_ZG`) and one energy-dependence indicator, then joined the two by `REF_AREA` and `TIME_PERIOD`.

### Why 2019, 2021, 2022 and 2023?

- I chose 2019 as a pre-pandemic comparator that avoids the sharp contraction many economies saw in 2020.
- I included 2021 because it reflects the uneven reopening and recovery after the pandemic, so it is not a typical year.
- I included 2022 because of the exceptional conditions in global energy markets and the narrower observation coverage, particularly for coal electricity.
- I added 2023 as a later snapshot, although far fewer economies report data for it in this CSV.

The scatter plots have a year dropdown that runs from 1960 to 2023, so any year can be selected. I captured 2019, 2021, 2022 and 2023 for this documentation, but the dashboard is interactive and supports a much longer period.

## Top 10 ranking rules

- I used the indicator `WB_ESG_EG_ELC_RNEW_ZS`, renewable electricity as a share of total electricity output.
- I retained ISO-coded country and economy observations that had values in both 2010 and 2020 (182 economies in this extract) and calculated the change as 2020 minus 2010, in percentage points.
- I ranked the economies in descending order and kept the top 10. In this dataset they are Luxembourg, Denmark, Uganda, Cambodia, Estonia, United Kingdom, Liberia, Ecuador, Nicaragua and Lithuania.
- In Power BI, I applied a Top N filter by `[Renewable Change 2010-2020 (pp)]` and then sorted the visual by the same measure in descending order.
- I left the total row out of the matrix, since summing annual percentage shares across countries has no meaningful interpretation.

## Data quality caveats

1. Indicators have unequal update schedules. Renewable electricity ends in 2021, while coal ends in 2023 in this file. The coal series has only 83 ISO-coded observations in 2022 and 33 in 2023. Do not use missing values as zeros.
2. The data measures *shares*, not absolute energy output, generation capacity, or CO2 avoided.
3. Area coverage includes aggregate regions such as `Arab World`, which should be excluded from country-only rankings and identified in scatter plots.
4. Comparisons across countries may reflect hydropower, biomass, nuclear, other non-fossil sources, electricity imports/exports, different total generation, economic structure, energy access, and policy decisions. The chosen charts cannot isolate these drivers.
5. This portfolio project examines mainly the **E** in ESG and does not calculate a broad sovereign ESG rating.

## Sources

- [World Bank Data360 WB_ESG dataset](https://data360.worldbank.org/en/dataset/WB_ESG)
- [World Bank Sovereign ESG framework](https://esgdata.worldbank.org/data/framework)
- [World Bank open-data licence guidance](https://datacatalog.worldbank.org/public-licenses)
