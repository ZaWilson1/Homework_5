---
title: "Homework_5"
author: "Zane Wilson"
date: "2023-11-25"
output: pdf_document
---




Goal is to create a map showing the locations of homicides in Atlanta, GA. Use sf and tigris to download boundaries for the sub city geography (tracts, block groups, county subdivisions) to show as a layer underneath the points showing homicides. Differentiate solved and unsolved homicides using facets and show the three race groups with the highest number of homicides for the city. 





```r
library(tidyverse)
```

```
## -- Attaching core tidyverse packages ------------------------ tidyverse 2.0.0 --
## v dplyr     1.1.2     v readr     2.1.4
## v forcats   1.0.0     v stringr   1.5.0
## v ggplot2   3.4.4     v tibble    3.2.1
## v lubridate 1.9.2     v tidyr     1.3.0
## v purrr     1.0.1     
## -- Conflicts ------------------------------------------ tidyverse_conflicts() --
## x dplyr::filter() masks stats::filter()
## x dplyr::lag()    masks stats::lag()
## i Use the ]8;;http://conflicted.r-lib.org/conflicted package]8;; to force all conflicts to become errors
```

```r
library(tigris)
```

```
## To enable caching of data, set `options(tigris_use_cache = TRUE)`
## in your R script or .Rprofile.
```

```r
library(sf)
```

```
## Linking to GEOS 3.9.3, GDAL 3.5.2, PROJ 8.2.1; sf_use_s2() is TRUE
```

```r
library(forcats)
homicide <- read.csv("~/Work/Fall 2023/R for Research/Homework_5/data/homicide-data.csv")
```



```r
#Data just for Atlanta 

homicide <- homicide %>% 
  unite(col = city_name, city, state, sep = "," )

atlanta <- homicide %>% 
  filter(city_name %in% "Atlanta,GA")

#three race groups with the highest number of homicides.

atlanta<-atlanta %>% 
  mutate(victim_race =fct_lump_n(victim_race %>% as.factor, n=3))
```


```r
#Separating cases into solved and unsolved 

at_uns<-atlanta %>% 
  filter(disposition %in% c("Closed without arrest", "Open/No arrest"))

at_sol<- atlanta %>% 
  filter(disposition %in% "Closed by arrest")
```

```r
#create maps of Atlanta without murders
library(ggplot2)

GA_counties<-counties(state = "GA", cb = TRUE, class = "sf")
```

```
## Retrieving data for the year 2021
```

```
## 
```

```r
##Specifying Fulton county since its the seat of Atlanta proper

#tracts for Fulton County
ful_tracts <- tracts("GA", "Fulton", cb = TRUE, year = 2017)
```

```
## 
```

```r
#Block Groups for Fulton County
ful_block_g <- block_groups("GA", "Fulton", year = 2017)
```

```
## 
```

```r
#subdivisions for Fulton County
ful_subd<- county_subdivisions("GA", "Fulton", year = 2017)
```

```
## 
```

```r
#plot of Fulton County with no murders, chose to do tracts as it looked the best  
ggplot()+
  geom_sf(data = ful_tracts, color = "red")
```

![](Homework_5_files/figure-latex/unnamed-chunk-4-1.pdf)<!-- --> 

```r
#try to add all murders to the plot
ggplot()+
  geom_sf(data = ful_tracts, color = "blue")+
  geom_point(data = atlanta, aes(x = lon, y = lat))+
  ggtitle("Murders in Atlanta, County seat of Fulton")+
  labs(x = "Longitude", y = "Latitude")
```

![](Homework_5_files/figure-latex/unnamed-chunk-5-1.pdf)<!-- --> 
Some murders fall outside of Fulton County, so I need to add DeKalb and Clayton counties as well 


```r
#tracts for DeKalb county
dek_tracts<- tracts("GA", "DeKalb", year = 2017, cb = TRUE)

#tracts for Clayton County
clay_tracts <- tracts("GA", "Clayton", year= 2017, cb = TRUE)
```

