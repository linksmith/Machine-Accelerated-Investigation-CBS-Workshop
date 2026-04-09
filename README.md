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

| Tool | What it's for | Install |
|---|---|---|
| **VS Code** | Code editor | [code.visualstudio.com/download](https://code.visualstudio.com/download) |
| **Git** | Cloning the workshop repo and skills | [git-scm.com/downloads](https://git-scm.com/downloads) |
| **Kilo Code** | AI coding agent (VS Code extension) | [marketplace.visualstudio.com](https://marketplace.visualstudio.com/items?itemName=kilocode.kilo-code) |
| **uv** | Python package manager — installs Python and all dependencies in one step | [docs.astral.sh/uv](https://docs.astral.sh/uv/getting-started/installation/) |

## Setup

### 1. Configure Kilo Code

1. Open VS Code and make sure Kilo Code is installed
2. Close the built-in Chat tab if it's open
3. Click the **Kilo Code icon** in the sidebar
4. Click the **settings icon** inside Kilo Code
5. Click **Providers**
6. Next to **OpenRouter**, click **+ Connect**
7. Paste in the API key shared at the start of the workshop
8. At the bottom of the screen, click the **Kilo Auto Free** dropdown
9. Search for **GLM-5** and select it — please don't select a different model, as the shared credits are limited

### 3. Install the workshop repository

```bash
git clone --recurse-submodules https://github.com/linksmith/Machine-Accelerated-Investigation-CBS-Workshop.git
cd Machine-Accelerated-Investigation-CBS-Workshop
```

**Mac / Linux** (with `make`):
```bash
make setup
make jupyter
```

**Windows** (or if `make` is not available):
```bash
uv sync --group dev --group viz
uv run jupyter lab
```

Installs all Python dependencies (~30 seconds), then opens JupyterLab in your browser.

For choropleth maps (Exercise 2 and beyond):

```bash
# Mac / Linux
make setup-geo

# Windows
uv sync --all-groups
```

### Updating skills

The five skills are git submodules — each lives in its own repository. To pull the latest version of all skills:

```bash
# Mac / Linux
make update

# Windows
git submodule update --remote --merge
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
