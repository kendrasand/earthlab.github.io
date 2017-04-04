---
layout: single
title: "An example of creating modular code in R - Efficient scientific programming"
excerpt: "This lesson provides an example of modularizing code in R. "
authors: ['Carson Farmer', 'Leah Wasser']
modified: '2017-04-04'
category: [course-materials]
class-lesson: ['intro-APIs-r']
permalink: /course-materials/earth-analytics/week-10/get-data-with-rcurl-r/
nav-title: 'Intro to RCurl'
week: 10
sidebar:
  nav:
author_profile: false
comments: true
order: 2
---

{% include toc title="In This Lesson" icon="file-text" %}

<div class='notice--success' markdown="1">

## <i class="fa fa-graduation-cap" aria-hidden="true"></i> Learning Objectives

After completing this tutorial, you will:

* Be able to access data from a remote URL (http or https) using the `getURL()` and `textConnection()` functions.
* Explain the difference between accessing data using `download.file()` compared to `getURL()`.

## <i class="fa fa-check-square-o fa-2" aria-hidden="true"></i> What you need

You will need a computer with internet access to complete this lesson and the
data that we already downloaded for week 6 of the course.

</div>





```r
library(dplyr)
library(ggplot2)
library(RCurl)
```

## Direct data access

In this lesson we will review how to access data via a direct download in `R`.
We downloaded data in the first week of this class using `download.file()`
When we used `download.file()`, we were literally downloading that file,
which happened to be in `.csv` (comma separated value) text format to our computer.

We specified the location where that file would download to, using the `destfile=`
argument. Notice below, I specified week 10 as the download location given
that is our current class week.


```r
# download text file to a specified location on our computer
download.file(url = "https://ndownloader.figshare.com/files/7010681",
              destfile = "data/week10/boulder-precip-aug-oct-2013.csv")
```


If `R` is able to communicate with the server (in this case Figshare) and download
the file, we can then open up the file and plot the data within it.


```r
# read data into R
boulder_precip <- read.csv("data/week10/boulder-precip-aug-oct-2013.csv")

# fix date
boulder_precip$DATE <- as.Date(boulder_precip$DATE)
# plot data with ggplot
ggplot(boulder_precip, aes(x = DATE, y=PRECIP)) +
  geom_point() +
      labs(x="Date (2013)",
           y="Precipitation (inches)",
          title="Precipitation - Boulder, CO ",
          subtitle = "August - October 2013")
```

<img src="{{ site.url }}/images/rfigs/course-materials/earth-analytics/week-10/in-class/2016-12-06-api02-get-started-r/unnamed-chunk-4-1.png" title=" " alt=" " width="100%" />


## Download data via human readable url

The file that we downloaded above is stored using a `.csv` or comma separated
value format. This
is a format that is human readable and structured using a simple, non hierarchical
(no nesting involved) format compared to **JSON** which can be hierarchical and thus
efficiently support more complex data. The `download.file()` function allows us
to store a copy of the file on our computer. Given the data are small and they
could be moved over time, this is a good idea as now we have a backup of the data.

<i class="fa fa-lightbulb-o" aria-hidden="true"></i> **Data Tip:** If we have a secure url (secure transfer protocols - i.e., `https`) we may not be
able to use `read.csv()`. Instead, we need to use functions in the `RCurl` package.
With that said `read.csv()` may work for some if not all computers now given
upgrades to the base R code.
{: .notice--success}

## Directly access & import data into R

We can import data directly into `R` rather than downloading it using the
`read.csv()` and/or `read.table()` functions. This solution will may have some
problems when the data are stored on a secure server. However, let's have a look
at how we use `read.csv()` to directly import data stored on a website or server,
into `R`. The `read.csv()` function is ideal for
data that are separated by commas (.csv) files whereas `read.table()` is ideal
for data in other formats - separated by spaces, tabs and other delimiters.



