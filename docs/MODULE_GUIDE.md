# HAGRID module guide

This guide describes the four notebooks and their main inputs, outputs, configuration parameters and execution stages. Each notebook contains user settings, path definitions, processing functions and selectively enabled stages.


## Module 01 — individual activity profiles

**Inputs:** Italian ISTAT Time-Use Surveys TUS source/aggregated files beneath `input/01_input_profiles_att/`; optional original TUS files for rebuild stages. **Processing:** prepare/validate aggregated TUS input data, estimate activity/place transition matrices and initial-state probabilities for individual classes and geographical policies, simulate annual 10-minute profiles, and extract selected seasonal weeks. **Outputs:** `output/01_Mtx_activ/`, `output/01_Mtx_places/`, `output/01_activity_profiles/yearly_profiles/`, `output/01_activity_profiles/weekly_profiles/`, `output/01_profiles_Pics/`, and `output/01_diagnostics/`.

Main stage flags: `RUN_AGGR_DATASETS`, `RUN_DEBUG_INPUTS`, `RUN_MATRICES`, `RUN_PLOTS`, `RUN_YEARLY_PROFILES`, `RUN_WEEKLY_PROFILES`. Its code explicitly recommends keeping stage groups separate. `RANDOM_SEED = None` creates non-identical stochastic realizations unless changed. `selected_periods` currently maps to weeks of 2023, and those windows must agree with downstream modules.



## Module 02 — synthetic households and conventional loads

**Inputs:** Module 01 weekly profiles, `input/02_consumption/Database_appliances_ENG-ITA.xlsx` and related assumptions. **Processing:** sample household compositions and select individual profiles; reconstruct household activities; associate appliance and lighting load profiles with the activities; calculate household/aggregate electricity and diagnostics, optionally plot appliance profiles. Compatibility helpers accept certain historic Italian-language labels and English canonical labels.

**Outputs:** `output/02_household_activity_profiles_Xls/`, `output/02_base_consumption_Xls/` and `output/02_base_consumption_Pics/`.

Main flags: `RUN_CREATE_HOUSEHOLDS`, `RUN_CONSUMPTION`, `RUN_APPLIANCE_PLOTS` and optional coherence checks. **Example settings in the current Module 02 notebook:** `num_households = 146`, `selected_region = 'Cen'`, `individual_profile_sample_size = 200`. These must be aligned with the other modules before executing a shared case study.



## Module 03 — occupancy and additional electrified loads

**Inputs:** Module 02 household profiles/conventional demand, `input/03_additional_cons/` (including heating/climate and cooling profiles). **Processing:** reconstruct occupancy, space heating/cooling, induction cooking, thermal Domestic Hot Water DHW demand and EV travel/charging data, then assemble household and aggregated time series and annual indicators.

Main implemented mechanisms:

- **DHW:** clusters nearby `Personal care & dressing` activity occurrences into hygiene events (`personal_care_cluster_max_gap_ts`), samples shower occurrence by time-of-day probabilities, handles simultaneity/temporal shifting and exports generation/shift audits. Relevant assumptions include storage capacity, charge power, standing-loss specifications and shower energy.
- **EV:** generates mobility-related battery-demand estimates and home/work charging reference profiles under assumed charging power and availability, along with per-person EV metadata. A separate read-only extreme/long-trip audit examines existing mobility profiles; it does not regenerate the demand profiles.
- **Heating/cooling and cooking:** constructs demand from activity/occupancy and thermal or appliance assumptions and reports aggregate/household indicators.
- **Plots/diagnostics:** weekly stacked load charts, combined activity-plus-demand plots and annual percentile summaries by household type or across households.

**Outputs:** `output/03_additional_consumption_Xls/` and `output/03_additional_consumption_Pics/<n>_households_<region>/`. Among the Excel outputs are weekly aggregates, annual values/statistics, DHW audits, EV energy/mobility metadata, and household consumption series.

Main flags: `RUN_ADDITIONAL_LOADS`, `RUN_ADDITIONAL_LOAD_PLOTS`, `RUN_REBUILD_THERMAL_PROFILE`, `RUN_EV_TRIP_EXTREME_AUDIT` and further `PLOT_*` controls. **Example settings in the current Module 03 notebook:** `num_households = 146`, `RUN_ADDITIONAL_LOADS = False`, `RUN_ADDITIONAL_LOAD_PLOTS = True`, `RUN_EV_TRIP_EXTREME_AUDIT = True`. To regenerate data, enable the relevant generation flags and align the household number with Module 02.



## Module 04 — local energy system optimization and scenario comparison

