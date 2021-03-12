---
title: "Identifying EINs in efiler data"
author: "Francisco Santamarina"
date: "March 11, 2021"
output:
  html_document:
    keep_md: true
    df_print: paged
    theme: readable
    highlight: tango
    toc: yes
    toc_float: yes
    code_fold: show
---

## Previous Code

Previously, I used the Nonprofit Open Data Collective's R script to create an index file. (*Note that it can take some time to download and build the functions in your local environment.*)





```r
# Source: https://nonprofit-open-data-collective.github.io/overview/

source( "https://raw.githubusercontent.com/Nonprofit-Open-Data-Collective/irs-990-efiler-database/master/BUILD_SCRIPTS/build_efile_database_functions.R" )

d <- buildIndex()
```

```
## Loading required package: R.oo
```

```
## Loading required package: R.methodsS3
```

```
## R.methodsS3 v1.8.1 (2020-08-26 16:20:06 UTC) successfully loaded. See ?R.methodsS3 for help.
```

```
## R.oo v1.24.0 (2020-08-26 16:11:58 UTC) successfully loaded. See ?R.oo for help.
```

```
## 
## Attaching package: 'R.oo'
```

```
## The following object is masked from 'package:R.methodsS3':
## 
##     throw
```

```
## The following objects are masked from 'package:methods':
## 
##     getClasses, getMethods
```

```
## The following objects are masked from 'package:base':
## 
##     attach, detach, load, save
```

```
## R.utils v2.10.1 (2020-08-26 22:50:31 UTC) successfully loaded. See ?R.utils for help.
```

```
## 
## Attaching package: 'R.utils'
```

```
## The following object is masked from 'package:jsonlite':
## 
##     validate
```

```
## The following object is masked from 'package:utils':
## 
##     timestamp
```

```
## The following objects are masked from 'package:base':
## 
##     cat, commandArgs, getOption, inherits, isOpen, nullfile, parse,
##     warnings
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
names( d )
```

```
##  [1] "EIN"              "TaxPeriod"        "DLN"              "FormType"        
##  [5] "URL"              "OrganizationName" "SubmittedOn"      "ObjectId"        
##  [9] "LastUpdated"      "TaxYear"
```

```r
#Subset the dataframe 
d.pf <- d[ d$FormType == "990PF", ]
```

Object `d.pf` contains information on all returns from 2009 through 2017 with e-filed information. The code below confirms this:


```r
sort( unique( d.pf$TaxYear ) )
```

```
## [1] 2009 2010 2011 2012 2013 2014 2015 2016 2017
```

The columns contained in or returned by the object `d.pf` include, in alphabetical order:


```r
sort( names( d.pf ) )
```

```
##  [1] "DLN"              "EIN"              "FormType"         "LastUpdated"     
##  [5] "ObjectId"         "OrganizationName" "SubmittedOn"      "TaxPeriod"       
##  [9] "TaxYear"          "URL"
```

what this looks like for the first observation in the object is:


```r
d.pf[ 1 , ]
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":[""],"name":["_rn_"],"type":[""],"align":["left"]},{"label":["EIN"],"name":[1],"type":["chr"],"align":["left"]},{"label":["TaxPeriod"],"name":[2],"type":["chr"],"align":["left"]},{"label":["DLN"],"name":[3],"type":["chr"],"align":["left"]},{"label":["FormType"],"name":[4],"type":["chr"],"align":["left"]},{"label":["URL"],"name":[5],"type":["chr"],"align":["left"]},{"label":["OrganizationName"],"name":[6],"type":["chr"],"align":["left"]},{"label":["SubmittedOn"],"name":[7],"type":["chr"],"align":["left"]},{"label":["ObjectId"],"name":[8],"type":["chr"],"align":["left"]},{"label":["LastUpdated"],"name":[9],"type":["chr"],"align":["left"]},{"label":["TaxYear"],"name":[10],"type":["dbl"],"align":["right"]}],"data":[{"1":"043338422","2":"201106","3":"93491316000001","4":"990PF","5":"https://s3.amazonaws.com/irs-form-990/201103169349100000_public.xml","6":"SUSAN DANIELS FAMILY CHARITABLE FOUNDATION","7":"2011-12-01","8":"201103169349100000","9":"2016-03-21T17:23:53","10":"2010","_rn_":"554"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

The provided URL should link to the XML file containing the 990-PF efiled by the Susan Daniels Family Charitable Foundation in 2011, for the 2010 tax year. 

## Making my own list of filings

while searching for a way to identify more recent efilings, I stumbled upon [this article by David Borenstein](https://appliednonprofitresearch.com/posts/2020/06/skip-the-irs-990-efile-indices/), who I worked with on some of the initial concordance/validation efforts back in 2018. In the linked article, David notes that there are various issues with the .JSON and .CSV files provided by the IRS used by scholars to create these index files, then provides python script to produce a similar index. 

To create your own version of all efiled documents currently hosted by AWs ([readme here](https://docs.opendata.aws/irs-990/readme.html), [about here](https://registry.opendata.aws/irs990/)), you can use the code below! 

*Note:* The first time you run the code below, it will take some time to run and install everything, ex. Miniconda, an open-source distribution of Python that comes pre-loaded with useful libraries. I also found that I needed to install some of the libraries to my machine before running them successfully, ex. [boto3](https://stackoverflow.com/questions/64075844/modulenotfounderror-no-module-named-boto3-using-anaconda).

As I already have Python installed on my computer, I want to force RStudio to look there (and pull in installed libraries from there). Guidance on how to do this is provided by [Xie's article on knitr options](https://yihui.org/knitr/options/) and various pages on the [reticulate library's guidance pages](https://rstudio.github.io/reticulate/). 


First, ensure that the Python library `boto3` is appropriately installed in the environment that RStudio will be using:


```r
library(reticulate)
py_install("boto3")
```

Second, run David's Python script. **WARNING:** This can take a very long time to run and be RAM intensive, so be warned. 


```python
import time
import datetime
from collections import deque
from typing import List, Deque, Iterable, Dict
import logging
import boto3
from concurrent.futures import ProcessPoolExecutor, ThreadPoolExecutor, as_completed, Future

logging.basicConfig(format='%(asctime)s : %(levelname)s : %(message)s', level=logging.INFO)

BUCKET: str = "irs-form-990"
EARLIEST_YEAR: int = 2009
cur_year: int = datetime.datetime.now().year

first_prefix: int = EARLIEST_YEAR * 100
last_prefix: int = (cur_year + 1) * 100

def get_keys_for_prefix(prefix: str) -> Iterable[str]:  
  """Return a collection of all key names starting with the specified prefix."""
  client = boto3.client('s3')
    
  # See https://boto3.amazonaws.com/v1/documentation/api/latest/guide/paginators.html
  paginator = client.get_paginator('list_objects_v2')
  page_iterator = paginator.paginate(Bucket=BUCKET, Prefix=prefix)

	# A deque is a collection with O(1) appends and O(n) iteration
  results: Deque[str] = deque()
  i = 0
  for i, page in enumerate(page_iterator):
      if "Contents" not in page:
          continue
        
      # You could also capture, e.g., the timestamp or checksum here
      page_keys: Iterable = (element["Key"] for element in page["Contents"])
      results.extend(page_keys)
  logging.info("Scanned {} page(s) with prefix {}.".format(i+1, prefix))
  return results

start: float = time.time()

# ProcessPoolExecutor starts a completely separate copy of Python for each worker
with ProcessPoolExecutor() as executor:
    futures: Deque[Future] = deque()
    for prefix in range(first_prefix, last_prefix):
        future: Future = executor.submit(get_keys_for_prefix, str(prefix))
        futures.append(future)
    
n = 0

# as_completed ignores submission order to prevent unnecessary waiting
for future in as_completed(futures):
    keys: Iterable = future.result()
    for key in keys:
    		# Do your analysis here
        n += 1
