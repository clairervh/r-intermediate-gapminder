---
layout: page
title: Intermediate R for reproducible scientific analysis
subtitle: data.table
minutes: 20
---


> ## Learning objectives {.objectives}
>
> * To know how to perform common data.frame tasks using data.table
> * To know how to examine the structure of objects in R
> * To be able to set keys for data.table
>

First, we will be learning about the `data.table` package. Data tables
have a number of advantages over data frames:

 * It provides a huge speed increase over data frames. Tasks that would
   normally take hours with a data frame take seconds with data tables.
 * It is much more concise to write and allows you to avoid complicated
   which statements and repeated subsetting.
 * Its key system allows you to merge/query multiple tables without 
   worrying about matching rownames.

### Reading in data

Let's load in the package and read in some data:


```r
library("data.table")
gap <- fread("data/gapminder-FiveYearData.csv")
gap
```

```
##           country year      pop continent lifeExp gdpPercap
##    1: Afghanistan 1952  8425333      Asia  28.801  779.4453
##    2: Afghanistan 1957  9240934      Asia  30.332  820.8530
##    3: Afghanistan 1962 10267083      Asia  31.997  853.1007
##    4: Afghanistan 1967 11537966      Asia  34.020  836.1971
##    5: Afghanistan 1972 13079460      Asia  36.088  739.9811
##   ---                                                      
## 1700:    Zimbabwe 1987  9216418    Africa  62.351  706.1573
## 1701:    Zimbabwe 1992 10704340    Africa  60.377  693.4208
## 1702:    Zimbabwe 1997 11404948    Africa  46.809  792.4500
## 1703:    Zimbabwe 2002 11926563    Africa  39.989  672.0386
## 1704:    Zimbabwe 2007 12311143    Africa  43.487  469.7093
```

We can see the data has loaded correctly using the `fread` function from the 
data.table package. Note that unlike data frames, R will print out the first and 
last 5 rows ofthe table.

`fread` works similary to `read.table`: it tries to make
sense of the data and read it in appropriately. It is much faster than 
`read.table` for large tables, but is slightly less sophisticated. You may find
yourself needing to use `read.table` or one of its derivative functions to load
in data correctly, then casting to a data table:


```r
# Note that as.data.table will throw out rownames unless you set 
#'keep.rownames = TRUE'
gap_df <- read.csv("data/gapminder-FiveYearData.csv")
gap_dt2 <- as.data.table(gap_df)
```

We can use the structure function (`str`) to examine what exactly a data.table 
is:


```r
str(gap)
```

```
## Classes 'data.table' and 'data.frame':	1704 obs. of  6 variables:
##  $ country  : chr  "Afghanistan" "Afghanistan" "Afghanistan" "Afghanistan" ...
##  $ year     : int  1952 1957 1962 1967 1972 1977 1982 1987 1992 1997 ...
##  $ pop      : num  8425333 9240934 10267083 11537966 13079460 ...
##  $ continent: chr  "Asia" "Asia" "Asia" "Asia" ...
##  $ lifeExp  : num  28.8 30.3 32 34 36.1 ...
##  $ gdpPercap: num  779 821 853 836 740 ...
##  - attr(*, ".internal.selfref")=<externalptr>
```

A data.table is simply a data.frame with an additional class attached to it, 
along with a special attribute, ".internal.selfref", which is an external pointer
that data.table uses to work with the data in memory in a lower level language.

Data tables are backwards compatible with most functions that require data 
frames, however you may occasionally find that you need to cast them using
`as.data.frame` on functions which check only the first class of the object.

To prove this to ourself, we can check the objets for equality:


```r
all.equal(gap, gap_df)
```

```
## Error in `:=`((i), .xi): Check that is.data.table(DT) == TRUE. Otherwise, := and `:=`(...) are defined for use in j, once only and in particular ways. See help(":=").
```

```r
all.equal(gap, gap_df, check.attributes = FALSE)
```

```
## Error in `:=`((i), .xi): Check that is.data.table(DT) == TRUE. Otherwise, := and `:=`(...) are defined for use in j, once only and in particular ways. See help(":=").
```

This shows us that the only the attributes of the object are different, but the
underlying data is the same.

### Basic operations

Nearly all operations on data tables are performed inside the `[` function, and
are performed in place in memory. Let's take a look at some operations and their
data frame equivalents:

To select or filter rows, we use the first argument of `[` just like with data
frames:


```r
gap[continent == "Oceania"]
```

```
##         country year      pop continent lifeExp gdpPercap
##  1:   Australia 1952  8691212   Oceania  69.120  10039.60
##  2:   Australia 1957  9712569   Oceania  70.330  10949.65
##  3:   Australia 1962 10794968   Oceania  70.930  12217.23
##  4:   Australia 1967 11872264   Oceania  71.100  14526.12
##  5:   Australia 1972 13177000   Oceania  71.930  16788.63
##  6:   Australia 1977 14074100   Oceania  73.490  18334.20
##  7:   Australia 1982 15184200   Oceania  74.740  19477.01
##  8:   Australia 1987 16257249   Oceania  76.320  21888.89
##  9:   Australia 1992 17481977   Oceania  77.560  23424.77
## 10:   Australia 1997 18565243   Oceania  78.830  26997.94
## 11:   Australia 2002 19546792   Oceania  80.370  30687.75
## 12:   Australia 2007 20434176   Oceania  81.235  34435.37
## 13: New Zealand 1952  1994794   Oceania  69.390  10556.58
## 14: New Zealand 1957  2229407   Oceania  70.260  12247.40
## 15: New Zealand 1962  2488550   Oceania  71.240  13175.68
## 16: New Zealand 1967  2728150   Oceania  71.520  14463.92
## 17: New Zealand 1972  2929100   Oceania  71.890  16046.04
## 18: New Zealand 1977  3164900   Oceania  72.220  16233.72
## 19: New Zealand 1982  3210650   Oceania  73.840  17632.41
## 20: New Zealand 1987  3317166   Oceania  74.320  19007.19
## 21: New Zealand 1992  3437674   Oceania  76.330  18363.32
## 22: New Zealand 1997  3676187   Oceania  77.550  21050.41
## 23: New Zealand 2002  3908037   Oceania  79.110  23189.80
## 24: New Zealand 2007  4115771   Oceania  80.204  25185.01
##         country year      pop continent lifeExp gdpPercap
```

```r
# data frame equivalent
gap_df[gap_df$continent == "Oceania",]
```

```
##          country year      pop continent lifeExp gdpPercap
## 61     Australia 1952  8691212   Oceania  69.120  10039.60
## 62     Australia 1957  9712569   Oceania  70.330  10949.65
## 63     Australia 1962 10794968   Oceania  70.930  12217.23
## 64     Australia 1967 11872264   Oceania  71.100  14526.12
## 65     Australia 1972 13177000   Oceania  71.930  16788.63
## 66     Australia 1977 14074100   Oceania  73.490  18334.20
## 67     Australia 1982 15184200   Oceania  74.740  19477.01
## 68     Australia 1987 16257249   Oceania  76.320  21888.89
## 69     Australia 1992 17481977   Oceania  77.560  23424.77
## 70     Australia 1997 18565243   Oceania  78.830  26997.94
## 71     Australia 2002 19546792   Oceania  80.370  30687.75
## 72     Australia 2007 20434176   Oceania  81.235  34435.37
## 1093 New Zealand 1952  1994794   Oceania  69.390  10556.58
## 1094 New Zealand 1957  2229407   Oceania  70.260  12247.40
## 1095 New Zealand 1962  2488550   Oceania  71.240  13175.68
## 1096 New Zealand 1967  2728150   Oceania  71.520  14463.92
## 1097 New Zealand 1972  2929100   Oceania  71.890  16046.04
## 1098 New Zealand 1977  3164900   Oceania  72.220  16233.72
## 1099 New Zealand 1982  3210650   Oceania  73.840  17632.41
## 1100 New Zealand 1987  3317166   Oceania  74.320  19007.19
## 1101 New Zealand 1992  3437674   Oceania  76.330  18363.32
## 1102 New Zealand 1997  3676187   Oceania  77.550  21050.41
## 1103 New Zealand 2002  3908037   Oceania  79.110  23189.80
## 1104 New Zealand 2007  4115771   Oceania  80.204  25185.01
```

The data table knows when we type `continent` to look for that as a column in
the table: removing the redundant text we'd need to type for data.frames. This 
becomes even more convenient when filtering on multiple columns:


```r
gap[continent == "Oceania" & country == "Australia" & year %in% c(1952, 2007)]
```

```
##      country year      pop continent lifeExp gdpPercap
## 1: Australia 1952  8691212   Oceania  69.120  10039.60
## 2: Australia 2007 20434176   Oceania  81.235  34435.37
```

```r
# data frame equivalent
gap_df[gap_df$continent == "Oceania" & gap_df$country == "Australia" & gap_df$year %in% c(1952, 2007),]
```

```
##      country year      pop continent lifeExp gdpPercap
## 61 Australia 1952  8691212   Oceania  69.120  10039.60
## 72 Australia 2007 20434176   Oceania  81.235  34435.37
```

Note that the data frame call will still work on the data table:


```r
gap[gap$continent == "Oceania" & gap$country == "Australia" & gap$year %in% c(1952, 2007),]
```

```
##      country year      pop continent lifeExp gdpPercap
## 1: Australia 1952  8691212   Oceania  69.120  10039.60
## 2: Australia 2007 20434176   Oceania  81.235  34435.37
```

To select columns we use the second argument to `[`, just like data.frames:


```r
gap[,continent]
```

```
##    [1] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
##    [7] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
##   [13] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
##   [19] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
##   [25] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##   [31] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##   [37] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##   [43] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##   [49] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
##   [55] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
##   [61] "Oceania"  "Oceania"  "Oceania"  "Oceania"  "Oceania"  "Oceania" 
##   [67] "Oceania"  "Oceania"  "Oceania"  "Oceania"  "Oceania"  "Oceania" 
##   [73] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
##   [79] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
##   [85] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
##   [91] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
##   [97] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
##  [103] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
##  [109] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
##  [115] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
##  [121] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [127] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [133] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
##  [139] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
##  [145] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
##  [151] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
##  [157] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [163] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [169] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
##  [175] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
##  [181] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
##  [187] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
##  [193] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [199] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [205] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [211] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [217] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
##  [223] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
##  [229] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [235] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [241] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
##  [247] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
##  [253] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [259] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [265] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [271] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [277] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
##  [283] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
##  [289] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
##  [295] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
##  [301] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
##  [307] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
##  [313] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [319] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [325] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [331] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [337] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [343] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [349] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
##  [355] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
##  [361] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [367] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [373] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
##  [379] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
##  [385] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
##  [391] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
##  [397] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
##  [403] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
##  [409] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
##  [415] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
##  [421] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [427] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [433] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
##  [439] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
##  [445] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
##  [451] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
##  [457] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [463] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [469] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
##  [475] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
##  [481] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [487] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [493] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [499] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [505] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [511] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [517] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
##  [523] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
##  [529] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
##  [535] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
##  [541] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [547] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [553] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [559] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [565] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
##  [571] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
##  [577] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [583] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [589] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
##  [595] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
##  [601] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
##  [607] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
##  [613] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [619] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [625] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [631] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [637] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
##  [643] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
##  [649] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
##  [655] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
##  [661] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
##  [667] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
##  [673] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
##  [679] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
##  [685] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
##  [691] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
##  [697] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
##  [703] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
##  [709] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
##  [715] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
##  [721] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
##  [727] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
##  [733] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
##  [739] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
##  [745] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
##  [751] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
##  [757] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
##  [763] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
##  [769] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
##  [775] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
##  [781] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
##  [787] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
##  [793] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
##  [799] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
##  [805] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
##  [811] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
##  [817] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [823] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [829] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
##  [835] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
##  [841] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
##  [847] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
##  [853] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
##  [859] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
##  [865] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
##  [871] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
##  [877] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [883] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [889] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [895] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [901] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [907] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [913] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [919] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [925] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [931] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [937] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
##  [943] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
##  [949] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [955] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [961] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [967] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [973] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [979] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
##  [985] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
##  [991] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
##  [997] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
## [1003] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
## [1009] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
## [1015] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
## [1021] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
## [1027] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
## [1033] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
## [1039] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
## [1045] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
## [1051] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
## [1057] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
## [1063] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
## [1069] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
## [1075] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
## [1081] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
## [1087] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
## [1093] "Oceania"  "Oceania"  "Oceania"  "Oceania"  "Oceania"  "Oceania" 
## [1099] "Oceania"  "Oceania"  "Oceania"  "Oceania"  "Oceania"  "Oceania" 
## [1105] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
## [1111] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
## [1117] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
## [1123] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
## [1129] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
## [1135] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
## [1141] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
## [1147] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
## [1153] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
## [1159] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
## [1165] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
## [1171] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
## [1177] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
## [1183] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
## [1189] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
## [1195] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
## [1201] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
## [1207] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
## [1213] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
## [1219] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
## [1225] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
## [1231] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
## [1237] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
## [1243] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
## [1249] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
## [1255] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
## [1261] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
## [1267] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
## [1273] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
## [1279] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
## [1285] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
## [1291] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
## [1297] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
## [1303] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
## [1309] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
## [1315] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
## [1321] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
## [1327] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
## [1333] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
## [1339] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
## [1345] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
## [1351] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
## [1357] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
## [1363] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
## [1369] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
## [1375] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
## [1381] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
## [1387] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
## [1393] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
## [1399] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
## [1405] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
## [1411] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
## [1417] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
## [1423] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
## [1429] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
## [1435] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
## [1441] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
## [1447] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
## [1453] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
## [1459] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
## [1465] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
## [1471] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
## [1477] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
## [1483] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
## [1489] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
## [1495] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
## [1501] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
## [1507] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
## [1513] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
## [1519] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
## [1525] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
## [1531] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
## [1537] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
## [1543] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
## [1549] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
## [1555] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
## [1561] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
## [1567] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
## [1573] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
## [1579] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
## [1585] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
## [1591] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
## [1597] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
## [1603] "Europe"   "Europe"   "Europe"   "Europe"   "Europe"   "Europe"  
## [1609] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
## [1615] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
## [1621] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
## [1627] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
## [1633] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
## [1639] "Americas" "Americas" "Americas" "Americas" "Americas" "Americas"
## [1645] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
## [1651] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
## [1657] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
## [1663] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
## [1669] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
## [1675] "Asia"     "Asia"     "Asia"     "Asia"     "Asia"     "Asia"    
## [1681] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
## [1687] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
## [1693] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"  
## [1699] "Africa"   "Africa"   "Africa"   "Africa"   "Africa"   "Africa"
```

```r
# data frame equivalent
gap_df[,"continent"]
```

```
##    [1] Asia     Asia     Asia     Asia     Asia     Asia     Asia    
##    [8] Asia     Asia     Asia     Asia     Asia     Europe   Europe  
##   [15] Europe   Europe   Europe   Europe   Europe   Europe   Europe  
##   [22] Europe   Europe   Europe   Africa   Africa   Africa   Africa  
##   [29] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
##   [36] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
##   [43] Africa   Africa   Africa   Africa   Africa   Africa   Americas
##   [50] Americas Americas Americas Americas Americas Americas Americas
##   [57] Americas Americas Americas Americas Oceania  Oceania  Oceania 
##   [64] Oceania  Oceania  Oceania  Oceania  Oceania  Oceania  Oceania 
##   [71] Oceania  Oceania  Europe   Europe   Europe   Europe   Europe  
##   [78] Europe   Europe   Europe   Europe   Europe   Europe   Europe  
##   [85] Asia     Asia     Asia     Asia     Asia     Asia     Asia    
##   [92] Asia     Asia     Asia     Asia     Asia     Asia     Asia    
##   [99] Asia     Asia     Asia     Asia     Asia     Asia     Asia    
##  [106] Asia     Asia     Asia     Europe   Europe   Europe   Europe  
##  [113] Europe   Europe   Europe   Europe   Europe   Europe   Europe  
##  [120] Europe   Africa   Africa   Africa   Africa   Africa   Africa  
##  [127] Africa   Africa   Africa   Africa   Africa   Africa   Americas
##  [134] Americas Americas Americas Americas Americas Americas Americas
##  [141] Americas Americas Americas Americas Europe   Europe   Europe  
##  [148] Europe   Europe   Europe   Europe   Europe   Europe   Europe  
##  [155] Europe   Europe   Africa   Africa   Africa   Africa   Africa  
##  [162] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
##  [169] Americas Americas Americas Americas Americas Americas Americas
##  [176] Americas Americas Americas Americas Americas Europe   Europe  
##  [183] Europe   Europe   Europe   Europe   Europe   Europe   Europe  
##  [190] Europe   Europe   Europe   Africa   Africa   Africa   Africa  
##  [197] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
##  [204] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
##  [211] Africa   Africa   Africa   Africa   Africa   Africa   Asia    
##  [218] Asia     Asia     Asia     Asia     Asia     Asia     Asia    
##  [225] Asia     Asia     Asia     Asia     Africa   Africa   Africa  
##  [232] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
##  [239] Africa   Africa   Americas Americas Americas Americas Americas
##  [246] Americas Americas Americas Americas Americas Americas Americas
##  [253] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
##  [260] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
##  [267] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
##  [274] Africa   Africa   Africa   Americas Americas Americas Americas
##  [281] Americas Americas Americas Americas Americas Americas Americas
##  [288] Americas Asia     Asia     Asia     Asia     Asia     Asia    
##  [295] Asia     Asia     Asia     Asia     Asia     Asia     Americas
##  [302] Americas Americas Americas Americas Americas Americas Americas
##  [309] Americas Americas Americas Americas Africa   Africa   Africa  
##  [316] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
##  [323] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
##  [330] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
##  [337] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
##  [344] Africa   Africa   Africa   Africa   Africa   Americas Americas
##  [351] Americas Americas Americas Americas Americas Americas Americas
##  [358] Americas Americas Americas Africa   Africa   Africa   Africa  
##  [365] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
##  [372] Africa   Europe   Europe   Europe   Europe   Europe   Europe  
##  [379] Europe   Europe   Europe   Europe   Europe   Europe   Americas
##  [386] Americas Americas Americas Americas Americas Americas Americas
##  [393] Americas Americas Americas Americas Europe   Europe   Europe  
##  [400] Europe   Europe   Europe   Europe   Europe   Europe   Europe  
##  [407] Europe   Europe   Europe   Europe   Europe   Europe   Europe  
##  [414] Europe   Europe   Europe   Europe   Europe   Europe   Europe  
##  [421] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
##  [428] Africa   Africa   Africa   Africa   Africa   Americas Americas
##  [435] Americas Americas Americas Americas Americas Americas Americas
##  [442] Americas Americas Americas Americas Americas Americas Americas
##  [449] Americas Americas Americas Americas Americas Americas Americas
##  [456] Americas Africa   Africa   Africa   Africa   Africa   Africa  
##  [463] Africa   Africa   Africa   Africa   Africa   Africa   Americas
##  [470] Americas Americas Americas Americas Americas Americas Americas
##  [477] Americas Americas Americas Americas Africa   Africa   Africa  
##  [484] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
##  [491] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
##  [498] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
##  [505] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
##  [512] Africa   Africa   Africa   Africa   Africa   Europe   Europe  
##  [519] Europe   Europe   Europe   Europe   Europe   Europe   Europe  
##  [526] Europe   Europe   Europe   Europe   Europe   Europe   Europe  
##  [533] Europe   Europe   Europe   Europe   Europe   Europe   Europe  
##  [540] Europe   Africa   Africa   Africa   Africa   Africa   Africa  
##  [547] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
##  [554] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
##  [561] Africa   Africa   Africa   Africa   Europe   Europe   Europe  
##  [568] Europe   Europe   Europe   Europe   Europe   Europe   Europe  
##  [575] Europe   Europe   Africa   Africa   Africa   Africa   Africa  
##  [582] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
##  [589] Europe   Europe   Europe   Europe   Europe   Europe   Europe  
##  [596] Europe   Europe   Europe   Europe   Europe   Americas Americas
##  [603] Americas Americas Americas Americas Americas Americas Americas
##  [610] Americas Americas Americas Africa   Africa   Africa   Africa  
##  [617] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
##  [624] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
##  [631] Africa   Africa   Africa   Africa   Africa   Africa   Americas
##  [638] Americas Americas Americas Americas Americas Americas Americas
##  [645] Americas Americas Americas Americas Americas Americas Americas
##  [652] Americas Americas Americas Americas Americas Americas Americas
##  [659] Americas Americas Asia     Asia     Asia     Asia     Asia    
##  [666] Asia     Asia     Asia     Asia     Asia     Asia     Asia    
##  [673] Europe   Europe   Europe   Europe   Europe   Europe   Europe  
##  [680] Europe   Europe   Europe   Europe   Europe   Europe   Europe  
##  [687] Europe   Europe   Europe   Europe   Europe   Europe   Europe  
##  [694] Europe   Europe   Europe   Asia     Asia     Asia     Asia    
##  [701] Asia     Asia     Asia     Asia     Asia     Asia     Asia    
##  [708] Asia     Asia     Asia     Asia     Asia     Asia     Asia    
##  [715] Asia     Asia     Asia     Asia     Asia     Asia     Asia    
##  [722] Asia     Asia     Asia     Asia     Asia     Asia     Asia    
##  [729] Asia     Asia     Asia     Asia     Asia     Asia     Asia    
##  [736] Asia     Asia     Asia     Asia     Asia     Asia     Asia    
##  [743] Asia     Asia     Europe   Europe   Europe   Europe   Europe  
##  [750] Europe   Europe   Europe   Europe   Europe   Europe   Europe  
##  [757] Asia     Asia     Asia     Asia     Asia     Asia     Asia    
##  [764] Asia     Asia     Asia     Asia     Asia     Europe   Europe  
##  [771] Europe   Europe   Europe   Europe   Europe   Europe   Europe  
##  [778] Europe   Europe   Europe   Americas Americas Americas Americas
##  [785] Americas Americas Americas Americas Americas Americas Americas
##  [792] Americas Asia     Asia     Asia     Asia     Asia     Asia    
##  [799] Asia     Asia     Asia     Asia     Asia     Asia     Asia    
##  [806] Asia     Asia     Asia     Asia     Asia     Asia     Asia    
##  [813] Asia     Asia     Asia     Asia     Africa   Africa   Africa  
##  [820] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
##  [827] Africa   Africa   Asia     Asia     Asia     Asia     Asia    
##  [834] Asia     Asia     Asia     Asia     Asia     Asia     Asia    
##  [841] Asia     Asia     Asia     Asia     Asia     Asia     Asia    
##  [848] Asia     Asia     Asia     Asia     Asia     Asia     Asia    
##  [855] Asia     Asia     Asia     Asia     Asia     Asia     Asia    
##  [862] Asia     Asia     Asia     Asia     Asia     Asia     Asia    
##  [869] Asia     Asia     Asia     Asia     Asia     Asia     Asia    
##  [876] Asia     Africa   Africa   Africa   Africa   Africa   Africa  
##  [883] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
##  [890] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
##  [897] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
##  [904] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
##  [911] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
##  [918] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
##  [925] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
##  [932] Africa   Africa   Africa   Africa   Africa   Asia     Asia    
##  [939] Asia     Asia     Asia     Asia     Asia     Asia     Asia    
##  [946] Asia     Asia     Asia     Africa   Africa   Africa   Africa  
##  [953] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
##  [960] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
##  [967] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
##  [974] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
##  [981] Africa   Africa   Africa   Africa   Americas Americas Americas
##  [988] Americas Americas Americas Americas Americas Americas Americas
##  [995] Americas Americas Asia     Asia     Asia     Asia     Asia    
## [1002] Asia     Asia     Asia     Asia     Asia     Asia     Asia    
## [1009] Europe   Europe   Europe   Europe   Europe   Europe   Europe  
## [1016] Europe   Europe   Europe   Europe   Europe   Africa   Africa  
## [1023] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
## [1030] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
## [1037] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
## [1044] Africa   Asia     Asia     Asia     Asia     Asia     Asia    
## [1051] Asia     Asia     Asia     Asia     Asia     Asia     Africa  
## [1058] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
## [1065] Africa   Africa   Africa   Africa   Asia     Asia     Asia    
## [1072] Asia     Asia     Asia     Asia     Asia     Asia     Asia    
## [1079] Asia     Asia     Europe   Europe   Europe   Europe   Europe  
## [1086] Europe   Europe   Europe   Europe   Europe   Europe   Europe  
## [1093] Oceania  Oceania  Oceania  Oceania  Oceania  Oceania  Oceania 
## [1100] Oceania  Oceania  Oceania  Oceania  Oceania  Americas Americas
## [1107] Americas Americas Americas Americas Americas Americas Americas
## [1114] Americas Americas Americas Africa   Africa   Africa   Africa  
## [1121] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
## [1128] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
## [1135] Africa   Africa   Africa   Africa   Africa   Africa   Europe  
## [1142] Europe   Europe   Europe   Europe   Europe   Europe   Europe  
## [1149] Europe   Europe   Europe   Europe   Asia     Asia     Asia    
## [1156] Asia     Asia     Asia     Asia     Asia     Asia     Asia    
## [1163] Asia     Asia     Asia     Asia     Asia     Asia     Asia    
## [1170] Asia     Asia     Asia     Asia     Asia     Asia     Asia    
## [1177] Americas Americas Americas Americas Americas Americas Americas
## [1184] Americas Americas Americas Americas Americas Americas Americas
## [1191] Americas Americas Americas Americas Americas Americas Americas
## [1198] Americas Americas Americas Americas Americas Americas Americas
## [1205] Americas Americas Americas Americas Americas Americas Americas
## [1212] Americas Asia     Asia     Asia     Asia     Asia     Asia    
## [1219] Asia     Asia     Asia     Asia     Asia     Asia     Europe  
## [1226] Europe   Europe   Europe   Europe   Europe   Europe   Europe  
## [1233] Europe   Europe   Europe   Europe   Europe   Europe   Europe  
## [1240] Europe   Europe   Europe   Europe   Europe   Europe   Europe  
## [1247] Europe   Europe   Americas Americas Americas Americas Americas
## [1254] Americas Americas Americas Americas Americas Americas Americas
## [1261] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
## [1268] Africa   Africa   Africa   Africa   Africa   Europe   Europe  
## [1275] Europe   Europe   Europe   Europe   Europe   Europe   Europe  
## [1282] Europe   Europe   Europe   Africa   Africa   Africa   Africa  
## [1289] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
## [1296] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
## [1303] Africa   Africa   Africa   Africa   Africa   Africa   Asia    
## [1310] Asia     Asia     Asia     Asia     Asia     Asia     Asia    
## [1317] Asia     Asia     Asia     Asia     Africa   Africa   Africa  
## [1324] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
## [1331] Africa   Africa   Europe   Europe   Europe   Europe   Europe  
## [1338] Europe   Europe   Europe   Europe   Europe   Europe   Europe  
## [1345] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
## [1352] Africa   Africa   Africa   Africa   Africa   Asia     Asia    
## [1359] Asia     Asia     Asia     Asia     Asia     Asia     Asia    
## [1366] Asia     Asia     Asia     Europe   Europe   Europe   Europe  
## [1373] Europe   Europe   Europe   Europe   Europe   Europe   Europe  
## [1380] Europe   Europe   Europe   Europe   Europe   Europe   Europe  
## [1387] Europe   Europe   Europe   Europe   Europe   Europe   Africa  
## [1394] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
## [1401] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
## [1408] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
## [1415] Africa   Africa   Europe   Europe   Europe   Europe   Europe  
## [1422] Europe   Europe   Europe   Europe   Europe   Europe   Europe  
## [1429] Asia     Asia     Asia     Asia     Asia     Asia     Asia    
## [1436] Asia     Asia     Asia     Asia     Asia     Africa   Africa  
## [1443] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
## [1450] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
## [1457] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
## [1464] Africa   Europe   Europe   Europe   Europe   Europe   Europe  
## [1471] Europe   Europe   Europe   Europe   Europe   Europe   Europe  
## [1478] Europe   Europe   Europe   Europe   Europe   Europe   Europe  
## [1485] Europe   Europe   Europe   Europe   Asia     Asia     Asia    
## [1492] Asia     Asia     Asia     Asia     Asia     Asia     Asia    
## [1499] Asia     Asia     Asia     Asia     Asia     Asia     Asia    
## [1506] Asia     Asia     Asia     Asia     Asia     Asia     Asia    
## [1513] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
## [1520] Africa   Africa   Africa   Africa   Africa   Asia     Asia    
## [1527] Asia     Asia     Asia     Asia     Asia     Asia     Asia    
## [1534] Asia     Asia     Asia     Africa   Africa   Africa   Africa  
## [1541] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
## [1548] Africa   Americas Americas Americas Americas Americas Americas
## [1555] Americas Americas Americas Americas Americas Americas Africa  
## [1562] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
## [1569] Africa   Africa   Africa   Africa   Europe   Europe   Europe  
## [1576] Europe   Europe   Europe   Europe   Europe   Europe   Europe  
## [1583] Europe   Europe   Africa   Africa   Africa   Africa   Africa  
## [1590] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
## [1597] Europe   Europe   Europe   Europe   Europe   Europe   Europe  
## [1604] Europe   Europe   Europe   Europe   Europe   Americas Americas
## [1611] Americas Americas Americas Americas Americas Americas Americas
## [1618] Americas Americas Americas Americas Americas Americas Americas
## [1625] Americas Americas Americas Americas Americas Americas Americas
## [1632] Americas Americas Americas Americas Americas Americas Americas
## [1639] Americas Americas Americas Americas Americas Americas Asia    
## [1646] Asia     Asia     Asia     Asia     Asia     Asia     Asia    
## [1653] Asia     Asia     Asia     Asia     Asia     Asia     Asia    
## [1660] Asia     Asia     Asia     Asia     Asia     Asia     Asia    
## [1667] Asia     Asia     Asia     Asia     Asia     Asia     Asia    
## [1674] Asia     Asia     Asia     Asia     Asia     Asia     Asia    
## [1681] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
## [1688] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
## [1695] Africa   Africa   Africa   Africa   Africa   Africa   Africa  
## [1702] Africa   Africa   Africa  
## Levels: Africa Americas Asia Europe Oceania
```

