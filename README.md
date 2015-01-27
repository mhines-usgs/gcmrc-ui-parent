# gcmrc-ui-parent aka the GCMRC Web Application
The Grand Canyon Monitoring and Research Center (GCMRC) in partnership with the USGS Center for Integrated Data Analytics (CIDA) have designed and built a database and web application for serving, and operating on, time-series measurements of key water discharge and suspended-sediment concentration values. This web application is currently called the Discharge, Sediment, and Water Quality Monitoring web application, or GCMRC web application. A live implementation can be seen online at: http://www.gcmrc.gov/discharge_qw_sediment

This web-based set of tools and processes continually aggregate, compute and display data of interest to resource managers and decision makers in support of the Glen Canyon Adaptive Management Program. First and foremost, these tools are being used on a monthly basis by engineers in the Bureau of Reclamation working within the Glen Canyon Dam Adaptive Management Program (GCDAMP) to plan and implement the release of controlled floods from Glen Canyon Dam to rebuild sandbar habitat in Grand Canyon National Park when downstream sand conditions warrant (U.S. Department of the Interior, 2012). The sediment-budgeting tools are also being used in GCDAMP to evaluate the sediment response to these controlled floods and to evaluate the effects of hydropower operations at Glen Canyon Dam on the sand resources in multiple reaches of the Colorado River in Grand Canyon National Park and Lake Mead National Recreation Area. National Park Service managers also employ the sediment-budgeting tool to seasonally evaluate the effects of flows released from Flaming Gorge Dam on the Green River in combination with natural flood flows from the Yampa River on the sediment resources in Dinosaur National Monument (e.g., Mueller et al., 2014a,b). Finally, the sediment-budgeting tool is being used by scientists to evaluate the effects of upstream Mexican dam releases and local tributary floods on channel and floodplain evolution in the Rio Grande in Big Bend National Park (Dean et al., 2015).

The majority of this documentation is drawn from a technical paper ("User-interactive Sediment Budgets in a Browser: A Web Application for River Science and Management", authored by David Sibley, USGS CIDA; David Topping, USGS Southwest Biological Science Center and GCMRC; Megan Hines, USGS CIDA; and Bradley Garner, USGS Arizona Water Science Center) for the joint 10th Federal Interagency Sedimentation Conference and 5th Federal Interagency Hydrologic Modeling Conference held in April 2015. View the full paper: https://drive.google.com/file/d/0B1kQkbqT0fOqUnJYVndfclN6akk/view

###Data sources and methods
The main data types for the GCRMC project include 1) continuous time-series data and 2) discrete episodically collected suspended- and bed-sediment data. 

The continuous time-series data are typically spaced at 15-minute intervals and include gage height, discharge, water temperature, specific conductance, dissolved oxygen, turbidity, and acoustic measurements of suspended-silt-and-clay concentration, suspended-sand concentration, and suspended-sand median grain size (Griffiths et al., 2012, 2014; Topping et al., 2003, 2007, 2015; Voichick, 2008; Voichick and Topping, 2010, 2014; Voichick and Wright, 2007).

The discrete sample data include equal-discharge-increment (EDI), equal-width-increment (EWI), single-vertical, and calibrated-pump suspended-sediment measurements, and bed-sediment measurements (Edwards and Glysson, 1999).

All of these data are collected using standard USGS methods and other peer-reviewed methods. 95-percent-confidence-level field and laboratory-processing errors for the EDI and EWI measurements are calculated using the methods of Topping et al. (2010, 2011); similar errors for the calibrated-pump measurements are calculated using unpublished analyses based on the methods of Topping et al. (2011). These errors are depicted in the user-interactive plots in the GCMRC web application.

In order to construct the visualization and modeling capabilities displayed within the GCMRC web application, the different datasets are aggregated, and derived-values calculations are performed and stored on new incoming data. The flow of this process is depicted below. 

