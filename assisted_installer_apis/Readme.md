# **OCP 4.9 Installation Experience on bare metal using Assisted Installer APIs and Postman Client**

## **Pre-requisites**

Certain pre-requisites that have to be met before proceeding with this venture are included in the following:-

* Users have to establish an account in [Red Hat Hybrid Cloud Platform](https://console.redhat.com/) before trying the cloud based Assisted Installer for OCP (be it either the GUI or APIs).

* Next, users must obtain access to the OpenShift Cluster Manager API token which can be obtained through the [platform console UI](https://console.redhat.com/openshift/token) or by using the [OpenShift Cluster Manager command-line tool](https://access.redhat.com/articles/6114701) `ocm-cli` and entering the command - `ocm token`.

> ![OCM API token](/Reference_Images/API-token.png "OCM API token")


>```
>[openshift@nfvpe-01  16:22:03 ~]$ ocm token
><Token output gets printed here>
>```
>Note:- [Github repo and guide for ocm-cli](https://github.com/openshift-online/ocm-cli)

* Users must obtain access to the pull secret associated with acquiring images which can be obtained through the [platform console UI](https://console.redhat.com/openshift/install/pull-secret) as well.

>![OCM pull secret](/Reference_Images/pull-secret.png "OCM pull secret")

* Users will have to obtain a set of machines/nodes that are to be part of the OCP cluster as master/worker nodes and a jump host which has direct access to them. Next, a public-private key pair for SSH must be generated in the host of choice that can access all the nodes and save the public key for AI API installation purposes.

* The Assisted Installer API [github repository](https://github.com/openshift/assisted-service) can be referred along with the [base URL](https://api.openshift.com/api/assisted-install/v2/) or API endpoint for all the APIs.

* The Postman API platform or any other API client can be installed to examine the AI APIs from the swagger spec. Using Postman, the user can export the [swagger specification for the AI APIs](https://github.com/openshift/assisted-service/blob/master/swagger.yaml)  and create a collection of these API requests. Later, they can be modified to serve the user’s objectives. Else, the user can individually pick specific API requests from the swagger spec (formatting the spec for easier visibility using the Swagger Editor might be a good option) and then create a new collection within a new workspace to work on.

## **Steps**

The following steps are specific to using Postman as the API client.

### **Step 1**
The user can set environment variables which will be used when API requests are done by setting them in Postman by creating a new environment for the AI APIs.

>![Postman env for AI APIs](/Reference_Images/env-variables-postman.png "Environment variables for AI APIs set on Postman")

Environment variables for the pull secret, ocm token and ssh public key can have values that the user obtained as part of the pre-requisites.

### **Step 2**
A list of API requests relevant to installing OCP 4.9 can be found in the collection.json file in the [CP4NA repository](https://github.com/redhat-eets/cp4na/blob/main/assisted_installer_apis/AssistedInstaller_APIs.postman_collection.json). If this is imported into Postman from the mentioned file, it can be viewed like the following screenshot.

>![Postman AI APIs collection](/Reference_Images/Postman-APIs.png "Postman AI APIs collection")

The above API requests can also be manually added to the collection for execution by getting the request URLs and relevant environment variables obtained in the steps that follow.
Authorization has to be from the API token for all the API requests by setting the authorization type to API key and then setting the value to the API token stored as the environment variable earlier.

>![OCM API token in Postman](/Reference_Images/token-in-postman.png "OCM API token in Postman")

### **Step 3**
Use the [Create Cluster Definition Request](#create-cluster-definition-request) to start creating the OCP cluster. This will create the initial framework of the cluster along with needed information. Once the cluster definition is ready from the previous step, the user can verify if the cluster has come up by trying out the [List Cluster Request](#list-cluster-request) and seeing if the response has the intended cluster definition.

### **Step 4**
Next, the user can create a discovery ISO/infra-env for the cluster definition that was created earlier as part of Step 3 to boot up machines/nodes to be associated with the cluster. This can be done by executing the [Create Infrastructure Environment Request](#create-infrastructure-environment-request)(Get Discovery ISO). The user can then verify if the infra-env was created by executing the [List Infrastructure Environment Request](#list-infrastructure-environment-request) and seeing if the response has the infra-env.

### **Step 5**
Next, the user can get the download URL to the discovery ISO that was created in the previous step by executing the [Get Discovery Image Download URL Request](#get-discovery-image-download-url-request). The response will contain the URL which can be directly accessed in a browser to download the ISO image or it can be done by using the wget command.

### **Step 6**
The discovery ISO that was obtained earlier can be now used to boot hosts that are to be a part of the cluster. To see the status of the hosts that get discovered after booting with the ISO, the user can run the [Get Hosts Request](#get-hosts-request) and check the response. If hosts do not turn up in the response, they are either not discovered yet or the booting with the ISO did not succeed. 

### **Step 7**
Now that the hosts have been discovered, the user can proceed to start and complete OCP cluster installation by running the [Install Cluster Request](#install-cluster-request). Users can check the status of the installation and what’s happening behind the scenes by periodically checking the response of the [Get Cluster Events Request](#get-cluster-events-request).


### **Step 8**
The user can retrieve cluster admin credentials by running the [Get Cluster Credentials Request](#get-cluster-credentials-request). Additionally, users can also get credentials like the cluster kubeconfig by running the [Download Cluster Credentials Request](#download-cluster-credentials-request) and checking the response.

## **Installation Notes**
The cluster installation can be done using the Assisted Installer APIs by following the above steps or running the appropriate requests in the correct sequence. Using any API client software platform, requests can be sent to the AI API base URL endpoint.

Users can make use of the Swagger Editor online to visualize all the AI APIs by importing the swagger spec in the [editor](https://editor.swagger.io/).

The following requests are the primary AI API requests used for the OCP cluster’s installation. Users can probe the request URL, plug in path values or query parameters and provide a request body to get intended request responses.




## Relevant AI API requests

### **List Cluster Request**
Retrieves the list of OpenShift clusters installed using the Assisted Installer.
|                |                        |
|----------------|-------------------------------|
|Request Type          |GET|
|Request URL          |[https://api.openshift.com/api/assisted-install/v2/clusters](https://api.openshift.com/api/assisted-install/v2/clusters)|
|Request Body         |None|

### **Create Cluster Definition Request**
This request creates a new cluster definition which in effect is the first step in creating an OCP cluster. In the AI UI, it will be seen as a draft after this request is done.(NOTE: below is example of creating HA cluster, for SNO clusters include `"high_availability_mode": "None"` in the Request Body.)
|                |                        |
|----------------|-------------------------------|
|Request Type          |POST|
|Request URL          |https://api.openshift.com/api/assisted-install/v2/clusters|
|Request Body         |`{"name":"ocp4ai", "openshift_version":"{{openshift_version}}", "base_dns_domain":"oot.lab.eng.bos.redhat.com","ssh_public_key":"{{ssh_public_key}}","pull_secret":"{{pull_secret}}","machine_networks":[{"cidr":"192.168.40.0/24"}]}`|

Here, the variables highlighted in orange represent environment variables that will be used by the API request if they were set as per Step 1 earlier.

|    Variable Name            |        Description                |
|----------------------|-------------------------------|
|name       |Cluster name to be used.|
|openshift_version        |Openshift version to be installed on the cluster.|
|base_dns_domain        |The base domain of your cloud provider. The base domain is used to create routes to your OpenShift Container Platform cluster components. The full DNS name for your cluster is a combination of the baseDomain and metadata.name parameter values that uses the `<metadata.name>.<baseDomain>` format. A fully-qualified domain or subdomain name, such as `example.com`.|
|ssh_public_key    |The public key of the jump host that was generated earlier.|
|pull_secret|The pull secret for images that was obtained earlier.|
|machine_networks:cidr    |The network subnet for the cluster to use. [Make sure this network is up and running by using a DHCP server or any other method]|

### Create Infrastructure Environment Request
This request creates a new OpenShift Discovery ISO/infra-env

|                |                        |
|----------------|-------------------------------|
|Request Type          |POST|
|Request URL          |https://api.openshift.com/api/assisted-install/v2/infra-envs|
|Request Body         |`{"name":"{{clustername}}", "image_type":"minimal-iso", "openshift_version":"{{openshift_version}}", "pull_secret": "{{pull_secret}}","ssh_authorized_key":"{{ssh_public_key}}","cluster_id":"{{clusterid}}"}`|

The body variables are as follows:

|    Variable Name            |        Description                |
|----------------------|-------------------------------|
|name       |Cluster name to be used.|
|openshift_version        |Openshift version to be installed on the cluster.|
|image_type       |Type of image to be used (Minimal or Full)|
|ssh_authorized_key    |The public key of the jump host that was generated earlier.|
|pull_secret|The pull secret for images that was obtained earlier.|
|cluster_id   |Cluster ID that was generated as part of the create cluster definition request|

### **List Infrastructure Environment Request**
This request lists the OpenShift Discovery ISOs/infra-envs that were created.

|                |                        |
|----------------|-------------------------------|
|Request Type          |GET|
|Request URL          |https://api.openshift.com/api/assisted-install/v2/infra-envs|
|Request Body         |None|


### **Get Discovery Image Download URL Request**
This request creates and retrieves a new pre-signed image download URL for the infra-env/discovery ISO image.

|                |                        |
|----------------|-------------------------------|
|Request Type          |GET|
|Request URL          |[https://api.openshift.com/api/assisted-install/v2/infra-envs/{{infra_env_id}}/downloads/image-url]()|
|Request Body         |None|

Request URL Path variables:

|    Variable Name            |        Description                |
|----------------------|-------------------------------|
|infra_env_id     |The unique value that was generated while creating the discovery ISO image/infra-env|


### Get Hosts Request
This request retrieves the list of OpenShift hosts that belong to the infra-env or the hosts that are to be booted with the discovery ISO.

|                |                        |
|----------------|-------------------------------|
|Request Type          |GET|
|Request URL          |[https://api.openshift.com/api/assisted-install/v2/infra-envs/{{infra_env_id}}/hosts]()|
|Request Body         |None|

Request URL Path variables:

|    Variable Name            |        Description                |
|----------------------|-------------------------------|
|infra_env_id     |The unique value that was generated while creating the discovery ISO image/infra-env|



### **Install Cluster Request**
This request proceeds to initiate the OCP installation over the cluster definition that should have been already created earlier and complete it.

|                |                        |
|----------------|-------------------------------|
|Request Type          |POST|
|Request URL          |[https://api.openshift.com/api/assisted-install/v2/infra-envs/{{cluster_id}}/actions/install]()|
|Request Body         |None|

Request URL Path variables:

|    Variable Name            |        Description                |
|----------------------|-------------------------------|
|cluster_id    |Cluster ID that was generated as part of the create cluster definition request|


### Get Cluster Events Request
This request lists the events for the given cluster.


|                |                        |
|----------------|-------------------------------|
|Request Type          |GET|
|Request URL          |[https://api.openshift.com/api/assisted-install/v2/events?cluster_id={{cluster_id}}&infra_env_id={{infra_env_id}}]()|
|Request Body         |None|

Request URL Query parameters:

|    Variable Name            |        Description                |
|----------------------|-------------------------------|
|cluster_id    |Cluster ID that was generated as part of the create cluster definition request|
|infra_env_id     |The unique value that was generated while creating the discovery ISO 


### **Get Cluster Credentials Request**

This request allows the user to retrieve the cluster admin credentials to access the cluster console once the OCP cluster installation successfully completes.

|                |                        |
|----------------|-------------------------------|
|Request Type          |GET|
|Request URL          |[https://api.openshift.com/api/assisted-install/v2/clusters/{{clusterid}}/credentials](https://api.openshift.com/api/assisted-install/v2/clusters/%7B%7Bclusterid%7D%7D/credentials)|
|Request Body         |None|

Request URL Path variables:

|    Variable Name            |        Description                |
|----------------------|-------------------------------|
|cluster_id    |Cluster ID that was generated as part of the create cluster definition request|

### **Download Cluster Credentials Request**
This request retrieves the cluster credentials for the installed OCP cluster based on the query parameters that are provided.

|                |                        |
|----------------|-------------------------------|
|Request Type          |GET|
|Request URL          |[https://api.openshift.com/api/assisted-install/v2/clusters/{{clusterid}}/downloads/credentials?file_name={{any-query-parameter-value}}]()|
|Request Body         |None|

Request URL Query Parameters:
> file_name
> 
It can be any of the following :-
|                |                        |
|----------------|-------------------------------|
|kubeadmin-password          |Retrieves cluster admin password for cluster console|
|kubeconfig        |Retrieves cluster kubeconfig|
|kubeconfig-noingress        |Retrieves cluster kubeconfig (the noingress version) 































