# World Map Distorted by Fertility Rate (R Shiny)

An R Shiny recreation of the DASIL (Grinnell College Data Analysis & Social Inquiry Lab)
"World Map Distorted by Fertility Rate". Each country is resized so its area is proportional to
**land area x total fertility rate** (births per woman), so high-fertility countries swell and
low-fertility countries shrink.

In the app, choose a year (1950-2026) and a type of distortion, then press **Update map**. The map
only redraws when you press the button, because drawing a map is slow. Clicking a country, or choosing
it in the menu, outlines it and shows its fertility rate and its history without redrawing the map.

The app has two tabs: **About** (what the map shows and the data source) and **Map** (the app itself,
which is the tab that opens first).

## Folder structure

```
World-Map-Fertility-Rate/
├── data/                              Raw UN CSV
├── implementation/
│   └── Implementation.Rmd             Step 1: cleans the data and builds the distorted maps
├── UI_run/
│   └── UI_run.Rmd                     Step 2: the Shiny app
└── output-data/                       Files written by Step 1, read by Step 2
```

### data/

`unpopulation_dataportal_*.csv`: UN World Population Prospects 2024, total fertility rate
(both sexes, median variant), one value per country and year, 1950-2026. It is the only
indicator in the file. The file labels every value from 2024 on, and some earlier ones, as
"Projection". The app shows that label.

### output-data/

Created every time `Implementation.Rmd` runs:

| File | Used for |
| --- | --- |
| `cartograms.rds` | The distorted country shapes for every year (77) and every type of distortion (3) |
| `tfr_clean.csv` | Cleaned fertility rate by country and year (details panel and history chart) |

These two files (about 23 MB) are stored in the repository on purpose, so the app runs right after
you download the project, without the 12-minute build. If you re-run Step 1, they are overwritten and
should be committed again.

## Instructions

1. Keep the folders as they are.
2. Install the R packages: `shiny`, `dplyr`, `readr`, `sf`, `plotly`, `rnaturalearth`,
   `rnaturalearthdata`, `cartogram`, `rmarkdown`, `knitr`.
3. **Step 1, only if `output-data/` is empty or you changed the data or `Implementation.Rmd`:**
   open **implementation/Implementation.Rmd** in RStudio and knit it. It reads `data/` and writes
   into `output-data/`. It takes about 12 minutes, because the contiguous maps are slow and are built
   for every year, using several cores. To reuse an existing `cartograms.rds` instead of rebuilding it,
   set `rebuild <- FALSE` in the `cartogram-run` chunk.
4. **Step 2:** open **UI_run/UI_run.Rmd** and click **Run Document** to start the app.

   If RStudio says `Bad CPU type in executable` (its own pandoc is an Intel build and your Mac is Apple
   silicon), run `Sys.setenv(RSTUDIO_PANDOC = "/opt/anaconda3/bin")` in the Console first, or point it at
   any working arm64 pandoc.

RStudio runs each `.Rmd` file from the folder it is in, so you do not need to set a working directory.
If you run the code from the console instead, set the working directory to `implementation` or to
`UI_run`.

If `UI_run.Rmd` says the cleaned files are missing, run Step 1.

## Notes

- **Three types of distortion:** contiguous (countries stay connected),
  non-contiguous (countries keep their shape and resize in place) and Dorling circles. The contiguous
  maps use a low iteration setting (`itermax = 8`) so that they build in minutes, so their shapes are a
  little rougher than a slower run would give. In the non-contiguous map, a few shapes (Russia, Kiribati)
  are bigger than the map window in some years and are cut at its edge.
- **No data:** territories on the map that are not in the UN file (the Aland Islands, the Vatican and a
  few other small ones) are shown in yellow as "No data". They are sized with the median fertility rate
  of the year so the map stays connected. Disputed areas without an ISO code on the map (Somaliland,
  Northern Cyprus) are left out, and small places that are in the UN file but not on the map
  (Gibraltar, Guadeloupe, Martinique, Mayotte, Reunion, Tokelau) are not drawn.
- **Date line:** a few countries have islands on both sides of the date line (Fiji, Kiribati,
  New Zealand, Russia, the USA). On a flat map those islands are at opposite edges, so only the islands
  on the same side as the country's biggest piece are kept. A few far-away islands, such as the Chatham
  Islands and the tip of Chukotka, are therefore missing from the map.
