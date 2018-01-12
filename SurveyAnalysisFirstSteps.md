---
title: "First Steps with a Survey"
---
# Survey Analysis: First Things First

This is a list of things I do first when recieving survey data.  This is not the same as
machine-collected data (traffic counts, GPS points), I would treat those things differently.

## 1: Basic Data Checks - Classification Variables

The obvious first thing is to look at the data and see if things look as expected. Summarize
all classification variables (such as people per household, workers per household, vehicles
per household, income) looking for the spread of values and the extent of missing values.

This can be done pretty easily with dplyr and tidyr. For the sake of experimentation, I
prepared a dataset from the National Household Travel Survey - this is a subset including
households, people, and trips from Urban Area Size of 1 million or more with no subway
or rail.  The data is [here](/files/NHTS09_Urbsize4.rda).

```
library(dplyr)

load("data/NHTS09_Urbsize4.rda")

HH %>%
  group_by(HHFAMINC) %>%
  summarize(n = n(), weighted_hh = sum(WTHHFIN))
```

The result is interesting as an example:

```
# A tibble: 21 x 3
   HHFAMINC     n weighted_hh
      <int> <int>       <dbl>
 1       -9     7    2978.131
 2       -8   611  401003.458
 3       -7  1608 1106251.962
 4        1   410  434079.009
 5        2   787  865340.424
 6        3  1113 1152447.130
 7        4  1263 1185127.702
 8        5  1126 1095830.388
 9        6  1606 1475247.109
10        7   949  745865.898
# ... with 11 more rows
```

