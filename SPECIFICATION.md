# Hotspot Policing & Demand Forecasting Demo — Application Specification

**Version:** 2.0
**Owner:** Simpson Associates
**File:** `index.html`
**Last Updated:** February 2025

---

## 1. Purpose & Overview

This document is the detailed specification for the Simpson Associates Hotspot Policing & Demand Forecasting demo web application. The app is a single-file HTML application that demonstrates how data analytics and AI can support evidence-based policing across all 43 police forces in England & Wales. It is built entirely on synthetic data and is designed for sales demonstrations, conference showcases, and stakeholder engagement.

### 1.1 Business Context

Simpson Associates is a data transformation consultancy. This demo positions the company as a credible supplier of analytics and AI solutions to UK policing, competing with vendors such as Telefonica Tech, Palantir, SAS, and Accenture. The demo is informed by publicly available research, government frameworks, and policing standards.

### 1.2 Key Design Decisions

| Decision | Rationale |
|---|---|
| Single HTML file | Zero-dependency deployment; can be opened from any file system, hosted on GitHub Pages, or shared via email |
| Synthetic data only | No real crime data is used; avoids GDPR and data sensitivity concerns |
| All 43 forces | Demonstrates national scale; every force sees their own area represented |
| Client-side rendering | No server required; all computation happens in the browser |
| Simpson Associates branding | Matches the company's production website for a professional, cohesive impression |

---

## 2. Technical Architecture

### 2.1 Technology Stack

| Component | Technology | Version | CDN Source |
|---|---|---|---|
| Mapping | Leaflet.js | 1.9.4 | cdnjs.cloudflare.com |
| Charts | Chart.js | 4.4.1 | cdnjs.cloudflare.com |
| Map tiles | CARTO | Dark (hero) / Light (analysis) | basemaps.cartocdn.com |
| Fonts | Silka (body) | woff2 | simpson-associates.co.uk |
| Fonts | JetBrains Mono (data) | Google Fonts | fonts.googleapis.com |

### 2.2 File Structure

The application is a single `index.html` file (~1,355 lines) organised into four logical sections:

```
index.html
├── <head>
│   ├── External resource imports (fonts, Leaflet CSS/JS, Chart.js)
│   └── <style> block (~260 lines)
│       ├── @font-face declarations (Silka Regular/Medium/SemiBold/Bold/Italic)
│       ├── CSS custom properties (:root variables)
│       ├── Component styles (header, hero, KPIs, grid, cards, maps, charts, etc.)
│       └── Responsive breakpoints (1100px, 700px)
├── <body>
│   ├── Header with SA logo SVG and top-level navigation
│   ├── Dashboard page (hero + 7 tabbed panels)
│   ├── Solutions page (6 solution cards)
│   ├── Case Studies page (4 case study cards)
│   ├── Contact page (demo booking form + office info)
│   └── Footer
└── <script> block (~390 lines)
    ├── Constants (CSS_WEIGHTS, CRIME_LABELS, REGIONS)
    ├── mkForce() factory function
    ├── All 43 force data definitions
    ├── NATIONAL aggregate calculations
    └── Application logic (navigation, rendering, charts, insights)
```

### 2.3 Branding & Design System

#### Colour Palette

| Token | Hex | Usage |
|---|---|---|
| `--navy` | `#2A4149` | Header, footer, dark backgrounds, body text |
| `--navy-2` | `#22353C` | Hero map wrapper, insight panel background |
| `--navy-3` | `#2E4A53` | Alternate dark surface |
| `--blue` | `#2C83D3` | Primary action, links, active tabs, chart primary |
| `--blue-d` | `#226A92` | Button hover state |
| `--blue-l` | `#5BA8E6` | Light blue accent |
| `--teal` / `--green` | `#10CA72` | SA green, success states, positive trends |
| `--teal-l` | `#3DD98D` | Hero stats, footer links, demo tag |
| `--purple` | `#8876E4` | SA purple, gradient start, tertiary accent |
| `--red` | `#EF4444` | Critical risk, negative trends, violence chart |
| `--orange` | `#F97316` | High risk, warning, orange chart series |
| `--amber` | `#EAB308` | Medium risk, caution |
| `--g50` | `#F3F1ED` | SA beige, page background |
| `--g100`–`--g900` | Various | Grey scale from light to dark |
| `--sa-gradient` | `linear-gradient(-37deg, #8876E4 0%, #1E8BC8 98%)` | Brand gradient (hero text, case study cards, logo mark) |

#### Typography

