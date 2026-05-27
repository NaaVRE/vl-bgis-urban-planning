# BGIS Urban Planning Virtual Lab

## What this does

This Virtual Lab runs the BGIS (Blue-Green Infrastructure Siting) analysis as a cloud workflow in NaaVRE. Give it land use polygons and a BGI catalog, and it works out which blue-green infrastructure types fit each parcel — then picks one per parcel to best meet your cooling and water retention targets.

Originally developed as two Python scripts by Kim Wang at Wageningen University & Research. Migrated to NaaVRE by Sven Tesselaar (UvA) as part of a BSc thesis supervised by Dr. Zhiming Zhao.

## Workflow components

1. **Data Loader**: reads a GeoPackage (land use polygons) and an Excel file (BGI catalog), parses them and saves to intermediate storage.
2. **Land Cover Mapper**: translates land use type labels into generic installation layers using a configurable dictionary. Defaults to Amsterdam BGT labels.
3. **Feasibility Checker**: checks each parcel against each BGI type on two criteria: installation layer compatibility and area constraints.
4. **Output Generator**: creates a suitability column per BGI type and writes the feasibility result as a GeoPackage.
5. **Optimizer**: takes the feasibility output and selects one BGI per parcel using binary integer programming (PuLP). Optimises toward user-defined cooling and water retention targets using a soft-goal formulation — it minimises the penalty-weighted shortfall rather than rejecting runs that fall short. Outputs a GeoPackage and a static PNG map.

## Input files

Upload these to `/home/jovyan/Cloud Storage/naa-vre-user-data/` before running:

- A GeoPackage file (`.gpkg`) with land use polygons. Each polygon needs a `type` column with a land use label and a geometry column. The example file `example_amsterdam_neighborhood.gpkg` covers a section of Amsterdam's canal district.
- An Excel file (`.xlsx`) with a BGI catalog. Each row is a BGI type with columns for `bgi_name`, `install_layer`, `area_min`, `area_max`, `temperature_reduction`, `water_storage_volume`, `CAPEX`, and other properties. The example file `bgi_catalog_v1.xlsx` contains 9 BGI types.

## Parameters

These can be changed at workflow runtime:

| Parameter | Default | Description |
|-----------|---------|-------------|
| `param_input_gpkg_path` | `.../naa-vre-user-data/example_amsterdam_neighborhood.gpkg` | Path to input GeoPackage in Cloud Storage |
| `param_input_catalog_path` | `.../naa-vre-user-data/bgi_catalog_v1.xlsx` | Path to BGI catalog Excel file in Cloud Storage |
| `param_landcover_dict_path` | `default` | Path to a custom landcover dictionary JSON file, or `"default"` for Amsterdam |
| `param_output_gpkg_path` | `.../naa-vre-user-data/testing_standalone.gpkg` | Path where the feasibility output GeoPackage will be saved |
| `param_cooling_goal` | `1.0` | Target average temperature reduction in °C per m² |
| `param_retention_goal` | `0.2` | Target average water storage depth in m per m² |
| `param_penalty_cooling` | `1.0` | Penalty weight for unmet cooling target (relative to retention) |
| `param_penalty_retention` | `1.0` | Penalty weight for unmet retention target (relative to cooling) |
| `param_optimization_output_path` | `.../naa-vre-user-data/bgis_optimization_result.gpkg` | Path where the optimization result GeoPackage will be saved |

The cooling and retention goals are expressed as averages across the total parcel area. The theoretical maximum for your dataset is printed to the workflow log at runtime — use that as a reference when setting targets.

## Custom landcover dictionary

The default mapping translates Amsterdam BGT type labels (like `rijbaan_lokale_weg`, `groenvoorziening`) to generic installation layers (like `pavement`, `ground`). If you are working with data from a different city, you can provide your own mapping as a JSON file.

Upload the JSON file to Cloud Storage and set `param_landcover_dict_path` to its path when running the workflow.

Example format:

```json
{
    "local_road": "pavement",
    "green_space": "ground",
    "flat_roof": "flatroof",
    "canal": "water"
}
```

## Viewing the output

The optimizer cell saves a static PNG map to Cloud Storage alongside the output GeoPackage. The PNG file has the same name as the GeoPackage with `_map.png` appended. You can open it directly in JupyterLab without downloading anything.

For a full interactive view, download the output GeoPackage from Cloud Storage and open it in QGIS:

**Feasibility output:**
1. Open the file in QGIS
2. Right-click the layer, go to Properties then Symbology
3. Change from Single Symbol to Categorized
4. Select a BGI column (e.g. `green_roof`)
5. Click Classify — parcels with value 1 are suitable, 0 are not

**Optimization output:**
1. Open the file in QGIS
2. Right-click the layer, go to Properties then Symbology
3. Change from Single Symbol to Categorized
4. Select the `selected_bgi` column
5. Click Classify — each colour shows a different BGI type; grey parcels have no BGI selected

## Dependencies

Managed through the `bgis-urban-planning` flavour (`environment.yaml`): `geopandas`, `pyogrio`, `pandas`, `numpy`, `openpyxl`, `matplotlib`, `pulp`.

## Links

- Virtual Lab: https://beta.naavre.net/vreapp/vl/bgis-urban-planning
- Flavour repo: https://github.com/NaaVRE/flavors/tree/main/flavors/bgis-urban-planning
- Original BGIS code: https://github.com/bgishub/BGIS

## Contact

- Sven Tesselaar (migration): sven.tesselaar@student.uva.nl
- Dr. Zhiming Zhao (supervisor): z.zhao@uva.nl
- Kim Wang (original BGIS tool): kim.wang@wur.nl
