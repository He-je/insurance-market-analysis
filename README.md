# Insurance Market Analysis: Premiums, Income and Population

An independent data analysis project examining how insurance premiums relate to GDP per capita across selected countries between **1995 and 2024**.

The project combines OECD insurance statistics with World Bank GDP per capita and population data. It covers data preparation, missing-data validation, premium aggregation, statistical analysis and a written research report.

**Author:** Heejung Heo

**Read the [full report](report/Insurance_premium_GDP_population_analysis.pdf)** or see the [raw-data sources and preparation notes](dataset_raw/README.md).

## Research questions

- How did Life, Non-life and combined gross written premiums (GWP) develop over the study period?
- How do their relationships with GDP per capita differ between 1995 and 2024?
- How does the comparison change when premiums are expressed per person or restricted to direct business?
- How sensitive are the findings to estimated values, population adjustment and individual countries?

## Data and scope

| Source | Data | Role in the analysis |
| --- | --- | --- |
| OECD Global Insurance Statistics | Annual insurance premiums, including insurance type, business category and observation status | Premium preparation, aggregation and missing-data assessment |
| World Bank | GDP per capita, PPP, constant 2021 international dollars | Cross-country income comparison |
| World Bank | Total population | Premiums per capita and population-adjusted models |

The main analysis covers **1995–2024**. Earlier insurance observations are used where eligible to support imputation.

Country samples serve different purposes:

- **30 countries:** preparation and data-quality assessment, selected using pre-imputation combined-GWP missingness strictly below 30%.
- **28 countries:** a fixed sample for full-period GWP trends and endpoint comparisons.
- **27 countries:** a matched sample for comparing aggregate GWP, GWP per capita and direct premiums per capita at both endpoints.

These are selected study samples, not a representative sample of all insurance markets. An available aggregate can still contain estimated values or partial components.

## Analytical approach

### Data preparation

The preparation stage inspects source dimensions, applies the study filters and aligns insurance and macroeconomic data by country and year. Source observations, reported zeros, missing values and reporting-status information are preserved.

Premium aggregation uses available Direct business and Reinsurance accepted components. Reported business Total is used as a fallback when both components are missing. Totals are not added to their own components. Composite life and non-life components are assigned to their corresponding sectors.

### Missing-data treatment

Candidate methods depend on the location of a gap and the usable history. They include linear interpolation, Kalman-based models, naive prediction, ETS and ARIMA, with log-scale alternatives where eligible.

Observed blocks are temporarily masked to evaluate predictions against known values. Validation follows the target gap's position and length, respects reported breaks and selects the eligible method with the lowest valid RMSE. The procedure requires at least 12 usable observations and 3–6 validation blocks. Blocks may overlap.

Observed values remain unchanged. Gaps remain unresolved when the history or validation evidence is insufficient and the reason is retained. Estimated values are identified separately from source observations.

### Statistical analysis

- Long-term premium trends and indexed comparisons.
- Pearson and Spearman correlations at the study endpoints.
- Comparisons of aggregate premiums, premiums per capita and direct premiums per capita on matched countries.
- Log-scale regressions with and without population adjustment, using HC3 robust confidence intervals.
- Sensitivity checks excluding estimated endpoint contributions and omitting countries one at a time.

## Findings reported in the PDF

The written report presents the following results for its documented inputs and implementation:

- Combined nominal GWP rises from **USD 1,973.9 billion in 1995** to **USD 7,229.1 billion in 2024** in the fixed 28-country sample.
- The log-scale Pearson correlation between aggregate combined GWP and GDP per capita is **0.363 in 1995** and **0.179 in 2024**. The confidence intervals are wide, and a difference between years is not formally established.
- On the matched 27-country sample, combined GWP per capita and direct premiums per capita have stronger correlations with GDP per capita than aggregate GWP. Most of this contrast appears at the population-scaling step.
- The GDP per capita coefficient is positive in both endpoint years after population adjustment. The 1995 estimate is more sensitive to excluding countries with estimated contributions.

These findings describe cross-country associations. They do not establish a causal effect of income on insurance premiums.

## Repository organization

The repository separates source data, prepared data, notebooks and the written report:

