## WebEventDisplay

### Introduction 

The idea of this project is to have a simple way to visualise event and geometry data using nothing more than a web browser. The data should be as detector-agnostic as possible. It can currently show arbitrary geometry in either JSON, or OBJ format, and show some event data (JSON format). It is being [trialled](https://twitter.com/trackmllhc/status/971376668525973504) in e.g. the TrackML Kaggle challenge.

Technically the 3D is done with [three.js](http://threejs.org), the menu (at the moment) uses [DAT.GUI](https://code.google.com/archive/p/dat-gui/) and the data format is just plain JSON. 

The main webpage for the project is [here](https://gitlab.cern.ch/emoyse/WebEventDisplay).

### Installation 

To install, you need to clone this repository, using e.g. 
`git clone ssh://git@gitlab.cern.ch:7999/emoyse/WebEventDisplay.git`

In order to have access to the geometry files, you would need to do:
`scp -r lxplus.cern.ch:/eos/user/e/emoyse/www/WebEventDisplay/geometry .`
[FIXME - this obviously isn't a long term solution]

To avoid permissions problems with modern browsers you're much better off running a local webserver, e.g. with `python -m http.server` (python 3.X) or `python -m SimpleHTTPServer` (Python 2.X) and then point your browser to `http://0.0.0.0:8000`
See [here](https://threejs.org/docs/index.html#manual/introduction/How-to-run-things-locally) for more details (and alternative solutions).

### Using this with your own data 

The JSON format is pretty simple, but I'm still in the process of documenting it (and it might evolve). If you're unsure, you can always have a look at the `.json` files in http://emoyse.web.cern.ch/emoyse/WebEventDisplay

Otherwise, here are some rough explanations:

#### Event data 
Currently WED supports the following physics objects:

* Tracks - the trajectory of a charged particle (usually in a magnetic field)
* Calorimeter clusters - deposits of energy in a calorimeter
* Jets - cones of activity within the detector
* Hits - individual measurements

And coming soon:

* Vertices
* Compound objects (i.e. 'Muons', which link 'Tracks' and 'Clusters')

The format is the following:

``` 
{ "event number":XXX, "run number":YYY, "OBJECT_TYPE":{"COLLECTION_NAME" : [ OBJECTS ]}}""
```

where

* "event number" and "run number" are hopefully obvious, 
* OBJECT_TYPE is one of the supported types mentioned above, 
* and COLLECTION_NAME is an arbitrary name used to label this particular collection (you can have as many collections of each OBJECT_TYPE as you like).

Uniquely for clusters, you need to define the plane(s) on which to project the clusters as a property of the collection itself, using the following notation 

```
'CYL':[30.2493,243.645,136.947,213.396,3.14159,2850]
```

(other shapes will be supported soon)

What follows in the list of objects depends on the type:

* 'Tracks' have the following information: 
  * 'chi2' - the chi2 of the fit, i.e. a number 
  * 'dof' - the degrees of freedom of the fit, i.e. a number (not necessarily an integer)
  * 'params' - the parameters of the tracks, defined as d0,z0, ABC
  * 'pos' - further positions along the track (a possible extension is to store positions and directions, to permit bezier curves, and perhaps a simple extrapolation system which would further reduce the amount of information needing to be stored)
* 'Clusters' have the following information: 
  * 'phi' - phi direction
  * 'eta' - eta direction
* 'Jets' have the following information:
  * 'coneR' - the radius of the jet cone
  * 'phi'- phi direction
  * 'eta' - eta direction
  
As an example: 

```
{ "event number":123, "run number":234, "Tracks" : {"Inner Detector Tracks":[ {"chi2":52.1087, "dof":34, "params": [-0.0150713, 0.725162, 2.11179, 2.86823, -3.23906e-05], "pos": [] }}
```

When rendered, this might look like e.g.:

![An example event display](img/ATLAS.png)


### Geometry
#### Volumes

WED supports a limited set of volumes at the moment. All are passed with the following notation 
```
{ "Name" : XXX, "Shape" : YYY, "Bounds", [...], "Layers" : [...layers...], "Volumes" : [ ... sub-volumes ... ],  }
```
where the contents are:

* Name - an arbitrary name for the volume
* Shape - the shape of the volume (see below)
* Bounds - defines the size of the shape (see below)
* Layers - layers contained by this volume
* Volumes - sub-volumes contained by this volume

Currently the supported shapes are:

* 'CUB'/'BOX' - a cube in space, defined by its width/height/depth.
* 'CYL' - a cylinder, defined by inner radius, outer radius, medium radius, deltaRadius and halflengthZ.

#### Loading geometry from OBJ files
Obviously WED allows you to load OBJ files, by calling e.g. 
```
window.EventDisplay.loadGeomFromObj('geometry/ATLASR2/toroids.obj', 'Toroids', 0x878181);
```
For an example at the [CTD display](https://emoyse.web.cern.ch/emoyse/WebEventDisplay/jsdisplay_CTD.html).