![Data flow diagram depicting the movement of data from raw form to client-side visualization](https://drive.google.com/uc?export=view&id=0B1kQkbqT0fOqbzloc0hwM1JJUzQ "Data flow diagram depicting the movement of data from raw form to client-side visualization")

- GDAWS is a single database organized with the star schema typical in data warehousing. It is implemented on an Oracle database server.
- Time-series data about our stations of interest from the U.S. Geological Survey’s National Water Information System (NWIS) database (USGS NWIS, 2014) are extracted twice daily and placed on a secure USGS File Transfer Protocol (FTP) site by a Python program called GADSYNC-NWIS.
- Twice daily, a C# program called GADSYNC-GDAWS retrieves the data placed on the secure FTP site, lightly processes them, and inserts their data into GDAWS. Time-series data not included in NWIS, and discrete sample data, are manually uploaded into GDAWS periodically using a custom web application (the GCMRC Upload Tool) that extracts, transforms and loads the comma-separated data values. To date, the data warehouse contains over 94 million time-series measurements from 58 sites throughout the aforementioned networks. The discrete sample data are critical for calibrating and verifying the acoustic suspended-sediment data and for calculating sediment loads.
- A C# program called Autoproc runs daily, calculating both instantaneous and cumulative sediment loads from the aggregated discharge, acoustic suspended-sediment time-series, and discrete suspended-sediment-sample information in GDAWS. Autoproc extracts data from the GDAWS database, processes parameter values based on the latest time-series data, and re-inserts the newly calculated values back into the database.

###Computation of concentrations: major rivers
On the Little Snake, Yampa, Green, and Colorado rivers, and the Rio Grande, suspended-sediment concentrations and grain-size distributions are measured using a combination of 15-minute acoustic data and episodic discrete EDI, EWI, and calibrated-pump sample measurements. The discrete sample measurements are used to both calibrate the acoustic data and then subsequently the acoustic calibrations using the methods of Topping et al. (2007, 2015).

###Computation of concentrations: major tributaries
On major tributary rivers where the suspended-sediment concentrations exceed the upper limit where acoustic measurements are possible, suspended-sediment concentrations are computed using a two-step process. First, suspended-silt-and-clay and suspended-sand concentrations are estimated using either physically based model curves (Topping, 1997) or statistically based curves (Topping et al., 2010). Second, these initial sediment-concentration estimates are adjusted to agree with EDI, EWI, single-vertical, and calibrated-pump measured sediment-concentrations as samples get processed through the laboratory, a time-consuming process (Topping et al., 2010). As more samples are incorporated, and the initial estimates become bona fide measurements of suspended-sediment concentration, the uncertainties applied to the tributary suspended-sediment data decrease in the web application.

###Computation of concentrations: minor tributaries
The suspended-sediment concentrations in smaller, less important tributary streams are determined using a combination of measurements (Griffiths et al., 2010, 2014, 2015) and indirect estimates (Topping et al., 2010). The uncertainties assigned to the sediment loads in these streams are much larger (typically 50 to 100 percent), but because the loads in these streams are much smaller than the loads in either the mainstem rivers or major tributaries, these large uncertainties do not generally affect the sediment-budget results.

###Computation of sediment loads
Sediment loads are calculated using 15-minute discharge and suspended-sediment-concentration data using the method of Porterfield (1972). Because sand, and silt and clay serve different physical and ecological purposes, sand loads and silt and clay loads are calculated independently. This approach allows construction of separate user-interactive mass-balance sand budgets and silt and clay budgets in the GCMRC web application. Construction of these mass-balance sediment budgets require sediment loads to be known on the mainstem rivers, major tributaries, and lesser tributaries, all with assigned uncertainties that are propagated through the budgets.
Computation of mass-balance sediment budgets
The mass-balance sediment budgets for each river reach are calculated using the methods described in Topping et al. (2010) and Grams et al. (2013).

###Computed uncertainty
Uncertainties are applied to all sediment loads used in these budgets. The default uncertainties in the web application are chosen such that they represent the largest potential persistent bias in the computed loads at each site. The user can modify these default uncertainties to explore their effect on the uncertainties in the sediment budgets. These bias-type uncertainties result largely from instrumentation bias and include the greatest likely persistent bias in both the discharge of water and the suspended-sediment concentration, and are constrained by consistent measured differences in either discharge or sediment concentration at adjacent cross sections (Topping et al., 2010). Because no difference in either water discharge or sediment concentration occur between these closely spaced cross sections, the differences in the measurements between these cross sections represent biases (generally < 5 percent) in how acoustic-Doppler current profilers, current meters, and suspended-sediment samplers perform in slightly different cross sections. As there is no way to independently know which measurement in which cross section is correct, there is no way to know the “true” value. Thus, uncertainties that represent the greatest likely magnitude of these
persistent differences are assigned to each load value. Because these uncertainties are biases, they accumulate over time, resulting in mass-balance sediment budgets in the GCMRC web application with uncertainty that gets larger over time.

###Service/browser interaction in handling of computed data
To serve the aggregated and calculated sediment data for use in the front end application, the system organizes requested data into JavaScript Object Notation (JSON) responses. In order to allow the user to adjust the above-described load uncertainties in real-time on their computer, the browser requires all the data be delivered as separate pieces in the JSON response. Usually, the required data are two to five 15-minute time-series that need to be transferred to the browser, which can be roughly a million values for an example span of six years. However, in order to accommodate the spectrum of browser memory capacities and variety of internet connection speeds, the application filters the plotted data to windowed local minimums and maximums on requests of periods of that length. The number of values that make it to the browser for the six-year example would therefore be reduced to a few hundred thousand. Because the application attempts to serve the truest visual representation of the data, it is set up to scale the windowed filtering based on the amount of data the user requests.

Full reference details available within the full paper: https://drive.google.com/file/d/0B1kQkbqT0fOqUnJYVndfclN6akk/view