| Role | Font | Weights | Source |
|---|---|---|---|
| Body / UI | Silka | 400, 500, 600, 700, 400 italic | SA hosted woff2 files |
| Data / Numbers | JetBrains Mono | 500, 600 | Google Fonts |

#### Spacing & Elevation

| Token | Value |
|---|---|
| `--radius` | `10px` |
| `--shadow` | `0 1px 3px rgba(42,65,73,.06), 0 4px 12px rgba(42,65,73,.05)` |
| `--shadow-lg` | `0 8px 30px rgba(42,65,73,.1)` |

#### Logo

The header displays the official Simpson Associates white wordmark SVG, loaded inline. The SVG includes the geometric cube icon, "Simpson" text, the "i" dot, and "Associates" subtitle. Height: 28px, width: auto.

---

## 3. Data Architecture

### 3.1 Constants

#### Crime Severity Score Weights (`CSS_WEIGHTS`)

Based on ONS Crime Severity Score methodology, where weights approximate average days of custody sentences:

| Crime Type | Weight | Source Basis |
|---|---|---|
| Violence Against the Person | 5.1 | Custody sentencing data |
| Sexual Offences | 9.0 | Highest severity weighting |
| Robbery | 5.5 | High severity |
| Burglary | 2.5 | Moderate |
| Theft Offences | 1.3 | Low |
| Criminal Damage & Arson | 1.1 | Low |
| Drug Offences | 1.0 | Baseline |
| Possession of Weapons | 4.8 | High |
| Public Order | 0.9 | Low |
| Misc Crimes Against Society | 1.2 | Low |
| Fraud | 0.7 | Lowest |
| Computer Misuse (Cyber) | 0.5 | Lowest |

#### Crime Labels (`CRIME_LABELS`)

Maps short keys (e.g., `violence`, `theft`) to full Home Office Counting Rules (HOCR) category names. 12 categories covering the standard HOCR classification.

#### Regions (`REGIONS`)

The 10 standard policing regions of England & Wales:
1. North East (3 forces)
2. North West (5 forces)
3. Yorkshire and the Humber (4 forces)
4. East Midlands (5 forces)
5. West Midlands (4 forces)
6. Eastern (6 forces)
7. South East (5 forces)
8. South West (5 forces)
9. London (2 forces)
10. Wales (4 forces)

### 3.2 Force Data Factory (`mkForce`)

Each of the 43 forces is created via the `mkForce()` factory function. This accepts base parameters and computes derived KPIs, crime mix, and forecast data.

#### Input Parameters

| Parameter | Type | Description |
|---|---|---|
| `k` | string | Unique force key (e.g., `'gmp'`, `'met'`, `'cleveland'`) |
| `n` | string | Full force name (e.g., `'Greater Manchester Police'`) |
| `reg` | string | Region name |
| `typ` | string | Force type: `'urban'`, `'mixed'`, or `'rural'` |
| `lat`, `lng` | float | Centre coordinates for map view |
| `z` | int | Default map zoom level (9–14) |
| `pop` | int | Population served |
| `off` | int | Total officer headcount |
| `area` | int | Area in square kilometres |
| `cr` | float | Crime rate per 1,000 population |
| `dep` | int | Deprivation rank (1 = most deprived, 43 = least deprived) |
| `hs` | array | Array of hotspot objects (see below) |
| `tmpl` | array | Optional 12-value temporal template (hourly incident volumes) |

#### Hotspot Object Structure

Each hotspot represents a named location within the force area:

```javascript
{
  name: 'Piccadilly Gardens',       // Real place name
  lat: 53.4808,                     // Latitude (accurate coordinates)
  lng: -2.2369,                     // Longitude
  risk: 94,                         // Composite risk score (0-100)
  level: 'crit',                    // Risk level: 'crit' | 'high' | 'med' | 'low'
  types: ['violence', 'public-order', 'robbery'],  // HOCR crime types present
  context: 'NTE peak zone',         // Contextual description
  css: 412,                         // Crime Severity Score for this hotspot
  deprivation: 1                    // IMD deprivation decile (1 = most deprived)
}
```

Risk level thresholds:
- **Critical:** risk >= 80
- **High:** risk >= 60
- **Medium:** risk >= 40
- **Low:** risk < 40

#### Computed Properties

The factory function generates the following from base parameters:

| Property | Computation |
|---|---|
| `crimeMix` | Percentage breakdown by HOCR category, differentiated by force type (urban/mixed/rural profiles) |
| `baseInc` | `Math.round(pop * cr / 365000)` — estimated daily incident count |
| `kpis.predAccuracy` | `72 + random * 10` — ML prediction accuracy (72–82%) |
| `kpis.responseTime` | Urban: 6–10 min, Mixed: 7–12 min, Rural: 9–15 min |
| `kpis.crimeReduction` | `-(10 + random * 15)` — percentage reduction (-10% to -25%) |
| `kpis.publicConfidence` | `60 + random * 25` — percentage (60–85%) |
| `kpis.patrolCompliance` | `88 + random * 10` — percentage (88–98%) |
| `kpis.victimSatisfaction` | `72 + random * 16` — percentage (72–88%) |
| `severityScore` | `Math.round(baseInc * 2.8)` |
| `forecast.values` | 7-day forecast using day-of-week multipliers (Thu=1, Fri=1.33, Sat=1.52, Sun=1.09, Mon=0.84, Tue=0.82, Wed=0.89) |
| `forecast.upper/lower` | ±12% confidence interval around forecast values |
| `forecast.confidence` | Per-day confidence scores (0.70–0.82) |
| `temporal` | 12-value array (bi-hourly incident volumes), scaled from template by baseInc |

#### National Aggregates (`NATIONAL`)

Computed from all 43 forces on initialisation:

| Metric | Formula |
|---|---|
| `population` | Sum of all force populations |
| `officers` | Sum of all force officer headcounts |
| `totalHotspots` | Sum of all force hotspot counts |
| `avgCrimeRate` | Mean of all force crime rates |
| `avgAccuracy` | Mean of all force prediction accuracies |
| `avgSeverity` | Mean of all force severity scores |

### 3.3 All 43 Forces

| # | Key | Force Name | Region | Type | Hotspots |
|---|---|---|---|---|---|
| 1 | `cleveland` | Cleveland Police | North East | urban | 5 |
| 2 | `durham` | Durham Constabulary | North East | mixed | 5 |
| 3 | `northumbria` | Northumbria Police | North East | urban | 6 |
| 4 | `cheshire` | Cheshire Constabulary | North West | mixed | 5 |
| 5 | `cumbria` | Cumbria Constabulary | North West | rural | 5 |
| 6 | `gmp` | Greater Manchester Police | North West | urban | 8 |
| 7 | `lancashire` | Lancashire Constabulary | North West | mixed | 6 |
| 8 | `merseyside` | Merseyside Police | North West | urban | 6 |
| 9 | `humberside` | Humberside Police | Yorkshire | mixed | 5 |
| 10 | `north-yorkshire` | North Yorkshire Police | Yorkshire | rural | 5 |
| 11 | `south-yorkshire` | South Yorkshire Police | Yorkshire | urban | 6 |
| 12 | `west-yorkshire` | West Yorkshire Police | Yorkshire | urban | 7 |
| 13 | `derbyshire` | Derbyshire Constabulary | East Midlands | mixed | 5 |
| 14 | `leicestershire` | Leicestershire Police | East Midlands | mixed | 5 |
| 15 | `lincolnshire` | Lincolnshire Police | East Midlands | rural | 5 |
| 16 | `northamptonshire` | Northamptonshire Police | East Midlands | mixed | 5 |
| 17 | `nottinghamshire` | Nottinghamshire Police | East Midlands | urban | 5 |
| 18 | `staffordshire` | Staffordshire Police | West Midlands | mixed | 5 |
| 19 | `warwickshire` | Warwickshire Police | West Midlands | mixed | 5 |
| 20 | `west-mercia` | West Mercia Police | West Midlands | rural | 5 |
| 21 | `west-midlands` | West Midlands Police | West Midlands | urban | 7 |
| 22 | `bedfordshire` | Bedfordshire Police | Eastern | mixed | 4 |
| 23 | `cambridgeshire` | Cambridgeshire Constabulary | Eastern | mixed | 4 |
| 24 | `essex` | Essex Police | Eastern | mixed | 6 |
| 25 | `hertfordshire` | Hertfordshire Constabulary | Eastern | mixed | 5 |
| 26 | `norfolk` | Norfolk Constabulary | Eastern | rural | 5 |
| 27 | `suffolk` | Suffolk Constabulary | Eastern | rural | 4 |
| 28 | `hampshire` | Hampshire Constabulary | South East | mixed | 5 |
| 29 | `kent` | Kent Police | South East | mixed | 6 |
| 30 | `surrey` | Surrey Police | South East | mixed | 4 |
| 31 | `sussex` | Sussex Police | South East | mixed | 5 |
| 32 | `thames-valley` | Thames Valley Police | South East | mixed | 6 |
| 33 | `avon-somerset` | Avon and Somerset Police | South West | mixed | 6 |
| 34 | `devon-cornwall` | Devon and Cornwall Police | South West | rural | 6 |
| 35 | `dorset` | Dorset Police | South West | rural | 4 |
| 36 | `gloucestershire` | Gloucestershire Constabulary | South West | mixed | 4 |
| 37 | `wiltshire` | Wiltshire Police | South West | rural | 4 |
| 38 | `met` | Metropolitan Police Service | London | urban | 8 |
| 39 | `city-of-london` | City of London Police | London | urban | 4 |
| 40 | `dyfed-powys` | Dyfed-Powys Police | Wales | rural | 5 |
| 41 | `gwent` | Gwent Police | Wales | mixed | 5 |
| 42 | `north-wales` | North Wales Police | Wales | mixed | 5 |
| 43 | `south-wales` | South Wales Police | Wales | urban | 6 |

