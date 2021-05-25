
****************************************
Data Viewer Tutorial
****************************************

    :keywords: Tutorial, Documentation

Author: Juna Kim

Introduction
============

This documentation details the funtionality and methods used to create a generic glue viewer. This documentation is also meant to help new Glue developers become aquainted with creating a new viewer and hope to be a resource for kickstarting their own code. Most of the information on this document can be found on the official Glue documentation page, and additional information may be found there. However, this file will be more streamlined to help developers create viewers compared to the original documentation.

Contents
=======================

This section describes the structure of documentation and its files.

#. Making the Data Viewer Skeleton
#. Modifying the Toolbar
#. Accessing the Data Collection
#. Listening for Changes with Messages
#. Plot Layers
#. Qt Design
#. Creating the Plugin (and how to update it easily)
#. Common bugs and how to fix them



Making the Data Viewer Skeleton
==========================

When starting to learn how to use Glue, the many technical terms might seem confusing. 

* DataViewer -  The default class used in making a Data Viewer (2d plot, Histogram, TableViewer, etc)
* LayerArtist - A class to draw layers of viewer, including logic about converting data/subsets into layers
* ViewerState - A class for the conceptual state of the viewer: what to show rather than how to show it
* LayerState - The state of a visualization layer: what data/subset is attached, and styling/display state


When creating a new data viewer, this `template file <https://github.com/jk31768/glue-statistics/blob/master/dataviewertemplate.py>`_ can help: 


Modifying the Toolbar
=======================
See link `Toolbar <http://docs.glueviz.org/en/stable/customizing_guide/toolbar.html>`_ for more official documentation

Accessing the Data Collection
=======================
To access the data collection of the current glue session, call the following variable:  

.. code-block:: python

    self.session.data_collection

This will allow you to directly access the data that is in Glue (the top left panel is a visual of what the data collection is).

**WARNING**: Do not use the variable "dc" in any of your code for the data viewer. The variable "dc" is already being used in the IPython Terminal as a default variable name for the data collection. If you use the variable "dc", then the IPython Terminal will break and not be able to call "dc". As a result, use a different variable name to assign the data collection. e.g

.. code-block:: python
    
    self.data_c = self.session.data_collection


Listening for Changes with Messages
=======================
A data viewer must be able to be responsive to changes in the glue environment. For example, if a dataset is added to glue, the data viewer may need to update its visual accordingly to add the newly added dataset. A viewer may also need to be updated if a dataset is delted, modified, etc. The action (dataset added, removed, modified, etc) that Glue listens for is called a ``Message``. If the a particulat action is performed, the corresponding Message is activated. With this activation, you can add more functionality to your viewer so it can update accordingly. 

link: `Full List of Messages <http://docs.glueviz.org/en/stable/_modules/glue/core/message.html#Message>`_

Messages:
-----------------

Most common:

* **DataCollectionAddMessage**: Activates when a new dataset is added
* **DataCollectionDeleteMessage**: Activates when a dataset is deleted
* **DataUpdateMessage**: Activates when a dataset is finished updating
* **SubsetCreateMessage**: Activates when a new subset is created
* **SubsetDeleteMessage**: Activates when a new subset is deleted
* **EditSubsetMessage**: Activates when a new subset is being edited 
* **SubsetUpdateMessage**: Activates when a subset is finished updating

Updates/Edits to a Dataset/Subset include the name, color, and size as well!

Other Messages:

* **DataCollectionMessage**: Activates when any change to DataCollection made (add/delete/modified)
* **DataAddComponentMessage**: Activates when a component is added to the data (e.g Arithmetic Attribute button on top can add Components to data)
* **DataRemoveComponentMessage**: Activates when a component is deleted from the data
* **LayerArtistVisibilityMessage**: Activates when a plot layer check is turned on/off (see middle left panel) 
* **LayerArtistUpdatedMessage**: Activates when any part of the LayerArtist changes (creating a new viewer, new subset, new data, etc)
* **ExternallyDerivableComponentsChangedMessage**: Activates when any datasets are linked 

