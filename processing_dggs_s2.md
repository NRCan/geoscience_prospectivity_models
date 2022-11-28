# load libraries
```
library(sf)			#for spatial operations
library(s2)			#for access to s2 functions
library(tidyverse)		#for data munging
library(rnaturalearth)		#for world polygons
```

# generate global polygon template 
```
# generate world map polygons using rnaturalearth::ne_countries
# exclude Antarctica
# fix geometries 
# fix geometries wrapping around dateline
# dissolve remaining country borders using using sf::union
mypoly <-     ne_countries(scale = "large", type = "countries", returnclass = c("sf")) %>%
              filter(!admin == "Antarctica") %>%
              st_make_valid() %>%
              st_shift_longitude() %>% 
              st_wrap_dateline() %>%
              st_union()

plot(mypoly)
```

# generate Canada polygon template
```
mypoly <- 	ne_countries(scale = "large", country = "Canada", returnclass = c("sf"))
plot(mypoly)
```

# calculate s2 cells for polygon template
# s2 cell statistics are here https://s2geometry.io/resources/s2cell_statistics.html
```
# use sf polygon to find overlapping s2 cells
mys2 <-       s2_covering_cell_ids(
              mypoly,
              min_level = 11,
              max_level = 11,
              max_cells = 100,
              buffer = 0,
              interior = FALSE,
              radius = s2_earth_radius_meters())

mys2

# convert list to vector of s2 cell ids
mys2_cell <-    unlist(mys2)

# calculate s2 cell centers
# convert to sf points
# extract longitude and latitude coordinates
mypoints <-     s2_cell_to_lnglat(mys2_cell)
mypoints_sf <-  st_as_sf(mypoints)
mylongitude <-  as.numeric(st_coordinates(mypoints_sf)[,1])
mylatitude <-   as.numeric(st_coordinates(mypoints_sf)[,2])

# convert vector of s2 cell ids to polygons 
mys2_geo <-     s2_cell_polygon(mys2_cell)
```

# generate s2 polygon grid
```
# convert s2 geography to sf polygons
# correct geometries
# correct polygon wrapping around dateline
mygrid <-       st_as_sf(mys2_geo) %>%
                mutate(s2_id = as.character(mys2_cell)) %>%
                mutate(s2_longitude = mylongitude) %>%
                mutate(s2_latitude = mylatitude) %>%
                st_make_valid() %>%
                st_wrap_dateline()

plot(mygrid)

st_write(mygrid, delete_layer = TRUE, "filepath here.gpkg")
```