**Total hotspots:** 224 named locations with accurate lat/lng coordinates.

---

## 4. Page & Navigation Structure

### 4.1 Top-Level Pages

The app has four top-level pages, toggled via the header navigation buttons:

| Page ID | Name | Description |
|---|---|---|
| `pg-dashboard` | Dashboard | Primary analytics interface with hero section and 7 tabbed panels |
| `pg-solutions` | Solutions | Marketing-oriented solution card grid (6 offerings) |
| `pg-casestudies` | Case Studies | 4 case study cards showcasing real SA engagements |
| `pg-contact` | Contact | Demo booking form with SA office locations |

Navigation is handled by `showPage(id, btn)` which toggles `.active` class on pages and tab buttons.

### 4.2 Dashboard Tabs

The dashboard page contains a secondary tab bar with 7 analytical panels:

| Panel ID | Tab Label | Content |
|---|---|---|
| `overview` | Overview | 5 KPI cards + 4 charts (crime trends, crime mix doughnut, risk doughnut, CSS trend) |
| `hotspots` | Hotspot Analysis | Force/crime/period filters, interactive Leaflet map with hexagonal hotspot overlays, ranked hotspot list, temporal bar chart |
| `forecast` | Demand Forecast | Force filter, 7-day forecast grid, demand trend line with 95% CI, category forecast bars, seasonal factors |
| `resources` | Resource Allocation | 6 resource deployment cards, Koper Curve evidence box, shift coverage vs demand chart, deployment efficiency radar |
| `comparison` | Force Comparison | Rank metric selector, region filter, multi-select comparison, sortable rankings table, radar overlay, deprivation scatter, force type bar chart |
| `peel` | PEEL Alignment | Force filter, 3 PEEL pillar panels (Effectiveness, Efficiency, Legitimacy) with metrics, HMICFRS alignment text blocks |
| `insights` | AI Insights | Force/priority filters, dynamically generated insight cards with priority badges, confidence scores, and source attribution |

---

## 5. Feature Specifications

### 5.1 Hero Section

- Full-width dark background with radial gradient overlay
- Headline: "Hotspot Policing & Demand Forecasting" (gradient text using `--sa-gradient`)
- 4 summary statistics: 43 Forces, 17% Avg Crime Reduction, 76.4% Prediction Accuracy, 2.3x ROI
- Embedded Leaflet map showing national overview with circle markers per force (sized by population, coloured by max risk)
- Live indicator: "Synthetic demo data" with animated pulse dot

### 5.2 KPI Cards (`renderKPIs`)

Displays 5 KPIs in a responsive grid. Adapts between national and force-specific views:

**National view:** Population covered, Active hotspots, Total officers (with ratio), Avg crime rate, Avg prediction accuracy.

**Force view:** Daily incidents (with rate), Active hotspots (with force type), Officers deployed (of total), Prediction accuracy, Response time (with target comparison).

Each card has a coloured top border, JetBrains Mono value, label, and trend indicator (up/down/neutral with directional colour coding: red = bad, green = good).

### 5.3 Interactive Hotspot Map (`renderHotspots`)

**Map engine:** Leaflet.js with CARTO Light tiles.

**Features:**
- Hexagonal grid overlays per hotspot (generated via `createHexagon()` function using 6-point polygon calculation)
- Hexagon size scales by risk score (default) or CSS value (when severity toggle is active)
- Colour coding: Critical = red, High = orange, Medium = amber, Low = green
- Circle markers at hotspot centres with white borders
- Rich tooltips showing: name, force, risk score, CSS, crime types, context, deprivation decile
- Fixed legend overlay (bottom-left) with risk level scale
- Clicking a hotspot list item zooms to the location at zoom level 15

