# Charged by Income? E-Mobility and Income in German Districts

**Is the spread of battery electric cars (BEVs) in Germany linked to income?**
This project combines five open government datasets into a district-level panel (399 districts × 10 years, 2017–2026) and analyses the relationship between the BEV share of all passenger cars and the median income per district.

Final project of the Data Analyst bootcamp at WBS Coding School.

**Interactive dashboards:** [Tableau Public](LINK_TO_YOUR_TABLEAU_PUBLIC_VIZ)

---

## Key findings

| | |
|---|---|
| **BEV boom** | The BEV share rose from 0.07 % (2017) to 4.1 % (2026): about 2 million BEVs, 60 times as many as in 2017. |
| **East–West gap** | In 2026 the share is 4.4 % in the West and 2.5 % in the East, and the gap is growing. |
| **Strong link with income** | Pearson r = 0.72 between median income and BEV share (2026), up from 0.64 in 2022. Spearman ρ = 0.68. |
| **Not just East vs. West** | The relationship also holds within regions: r = 0.54 (West) and r = 0.84 (East). |
| **Income groups** | The richest fifth of districts reaches 5.2 % BEVs, the poorest fifth 2.3 %. The poorest districts are more than three years behind. |
| **Charging is not the bottleneck** | The poorest districts have fewer public charging points per inhabitant (179 vs. 269 per 100,000), but fewer BEVs per charging point (7.5 vs. 10–12). |
| **Outliers** | Carmaker and fleet hubs (e.g. Wolfsburg, 12.5 %) have very high car density (> 750 cars per 1,000 inhabitants). Without them, r rises to 0.75. |

**Conclusion:** So far, e-mobility in Germany is largely a question of income. Targeted purchase incentives for lower-income regions are likely to be more effective than expanding charging infrastructure there first.

---

## Data sources

All data is open and free to download.