The codes are on page 5 of [the codebook](http://nhts.ornl.gov/2009/pub/Codebook.pdf).

This can also be visualized:

Method 1 (quick, built into R):

```
hist(HH$HHFAMINC)
```

Which shows this...

![Quick Histogram]({{ "img/quick_histogram.png" | absolute_url}})

A second method provides far more control over the plot itself, but requires the
ggplot2 library (hint: `install.packages(ggplot2)`).

```
library(ggplot2)

ggplot(HH, aes(x = HHFAMINC)) + geom_histogram(binwidth = 1) +
  ggtitle("Histogram of Income") +
  xlab("Household Income Category") +
  ylab("Number of HHs in the Survey")
```

In the above example, I optionally used the ggtitle, xlab, and ylab commands to
set the plot title, x label, and y label.

![Better Histogram]({{ "img/better_histogram.png" | absolute_url}})

This should obviously be repeated for other classification variables.

Looking at the income data, I would certainly look into methods to impute the missing
income data.

## 2: Basic Data Checks - Continuous Variables

For continuous variables (for example, trip distance and time), I tend to look at a
histogram first, sometimes adjusting the number of breaks or subsetting the data.

```
hist(Trip$TRPMILES, breaks = 1000)

hist(Trip[Trip$TRPMILES < 100, "TRPMILES"], breaks = 100)
```

![Histogram 1]({{ "img/hist_miles.png" | absolute_url}})

![Histogram 2]({{ "img/hist_subset.png" | absolute_url}})

I would also check relationships using both correlation and a chart.

```
# Note that I am subsetting the trips down to those that had a positive distance
# in the codebook, negative values are N/A, DK, RF, or a skip.

cor(Trip[Trip$TRPMILES > 0, c("TRPMILES", "TRVL_MIN")])
```

The resulting correlation shows a decent, but not perfect, linear relationship.
The cor method (in this case) is correlating TRPMILES to TRPMILES, TRPMILES to
TRVL_MIN, TRVL_MIN to TRPMILES, and TRVL_MIN to TRVL_MIN - this is why four values
are being returned.  There are nicer-looking ways to do this, but for the sake of
time it is not really necessary.  Additionally, this can support more than two
columns of data.

```
          TRPMILES  TRVL_MIN
TRPMILES 1.0000000 0.6026497
TRVL_MIN 0.6026497 1.0000000
```

The scatterplot:

```
library(ggplot2) #does not need to be called if in a session where it has been loaded

ggplot(Trip[Trip$TRPMILES > 0,], aes(x = TRPMILES, y = TRVL_MIN)) + geom_point() +
  geom_abline(slope = 1, intercept = 0, color = "red") +
  ggtitle("Trip distance to time scatterplot") +
  xlab("Distance (miles)") + ylab("Time (min)")
```

![Scatterplot]({{ "img/distance-time-scatter.png" | absolute_url}})

I added the geom_abline here to provide a reference.  There are a few outliers
in this data. If it were my data, I'd start looking into these to see if something
went awry (for example, in ODOT's/OKI's 2010 GPS HHTS, I found a trip that took
around 8 hours to go from a participant's garage door to their mailbox).

This can be taken one step further with ggplot by using Loess smoothing.  I took a
sample of the data, and this took a minute or two to run.

```
TripS = Trip[sample(nrow(Trip),20000),]

ggplot(TripS, aes(x = TRPMILES, y = TRVL_MIN)) + geom_point() +
  geom_abline(slope = 1, intercept = 0, color = "red") +
  geom_smooth(method = "loess", size = 1.5) +
  ggtitle("Trip distance to time scatterplot + smoothing") +
  xlab("Distance (miles)") + ylab("Time (min)")
```

![Scatterplot with Loess Smoothing]({{ "img/scatter-loess.png" | absolute_url}})

Note that I don't use this often, and there are several options that can be used
in the geom_smooth function (hint: `?geom_smooth`).

## 3: Basic Data Checks - Relationships

With household travel surveys, there are frequently multiple tables (like in the
NHTS), and they relate together. This relationship should be checked to ensure
that the dataset is complete. This can be quickly checked using dplyr:

```
library(dplyr)

HH %>%
  select(HOUSEID, HHSIZE) %>%
  left_join(Person %>%
              group_by(HOUSEID) %>%
              summarize(nPerson = n()),
            by = "HOUSEID") %>%
  filter(HHSIZE != nPerson)
```

The result of this is a very large list (and if one were to work closely with this
data, they should direct the output of this to a data frame where they could start
looking closer at the households with missing people).  If you want just a summary
of now many rows are missing:

```
nrow(HH %>%
  select(HOUSEID, HHSIZE) %>%
  left_join(Person %>%
              group_by(HOUSEID) %>%
              summarize(nPerson = n()),
            by = "HOUSEID") %>%
  filter(HHSIZE != nPerson))
```

The above code returns just the number of rows in the resulting dataset, which
happens to be 5,824 (there are 27,509 in the HH subset, so it's about 21% that
are missing people).

Interestingly, we can take this one step further and look at how many are flagged
as not having 100% of the HHs complete the interview:

```
HH %>%
  select(HOUSEID, FLAG100, HHSIZE) %>%
  left_join(Person %>%
              group_by(HOUSEID) %>%
              summarize(nPerson = n()),
            by = "HOUSEID") %>%
  filter(HHSIZE != nPerson) %>%
  group_by(FLAG100) %>%
  summarize(n = n())

# A tibble: 2 x 2
    FLAG100     n
      <int> <int>
  1       1  2067
  2       2  3757
```

According to the codebook, 1 = 100% of HH members completed the interview.

Looking a trips is only slightly more complex because you have to join based on
two fields:

```
Person %>%
select(HOUSEID, PERSONID) %>%
left_join(Trip %>%
            group_by(HOUSEID, PERSONID) %>%
            summarize(nTrips = n()),
          by = c("HOUSEID", "PERSONID"))

#         HOUSEID PERSONID nTrips
   1     20002413        1      2
   2     20002413        2      2
   3     20002413        3     NA
   4     20002570        1      9
   5     20002570        2     NA
   6     20002570        3      4
   7     20002570        4      8
<etc>
```

This is a nice trip list, but the immediate question I'd want to know is how many
households did not make ANY trips...

```
Person %>%
  select(HOUSEID, PERSONID) %>%
  left_join(Trip %>%
              group_by(HOUSEID, PERSONID) %>%
              summarize(nTrips = n()),
            by = c("HOUSEID", "PERSONID")) %>%
  mutate(nTrips = ifelse(is.na(nTrips), 0, nTrips)) %>%
  group_by(HOUSEID) %>%
  summarize(nHHTrips = sum(nTrips)) %>%
  filter(nHHTrips == 0)

# A tibble: 2,281 x 2
      HOUSEID nHHTrips
        <int>    <dbl>
   1 20024451        0
   2 20091686        0
   3 20100739        0
   4 20137959        0
   5 20141787        0
   6 20171355        0
   7 20229733        0
   8 20293471        0
   9 20328618        0
  10 20346948        0
  # ... with 2,271 more rows
```

Hmmm.... 2,281 HHs with no trips.  I think I would look into all those and see
if those are legit.  Mind you, this is 8.3% of the HH table, so if this was a
normal MPO household survey of 3,500 (for example), the number of households to
check into would likely be smaller (290 in this case).