**Filters:**
- Force selector (grouped by region, with "All Forces" national option)
- Crime type filter (10 HOCR categories)
- Period filter (7, 28, 90 days — UI only, does not alter synthetic data)
- Severity view toggle (switches hexagon sizing between risk score and CSS value)

**Hotspot List Panel:**
- Ranked by risk score (descending), max 20 displayed
- Shows rank badge (coloured by level), name, crime types, context, force name
- JetBrains Mono risk score and CSS value per item
- Scrollable list (max-height: 440px)

### 5.4 Crime Trend Charts (`initCharts`)

#### 5.4.1 Crime Trends (12 Months)
- **Type:** Multi-series line chart (filled)
- **Series:** Violence (red), Theft (SA blue), Criminal Damage (orange), Drug Offences (SA purple)
- **Toggle:** "Severity weighted" checkbox switches to CSS Trend chart view
- **Y-axis:** Compact labels (e.g., "4k" instead of "4000")

#### 5.4.2 Crime Mix by Category
- **Type:** Doughnut chart (60% cutout)
- **Data:** 12 HOCR categories with percentage breakdown
- **Legend:** Right-positioned with point-style circles

#### 5.4.3 Hotspot Risk Distribution
- **Type:** Doughnut chart (62% cutout)
- **Data:** Aggregated from all 224 hotspots (critical/high/medium/low counts)

#### 5.4.4 Crime Severity Score Trend
- **Type:** Multi-series line chart (filled)
- **Series:** Total CSS (SA blue), Violence CSS (red), Robbery CSS (orange)
- **Y-axis:** "Severity Score (weighted days)"

### 5.5 Temporal Pattern Analysis
- **Type:** Bar chart
- **Data:** 12 bi-hourly time slots (00:00–22:00)
- **Colour:** Dynamic per-bar based on volume thresholds (>150: red, >100: orange, >70: amber, else: green)
- **Updates:** Reflects selected force's temporal data when a force is filtered

### 5.6 Demand Forecast (`renderForecast`)

#### 7-Day Forecast Grid
- 7-column card grid showing each forecast day
- First card styled as "Today" (blue background)
- Per day: day name, date, predicted value (JetBrains Mono, large), % vs average, confidence interval range
- Colour-coded: >20% above avg = red, >5% = orange, <-10% = green

#### Forecast Charts
- **Demand Trend & Confidence Band:** Line chart with actual (solid blue), predicted (dashed green), 95% CI upper/lower (filled green band). CI labels filtered from legend.
- **Crime Type Predictions:** Grouped bar chart — current week vs forecast next week across 7 HOCR categories
- **Seasonal & Event Demand Factors:** Combined bar + line chart. Bars show seasonal index (colour-coded by threshold), line shows NTE multiplier.

### 5.7 Resource Allocation

#### Deployment Cards
6 resource types with name, value, and operational description:
- Foot Patrol Officers (186), Response Vehicles (42), PCSOs (94), Control Room Staff (28), Specialist Units (16), Special Constables (45)

#### Koper Curve Evidence Box
Styled panel explaining the evidence base for optimal patrol duration (10–16 minutes), recommended 4 visits per shift, and the 14–17% crime reduction from the Home Office Hot Spot Response Programme.

#### Charts
- **Shift Coverage vs Demand:** Multi-line chart — predicted demand (red), current staffing (green), recommended (dashed blue)
- **Deployment Efficiency Radar:** 6-axis radar comparing "Current" vs "With Hotspot Analytics" across: Response Time, Area Coverage, Utilisation, Patrol Compliance, Cost Efficiency, Positive Outcomes

### 5.8 Force Comparison

#### Rankings Table (`renderRankings`)
- Sortable by 8 metrics: Crime Rate, Severity Score, Officer:Population Ratio, Prediction Accuracy, Response Time, Crime Reduction %, Public Confidence, Deprivation Index
- Filterable by region (10 regions + "All")
- Columns: Rank #, Force name, Region, Type (urban/mixed/rural), Metric value, Visual bar
- Scrollable container (max-height: 500px) with sticky header

#### Comparison Radar (`renderCompareRadar`)
- Multi-select (2–4 forces) radar overlay
- 6 normalised axes: Response Time, Prediction Accuracy, Patrol Compliance, Public Confidence, Crime Reduction, Victim Satisfaction
- 4 distinct colours per force (SA blue, red, SA green, SA purple)

#### Deprivation vs Crime Rate Scatter (`renderDeprivationChart`)
- X-axis: Deprivation rank (1 = most deprived)
- Y-axis: Crime rate per 1,000
- Colour-coded by force type: Urban (red), Mixed (orange), Rural (green)
- City of London excluded (outlier)
- Tooltips show force name, deprivation rank, crime rate

