+++
title = "Creating indicators via the DHIS 2 API"
+++

Creating indicators manually in DHIS 2 is not difficult, but it is time-consuming if you have to do a lot. I needed to create health facility-level rates for over 30 data elements. The R code below takes a manual download of DHIS 2 data element metadata, filters them to those with a certain pattern in their names (in my case, "(new cases)" - not case sensitive) and creates a JSON file to be uploaded into DHIS 2 via the Data Import/Export app. 

You need these packages and this function:

```r
library(httr)
library(jsonlite)

create_indicators_upload <- function(name,
                             numerator = '#{xxxxxxxxxxx}',
                             numeratorDescription = 'Numerator',
                             denominator = '#{xxxxxxxxxxx}',
                             denominatorDescription = 'Denominator',
                             shortName = NULL,
                             annualized = FALSE,
                             decimals = 2,
                             indicatorType.id = 'xxxxxxxxxxx') {
  if (is.null(shortName))
    shortName <- substr(name, 1, 50)
  list(
    name = name,
    shortName = shortName,
    numerator = paste0('#{', numerator, '}', collapse = ''),
    numeratorDescription = numeratorDescription,
    denominator = denominator,
    denominatorDescription = denominatorDescription,
    annualized = annualized,
    decimals = decimals,
    indicatorType = list(id = indicatorType.id)
  )
}
```

All this does is create a named list with the required structure. Note that the numerator and denominator arguments need identifiers to be formatted as `#{<uid>}`. 

Then read in the data elements metadata download (I called mine "data_elements.json"), filter them by your preferred pattern and extract their names and identifiers.

```r
data_elements <- read_json('data_elements.json')$dataElements
new_cases <- Filter(\(x) grepl('(New Cases)', x$name, ignore.case = TRUE),
                    data_elements)
new_case_names <- sapply(new_cases, `[[`, 'name')
new_case_ids <- sapply(new_cases, `[[`, 'id')
```

Then use those to create a JSON file in the correct format for upload:

```r
indicators <-
  mapply(
    create_indicators_upload,
    new_case_names,
    new_case_ids,
    MoreArgs = list(denominator = '#{xxxxxxxxxxx}', indicatorType.id = 'xxxxxxxxxxx'),
    SIMPLIFY = FALSE,
    USE.NAMES = FALSE
  )
indicators <- list(indicators = indicators)
write_json(indicators,
           'created_indicators.json',
           auto_unbox = TRUE,
           pretty = FALSE)
```

You can use `browseURL('created_indicators.json')` to check the results. 

Do a dry run of importing them into DHIS 2 - if it all works then do the import proper.

