java c
GGR273 Lab 2: Environmental Conservation
Lab 2: Environmental Conservation - Creating a Fish   Stocking Web   Map
Due: Aug 7th   2024 @   11:59 pm EST through the   Quizzes tab
Submit through Lab 2 and answer questions and   submit   any   links   and   files
Objective: Create a web map using open-source tools and data to   display   waterbodies   stocked with fish by the Alberta public conservation authority. The final web map must   include   species,   number stocked, and date of   stocking. Your final web map will be hosted   on   Github.
Step   1: Setup Environment
1.       Open QGIS   and   Create a   New   Python   Workbook:
o      Open   QGIS, goto   Plugins   >
Python Console >   Show Editor   to create   a new Python workbook.
o    The   Show   Editor button   looks
like a little notepad with   a pencil.
o      Save your workbook into the
local folder you have created   for   this lab. It will   save   as   a   .py   file.
o    As you work through this   lab   and enter script,   save   this
workbook.
o    This is different from the python   console - which   is more   like   a   live   chat.   In the console you can quickly execute commands. The workbook is   like   a   journal.   You   can   keep   track   of   your   workflow   this   way.
o    When you run lines of   code in the workbook you will   see the   output   in the
console. You can also test it in the console before you   add   it to your   workbook.
o    Start by adding annotation   at the top   (start with   #)   and   at   the   top   enter   your   name,   the date, the course code, the   assignment details.

2.       Setup Python Environment:
import   os
from qgis.core import QgsProject, QgsVectorLayer,   QgsRasterLayer
#   Set   up   logging import   logging
logging.basicConfig(level=logging.INFO)
logging.info("Project   loaded   successfully")
#Define your working directory. This   is   the   file   path   on   your   local      #computer that you have made   for   this project.   Ensure   you   change   any   #backslashes to   forward   slashes
Working_directory   = 'C:/path/to/your/directory'
#You can also create   a   working   directory   if   yours   doesn’t   exist.   if   not   os.path.exists(working_directory):
os.makedirs(working_directory)
# Set   the   project   file   path
#We   are   saving   the   .qgz   file   as   GGR273   Lab   2,   you   can   change   the   file #name to anything you   want   and   it   will   save   the   file   to   your   folder
project_path   = os.path.join(working_directory,   'GGR273_Lab2   .qgz')
# Save the   QGIS   project   to   the   specified   path   project   = QgsProject.instance()
project.write(project_path)
# Set the working   directory   as   the   default   data   source   location   os.chdir(working_directory)
# Verify the   current working   directory
print(f"Current   working   directory:   {os.getcwd()}")
# Create   folders   for   temp   files temp_dir   =   'path/to/temp'
if   not   os.path.exists(temp_dir):   os.makedirs(temp_dir)
Step 2: Import and   Reproject Data
Download the geospatial data as shapefiles and the   stocking report   as   a pdf   from the   following   sources. Ensure they are saved in your working directory:
1.       Water Bodies Data:
o      URL:https://open.canada.ca/data/en/dataset/448ec403-6635-456b-8ced-   d3ac24143add/resource/5610a875-6022-4627-ab71-c3829404c537
2.       Provincial Boundary   File:
o      URL:https://www12.statcan.gc.ca/census-recensement/2021/geo/sip-   pis/boundary-limites/index2021-eng.cfm?year=21
•         Download the cartography file for provinces as a   shapefile
3.       Trout Planned Stocked Dates   2024   .csv:
o    Part of   Quercus lab data to download:   epa-fish-stocking-planned-dates- 2024_Lab2Data.csv
Additional Required Data:
Additional data will be added through plugins or created through the lab tutorial   steps.
•             Basemap: An open-source web map service such   as OpenStreetMap.
•             Fish Stocking Data: Extracted from the PDF   and   converted to   CSV.
•             Attribute Data for Water Bodies: To include specific   information   about   each water   body from various   sources.
These data sources will be used to complete the lab and   create   the   final   web   map.
1.       Unzip shapefiles and   Import   Data:
import   zipfile
# Unzip all   .zip   files   in   the working   directory
for file_name in   os.listdir(working_directory):   if file_name.endswith('.zip'):
with zipfile.ZipFile(os.path.join(working_directory, file_name),   'r') as zip_ref:
zip_ref.extractall(working_directory)   print(f"Extracted:   {file_name}")
# Walk through the directory   to   find   all   shapefiles   and   #import   them   into   QGIS   for root, dirs,   files   in   os.walk(working_directory):
for   file   in   files:
if   file.endswith('.shp'):
# Construct   the   full   file path
file_path   = os.path.join(root,   file)   # Create   a   vector   layer
layer   = QgsVectorLayer(file_path, os.path.basename(file_path),   'ogr')
# Check   if   the   layer   is   valid   if   not   layer.isValid():
print(f"Failed   to   load   {file_path}")   else:
# Add the   layer   to   the   project
QgsProject.instance()   .addMapLayer(layer)   print(f"Loaded   {file_path}")

2.       Reproject Data:
o    Ensure all layers use the   same   CRS   EPSG:3403).
EPSG:3403 refers to   "NAD83 /   Alberta   10-TM (Forest)". This Coordinate
Reference System (CRS) is used in   Alberta, Canada,   for forestry purposes.   It   is   based on the NAD83 datum and uses a Transverse Mercator projection.
from qgis.core import QgsProject, QgsCoordinateReferenceSystem,
QgsProcessingFeatureSourceDefinition, QgsFeatureRequest, QgsVectorFileWriter
# Define   the   target   CRS   (EPSG:3403   -   NAD83   /   Alberta   10-TM   (Forest))
target_crs   = QgsCoordinateReferenceSystem('EPSG:3403')
#set the   entire project   CRS
project.setCrs(target_crs)
# Get the   list   of   layers   in   the   current   project
layers   = QgsProject.instance().mapLayers().values()
# Iterate   over   each   layer   and   reproject   for   layer   in   layers:
# Define the   parameters   for   reprojection   params   =   {
'INPUT': QgsProcessingFeatureSourceDefinition(layer.id(),   selectedFeaturesOnly=False, featureLimit=-1,
geometryCheck=QgsFeatureRequest.GeometryAbortOnInvalid),
'TARGET_CRS': target_crs,
'OUTPUT':   'memory:'    #   Temporary   layer }
# Run the   reprojection   algorithm
result   = processing.run('native:reprojectlayer', params)
# Get   the   reprojected   layer
reprojected_layer   = result['OUTPUT']
# Set   the new   name   for   the   reprojected   layer
new_layer_name   = layer.name()   +   "_reprojected"   reprojected_layer.setName(new_layer_name)
# Add the   reprojected   layer   to   the project
QgsProject.instance().addMapLayer(reprojected_layer)
# Remove the   original   layer   from   the project
QgsProject.instance().removeMapLayer(layer.id())
print(f"Reprojected   {layer.name()} to   {target_crs.authid()} and added   as   {new_layer_name}")
Step 3: Update .csv of fish stocking   data
1.       Normalize Species Names:
o    Convert   species acronyms   to   full   names
# Define the working directory and   file paths
#set   the   filename   to   the   saved   .csv   file   from   Quercus   of   fish   stocking working_directory = 'C:/path/to/your/directory'    # Update this path
csv_file_path = os.path.join(working_directory, 'your_file.csv')
output_csv_file_path = os.path.join(working_directory, 'output_file.csv')
# Read the   CSV   file
df   = pd.read_csv(csv_file_path)
# Define the mapping from acronyms to   full names
species_mapping =   {
'RNTR': 'Rainbow Trout',   'BNTR': 'Brown Trout',
'BKTR': 'Brook Trout',
'CTTR': 'Cutthroat Trout',   'TGTR': 'Tiger Trout'
}
# Convert acronyms to   full names
df['Species'] = df['Species'].map(species_mapping)   #   Save the updated DataFrame. to a new CSV   file            df.to_csv(output_csv_file_path, index=False)
# Drop columns with   "Unnamed" in their name
df   = df.loc[:, ~df.columns.str.contains('^Unnamed')] #   Save the updated DataFrame. to a new CSV   file
df.to_csv(output_csv_file_path, index=False)
# Print the data types of   each column and the first   three   rows   to   verify   the   data
print(df.dtypes)         print(df.head(3))
Step 4: Data Preparation
1.       Select Alberta Boundaries:
First, select the Alberta boundaries from the all-Canada provinces and territories file   and   export it, then import it back into the   QGIS project.
import   os
from qgis.core   import
(QgsProject,QgsVectorLayer,QgsVectorFileWriter,QgsFeatureRequest,QgsFie   ld)
# Define the   working   directory   and   file   paths
working_directory   =   'C:/path/to/your/directory'    # Update   this   path
shapefile_path   = os.path.join(working_directory,   'Provinces_reprojected.shp')
output_shapefile_path   = os.path.join(working_directory,   'AlbertaBoundary.shp')
# Load   the   shapefile
layer   = QgsVectorLayer(shapefile_path,   'Provinces',   'ogr')   if   not   layer.isValid():
print("Layer   failed   to   load!")
# Select   features   where   PRNAME   is   'Alberta'
expression   =   '"PRNAME" =   \   'Alberta\   ''
request = 代 写GGR273 Lab 2: Environmental Conservation - Creating a Fish Stocking Web MapPython
代做程序编程语言QgsFeatureRequest().setFilterExpression(expression)
alberta_features   =   [feat   for   feat   in   layer.getFeatures(request)]
# Check   if   any   features were   selected   if   len(alberta_features)   ==   0:
print("No   features   found   with   PRNAME   'Alberta'")
else:
# Create a   new   layer   to   store   the   selected   features   writer = QgsVectorFileWriter(
output_shapefile_path,   'UTF-8', layer.fields(),
layer.wkbType(), layer.sourceCrs(),   'ESRI   Shapefile'   )
# Add selected   features   to   the   new   layer
for feat   in   alberta_features:   writer.addFeature(feat)
# Delete the   writer   to   flush   and   close   the   file   del writer
# Load   the   new   shapefile   into   the   QGIS project
alberta_layer   =
QgsVectorLayer(output_shapefile_path,   'Alberta   Boundary',   'ogr')
if   not   alberta_layer.isValid():
print("Layer   failed   to   load!")   else:
QgsProject.instance().addMapLayer(alberta_layer)   print("Layer   loaded   successfully!")
   
