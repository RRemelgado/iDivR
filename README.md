### Global, monthly Land Surface Temperature (LST)
<p align="justify">
This exercise aims to derive global, day/night, monthly LST based on 8-day TERRA (<a href="https://lpdaac.usgs.gov/dataset_discovery/modis/modis_products_table/mod11a2_v006">MOD11A2</a>) and AQUA (<a href="https://lpdaac.usgs.gov/dataset_discovery/modis/modis_products_table/myd11a2_v006">MYD11A2</a>) data. My algorithm (Fig. 1) focuses on tile-wise processing. For each target tile, the algorithm performs the following, general steps:
<item>Downloads TERRA and AQUA data from the <a href="https://ladsweb.modaps.eosdis.nasa.gov/">LAADS DAAC</a> server</item>
<item>Combines TERRA and AQUA data on a daily basis</item>
<item>Interpolates data gaps and derives mean LST</item>
Finnally, a global mosaic is build for each month using all tiles. 
</p>

<figure>
  <p align="center"><img src="https://github.com/RRemelgado/iDivR/blob/master/inst/extdata/diagram.jpg" width="600"></p>
  <p align="center"><small>Figure 1 - Algorithm work flow</small></p>
</figure>

</br>

### Programming language
<p aligh="justify">
While my algorithm uses R, this language serves mainly as a wrapper for tasks such as data download. When perfoming RAM demanding tasks such as e.g. gap-filling, the algorithm calls the Python provided through the the `reticulate` package and the GDAL bindings offered by the `gdalUtils` package delivering the data processing to the respective platforms.
</p>

</br>

### Selection of target tiles
<p align="justify">
Looking at the <a href="https://ladsweb.modaps.eosdis.nasa.gov/">LAADS DAAC</a> server, we can already see that tiles with no overlapping land masses were already excluded. However, there are still tiles that overlap with very small land masses (i.e. area smaller than the product's pixel resolution). To avoid the time consuming download of these tiles, I first retrived a shapefile with the world's administrative boundaries (acquired <a href="https://biogeo.ucdavis.edu/data/gadm3.6/gadm36_shp.zip">here</a>) and, for each polygon, estimated the percent pixel coverage for a 1200 x 1200m grid (i.e. final product grid resolution) using the `poly2sample()` function of the <a href="">fieldRS</a> package. This function identifies all the pixels that are covered by a polygon and, for each pixel, quantifies the percent overlap. 
</p>


```r
require(fieldRS)

# read auxiliary data
cadm <- shapefile("country administrative boundaries")
tile <- shapefile("MODIS tile shapefile")

# filter polygons where no pixel is completely comtained
cadm.points <- poly2sample(s.tmp, 1200)
cadm <- intersect(cadm, cadm.points[cadm.points$cover == 100,])

# target tiles
tiles <- unique(cadm$tile)

```

<p align="justify">
Using this data, I filtered out all polygons where the minimum percent overlap was lesser than 100% (Fig. 2). Then, I downloaded a shapefile with the MODIS tiles (acquired <a href="http://book.ecosens.org/wp-content/uploads/2016/06/modis_grid.zip">here</a>) and queried the final set of tiles. Considering the build-up of a LST global, monthly composites for 1 year, <b><u>this step avoided the download of 1.4 Tb</u></b> of data.
</p>

<figure>
  <p align="center"><img src="https://github.com/RRemelgado/iDivR/blob/master/inst/extdata/admFilter.jpeg" width="800"></p>
  <p align="center"><small>Figure 2 - Red circles highlight land masses that were excluded from further processing</small></p>
</figure>

</br>

### Data storage: how is the dta handled?
</p align="align">
My programming solution avoids the storage of large amounts of data unless necessary. To achieve this, I keep the most basic tasks (i.e. data download, masking, interpolation, compositing) on a tile-by-tile basis. For each tile, once the pre-procesisng is completed, all temporaly files (e.g. .hdf's) are deleted. Moreover, only one image is kept for each 8-day composite achieved by combining TERRA and AQUA thus halving the required data storage.
</p

</br>

### Parallel processing
<p align="justify">
For the base processing, tile-wise tasks, my algorithm uses 
</p>