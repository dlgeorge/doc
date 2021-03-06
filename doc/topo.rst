

.. _topo:

*****************************************************************
Topography data
*****************************************************************

The :ref:`geoclaw` software for flow over topography requires at least one
topo file to be input, see :ref:`setrun_geoclaw`.

Currently topo files are restricted to three possible formats as ASCII files.
A future project is to allow other formats including NetCDF.

In the descriptions below it is assumed that the topo file gives the
elevation of the topography (relative to some reference level) as a value of
z at each (x,y) point on a rectangular grid.  Only uniformly spaced
rectangular topo grids are currently recognized.  

More than one topo file can be specified (see :ref:`setrun_topo`) that might
cover overlapping regions at different resolutions.  The union of all the
topo files should cover the full computational domain specified (and may
extend outside it).  Internally in :ref:`geoclaw` a single
piecewise-bilinear function is constructed from the union of the topo files,
using the best information available in regions of overlap.  This function
is then integrated over computational grid cells to obtain the single topo value
in each grid cell needed when solving depth averaged equations such as the
shallow water equations with these finite volume methods.  Note that this
has the feature that if a grid cell is refined at some stage in the
computation, the topo used in the fine cells have an average value that is
equal to the coarse cell value.  This is crucial in maintaining the
ocean-at-rest steady state, for example.

The recognized topotypes are:

  **topotype = 1**

    x,y,z values on each line, progressing from upper left (NW) corner across
    rows (moving east), then down in standard GIS form.  
    The size of the grid and spacing
    between the grid points is deduced from the data.  

    *Example:* if you want a flat bottom at B = -1000.
    over a domain  0. <= x <= 10. and  20. <= y <= 30.
    then the topo file could be simply::

        0.  30.  -1000.
        10. 30.  -1000.
        0.  20.  -1000.
        10. 20.  -1000.

    These files are larger than necessary since they store the x,y values at
    each point even though the points are required to be equally spaced.
    Many data sets come this way, but note that you can convert a file of
    this type to one of the more compact types below using
    `clawpack.geoclaw.topotools.converttopotype(inputfile, outputfile,
    topotypein=1, topotypeout=2, nodata_value=None)`.



  **topotype = 2**

    The file starts with a header consisting of 6 lines containing::

      mx
      my
      xllcorner
      yllcorner
      cellsize
      nodataval

    and is followed by mx*my lines containing the z values at each x,y,
    again progressing from upper left (NW) corner across
    rows (moving east), then down in standard GIS form.  
    The lower left corner of the grid
    is *(xllcorner, yllcorner)* and the distance between grid points in both
    x and y is *cellsize*.  The value *nodataval* indicates what value of z
    is specified for missing data points (often something like 9999. in data
    sets with missing values).

    *Example:*  For the same example as above, the topo file with
    topotype==2 would be::

      2         mx
      2         my
      0.        xllcorner
      20.       yllcorner
      10.       cellsize
      9999.     nodatavalue
      -1000.
      -1000.
      -1000.
      -1000.


  **topotype = 3**

    The file starts with a header consisting of 6 lines as for *topotype=2*,
    followed by *my* lines, each containing *mx* values for one row of data
    (ordered as before, so the first line of data is the northernmost line
    of data, going from west to east).

    *Example:*  For the same example as above, the topo file with
    topotype==3 would be::

      2         mx
      2         my
      0.        xllcorner
      20.       yllcorner
      10.       cellsize
      9999.     nodatavalue
      -1000.  -1000.
      -1000.  -1000.

This is essentially the same as the `ESRI ASCII Raster format
<http://resources.esri.com/help/9.3/arcgisengine/java/GP_ToolRef/spatial_analyst_tools/esri_ascii_raster_format.htm>`_
except that the Fortran code assumes the parameter values come first rather
than the labels.

**New in 5.3.0:** The Fortran code will recognize headers for *topotype* 2
or 3 that have the labels first and then the parameter values.  Note that
the label strings are ignored in either case, the order of lines 
is important.

It is also possible to specify values -1, -2, or -3 for *topotype*, in which
case the *z* values will be negated as they are read in (since some data
sets use different convensions for positive and negative values relative to
sea level). 

For :ref:`geoclaw` applications in the ocean or lakes (such as tsunami
modeling), it is generally assumed that *sea_level = 0* has been set in
:ref:`setrun_geoclaw` and that *z<0* corresponds to subsurface bathymetry
and *z>0* to topograpy above sea level.

.. _topo_sources:

Downloading topography files
----------------------------

The example
`$CLAW/examples/tsunami/chile2010
<claw/examples/tsunami/chile2010/README.html>`_
is set up to automatically download topo files via::

	$ make topo

See the `maketopo.py` file in that directory.

Other such examples will appear in the future.  

Several on-line databases are available for topograpy, see 
:ref:`tsunamidata` for some links.

Some Python tools for working with topography files are available, see
:ref:`topotools`.

.. _topo_dtopo:

Topography displacement files
-----------------------------

.. warning::  Some problems have recently been observed when trying to
   specify time-varying topography with `dtopo` files.  Nearly instantaneous
   displacement occuring at the start seems to work ok, but slowly varying
   displacement does not always work well when AMR is also being used.
   A better version of this code is currently being developed, but for now
   use with caution!

   This has been fixed in Clawpack 5.1.0.


For tsunami generation a file *dtopo* is generally used to specify the
displacement of the topography relative to that specified in the topo files.

Currently two formats are supported for this file: 

    **dtopotype=1:** 

    Similar to
    topo files with *topotype=1* as described above, except that each line
    starts with a *t* value for the time, so each line contains t,x,y,dz

    The x,y,dz values give the displacement dz at x,y at time t.  It is assumed
    that the grid is uniform and that the file contains mx*my*mt lines if mt
    different times are specified for an mx*my grid.  

    **dtopotype=3:** 

    Similar to
    topo files with *topotype=3* as described above, but the header is
    different, and contains lines specifying *mx, my, mt, xlower, ylower, t0,
    dx, dy*, and *dt*.  These are followed by *mt* sets of *my* lines, 
    each line containing *mx* values of *dz*.

The Okada model can be used to generate *dtopo* files from fault parameters,
as described in :ref:`okada`. 

Note that if the topography is moving, it is important to insure that the
time step is small enough to capture the motion.  Starting in Version 5.1.0,
there is a new parameter that can be specified in `setrun.py`
to limit the size time step used during the time when topography is moving. 
See :ref:`setrun_topo`.

.. _qinit_file:

qinit data file
---------------

Instead of (or in addition to) specifying a displacement of the topography
it is possible to specify a perturbation to the depth, momentum, or surface
elevation of the initial data.  This is generally useful only for tsunami
modeling where the initial data specified in the default *qinit.f90* function
is the stationary water with surface elevation equal to *sea_level* as set in
`setrun.py` (see :ref:`setrun_geoclaw`).  

Of course it is possible to copy the *qinit.f90* function to your
directory and modify it, but for some applications the initial elevation may
be given on grid of the same type as described above.  In this case file can
be provided as described at :ref:`setrun_qinit` containing this
perturbation.

The file format is similar to what is described above for *topotype=1*, but
now each line contains *x,y,dq* where *dq* is a perturbation to one of the 
components of *q* as specified by the value of *iqinit* specified (see
:ref:`setrun_qinit`).  If *iqinit = 4*, the value *dq* is instead the
surface elevation desired for the initial data and the depth *h* (first
component of *q*) is set accordingly.