2.       Reduce Waterbodies to Provincial Boundary
a.       Test out   code   options   to   limit   the waterbodies   in Canada to only the water bodies in Alberta
b.       Record your code to submit   later
3. Geocode the fish   stocking   data
from   shapely.geometry
import geopandas   as   gpd
import Point from qgis.core   import   QgsVectorLayer,   QgsProject
# Define the   working   directory   and   file   paths
working_directory   =   'C:/path/to/your/directory'    # Update   this   path
   
csv_file_path   = os.path.join(working_directory,   'epa-   fish-stocking-planned-dates-2024_Lab2Data.csv')
output_shapefile_path   = os.path.join(working_directory,   'fish_stocking_points.shp')
# Read   the   CSV   file
df   = pd.read_csv(csv_file_path)
# Remove   any   columns   named   "Unnamed"
df   = df.loc[:, ~df.columns.str.contains('^Unnamed')]   # Create a   GeoDataFrame   from   the   CSV   file   using   the
Latitude and   Longitude   columns   gdf   = gpd.GeoDataFrame(df,
geometry=gpd.points_from_xy(df['Longitude'],   df['Latitude']))
# Set   the   initial   CRS   to   WGS   84   (EPSG:4326)   gdf.set_crs(epsg=4326, inplace=True)
# Reproject   to   UTM   Zone   10N   (EPSG:32610)   gdf   = gdf.to_crs(epsg=32610)
# Save   the   GeoDataFrame   to   a   shapefile
gdf.to_file(output_shapefile_path, driver='ESRI   Shapefile')   # Load the   new   shapefile   into   the   QGIS   project
fish_stocking_layer   = QgsVectorLayer(output_shapefile_path,   'Fish   Stocking Points',   'ogr')
if   not   fish_stocking_layer.isValid():
print("Layer   failed   to   load!")   else:
QgsProject.instance().addMapLayer(fish_stocking_layer)   print("Layer   loaded   successfully!")
Visually test your geocoding by selecting a fish stocking   location   and   compare   it   to   the water
bodies   features. Look   of   the   name   of   the   lake   across   each   file   and   ensure   that   the   names   of   lakes are the   same.
Step 6: Update Fish Stocking   Variables
1. Update the Fish   Stocking Planned dates data so that   every record/row has   a   Map
Author field. Populate this field with your first and lastname   as   a text   field.   This way,   for any location in your final map, your name will be attached to the data.   Keep   the python code you used to complete this for your final submission.
Step 7: Create   Web   Map
Customize Web Map in QGIS:
o    Use QGIS2Web or similar plugin   for web map   creation.
1.       Install qgis2web   Plugin:
•             Go   to   Plugins   > Manage   and   Install   Plugins...
•             Search   for   qgis2web   and   install   it.
2.       Update the symbology for   all   layers
o    Ensure fish stocking uses graduated   symbols   for number of   fish.   Consider the
skew of   data and choose the appropriate classification   scheme and   colour   scheme.
1.       You can update the number of   fish stocked field   in   python
2.       To remove the comma from   the number use the following expression   in   the field   calculator:
CASE
WHEN "STOCKING N" IS NULL OR " STOCKING N " = '-' OR   "   STOCKING N " = 'TBD' THEN NULL
ELSE to_real(replace("STOCKING N ", ',', ''))
END
   
