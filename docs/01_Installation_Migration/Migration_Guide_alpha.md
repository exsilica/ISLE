### ISLE Migration Guide (draft 02.28.2018)

This Migration guide will help you migrate your existing production Islandora environment to utilize this ISLE framework for easily maintaining Islandora. This guide will walk you through how to identify and copy your institution's Islandora data and files (including your data volume, Drupal site or sites, and commonly customized xml and xslt files) to your ISLE framework.


### 1 - Assumptions / Prerequisites

* 1.1 - You have already installed ISLE on your server.

     * 1.1.1 - _If you have not yet installed ISLE on your server:_

        * 1.1.1.1 - [Please Install ISLE using the Setup Guide]()

* 1.5 - You have ssh access to your ISLE host server

* 1.6 - You have the ability or access to create git repositories

* 1.7 - You (or an appropriate IT resource) can **COPY** data, configuration files etc. safely from their Islandora Production server(s) to the new ISLE host server.

[TODO: dwk2 thinks we can eliminate 1.2, 1.3, 1.4, 1.8]

* 1.2 - The fully qualified domain name of their ISLE host server that will run all of the containers has been created and resolves properly in DNS.

* 1.3 - You have the IP address of your ISLE host server documented

* 1.4 - You have the expected fully qualified domain name(s) (fqdn) of the Islandora / Drupal website(s) documented and assigned to the appropriate IP(s) in DNS.

* 1.8 - Production data, configuration files etc. have been **copied** from the currently running Islandora production server to the new ISLE host server following the checklist below of required data, configuration files etc.

---

### 2 - Migration to ISLE Process Overview

2.1 - As this is a large guide, here's a quick not very detailed overview of what's going to happen in the next steps:

* 2.1.1 - Ensure that the destination ISLE host server has the same (or more) amount of storage as the production server.
* 2.1.2 - Create appropriate Islandora Production data storage structure on new ISLE host server
* 2.1.3 - Copy current production data and config files as directed by the [export checklist](migration_export_checklist.md) to an appropriate location on the new ISLE host Server.
* 2.1.4 - Create the required new config directory by copying the template `/opt/ISLE/config/isle-prod-project.institution` to a renamed directory within `/opt/ISLE/config`
* 2.1.5 - Customizations: Carefully compare the following most frequently customized files on your production Islandora server with the new, default versions found within your new ISLE config folder `/opt/ISLE/config/enduser-renamed-directory.institution`. Use a "Diff" tool (example: [Beyond Compare](https://www.scootersoftware.com/download.php) to merge any desired customizations from your production Islandora files to persist within the new ISLE config folder of files:
   * 2.1.5.1 - Compare and merge the Solr files: `schema.xml`
   * 2.1.5.2 - Compare and merge the Solr files: `solrconfig.xml`
   * 2.1.5.3 - Compare and merge the Solr files: `stopwords`
   * 2.1.5.4 - Compare and merge the Fedora GSearch Islandora Transform (XSLTs) folder of files: `islandora_transforms`
* 2.1.6 - Edit the `docker-compose.yml` file to:
   * 2.1.6.1 - Point to the new associated `islandora_production_data_storage` structure.
   * 2.1.6.2 - Point to the new directories and config settings in `/opt/ISLE/config/enduser-renamed-directory.institution`
* 2.1.7 - Spin up new containers one at a time with the new config settings
* 2.1.8 - Check that all services are running properly
* 2.1.9 - Repeat entire process (_if necessary_) for additional ISLE environments e.g. _production, staging and development_ (TODO: MUST CLARIFY where to put this new environment, and do we copy our current ISLE folder or... what?)

---

### 3 - Step 1: Create appropriate Islandora Production data storage structure on new ISLE host server

3.1 - _Friendly note: While the following process may seem overly cautious or redundant, it saves time and establishes a safer conditions for you to work with valuable data._

3.2 - We recommend that you use a large volume or network attached drive that can store a backup copy of the entire Islandora production storage, an merged copy of the ISLE production storage and associated config files as outlined in the [Migration Export Checklist](migration_export_checklist.md) and additional datastores.

* 3.2.1 - In an appropriate area / path on one's intended ISLE host server e.g. `/opt/` or `/mnt/`, create a directory e.g. `islandora_production_data_storage` with the following sub-directories:

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

### 4 - Step 2: Create appropriate ISLE Production data storage structure on new ISLE host server

4.1 - _Friendly note: While the following process may seem overly cautious or redundant, it saves time and establishes a safer conditions for you to continue to work with valuable data._

4.2 - In the previous step the `islandora_production_data_storage` directory was setup as a source of materials to continue to **COPY** from i.e. **Islandora Production Data**.

4.3 - This step is to now create the working directory for **ISLE Production Data**.

4.4 - We recommend that you use a large volume or network attached drive that can store the entire ISLE production storage (which is the merged copy of the Islandora production storage and associated config files as outlined in the [Migration Export Checklist](migration_export_checklist.md) and additional datastores.)

* 4.4.1 - In an appropriate area / path on one's intended ISLE host server e.g. `/opt/` or `/mnt/`, create a parent directory e.g. `isle_production_data_storage` with the appropriate institutional ISLE environment as a sub-directory. Create as many as required. For this guide only the production environment will be created e.g. `enduser-renamed-directory-prod.institution`

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

* 4.6 - Later steps outlined in this guide will have the enduser copy data to these directories.

* 4.7 - The `docker-compose.yml` file found within `/opt/ISLE/config/enduser-renamed-directory-prod.institution` is ready for editing to "point" to these data directories. This process will be explained in further detail in Step 6 of this guide.

* 4.8 - Once the migration process is confirmed as completed and successful, the enduser will continue to backup up this data.

### 5 - Step 3: Copy Production Data to ISLE Host server

5.1 - Please review and follow the [Migration Export Checklist](migration_export_checklist.md) to ensure all production data detailed within has been **COPIED** over to the ISLE Host Server **PRIOR** to proceeding further with this Migration guide.

5.2 - Once all steps have been followed, continue on to Step 4.

---

### 6 - Step 4: Setup config directory within ISLE Git repo for institutional Docker configuration

This process is necessary for running multiple versions of ISLE e.g. production, staging and development / sandbox. The config folder is the location for creating multiple versions of ISLE environments such as the example provided below.

This example below displays, along with the baked in ISLE sample dev site, an institution running a development, staging and production environment all on one ISLE host server. This would be a recommended "typical" setup for long term usage.
```
ISLE/config/  
├──enduser-renamed-directory-dev.institution      
├──enduser-renamed-directory-prod.institution    (this guide uses this as an example only)
├──enduser-renamed-directory-stage.institution
├──isle_localdomain                              (ISLE sample site)   
└──isle-prod-project.institution                 (only copy this template / never delete or modify)
```

For a more detailed explanation please refer to section `2.7 Managing Multiple Environments.`

**Please note:** While one over time can maintain and update profiles in the `config` directory manually, we highly recommend that this new directory be kept in a private git repository for ease of use in making and preserving changes. For more information on how to use git in an ISLE multi environment setup, please refer to section [2.7 Environments Git Structure](../../2_enduser_guide/2_7_managing_multiple_environments/env-git-structure.md)

**Setup process**  

* Create a private Git repository on their associated Git server (`private institutional Git repository`) or git hosting service (`Github.com, Bitbucket.com, Gitlab.com`)  
   * The enduser may need to add the `/home/islandora/.ssh/id_rsa.pub` to the appropriate git repository as a git ssh deploy key to be able to push pull from the server.

* Navigate to the ISLE directory `/opt/ISLE/config/``

* Create a directory on the ISLE server directory e.g. `/opt/ISLE/config/enduser-renamed-directory-prod.institution`

* Copy the contents of the `/opt/ISLE/config/isle-prod-project.institution` to this new directory e.g. `/opt/ISLE/config/enduser-renamed-directory-prod.institution`

---

### Step 5: Edit, merge or copy in Islandora production files or data to the new ISLE Production config or data directories

* Compare the data and settings of the files found within `islandora_production_data_storage`, and then merge, edit or copy as necessary with the templated settings found within the newly renamed directory of `/opt/ISLE/config/enduser-renamed-directory-prod.institution` as guided in the [Migration Merge Checklist](migration_merge_checklist.md).

---

### Step 6: Edit the `docker-compose.yml` file

Edit the `docker-compose.yml` found within the `/opt/ISLE/config/enduser-renamed-directory-prod.institution` directory as suggested in the [Migration Docker Compose Edit Guidelines](migration_docker_compose.md).

---

### Step 7: Review or pull down ISLE Docker images

_Please Note: You may have already done this in setting up the host server manually and / or with Ansible. However it is always a good idea to review and check using the first command below._

* Check if all ISLE images have been downloaded
  * `docker image ls`

```
  * **TO DO:**  show sample output here
```

  * If yes, then proceed to Step 7

  * If no, the perform the following:
    * `docker pull islandoracollabgroup/isle-mysql:latest`
    * `docker pull islandoracollabgroup/isle-fedora:latest`
    * `docker pull islandoracollabgroup/isle-solr:latest`
    * `docker pull islandoracollabgroup/isle-apache:latest`

---

### Step 8: Spin up mysql container and import production databases

* `cd /opt/ISLE/config/enduser-renamed-directory-prod.institution`
* `docker-compose up -d mysql`

```
**TO DO:**  steps and scripts for mysql cli import
       use GUI and recommended clients e.g. Sequel pro, Navicat etc.
       if encountering errors use _Alternative longer method_ from above

       * _Alternative longer method_
         * one can skip running the drush command on the production apache webserver
         * export the databases as usual from the production mysql server
         * import databases into the isle-mysql container (_with errors being ignored_)
         * truncate all tables that start with `cache` on the isle-mysql container
         * export this new database to the `mysql` directory on the isle host server
         * delete all tables (_not the database itself_) on the isle-mysql container
         * Reimport the new lighter database to the isle-mysql container


```

---

### Step 9: Spin up fedora container and run reindex processes
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

### Step 10: Spin up solr container and rerun index processes
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

### Step 11: Spin up apache container and run provision script
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

[TODO: Setup a private git repository to store your ISLE-config folder which will include your docker-compose.yml with institutional passwords, and your customized files.]
