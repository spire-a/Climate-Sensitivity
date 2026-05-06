README FOR CLIMATE SENSITIVITY ANALYSIS WORKFLOW
====================

Project: Structural breaks reveal accelerating climate sensitivity of global cereal production

This archive contains the scripts, input data, and prepared output folders needed to reproduce the structural-break and first-difference panel results used in the manuscript and supplementary information.


1. ARCHIVE STRUCTURE
--------------------

After unzipping, please keep the folder structure unchanged:

  project_root/
  |-- 1_Climate_sensitivity_Define_breaks_.qmd
  |-- 2_Climate_Sensitivity_diff_panels.qmd
  |-- final_curated_supplementary_information.qmd
  |-- data/
  |-- output/
  |-- README_for_users.txt

The scripts are written to use paths relative to the folder in which the scripts are located. In other words, the `data/` and `output/` folders should remain inside the same unzipped root folder as the `.qmd` files.

The `data/` folder contains the input data files needed by the first script, including FAOSTAT crop data, country and crop mappings, ERA5 monthly climate series, NDVI growing-season information, cropland area data, and map files.

The `output/` folder contains prepared outputs and is also where regenerated results are written if the scripts are rerun.


2. SOFTWARE REQUIREMENTS
------------------------

Recommended environment:

  - R, preferably version 4.3 or later
  - RStudio with Quarto support, or the Quarto command-line interface
  - A working LaTeX / PDF device is useful for PDF figure export, but not required for the CSV outputs

The main R packages used across the scripts are:

  readxl, dplyr, purrr, stringdist, writexl, readr, tidyr, stringr,
  broom, ggplot2, future.apply, strucchange, zoo, patchwork, WDI,
  marginaleffects, kableExtra, lmtest, plm, sp, margins, VIM,
  ggbeeswarm, flextable, openxlsx, sf, rnaturalearth, rnaturalearthdata,
  scales, e1071, gridExtra, knitr, ragg

If packages are missing, they can be installed from CRAN. For example:

  install.packages(c(
    "readxl", "dplyr", "purrr", "stringdist", "writexl", "readr",
    "tidyr", "stringr", "broom", "ggplot2", "future.apply",
    "strucchange", "zoo", "patchwork", "WDI", "marginaleffects",
    "kableExtra", "lmtest", "plm", "sp", "margins", "VIM",
    "ggbeeswarm", "flextable", "openxlsx", "sf", "rnaturalearth",
    "rnaturalearthdata", "scales", "e1071", "gridExtra", "knitr", "ragg"
  ))


3. PATH SETUP
-------------

The scripts are intended to be run from the unzipped project root. They define the input and output folders relative to the script location.

When using `paste0()` for paths, the path variables should include a trailing slash:

  script_dir <- dirname(rstudioapi::getActiveDocumentContext()$path)
  datpth    <- paste0(script_dir, "/data/")
  datpthout <- paste0(script_dir, "/output/")

This allows calls such as:

  readr::read_rds(paste0(datpth, "globalmap.rds"))

and:

  write.csv(results, paste0(datpthout, "some_output.csv"))

If using `file.path()` instead, the trailing slash is not needed. The important point is that all scripts should point to the local `data/` and `output/` folders inside the unzipped archive.

If a script still contains a local absolute path such as `C:/Users/.../output/`, replace it with the relative `datpthout` definition above before rendering.


4. RECOMMENDED RUN ORDER
------------------------

The scripts should be run in the following order.