```r
boulder_precip2 <- read.csv("https://ndownloader.figshare.com/files/7010681")
# fix date
boulder_precip2$DATE <- as.Date(boulder_precip2$DATE)
# plot data with ggplot
ggplot(boulder_precip2, aes(x = DATE, y=PRECIP)) +
  geom_point() +
      labs(x="Date (2013)",
           y="Precipitation (inches)",
          title="Precipitation Data Imported with read.csv()",
          subtitle = "August - October 2013")
```

<img src="{{ site.url }}/images/rfigs/course-materials/earth-analytics/week-10/in-class/2016-12-06-api02-get-started-r/import-plot-data-1.png" title=" " alt=" " width="100%" />

### read.csv() vs RCURL

While using `read.csv()` to get data directly works, it may fail sometimes if:

1. You are trying to access data from a secure (https or ftps server) or
2. You are trying to access data from an API that requires authentication (more on that later)

Because of these potential limitations, we are going to use functions in the
RCurl package for the rest of this module.

## Use RCurl to download data

RCurl is a powerful package that:

* Provides a set of tools to allow `R` to act like a *web client*
* Provides a number of helper functions to grab data files from the web

The `getURL()` function works for most secure web download protocols (e.g.,
`http(s)`, `ftp(S)`). It also helps with web scraping, direct access to web
resources, and even API data access.

## Download data with RCurl

Next, we will use the `RCurl::getURL()` function to download data from a Princeton University data website.

<i class="fa fa-lightbulb-o" aria-hidden="true"></i> **Data Tip:** The syntax
`package::functionName()` is a common way to tell R to use a function from a particular
package.  In the example above: we specify that we are using getURL() from the
RCurl package using the syntax: `RCurl::getURL()`. This syntax is not necessary to call
getURL UNLESS there is another `getURL()` function available in your `R` session.
{: .notice--success}

## Access birthrate data using getURL

Birth rate data for several countries are available via a
<a href="http://data.princeton.edu/wws509/datasets" target="_blank">Princeton University data website</a>. The birth rate data show how much effort went into considering family planning
efforts that were in place to attempt to reduce birth rates in various countries.
The outcome variable is the associated percent decline in birth rate by country
over 10 years. An excerpt from the website where we are getting the data is below.

>Here are the famous program effort data from Mauldin and Berelson. These data
consist of observations on an index of social setting, an index of family
planning effort, and the percent decline in the crude birth rate (CBR) between
1965 and 1975, for 20 countries in Latin America.

The data have 3 variables:

1. Birth rate
1. Index of social setting
1. Index of family planning effort

We can read these data in `R` using the `read.table()` function.

<i class="fa fa-lightbulb-o" aria-hidden="true"></i> **Data Tip:** Note that we
are using `read.table()` rather than `read.csv()` because in this instance,
the data are not stored in a `.csv` (comma separated value) format. Rather, they
are stored in a `.dat` format.
{: .notice--success }


```r
the_url <- "http://data.princeton.edu/wws509/datasets/effort.dat"
the_data <- read.table(the_url)
head(the_data)
##           setting effort change
## Bolivia        46      0      1
## Brazil         74      0     10
## Chile          89     16     29
## Colombia       77     16     25
## CostaRica      84     21     29
## Cuba           89     15     40
```

While read.table does work to directly open the data, it is more robust to use
`getURL()`. `getURL()`, a function that comes from the RCurl `R` package, allows you to
consistently access secure servers and also has additional authentication support.

To use getURL to open text files we do the following:

1. We *grab* the URL using getURL()
2. We read in the data using `read.csv()` (or `read.table()` ) via the `textConnection()` function.

Note that the `textConnection()` function tells `R` that the data that we are
accessing should be read as a text file.


```r
the_url <- "http://data.princeton.edu/wws509/datasets/effort.dat"
the_data <- getURL(the_url)
# read in the data
birth_rates <- read.table(textConnection(the_data))
```

## Working with Web Data

The `birth_rates` data that we just accessed, import into a
`data.frame` format that we are used to working with. We can analyze and visualize
the data using `ggplot()` just like we did with the precipitation data earlier.
For example:

Here's the top 6 rows (or `head()`) of the `data.frame`:


```r
str(birth_rates)
## 'data.frame':	20 obs. of  3 variables:
##  $ setting: int  46 74 89 77 84 89 68 70 60 55 ...
##  $ effort : int  0 0 16 16 21 15 14 6 13 9 ...
##  $ change : int  1 10 29 25 29 40 21 0 13 4 ...
head(birth_rates)
##           setting effort change
## Bolivia        46      0      1
## Brazil         74      0     10
## Chile          89     16     29
## Colombia       77     16     25
## CostaRica      84     21     29
## Cuba           89     15     40
```

We can plot these data to see the relationships between effort and percent change in
birth rates.


```r
ggplot(birth_rates, aes(x=effort, y=change)) +
  geom_point() +
      labs(x="Effort",
           y="Percent Change",
          title="Decline in birth rate vs. planning effort",
          subtitle = "For 20 Latin America Countries")
```

<img src="{{ site.url }}/images/rfigs/course-materials/earth-analytics/week-10/in-class/2016-12-06-api02-get-started-r/unnamed-chunk-6-1.png" title=" " alt=" " width="100%" />

Remember that here we've imported a tabular dataset directly from the Princeton
University website. The data file itself is NOT on our computer so we do not
have a backup in the event that the data are removed from the Princeton website -
out code would not run.

<i class="fa fa-lightbulb-o" aria-hidden="true"></i> **Data Tip:** Consider when
you directly access a dataset via an API that - that data may not always
be available. It is often a good idea to save backup copies of certain datasets on
your computer if the data are not too large. For example, what happens if the
data API or server goes down, is taken away, etc? Many data repositories have
documented terms of data longevity - or explicit provisions that specify how long
the data will be available on the repository and available for (public) use. Look
into this before assuming the
data will always be there!
{: .notice--success }

<div class="notice--warning" markdown="1">

## <i class="fa fa-pencil-square-o" aria-hidden="true"></i> Optional challenge

Using the tools that we learned above, import the Princeton salary data below
using the Rcurl functions `getURL()` & `textConnection()`, combined with `read.table()`.

<a href="http://data.princeton.edu/wws509/datasets/#salary" target="_blank">Learn more about the Princeton salary data</a>

As described on the website:

> These are the salary data used in Weisberg's book, consisting of observations on six variables for 52 tenure-track professors in a small college. The variables are:

* **sx** Sex, coded 1 for female and 0 for male
* **rk** Rank, coded
  * **1** for assistant professor,
  * **2** for associate professor, and
  * **3** for full professor
* **yr** Number of years in current rank
* **dg** Highest degree, coded 1 if doctorate, 0 if masters
* **yd** Number of years since highest degree was earned
* **sl** Academic year salary, in dollars.

HINT: these data have a header. You will have to look up the appropriate argument
to ensure that the data import properly using `read.table()`.

HINT2: You can add facets or individual plots for particular subsets of data (
in this case rank) using the `facet_wrap()` argument in a `ggplot()` plot. For example
 `+ facet_wrap(~dg)` will create a `f` plot with sub plots filtered by highest
 degree.)

Plot the following:

Experience (x axis) vs. salary (y axis). Color your points by SEX and use facets
to add a facet for each of the three ranks.

</div>


<img src="{{ site.url }}/images/rfigs/course-materials/earth-analytics/week-10/in-class/2016-12-06-api02-get-started-r/all-data-1.png" title=" " alt=" " width="100%" />

## Example homework
Data faceted by rank. You can add the argument `+ facet_wrap(~variableHere)` to
create a faceted plot like the one below.

<img src="{{ site.url }}/images/rfigs/course-materials/earth-analytics/week-10/in-class/2016-12-06-api02-get-started-r/facet-by-rank-1.png" title=" " alt=" " width="100%" />

You can also ad a linear model regression to the data if you want using
`geom_smooth()`.

<img src="{{ site.url }}/images/rfigs/course-materials/earth-analytics/week-10/in-class/2016-12-06-api02-get-started-r/all-data-lm-1.png" title=" " alt=" " width="100%" />