3.       Create   web map
o    Go to Web   >   qgis2web   >   Create   a web   map.
o    Select   the   layers   you   want   to   include
o      You   only   need   popups   on   the   fish stocking report data
o      Identify   the   fields   you   want   to   have   as   popups
1.       Make sure your Fish   Stocking - Map Author field is visible   as   a header
2.       For the rest – decide what you think makes sense, but don’t have all   as   headers because its too cluttered
•         Hidden Field: The field is not displayed   in the pop-up.
•         Inline: The field is displayed directly within the pop-up without additional   formatting.
•         Header: The field is displayed as a header within   the pop-up.
•         Image: The field value is treated as a URL to   an   image,   displaying the   image   in   the   pop-up.
•         Link: The field value is treated as a URL, displaying   it   as   a   clickable   link   in   the   pop-   up.
•         Chart: Displays the field values as a   chart in   the   pop-up.
o    Go through each tab and update the   information: Appearance, Export,   Settings   and WIKI
o    Make sure you are choosing   options that   have your reader   focus   on   identifying   water   bodies   in   Alberta, and   then   finding   the   number   of   fish   and   species   of
fish that are stocked there.   Any other information is   less important   or   not   important at all. The Map Author field should be   highlighted and visible.
o    When youre happy with your settings,   select   Export
4.       Locate and View   Web Map
o    Find   the   exported   files
1.       The exported files will   be saved wherever you   exported them to
2.       They will be in the working directory if   you didn’t   save them to   a   different   place
5.       Create a GitHub   account   and   repository
o    The web map needs a website to host   and view the   web map
o    There are   free websites where you   can   do   this
o    We are going to use GitHub because it has   free   web   hosting,   version   control   for   files, project management, accessibility, open source, and integrates with many different tools. This is also useful to add to as you   create more projects   and you   can send as a link to potential employers   or link   on   your   cover   letters   for   job interviews. Just make sure everything actually works!
a)       Create GitHub Repository: Create a new repository on   GitHub.
o   Go   to   https://github.com/
o   Click on   "Sign up" in the upper-right corner.
o   Enter your email address and click   "Continue".
o   Create a password and continue.
o   Choose a username and continue.
o   Follow the remaining steps to complete the sign-up process.
b)      Upload Files: Upload the exported web map files to the repository.
o    Log   in to your   GitHub   account.
o    Click on the + icon in the upper-right   corner   and   select New   repository.
o      Enter   a   repository   name   (e.g., Alberta-Fish-Stocking-2024).
o      Click Create repository.
o    In your repository, click on   Add   file   > Upload   files.
o    Drag and drop the exported web map   files   into   the   repository.
o    Commit   the uploaded   files.
c)       Enable GitHub Pages: Enable GitHub Pages in the repository   settings to host your web   map.
o   Go to   Settings in your repository.
o   Scroll down to the GitHub Pages   section.
o   Under Source, select the branch (usually main) and the /root   directory.
o   Save the   settings.
o   GitHub will provide a URL for your hosted web map, typically   https://username.github.io/repository-name/.
a)      Share URL: GitHub provides a URL that you can share to view   the   web   map.
Ensure this works by sending to a friend and opening   on   different   devices   like   your
phone. You will be submitting this link for your Lab 2 assignment. Be   sure to   test the pop   ups.
Step 8:   Answering Questions (20 points/20%): Answer the   following questions   based on   your analysis   in the Quizzes tab for Lab 2   by Aug   7th   at   11:59pm   EST:
1.   10%   Submit your URL link on GitHub that displays your webmap.
2. 4% Submit your QGIS workbook   file
3. 2% Enter the lines of   code, with annotation and all   of   your pathways, that   you used   to   reduce the waterbodies file to the alberta provincial boundaries
4. 2% Enter the lines of   code, with annotation and all   of   your pathways, that   you used   to   update       the Fish Stocking Planned dates data so   that every record/row has a Map   Author   field. Populate   this      field   with   your   first and last name as a   text   field. Keep   the python code   you used to complete   this   for your   final submission.
5. 1%   Fill in the blank: The command to remove commas from a   field value   in   QGIS   is                                                    
6.   1% True/False: The   expression   CASE   WHEN   "FIELD" IS   NULL   THEN   0   ELSE   "FIELD"   END   can be used to handle null values in QGIS.







         
加QQ：99515681  WX：codinghelp  Email: 99515681@qq.com
