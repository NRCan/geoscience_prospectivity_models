# load libraries
```
library(tidyverse)      #for data munging
library(sf)             #for vector operations
library(raster)         #for raster operations

library(fasterize)      #for rasterizing vector data
library(exactextractr)  #for zonal statistics
library(tictoc)         #for time calculations
```

# load input raster data
```
r <- raster("filepath here")
plot(r)
```

# load H3 grid as polygon
```
grid <- st_read("filepath here")
```

# convert raster data to H3 grid using zonal statistics
```
tic()
calc <- exactextractr::exact_extract(r, grid, c("majority", "minority", "max", "min", "mean"))
toc()

dropnames <- colnames(calc)
```

# prepare lookup table to join zonal statistics with H3 grid
```
forbind <-	  calc %>%
		          mutate(majority = majority, minority = minority, maximum = max, minimum = min, mean = mean) %>%
		          dplyr::select(-all_of(dropnames))
```

# combine lookup table with H3 grid
```
grid_join <-	bind_cols(grid, forbind)
```

# drop geometries and convert to dataframe for export
```
grid_df <-	  grid_join %>%
		          st_drop_geometry() %>%
		          as.data.frame() 
```

# write dataframe to csv
```
write.csv(grid_df, row.names = FALSE, "filepath here")
```