**Inputs:** Module 02 and Module 03 outputs; 10-minute PV and electricity prices (or their source hourly data for generation); pre-generated LV Kerber blueprint JSON files in `output/05_Kerber_grid_archetypes_Py/` for selected Kerber archetypes; optional power-flow-derived limits and existing RB/optimization output for reuse. The location under `output/` is historical and is retained by the current notebook; these JSON files are **inputs** to Module 04. The Module 05 power-flow notebook is not part of this release.

**Model:** 10-minute simulation of the selected population on a simplified Kerber LV network archetype. The `ELECTRIFICATION_EXPERIMENTS` catalog defines physical technology penetration scenarios; `CONTROL_GROUPS` selects conventional versus optimized control combinations. Household technology ownership is assigned and shared as relevant across scenario comparisons. Operations are modelled with `oemof.solph`, with simplified line capacity limits and economic variables. Gurobi is the named solver in the current implementation (`po.SolverFactory('gurobi')`).

**Energy resources and flexibility:** PV, BESS, EV/BEV (including V2G), ESWH/DHW, heat pumps, cooling and induction loads. EV public/outdoor charging is parametrized separately from home charging. ESWH reference control uses SOC hysteresis (`SOC_HYST_DEFAULT = (0.89, 1)` in the current notebook configuration), whereas the optimized ESWH storage uses its own `min_storage_level_ESWH` parameter (0.1 in the current notebook configuration) and additional demand/boost feasibility measures. Do not confuse the two distinct lower bounds. Optimized BESS, BEV and DHW storage states are extracted from `storage_content`, with timing alignment and diagnostics; the initial state in models using `balanced=True`, `initial_storage_level=None` is decision-dependent but constrained cyclically.

**Run structure:** `ACTIVE_ELECTRIFICATION_EXPERIMENT`, `ACTIVE_CONTROL_GROUP`, `RUN_REVISION`, target technology shares, grid archetype, model settings, and random seeds contribute to the run configuration and deterministic `run_uid`. A central `HAGRID_run_manifest.csv`/`.xlsx` tracks saved runs, locations and statuses. RB profile caches are revision-aware; when reusing older references the explicitly provided revision override must be consistent with the change in assumptions.

**Software version versus run revision:** `RUN_REVISION` identifies an assumption/revision context for simulated runs and contributes to output identity; it is distinct from the version tag of the source-code release on GitHub.

**Post-processing:** household/system economic accounting, objective-cost reconciliation, SOC comparison/distribution plots (including minimum curves), annual/multi-run summary and configuration comparison charts, LV line-loading outputs, and downstream power-flow input exports. Optional import of PF-calibrated constraints is explicitly described as under development in the code.

**Main execution flags:**

| Flag | Action |
| --- | --- |
| `RUN_BUILD_10MIN_PROFILES` | Rebuild 10-minute PV/price input profiles. |
| `RUN_BUILD_RB_REFERENCE` | Create rule-based reference profiles. |
| `RUN_SOLVE_MIXED_SCENARIOS` | Build/solve selected operational optimization runs. |
| `RUN_PLOT_SCENARIO_GRAPHS` | Plot individual saved configurations. |
| `RUN_COMPARE_MULTIRUN` | Produce cross-run comparisons from saved results. |
| `RUN_COMPARE_DHW_SOC_TWO_CONFIGS` | Compare ESWH SOC across two selected saved configurations. |
| `USE_PF_LINE_LIMITS_V1` | Optionally read externally derived power-flow limits; development workflow. |

The number of synthetic households (`num_households`), selected geographical region (`selected_region`), and individual profile sample size (`individual_profile_sample_size`) are user-configurable. The household population and other relevant settings must be consistent with the downstream modules when executing the complete workflow.

**Working-directory requirement:** the notebook uses `PROJECT_ROOT = os.path.abspath(os.getcwd())` and expects `input/` beneath this location. Start the Jupyter kernel in the repository root and ensure required datasets and a working Gurobi environment are available.

## Running the complete workflow

1. Ensure input datasets are present and that licensing allows their distribution.
2. Align `num_households`, `selected_region`, period dates, sample sizes and the power/PV/thermal time reference across modules.
3. Choose reproducible seeds. Decide whether to build new profiles or reuse cached/intermediate outputs.
4. Enable generation stages in order and verify that each expected output exists before the next module.
5. In Module 04, decide whether RB caches need rebuilding; ensure selected experiment/control cases and run revision are coherent.
6. Solve a small representative test configuration, then verify nonempty outputs, SOC consistency, economic reconciliation and manifest entries before launching broad studies.
