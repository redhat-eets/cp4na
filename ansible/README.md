## General Notes
This folder represents the Ansible working directory for the IBM CP4NA project. All files in this folder are intended to be consumed through Ansible and its playbooks. 

Current capabilities that can be run using Ansible playbooks include:
- ODF operator installation including the local storage operator
- RHACM operator installation and multi cluster hub resource deployment
- OpenLDAP application deployment
- IBM CP4NA operator installation and instance creation	


## Pre-Requisites
- This exercise can be done by running Ansible within a container (in the case here, Podman was used) and making sure that Ansible's working directory is pointing towards the user's directory containing this folder (Ansible directory) acquired from the repo. For example, in the case here, the following command was used to run/start the ansible container:
`sudo podman run --name ansible --net=host --cap-add=NET_ADMIN -it -v $(pwd):/ansible/ -v ~/.kube:/root/.kube -v ~/.ssh/id_rsa:/root/id_rsa willhallonline/ansible:latest /bin/sh`
- A key note here is that the `.kube` folder or any other folder containing the OCP cluster (operators to be installed on this cluster) kubeconfig has to be mounted to the ansible container as well to be able to gain access to the OCP cluster intended to be used.
- Execute the `pre-req.yaml` playbook to install required ansible/python modules within the Ansible container by running the command `sudo podman exec <container-name> ansible-playbook pre-req.yaml` in the machine hosting the container. 


## Ansible Playbooks
This directory contains multiple playbooks that will execute tasks to initiate and complete ODF/ACM/CP4NA operator installation and corresponding instance creation. 

### Passing variables
These playbooks contain variables which can be tweaked by users to suit their needs. The intention is to configure Ansible to utilize dynamic values while installing and deploying operators,etc. These variables can be plugged in against the `vars` key as seen in the example below inside the playbook `acm_operator_install.yaml` for RHACM operator install:

```
      vars:
       ansible_work_dir: /ansible         #This variable points to the current ansible directory
       channel_version : release-2.4      #This variable sets the channel version to be followed while subscribing to the RHACM operator
       file_dest_dir: /home/acm_yaml_files  #This variable points to the temporary directory that will be created on the user's machine and where yaml manifest files/templates will be downloaded and consumed for use by Ansible
       role_name: acm_operator_install   #This variable points to the Ansible role to refer to.

```


- The `acm_operator_install.yaml` file is a playbook that installs the RHACM playbook.
- The `acm_multiclusterhub.yaml` playbook creates the `multiclusterhub` resource after the RHACM operator is installed.
- The `openshift_lso_install.yaml` playbook installs and deploys the local storage operator to meet a pre-requisite before installing the ODF operator.
- The `openshift_lso_volume_set.yaml` playbook creates a local volume set resource for the local storage operator.
- The `openshift_so_install.yaml` playbook installs and deploys the ODF operator.
- The `openshift_storage_system.yaml` playbook creates a storage system resource (needed before creating a storage cluster).
- The `openshift_storage_cluster.yaml` playbook creates a storage cluster.
- The `openshift_default_storage_class.yaml` playbook sets the default openshift storage class.
- The `openldap_playbook.yaml` playbook installs and deploys the OpenLDAP application needed for the IBM CP4NA operator instance.
- The `cp4na_operator_install.yaml` playbook installs and deploys the CP4NA operator.
- The `cp4na_instance.yaml` playbook deploys an orchestration instance of the CP4NA operator.



## General Flow
The order that is recommended is as follows:
- Run the playbooks intended for the ODF operator in the order below:
        - `openshift_lso_install.yaml` 
        - `openshift_lso_volume_set.yaml`
        - `openshift_so_install.yaml`
        - `openshift_storage_system.yaml` 
        - `openshift_storage_cluster.yaml`
        - `openshift_default_storage_class.yaml`

- Run the playbooks intended for the RHACM operator in the order below:
        - `acm_operator_install.yaml`
        - `acm_multiclusterhub.yaml`

- Run the playbook intended for the OpenLDAP application
        - `openldap_playbook.yaml`

- Run the playbooks intended for the IBM CP4NA operator in the order below:
        - `cp4na_operator_install.yaml`
        - `cp4na_instance.yaml`