#### Crime Severity by Force Type (`renderForceTypeChart`)
- Combined bar + line chart
- Bars: Average severity score by type (Urban/Mixed/Rural)
- Line: Average crime rate per 1,000 (secondary y-axis)

### 5.9 PEEL Alignment (`renderPEEL`)

Structured around the three HMICFRS PEEL inspection pillars:

#### Effectiveness Panel (blue)
- Metrics: Crime Reduction, Repeat Victim Prevention, Serious Violence Reduction, Positive Outcomes
- Icon: Shield SVG

#### Efficiency Panel (green)
- Metrics: Resource Utilisation, Response Time, Cost per Incident, Patrol Compliance
- Icon: Clock SVG

#### Legitimacy Panel (purple)
- Metrics: Public Confidence, Stop & Search Fairness, Complaints Trend, Victim Satisfaction
- Icon: People SVG

#### HMICFRS Alignment Text
3-column grid explaining how hotspot analytics supports each PEEL pillar, referencing:
- College of Policing systematic review (16% crime reduction)
- Koper Curve evidence (10–16 minute optimal dosage)
- Procedural justice elements and ALGO-CARE ethical framework

### 5.10 AI Insights Engine (`generateInsights`)

Generates contextual insight cards dynamically based on force data:

#### National Insights (when "All Forces" selected)
9 insights covering:
1. **Critical:** National NTE surge — Friday evening violence risk
2. **High:** Cross-border drug supply pattern displacement
3. **High:** Deprivation-crime nexus (78% of critical hotspots in IMD decile 1–3)
4. **Medium:** Weekend demand spike (52% above weekday average)
5. **Medium:** Koper Curve compliance monitoring
6. **Low:** Seasonal forecast (February baseline)
7. **Low:** Model performance summary
8. **Medium:** ASB seasonal pattern
9. **High:** Weapons hotspot concentration (62% in 8 forces)

#### Force-Specific Insights (when a force is selected)
6 insights generated from that force's data:
1. **Critical:** Top hotspot alert with Koper patrol recommendation
2. **High:** NTE violence pattern with officer deployment recommendation
3. **High:** Deprivation correlation analysis
4. **Medium:** Forecast confidence and weekend peak
5. **Medium:** Resource optimisation with reallocation suggestion
6. **Low:** Seasonal trend for force type

Each insight card displays:
- Priority badge (Critical/High/Medium/Low with colour coding)
- Rich HTML text with bold highlights
- Metadata: time ago, confidence percentage, source attribution label

Filterable by priority level.

### 5.11 Solutions Page

6 solution cards in a 3-column grid:

| # | Solution | Icon Colour | Key Stats |
|---|---|---|---|
| 1 | Hotspot Policing Analytics | SA blue | 17% avg crime reduction, 14% violence reduction |
| 2 | Demand Forecasting | SA green | 76%+ prediction accuracy, 7-day horizon |
| 3 | Resource Allocation Engine | SA purple | 94% utilisation rate, 15 min optimal patrol |
| 4 | Data Analytics Accelerator | Amber | 12+ data sources, 33k LSOAs mapped |
| 5 | RedactXpert | Red | 50% time savings, 99.2% PII detection |
| 6 | PEEL Performance Framework | Green | 3 PEEL pillars, 12 KPI categories |

### 5.12 Case Studies Page

4 case study cards in a 2-column grid:

| # | Title | Gradient | Tag |
|---|---|---|---|
| 1 | Derbyshire Constabulary: Data Analytics Transformation | SA gradient (purple → blue) | Data Platform |
| 2 | South Wales & Gwent: Joint Data Analytics Programme | Blue gradient | Intelligence Led |
| 3 | Cleveland Police: AI-Powered Redaction | Red → navy gradient | AI Redaction |
| 4 | TOEX: National Capabilities Environment | Green → navy gradient | National Programme |

Each card includes a gradient header with icon, description text, 2 metric highlights, and a coloured tag badge.

### 5.13 Contact Page

**Form fields:**
- Name (text input)
- Email (email input, placeholder: `your.email@force.police.uk`)
- Force / Organisation (populated from all 43 forces via `populateForceDropdowns`)
- Area of Interest (select: 6 options from Hotspot Analytics to Full Suite Demo)
- Message (textarea)
- Submit button (triggers alert confirmation)