See the full list of `Messages <http://docs.glueviz.org/en/stable/_modules/glue/core/message.html#Message>`_

To connect a Message to a method, add the following method into the DataViewer class:

.. code-block:: python

    def register_to_hub(self, hub):
        super(StatsDataViewer, self).register_to_hub(hub)
        
        #hub.subscribe(self, "MESSAGE TO LISTEN FOR', handler = 'METHOD TO ACTIVATE WHEN MESSAGE IS ACTIVATED')
        #EXAMPLE:
        hub.subscribe(self, DataCollectionAddMessage, handler = self.newDataAddedMessage)

Replace the 'MESSAGE TO LISTEN FOR' and the 'METHOD TO ACTIVATE WHEN MESSAGE IS RECEIVED' with Messages and methods of your own as done in the example comment below. In the example, the method self.newDataAddedMessage is a method that the user has created, and not a built-in function. This method should update your viewer depending on how the viewer works. In the above example, the method newDataAddedMessage() will be activated when a new dataset is imported into Glue.

When a Message is sent to the method you "subscribed"/connected it to, the function intakes a String Message as a parameter. It is with this String Message you can determine which data/subset sent the message as well as its new values. A good starting point to gather more information about the Message is to take apart or print the String Message as done in the example:

.. code-block:: python
    
    def newDataAddedMessage(self, message):
		print(message)
		index1 = str(message).index("(label: ") + len("(label: ")
		index2 = str(message).index(")")
		newDatasetName = str(message)[index1:index2]
        
        #Now you know the new dataset name, add more code as necessary to update your viewer or get more info out of the message string
                 
It is important to note that each Messages can send unique Message String formats, so make sure that when you splice the string you are doing it properly for each message.

Plot Layers
=======================
The plot layer is the left middle panel on Glue. Here, the user can toggle which data or subsets are visible, drag them around for reordering, as well as change other attributes of the particular data. In order to make the plot layer interactive with your viewer, you must connect a method (adding a callback) that activates everytime the plot layer changes. This can be done in the following line of code that goes in the __init__ method of the DataViewer.

.. code-block:: python

    self.state.add_callback('layers', self.exampleCallbackMethod)
    
    
The method that is connected to the plot layers will need to intake a callback list. A call back list is simply a list that has a method connected to it. **what is that method**

In order to access the data in the plot layers, you can use the following to get a list of properties in the plot layers:

.. code-block:: python

	self.state.layers


Qt Design
=======================

Qt is a huge Python library allowing users to design UI. There are many widgets and views that Qt offers, such as QTreeView, QTabWidget, QTableWidget, and many more. In order to add a widget to your Data Viewer, initialize the widget of your choice and make it the central widget in the __init__ function:

.. code-block:: python

	def __init__(self, *args, **kwargs):
		
		...
		
		self.tree = QTreeView()
		self.setCentralWidget(self.tree)
		
		...


Obviously, you should do more customization than just declaring a widget before you add the widget to the viewer. One of the more helpful ways to get familiar with Qt is to go to Qt's official documentation site and browsing the methods of each class to unlock features you need displayed in the viewer. 

Pop-up messages
-----------------
Pop-up messages can be useful in issuing warnings or for information. As a resource, a quick example template is shown below:

.. code-block:: python

	def showPopUp(self):
		self.popupWindow = QMainWindow()
		self.popupWindow.resize(500,250)
		self.popupWindow.setWindowTitle("Popup")
		self.popupLabel = QLabel()
		self.popupLabel.setTextFormat(1) #set Format to 1 for HTML , else no need to do this
		self.popupLabel.setText("Text or HTML here")
		self.popupWindow.setCentralWidget(self.popupLabel)
		self.popupWindow.layout().setContentsMargins(10,10,20,20)
		self.popupWindow.setContentsMargins(10,10,20,20)
		self.popupWindow.show()
		
