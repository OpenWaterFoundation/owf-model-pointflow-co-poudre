# owf-model-pointflow-co-poudre

# Model Development
The Open Water Foundation has developed a point flow model for the Cache la Poudre River downstream of the Poudre Canyon (USGS gage 06752000)
to its confluence with the South Platte River (USGS gage 06752500).  The overall objective has been to determine if a point flow model approach
can be developed in a general way in TSTool (and/or other software) so that it can be applied to any river basin with minimal changes to the workflow.
The development of a point flow model is comprised of the following steps, which are discussed below:

1. Build the node network
2. Gather time series data
3. Build and run the point flow model
4. Visualize the results

##1. Build the Node Network

A point flow analysis relies on a "node network" representation of a stream system, in which measurement/calculation points represent discrete points
of information.  Each node is allowed to have one downstream node, which represents most stream systems where water flows downhill and may contain 
tributaries that merge at confluences.

The node network is an [Excel file] (https://github.com/OpenWaterFoundation/owf-model-pointflow-co-poudre/blob/master/data-files/Poudre_PointFlow_Dataset.xlsx) 
in which each row in the datasheet represents a node in the network.  At its most basic level, the datasheet needs to contain
the following columns of data:
*NodeID -- the location ID of the node, typically corresponding to the location ID in time series data
*NodeName -- a descriptive name for the node, which gives more detail than the NodeID
*NodeType -- describes the node behavior (e.g., whether discharge is added, subtracted, or reset at the node)
*NodeDist -- the node distance along the flow path; is used to estimate gain/loss
*NodeWeight -- can be used to estimate gain/loss; weights indicate the relative weight of the reach gain/loss to be distributed between nodes on the reach
*DownstreamNodeID -- the location ID for the next downstream node; needed to define network connectivity

Additionally, the following columns are included in the datasheet:
*RealTimeNodeID -- the location ID of the node for real-time data, which can differ from the NodeID
*RealTimeDownstreamNodeID -- the location ID for the next downstream node based on the RealTimeNodeID
*Longitude -- in decimal degrees
*Latitude -- in decimal degrees
*DataSource -- agency that provides the time series data; allows for faster processing of time series in TSTool

Currently, the node network file must be manually built.  Building the network can be simplified by using Colorado Division of Water Resources' Map Viewer.
Within Map Viewer, it is possible to draw a polygon around the river of interest and export data layers (such as diversion records) to csv files.  Many of the data 
layers contain data regarding the water source mile, which simplifies creating the node order and calculating node distances.  It is still necessary to go 
through the river by hand to make sure all active diversions, returns and stream gages are included in the network.  Future enhancements may include using 
TSTool to automate building some parts of the node network file using the csv files that can be downloaded from DWR's Map Viewer.

The node network file is also saved as a [csv file](https://github.com/OpenWaterFoundation/owf-model-pointflow-co-poudre/blob/master/data-files/Poudre_PointFlow_Dataset.csv).  
This file is used for some of the visualizations.

##2.  Gather Time Series Data

Diversion, return and streamflow time series data are downloaded via the [CDSS TSTool Software](http://cdss.state.co.us/software/Pages/TSTool.aspx), which automates
the data collection process.  TSTool provides features to access time series, tables and spatial data in databases, files and from internet web services.  For the 
point flow model, the HydroBase datastore is used (ColoradoWaterHBGuest).  The command file `Poudre_Nodes_Data_Download.TSTool` downloads daily and hourly time series
data for all nodes that contain data.  For daily data, the default period of record is 2000-2016, but this can be changed by the user.  For hourly data, the period of 
record is the previous seven days up to the most current measurement.  

Several diversions and returns do not have actively-maintained records.  In these instances, OWF creates blank time series for those nodes, which are then typically
filled with zeroes to indicate a lack of data.  OWF feels it is necessary to have these nodes within the node network because they are known locations where water is
removed from or put into the river, even if no data are currently associated with the nodes.

Time series data are then saved as `.dv` files which are read into other TSTool command files.

##3.  Build and Run the Point Flow Model

Point flow models use relatively simple water balance calculations to account for diversions and returns (or inflows) between known flow points, typically at stream gages.
Error calculated in the water balance in a reach is attributed to a gain or loss in the reach and is typically distributed back to each location (node) via a stream mile 
proration or other approach.

Building and running the point flow model is done through the TSTool software.  The command files `Poudre_Daily_PointFlowModel.TSTool` and `Poudre_Hourly_PointFlowModel.TSTool`
contain the steps necessary to run the daily and hourly point flow models, respectively.  The `.dv` files created in the previous step are read into the file.  
For the daily point flow model, any missing data for diversions and returns are filled with zeroes so that the model can run.  Missing stream gage data are filled by 
repeating nearby values; this is adequate for small gaps in the data but needs to be further evaluated for larger gaps.  For the hourly point flow model, missing data
 for diversions, returns and stream gages are filled by repeating nearby values and thus far appears adequate for this smaller time step.
 
The point flow model is then run using the AnalyzeNetworkPointFlow() command in TSTool.  The command uses the time series data and the node network described in step 1.
The network is analyzed from the most-upstream node to the most-downstream node and all timesteps are processed for a node before moving on to the next node.
Several output time series are generated for each node; the primary time series of interest is NodeOutflow, which is essentially natural flow.

The NodeOutflow time series are then written to a table, which is then exported as a `.csv` file to be used in visualizations.

** The AnalyzeNetworkPointFlow() command currently does not handle branching networks (tributaries) in calculations.  It also does not handle reservoirs and storage
calculations.  These limitations will be addressed with future enhancements to TSTool.**

##4.  Visualize the Results

Point flow model results are currently visualized as a heat map (also called a raster graph).  Heat maps represent individual data values in a matrix as colors.  The
layout represents upstream to downstream distance on the x-axis and day (or hour) on the y-axis.  Hovering on a cell displays the discharge for that day/hour in a popup.  

The heat map is created with [Highcharts](https://www.highcharts.com/).  Highcharts is a JavaScript charting library that allows a user to add interactive charts to a
 web site or web application.  
 
For the daily point flow model, the heat map shows daily discharge at each node over a one-year period.  For this demonstration, OWF has chosen to model 2002 but 
future enhancements may allow for the selection of any year from 2000 to 2016.

# Repository Contents

The contents of this repository are a work in progress and reflect ongoing enhancements to the point flow model.  Specific folders in the repository include 
the following:

* [data-files](https://github.com/OpenWaterFoundation/owf-model-pointflow-co-poudre/tree/master/data-files) -- the node network data files in both `.xlsx` and `.csv` format
* [javascript](https://github.com/OpenWaterFoundation/owf-model-pointflow-co-poudre/tree/master/javascript) -- files related to the development of visualizations
* [TSTool](https://github.com/OpenWaterFoundation/owf-model-pointflow-co-poudre/tree/master/TSTool) -- TSTool command files that download time series data and build and run the point flow models
* [visualizations](https://github.com/OpenWaterFoundation/owf-model-pointflow-co-poudre/tree/master/visualizations/website) -- files that contain visualizations of point flow model results 

# How to Run the Model and View Results

The following are the basic instructions for downloading the point flow model files from the repository
and running the model.  Replace `someUser` with the appropriate user name on the computer being used.

## 1. Install Git Client Software (Optional)

If you intend to run the point flow model but don't expect to contribute changes to the maintainers of the point flow model repository,
you can download the repository as a zip file and don't need to install Git client software.

If you intend to modify the point flow model and track changes using Git version control software,
you must install Git client software on the computer.
To install, [download Git for Windows](https://git-scm.com/download/win) (**this will start the download - for more options [click here](https://git-scm.com/download)**).
Git for Windows will install Git Bash command line Git interface and Git GUI graphical user interface.
It is also assumed that you have a basic understanding of how to use Git.  Git commands are shown below for cloning the repository.

## 2. Create a top-level folder structure to house the repository files

The point flow model is part of a system and it is useful to install the files in a folder
structure that will accommodate other related downloads.
For example, create the folder structure:  `C:\Users\someUser\model-pointflow-co-poudre\git-repos`.
This naming convention reflects the fact that work is related to the Poudre point flow model and
a Git repository is being located in the folder.

## 3. Download repository files

If you intend to run the point flow model but don't expect to contribute changes to the maintainers of the point flow model repository,
you can download the repository as a zip file and don't need to install Git client software.
To download a zip file use the ***Clone or download*** button on GitHub and then ***Download ZIP***, and then unzip in the folder created above.

If you intend to modify the point flow model, track changes using Git version control software,
and possibly submit those changes to the maintainers of the point flow model, use the Git software to clone the repository.
For example, if using the Git Bash command line software on Windows do the following (the URL for the repository can be copied
from the ***Clone or download*** button on GitHub):

```text
C:
cd \Users\someUser\model-pointflow-co-poudre\git-repos
git clone https://github.com/OpenWaterFoundation/owf-model-pointflow-co-poudre.git

```

The repository is configured with [`.gitignore` files](https://git-scm.com/docs/gitignore) so that
dynamic files created when running the model are NOT saved to the repository.
Consequently, the point flow model must be run to see the results and the associated heat map.

## 4. Download the most current version of TSTool

Download and install the latest TSTool version from the [Open Water Foundation TSTool page](http://openwaterfoundation.org/software-tools/tstool).
The point flow model has been tested with TSTool 12.00.00

## 5. Run the point flow model

To run either the daily or hourly point flow models, first start TSTool and open the
 [Poudre_Nodes_Data_Download.TSTool](https://github.com/OpenWaterFoundation/owf-model-pointflow-co-poudre/blob/master/TSTool/Poudre_Nodes_Data_Download.TSTool)
 command file.  Click the "Run All Commands" button in the middle of the screen; the file will take about 5 minutes to complete.  Next, open the
 [Poudre_Daily_PointFlowModel.TSTool]
 (https://github.com/OpenWaterFoundation/owf-model-pointflow-co-poudre/blob/master/TSTool/Poudre_Daily_PointFlowModel.TSTool)
 command file or the [Poudre_Hourly_PointFlowModel.TSTool]
 (https://github.com/OpenWaterFoundation/owf-model-pointflow-co-poudre/blob/master/TSTool/Poudre_Hourly_PointFlowModel.TSTool)
 command file depending on the time interval of interest.  Click the "Run All Commands" button and the file will complete shortly.

## 6. View the point flow model heat map in a browser

Heat maps are contained in the [visualizations](https://github.com/OpenWaterFoundation/owf-model-pointflow-co-poudre/tree/master/visualizations/website) folder.
Each heat map is within its own folder and is titled `index.html`.  Opening the html file requires a few extra steps.  The following procedure assumes the user
is using Google Chrome as their web browser. 

Open a new tab in Chrome and select "Apps" in the upper lefthand corner of the screen.  Select "Web Store" and search for an app titled "Web Server for Chrome".
It should have a yellow circle icon that says "200 OK!".  Download the app.  Now open a new tab, click on "Apps" and the Web Server app should appear.  Click on
the app and click on the "CHOOSE FOLDER" button.  Navigate to the "owf-model-pointflow-co-poudre" folder.  Then click on the available Web Server URL.  A new web page
will open with the title "Index of current directory..."  Click on the `visualizations` folder, then the `website` folder, then either the daily or hourly streamflow
folder.  Click on `index.html`.  The heat map should now be visible.  Hovering on a cell displays the discharge for that day/hour in a popup. 