**Contact information cards:**
- York Office: Suite 3, 20 George Hudson Street, York YO1 6WR
- Sheffield Office: 3 Concourse Way, Sheffield City Centre, Sheffield S1 2BJ
- Phone: +44 (0) 1904 234 510
- Website: www.simpson-associates.co.uk
- Accreditations: Microsoft Partner of the Year 2024, ISO 27001, ISO 9001, Cyber Essentials Plus, Databricks Partner, Police Industry Charter

---

## 6. Policing Domain Knowledge References

The application content is grounded in the following UK policing frameworks and research:

| Reference | How It's Used |
|---|---|
| **College of Policing — Hot Spot Policing** | Evidence base for 16% average crime reduction; systematic review cited in PEEL alignment and Koper box |
| **Koper Curve (Koper, 1995)** | 10–16 minute optimal patrol dosage; referenced in Resource Allocation panel and AI insights |
| **Home Office Counting Rules (HOCR)** | 12-category crime classification used throughout all charts, filters, and data structures |
| **ONS Crime Severity Score (CSS)** | Severity weighting methodology; CSS_WEIGHTS constant, severity toggle on map, CSS Trend chart |
| **Home Office Hot Spot Response Programme** | All 43 forces participation; 14–17% crime reduction cited in Koper evidence box |
| **HMICFRS PEEL Framework** | Effectiveness, Efficiency, Legitimacy pillars structure the PEEL Alignment panel |
| **Index of Multiple Deprivation (IMD)** | Deprivation deciles per hotspot; deprivation vs crime scatter chart; AI insight on deprivation-crime nexus |
| **ALGO-CARE Framework** | Ethical AI framework referenced in Legitimacy alignment text |
| **Night-Time Economy (NTE)** | Pattern recognition for violence/sexual offences during 22:00–03:00; NTE multiplier in seasonal chart |
| **data.police.uk** | Referenced as data source in Analytics Accelerator solution card |

---

## 7. Responsive Design

Two breakpoints handle layout adaptation:

### 1100px (Tablet / Narrow Desktop)
- Hero grid collapses to single column
- KPI grid: 5 → 3 columns
- All grid spans collapse to span-12 (full width)
- PEEL grid, insight grid, solution grid: 3 → 1 column
- Case study / contact grids: 2 → 1 column
- National summary bar: 6 → 3 columns
- Hero stats: 4 → 2 columns

### 700px (Mobile)
- Hero heading: 3.2rem → 2rem
- KPI grid: 3 → 1 column
- Forecast grid: 7 → 4 columns
- Resource grid: 3 → 1 column
- Hero stats: 2 → 1 column
- Top navigation: hidden
- National bar: 3 → 2 columns

---

## 8. JavaScript Function Reference

| Function | Purpose | Triggered By |
|---|---|---|
| `populateForceDropdowns()` | Fills all `<select>` elements with 43 forces grouped by region | `DOMContentLoaded` |
| `showPage(id, btn)` | Toggles top-level page visibility | Header nav buttons |
| `initHeroMap()` | Creates national overview Leaflet map with force circle markers | `DOMContentLoaded` |
| `initMainMap()` | Creates main analysis Leaflet map | `DOMContentLoaded` |
| `createHexagon(center, sizeKm)` | Generates 6-point polygon coordinates for hexagonal overlay | `renderHotspots()` |
| `renderHotspots(forceKey, crimeFilter)` | Renders map markers, hexagons, hotspot list, and KPIs for selected force/crime | Filter changes, tab switches |
| `renderHotspotList(hotspots, forceName)` | Populates ranked hotspot sidebar list | `renderHotspots()` |
| `applyFilters()` | Reads filter values and calls `renderHotspots()` | Force/crime/period/toggle change events |
| `renderKPIs(force)` | Renders 5 KPI cards (national or force-specific) | `renderHotspots()`, `DOMContentLoaded` |
| `initCharts()` | Creates all Chart.js instances (11 charts total) | `DOMContentLoaded` |
| `renderForecast(forceKey)` | Populates 7-day forecast grid with day cards | Force selector change, `DOMContentLoaded` |
| `renderResources()` | Populates resource deployment cards | `DOMContentLoaded` |
| `renderPEEL(forceKey)` | Populates PEEL pillar metrics and alignment text | Force selector change, `DOMContentLoaded` |
| `renderRankings()` | Builds sortable force rankings table | Metric/region selector change, `DOMContentLoaded` |
| `renderCompareRadar()` | Creates multi-force radar overlay chart | Multi-select change |
| `renderDeprivationChart()` | Creates deprivation vs crime scatter plot | `DOMContentLoaded` |
| `renderForceTypeChart()` | Creates urban/mixed/rural comparison chart | `DOMContentLoaded` |
| `generateInsights(forceKey, priority)` | Generates AI insight cards (national or force-specific) | Force/priority selector change, `DOMContentLoaded` |

