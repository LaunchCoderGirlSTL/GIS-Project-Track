Rendering Data In OpenLayers
============================

Adding a Base Map
-----------------

Open up the OpenLayers project you set up at the beginning of this
tutorial. In a seperate terminal window, go ahead and build and serve
the project with ``npm start``.

Create a new folder at the top level of the project named ``data``.
Create a new file in data named ``osm-basemap.json``. Copy/paste the
`contents <https://gist.github.com/chrisbay/9adf9b57611fa1806d2c1dffa65f1df0>`__
of our ``osm-basemap.json`` file and save. This file is derived from the
``bright.json`` file that you worked with on the OpenLayers Workshop,
with a bit of customization. We’ve removed several unneeded layers to
clean up the map, and have scaled and cenetered the map on St. Louis.
Scroll down to line 41 and you’ll see:

.. code-block:: js

   "center": [
     -90.2549,38.6447
   ],
   "zoom": 13,
     

You can customize this if you want a slightly different initial center
or scaling.

Open main.js, and add the following lines:

.. code-block:: js

   import {apply} from 'ol-mapbox-style';

   const map = apply('map-container', './data/osm-basemap.json');

Visit `localhost:3000 <http://localhost:3000/>`__ in your browser and
you should see something like the following.


.. figure:: /_static/images/basemap.png
   :alt: The OpenStreetMaps base map centered on St. Louis

   Our base map, centered on St. Louis

It’s pretty, but also sort of plain. We’ll add our parks in the next
step.

If you do not see a map in your application, check to be sure the JSON
was copied correctly.

Enabling CORS in GeoServer
--------------------------

CORS stands for **Cross-Origin Resource Sharing**, and it refers to the
ability of a JavaScript resource from one origin to access resources
from another origin.

For example, a script hosted at ``mydomain.com`` may not access
resources at ``yourdomain.com`` unless the latter domain allows it to do
so. This is a security precaution, to prevent malicious scripts from
accessing resources they shouldn’t have access to. Most servers disable
CORS to some degree (or entirely). Resource sharing can be enabled
at-large (for requests from any origin) or on a case-by-case basis (only
for some origins).

We encourage you to `read more about
CORS <https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS>`__, as it
is an essential topic for anybody doing front-end development.

If you were to try to access data from GeoServer (hosted at
``localhost:8080``) via your OpenLayers/JavaScript app (hosted at
``localhost:3000``) you would likely see an error in your browser
console referring to the ``Access-Control-Allow-Origin`` HTTP header
(i.e. GeoServer). This header is set by the server that is restricting
cross-origin access. To enable CORS, we need to update GeoServer’s
configuration.

Getting Our Hands Dirty
^^^^^^^^^^^^^^^^^^^^^^^

From a terminal, SSH into your GeoServer VM:
``ssh -p 2020 training@localhost``. From there, go to the directory for
GeoServer, and then into the WEB-INF subdirectory.

::

   $ cd /opt/boundless/suite/geoserver/
   $ cd WEB-INF

This directory contains a file, ``web.xml``, that configures the
GeoServer application. Open the file with:

::

   $ sudo nano web.xml

Move your cursor to a blank line just inside the ``<web-app>`` tag (you
can create such a blank line, if necessary, but don’t go within any
other XML tag). Paste this code block:

.. code-block:: xml

   <filter>
     <filter-name>CorsFilter</filter-name>
     <filter-class>org.apache.catalina.filters.CorsFilter</filter-class>
   </filter>
   <filter-mapping>
     <filter-name>CorsFilter</filter-name>
     <url-pattern>/*</url-pattern>
   </filter-mapping>

This will allow access to GeoServer resources from *any* origin. For a
production server, this is definitely not something we’d want to allow,
but it’s fine for local development.

Save the file by entering ``ctrl+X`` and then answering "Y" when asked
if you want to save your changes. For the changes to take effect, we
need to restart the Tomcat server (i.e. the application server running
GeoServer).

::

   $ sudo service tomcat8 stop
   $ sudo service tomcat8 start

That’s it! In upcoming sections, you’ll be able to fetch geo data from
your OpenLayers app with no CORS errors.

Fetching Parks Data
-------------------

For these last sections, we leave some details to you to figure out.
Refer to the `OpenLayers
Workshop <https://openlayers.org/workshop/en/>`__ and the `OpenLayers
API <http://openlayers.org/en/latest/apidoc/index.html>`__ whenever get
stuck or need more info. Everything we’ll ask you to do will either have
been covered in the workshop, or will be covered in another related
resource that we’ll provide.

Creating the Layer
^^^^^^^^^^^^^^^^^^

Before we can fetch data from GeoServer, we need a layer in which to put
it. We’ll create a vector layer to store the features fetched from the
parks WFS. In this and all remaining sections, all JavaScript code
referenced will be in ``main.js``.

First, add a couple of imports:

.. code-block:: js

   import VectorLayer from 'ol/layer/vector';
   import VectorSource from 'ol/source/vector';

Then, below the line which creates the map, create a vector source and
layer, and add them to the map.

.. code-block:: js

   let parksSource = new VectorSource();
   let parksLayer = new VectorLayer({
     source: parksSource
   });

   parksLayer.setZIndex(50);
   map.addLayer(parksLayer);

Note the next to last line in the snippet above. This sets the z-index
of the parks layer to a relatively high number, which ensures that the
parks will display “on top of” other layers. Layers with higher z-index
are displayed above layers with lower z-index.

If you were to refresh your app, nothing would change. While we’ve added
a layer, there are not features in the layer to display!

Populaing the Layer
^^^^^^^^^^^^^^^^^^^

We’ll follow the `WFS - GetFeature
demo <https://openlayers.org/en/latest/examples/vector-wfs-getfeature.html>`__
provided by OpenLayers. Have a look at this code and try to decipher
what it is doing before continuing on here. Note that the code is
structured to be embedded in an HTML file, rather than in a separate JS
file. Hence, there are no imports and the OpenLayers references have the
full object namespaces referenced.

Let’s adapt this example for our purposes. In particular, we want to
adapt the two code blocks near the bottom, which look like this in the
example:

.. code-block:: js

   // generate a GetFeature request
   var featureRequest = new ol.format.WFS().writeGetFeature({
     srsName: 'EPSG:3857',
     featureNS: 'http://openstreemap.org',
     featurePrefix: 'osm',
     featureTypes: ['water_areas'],
     outputFormat: 'application/json',
     filter: ol.format.filter.and(
         ol.format.filter.like('name', 'Mississippi*'),
         ol.format.filter.equalTo('waterway', 'riverbank')
     )
   });

   // then post the request and add the received features to a layer
   fetch('https://ahocevar.com/geoserver/wfs', {
     method: 'POST',
     body: new XMLSerializer().serializeToString(featureRequest)
   }).then(function(response) {
     return response.json();
   }).then(function(json) {
     var features = new ol.format.GeoJSON().readFeatures(json);
     vectorSource.addFeatures(features);
     map.getView().fit(vectorSource.getExtent());
   });

This code is fetching water features from the WFS at
``https://ahocevar.com/geoserver/wfs``. Your task is to adapt each of
these blocks so that they work in your code. We’ll describe what each
portion of this example is doing, and leave the task of modifying it for
your app up to you.

Building the Request
~~~~~~~~~~~~~~~~~~~~

A GeoServer WFS request must be made in XML format. Building up XML
within JavaScript code would be very painful and clunky, so thankfully
OpenLayers provides an interface for us to configure our requests using
its JS API, and then turn a request object into XML. The first block in
the code above builds up the request. It specifies the request
parameters that we want to pass to GeoServer to cusomize the feature set
we get back.

Let’s look at each parameter of the call to ``writeGetFeature``:

-  ``srsName`` - specifies the SRS (or coordinate system) that we want
   our data in. Recall that the SRS of our parks data is EPSG:102696.
   This is different from the SRS of our base map. Thus, you’ll need to
   ask GeoServer to convert the park data to the same SRS as the base
   map by setting the value of this paramter. Which SRS is your OSM
   basemap rendered in? Figure that out, and use the value to customize
   your own request.
-  ``featureNS`` - the URI of the workspace that owns the layer
-  ``featurePrefix`` - the name of the workspace that owns the layer
-  ``featureTypes`` - the name of the WFS layer
-  ``outputFormat`` - the format that we want the data returned in (can
   be ``application/json``, ``application/xml``, etc)
-  ``filter`` - the filter to apply to the feature set. In the example
   above, only river bank water features that start with “Mississippi”
   are returned. If you leave off the ``filter`` parameter, all features
   will be returned.

For additional parameters that can be used with ``GetFeature``, see the
`OpenLayers API
documentation <http://openlayers.org/en/latest/apidoc/ol.format.WFS.html#writeGetFeature>`__.

Add a customized version of the above GetFeature call to your app. Be
sure to add the following import:

.. code-block:: js

   import WFS from 'ol/format/wfs';

With these, you can modify the first line to simply:

.. code-block:: js

   let featureRequest = new WFS().writeGetFeature({

Making the Request
~~~~~~~~~~~~~~~~~~

Now we will customize the second block, which looks like this:

.. code-block:: js

   fetch('https://ahocevar.com/geoserver/wfs', {
     method: 'POST',
     body: new XMLSerializer().serializeToString(featureRequest)
   }).then(function(response) {
     return response.json();
   }).then(function(json) {
     var features = new ol.format.GeoJSON().readFeatures(json);
     vectorSource.addFeatures(features);
     map.getView().fit(vectorSource.getExtent());
   });

In this example, a POST request is made to
``https://ahocevar.com/geoserver/wfs`` with body that contains XML
generated from the previously configured request object. In particular,
``new XMLSerializer().serializeToString(featureRequest)`` creates the
given XML.

The request is first handled by this function:

.. code-block:: js

   .then(function(response) {
     return response.json();
   })

which turns the response string into JSON.

The next handler uses the OpenLayers API to parse the JSON object into
GeoJSON format and extract the features. It then adds the features to
the map. Finally, it zooms the map to fit around the given set of
features.

.. code-block:: js

   .then(function(json) {
     var features = new ol.format.GeoJSON().readFeatures(json);
     vectorSource.addFeatures(features);
     map.getView().fit(vectorSource.getExtent());
   });

For your own code, you can make minimal modifications to this block to
get it working. In particular, you’ll need to customize the URL for the
request, and make sure all variable names are correct. We also recommend
using ``let`` instead of ``var`` to create your variables. You’ll also
likely want to remove the last line, which shifts the map view. Finally,
you’ll need the import:

.. code-block:: js

   import GeoJSON from 'ol/format/geojson';

Which will allow you to modify the line calling ``GeoJSON()``
accordingly.

Once you modify the code discussed to fit your projeject, you should see
parks rendered on the map!


.. figure:: /_static/images/park-features.png
   :alt: Map with park features outlined

   Park features rendered on our map

Next, we’ll style the parks to look nice.

Styling Features
----------------

For this section, we refer you to the `Making It Look
Nice <https://openlayers.org/workshop/en/vector/style.html>`__ section
of the OpenLayers workshop. Add a border (i.e. stroke) color and fill
color to the park features, making them look however you like.

Here’s how our map looks after styling:


.. figure:: /_static/images/park-styles.png
   :alt: Map with park features styled with dark green outlines and green fill color

   How the map will look after adding some styles

Making It Interactive
---------------------

Now we’ll turn our attention to display park names on user click. We’ll
largely follow the `OpenLayers workshop instructions on adding a
popup <https://openlayers.org/workshop/en/basics/popup.html>`__.

In fact, the tasks are so similar that we’ll let you review those
instructions and only mention the parts that are different for our case.

The only modifications you’ll need to make are in the click handler,
which we include here for reference:

::

   ﻿﻿﻿map.on('click', function(e) {
     overlay.setPosition();
     var features = map.getFeaturesAtPixel(e.pixel);
     if (features) {
       var coords = features[0].getGeometry().getCoordinates();
       var hdms = coordinate.toStringHDMS(proj.toLonLat(coords));
       overlay.getElement().innerHTML = hdms;
       overlay.setPosition(coords);
     }
   });

Getting Park Names
^^^^^^^^^^^^^^^^^^

The array of features, ``features``, representes all features at the
location that was clicked. For us, we this will only be one (there can
be only one park at a given location). Thus, ``features[0]`` will be the
park feature in question. You’ll want to put this feature in its own
variable, and then get the properties associated with the park using its
``getProperties`` method. See the `OpenLayers API
docs <http://openlayers.org/en/latest/apidoc/ol.Feature.html#getProperties>`__
for usage.

The park name will be one of the properties returned. Which one? Use
``console.log`` to print all of the properties to the browser JS console
to find out. Once you know the name of the property that contains each
park’s name, put it in the overlay.

Positioning the Overlay
^^^^^^^^^^^^^^^^^^^^^^^

The example code above works to position the overlay on features that
are points. If you try to use the same positioning calls with your
parks, which have extents (i.e. width and height) you’ll get some odd
results. Let’s look at how we can position the overlay more nicely.

The ``getGeometry`` method will get the full geometry of a feature. Use
this method to get the geometry for the park and place it in a variable
(and `read about
it <http://openlayers.org/en/latest/apidoc/ol.Feature.html#getGeometry>`__).

Once we have a park’s geometry, we can get its extent, which represents
the bounding box for the park. Calling ``getExtent`` on the geometry
object will return the extent in the form of an array, which looks like
``[minx, miny, maxx, maxy]``, where the four values represent the
corresponding coordinate values for the corners of the park’s bounding
box. Put the extent in yet another variable. `Read about the
method <http://openlayers.org/en/latest/apidoc/ol.geom.Geometry.html#getExtent>`__.

Once you have the extent, you might try to position the overlay at, say,
the top left using the ``minx`` and ``maxy`` coordinates. Try this and
see what the results are.

When you tried using the exact values of the extent, the overlay was
rarely, if ever, positioned in a way in which it touched the given park.
This is because the corner of the bounding box is usually outside of a
feature’s geometry. What we really want to do is to position the overlay
at the point on the park’s geometry that is *closest* to the top left
corner. Thankfully, there’s a method to help us with this!

``getClosestPoint`` can be called on a geometry object. It takes as a
parameter a point, and it returns the point within the given geometry
that is closest to the point paramter that it is given. `Read about the
method <http://openlayers.org/en/latest/apidoc/ol.geom.Geometry.html#getClosestPoint>`__,
and then use it to obtain the point within a park that is closest to the
top left corner of its bounding box. Then, position the overlay at this
point.

When you have it working, your map will look something like this:


.. figure:: /_static/images/overlay-positioned.png
   :alt: The final application. Park features, and the name of Tower Grove Park appears next to the park after it has been clicked on.

   Our final application

Congrats! Your St. Louis City parks map is looking great!

Continue on to the next section to read about some additional ways that
you might extend the app to continue learning about mapping
applications, OpenLayers, and GeoServer.






