# CLAUDE.md - AI Assistant Guide for Simpson Associates Hotspot Policing Demo

## Project Overview

This is a **single-page HTML application** serving as an interactive demo/dashboard for Simpson Associates' policing analytics and demand forecasting platform. It showcases AI-powered crime analytics capabilities designed for UK police forces.

**Key Features:**
- Real-time hotspot policing analytics with interactive maps
- ML-driven demand forecasting (incident predictions)
- Resource allocation optimization recommendations
- PEEL alignment metrics (Effectiveness, Efficiency, Legitimacy)
- AI-generated actionable insights

**Target Audience:** UK police forces evaluating data analytics solutions

---

## Codebase Structure

```
/hotspotdemo/
├── CLAUDE.md                                        # This file
├── simpson-associates-hotspot-policing-demo (1).html  # Complete application (single file)
└── .git/                                            # Version control
```

### Single-File Architecture

The entire application is contained in one HTML file (~870 lines):

| Section | Lines | Description |
|---------|-------|-------------|
| `<head>` | 1-235 | Metadata, CDN imports, embedded CSS |
| CSS (embedded) | 11-234 | Design tokens, components, responsive styles |
| HTML body | 236-487 | Header, pages, dashboard panels, footer |
| JavaScript | 488-868 | Data, navigation, maps, charts, event handlers |

---

## Technologies & Dependencies

| Technology | Version | Purpose | Source |
|-----------|---------|---------|--------|
| **HTML5** | - | Markup structure | Native |
| **CSS3** | - | Styling with CSS custom properties | Embedded |
| **JavaScript** | ES6+ | Interactivity and logic | Embedded |
| **Leaflet.js** | 1.9.4 | Interactive maps | CDN (cdnjs) |
| **Chart.js** | 4.4.1 | Data visualizations | CDN (cdnjs) |
| **Google Fonts** | - | Outfit, JetBrains Mono | Google CDN |

**CDN Dependencies (require internet):**
```html
<link href="https://fonts.googleapis.com/css2?family=Outfit:wght@400;500;600;700;800&family=JetBrains+Mono:wght@500;600&display=swap" rel="stylesheet">
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.9.4/leaflet.min.css"/>
<script src="https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.9.4/leaflet.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.min.js"></script>
```

---

## Development Workflow

### No Build Process Required

This is a static HTML file with zero build steps:

1. **Edit** the HTML file directly
2. **Test** by opening in any modern browser
3. **Deploy** by uploading the HTML file to any web server

### Running Locally

Simply open the HTML file in a browser:
```bash
# Using Python's built-in server (optional)
python -m http.server 8000
# Then visit http://localhost:8000

# Or just open the file directly in a browser
open "simpson-associates-hotspot-policing-demo (1).html"
```

### Testing

**No automated tests.** Testing is manual:
- Verify filter interactions work (force area, crime type, time period)
- Check map renders correctly with Leaflet
- Ensure charts update properly with Chart.js
- Test tab navigation across dashboard panels
- Validate responsive design at different breakpoints

---

## Code Conventions & Patterns

### CSS Naming

**Design Tokens** (CSS custom properties in `:root`):
```css
/* Colors */
--navy, --navy-2, --navy-3     /* Dark backgrounds */
--blue, --blue-d, --blue-l     /* Primary accent */
--teal, --teal-l               /* Secondary accent */
--red, --orange, --amber, --green  /* Semantic colors */
--g50 to --g900                /* Grayscale scale */

/* Layout */
--radius: 14px                 /* Border radius */
--shadow, --shadow-lg          /* Box shadows */
```

**Class Naming** (BEM-inspired, short):
```css
.kpi, .kpi.c-blue            /* Component with modifier */
.hs-item, .hs-item.crit      /* Hotspot item with level */
.d-tab, .d-tab.active        /* Dashboard tab states */
```

### JavaScript Patterns

**Module-like Organization:**
```javascript
// Data layer
const FORCES = { /* force datasets */ }

// View functions
function initHeroMap() { /* setup */ }
function initMainMap() { /* setup */ }
function initCharts() { /* setup */ }

// Render functions
function renderHotspots(forceKey, crimeFilter) { /* ... */ }
function renderHotspotList(hotspots, forceName) { /* ... */ }

// Event handlers
function showPage(id) { /* navigation */ }
function applyFilters() { /* filtering */ }
```

**Variable Naming:**
- camelCase for variables: `heroMapInstance`, `mainMap`, `temporalChartInstance`
- UPPER_CASE for constants: `FORCES`
- Descriptive function names: `renderHotspotList`, `applyFilters`

**DOM Manipulation:**
```javascript
// Query elements
document.querySelectorAll('.d-tab')
document.getElementById('mainMap')

// Class toggling
element.classList.add('active')
element.classList.remove('active')

// Event delegation
document.querySelectorAll('.d-tab').forEach(tab => {
  tab.addEventListener('click', handler)
})
```

---

## Data Structure

### FORCES Object

The main data source is the `FORCES` object containing synthetic datasets for UK police forces:

```javascript
FORCES = {
  [forceKey]: {           // 'all', 'gmp', 'wmp', 'met', 'mers', 'wyp', 'swp'
    name: string,         // Display name
    center: [lat, lng],   // Map center coordinates
    zoom: number,         // Initial zoom level
    hotspots: [           // Array of hotspot objects
      {
        name: string,     // Location name
        lat: number,      // Latitude
        lng: number,      // Longitude
        risk: number,     // Risk score (0-100)
        level: string,    // 'crit'|'high'|'med'|'low'
        types: string,    // Crime types (comma-separated)
        context: string,  // Descriptive context
        area: number      // Hexagon size factor
      }
    ],
    temporal: number[],   // 12 hourly incident values
    kpis: {               // Key performance indicators
      incidents: string,
      hotspots: string,
      officers: string,
      accuracy: string
    }
  }
}
```

