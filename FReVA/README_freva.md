# FreVA in NaaVRE

NaaVRE migration of Yujun Wei's FreVA linear programming model (Food Resource Valorisation and Allocation). Second case study in the thesis after BGIS.

## What this does

The model allocates food-loss streams to platform chemicals subject to supply and demand constraints. Three regional datasets are available: CN (22 streams), EU (19 streams), US (13 streams). Two objectives: maximise profit, or maximise GWP saving (in CO2-equivalent terms).

Originally a Jupyter notebook from Yujun Wei using `pyomo` and the GLPK solver. Migrated into a four-cell NaaVRE workflow.

## Workflow components

1. **Data Loader**: reads the five sheets from the FL2CH Excel file (eta, FL, CH, profit, GWP), converts them to the dict structures Pyomo expects, pickles them to `/tmp/data/` for the next cell.
2. **Solve LP**: builds the Pyomo `ConcreteModel` (sets, parameters, decision variables, objective, constraints), solves with GLPK, writes solution values to `/tmp/data/` as JSON. Building and solving live in the same cell because a Pyomo model cannot be unpickled across container boundaries; the constraint rule functions are defined per-container.
3. **Results Writer**: reads the solution JSON, writes the allocation as a CSV and a summary as a JSON to Cloud Storage.

(A fourth cell, the params cell, holds the `param_` variables. It is not containerised; values flow from it into the three containerised cells via the workflow engine.)

## Input files

One Excel file per region, with five sheets: `eta` (yield matrix), `FL` (food-loss quantities), `CH` (chemical demand), `profit` (per-chemical revenue), `GWP` (per-chemical GWP saving).

The default inputs are pre-uploaded to the public bucket:

```
/home/jovyan/Cloud Storage/naa-vre-public/vl-bgis-urban-planning/
  FL2CH_CN.xlsx   (China, 22 food-loss streams, 11 chemicals)
  FL2CH_EU.xlsx   (Europe, 19 food-loss streams, 11 chemicals)
  FL2CH_US.xlsx   (US, 13 food-loss streams, 11 chemicals)
```

To run with your own data, upload your file to `/home/jovyan/Cloud Storage/naa-vre-user-data/` and override the `param_input_xlsx_path` parameter.

## Parameters

These can be changed at workflow runtime:

| Parameter | Default | Description |
|-----------|---------|-------------|
| `param_input_xlsx_path` | `naa-vre-public/.../FL2CH_CN.xlsx` | Path to FL2CH Excel input |
| `param_region` | `CN` | `CN`, `EU`, or `US`. Used in the summary output and as a fallback for path resolution when `param_input_xlsx_path` is empty. |
| `param_objective` | `profit` | `profit` or `gwp` |
| `param_num_fl_streams` | `0` | Number of food-loss streams. `0` means auto-detect from the sheet. |
| `param_num_chemicals` | `0` | Number of chemicals. `0` means auto-detect from the sheet. |
| `param_output_csv_path` | `naa-vre-user-data/freva_results.csv` | Allocation flows output |
| `param_output_summary_path` | `naa-vre-user-data/freva_summary.json` | Solver summary output |

## Outputs

`<param_output_csv_path>` is one row per non-zero flow with columns `fl_stream`, `chemical`, `flow`, `yield_eta`, `chemical_produced`.

`<param_output_summary_path>` is a JSON with objective value, solver status, set sizes, per-stream FL usage, per-chemical CH produced.

## Verification

Output matches Yujun's original notebook to four decimal places on the CN dataset. Profit objective value `27,092,387,898.71` with status `ok` and termination `optimal`.

## Dependencies

Managed through the `bgis-urban-planning` flavour (`environment.yaml`): `pandas`, `openpyxl`, `pyomo`, `glpk`. GLPK is a system binary and must be in the conda dependencies, not pip.

## Links

- Virtual Lab: https://beta.naavre.net/vreapp/vl/bgis-urban-planning
- Flavour repo: https://github.com/NaaVRE/flavors/tree/main/flavors/bgis-urban-planning
- Original FreVA code: https://github.com/weiyujun18/Food2Chemical

## Contact

- Sven Tesselaar (migration): sven.tesselaar@student.uva.nl
- Dr. Zhiming Zhao (supervisor): z.zhao@uva.nl
- Yujun Wei (original FreVA model): yujun.wei@wur.nl
