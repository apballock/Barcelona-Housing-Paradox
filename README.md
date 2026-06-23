# The Barcelona Housing Paradox

A data-driven policy analysis examining why Barcelona's 2023 rent regulation succeeded on paper while the lived cost of housing in the city continued to rise. The project combines official government data, independent market data, peer-reviewed academic research, and a structured six-year personal case study to document a pattern this analysis terms *tenancy migration*: a reconfiguration of rental contracts that moves housing supply away from regulated, long-term tenancy and toward seasonal, room-based, and short-term arrangements that fall outside the law's reach.

The full analytical writeup is available as a standalone policy brief: [`Barcelona_Housing_Paradox_Policy_Brief.pdf`](./Barcelona_Housing_Paradox_Policy_Brief.pdf). This README documents the technical pipeline behind it.

## The question

Barcelona's official rent index and the city's actual asking-price market tell two different stories after 2023. INCASÒL's registered contract data shows rent growth slowing in line with the new regulation. Idealista's advertised market prices show the opposite — accelerating growth that reaches a 52% gap against the official figure by 2025. This project investigates that gap using public data, asking not only *whether* the regulation worked, but *for whom*, and through *which specific mechanisms* the rest of the market kept moving.

## Data sources

The analysis draws on official datasets rather than a single pre-cleaned source, which is where most of the early work in this project went:

- **INCASÒL** (Generalitat de Catalunya) — annual and quarterly rent-per-m² data by district and neighbourhood, 2000–2025
- **Idealista** — monthly historical asking-price series for Barcelona, 2008–2026
- **Sindicat de Llogateres** — independent tenants' union assessment of the regulation's two-year impact
- **Ajuntament de Barcelona** — municipal census (padrón) demographic data, 2025
- **Spain's Permanent Migration Observatory** — digital nomad residency permit statistics
- **Orozco-Martínez & Gil-Alonso (2024)**, *Critical Housing Analysis*, and **Cocola-Gant, Hodkinson & Janoschka (2025)**, *European Urban and Regional Studies* — peer-reviewed research independently confirming the contract-migration pattern this project identifies from first principles

None of these arrived analysis-ready. The INCASÒL spreadsheets carried metadata rows, footnotes, and an `'nd'` placeholder for missing values that silently converted entire numeric columns to text. The Idealista historical series came as a scraped HTML table with Spanish month names and comma decimal separators. Reconciling these — and deciding what to do when a source simply didn't exist (no public series tracks Barcelona's room rental prices over time) — was a substantial part of the work, and is documented step by step in the notebook.

## Pipeline

**`barcelona_housing_analysis.ipynb`** covers data acquisition through analysis:

- Loading and cleaning the INCASÒL workbook, including handling header rows embedded mid-file, an inconsistent `'nd'` missing-value code, and mixed string/integer year columns
- Reshaping from wide format (years as columns) to long format for time-series analysis, using `melt`
- Splitting district-level and neighbourhood-level series, since they cover different date ranges and granularities
- Parsing and cleaning the Idealista monthly series — translating Spanish month names, converting comma decimals, and aggregating to annual averages for direct comparison against INCASÒL
- Merging the two price series on year to produce the central INCASÒL-vs-Idealista comparison dataset
- Exploratory analysis: top and bottom-priced neighbourhoods, decade-long trend visualisation, and a five-neighbourhood comparison selected on documented tourism, coliving, and digital-nomad concentration rather than on price alone

Each transformation step is annotated with the reasoning behind it, not just the operation — for instance, why neighbourhood-level data was prioritised over district-level despite its shorter time range, and why Barcelona's citywide aggregate was preserved separately rather than discarded once the analysis moved to neighbourhood granularity.

## Dashboard

**`barcelona-housing-paradox-graphics.pdf`** (exported from Power BI) presents the analysis across five pages:

1. **The Crisis in Numbers** — citywide KPIs and the 2014–2025 rent trend
2. **The Paradox** — INCASÒL vs Idealista price divergence, and rent before/after the regulation by neighbourhood
3. **The Neighbourhood Story** — the five tracked neighbourhoods over time, plus the most and least expensive areas in 2025
4. **The Loophole** — seasonal contract share and absolute volume, 2023–2024
5. **Enforcement** — price growth before and after regulation, and the sanctions funnel from complaints filed to penalties confirmed

Building this in Power BI surfaced a recurring locale problem: a Spanish-locale Power BI install reads decimal points as thousands separators by default, which silently turned `11.18` into `1118` on import. The fix that held up across every table was converting numeric columns to string in pandas before export, then doing the decimal-comma replacement inside Power Query — a workaround documented in the notebook for the next dataset that hits the same wall.

## From data to policy

The policy brief itself follows a deliberate structure: each analytical chapter separates *evidence* (what the data shows), *interpretation* (what this analysis infers from it, with its limits stated explicitly), and *mechanism* (the plausible causal pathway connecting the two) — a structure adopted specifically to avoid the most common failure mode of this kind of analysis, where correlation and timing get written as if they were proof. Where the data could only establish a well-timed correlation rather than isolated causation — as with the link between the 2023 regulation and the subsequent surge in seasonal contracts — the brief says so directly, and cites independent academic research that reached a parallel conclusion using the same government data source.

The brief also includes a structured first-person case study: eleven housing arrangements across six years, ten of them informal, presented explicitly as a *mechanism-exposing case* rather than as statistically representative evidence — one account that illustrates, at the level of lived experience, the same gap between regulated and actual housing access that the aggregate data documents at the level of the city.

## Stack

Python (pandas) for acquisition and cleaning · Power BI (Power Query, DAX) for the dashboard · web research and citation verification for the academic and regulatory sources underpinning the policy brief.

## Files

- `barcelona_housing_analysis.ipynb` — full data pipeline and exploratory analysis
- `barcelona-housing-paradox-graphics.pdf` — five-page Power BI dashboard export
- `Barcelona_Housing_Paradox_Policy_Brief.pdf` — full written analysis and policy recommendations
- `*.csv` — cleaned intermediate datasets (district, neighbourhood, and comparison series)
