# DN-to-Reflectance Calibration (DJI Mavic 3M, Empirical Line Method)

Converts DJI Mavic 3 Multispectral (M3M) DN orthomosaics to surface reflectance using the
Empirical Line Method (ELM), fitted from a single capture containing four reference tarps
(10%, 20%, 30%, 40% reflectance).

Compared to Metashape's built-in radiometric calibration (which only supports one reflectance
target), this notebook fits a per-band linear model from all four known reflectance levels,
and compares it against a black-level-corrected, origin-forced variant that mirrors
Metashape's single-panel assumption.

## Data availability

**No imagery is included in this repository.** The drone captures used to develop and test
this notebook belong to the source research institution and are not redistributed here.
`examples/` contains the result figures (calibration fits, model comparison, validation) from
an actual run, so you can see what the pipeline produces without needing the raw data. To run
the notebook yourself, supply your own DJI M3M tarp capture and DN orthomosaic (see below).

| Figure | Shows |
|---|---|
| `examples/elm_fit.png` | Free-intercept ELM fit per band (DN vs. known tarp reflectance) |
| `examples/elm_model_comparison.png` | Free-intercept vs. black-level-corrected model, extrapolated to DN=0 |
| `examples/elm_validation.png` | Calibrated vs. target reflectance at the tarp locations (RMSE per band) |

## Requirements

- Python 3.10+ (developed against a `conda` environment; see `requirements.txt` for pinned
  package versions)
- A Jupyter-capable environment with GUI support (the ROI-selection step opens OpenCV
  windows and requires a display — it will not run headlessly, e.g. over SSH without X
  forwarding, or in CI)

```powershell
pip install -r requirements.txt
```

## Data layout

The notebook does not ship any imagery (see above). Point it at your own data either by
editing the default paths in the notebook's config cell, or by setting environment variables
before launching Jupyter:

| Environment variable | Purpose | Default |
|---|---|---|
| `DJI_M3M_TARP_DIR` | Folder containing the 4 single-band tarp TIFFs (`tarp_green.tif`, `tarp_red.tif`, `tarp_rededge.tif`, `tarp_nir.tif`) | `./data/tarps` |
| `DJI_M3M_MAIN_DN_PATH` | DN orthomosaic to calibrate (band order: Green, Red, RedEdge, NIR) | `./data/sample_dn_orthomosaic.tif` |
| `DJI_M3M_OUTPUT_DIR` | Where calibrated GeoTIFFs and validation figures are written | `./output/` |

`data/` and `output/` are git-ignored — populate them locally, they will never be committed.

```powershell
$env:DJI_M3M_TARP_DIR = "D:\data\DJI_M3M\<site>\<date>\<flight>\"
$env:DJI_M3M_MAIN_DN_PATH = "D:\projects\drone\Metashape\<site>\<ortho>.tif"
jupyter lab dn_to_reflectance.ipynb
```

## Usage

Run `dn_to_reflectance.ipynb` top to bottom in an interactive Jupyter session:

1. **Config cell** — verifies that all input files exist.
2. **Manual ROI selection** — for each of the 4 bands, drag a rough box around the tarp
   area, then click polygon vertices around each of the 4 tarp panels (10% → 40%). This
   step is interactive by design (tarps are not axis-aligned, and per-band optical centers
   differ slightly) and cannot be automated.
3. **Model fitting** — fits both a free-intercept ELM model and a black-level-corrected
   model per band, and plots the fit.
4. **Apply & export** — applies both models to the full DN orthomosaic and writes two
   calibrated GeoTIFFs.
5. **Validation** — compares calibrated reflectance at the tarp locations against the known
   tarp reflectance values (RMSE per band).

## Notes

- Figures are saved at `dpi=300`; axis tick labels use `labelsize=12`.
