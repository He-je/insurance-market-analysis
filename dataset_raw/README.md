# Raw data: sources and preparation notes

This folder contains the source CSV inputs read by the Python notebooks. The workflow reads these files without overwriting them and writes prepared CSVs to `../dataset_analysis/`.

## Files and sources

| Local file | Provider and dataset | Indicator or identifier | Unit used in the analysis |
| --- | --- | --- | --- |
| `OECD_premiums_claims_expenses.csv.gz` | OECD, Premiums, claims and expenses of insurance companies | `OECD.DAF.CM:DSD_INS@DF_UNDERWRITING(1.0)`; selected measure `GRS` | Nominal USD millions for selected premium rows |
| `worldbank_gdp.csv` | World Bank, World Development Indicators | `NY.GDP.PCAP.PP.KD` | GDP per capita, PPP, constant 2021 international dollars |
| `worldbank_population_total.csv` | World Bank, World Development Indicators | `SP.POP.TOTL` | Persons |

Official source pages:

- [OECD: Premiums, claims and expenses of insurance companies](https://data-explorer.oecd.org/vis?df%5Bag%5D=OECD.DAF.CM&df%5Bds%5D=dsDisseminateFinalDMZ&df%5Bid%5D=DSD_INS%40DF_UNDERWRITING&df%5Bvs%5D=1.0).
- [World Bank: GDP per capita, PPP (constant 2021 international dollars)](https://data.worldbank.org/indicator/NY.GDP.PCAP.PP.KD).
- [World Bank: Population, total](https://data.worldbank.org/indicator/SP.POP.TOTL).

**Original download dates:** not recorded in the available provenance information. File-system creation or modification dates are not treated as download dates. The included files are local snapshots and may differ from the providers' latest releases.

## Obtaining the inputs

Keep the three inputs in this folder using the exact local filenames above. The OECD input is a gzip-compressed CSV (`.csv.gz`); the World Bank inputs are uncompressed CSVs. The notebooks expect the column layouts described below. Simply renaming a provider's ZIP archive or a differently structured export is not sufficient.

The OECD file is approximately **11.1 MiB compressed** and **711 MiB uncompressed**. Gzip preserves the CSV contents. Notebook 01 reads the `.csv.gz` file directly with pandas, including chunked processing, so manual extraction and Git LFS are not required. The repository's `.gitignore` excludes an uncompressed `OECD_premiums_claims_expenses.csv` copy if one is created locally.

The source pages provide access to the datasets, but a fresh download may contain revisions and may not reproduce this project's snapshots. To inspect the analysis without rebuilding the raw-data stage, use notebook 04 with the supplied files in `dataset_analysis/`.

## OECD file structure and study filters

The raw OECD export contains more measures and reporting dimensions than the study uses. Multiple observations for the same country and year can represent different insurance types, business categories or reporting scopes. They are not automatically duplicates.

Important fields:

| Fields | Meaning and handling |
| --- | --- |
| `REF_AREA`, `Reference area` | Country code and label; codes are used for matching |
| `TIME_PERIOD` | Observation year |
| `MEASURE`, `Measure` | Measure code and label; the study selects gross written premiums |
| `INSURANCE_TYPE`, `Insurance type` | Insurance classification; composite components are assigned to Life or Non-life |
| `INSURANCE_BUSINESS`, `Insurance business` | Direct business, accepted reinsurance and reported business totals |
| `OBS_VALUE` | Source value; blank values remain missing and reported zeros remain zeros |
| `OBS_STATUS`, `CONF_STATUS` | Observation and confidentiality status; retained for interpretation |
| `UNIT_MEASURE`, `UNIT_MULT`, `CURRENCY` | Unit, scale and currency; selected premium values use USD with multiplier 6, meaning millions |

Notebook 01 applies the following scope filters:

| Dimension | Selected value |
| --- | --- |
| `FREQ` | `A` (annual) |
| `MEASURE` | `GRS` (gross written premiums) |
| `UNIT_MEASURE`, `CURRENCY` | `USD` |
| `Ownership` | `All undertakings` |
| `CONTRACT_TYPE` | `_T` |
| `RISK_LOCATION` | `_Z` (not applicable) |
| `PREMIUMS`, `INSURER_TYPE`, `EMPLOYER_TYPE`, `COUNTERPART_AREA`, `INSURANCE_CLASS`, `DESTINATION` | `_Z` |
| `Insurance type` | `Life`, `Composite: life`, `Non-life`, `Composite: non-life` |
| `Insurance business` | `Direct business`, `Reinsurance accepted`, `Total` |

`RISK_LOCATION = _Z` does not mean domestic risks. The source unit multiplier must be retained when converting premiums from USD millions to USD per person.

The main study covers **1995–2024**, while eligible earlier observations are retained to support imputation. Country selection occurs in notebook 02: pre-imputation combined-GWP missingness must be strictly below 30% over the study period. The current selection contains 30 countries. The fixed analysis samples contain 28 GWP countries and 27 matched direct-premium countries.

Aggregation uses available Direct business and Reinsurance accepted components. Reported business Total is used only when both components are missing. A reported Total is not added to its components. Partial aggregates remain distinguishable from complete reporting.

## World Bank file structure

The local files are wide CSV tables with a normal header row:

- Metadata columns: `Country Name`, `Country Code`, `Indicator Name`, `Indicator Code`.
- Annual columns: `1960` through `2025` in the current local files. A year column's presence does not imply that values are available for every country or indicator.
- Rows can include both individual economies and regional aggregates. The notebooks select the retained OECD country codes and the years **1995–2024**.

GDP per capita and population are joined by country code and year. Missing values are not converted to zero. Only the GDP and total-population indicators are required.

Premiums use nominal USD, while GDP per capita uses constant PPP international dollars. These are different valuation bases. Their ratio is not a consistently defined insurance-penetration measure.


## Attribution

The datasets originate from the OECD and World Bank. Project analysis and documentation do not replace the providers' attribution requirements or applicable reuse terms. Consult the linked source pages for those terms and cite the relevant dataset and indicator when reusing the data.
