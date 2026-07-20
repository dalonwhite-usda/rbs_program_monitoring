# rbs_program_monitoring
RBS program monitoring. Supporting data pulls for original RBS approach. Build future capacity for analyses.

## F280 monitoring report

Set the input filters in `/home/runner/work/rbs_program_monitoring/rbs_program_monitoring/params.csv`:

```csv
param_type,commodity,country,location_id,implementation_date,before_days,after_days
commodity_country,grapes,chile,,,,
commodity_country,rambutan,ecuador,,,,
period,,,101,2024-07-01,365,365
period,,,all,2025-01-01,365,365
```

Then render `/home/runner/work/rbs_program_monitoring/rbs_program_monitoring/f280_program_monitoring_report.Rmd` as HTML with:

```r
rmarkdown::render(
  "/home/runner/work/rbs_program_monitoring/rbs_program_monitoring/f280_program_monitoring_report.Rmd",
  params = list(
    params_csv = "/home/runner/work/rbs_program_monitoring/rbs_program_monitoring/params.csv"
  ),
  output_file = "/absolute/path/to/f280_program_monitoring_report.html"
)
```

The report:
- connects with the provided ODBC SQL Server connection string
- reads commodity/country and period inputs from `params.csv`
- uses `param_type = commodity_country` rows for `COMMODITY` and `ORIGIN_NM` filters
- uses `param_type = period` rows for before/after date windows relative to each `implementation_date`
- accepts numeric `LOCATION_ID` values or `all` in `params.csv`
- returns the requested columns in an HTML report table
