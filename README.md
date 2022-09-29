
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

# Setting up the environment

## Python Libraries

The control system requires the following packages in Python:
* PyQt5
* bitstream
* pandas
* influxdb-client
* pymongo
* uhal
* Pillow

All of them except uhal can be installed with pip:
```
pip install PyQt5
pip install bitstream
pip install pandas
pip install influxdb-client
pip install pymongo
pip install Pillow
```

Installing uhal requires more steps to install, and must be built using from the IPbus repository. Following the instructions that can be found on https://wiki.uib.no/pct/index.php/Using_IPbus.

First, install required packages:

```
sudo apt update
sudo apt install git g++ make erlang python3 python-is-python3 libboost1.71-all-dev libpugixml-dev
```

Then to build the project, go to the directory where the repository was downloaded:

```
make
sudo make install prefix=/home/user/ipbus_install
```
Remember to change 'user' with your own user name. Finally add these paths to your environment:

```
export LD_LIBRARY_PATH=/home/user/ipbus-install/lib:$LD_LIBRARY_PATH
export PYTHONPATH=/home/user/ipbus-install/lib/python3.8/site-packages:$PYTHONPATH
```
Again, remember to change user with your own user name.

## InfluxDB

There are different versions of InfluxDB that can be installed depending on the OS. These can be found at: https://portal.influxdata.com/downloads/.
For this project, the linux binaries option was used to install the database:

```
wget https://dl.influxdata.com/influxdb/releases/influxdb2-2.4.0-linux-amd64.tar.gz
tar xvfz influxdb2-2.4.0-linux-amd64.tar.gz
```

The InfluxDB server can then be started with the command: 
```
influxd
```

the server is by default set to port 8086 on localhost.

Note that the python-influxdb API requires an API token to perform read and write operations to the database. This token can be generated in the influx web interface and then said token must be inserted into the influx API python class found in the IPbusAPI folder.

## Grafana

Again, there are different versions of Grafana, depending on the OS used, which can be found here: https://grafana.com/docs/grafana/latest/setup-grafana/installation/.
We will assume that Ubuntu is used, and we will install the Enterprise edition of Grafana, which is the default. To install Grafana:

```
sudo apt-get install -y apt-transport-https
sudo apt-get install -y software-properties-common wget
sudo wget -q -O /usr/share/keyrings/grafana.key https://packages.grafana.com/gpg.key
```

Then add this repository:
```
echo "deb [signed-by=/usr/share/keyrings/grafana.key] https://packages.grafana.com/enterprise/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
```

Then finally: 
```
sudo apt-get update
sudo apt-get install grafana-enterprise
```

The server can be started with the command:
```
sudo service grafana-server start
```
Grafana is by default set to port 3000 on localhost.

## MongoDB

To install MongoDB community edition:
```
wget -qO - https://www.mongodb.org/static/pgp/server-4.2.asc | sudo apt-key add -
sudo apt-get install gnupg
wget -qO - https://www.mongodb.org/static/pgp/server-4.2.asc | sudo apt-key add -
```
Then add the repository:
```
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.2.list
```
Then it can be installed with:
```
sudo apt-get update
sudo apt-get install -y mongodb-org

```
The server can then be started with:

```
sudo service mongod start
```
MongoDB is by default on port 27017 on localhost. This version of MongoDB does not have a web interface, but you can access the command line for MongoDB with the command:

```
mongo
```

# Booting up the System

The main_hub.py file functions as a top level GUI which can be used to access the different parts of the control system. The main hub gui has two buttons that will open up either the monitoring or configuration GUIs. To start the GUI, go to the gui directory and run the main_hub.py file

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