To select multiple columns, we need to pass in the column names as a list:


```r
gap[, list(continent, country, pop)]
```

```
##       continent     country      pop
##    1:      Asia Afghanistan  8425333
##    2:      Asia Afghanistan  9240934
##    3:      Asia Afghanistan 10267083
##    4:      Asia Afghanistan 11537966
##    5:      Asia Afghanistan 13079460
##   ---                               
## 1700:    Africa    Zimbabwe  9216418
## 1701:    Africa    Zimbabwe 10704340
## 1702:    Africa    Zimbabwe 11404948
## 1703:    Africa    Zimbabwe 11926563
## 1704:    Africa    Zimbabwe 12311143
```

```r
# data frame equivalent
gap_df[, c("continent", "country", "pop")]
```

```
##      continent                  country        pop
## 1         Asia              Afghanistan    8425333
## 2         Asia              Afghanistan    9240934
## 3         Asia              Afghanistan   10267083
## 4         Asia              Afghanistan   11537966
## 5         Asia              Afghanistan   13079460
## 6         Asia              Afghanistan   14880372
## 7         Asia              Afghanistan   12881816
## 8         Asia              Afghanistan   13867957
## 9         Asia              Afghanistan   16317921
## 10        Asia              Afghanistan   22227415
## 11        Asia              Afghanistan   25268405
## 12        Asia              Afghanistan   31889923
## 13      Europe                  Albania    1282697
## 14      Europe                  Albania    1476505
## 15      Europe                  Albania    1728137
## 16      Europe                  Albania    1984060
## 17      Europe                  Albania    2263554
## 18      Europe                  Albania    2509048
## 19      Europe                  Albania    2780097
## 20      Europe                  Albania    3075321
## 21      Europe                  Albania    3326498
## 22      Europe                  Albania    3428038
## 23      Europe                  Albania    3508512
## 24      Europe                  Albania    3600523
## 25      Africa                  Algeria    9279525
## 26      Africa                  Algeria   10270856
## 27      Africa                  Algeria   11000948
## 28      Africa                  Algeria   12760499
## 29      Africa                  Algeria   14760787
## 30      Africa                  Algeria   17152804
## 31      Africa                  Algeria   20033753
## 32      Africa                  Algeria   23254956
## 33      Africa                  Algeria   26298373
## 34      Africa                  Algeria   29072015
## 35      Africa                  Algeria   31287142
## 36      Africa                  Algeria   33333216
## 37      Africa                   Angola    4232095
## 38      Africa                   Angola    4561361
## 39      Africa                   Angola    4826015
## 40      Africa                   Angola    5247469
## 41      Africa                   Angola    5894858
## 42      Africa                   Angola    6162675
## 43      Africa                   Angola    7016384
## 44      Africa                   Angola    7874230
## 45      Africa                   Angola    8735988
## 46      Africa                   Angola    9875024
## 47      Africa                   Angola   10866106
## 48      Africa                   Angola   12420476
## 49    Americas                Argentina   17876956
## 50    Americas                Argentina   19610538
## 51    Americas                Argentina   21283783
## 52    Americas                Argentina   22934225
## 53    Americas                Argentina   24779799
## 54    Americas                Argentina   26983828
## 55    Americas                Argentina   29341374
## 56    Americas                Argentina   31620918
## 57    Americas                Argentina   33958947
## 58    Americas                Argentina   36203463
## 59    Americas                Argentina   38331121
## 60    Americas                Argentina   40301927
## 61     Oceania                Australia    8691212
## 62     Oceania                Australia    9712569
## 63     Oceania                Australia   10794968
## 64     Oceania                Australia   11872264
## 65     Oceania                Australia   13177000
## 66     Oceania                Australia   14074100
## 67     Oceania                Australia   15184200
## 68     Oceania                Australia   16257249
## 69     Oceania                Australia   17481977
## 70     Oceania                Australia   18565243
## 71     Oceania                Australia   19546792
## 72     Oceania                Australia   20434176
## 73      Europe                  Austria    6927772
## 74      Europe                  Austria    6965860
## 75      Europe                  Austria    7129864
## 76      Europe                  Austria    7376998
## 77      Europe                  Austria    7544201
## 78      Europe                  Austria    7568430
## 79      Europe                  Austria    7574613
## 80      Europe                  Austria    7578903
## 81      Europe                  Austria    7914969
## 82      Europe                  Austria    8069876
## 83      Europe                  Austria    8148312
## 84      Europe                  Austria    8199783
## 85        Asia                  Bahrain     120447
## 86        Asia                  Bahrain     138655
## 87        Asia                  Bahrain     171863
## 88        Asia                  Bahrain     202182
## 89        Asia                  Bahrain     230800
## 90        Asia                  Bahrain     297410
## 91        Asia                  Bahrain     377967
## 92        Asia                  Bahrain     454612
## 93        Asia                  Bahrain     529491
## 94        Asia                  Bahrain     598561
## 95        Asia                  Bahrain     656397
## 96        Asia                  Bahrain     708573
## 97        Asia               Bangladesh   46886859
## 98        Asia               Bangladesh   51365468
## 99        Asia               Bangladesh   56839289
## 100       Asia               Bangladesh   62821884
## 101       Asia               Bangladesh   70759295
## 102       Asia               Bangladesh   80428306
## 103       Asia               Bangladesh   93074406
## 104       Asia               Bangladesh  103764241
## 105       Asia               Bangladesh  113704579
## 106       Asia               Bangladesh  123315288
## 107       Asia               Bangladesh  135656790
## 108       Asia               Bangladesh  150448339
## 109     Europe                  Belgium    8730405
## 110     Europe                  Belgium    8989111
## 111     Europe                  Belgium    9218400
## 112     Europe                  Belgium    9556500
## 113     Europe                  Belgium    9709100
## 114     Europe                  Belgium    9821800
## 115     Europe                  Belgium    9856303
## 116     Europe                  Belgium    9870200
## 117     Europe                  Belgium   10045622
## 118     Europe                  Belgium   10199787
## 119     Europe                  Belgium   10311970
## 120     Europe                  Belgium   10392226
## 121     Africa                    Benin    1738315
## 122     Africa                    Benin    1925173
## 123     Africa                    Benin    2151895
## 124     Africa                    Benin    2427334
## 125     Africa                    Benin    2761407
## 126     Africa                    Benin    3168267
## 127     Africa                    Benin    3641603
## 128     Africa                    Benin    4243788
## 129     Africa                    Benin    4981671
## 130     Africa                    Benin    6066080
## 131     Africa                    Benin    7026113
## 132     Africa                    Benin    8078314
## 133   Americas                  Bolivia    2883315
## 134   Americas                  Bolivia    3211738
## 135   Americas                  Bolivia    3593918
## 136   Americas                  Bolivia    4040665
## 137   Americas                  Bolivia    4565872
## 138   Americas                  Bolivia    5079716
## 139   Americas                  Bolivia    5642224
## 140   Americas                  Bolivia    6156369
## 141   Americas                  Bolivia    6893451
## 142   Americas                  Bolivia    7693188
## 143   Americas                  Bolivia    8445134
## 144   Americas                  Bolivia    9119152
## 145     Europe   Bosnia and Herzegovina    2791000
## 146     Europe   Bosnia and Herzegovina    3076000
## 147     Europe   Bosnia and Herzegovina    3349000
## 148     Europe   Bosnia and Herzegovina    3585000
## 149     Europe   Bosnia and Herzegovina    3819000
## 150     Europe   Bosnia and Herzegovina    4086000
## 151     Europe   Bosnia and Herzegovina    4172693
## 152     Europe   Bosnia and Herzegovina    4338977
## 153     Europe   Bosnia and Herzegovina    4256013
## 154     Europe   Bosnia and Herzegovina    3607000
## 155     Europe   Bosnia and Herzegovina    4165416
## 156     Europe   Bosnia and Herzegovina    4552198
## 157     Africa                 Botswana     442308
## 158     Africa                 Botswana     474639
## 159     Africa                 Botswana     512764
## 160     Africa                 Botswana     553541
## 161     Africa                 Botswana     619351
## 162     Africa                 Botswana     781472
## 163     Africa                 Botswana     970347
## 164     Africa                 Botswana    1151184
## 165     Africa                 Botswana    1342614
## 166     Africa                 Botswana    1536536
## 167     Africa                 Botswana    1630347
## 168     Africa                 Botswana    1639131
## 169   Americas                   Brazil   56602560
## 170   Americas                   Brazil   65551171
## 171   Americas                   Brazil   76039390
## 172   Americas                   Brazil   88049823
## 173   Americas                   Brazil  100840058
## 174   Americas                   Brazil  114313951
## 175   Americas                   Brazil  128962939
## 176   Americas                   Brazil  142938076
## 177   Americas                   Brazil  155975974
## 178   Americas                   Brazil  168546719
## 179   Americas                   Brazil  179914212
## 180   Americas                   Brazil  190010647
## 181     Europe                 Bulgaria    7274900
## 182     Europe                 Bulgaria    7651254
## 183     Europe                 Bulgaria    8012946
## 184     Europe                 Bulgaria    8310226
## 185     Europe                 Bulgaria    8576200
## 186     Europe                 Bulgaria    8797022
## 187     Europe                 Bulgaria    8892098
## 188     Europe                 Bulgaria    8971958
## 189     Europe                 Bulgaria    8658506
## 190     Europe                 Bulgaria    8066057
## 191     Europe                 Bulgaria    7661799
## 192     Europe                 Bulgaria    7322858
## 193     Africa             Burkina Faso    4469979
## 194     Africa             Burkina Faso    4713416
## 195     Africa             Burkina Faso    4919632
## 196     Africa             Burkina Faso    5127935
## 197     Africa             Burkina Faso    5433886
## 198     Africa             Burkina Faso    5889574
## 199     Africa             Burkina Faso    6634596
## 200     Africa             Burkina Faso    7586551
## 201     Africa             Burkina Faso    8878303
## 202     Africa             Burkina Faso   10352843
## 203     Africa             Burkina Faso   12251209
## 204     Africa             Burkina Faso   14326203
## 205     Africa                  Burundi    2445618
## 206     Africa                  Burundi    2667518
## 207     Africa                  Burundi    2961915
## 208     Africa                  Burundi    3330989
## 209     Africa                  Burundi    3529983
## 210     Africa                  Burundi    3834415
## 211     Africa                  Burundi    4580410
## 212     Africa                  Burundi    5126023
## 213     Africa                  Burundi    5809236
## 214     Africa                  Burundi    6121610
## 215     Africa                  Burundi    7021078
## 216     Africa                  Burundi    8390505
## 217       Asia                 Cambodia    4693836
## 218       Asia                 Cambodia    5322536
## 219       Asia                 Cambodia    6083619
## 220       Asia                 Cambodia    6960067
## 221       Asia                 Cambodia    7450606
## 222       Asia                 Cambodia    6978607
## 223       Asia                 Cambodia    7272485
## 224       Asia                 Cambodia    8371791
## 225       Asia                 Cambodia   10150094
## 226       Asia                 Cambodia   11782962
## 227       Asia                 Cambodia   12926707
## 228       Asia                 Cambodia   14131858
## 229     Africa                 Cameroon    5009067
## 230     Africa                 Cameroon    5359923
## 231     Africa                 Cameroon    5793633
## 232     Africa                 Cameroon    6335506
## 233     Africa                 Cameroon    7021028
## 234     Africa                 Cameroon    7959865
## 235     Africa                 Cameroon    9250831
## 236     Africa                 Cameroon   10780667
## 237     Africa                 Cameroon   12467171
## 238     Africa                 Cameroon   14195809
## 239     Africa                 Cameroon   15929988
## 240     Africa                 Cameroon   17696293
## 241   Americas                   Canada   14785584
## 242   Americas                   Canada   17010154
## 243   Americas                   Canada   18985849
## 244   Americas                   Canada   20819767
## 245   Americas                   Canada   22284500
## 246   Americas                   Canada   23796400
## 247   Americas                   Canada   25201900
## 248   Americas                   Canada   26549700
## 249   Americas                   Canada   28523502
## 250   Americas                   Canada   30305843
## 251   Americas                   Canada   31902268
## 252   Americas                   Canada   33390141
## 253     Africa Central African Republic    1291695
## 254     Africa Central African Republic    1392284
## 255     Africa Central African Republic    1523478
## 256     Africa Central African Republic    1733638
## 257     Africa Central African Republic    1927260
## 258     Africa Central African Republic    2167533
## 259     Africa Central African Republic    2476971
## 260     Africa Central African Republic    2840009
## 261     Africa Central African Republic    3265124
## 262     Africa Central African Republic    3696513
## 263     Africa Central African Republic    4048013
## 264     Africa Central African Republic    4369038
## 265     Africa                     Chad    2682462
## 266     Africa                     Chad    2894855
## 267     Africa                     Chad    3150417
## 268     Africa                     Chad    3495967
## 269     Africa                     Chad    3899068
## 270     Africa                     Chad    4388260
## 271     Africa                     Chad    4875118
## 272     Africa                     Chad    5498955
## 273     Africa                     Chad    6429417
## 274     Africa                     Chad    7562011
## 275     Africa                     Chad    8835739
## 276     Africa                     Chad   10238807
## 277   Americas                    Chile    6377619
## 278   Americas                    Chile    7048426
## 279   Americas                    Chile    7961258
## 280   Americas                    Chile    8858908
## 281   Americas                    Chile    9717524
## 282   Americas                    Chile   10599793
## 283   Americas                    Chile   11487112
## 284   Americas                    Chile   12463354
## 285   Americas                    Chile   13572994
## 286   Americas                    Chile   14599929
## 287   Americas                    Chile   15497046
## 288   Americas                    Chile   16284741
## 289       Asia                    China  556263528
## 290       Asia                    China  637408000
## 291       Asia                    China  665770000
## 292       Asia                    China  754550000
## 293       Asia                    China  862030000
## 294       Asia                    China  943455000
## 295       Asia                    China 1000281000
## 296       Asia                    China 1084035000
## 297       Asia                    China 1164970000
## 298       Asia                    China 1230075000
## 299       Asia                    China 1280400000
## 300       Asia                    China 1318683096
## 301   Americas                 Colombia   12350771
## 302   Americas                 Colombia   14485993
## 303   Americas                 Colombia   17009885
## 304   Americas                 Colombia   19764027
## 305   Americas                 Colombia   22542890
## 306   Americas                 Colombia   25094412
## 307   Americas                 Colombia   27764644
## 308   Americas                 Colombia   30964245
## 309   Americas                 Colombia   34202721
## 310   Americas                 Colombia   37657830
## 311   Americas                 Colombia   41008227
## 312   Americas                 Colombia   44227550
## 313     Africa                  Comoros     153936
## 314     Africa                  Comoros     170928
## 315     Africa                  Comoros     191689
## 316     Africa                  Comoros     217378
## 317     Africa                  Comoros     250027
## 318     Africa                  Comoros     304739
## 319     Africa                  Comoros     348643
## 320     Africa                  Comoros     395114
## 321     Africa                  Comoros     454429
## 322     Africa                  Comoros     527982
## 323     Africa                  Comoros     614382
## 324     Africa                  Comoros     710960
## 325     Africa          Congo Dem. Rep.   14100005
## 326     Africa          Congo Dem. Rep.   15577932
## 327     Africa          Congo Dem. Rep.   17486434
## 328     Africa          Congo Dem. Rep.   19941073
## 329     Africa          Congo Dem. Rep.   23007669
## 330     Africa          Congo Dem. Rep.   26480870
## 331     Africa          Congo Dem. Rep.   30646495
## 332     Africa          Congo Dem. Rep.   35481645
## 333     Africa          Congo Dem. Rep.   41672143
## 334     Africa          Congo Dem. Rep.   47798986
## 335     Africa          Congo Dem. Rep.   55379852
## 336     Africa          Congo Dem. Rep.   64606759
## 337     Africa               Congo Rep.     854885
## 338     Africa               Congo Rep.     940458
## 339     Africa               Congo Rep.    1047924
## 340     Africa               Congo Rep.    1179760
## 341     Africa               Congo Rep.    1340458
## 342     Africa               Congo Rep.    1536769
## 343     Africa               Congo Rep.    1774735
## 344     Africa               Congo Rep.    2064095
## 345     Africa               Congo Rep.    2409073
## 346     Africa               Congo Rep.    2800947
## 347     Africa               Congo Rep.    3328795
## 348     Africa               Congo Rep.    3800610
## 349   Americas               Costa Rica     926317
## 350   Americas               Costa Rica    1112300
## 351   Americas               Costa Rica    1345187
## 352   Americas               Costa Rica    1588717
## 353   Americas               Costa Rica    1834796
## 354   Americas               Costa Rica    2108457
## 355   Americas               Costa Rica    2424367
## 356   Americas               Costa Rica    2799811
## 357   Americas               Costa Rica    3173216
## 358   Americas               Costa Rica    3518107
## 359   Americas               Costa Rica    3834934
## 360   Americas               Costa Rica    4133884
## 361     Africa            Cote d'Ivoire    2977019
## 362     Africa            Cote d'Ivoire    3300000
## 363     Africa            Cote d'Ivoire    3832408
## 364     Africa            Cote d'Ivoire    4744870
## 365     Africa            Cote d'Ivoire    6071696
## 366     Africa            Cote d'Ivoire    7459574
## 367     Africa            Cote d'Ivoire    9025951
## 368     Africa            Cote d'Ivoire   10761098
## 369     Africa            Cote d'Ivoire   12772596
## 370     Africa            Cote d'Ivoire   14625967
## 371     Africa            Cote d'Ivoire   16252726
## 372     Africa            Cote d'Ivoire   18013409
## 373     Europe                  Croatia    3882229
## 374     Europe                  Croatia    3991242
## 375     Europe                  Croatia    4076557
## 376     Europe                  Croatia    4174366
## 377     Europe                  Croatia    4225310
## 378     Europe                  Croatia    4318673
## 379     Europe                  Croatia    4413368
## 380     Europe                  Croatia    4484310
## 381     Europe                  Croatia    4494013
## 382     Europe                  Croatia    4444595
## 383     Europe                  Croatia    4481020
## 384     Europe                  Croatia    4493312
## 385   Americas                     Cuba    6007797
## 386   Americas                     Cuba    6640752
## 387   Americas                     Cuba    7254373
## 388   Americas                     Cuba    8139332
## 389   Americas                     Cuba    8831348
## 390   Americas                     Cuba    9537988
## 391   Americas                     Cuba    9789224
## 392   Americas                     Cuba   10239839
## 393   Americas                     Cuba   10723260
## 394   Americas                     Cuba   10983007
## 395   Americas                     Cuba   11226999
## 396   Americas                     Cuba   11416987
## 397     Europe           Czech Republic    9125183
## 398     Europe           Czech Republic    9513758
## 399     Europe           Czech Republic    9620282
## 400     Europe           Czech Republic    9835109
## 401     Europe           Czech Republic    9862158
## 402     Europe           Czech Republic   10161915
## 403     Europe           Czech Republic   10303704
## 404     Europe           Czech Republic   10311597
## 405     Europe           Czech Republic   10315702
## 406     Europe           Czech Republic   10300707
## 407     Europe           Czech Republic   10256295
## 408     Europe           Czech Republic   10228744
## 409     Europe                  Denmark    4334000
## 410     Europe                  Denmark    4487831
## 411     Europe                  Denmark    4646899
## 412     Europe                  Denmark    4838800
## 413     Europe                  Denmark    4991596
## 414     Europe                  Denmark    5088419
## 415     Europe                  Denmark    5117810
## 416     Europe                  Denmark    5127024
## 417     Europe                  Denmark    5171393
## 418     Europe                  Denmark    5283663
## 419     Europe                  Denmark    5374693
## 420     Europe                  Denmark    5468120
## 421     Africa                 Djibouti      63149
## 422     Africa                 Djibouti      71851
## 423     Africa                 Djibouti      89898
## 424     Africa                 Djibouti     127617
## 425     Africa                 Djibouti     178848
## 426     Africa                 Djibouti     228694
## 427     Africa                 Djibouti     305991
## 428     Africa                 Djibouti     311025
## 429     Africa                 Djibouti     384156
## 430     Africa                 Djibouti     417908
## 431     Africa                 Djibouti     447416
## 432     Africa                 Djibouti     496374
## 433   Americas       Dominican Republic    2491346
## 434   Americas       Dominican Republic    2923186
## 435   Americas       Dominican Republic    3453434
## 436   Americas       Dominican Republic    4049146
## 437   Americas       Dominican Republic    4671329
## 438   Americas       Dominican Republic    5302800
## 439   Americas       Dominican Republic    5968349
## 440   Americas       Dominican Republic    6655297
## 441   Americas       Dominican Republic    7351181
## 442   Americas       Dominican Republic    7992357
## 443   Americas       Dominican Republic    8650322
## 444   Americas       Dominican Republic    9319622
## 445   Americas                  Ecuador    3548753
## 446   Americas                  Ecuador    4058385
## 447   Americas                  Ecuador    4681707
## 448   Americas                  Ecuador    5432424
## 449   Americas                  Ecuador    6298651
## 450   Americas                  Ecuador    7278866
## 451   Americas                  Ecuador    8365850
## 452   Americas                  Ecuador    9545158
## 453   Americas                  Ecuador   10748394
## 454   Americas                  Ecuador   11911819
## 455   Americas                  Ecuador   12921234
## 456   Americas                  Ecuador   13755680
## 457     Africa                    Egypt   22223309
## 458     Africa                    Egypt   25009741
## 459     Africa                    Egypt   28173309
## 460     Africa                    Egypt   31681188
## 461     Africa                    Egypt   34807417
## 462     Africa                    Egypt   38783863
## 463     Africa                    Egypt   45681811
## 464     Africa                    Egypt   52799062
## 465     Africa                    Egypt   59402198
## 466     Africa                    Egypt   66134291
## 467     Africa                    Egypt   73312559
## 468     Africa                    Egypt   80264543
## 469   Americas              El Salvador    2042865
## 470   Americas              El Salvador    2355805
## 471   Americas              El Salvador    2747687
## 472   Americas              El Salvador    3232927
## 473   Americas              El Salvador    3790903
## 474   Americas              El Salvador    4282586
## 475   Americas              El Salvador    4474873
## 476   Americas              El Salvador    4842194
## 477   Americas              El Salvador    5274649
## 478   Americas              El Salvador    5783439
## 479   Americas              El Salvador    6353681
## 480   Americas              El Salvador    6939688
## 481     Africa        Equatorial Guinea     216964
## 482     Africa        Equatorial Guinea     232922
## 483     Africa        Equatorial Guinea     249220
## 484     Africa        Equatorial Guinea     259864
## 485     Africa        Equatorial Guinea     277603
## 486     Africa        Equatorial Guinea     192675
## 487     Africa        Equatorial Guinea     285483
## 488     Africa        Equatorial Guinea     341244
## 489     Africa        Equatorial Guinea     387838
## 490     Africa        Equatorial Guinea     439971
## 491     Africa        Equatorial Guinea     495627
## 492     Africa        Equatorial Guinea     551201
## 493     Africa                  Eritrea    1438760
## 494     Africa                  Eritrea    1542611
## 495     Africa                  Eritrea    1666618
## 496     Africa                  Eritrea    1820319
## 497     Africa                  Eritrea    2260187
## 498     Africa                  Eritrea    2512642
## 499     Africa                  Eritrea    2637297
## 500     Africa                  Eritrea    2915959
## 501     Africa                  Eritrea    3668440
## 502     Africa                  Eritrea    4058319
## 503     Africa                  Eritrea    4414865
## 504     Africa                  Eritrea    4906585
## 505     Africa                 Ethiopia   20860941
## 506     Africa                 Ethiopia   22815614
## 507     Africa                 Ethiopia   25145372
## 508     Africa                 Ethiopia   27860297
## 509     Africa                 Ethiopia   30770372
## 510     Africa                 Ethiopia   34617799
## 511     Africa                 Ethiopia   38111756
## 512     Africa                 Ethiopia   42999530
## 513     Africa                 Ethiopia   52088559
## 514     Africa                 Ethiopia   59861301
## 515     Africa                 Ethiopia   67946797
## 516     Africa                 Ethiopia   76511887
## 517     Europe                  Finland    4090500
## 518     Europe                  Finland    4324000
## 519     Europe                  Finland    4491443
## 520     Europe                  Finland    4605744
## 521     Europe                  Finland    4639657
## 522     Europe                  Finland    4738902
## 523     Europe                  Finland    4826933
## 524     Europe                  Finland    4931729
## 525     Europe                  Finland    5041039
## 526     Europe                  Finland    5134406
## 527     Europe                  Finland    5193039
## 528     Europe                  Finland    5238460
## 529     Europe                   France   42459667
## 530     Europe                   France   44310863
## 531     Europe                   France   47124000
## 532     Europe                   France   49569000
## 533     Europe                   France   51732000
## 534     Europe                   France   53165019
## 535     Europe                   France   54433565
## 536     Europe                   France   55630100
## 537     Europe                   France   57374179
## 538     Europe                   France   58623428
## 539     Europe                   France   59925035
## 540     Europe                   France   61083916
## 541     Africa                    Gabon     420702
## 542     Africa                    Gabon     434904
## 543     Africa                    Gabon     455661
## 544     Africa                    Gabon     489004
## 545     Africa                    Gabon     537977
## 546     Africa                    Gabon     706367
## 547     Africa                    Gabon     753874
## 548     Africa                    Gabon     880397
## 549     Africa                    Gabon     985739
## 550     Africa                    Gabon    1126189
## 551     Africa                    Gabon    1299304
## 552     Africa                    Gabon    1454867
## 553     Africa                   Gambia     284320
## 554     Africa                   Gambia     323150
## 555     Africa                   Gambia     374020
## 556     Africa                   Gambia     439593
## 557     Africa                   Gambia     517101
## 558     Africa                   Gambia     608274
## 559     Africa                   Gambia     715523
## 560     Africa                   Gambia     848406
## 561     Africa                   Gambia    1025384
## 562     Africa                   Gambia    1235767
## 563     Africa                   Gambia    1457766
## 564     Africa                   Gambia    1688359
## 565     Europe                  Germany   69145952
## 566     Europe                  Germany   71019069
## 567     Europe                  Germany   73739117
## 568     Europe                  Germany   76368453
## 569     Europe                  Germany   78717088
## 570     Europe                  Germany   78160773
## 571     Europe                  Germany   78335266
## 572     Europe                  Germany   77718298
## 573     Europe                  Germany   80597764
## 574     Europe                  Germany   82011073
## 575     Europe                  Germany   82350671
## 576     Europe                  Germany   82400996
## 577     Africa                    Ghana    5581001
## 578     Africa                    Ghana    6391288
## 579     Africa                    Ghana    7355248
## 580     Africa                    Ghana    8490213
## 581     Africa                    Ghana    9354120
## 582     Africa                    Ghana   10538093
## 583     Africa                    Ghana   11400338
## 584     Africa                    Ghana   14168101
## 585     Africa                    Ghana   16278738
## 586     Africa                    Ghana   18418288
## 587     Africa                    Ghana   20550751
## 588     Africa                    Ghana   22873338
## 589     Europe                   Greece    7733250
## 590     Europe                   Greece    8096218
## 591     Europe                   Greece    8448233
## 592     Europe                   Greece    8716441
## 593     Europe                   Greece    8888628
## 594     Europe                   Greece    9308479
## 595     Europe                   Greece    9786480
## 596     Europe                   Greece    9974490
## 597     Europe                   Greece   10325429
## 598     Europe                   Greece   10502372
## 599     Europe                   Greece   10603863
## 600     Europe                   Greece   10706290
## 601   Americas                Guatemala    3146381
## 602   Americas                Guatemala    3640876
## 603   Americas                Guatemala    4208858
## 604   Americas                Guatemala    4690773
## 605   Americas                Guatemala    5149581
## 606   Americas                Guatemala    5703430
## 607   Americas                Guatemala    6395630
## 608   Americas                Guatemala    7326406
## 609   Americas                Guatemala    8486949
## 610   Americas                Guatemala    9803875
## 611   Americas                Guatemala   11178650
## 612   Americas                Guatemala   12572928
## 613     Africa                   Guinea    2664249
## 614     Africa                   Guinea    2876726
## 615     Africa                   Guinea    3140003
## 616     Africa                   Guinea    3451418
## 617     Africa                   Guinea    3811387
## 618     Africa                   Guinea    4227026
## 619     Africa                   Guinea    4710497
## 620     Africa                   Guinea    5650262
## 621     Africa                   Guinea    6990574
## 622     Africa                   Guinea    8048834
## 623     Africa                   Guinea    8807818
## 624     Africa                   Guinea    9947814
## 625     Africa            Guinea-Bissau     580653
## 626     Africa            Guinea-Bissau     601095
## 627     Africa            Guinea-Bissau     627820
## 628     Africa            Guinea-Bissau     601287
## 629     Africa            Guinea-Bissau     625361
## 630     Africa            Guinea-Bissau     745228
## 631     Africa            Guinea-Bissau     825987
## 632     Africa            Guinea-Bissau     927524
## 633     Africa            Guinea-Bissau    1050938
## 634     Africa            Guinea-Bissau    1193708
## 635     Africa            Guinea-Bissau    1332459
## 636     Africa            Guinea-Bissau    1472041
## 637   Americas                    Haiti    3201488
## 638   Americas                    Haiti    3507701
## 639   Americas                    Haiti    3880130
## 640   Americas                    Haiti    4318137
## 641   Americas                    Haiti    4698301
## 642   Americas                    Haiti    4908554
## 643   Americas                    Haiti    5198399
## 644   Americas                    Haiti    5756203
## 645   Americas                    Haiti    6326682
## 646   Americas                    Haiti    6913545
## 647   Americas                    Haiti    7607651
## 648   Americas                    Haiti    8502814
## 649   Americas                 Honduras    1517453
## 650   Americas                 Honduras    1770390
## 651   Americas                 Honduras    2090162
## 652   Americas                 Honduras    2500689
## 653   Americas                 Honduras    2965146
## 654   Americas                 Honduras    3055235
## 655   Americas                 Honduras    3669448
## 656   Americas                 Honduras    4372203
## 657   Americas                 Honduras    5077347
## 658   Americas                 Honduras    5867957
## 659   Americas                 Honduras    6677328
## 660   Americas                 Honduras    7483763
## 661       Asia          Hong Kong China    2125900
## 662       Asia          Hong Kong China    2736300
## 663       Asia          Hong Kong China    3305200
## 664       Asia          Hong Kong China    3722800
## 665       Asia          Hong Kong China    4115700
## 666       Asia          Hong Kong China    4583700
## 667       Asia          Hong Kong China    5264500
## 668       Asia          Hong Kong China    5584510
## 669       Asia          Hong Kong China    5829696
## 670       Asia          Hong Kong China    6495918
## 671       Asia          Hong Kong China    6762476
## 672       Asia          Hong Kong China    6980412
## 673     Europe                  Hungary    9504000
## 674     Europe                  Hungary    9839000
## 675     Europe                  Hungary   10063000
## 676     Europe                  Hungary   10223422
## 677     Europe                  Hungary   10394091
## 678     Europe                  Hungary   10637171
## 679     Europe                  Hungary   10705535
## 680     Europe                  Hungary   10612740
## 681     Europe                  Hungary   10348684
## 682     Europe                  Hungary   10244684
## 683     Europe                  Hungary   10083313
## 684     Europe                  Hungary    9956108
## 685     Europe                  Iceland     147962
## 686     Europe                  Iceland     165110
## 687     Europe                  Iceland     182053
## 688     Europe                  Iceland     198676
## 689     Europe                  Iceland     209275
## 690     Europe                  Iceland     221823
## 691     Europe                  Iceland     233997
## 692     Europe                  Iceland     244676
## 693     Europe                  Iceland     259012
## 694     Europe                  Iceland     271192
## 695     Europe                  Iceland     288030
## 696     Europe                  Iceland     301931
## 697       Asia                    India  372000000
## 698       Asia                    India  409000000
## 699       Asia                    India  454000000
## 700       Asia                    India  506000000
## 701       Asia                    India  567000000
## 702       Asia                    India  634000000
## 703       Asia                    India  708000000
## 704       Asia                    India  788000000
## 705       Asia                    India  872000000
## 706       Asia                    India  959000000
## 707       Asia                    India 1034172547
## 708       Asia                    India 1110396331
## 709       Asia                Indonesia   82052000
## 710       Asia                Indonesia   90124000
## 711       Asia                Indonesia   99028000
## 712       Asia                Indonesia  109343000
## 713       Asia                Indonesia  121282000
## 714       Asia                Indonesia  136725000
## 715       Asia                Indonesia  153343000
## 716       Asia                Indonesia  169276000
## 717       Asia                Indonesia  184816000
## 718       Asia                Indonesia  199278000
## 719       Asia                Indonesia  211060000
## 720       Asia                Indonesia  223547000
## 721       Asia                     Iran   17272000
## 722       Asia                     Iran   19792000
## 723       Asia                     Iran   22874000
## 724       Asia                     Iran   26538000
## 725       Asia                     Iran   30614000
## 726       Asia                     Iran   35480679
## 727       Asia                     Iran   43072751
## 728       Asia                     Iran   51889696
## 729       Asia                     Iran   60397973
## 730       Asia                     Iran   63327987
## 731       Asia                     Iran   66907826
## 732       Asia                     Iran   69453570
## 733       Asia                     Iraq    5441766
## 734       Asia                     Iraq    6248643
## 735       Asia                     Iraq    7240260
## 736       Asia                     Iraq    8519282
## 737       Asia                     Iraq   10061506
## 738       Asia                     Iraq   11882916
## 739       Asia                     Iraq   14173318
## 740       Asia                     Iraq   16543189
## 741       Asia                     Iraq   17861905
## 742       Asia                     Iraq   20775703
## 743       Asia                     Iraq   24001816
## 744       Asia                     Iraq   27499638
## 745     Europe                  Ireland    2952156
## 746     Europe                  Ireland    2878220
## 747     Europe                  Ireland    2830000
## 748     Europe                  Ireland    2900100
## 749     Europe                  Ireland    3024400
## 750     Europe                  Ireland    3271900
## 751     Europe                  Ireland    3480000
## 752     Europe                  Ireland    3539900
## 753     Europe                  Ireland    3557761
## 754     Europe                  Ireland    3667233
## 755     Europe                  Ireland    3879155
## 756     Europe                  Ireland    4109086
## 757       Asia                   Israel    1620914
## 758       Asia                   Israel    1944401
## 759       Asia                   Israel    2310904
## 760       Asia                   Israel    2693585
## 761       Asia                   Israel    3095893
## 762       Asia                   Israel    3495918
## 763       Asia                   Israel    3858421
## 764       Asia                   Israel    4203148
## 765       Asia                   Israel    4936550
## 766       Asia                   Israel    5531387
## 767       Asia                   Israel    6029529
## 768       Asia                   Israel    6426679
## 769     Europe                    Italy   47666000
## 770     Europe                    Italy   49182000
## 771     Europe                    Italy   50843200
## 772     Europe                    Italy   52667100
## 773     Europe                    Italy   54365564
## 774     Europe                    Italy   56059245
## 775     Europe                    Italy   56535636
## 776     Europe                    Italy   56729703
## 777     Europe                    Italy   56840847
## 778     Europe                    Italy   57479469
## 779     Europe                    Italy   57926999
## 780     Europe                    Italy   58147733
## 781   Americas                  Jamaica    1426095
## 782   Americas                  Jamaica    1535090
## 783   Americas                  Jamaica    1665128
## 784   Americas                  Jamaica    1861096
## 785   Americas                  Jamaica    1997616
## 786   Americas                  Jamaica    2156814
## 787   Americas                  Jamaica    2298309
## 788   Americas                  Jamaica    2326606
## 789   Americas                  Jamaica    2378618
## 790   Americas                  Jamaica    2531311
## 791   Americas                  Jamaica    2664659
## 792   Americas                  Jamaica    2780132
## 793       Asia                    Japan   86459025
## 794       Asia                    Japan   91563009
## 795       Asia                    Japan   95831757
## 796       Asia                    Japan  100825279
## 797       Asia                    Japan  107188273
## 798       Asia                    Japan  113872473
## 799       Asia                    Japan  118454974
## 800       Asia                    Japan  122091325
## 801       Asia                    Japan  124329269
## 802       Asia                    Japan  125956499
## 803       Asia                    Japan  127065841
## 804       Asia                    Japan  127467972
## 805       Asia                   Jordan     607914
## 806       Asia                   Jordan     746559
## 807       Asia                   Jordan     933559
## 808       Asia                   Jordan    1255058
## 809       Asia                   Jordan    1613551
## 810       Asia                   Jordan    1937652
## 811       Asia                   Jordan    2347031
## 812       Asia                   Jordan    2820042
## 813       Asia                   Jordan    3867409
## 814       Asia                   Jordan    4526235
## 815       Asia                   Jordan    5307470
## 816       Asia                   Jordan    6053193
## 817     Africa                    Kenya    6464046
## 818     Africa                    Kenya    7454779
## 819     Africa                    Kenya    8678557
## 820     Africa                    Kenya   10191512
## 821     Africa                    Kenya   12044785
## 822     Africa                    Kenya   14500404
## 823     Africa                    Kenya   17661452
## 824     Africa                    Kenya   21198082
## 825     Africa                    Kenya   25020539
## 826     Africa                    Kenya   28263827
## 827     Africa                    Kenya   31386842
## 828     Africa                    Kenya   35610177
## 829       Asia          Korea Dem. Rep.    8865488
## 830       Asia          Korea Dem. Rep.    9411381
## 831       Asia          Korea Dem. Rep.   10917494
## 832       Asia          Korea Dem. Rep.   12617009
## 833       Asia          Korea Dem. Rep.   14781241
## 834       Asia          Korea Dem. Rep.   16325320
## 835       Asia          Korea Dem. Rep.   17647518
## 836       Asia          Korea Dem. Rep.   19067554
## 837       Asia          Korea Dem. Rep.   20711375
## 838       Asia          Korea Dem. Rep.   21585105
## 839       Asia          Korea Dem. Rep.   22215365
## 840       Asia          Korea Dem. Rep.   23301725
## 841       Asia               Korea Rep.   20947571
## 842       Asia               Korea Rep.   22611552
## 843       Asia               Korea Rep.   26420307
## 844       Asia               Korea Rep.   30131000
## 845       Asia               Korea Rep.   33505000
## 846       Asia               Korea Rep.   36436000
## 847       Asia               Korea Rep.   39326000
## 848       Asia               Korea Rep.   41622000
## 849       Asia               Korea Rep.   43805450
## 850       Asia               Korea Rep.   46173816
## 851       Asia               Korea Rep.   47969150
## 852       Asia               Korea Rep.   49044790
## 853       Asia                   Kuwait     160000
## 854       Asia                   Kuwait     212846
## 855       Asia                   Kuwait     358266
## 856       Asia                   Kuwait     575003
## 857       Asia                   Kuwait     841934
## 858       Asia                   Kuwait    1140357
## 859       Asia                   Kuwait    1497494
## 860       Asia                   Kuwait    1891487
## 861       Asia                   Kuwait    1418095
## 862       Asia                   Kuwait    1765345
## 863       Asia                   Kuwait    2111561
## 864       Asia                   Kuwait    2505559
## 865       Asia                  Lebanon    1439529
## 866       Asia                  Lebanon    1647412
## 867       Asia                  Lebanon    1886848
## 868       Asia                  Lebanon    2186894
## 869       Asia                  Lebanon    2680018
## 870       Asia                  Lebanon    3115787
## 871       Asia                  Lebanon    3086876
## 872       Asia                  Lebanon    3089353
## 873       Asia                  Lebanon    3219994
## 874       Asia                  Lebanon    3430388
## 875       Asia                  Lebanon    3677780
## 876       Asia                  Lebanon    3921278
## 877     Africa                  Lesotho     748747
## 878     Africa                  Lesotho     813338
## 879     Africa                  Lesotho     893143
## 880     Africa                  Lesotho     996380
## 881     Africa                  Lesotho    1116779
## 882     Africa                  Lesotho    1251524
## 883     Africa                  Lesotho    1411807
## 884     Africa                  Lesotho    1599200
## 885     Africa                  Lesotho    1803195
## 886     Africa                  Lesotho    1982823
## 887     Africa                  Lesotho    2046772
## 888     Africa                  Lesotho    2012649
## 889     Africa                  Liberia     863308
## 890     Africa                  Liberia     975950
## 891     Africa                  Liberia    1112796
## 892     Africa                  Liberia    1279406
## 893     Africa                  Liberia    1482628
## 894     Africa                  Liberia    1703617
## 895     Africa                  Liberia    1956875
## 896     Africa                  Liberia    2269414
## 897     Africa                  Liberia    1912974
## 898     Africa                  Liberia    2200725
## 899     Africa                  Liberia    2814651
## 900     Africa                  Liberia    3193942
## 901     Africa                    Libya    1019729
## 902     Africa                    Libya    1201578
## 903     Africa                    Libya    1441863
## 904     Africa                    Libya    1759224
## 905     Africa                    Libya    2183877
## 906     Africa                    Libya    2721783
## 907     Africa                    Libya    3344074
## 908     Africa                    Libya    3799845
## 909     Africa                    Libya    4364501
## 910     Africa                    Libya    4759670
## 911     Africa                    Libya    5368585
## 912     Africa                    Libya    6036914
## 913     Africa               Madagascar    4762912
## 914     Africa               Madagascar    5181679
## 915     Africa               Madagascar    5703324
## 916     Africa               Madagascar    6334556
## 917     Africa               Madagascar    7082430
## 918     Africa               Madagascar    8007166
## 919     Africa               Madagascar    9171477
## 920     Africa               Madagascar   10568642
## 921     Africa               Madagascar   12210395
## 922     Africa               Madagascar   14165114
## 923     Africa               Madagascar   16473477
## 924     Africa               Madagascar   19167654
## 925     Africa                   Malawi    2917802
## 926     Africa                   Malawi    3221238
## 927     Africa                   Malawi    3628608
## 928     Africa                   Malawi    4147252
## 929     Africa                   Malawi    4730997
## 930     Africa                   Malawi    5637246
## 931     Africa                   Malawi    6502825
## 932     Africa                   Malawi    7824747
## 933     Africa                   Malawi   10014249
## 934     Africa                   Malawi   10419991
## 935     Africa                   Malawi   11824495
## 936     Africa                   Malawi   13327079
## 937       Asia                 Malaysia    6748378
## 938       Asia                 Malaysia    7739235
## 939       Asia                 Malaysia    8906385
## 940       Asia                 Malaysia   10154878
## 941       Asia                 Malaysia   11441462
## 942       Asia                 Malaysia   12845381
## 943       Asia                 Malaysia   14441916
## 944       Asia                 Malaysia   16331785
## 945       Asia                 Malaysia   18319502
## 946       Asia                 Malaysia   20476091
## 947       Asia                 Malaysia   22662365
## 948       Asia                 Malaysia   24821286
## 949     Africa                     Mali    3838168
## 950     Africa                     Mali    4241884
## 951     Africa                     Mali    4690372
## 952     Africa                     Mali    5212416
## 953     Africa                     Mali    5828158
## 954     Africa                     Mali    6491649
## 955     Africa                     Mali    6998256
## 956     Africa                     Mali    7634008
## 957     Africa                     Mali    8416215
## 958     Africa                     Mali    9384984
## 959     Africa                     Mali   10580176
## 960     Africa                     Mali   12031795
## 961     Africa               Mauritania    1022556
## 962     Africa               Mauritania    1076852
## 963     Africa               Mauritania    1146757
## 964     Africa               Mauritania    1230542
## 965     Africa               Mauritania    1332786
## 966     Africa               Mauritania    1456688
## 967     Africa               Mauritania    1622136
## 968     Africa               Mauritania    1841240
## 969     Africa               Mauritania    2119465
## 970     Africa               Mauritania    2444741
## 971     Africa               Mauritania    2828858
## 972     Africa               Mauritania    3270065
## 973     Africa                Mauritius     516556
## 974     Africa                Mauritius     609816
## 975     Africa                Mauritius     701016
## 976     Africa                Mauritius     789309
## 977     Africa                Mauritius     851334
## 978     Africa                Mauritius     913025
## 979     Africa                Mauritius     992040
## 980     Africa                Mauritius    1042663
## 981     Africa                Mauritius    1096202
## 982     Africa                Mauritius    1149818
## 983     Africa                Mauritius    1200206
## 984     Africa                Mauritius    1250882
## 985   Americas                   Mexico   30144317
## 986   Americas                   Mexico   35015548
## 987   Americas                   Mexico   41121485
## 988   Americas                   Mexico   47995559
## 989   Americas                   Mexico   55984294
## 990   Americas                   Mexico   63759976
## 991   Americas                   Mexico   71640904
## 992   Americas                   Mexico   80122492
## 993   Americas                   Mexico   88111030
## 994   Americas                   Mexico   95895146
## 995   Americas                   Mexico  102479927
## 996   Americas                   Mexico  108700891
## 997       Asia                 Mongolia     800663
## 998       Asia                 Mongolia     882134
## 999       Asia                 Mongolia    1010280
## 1000      Asia                 Mongolia    1149500
## 1001      Asia                 Mongolia    1320500
## 1002      Asia                 Mongolia    1528000
## 1003      Asia                 Mongolia    1756032
## 1004      Asia                 Mongolia    2015133
## 1005      Asia                 Mongolia    2312802
## 1006      Asia                 Mongolia    2494803
## 1007      Asia                 Mongolia    2674234
## 1008      Asia                 Mongolia    2874127
## 1009    Europe               Montenegro     413834
## 1010    Europe               Montenegro     442829
## 1011    Europe               Montenegro     474528
## 1012    Europe               Montenegro     501035
## 1013    Europe               Montenegro     527678
## 1014    Europe               Montenegro     560073
## 1015    Europe               Montenegro     562548
## 1016    Europe               Montenegro     569473
## 1017    Europe               Montenegro     621621
## 1018    Europe               Montenegro     692651
## 1019    Europe               Montenegro     720230
## 1020    Europe               Montenegro     684736
## 1021    Africa                  Morocco    9939217
## 1022    Africa                  Morocco   11406350
## 1023    Africa                  Morocco   13056604
## 1024    Africa                  Morocco   14770296
## 1025    Africa                  Morocco   16660670
## 1026    Africa                  Morocco   18396941
## 1027    Africa                  Morocco   20198730
## 1028    Africa                  Morocco   22987397
## 1029    Africa                  Morocco   25798239
## 1030    Africa                  Morocco   28529501
## 1031    Africa                  Morocco   31167783
## 1032    Africa                  Morocco   33757175
## 1033    Africa               Mozambique    6446316
## 1034    Africa               Mozambique    7038035
## 1035    Africa               Mozambique    7788944
## 1036    Africa               Mozambique    8680909
## 1037    Africa               Mozambique    9809596
## 1038    Africa               Mozambique   11127868
## 1039    Africa               Mozambique   12587223
## 1040    Africa               Mozambique   12891952
## 1041    Africa               Mozambique   13160731
## 1042    Africa               Mozambique   16603334
## 1043    Africa               Mozambique   18473780
## 1044    Africa               Mozambique   19951656
## 1045      Asia                  Myanmar   20092996
## 1046      Asia                  Myanmar   21731844
## 1047      Asia                  Myanmar   23634436
## 1048      Asia                  Myanmar   25870271
## 1049      Asia                  Myanmar   28466390
## 1050      Asia                  Myanmar   31528087
## 1051      Asia                  Myanmar   34680442
## 1052      Asia                  Myanmar   38028578
## 1053      Asia                  Myanmar   40546538
## 1054      Asia                  Myanmar   43247867
## 1055      Asia                  Myanmar   45598081
## 1056      Asia                  Myanmar   47761980
## 1057    Africa                  Namibia     485831
## 1058    Africa                  Namibia     548080
## 1059    Africa                  Namibia     621392
## 1060    Africa                  Namibia     706640
## 1061    Africa                  Namibia     821782
## 1062    Africa                  Namibia     977026
## 1063    Africa                  Namibia    1099010
## 1064    Africa                  Namibia    1278184
## 1065    Africa                  Namibia    1554253
## 1066    Africa                  Namibia    1774766
## 1067    Africa                  Namibia    1972153
## 1068    Africa                  Namibia    2055080
## 1069      Asia                    Nepal    9182536
## 1070      Asia                    Nepal    9682338
## 1071      Asia                    Nepal   10332057
## 1072      Asia                    Nepal   11261690
## 1073      Asia                    Nepal   12412593
## 1074      Asia                    Nepal   13933198
## 1075      Asia                    Nepal   15796314
## 1076      Asia                    Nepal   17917180
## 1077      Asia                    Nepal   20326209
## 1078      Asia                    Nepal   23001113
## 1079      Asia                    Nepal   25873917
## 1080      Asia                    Nepal   28901790
## 1081    Europe              Netherlands   10381988
## 1082    Europe              Netherlands   11026383
## 1083    Europe              Netherlands   11805689
## 1084    Europe              Netherlands   12596822
## 1085    Europe              Netherlands   13329874
## 1086    Europe              Netherlands   13852989
## 1087    Europe              Netherlands   14310401
## 1088    Europe              Netherlands   14665278
## 1089    Europe              Netherlands   15174244
## 1090    Europe              Netherlands   15604464
## 1091    Europe              Netherlands   16122830
## 1092    Europe              Netherlands   16570613
## 1093   Oceania              New Zealand    1994794
## 1094   Oceania              New Zealand    2229407
## 1095   Oceania              New Zealand    2488550
## 1096   Oceania              New Zealand    2728150
## 1097   Oceania              New Zealand    2929100
## 1098   Oceania              New Zealand    3164900
## 1099   Oceania              New Zealand    3210650
## 1100   Oceania              New Zealand    3317166
## 1101   Oceania              New Zealand    3437674
## 1102   Oceania              New Zealand    3676187
## 1103   Oceania              New Zealand    3908037
## 1104   Oceania              New Zealand    4115771
## 1105  Americas                Nicaragua    1165790
## 1106  Americas                Nicaragua    1358828
## 1107  Americas                Nicaragua    1590597
## 1108  Americas                Nicaragua    1865490
## 1109  Americas                Nicaragua    2182908
## 1110  Americas                Nicaragua    2554598
## 1111  Americas                Nicaragua    2979423
## 1112  Americas                Nicaragua    3344353
## 1113  Americas                Nicaragua    4017939
## 1114  Americas                Nicaragua    4609572
## 1115  Americas                Nicaragua    5146848
## 1116  Americas                Nicaragua    5675356
## 1117    Africa                    Niger    3379468
## 1118    Africa                    Niger    3692184
## 1119    Africa                    Niger    4076008
## 1120    Africa                    Niger    4534062
## 1121    Africa                    Niger    5060262
## 1122    Africa                    Niger    5682086
## 1123    Africa                    Niger    6437188
## 1124    Africa                    Niger    7332638
## 1125    Africa                    Niger    8392818
## 1126    Africa                    Niger    9666252
## 1127    Africa                    Niger   11140655
## 1128    Africa                    Niger   12894865
## 1129    Africa                  Nigeria   33119096
## 1130    Africa                  Nigeria   37173340
## 1131    Africa                  Nigeria   41871351
## 1132    Africa                  Nigeria   47287752
## 1133    Africa                  Nigeria   53740085
## 1134    Africa                  Nigeria   62209173
## 1135    Africa                  Nigeria   73039376
## 1136    Africa                  Nigeria   81551520
## 1137    Africa                  Nigeria   93364244
## 1138    Africa                  Nigeria  106207839
## 1139    Africa                  Nigeria  119901274
## 1140    Africa                  Nigeria  135031164
## 1141    Europe                   Norway    3327728
## 1142    Europe                   Norway    3491938
## 1143    Europe                   Norway    3638919
## 1144    Europe                   Norway    3786019
## 1145    Europe                   Norway    3933004
## 1146    Europe                   Norway    4043205
## 1147    Europe                   Norway    4114787
## 1148    Europe                   Norway    4186147
## 1149    Europe                   Norway    4286357
## 1150    Europe                   Norway    4405672
## 1151    Europe                   Norway    4535591
## 1152    Europe                   Norway    4627926
## 1153      Asia                     Oman     507833
## 1154      Asia                     Oman     561977
## 1155      Asia                     Oman     628164
## 1156      Asia                     Oman     714775
## 1157      Asia                     Oman     829050
## 1158      Asia                     Oman    1004533
## 1159      Asia                     Oman    1301048
## 1160      Asia                     Oman    1593882
## 1161      Asia                     Oman    1915208
## 1162      Asia                     Oman    2283635
## 1163      Asia                     Oman    2713462
## 1164      Asia                     Oman    3204897
## 1165      Asia                 Pakistan   41346560
## 1166      Asia                 Pakistan   46679944
## 1167      Asia                 Pakistan   53100671
## 1168      Asia                 Pakistan   60641899
## 1169      Asia                 Pakistan   69325921
## 1170      Asia                 Pakistan   78152686
## 1171      Asia                 Pakistan   91462088
## 1172      Asia                 Pakistan  105186881
## 1173      Asia                 Pakistan  120065004
## 1174      Asia                 Pakistan  135564834
## 1175      Asia                 Pakistan  153403524
## 1176      Asia                 Pakistan  169270617
## 1177  Americas                   Panama     940080
## 1178  Americas                   Panama    1063506
## 1179  Americas                   Panama    1215725
## 1180  Americas                   Panama    1405486
## 1181  Americas                   Panama    1616384
## 1182  Americas                   Panama    1839782
## 1183  Americas                   Panama    2036305
## 1184  Americas                   Panama    2253639
## 1185  Americas                   Panama    2484997
## 1186  Americas                   Panama    2734531
## 1187  Americas                   Panama    2990875
## 1188  Americas                   Panama    3242173
## 1189  Americas                 Paraguay    1555876
## 1190  Americas                 Paraguay    1770902
## 1191  Americas                 Paraguay    2009813
## 1192  Americas                 Paraguay    2287985
## 1193  Americas                 Paraguay    2614104
## 1194  Americas                 Paraguay    2984494
## 1195  Americas                 Paraguay    3366439
## 1196  Americas                 Paraguay    3886512
## 1197  Americas                 Paraguay    4483945
## 1198  Americas                 Paraguay    5154123
## 1199  Americas                 Paraguay    5884491
## 1200  Americas                 Paraguay    6667147
## 1201  Americas                     Peru    8025700
## 1202  Americas                     Peru    9146100
## 1203  Americas                     Peru   10516500
## 1204  Americas                     Peru   12132200
## 1205  Americas                     Peru   13954700
## 1206  Americas                     Peru   15990099
## 1207  Americas                     Peru   18125129
## 1208  Americas                     Peru   20195924
## 1209  Americas                     Peru   22430449
## 1210  Americas                     Peru   24748122
## 1211  Americas                     Peru   26769436
## 1212  Americas                     Peru   28674757
## 1213      Asia              Philippines   22438691
## 1214      Asia              Philippines   26072194
## 1215      Asia              Philippines   30325264
## 1216      Asia              Philippines   35356600
## 1217      Asia              Philippines   40850141
## 1218      Asia              Philippines   46850962
## 1219      Asia              Philippines   53456774
## 1220      Asia              Philippines   60017788
## 1221      Asia              Philippines   67185766
## 1222      Asia              Philippines   75012988
## 1223      Asia              Philippines   82995088
## 1224      Asia              Philippines   91077287
## 1225    Europe                   Poland   25730551
## 1226    Europe                   Poland   28235346
## 1227    Europe                   Poland   30329617
## 1228    Europe                   Poland   31785378
## 1229    Europe                   Poland   33039545
## 1230    Europe                   Poland   34621254
## 1231    Europe                   Poland   36227381
## 1232    Europe                   Poland   37740710
## 1233    Europe                   Poland   38370697
## 1234    Europe                   Poland   38654957
## 1235    Europe                   Poland   38625976
## 1236    Europe                   Poland   38518241
## 1237    Europe                 Portugal    8526050
## 1238    Europe                 Portugal    8817650
## 1239    Europe                 Portugal    9019800
## 1240    Europe                 Portugal    9103000
## 1241    Europe                 Portugal    8970450
## 1242    Europe                 Portugal    9662600
## 1243    Europe                 Portugal    9859650
## 1244    Europe                 Portugal    9915289
## 1245    Europe                 Portugal    9927680
## 1246    Europe                 Portugal   10156415
## 1247    Europe                 Portugal   10433867
## 1248    Europe                 Portugal   10642836
## 1249  Americas              Puerto Rico    2227000
## 1250  Americas              Puerto Rico    2260000
## 1251  Americas              Puerto Rico    2448046
## 1252  Americas              Puerto Rico    2648961
## 1253  Americas              Puerto Rico    2847132
## 1254  Americas              Puerto Rico    3080828
## 1255  Americas              Puerto Rico    3279001
## 1256  Americas              Puerto Rico    3444468
## 1257  Americas              Puerto Rico    3585176
## 1258  Americas              Puerto Rico    3759430
## 1259  Americas              Puerto Rico    3859606
## 1260  Americas              Puerto Rico    3942491
## 1261    Africa                  Reunion     257700
## 1262    Africa                  Reunion     308700
## 1263    Africa                  Reunion     358900
## 1264    Africa                  Reunion     414024
## 1265    Africa                  Reunion     461633
## 1266    Africa                  Reunion     492095
## 1267    Africa                  Reunion     517810
## 1268    Africa                  Reunion     562035
## 1269    Africa                  Reunion     622191
## 1270    Africa                  Reunion     684810
## 1271    Africa                  Reunion     743981
## 1272    Africa                  Reunion     798094
## 1273    Europe                  Romania   16630000
## 1274    Europe                  Romania   17829327
## 1275    Europe                  Romania   18680721
## 1276    Europe                  Romania   19284814
## 1277    Europe                  Romania   20662648
## 1278    Europe                  Romania   21658597
## 1279    Europe                  Romania   22356726
## 1280    Europe                  Romania   22686371
## 1281    Europe                  Romania   22797027
## 1282    Europe                  Romania   22562458
## 1283    Europe                  Romania   22404337
## 1284    Europe                  Romania   22276056
## 1285    Africa                   Rwanda    2534927
## 1286    Africa                   Rwanda    2822082
## 1287    Africa                   Rwanda    3051242
## 1288    Africa                   Rwanda    3451079
## 1289    Africa                   Rwanda    3992121
## 1290    Africa                   Rwanda    4657072
## 1291    Africa                   Rwanda    5507565
## 1292    Africa                   Rwanda    6349365
## 1293    Africa                   Rwanda    7290203
## 1294    Africa                   Rwanda    7212583
## 1295    Africa                   Rwanda    7852401
## 1296    Africa                   Rwanda    8860588
## 1297    Africa    Sao Tome and Principe      60011
## 1298    Africa    Sao Tome and Principe      61325
## 1299    Africa    Sao Tome and Principe      65345
## 1300    Africa    Sao Tome and Principe      70787
## 1301    Africa    Sao Tome and Principe      76595
## 1302    Africa    Sao Tome and Principe      86796
## 1303    Africa    Sao Tome and Principe      98593
## 1304    Africa    Sao Tome and Principe     110812
## 1305    Africa    Sao Tome and Principe     125911
## 1306    Africa    Sao Tome and Principe     145608
## 1307    Africa    Sao Tome and Principe     170372
## 1308    Africa    Sao Tome and Principe     199579
## 1309      Asia             Saudi Arabia    4005677
## 1310      Asia             Saudi Arabia    4419650
## 1311      Asia             Saudi Arabia    4943029
## 1312      Asia             Saudi Arabia    5618198
## 1313      Asia             Saudi Arabia    6472756
## 1314      Asia             Saudi Arabia    8128505
## 1315      Asia             Saudi Arabia   11254672
## 1316      Asia             Saudi Arabia   14619745
## 1317      Asia             Saudi Arabia   16945857
## 1318      Asia             Saudi Arabia   21229759
## 1319      Asia             Saudi Arabia   24501530
## 1320      Asia             Saudi Arabia   27601038
## 1321    Africa                  Senegal    2755589
## 1322    Africa                  Senegal    3054547
## 1323    Africa                  Senegal    3430243
## 1324    Africa                  Senegal    3965841
## 1325    Africa                  Senegal    4588696
## 1326    Africa                  Senegal    5260855
## 1327    Africa                  Senegal    6147783
## 1328    Africa                  Senegal    7171347
## 1329    Africa                  Senegal    8307920
## 1330    Africa                  Senegal    9535314
## 1331    Africa                  Senegal   10870037
## 1332    Africa                  Senegal   12267493
## 1333    Europe                   Serbia    6860147
## 1334    Europe                   Serbia    7271135
## 1335    Europe                   Serbia    7616060
## 1336    Europe                   Serbia    7971222
## 1337    Europe                   Serbia    8313288
## 1338    Europe                   Serbia    8686367
## 1339    Europe                   Serbia    9032824
## 1340    Europe                   Serbia    9230783
## 1341    Europe                   Serbia    9826397
## 1342    Europe                   Serbia   10336594
## 1343    Europe                   Serbia   10111559
## 1344    Europe                   Serbia   10150265
## 1345    Africa             Sierra Leone    2143249
## 1346    Africa             Sierra Leone    2295678
## 1347    Africa             Sierra Leone    2467895
## 1348    Africa             Sierra Leone    2662190
## 1349    Africa             Sierra Leone    2879013
## 1350    Africa             Sierra Leone    3140897
## 1351    Africa             Sierra Leone    3464522
## 1352    Africa             Sierra Leone    3868905
## 1353    Africa             Sierra Leone    4260884
## 1354    Africa             Sierra Leone    4578212
## 1355    Africa             Sierra Leone    5359092
## 1356    Africa             Sierra Leone    6144562
## 1357      Asia                Singapore    1127000
## 1358      Asia                Singapore    1445929
## 1359      Asia                Singapore    1750200
## 1360      Asia                Singapore    1977600
## 1361      Asia                Singapore    2152400
## 1362      Asia                Singapore    2325300
## 1363      Asia                Singapore    2651869
## 1364      Asia                Singapore    2794552
## 1365      Asia                Singapore    3235865
## 1366      Asia                Singapore    3802309
## 1367      Asia                Singapore    4197776
## 1368      Asia                Singapore    4553009
## 1369    Europe          Slovak Republic    3558137
## 1370    Europe          Slovak Republic    3844277
## 1371    Europe          Slovak Republic    4237384
## 1372    Europe          Slovak Republic    4442238
## 1373    Europe          Slovak Republic    4593433
## 1374    Europe          Slovak Republic    4827803
## 1375    Europe          Slovak Republic    5048043
## 1376    Europe          Slovak Republic    5199318
## 1377    Europe          Slovak Republic    5302888
## 1378    Europe          Slovak Republic    5383010
## 1379    Europe          Slovak Republic    5410052
## 1380    Europe          Slovak Republic    5447502
## 1381    Europe                 Slovenia    1489518
## 1382    Europe                 Slovenia    1533070
## 1383    Europe                 Slovenia    1582962
## 1384    Europe                 Slovenia    1646912
## 1385    Europe                 Slovenia    1694510
## 1386    Europe                 Slovenia    1746919
## 1387    Europe                 Slovenia    1861252
## 1388    Europe                 Slovenia    1945870
## 1389    Europe                 Slovenia    1999210
## 1390    Europe                 Slovenia    2011612
## 1391    Europe                 Slovenia    2011497
## 1392    Europe                 Slovenia    2009245
## 1393    Africa                  Somalia    2526994
## 1394    Africa                  Somalia    2780415
## 1395    Africa                  Somalia    3080153
## 1396    Africa                  Somalia    3428839
## 1397    Africa                  Somalia    3840161
## 1398    Africa                  Somalia    4353666
## 1399    Africa                  Somalia    5828892
## 1400    Africa                  Somalia    6921858
## 1401    Africa                  Somalia    6099799
## 1402    Africa                  Somalia    6633514
## 1403    Africa                  Somalia    7753310
## 1404    Africa                  Somalia    9118773
## 1405    Africa             South Africa   14264935
## 1406    Africa             South Africa   16151549
## 1407    Africa             South Africa   18356657
## 1408    Africa             South Africa   20997321
## 1409    Africa             South Africa   23935810
## 1410    Africa             South Africa   27129932
## 1411    Africa             South Africa   31140029
## 1412    Africa             South Africa   35933379
## 1413    Africa             South Africa   39964159
## 1414    Africa             South Africa   42835005
## 1415    Africa             South Africa   44433622
## 1416    Africa             South Africa   43997828
## 1417    Europe                    Spain   28549870
## 1418    Europe                    Spain   29841614
## 1419    Europe                    Spain   31158061
## 1420    Europe                    Spain   32850275
## 1421    Europe                    Spain   34513161
## 1422    Europe                    Spain   36439000
## 1423    Europe                    Spain   37983310
## 1424    Europe                    Spain   38880702
## 1425    Europe                    Spain   39549438
## 1426    Europe                    Spain   39855442
## 1427    Europe                    Spain   40152517
## 1428    Europe                    Spain   40448191
## 1429      Asia                Sri Lanka    7982342
## 1430      Asia                Sri Lanka    9128546
## 1431      Asia                Sri Lanka   10421936
## 1432      Asia                Sri Lanka   11737396
## 1433      Asia                Sri Lanka   13016733
## 1434      Asia                Sri Lanka   14116836
## 1435      Asia                Sri Lanka   15410151
## 1436      Asia                Sri Lanka   16495304
## 1437      Asia                Sri Lanka   17587060
## 1438      Asia                Sri Lanka   18698655
## 1439      Asia                Sri Lanka   19576783
## 1440      Asia                Sri Lanka   20378239
## 1441    Africa                    Sudan    8504667
## 1442    Africa                    Sudan    9753392
## 1443    Africa                    Sudan   11183227
## 1444    Africa                    Sudan   12716129
## 1445    Africa                    Sudan   14597019
## 1446    Africa                    Sudan   17104986
## 1447    Africa                    Sudan   20367053
## 1448    Africa                    Sudan   24725960
## 1449    Africa                    Sudan   28227588
## 1450    Africa                    Sudan   32160729
## 1451    Africa                    Sudan   37090298
## 1452    Africa                    Sudan   42292929
## 1453    Africa                Swaziland     290243
## 1454    Africa                Swaziland     326741
## 1455    Africa                Swaziland     370006
## 1456    Africa                Swaziland     420690
## 1457    Africa                Swaziland     480105
## 1458    Africa                Swaziland     551425
## 1459    Africa                Swaziland     649901
## 1460    Africa                Swaziland     779348
## 1461    Africa                Swaziland     962344
## 1462    Africa                Swaziland    1054486
## 1463    Africa                Swaziland    1130269
## 1464    Africa                Swaziland    1133066
## 1465    Europe                   Sweden    7124673
## 1466    Europe                   Sweden    7363802
## 1467    Europe                   Sweden    7561588
## 1468    Europe                   Sweden    7867931
## 1469    Europe                   Sweden    8122293
## 1470    Europe                   Sweden    8251648
## 1471    Europe                   Sweden    8325260
## 1472    Europe                   Sweden    8421403
## 1473    Europe                   Sweden    8718867
## 1474    Europe                   Sweden    8897619
## 1475    Europe                   Sweden    8954175
## 1476    Europe                   Sweden    9031088
## 1477    Europe              Switzerland    4815000
## 1478    Europe              Switzerland    5126000
## 1479    Europe              Switzerland    5666000
## 1480    Europe              Switzerland    6063000
## 1481    Europe              Switzerland    6401400
## 1482    Europe              Switzerland    6316424
## 1483    Europe              Switzerland    6468126
## 1484    Europe              Switzerland    6649942
## 1485    Europe              Switzerland    6995447
## 1486    Europe              Switzerland    7193761
## 1487    Europe              Switzerland    7361757
## 1488    Europe              Switzerland    7554661
## 1489      Asia                    Syria    3661549
## 1490      Asia                    Syria    4149908
## 1491      Asia                    Syria    4834621
## 1492      Asia                    Syria    5680812
## 1493      Asia                    Syria    6701172
## 1494      Asia                    Syria    7932503
## 1495      Asia                    Syria    9410494
## 1496      Asia                    Syria   11242847
## 1497      Asia                    Syria   13219062
## 1498      Asia                    Syria   15081016
## 1499      Asia                    Syria   17155814
## 1500      Asia                    Syria   19314747
## 1501      Asia                   Taiwan    8550362
## 1502      Asia                   Taiwan   10164215
## 1503      Asia                   Taiwan   11918938
## 1504      Asia                   Taiwan   13648692
## 1505      Asia                   Taiwan   15226039
## 1506      Asia                   Taiwan   16785196
## 1507      Asia                   Taiwan   18501390
## 1508      Asia                   Taiwan   19757799
## 1509      Asia                   Taiwan   20686918
## 1510      Asia                   Taiwan   21628605
## 1511      Asia                   Taiwan   22454239
## 1512      Asia                   Taiwan   23174294
## 1513    Africa                 Tanzania    8322925
## 1514    Africa                 Tanzania    9452826
## 1515    Africa                 Tanzania   10863958
## 1516    Africa                 Tanzania   12607312
## 1517    Africa                 Tanzania   14706593
## 1518    Africa                 Tanzania   17129565
## 1519    Africa                 Tanzania   19844382
## 1520    Africa                 Tanzania   23040630
## 1521    Africa                 Tanzania   26605473
## 1522    Africa                 Tanzania   30686889
## 1523    Africa                 Tanzania   34593779
## 1524    Africa                 Tanzania   38139640
## 1525      Asia                 Thailand   21289402
## 1526      Asia                 Thailand   25041917
## 1527      Asia                 Thailand   29263397
## 1528      Asia                 Thailand   34024249
## 1529      Asia                 Thailand   39276153
## 1530      Asia                 Thailand   44148285
## 1531      Asia                 Thailand   48827160
## 1532      Asia                 Thailand   52910342
## 1533      Asia                 Thailand   56667095
## 1534      Asia                 Thailand   60216677
## 1535      Asia                 Thailand   62806748
## 1536      Asia                 Thailand   65068149
## 1537    Africa                     Togo    1219113
## 1538    Africa                     Togo    1357445
## 1539    Africa                     Togo    1528098
## 1540    Africa                     Togo    1735550
## 1541    Africa                     Togo    2056351
## 1542    Africa                     Togo    2308582
## 1543    Africa                     Togo    2644765
## 1544    Africa                     Togo    3154264
## 1545    Africa                     Togo    3747553
## 1546    Africa                     Togo    4320890
## 1547    Africa                     Togo    4977378
## 1548    Africa                     Togo    5701579
## 1549  Americas      Trinidad and Tobago     662850
## 1550  Americas      Trinidad and Tobago     764900
## 1551  Americas      Trinidad and Tobago     887498
## 1552  Americas      Trinidad and Tobago     960155
## 1553  Americas      Trinidad and Tobago     975199
## 1554  Americas      Trinidad and Tobago    1039009
## 1555  Americas      Trinidad and Tobago    1116479
## 1556  Americas      Trinidad and Tobago    1191336
## 1557  Americas      Trinidad and Tobago    1183669
## 1558  Americas      Trinidad and Tobago    1138101
## 1559  Americas      Trinidad and Tobago    1101832
## 1560  Americas      Trinidad and Tobago    1056608
## 1561    Africa                  Tunisia    3647735
## 1562    Africa                  Tunisia    3950849
## 1563    Africa                  Tunisia    4286552
## 1564    Africa                  Tunisia    4786986
## 1565    Africa                  Tunisia    5303507
## 1566    Africa                  Tunisia    6005061
## 1567    Africa                  Tunisia    6734098
## 1568    Africa                  Tunisia    7724976
## 1569    Africa                  Tunisia    8523077
## 1570    Africa                  Tunisia    9231669
## 1571    Africa                  Tunisia    9770575
## 1572    Africa                  Tunisia   10276158
## 1573    Europe                   Turkey   22235677
## 1574    Europe                   Turkey   25670939
## 1575    Europe                   Turkey   29788695
## 1576    Europe                   Turkey   33411317
## 1577    Europe                   Turkey   37492953
## 1578    Europe                   Turkey   42404033
## 1579    Europe                   Turkey   47328791
## 1580    Europe                   Turkey   52881328
## 1581    Europe                   Turkey   58179144
## 1582    Europe                   Turkey   63047647
## 1583    Europe                   Turkey   67308928
## 1584    Europe                   Turkey   71158647
## 1585    Africa                   Uganda    5824797
## 1586    Africa                   Uganda    6675501
## 1587    Africa                   Uganda    7688797
## 1588    Africa                   Uganda    8900294
## 1589    Africa                   Uganda   10190285
## 1590    Africa                   Uganda   11457758
## 1591    Africa                   Uganda   12939400
## 1592    Africa                   Uganda   15283050
## 1593    Africa                   Uganda   18252190
## 1594    Africa                   Uganda   21210254
## 1595    Africa                   Uganda   24739869
## 1596    Africa                   Uganda   29170398
## 1597    Europe           United Kingdom   50430000
## 1598    Europe           United Kingdom   51430000
## 1599    Europe           United Kingdom   53292000
## 1600    Europe           United Kingdom   54959000
## 1601    Europe           United Kingdom   56079000
## 1602    Europe           United Kingdom   56179000
## 1603    Europe           United Kingdom   56339704
## 1604    Europe           United Kingdom   56981620
## 1605    Europe           United Kingdom   57866349
## 1606    Europe           United Kingdom   58808266
## 1607    Europe           United Kingdom   59912431
## 1608    Europe           United Kingdom   60776238
## 1609  Americas            United States  157553000
## 1610  Americas            United States  171984000
## 1611  Americas            United States  186538000
## 1612  Americas            United States  198712000
## 1613  Americas            United States  209896000
## 1614  Americas            United States  220239000
## 1615  Americas            United States  232187835
## 1616  Americas            United States  242803533
## 1617  Americas            United States  256894189
## 1618  Americas            United States  272911760
## 1619  Americas            United States  287675526
## 1620  Americas            United States  301139947
## 1621  Americas                  Uruguay    2252965
## 1622  Americas                  Uruguay    2424959
## 1623  Americas                  Uruguay    2598466
## 1624  Americas                  Uruguay    2748579
## 1625  Americas                  Uruguay    2829526
## 1626  Americas                  Uruguay    2873520
## 1627  Americas                  Uruguay    2953997
## 1628  Americas                  Uruguay    3045153
## 1629  Americas                  Uruguay    3149262
## 1630  Americas                  Uruguay    3262838
## 1631  Americas                  Uruguay    3363085
## 1632  Americas                  Uruguay    3447496
## 1633  Americas                Venezuela    5439568
## 1634  Americas                Venezuela    6702668
## 1635  Americas                Venezuela    8143375
## 1636  Americas                Venezuela    9709552
## 1637  Americas                Venezuela   11515649
## 1638  Americas                Venezuela   13503563
## 1639  Americas                Venezuela   15620766
## 1640  Americas                Venezuela   17910182
## 1641  Americas                Venezuela   20265563
## 1642  Americas                Venezuela   22374398
## 1643  Americas                Venezuela   24287670
## 1644  Americas                Venezuela   26084662
## 1645      Asia                  Vietnam   26246839
## 1646      Asia                  Vietnam   28998543
## 1647      Asia                  Vietnam   33796140
## 1648      Asia                  Vietnam   39463910
## 1649      Asia                  Vietnam   44655014
## 1650      Asia                  Vietnam   50533506
## 1651      Asia                  Vietnam   56142181
## 1652      Asia                  Vietnam   62826491
## 1653      Asia                  Vietnam   69940728
## 1654      Asia                  Vietnam   76048996
## 1655      Asia                  Vietnam   80908147
## 1656      Asia                  Vietnam   85262356
## 1657      Asia       West Bank and Gaza    1030585
## 1658      Asia       West Bank and Gaza    1070439
## 1659      Asia       West Bank and Gaza    1133134
## 1660      Asia       West Bank and Gaza    1142636
## 1661      Asia       West Bank and Gaza    1089572
## 1662      Asia       West Bank and Gaza    1261091
## 1663      Asia       West Bank and Gaza    1425876
## 1664      Asia       West Bank and Gaza    1691210
## 1665      Asia       West Bank and Gaza    2104779
## 1666      Asia       West Bank and Gaza    2826046
## 1667      Asia       West Bank and Gaza    3389578
## 1668      Asia       West Bank and Gaza    4018332
## 1669      Asia               Yemen Rep.    4963829
## 1670      Asia               Yemen Rep.    5498090
## 1671      Asia               Yemen Rep.    6120081
## 1672      Asia               Yemen Rep.    6740785
## 1673      Asia               Yemen Rep.    7407075
## 1674      Asia               Yemen Rep.    8403990
## 1675      Asia               Yemen Rep.    9657618
## 1676      Asia               Yemen Rep.   11219340
## 1677      Asia               Yemen Rep.   13367997
## 1678      Asia               Yemen Rep.   15826497
## 1679      Asia               Yemen Rep.   18701257
## 1680      Asia               Yemen Rep.   22211743
## 1681    Africa                   Zambia    2672000
## 1682    Africa                   Zambia    3016000
## 1683    Africa                   Zambia    3421000
## 1684    Africa                   Zambia    3900000
## 1685    Africa                   Zambia    4506497
## 1686    Africa                   Zambia    5216550
## 1687    Africa                   Zambia    6100407
## 1688    Africa                   Zambia    7272406
## 1689    Africa                   Zambia    8381163
## 1690    Africa                   Zambia    9417789
## 1691    Africa                   Zambia   10595811
## 1692    Africa                   Zambia   11746035
## 1693    Africa                 Zimbabwe    3080907
## 1694    Africa                 Zimbabwe    3646340
## 1695    Africa                 Zimbabwe    4277736
## 1696    Africa                 Zimbabwe    4995432
## 1697    Africa                 Zimbabwe    5861135
## 1698    Africa                 Zimbabwe    6642107
## 1699    Africa                 Zimbabwe    7636524
## 1700    Africa                 Zimbabwe    9216418
## 1701    Africa                 Zimbabwe   10704340
## 1702    Africa                 Zimbabwe   11404948
## 1703    Africa                 Zimbabwe   11926563
## 1704    Africa                 Zimbabwe   12311143
```

