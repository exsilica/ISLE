### ISLE Migration Guide (draft 02.28.2018)
This Migration guide will help you migrate your existing production Islandora environment to utilize this ISLE framework for easily maintaining Islandora. This guide will walk you through how to identify and copy your institution's Islandora data and files (including your data volume, Drupal site or sites, and commonly customized XML and XSLT files) to your ISLE framework.
### 1 - Prerequisites
* 1.1 - You have already installed ISLE on your server.
     * 1.1.1 - _If you have not yet installed ISLE on your server:_
        * 1.1.1.1 - [Please Install ISLE using the Setup Guide]()
* 1.2 - You are able to access your ISLE server via ssh (with sudo permission???)
* 1.3 - You have the ability or access to create a private git repository
* 1.4 - You (or an appropriate IT resource) can **COPY** data, configuration files etc. safely from their Islandora Production server(s) to the new ISLE server.
---
### 2 - Quick Overview
* 2.1 - Ensure that the ISLE server has the same (or more) storage capacity as the production server.
* 2.2 - Create appropriate Islandora Production data storage mount or volume on new ISLE server.
* 2.3 - Copy current Islandora Production data and config files to the new ISLE server.
* 2.4 - Create the required new config directory on the new ISLE server.
* 2.5 - Customizations: Carefully compare the following most frequently customized files (XML, stopwords, XSLTs) on your Islandora Production server with the new, default versions found within your new ISLE config folder. Carefully merge any desired customizations from your production Islandora files to persist within the new ISLE config folder of files.
* 2.6 - Edit the `docker-compose.yml` file override default Islandora files with your persisting customized files.
* 2.7 - Stop Docker containers, reindex your SOLR index, restart Docker containers. Test the new config settings in UI (browser)
* 2.8 - Check that all services are running properly
* 2.9 - Repeat entire process (_if necessary_) for additional ISLE environments e.g. _production, staging and development_ (TODO: MUST CLARIFY where to put this new environment, and do we copy our current ISLE folder or... what?)
---
### 3 - Create appropriate Islandora Production data storage structure on new ISLE server
3.1 - _Friendly note: While the following process may seem overly cautious or redundant, it saves time and establishes a safer conditions for you to work with valuable data._
3.2 - We recommend that you use a large volume or network attached drive that can store a backup copy of the entire Islandora production storage, an merged copy of the ISLE production storage and associated config files as outlined in the [Migration Export Checklist](migration_export_checklist.md) and additional datastores.
* 3.2.1 - In an appropriate area / path on one's intended ISLE server e.g. `/opt/` or `/mnt/`, create a directory e.g. `islandora_production_data_storage` with the following sub-directories:
  * **Example directory structure**:
```
    /islandora_production_data_storage/
    ├── apache
    ├── fedora
    │   ├── apache
    │   ├── fedora
    │   └── gsearch
    ├── mysql
    ├── solr
    └── tomcat
```
* 3.2.2 - We recommend that you copy your data here to allow for faster workflow and for you to easily sort through production data.
* 3.2.3 - This directory will serve as the "backup" of all production data about to be migrated into ISLE
* 3.2.4 - Once the migration process is confirmed as completed and successful, you can decide if this data is still worth keeping or backed up.
---
### 4 - Create appropriate ISLE Production data storage structure on new ISLE server
4.1 - _Friendly note: While the following process may seem overly cautious or redundant, it saves time and establishes a safer conditions for you to continue to work with valuable data._
4.2 - In the previous step the `islandora_production_data_storage` directory was setup as a source of materials to continue to **COPY** from i.e. **Islandora Production Data**.
4.3 - This step is to now create the working directory for **ISLE Production Data**.
4.4 - We recommend that you use a large volume or network attached drive that can store the entire ISLE production storage (which is the merged copy of the Islandora production storage and associated config files as outlined in the [Migration Export Checklist](migration_export_checklist.md) and additional datastores.)
* 4.4.1 - In an appropriate area / path on one's intended ISLE server e.g. `/opt/` or `/mnt/`, create a parent directory e.g. `isle_production_data_storage` with the appropriate institutional ISLE environment as a sub-directory. Create as many as required. For this guide only the production environment will be created e.g. `enduser-renamed-directory-prod.institution`
  * 4.4.1.1 - **Example guide directory structure**:
```
    /path_to/isle_production_data_storage/
             └── enduser-renamed-directory-prod.institution
                 ├── apache
                 ├── fedora
                 ├── mysql
                 └── solr
```
* 4.4.2 - While the `/path_to/isle_production_data_storage/` directory will serve as the storage for all ISLE production data, it is highly important to further differentiate future ISLE production data by environment e.g. hence the use of `enduser-renamed-directory-prod.institution`.
4.5 - This structure ensures no accidental data overwrites between ISLE environments and proper functioning.
* 4.5.1 - **Example future directory structure**:
```
  /path_to/isle_production_data_storage/
           |
           ├──enduser-renamed-directory-dev.institution
           |  ├── apache
           |  ├── fedora
           |  ├── mysql
           |  └── solr
           |
           ├──enduser-renamed-directory-prod.institution
           |  ├── apache
           |  ├── fedora
           |  ├── mysql
           |  └── solr
           |
           ├──enduser-renamed-directory-stage.institution
           |  ├── apache
           |  ├── fedora
           |  ├── mysql
           |  └── solr
           |
```
* 4.6 - Later steps in this guide will have you copy data to these directories.
* 4.7 - The `docker-compose.yml` file found within `/opt/ISLE/config/enduser-renamed-directory-prod.institution` is ready for editing to "point" to these data directories. This process will be explained in further detail in section 8 of this guide.
* 4.8 - Once the migration process is confirmed as completed and successful, you will continue to backup up this data.
### 5 - Copy Production Data to ISLE server
* 5.1 - Please review and follow the [Migration Export Checklist](migration_export_checklist.md) to ensure all production data detailed within has been **COPIED** over to the ISLE server **PRIOR** to proceeding further with this Migration guide.
* 5.2 - Important: Once all steps above have been followed, you may proceed to the next section.
---
### 6 - Setup config directory within ISLE Git repo for institutional Docker configuration
* 6.1 - This process is necessary for running multiple versions of ISLE e.g. production, staging and development / sandbox. The config folder is the location for creating multiple versions of ISLE environments such as the example provided below.
* 6.2 - This example below displays, along with the baked in ISLE sample dev site, an institution running a development, staging and production environment all on one ISLE server. This would be a recommended "typical" setup for long term usage.
```
ISLE/config/
├──enduser-renamed-directory-dev.institution
├──enduser-renamed-directory-prod.institution    (this guide uses this as an example only)
├──enduser-renamed-directory-stage.institution
├──isle_localdomain                              (ISLE sample site)
└──isle-prod-project.institution                 (only copy this template / never delete or modify)
```
* 6.3 - For a more detailed explanation please refer to section `2.7 Managing Multiple Environments.`
* 6.4 - **Please note:** While one over time can maintain and update profiles in the `config` directory manually, we highly recommend that this new directory be kept in a private git repository for ease of use in making and preserving changes. For more information on how to use git in an ISLE multi environment setup, please refer to section [2.7 Environments Git Structure](../../2_enduser_guide/2_7_managing_multiple_environments/env-git-structure.md)
* 6.5 - **Setup process**
  * 6.5.1 - * Create a private Git repository on their associated Git server (`private institutional Git repository`) or git hosting service (`Github.com, Bitbucket.com, Gitlab.com`)
  * 6.5.2 - You may need to add the `/home/islandora/.ssh/id_rsa.pub` to the appropriate git repository as a git ssh deploy key to be able to push pull from the server. [WE MUST EXPLAIN OR LINK HOW TO DO THIS STEP AS THIS IS NOT INTUITIVE.]
  * 6.5.3 - Navigate to the ISLE directory `/opt/ISLE/config/``
  * 6.5.4 - Create a directory on the ISLE server directory e.g. `/opt/ISLE/config/enduser-renamed-directory-prod.institution`
  * 6.5.5 - Copy the contents of the `/opt/ISLE/config/isle-prod-project.institution` to this new directory e.g. `/opt/ISLE/config/enduser-renamed-directory-prod.institution`
---
### 7 - Customizations: Merge the customizations from your Islandora production files to the new ISLE config folder of files.
* 7.1 - Customizations: Carefully compare the following most frequently customized files from your Islandora Production server `/opt/ISLE/config/enduser-renamed-directory-prod.institution` with the new, default versions found within your new ISLE config folder `/opt/ISLE/config/enduser-renamed-directory.institution`. Use a "Diff" tool (example: [Beyond Compare](https://www.scootersoftware.com/download.php) to merge any desired customizations from your production Islandora files to persist within the new ISLE config folder of files:
  * 7.1.1 - Compare and merge the Solr files: `schema.XML`
  * 7.1.2 - Compare and merge the Solr files: `solrconfig.XML`
  * 7.1.3 - Compare and merge the Solr files: `stopwords`
  * 7.1.4 - Compare and merge the Fedora GSearch Islandora Transform (XSLTs) folder of files: `islandora_transforms`
* 7.2 - as guided in the [Migration Merge Checklist](migration_merge_checklist.md).
---
### 8 - Edit the `docker-compose.yml` file to:
* 8.1 - Point to the new associated `islandora_production_data_storage` structure.
* 8.2 - Point to the new directories and config settings in `/opt/ISLE/config/enduser-renamed-directory.institution`
* 8.3 - Edit the `docker-compose.yml` found within the `/opt/ISLE/config/enduser-renamed-directory-prod.institution` directory as suggested in the [Migration Docker Compose Edit Guidelines](migration_docker_compose.md).
---
### 9: Review or pull down ISLE Docker images
* 9.1 - _Please Note: You may have already done this in setting up the host server manually and / or with Ansible. However it is always a good idea to review and check using the first command below._
* 9.1 - Check if all ISLE images have been downloaded
  * 9.1.1 - Copy and paste: `docker image ls`
  * 9.1.2 - Does your output match the following output?
  ```
  **TO DO:**  show sample output here
  ```
  * 9.1.3 - If yes, then proceed to section 10
  * 9.1.4 - If no, then perform the following:
    * `docker pull islandoracollabgroup/isle-mysql:latest`
    * `docker pull islandoracollabgroup/isle-fedora:latest`
    * `docker pull islandoracollabgroup/isle-solr:latest`
    * `docker pull islandoracollabgroup/isle-apache:latest`
---
### 10: Spin up mysql container and import production databases
* 10.1 - `cd /opt/ISLE/config/enduser-renamed-directory-prod.institution`
* 10.2 - `docker-compose up -d mysql`
* 10.3 -
```
  **TO DO:**  steps and scripts for mysql cli import
     use GUI and recommended clients e.g. Sequel pro, Navicat etc.
     if encountering errors use _Alternative longer method_ from above
   * _Alternative longer method_
     * one can skip running the drush command on the production apache webserver
     * export the databases as usual from the production mysql server
     * import databases into the isle-mysql container (_with errors being ignored_)
     * truncate all tables that start with `cache` on the isle-mysql container
     * export this new database to the `mysql` directory on the ISLE server
     * delete all tables (_not the database itself_) on the isle-mysql container
     * Reimport the new lighter database to the isle-mysql container