elapsed: float = time.time() - start
logging.info("Discovered {:,} keys in {:,.1f} seconds.".format(n, elapsed))
```

## If it ain't broke...

Why reinvent the wheel? If we look at the raw code of the [R script to build the efile database functions](https://raw.githubusercontent.com/Nonprofit-Open-Data-Collective/irs-990-efiler-database/master/BUILD_SCRIPTS/build_efile_database_functions.R), we can see that the function we used earlier, `buildIndex()`, has an argument in which we can define the file years. By playing around with the link of the AWS-hosted IRS efiler index files, we can tweak the web address to determine which are the latest years available. The address formula consists of 

"https://s3.amazonaws.com/irs-form-990/index_" + a year + ".csv"

If we use the example year of 2016, the address would thus be:

https://s3.amazonaws.com/irs-form-990/index_2016.csv

The `buildIndex()` function actually leverages this same consistency to build the files. I want to capture any efilings that may have been made and registered by the IRS as of March 10th, 2021, so I will create a range of 2017 to 2021 for my new dataframe.


```r
dat <- buildIndex( file.years = 2017:2021 )
names( dat )
```

```
##  [1] "EIN"              "TaxPeriod"        "DLN"              "FormType"        
##  [5] "URL"              "OrganizationName" "SubmittedOn"      "ObjectId"        
##  [9] "LastUpdated"      "TaxYear"
```

```r
#Subset the dataframe 
dat.pf <- dat[ dat$FormType == "990PF", ]
```

Object `dat.pf` contains information on all returns filed from 2017 through 2021 with e-filed information. The code below confirms this:


```r
sort( unique( dat.pf$TaxYear ) )
```

```
## [1] 2013 2014 2015 2016 2017 2018 2019
```

Now, this code seems a bit confusing; we can see that there are some private foundations (since we subset the data to just 990-PFs) that are filing records from as far back as 2013 during the period in question. While we could investigate further (and, indeed, this is precisely why we could/should use David's approach to generating our own list of efiler web addresses), that's beyond the scope of this effort. 

### Filtering Down

We have a list of EINs we want to search again, so let's import that list. We also want to ensure that values are imported as characters, so that we do not have type mismatches later on.


```r
searchValues <- read.csv2( "wa_found_list.csv", colClasses = "character" )
class( searchValues)
```

```
## [1] "data.frame"
```

Let's drop all 990-PFs that do not have EINs in our list.


```r
#Because searchValues is a dataframe, we need to be sure to reference the appropriate column in the next step.
#Otherwise, it will return with 0 matches
dat.pf <- dat.pf[dat.pf$EIN %in% searchValues[ , 1 ], ]

#Alternatively, we could the package 'dplyr' to achieve a similar outcome: 
# library( dplyr )
# filter(dat.pf, EIN %in% searchValues[ , 1 ] )
```

We can now generate a file sorted first on EIN and second by tax year.


```r
library( dplyr )

dat.pf <- arrange( .data = dat.pf, EIN, TaxYear ) 
```

### Stats on Filtered

Let's get some quick statistics on our new dataset.


```r
EINcounts <- dat.pf %>% count( EIN )

paste0( "Number of observations: ", nrow( dat.pf ))
```

```
## [1] "Number of observations: 3067"
```

```r
paste0( "Number of unique EINs: ", length( unique ( dat.pf$EIN  ) ) )
```

```
## [1] "Number of unique EINs: 1013"
```

```r
paste0( "Fewest number of efilings: ", min(EINcounts[ , 2 ] ) )
```

```
## [1] "Fewest number of efilings: 1"
```

```r
paste0( "Number of EINs with that number of efilings: ", count( EINcounts[ EINcounts$n == min(EINcounts[ , 2 ] ) , ]) )
```

```
## [1] "Number of EINs with that number of efilings: 92"
```

```r
paste0( "Most number of efilings: ", max(EINcounts[ , 2 ] ) )
```

```
## [1] "Most number of efilings: 6"
```

```r
paste0( "Number of EINs with that number of efilings: ", count( EINcounts[ EINcounts$n == max(EINcounts[ , 2 ] ) , ]) )
```

```
## [1] "Number of EINs with that number of efilings: 1"
```



Finally, let's save a copy of the output as a .csv file.


```r
write.csv( dat.pf, file = paste0("EINS of organizations that filed from 2017 to March 2021 ", format(Sys.time(), '%Y_%m_%d'),".csv"), quote = F, row.names = F)
```




