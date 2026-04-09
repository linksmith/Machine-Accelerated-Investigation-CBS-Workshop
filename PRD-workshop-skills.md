# PRD: Data Journalism Workshop Skills for Kilo Code

## Context

This PRD defines four new Agent Skills to be added to a data journalism workshop repository. The workshop teaches Dutch data journalists to use AI agentic coding (Kilo Code) to work with CBS StatLine open data about housing and energy transition in the Netherlands.

The workshop repo already includes the `cbs-statline-skill` (installed from `https://github.com/linksmith/cbs-statline-skill`). That skill handles **all CBS data retrieval**: table discovery, OData v4 API access, metadata inspection, data download with pagination/filtering, CBS code conventions (period codes like `2023JJ00`, region codes like `GM0363`, trailing space stripping), the `cbs_client.py` helper module, analysis recipe suggestions, and a reference to PDOK geo-endpoints.

**The four new skills must not duplicate any of that.** They pick up where the CBS skill leaves off: once you have a pandas DataFrame from CBS, what do you do with it?

## Target directory

All skills go in `.kilo/skills/` in the workshop repo root:

```
.kilo/skills/
├── cbs-statline-skill/          # ALREADY INSTALLED — do not modify
│   ├── SKILL.md
│   ├── references/
│   │   ├── table-registry.md
│   │   ├── analysis-recipes.md
│   │   ├── odata-v4-guide.md
│   │   └── geo-pdok.md
│   └── scripts/
│       └── cbs_client.py
├── data-cleaning-dutch/         # NEW — Skill 1
│   ├── SKILL.md
│   └── references/
│       └── dutch-data-quirks.md
├── data-analysis-journalism/    # NEW — Skill 2
│   ├── SKILL.md
│   └── references/
│       └── eda-checklist.md
├── data-viz-journalism/         # NEW — Skill 3
│   ├── SKILL.md
│   ├── references/
│   │   └── chart-selection-guide.md
│   └── assets/
│       └── journalism_style.py
├── dutch-choropleth-maps/       # NEW — Skill 4
│   ├── SKILL.md
│   └── references/
│       ├── pdok-endpoints.md
│       └── crs-guide.md
```

## Kilo Code skill format requirements

Every skill folder must contain a `SKILL.md` with:

1. **YAML frontmatter** between `---` delimiters with `name` (must match folder name exactly) and `description` (concise, ~100 tokens max — this is what the agent sees at startup for matching)
2. **Markdown body** with instructions (under 500 lines, under 5000 tokens)
3. Optional `references/`, `scripts/`, `assets/` subdirectories for content loaded on demand

The agent loads only the `name` + `description` at startup. It reads the full SKILL.md only when a task matches. Reference files are loaded only when the SKILL.md explicitly instructs the agent to read them.

## Audience

Mixed-level: data journalists who can read Python but may not write it fluently, alongside developers who may not know Dutch data conventions. Every skill must:

- Generate complete, runnable Python code with clear comments
- Explain the approach in plain language before writing code
- Explain what output means journalistically after running code
- Respond in Dutch if the user writes in Dutch
- Use Dutch labels in visualizations unless told otherwise

## Boundary rule with `cbs-statline-skill`

The CBS skill owns everything up to and including producing a pandas DataFrame from CBS data. The new skills own everything after that point. Specifically:

| Responsibility | Owned by |
|---|---|
| Finding CBS tables, inspecting metadata | `cbs-statline-skill` |
| OData v4 API calls, pagination, filtering | `cbs-statline-skill` |
| CBS period code parsing (`2023JJ00` → date) | `cbs-statline-skill` |
| CBS region code conventions (`GM`, `WK`, `BU`) | `cbs-statline-skill` |
| Stripping trailing spaces from CBS codes | `cbs-statline-skill` |
| Code-to-label resolution | `cbs-statline-skill` |
| Cross-table joins on CBS keys | `cbs-statline-skill` |
| Fetching PDOK geodata boundaries | `cbs-statline-skill` (via `references/geo-pdok.md`) |
| Cleaning general CSV/Excel files, Dutch locale issues | `data-cleaning-dutch` |
| EDA, groupby, outlier detection, story-finding | `data-analysis-journalism` |
| Charts, styling, publication-ready figures | `data-viz-journalism` |
| Merging stat data with geodata, CRS transforms, rendering maps | `dutch-choropleth-maps` |

