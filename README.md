## IBCSO

International Bathymetric Chart of the Southern Ocean (IBCSO) is the authoritative map of the Southern Ocean, providing a raster bathymetric chart of 
depths at  

currently at v2…

We download it and convert to Cloud Optimized GeoTIFF and then host it
at this url:

```
wget https://download.pangaea.de/dataset/937574/files/IBCSO_v2_ice-surface.tif

gdal_translate IBCSO_v2_ice-surface.tif IBCSO_v2_ice-surface_cog.tif -of COG -co COMPRESS=ZSTD -co PREDICTOR=2 -co SPARSE_OK=YES -co OVERVIEW_RESAMPLING=AVERAGE -co BLOCKSIZE=480 -co NUM_THREADS=ALL_CPUS
```

We chose options blocksize 480 so that the smallest overview would be that size (at 4800 we only end up with 2 overview levels). 

SPARSE_OK: ensures unused blocks are not stored (they are all 0, or nodata). 

COMPRESS: could choose ZSTD instead

OVERVIEW_RESAMPLING: AVERAGE so that the average value is stored from higher resolution levels, not just a sample

NUM_THREADS: this just makes it go faster (for the compression). 

Some comparisons on performance are shown below:



```R
dsn <- "/vsicurl/https://github.com/mdsumner/ibcso-cog/raw/main/IBCSO_v2_ice-surface_cog.tif"
library(terra)
system.time({
r <- rast("")
plot(project(r, rast(), by_util = T))
template <- rast(r)
res(template) <- res(r) * 20


## this extent is one tile at 1/20 resolution
project(r, rast(ext(c(90000, 1e+05, 3930000, 3940000)), crs = crs(r), res = 500), by_util = TRUE)

## these work very quickly
plot(project(r, rast(ext(-180, 180, -90, -50)), by_util = T))
plot(project(r, rast(ext(-180, 180, -90, -50), res = 1), by_util = T))
plot(project(r, rast(ext(-180, 180, -90, -50), res = .25), by_util = T))
})

## if we use the local .tif it's much slower
system.time(project(rast("IBCSO_v2_ice-surface.tif"), rast(ext(-180, 180, -90, -50), res = .25), by_util = T))

```

In Python it's similar, we can do 

```python
import rasterio
ds = rasterio.open("/vsicurl/https://github.com/mdsumner/ibcso-cog/raw/main/IBCSO_v2_ice-surface_cog.tif")

```
