# DataLake-as-a-Service for Open Science

This service provides the end-user with a Notebook ready-to-be-used and fully integrated with the ESCAPE Data Lake. The service is hosted in [https://escape-notebook.cern.ch/](https://escape-notebook.cern.ch/).

## Authentication
Authenticate to IAM. Once you log into the service for the first time, everything will be configured for the OpenID connect by default. You will be able to modify it later and make it persistent.
If you don't have an IAM account already, see the [authentication](index.md) section.

## Server Options
Select your server option based on your requirements:

* Minimal environment: Based on jupyter/scipy-notebook.
* ROOT environment: If you need to use PyROOT.
* ROOT environment (Xcache testing): Run the extension in Download mode.

## Service Usage
Once you are inside the service, you will find 5 panels in the left part menu.


##### 1. File Browser:
It enables you to work with the files and directories on your sistem. You can find more information in the [Jupyerlab documentation](https://jupyterlab.readthedocs.io/en/stable/user/files.html).


In case you want to upload a file you will need to use this pannel, right click on the file and select "Upload File(s) to Rucio". Fill in the form and note that the upload may fail if the destination RSE does not support your authentication method.

##### 2. Running Terminals and Kernels
The Running panel in the left sidebar shows a list of all the kernels and terminals currently running across all notebooks, code consoles, and directories. Find more information in the [Jupyerlab documentation](https://jupyterlab.readthedocs.io/en/stable/user/running.html).

##### 3. Table of Contents
This makes it easy to see and navigate the structure of a document. A table of contents is auto-generated in the left sidebar when you have a notebook, markdown, latex or python files opened. The entries are clickable, and scroll the document to the heading in question. Find more information in the [Jupyerlab documentation](https://jupyterlab.readthedocs.io/en/stable/user/toc.html).

##### 4. Rucio extension
* **EXPLORE**:
This is where you will interact with the DataLake and see all the files (Files, datasets and containers). If you want to use a specific data set, you will need to make it available and add it to your notebook. To do so, click on the folder button to see all the available scopes, select the one that stores your data and complete the search with the absolute DID. The DID syntax has some search flexibility:
    - If you want to see all the files that are inside a specific scope your sintaxis will be ``` SCOPE:* ``` (Example: ```ATLAS_OD_EDU:*```).
    - In case you want to see all the files that starts with a specific term in a specific scope your sintaxis will be ```SCOPE:TERM*``` (Example: ```ATLAS_OD_EDU:data*```).
    - Finally if you want to see a specific file type the complete DID: ```SCOPE:NAME``` (Example: ```ATLAS_OD_EDU:data_B.GamGam.root```).

    In case you want to know if there is a replication attached to that file, you need to click on the Availability button and see the rucio web UI. This availability button has some difererent states: Available, Not Available, Collection is empty, Something went wrong... If the state is Not Available you will need to make it Available. Once done you can add it to your notebook.

* **NOTEBOOK**:
    Here you will be able to see all the files you added to the notebook and are ready to be used.

* **CONFIGURATION**:
    There are multiple authentication methods, OpenID connect will be selected by default. Change the authentication method in case you want to use another one. You can select one of the following:
        - OpenID Connect
        - X.509 User Certificate
        - X.509 Proxy Certificate
        - Username & Password

##### 5. Extension manager
You can use this panel in case you want to manage your extensions. This is an option you will mostly not need.