```r
#plot with DeKalb and Clayton Counties as well
ggplot()+
  geom_sf(data = ful_tracts, color = "blue")+
  geom_sf(data = clay_tracts, color = "green")+
  geom_sf(data = dek_tracts, color = "red")+
  geom_point(data = atlanta, aes(x = lon, y = lat), alpha = 0.5, size = 0.75, color = "black")+
  ggtitle("Murders in Atlanta")+
  labs(x = "Longitude", y = "Latitude")+
  theme_dark()
```

![](Homework_5_files/figure-latex/unnamed-chunk-7-1.pdf)<!-- --> 

```r
#plot with unsolved murders and solved murders 
homicide_plot<- ggplot()+
  geom_sf(data = ful_tracts, color = "green")+
  geom_sf(data = clay_tracts, color = "cyan")+
  geom_sf(data = dek_tracts, color = "black")+
  geom_point(data = at_sol, aes(x = lon, y = lat), alpha = 0.5, size = 0.75, color = "blue")+
  geom_point(data = at_uns, aes(x = lon, y = lat), alpha = 0.5, size = 0.75, color = "red")+
  ggtitle("Murders in Atlanta")+
  labs(x = "Longitude", y = "Latitude")+
  theme_dark()
homicide_plot+coord_sf(ylim = c(33.63, 33.85), xlim = c(-84.55, -84.25))
```

![](Homework_5_files/figure-latex/unnamed-chunk-8-1.pdf)<!-- --> 


```r
#same plot, with races added in 
race_plot<- ggplot()+
  geom_sf(data = ful_tracts, color = "green")+
  geom_sf(data = clay_tracts, color = "cyan")+
  geom_sf(data = dek_tracts, color = "black")+
  geom_point(data = at_sol, aes(x = lon, y = lat, shape = victim_race), alpha = 0.5, size = 0.75, color = "blue")+
  geom_point(data = at_uns, aes(x = lon, y = lat, shape = victim_race), alpha = 0.5, size = 0.75, color = "red")+
  ggtitle("Murders in Atlanta")+
  labs(x = "Longitude", y = "Latitude")+
  theme_dark()
race_plot+coord_sf(ylim = c(33.63, 33.85), xlim = c(-84.55, -84.25))
```

![](Homework_5_files/figure-latex/unnamed-chunk-9-1.pdf)<!-- --> 


```r
##Going to try and create the same plot but with the original Atlanta data set, creating a new variable that determines if the case is solved or unsolved

atlanta_1 <- atlanta %>% 
  mutate(solved = case_when(disposition == 'Closed by arrest'~ 'solved',
                            disposition == 'Open/No arrest' | disposition == 'Closed without arrest' ~ 'unsolved'))
```


```r
#combine all the aspects from above into a comprehensive plot, this time with legends for solved vs unsolved.
final_plot<- ggplot()+
  geom_sf(data = ful_tracts, color = "green")+
  geom_sf(data = clay_tracts, color = "cyan")+
  geom_sf(data = dek_tracts, color = "black")+
  geom_point(data = atlanta_1, aes(x = lon, y = lat, color = victim_race, shape = solved), alpha = 0.75, size = 0.75)+
  ggtitle("Murders in Atlanta")+
  labs(x = "Longitude", y = "Latitude")+
  theme_dark()
final_plot+coord_sf(ylim = c(33.63, 33.85), xlim = c(-84.55, -84.25))
```

![](Homework_5_files/figure-latex/unnamed-chunk-11-1.pdf)<!-- --> 


```r
final_plot2<- ggplot()+
  geom_sf(data = ful_tracts, color = "gray")+
  geom_sf(data = clay_tracts, color = "cyan")+
  geom_sf(data = dek_tracts, color = "black")+
  geom_point(data = atlanta_1, aes(x = lon, y = lat, color = victim_race), alpha = 0.75, size = 0.75)+
  ggtitle("Murders in Atlanta")+
  labs(x = "Longitude", y = "Latitude", color = "victim race")+
  facet_wrap(~solved)+
  theme_bw()


final_plot2+coord_sf(ylim = c(33.63, 33.85), xlim = c(-84.55, -84.25))
```

![](Homework_5_files/figure-latex/unnamed-chunk-12-1.pdf)<!-- --> 