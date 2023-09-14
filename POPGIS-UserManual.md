POPGIS - Developer Guides
====================
***Improving Air Pollution Monitoring and Management of Vietnam with satellite PM2.5 observation***
***Laser Pulse Project***

# Overview

This document provides: 
- user manual for the existing POPGIS. The system is running at [https://popgis.vnu.edu.vn](https://popgis.vnu.edu.vn). 
- User manual for the existing Pipeline Tracker. The system is running at [https://pptracker.duckdns.org](https://pptracker.duckdns.org)

This application is targeted to air quality in Vietnam. All mapps, locations, administrative units (provinces, districts, cities, etc) are limited to ones in Vietnam only.

# POPGIS Web Application User Manual

## Users, Data, and Features

Data presented by POPGIS Web application are fine-grained air quality maps over Vietnam territories. The maps have _3km_ x _3km_ resolution and they are produced on a daily basis. There is a quality map for Vietnam every day. 

Target users of the application includes:

- _Normal users_: Normal users are ones who want to acqquire air quality related information in Vietnam. Normal users can be anonymous or authenticated. This is the major actor of the software 
- _Administrators_: Administrators are normal users with one additional feature, user management. The additional feature allows an administrator to support normal users in account-related features (registration, correcting user profiles, recover password) or to moderate normal users' activities. 

The following core features and/or information that the POPGIS Web Application delivers to its users includes:

1. Daily air quality maps of Vietnam
2. Air quality related data from observation stations
3. Ranking table for all provinces/cities in Vietnam in terms of air quality 
4. Time-based air quality trends for any locations in Vietnam territories. The air quality trends can be derived from 2 data sources:
    - POPGIS's daily air quality maps
    - Air quality measurements from observation stations.
5. Data analysis dashboards for every provinces/cities. This feature includes:
    - Select any time range and any province/city for data  analysis. Users can choose a single period and a single province/city at a time.
    - For each pair of _selected period_ and _province/city_, the following data visualization are shown
        -  *Province-level air quality trend*: A time-based air-quality trend (line-chart) of the selected province in the selected period
        -  *Province-level map*: A provincial map with all its districts colored by air quality levels,
        -  *District-level barchart*: A barchart showing air-quality levels of all districts in the selected province for the selected period.
        -  *District-level air quality trend*: A time-based air-quality trend (line-hart) of a selected district for the selectec period. A district can be selected by clicking on *province-level map* or on *district-level barchart*
6. User profile management:
    - User can view and edit his/her basic information (e.g.: Name, date of birth)
    - User can change his/her password.
7. User management (Administrator only):
    - Administrator can view a list of all user accounts in the system
    - Administrator can create a new user account

POPGIS Web Application has the following pages:

- `Main Page`
- `Data Analysis Page`
- `Login Page`
- `Register Page`
- `Password Recover Page`
- `User Profile Page`
- `Edit Profile Page`
- `Change Password Page`
- `User List Page` (Adminitrator only)
- `Create User Page` (Administrator only)

## POPGIS Main Page

### Components
`Main Page` of POPGIS Web Application are shown in figure 1, figure 2 and figure 3. On those figures, components of the page are numbered. Components of `Main Page` are summarised in table 1.

![Figure 1: `Main Page`](https://lh3.googleusercontent.com/drive-viewer/AFGJ81pz33txsO-qPdOQfZXqMpO5WwSUllvFAKqNKcp1qW27HAhq9Ysh8zDj2eoZmiFRzj5XNib7XpHvJnYyCLF99yWQZ37dlw=w2250-h1896)

![Figure 2: Menu block and Control Window](https://lh3.googleusercontent.com/drive-viewer/AFGJ81r103M1qhkFBbYE-K3Cu2OR6Y22573lKnn5SvUA0g2tqNijzc0pm6QHQTq1eNsQtwG1KasgtexKE3Nkxy_K68fjEeT5=w3360-h1896)

| **SeqNo** | **Component**             |                                                                                  **Description**                                                                                 |
|-----------|---------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|     1     | Base map                  | Multi-layer base map                                                                                                                                                             |
|     2     | Information Window        | A floating window on top of the base map                                                                                                                                         |
|    2.1    | Tab controls              | The information window is multi-tabbed. This component can be used to switch between those tabs. There are 2 tabs: - Ranking - Selected Location                                 |
|    2.2    | Favorite provinces/cities | Favorite provinces/cities                                                                                                                                                        |
|    2.3    | Ranking table             | A table that sort provinces in descending order for the most recent air quality index                                                                                            |
|     3     | Data layer switches       | Switches that turn on/off data layers. There are 2 data layers: (1) Satellite; and (2) Air pollution measuring stations                                                          |
|     4     | Search box                | Location search box                                                                                                                                                              |
|    4.1    | Menu activation button    | Button that helps activate/deactivate application menu (See figure 2)                                                                                                            |
|     5     | Time slider               | A time slide to change the day of observation.                                                                                                                                   |
|     6     | Map zoom controls         |                                                                                                                                                                                  |
|     7     | Menu block                | The block contain application menu. It is activated by component 4.1. Menu items of the block when user is not logged in includes: (1) Home; (2) Data analysis; (3) News; (4) About; (5) Login |
|    7.1    | Language switch           | The switch to change language                                                                                                                                                    |                                                                                                                                          |

***Table 1: Components of the `Main Page`***

### Features

#### Switching between languages

Users can switch between languages via _language switch_ (component 7.1). POPGIS currently supports 2 languages, Vietnamese (default) and English. Note that, to access the language switch, a user should activate application menu block (component 7).

#### Interacting with base map

##### Basic interactions
User can use his/her pointing device (e.g. mouse) to zoom and pan the base map. Scroll up/down actions are for zooming out/in, and drag/drop the base map are for panning. Users can also zoom in/out using component 6 (map zoom controls). 

Note that, there are constrains on zooming and panning the base map. 

- Zoom levels are bounded between min and max values
- Vietnam will be kept in the view port at the minimum zoom level

##### Toggling data layers

Users can click on data layer switches for turning on/off a data layer. There are two data layers:

- Satellite: This layer contains air quality maps calculated by the software (POPGIS processing pipeline).
- Air pollution measuring station: This layer contains sampled air quality measurements from air quality stations. 

Satellite layers contains maps for Vietnam only. When this layer is on, a semi-transparent colored layer covers Vietnam's teritories. The layer is colored according to the color scale show at the bottom right of the `Main Page`.

When "Air pollution measuring station" layer is on, there are colored dots (circles) representing air monitoring stations appeared on the map as shown in figure 3

![Figure 3: Air monitoring stations shown on the base map](https://lh3.googleusercontent.com/drive-viewer/AFGJ81qKcNFyxzpMqxzDjuBD7jhe0xu7PeGTV_zj7_pfrycg-rcGy1S5LtoOC5u-b4ywaT_vdaMP6QcPC214ANOQjj4lr0gLUg=w3360-h1896)

##### Picking a location

Users can "pick a location" by clicking on the base map. When picking a location, a flag appears at the selected position on the map. And information related to the location will be shown (see figure 4). Those information includes:

- Geographical and administrative Information about the location (e.g. Province, district, longitude, lattitude, etc)
- Overview air quality evaluation with indication by colors (Good, bad, etc) together with health related recommendations.
- Data type: indicating where the data comes from. There are two data types: "Satellite AQI" and "Station" corrensponding to the 2 data layers available in the system
- And a time-based line chart that shows air quality trending in rencent period of time. The x axis will change accordingly when time slider value is changed. And the y axis can be changed to either PM2.5 or AQI scale.

![Figure 4: Picking a location](https://lh3.googleusercontent.com/drive-viewer/AFGJ81r3fgllm0uBS9d6IH54WC6DkK3Z7XgDa4UNZQE2cigbgeEZ_XER8VnnI1KvPPNPSWwf6Z1tnQpbsJ_ihcWjXbc1GjY4Og=w2250-h1896)

As annotated in figure 4, (1) is an indicator showing picked location; (2) are switches for data types; (3) is detailed information of the location; (4) is air quality measurements/observations with health advice; and (5) is air quality trending chart.

Note that, users should select/pick a location that has data available. That means he/she should click on a region that cover by an air quality map or at a position where there is a air monitoring station. 

Also note that, when an user select a location, the information window will automatically show "Selected location" tab. The user can freely switch back to "ranking" tab by accessing the tab controls at the top of the window.

#### Interacting with the time slider

Users can jump to any recent observation date by click on the time slider. When a new observation date is set via the time slider, the base map will change accordingly and so does the air quality trending chart of the selected location. 

Users can also "play" the time slider by clicking on the play button on the left end of the time slider. When the time slider is under playing, the observation time will advance periodically after every 2 seconds.

#### Favorite locations

There are favorite locations appeared at the top of information window (component 2 in table 1). When a user is not logged in, those favorite locations are 6 default favorite cities/provinces of Vietnam: 

- Ha Noi
- Ho Chi Minh
- Da Nang
- Thua Thien Hue
- Bac Ninh
- Quang Ninh

A user can have his/her own list of favorite locations if he/she login to the application. For registering an account and loging in to the system, please see  [Login, Register, and Recover password](#login_register_recover_password) section. Once an user successfully login to the system, he/she can modify the favorite list by clicking at the icon next to a province/city in the ranking table (See figure 5). 

![Figure 5: Favorite list manipulation](https://lh3.googleusercontent.com/drive-viewer/AFGJ81pQx23ofIV1GgU3qaZEIqD6DNReSDAIDbPjiB4dxUGD-aQ00QKY9zlxP7AbXBmPTx9sw6TzIGNeMsA1EaYY3zl8sa8wYQ=w2250-h1896)

As shown in figure 5, if an user click on the icon, and the corresponding province is not marked as a favorite, the icon looks like as number (1) in the figure. If the province is in favorite already, the icon looks like number (2). Clicking on the icon will add/remove the province to the favorite list and apprearance of the icon will change accordingly.

#### Login, Register, Recover Password and other Features

From the `Main Page`, user can login to the application by clicking to Login button in Menu block (Component number 7). From `Login Page`, users can jump to `Registration Page` to register a new account or to `Recover Password Page` to reset a password. Since those features are simple and popular in all applications, we skip describing them for brevity.

Also, via the menu block, users can access POPGIS website where they can find general information about POPGIS application and project.

#### Application's Menu Block for Authenticated Users

Once an user logged in to the POPGIS Web Application, two additional menu items are added to the menu block. The first item is "Data request". Clicking on it will bring the user to a Google form where he/she can complete it to make a request for datasets of interest. The second item is for accessing user's profile and administrative features (for administrator only). Those features are standard and simple, we also skip them for brevity

## POPGIS Data Analysis Page

### Components
The `Data Analysis Page` can be accessed via `Main Page`'s menu block. Components of the `Data Analysis Page` are summarized in table 2 and depicted by Figure 6.

![Figure 6: Components of `Data Analysis Page`](https://lh3.googleusercontent.com/drive-viewer/AFGJ81outDZvpefHOI_3V2SBBfBWXzXiKdQXE7NXWysPOhxAxyxq1eVJdvPDewtx0ztCrc9_YryTshGGmLvGq5ulWzZ0V5JviA=w3360-h1896)

| **SeqNo**  | **Component**             |                                                                          **Description**                                                                         |
|------------|---------------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------|
|      1     | Filter Settings Controls  | Three input boxes for settings: (a) selected province; (b) selected period; and (c) scale                                                                        |
| 2, 3, 4, 5 | Overview metric cards     | Four overview metric cards showing (a) Days of pollutions; (b) number of districts with pollution; (c) the cleanest district; and (d) the most polluted district |
|      6     | Province-level line chart | A line chart that shows air quality trend for the selected province over the selected period                                                                     |
|      7     | Province map              | A map of the selected province. This map shows all authoritative districts                                                                                       |
|      8     | District-based barchart   | A barchart showing air quality level for all districts of the selected province                                                                                  |
|      9     | District level line chart | A line chart showing air quality trend for a selected district over the selected period.                                                                         |

***Table 2: Components of `Data Analysis Page`***

### Features

#### Filter control settings

To work with the `Data Analysis Page`, a user should set desired values on `Filter setting controls` (component 1 from table 2). The component has 3 input boxes for choosing a province, an observation period, and a measurement scale. The first input box is a dropdown showing all provinces or cities in Vietnam. The second one is a datetime picker that helps user in choosing start date and end date. The last dropdown input box allows users to choose a scale from either AQI or PM2.5. When values in filter settings controls are changed, other components (components from 2 to 9) will be updated instantly.

#### Data visualization elements

Remaining components on the page are data visualization widgets that represent some statistical factor for the selected data. All of them are basic and intuitive. Their meanings and intepretations can be found in table 2. Users can also hover their mouses over "question mark" icons on _overview metric cards_ to show  definition of corresponding metric cards.

All data visualization charts of the pages are colored by a consistent color scale that gradually changed from green (cleanest) to purple (most polluted). All data a relevant for settings in the filter control settings

#### District-level air quality trend

After the `Data Analysis Page` is loaded, component 9, _district-level line chart_, is empty since there is no district is selected. To activate this component, users can click on the _province map_ (component 7) or click on _district-level bar chart_ (component 8). The clicked district of the map or datum of the bar chart implied a selected district for analysis and the corresponding line chart will be shown in the region of component 9. 

Also note that, users can pan and zoom the province map using their pointing devices. Also, when a datum on the _district-level bar chart_ is clicked, the province map will automatically pan and zoom so that the selected district will fall within the viewport. Such kind of linkages between those charts are made for enhancing user experience in indentifying and locating districts of interest.

# Pipeline Tracker User Manual

Pipeline Tracker application is made for administration and operation purposes. Using the application, an administrator or operator can monitor the progress and status of POPGIS processing pipelines. Throught the application, he/she can rerun individual stages or the whole pipeline.

The application can be accessed via the URL listed in [Overview](#overview) section. For security reason, a login challenge will be shown to users. Also, users (administrators or operators) are recommended to access the application from the site where the application is running. 

Pipeline Tracker's user interface contains a single page which is shown in figure 7. Components of the page are summarised in table 3

![Figure 7: Pipeline Tracker user interface](https://lh3.googleusercontent.com/drive-viewer/AFGJ81rMrGilLBZ437GxfgTabgWfsZzVOZ3RazDExC4m1perEqXE3L4hCP8o5b1haczg2ufXCwlPgT02GXrQpZY1E6UjZ6x0=w3360-h1896)

| **SeqNo** | **Component**               |                                      **Description**                                      |
|-----------|-----------------------------|:-----------------------------------------------------------------------------------------|
|     1     | Worker list                 | Online workers of the POPGIS system                                                       |
|     2     | Pipelines of a specific day | All stages of pipelines for a day                                                         |
|     3     | Start day                   | Date picker that set "start day". All pipelines since start day will be shown on the page |

***Table 3: Components of Pipeline Tracker user interface***

### Worker list

In figure 7, component number (1) are "worker list". Status of a worker is presented by its color. "Green" means available and "red" mean busy (or running). Task that is running at a worker is also shown on the screen. 

Label of a worker have a tag, "[P]" or "[S]", indicating the job queue that worker is waiting on. "P" means Primary and "S" mean Secondary. The label also shows hostname and process identifier of the worker process.

### Pipelines

Components that are marked with (2) in figure 7 are pipelines for a specific day. There are multiple stages that making pipelines. For details about pipelines and stages, please refer to the "POPGIS development guide" document. Each stage is a box and its color implies the stage's status. The color codes for stage's status are as follows:

- Gray: not run
- Green: success
- Yellow: running
- and Maroon: Failure.


Clicking on a stage activate stage context menu as shown in figure 8. There are two items in the menu: (a) Rerun, and (b) Force rerun.

![Figure 8: stage's context menu](https://lh3.googleusercontent.com/drive-viewer/AFGJ81qP1qy_4sRfspt8DA06EbpkyZYuO18NkRDkUdREKHiqBmXMhJUrP-QNWePkrGPN8-eJKNnFrpbjHRoYZoywR_lNjgX2ZQ=w3360-h1896)

If Rerun command is chosen, the stage will be effectively invoked only if it is "Not run" or "Failed". It the stage is "Success" or "Running" then it will not be actually run. "Force Rerun" command will always reinvoke the stage in any condition.

# References


