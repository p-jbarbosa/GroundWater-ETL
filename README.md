# GroundWater ETL Program

ETL repository for GroundWater. Performs DW loads for dimension and fact tables.

## Program Structure

The ETL Program is controlled by the following projects:

* **GroundWater-ETL** - the major code is here (on a multiple project Program there will be multiple folders like this one).
  * content-pdi
    * jobs
    * transformations
  * content-ba
  * documentation
  * file-mgmt - TO DECIDE IF IT'S NEED
    * input
    * output
  * log
  * sql
  * scripts
* **GroundWater-Common** - holds the common program code
  * content-pdi
    * jobs
    * transformations
  * content-ba
  * documentation
  * file-mgmt  - TO DECIDE IF IT'S NEED
    * input
    * output
  * log
  * sql
  * scripts
* **GroundWater-Configs**
  * controls the available configurations and supplies a folder to centralize uploading files (deploy-pdi)

``` txt
├── config-pdi-dev
│   ├── jb_set_variables_spoon.kjb
│   ├── properties
│   ├── spoon.sh
│   └── start-pentaho.sh
├── config-pdi-local
│   ├── jb_set_variables_spoon.kjb
│   ├── properties
│   └── spoon.sh
└── README.md
```

**GroundWater-Common** and **GroundWater-ETL** will have folders both one the local file system and on the BA server (content-pdi and probably content-ba).

## Configurations

Project configurations are available under GroundWater-Configs folder. Each folder on this project represents a different environment configuration. There are as much folders as different environments:

* **config-pdi-local** - for local developement
* **config-pdi-dev** - for testing on BA Server
* **config-pdi-uat** - for unit and quality tests
* **config-pdi-prod** - to defined production environment

Variables that will point to the code content and must be set up accordingly:

* _CONTENT\_DIR_ - .kettle.properties, points to either file system location (developer), or repository location
* _CONTENT\_COMMON\_HOME_ - common.properties, points to _GroundWater-Common/content-pdi_
* _CONTENT\_HOME_ - \<project\>.properties, points _content-pdi_ folder where the project content is.

## Development

The ETL Program as a file-based development approach. Each developer will work on his local machine, developing routines and executing them using OS file system.

The whole ETL process, either on local development (file based), either on BA server (for test or production), relies on one requirement: the properties/configurations load before any ETL process execution. For local development, before starting a session each developer must execute the ../config-pdi-local/jb_set_variables_spoon.kjb - this will load all the properties into the JVM that is running spoon and allowing to execute parametrized transformations.

The main ETL jobs should always be trigged by one master job - \<commom-projec\>/content-pdi/jobs/jb-wrapper.kjb. This job will load the properties required and trigger the main job to be executed. To achieve this we will be receiving two parameters:

* _P\_JOB\_NAME_ - the job name that will be executed (without the .kjb extension)
* _P\_PROJECT\_NAME_ - the project name (this should be the same string that is on the \<project\>.properties)

And there are three levels of properties:

* Server wide - available on ../\<config-enviroment\>/.kettle/kettle.properties (reference to common folders, for example)
* Cross-Project - available on ../\<config-enviroment\>/properties/common.properties (common connections)
* Project specific - available on ../\<config-enviroment\>/properties/\<project\>.properties
* Job specific - ../\<config-enviroment\>/properties/\<project\>-\<job\>.properties

## Logging

Logging is made on two ways:

* on the job/transformation calling step - here the configuration tells PDI to save the logs on a text file located in a specific folder

``` txt

├── _20180330_144812.log - no job associated to this cal
├── _20180330_144910.log
├── README.MD
├── test-deploy-job_20180330_141615.log
├── test-deploy-job_20180330_141716.log
├── test-deploy-job_20180330_143033.log
├── test-deploy-job_20180330_144704.log
├── test-deploy-job_20180330_145033.log
└── test-deploy-job_20180330_145731.log

```

* inside the job/transformation - where you can specify the DB information for the logging (Operations Mart like logging)
  * depending on the needs we can log for the entire project through kettle.properties or to specific project putting those variables on the project.properties

## Monitoring

Two different levels of monitoring: one for ETL management control, the oher for ETL control

* using the Operations Mart, configured with the environments configuration strucuture provided we can set job/transformation monitoring
* a separated table was setup to create control job concurrency and intermediate logging
  * on the GroundWater project there is the _batch_ table
  * on the test-deploy project there is the _job_ table

## Git operations

TO BE UPDATED.

* always develop on dev branch
* merge dev to master
  * git checkout dev
  * perform devs
  * git commit -m ...
  * git push origin dev
  * git checkout master
  * git merge --no-ff dev
  * git push (when following the opposite path git push origin dev)

## Running on BA Server

To running this ETL setup the BA server needs to know where the Program special .kettle folder is located before the launch. This is made with a special _start\_pentaho.sh_

### BA Server special start

```sh

#! /bin/sh

PBA_DIR=/home/malskat/Pentaho/server/pentaho-server

export KETTLE_HOME="$PWD"
echo "starting kettle on "$KETTLE_HOME
KETTLE_META_HOME="$PWD"

sh $PBA_DIR/start-pentaho.sh

```

### API examples to run on BA server

* API call examples

``` txt

GET http://di-server:8080/pentaho/kettle/executeJob/?rep=Localhost&user=admin&pass=password&job=/public/GroundWater/GroundWater-Common/content-pdi/jobs/jb-wrapper&P_JOB_NAME=test-deploy-job

POST http://di-server:8080/pentaho/kettle/executeJob/?rep=Localhost&user=admin&pass=password&job=/public/GroundWater/GroundWater-Common/content-pdi/jobs/jb-wrapper&P_JOB_NAME=test-deploy-job

```

### Importing to BA Server

* zip project files to deploy-folder

``` sh
zip -r GroundWater-Configs/deploy-pdi/GroundWater-ETL.zip GroundWater-ETL/ -x *.git* *sql* *file-mgmt* *log* .gitignore
zip -r GroundWater-Configs/deploy-pdi/GroundWater-Common.zip GroundWater-Common/ -x *.git* *sql*
```

* import projects

``` sh
cd ~/Pentaho/server/pentaho-server/

./import-export.sh --import --url=http://localhost:8080/pentaho --username=admin --password=password --overwrite=true --path=/public/GroundWater --file-path=/home/malskat/sandbox/GroundWater-Configs/deploy-pdi/GroundWater-ETL.zip

./import-export.sh --import --url=http://localhost:8080/pentaho --username=admin --password=password --overwrite=true --path=/public/GroundWater --file-path=/home/malskat/sandbox/GroundWater-Configs/deploy-pdi/GroundWater-Common.zip
```

### Running with kitchen

Run the script from the configuration folder.

``` sh
./ground-water-kitchen.sh -file=/home/malskat/sandbox/GroundWater-Common/content-pdi/jobs/jb-wrapper.kjb -param:P_JOB_NAME=test-deploy-job -level=Minimal
```