```text
.
├── README.md
├── .gitignore                           # Local artifacts and uncompressed OECD copy
├── dataset_raw/
│   ├── README.md                        # Sources, units, filters
│   ├── OECD_premiums_claims_expenses.csv.gz
│   ├── worldbank_gdp.csv
│   └── worldbank_population_total.csv
├── dataset_analysis/                    # Prepared inputs and Python estimates
│   ├── 01_country_labels.csv
│   ├── 01_scoped_insurance_history.csv
│   ├── 02_selected_insurance_rows.csv
│   ├── 02_worldbank_gdp.csv
│   ├── 02_worldbank_population_total.csv
│   └── 03_imputed_insurance_rows_python.csv
├── code/
│   ├── 01_data_overview.ipynb
│   ├── 02_missingness_and_country_selection.ipynb
│   ├── 03_imputation.ipynb
│   └── 04_data_analysis.ipynb
├── report/
│   └── Insurance_premium_GDP_population_analysis.pdf
└── References/
    └── OECD Insurance Statistics 2022.pdf
```

The `01_` CSVs contain country labels and scoped insurance history, including earlier observations for imputation. The `02_` CSVs contain selected insurance rows and the matching GDP and population inputs. The `03_` CSV contains Python imputation results, source values, estimation flags, methods and validation information. It retains source-level dimensions; it is not a single aggregated country-year panel.

The publication package contains the Python workflow. The R-generated imputation CSV is not required by these notebooks and is not included.

## Running the notebooks

Use a Python 3 environment with Jupyter and the following packages:

```bash
python -m pip install jupyter pandas numpy matplotlib scipy statsmodels IPython
```

Start Jupyter from the repository root or its `code/` folder:

```bash
jupyter notebook
```

Run the notebooks in numeric order, executing cells from top to bottom:

1. `01_data_overview.ipynb` inspects raw inputs and prepares scoped insurance history.
2. `02_missingness_and_country_selection.ipynb` assesses coverage and selects countries.
3. `03_imputation.ipynb` validates candidate methods and exports estimates with their supporting information.
4. `04_data_analysis.ipynb` builds analysis variables and produces statistical tables and figures.

The notebooks automatically locate the repository by looking for `code/`, `dataset_raw/` and `dataset_analysis/` in the current folder or its parents. No personal drive path needs to be edited. Keep these three directories together when moving or cloning the project.

Raw inputs are read from `dataset_raw/`. Generated CSVs are written to `dataset_analysis/`, and rerunning a notebook replaces its corresponding generated CSVs. Notebook 04 reads `03_imputed_insurance_rows_python.csv` produced by notebook 03. Where an `output_folder` is defined, it refers to `output/` at the repository root, not a folder inside `code/`.

To inspect the supplied Python analysis results, notebook 04 can use the included prepared CSVs directly. To rebuild them from source, run all four notebooks in order with the three raw inputs available. The OECD input is supplied as an approximately 11.1 MiB gzip-compressed CSV. Notebook 01 reads it directly without manual extraction or Git LFS. See the [raw-data availability notes](dataset_raw/README.md#obtaining-the-inputs).

The PDF findings were produced using the report's R implementation. The published notebooks use a Python implementation of imputation. The two implementations produce some different estimates, so the Python workflow should not be assumed to reproduce every PDF figure exactly. The R source and LaTeX build files are outside this publication package.

Path portability and notebook 02 execution were checked in a relocated temporary folder without the unused age-65+ population source. A full rerun of all four notebooks has not been performed as part of preparing this README.

## Interpretation and limitations

- **Units differ:** premiums are nominal USD, while GDP per capita uses constant PPP international dollars. Their trends are not directly comparable real-growth measures, and dividing the two does not produce a consistently defined insurance-penetration ratio.
- **Coverage is imperfect:** partial reporting and unresolved gaps remain. The country-selection threshold is a project choice and can affect the findings.
- **Imputation adds uncertainty:** single estimates are used, and the reported confidence intervals do not incorporate imputation uncertainty. Log-scale estimates are exponentiated without a mean-bias correction.
- **Premium definitions matter:** GWP includes accepted reinsurance and can contain economic overlap. Direct premiums per capita do not measure an average resident's insurance spending.
- **Per-capita comparisons share population scaling:** strong correlations alone do not establish a behavioral income relationship.

## Tools

Python · Jupyter Notebook · pandas · NumPy · Matplotlib · SciPy · statsmodels

The written report was developed with R and LaTeX; the notebooks in this repository run with Python.

## AI assistance disclosure

AI assistance, including OpenAI Codex, was used for parts of code generation, refactoring, debugging and documentation. The author remains responsible for the project's analytical decisions, interpretation and final published content. The scope of verification and remaining reproducibility limitations are described in the running instructions above.
