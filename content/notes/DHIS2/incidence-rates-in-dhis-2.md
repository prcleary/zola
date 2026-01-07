+++
title = "Incidence rates in DHIS 2"
+++

Fair comparison of disease activity across areas typically requires calculation of rates. Here are some notes on uploading denominator data into DHIS 2 and creating indicators for incidence.

We were tasked with creating disease incidence rates using estimated health facility catchment populations as denominators. After many failed attempts at cleaning up some data files I had been sent, that had many spelling variations in the transliteration of names of health facilities, I decided to create a template and ask partners to complete it. 

I downloaded Organisation Unit and Organisation Unit Level from the Import/Export app (Metadata export tab) as uncompressed JSON. This can then be read into R and put into more manageable form with:

```r
library(data.table)
library(jsonlite)
extract_list_elements <- function(x) {
  list(
    name = x$name,
    shortName = x$shortName,
    parent_id = x$parent$id,
    path = x$path,
    id = x$id,
    level = x$level
  )
}
ou_json <- read_json('metadata.json')$organisationUnits
ou_list <- lapply(ou_json, extract_list_elements)
ou_data <- rbindlist(ou_list, fill = TRUE, use.names = TRUE)
```

You now have a list of all the organisation units in your system and need to filter to health facilities and show the hierarchy. In our system, organisation units can have the following levels: national (1), provincial (2), district (3), [tehsil](https://en.wikipedia.org/wiki/Tehsil) (4), [union council](https://en.wikipedia.org/wiki/Union_councils_of_Pakistan) (5) and health facility (6). 

I then divided the data by geographical level:

```r
hf_data <- ou_data[level %in% 6]
uc_data <- ou_data[level %in% 5]
tehsil_data <- ou_data[level %in% 4]
district_data <- ou_data[level %in% 3]
province_data <- ou_data[level %in% 2]
```

The next horrendous bit of code just joins these all up so that you can see the full hierarchy, and saves it as a CSV file:

```r
hf_data[, c(
  'ignore',
  'national_id',
  'province_id',
  'district_id',
  'tehsil_id',
  'unioncouncil_id',
  'healthfacility_id'
) := tstrsplit(path, '/')]
hf_data[, name_national := 'Pakistan']
hf_data2 <- merge(
  hf_data,
  province_data,
  by.x = 'province_id',
  by.y = 'id',
  all.x = TRUE,
  suffixes = c('', '_province')
)
hf_data3 <- merge(
  hf_data2,
  district_data,
  by.x = 'district_id',
  by.y = 'id',
  all.x = TRUE,
  suffixes = c('', '_district')
)
hf_data4 <- merge(
  hf_data3,
  tehsil_data,
  by.x = 'tehsil_id',
  by.y = 'id',
  all.x = TRUE,
  suffixes = c('', '_tehsil')
)
hf_data5 <- merge(
  hf_data4,
  uc_data,
  by.x = 'unioncouncil_id',
  by.y = 'id',
  all.x = TRUE,
  suffixes = c('', '_uc')
)
hf_data_final <- hf_data5[, .(
  name_national,
  name_province,
  shortName_province,
  name_district,
  shortName_district,
  name_tehsil,
  shortName_tehsil,
  name_uc,
  shortName_uc,
  name_hf = name,
  shortName_hf = shortName,
  hf_id = id
)]
fwrite(hf_data_final, 'dhis2_hf_details.csv')
```

I could then create a nicely formatted Excel spreadsheet out of this and send it out for completion. I added a column for catchment population denominator and asked for the other columns not to changed. After some months most of the data was in, so I could upload it into DHIS 2. 

In DHIS 2 I created an aggregate data element for the denominator values, of type "Number" and (very important) aggregation type of "Average". I took note of the ID of the data element created. 

I could then manually reformat the returned spreadsheet into the format required for import into DHIS 2. It needs to be in CSV format and to have the following structure (first two lines shown only):

| dataelement | period | orgunit     | categoryoptioncombo | attributeoptioncombo | value | storedby    | lastupdated | comment                 | followup | deleted |
|-------------|--------|-------------|---------------------|----------------------|-------|-------------|-------------|-------------------------|----------|---------|
| gEzZS4YRmT0 | 2025   | sgVCYsjiu5i | HllvX50cXC0         | HllvX50cXC0          | 2150  | Paul Cleary |             | Requires annual refresh |          |         |

`categoryoptioncombo` and `attributeoptioncombo` can be as shown, but add your own IDs and values for the rest. 

Using the Import/Export app (Data import tab) I imported this into DHIS 2 (select CSV; tick "First row is a header") after a successful dry run import. After running analytics this was available for analysis and I presented it in some charts in a dashboard for checking (some of the numbers were surprisingly large).

Before you can create an indicator for a rate, you need to create an indicator type - this is just a factor that the rate will be multipled with for presentation. I created an indicator type called "Per 100,000" with a factor of 100,000.

I created a rate indicator for acute watery diarrhoea for starters, specifying presentation with 2 decimal places, the "Per 100,000" indicator type and the numerator (acute watery diarrhoea cases) and denominator (catchment population). If you prefer, you can "annualise" the rate (which for a weekly rate just means multiplying it by 52). After running analytics again you can show your indicator in charts and dashboards. 

There are a couple of minor issues to mention. The denominator will need to be updated for 2026 - I have set a calendar reminder. Also, some of our health facilities appear more than once in the list, presumably because their catchment populations span multiple administrative areas, and there is also a risk of double counting (e.g. for hospitals covering the same population as health facilities), so this is really only for looking at rates at health facility level - you can't aggregate it up. 

I am now looking at using the API to create all the remaining rate indicators required. 
