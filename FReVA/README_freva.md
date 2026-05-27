# FreVA LP in NaaVRE

NaaVRE migration of Yujun Wei's FreVA linear programming model (Food Resource
Valorisation and Allocation). Second case study in the thesis after BGIS.

## What this is

The model allocates food-loss streams to platform chemicals subject to supply
and demand constraints. Three regional datasets are available: CN (22 streams),
EU (19 streams), US (13 streams). Two objectives: maximise profit, or maximise
GWP saving.

## Files

```
freva_lp_v2.ipynb       The workflow notebook
data/
  FL2CH_CN.xlsx         China dataset
  FL2CH_EU.xlsx         Europe dataset
  FL2CH_US.xlsx         US dataset
```

## Setup in NaaVRE

1. Upload the notebook and the `data/` folder to your JupyterLab workspace.
2. Make sure `glpk` is in the bgis-urban-planning flavour's `environment.yaml`.
   It's a system binary, not a Python package, so it needs to be in the conda
   dependencies, not pip.
3. Open the notebook, edit Cell 1 if you want to switch region or objective,
   run all cells.

## Containerisation

Same pattern as BGIS:

- Cell 0 (markdown header): not containerised.
- Cell 1 (params): not containerised. Holds all `param_` variables.
- Cells 2 through 5: containerise each one.
- Do not include a `dependencies` block in the cell YAML. Auto-detection
  handles imports correctly. Adding the block strips imports that aren't
  listed (this was the issue with `pickle` in BGIS).

If files are uploaded to Cloud Storage instead of the workspace, set
`param_input_xlsx_path` to the absolute Cloud Storage path. This overrides
`param_region`.

## Parameters

| Parameter | Default | Meaning |
|---|---|---|
| `param_region` | `CN` | `CN`, `EU`, or `US`. Used to build the data path. |
| `param_input_xlsx_path` | `""` (empty) | Explicit path override. Overrides `param_region` when set. |
| `param_objective` | `profit` | `profit` or `gwp` |
| `param_num_fl_streams` | `0` | I size; 0 = auto from sheet |
| `param_num_chemicals` | `0` | J size; 0 = auto from sheet |
| `param_output_csv_path` | `freva_results.csv` | Allocation flows output |
| `param_output_summary_path` | `freva_summary.json` | Solver summary output |

## Cells

0. Header (markdown)
1. Params (top params cell, not containerised)
2. Data loader (Excel sheets to Pyomo dicts, resolves path from region)
3. Model builder (sets, params, vars, objective, constraints)
4. Solver (GLPK)
5. Results writer (CSV + JSON)

## Outputs

`<param_output_csv_path>` is one row per non-zero flow with columns
`fl_stream`, `chemical`, `flow`, `yield_eta`, `chemical_produced`.

`<param_output_summary_path>` is a JSON with objective value, solver status,
set sizes, per-stream FL usage, per-chemical CH produced.

## Verification

Output matches Yujun's original notebook to four decimal places on the CN
dataset. Profit objective value 27,092,387,898.71 with status `ok` and
termination `optimal`.
