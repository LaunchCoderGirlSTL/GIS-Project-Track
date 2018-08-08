Publishing Data In GeoServer
============================

Obtaining the Data
------------------

Visit the `City of St. Louis Open
Data <https://www.stlouis-mo.gov/data/>`__ project page and navigate to
the "Urban Development" section. Once there, look for "Park Data" and
click through. From this page, download the "Parks Shapefile." The
download is a ``.zip`` file which contains the ``.shp`` file along with
its sidecar files. Don’t unzip the downloaded file just yet.

Staging the Shapefile
^^^^^^^^^^^^^^^^^^^^^

At this point, make sure your GeoServer VM is up and running in
VirtualBox.

For GeoServer to be able to use the shapefile, it must be placed within
GeoServer’s data directory. Where is this directory? Our GeoServer VM is
helpful enough to tell us when we log into it:

::

   $ ssh -p 2020 training@localhost
   Last login: Tue Mar 13 19:33:00 2018 from 10.0.2.2
   ======================================================================
   Welcome! This virtual machine is designed for use with the following
   Boundless course: **Boundless Suite: GeoServer I**

   Useful commands:
   sudo service tomcat8 start (start Tomcat)
   sudo service tomcat8 stop  (stop Tomcat)
   sudo poweroff              (shut down the virtual machine)

   Useful directories:
   /opt/boundless/suite/geoserver          (GeoServer install directory)
   /var/opt/boundless/suite/geoserver/data (GeoServer data directory)
   /media/sf_share                         (share directory between host and guest)

   For more information: http://boundlessgeo.com
   ======================================================================
   training@boundless:~$

Note near the bottom of the login message that the data directory is
``/var/opt/boundless/suite/geoserver/data``. This directory lives within
the virtual machine, so we first need to place the ``.zip`` file on the
VM before we can move it to the data directory.

The welcome message also tells us that ``/media/sf_share`` is the
directory on the VM that is shared with our host operating system. To
see which directory on the host system is mapped to this directory on
the VM, go to the VirtualBox application, right-click on the *Boundless
GeoServer I* VM, and select *Settings*. Select the *Shared Folders* pane
and you’ll see the the directory on the host that is mapped to shared
directory on the guest (i.e. the VM). In the example below, it is
``/Users/chris``.

.. figure:: /_static/images/shared.png
   :alt: VirtualBox settings pane for configuring folders to share between the host and guest operating systems

   Shared folder configuration

Place the ``parks.zip`` file in this directory. Then SSH into
the VM using ``ssh -p 2020 training@localhost``. Run
``ls /media/sf_share/`` and make sure you see ``parks.zip`` listed.

Now, move the file to the GeoServer data directory. Note that we need to
prepend the commands with sudo since our VM user doesn’t have permission
to write to the destination directory. Also recall that the password on
the VM is empty (i.e. just press Enter when prompted for a password).

::

   training@boundless:~$ sudo mv /media/sf_share/parks.zip /var/opt/boundless/suite/geoserver/data

Then go to this directory and unzip the file.

::

   training@boundless:~$ cd /var/opt/boundless/suite/geoserver/data
   training@boundless:/var/opt/boundless/suite/geoserver/data$ sudo unzip parks.zip -d stl-parks
   Archive:  parks.zip
     inflating: stl-parks/parks.dbf   
     inflating: stl-parks/parks.prj   
     inflating: stl-parks/parks.sbn   
     inflating: stl-parks/parks.sbx   
     inflating: stl-parks/parks.shp   
     inflating: stl-parks/parks.shp.xml  
     inflating: stl-parks/parks.shx

In the next section, we will configure GeoServer to use this data.

Creating a Data Store
---------------------

Log into the GeoServer admin portal by visiting
http://localhost:8080/geoserver/ in your browser and entering the
credentials admin/geoserver.

Creating a Workspace
^^^^^^^^^^^^^^^^^^^^

Recall that in GeoServer, every data store belongs to a workspace.
Create a new workspace by selecting *Data > Workspaces* from the menu at
left. Select *Add new workspace*. Name the workspace **lc** and give it
the URI **http://launchcode.org**.


.. figure:: /_static/images/workspace.png
   :alt: GeoServer page for creating a new workspace 

   Create a new workspace

Creating the Store
^^^^^^^^^^^^^^^^^^

Visit *Data > Stores* and select *Add store*. Select *Shapefile* from
the *Vector Data Sources* list. Fill out the required fields, and then
browse to the *.shp* file that you previously placed in *stl-parks*
subdirectory of GeoServer’s data directory.


