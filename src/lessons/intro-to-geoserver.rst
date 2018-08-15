.. _intro-to-geoserver:

Introduction to GeoServer
=========================

Up to this point, we have been working with OpenLayers applications that use data provided by others. This limits what we're able to do. The ability to create and customize our own GIS data sources is a key skill in building GIS applications. `GeoServer <http://geoserver.org/>`_ is an open-source application that will enable us to do just that. 


Before diving in and learning how to use GeoServer, let's first explore some concepts that are critical for understanding how GeoServer works.

Web Services
------------

The primary feature of GeoServer is to provide a means of easily creating web services that provide access to GIS data. What is a web service? We're about to find out. 

.. note:: The terms "web service" and "API" can have slightly different meanings in some contexts, but are also often used interchangeably. We will do this as well. For us, a web service and an API will be the same thing.

Read the linked articles, answering the questions provided as you go.

`What is an API? In English, please. <https://medium.freecodecamp.org/what-is-an-api-in-english-please-b880a3214a82>`_

This is a general introduction to the concept of an API. 

* What is the protocol used to access and API?
* Are APIs typically accessed via a web browser, via application code, or both?
* What type of data does an API often return in response to a request?
* Can an application use more than one API?

.. note:: There is one common data format often provided by APIs that is not mentioned in the article: `XML <https://en.wikipedia.org/wiki/XML>`_.

`Introduction to GeoServer: Server Basics <http://workshops.boundlessgeo.com/geoserver-intro/overview/index.html#geoserver-overview>`_

This reading includes some information a bit more specific to how GeoServer acts as a web service.

* What are the two protocols that GeoServer uses to deliver map data? (We'll learn more about each of these in a bit.)
* What is the name of the company that created and owns ArcGIS, an alternative to GeoServer? 
* Among the several GIS file types listed as being supported by GeoServer, find at least one vector file type and one raster file type.

.. note:: You will find `wiki.gis.com <http://wiki.gis.com/wiki/index.php/Main_Page>`_ useful in your research. If you need a refresher on what vector and raster mean, you'll also find that here. 

* Which organization is responsible for defining the WMS and WFS protocals? 

WMS and WFS
-----------

Read the following brief overviews of the Web Map Service (WMS) and Web Feature Service (WFS) protocols. These contain some technical terms that may be new to you, so focus on the high-level question of: What type of data does each service provide? 

`Introduction to GeoServer: Web Map Service (WMS) <http://workshops.boundlessgeo.com/geoserver-intro/overview/wms.html>`_

`Introduction to GeoServer: Web Feature Service (WFS) <http://workshops.boundlessgeo.com/geoserver-intro/overview/wfs.html>`_

.. admonition:: Example

   Open the `Google Maps search results <https://www.google.com/maps/search/restaurants/@38.6565286,-90.3064753,13z/data=!3m1!4b1!4m8!2m7!3m6!1srestaurants!2sLaunchCode+Mentor+Center,+4811+Delmar+Blvd,+St.+Louis,+MO+63108!3s0x87d8b4d4dfb1118d:0x46ba750d4f6e9fe1!4m2!1d-90.2595059!2d38.6515134>`_ for "restaurants near LaunchCode". There are two types of GIS data displayed here. The first is the base map, which contains roads, parks, municipality names, etc. This data is typically accessed via a WMS. The other data is the specific locations and names of restaurants. These are points (i.e their geometry is a single coordinate) and such data is typically accessed via a WFS.

Coordinate Systems
------------------

The Earth is not flat, yet maps typically are. This seemingly obvious statement has significant consequences to how we represent geospatial data. In particular, every piece of GIS data that we will interact with will have an associated **coordinate system**.

While this may sound like a new concept, you already know of one coordinate system, that defined by latitude and longitude.

Hop on over to Wikipedia and read the introductions to the articles on `latitude <https://en.wikipedia.org/wiki/Latitude>`_ and `longitude <https://en.wikipedia.org/wiki/Longitude>`_.

Now, find one example of a city with each of:

* both latitude and longitude postive
* positive latitude, negative longitude
* negative latitude, positive longitude
* both latitude and longitude negative

Another commonly-used coordinate system is **Universal Transverse Mercator (UTM)**. Read the introduction to the `Wikipedia article on UTM <http://wiki.gis.com/wiki/index.php?title=Universal_Transverse_Mercator&redirect=no>`_, along with the section titled, `"Locating a position using UTM coordinates" <http://wiki.gis.com/wiki/index.php?title=Universal_Transverse_Mercator&redirect=no#Locating_a_position_using_UTM_coordinates>`_.

* What are the three pieces of information needed to specify a location using UTM cooridnates?
* What are the coordinates of St. Louis in latitude/longitude? UTM? Use Google to find these. 

.. note:: St. Louis is not a point, of course. It's geometry is more properly a polygon. So when we talk about the "coordinates of St. Louis" we mean a point within the city limits. 

In the world of GIS, coordinate systems are often called **spatial reference systems (SRS)**. More specifically, an SRS is a coordinate system that has been formally defined and catalogued for use by GIS analysts.

The `EPSG registry <http://epsg.io/>`_ is a public database of all spatial reference systems. In this database, coordinate systems are referred to using their **spatial reference identifier**, which looks something like "EPSG:4326" (sometimes the "EPSG" portion is omitted). 

Visit the EPSG registry, and search for "4326." 

* What is the name of this SRS? 
* What are a few of the data formats that are provided to specify the SRS?

The concept of an SRS, or coordinate system, will be critical to displaying our maps correctly. Since data from different sources might be represented in different coordinate systems, we'll need to account for the SRS of a data source to be sure that the various componetns of our maps align correctly. 

With these concepts, you are now prepared to get your hands dirty with GeoServer. Start with :ref:`geoserver-setup`.