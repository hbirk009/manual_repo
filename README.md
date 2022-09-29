# Overview

This repo contains classes and testbenches for creating a monitoring and configuration system for the pCT project. The folders and their purpose is as follows:

* IPbusAPI: contains API classes for the IPbus module, the microcontroller and the configuration system. 
* bsp: contains XML-documents necessary for setting up the IPbus communication
* doxygen: doxygen documentation, incomplete
* gui: Contains gui classes automatically generated from qtdesigner and custom gui classes that connects the gui with the microcontroller API and database
* images: images from various tests and diagrams of system
* influxDB: Contains API for connecting python with influx database and panda data frame class to allow for custom filtering
* mongoDB database manager that allow for easily setting up configuration sets in a mongo database
* simulation: Contains simple simulation classes of the IPbus module and microcontroller
* test_functions: test scripts for various classes and hardware tests 

#Setting up the environment

## Python Libraries

The control system requires the following packages in Python:
* PyQt5
* bitstream
* pandas
* influxdb-client
* pymongo
* uhal

All of them except uhal can be installed with pip:
```
pip install PyQt5
pip install bitstream
pip install pandas
pip install influxdb-client
pip install pymongo
```




# Configuration and Monitoring

## Configuration 

the microcontrollers have 8 registers that should be able to configure through the GUI:

| Register          | Comment           |
| ------------------|:-----------------:|
| DVDD threshold 1  | warning threshold |
| DVDD threshold 2  | error threshold   |
| AVDD threshold 1  | Warning threshold |
| AVDD threshold 2  | error threshold   |
| PWELL threshold 1 | Warning threshold |
| PWELL threshold 2 | error threshold   |
| Temperature limit | thermistor value  |
| Enable signal     | each bit corresponds to one string  |


*db_manager* class sets up and store configuration sets, *config_db.json* gives an example of how the different sets are stored in the database.
The configuration API is a high level interface built on top of the microcontroller API. The hierarchy is as follows:

![](Images/interface_hierarchy.png)

The *config_gui_wrapper* class connects the database with the IPbus API, which allows for creating and loading sets using the GUI


## Monitoring

There are 4 values that needs to be monitored through the monitoring system:

* DVDD
* AVDD
* PWELL-voltage
* Temperature

The *influx_api* class sets up the python-influxdb client and allows for inserting data directly to the database. The *WriteOptions* parameter when instantiating the client is very important, as it determines how it will batch the data points, which can have a major impact on system performance. 
The API is also compatible with the *panda_filter* class, which uses panda data frame for manipulating data before sending it, allowing for custom filtering of data.
We also will perform a standard filtering option through InfluxDB, so the processing of data through the monitoring system will look like:

![](Images/processing_data.png)

The grafana dashboard has been made that gives overview of all layers and all measurements and a json file of this dashboard is in InfluxDB folder