When a new skill needs CBS data as input, it should tell the agent: "Use the `cbs-statline-skill` to retrieve the data first, then return here for [cleaning/analysis/visualization/mapping]."

---

## Skill 1: `data-cleaning-dutch`

### Description (for frontmatter)

```
Use this skill when the user has a CSV, Excel, or pandas DataFrame that needs
cleaning — especially Dutch data with comma decimals, semicolon delimiters,
mixed encodings, Dutch date formats (dd-mm-yyyy), missing value markers, or
messy column names. Also trigger for deduplication, type conversion, outlier
flagging, or general data quality checks. Do NOT use for CBS StatLine API
access or CBS-specific code conventions — those belong to cbs-statline-skill.
```

### SKILL.md body — sections to include

**Purpose**: Clean and validate tabular data for analysis. Focused on Dutch data quirks that trip up pandas defaults.

**When to use**: User has data (CSV, Excel, or DataFrame) that needs cleaning before analysis. User mentions dirty data, missing values, encoding issues, or asks to "clean" or "prepare" the data.

**When NOT to use**: If the user needs to fetch data from CBS StatLine, use `cbs-statline-skill` instead. That skill handles CBS-specific cleaning (trailing spaces, period codes, code-to-label resolution).

**Standard opening sequence**: Always start with a data health check:
```python
import pandas as pd

# Data health check
print(f"Shape: {df.shape}")
print(f"\nDtypes:\n{df.dtypes}")
print(f"\nMissing values:\n{df.isnull().sum()}")
print(f"\nDuplicate rows: {df.duplicated().sum()}")
print(f"\nSample:\n{df.head()}")
```

**Core cleaning patterns** — include code snippets for each:

1. **Dutch CSV import**: `pd.read_csv(path, sep=';', decimal=',', encoding='utf-8-sig')` — Dutch CSVs from government sources almost always use semicolon separator and comma decimal. The `utf-8-sig` encoding handles BOM markers common in Excel exports.
2. **Column name cleanup**: Strip whitespace, lowercase, replace spaces with underscores. `df.columns = df.columns.str.strip().str.lower().str.replace(' ', '_', regex=False)`
3. **Dutch date parsing**: `pd.to_datetime(df['datum'], format='%d-%m-%Y')` — warn that `dayfirst=True` is unreliable, always use explicit format.
4. **Comma-decimal numbers stored as strings**: `df['bedrag'] = df['bedrag'].str.replace('.', '', regex=False).str.replace(',', '.', regex=False).astype(float)` — first remove thousand separators (dots), then convert decimal commas to dots.
5. **Missing value markers**: Dutch government data uses various markers: empty string, `-`, `.`, `x`, `..`, `...`, `geheim`. Map all to NaN: `df.replace(['-', '.', 'x', '..', '...', 'geheim', ''], pd.NA, inplace=True)`
6. **Encoding detection**: When `utf-8` and `utf-8-sig` fail, try `latin-1` or `cp1252`. Include a try/except pattern.
7. **Postcode validation**: Dutch postcodes follow `1234AB` format. Normalize with `df['postcode'] = df['postcode'].str.replace(' ', '').str.upper()` and validate with regex `r'^\d{4}[A-Z]{2}$'`.
8. **Deduplication**: Show both exact (`df.drop_duplicates()`) and fuzzy approaches.

**Gotchas section**:
- Excel files from Dutch government often have merged cells in headers — use `header=None` and clean manually
- Watch for mixed types in columns (some cells numeric, some text) — always check with `df[col].apply(type).value_counts()`
- CBS "geheim" means the value is suppressed for privacy (fewer than 5 observations)

**Output format**: After cleaning, always show a before/after comparison: row count, null count, dtype changes.

### Reference file: `references/dutch-data-quirks.md`