Usually, the pop-up messages will seem incredibly dull and boring without using HTML. So it is strongly recommended that the text format is set to 1, and use the website https://html5-editor.net/ to design your pop-up window and paste the HTML code into .setText(). 


Creating the Plugin (and how to update it easily)
=======================

`See info in official documentation <http://docs.glueviz.org/en/stable/customizing_guide/writing_plugin.html>`_

Up until now, the code was written in a file named config.py, which could simply be placed in the working directory. Before you begin converting the config.py file to a plugin, make sure to get RID of the code 

.. code-block:: python

	qt_client.add(YOURDATAVIEWER)
	
from the config.py file.


In order to transform your config.py file into a Glue plugin, follow the instructions from the official `documentation <http://docs.glueviz.org/en/stable/customizing_guide/writing_plugin.html>`_ and template on the https://github.com/glue-viz/glue-plugin-template to create a new github repository for your viewer. 

Your repository should have:

* A folder named YOURDATAVIEWER
	* The config.py file renamed as YOURDATAVIEWER.py
	* __init__.py
	* version.py
	* Any additional files you might need(.pngs, .ui , etc)
	
* setup.py
* setup.cfg
* README or other instructional files

Don't forget that you need to upload the .ui file as well as any pictures/images you used in your data viewer if it is not part of glue's icons/pictures in the folder.

The setup.cfg file is needed to allow these additional files to be downloaded as part of the plugin. See this `setup.cfg <https://github.com/jk31768/glue-statistics/blob/master/setup.cfg>`_ file on how it is written and add any additional file types as necessary.

If you are confused on what your repository should look like or what code needs to go in the setup.py or additional files, see the `Glue Statistics Viewer Repository <https://github.com/jk31768/glue-statistics>`_ for inspiration.


Accessing pictures and additional files
----------------------------------------
In order to allow the plugin to access the pictures in the repository, create instance variable of each picture in the __init__.py file:

.. code-block:: python

	EXAMPLE_LOGO = os.path.abspath(os.path.join(os.path.dirname(__file__), 'MYEXAMPLELOGO.png'))
	
Then you can import the logo by adding the following line to the top of YOURDATAVIEWER.py(previously the config.py):

.. code-block:: python

	from YOURDATAVIEWER import EXAMPLE_LOGO
	
Now you can use the variable EXAMPLE_LOGO whenever you need to put a image link. For example, if this logo was to be used in the toolbar, this could simple be done by:

.. code-block:: python

	@viewer_tool
	class ButtonOnToolbar(Tool):

		icon = EXAMPLE_LOGO  #USED THE VARIABLE HERE
		tool_id = 'Example'
		action_text = 'Example'
		tool_tip = 'Click to see Instructions'
		status_tip = 'Click to see Instructions'
		shortcut = 'I'

		def __init__(self,viewer):
			self.viewer = viewer

		def activate(self):
			self.viewer.exampleActivate()

		def close(self):
			pass


If you have followed all the steps, you should be able to test if you are able to download the plugin straight from github. 
Open your anaconda command prompt and pip install the plugin using:

.. code-block:: pip

	pip install git + link_to_your_github_repository

You can also use the pip install -e command to install the plugin in development mode to avoid reinstalling the plugin everytime you need to make an update. 


To uninstall: 

.. code-block:: pip

	pip uninstall YOURDATAVIEWER

For Anaconda users, the plugin is located in anaconda(version#)/Lib/site-packages/YOURDATAVIEWER


Updating the Plugin
-------------------
To update the plugin, you can simply update the github repository and remove and reinstall the plugin (or use pip install -e for developing mode) to see your changes. It might be easier to make changes to the config.py file in the working directory while still in development and update the github once significant progress has been made. 

Common bugs and how to fix them
=======================

The IPython Terminal does not work - Make sure you are not using any variable named "dc" as part of the DataViewer Class
Nothing shows up - Make sure that you are not accidently overriding any methods you aren't supposed to in the parent class
Glue Freezes up while running data viewer - Make sure there are no infinite loops, but Glue often freezes if your code throws an error. Fix your code and try restarting your Glue session.