Step 1: Define structural breaks and prepare the main processed dataset

  Script:
    1_Climate_sensitivity_Define_breaks_.qmd

  Purpose:
    This is the main data-preparation and structural-break script. It:
      - loads and cleans FAOSTAT crop yield, area, and production data;
      - constructs selected synthetic regions where needed;
      - loads ERA5 monthly climate variables;
      - assigns NDVI-based growing-season windows;
      - computes seasonal GDD and precipitation measures;
      - merges yield and climate data;
      - runs country-crop structural-break tests;
      - identifies dominant break years;
      - creates before/after break flags;
      - exports processed datasets, diagnostic tables, and selected figures.

  Main outputs written to `output/`:
    - Processed_data_breaks_flagged_YYYYMMDD.csv
      Main processed dataset with climate variables and before/after break flags.

    - Processed_only_breaks_flagged_YYYYMMDD.csv
      Subset containing only country-crop panels with statistically validated breaks.

    - Fitted_lines_breaksYYYYMMDD.csv
      Fitted levels-model summaries for pre- and post-break segments.

    - Structural_Change_Appendices_FINAL.xlsx
      Multi-tab Excel workbook containing model-selection, break-driver, phase-performance,
      F-statistic-path, and BIC-path outputs.

    - France_StructuralBreak_Diagnostics.png / .pdf
      France case-study diagnostic figure.

    - Figure_Breaks_Nature.png / .pdf
      Main structural-break distribution figure.

    - Supplementary/Table_S1_Break_Summary.csv
    - Supplementary/Table_S2_Regime_Levels_AMEs.csv
    - Supplementary/Table_S3_Crop_Level_Summary.csv
    - Supplementary/Table_S4_Heat_Penalty_Direction.csv
      Supplementary CSV tables generated from the levels-based break analysis.

  Notes:
    This is the longest-running script because it estimates structural breaks over many country-crop panels. It should be run first if full regeneration is required.


Step 2: Estimate first-difference panel models

  Script:
    2_Climate_Sensitivity_diff_panels.qmd

  Purpose:
    This script uses the latest `Processed_data_breaks_flagged_YYYYMMDD.csv` file in `output/` and estimates the first-difference panel models used for the main climate-sensitivity interpretation. It:
      - loads the processed break-flagged dataset;
      - keeps country-crop panels with validated breaks for the baseline model;
      - estimates crop-specific first-difference panel models;
      - applies country-clustered robust standard errors;
      - computes average marginal effects by regime;
      - generates stress-test / placebo specifications.

  Main outputs written to `output/`:
    - FD_Pool_coeffs_YYYYMMDD.csv
      First-difference pooled model coefficients by crop.

    - FD_Pool_AMEs_YYYYMMDD.csv
      Regime-specific average marginal effects by crop.

  Optional output:
    - Supplementary/Annex_1_Supplementary_FD_Panel_and_Robustness_YYYYMMDD.pdf
      Optional annex PDF. The annex block is marked `eval=FALSE` in the script and can be enabled if needed.

  Notes:
    This script depends on the output of Step 1. If `Processed_data_breaks_flagged_YYYYMMDD.csv` already exists in `output/`, the script can be run without rerunning Step 1.


Step 3: Render the curated supplementary information document

  Script:
    final_curated_supplementary_information.qmd

  Purpose:
    This script creates the user-facing supplementary information document. It reads the latest relevant files from `output/` and assembles a curated document containing:
      - France-specific levels-based structural-break diagnostics;
      - compact sample-definition information;
      - first-difference coefficient decomposition;
      - regime-specific average marginal effects.

  Expected output:
    - final_curated_supplementary_information.docx

  Notes:
    This script assumes the main outputs from Steps 1 and 2 are already present in `output/`. If the document cannot find a table or figure, check that the corresponding CSV or PNG file exists in `output/` and that `datpthout` points to the local `output/` folder.


5. HOW TO RUN THE SCRIPTS
-------------------------

Option A: From RStudio

  1. Unzip the archive.
  2. Open the unzipped folder in RStudio, or open the `.qmd` files directly from that folder.
  3. Render the scripts in this order:
       1_Climate_sensitivity_Define_breaks_.qmd
       2_Climate_Sensitivity_diff_panels.qmd
       final_curated_supplementary_information.qmd
  4. Check the `output/` folder for regenerated CSV, XLSX, PNG, PDF, and DOCX files.