---

## 9. Chart Inventory

| # | Canvas ID | Chart Type | Panel | Description |
|---|---|---|---|---|
| 1 | `trendChart` | Line (multi-series, filled) | Overview | 12-month crime trends by category |
| 2 | `crimeMixChart` | Doughnut | Overview | National crime mix by HOCR category |
| 3 | `riskChart` | Doughnut | Overview | Hotspot risk level distribution |
| 4 | `cssTrendChart` | Line (multi-series, filled) | Overview | Crime Severity Score trend |
| 5 | `temporalChart` | Bar (dynamic colour) | Hotspot Analysis | Bi-hourly incident volume |
| 6 | `forecastChart` | Line (with CI band) | Demand Forecast | Actual vs predicted with 95% CI |
| 7 | `categoryChart` | Bar (grouped) | Demand Forecast | Current vs forecast by crime type |
| 8 | `seasonalChart` | Bar + Line (combo) | Demand Forecast | Seasonal index + NTE multiplier |
| 9 | `shiftChart` | Line (multi-series) | Resource Allocation | Demand vs staffing vs recommended |
| 10 | `effChart` | Radar | Resource Allocation | Current vs with-analytics efficiency |
| 11 | `compareRadar` | Radar (dynamic) | Force Comparison | Multi-force performance overlay |
| 12 | `deprivationChart` | Scatter | Force Comparison | IMD deprivation vs crime rate |
| 13 | `forceTypeChart` | Bar + Line (combo) | Force Comparison | Severity and crime rate by force type |

---

## 10. Event Listeners

| Element | Event | Handler |
|---|---|---|
| `.d-tab` buttons | `click` | Switch dashboard panel, invalidate map if switching to hotspots |
| `#filterForce` | `change` | `applyFilters()` |
| `#filterCrime` | `change` | `applyFilters()` |
| `#filterPeriod` | `change` | `applyFilters()` |
| `#cssMapToggle` | `change` | `applyFilters()` |
| `#forecastForce` | `change` | `renderForecast(value)` |
| `#peelForce` | `change` | `renderPEEL(value)` |
| `#rankMetric` | `change` | `renderRankings()` |
| `#regionFilter` | `change` | `renderRankings()` |
| `#compareForces` | `change` | `renderCompareRadar()` |
| `#insightForce` | `change` | `generateInsights(value, priority)` |
| `#insightPriority` | `change` | `generateInsights(force, value)` |
| `.hs-item` | `click` (inline) | `mainMap.setView([lat, lng], 15)` |
| `.c-btn` | `click` (inline) | `alert('Thank you...')` |

---

## 11. Initialisation Sequence

On `DOMContentLoaded`, the following functions are called in order:

1. `populateForceDropdowns()` — Fill all select elements
2. `initHeroMap()` — National overview map
3. `initMainMap()` — Analysis map with default "all" hotspots
4. `initCharts()` — Create all 13 Chart.js instances
5. `renderKPIs(null)` — National KPI view
6. `renderForecast('all')` — National forecast
7. `renderResources()` — Static resource cards
8. `renderPEEL('all')` — National PEEL metrics
9. `renderRankings()` — Default ranking (crime rate, all regions)
10. `renderDeprivationChart()` — Deprivation scatter
11. `renderForceTypeChart()` — Force type comparison
12. `generateInsights('all', 'all')` — National insights, all priorities

---

## 12. Known Limitations & Future Considerations

| Item | Detail |
|---|---|
| **Synthetic data** | All data is algorithmically generated. KPIs include a random component and will vary on each page load. |
| **No persistence** | Filter selections and state are not saved. Refreshing the page resets everything. |
| **Period filter** | The "Last 7 / 28 / 90 days" filter is UI-only; it does not alter the underlying synthetic data. |
| **Silka font** | Loaded from Simpson Associates' production CDN. If that server is unavailable, the browser falls back to system sans-serif. |
| **City of London** | Excluded from the hero map and deprivation scatter due to its unique characteristics (tiny area, resident population of ~10,000, very high transient population). Still included in all other views. |
| **Mobile navigation** | Top nav is hidden below 700px. No hamburger menu is implemented. |
| **Accessibility** | Colour contrast meets WCAG AA for most elements. No ARIA roles or screen reader optimisation has been applied. |
| **Print** | No print stylesheet is included. |
| **Contact form** | Non-functional; triggers a JavaScript alert only. |