### Risk Level Mapping

| Level | Score Range | Color | CSS Variable |
|-------|-------------|-------|--------------|
| Critical | 80-100 | Red | `--red` (#EF4444) |
| High | 60-79 | Orange | `--orange` (#F97316) |
| Medium | 40-59 | Amber | `--amber` (#EAB308) |
| Low | 0-39 | Green | `--green` (#22C55E) |

---

## UI Components

### Page Structure

| Page | ID | Description |
|------|-----|-------------|
| Dashboard | `pg-dashboard` | Main analytics dashboard |
| Solutions | `pg-solutions` | Product offerings |
| Case Studies | `pg-casestudies` | Customer success stories |
| Contact | `pg-contact` | Contact form and info |

### Dashboard Panels

| Panel | ID | Data Attribute | Description |
|-------|-----|----------------|-------------|
| Overview | `overview` | `data-panel="overview"` | KPIs and trend charts |
| Hotspot Analysis | `hotspots` | `data-panel="hotspots"` | Interactive map and list |
| Demand Forecast | `forecast` | `data-panel="forecast"` | 7-day predictions |
| Resource Allocation | `resources` | `data-panel="resources"` | Deployment recommendations |
| PEEL Alignment | `peel` | `data-panel="peel"` | Performance metrics |
| AI Insights | `insights` | `data-panel="insights"` | AI-generated recommendations |

### Charts (Chart.js)

| Chart ID | Type | Purpose |
|----------|------|---------|
| `trendChart` | Line | 12-month crime trends by category |
| `riskChart` | Doughnut | Hotspot risk distribution |
| `temporalChart` | Bar | Hourly incident patterns |
| `forecastChart` | Line | Actual vs predicted demand |
| `categoryChart` | Bar | Crime type predictions |
| `shiftChart` | Line | Demand vs staffing coverage |
| `effChart` | Radar | Resource efficiency metrics |

---

## Making Changes

### Adding a New Force Area

1. Add entry to `FORCES` object:
```javascript
FORCES.newforce = {
    name: 'Force Name',
    center: [lat, lng],
    zoom: 12,
    hotspots: [/* hotspot objects */],
    temporal: [/* 12 hourly values */],
    kpis: {incidents: '000', hotspots: '0', officers: '000', accuracy: '00.0%'}
};
```

2. Add option to filter dropdown:
```html
<option value="newforce">Force Name</option>
```

### Adding a New Dashboard Panel

1. Add tab button in `.dash-nav`:
```html
<button class="d-tab" data-panel="newpanel">New Panel</button>
```

2. Add panel content:
```html
<div class="panel" id="newpanel">
    <!-- Panel content -->
</div>
```

### Modifying Styles

Update CSS custom properties in `:root` for global changes:
```css
:root {
    --blue: #2563EB;  /* Change primary color */
    --radius: 14px;   /* Change border radius */
}
```

### Updating Chart Data

Charts are initialized in `initCharts()`. Modify dataset values there:
```javascript
datasets: [
    {label:'Violence', data:[/* monthly values */], borderColor:'#EF4444', ...}
]
```

---

## Responsive Breakpoints

```css
/* Desktop: > 1100px (default) */

@media(max-width:1100px) {
    /* Tablet: stack grids, single column layouts */
}

@media(max-width:700px) {
    /* Mobile: further simplification, hide nav */
}
```

---

## Important Notes for AI Assistants

### Do's

- **Maintain single-file structure** - Keep all code in one HTML file
- **Use existing CSS variables** - Leverage the design token system
- **Follow naming conventions** - camelCase for JS, BEM-inspired for CSS
- **Test map interactions** - Leaflet requires `invalidateSize()` after container changes
- **Update FORCES data** - When adding/modifying police force data
- **Preserve responsive design** - Check changes at all breakpoints

### Don'ts

- **Don't split into multiple files** without explicit request
- **Don't add build tools** (webpack, etc.) - Keep it simple
- **Don't remove CDN dependencies** - They're required for maps/charts
- **Don't modify core Leaflet/Chart.js behavior** - Use their APIs
- **Don't add backend functionality** - This is a frontend demo only

### Common Tasks

| Task | Location |
|------|----------|
| Update KPI values | `FORCES[key].kpis` object |
| Change map styling | `renderHotspots()` function |
| Modify chart colors | `initCharts()` function, dataset configs |
| Add filter options | HTML `<select>` elements + `applyFilters()` |
| Update AI insights | HTML in `#insights` panel |

### File Size Considerations

Current file is ~61KB. Keep it lightweight:
- Avoid large inline images (use SVG or external URLs)
- Don't duplicate code unnecessarily
- CSS is already minified-style (single line per rule set)

---

## Contact & Support

**Simpson Associates**
- Website: [www.simpson-associates.co.uk](https://www.simpson-associates.co.uk)
- York Office: Suite 3, 20 George Hudson Street, York YO1 6WR
- Sheffield Office: 3 Concourse Way, Sheffield City Centre, Sheffield S1 2BJ
- Phone: +44 (0) 1904 234 510

**Accreditations:** Microsoft Partner of the Year 2024, ISO 27001, ISO 9001, Cyber Essentials Plus, Databricks Partner, Police Industry Charter Signatory