We can also rename columns in the output using the list:


```r
gap[, list(a=continent, b=country, c=pop)] # This does not alter the data table
```

```
##            a           b        c
##    1:   Asia Afghanistan  8425333
##    2:   Asia Afghanistan  9240934
##    3:   Asia Afghanistan 10267083
##    4:   Asia Afghanistan 11537966
##    5:   Asia Afghanistan 13079460
##   ---                            
## 1700: Africa    Zimbabwe  9216418
## 1701: Africa    Zimbabwe 10704340
## 1702: Africa    Zimbabwe 11404948
## 1703: Africa    Zimbabwe 11926563
## 1704: Africa    Zimbabwe 12311143
```

We can create temporary columns, those that only exist in the output data 
structure using list arguments:


```r
# total_gdp only exists in the output
gap[,list(continent, country, year, total_gdp=pop*gdpPercap)]
```

```
##       continent     country year  total_gdp
##    1:      Asia Afghanistan 1952 6567086330
##    2:      Asia Afghanistan 1957 7585448670
##    3:      Asia Afghanistan 1962 8758855797
##    4:      Asia Afghanistan 1967 9648014150
##    5:      Asia Afghanistan 1972 9678553274
##   ---                                      
## 1700:    Africa    Zimbabwe 1987 6508240905
## 1701:    Africa    Zimbabwe 1992 7422611852
## 1702:    Africa    Zimbabwe 1997 9037850590
## 1703:    Africa    Zimbabwe 2002 8015110972
## 1704:    Africa    Zimbabwe 2007 5782658337
```

