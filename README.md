# First Responder Response Times in North American Cities

How long does it take emergency responders to arrive after being dispatched? This project compares 2025 response times (arrival time minus dispatch time) across **Waterloo, New York City, Seattle and San Francisco**. Each city's open 911/dispatch data was cleaned in Python. We then designed a relational database and executed it on MySQL. The final relational database is 725,481-rows.

Group project for DATA 604 (MDSA program).

## Key findings

| City          | Calls analyzed (2025) | Mean response | Median response |
|---------------|----------------------:|--------------:|----------------:|
| San Francisco |               272,634 |       7.0 min |         4.6 min |
| Seattle       |               146,831 |       7.3 min |         6.7 min |
| Waterloo      |               106,016 |      29.9 min |        10.0 min |
| New York City |             2,450,345 |      39.8 min |        11.4 min |

- **Waterloo and New York were over 5× slower on average** than Seattle and San Francisco. The two West Coast cities also varied the least from month to month.
- **Snow wasn't the cause of slow months.** In both snowy cities, January and December were among the *fastest* months.
- **Waterloo slowed down in June and September.** We think September is the return of university students, and June lines up with the Bleams Road construction project (April–June 2025).
- **New York's slowest month was July**, which lines up with the July 2025 flash flooding.
- **Means overstate typical waits.** A small number of very long calls pull the averages up (Waterloo's longest call took about 4.9 days), so the median is the better "typical" number.

## Data

All four datasets come from each city's open data portal and were filtered to the 2025 calendar year.

| City | Source | License | Original size | Cleaned |
|----|----|----|----|----|
| Waterloo | [WRPS 2025 Occurrence Data](https://wrps.ca/resource/2025-occurrence-data) | Waterloo Open Data User Licence | 292,876 rows × 23 cols | 106,016 rows |
| New York City | [NYPD Calls for Service](https://data.cityofnewyork.us/Public-Safety/NYPD-Calls-for-Service-Year-to-Date-/n2zq-pubd) | Public domain | 54.9M rows × 20 cols (2018–2026) | 2,450,345 rows |
| Seattle | [Seattle Call Data](https://data.seattle.gov/Public-Safety/Call-Data/33kz-ixgy/about_data) | Public domain | 11M rows × 48 cols | 146,831 rows |
| San Francisco | [SFFD & EMS Dispatched Calls](https://data.sf.gov/Public-Safety/Fire-Department-and-Emergency-Medical-Services-Dis/nuek-vuh3/about_data) | See DataSF portal | 7.5M rows × 36 cols | 272,634 rows |

Every cleaned file has the same 7 columns:

| Column                     | Description                                |
|----------------------------|--------------------------------------------|
| `city`                     | City name (used to combine the tables)     |
| `dispatchdateandtime`      | When a unit was dispatched                 |
| `arrivaldateandtime`       | When the unit arrived on scene             |
| `response_time`            | Arrival minus dispatch, in **seconds**     |
| `finalcalltypedescription` | Call type (e.g. alarms, traffic collision) |
| `latitude`, `longitude`    | Incident location                          |

> **The NYC file isn't in this repo.** At 295 MB it's over GitHub's 100 MB file limit. Download the 2025 data from the NYC link above and save the cleaned version as `data/nyc_2025_clean_w_response_time.csv`.

## Method

1.  **Clean each city in pandas.** Filter to 2025, calculate `response_time`, drop missing or invalid times and keep the 7 shared columns.
2.  **Load into MySQL** with SQLAlchemy (`DataFrame.to_sql`), one table per city.
3.  **Combine with `UNION ALL`** into a single table. NYC was randomly sampled down to 200,000 rows (`random_state=42`) to fit the JupyterHub VM's memory, giving 725,481 rows in total.
4.  **Analyze with SQL.** Aggregates, `GROUP BY` division and month, and nested queries for seasonal comparisons.

**Tools:** Python, pandas, SQLAlchemy, MySQL/MariaDB, Jupyter.

## Repository structure

```         
├── code/
│   ├── GroupProjectFirstResponder.ipynb   # loads all 4 cities into MySQL and unions them
│   └── milestone_hand_in.ipynb            # Waterloo deep-dive: cleaning + SQL queries
├── data/
│   ├── waterloo_clean.csv
│   ├── seattle_2025_cleaned.csv
│   ├── san_francisco_cleaned.csv
│   └── nyc_2025_clean_w_response_time.csv  # not tracked; see Data section
└── presentations/
    ├── 604_proposal.pdf
    └── 604_presentation.pptx
```

## How the project changed from the proposal

- **Toronto was replaced with San Francisco**, so the comparison isn't Canada-only.
- **The construction-zone analysis was dropped.** Matching incident coordinates to construction areas was too complex, and complete construction records weren't available for every city.

## Team

Dawson Conlon · Ricky Doucette · Sagar Gupta · Qahir Ramji

## References

- 2025 Occurrence Data \| Waterloo Regional Police. <https://wrps.ca/resource/2025-occurrence-data>
- Call Data \| City of Seattle Open Data Portal. <https://data.seattle.gov/Public-Safety/Call-Data/33kz-ixgy/about_data>
- NYPD Calls for Service (Year to Date) \| NYC Open Data. <https://data.cityofnewyork.us/Public-Safety/NYPD-Calls-for-Service-Year-to-Date-/n2zq-pubd>
- Fire Department and EMS Dispatched Calls for Service \| DataSF. <https://data.sf.gov/Public-Safety/Fire-Department-and-Emergency-Medical-Services-Dis/nuek-vuh3/about_data>
- Bleams Road and Trussler Road \| Region of Waterloo. <https://www.regionofwaterloo.ca/programs-and-services/roads-and-transportation/road-construction/construction-projects/bleams-road-and-trussler-road/>
- Holthaus, E. (2025, July 16). How climate crisis makes rainstorms that flooded New York more common. *The Guardian*. <https://www.theguardian.com/us-news/2025/jul/16/new-york-floods-storm-climate-change>

*Analysis and conclusions are the authors' own and do not represent WRPS or any city agency.*
