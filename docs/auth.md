# Authentication

The interaction with the [Data Lake](https://gitlab.cern.ch/escape-wp2/datalake-as-a-service-docs/-/tree/master/base) requires authentication.
It is required to register to the INDIGO Identity and Access Management ([IAM](https://indigo-iam.github.io/v/v1.7.1/docs/overview/)) service and obtain a X.509 Grid certificate. Follow the instructions in the next sections.

### ESCAPE IAM registration

Navigate [here](https://iam-escape.cloud.cnaf.infn.it/login) and register for a new account, filling in the required details.
After the account has been approved, you will need to log in and add the Escape group to your Groups. To do this, click on the green **Join a group** botton in the **Group requests** panel. Enter 'escape' in the group box, adding a brief motivation for the request. You will need to wait in order for this to be approved.

### X.509 Grid certificate    

#### *Getting the certificate*    

The following procedure is described for a UK CA/RA, but it is different depending on the country you are located in. Check this [link](https://www.eugridpma.org/members/worldmap/) to see what you need.  

An X.509 grid certificate is sought by first contacting the appropriate certificate authority (CA). This can be done [here](https://portal.ca.grid-support.ac.uk/). Make a note of both the pin and key password, as they will be required later.

After your request, you will receive an email asking for further information to be passed to the registration authority (RA). 
You will need some form of I.D. (e.g. site I.D., driver’s licence, passport) in addition to the pin that was entered in the request to the CA. You will need to pass these to one of the contacts listed in the email, as instructed, who will likely want to arrange a brief video call to confirm your identity against your provided documents.

*For **CERN** staff members, the request can be made [here](https://ca.cern.ch/ca/), under the ‘Grid Certificates’ section with ‘New Grid User Certificate’.* 

Once your identity has been confirmed, you will be emailed a link to download your certificate in .p12 format (PKCS 12). Download the file and store it in a safe place on your computer, where it is always easily accessible. 

#### *Uploading the certificate* 

Before linking the Grid certificate to your IAM account, you will need to add the certificate to your browser. The procedure you will need to follow varies depending on what browser you are using. For Chrome, this can be done by navigating to chrome://settings/security and following the instructions presented upon selecting Advanced > Manage Certificates > Import.
Once the certificate has been imported, log into the ESCAPE IAM and select the green **Link certificate** botton in the **X.509 certificates** panel. 

#### *Changing the certificate* 

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

