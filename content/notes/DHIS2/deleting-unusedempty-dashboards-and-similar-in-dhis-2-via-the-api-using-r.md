+++
title = "Deleting unused/empty dashboards and similar in DHIS 2 via the API using R"
+++

The Data Administration app (Data Integrity section) in DHIS 2 can give you a list of dashboards which are empty or which have not been viewed within the last year. It is easy to delete those dashboards manually if you can access those dashboards as another user, but what if the dashboards have not been shared with other users?

We found that we had several hundred such dashboards, which may have been created by users during training or experimentation. 

First you need a list of all the URLs for unused/empty dashboards. Run the Data Integrity check so that the relevant dashboards are listed in the page as links. I then used a browser add-on to get all the links as URLs: [Link Gopher – Get this Extension for 🦊 Firefox (en-US)](https://addons.mozilla.org/en-US/firefox/addon/link-gopher/).

I then copied and pasted that list of URLs into an R script and reformatted it into an R vector. I then used this convenience R function to make the API calls (it assumes login details have been saved with `keyring`, and that you have the `httr` package installed): 

```r
urls <- c(
  "https://dhis2.example.com/dhis-web-dashboard/#/A4jBIftP0TS",
  "https://dhis2.example.com/dhis-web-dashboard/#/AUIhabtJtFK"
)

dhis2_delete <- function(url) {
  url2 <- paste('https://dhis2.example.com/api/dashboards/',
                basename(url),
                sep = '')
  message('DELETE: ', url2)
  response <- httr::DELETE(url = url2,
                           httr::authenticate(
                             keyring::key_get('dhis2_username'),
                             keyring::key_get('dhis2_password')
                           ))
  message('Status code: ', response$status_code)
}

for (i in urls) {
  Sys.sleep(1)
  dhis2_delete(i)
}
```
	
If the function returns 200 then the request was successful. If not then you will see other numbers. You can tweak the `Sys.sleep(1)` line to make the script pause more than one second between each call if required, as you can come up against rate limiting. 

You can easily amend the code above to remove other unused resources such as maps. 
