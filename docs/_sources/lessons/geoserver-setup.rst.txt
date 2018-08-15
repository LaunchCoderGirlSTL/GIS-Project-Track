.. _geoserver-setup:

GeoServer Setup
===============

Installation
------------

We will be working with a GeoServer instance that has been set up for you inside of a virtual machine. This will minimize the amount of installation and configuration, while provide the same basic setup for all students, regardless of operating system.

VirtualBox
^^^^^^^^^^

If you don't have it already installed, install VirtualBox by finding the package specific to your operating system on the `VirtualBox downloads page <https://www.virtualbox.org/wiki/Downloads>`_.

VM Installation
^^^^^^^^^^^^^^^

Download the `GeoServer virtual machine image <https://www.dropbox.com/s/tioi1tp7dn8irak/GeoServer%20%2864%20bit%29.ova?dl=0>`_. Double-click on the downloaded file to set up the image in VirtualBox.

Accessing GeoServer
^^^^^^^^^^^^^^^^^^^

With your virtual machine installed, you can access GeoServer with a few steps.

1. Start the VM by right-clicking on it in the VirtualBox pane and selecting *Start > Headless Start*. This will start up the VM "in the background."

.. figure:: /_static/images/vm-headless-start.png
   :alt: Screenshot of the VirtualBox menu for starting a VM in headless mode
   :width: 500px

   Starting the virtual machine in headless mode

2. In a web browser, navigate to `localhost:8080/geoserver <http://localhost:8080/geoserver>`_.

3. Using the username / password combo of **admin / geoserver**, log in to the GeoServer admin panel.

You will repeat these steps whenever you want to work with GeoServer.

When you are done working with GeoServer for the day, pause it by right-clicking on the VM in VirtualBox and selecting *Close > Save State*.

.. figure:: /_static/images/vm-save-state.png
   :alt: Screenshot of the VirtualBox menu for pausing a VM
   :width: 500px

   Pausing our virtual machine

GeoServer + OpenLayers Tutorial Overview
----------------------------------------

The next few lessons demonstrate how to use GeoServer and OpenLayers together to create dynamic, interactive mapping applications. It assumes that you have completed all prior lessons, including :ref:`intro-to-openlayers` and :ref:`intro-to-geoserver`.

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