```r
# Lets see what gap contains again:
gap
```

```
##           country year      pop continent lifeExp gdpPercap
##    1: Afghanistan 1952  8425333      Asia  28.801  779.4453
##    2: Afghanistan 1957  9240934      Asia  30.332  820.8530
##    3: Afghanistan 1962 10267083      Asia  31.997  853.1007
##    4: Afghanistan 1967 11537966      Asia  34.020  836.1971
##    5: Afghanistan 1972 13079460      Asia  36.088  739.9811
##   ---                                                      
## 1700:    Zimbabwe 1987  9216418    Africa  62.351  706.1573
## 1701:    Zimbabwe 1992 10704340    Africa  60.377  693.4208
## 1702:    Zimbabwe 1997 11404948    Africa  46.809  792.4500
## 1703:    Zimbabwe 2002 11926563    Africa  39.989  672.0386
## 1704:    Zimbabwe 2007 12311143    Africa  43.487  469.7093
```

```r
# The equivalent data frame call:
cbind(gap_df[,c("continent", "country", "year")], total_gdp = gap_df$pop * gap_df$gdpPercap)
```

```
##      continent                  country year    total_gdp
## 1         Asia              Afghanistan 1952 6.567086e+09
## 2         Asia              Afghanistan 1957 7.585449e+09
## 3         Asia              Afghanistan 1962 8.758856e+09
## 4         Asia              Afghanistan 1967 9.648014e+09
## 5         Asia              Afghanistan 1972 9.678553e+09
## 6         Asia              Afghanistan 1977 1.169766e+10
## 7         Asia              Afghanistan 1982 1.259856e+10
## 8         Asia              Afghanistan 1987 1.182099e+10
## 9         Asia              Afghanistan 1992 1.059590e+10
## 10        Asia              Afghanistan 1997 1.412200e+10
## 11        Asia              Afghanistan 2002 1.836341e+10
## 12        Asia              Afghanistan 2007 3.107929e+10
## 13      Europe                  Albania 1952 2.053670e+09
## 14      Europe                  Albania 1957 2.867792e+09
## 15      Europe                  Albania 1962 3.996989e+09
## 16      Europe                  Albania 1967 5.476396e+09
## 17      Europe                  Albania 1972 7.500110e+09
## 18      Europe                  Albania 1977 8.864476e+09
## 19      Europe                  Albania 1982 1.009420e+10
## 20      Europe                  Albania 1987 1.149842e+10
## 21      Europe                  Albania 1992 8.307722e+09
## 22      Europe                  Albania 1997 1.094591e+10
## 23      Europe                  Albania 2002 1.615393e+10
## 24      Europe                  Albania 2007 2.137641e+10
## 25      Africa                  Algeria 1952 2.272563e+10
## 26      Africa                  Algeria 1957 3.095611e+10
## 27      Africa                  Algeria 1962 2.806140e+10
## 28      Africa                  Algeria 1967 4.143324e+10
## 29      Africa                  Algeria 1972 6.173941e+10
## 30      Africa                  Algeria 1977 8.422742e+10
## 31      Africa                  Algeria 1982 1.150971e+11
## 32      Africa                  Algeria 1987 1.321197e+11
## 33      Africa                  Algeria 1992 1.321024e+11
## 34      Africa                  Algeria 1997 1.394670e+11
## 35      Africa                  Algeria 2002 1.654477e+11
## 36      Africa                  Algeria 2007 2.074449e+11
## 37      Africa                   Angola 1952 1.489956e+10
## 38      Africa                   Angola 1957 1.746062e+10
## 39      Africa                   Angola 1962 2.060359e+10
## 40      Africa                   Angola 1967 2.898060e+10
## 41      Africa                   Angola 1972 3.226426e+10
## 42      Africa                   Angola 1977 1.854132e+10
## 43      Africa                   Angola 1982 1.934385e+10
## 44      Africa                   Angola 1987 1.913602e+10
## 45      Africa                   Angola 1992 2.295683e+10
## 46      Africa                   Angola 1997 2.248682e+10
## 47      Africa                   Angola 2002 3.013483e+10
## 48      Africa                   Angola 2007 5.958390e+10
## 49    Americas                Argentina 1952 1.056763e+11
## 50    Americas                Argentina 1957 1.344666e+11
## 51    Americas                Argentina 1962 1.518208e+11
## 52    Americas                Argentina 1967 1.846882e+11
## 53    Americas                Argentina 1972 2.339966e+11
## 54    Americas                Argentina 1977 2.719707e+11
## 55    Americas                Argentina 1982 2.640107e+11
## 56    Americas                Argentina 1987 2.890048e+11
## 57    Americas                Argentina 1992 3.161041e+11
## 58    Americas                Argentina 1997 3.970536e+11
## 59    Americas                Argentina 2002 3.372234e+11
## 60    Americas                Argentina 2007 5.150336e+11
## 61     Oceania                Australia 1952 8.725625e+10
## 62     Oceania                Australia 1957 1.063492e+11
## 63     Oceania                Australia 1962 1.318846e+11
## 64     Oceania                Australia 1967 1.724580e+11
## 65     Oceania                Australia 1972 2.212238e+11
## 66     Oceania                Australia 1977 2.580373e+11
## 67     Oceania                Australia 1982 2.957428e+11
## 68     Oceania                Australia 1987 3.558531e+11
## 69     Oceania                Australia 1992 4.095112e+11
## 70     Oceania                Australia 1997 5.012233e+11
## 71     Oceania                Australia 2002 5.998472e+11
## 72     Oceania                Australia 2007 7.036584e+11
## 73      Europe                  Austria 1952 4.251627e+10
## 74      Europe                  Austria 1957 6.159630e+10
## 75      Europe                  Austria 1962 7.665118e+10
## 76      Europe                  Austria 1967 9.468084e+10
## 77      Europe                  Austria 1972 1.256987e+11
## 78      Europe                  Austria 1977 1.494721e+11
## 79      Europe                  Austria 1982 1.635896e+11
## 80      Europe                  Austria 1987 1.795277e+11
## 81      Europe                  Austria 1992 2.140367e+11
## 82      Europe                  Austria 1997 2.348005e+11
## 83      Europe                  Austria 2002 2.641488e+11
## 84      Europe                  Austria 2007 2.962294e+11
## 85        Asia                  Bahrain 1952 1.188461e+09
## 86        Asia                  Bahrain 1957 1.613362e+09
## 87        Asia                  Bahrain 1962 2.191816e+09
## 88        Asia                  Bahrain 1967 2.993238e+09
## 89        Asia                  Bahrain 1972 4.216406e+09
## 90        Asia                  Bahrain 1977 5.751940e+09
## 91        Asia                  Bahrain 1982 7.261180e+09
## 92        Asia                  Bahrain 1987 8.421244e+09
## 93        Asia                  Bahrain 1992 1.007917e+10
## 94        Asia                  Bahrain 1997 1.214601e+10
## 95        Asia                  Bahrain 2002 1.536203e+10
## 96        Asia                  Bahrain 2007 2.111268e+10
## 97        Asia               Bangladesh 1952 3.208206e+10
## 98        Asia               Bangladesh 1957 3.398532e+10
## 99        Asia               Bangladesh 1962 3.901117e+10
## 100       Asia               Bangladesh 1967 4.530627e+10
## 101       Asia               Bangladesh 1972 4.459489e+10
## 102       Asia               Bangladesh 1977 5.307281e+10
## 103       Asia               Bangladesh 1982 6.300969e+10
## 104       Asia               Bangladesh 1987 7.802857e+10
## 105       Asia               Bangladesh 1992 9.526285e+10
## 106       Asia               Bangladesh 1997 1.199574e+11
## 107       Asia               Bangladesh 2002 1.541591e+11
## 108       Asia               Bangladesh 2007 2.093118e+11
## 109     Europe                  Belgium 1952 7.283869e+10
## 110     Europe                  Belgium 1957 8.732886e+10
## 111     Europe                  Belgium 1962 1.013213e+11
## 112     Europe                  Belgium 1967 1.256588e+11
## 113     Europe                  Belgium 1972 1.618715e+11
## 114     Europe                  Belgium 1977 1.877729e+11
## 115     Europe                  Belgium 1982 2.067837e+11
## 116     Europe                  Belgium 1987 2.223318e+11
## 117     Europe                  Belgium 1992 2.569225e+11
## 118     Europe                  Belgium 1997 2.811183e+11
## 119     Europe                  Belgium 2002 3.143695e+11
## 120     Europe                  Belgium 2007 3.501412e+11
## 121     Africa                    Benin 1952 1.847398e+09
## 122     Africa                    Benin 1957 1.847398e+09
## 123     Africa                    Benin 1962 2.043222e+09
## 124     Africa                    Benin 1967 2.514309e+09
## 125     Africa                    Benin 1972 2.998327e+09
## 126     Africa                    Benin 1977 3.260658e+09
## 127     Africa                    Benin 1982 4.653596e+09
## 128     Africa                    Benin 1987 5.202273e+09
## 129     Africa                    Benin 1992 5.934205e+09
## 130     Africa                    Benin 1997 7.479327e+09
## 131     Africa                    Benin 2002 9.645995e+09
## 132     Africa                    Benin 2007 1.164315e+10
## 133   Americas                  Bolivia 1952 7.719575e+09
## 134   Americas                  Bolivia 1957 6.833571e+09
## 135   Americas                  Bolivia 1962 7.838236e+09
## 136   Americas                  Bolivia 1967 1.045274e+10
## 137   Americas                  Bolivia 1972 1.360781e+10
## 138   Americas                  Bolivia 1977 1.802333e+10
## 139   Americas                  Bolivia 1982 1.780974e+10
## 140   Americas                  Bolivia 1987 1.695274e+10
## 141   Americas                  Bolivia 1992 2.041633e+10
## 142   Americas                  Bolivia 1997 2.558864e+10
## 143   Americas                  Bolivia 2002 2.882546e+10
## 144   Americas                  Bolivia 2007 3.485465e+10
## 145     Europe   Bosnia and Herzegovina 1952 2.717131e+09
## 146     Europe   Bosnia and Herzegovina 1957 4.164871e+09
## 147     Europe   Bosnia and Herzegovina 1962 5.725731e+09
## 148     Europe   Bosnia and Herzegovina 1967 7.787883e+09
## 149     Europe   Bosnia and Herzegovina 1972 1.092299e+10
## 150     Europe   Bosnia and Herzegovina 1977 1.441737e+10
## 151     Europe   Bosnia and Herzegovina 1982 1.721909e+10
## 152     Europe   Bosnia and Herzegovina 1987 1.871884e+10
## 153     Europe   Bosnia and Herzegovina 1992 1.083913e+10
## 154     Europe   Bosnia and Herzegovina 1997 1.719225e+10
## 155     Europe   Bosnia and Herzegovina 2002 2.507154e+10
## 156     Europe   Bosnia and Herzegovina 2007 3.389703e+10
## 157     Africa                 Botswana 1952 3.765108e+08
## 158     Africa                 Botswana 1957 4.358290e+08
## 159     Africa                 Botswana 1962 5.043823e+08
## 160     Africa                 Botswana 1967 6.723914e+08
## 161     Africa                 Botswana 1972 1.401970e+09
## 162     Africa                 Botswana 1977 2.512321e+09
## 163     Africa                 Botswana 1982 4.416187e+09
## 164     Africa                 Botswana 1987 7.144114e+09
## 165     Africa                 Botswana 1992 1.067930e+10
## 166     Africa                 Botswana 1997 1.328665e+10
## 167     Africa                 Botswana 2002 1.793969e+10
## 168     Africa                 Botswana 2007 2.060363e+10
## 169   Americas                   Brazil 1952 1.193716e+11
## 170   Americas                   Brazil 1957 1.630498e+11
## 171   Americas                   Brazil 1962 2.537119e+11
## 172   Americas                   Brazil 1967 3.019989e+11
## 173   Americas                   Brazil 1972 5.027594e+11
## 174   Americas                   Brazil 1977 7.613445e+11
## 175   Americas                   Brazil 1982 9.067173e+11
## 176   Americas                   Brazil 1987 1.115931e+12
## 177   Americas                   Brazil 1992 1.084077e+12
## 178   Americas                   Brazil 1997 1.341292e+12
## 179   Americas                   Brazil 2002 1.462921e+12
## 180   Americas                   Brazil 2007 1.722599e+12
## 181     Europe                 Bulgaria 1952 1.778194e+10
## 182     Europe                 Bulgaria 1957 2.302010e+10
## 183     Europe                 Bulgaria 1962 3.408978e+10
## 184     Europe                 Bulgaria 1967 4.634615e+10
## 185     Europe                 Bulgaria 1972 5.658143e+10
## 186     Europe                 Bulgaria 1977 6.696505e+10
## 187     Europe                 Bulgaria 1982 7.313032e+10
## 188     Europe                 Bulgaria 1987 7.392763e+10
## 189     Europe                 Bulgaria 1992 5.457130e+10
## 190     Europe                 Bulgaria 1997 4.815750e+10
## 191     Europe                 Bulgaria 2002 5.897116e+10
## 192     Europe                 Bulgaria 2007 7.821393e+10
## 193     Africa             Burkina Faso 1952 2.428340e+09
## 194     Africa             Burkina Faso 1957 2.909042e+09
## 195     Africa             Burkina Faso 1962 3.554493e+09
## 196     Africa             Burkina Faso 1967 4.075819e+09
## 197     Africa             Burkina Faso 1972 4.644538e+09
## 198     Africa             Burkina Faso 1977 4.378233e+09
## 199     Africa             Burkina Faso 1982 5.355437e+09
## 200     Africa             Burkina Faso 1987 6.919414e+09
## 201     Africa             Burkina Faso 1992 8.272383e+09
## 202     Africa             Burkina Faso 1997 9.796843e+09
## 203     Africa             Burkina Faso 2002 1.271241e+10
## 204     Africa             Burkina Faso 2007 1.743546e+10
## 205     Africa                  Burundi 1952 8.297895e+08
## 206     Africa                  Burundi 1957 1.012495e+09
## 207     Africa                  Burundi 1962 1.052082e+09
## 208     Africa                  Burundi 1967 1.375624e+09
## 209     Africa                  Burundi 1972 1.638263e+09
## 210     Africa                  Burundi 1977 2.132331e+09
## 211     Africa                  Burundi 1982 2.563212e+09
## 212     Africa                  Burundi 1987 3.187458e+09
## 213     Africa                  Burundi 1992 3.669694e+09
## 214     Africa                  Burundi 1997 2.835010e+09
## 215     Africa                  Burundi 2002 3.134234e+09
## 216     Africa                  Burundi 2007 3.608510e+09
## 217       Asia                 Cambodia 1952 1.729534e+09
## 218       Asia                 Cambodia 1957 2.310185e+09
## 219       Asia                 Cambodia 1962 3.023033e+09
## 220       Asia                 Cambodia 1967 3.643124e+09
## 221       Asia                 Cambodia 1972 3.141354e+09
## 222       Asia                 Cambodia 1977 3.663575e+09
## 223       Asia                 Cambodia 1982 4.541489e+09
## 224       Asia                 Cambodia 1987 5.725431e+09
## 225       Asia                 Cambodia 1992 6.925441e+09
## 226       Asia                 Cambodia 1997 8.652054e+09
## 227       Asia                 Cambodia 2002 1.158525e+10
## 228       Asia                 Cambodia 2007 2.421888e+10
## 229     Africa                 Cameroon 1952 5.873971e+09
## 230     Africa                 Cameroon 1957 7.037837e+09
## 231     Africa                 Cameroon 1962 8.108812e+09
## 232     Africa                 Cameroon 1967 9.556814e+09
## 233     Africa                 Cameroon 1972 1.182444e+10
## 234     Africa                 Cameroon 1977 1.419588e+10
## 235     Africa                 Cameroon 1982 2.190581e+10
## 236     Africa                 Cameroon 1987 2.805846e+10
## 237     Africa                 Cameroon 1992 2.235567e+10
## 238     Africa                 Cameroon 1997 2.405249e+10
## 239     Africa                 Cameroon 2002 3.080878e+10
## 240     Africa                 Cameroon 2007 3.613752e+10
## 241   Americas                   Canada 1952 1.680701e+11
## 242   Americas                   Canada 1957 2.124560e+11
## 243   Americas                   Canada 1962 2.555967e+11
## 244   Americas                   Canada 1967 3.347108e+11
## 245   Americas                   Canada 1972 4.227497e+11
## 246   Americas                   Canada 1977 5.256835e+11
## 247   Americas                   Canada 1982 5.770931e+11
## 248   Americas                   Canada 1987 7.069260e+11
## 249   Americas                   Canada 1992 7.513913e+11
## 250   Americas                   Canada 1997 8.775034e+11
## 251   Americas                   Canada 2002 1.063270e+12
## 252   Americas                   Canada 2007 1.212704e+12
## 253     Africa Central African Republic 1952 1.383807e+09
## 254     Africa Central African Republic 1957 1.657994e+09
## 255     Africa Central African Republic 1962 1.817614e+09
## 256     Africa Central African Republic 1967 1.969511e+09
## 257     Africa Central African Republic 1972 2.062194e+09
## 258     Africa Central African Republic 1977 2.404605e+09
## 259     Africa Central African Republic 1982 2.369849e+09
## 260     Africa Central African Republic 1987 2.399456e+09
## 261     Africa Central African Republic 1992 2.442004e+09
## 262     Africa Central African Republic 1997 2.737291e+09
## 263     Africa Central African Republic 2002 2.990229e+09
## 264     Africa Central African Republic 2007 3.084613e+09
## 265     Africa                     Chad 1952 3.161727e+09
## 266     Africa                     Chad 1957 3.787905e+09
## 267     Africa                     Chad 1962 4.378505e+09
## 268     Africa                     Chad 1967 4.184010e+09
## 269     Africa                     Chad 1972 4.304977e+09
## 270     Africa                     Chad 1977 4.976221e+09
## 271     Africa                     Chad 1982 3.889896e+09
## 272     Africa                     Chad 1987 5.237128e+09
## 273     Africa                     Chad 1992 6.802737e+09
## 274     Africa                     Chad 1997 7.599529e+09
## 275     Africa                     Chad 2002 1.021572e+10
## 276     Africa                     Chad 2007 1.744758e+10
## 277   Americas                    Chile 1952 2.512768e+10
## 278   Americas                    Chile 1957 3.041835e+10
## 279   Americas                    Chile 1962 3.597768e+10
## 280   Americas                    Chile 1967 4.523938e+10
## 281   Americas                    Chile 1972 5.338831e+10
## 282   Americas                    Chile 1977 5.042071e+10
## 283   Americas                    Chile 1982 5.853448e+10
## 284   Americas                    Chile 1987 6.913502e+10
## 285   Americas                    Chile 1992 1.031022e+11
## 286   Americas                    Chile 1997 1.477229e+11
## 287   Americas                    Chile 2002 1.670393e+11
## 288   Americas                    Chile 2007 2.144967e+11
## 289       Asia                    China 1952 2.227550e+11
## 290       Asia                    China 1957 3.671387e+11
## 291       Asia                    China 1962 3.246787e+11
## 292       Asia                    China 1967 4.623171e+11
## 293       Asia                    China 1972 5.835082e+11
## 294       Asia                    China 1977 6.993242e+11
## 295       Asia                    China 1982 9.626918e+11
## 296       Asia                    China 1987 1.494780e+12
## 297       Asia                    China 1992 1.928939e+12
## 298       Asia                    China 1997 2.815930e+12
## 299       Asia                    China 2002 3.993927e+12
## 300       Asia                    China 2007 6.539501e+12
## 301   Americas                 Colombia 1952 2.648147e+10
## 302   Americas                 Colombia 1957 3.366263e+10
## 303   Americas                 Colombia 1962 4.239461e+10
## 304   Americas                 Colombia 1967 5.294249e+10
## 305   Americas                 Colombia 1972 7.359487e+10
## 306   Americas                 Colombia 1977 9.575545e+10
## 307   Americas                 Colombia 1982 1.220971e+11
## 308   Americas                 Colombia 1987 1.518245e+11
## 309   Americas                 Colombia 1992 1.862218e+11
## 310   Americas                 Colombia 1997 2.303666e+11
## 311   Americas                 Colombia 2002 2.360130e+11
## 312   Americas                 Colombia 2007 3.098839e+11
## 313     Africa                  Comoros 1952 1.697900e+08
## 314     Africa                  Comoros 1957 2.070192e+08
## 315     Africa                  Comoros 1962 2.696390e+08
## 316     Africa                  Comoros 1967 4.078076e+08
## 317     Africa                  Comoros 1972 4.844467e+08
## 318     Africa                  Comoros 1977 3.573379e+08
## 319     Africa                  Comoros 1982 4.417656e+08
## 320     Africa                  Comoros 1987 5.199624e+08
## 321     Africa                  Comoros 1992 5.666309e+08
## 322     Africa                  Comoros 1997 6.196493e+08
## 323     Africa                  Comoros 2002 6.609593e+08
## 324     Africa                  Comoros 2007 7.011117e+08
## 325     Africa          Congo Dem. Rep. 1952 1.100565e+10
## 326     Africa          Congo Dem. Rep. 1957 1.411143e+10
## 327     Africa          Congo Dem. Rep. 1962 1.567335e+10
## 328     Africa          Congo Dem. Rep. 1967 1.718109e+10
## 329     Africa          Congo Dem. Rep. 1972 2.081955e+10
## 330     Africa          Congo Dem. Rep. 1977 2.107235e+10
## 331     Africa          Congo Dem. Rep. 1982 2.064801e+10
## 332     Africa          Congo Dem. Rep. 1987 2.387116e+10
## 333     Africa          Congo Dem. Rep. 1992 1.907414e+10
## 334     Africa          Congo Dem. Rep. 1997 1.492229e+10
## 335     Africa          Congo Dem. Rep. 2002 1.335573e+10
## 336     Africa          Congo Dem. Rep. 2007 1.793173e+10
## 337     Africa               Congo Rep. 1952 1.817162e+09
## 338     Africa               Congo Rep. 1957 2.177213e+09
## 339     Africa               Congo Rep. 1962 2.582905e+09
## 340     Africa               Congo Rep. 1967 3.159326e+09
## 341     Africa               Congo Rep. 1972 4.307096e+09
## 342     Africa               Congo Rep. 1977 5.008605e+09
## 343     Africa               Congo Rep. 1982 8.659833e+09
## 344     Africa               Congo Rep. 1987 8.671665e+09
## 345     Africa               Congo Rep. 1992 9.675414e+09
## 346     Africa               Congo Rep. 1997 9.758960e+09
## 347     Africa               Congo Rep. 2002 1.159773e+10
## 348     Africa               Congo Rep. 2007 1.380594e+10
## 349   Americas               Costa Rica 1952 2.433444e+09
## 350   Americas               Costa Rica 1957 3.325789e+09
## 351   Americas               Costa Rica 1962 4.655607e+09
## 352   Americas               Costa Rica 1967 6.611808e+09
## 353   Americas               Costa Rica 1972 9.390756e+09
## 354   Americas               Costa Rica 1977 1.249657e+10
## 355   Americas               Costa Rica 1982 1.275880e+10
## 356   Americas               Costa Rica 1987 1.576270e+10
## 357   Americas               Costa Rica 1992 1.954833e+10
## 358   Americas               Costa Rica 1997 2.349056e+10
## 359   Americas               Costa Rica 2002 2.961891e+10
## 360   Americas               Costa Rica 2007 3.987157e+10
## 361     Africa            Cote d'Ivoire 1952 4.133873e+09
## 362     Africa            Cote d'Ivoire 1957 4.952957e+09
## 363     Africa            Cote d'Ivoire 1962 6.625733e+09
## 364     Africa            Cote d'Ivoire 1967 9.736713e+09
## 365     Africa            Cote d'Ivoire 1972 1.443971e+10
## 366     Africa            Cote d'Ivoire 1977 1.878124e+10
## 367     Africa            Cote d'Ivoire 1982 2.349193e+10
## 368     Africa            Cote d'Ivoire 1987 2.321122e+10
## 369     Africa            Cote d'Ivoire 1992 2.105018e+10
## 370     Africa            Cote d'Ivoire 1997 2.612586e+10
## 371     Africa            Cote d'Ivoire 2002 2.679751e+10
## 372     Africa            Cote d'Ivoire 2007 2.782622e+10
## 373     Europe                  Croatia 1952 1.210959e+10
## 374     Europe                  Croatia 1957 1.731493e+10
## 375     Europe                  Croatia 1962 2.233093e+10
## 376     Europe                  Croatia 1967 2.905483e+10
## 377     Europe                  Croatia 1972 3.872112e+10
## 378     Europe                  Croatia 1977 4.882426e+10
## 379     Europe                  Croatia 1982 5.835277e+10
## 380     Europe                  Croatia 1987 6.198475e+10
## 381     Europe                  Croatia 1992 3.796450e+10
## 382     Europe                  Croatia 1997 4.389306e+10
## 383     Europe                  Croatia 2002 5.210704e+10
## 384     Europe                  Croatia 2007 6.568873e+10
## 385   Americas                     Cuba 1952 3.356279e+10
## 386   Americas                     Cuba 1957 4.045662e+10
## 387   Americas                     Cuba 1962 3.758314e+10
## 388   Americas                     Cuba 1967 4.631498e+10
## 389   Americas                     Cuba 1972 4.685423e+10
## 390   Americas                     Cuba 1977 6.085708e+10
## 391   Americas                     Cuba 1982 7.162695e+10
## 392   Americas                     Cuba 1987 7.713594e+10
## 393   Americas                     Cuba 1992 5.997352e+10
## 394   Americas                     Cuba 1997 5.965959e+10
## 395   Americas                     Cuba 2002 7.118643e+10
## 396   Americas                     Cuba 2007 1.021604e+11
## 397     Europe           Czech Republic 1952 6.274604e+10
## 398     Europe           Czech Republic 1957 7.854886e+10
## 399     Europe           Czech Republic 1962 9.751952e+10
## 400     Europe           Czech Republic 1967 1.121148e+11
## 401     Europe           Czech Republic 1972 1.292776e+11
## 402     Europe           Czech Republic 1977 1.503980e+11
## 403     Europe           Czech Republic 1982 1.584424e+11
## 404     Europe           Czech Republic 1987 1.681867e+11
## 405     Europe           Czech Republic 1992 1.474838e+11
## 406     Europe           Czech Republic 1997 1.653110e+11
## 407     Europe           Czech Republic 2002 1.804719e+11
## 408     Europe           Czech Republic 2007 2.335561e+11
## 409     Europe                  Denmark 1952 4.200680e+10
## 410     Europe                  Denmark 1957 4.981340e+10
## 411     Europe                  Denmark 1962 6.312029e+10
## 412     Europe                  Denmark 1967 7.711698e+10
## 413     Europe                  Denmark 1972 9.417248e+10
## 414     Europe                  Denmark 1977 1.039203e+11
## 415     Europe                  Denmark 1982 1.109953e+11
## 416     Europe                  Denmark 1987 1.287712e+11
## 417     Europe                  Denmark 1992 1.365596e+11
## 418     Europe                  Denmark 1997 1.574761e+11
## 419     Europe                  Denmark 2002 1.728851e+11
## 420     Europe                  Denmark 2007 1.929066e+11
## 421     Africa                 Djibouti 1952 1.685781e+08
## 422     Africa                 Djibouti 1957 2.058509e+08
## 423     Africa                 Djibouti 1962 2.715809e+08
## 424     Africa                 Djibouti 1967 3.854098e+08
## 425     Africa                 Djibouti 1972 6.607025e+08
## 426     Africa                 Djibouti 1977 7.047803e+08
## 427     Africa                 Djibouti 1982 8.810913e+08
## 428     Africa                 Djibouti 1987 8.957839e+08
## 429     Africa                 Djibouti 1992 9.131988e+08
## 430     Africa                 Djibouti 1997 7.919428e+08
## 431     Africa                 Djibouti 2002 8.537864e+08
## 432     Africa                 Djibouti 2007 1.033690e+09
## 433   Americas       Dominican Republic 1952 3.482197e+09
## 434   Americas       Dominican Republic 1957 4.514577e+09
## 435   Americas       Dominican Republic 1962 5.740082e+09
## 436   Americas       Dominican Republic 1967 6.696166e+09
## 437   Americas       Dominican Republic 1972 1.022962e+10
## 438   Americas       Dominican Republic 1977 1.422205e+10
## 439   Americas       Dominican Republic 1982 1.707600e+10
## 440   Americas       Dominican Republic 1987 1.929931e+10
## 441   Americas       Dominican Republic 1992 2.237857e+10
## 442   Americas       Dominican Republic 1997 2.888519e+10
## 443   Americas       Dominican Republic 2002 3.947841e+10
## 444   Americas       Dominican Republic 2007 5.615422e+10
## 445   Americas                  Ecuador 1952 1.249910e+10
## 446   Americas                  Ecuador 1957 1.534291e+10
## 447   Americas                  Ecuador 1962 1.912999e+10
## 448   Americas                  Ecuador 1967 2.487547e+10
## 449   Americas                  Ecuador 1972 3.326314e+10
## 450   Americas                  Ecuador 1977 4.862008e+10
## 451   Americas                  Ecuador 1982 6.034950e+10
## 452   Americas                  Ecuador 1987 6.186959e+10
## 453   Americas                  Ecuador 1992 7.635339e+10
## 454   Americas                  Ecuador 1997 8.849833e+10
## 455   Americas                  Ecuador 2002 7.459486e+10
## 456   Americas                  Ecuador 2007 9.454640e+10
## 457     Africa                    Egypt 1952 3.153093e+10
## 458     Africa                    Egypt 1957 3.648709e+10
## 459     Africa                    Egypt 1962 4.770687e+10
## 460     Africa                    Egypt 1967 5.749758e+10
## 461     Africa                    Egypt 1972 7.045050e+10
## 462     Africa                    Egypt 1977 1.080322e+11
## 463     Africa                    Egypt 1982 1.600567e+11
## 464     Africa                    Egypt 1987 2.051487e+11
## 465     Africa                    Egypt 1992 2.254168e+11
## 466     Africa                    Egypt 1997 2.759904e+11
## 467     Africa                    Egypt 2002 3.485722e+11
## 468     Africa                    Egypt 2007 4.479709e+11
## 469   Americas              El Salvador 1952 6.227271e+09
## 470   Americas              El Salvador 1957 8.060442e+09
## 471   Americas              El Salvador 1962 1.037747e+10
## 472   Americas              El Salvador 1967 1.409102e+10
## 473   Americas              El Salvador 1972 1.713581e+10
## 474   Americas              El Salvador 1977 2.200788e+10
## 475   Americas              El Salvador 1982 1.833957e+10
## 476   Americas              El Salvador 1987 2.004882e+10
## 477   Americas              El Salvador 1992 2.344176e+10
## 478   Americas              El Salvador 1997 2.981262e+10
## 479   Americas              El Salvador 2002 3.400216e+10
## 480   Americas              El Salvador 2007 3.975299e+10
## 481     Africa        Equatorial Guinea 1952 8.150103e+07
## 482     Africa        Equatorial Guinea 1957 9.924723e+07
## 483     Africa        Equatorial Guinea 1962 1.452559e+08
## 484     Africa        Equatorial Guinea 1967 2.379304e+08
## 485     Africa        Equatorial Guinea 1972 1.866637e+08
## 486     Africa        Equatorial Guinea 1977 1.846919e+08
## 487     Africa        Equatorial Guinea 1982 2.648784e+08
## 488     Africa        Equatorial Guinea 1987 3.299477e+08
## 489     Africa        Equatorial Guinea 1992 4.390540e+08
## 490     Africa        Equatorial Guinea 1997 1.238290e+09
## 491     Africa        Equatorial Guinea 2002 3.818061e+09
## 492     Africa        Equatorial Guinea 2007 6.699346e+09
## 493     Africa                  Eritrea 1952 4.732665e+08
## 494     Africa                  Eritrea 1957 5.309079e+08
## 495     Africa                  Eritrea 1962 6.349745e+08
## 496     Africa                  Eritrea 1967 8.533564e+08
## 497     Africa                  Eritrea 1972 1.162469e+09
## 498     Africa                  Eritrea 1977 1.270778e+09
## 499     Africa                  Eritrea 1982 1.384254e+09
## 500     Africa                  Eritrea 1987 1.519606e+09
## 501     Africa                  Eritrea 1992 2.138181e+09
## 502     Africa                  Eritrea 1997 3.707156e+09
## 503     Africa                  Eritrea 2002 3.378917e+09
## 504     Africa                  Eritrea 2007 3.146934e+09
## 505     Africa                 Ethiopia 1952 7.554712e+09
## 506     Africa                 Ethiopia 1957 8.644931e+09
## 507     Africa                 Ethiopia 1962 1.054739e+10
## 508     Africa                 Ethiopia 1967 1.437922e+10
## 509     Africa                 Ethiopia 1972 1.742354e+10
## 510     Africa                 Ethiopia 1977 1.927548e+10
## 511     Africa                 Ethiopia 1982 2.202329e+10
## 512     Africa                 Ethiopia 1987 2.467061e+10
## 513     Africa                 Ethiopia 1992 2.194769e+10
## 514     Africa                 Ethiopia 1997 3.088181e+10
## 515     Africa                 Ethiopia 2002 3.601544e+10
## 516     Africa                 Ethiopia 2007 5.285484e+10
## 517     Europe                  Finland 1952 2.627950e+10
## 518     Europe                  Finland 1957 3.262638e+10
## 519     Europe                  Finland 1962 4.209310e+10
## 520     Europe                  Finland 1967 5.030226e+10
## 521     Europe                  Finland 1972 6.662026e+10
## 522     Europe                  Finland 1977 7.395257e+10
## 523     Europe                  Finland 1982 8.945831e+10
## 524     Europe                  Finland 1987 1.042617e+11
## 525     Europe                  Finland 1992 1.040832e+11
## 526     Europe                  Finland 1997 1.218084e+11
## 527     Europe                  Finland 2002 1.464675e+11
## 528     Europe                  Finland 2007 1.739540e+11
## 529     Europe                   France 1952 2.984834e+11
## 530     Europe                   France 1957 3.838577e+11
## 531     Europe                   France 1962 4.976523e+11
## 532     Europe                   France 1967 6.443929e+11
## 533     Europe                   France 1972 8.332572e+11
## 534     Europe                   France 1977 9.725283e+11
## 535     Europe                   France 1982 1.104669e+12
## 536     Europe                   France 1987 1.227558e+12
## 537     Europe                   France 1992 1.417360e+12
## 538     Europe                   France 1997 1.517748e+12
## 539     Europe                   France 2002 1.733394e+12
## 540     Europe                   France 2007 1.861228e+12
## 541     Africa                    Gabon 1952 1.806274e+09
## 542     Africa                    Gabon 1957 2.164168e+09
## 543     Africa                    Gabon 1962 3.021697e+09
## 544     Africa                    Gabon 1967 4.087468e+09
## 545     Africa                    Gabon 1972 6.133986e+09
## 546     Africa                    Gabon 1977 1.536036e+10
## 547     Africa                    Gabon 1982 1.139357e+10
## 548     Africa                    Gabon 1987 1.044539e+10
## 549     Africa                    Gabon 1992 1.332932e+10
## 550     Africa                    Gabon 1997 1.658070e+10
## 551     Africa                    Gabon 2002 1.626951e+10
## 552     Africa                    Gabon 2007 1.921368e+10
## 553     Africa                   Gambia 1952 1.379608e+08
## 554     Africa                   Gambia 1957 1.683375e+08
## 555     Africa                   Gambia 1962 2.242812e+08
## 556     Africa                   Gambia 1967 3.230054e+08
## 557     Africa                   Gambia 1972 3.909733e+08
## 558     Africa                   Gambia 1977 5.381736e+08
## 559     Africa                   Gambia 1982 5.980410e+08
## 560     Africa                   Gambia 1987 5.189350e+08
## 561     Africa                   Gambia 1992 6.825206e+08
## 562     Africa                   Gambia 1997 8.078582e+08
## 563     Africa                   Gambia 2002 9.629792e+08
## 564     Africa                   Gambia 2007 1.270912e+09
## 565     Europe                  Germany 1952 4.939866e+11
## 566     Europe                  Germany 1957 7.235300e+11
## 567     Europe                  Germany 1962 9.514162e+11
## 568     Europe                  Germany 1967 1.126101e+12
## 569     Europe                  Germany 1972 1.418181e+12
## 570     Europe                  Germany 1977 1.603306e+12
## 571     Europe                  Germany 1982 1.725846e+12
## 572     Europe                  Germany 1987 1.914916e+12
## 573     Europe                  Germany 1992 2.136268e+12
## 574     Europe                  Germany 1997 2.278996e+12
## 575     Europe                  Germany 2002 2.473468e+12
## 576     Europe                  Germany 2007 2.650871e+12
## 577     Africa                    Ghana 1952 5.085960e+09
## 578     Africa                    Ghana 1957 6.669702e+09
## 579     Africa                    Ghana 1962 8.753048e+09
## 580     Africa                    Ghana 1967 9.557409e+09
## 581     Africa                    Ghana 1972 1.102125e+10
## 582     Africa                    Ghana 1977 1.046669e+10
## 583     Africa                    Ghana 1982 9.987067e+09
## 584     Africa                    Ghana 1987 1.200047e+10
## 585     Africa                    Ghana 1992 1.505881e+10
## 586     Africa                    Ghana 1997 1.851491e+10
## 587     Africa                    Ghana 2002 2.285212e+10
## 588     Africa                    Ghana 2007 3.036685e+10
## 589     Europe                   Greece 1952 2.730371e+10
## 590     Europe                   Greece 1957 3.980344e+10
## 591     Europe                   Greece 1962 5.083463e+10
## 592     Europe                   Greece 1967 7.420391e+10
## 593     Europe                   Greece 1972 1.131063e+11
## 594     Europe                   Greece 1977 1.321387e+11
## 595     Europe                   Greece 1982 1.494241e+11
## 596     Europe                   Greece 1987 1.607940e+11
## 597     Europe                   Greece 1992 1.811235e+11
## 598     Europe                   Greece 1997 1.968953e+11
## 599     Europe                   Greece 2002 2.387381e+11
## 600     Europe                   Greece 2007 2.948342e+11
## 601   Americas                Guatemala 1952 7.640161e+09
## 602   Americas                Guatemala 1957 9.528740e+09
## 603   Americas                Guatemala 1962 1.157589e+10
## 604   Americas                Guatemala 1967 1.520998e+10
## 605   Americas                Guatemala 1972 2.076006e+10
## 606   Americas                Guatemala 1977 2.783270e+10
## 607   Americas                Guatemala 1982 3.083010e+10
## 608   Americas                Guatemala 1987 3.111148e+10
## 609   Americas                Guatemala 1992 3.767739e+10
## 610   Americas                Guatemala 1997 4.592443e+10
## 611   Americas                Guatemala 2002 5.430977e+10
## 612   Americas                Guatemala 2007 6.520383e+10
## 613     Africa                   Guinea 1952 1.359290e+09
## 614     Africa                   Guinea 1957 1.657762e+09
## 615     Africa                   Guinea 1962 2.155215e+09
## 616     Africa                   Guinea 1967 2.446225e+09
## 617     Africa                   Guinea 1972 2.826777e+09
## 618     Africa                   Guinea 1977 3.697320e+09
## 619     Africa                   Guinea 1982 4.038075e+09
## 620     Africa                   Guinea 1987 4.551696e+09
## 621     Africa                   Guinea 1992 5.552952e+09
## 622     Africa                   Guinea 1997 6.998057e+09
## 623     Africa                   Guinea 2002 8.328528e+09
## 624     Africa                   Guinea 2007 9.377349e+09
## 625     Africa            Guinea-Bissau 1952 1.741090e+08
## 626     Africa            Guinea-Bissau 1957 2.595471e+08
## 627     Africa            Guinea-Bissau 1962 3.277436e+08
## 628     Africa            Guinea-Bissau 1967 4.302693e+08
## 629     Africa            Guinea-Bissau 1972 5.129365e+08
## 630     Africa            Guinea-Bissau 1977 5.698952e+08
## 631     Africa            Guinea-Bissau 1982 6.922795e+08
## 632     Africa            Guinea-Bissau 1987 6.830430e+08
## 633     Africa            Guinea-Bissau 1992 7.835162e+08
## 634     Africa            Guinea-Bissau 1997 9.509847e+08
## 635     Africa            Guinea-Bissau 2002 7.671029e+08
## 636     Africa            Guinea-Bissau 2007 8.526529e+08
## 637   Americas                    Haiti 1952 5.891913e+09
## 638   Americas                    Haiti 1957 6.057406e+09
## 639   Americas                    Haiti 1962 6.970999e+09
## 640   Americas                    Haiti 1967 6.270184e+09
## 641   Americas                    Haiti 1972 7.773137e+09
## 642   Americas                    Haiti 1977 9.200098e+09
## 643   Americas                    Haiti 1982 1.045481e+10
## 644   Americas                    Haiti 1987 1.049365e+10
## 645   Americas                    Haiti 1992 9.213607e+09
## 646   Americas                    Haiti 1997 9.276090e+09
## 647   Americas                    Haiti 2002 9.664493e+09
## 648   Americas                    Haiti 2007 1.021730e+10
## 649   Americas                 Honduras 1952 3.330697e+09
## 650   Americas                 Honduras 1957 3.931129e+09
## 651   Americas                 Honduras 1962 4.788889e+09
## 652   Americas                 Honduras 1967 6.347422e+09
## 653   Americas                 Honduras 1972 7.501352e+09
## 654   Americas                 Honduras 1977 9.786553e+09
## 655   Americas                 Honduras 1982 1.145514e+10
## 656   Americas                 Honduras 1987 1.321759e+10
## 657   Americas                 Honduras 1992 1.564683e+10
## 658   Americas                 Honduras 1997 1.854541e+10
## 659   Americas                 Honduras 2002 2.069790e+10
## 660   Americas                 Honduras 2007 2.655487e+10
## 661       Asia          Hong Kong China 1952 6.493394e+09
## 662       Asia          Hong Kong China 1957 9.930242e+09
## 663       Asia          Hong Kong China 1962 1.551014e+10
## 664       Asia          Hong Kong China 1967 2.307378e+10
## 665       Asia          Hong Kong China 1972 3.422587e+10
## 666       Asia          Hong Kong China 1977 5.127392e+10
## 667       Asia          Hong Kong China 1982 7.665391e+10
## 668       Asia          Hong Kong China 1987 1.119051e+11
## 669       Asia          Hong Kong China 1992 1.443293e+11
## 670       Asia          Hong Kong China 1997 1.843388e+11
## 671       Asia          Hong Kong China 2002 2.042877e+11
## 672       Asia          Hong Kong China 2007 2.772967e+11
## 673     Europe                  Hungary 1952 5.002596e+10
## 674     Europe                  Hungary 1957 5.942933e+10
## 675     Europe                  Hungary 1962 7.597927e+10
## 676     Europe                  Hungary 1967 9.535022e+10
## 677     Europe                  Hungary 1972 1.056939e+11
## 678     Europe                  Hungary 1977 1.241872e+11
## 679     Europe                  Hungary 1982 1.343115e+11
## 680     Europe                  Hungary 1987 1.378221e+11
## 681     Europe                  Hungary 1992 1.090299e+11
## 682     Europe                  Hungary 1997 1.199937e+11
## 683     Europe                  Hungary 2002 1.496760e+11
## 684     Europe                  Hungary 2007 1.792990e+11
## 685     Europe                  Iceland 1952 1.075342e+09
## 686     Europe                  Iceland 1957 1.526277e+09
## 687     Europe                  Iceland 1962 1.884278e+09
## 688     Europe                  Iceland 1967 2.646344e+09
## 689     Europe                  Iceland 1972 3.306140e+09
## 690     Europe                  Iceland 1977 4.359923e+09
## 691     Europe                  Iceland 1982 5.445018e+09
## 692     Europe                  Iceland 1987 6.587462e+09
## 693     Europe                  Iceland 1992 6.512699e+09
## 694     Europe                  Iceland 1997 7.609946e+09
## 695     Europe                  Iceland 2002 8.975937e+09
## 696     Europe                  Iceland 2007 1.092410e+10
## 697       Asia                    India 1952 2.033225e+11
## 698       Asia                    India 1957 2.413354e+11
## 699       Asia                    India 1962 2.988896e+11
## 700       Asia                    India 1967 3.545899e+11
## 701       Asia                    India 1972 4.105264e+11
## 702       Asia                    India 1977 5.156559e+11
## 703       Asia                    India 1982 6.058523e+11
## 704       Asia                    India 1987 7.694920e+11
## 705       Asia                    India 1992 1.015363e+12
## 706       Asia                    India 1997 1.399006e+12
## 707       Asia                    India 2002 1.806461e+12
## 708       Asia                    India 2007 2.722925e+12
## 709       Asia                Indonesia 1952 6.151288e+10
## 710       Asia                Indonesia 1957 7.740753e+10
## 711       Asia                Indonesia 1962 8.410347e+10
## 712       Asia                Indonesia 1967 8.336658e+10
## 713       Asia                Indonesia 1972 1.347574e+11
## 714       Asia                Indonesia 1977 1.890499e+11
## 715       Asia                Indonesia 1982 2.326019e+11
## 716       Asia                Indonesia 1987 2.959549e+11
## 717       Asia                Indonesia 1992 4.404426e+11
## 718       Asia                Indonesia 1997 6.216150e+11
## 719       Asia                Indonesia 2002 6.065681e+11
## 720       Asia                Indonesia 2007 7.915020e+11
## 721       Asia                     Iran 1952 5.242615e+10
## 722       Asia                     Iran 1957 6.512078e+10
## 723       Asia                     Iran 1962 9.578098e+10
## 724       Asia                     Iran 1967 1.567528e+11
## 725       Asia                     Iran 1972 2.943174e+11
## 726       Asia                     Iran 1977 4.218154e+11
## 727       Asia                     Iran 1982 3.277119e+11
## 728       Asia                     Iran 1987 3.446971e+11
## 729       Asia                     Iran 1992 4.370188e+11
## 730       Asia                     Iran 1997 5.233165e+11
## 731       Asia                     Iran 2002 6.182793e+11
## 732       Asia                     Iran 2007 8.060583e+11
## 733       Asia                     Iraq 1952 2.247322e+10
## 734       Asia                     Iraq 1957 3.892488e+10
## 735       Asia                     Iraq 1962 6.039635e+10
## 736       Asia                     Iraq 1967 7.608962e+10
## 737       Asia                     Iraq 1972 9.634936e+10
## 738       Asia                     Iraq 1977 1.745391e+11
## 739       Asia                     Iraq 1982 2.057669e+11
## 740       Asia                     Iraq 1987 1.926218e+11
## 741       Asia                     Iraq 1992 6.690428e+10
## 742       Asia                     Iraq 1997 6.391104e+10
## 743       Asia                     Iraq 2002 1.053852e+11
## 744       Asia                     Iraq 2007 1.229526e+11
## 745     Europe                  Ireland 1952 1.538156e+10
## 746     Europe                  Ireland 1957 1.611538e+10
## 747     Europe                  Ireland 1962 1.876742e+10
## 748     Europe                  Ireland 1967 2.220192e+10
## 749     Europe                  Ireland 1972 2.882487e+10
## 750     Europe                  Ireland 1977 3.648490e+10
## 751     Europe                  Ireland 1982 4.391176e+10
## 752     Europe                  Ireland 1987 4.910856e+10
## 753     Europe                  Ireland 1992 6.247007e+10
## 754     Europe                  Ireland 1997 8.992769e+10
## 755     Europe                  Ireland 2002 1.321902e+11
## 756     Europe                  Ireland 2007 1.671412e+11
## 757       Asia                   Israel 1952 6.623901e+09
## 758       Asia                   Israel 1957 1.047114e+10
## 759       Asia                   Israel 1962 1.642043e+10
## 760       Asia                   Israel 1967 2.260926e+10
## 761       Asia                   Israel 1972 3.958697e+10
## 762       Asia                   Israel 1977 4.651885e+10
## 763       Asia                   Israel 1982 5.929247e+10
## 764       Asia                   Israel 1987 7.196832e+10
## 765       Asia                   Israel 1992 8.911224e+10
## 766       Asia                   Israel 1997 1.155872e+11
## 767       Asia                   Israel 2002 1.320804e+11
## 768       Asia                   Israel 2007 1.640299e+11
## 769     Europe                    Italy 1952 2.350603e+11
## 770     Europe                    Italy 1957 3.073214e+11
## 771     Europe                    Italy 1962 4.191301e+11
## 772     Europe                    Italy 1967 5.278508e+11
## 773     Europe                    Italy 1972 6.670260e+11
## 774     Europe                    Italy 1977 7.991797e+11
## 775     Europe                    Italy 1982 9.349571e+11
## 776     Europe                    Italy 1987 1.089621e+12
## 777     Europe                    Italy 1992 1.251274e+12
## 778     Europe                    Italy 1997 1.418307e+12
## 779     Europe                    Italy 2002 1.620108e+12
## 780     Europe                    Italy 2007 1.661264e+12
## 781   Americas                  Jamaica 1952 4.133580e+09
## 782   Americas                  Jamaica 1957 7.301695e+09
## 783   Americas                  Jamaica 1962 8.735441e+09
## 784   Americas                  Jamaica 1967 1.139866e+10
## 785   Americas                  Jamaica 1972 1.485006e+10
## 786   Americas                  Jamaica 1977 1.434323e+10
## 787   Americas                  Jamaica 1982 1.394626e+10
## 788   Americas                  Jamaica 1987 1.477683e+10
## 789   Americas                  Jamaica 1992 1.761348e+10
## 790   Americas                  Jamaica 1997 1.802781e+10
## 791   Americas                  Jamaica 2002 1.863869e+10
## 792   Americas                  Jamaica 2007 2.035301e+10
## 793       Asia                    Japan 1952 2.781349e+11
## 794       Asia                    Japan 1957 3.953411e+11
## 795       Asia                    Japan 1962 6.302519e+11
## 796       Asia                    Japan 1967 9.929060e+11
## 797       Asia                    Japan 1972 1.584113e+12
## 798       Asia                    Japan 1977 1.891465e+12
## 799       Asia                    Japan 1982 2.296144e+12
## 800       Asia                    Japan 1987 2.731908e+12
## 801       Asia                    Japan 1992 3.335120e+12
## 802       Asia                    Japan 1997 3.629636e+12
## 803       Asia                    Japan 2002 3.634667e+12
## 804       Asia                    Japan 2007 4.035135e+12
## 805       Asia                   Jordan 1952 9.403869e+08
## 806       Asia                   Jordan 1957 1.408070e+09
## 807       Asia                   Jordan 1962 2.192005e+09
## 808       Asia                   Jordan 1967 3.441113e+09
## 809       Asia                   Jordan 1972 3.405974e+09
## 810       Asia                   Jordan 1977 5.526865e+09
## 811       Asia                   Jordan 1982 9.766972e+09
## 812       Asia                   Jordan 1987 1.254546e+10
## 813       Asia                   Jordan 1992 1.327138e+10
## 814       Asia                   Jordan 1997 1.649984e+10
## 815       Asia                   Jordan 2002 2.040678e+10
## 816       Asia                   Jordan 2007 2.735717e+10
## 817     Africa                    Kenya 1952 5.517328e+09
## 818     Africa                    Kenya 1957 7.040579e+09
## 819     Africa                    Kenya 1962 7.784374e+09
## 820     Africa                    Kenya 1967 1.076974e+10
## 821     Africa                    Kenya 1972 1.472306e+10
## 822     Africa                    Kenya 1977 1.838090e+10
## 823     Africa                    Kenya 1982 2.381163e+10
## 824     Africa                    Kenya 1987 2.887045e+10
## 825     Africa                    Kenya 1992 3.357560e+10
## 826     Africa                    Kenya 1997 3.845251e+10
## 827     Africa                    Kenya 2002 4.041102e+10
## 828     Africa                    Kenya 2007 5.210657e+10
## 829       Asia          Korea Dem. Rep. 1952 9.648113e+09
## 830       Asia          Korea Dem. Rep. 1957 1.478655e+10
## 831       Asia          Korea Dem. Rep. 1962 1.770483e+10
## 832       Asia          Korea Dem. Rep. 1967 2.704507e+10
## 833       Asia          Korea Dem. Rep. 1972 5.471456e+10
## 834       Asia          Korea Dem. Rep. 1977 6.703668e+10
## 835       Asia          Korea Dem. Rep. 1982 7.246998e+10
## 836       Asia          Korea Dem. Rep. 1987 7.830076e+10
## 837       Asia          Korea Dem. Rep. 1992 7.717190e+10
## 838       Asia          Korea Dem. Rep. 1997 3.649516e+10
## 839       Asia          Korea Dem. Rep. 2002 3.658333e+10
## 840       Asia          Korea Dem. Rep. 2007 3.712117e+10
## 841       Asia               Korea Rep. 1952 2.158840e+10
## 842       Asia               Korea Rep. 1957 3.363680e+10
## 843       Asia               Korea Rep. 1962 4.059069e+10
## 844       Asia               Korea Rep. 1967 6.114267e+10
## 845       Asia               Korea Rep. 1972 1.015495e+11
## 846       Asia               Korea Rep. 1977 1.696905e+11
## 847       Asia               Korea Rep. 1982 2.211278e+11
## 848       Asia               Korea Rep. 1987 3.551642e+11
## 849       Asia               Korea Rep. 1992 5.302334e+11
## 850       Asia               Korea Rep. 1997 7.384822e+11
## 851       Asia               Korea Rep. 2002 9.226381e+11
## 852       Asia               Korea Rep. 2007 1.145105e+12
## 853       Asia                   Kuwait 1952 1.734118e+10
## 854       Asia                   Kuwait 1957 2.416294e+10
## 855       Asia                   Kuwait 1962 3.419940e+10
## 856       Asia                   Kuwait 1967 4.651480e+10
## 857       Asia                   Kuwait 1972 9.206369e+10
## 858       Asia                   Kuwait 1977 6.758380e+10
## 859       Asia                   Kuwait 1982 4.695248e+10
## 860       Asia                   Kuwait 1987 5.318564e+10
## 861       Asia                   Kuwait 1992 4.953820e+10
## 862       Asia                   Kuwait 1997 7.114450e+10
## 863       Asia                   Kuwait 2002 7.413713e+10
## 864       Asia                   Kuwait 2007 1.185305e+11
## 865       Asia                  Lebanon 1952 6.959841e+09
## 866       Asia                  Lebanon 1957 1.003239e+10
## 867       Asia                  Lebanon 1962 1.078251e+10
## 868       Asia                  Lebanon 1967 1.313664e+10
## 869       Asia                  Lebanon 1972 2.006364e+10
## 870       Asia                  Lebanon 1977 2.698177e+10
## 871       Asia                  Lebanon 1982 2.358534e+10
## 872       Asia                  Lebanon 1987 1.661173e+10
## 873       Asia                  Lebanon 1992 2.218836e+10
## 874       Asia                  Lebanon 1997 3.003292e+10
## 875       Asia                  Lebanon 2002 3.425462e+10
## 876       Asia                  Lebanon 2007 4.102072e+10
## 877     Africa                  Lesotho 1952 2.237602e+08
## 878     Africa                  Lesotho 1957 2.732792e+08
## 879     Africa                  Lesotho 1962 3.677968e+08
## 880     Africa                  Lesotho 1967 4.968340e+08
## 881     Africa                  Lesotho 1972 5.545719e+08
## 882     Africa                  Lesotho 1977 9.328479e+08
## 883     Africa                  Lesotho 1982 1.125582e+09
## 884     Africa                  Lesotho 1987 1.237770e+09
## 885     Africa                  Lesotho 1992 1.762598e+09
## 886     Africa                  Lesotho 1997 2.351922e+09
## 887     Africa                  Lesotho 2002 2.610012e+09
## 888     Africa                  Lesotho 2007 3.158513e+09
## 889     Africa                  Liberia 1952 4.968968e+08
## 890     Africa                  Liberia 1957 6.060357e+08
## 891     Africa                  Liberia 1962 7.057298e+08
## 892     Africa                  Liberia 1967 9.129888e+08
## 893     Africa                  Liberia 1972 1.190558e+09
## 894     Africa                  Liberia 1977 1.090864e+09
## 895     Africa                  Liberia 1982 1.119723e+09
## 896     Africa                  Liberia 1987 1.148582e+09
## 897     Africa                  Liberia 1992 1.217843e+09
## 898     Africa                  Liberia 1997 1.340624e+09
## 899     Africa                  Liberia 2002 1.495937e+09
## 900     Africa                  Liberia 2007 1.323912e+09
## 901     Africa                    Libya 1952 2.434652e+09
## 902     Africa                    Libya 1957 4.143383e+09
## 903     Africa                    Libya 1962 9.742713e+09
## 904     Africa                    Libya 1967 3.302548e+10
## 905     Africa                    Libya 1972 4.588653e+10
## 906     Africa                    Libya 1977 5.974643e+10
## 907     Africa                    Libya 1982 5.806742e+10
## 908     Africa                    Libya 1987 4.472642e+10
## 909     Africa                    Libya 1992 4.207439e+10
## 910     Africa                    Libya 1997 4.506192e+10
## 911     Africa                    Libya 2002 5.118773e+10
## 912     Africa                    Libya 2007 7.279009e+10
## 913     Africa               Madagascar 1952 6.872938e+09
## 914     Africa               Madagascar 1957 8.234739e+09
## 915     Africa               Madagascar 1962 9.372769e+09
## 916     Africa               Madagascar 1967 1.035096e+10
## 917     Africa               Madagascar 1972 1.238407e+10
## 918     Africa               Madagascar 1977 1.236489e+10
## 919     Africa               Madagascar 1982 1.194932e+10
## 920     Africa               Madagascar 1987 1.221145e+10
## 921     Africa               Madagascar 1992 1.270707e+10
## 922     Africa               Madagascar 1997 1.397099e+10
## 923     Africa               Madagascar 2002 1.473778e+10
## 924     Africa               Madagascar 2007 2.002579e+10
## 925     Africa                   Malawi 1952 1.077151e+09
## 926     Africa                   Malawi 1957 1.341226e+09
## 927     Africa                   Malawi 1962 1.552685e+09
## 928     Africa                   Malawi 1967 2.055025e+09
## 929     Africa                   Malawi 1972 2.765845e+09
## 930     Africa                   Malawi 1977 3.738755e+09
## 931     Africa                   Malawi 1982 4.115013e+09
## 932     Africa                   Malawi 1987 4.972763e+09
## 933     Africa                   Malawi 1992 5.640025e+09
## 934     Africa                   Malawi 1997 7.213508e+09
## 935     Africa                   Malawi 2002 7.868292e+09
## 936     Africa                   Malawi 2007 1.011992e+10
## 937       Asia                 Malaysia 1952 1.235718e+10
## 938       Asia                 Malaysia 1957 1.400853e+10
## 939       Asia                 Malaysia 1962 1.814128e+10
## 940       Asia                 Malaysia 1967 2.313020e+10
## 941       Asia                 Malaysia 1972 3.259781e+10
## 942       Asia                 Malaysia 1977 4.917111e+10
## 943       Asia                 Malaysia 1982 7.105937e+10
## 944       Asia                 Malaysia 1987 8.573865e+10
## 945       Asia                 Malaysia 1992 1.333277e+11
## 946       Asia                 Malaysia 1997 2.074824e+11
## 947       Asia                 Malaysia 2002 2.313143e+11
## 948       Asia                 Malaysia 2007 3.090661e+11
## 949     Africa                     Mali 1952 1.736145e+09
## 950     Africa                     Mali 1957 2.080144e+09
## 951     Africa                     Mali 1962 2.327242e+09
## 952     Africa                     Mali 1967 2.840818e+09
## 953     Africa                     Mali 1972 3.388310e+09
## 954     Africa                     Mali 1977 4.455837e+09
## 955     Africa                     Mali 1982 4.325021e+09
## 956     Africa                     Mali 1987 5.222971e+09
## 957     Africa                     Mali 1992 6.219704e+09
## 958     Africa                     Mali 1997 7.416559e+09
## 959     Africa                     Mali 2002 1.006608e+10
## 960     Africa                     Mali 2007 1.254413e+10
## 961     Africa               Mauritania 1952 7.598776e+08
## 962     Africa               Mauritania 1957 9.111463e+08
## 963     Africa               Mauritania 1962 1.210856e+09
## 964     Africa               Mauritania 1967 1.748779e+09
## 965     Africa               Mauritania 1972 2.114934e+09
## 966     Africa               Mauritania 1977 2.181379e+09
## 967     Africa               Mauritania 1982 2.402627e+09
## 968     Africa               Mauritania 1987 2.617513e+09
## 969     Africa               Mauritania 1992 2.885376e+09
## 970     Africa               Mauritania 1997 3.625884e+09
## 971     Africa               Mauritania 2002 4.466822e+09
## 972     Africa               Mauritania 2007 5.896423e+09
## 973     Africa                Mauritius 1952 1.016559e+09
## 974     Africa                Mauritius 1957 1.240389e+09
## 975     Africa                Mauritius 1962 1.772917e+09
## 976     Africa                Mauritius 1967 1.953846e+09
## 977     Africa                Mauritius 1972 2.192597e+09
## 978     Africa                Mauritius 1977 3.388220e+09
## 979     Africa                Mauritius 1982 3.658681e+09
## 980     Africa                Mauritius 1987 4.987669e+09
## 981     Africa                Mauritius 1992 6.641070e+09
## 982     Africa                Mauritius 1997 8.538210e+09
## 983     Africa                Mauritius 2002 1.082804e+10
## 984     Africa                Mauritius 2007 1.370590e+10
## 985   Americas                   Mexico 1952 1.048457e+11
## 986   Americas                   Mexico 1957 1.446684e+11
## 987   Americas                   Mexico 1962 1.884026e+11
## 988   Americas                   Mexico 1967 2.762017e+11
## 989   Americas                   Mexico 1972 3.812198e+11
## 990   Americas                   Mexico 1977 4.893533e+11
## 991   Americas                   Mexico 1982 6.885513e+11
## 992   Americas                   Mexico 1987 6.961167e+11
## 993   Americas                   Mexico 1992 8.346215e+11
## 994   Americas                   Mexico 1997 9.366364e+11
## 995   Americas                   Mexico 2002 1.100885e+12
## 996   Americas                   Mexico 2007 1.301973e+12
## 997       Asia                 Mongolia 1952 6.297750e+08
## 998       Asia                 Mongolia 1957 8.050907e+08
## 999       Asia                 Mongolia 1962 1.067213e+09
## 1000      Asia                 Mongolia 1967 1.409334e+09
## 1001      Asia                 Mongolia 1972 1.877410e+09
## 1002      Asia                 Mongolia 1977 2.517398e+09
## 1003      Asia                 Mongolia 1982 3.513123e+09
## 1004      Asia                 Mongolia 1987 4.711398e+09
## 1005      Asia                 Mongolia 1992 4.129281e+09
## 1006      Asia                 Mongolia 1997 4.745744e+09
## 1007      Asia                 Mongolia 2002 5.724838e+09
## 1008      Asia                 Mongolia 2007 8.897643e+09
## 1009    Europe               Montenegro 1952 1.095661e+09
## 1010    Europe               Montenegro 1957 1.630611e+09
## 1011    Europe               Montenegro 1962 2.206362e+09
## 1012    Europe               Montenegro 1967 2.960040e+09
## 1013    Europe               Montenegro 1972 4.104498e+09
## 1014    Europe               Montenegro 1977 5.374421e+09
## 1015    Europe               Montenegro 1982 6.313244e+09
## 1016    Europe               Montenegro 1987 6.681348e+09
## 1017    Europe               Montenegro 1992 4.353423e+09
## 1018    Europe               Montenegro 1997 4.478414e+09
## 1019    Europe               Montenegro 2002 4.722688e+09
## 1020    Europe               Montenegro 2007 6.336476e+09
## 1021    Africa                  Morocco 1952 1.677942e+10
## 1022    Africa                  Morocco 1957 1.872925e+10
## 1023    Africa                  Morocco 1962 2.045126e+10
## 1024    Africa                  Morocco 1967 2.527264e+10
## 1025    Africa                  Morocco 1972 3.215834e+10
## 1026    Africa                  Morocco 1977 4.361216e+10
## 1027    Africa                  Morocco 1982 5.458950e+10
## 1028    Africa                  Morocco 1987 6.333136e+10
## 1029    Africa                  Morocco 1992 7.605443e+10
## 1030    Africa                  Morocco 1997 8.507788e+10
## 1031    Africa                  Morocco 2002 1.015601e+11
## 1032    Africa                  Morocco 2007 1.289583e+11
## 1033    Africa               Mozambique 1952 3.020267e+09
## 1034    Africa               Mozambique 1957 3.487957e+09
## 1035    Africa               Mozambique 1962 4.335999e+09
## 1036    Africa               Mozambique 1967 4.919203e+09
## 1037    Africa               Mozambique 1972 7.111151e+09
## 1038    Africa               Mozambique 1977 5.589748e+09
## 1039    Africa               Mozambique 1982 5.817958e+09
## 1040    Africa               Mozambique 1987 5.026265e+09
## 1041    Africa               Mozambique 1992 5.407703e+09
## 1042    Africa               Mozambique 1997 7.842520e+09
## 1043    Africa               Mozambique 2002 1.170532e+10
## 1044    Africa               Mozambique 2007 1.643389e+10
## 1045      Asia                  Myanmar 1952 6.650782e+09
## 1046      Asia                  Myanmar 1957 7.606145e+09
## 1047      Asia                  Myanmar 1962 9.170161e+09
## 1048      Asia                  Myanmar 1967 9.028725e+09
## 1049      Asia                  Myanmar 1972 1.016250e+10
## 1050      Asia                  Myanmar 1977 1.169692e+10
## 1051      Asia                  Myanmar 1982 1.470451e+10
## 1052      Asia                  Myanmar 1987 1.464100e+10
## 1053      Asia                  Myanmar 1992 1.406965e+10
## 1054      Asia                  Myanmar 1997 1.794786e+10
## 1055      Asia                  Myanmar 2002 2.786043e+10
## 1056      Asia                  Myanmar 2007 4.508731e+10
## 1057    Africa                  Namibia 1952 1.177548e+09
## 1058    Africa                  Namibia 1957 1.436763e+09
## 1059    Africa                  Namibia 1962 1.971811e+09
## 1060    Africa                  Namibia 1967 2.680776e+09
## 1061    Africa                  Namibia 1972 3.078462e+09
## 1062    Africa                  Namibia 1977 3.787428e+09
## 1063    Africa                  Namibia 1982 4.606061e+09
## 1064    Africa                  Namibia 1987 4.721268e+09
## 1065    Africa                  Namibia 1992 5.913215e+09
## 1066    Africa                  Namibia 1997 6.920743e+09
## 1067    Africa                  Namibia 2002 8.031247e+09
## 1068    Africa                  Namibia 2007 9.887114e+09
## 1069      Asia                    Nepal 1952 5.012432e+09
## 1070      Asia                    Nepal 1957 5.789422e+09
## 1071      Asia                    Nepal 1962 6.740602e+09
## 1072      Asia                    Nepal 1967 7.617883e+09
## 1073      Asia                    Nepal 1972 8.375870e+09
## 1074      Asia                    Nepal 1977 9.671206e+09
## 1075      Asia                    Nepal 1982 1.134765e+10
## 1076      Asia                    Nepal 1987 1.389715e+10
## 1077      Asia                    Nepal 1992 1.824766e+10
## 1078      Asia                    Nepal 1997 2.325164e+10
## 1079      Asia                    Nepal 2002 2.735407e+10
## 1080      Asia                    Nepal 2007 3.154225e+10
## 1081    Europe              Netherlands 1952 9.283129e+10
## 1082    Europe              Netherlands 1957 1.243356e+11
## 1083    Europe              Netherlands 1962 1.510048e+11
## 1084    Europe              Netherlands 1967 1.935281e+11
## 1085    Europe              Netherlands 1972 2.505316e+11
## 1086    Europe              Netherlands 1977 2.938089e+11
## 1087    Europe              Netherlands 1982 3.062349e+11
## 1088    Europe              Netherlands 1987 3.468532e+11
## 1089    Europe              Netherlands 1992 4.065324e+11
## 1090    Europe              Netherlands 1997 4.719747e+11
## 1091    Europe              Netherlands 2002 5.437385e+11
## 1092    Europe              Netherlands 2007 6.097643e+11
## 1093   Oceania              New Zealand 1952 2.105819e+10
## 1094   Oceania              New Zealand 1957 2.730443e+10
## 1095   Oceania              New Zealand 1962 3.278833e+10
## 1096   Oceania              New Zealand 1967 3.945974e+10
## 1097   Oceania              New Zealand 1972 4.700045e+10
## 1098   Oceania              New Zealand 1977 5.137809e+10
## 1099   Oceania              New Zealand 1982 5.661150e+10
## 1100   Oceania              New Zealand 1987 6.305001e+10
## 1101   Oceania              New Zealand 1992 6.312712e+10
## 1102   Oceania              New Zealand 1997 7.738526e+10
## 1103   Oceania              New Zealand 2002 9.062660e+10
## 1104   Oceania              New Zealand 2007 1.036557e+11
## 1105  Americas                Nicaragua 1952 3.628363e+09
## 1106  Americas                Nicaragua 1957 4.698034e+09
## 1107  Americas                Nicaragua 1962 5.780809e+09
## 1108  Americas                Nicaragua 1967 8.662204e+09
## 1109  Americas                Nicaragua 1972 1.023477e+10
## 1110  Americas                Nicaragua 1977 1.401547e+10
## 1111  Americas                Nicaragua 1982 1.033961e+10
## 1112  Americas                Nicaragua 1987 9.885855e+09
## 1113  Americas                Nicaragua 1992 8.719537e+09
## 1114  Americas                Nicaragua 1997 1.038547e+10
## 1115  Americas                Nicaragua 2002 1.273613e+10
## 1116  Americas                Nicaragua 2007 1.560338e+10
## 1117    Africa                    Niger 1952 2.574747e+09
## 1118    Africa                    Niger 1957 3.084906e+09
## 1119    Africa                    Niger 1962 4.066903e+09
## 1120    Africa                    Niger 1967 4.780646e+09
## 1121    Africa                    Niger 1972 4.828549e+09
## 1122    Africa                    Niger 1977 4.596223e+09
## 1123    Africa                    Niger 1982 5.856052e+09
## 1124    Africa                    Niger 1987 4.900402e+09
## 1125    Africa                    Niger 1992 4.877761e+09
## 1126    Africa                    Niger 1997 5.609376e+09
## 1127    Africa                    Niger 2002 6.696364e+09
## 1128    Africa                    Niger 2007 7.990650e+09
## 1129    Africa                  Nigeria 1952 3.567860e+10
## 1130    Africa                  Nigeria 1957 4.091270e+10
## 1131    Africa                  Nigeria 1962 4.819089e+10
## 1132    Africa                  Nigeria 1967 4.797409e+10
## 1133    Africa                  Nigeria 1972 9.127156e+10
## 1134    Africa                  Nigeria 1977 1.232956e+11
## 1135    Africa                  Nigeria 1982 1.151812e+11
## 1136    Africa                  Nigeria 1987 1.129513e+11
## 1137    Africa                  Nigeria 1992 1.512359e+11
## 1138    Africa                  Nigeria 1997 1.725815e+11
## 1139    Africa                  Nigeria 2002 1.936749e+11
## 1140    Africa                  Nigeria 2007 2.719497e+11
## 1141    Europe                   Norway 1952 3.359482e+10
## 1142    Europe                   Norway 1957 4.069495e+10
## 1143    Europe                   Norway 1962 4.894492e+10
## 1144    Europe                   Norway 1967 6.194638e+10
## 1145    Europe                   Norway 1972 7.458964e+10
## 1146    Europe                   Norway 1977 9.425256e+10
## 1147    Europe                   Norway 1982 1.082133e+11
## 1148    Europe                   Norway 1987 1.320352e+11
## 1149    Europe                   Norway 1992 1.455889e+11
## 1150    Europe                   Norway 1997 1.818801e+11
## 1151    Europe                   Norway 2002 2.026682e+11
## 1152    Europe                   Norway 2007 2.284214e+11
## 1153      Asia                     Oman 1952 9.284357e+08
## 1154      Asia                     Oman 1957 1.260372e+09
## 1155      Asia                     Oman 1962 1.837152e+09
## 1156      Asia                     Oman 1967 3.374412e+09
## 1157      Asia                     Oman 1972 8.802885e+09
## 1158      Asia                     Oman 1977 1.190205e+10
## 1159      Asia                     Oman 1982 1.685480e+10
## 1160      Asia                     Oman 1987 2.887353e+10
## 1161      Asia                     Oman 1992 3.565487e+10
## 1162      Asia                     Oman 1997 4.499230e+10
## 1163      Asia                     Oman 2002 5.365827e+10
## 1164      Asia                     Oman 2007 7.152110e+10
## 1165      Asia                 Pakistan 1952 2.830574e+10
## 1166      Asia                 Pakistan 1957 3.487382e+10
## 1167      Asia                 Pakistan 1962 4.265804e+10
## 1168      Asia                 Pakistan 1967 5.714943e+10
## 1169      Asia                 Pakistan 1972 7.278799e+10
## 1170      Asia                 Pakistan 1977 9.190140e+10
## 1171      Asia                 Pakistan 1982 1.320191e+11
## 1172      Asia                 Pakistan 1987 1.793107e+11
## 1173      Asia                 Pakistan 1992 2.367477e+11
## 1174      Asia                 Pakistan 1997 2.778199e+11
## 1175      Asia                 Pakistan 2002 3.210295e+11
## 1176      Asia                 Pakistan 2007 4.411104e+11
## 1177  Americas                   Panama 1952 2.331756e+09
## 1178  Americas                   Panama 1957 3.149893e+09
## 1179  Americas                   Panama 1962 4.299460e+09
## 1180  Americas                   Panama 1967 6.213666e+09
## 1181  Americas                   Panama 1972 8.670687e+09
## 1182  Americas                   Panama 1977 9.846352e+09
## 1183  Americas                   Panama 1982 1.427369e+10
## 1184  Americas                   Panama 1987 1.585385e+10
## 1185  Americas                   Panama 1992 1.644756e+10
## 1186  Americas                   Panama 1997 1.945261e+10
## 1187  Americas                   Panama 2002 2.200097e+10
## 1188  Americas                   Panama 2007 3.180308e+10
## 1189  Americas                 Paraguay 1952 3.037550e+09
## 1190  Americas                 Paraguay 1957 3.623539e+09
## 1191  Americas                 Paraguay 1962 4.317133e+09
## 1192  Americas                 Paraguay 1967 5.260939e+09
## 1193  Americas                 Paraguay 1972 6.596268e+09
## 1194  Americas                 Paraguay 1977 9.694751e+09
## 1195  Americas                 Paraguay 1982 1.433599e+10
## 1196  Americas                 Paraguay 1987 1.554168e+10
## 1197  Americas                 Paraguay 1992 1.881648e+10
## 1198  Americas                 Paraguay 1997 2.189162e+10
## 1199  Americas                 Paraguay 2002 2.226500e+10
## 1200  Americas                 Paraguay 2007 2.782093e+10
## 1201  Americas                     Peru 1952 3.016478e+10
## 1202  Americas                     Peru 1957 3.882754e+10
## 1203  Americas                     Peru 1962 5.213069e+10
## 1204  Americas                     Peru 1967 7.022231e+10
## 1205  Americas                     Peru 1972 8.286060e+10
## 1206  Americas                     Peru 1977 1.004385e+11
## 1207  Americas                     Peru 1982 1.166262e+11
## 1208  Americas                     Peru 1987 1.284651e+11
## 1209  Americas                     Peru 1992 9.973432e+10
## 1210  Americas                     Peru 1997 1.444881e+11
## 1211  Americas                     Peru 2002 1.581811e+11
## 1212  Americas                     Peru 2007 2.124486e+11
## 1213      Asia              Philippines 1952 2.856178e+10
## 1214      Asia              Philippines 1957 4.035832e+10
## 1215      Asia              Philippines 1962 5.002310e+10
## 1216      Asia              Philippines 1967 6.414138e+10
## 1217      Asia              Philippines 1972 8.126621e+10
## 1218      Asia              Philippines 1977 1.111869e+11
## 1219      Asia              Philippines 1982 1.391626e+11
## 1220      Asia              Philippines 1987 1.314170e+11
## 1221      Asia              Philippines 1992 1.531381e+11
## 1222      Asia              Philippines 1997 1.902731e+11
## 1223      Asia              Philippines 2002 2.200134e+11
## 1224      Asia              Philippines 2007 2.905804e+11
## 1225    Europe                   Poland 1952 1.036769e+11
## 1226    Europe                   Poland 1957 1.336733e+11
## 1227    Europe                   Poland 1962 1.619223e+11
## 1228    Europe                   Poland 1967 2.084216e+11
## 1229    Europe                   Poland 1972 2.645313e+11
## 1230    Europe                   Poland 1977 3.291838e+11
## 1231    Europe                   Poland 1982 3.061768e+11
## 1232    Europe                   Poland 1987 3.427744e+11
## 1233    Europe                   Poland 1992 2.969463e+11
## 1234    Europe                   Poland 1997 3.927183e+11
## 1235    Europe                   Poland 2002 4.635982e+11
## 1236    Europe                   Poland 2007 5.927928e+11
## 1237    Europe                 Portugal 1952 2.616065e+10
## 1238    Europe                 Portugal 1957 3.328285e+10
## 1239    Europe                 Portugal 1962 4.264521e+10
## 1240    Europe                 Portugal 1967 5.790890e+10
## 1241    Europe                 Portugal 1972 8.093362e+10
## 1242    Europe                 Portugal 1977 9.829266e+10
## 1243    Europe                 Portugal 1982 1.158888e+11
## 1244    Europe                 Portugal 1987 1.292885e+11
## 1245    Europe                 Portugal 1992 1.609006e+11
## 1246    Europe                 Portugal 1997 1.791696e+11
## 1247    Europe                 Portugal 2002 2.083738e+11
## 1248    Europe                 Portugal 2007 2.182808e+11
## 1249  Americas              Puerto Rico 1952 6.863524e+09
## 1250  Americas              Puerto Rico 1957 8.830173e+09
## 1251  Americas              Puerto Rico 1962 1.250546e+10
## 1252  Americas              Puerto Rico 1967 1.835539e+10
## 1253  Americas              Puerto Rico 1972 2.597450e+10
## 1254  Americas              Puerto Rico 1977 3.010131e+10
## 1255  Americas              Puerto Rico 1982 3.387532e+10
## 1256  Americas              Puerto Rico 1987 4.230269e+10
## 1257  Americas              Puerto Rico 1992 5.249267e+10
## 1258  Americas              Puerto Rico 1997 6.390818e+10
## 1259  Americas              Puerto Rico 2002 7.277521e+10
## 1260  Americas              Puerto Rico 2007 7.620326e+10
## 1261    Africa                  Reunion 1952 7.006567e+08
## 1262    Africa                  Reunion 1957 8.549298e+08
## 1263    Africa                  Reunion 1962 1.139049e+09
## 1264    Africa                  Reunion 1967 1.664863e+09
## 1265    Africa                  Reunion 1972 2.330166e+09
## 1266    Africa                  Reunion 1977 2.125754e+09
## 1267    Africa                  Reunion 1982 2.727419e+09
## 1268    Africa                  Reunion 1987 2.980684e+09
## 1269    Africa                  Reunion 1992 3.796146e+09
## 1270    Africa                  Reunion 1997 4.158126e+09
## 1271    Africa                  Reunion 2002 4.699107e+09
## 1272    Africa                  Reunion 2007 6.121479e+09
## 1273    Europe                  Romania 1952 5.229492e+10
## 1274    Europe                  Romania 1957 7.030764e+10
## 1275    Europe                  Romania 1962 8.845317e+10
## 1276    Europe                  Romania 1967 1.247895e+11
## 1277    Europe                  Romania 1972 1.655370e+11
## 1278    Europe                  Romania 1977 2.026464e+11
## 1279    Europe                  Romania 1982 2.147434e+11
## 1280    Europe                  Romania 1987 2.199733e+11
## 1281    Europe                  Romania 1992 1.504241e+11
## 1282    Europe                  Romania 1997 1.657562e+11
## 1283    Europe                  Romania 2002 1.766663e+11
## 1284    Europe                  Romania 2007 2.407702e+11
## 1285    Africa                   Rwanda 1952 1.250540e+09
## 1286    Africa                   Rwanda 1957 1.524741e+09
## 1287    Africa                   Rwanda 1962 1.823035e+09
## 1288    Africa                   Rwanda 1967 1.763376e+09
## 1289    Africa                   Rwanda 1972 2.357669e+09
## 1290    Africa                   Rwanda 1977 3.120614e+09
## 1291    Africa                   Rwanda 1982 4.855308e+09
## 1292    Africa                   Rwanda 1987 5.384206e+09
## 1293    Africa                   Rwanda 1992 5.373380e+09
## 1294    Africa                   Rwanda 1997 4.255024e+09
## 1295    Africa                   Rwanda 2002 6.169268e+09
## 1296    Africa                   Rwanda 2007 7.647471e+09
## 1297    Africa    Sao Tome and Principe 1952 5.278469e+07
## 1298    Africa    Sao Tome and Principe 1957 5.278469e+07
## 1299    Africa    Sao Tome and Principe 1962 7.002051e+07
## 1300    Africa    Sao Tome and Principe 1967 9.802871e+07
## 1301    Africa    Sao Tome and Principe 1972 1.174190e+08
## 1302    Africa    Sao Tome and Principe 1977 1.508134e+08
## 1303    Africa    Sao Tome and Principe 1982 1.863623e+08
## 1304    Africa    Sao Tome and Principe 1987 1.680492e+08
## 1305    Africa    Sao Tome and Principe 1992 1.798988e+08
## 1306    Africa    Sao Tome and Principe 1997 1.949802e+08
## 1307    Africa    Sao Tome and Principe 2002 2.305291e+08
## 1308    Africa    Sao Tome and Principe 2007 3.190141e+08
## 1309      Asia             Saudi Arabia 1952 2.587489e+10
## 1310      Asia             Saudi Arabia 1957 3.605370e+10
## 1311      Asia             Saudi Arabia 1962 5.746973e+10
## 1312      Asia             Saudi Arabia 1967 9.496468e+10
## 1313      Asia             Saudi Arabia 1972 1.607666e+11
## 1314      Asia             Saudi Arabia 1977 2.777328e+11
## 1315      Asia             Saudi Arabia 1982 3.792056e+11
## 1316      Asia             Saudi Arabia 1987 3.099132e+11
## 1317      Asia             Saudi Arabia 1992 4.209625e+11
## 1318      Asia             Saudi Arabia 1997 4.370505e+11
## 1319      Asia             Saudi Arabia 2002 4.658854e+11
## 1320      Asia             Saudi Arabia 2007 5.976958e+11
## 1321    Africa                  Senegal 1952 3.996588e+09
## 1322    Africa                  Senegal 1957 4.788470e+09
## 1323    Africa                  Senegal 1962 5.677013e+09
## 1324    Africa                  Senegal 1967 6.394540e+09
## 1325    Africa                  Senegal 1972 7.331415e+09
## 1326    Africa                  Senegal 1977 8.216241e+09
## 1327    Africa                  Senegal 1982 9.335285e+09
## 1328    Africa                  Senegal 1987 1.033908e+10
## 1329    Africa                  Senegal 1992 1.136440e+10
## 1330    Africa                  Senegal 1997 1.327667e+10
## 1331    Africa                  Senegal 2002 1.651849e+10
## 1332    Africa                  Senegal 2007 2.100774e+10
## 1333    Europe                   Serbia 1952 2.456934e+10
## 1334    Europe                   Serbia 1957 3.621818e+10
## 1335    Europe                   Serbia 1962 4.790219e+10
## 1336    Europe                   Serbia 1967 6.370367e+10
## 1337    Europe                   Serbia 1972 8.747298e+10
## 1338    Europe                   Serbia 1977 1.127549e+11
## 1339    Europe                   Serbia 1982 1.371281e+11
## 1340    Europe                   Serbia 1987 1.465006e+11
## 1341    Europe                   Serbia 1992 9.163182e+10
## 1342    Europe                   Serbia 1997 8.180712e+10
## 1343    Europe                   Serbia 2002 7.316800e+10
## 1344    Europe                   Serbia 2007 9.933592e+10
## 1345    Africa             Sierra Leone 1952 1.885604e+09
## 1346    Africa             Sierra Leone 1957 2.305973e+09
## 1347    Africa             Sierra Leone 1962 2.755750e+09
## 1348    Africa             Sierra Leone 1967 3.210717e+09
## 1349    Africa             Sierra Leone 1972 3.897492e+09
## 1350    Africa             Sierra Leone 1977 4.234825e+09
## 1351    Africa             Sierra Leone 1982 5.075562e+09
## 1352    Africa             Sierra Leone 1987 5.008096e+09
## 1353    Africa             Sierra Leone 1992 4.553591e+09
## 1354    Africa             Sierra Leone 1997 2.630861e+09
## 1355    Africa             Sierra Leone 2002 3.748630e+09
## 1356    Africa             Sierra Leone 2007 5.299935e+09
## 1357      Asia                Singapore 1952 2.609161e+09
## 1358      Asia                Singapore 1957 4.110927e+09
## 1359      Asia                Singapore 1962 6.431522e+09
## 1360      Asia                Singapore 1967 9.843343e+09
## 1361      Asia                Singapore 1972 1.850581e+10
## 1362      Asia                Singapore 1977 2.606682e+10
## 1363      Asia                Singapore 1982 4.022663e+10
## 1364      Asia                Singapore 1987 5.270953e+10
## 1365      Asia                Singapore 1992 8.015202e+10
## 1366      Asia                Singapore 1997 1.274514e+11
## 1367      Asia                Singapore 2002 1.512169e+11
## 1368      Asia                Singapore 2007 2.146433e+11
## 1369    Europe          Slovak Republic 1952 1.805633e+10
## 1370    Europe          Slovak Republic 1957 2.342419e+10
## 1371    Europe          Slovak Republic 1962 3.170033e+10
## 1372    Europe          Slovak Republic 1967 3.737211e+10
## 1373    Europe          Slovak Republic 1972 4.443764e+10
## 1374    Europe          Slovak Republic 1977 5.273247e+10
## 1375    Europe          Slovak Republic 1982 5.728795e+10
## 1376    Europe          Slovak Republic 1987 6.258558e+10
## 1377    Europe          Slovak Republic 1992 5.036931e+10
## 1378    Europe          Slovak Republic 1997 6.527562e+10
## 1379    Europe          Slovak Republic 2002 7.378650e+10
## 1380    Europe          Slovak Republic 2007 1.017502e+11
## 1381    Europe                 Slovenia 1952 6.278381e+09
## 1382    Europe                 Slovenia 1957 8.987280e+09
## 1383    Europe                 Slovenia 1962 1.171756e+10
## 1384    Europe                 Slovenia 1967 1.549001e+10
## 1385    Europe                 Slovenia 1972 2.098394e+10
## 1386    Europe                 Slovenia 1977 2.668773e+10
## 1387    Europe                 Slovenia 1982 3.325447e+10
## 1388    Europe                 Slovenia 1987 3.634600e+10
## 1389    Europe                 Slovenia 1992 2.841820e+10
## 1390    Europe                 Slovenia 1997 3.452149e+10
## 1391    Europe                 Slovenia 2002 4.155757e+10
## 1392    Europe                 Slovenia 2007 5.177474e+10
## 1393    Africa                  Somalia 1952 2.870033e+09
## 1394    Africa                  Somalia 1957 3.498172e+09
## 1395    Africa                  Somalia 1962 4.218234e+09
## 1396    Africa                  Somalia 1967 4.405143e+09
## 1397    Africa                  Somalia 1972 4.817774e+09
## 1398    Africa                  Somalia 1977 6.317137e+09
## 1399    Africa                  Somalia 1982 6.859481e+09
## 1400    Africa                  Somalia 1987 7.567286e+09
## 1401    Africa                  Somalia 1992 5.654271e+09
## 1402    Africa                  Somalia 1997 6.173124e+09
## 1403    Africa                  Somalia 2002 6.839054e+09
## 1404    Africa                  Somalia 2007 8.445270e+09
## 1405    Africa             South Africa 1952 6.740603e+10
## 1406    Africa             South Africa 1957 8.862523e+10
## 1407    Africa             South Africa 1962 1.058946e+11
## 1408    Africa             South Africa 1967 1.493850e+11
## 1409    Africa             South Africa 1972 1.858846e+11
## 1410    Africa             South Africa 1977 2.178168e+11
## 1411    Africa             South Africa 1982 2.668161e+11
## 1412    Africa             South Africa 1987 2.812083e+11
## 1413    Africa             South Africa 1992 2.887438e+11
## 1414    Africa             South Africa 1997 3.203711e+11
## 1415    Africa             South Africa 2002 3.426253e+11
## 1416    Africa             South Africa 2007 4.078448e+11
## 1417    Europe                    Spain 1952 1.094612e+11
## 1418    Europe                    Spain 1957 1.362211e+11
## 1419    Europe                    Spain 1962 1.774091e+11
## 1420    Europe                    Spain 1967 2.625891e+11
## 1421    Europe                    Spain 1972 3.671769e+11
## 1422    Europe                    Spain 1977 4.823402e+11
## 1423    Europe                    Spain 1982 5.289620e+11
## 1424    Europe                    Spain 1987 6.129536e+11
## 1425    Europe                    Spain 1992 7.357407e+11
## 1426    Europe                    Spain 1997 8.148564e+11
## 1427    Europe                    Spain 2002 9.972067e+11
## 1428    Europe                    Spain 2007 1.165760e+12
## 1429      Asia                Sri Lanka 1952 8.649123e+09
## 1430      Asia                Sri Lanka 1957 9.790791e+09
## 1431      Asia                Sri Lanka 1962 1.119808e+10
## 1432      Asia                Sri Lanka 1967 1.332798e+10
## 1433      Asia                Sri Lanka 1972 1.579445e+10
## 1434      Asia                Sri Lanka 1977 1.904044e+10
## 1435      Asia                Sri Lanka 1982 2.539716e+10
## 1436      Asia                Sri Lanka 1987 3.095784e+10
## 1437      Asia                Sri Lanka 1992 3.787794e+10
## 1438      Asia                Sri Lanka 1997 4.982214e+10
## 1439      Asia                Sri Lanka 2002 5.903142e+10
## 1440      Asia                Sri Lanka 2007 8.090355e+10
## 1441    Africa                    Sudan 1952 1.374347e+10
## 1442    Africa                    Sudan 1957 1.726679e+10
## 1443    Africa                    Sudan 1962 2.191458e+10
## 1444    Africa                    Sudan 1967 2.146480e+10
## 1445    Africa                    Sudan 1972 2.422598e+10
## 1446    Africa                    Sudan 1977 3.768209e+10
## 1447    Africa                    Sudan 1982 3.860665e+10
## 1448    Africa                    Sudan 1987 3.728228e+10
## 1449    Africa                    Sudan 1992 4.212112e+10
## 1450    Africa                    Sudan 1997 5.249309e+10
## 1451    Africa                    Sudan 2002 7.393574e+10
## 1452    Africa                    Sudan 2007 1.100629e+11
## 1453    Africa                Swaziland 1952 3.333083e+08
## 1454    Africa                Swaziland 1957 4.066973e+08
## 1455    Africa                Swaziland 1962 6.867985e+08
## 1456    Africa                Swaziland 1967 1.099306e+09
## 1457    Africa                Swaziland 1972 1.615475e+09
## 1458    Africa                Swaziland 1977 2.085164e+09
## 1459    Africa                Swaziland 1982 2.531614e+09
## 1460    Africa                Swaziland 1987 3.105577e+09
## 1461    Africa                Swaziland 1992 3.419230e+09
## 1462    Africa                Swaziland 1997 4.087998e+09
## 1463    Africa                Swaziland 2002 4.665883e+09
## 1464    Africa                Swaziland 2007 5.114071e+09
## 1465    Europe                   Sweden 1952 6.075810e+10
## 1466    Europe                   Sweden 1957 7.298911e+10
## 1467    Europe                   Sweden 1962 9.323016e+10
## 1468    Europe                   Sweden 1967 1.200512e+11
## 1469    Europe                   Sweden 1972 1.448369e+11
## 1470    Europe                   Sweden 1977 1.555908e+11
## 1471    Europe                   Sweden 1982 1.720613e+11
## 1472    Europe                   Sweden 1987 1.986350e+11
## 1473    Europe                   Sweden 1992 2.082067e+11
## 1474    Europe                   Sweden 1997 2.248125e+11
## 1475    Europe                   Sweden 2002 2.627301e+11
## 1476    Europe                   Sweden 2007 3.057904e+11
## 1477    Europe              Switzerland 1952 7.094533e+10
## 1478    Europe              Switzerland 1957 9.180404e+10
## 1479    Europe              Switzerland 1962 1.157626e+11
## 1480    Europe              Switzerland 1967 1.392437e+11
## 1481    Europe              Switzerland 1972 1.740868e+11
## 1482    Europe              Switzerland 1977 1.704316e+11
## 1483    Europe              Switzerland 1982 1.836800e+11
## 1484    Europe              Switzerland 1987 2.013716e+11
## 1485    Europe              Switzerland 1992 2.229556e+11
## 1486    Europe              Switzerland 1997 2.311738e+11
## 1487    Europe              Switzerland 2002 2.538404e+11
## 1488    Europe              Switzerland 2007 2.833483e+11
## 1489      Asia                    Syria 1952 6.017702e+09
## 1490      Asia                    Syria 1957 8.786330e+09
## 1491      Asia                    Syria 1962 1.060250e+10
## 1492      Asia                    Syria 1967 1.069085e+10
## 1493      Asia                    Syria 1972 1.723155e+10
## 1494      Asia                    Syria 1977 2.534819e+10
## 1495      Asia                    Syria 1982 3.540075e+10
## 1496      Asia                    Syria 1987 3.504142e+10
## 1497      Asia                    Syria 1992 4.415884e+10
## 1498      Asia                    Syria 1997 6.053880e+10
## 1499      Asia                    Syria 2002 7.018315e+10
## 1500      Asia                    Syria 2007 8.082349e+10
## 1501      Asia                   Taiwan 1952 1.031984e+10
## 1502      Asia                   Taiwan 1957 1.532623e+10
## 1503      Asia                   Taiwan 1962 2.172678e+10
## 1504      Asia                   Taiwan 1967 3.608521e+10
## 1505      Asia                   Taiwan 1972 6.185615e+10
## 1506      Asia                   Taiwan 1977 9.393868e+10
## 1507      Asia                   Taiwan 1982 1.373979e+11
## 1508      Asia                   Taiwan 1987 2.184138e+11
## 1509      Asia                   Taiwan 1992 3.147651e+11
## 1510      Asia                   Taiwan 1997 4.370453e+11
## 1511      Asia                   Taiwan 2002 5.217337e+11
## 1512      Asia                   Taiwan 2007 6.655258e+11
## 1513    Africa                 Tanzania 1952 5.964625e+09
## 1514    Africa                 Tanzania 1957 6.603136e+09
## 1515    Africa                 Tanzania 1962 7.843819e+09
## 1516    Africa                 Tanzania 1967 1.069376e+10
## 1517    Africa                 Tanzania 1972 1.347102e+10
## 1518    Africa                 Tanzania 1977 1.648707e+10
## 1519    Africa                 Tanzania 1982 1.734880e+10
## 1520    Africa                 Tanzania 1987 1.916570e+10
## 1521    Africa                 Tanzania 1992 2.196767e+10
## 1522    Africa                 Tanzania 1997 2.421767e+10
## 1523    Africa                 Tanzania 2002 3.110237e+10
## 1524    Africa                 Tanzania 2007 4.223897e+10
## 1525      Asia                 Thailand 1952 1.613305e+10
## 1526      Asia                 Thailand 1957 1.987270e+10
## 1527      Asia                 Thailand 1962 2.932775e+10
## 1528      Asia                 Thailand 1967 4.407708e+10
## 1529      Asia                 Thailand 1972 5.987095e+10
## 1530      Asia                 Thailand 1977 8.658470e+10
## 1531      Asia                 Thailand 1982 1.168541e+11
## 1532      Asia                 Thailand 1987 1.578132e+11
## 1533      Asia                 Thailand 1992 2.616261e+11
## 1534      Asia                 Thailand 1997 3.524257e+11
## 1535      Asia                 Thailand 2002 3.713881e+11
## 1536      Asia                 Thailand 2007 4.853040e+11
## 1537    Africa                     Togo 1952 1.048204e+09
## 1538    Africa                     Togo 1957 1.256870e+09
## 1539    Africa                     Togo 1962 1.631298e+09
## 1540    Africa                     Togo 1967 2.564443e+09
## 1541    Africa                     Togo 1972 3.392280e+09
## 1542    Africa                     Togo 1977 3.538541e+09
## 1543    Africa                     Togo 1982 3.556093e+09
## 1544    Africa                     Togo 1987 3.792060e+09
## 1545    Africa                     Togo 1992 3.876090e+09
## 1546    Africa                     Togo 1997 4.244354e+09
## 1547    Africa                     Togo 2002 4.411055e+09
## 1548    Africa                     Togo 2007 5.034323e+09
## 1549  Americas      Trinidad and Tobago 1952 2.003976e+09
## 1550  Americas      Trinidad and Tobago 1957 3.136391e+09
## 1551  Americas      Trinidad and Tobago 1962 4.435293e+09
## 1552  Americas      Trinidad and Tobago 1967 5.397385e+09
## 1553  Americas      Trinidad and Tobago 1972 6.455380e+09
## 1554  Americas      Trinidad and Tobago 1977 8.207708e+09
## 1555  Americas      Trinidad and Tobago 1982 1.018176e+10
## 1556  Americas      Trinidad and Tobago 1987 8.802303e+09
## 1557  Americas      Trinidad and Tobago 1992 8.724813e+09
## 1558  Americas      Trinidad and Tobago 1997 1.000684e+10
## 1559  Americas      Trinidad and Tobago 2002 1.262766e+10
## 1560  Americas      Trinidad and Tobago 2007 1.902793e+10
## 1561    Africa                  Tunisia 1952 5.356610e+09
## 1562    Africa                  Tunisia 1957 5.512353e+09
## 1563    Africa                  Tunisia 1962 7.116976e+09
## 1564    Africa                  Tunisia 1967 9.250181e+09
## 1565    Africa                  Tunisia 1972 1.460207e+10
## 1566    Africa                  Tunisia 1977 1.874106e+10
## 1567    Africa                  Tunisia 1982 2.397496e+10
## 1568    Africa                  Tunisia 1987 2.943540e+10
## 1569    Africa                  Tunisia 1992 3.692811e+10
## 1570    Africa                  Tunisia 1997 4.502099e+10
## 1571    Africa                  Tunisia 2002 5.591598e+10
## 1572    Africa                  Tunisia 2007 7.288800e+10
## 1573    Europe                   Turkey 1952 4.378429e+10
## 1574    Europe                   Turkey 1957 5.695751e+10
## 1575    Europe                   Turkey 1962 6.919526e+10
## 1576    Europe                   Turkey 1967 9.443229e+10
## 1577    Europe                   Turkey 1972 1.293768e+11
## 1578    Europe                   Turkey 1977 1.810280e+11
## 1579    Europe                   Turkey 1982 2.007383e+11
## 1580    Europe                   Turkey 1987 2.691154e+11
## 1581    Europe                   Turkey 1992 3.303614e+11
## 1582    Europe                   Turkey 1997 4.162046e+11
## 1583    Europe                   Turkey 2002 4.380523e+11
## 1584    Europe                   Turkey 2007 6.018795e+11
## 1585    Africa                   Uganda 1952 4.279790e+09
## 1586    Africa                   Uganda 1957 5.169315e+09
## 1587    Africa                   Uganda 1962 5.899397e+09
## 1588    Africa                   Uganda 1967 8.089642e+09
## 1589    Africa                   Uganda 1972 9.688269e+09
## 1590    Africa                   Uganda 1977 9.667290e+09
## 1591    Africa                   Uganda 1982 8.828116e+09
## 1592    Africa                   Uganda 1987 9.440713e+09
## 1593    Africa                   Uganda 1992 1.175753e+10
## 1594    Africa                   Uganda 1997 1.731943e+10
## 1595    Africa                   Uganda 2002 2.295170e+10
## 1596    Africa                   Uganda 2007 3.081503e+10
## 1597    Europe           United Kingdom 1952 5.032666e+11
## 1598    Europe           United Kingdom 1957 5.802938e+11
## 1599    Europe           United Kingdom 1962 6.649337e+11
## 1600    Europe           United Kingdom 1967 7.772769e+11
## 1601    Europe           United Kingdom 1972 8.913822e+11
## 1602    Europe           United Kingdom 1977 9.791297e+11
## 1603    Europe           United Kingdom 1982 1.027209e+12
## 1604    Europe           United Kingdom 1987 1.234495e+12
## 1605    Europe           United Kingdom 1992 1.313861e+12
## 1606    Europe           United Kingdom 1997 1.533398e+12
## 1607    Europe           United Kingdom 2002 1.766159e+12
## 1608    Europe           United Kingdom 2007 2.017969e+12
## 1609  Americas            United States 1952 2.204242e+12
## 1610  Americas            United States 1957 2.553468e+12
## 1611  Americas            United States 1962 3.016906e+12
## 1612  Americas            United States 1967 3.880918e+12
## 1613  Americas            United States 1972 4.577000e+12
## 1614  Americas            United States 1977 5.301732e+12
## 1615  Americas            United States 1982 5.806915e+12
## 1616  Americas            United States 1987 7.256026e+12
## 1617  Americas            United States 1992 8.221624e+12
## 1618  Americas            United States 1997 9.761353e+12
## 1619  Americas            United States 2002 1.124728e+13
## 1620  Americas            United States 2007 1.293446e+13
## 1621  Americas                  Uruguay 1952 1.287968e+10
## 1622  Americas                  Uruguay 1957 1.491537e+10
## 1623  Americas                  Uruguay 1962 1.456013e+10
## 1624  Americas                  Uruguay 1967 1.496497e+10
## 1625  Americas                  Uruguay 1972 1.613794e+10
## 1626  Americas                  Uruguay 1977 1.869035e+10
## 1627  Americas                  Uruguay 1982 2.044232e+10
## 1628  Americas                  Uruguay 1987 2.269370e+10
## 1629  Americas                  Uruguay 1992 2.562556e+10
## 1630  Americas                  Uruguay 1997 3.011678e+10
## 1631  Americas                  Uruguay 2002 2.598656e+10
## 1632  Americas                  Uruguay 2007 3.658298e+10
## 1633  Americas                Venezuela 1952 4.182919e+10
## 1634  Americas                Venezuela 1957 6.570268e+10
## 1635  Americas                Venezuela 1962 6.859144e+10
## 1636  Americas                Venezuela 1967 9.264344e+10
## 1637  Americas                Venezuela 1972 1.209749e+11
## 1638  Americas                Venezuela 1977 1.774902e+11
## 1639  Americas                Venezuela 1982 1.742092e+11
## 1640  Americas                Venezuela 1987 1.770168e+11
## 1641  Americas                Venezuela 1992 2.175291e+11
## 1642  Americas                Venezuela 1997 2.274468e+11
## 1643  Americas                Venezuela 2002 2.089966e+11
## 1644  Americas                Venezuela 2007 2.977774e+11
## 1645      Asia                  Vietnam 1952 1.588108e+10
## 1646      Asia                  Vietnam 1957 1.961129e+10
## 1647      Asia                  Vietnam 1962 2.609228e+10
## 1648      Asia                  Vietnam 1967 2.514338e+10
## 1649      Asia                  Vietnam 1972 3.123626e+10
## 1650      Asia                  Vietnam 1977 3.605753e+10
## 1651      Asia                  Vietnam 1982 3.970576e+10
## 1652      Asia                  Vietnam 1987 5.156795e+10
## 1653      Asia                  Vietnam 1992 6.917300e+10
## 1654      Asia                  Vietnam 1997 1.053961e+11
## 1655      Asia                  Vietnam 2002 1.427589e+11
## 1656      Asia                  Vietnam 2007 2.081746e+11
## 1657      Asia       West Bank and Gaza 1952 1.561947e+09
## 1658      Asia       West Bank and Gaza 1957 1.955765e+09
## 1659      Asia       West Bank and Gaza 1962 2.491712e+09
## 1660      Asia       West Bank and Gaza 1967 3.027660e+09
## 1661      Asia       West Bank and Gaza 1972 3.414075e+09
## 1662      Asia       West Bank and Gaza 1977 4.644386e+09
## 1663      Asia       West Bank and Gaza 1982 6.182644e+09
## 1664      Asia       West Bank and Gaza 1987 8.637343e+09
## 1665      Asia       West Bank and Gaza 1992 1.266583e+10
## 1666      Asia       West Bank and Gaza 1997 2.009507e+10
## 1667      Asia       West Bank and Gaza 2002 1.530560e+10
## 1668      Asia       West Bank and Gaza 2007 1.215686e+10
## 1669      Asia               Yemen Rep. 1952 3.880312e+09
## 1670      Asia               Yemen Rep. 1957 4.425030e+09
## 1671      Asia               Yemen Rep. 1962 5.052881e+09
## 1672      Asia               Yemen Rep. 1967 5.813537e+09
## 1673      Asia               Yemen Rep. 1972 9.370298e+09
## 1674      Asia               Yemen Rep. 1977 1.537733e+10
## 1675      Asia               Yemen Rep. 1982 1.909849e+10
## 1676      Asia               Yemen Rep. 1987 2.212164e+10
## 1677      Asia               Yemen Rep. 1992 2.512511e+10
## 1678      Asia               Yemen Rep. 1997 3.351236e+10
## 1679      Asia               Yemen Rep. 2002 4.179396e+10
## 1680      Asia               Yemen Rep. 2007 5.065987e+10
## 1681    Africa                   Zambia 1952 3.065823e+09
## 1682    Africa                   Zambia 1957 3.956862e+09
## 1683    Africa                   Zambia 1962 4.969775e+09
## 1684    Africa                   Zambia 1967 6.930602e+09
## 1685    Africa                   Zambia 1972 7.992265e+09
## 1686    Africa                   Zambia 1977 8.287472e+09
## 1687    Africa                   Zambia 1982 8.593513e+09
## 1688    Africa                   Zambia 1987 8.823720e+09
## 1689    Africa                   Zambia 1992 1.014862e+10
## 1690    Africa                   Zambia 1997 1.008978e+10
## 1691    Africa                   Zambia 2002 1.135462e+10
## 1692    Africa                   Zambia 2007 1.493170e+10
## 1693    Africa                 Zimbabwe 1952 1.253572e+09
## 1694    Africa                 Zimbabwe 1957 1.891591e+09
## 1695    Africa                 Zimbabwe 1962 2.255531e+09
## 1696    Africa                 Zimbabwe 1967 2.846373e+09
## 1697    Africa                 Zimbabwe 1972 4.685170e+09
## 1698    Africa                 Zimbabwe 1977 4.553747e+09
## 1699    Africa                 Zimbabwe 1982 6.024110e+09
## 1700    Africa                 Zimbabwe 1987 6.508241e+09
## 1701    Africa                 Zimbabwe 1992 7.422612e+09
## 1702    Africa                 Zimbabwe 1997 9.037851e+09
## 1703    Africa                 Zimbabwe 2002 8.015111e+09
## 1704    Africa                 Zimbabwe 2007 5.782658e+09
```

