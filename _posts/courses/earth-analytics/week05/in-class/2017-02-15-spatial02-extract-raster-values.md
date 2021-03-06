---
layout: single
title: "Extract raster values using vector boundaries in R"
excerpt: "This lesson reviews how to extract pixels from a raster dataset using a
vector boundary. We can use the extracted pixels to calculate mean and max tree height for a study area (in this case a field site where we measured tree heights on the ground. Finally we will compare tree heights derived from lidar data compared to tree height measured by humans on the ground. "
authors: ['Leah Wasser']
modified: '2017-09-18'
category: [courses]
class-lesson: ['remote-sensing-uncertainty-r']
permalink: /courses/earth-analytics/week-5/extract-data-from-raster/
nav-title: 'Extract data from raster'
week: 5
course: "earth-analytics"
sidebar:
  nav:
author_profile: false
comments: true
order: 2
topics:
  remote-sensing: ['lidar']
  earth-science: ['vegetation', 'uncertainty']
  reproducible-science-and-programming:
  spatial-data-and-gis: ['vector-data', 'raster-data']
---

{% include toc title="In this lesson" icon="file-text" %}

<div class='notice--success' markdown="1">

## <i class="fa fa-graduation-cap" aria-hidden="true"></i> Learning objectives

After completing this tutorial, you will be able to:

* Use the `extract()` function to extract raster values using a vector extent or set of extents
* Create a scatter plot with a one-to-one line in `R`
* Understand the concept of uncertainty as it's associated with remote sensing data

## <i class="fa fa-check-square-o fa-2" aria-hidden="true"></i> What you need

You will need a computer with internet access to complete this lesson and the data for week 5 of the course.

[<i class="fa fa-download" aria-hidden="true"></i> Download Week 5 Data (~500 MB)](https://ndownloader.figshare.com/files/7525363){:data-proofer-ignore='' .btn }


</div>



```r
# load libraries
library(raster)
library(rgdal)
library(rgeos)
library(ggplot2)
library(dplyr)

options(stringsAsFactors = FALSE)

# set working directory
# setwd("path-here/earth-analytics")
```

## Import canopy height model

First, we will import a canopy height model created by the NEON project. In the
previous lessons / weeks we learned how to make a canopy height model by
subtracting the digital elevation model (`DEM`) from the digital surface model (`DSM`).


```r
# import canopy height model (CHM).
SJER_chm <- raster("data/week_05/california/SJER/2013/lidar/SJER_lidarCHM.tif")
## Error in .rasterObjectFromFile(x, band = band, objecttype = "RasterLayer", : Cannot create a RasterLayer object from this file. (file does not exist)
SJER_chm
## Error in eval(expr, envir, enclos): object 'SJER_chm' not found

# plot the data
hist(SJER_chm,
     main = "Histogram of Canopy Height\n NEON SJER Field Site",
     col = "springgreen",
     xlab = "Height (m)")
## Error in hist(SJER_chm, main = "Histogram of Canopy Height\n NEON SJER Field Site", : object 'SJER_chm' not found
```


There are a lot of values in our `CHM` that == 0. Let's set those to `NA` and plot
again.


```r

# set values of 0 to NA as these are not trees
SJER_chm[SJER_chm==0] <- NA
## Error in SJER_chm[SJER_chm == 0] <- NA: object 'SJER_chm' not found

# plot the modified data
hist(SJER_chm,
     main = "Histogram of Canopy Height\n pixels==0 set to NA",
     col = "springgreen",
     xlab = "Height (m)")
## Error in hist(SJER_chm, main = "Histogram of Canopy Height\n pixels==0 set to NA", : object 'SJER_chm' not found
```

## Part 2. Does our CHM data compare to field measured tree heights?

We now have a canopy height model for our study area in California. However, how
do the height values extracted from the `CHM` compare to our laboriously collected,
field measured canopy height data? To figure this out, we will use *in situ* collected
tree height data, measured within circular plots across our study area. We will compare
the maximum measured tree height value to the maximum lidar derived height value
for each circular plot using regression.

For this activity, we will use the a `csv` (comma separate value) file,
located in `SJER/2013/insitu/veg_structure/D17_2013_SJER_vegStr.csv`.


```r
# import plot centroids
SJER_plots <- readOGR("data/week_05/california/SJER/vector_data",
                      "SJER_plot_centroids")
## Error in ogrInfo(dsn = dsn, layer = layer, encoding = encoding, use_iconv = use_iconv, : Cannot open data source

# Overlay the centroid points and the stem locations on the CHM plot
plot(SJER_chm,
     main = "SJER  Plot Locations",
     col=gray.colors(100, start=.3, end=.9))
## Error in plot(SJER_chm, main = "SJER  Plot Locations", col = gray.colors(100, : object 'SJER_chm' not found

# pch 0 = square
plot(SJER_plots,
     pch = 16,
     cex = 2,
     col = 2,
     add=TRUE)
## Error in plot(SJER_plots, pch = 16, cex = 2, col = 2, add = TRUE): object 'SJER_plots' not found
```

### Extract CMH data within 20 m radius of each plot centroid

Next, we will create a boundary region (called a buffer) representing the spatial
extent of each plot (where trees were measured). We will then extract all `CHM` pixels
that fall within the plot boundary to use to estimate tree height for that plot.

There are a few ways to go about this task. If our plots are circular, then we can
use the `extract()` function.

<figure>
    <img src="{{ site.url }}/images/courses/earth-analytics/week-5/buffer-circular.png" alt="buffer circular">
    <figcaption>The extract function in R allows you to specify a circular buffer
    radius around an x,y point location. Values for all pixels in the specified
    raster that fall within the circular buffer are extracted. In this case, we
    will tell R to extract the maximum value of all pixels using the fun=max
    command. Source: Colin Williams, NEON
    </figcaption>
</figure>

### Extract plot data using circle: 20m radius plots


```r
# Insitu sampling took place within 40m x 40m square plots, so we use a 20m radius.
# Note that below will return a data.frame containing the max height
# calculated from all pixels in the buffer for each plot
SJER_height <- extract(SJER_chm,
                    SJER_plots,
                    buffer = 20, # specify a 20 m radius
                    fun=mean, # extract the MEAN value from each plot
                    sp=TRUE, # create spatial object
                    stringsAsFactors=FALSE)
## Error in extract(SJER_chm, SJER_plots, buffer = 20, fun = mean, sp = TRUE, : object 'SJER_chm' not found
```

#### Explore the data distribution

If you want to explore the data distribution of pixel height values in each plot,
you could remove the `fun` call to max and generate a list.
`cent_ovrList <- extract(chm,centroid_sp,buffer = 20)`. It's good to look at the
distribution of values we've extracted for each plot. Then you could generate a
histogram for each plot `hist(cent_ovrList[[2]])`. If we wanted, we could loop
through several plots and create histograms using a `for loop`.


```r
# cent_ovrList <- extract(chm,centroid_sp,buffer = 20)
# create histograms for the first 5 plots of data
# for (i in 1:5) {
#  hist(cent_ovrList[[i]], main=(paste("plot",i)))
#  }

```


### Derive square plot boundaries, then CHM values around a point
For how to extract square plots using a plot centroid value, check out the
<a href="http://neondataskills.org/working-with-field-data/Field-Data-Polygons-From-Centroids" target="_blank"> extracting square shapes activity </a>.

 <figure>
    <img src="{{ site.url }}/images/courses/earth-analytics/week-5/buffer-square.png" alt="Image showing the buffer area for a plot.">
    <figcaption>If you had square shaped plots, the code in the link above would
    extract pixel values within a square shaped buffer. Source: Colin Williams, NEON
    </figcaption>
</figure>



## Extract descriptive stats from *In situ* Data
In our final step, we will extract summary height values from our field data.
We will use the `dplyr` library to do this efficiently.

First let's see how many plots are in our tree height data. Note that our tree
height data is stored in `csv` format.


```r
# import the centroid data and the vegetation structure data
SJER_insitu <- read.csv("data/week_05/california/SJER/2013/insitu/veg_structure/D17_2013_SJER_vegStr.csv",
                        stringsAsFactors = FALSE)
## Warning in file(file, "rt"): cannot open file 'data/week_05/california/
## SJER/2013/insitu/veg_structure/D17_2013_SJER_vegStr.csv': No such file or
## directory
## Error in file(file, "rt"): cannot open the connection

# get list of unique plots
unique(SJER_plots$Plot_ID)
## Error in unique(SJER_plots$Plot_ID): object 'SJER_plots' not found
```

## Extract max tree height

Next, we can use dplyr to extract a summary tree height value for each plot. In
this case, we will calculate the mean MEASURED tree height value for each
plot. This value represents the average tree in each plot. We will also calculate
the max height representing the max height for each plot.

FInally, we will compare the mean measured tree height per plot to the mean
tree height extracted from the lidar `CHM`.


```r
# find the max and mean stem height for each plot
insitu_stem_height <- SJER_insitu %>%
  group_by(plotid) %>%
  summarise(insitu_max = max(stemheight), insitu_avg = mean(stemheight))
## Error in eval(lhs, parent, parent): object 'SJER_insitu' not found

# view the data frame to make sure we're happy with the column names.
head(insitu_stem_height)
## Error in head(insitu_stem_height): object 'insitu_stem_height' not found
```


### Merge insitu data With spatial data.frame

Once we have our summarized insitu data, we 'an `merge` it into the centroids
`data.frame`. Merge requires two `data.frame`s and the names of the columns
containing the unique ID that we will merge the data on. In this case, we will
merge the data on the plot_id column. Notice that it's spelled slightly differently
in both `data.frame`s so we'll need to tell `R` what it's called in each `data.frame`.


```r
# merge the insitu data into the centroids data.frame
SJER_height <- merge(SJER_height,
                     insitu_stem_height,
                   by.x = 'Plot_ID',
                   by.y = 'plotid')
## Error in merge(SJER_height, insitu_stem_height, by.x = "Plot_ID", by.y = "plotid"): object 'SJER_height' not found

SJER_height@data
## Error in eval(expr, envir, enclos): object 'SJER_height' not found
```

## Plot by height


```r
# plot canopy height model
plot(SJER_chm,
     main = "Vegetation Plots \nSymbol size by Average Tree Height",
     legend=F)
## Error in plot(SJER_chm, main = "Vegetation Plots \nSymbol size by Average Tree Height", : object 'SJER_chm' not found

# add plot location sized by tree height
plot(SJER_height,
     pch=19,
     cex=(SJER_height$SJER_lidarCHM)/10, # size symbols according to tree height attribute normalized by 10
     add=T)
## Error in plot(SJER_height, pch = 19, cex = (SJER_height$SJER_lidarCHM)/10, : object 'SJER_height' not found

# place legend outside of the plot
par(xpd=T)
legend(SJER_chm@extent@xmax+250,
       SJER_chm@extent@ymax,
       legend="plot location \nsized by \ntree height",
       pch=19,
       bty='n')
## Error in legend(SJER_chm@extent@xmax + 250, SJER_chm@extent@ymax, legend = "plot location \nsized by \ntree height", : object 'SJER_chm' not found
```


### Plot data (CHM vs measured)
Let's create a plot that illustrates the relationship between insitu measured
max canopy height values and lidar derived max canopy height values.



```r
# create plot
ggplot(SJER_height@data, aes(x=SJER_lidarCHM, y = insitu_avg)) +
  geom_point() +
  theme_bw() +
  ylab("Mean measured height") +
  xlab("Mean LiDAR pixel") +
  ggtitle("Lidar Derived Mean Tree Height \nvs. InSitu Measured Mean Tree Height (m)")
## Error in ggplot(SJER_height@data, aes(x = SJER_lidarCHM, y = insitu_avg)): object 'SJER_height' not found
```

Next, let's fix the plot adding a 1:1 line and making the x and y axis the same .


```r
# create plot
ggplot(SJER_height@data, aes(x=SJER_lidarCHM, y = insitu_avg)) +
  geom_point() +
  theme_bw() +
  ylab("Mean measured height") +
  xlab("Mean LiDAR pixel") +
  xlim(0,15) + ylim(0,15) + # set x and y limits to 0-20
  geom_abline(intercept = 0, slope=1) + # add one to one line
  ggtitle("Lidar Derived Tree Height \nvs. InSitu Measured Tree Height")
## Error in ggplot(SJER_height@data, aes(x = SJER_lidarCHM, y = insitu_avg)): object 'SJER_height' not found
```

We can also add a regression fit to our plot. Explore the `GGPLOT` options and
customize your plot.


```r
# plot with regression fit
p <- ggplot(SJER_height@data, aes(x=SJER_lidarCHM, y = insitu_avg)) +
  geom_point() +
  ylab("Maximum Measured Height") +
  xlab("Maximum LiDAR Height")+
    xlim(0,15) + ylim(0,15) + # set x and y limits to 0-20
  geom_abline(intercept = 0, slope=1)+
  geom_smooth(method=lm)
## Error in ggplot(SJER_height@data, aes(x = SJER_lidarCHM, y = insitu_avg)): object 'SJER_height' not found

p + theme(panel.background = element_rect(colour = "grey")) +
  ggtitle("LiDAR CHM Derived vs Measured Tree Height") +
  theme(plot.title=element_text(family="sans", face="bold", size=20, vjust=1.9)) +
  theme(axis.title.y = element_text(family="sans", face="bold", size=14, angle=90, hjust=0.54, vjust=1)) +
  theme(axis.title.x = element_text(family="sans", face="bold", size=14, angle=00, hjust=0.54, vjust=-.2))
## Error in p + theme(panel.background = element_rect(colour = "grey")) + : non-numeric argument to binary operator
```


## View difference: lidar vs measured


```r
# Calculate difference
SJER_height@data$ht_diff <-  (SJER_height@data$SJER_lidarCHM - SJER_height@data$insitu_avg)
## Error in eval(expr, envir, enclos): object 'SJER_height' not found



# create bar plot using ggplot()
ggplot(data=SJER_height@data,
       aes(x=Plot_ID, y=ht_diff, fill=Plot_ID)) +
       geom_bar(stat="identity") +
       xlab("Plot Name") + ylab("Height difference (m)")
## Error in ggplot(data = SJER_height@data, aes(x = Plot_ID, y = ht_diff, : object 'SJER_height' not found
       ggtitle("Difference: \nLidar avg height - in situ avg height (m)")
## $title
## [1] "Difference: \nLidar avg height - in situ avg height (m)"
## 
## $subtitle
## NULL
## 
## attr(,"class")
## [1] "labels"
```

You have now successfully created a canopy height model using lidar data AND compared lidar
derived vegetation height, within plots, to actual measured tree height data.
Does the relationship look good or not? Would you use lidar data to estimate tree
height over larger areas? Would other metrics be a better comparison (see challenge
below).

<div class="notice--warning" markdown="1">

## <i class="fa fa-pencil-square-o" aria-hidden="true"></i> Test your skills: lidar vs insitu comparison

Create a plot of lidar max height vs *insitu* max height. Add labels to your plot.
Customize the colors, fonts and the look of your plot. If you are happy with the
outcome, share your plot in the comments below!
 </div>


## Interactive plot

<iframe width="460" height="293" frameborder="0" seamless="seamless" scrolling="no" src="https://plot.ly/~leahawasser/24.embed?width=460&height=293"></iframe>