Option B: From the command line

  From the unzipped project root, run:

    quarto render 1_Climate_sensitivity_Define_breaks_.qmd
    quarto render 2_Climate_Sensitivity_diff_panels.qmd
    quarto render final_curated_supplementary_information.qmd


6. WHERE TO FIND KEY RESULTS
----------------------------

Main processed dataset:

  output/Processed_data_breaks_flagged_YYYYMMDD.csv

This is the key bridge between the structural-break analysis and the first-difference panel analysis. It contains yield, GDD, precipitation, break-year, and before/after regime indicators.

Structural-break diagnostics:

  output/Structural_Change_Appendices_FINAL.xlsx

Important tabs include:

  - A_Model_Selection
  - B_Break_Drivers
  - C_Phase_Performance
  - D_FStat_Mountains
  - E_BIC_Elbow_Paths

First-difference panel results:

  output/FD_Pool_coeffs_YYYYMMDD.csv
  output/FD_Pool_AMEs_YYYYMMDD.csv

user-facing supplementary information:

  final_curated_supplementary_information.docx

Selected figures:

  output/France_StructuralBreak_Diagnostics.png
  output/France_StructuralBreak_Diagnostics.pdf
  output/Figure_Breaks_Nature.png
  output/Figure_Breaks_Nature.pdf

Supplementary CSV tables:

  output/Supplementary/Table_S1_Break_Summary.csv
  output/Supplementary/Table_S2_Regime_Levels_AMEs.csv
  output/Supplementary/Table_S3_Crop_Level_Summary.csv
  output/Supplementary/Table_S4_Heat_Penalty_Direction.csv


7. INTERPRETING THE MAIN FILES
------------------------------

`Processed_data_breaks_flagged_YYYYMMDD.csv`

  Main processed panel dataset. Important variables include:

    ccc                 country or region code
    ppp                 crop code
    yyy                 year
    YLD                 yield
    Total_GDD           seasonal growing degree days
    Tot_Precip          seasonal precipitation total
    Delta_Yield         first difference of yield
    Delta_GDD           first difference of GDD
    Delta_Precip        first difference of precipitation
    break_year          dominant validated break year; no-break panels are assigned 2022 by convention
    flag_before_after   0 for pre-break observations and 1 for post-break observations;
                        no-break panels are assigned 1 by convention
    has_break           TRUE/FALSE indicator for validated breaks, where available

`Structural_Change_Appendices_FINAL.xlsx`

  Contains the full structural-break evidence used to support the selected break years, including F-statistic paths, BIC paths, break-driver coefficients, and segment summaries.

`FD_Pool_AMEs_YYYYMMDD.csv`

  Contains the regime-specific average marginal effects from the first-difference panel models. Effects are reported by crop, climate term, and regime.


8. REPRODUCIBILITY NOTES
------------------------

- The first script is computationally heavier than the other scripts because it runs structural-break procedures across many country-crop panels.

- The second and third scripts are downstream scripts. They can be run directly if the required processed outputs already exist in `output/`.

- The scripts select the latest dated output files where applicable. If multiple dated versions are present, the most recently modified matching file is used.

- For a clean rerun, the user may optionally move old dated outputs out of the `output/` folder before rendering. This is not required, but it avoids confusion if several versions of the same output are present.

- Some figure exports use PNG/PDF devices. If a graphics-device error occurs, the CSV outputs should still be usable. Installing `ragg` and ensuring that the R installation supports Cairo PDF usually resolves figure-device issues.

- The archive is intended to be self-contained. No online data download should be required for the main reproduction workflow, assuming all input files are present in `data/`.


9. CROP CODES USED IN THE ANALYSIS
----------------------------------

  BA  Barley
  MA  Maize
  RP  Rapeseed / rape or colza seed
  SB  Soybeans
  SF  Sunflower seed
  WT  Wheat


10. CONTACT
-----------

For questions about the replication archive or interpretation of the scripts, please contact the corresponding author. (Not available during rewview process)