To create a new column in the data table, we have to use the special operator 
`:=`:


```r
# Add a new column to the gapminder data with total gdp
gap[, total_gdp := gdpPercap * pop]
# data frame equivalent
gap_df <- cbind(gap_df, total_gdp = gap_df$gdpPercap * gap_df$pop)
```

To delete a column, we assign it `NULL`


```r
# Delete the total_gdp column
gap[, total_gdp := NULL]
# data frame equivalent
gap_df <- gap_df[, -which(names(gap_df) == "total_gdp")]
```

### Data table specific operations

Data tables have a number of special variables that are useful in calculations:


```r
# get all the columns
gap[,.SD]
```

```
##           country year      pop continent lifeExp gdpPercap
##    1: Afghanistan 1952  8425333      Asia  28.801  779.4453
##    2: Afghanistan 1957  9240934      Asia  30.332  820.8530
##    3: Afghanistan 1962 10267083      Asia  31.997  853.1007
##    4: Afghanistan 1967 11537966      Asia  34.020  836.1971
##    5: Afghanistan 1972 13079460      Asia  36.088  739.9811
##   ---                                                      
## 1700:    Zimbabwe 1987  9216418    Africa  62.351  706.1573
## 1701:    Zimbabwe 1992 10704340    Africa  60.377  693.4208
## 1702:    Zimbabwe 1997 11404948    Africa  46.809  792.4500
## 1703:    Zimbabwe 2002 11926563    Africa  39.989  672.0386
## 1704:    Zimbabwe 2007 12311143    Africa  43.487  469.7093
```

