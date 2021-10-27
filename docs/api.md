# Rucio RESTful APIs    

Interacting with the client environment can be done through the Docker container, or through the RestAPI. This is a way for developers to be able to integrate any kind of scripts into the environment without the need of installing all the Rucio CLI dependencies.

#### *Token generation* 

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

#### *Proxy generation*

The proxy generation with the x509 certificate works in the same way, the only difference is the command to get the toke: 

```console
$ curl -i --key ~/.globus/userkey.pem --cert ~/.globus/usercert.pem -H "X-Rucio-Account: <myrucioaccount>" -X GET https://escape-rucio-auth.cern.ch/auth/x509_proxy | sed -n -e 's/^.*X-Rucio-Auth-Token: //p'
```
Again, using curl with the --insecure option might be useful. Follow the same steps as above to make Rest API requests. 

To find all the Rucio REST API commands, navigate [here](https://rucio.readthedocs.io/en/latest/rest.html).

