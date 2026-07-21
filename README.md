# RBS Program Monitoring

This repository contains an R Markdown workflow for pulling PPQ Form 280 records from `F280_MV` to support RBS program monitoring. The current report compares records before and after implementation dates for selected commodity, country, and location combinations.

## Repository Contents

- `f280_program_monitoring_report.Rmd`: R Markdown report that reads parameters, queries SQL Server, filters records, and writes report outputs.
- `params.csv`: Input file used to define commodity/country filters and evaluation periods.
- `README.md`: Usage notes for running and updating the report.

## Requirements

Before rendering the report, make sure you have:

- R with the packages `rmarkdown`, `DBI`, `odbc`, `knitr`, `openxlsx`, and `dplyr` installed.
- Access to the SQL Server database in the report connection string.
- A working ODBC SQL Server driver.
- Permission to query `[PPQ_AQI_ARMDMV2].[AQASMSG].[F280_MV]`.

The default connection string is defined in `f280_program_monitoring_report.Rmd` and points to `PPQ_AQI_ARMDMV2` on `AAP00VA3PPQSQL0\MSSQLSERVER,1433` using trusted authentication.

## How the Report Works

The report reads `params.csv`, builds a date and location query against `F280_MV`, then applies the commodity/country filters in R. It returns matching F280 records with location, origin, pathway, quantity, disposition, and a `Test Period` value of `before` or `after`.

The report creates:

- An HTML report from the R Markdown render.
- `f280_program_monitoring_report.xlsx` in the working directory, with sheets for report data, metadata, and report date summaries.

## Render the Report

From R, render the report with:

```r
rmarkdown::render(
  "f280_program_monitoring_report.Rmd",
  params = list(
    params_csv = "params.csv"
  ),
  output_file = "f280_program_monitoring_report.html"
)
```

If you run the command from outside the repository folder, use full paths:

```r
rmarkdown::render(
  "C:/path/to/rbs_program_monitoring/f280_program_monitoring_report.Rmd",
  params = list(
    params_csv = "C:/path/to/rbs_program_monitoring/params.csv"
  ),
  output_file = "C:/path/to/output/f280_program_monitoring_report.html"
)
```

You can also override the report title or connection string when rendering:

```r
rmarkdown::render(
  "f280_program_monitoring_report.Rmd",
  params = list(
    report_title = "Custom RBS Program Monitoring Report",
    params_csv = "params.csv",
    connection_string = "Driver=SQL Server;Server=SERVER_NAME;Database=DATABASE_NAME;trusted_connection=yes"
  )
)
```

## Update Parameters

Edit `params.csv` to change what the report pulls. The file must keep this header:

```csv
param_type,commodity,country,location_id,implementation_date,before_days,after_days
```

There are two parameter row types:

- `commodity_country`: Defines a commodity and origin country combination.
- `period`: Defines a location and before/after date window.

Leave unused columns blank for each row type.

### Commodity and Country Combinations

Use `param_type = commodity_country` rows to add commodity/country combinations. The report matches these values against `COMMODITY` and `ORIGIN_NM` from `F280_MV`. Matching is case-insensitive and ignores leading or trailing spaces.

Example:

```csv
param_type,commodity,country,location_id,implementation_date,before_days,after_days
commodity_country,GRAPE,chile,,,,
commodity_country,RAMBUTAN,ecuador,,,,
commodity_country,MANGO,peru,,,,
```

To add another combination, add another `commodity_country` row with the new commodity and country. Do not fill in `location_id`, `implementation_date`, `before_days`, or `after_days` on those rows.

### Periods and Location IDs

Use `param_type = period` rows to define implementation dates, before windows, after windows, and locations.

Example:

```csv
param_type,commodity,country,location_id,implementation_date,before_days,after_days
period,,,881,2024-12-18,365,365
period,,,all,2025-12-17,365,365
```

Each `period` row uses these fields:

- `location_id`: A numeric `LOCATION_ID` value, or `all`.
- `implementation_date`: The first date in the after period, formatted as `YYYY-MM-DD`.
- `before_days`: Number of days before the implementation date to include. Must be a positive integer.
- `after_days`: Number of days beginning on the implementation date to include. Must be a positive integer.

For a period row with `implementation_date = 2024-12-18`, `before_days = 365`, and `after_days = 365`, the report uses:

- Before period: `2023-12-19` through `2024-12-17`.
- After period: `2024-12-18` through `2025-12-17`.

Use a numeric `location_id` to pull one location. Use `all` to pull all other locations. When the parameter file includes both specific location IDs and an `all` row, the `all` row excludes the specifically listed location IDs so those locations are not double-counted.

Example with one specific location and a comparison group for all other locations:

```csv
param_type,commodity,country,location_id,implementation_date,before_days,after_days
commodity_country,GRAPE,chile,,,,
period,,,881,2024-12-18,365,365
period,,,all,2025-12-17,365,365
```

## Output Columns

The report output includes:

- `PPQ280 Record ID`
- `Commodity`
- `Commodity Type Name`
- `Disposition Code`
- `Disposition`
- `Create Date`
- `Report Date`
- `Location`
- `Location ID`
- `Location Site Name`
- `Origin`
- `Pathway`
- `Quantity`
- `Units of Measure`
- `Test Period`

## Troubleshooting

- If rendering fails because a package is missing, install the package named in the error message and render again.
- If `params_csv` is not found, confirm that the render command points to the correct `params.csv` path.
- If a `period` row fails validation, confirm that `location_id` is numeric or `all`, `implementation_date` uses `YYYY-MM-DD`, and `before_days` and `after_days` are positive integers.
- If the report returns zero rows, confirm that the commodity and country values match `COMMODITY` and `ORIGIN_NM` values in `F280_MV`, and that the requested locations and date windows contain records.
