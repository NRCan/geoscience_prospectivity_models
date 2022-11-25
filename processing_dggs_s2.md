# load libraries
```
library(sf)			#for spatial operations
library(s2)			#for access to s2 functions
library(tidyverse)		#for data munging
library(rnaturalearth)		#for world polygons
```

# generate polygon
```
# generate world map polygons using rnaturalearth::ne_countries
# exclude Antarctica
# fix geometries 
# fix geometries wrapping around dateline
# dissolve remaining country borders using using sf::union
mypoly <- 	ne_countries(scale = "large", type = "countries", returnclass = c("sf")) %>%
		filter(!admin == "Antarctica") %>%
		st_make_valid() %>%
		st_shift_longitude() %>% 
  		st_wrap_dateline() %>%
		st_union()

plot(mypoly)
```

# calculate s2 cells
```
# use sf polygon to find overlapping s2 cells
mys2 <- 
s2_covering_cell_ids(
mypoly,
min_level = 11,
max_level = 11,
max_cells = 100,
buffer = 0,
interior = FALSE,
radius = s2_earth_radius_meters())

mys2

# convert list to vector of s2 cell ids
mys2_cell <- unlist(mys2)

# convert vector of s2 cell ids to polygons 
mys2_geo <- s2_cell_polygon(mys2_cell)
```

# generate s2 polygon grid
```
# convert s2 geography to sf polygon
# correct geometries
# correct polygon wrapping around dateline
mygrid <- 	
st_as_sf(mys2_geo) %>%
mutate(s2_id = as.character(mys2_cell)) %>%
st_make_valid() %>%
st_wrap_dateline()

plot(mygrid)
```