Extended reference covering:
- Complete list of Dutch government missing value markers with their meanings
- Common Dutch government data sources and their default export formats (CBS: semicolon/comma, Kadaster: tab-separated, RVO: pipe-separated, RIVM: varies)
- Dutch locale number formatting rules (1.234.567,89)
- Municipality name changes 2020–2025 (herindelingen) with old→new mapping for the most common ones
- Postcode ↔ wijk/buurt mapping guidance (use CBS's postcode-huisnummer tables)
- Character encoding cheat sheet (when to use utf-8, utf-8-sig, latin-1, cp1252)

---

## Skill 2: `data-analysis-journalism`

### Description (for frontmatter)

```
Use this skill when the user has a clean pandas DataFrame and wants to explore
it for story leads — finding outliers, trends, comparisons, rankings, or
surprising patterns. Trigger on requests like "analyze this data", "what
stories are in here", "find outliers", "compare regions", "show me trends",
or any exploratory data analysis for journalism. Do NOT use for data
retrieval (use cbs-statline-skill) or data cleaning (use data-cleaning-dutch).
```

### SKILL.md body — sections to include

**Purpose**: Find stories in data. This is journalistic EDA, not data science EDA. The goal is newsworthy findings, not model features.

**When to use**: User has a clean DataFrame and wants to understand what's in it. User asks "what's interesting" or "what are the stories here."

**Persona**: You are a data journalism editor helping a reporter find the lede. Every number you surface should be contextualized: "this means X has 3x the national average" not just "the value is 45.2."

**Standard EDA sequence** — always follow this order:

1. **Shape & structure**: `df.shape`, `df.dtypes`, `df.describe()`, `df.nunique()`
2. **Distributions**: For each numeric column, check skewness. Flag columns where median and mean diverge significantly (sign of outliers or skewed distribution).
3. **Rankings**: For any geographic column, rank by each metric. Always show top 5 and bottom 5. Contextualize with the national average/median.
4. **Time trends**: If the data has a time dimension, compute year-over-year change (absolute and percentage). Flag inflection points where the direction changes.
5. **Comparisons**: Group by categorical variables (region type, province, year). Always compute both absolute values and per-capita/per-household rates where denominators are available.
6. **Outliers**: Use IQR method (below Q1 - 1.5×IQR or above Q3 + 1.5×IQR). Present outliers as potential story leads: "Gemeente X is an outlier because..."
7. **Correlations**: For pairs of numeric columns, compute Pearson correlation. Flag strong correlations (|r| > 0.7) and surprising weak ones. Warn about ecological fallacy when correlating aggregated geographic data.
8. **Missing patterns**: Check if missing values cluster geographically or temporally — this itself can be a story (why doesn't gemeente X report this?).

**Story framing rules** — the agent must follow these:

- After every analysis step, write a **one-sentence journalistic finding** in bold. Example: **"Utrecht heeft de snelste groei in warmtepompen: +34% in twee jaar, terwijl het landelijk gemiddelde op +18% ligt."**
- Always provide **context denominators**: never say "Amsterdam has the most X" without per-capita or per-household rates, because Amsterdam is simply the largest city.
- Flag **counterintuitive findings** — these are the best stories. "You'd expect wealthy municipalities to lead in solar panels, but the data shows..."
- Suggest **2–3 follow-up questions** the data raises but can't answer.

**Percentage calculation rules**:
- Year-over-year: `((new - old) / old) * 100`
- Share of total: `(part / whole) * 100`
- Always round to 1 decimal place for presentation
- When the base is small (<50), warn that percentages can be misleading

**Code patterns to include**:
```python
# Ranking with national context
national_avg = df['metric'].mean()
ranking = df.groupby('gemeente')['metric'].mean().sort_values(ascending=False)
ranking_with_context = pd.DataFrame({
    'value': ranking,
    'vs_national': ((ranking - national_avg) / national_avg * 100).round(1)
})
```

```python
# Year-over-year change
yoy = df.pivot_table(index='gemeente', columns='year', values='metric')
yoy['change_pct'] = ((yoy[latest_year] - yoy[previous_year]) / yoy[previous_year] * 100).round(1)
```

**Output format**: A structured findings summary:
1. **Headline finding** (the lede)
2. **Key numbers** (3–5 facts with context)
3. **Outliers and surprises** (with possible explanations)
4. **Caveats** (data limitations, methodology notes)
5. **Follow-up questions** (what to investigate next)

### Reference file: `references/eda-checklist.md`

A printable checklist format covering:
- The complete EDA sequence as a numbered checklist
- Statistical test selection guide (when to use what): chi-square for categorical comparisons, Mann-Whitney U for non-normal distributions, Kruskal-Wallis for multi-group comparisons
- Ecological fallacy explainer (why gemeente-level correlations don't prove individual-level causation)
- Common journalistic denominator sources (population: CBS 03759NED, households: CBS 71486NED, surface area: CBS 70262NED)
- Dutch number presentation conventions for journalism (use comma as decimal separator in Dutch-language articles, use dot in English)

---

## Skill 3: `data-viz-journalism`

### Description (for frontmatter)

```
Use this skill when the user wants to create charts, graphs, or static
visualizations from data for journalistic publication. Trigger on "make a
chart", "visualize", "plot", "graph", "bar chart", "line chart", or any
request for publication-ready figures. Covers Altair (preferred), matplotlib,
and plotly. Do NOT use for geographic maps (use dutch-choropleth-maps).
```

### SKILL.md body — sections to include

**Purpose**: Create publication-ready data visualizations for journalism. Clean, accessible, story-driven charts — not exploratory plots.

**Library preference order**:
1. **Altair** (preferred) — declarative, clean defaults, excellent for statistical visualization. Best for: bar charts, line charts, scatter plots, small multiples, heatmaps.
2. **Plotly** — when interactivity is needed (HTML embeds for web articles).
3. **Matplotlib** — when fine-grained control is needed or Altair can't handle the chart type.

**Chart selection guide** — use this decision tree:
- Comparing categories → **bar chart** (horizontal if >5 categories or long labels)
- Trend over time → **line chart** (one line per group, max 5–6 lines)
- Part of whole → **stacked bar** (never pie charts — they are hard to read)
- Relationship between two variables → **scatter plot**
- Comparing across two dimensions → **heatmap** or **small multiples**
- Distribution → **histogram** or **box plot**

**Journalism-specific rules**:

1. **Title states the finding, not the data**: "Amsterdam heeft de hoogste huurstijging" not "Huurprijzen per gemeente". The title is the headline.
2. **Subtitle provides context**: Method, time period, source. Example: "Gemiddelde huurstijging per jaar, 2020–2024. Bron: CBS StatLine"
3. **Axis labels in plain language**: "Percentage woningen met energielabel A" not "pct_label_a"
4. **Source line**: Always include "Bron: CBS StatLine" (or other source) as a text annotation at the bottom.
5. **Accessible colors**: Use colorblind-safe palettes. Default to the Tableau10 palette in Altair or include a curated Dutch journalism palette.
6. **No chartjunk**: Remove gridlines where possible, remove unnecessary borders, use direct labeling instead of legends when feasible.
7. **Highlight the story**: Use color contrast to draw attention to the key finding. Grey out non-focal data points. Annotate the key data point.

**Altair patterns** — include these as templates:

```python
import altair as alt

# Basic bar chart — journalism style
chart = alt.Chart(df).mark_bar().encode(
    x=alt.X('value:Q', title='Percentage woningen met zonnepanelen'),
    y=alt.Y('gemeente:N', sort='-x', title=None),
    color=alt.condition(
        alt.datum.gemeente == 'Utrecht',  # Highlight story focus
        alt.value('#d62728'),
        alt.value('#bdbdbd')
    )
).properties(
    title={
        'text': 'Utrecht loopt voorop met zonnepanelen',
        'subtitle': 'Aandeel woningen met zonnepanelen, 2024. Bron: CBS StatLine'
    },
    width=500,
    height=400
)
```

```python
# Line chart with annotation
line = alt.Chart(df).mark_line().encode(
    x=alt.X('year:O', title='Jaar'),
    y=alt.Y('value:Q', title='Aantal warmtepompen (×1000)'),
    color='regio:N'
)

annotation = alt.Chart(pd.DataFrame({
    'x': [2022], 'y': [45], 'text': ['Subsidieregeling ISDE verruimd']
})).mark_text(align='left', dx=5, fontSize=11).encode(
    x='x:O', y='y:Q', text='text:N'
)

(line + annotation).properties(title='Warmtepompen in opkomst sinds 2020')
```

**Saving figures**:
```python
# Always save both formats
chart.save('output/chart_name.png', scale_factor=2)  # 2x for retina/print
chart.save('output/chart_name.svg')                   # Vector for publication
# For plotly:
fig.write_image('output/chart_name.png', scale=2, width=800, height=500)
fig.write_html('output/chart_name.html')               # Interactive embed
```

**Output format**: Every visualization must include:
1. The chart itself (saved as PNG + SVG or HTML)
2. A one-sentence description of what it shows (for alt text / accessibility)
3. The source attribution line

### Reference file: `references/chart-selection-guide.md`

Extended guide with:
- Decision tree diagram (text-based) for chart type selection
- Altair, matplotlib, and plotly code templates for each chart type
- Color palette definitions (colorblind-safe, Dutch journalism conventions)
- Typography and sizing recommendations for web vs print
- Small multiples patterns for comparing across many categories
- Annotation patterns (callouts, reference lines, shaded regions)

### Asset file: `assets/journalism_style.py`

A small Python module with reusable styling functions:
```python
"""Reusable styling for journalism visualizations."""

# Colorblind-safe palette (Tableau10 subset + highlight red)
COLORS = {
    'highlight': '#d62728',
    'secondary': '#1f77b4',
    'muted': '#bdbdbd',
    'palette': ['#1f77b4', '#ff7f0e', '#2ca02c', '#d62728', '#9467bd',
                '#8c564b', '#e377c2', '#7f7f7f', '#bcbd22', '#17becf']
}

def altair_journalism_theme():
    """Register a journalism-friendly Altair theme."""
    return {
        'config': {
            'view': {'strokeWidth': 0},
            'axis': {
                'labelFontSize': 12,
                'titleFontSize': 13,
                'gridOpacity': 0.3,
            },
            'title': {
                'fontSize': 16,
                'subtitleFontSize': 12,
                'subtitleColor': '#666666',
            },
            'legend': {'labelFontSize': 11},
        }
    }

# Usage: alt.themes.register('journalism', altair_journalism_theme)
#        alt.themes.enable('journalism')
```

---

## Skill 4: `dutch-choropleth-maps`

### Description (for frontmatter)

```
Use this skill when the user wants to create a map of the Netherlands showing
data by gemeente, wijk, or buurt — a choropleth map. Trigger on "map",
"choropleth", "kaart", "geographical visualization", "show on a map", or any
request to visualize data spatially across Dutch regions. Covers merging
statistical data with PDOK geodata, CRS conversion, and rendering with
geopandas, folium, and plotly. For fetching CBS data, use cbs-statline-skill
first. For non-map charts, use data-viz-journalism instead.
```

### SKILL.md body — sections to include

**Purpose**: Create choropleth maps of Dutch statistical data at gemeente, wijk, or buurt level. This skill handles the entire pipeline from a DataFrame with region codes to a rendered map.

**Prerequisites**: The user should already have a pandas DataFrame with a column containing CBS region codes (GM, WK, or BU codes). If they don't, tell them to use `cbs-statline-skill` first.

**Required packages**: `geopandas`, `folium`, `matplotlib`, `plotly` (optional for interactive), `requests`

**The mapping pipeline** — always follow these steps:

### Step 1: Identify the geographic level

Detect from the data which level is present:
```python
# Auto-detect geographic level from region codes
sample_code = df['regio_code'].dropna().iloc[0].strip()
if sample_code.startswith('BU'):
    level, pdok_layer = 'buurt', 'buurt_gegeneraliseerd'
elif sample_code.startswith('WK'):
    level, pdok_layer = 'wijk', 'wijk_gegeneraliseerd'
elif sample_code.startswith('GM'):
    level, pdok_layer = 'gemeente', 'gemeente_gegeneraliseerd'
else:
    raise ValueError(f"Unrecognized region code format: {sample_code}")
print(f"Detected level: {level} — will fetch {pdok_layer} boundaries")
```

### Step 2: Fetch geographic boundaries

Read `references/pdok-endpoints.md` for the full endpoint reference. Use the gegeneraliseerd (simplified) versions for maps — they render much faster.

```python
import geopandas as gpd

year = 2024  # Match the year of your statistical data
wfs_url = (
    f"https://service.pdok.nl/cbs/gebiedsindelingen/{year}/wfs/v1_0"
    f"?request=GetFeature&service=WFS&version=2.0.0"
    f"&typeName={pdok_layer}&outputFormat=json"
)
geo = gpd.read_file(wfs_url)
```

**Fallback**: If PDOK WFS is slow or returns a timeout (common for buurt-level with 13,000+ polygons), use pre-simplified GeoJSON from `cartomap/nl` on GitHub:
```python
# Fallback: simplified boundaries from cartomap/nl
geo = gpd.read_file(
    f"https://cartomap.github.io/nl/wgs84/gemeente_2024.geojson"
)
```

### Step 3: Convert coordinate reference system

PDOK delivers data in **EPSG:28992** (Rijksdriehoekscoördinaten / RD New), the Dutch national projection. Web maps (Folium, Plotly) need **EPSG:4326** (WGS84). Static maps with geopandas can stay in RD New (it looks better for NL — less distortion).

```python
# For web maps (Folium/Plotly): convert to WGS84
geo_wgs84 = geo.to_crs(epsg=4326)

# For static maps (geopandas plot): keep RD New or convert
# EPSG:28992 preserves Dutch proportions better
```

Read `references/crs-guide.md` for details on when to use which CRS.

### Step 4: Merge statistical data with geodata

The join key depends on the PDOK dataset version. For recent years:
```python
# Normalize join keys — PDOK uses 'statcode', CBS data varies
geo['statcode'] = geo['statcode'].str.strip()
df['regio_code'] = df['regio_code'].str.strip()

merged = geo.merge(df, left_on='statcode', right_on='regio_code', how='left')

# Check merge quality
n_matched = merged[merged['metric'].notna()].shape[0]
n_total = geo.shape[0]
print(f"Matched {n_matched}/{n_total} regions ({n_matched/n_total*100:.1f}%)")
```

**Critical gotcha — municipality mergers**: The Netherlands regularly merges municipalities. GM codes change when this happens. If your data is from 2020 but your geodata is from 2024, some codes won't match. Always use geodata from the same year as the statistical data, or include a merger mapping. Read `references/pdok-endpoints.md` for the year-specific endpoint pattern.

### Step 5: Render the map

**Option A: Static map with geopandas** (for print / static reports)
```python
import matplotlib.pyplot as plt

fig, ax = plt.subplots(1, 1, figsize=(10, 12))
merged.plot(
    column='metric',
    cmap='YlOrRd',
    legend=True,
    legend_kwds={'label': 'Metric label here', 'shrink': 0.6},
    missing_kwds={'color': 'lightgrey', 'label': 'Geen data'},
    ax=ax
)
ax.set_axis_off()
ax.set_title('Title: de bevinding, niet de data', fontsize=16, pad=20)
ax.annotate('Bron: CBS StatLine', xy=(0.02, 0.02), xycoords='axes fraction',
            fontsize=9, color='grey')
plt.tight_layout()
plt.savefig('output/kaart.png', dpi=300, bbox_inches='tight')
plt.savefig('output/kaart.svg', bbox_inches='tight')
```

**Option B: Interactive map with Folium** (for web articles)
```python
import folium

m = folium.Map(location=[52.1, 5.3], zoom_start=7, tiles='cartodbpositron')
folium.Choropleth(
    geo_data=merged_wgs84.to_json(),
    data=df,
    columns=['regio_code', 'metric'],
    key_on='feature.properties.statcode',
    fill_color='YlOrRd',
    fill_opacity=0.7,
    line_opacity=0.2,
    legend_name='Metric label'
).add_to(m)

# Add tooltips
folium.GeoJson(
    merged_wgs84,
    tooltip=folium.GeoJsonTooltip(fields=['statnaam', 'metric'],
                                    aliases=['Gemeente:', 'Waarde:'])
).add_to(m)

m.save('output/kaart.html')
```

**Option C: Interactive with Plotly** (for dashboards)
```python
import plotly.express as px

fig = px.choropleth_mapbox(
    merged_wgs84,
    geojson=merged_wgs84.geometry.__geo_interface__,
    locations=merged_wgs84.index,
    color='metric',
    hover_name='statnaam',
    mapbox_style='carto-positron',
    center={'lat': 52.1, 'lon': 5.3},
    zoom=6,
    color_continuous_scale='YlOrRd',
    title='Title: de bevinding'
)
fig.write_html('output/kaart.html')
```

**Journalism rules for maps**:
- Title states the finding, not the variable
- Use sequential color scales (YlOrRd, YlGnBu) for rates/quantities, diverging (RdBu) for deviation from average
- Always show a legend with clear units
- Grey out regions with no data — don't hide them
- Include source attribution: "Bron: CBS StatLine, PDOK gebiedsindelingen"
- For buurt/wijk maps: consider showing only one gemeente zoomed in, since national buurt maps are unreadable

**Output format**: The map file (PNG+SVG for static, HTML for interactive) plus a 1-sentence alt text description.

### Reference file: `references/pdok-endpoints.md`

Complete reference covering:
- WFS URL template for each geographic level (gemeente, wijk, buurt) and year
- Note that the PDOK endpoint structure changed in 2023 — old format (`geodata.nationaalgeoregister.nl`) vs new format (`service.pdok.nl/cbs/gebiedsindelingen`)
- Field name mapping: `statcode` (region code), `statnaam` (region name), `jrstatcode` (year-specific code)
- Gegeneraliseerd vs non-gegeneraliseerd: always use gegeneraliseerd for maps (simplified geometry, much smaller files)
- `cartomap/nl` GitHub URLs as fallback for each level and year (pre-simplified, WGS84-projected GeoJSON)
- Rate limits and timeout handling for PDOK WFS (no auth required, but buurt level can be slow)
- Year availability: which years of geodata are available (2013–2025 for gemeenten)

### Reference file: `references/crs-guide.md`

Short explainer covering:
- **EPSG:28992** (RD New / Rijksdriehoekscoördinaten): Dutch national CRS, used by PDOK. Best for static maps of NL — preserves shape and area.
- **EPSG:4326** (WGS84): Global latitude/longitude. Required for Folium, Plotly mapbox, Leaflet. Used by cartomap/nl pre-projected files.
- **EPSG:3857** (Web Mercator): Used internally by web map tiles. You rarely need to convert to this explicitly.
- When to convert: always convert to WGS84 for interactive web maps, keep RD New for static geopandas plots.
- Code snippet for conversion: `gdf = gdf.to_crs(epsg=4326)`

---

## Dependencies

All skills should assume these packages are available (the workshop repo's `requirements.txt` should include them):

```
pandas>=2.0
altair>=5.0
matplotlib>=3.7
geopandas>=0.14
folium>=0.15
plotly>=5.18
requests
openpyxl          # Excel file support
```

## Implementation notes for Claude Code

1. **Install the CBS skill first** by cloning `https://github.com/linksmith/cbs-statline-skill` into `.kilo/skills/cbs-statline-skill/`
2. **Create each new skill** as a directory in `.kilo/skills/` with the structure defined above
3. **Verify the `name` field** in each SKILL.md matches the directory name exactly (Kilo Code requirement)
4. **Keep SKILL.md files under 500 lines** — move detailed content to reference files
5. **Test descriptions are distinct**: No two skills should trigger on the same request. The descriptions are carefully scoped to avoid overlap.
6. Each skill's SKILL.md should include a "When NOT to use" section that redirects to the correct skill, to prevent false matches.
