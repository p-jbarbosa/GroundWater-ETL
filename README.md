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
  * file-mgmt
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
  * file-mgmt
    * input
    * output
  * log
  * sql
  * scripts
* **GroundWater-Configs** - controls the available configurations

## Configurations

Project configurations are available under GroundWater-Configs folder. Each folder on this project represents a different environment configuration. There are as much folders as different environments:

* **config-pdi-local** - for local developement
* **config-pdi-dev** - for testing on BA Server
* **config-pdi-uat** - for unit and quality tests
* **config-pdi-prod** - to defined production environment

There are three levels of properties:

* Server wide - available on ../\<config-enviroment\>/.kettle/kettle.properties (reference to common folders, for example)
* Cross-Project - available on ../\<config-enviroment\>/properties/common.properties (common connections)
* Project specific - available on ../\<config-enviroment\>/properties/\<project\>.properties
* Job specific - ../\<config-enviroment\>/properties/\<project\>-\<job\>.properties

Variables that will point to the code content and must be set up accordingly:

* _CONTENT\_DIR_ - .kettle.properties, points to either file system location (developer), or repository location
* _CONTENT\_COMMON\_HOME_ - common.properties, points to _GroundWater-Common/content-pdi_
* _CONTENT\_HOME_ - \<project\>.properties, points _content-pdi_ folder where the project content is.

## Development

The program as a file-based development approach. Each developer will work on his local machine, developing routines and executing them using OS file system.

The whole ETL process, either on local development (file based), either on BA server (for test or production), relies on one requirement: the properties/configurations load before any ETL process execution. For local development, before starting a session each developer must execute the ../config-pdi-local/jb_set_variables_spoon.kjb - this will load all the properties into the JVM that is running spoon.

The main jobs should always be trigged by one master job - \<commom-projec\>/content-pdi/jobs/jb-wrapper.kjb. This job will perform the properties load required and trigger the main job to be executed. To achieve this we will be receiving two parameters:

* _P\_JOB\_NAME_ - the job name that will be executed (without the .kjb extension)
* _P\_PROJECT\_NAME_ - the project name (this should be the same string that is on the \<project\>.properties)

## DBs

TO BE UPDATED.

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