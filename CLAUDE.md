# Agents

## CBS StatLine Data

When accessing or analyzing CBS (Statistics Netherlands) data, always use the `cbs-statline-skill` skill located in `.kilo/skills/cbs-statline-skill/`. This applies to any task involving:

- Discovering or searching CBS StatLine tables
- Downloading or querying CBS OData APIs
- Working with Dutch housing, energy transition, or regional statistics
- Joining CBS datasets across tables (e.g., housing × energy labels)

To activate the skill, load it with the skill tool before starting work. The skill provides:

- A curated **table registry** of ~35 vetted CBS tables with join keys
- A **CBS OData v4 client** (`cbs_client.py`) for metadata inspection, data download, pagination, and code-to-label resolution
- **Analysis recipes** for common data journalism story angles
- Knowledge of CBS conventions (Perioden codes, RegioS codes, data status)

Key CBS conventions to remember:
- **Period codes**: `2023JJ00` = year, `2023KW01` = quarter, `2023MM06` = month
- **Region codes**: `GM####` = gemeente, `WK######` = wijk, `BU########` = buurt. Always `.strip()` trailing spaces.
- **Data status**: `Definitief` (final), `Voorlopig` (provisional `*`), `Nader voorlopig` (revised provisional `**`)
- Check for **deprecated tables** before use

## After retrieving CBS data

Once you have a DataFrame from CBS, use the appropriate downstream skill:

| Task | Skill |
|------|-------|
| Clean CSV/Excel files or fix Dutch data quirks | `data-cleaning-dutch` |
| Find story leads, outliers, trends, rankings | `data-analysis-journalism` |
| Create charts and publication-ready figures | `data-viz-journalism` |
| Create choropleth maps by gemeente, wijk, or buurt | `dutch-choropleth-maps` |