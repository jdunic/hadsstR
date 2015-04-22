A library to examine change in Sea Surface Temperatures
-------------------------------------------------------

This library was developed following my own interest in replicating some
of the analyses from Burrows et al. 2011 for SST change and climate
velocity using the Met Office Hadley Centre Sea Surface Temperature 1.1
dataset, available at
<http://www.metoffice.gov.uk/hadobs/hadisst/data/download.html> in
netcdf format (the file HadISST\_sst.nc.gz gunzipped).

It provides a simple function to load the netcdf data.

Installation
------------

    install.packages("ncdf")
    install.packages("chron")

    #install.packages("devtools")
    library(devtools)
    install_github("hadsstR", "jebyrnes")

Loading and Creating Derived Data for Analysis
----------------------------------------------

    library(hadsstR)
    sstData <- loadHadSST(directory="../", hadsstFilename="HadISST_sst.nc") 
    summary(sstData)

    ##          Length    Class  Mode   
    ## sstArray 112557600 -none- numeric
    ## lat            180 -none- numeric
    ## lon            360 -none- numeric
    ## tdates        1737 Date   numeric

One can then work with the dataset in a number of ways. For now, the
functions all begin by creating annual averages, and working from there.
The getClimateChange function generates a list of different matrices
that can be used for further analyses of climate change.

    climateChangeMats <- getClimateChange(sstData, years=1960:2009)

We can plot a number of derived quantities from this object. For
example, average temperature over the timespan

    #Let's plot some of these matrices
    pal <- colorRampPalette(c("blue","white", "red"))

    with(climateChangeMats, image(lon, lat, averageMat, col=pal(80)))

![](README_files/figure-markdown_strict/averagePlot-1.png)

Or the slope of the regression between temperature and year for each
lat/long cell

    library(lattice)
    latLonGrid <- expand.grid(lon = climateChangeMats$lon, lat = climateChangeMats$lat)

    levelplot(climateChangeMats$linearChangeMat ~ lon * lat, 
      data = latLonGrid, col.regions = pal(101), at=seq(-1,1,length.out=101))

![](README_files/figure-markdown_strict/linearChangePlot-1.png)

We can look at spatial gradients in temperature

    pal2 <- colorRampPalette(c("darkblue", "blue", "green", "white", "yellow", "orange", "red"))
    with(climateChangeMats, image(lon, lat, spatialGradMat, col=pal2(101)))

![](README_files/figure-markdown_strict/gradientPlots-1.png)

    palNS <- colorRampPalette(c("green", "white", "purple"))
    levelplot(climateChangeMats$NSmat ~ lon * lat, col.regions=palNS(100),
              data = latLonGrid, at=seq(-0.03, 0.03, length.out=100))

![](README_files/figure-markdown_strict/gradientPlots-2.png)

    palWE <- colorRampPalette(c("orange", "white", "blue"))
    levelplot(climateChangeMats$WEmat ~ lon * lat, col.regions=palWE(100),
      data = latLonGrid, at=seq(-0.025, 0.025, length.out=100))

![](README_files/figure-markdown_strict/gradientPlots-3.png)

And with all of this, we can see climate change velocity

    #create a velocity matrix where values >200 and < -200 are truncated to those limits
    #for easier plotting, as in Burrows et al. 20011
    velMatTruncated <- climateChangeMats$velocityMat
    velMatTruncated[velMatTruncated >200] <- 200
    velMatTruncated[velMatTruncated < -200] <- -200

    levelplot(velMatTruncated ~ lon * lat, data = latLonGrid, #at = cutpts, 
               pretty = T, 
              col.regions = pal(100),
               at=seq(-200,200,length.out=100))

![](README_files/figure-markdown_strict/velocityPlots-1.png)

If we want to extract values for particular metrics at particular
latitudes and longitudes, we can get the value at the closes lat/long in
the data.

    getClimateLatLon(climateChangeMats, -50.232, -100.55, "linearChange")

    ## [1] 0.0268986

Note that in the future I hope to add more functionality and deal with
seasonal data. These methods *should* also work for other Hadley Centre
data sets, but I have not yet tested them.

Version numbers
---------------

`hadsstR` uses [semantic versioning](http://semver.org/). The version
numbering scheme is `major`.`minor`.`revision`. Unless `major` is 1, the
package should not be considered stable. All releases with the same
`major` versions are compatible. Increases in `minor` represents the
addition of backwards-compatible additions. Increases in `revision`
represents either bug fixes or improvements.

Contributions
-------------

People wanting to contribute are welcome to do so by forking the
repository, and submitting a pull request when their work is done.
Please also edit the `DESCRIPTION` file to add your name to the
`Authors` field.
