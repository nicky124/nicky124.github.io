---
layout: post
title: Cefas Data Hub APIs
#subtitle: Excerpt from Soulshaping by Jeff Brown
bigimg: /img/path.jpg
tags: [data, data.cefas.co.uk, API, r]
---

This post is about accessing data from the Cefas Data Hub. I'm by no means an expert in APIs, but I've figured out a fairly rudimentary ways to get data off it, which might help some people inside or outside of Cefas. Feel free to use and adapt this as you wish.

Firstly, a few things to note.

When you visit a holding on the data hub, the number at the end is referred to as the "Holding ID". I used to think a "holding" was a slightly awkward name - why not just call it a dataset? It's useful, though, because you can have multiple datasets associated with a single holding. 

http://data.cefas.co.uk/#/View/20141

The example holding here has 2 datasets associated with it: delay_med.csv and survey_data.csv. In MDR/Datahub speak these are referred to as "recordsets". So we have 2 recordsets (csv files) within one holding (page/result on the data hub).

So, how do we fetch and use this data in R? Well, if you only want to do it once, then you can use the blue "download as csv" button on the data hub. If you want to do this dynamically, then read on.

I preface this by saying that I'm not particularly familiar with any of the packages here, and it's likely there are better ways of doing all this, but I've been using it for years without issue. 

First, you need to find out the recordset ID. Here's a function I've written that you can use to return the IDs of all the recordsets associated with one holding. 

For now, I'd recommend keeping this lookup within your code. This is because the recordset ID can change if the data is updated or reloaded. The holding ID is static.

# Load data files from the Cefas Data Hub
# Holding ID is fixed but recordsetID may change so look these up each time.
getRecordSetID = function(HOLDINGId){
  url<- "https://cefasapp.cefas.co.uk/api/Recordsets"
  r=GET(url)
  x=httr::content(r, as = "text")
  y=fromJSON(x)
  Row=which(y$HoldingId==HOLDINGId)
  return(y[Row,"Id"])
}

Use the function as follows to fetch the first recordset:

```
recordNo=getRecordSetID(20141)[1]
url<- paste("https://cefasapp.cefas.co.uk/api/recordsets/",recordNo,"/data",sep="")
n=as.data.frame(fromJSON(httr::content(GET(url), as = "text"))$TotalItems)
url=paste("https://cefasapp.cefas.co.uk/api/recordsets/",recordNo,"/data?page=1&resultsPerPage=",n,sep="")
delay=as.data.frame(fromJSON(httr::content(GET(url), as = "text"))$Items)
```

recordNo=getRecordSetID(20141)[2]
url<- paste("https://cefasapp.cefas.co.uk/api/recordsets/",recordNo,"/data",sep="")
n=as.data.frame(fromJSON(httr::content(GET(url), as = "text"))$TotalItems)
url=paste("https://cefasapp.cefas.co.uk/api/recordsets/",recordNo,"/data?page=1&resultsPerPage=",n,sep="")
disease=as.data.frame(fromJSON(httr::content(GET(url), as = "text"))$Items)
