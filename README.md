# Machine-Accelerated Investigation — CBS Workshop

Preparation material for the [data/expedities](https://dataexpedities.nl/) workshops — a collaboration between [Stichting Momus](https://www.momus.nl/), [CBS](https://www.cbs.nl/), [Open State Foundation](https://openstate.eu/), and the Dutch Ministry of the Interior.

The workshops bring together data journalists and programmers to build investigative stories using CBS StatLine open data and AI coding agents. The goal: go from idea to working data pipeline in minutes instead of days.

## What this repo contains

Five AI agent skills that work together in sequence — from pulling raw CBS data all the way to a publication-ready chart or map:

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

Run `make help` to see all available commands.

### Updating skills

The five skills are git submodules — each lives in its own repository. To pull the latest version of all skills:

```bash
make update
# equivalent to: git submodule update --remote --merge
```

When you pull changes to the workshop repo itself, include submodule updates in one step:

```bash
git pull --recurse-submodules
```

If you cloned without `--recurse-submodules` and the skill folders are empty, initialize them first:

```bash
git submodule update --init --recursive
```

## Using the skills

Open your AI coding agent and load a skill before starting work. In Kilo Code, click **Add Skill** and point it at the skill folder, or install from the GitHub release URL. In Claude Code, use `/skill`.

The skills are designed to chain: start with `cbs-statline-skill` to get data, then pass the resulting DataFrame to whichever downstream skill fits your story.

## Exercises

### Exercise 1 — Lokale dataset: Woningdruk

Analyse a pre-downloaded CBS dataset (kerncijfers wijken en buurten 2025) to find where housing pressure is growing fastest in the Netherlands. Data is included as a CSV — no API access needed.

### Exercise 2 — Vrije API-opdracht

Open-ended: pick any CBS StatLine table via the API and tell a data-driven story. Use the full skill chain from discovery to map.

## About data/expedities

[data/expedities](https://dataexpedities.nl/) are intensive hackathons where data journalists use CBS statistical data to develop investigative stories. This workshop repo is the preparation material: it gives participants the tools and skills to hit the ground running on the day itself.

Sessions are held at Open State Foundation in Amsterdam.

## License

MIT
