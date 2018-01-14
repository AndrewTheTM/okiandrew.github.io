---
title: "Basic Statistical Checks"
---

# Basic Statistical Checks on Data

This assumes you have the NHTS dataset loaded.

There is some (a lot, really) of overlap between this and the Survey Analysis steps. This is
really just more quick and to-the-point, whereas the Survey Analysis Steps pages are more
about actually looking for problems and issues, particularly where relational databases
(like household surveys) are concerned.

## Mean, Median, Standard Deviation, Minimum, Maximum

There are two ways to do this. One works for data with few columns but will show all:

```
summary(HH)

Results:

HOUSEID            VARSTRAT         WTHHFIN            DRVRCNT         CDIVMSAR        CENSUS_D    
Min.   :20002413   Min.   :  1.00   Min.   :    8.79   Min.   :0.000   Min.   :11.00   Min.   :1.000  
1st Qu.:32254747   1st Qu.: 24.00   1st Qu.:  107.63   1st Qu.:1.000   1st Qu.:52.00   1st Qu.:5.000  
Median :44708681   Median : 48.00   Median :  232.12   Median :2.000   Median :72.00   Median :7.000  
Mean   :44814213   Mean   : 48.71   Mean   :  768.46   Mean   :1.796   Mean   :72.33   Mean   :7.038  
3rd Qu.:57338323   3rd Qu.: 72.00   3rd Qu.:  543.68   3rd Qu.:2.000   3rd Qu.:91.00   3rd Qu.:9.000  
Max.   :69998300   Max.   :100.00   Max.   :51021.05   Max.   :8.000   Max.   :92.00   Max.   :9.000  

CENSUS_R        HH_HISP          HH_RACE          HHFAMINC        HHRELATD           HHRESP     
Min.   :1.000   Min.   :-9.000   Min.   :-9.000   Min.   :-9.00   Min.   :-9.0000   Min.   :1.000  
1st Qu.:3.000   1st Qu.: 2.000   1st Qu.: 1.000   1st Qu.: 5.00   1st Qu.:-1.0000   1st Qu.:1.000  
Median :3.000   Median : 2.000   Median : 1.000   Median :11.00   Median :-1.0000   Median :1.000  
Mean   :3.345   Mean   : 1.861   Mean   : 2.558   Mean   :10.15   Mean   :-0.9999   Mean   :1.003  
3rd Qu.:4.000   3rd Qu.: 2.000   3rd Qu.: 1.000   3rd Qu.:17.00   3rd Qu.:-1.0000   3rd Qu.:1.000  
Max.   :4.000   Max.   : 2.000   Max.   :97.000   Max.   :18.00   Max.   : 1.0000   Max.   :8.000

<<SNIP>>  
```

To get the data for specific columns:

```
> mean(HH$HHSIZE)
[1] 2.389545
> median(HH$HHSIZE)
[1] 2
> sd(HH$HHSIZE)
[1] 1.278871
> min(HH$HHSIZE)
[1] 1
> max(HH$HHSIZE)
[1] 12
```

## Correlation of two or more columns:
```
> cor(HH[,c("HHSIZE", "WRKCOUNT", "HHVEHCNT")])
            HHSIZE  WRKCOUNT  HHVEHCNT
HHSIZE   1.0000000 0.4709591 0.4289851
WRKCOUNT 0.4709591 1.0000000 0.4614459
HHVEHCNT 0.4289851 0.4614459 1.0000000
```

The output of the above is a matrix of Pearson correlations among all the columns.

## Histograms

The historgram is one of the better things to take a look at. This is simply done with the command:

```
hist(HH$HHSIZE)
```

![Quick Histogram]({{ "img/hist_hhsize.png" | absolute_url}})
