---
layout: single
title: "Landsat tif files in R"
excerpt: ". "
authors: ['Leah Wasser']
modified: '2017-02-20'
category: [course-materials]
class-lesson: ['spectral-data-fire-r']
permalink: /course-materials/earth-analytics/week-6/landsat-bands-geotif-in-R/
nav-title: 'Landsat tifs in R'
week: 6
sidebar:
  nav:
author_profile: false
comments: true
order: 3
---

{% include toc title="In This Lesson" icon="file-text" %}

<div class='notice--success' markdown="1">

## <i class="fa fa-graduation-cap" aria-hidden="true"></i> Learning Objectives

After completing this tutorial, you will be able to:

*

## <i class="fa fa-check-square-o fa-2" aria-hidden="true"></i> What you need

You will need a computer with internet access to complete this lesson and the data for week 5 of the course.


</div>

In the previous lesson, we learned how to import a multi-band image into R using
the stack() function. We then plotted the data as a composite, RGB (and CIR) image
using plotRGB(). However, sometimes data are downloaded in individual bands rather
than a composite raster stack.

In this lesson we will learn how to work with Lansat data in R. In this case, our
data are downloaded in .tif format with each file representing a single band rather
than a stack of bands.

## About Landsat data

Stuff here including the list of bands, etc etc...

something about how the string of numbers that make up the director and file
name tell us about the file name....


```r
# load spatial packages
library(raster)
library(rgdal)
library(rgeos)
# turn off factors
options(stringsAsFactors = F)
```


If we look at the directory that contains our landsat data, we will see that
each of the individual bands is stored individually as a geotiff rather than
being stored as a stacked or layered raster.

Why...

more here about why this is the case...



```r
# get list of all tifs
list.files("data/week6/landsat/LC80340322016205-SC20170127160728/crop")
##  [1] "LC80340322016205LGN00_bqa_crop.tif"        
##  [2] "LC80340322016205LGN00_cfmask_conf_crop.tif"
##  [3] "LC80340322016205LGN00_cfmask_crop.tif"     
##  [4] "LC80340322016205LGN00_sr_band1_crop.tif"   
##  [5] "LC80340322016205LGN00_sr_band2_crop.tif"   
##  [6] "LC80340322016205LGN00_sr_band3_crop.tif"   
##  [7] "LC80340322016205LGN00_sr_band4_crop.tif"   
##  [8] "LC80340322016205LGN00_sr_band5_crop.tif"   
##  [9] "LC80340322016205LGN00_sr_band6_crop.tif"   
## [10] "LC80340322016205LGN00_sr_band7_crop.tif"   
## [11] "LC80340322016205LGN00_sr_cloud_crop.tif"   
## [12] "LC80340322016205LGN00_sr_ipflag_crop.tif"

# but really we just want the tif files
all_landsat_bands <- list.files("data/week6/Landsat/LC80340322016205-SC20170127160728/crop",
                      pattern=".tif$",
                      full.names = T) # make sure we have the full path to the file
all_landsat_bands
##  [1] "data/week6/Landsat/LC80340322016205-SC20170127160728/crop/LC80340322016205LGN00_bqa_crop.tif"        
##  [2] "data/week6/Landsat/LC80340322016205-SC20170127160728/crop/LC80340322016205LGN00_cfmask_conf_crop.tif"
##  [3] "data/week6/Landsat/LC80340322016205-SC20170127160728/crop/LC80340322016205LGN00_cfmask_crop.tif"     
##  [4] "data/week6/Landsat/LC80340322016205-SC20170127160728/crop/LC80340322016205LGN00_sr_band1_crop.tif"   
##  [5] "data/week6/Landsat/LC80340322016205-SC20170127160728/crop/LC80340322016205LGN00_sr_band2_crop.tif"   
##  [6] "data/week6/Landsat/LC80340322016205-SC20170127160728/crop/LC80340322016205LGN00_sr_band3_crop.tif"   
##  [7] "data/week6/Landsat/LC80340322016205-SC20170127160728/crop/LC80340322016205LGN00_sr_band4_crop.tif"   
##  [8] "data/week6/Landsat/LC80340322016205-SC20170127160728/crop/LC80340322016205LGN00_sr_band5_crop.tif"   
##  [9] "data/week6/Landsat/LC80340322016205-SC20170127160728/crop/LC80340322016205LGN00_sr_band6_crop.tif"   
## [10] "data/week6/Landsat/LC80340322016205-SC20170127160728/crop/LC80340322016205LGN00_sr_band7_crop.tif"   
## [11] "data/week6/Landsat/LC80340322016205-SC20170127160728/crop/LC80340322016205LGN00_sr_cloud_crop.tif"   
## [12] "data/week6/Landsat/LC80340322016205-SC20170127160728/crop/LC80340322016205LGN00_sr_ipflag_crop.tif"
```

Above, we use the $ after .tif to tell R to look for files that end with .tif.
However, we want to grab all bands that both end with .tif AND contain the text
"band" in them. To do this we use the function glob2rx() which allows us to specify
both conditions. Here we tell R to select all files that have the word **band**
in the filename. We use a * sign before and after band because we don't know
exactly what text will occur before or after band. We use .tif$ to tell R that
each file needs to end with .tif.



