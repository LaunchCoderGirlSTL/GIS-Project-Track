Introduction to GeoServer
=========================

TODO - intro paragraph

GeoServer Concepts
------------------

What is an API? 

What is a web service?

- WFS
- WMS

GeoServer Setup
---------------

- Install VirtualBox
- Download and install VM image
- Test

GeoServer + OpenLayers Tutorial Overview
----------------------------------------

The next few lessons demonstrate how to use GeoServer and OpenLayers together to create dynamic, interactive mapping applications. It assumes that you have completed all prior lessons, including the OpenLayers workshop.

The Project
^^^^^^^^^^^

We will use a data set published by the `City of St. Louis Open Data <https://www.stlouis-mo.gov/data/>`_ project to render St. Louis City parks on a map. The end result will look something like this, displaying park names when the user clicks on a given park.

.. figure:: /_static/images/stl-parks.png
    :alt: Screenshot of a web application displaying a map of St. Louis parks

    The St. Louis Parks app, using GeoServer and OpenLayers

.. _ol-geoserver-setup:

Setup
^^^^^

Clone the repository, renaming the project directory as you do so:

::

    $ git clone https://github.com/LaunchCodeEducation/openlayers-template.git stl-parks

Then create a new GitHub repository named ``stl-parks`` in your account to store your project code. Connect your local repository to your new remote repository by running these Git commands from the project root

::

    $ git remote rm origin
    $ git remote add origin REPO_URL
    $ git push origin master

Here, ``REPO_URL`` is the project URL (ending in ``.git``) for the repository you created above.

Install the project dependencies via NPM. From the project root, run ``npm install``.

Then make sure the project builds and runs by running ``npm start`` and visiting ``localhost:3000`` in your browser. You should see a "Hello World" message in an alert box.

You are now ready to learn about :ref:`publishing-data-in-geoserver`.