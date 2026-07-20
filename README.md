# rbs_program_monitoring
RBS program monitoring. Supporting data pulls for original RBS approach. Build future capacity for analyses.

## F280 monitoring report

Render `/home/runner/work/rbs_program_monitoring/rbs_program_monitoring/f280_program_monitoring_report.Rmd` as HTML with:

```r
rmarkdown::render(
  "/home/runner/work/rbs_program_monitoring/rbs_program_monitoring/f280_program_monitoring_report.Rmd",
  params = list(
    commodity_country_filters = data.frame(
      commodity = c("grapes", "rambutan"),
      country = c("chile", "ecuador"),
      stringsAsFactors = FALSE
    ),
    period_filters = data.frame(
      location_id = c("101", "all"),
      implementation_date = as.Date(c("2024-07-01", "2025-01-01")),
      before_days = c(365, 365),
      after_days = c(365, 365),
      stringsAsFactors = FALSE
    )
  ),
  output_file = "/absolute/path/to/f280_program_monitoring_report.html"
)
```

The report:
- connects with the provided ODBC SQL Server connection string
- filters by commodity/country combinations using `COMMODITY` and `ORIGIN_NM`
- filters by before/after date windows relative to each `implementation_date`
- accepts numeric `LOCATION_ID` values or `all` in `period_filters$location_id`
- returns the requested columns in an HTML report table