```r
all_landsat_bands <- list.files("data/week6/Landsat/LC80340322016205-SC20170127160728/crop",
           pattern=glob2rx("*band*.tif$"),
           full.names = T) # use the dollar sign at the end to get all files that END WITH
all_landsat_bands
## [1] "data/week6/Landsat/LC80340322016205-SC20170127160728/crop/LC80340322016205LGN00_sr_band1_crop.tif"
## [2] "data/week6/Landsat/LC80340322016205-SC20170127160728/crop/LC80340322016205LGN00_sr_band2_crop.tif"
## [3] "data/week6/Landsat/LC80340322016205-SC20170127160728/crop/LC80340322016205LGN00_sr_band3_crop.tif"
## [4] "data/week6/Landsat/LC80340322016205-SC20170127160728/crop/LC80340322016205LGN00_sr_band4_crop.tif"
## [5] "data/week6/Landsat/LC80340322016205-SC20170127160728/crop/LC80340322016205LGN00_sr_band5_crop.tif"
## [6] "data/week6/Landsat/LC80340322016205-SC20170127160728/crop/LC80340322016205LGN00_sr_band6_crop.tif"
## [7] "data/week6/Landsat/LC80340322016205-SC20170127160728/crop/LC80340322016205LGN00_sr_band7_crop.tif"
```

Now we have a list of all of the landsat bands in our folder. We could chose to
open each file individually using the raster() function.


```r
# get first file
all_landsat_bands[2]
## [1] "data/week6/Landsat/LC80340322016205-SC20170127160728/crop/LC80340322016205LGN00_sr_band2_crop.tif"
landsat_band1 <- raster(all_landsat_bands[2])
plot(landsat_band1,
     main="Landsat cropped band 1\nColdsprings fire scar",
     col=gray(0:100 / 100))
```

<img src="{{ site.url }}/images/rfigs/course-materials/earth-analytics/week-6/2016-12-06-spectral03-landsat-tifs-in-R/unnamed-chunk-1-1.png" title=" " alt=" " width="100%" />

However, that is not a very efficient approach.
It's more efficiently to open all of the layers together as a stack. Then we can
access each of the bands and plot / use them as we want. We can do that using the
stack() function.


```r
# stack the data
landsat_stack_csf <- stack(all_landsat_bands)
# view stack attributes
landsat_stack_csf
## class       : RasterStack 
## dimensions  : 177, 246, 43542, 7  (nrow, ncol, ncell, nlayers)
## resolution  : 30, 30  (x, y)
## extent      : 455655, 463035, 4423155, 4428465  (xmin, xmax, ymin, ymax)
## coord. ref. : +proj=utm +zone=13 +datum=WGS84 +units=m +no_defs +ellps=WGS84 +towgs84=0,0,0 
## names       : LC8034032//band1_crop, LC8034032//band2_crop, LC8034032//band3_crop, LC8034032//band4_crop, LC8034032//band5_crop, LC8034032//band6_crop, LC8034032//band7_crop 
## min values  :                     0,                     0,                     0,                     0,                     0,                     0,                     0 
## max values  :                  3488,                  3843,                  4746,                  5152,                  5674,                  4346,                  3767
```

Let's plot each individual band in our stack.


```r
plot(landsat_stack_csf,
     col=gray(20:100 / 100))
```

<img src="{{ site.url }}/images/rfigs/course-materials/earth-analytics/week-6/2016-12-06-spectral03-landsat-tifs-in-R/plot-stack-1.png" title="plot individual landsat bands" alt="plot individual landsat bands" width="100%" />

## Plot RGB image

Next, let's plot an RGB image using landsat. Refer to the landsat bands table
below:


TABLE HERE

https://blogs.esri.com/esri/arcgis/2013/07/24/band-combinations-for-landsat-8/


```r
par(col.axis="white", col.lab="white", tck=0)
plotRGB(landsat_stack_csf,
     r=4, g=3, b=2,
     stretch="lin",
     axes=T,
     main="RGB composite image\n Landsat Bands 4, 3, 2")
box(col="white")
```

<img src="{{ site.url }}/images/rfigs/course-materials/earth-analytics/week-6/2016-12-06-spectral03-landsat-tifs-in-R/plot-rgb-1.png" title="plot rgb composite" alt="plot rgb composite" width="100%" />

## create landsat bands

CIR
other combos


```r
par(col.axis="white", col.lab="white", tck=0)
plotRGB(landsat_stack_csf,
     r=5, g=4, b=3,
     stretch="lin",
     axes=T,
     main="Color infrared composite image\n Landsat Bands 5, 4, 3")
box(col="white")
```

<img src="{{ site.url }}/images/rfigs/course-materials/earth-analytics/week-6/2016-12-06-spectral03-landsat-tifs-in-R/plot-cir-1.png" title="plot rgb composite" alt="plot rgb composite" width="100%" />