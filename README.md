# India Air Quality Analysis — Impact Metrics
### 🥇 First Prize — Impact Metrics, Jagriti 2026 | IIT (BHU) Varanasi

> **Team TBD** | IIT Delhi  
> **Aayush Agarwal** *(my contributions: Station Performance & Data Quality Audit module + Excel Dashboard)*

---

## Competition

**Impact Metrics** is the data analytics track of Jagriti, the annual socio-awareness festival of IIT (BHU) Varanasi. The problem: analyse five years of India's air quality data across 230+ monitoring stations and 26 cities, surface actionable insights, and present to a panel of judges.

**Judging:** 40% presentation · 30% Q&A · 30% communication. Shortlisted teams presented offline on 8 February 2026.

---

## Dataset

Five years (2015–2020) of hourly and daily observations from the Central Pollution Control Board (CPCB):

| File | Granularity | Description |
|------|------------|-------------|
| `station_hour.csv` | Hourly, per station | Primary analysis file — 230+ stations |
| `station_day.csv` | Daily, per station | — |
| `city_hour.csv` | Hourly, per city | — |
| `city_day.csv` | Daily, per city | Aggregated city-level |

**Pollutants:** PM2.5, PM10, NO, NO₂, NOₓ, SO₂, CO, O₃, NH₃, Benzene, Toluene, Xylene, AQI

> Dataset not included (large CSVs). See the Google Drive link in the bibliography section of `report_station_performance.docx`.

---

## Project Structure

```
.
├── station_performance_audit.ipynb   # My module: data quality audit pipeline (Colab)
├── dashboard.xlsm                    # My contribution: interactive Excel dashboard
├── report_station_performance.docx   # My contribution: Station Performance audit report
├── report_full.docx                  # Full team report (all modules)
└── README.md
```

---

## My Contributions

### 1. Station Performance & Data Quality Audit

A chunked, memory-efficient pipeline (`station_performance_audit.ipynb`) that processes the full `station_hour.csv` in 200,000-row chunks — necessary given the dataset size. The audit computes four metrics for every station × season × pollutant combination:

#### Metrics Defined

**Completeness**
```
Completeness = Actual Readings / Expected Readings
```
Measures sensor coverage — what fraction of expected hourly readings were actually reported.

**Validity**
```
Validity = (Actual Readings − Invalid Readings) / Actual Readings
```
Validity checks applied:
- All values must be non-negative
- PM2.5 < PM10 (physical constraint — fine particles are a subset of coarse)
- NO + NO₂ ≤ NOₓ (chemical constraint)
- AQI must match CPCB sub-index calculation within ±50 tolerance
- Empirical upper bounds derived from CPCB breakpoints scaled 2–4× (e.g. PM2.5 ≤ 1000 µg/m³)

**Uptime (Operational Period)**
```
Uptime = 1 − (Outage Hours / Total Hours)
```
An outage is defined as a contiguous block of rows where all pollutant readings are zero or null. Outage blocks are tracked across chunk boundaries to avoid false splits.

**Reliability Score**
```
Score = 0.5 × Completeness + 0.3 × Validity + 0.2 × Uptime
```
Weights reflect that completeness and validity directly affect analytical integrity, while uptime is a secondary operational concern.

#### Key Implementation Details

- **Chunked streaming:** avoids loading the full CSV into memory; state (outage blocks, running counts) is carried across chunks
- **Cross-chunk outage tracking:** `last_status`, `last_run_length`, `last_season` dictionaries bridge chunk boundaries so a multi-hour outage straddling two chunks is counted as one block
- **CPCB AQI recomputation:** `subindex()` implements the official linear interpolation formula; `detect_aqi_mismatch()` flags sensor rows where reported AQI deviates from recomputed value by >50

#### Selected Findings

| Dimension | Key Finding |
|-----------|------------|
| Season | Monsoon: most accurate readings but highest outage frequency — likely power disruption |
| State | Best: Chandigarh (highest completeness + uptime). Worst: Gujarat, Jharkhand, Tamil Nadu |
| Pollutant | Xylene completeness: ~20% — sensors either absent or non-functional across most stations |
| AQI | Widespread mismatch — rows reporting AQI with no pollutant readings at all |
| Anomaly | Ahmedabad CO readings consistently 10–100× above CPCB thresholds — confirmed sensor fault; AQI recalculated excluding CO sub-index |

---

### 2. Excel Dashboard

An interactive Excel dashboard (`dashboard.xlsm`) built on the output CSVs from the audit pipeline. Allows filtering by station, state, season, and pollutant to explore reliability scores, outage statistics, and completeness/validity breakdowns.

---

## Full Team Analysis (report_full.docx)

The complete project covered five modules:

| Module | Description |
|--------|-------------|
| Preprocessing & Imputation | Gap-aware multi-layer imputation (forward fill → station median → city median → seasonal median) across all four data files |
| Hotspot Identification | Top stations/cities by average AQI and percentage of bad air days; seasonal decomposition (peak: January +70 AQI, trough: August −67 AQI) |
| Pollutant Correlation & Source Attribution | Pearson correlation matrix; PM2.5/PM10, NO₂/NOₓ, PM2.5/SO₂ ratios as source indicators; k-means clustering of 26 cities into 11 pollution profiles |
| Impact Metrics & Public Health | WHO-based exposure and intensity scores; Risk Score = Intensity × (1 + Exposure); YoY ΔR for each city |
| **Station Performance & Audit** | **My module — see above** |

---

## Policy Recommendations

1. **Power infrastructure** — primary driver of monsoon outages; backup power at high-priority stations
2. **Sensor upgrades** — Toluene, NH₃, Xylene sensors largely non-functional; completeness under 20% is not analytically usable
3. **Targeted funding** — Gujarat, Jharkhand, Tamil Nadu show consistently poor reliability across all metrics
4. **Dormant station reactivation** — metadata shows multiple stations offline; Madhya Pradesh has only one active station
5. **AQI recalculation protocol** — rows with no pollutant readings should not carry an AQI value; this corrupted aggregate rankings

---

## Technical Stack

- **Python:** Pandas, NumPy, Matplotlib — Colab environment
- **Data processing:** chunked streaming, defaultdict-based nested aggregation
- **Visualisation:** PivotTables, Excel dashboard (xlsm)
- **Standards:** CPCB AQI breakpoints and sub-index formula; WHO pollutant exposure guidelines