```r
# get the number of rows
gap[,.N]
```

```
## [1] 1704
```

```r
# generate indices for the rows
gap[,.I]
```

```
##    [1]    1    2    3    4    5    6    7    8    9   10   11   12   13
##   [14]   14   15   16   17   18   19   20   21   22   23   24   25   26
##   [27]   27   28   29   30   31   32   33   34   35   36   37   38   39
##   [40]   40   41   42   43   44   45   46   47   48   49   50   51   52
##   [53]   53   54   55   56   57   58   59   60   61   62   63   64   65
##   [66]   66   67   68   69   70   71   72   73   74   75   76   77   78
##   [79]   79   80   81   82   83   84   85   86   87   88   89   90   91
##   [92]   92   93   94   95   96   97   98   99  100  101  102  103  104
##  [105]  105  106  107  108  109  110  111  112  113  114  115  116  117
##  [118]  118  119  120  121  122  123  124  125  126  127  128  129  130
##  [131]  131  132  133  134  135  136  137  138  139  140  141  142  143
##  [144]  144  145  146  147  148  149  150  151  152  153  154  155  156
##  [157]  157  158  159  160  161  162  163  164  165  166  167  168  169
##  [170]  170  171  172  173  174  175  176  177  178  179  180  181  182
##  [183]  183  184  185  186  187  188  189  190  191  192  193  194  195
##  [196]  196  197  198  199  200  201  202  203  204  205  206  207  208
##  [209]  209  210  211  212  213  214  215  216  217  218  219  220  221
##  [222]  222  223  224  225  226  227  228  229  230  231  232  233  234
##  [235]  235  236  237  238  239  240  241  242  243  244  245  246  247
##  [248]  248  249  250  251  252  253  254  255  256  257  258  259  260
##  [261]  261  262  263  264  265  266  267  268  269  270  271  272  273
##  [274]  274  275  276  277  278  279  280  281  282  283  284  285  286
##  [287]  287  288  289  290  291  292  293  294  295  296  297  298  299
##  [300]  300  301  302  303  304  305  306  307  308  309  310  311  312
##  [313]  313  314  315  316  317  318  319  320  321  322  323  324  325
##  [326]  326  327  328  329  330  331  332  333  334  335  336  337  338
##  [339]  339  340  341  342  343  344  345  346  347  348  349  350  351
##  [352]  352  353  354  355  356  357  358  359  360  361  362  363  364
##  [365]  365  366  367  368  369  370  371  372  373  374  375  376  377
##  [378]  378  379  380  381  382  383  384  385  386  387  388  389  390
##  [391]  391  392  393  394  395  396  397  398  399  400  401  402  403
##  [404]  404  405  406  407  408  409  410  411  412  413  414  415  416
##  [417]  417  418  419  420  421  422  423  424  425  426  427  428  429
##  [430]  430  431  432  433  434  435  436  437  438  439  440  441  442
##  [443]  443  444  445  446  447  448  449  450  451  452  453  454  455
##  [456]  456  457  458  459  460  461  462  463  464  465  466  467  468
##  [469]  469  470  471  472  473  474  475  476  477  478  479  480  481
##  [482]  482  483  484  485  486  487  488  489  490  491  492  493  494
##  [495]  495  496  497  498  499  500  501  502  503  504  505  506  507
##  [508]  508  509  510  511  512  513  514  515  516  517  518  519  520
##  [521]  521  522  523  524  525  526  527  528  529  530  531  532  533
##  [534]  534  535  536  537  538  539  540  541  542  543  544  545  546
##  [547]  547  548  549  550  551  552  553  554  555  556  557  558  559
##  [560]  560  561  562  563  564  565  566  567  568  569  570  571  572
##  [573]  573  574  575  576  577  578  579  580  581  582  583  584  585
##  [586]  586  587  588  589  590  591  592  593  594  595  596  597  598
##  [599]  599  600  601  602  603  604  605  606  607  608  609  610  611
##  [612]  612  613  614  615  616  617  618  619  620  621  622  623  624
##  [625]  625  626  627  628  629  630  631  632  633  634  635  636  637
##  [638]  638  639  640  641  642  643  644  645  646  647  648  649  650
##  [651]  651  652  653  654  655  656  657  658  659  660  661  662  663
##  [664]  664  665  666  667  668  669  670  671  672  673  674  675  676
##  [677]  677  678  679  680  681  682  683  684  685  686  687  688  689
##  [690]  690  691  692  693  694  695  696  697  698  699  700  701  702
##  [703]  703  704  705  706  707  708  709  710  711  712  713  714  715
##  [716]  716  717  718  719  720  721  722  723  724  725  726  727  728
##  [729]  729  730  731  732  733  734  735  736  737  738  739  740  741
##  [742]  742  743  744  745  746  747  748  749  750  751  752  753  754
##  [755]  755  756  757  758  759  760  761  762  763  764  765  766  767
##  [768]  768  769  770  771  772  773  774  775  776  777  778  779  780
##  [781]  781  782  783  784  785  786  787  788  789  790  791  792  793
##  [794]  794  795  796  797  798  799  800  801  802  803  804  805  806
##  [807]  807  808  809  810  811  812  813  814  815  816  817  818  819
##  [820]  820  821  822  823  824  825  826  827  828  829  830  831  832
##  [833]  833  834  835  836  837  838  839  840  841  842  843  844  845
##  [846]  846  847  848  849  850  851  852  853  854  855  856  857  858
##  [859]  859  860  861  862  863  864  865  866  867  868  869  870  871
##  [872]  872  873  874  875  876  877  878  879  880  881  882  883  884
##  [885]  885  886  887  888  889  890  891  892  893  894  895  896  897
##  [898]  898  899  900  901  902  903  904  905  906  907  908  909  910
##  [911]  911  912  913  914  915  916  917  918  919  920  921  922  923
##  [924]  924  925  926  927  928  929  930  931  932  933  934  935  936
##  [937]  937  938  939  940  941  942  943  944  945  946  947  948  949
##  [950]  950  951  952  953  954  955  956  957  958  959  960  961  962
##  [963]  963  964  965  966  967  968  969  970  971  972  973  974  975
##  [976]  976  977  978  979  980  981  982  983  984  985  986  987  988
##  [989]  989  990  991  992  993  994  995  996  997  998  999 1000 1001
## [1002] 1002 1003 1004 1005 1006 1007 1008 1009 1010 1011 1012 1013 1014
## [1015] 1015 1016 1017 1018 1019 1020 1021 1022 1023 1024 1025 1026 1027
## [1028] 1028 1029 1030 1031 1032 1033 1034 1035 1036 1037 1038 1039 1040
## [1041] 1041 1042 1043 1044 1045 1046 1047 1048 1049 1050 1051 1052 1053
## [1054] 1054 1055 1056 1057 1058 1059 1060 1061 1062 1063 1064 1065 1066
## [1067] 1067 1068 1069 1070 1071 1072 1073 1074 1075 1076 1077 1078 1079
## [1080] 1080 1081 1082 1083 1084 1085 1086 1087 1088 1089 1090 1091 1092
## [1093] 1093 1094 1095 1096 1097 1098 1099 1100 1101 1102 1103 1104 1105
## [1106] 1106 1107 1108 1109 1110 1111 1112 1113 1114 1115 1116 1117 1118
## [1119] 1119 1120 1121 1122 1123 1124 1125 1126 1127 1128 1129 1130 1131
## [1132] 1132 1133 1134 1135 1136 1137 1138 1139 1140 1141 1142 1143 1144
## [1145] 1145 1146 1147 1148 1149 1150 1151 1152 1153 1154 1155 1156 1157
## [1158] 1158 1159 1160 1161 1162 1163 1164 1165 1166 1167 1168 1169 1170
## [1171] 1171 1172 1173 1174 1175 1176 1177 1178 1179 1180 1181 1182 1183
## [1184] 1184 1185 1186 1187 1188 1189 1190 1191 1192 1193 1194 1195 1196
## [1197] 1197 1198 1199 1200 1201 1202 1203 1204 1205 1206 1207 1208 1209
## [1210] 1210 1211 1212 1213 1214 1215 1216 1217 1218 1219 1220 1221 1222
## [1223] 1223 1224 1225 1226 1227 1228 1229 1230 1231 1232 1233 1234 1235
## [1236] 1236 1237 1238 1239 1240 1241 1242 1243 1244 1245 1246 1247 1248
## [1249] 1249 1250 1251 1252 1253 1254 1255 1256 1257 1258 1259 1260 1261
## [1262] 1262 1263 1264 1265 1266 1267 1268 1269 1270 1271 1272 1273 1274
## [1275] 1275 1276 1277 1278 1279 1280 1281 1282 1283 1284 1285 1286 1287
## [1288] 1288 1289 1290 1291 1292 1293 1294 1295 1296 1297 1298 1299 1300
## [1301] 1301 1302 1303 1304 1305 1306 1307 1308 1309 1310 1311 1312 1313
## [1314] 1314 1315 1316 1317 1318 1319 1320 1321 1322 1323 1324 1325 1326
## [1327] 1327 1328 1329 1330 1331 1332 1333 1334 1335 1336 1337 1338 1339
## [1340] 1340 1341 1342 1343 1344 1345 1346 1347 1348 1349 1350 1351 1352
## [1353] 1353 1354 1355 1356 1357 1358 1359 1360 1361 1362 1363 1364 1365
## [1366] 1366 1367 1368 1369 1370 1371 1372 1373 1374 1375 1376 1377 1378
## [1379] 1379 1380 1381 1382 1383 1384 1385 1386 1387 1388 1389 1390 1391
## [1392] 1392 1393 1394 1395 1396 1397 1398 1399 1400 1401 1402 1403 1404
## [1405] 1405 1406 1407 1408 1409 1410 1411 1412 1413 1414 1415 1416 1417
## [1418] 1418 1419 1420 1421 1422 1423 1424 1425 1426 1427 1428 1429 1430
## [1431] 1431 1432 1433 1434 1435 1436 1437 1438 1439 1440 1441 1442 1443
## [1444] 1444 1445 1446 1447 1448 1449 1450 1451 1452 1453 1454 1455 1456
## [1457] 1457 1458 1459 1460 1461 1462 1463 1464 1465 1466 1467 1468 1469
## [1470] 1470 1471 1472 1473 1474 1475 1476 1477 1478 1479 1480 1481 1482
## [1483] 1483 1484 1485 1486 1487 1488 1489 1490 1491 1492 1493 1494 1495
## [1496] 1496 1497 1498 1499 1500 1501 1502 1503 1504 1505 1506 1507 1508
## [1509] 1509 1510 1511 1512 1513 1514 1515 1516 1517 1518 1519 1520 1521
## [1522] 1522 1523 1524 1525 1526 1527 1528 1529 1530 1531 1532 1533 1534
## [1535] 1535 1536 1537 1538 1539 1540 1541 1542 1543 1544 1545 1546 1547
## [1548] 1548 1549 1550 1551 1552 1553 1554 1555 1556 1557 1558 1559 1560
## [1561] 1561 1562 1563 1564 1565 1566 1567 1568 1569 1570 1571 1572 1573
## [1574] 1574 1575 1576 1577 1578 1579 1580 1581 1582 1583 1584 1585 1586
## [1587] 1587 1588 1589 1590 1591 1592 1593 1594 1595 1596 1597 1598 1599
## [1600] 1600 1601 1602 1603 1604 1605 1606 1607 1608 1609 1610 1611 1612
## [1613] 1613 1614 1615 1616 1617 1618 1619 1620 1621 1622 1623 1624 1625
## [1626] 1626 1627 1628 1629 1630 1631 1632 1633 1634 1635 1636 1637 1638
## [1639] 1639 1640 1641 1642 1643 1644 1645 1646 1647 1648 1649 1650 1651
## [1652] 1652 1653 1654 1655 1656 1657 1658 1659 1660 1661 1662 1663 1664
## [1665] 1665 1666 1667 1668 1669 1670 1671 1672 1673 1674 1675 1676 1677
## [1678] 1678 1679 1680 1681 1682 1683 1684 1685 1686 1687 1688 1689 1690
## [1691] 1691 1692 1693 1694 1695 1696 1697 1698 1699 1700 1701 1702 1703
## [1704] 1704
```

