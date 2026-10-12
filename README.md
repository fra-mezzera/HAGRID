# HAGRID

**HAGRID** (Household Activity-based Grid-aware Residential Integrated Demand model) is a Python/Jupyter framework for modelling residential local energy systems. It combines behaviour-driven electricity-demand reconstruction from time-use information, additional electrified end uses, and local energy system operational optimization with simplified low-voltage (LV) grid constraints.

The grid-aware optimization uses simplified line-capacity constraints. It should not be interpreted as a detailed AC power-flow analysis.



## Framework modules

| Notebook | Purpose |
| --- | --- |
| `MODULE_01_individual_activity_profiles.ipynb` | Builds synthetic individual activity and location profiles from Italian Time-Use Survey (TUS) data. |
| `MODULE_02_households_profiles_and_conventional_loads.ipynb` | Combines individual profiles into synthetic households and reconstructs activity-linked conventional electricity demand. |
| `MODULE_03_additional_loads.ipynb` | Estimates occupancy, space heating and cooling, cooking, domestic hot water (DHW), and EV mobility/charging requirements. |
| `MODULE_04_local_energy_system_modeling.ipynb` | Models electrification and control scenarios, rule-based operation, optimization of flexible assets, and technical/economic indicators. |

Module 04 solves an operational cost-minimization problem for residential local energy systems, considering distributed resources, flexible demand and simplified low-voltage line-capacity constraints. It is **not** a full AC power-flow model.

The standard execution sequence is **Module 01 -> Module 02 -> Module 03 -> Module 04**. Some stages can be run independently if their upstream outputs are available. The notebooks use configurable execution flags; saved settings do not necessarily activate all generation stages.

See the [Module guide](docs/MODULE_GUIDE.md) for model components, configuration and execution, and the [Output structure](docs/OUTPUT_STRUCTURE.md) for file locations and run metadata.



## Installation and requirements

HAGRID is designed for Jupyter Notebook and Python and has been developed using Visual Studio Code. Install the Python dependencies declared in the project's tested `environment.yml`:

```bash
conda env create -f environment.yml
conda activate HAGRID
```

The environment name is determined by the `name:` field in `environment.yml`.

**Optimization requirements.** Module 04 formulates its optimization model with **oemof.solph** and **Pyomo**. Its current implementation calls the **Gurobi** solver and imports `gurobipy`; optimization runs therefore require a working Gurobi installation and an appropriate license. Installation of CBC alone does not enable CBC execution in the supplied Module 04; this alternative would require changes to solver selection and Gurobi-specific diagnostics.

- [oemof.solph documentation](https://oemof-solph.readthedocs.io/)
- [Gurobi documentation](https://docs.gurobi.com/)

The notebooks also require the input datasets and assumptions referenced in `input/`. Availability and redistribution of third-party input data depend on their licenses and sources.



## Running HAGRID

1. Work from the **repository root** so that `input/` and `output/` are resolved correctly. Module 04 uses the current working directory to identify the project root.
2. Confirm that the necessary input files and previously generated intermediate profiles are available for the stages you intend to run.
3. Set consistent household numbers, geographical region, selected periods, time resolution and configuration parameters across the notebooks.
4. Configure the relevant `RUN_*` flags. Enable upstream generation stages for a new case study; comparison-only modes require existing results.
5. Execute the relevant stages in module order, inspecting their diagnostics and output files before proceeding.

The current source notebooks contain development-specific settings; adapt them to the chosen case study before a full run.



## Scenarios, run identity and outputs

Module 04 separates the physical electrification experiment (`ACTIVE_ELECTRIFICATION_EXPERIMENT`) from the control-case group (`ACTIVE_CONTROL_GROUP`). A `RUN_REVISION` tag labels modelling assumptions associated with a run. The resulting `run_uid` and `run_label` are used to organize output folders and to index results in `HAGRID_run_manifest.csv` / `.xlsx`.

`RUN_REVISION` is a **simulation metadata field**, not the software release version published on GitHub.

Module 04 can report household and system energy/economic indicators, optimization results, SOC for flexible storages, and scenario comparisons. SOC for optimized storage components is recovered from `oemof.solph` `storage_content` results, with additional checks. Exports supporting separate power-flow studies are available, but they are not themselves a full AC power-flow simulation.

**Repository contents.** This repository provides the four HAGRID source notebooks, selected input data and representative intermediate/output examples. The complete set of generated profiles, optimized scenarios, run caches and simulation results is not distributed. Some analysis and comparison modes require upstream outputs to be regenerated or otherwise supplied. For Kerber network scenarios, Module 04 uses pre-generated grid blueprint JSON files in `output/05_Kerber_grid_archetypes_Py/`; the Module 05 power-flow workflow is not included in this release.

Generated outputs and intermediate profiles are described in the [Output structure](docs/OUTPUT_STRUCTURE.md).



## Reproducibility and limitations

- Record input data versions, stochastic seeds, household compositions, geographical assumptions, active flags, experiment selections and run revisions when reporting results.
- Reuse or regenerate intermediate profiles consistently; later modules depend on upstream file contents and naming conventions.
- The optimization model imposes simplified LV-network constraints, not a full AC power-flow formulation.
- Solver availability and the required input data affect which workflows can be executed in a given installation.


## Citation and license

Use [`CITATION.cff`](CITATION.cff) to cite the software. Details of associated scientific publications can be added when available. HAGRID is distributed under the [MIT License](LICENSE).
