#### GISC 422 T1 2021

# Other stuff
Spatial interpolation is widely used in many domains and consequently there are many packages that do some part of it.

This page is a bit of a grab-bag of notes and observations first about other *R* packages, then about other platforms.

## Other *R* packages
Often like `fields` they aren't specifically geographical, so know nothing of projected coordinate systems or maps or cartography. That can make them tricky to work with when you have data in geospatial formats. Nevertheless it's worth knowing about a few of them:

### Proximity polygons without rasters
You can do 'interpolation' like this (and no... I don't understand why it requires quite so many steps as it does).

```{r}
library(sf)
interp_ext <- st_read("data/interp-ext.gpkg")
controls <- st_read("data/controls.gpkg")
controls_polys <- controls %>%
  st_union() %>%
  st_voronoi() %>%
  st_cast() %>%
  st_sf() %>%
  st_intersection(interp_ext) %>%
  st_join(controls)
plot(controls_polys)
```

### Inverse distance weighted interpolation in `spatstat`
Our old friend `spatstat` has an `idw` function. Understandably, it expects data in its own `ppp` format. Here's a recipe:

```{r message = FALSE}
library(maptools) # need this for the conversion
library(spatstat)
controls.pp <- as(as(controls, "Spatial"), "ppp")
idw_ss <- spatstat::idw(controls.pp, power = 2) # use package name to avoid clash with gstat
plot(idw_ss)
```

I leave it as an exercise for the reader to make that thing into a projected raster dataset...

### Geostatistics
`sgeostat` has been around forever and does a small part of the kriging puzzle, particularly variogram estimation. It's `spacecloud` and `spacebox` plots are particularly good for getting a feel for variogram estimation. For example:

```{r}
library(sgeostat)
controls_xyz <- read.csv("data/controls-xyz.csv")
pts <- sgeostat::point(controls_xyz)
prs <- sgeostat::pair(pts)
spacecloud(pts, prs, "z", cex = 0.35, pch = 19)
spacebox(pts, prs, "z")
```

### Splines
I'm sure there are others, but two packages that do splines are briefly discussed below

#### `MBA`
```{r}
library(MBA)
spline.mba <- mba.surf(controls_xyz,
                       no.X = 61, no.Y = 87, # much jiggery-pokery required
                       n = 87/61, m = 1,
                       extend = T, sp = T)
r <- raster(spline.mba$xyz.est)
crs(r) <- st_crs(controls)
persp(r, scale = FALSE, expand = 2, theta = 35, phi = 30, lwd = 0.5)
```

#### `akima`
```{r}
library(akima)
spline.akima <- interp(controls_xyz$x, controls_xyz$y, controls_xyz$z,
                       nx = 61, ny = 87,
                       extrap = T, linear = F)

r.spline.akima <- raster(spline.akima)
crs(r.spline.akima) <- st_crs(controls)

p <- persp(r.spline.akima, scale = FALSE, expand = 2, theta = 35, phi = 30, lwd = 0.5)
```

## Other platforms
Other platforms are available!

The goal here is to give insight into the wide array of options out there. We'll quickly look at tools in QGIS in class. Similar tools are available in the Esri ecosystem, which still supports the rather nice _Geostatistical Analyst_ tool.

Probably the one thing these both have that I can't find anywhere in *R*-land is _natural neighbours_ interpolation. (**UPDATE:** it does exist in *R*-land in a package called `whitebox` but good luck getting that setup to run cleanly).

The striking thing about both these menu / dialogue driven pathways is that with so many options to set things quickly become quite hard to replicate. As you experiment with options and find a preferred solution, it can be difficult to find your way back to it!

In any particular 'real-world' setting there may be other tools used for this very common family of operations.

This is the end of the line: back to [the overview](README.md)