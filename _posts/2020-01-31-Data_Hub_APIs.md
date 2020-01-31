---
layout: post
title: Cefas Data Hub APIs
#subtitle: Excerpt from Soulshaping by Jeff Brown
#bigimg: /img/path.jpg
tags: [data, data.cefas.co.uk, API, r]
---

This post is about accessing data from the Cefas Data Hub. I'm by no means an expert in APIs, but I've been using this code for years without issue. Feel free to use and adapt this as you wish.

### Terminology

When you visit a holding on the data hub, the number at the end is referred to as the "Holding ID". I used to think a "holding" was a slightly awkward name - why not just call it a dataset? It's useful, though, because you can have multiple datasets associated with a single holding. 

This [example holding](http://data.cefas.co.uk/#/View/20141) has 2 datasets associated with it: delay_med.csv and survey_data.csv. In MDR/Datahub speak these are referred to as "recordsets". So we have 2 recordsets (csv files) within one holding (page/result on the data hub).

### Get the RecordsetIDs

So, how do we fetch and use this data in R? Well, if you only want to do it once, then you can use the blue "download as csv" button on the data hub. If you want to do this dynamically, then read on.

You'll need the :httr: and :jsonlite: packages installed, first. 

First, you need to find out the recordset ID. Here's a function I wrote which returns the IDs of all the recordsets associated with one holding. If you only need to use it once then you can use [my lookup tool](http://linlamp:3838/nm02/MDRLookup/) (currently only works within Cefas networks - sorry). It also lets you lookup the holding ID from the recordset ID. I've only ever really done this to check my answers, but it's there if you need it. 

```r
library(httr)
library(jsonlite)
getRecordSetID = function(HOLDINGId){
  url<- "https://cefasapp.cefas.co.uk/api/Recordsets"
  r=GET(url)
  x=httr::content(r, as = "text")
  y=fromJSON(x)
  Row=which(y$HoldingId==HOLDINGId)
  return(y[Row,"Id"])
}
```

Use the function as follows to fetch the first recordset:

```r
recordNo=getRecordSetID(20141)[1]
```

For now, I'd recommend keeping this lookup within your code, even if it seems a little redundant to look it up every time. This is because the recordset ID can change if the data is updated or reloaded. The holding ID is static.

### Actually download the data
Now that we know where the data is, we can grab it. I don't know why I can't just say "give me the whole file" rather than "how many rows are in the file? n? OK I'll have n rows, then", but I've managed fine without knowing.

```r
url<- paste("https://cefasapp.cefas.co.uk/api/recordsets/",recordNo,"/data",sep="")
n=as.data.frame(fromJSON(httr::content(GET(url), as = "text"))$TotalItems)
url=paste("https://cefasapp.cefas.co.uk/api/recordsets/",recordNo,"/data?page=1&resultsPerPage=",n,sep="")
delay=as.data.frame(fromJSON(httr::content(GET(url), as = "text"))$Items)
```

With that, you should now have a data frame called "delay" in your working environment. Please feel free to share or adapt this as you like, and let me know if you find it useful!