These become useful in conjuction with data table's extra arguments to `[`:


```r
# How many countries in each continent?
gap[year == 2007, list(countries=.N), by=continent]
```

```
##    continent countries
## 1:      Asia        33
## 2:    Europe        30
## 3:    Africa        52
## 4:  Americas        25
## 5:   Oceania         2
```

The `by` argument lets you calculate things within groups:


```r
# Mean life expectancy per continent per year:
gap[, list(avgLifeExp=mean(lifeExp)), by=list(continent, year)]
```

```
##     continent year avgLifeExp
##  1:      Asia 1952   46.31439
##  2:      Asia 1957   49.31854
##  3:      Asia 1962   51.56322
##  4:      Asia 1967   54.66364
##  5:      Asia 1972   57.31927
##  6:      Asia 1977   59.61056
##  7:      Asia 1982   62.61794
##  8:      Asia 1987   64.85118
##  9:      Asia 1992   66.53721
## 10:      Asia 1997   68.02052
## 11:      Asia 2002   69.23388
## 12:      Asia 2007   70.72848
## 13:    Europe 1952   64.40850
## 14:    Europe 1957   66.70307
## 15:    Europe 1962   68.53923
## 16:    Europe 1967   69.73760
## 17:    Europe 1972   70.77503
## 18:    Europe 1977   71.93777
## 19:    Europe 1982   72.80640
## 20:    Europe 1987   73.64217
## 21:    Europe 1992   74.44010
## 22:    Europe 1997   75.50517
## 23:    Europe 2002   76.70060
## 24:    Europe 2007   77.64860
## 25:    Africa 1952   39.13550
## 26:    Africa 1957   41.26635
## 27:    Africa 1962   43.31944
## 28:    Africa 1967   45.33454
## 29:    Africa 1972   47.45094
## 30:    Africa 1977   49.58042
## 31:    Africa 1982   51.59287
## 32:    Africa 1987   53.34479
## 33:    Africa 1992   53.62958
## 34:    Africa 1997   53.59827
## 35:    Africa 2002   53.32523
## 36:    Africa 2007   54.80604
## 37:  Americas 1952   53.27984
## 38:  Americas 1957   55.96028
## 39:  Americas 1962   58.39876
## 40:  Americas 1967   60.41092
## 41:  Americas 1972   62.39492
## 42:  Americas 1977   64.39156
## 43:  Americas 1982   66.22884
## 44:  Americas 1987   68.09072
## 45:  Americas 1992   69.56836
## 46:  Americas 1997   71.15048
## 47:  Americas 2002   72.42204
## 48:  Americas 2007   73.60812
## 49:   Oceania 1952   69.25500
## 50:   Oceania 1957   70.29500
## 51:   Oceania 1962   71.08500
## 52:   Oceania 1967   71.31000
## 53:   Oceania 1972   71.91000
## 54:   Oceania 1977   72.85500
## 55:   Oceania 1982   74.29000
## 56:   Oceania 1987   75.32000
## 57:   Oceania 1992   76.94500
## 58:   Oceania 1997   78.19000
## 59:   Oceania 2002   79.74000
## 60:   Oceania 2007   80.71950
##     continent year avgLifeExp
```

