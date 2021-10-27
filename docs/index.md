# Authentication

The interaction with the [Data Lake](https://gitlab.cern.ch/escape-wp2/datalake-as-a-service-docs/-/tree/master/base) requires authentication.
It is required to register to the INDIGO Identity and Access Management ([IAM](https://indigo-iam.github.io/v/v1.7.1/docs/overview/)) service and obtain a X.509 Grid certificate. Follow the instructions in the next sections.

### ESCAPE IAM registration

Navigate [here](https://iam-escape.cloud.cnaf.infn.it/login) and register for a new account, filling in the required details.
After the account has been approved, you will need to log in and add the Escape group to your Groups. To do this, click on the green **Join a group** botton in the **Group requests** panel. Enter 'escape' in the group box, adding a brief motivation for the request. You will need to wait in order for this to be approved.

### X.509 Grid certificate    

##### Getting the certificate    

The following procedure is described for a UK CA/RA, but it is different depending on the country you are located in. Check this [link](https://www.eugridpma.org/members/worldmap/) to see what you need. 

An X.509 grid certificate is sought by first contacting the appropriate certificate authority (CA). This can be done [here](https://portal.ca.grid-support.ac.uk/). Make a note of both the pin and key password, as they will be required later.

After your request, you will receive an email asking for further information to be passed to the registration authority (RA). 
You will need some form of I.D. (e.g. site I.D., driver’s licence, passport) in addition to the pin that was entered in the request to the CA. You will need to pass these to one of the contacts listed in the email, as instructed, who will likely want to arrange a brief video call to confirm your identity against your provided documents.

*For **CERN** staff members, the request can be made [here](https://ca.cern.ch/ca/), under the ‘Grid Certificates’ section with ‘New Grid User Certificate’.* 

Once your identity has been confirmed, you will be emailed a link to download your certificate in .p12 format (PKCS 12). Download the file and store it in a safe place on your computer, where it is always easily accessible. 

##### Uploading the certificate

Before linking the Grid certificate to your IAM account, you will need to add the certificate to your browser. The procedure you will need to follow varies depending on what browser you are using. For Chrome, this can be done by navigating to chrome://settings/security and following the instructions presented upon selecting Advanced > Manage Certificates > Import.
Once the certificate has been imported, log into the ESCAPE IAM and select the green **Link certificate** botton in the **X.509 certificates** panel. 

##### Changing the certificate 

Separate out the .p12 object into a separate key (.key) and certificate (.crt). 
Place the .p12 certificate file in the .globus directory in your home area. If the directory doesn’t exist, create it:
```console
$ cd ~
$ mkdir .globus
$ cd ~/.globus
$ mv /path/to/mycert.p12 .
```
From the .globus directory, execute the following shell commands:

```console
$ rm -f client.crt
$ rm -f client.key
$ openssl pkcs12 -in myCert.p12 -clcerts -nokeys -out client.crt
$ openssl pkcs12 -in myCert.p12 -nocerts -nodes -out client.key
$ chmod 400 client.crt
$ chmod 400 client.key
```
This completes the X.509 certificate set-up for ESCAPE.

# Installing the Rucio Client Environment 

This guide takes a look at how to install the Rucio client environment in three different ways: using Docker, using Singularity, or manually by downloading rucio on your machine. 
Docker and Singularity mitigate dependency and platform specific issues, and are therefore recommended methods. 

### Docker installation 

##### As root

Docker needs to be installed following the [Docker installation](https://docs.docker.com/get-docker/) instructions. The procedure will change depending on your operating system. 
The Docker file will extend the Rucio image and will enable the user to interact with the Data Lake. Further information can be found [here](https://github.com/ESCAPE-WP2/Rucio-Client-Containers/tree/master/rucio-client-container). 
The Docker image for this project can be pulled and tagged with:

```console
$ docker pull projectescape/rucio-client:latest
$ docker tag projectescape/rucio-client rucio-client
```
Alternatively, to build your own image locally, clone the repository linked above and follow the instructions in the ‘Build image’ section of the [README](https://github.com/ESCAPE-WP2/Rucio-Client-Containers/blob/master/rucio-client-container/README.md) file. 

A Rucio instance is fully described by the configuration file /opt/rucio/etc/rucio.cfg. A default for this file is incorporated into the image and supplied to the container when it is run. The values from this default can be partially or fully overridden by either specifying parameters as environment variables or by mounting a bespoke configuration file. Examples of both methods are discussed in the [README](https://github.com/ESCAPE-WP2/Rucio-Client-Containers/blob/master/rucio-client-container/README.md) file.

Note that if you are uploading data from the host machine where the Docker container is running, you will need to bind that directory for it to be accessible inside the container.
If you are on a HPC cluster, or somewhere where Docker is not usable, try running the container with [Singularity](### Singularity installation). 

To see whether you pulled or created the Docker image correctly, refer to the [Docker documentation](https://docs.docker.com/get-started/). For example, you could run: 

```console
$ docker ps -a
```
Assuming that your .crt and .key files were generated in the .globus directory, they must now be volume bound to /opt/rucio/etc/ in the container on initialisation. The following command will link the key pairs from the local path to the rucio path:

```console
$ docker run -e RUCIO_CFG_ACCOUNT=<myrucioaccount> -v ~/.globus/client.crt:/opt/rucio/etc/client.crt -v ~/.globus/client.key:/opt/rucio/etc/client.key -it --name=rucio-client rucio-client
```
Replace \< myrucioaccount \> with your IAM username. If you encounter permission problems, execute the above command with root permission:

```console
$ docker run --user root -e RUCIO_CFG_ACCOUNT=<myrucioaccount> -v ~/.globus/client.crt:/opt/rucio/etc/client.crt -v ~/.globus/client.key:/opt/rucio/etc/client.key -it --name=rucio-client rucio-client
```

##### As user - no root permissions 

If you cannot log in as root and you get permission errors, add your user account to the docker group:

```console
$ sudo groupadd docker
$ sudo gpasswd -a $USER docker
```
Sometimes you will still need to give the /var/run/docker.sock socket and /var/run/docker directory the proper permissions to make it work:

```console
$ sudo chown root:docker /var/run/docker.sock
$ sudo chown -R root:docker /var/run/docker
```
Place your original .p12 complete certificate in your .globus/ directory and run the following command:

```console
$ docker run --user "$(id -u):$(id -g)" -e RUCIO_CFG_ACCOUNT=<myrucioaccount> --mount "type=bind,src=$(pwd)/.globus,dst=/opt/rucio/etc" --workdir /opt/rucio/etc -it --name=rucio-client rucio-client
```
You will be automatically logged into the Docker container, within a Rucio environment. 
Once this command brings you inside the docker container, execute the following to allow all the VOMs to take place. 

```console
$ voms-proxy-init --voms escape --cert opt/rucio/etc/<certificate_name>.p12
```
**General note:** To access the rucio-client Docker container in the future, always use this command from the machine where you have Docker installed:

```console
$ sudo docker exec -it rucio-client /bin/bash
```
This will log you into the rucio-client container. You can then start Interacting with [Rucio](https://rucio.readthedocs.io/en/latest/index.html) through the ESCAPE Data Lake in your new environment. Take a look at some [Rucio CLI Quickstart commands](https://docs.google.com/document/d/1LKJu56VMg7jkh19BtWoS3xSNYPf_kJN2xRKQrankfB8/edit#heading=h.avprao92dhlc) to get you familiarised with the main concepts. 

### Singularity installation   

To use the client in [Singularity](https://sylabs.io/guides/3.3/user-guide/quick_start.html#quick-installation-steps), first create a Singularity image based on the Docker version
```console
$ n
```
Then, create a directory and add your credentials as follows. You will have to replace \< myrucioaccount \> with your IAM username. 

```console
$ mkdir -p ${HOME}/.rucio
$ export RUCIO_CFG_ACCOUNT=<myrucioaccount>
$ singularity run -B ${HOME}/.rucio/:/opt/rucio/etc -B ${HOME}/.globus/client.crt:/opt/rucio/etc/client.crt -B ${HOME}/.globus/client.key:/opt/rucio/etc/client.key rucio-cli.simg
```
When running the containerised client, the directory that contains the configuration files should be writable at run time. 
This means that you have to bind it to the container. 
Note that this will cause the configuration to be kept between runs, as opposed to running the client in a Docker container which will write a new configuration file at run time (this means that setting the RUCIO_CFG_ACCOUNT variable will only be needed during the first run). 
If for any reason you want to reset the configuration, just erase the rucio.cfg in the configuration directory (in the example: $HOME/.rucio).

### Manual installation  

If you installed the Rucio CLI with Docker, then you will already have the ca-certificates and VOMS installed, but in the manual installation you will have to do it yourself by running the following steps:

##### Certificates

Install the necessary certificates as specified in the [Rucio client Docker file](https://github.com/ESCAPE-WP2/Rucio-Client-Containers/blob/master/rucio-client-container/Dockerfile#L8) : 

```console
$ curl -o /etc/yum.repos.d/EGI-trustanchors.repo http://repository.egi.eu/sw/production/cas/1/current/repo-files/EGI-trustanchors.repo && yum -y update
$ yum install -y ca-certificates ca-policy-egi-core fetch-crl
```
##### Setting up VOMS environment 

On your local machine or virtual machine, move your certificates from the  /opt/rucio/etc/ directory to .globus/, and rename the certificates as specified below. Then install the voms-clients. 

```console
$ mv client.crt usercert.pem
$ mv client.key userkey.pem
$ yum install voms-clients
```
After you have done so, follow [these instructions](https://github.com/indigo-iam/escape-docs#escape-vo-configuration) under the section ‘ESCAPE VO configuration’. 
You will have to create the directory /etc/grid-security/vomsdir/escape and place the LCS file in it (create it with touch), and do the same thing for the VOMSES file if the directory doesn’t already exist. 

##### Rucio CLI    

```console
$ voms-proxy-init --cert .globus/usercert.pem --key .globus/userkey.pem --voms escape
```
Then, install rucio with pip. Get the available pip versions, which will likely be pip3:
```console
$ yum info python*-pip
$ yum install python3-pip 
```
And then run:
```console
$ pip3 install rucio-clients 
```
By typing in the terminal ‘rucio whoami’, you should see that the daemon is active. If not, you will be able to locate the rucio.cfg file and change the path to the .pem certificates. If you do not have a rucio.cfg file yet, create the directories /opt/rucio/etc/ and copy and paste the following into a new rucio.cfg file:

```console
# Authors:
# - Vincent Garonne, <vgaronne@gmail.com>, 2018

[client]
rucio_host = https://escape-rucio.cern.ch
auth_host = https://escape-rucio-auth.cern.ch
ca_cert = /etc/pki/tls/certs/CERN-bundle.pem
auth_type = x509
username =
password =
account = egazzarr
client_cert = .globus/usercert.pem
client_key = .globus/userkey.pem
client_x509_proxy =
request_retries = 3

[policy]
permission = generic
schema = generic
lfn2pfn_algorithm_default = hash
support = https://github.com/rucio/rucio/issues/
support_rucio = https://github.com/rucio/rucio/issues/
```

# Rucio RESTful APIs    

Interacting with the client environment can be done through the Docker container, or through the RestAPI. This is a way for developers to be able to integrate any kind of scripts into the environment without the need of installing all the Rucio CLI dependencies.

### Token generation

Now that you have the VOMS in place, you will just need to add your token, which will expire and will be regenerated every hour, to your Rest API request. 
To get the token, run the command:

```console
$ curl -i --key ~/.globus/userkey.pem --cert ~/.globus/usercert.pem -H "X-Rucio-Account: <myrucioaccount>" -X GET https://escape-rucio-auth.cern.ch/auth/x509 | sed -n -e 's/^.*X-Rucio-Auth-Token: //p'
```
The \< myrucioaccount \> instance needs to be replaced with your Rucio UI account name, the same as your IAM account (but you can also find it by navigating [here](https://escape-rucio-webui.cern.ch/) and selecting the ‘Account management’ option under the Admin scroll-down menu). 

*Note: Using curl with the --insecure option allows curl to make insecure connections (i.e. curl does not verify the certificate), bypassing: curl: (60) SSL certificate problem: self signed certificate in certificate chain. Another solution is proposed [here](https://escape-rucio-webui.cern.ch/).*

Create a variable by copying the token which has been generated and running in the command line:

```console
$ token="<your_token>"
```
Do not forget the “ symbol. 

After the token has been saved, you can run any Rest Api command, for example listing all the available RSEs or all the scopes:

```console
$ curl -X GET -H "X-Rucio-Auth-Token: $token" https://escape-rucio.cern.ch/rses/
$ curl -X GET -H "X-Rucio-Auth-Token: $token" https://escape-rucio.cern.ch/scopes/
```

### Proxy generation

The proxy generation with the x509 certificate works in the same way, the only difference is the command to get the toke: 

```console
$ curl -i --key ~/.globus/userkey.pem --cert ~/.globus/usercert.pem -H "X-Rucio-Account: <myrucioaccount>" -X GET https://escape-rucio-auth.cern.ch/auth/x509_proxy | sed -n -e 's/^.*X-Rucio-Auth-Token: //p'
```
Again, using curl with the --insecure option might be useful. Follow the same steps as above to make Rest API requests. 

To find all the Rucio REST API commands, navigate [here](https://rucio.readthedocs.io/en/latest/rest.html).