| Dataset | Publisher | Used for | Reference date |
|---|---|---|---|
| [FZ 1 – Vehicle stock by registration district](https://www.kba.de/DE/Statistik/Kalender/fz1_01_01_2026.html) | Federal Motor Transport Authority (KBA) | Passenger cars and BEVs per district | 1 Jan, 2017–2026 |
| [Earnings statistics (Entgeltstatistik)](https://statistik.arbeitsagentur.de/DE/Navigation/Statistiken/Fachstatistiken/Beschaeftigung/Entgelt/Entgelt-Nav.html) | Federal Employment Agency (BA) | Median gross monthly earnings of full-time employees by place of residence and by place of work | 31 Dec, 2016–2025 |
| [Regional database, table 12411-01-01-4](https://www.regionalstatistik.de/) | Statistical Offices of the Federation and the Länder (Destatis) | Population per district | 31 Dec, 2016–2025 |
| [Charging station register (Ladesäulenregister)](https://www.bundesnetzagentur.de/ladeinfrastruktur) | Federal Network Agency (BNetzA) | Public charging points per district, by commissioning date | as of Sep 2026 |
| [Administrative areas VG250](https://gdz.bkg.bund.de/) | Federal Agency for Cartography and Geodesy (BKG) | District boundaries for maps | current |

### Raw files used

| Folder in `RAW/` | File(s) | Notes |
|---|---|---|
| `kba_stock/` | `kba_fz1_2017.xlsx` … `kba_fz1_2026.xlsx` | FZ 1, one file per year (renamed). From 2020 on, sheet `FZ1.2`; before, sheet `Pkw`. 2017 is only published as PDF and was transferred to Excel. |
| `income/` | `entgelt-d-0-202012-xlsx.xlsx`, `entgelt-dwolk-0-202512-xlsx.xlsx` | Regional tables, editions Dec 2020 and Dec 2025. Sheet 8.2: place of work; sheet 9.2: place of residence. |
| `population/` | `12411-01-01-4.csv` | Population by sex, districts, 31 Dec |
| `charging_points/` | `Ladesaeulenregister_BNetzA_2026-09-01.csv` | Full register, as of 1 Sep 2026 |
| `area_boundaries/` | `vg250-ew_12-31.utm32s.shape.ebenen/` | VG250-EW, as of 31 Dec, UTM32s, shapefile by levels (unzipped as downloaded); district layer `VG250_KRS` |

---

## Methodology

- **Unit of analysis:** district (Kreis / kreisfreie Stadt) per year, linked via the 5-digit official district key (AGS).
- **Year alignment:** `year` is the state at the start of the year. KBA stock is as of 1 January; population and income as of 31 December are assigned to the following year.
- **Harmonised territories:** Three cases are merged so the time series stays comparable: Eisenach → Wartburgkreis, Hanau → Main-Kinzig-Kreis, Trier-Saarburg → Trier. Counts are summed, and incomes are combined as a simple mean of the two medians.
- **Income measure:** the median by place of residence is the main measure, because cars are registered where their owners live (available 2022–2026). The median by place of work (2017–2026) is used as a robustness check.
- **Shares:** BEV shares for groups and regions are weighted (`sum(BEV) / sum(cars) × 100`), not averaged across districts.
- **Income groups:** quintiles of districts by median income, formed per year (`pd.qcut`).
- **Correlation:** Pearson (main) and Spearman (robustness), per year and per region.
- **High car density:** more than 750 passenger cars per 1,000 inhabitants (2026). This flags districts where company and fleet registrations are likely to distort the BEV share.
- **Charging points:** counted per district and year from the commissioning date in the register.
- **Validation:** District totals were checked against the KBA national totals. In 2026 the sum is 2,031,870 BEVs, compared with 2,034,260 nationally; the 0.1 % difference comes from vehicles not assigned to a district.

---

## Repository structure

```
charged-by-income/
├── RAW/                         # original downloads (not included, see below)
│   ├── area_boundaries/
│   ├── charging_points/
│   ├── income/
│   ├── kba_stock/
│   └── population/
├── CLEAN/                       # cleaned tables, one per source + final panel
│   ├── kba_stock.csv
│   ├── population.csv
│   ├── income_residence.csv
│   ├── income_workplace.csv
│   ├── charging_points.csv
│   └── panel_districts.csv      # 399 districts × 10 years = 3,990 rows
├── OUTPUT/                      # analysis results and Tableau input
│   ├── bev_share_trend.csv
│   ├── correlation_by_year.csv
│   ├── bev_share_by_income_group.csv
│   ├── districts.geojson
│   └── tableau_panel.csv
├── Presentation_Backup/         # screenshots of all dashboards
├── notebooks/
│   ├── 01_read_kba_stock.ipynb
│   ├── 02_read_population.ipynb
│   ├── 03_01_read_income_residence.ipynb
│   ├── 03_02_read_income_workplace.ipynb
│   ├── 04_read_charging_points.ipynb
│   ├── 05_build_panel.ipynb
│   ├── 06_analysis.ipynb
│   └── 07_prepare_maps.ipynb
└── README.md
```

The notebooks are numbered and run in order: 01–04 read and clean one source each, 05 merges them into the panel, 06 runs the analysis, and 07 prepares the map and the Tableau input.

---

## Dashboards (Tableau Public)

1. **BEV boom:** KPI tiles, choropleth map of the BEV share by district, East vs. West trend 2017–2026, year slider.
2. **Income:** scatter plot of income vs. BEV share with trend line, BEV share by income group, correlation by region, year slider (2022–2026).
3. **Background:** charging infrastructure by income group, top 10 districts with car density flag, key findings and limitations.

---

## How to reproduce

1. Download the raw files from the sources above into the matching `RAW/` subfolders.
2. Create the environment:
   ```bash
   conda create -n WBS_final_project python=3.14
   conda activate WBS_final_project
   conda install pandas numpy geopandas openpyxl jupyter
   ```
3. In the first cell of each notebook, set `PROJECT_DIR` to the path of your local project folder.
4. Run the notebooks in `notebooks/` in numerical order.
5. Load `OUTPUT/tableau_panel.csv` and `OUTPUT/districts.geojson` into Tableau Public and relate them on `district_id`.

The raw data is not included in this repository because of file size. All files can be downloaded from the sources above.

---

## Limitations

- **Correlation, not causation.** Income may stand in for other factors such as housing type, commuting distance or urbanity.
- **District-level data.** No conclusions about individual households (ecological fallacy).
- **Income covers full-time employees subject to social insurance only**, excluding pensioners and the self-employed.
- **Company cars** are registered at company locations and distort some districts.
- **Charging register** contains only reported public charging points. Private wallboxes are missing, and decommissioned points are no longer listed.
- **Census 2022** corrected the population figures, which creates a small break in per-capita values.

---

## Tools

Python (pandas, NumPy, geopandas) · Jupyter · Tableau Public

## Author

**Waldemar Blum**, Data Analyst bootcamp, WBS Coding School, November 2026