The `with` argument lets you pass in column names as a character vector:


```r
gap[,c("continent", "country", "year"), with=FALSE]
```

```
##       continent     country year
##    1:      Asia Afghanistan 1952
##    2:      Asia Afghanistan 1957
##    3:      Asia Afghanistan 1962
##    4:      Asia Afghanistan 1967
##    5:      Asia Afghanistan 1972
##   ---                           
## 1700:    Africa    Zimbabwe 1987
## 1701:    Africa    Zimbabwe 1992
## 1702:    Africa    Zimbabwe 1997
## 1703:    Africa    Zimbabwe 2002
## 1704:    Africa    Zimbabwe 2007
```

### Keys 

One of the advantages of data table is the ability to set each tables "keys":
the columns which will act as unique identifiers for each row, for example:


```r
setkey(gap, continent, country, year)
```

We can see the change using the `tables` function, which shows all data tables 
in the R session:


```r
tables()
```

```
##      NAME     NROW NCOL MB COLS                                        
## [1,] gap     1,704    6  1 country,year,pop,continent,lifeExp,gdpPercap
## [2,] gap_dt2 1,704    6  1 country,year,pop,continent,lifeExp,gdpPercap
##      KEY                   
## [1,] continent,country,year
## [2,]                       
## Total: 2MB
```

This is really useful when you have multiple tables: allowing you to efficiently
merge tables together, confidently and concisely:


```r
landSize <- data.table(
  country=c("Australia", "New Zealand"),
  size=c(7692024, 268021)
)
setkey(landSize, country)
setkey(gap, country)
# Join landSize to gap, keeping only 'keys' (rows) in gap that also exist in
# landSize
gap[landSize]
```

```
##         country year      pop continent lifeExp gdpPercap    size
##  1:   Australia 1952  8691212   Oceania  69.120  10039.60 7692024
##  2:   Australia 1957  9712569   Oceania  70.330  10949.65 7692024
##  3:   Australia 1962 10794968   Oceania  70.930  12217.23 7692024
##  4:   Australia 1967 11872264   Oceania  71.100  14526.12 7692024
##  5:   Australia 1972 13177000   Oceania  71.930  16788.63 7692024
##  6:   Australia 1977 14074100   Oceania  73.490  18334.20 7692024
##  7:   Australia 1982 15184200   Oceania  74.740  19477.01 7692024
##  8:   Australia 1987 16257249   Oceania  76.320  21888.89 7692024
##  9:   Australia 1992 17481977   Oceania  77.560  23424.77 7692024
## 10:   Australia 1997 18565243   Oceania  78.830  26997.94 7692024
## 11:   Australia 2002 19546792   Oceania  80.370  30687.75 7692024
## 12:   Australia 2007 20434176   Oceania  81.235  34435.37 7692024
## 13: New Zealand 1952  1994794   Oceania  69.390  10556.58  268021
## 14: New Zealand 1957  2229407   Oceania  70.260  12247.40  268021
## 15: New Zealand 1962  2488550   Oceania  71.240  13175.68  268021
## 16: New Zealand 1967  2728150   Oceania  71.520  14463.92  268021
## 17: New Zealand 1972  2929100   Oceania  71.890  16046.04  268021
## 18: New Zealand 1977  3164900   Oceania  72.220  16233.72  268021
## 19: New Zealand 1982  3210650   Oceania  73.840  17632.41  268021
## 20: New Zealand 1987  3317166   Oceania  74.320  19007.19  268021
## 21: New Zealand 1992  3437674   Oceania  76.330  18363.32  268021
## 22: New Zealand 1997  3676187   Oceania  77.550  21050.41  268021
## 23: New Zealand 2002  3908037   Oceania  79.110  23189.80  268021
## 24: New Zealand 2007  4115771   Oceania  80.204  25185.01  268021
##         country year      pop continent lifeExp gdpPercap    size
```

```r
# And vice-versa:
landSize[gap]
```

```
##           country size year      pop continent lifeExp gdpPercap
##    1: Afghanistan   NA 1952  8425333      Asia  28.801  779.4453
##    2: Afghanistan   NA 1957  9240934      Asia  30.332  820.8530
##    3: Afghanistan   NA 1962 10267083      Asia  31.997  853.1007
##    4: Afghanistan   NA 1967 11537966      Asia  34.020  836.1971
##    5: Afghanistan   NA 1972 13079460      Asia  36.088  739.9811
##   ---                                                           
## 1700:    Zimbabwe   NA 1987  9216418    Africa  62.351  706.1573
## 1701:    Zimbabwe   NA 1992 10704340    Africa  60.377  693.4208
## 1702:    Zimbabwe   NA 1997 11404948    Africa  46.809  792.4500
## 1703:    Zimbabwe   NA 2002 11926563    Africa  39.989  672.0386
## 1704:    Zimbabwe   NA 2007 12311143    Africa  43.487  469.7093
```

To learn more about data table, you can check out the package Vignette on CRAN:
<http://cran.r-project.org/web/packages/data.table/vignettes/datatable-intro.pdf>




