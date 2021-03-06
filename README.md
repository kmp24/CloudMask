
### title: "Cloud Mask"
### author: "Kaitlyn Perham"
### date: "May 30, 2017"


```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```
```{r include = FALSE}
library(raster)
library(rgdal)
library(rgeos)
options(stringsAsFactors = F)
```

This exercise will examine how to make a cloud mask using r.
![Fire Scar](https://github.com/kmp24/CloudMask/blob/master/1.png)

```{r echo = FALSE}
all_landsat_bands <- list.files("C:/Users/kaitlyn/documents/earth-analytics/data/week6/Landsat/LC80340322016189-SC20170128091153/crop",
                            pattern=glob2rx("*band*.tif$"),
                            full.names = T)
all_landsat_bands_st <- stack(all_landsat_bands)
```
```{r echo = FALSE}
par(col.axis="white", col.lab="white", tck=0)
plotRGB(all_landsat_bands_st,
        r=4, g=3, b=2,
        stretch="hist",
        main="Pre fire RGB image, with cloud\n Cold Springs Fire",
        axes=T)
box(col="white")
```
![Fire Scar w/cloud](https://github.com/kmp24/CloudMask/blob/master/2.png)

![Fire Scar w/cloud and shadow](https://github.com/kmp24/CloudMask/blob/master/3.png)

This file shows not only cloud cover, but cloud shadow and water in the image.
```{r echo = FALSE}
cloud_mask_189_conf <- raster("C:/Users/kaitlyn/documents/earth-analytics/data/week6/Landsat/LC80340322016189-SC20170128091153/crop/LC80340322016189LGN00_cfmask_conf_crop.tif")
plot(cloud_mask_189_conf,
  main="Landsat Julian Day 189 - Cloud mask layer.")
cloud_mask_189 <- raster("C:/Users/kaitlyn/documents/earth-analytics/data/week6/Landsat/LC80340322016189-SC20170128091153/crop/LC80340322016189LGN00_cfmask_crop.tif")
plot(cloud_mask_189,
  main="Landsat Julian Day 189 - Cloud mask layer with shadows.")
```

Next, we'll evaluate our data using NA values in place of cloud and shadow values.

```{r echo = TRUE}
cloud_mask_189[cloud_mask_189 > 0] <- NA
plot(cloud_mask_189,
     main="New raster mask",
     col=c("green"),
     legend=F,
     axes=F,
     box=F)
par(xpd=T) # force legend to plot outside of the plot extent
legend(x = cloud_mask_189@extent@xmax, cloud_mask_189@extent@ymax,
       c("Not masked", "Masked"),
       fill=c("green", "white"),
       bty="n")
```

![Mask](https://github.com/kmp24/CloudMask/blob/master/4.png)

Next the mask is applied to all bands in the data:

```{r echo=TRUE}
all_landsat_bands_mask <- mask(all_landsat_bands_st, mask = cloud_mask_189)
par(col.axis="white", col.lab="white", tck=0)
plotRGB(all_landsat_bands_mask,
        r=4, g=3, b=2,
        stretch="lin",
        main="RGB image - clouds removed from image",
        axes=T)
box(col="white")
```
![Result](https://github.com/kmp24/CloudMask/blob/master/5.png)

It appears this image has a large proportion of the study area with NA values. The next step is to search for other imagery for this region with the same quality, in the timeframe needed. The affected area under study is small, limiting the options for data sources.
