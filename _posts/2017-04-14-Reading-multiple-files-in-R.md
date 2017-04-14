---
layout: post
title: Reading multiple files in R
---

This is a quick post to read multiple files from directory into R for data manipulation.

**Aims**
1. To import hourly weather station data for January 2016 at three Singapore weather stations in R.
2. To combine temperature and humidity data into a single data.frame and export it dynamically as separate files


**Weather station data**

![Files in working directory](/images/20170414/files_directory.jpg)

* Data-files in the working directory `c:\test\` in the format of "YYYY_x1601.txt", where
  * YYYY refer to the three weather stations in Singapore (WSAP, WSSL and WSSS); and
  * x refer to temperature (t) or humidity (h) data


**Importing separate files into R**

```
setwd("c:/test") # set working directory where the files are kept
all <- list.files(pattern="\\.txt", full=T) # names of files in directory

all
[1] "./WSAP_h1601.txt" "./WSAP_t1601.txt" "./WSSL_h1601.txt" "./WSSL_t1601.txt" "./WSSS_h1601.txt"
[6] "./WSSS_t1601.txt"

# import all the data from each file into a list
df_list <- lapply(all, read.table, header=T, sep="\t")

# create names for each data.frame as WSAP_h1601, WSAP_t1601, etc.
df_list_name <- list.files(pattern="\\.txt")
df_list_name <- gsub(".txt", "", df_list_name) # remove ".txt" extension

names(df_list) <- df_list_name

```
This will allow the manipulation of data within the R environment.

**Combining data into a single data.frame**


* force rbind across all data.frame in df_list, even when all data have different headers

```
library(plyr)
library(dplyr)


df <- plyr::ldply(df_list)

head(df)
         .id                Time  Time_08 Humidity  Station TemperatureC
1 WSAP_h1601 2016-01-01 05:00:00  5:00 AM       94     WSAP           NA
2 WSAP_h1601 2016-01-01 06:00:00  6:00 AM       94     WSAP           NA
3 WSAP_h1601 2016-01-01 07:00:00  7:00 AM       94     WSAP           NA
4 WSAP_h1601 2016-01-01 08:00:00  8:00 AM       89     WSAP           NA
5 WSAP_h1601 2016-01-01 09:00:00  9:00 AM       89     WSAP           NA
6 WSAP_h1601 2016-01-01 10:00:00 10:00 AM       89     WSAP           NA
```

* write a small function to accept `NA` for sum

```
sums <- function(x) sum(x, na.rm=T)
```

* create summarise data based on `Station`, `Time` and `Time_08`

```
df <- df %>%
      group_by(Station, Time, Time_08) %>%
      select(-.id) %>% # remove ".id" column
      summarise_each(funs(sums))

head(df)
Source: local data frame [6 x 5]
Groups: Station, Time [6]

  Station                Time  Time_08 Humidity TemperatureC
   <fctr>              <fctr>   <fctr>    <int>        <int>
1    WSAP 2016-01-01 05:00:00  5:00 AM       94           26
2    WSAP 2016-01-01 06:00:00  6:00 AM       94           26
3    WSAP 2016-01-01 07:00:00  7:00 AM       94           26
4    WSAP 2016-01-01 08:00:00  8:00 AM       89           27
5    WSAP 2016-01-01 09:00:00  9:00 AM       89           28
6    WSAP 2016-01-01 10:00:00 10:00 AM       89           27

```

* write data.frame for each station within df as separate files

```
for(i in levels(df$Station)){
  output <- df %>%
            filter(Station==i)
  write.table(output, paste(i, "Jan2016.txt", sep="_"), row.names=F, sep="\t")
}
```