.. figure:: /_static/images/data-source.png
   :alt: GeoServer page for creating a new vector data source

   Create a new data source

Upon saving the store, you will immediately be presented with
the option of publishing a new layer named *parks*. Let’s try to do
this.

After clicking on the *Publish* link, you will see the **Edit Layer**
form. Scroll down to the **Coordinate Reference Systems** section. This
is where we configure the native and published SRS for the layer. Notice
that the **Native SRS** field contains UNKNOWN, with the link at right
showing an SRS named "NAD_1983_StatePlane_Missouri_East_FIPS_2401_Feet…"
This SRS is the spatial reference system declared within the shapefile
itself, but if we try to find it within GeoServer, we will fail.

Click on *Find* next to the **Declared SRS**\ field and search for
"1983". The only result returned is *not* the SRS described by the file.
In the next section, we’ll fix this issue by adding the SRS to GeoServer
and finally publishing the layer.

Publishing a Layer
------------------

In the previous section, we created a new workspace and data store from
a shapefile, and discovered that the geo data within the shapefile used
an SRS that was unknown to our GeoServer. In order for the data to be
properly served by GeoServer, a layer must be configured with the
correct SRS. Fortunately, we can add new SRS configurations to
GeoServer.

Adding a New SRS
^^^^^^^^^^^^^^^^

We will add a new SRS definition to GeoServer following the
`instructions in the GeoServer
documentation <http://docs.geoserver.org/stable/en/user/configuration/crshandling/customcrs.html>`__.

If you don’t have a terminal open and logged in to your GeoServer VM, do
so now:

::

   $ ssh -p 2020 training@localhost

Recall that the password is empty.

Once logged into your GeoServer VM, go to the data directory:

::

   training@boundless:~$ cd /var/opt/boundless/suite/geoserver/data

From here, descend one more directory into ``user_projections``:

::

   training@boundless:/var/opt/boundless/suite/geoserver/data$ cd user_projections/

Open the file ``epsg.properties`` for editing in the text editor nano:

::

   training@boundless:/var/opt/boundless/suite/geoserver/data/user_projections$ sudo nano epsg.properties

Use the down arrow key to navigate to the end of the file. We’ll add the
new SRS definition here. We need to grab the SRS defintion, so let’s
head over to `epsg.io <http://epsg.io/>`__, a site that is a reference
for coordinate system definitions.

Search for "1983 missouri 2401" and the first result you see should be
the coordinate system we’re looking for, "NAD 1983 StatePlane Missouri
East FIPS 2401 Feet." Select this result to see the full details. Select
and copy the GeoServer version of the coordinate system.

Back in your open text editor, on a new line at the end of the file,
paste in the definition. Then use ``ctrl+X`` to quit nano, select "Yes"
when asked if you want to save the file.

To finish up, we need to restart GeoServer:

::

   $ sudo service tomcat8 stop
   [sudo] password for training: 
   Stopping tomcat8:  * 
   $ sudo service tomcat8 start
   Starting tomcat8:  * 

Verify that the new SRS was found by GeoServer by selecting *Demos* from
the left-hand menu, and then choosing *SRS List*. Search for the
coordinate system by number, 102696. If it isn’t, retrace your steps
from the instructions above.

Publishing the Layer
^^^^^^^^^^^^^^^^^^^^

We can now properly publish the layer. Go to *Data > Layers* and select
*Add a new layer*. Choose the data store you created for the parks data,
and click on *Publish* next to the ``parks`` layer.

The only settings on the Edit Layer page that need to be changed are in
the **Coordinate Reference Systems** and **Bounding Box** sections.
Update these to match the displayed settings below, finding the
**Declared SRS** using the Find dialog and searching for "102696." The
**Bounding Box** fields are populated by simply clicking on the
highlighted links.


.. figure:: /_static/images/crs-config.png
   :alt: Page for configuring the new layer's properties

   Configuring the coordinate reference system and bounding box

Save the new layer, and give yourself a high five for publishing your
first data set in GeoServer!

Previewing the Layer
^^^^^^^^^^^^^^^^^^^^

Let’s make sure everything looks good with the layer before moving on.
Navigate to *Data > Layer Preview* and choose the *OpenLayers* option
for the newly-created layer. A new tab will open with the geometries in
the given layer displayed in a small box. Without a base layer to give
them context, they’ll look a bit funny, but you should recocnize some
familiar city parks. If your preview doesn’t look like this, then edit
the layer and make sure you have the CRS and Bounding Box settings
correct.


.. figure:: /_static/images/layer-preview.png
   :alt: Preview image of the parks data layer
   :width: 500px

   Park data layer preview