```
---
### 11: Spin up fedora container and run reindex processes
* 11.1 -
```
  **TO DO:**  refine this
  * Staying within `/opt/ISLE/config/enduser-renamed-directory-prod.institution`
  * `docker-compose up -d fedora`
  * check if fedora is running properly e.g. `http://isle-prod-project.institution:8080/manager/html`
  * `docker exec -it isle-fedora-institution bash`
  * turn off tomcat `service tomcat stop`
  * reindex Fedora RI `process steps here`
  * reindex SQL RI `process steps here`
  * confirm PID contents in SQL table `QC process here`
```
---
### 12: Spin up solr container and rerun index processes
* 12.1 -
```
  **TO DO:**  refine this
  * Staying within `/opt/ISLE/config/enduser-renamed-directory-prod.institution`
  * `docker-compose up -d solr`
  * check if solr is running properly e.g. `http://isle-prod-project.institution:8777/manager/html`
  * `docker exec -it isle-fedora-institution bash` NOTE FEDORA NOT SOLR
  * reindex SOLR `process steps here` Use screen
  * TAKES HOURS DEPENDING ON DATA SIZE
```
---
### 13: Spin up apache container and run provision script
* 13.1 -
```
  **TO DO:**  refine this
  * Staying within `/opt/ISLE/config/enduser-renamed-directory-prod.institution`
  * `docker-compose up -d apache`
  * check if apache is running properly e.g. `http://isle-prod-project.institution`
  * `docker exec -it isle-apache-institution bash`
  * cd /tmp location
  * `/tmp/isle-build-tools/apache-provision.sh` (check if this is appropriate)
  * Check site and outline QC process
```
* 13.2 - [TODO: Setup a private git repository to store your ISLE-config folder which will include your docker-compose.yml with institutional passwords, and your customized files.]
===============
## STUFF DAVID-KC REMOVED:
* 1.2 - The fully qualified domain name of their ISLE server that will run all of the containers has been created and resolves properly in DNS.
* 1.3 - You have the IP address of your ISLE server documented
* 1.4 - You have the expected fully qualified domain name(s) (fqdn) of the Islandora / Drupal website(s) documented and assigned to the appropriate IP(s) in DNS.
* 1.8 - Production data, configuration files etc. have been **copied** from the currently running Islandora production server to the new ISLE server following the checklist below of required data, configuration files etc.
