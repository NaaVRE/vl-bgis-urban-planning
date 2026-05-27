# BGIS Urban Planning Virtual Lab

Two migrated research workflows hosted in the NaaVRE `bgis-urban-planning` Virtual Lab. Part of Sven Tesselaar's BSc thesis at the University of Amsterdam.

## Workflows

**BGIS**: Blue-green infrastructure siting and optimization. Originally by Kim Wang at Wageningen University & Research. See [BGIS/README_bgis.md](BGIS/README_bgis.md) for details.

**FreVA**: Linear programming model that allocates food-loss streams to platform chemicals. Originally by Yujun Wei at Wageningen University & Research. See [FReVA/README_freva.md](FReVA/README_freva.md) for details.

The two workflows are independent. They share the same Virtual Lab, flavour image and Cloud Storage backend, nothing else. The Virtual Lab demonstrates that one shared cloud environment can host workflows from multiple researchers.

## Running a workflow

1. Open the NaaVRE Virtual Lab in your browser: https://beta.naavre.net/vreapp/vl/bgis-urban-planning
2. Log in
3. From the file browser, open one of the `.naavrewf` workflow files (BGIS or FreVA)
4. Click **Run** at the top of the workflow editor
5. Edit parameters in the dialog or accept the defaults
6. Click **Run** to submit

You should see a "Workflow submitted!" confirmation. Click "Show in workflow engine" to monitor progress; green checkmarks indicate completed steps.

You do not need to edit the source notebooks to run the workflows. The notebooks are kept in the repo as reference; the `.naavrewf` files are the actual workflows.

## Cloud Storage

The Virtual Lab uses two Cloud Storage areas:

**Public sample data** (read-only): `/home/jovyan/Cloud Storage/naa-vre-public/vl-bgis-urban-planning/`

The default input files for both workflows live here. Everyone in the Virtual Lab can read them. Use them to try a workflow without uploading anything.

```
example_amsterdam_neighborhood.gpkg     BGIS example input
bgi_catalog_v1.xlsx                     BGIS BGI catalog
FL2CH_CN.xlsx                           FreVA China dataset
FL2CH_EU.xlsx                           FreVA Europe dataset
FL2CH_US.xlsx                           FreVA US dataset
```

**Per-user storage** (read-write): `/home/jovyan/Cloud Storage/naa-vre-user-data/`

Workflow outputs go here, and so do any custom input files you upload. Each user has their own copy.

To run a workflow with your own data: upload your file to `naa-vre-user-data/` (drag and drop into the JupyterLab file browser) and override the relevant input path parameter when submitting the workflow.

## Repo structure

```
BGIS/
  bgis.naavrewf            BGIS workflow (run this)
  bgis.ipynb               Source notebook (reference)
  README_bgis.md           BGIS documentation
FReVA/
  freva.naavrewf           FreVA workflow (run this)
  freva.ipynb              Source notebook (reference)
  README_freva.md          FreVA documentation
README.md                  This file
LICENSE
CITATION.cff
```

## Flavour

Both workflows use the `bgis-urban-planning` flavour. Required packages: `geopandas`, `pyogrio`, `pandas`, `numpy`, `openpyxl`, `matplotlib`, `pulp`, `pyomo`, `glpk`.

## Contact

- Sven Tesselaar (migration): sven.tesselaar@student.uva.nl
- Dr. Zhiming Zhao (supervisor): z.zhao@uva.nl
