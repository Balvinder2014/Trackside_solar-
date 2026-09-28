# Trackside solar for India's electrified railway

Code accompanying the article

> **Trackside solar with daily storage could supply most of India's electric rail traction**
> [Authors], [Journal], [Year]. DOI: [to be added]

The pipeline maps India's railway network from OpenStreetMap, couples every
200 m of route to 4 km INSAT-3DR satellite insolation, computes trackside
photovoltaic output under 27 deployment scenarios, estimates the traction
substation layer, matches generation to state traction demand, and resolves
that match half-hourly with battery storage.

---

## Repository layout

```
trackside-solar-india/
├── python/             data download and network preparation
├── matlab/             modelling, analysis and figures
├── figures_scripts/    Fig. 1 (framework) and Fig. 7 redraw
├── data/
│   ├── benchmarks/     official statistics used for validation and demand
│   └── README.md       how to obtain the large input datasets
├── requirements.txt    Python dependencies
├── CITATION.cff
└── LICENSE
```

## Requirements

**Python 3.10+** with the packages in `requirements.txt`:

```
pip install -r requirements.txt
```

**MATLAB R2023b or later** (developed on R2025b) with:

- Statistics and Machine Learning Toolbox (`knnsearch`)
- Mapping Toolbox (`shaperead`, state outlines in figures only)

No other toolboxes are required. Solar geometry and the photovoltaic model are
implemented directly and need no toolbox.

## Data

The large inputs are not redistributed here. `data/README.md` explains how to
obtain each one:

| Dataset | Source | Folder |
| --- | --- | --- |
| Railway network | OpenStreetMap via Geofabrik | `osm/` |
| INSAT-3DR L2C insolation | MOSDAC, ISRO (registration) | `insat/` |
| ERA5 hourly single levels | Copernicus Climate Data Store | `era5_india/` |
| NASA POWER hourly | NASA POWER API | `power_india/` |
| State boundaries | Natural Earth, 1:10m admin-1 | `outputs/` |
| Railway statistics | Indian Railways Year Book 2021-22 | `data/benchmarks/` |

All data folders sit under one **data root**. The simplest setup is to use the
repository folder itself: clone it, then create `osm/`, `insat/`,
`era5_india/` and `power_india/` inside it. These folders are listed in
`.gitignore`, so they are never committed.

```
git clone https://github.com/[username]/trackside-solar-india.git
cd trackside-solar-india
mkdir outputs
copy data\benchmarks\*.csv outputs\          (Windows)
cp data/benchmarks/*.csv outputs/              (Linux, macOS)
```

To keep data elsewhere, set the root explicitly and run the Python scripts
from that folder:

```
set RAILSOLAR_ROOT=D:\RailSolar                  (Windows)
export RAILSOLAR_ROOT=/data/RailSolar          (Linux, macOS)
setenv('RAILSOLAR_ROOT','D:\RailSolar')        (MATLAB)
```

## Running the pipeline

Run the Python steps from the data root (the repository folder in the default
setup), then MATLAB after `addpath('matlab')`.

| Step | Command | Output |
| --- | --- | --- |
| 1 | `python python/download_era5.py` | `era5_india/` |
| 2 | `python python/build_rail_network.py` | route points, grid cells |
| 3 | `python python/download_power.py` | `power_india/` |
| 4 | `python python/add_state_electrified.py` | points with state names, validation table |
| 5 | MATLAB: `extract_insat` | INSAT values at railway pixels |
| 6 | MATLAB: `run_all_results` | every result, figure and summary |

`run_all_results` executes, in order:

1. `run_resolution_comparison` - PV output from INSAT, ERA5 and NASA POWER; resolution bias
2. `module2_capacity_generation` - capacity, generation and CO2 across 27 scenarios
3. `module3_substations` - substation layer and generation by distance band
4. `module4_supply_demand` - state generation against traction demand
5. `module5_hourly_matching` - half-hourly matching and battery storage
6. `make_figures` - Figs 2-6 with source data and captions
7. a results summary with every number quoted in the article

Use `run_all_results(true)` to recompute every stage. Diagnostics:
`check_datasets` compares raw irradiance between products and reports
satellite coverage; `robustness_coverage` repeats the headline comparison on
high-coverage days only.

Figs 1 and 7 at publication styling:

```
python figures_scripts/make_flowchart.py
python figures_scripts/make_fig7.py
```

## Outputs

Everything is written to `<data root>/outputs/results/`:

| File | Content |
| --- | --- |
| `results_summary.txt` | all headline numbers |
| `scenarios_national.csv` | 27 deployment scenarios |
| `resolution_bias_by_point.csv` | bias of coarse products at every route point |
| `deliverable_potential.csv` | generation by distance to substation |
| `supply_demand_by_state.csv` | generation against traction demand |
| `hourly_matching_by_state.csv`, `storage_curve_national.csv` | hourly coverage with storage |
| `Fig*.pdf / .png / .tif` | figures at 180 mm width, 600 dpi |
| `source_data/` | values behind every figure panel |

## Key assumptions

| Parameter | Value |
| --- | --- |
| Usable corridor width, one side | 3 / 4 / 5 m |
| Module efficiency | 20 / 24 / 28 % |
| Ground cover ratio (fixed / single-axis / dual-axis) | 0.45 / 0.33 / 0.25 |
| Albedo, NOCT, temperature coefficient | 0.20, 45 C, -0.0046 /K |
| Substation spacing | 35 km design grid, 18 km merge radius |
| Traction load peak-to-mean ratio | 1.11 (sensitivity 1.05-1.25) |
| Battery round-trip efficiency | 90 % |
| Grid emission factor | 0.70 t CO2 / MWh |

All are set in `matlab/railsolar_config.m` or at the top of each module.

## Citation

If you use this code, please cite the article above and this repository
(see `CITATION.cff`).

## Licence

Code: MIT licence (see `LICENSE`). OpenStreetMap-derived data remain under the
Open Database License; other input datasets remain under their providers' terms.
