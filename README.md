# Machine-Accelerated Investigation — CBS Workshop

A hands-on workshop for Dutch data journalists: use AI coding agents (Kilo Code / Claude Code) to investigate housing, energy, and regional statistics from [CBS StatLine](https://opendata.cbs.nl/statline/).

## What's in this repo

Five AI agent skills that work together in sequence:

| Skill | What it does |
|---|---|
| `cbs-statline-skill` | Discover and download CBS OData tables, resolve region/period codes |
| `data-cleaning-dutch` | Fix Dutch data quirks (semicolon CSVs, comma decimals, missing-value markers) |
| `data-analysis-journalism` | Find story leads — outliers, trends, rankings, correlations |
| `data-viz-journalism` | Create publication-ready charts (Altair, matplotlib, Plotly) |
| `dutch-choropleth-maps` | Make choropleth maps by gemeente, wijk, or buurt |

Two exercises are included under `exercises/`.

## Prerequisites

- [uv](https://docs.astral.sh/uv/getting-started/installation/) — Python package manager
- [Kilo Code](https://kilocode.ai/) (VS Code extension) or [Claude Code](https://claude.ai/code) (CLI)

## Setup

```bash
git clone --recurse-submodules https://github.com/linksmith/Machine-Accelerated-Investigation-CBS-Workshop.git
cd Machine-Accelerated-Investigation-CBS-Workshop
make setup
make jupyter
```

`make setup` installs all dependencies and takes about 30 seconds. `make jupyter` opens JupyterLab in your browser.

If you need choropleth maps (Exercise 2 and beyond):

```bash
make setup-geo
```

### All available commands

```
make help
```

## Using the skills

Open your AI coding agent and load a skill before starting work. In Kilo Code, click **Add Skill** and point it at the skill folder, or install from the GitHub release URL. In Claude Code, use `/skill`.

The skills are designed to chain: start with `cbs-statline-skill` to get data, then pass the resulting DataFrame to whichever downstream skill fits your story.

## Exercises

### Exercise 1 — Lokale dataset: Woningdruk

Analyse a pre-downloaded CBS dataset (kerncijfers wijken en buurten 2025) to find where housing pressure is growing fastest in the Netherlands. Data is included as a CSV — no API access needed.

### Exercise 2 — Vrije API-opdracht

Open-ended: pick any CBS StatLine table via the API and tell a data-driven story. Use the full skill chain from discovery to map.

## License

MIT
