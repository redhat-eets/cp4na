This directory contains the YAML files that can be used to install Red Hat Advanced Cluster Management for Kubernetes (RHACM) hub cluster and to create the `MultiClusterHub` custom resource. These YAML files have been modified to be consumed by Ansible playbooks.
(These YAML files can be template files where values are plugged in through Ansible). These files are automatically referred and run within Ansible if the user decides to do so to install RHACM.


## Pre-Requisites:
- Red Hat OpenShift Container Platform cluster must be installed.
- OpenShift Container Platform CLI must be installed.
- No previous installations of RHACM should be present. 



## Steps to install RHACM and its custom instance manually:
1.  The first YAML file `01-acm-operator-install.yaml` is to create a custom RHACM hub cluster namespace (which is `open-cluster-management` in the case here), to create an `OperatorGroup` resource for RHACM and to create and configure a subscription for the RHACM operator. This file is referred and run within Ansible if the user decides to do so to install RHACM.
- A key note here is that this YAML file is a template to be used by Ansible to fill in the RHACM subscription channel version (you will be able to see the template variables in curly braces). The RHACM subscription manifest section has a `channel` key which includes a placeholder `{{ channel_version }}` for the channel version. To run the command manually without Ansible, replace the `channel` key value with an actual value and then run the command `oc create -f https://raw.githubusercontent.com/redhat-eets/cp4na/main/acm/2.4/01-namespace-acm.yaml`.
- The YAML files that follow have been modified to use the `open-cluster-management` namespace as the namespace to be worked on. 

2. Run the command `oc create -f https://github.com/redhat-eets/cp4na/blob/main/acm/2.4/02-multiclusterhub-instance-acm.yaml` to create and configure the `MultiClusterHub` custom resource.

3. Once the custom resource is installed, run the command `oc get mch -o=jsonpath='{.items[0].status.phase}' -n open-cluster-management` to check the status of the custom resource. It has to display `Running` as the status.

4. To acquire the link to the RHACM hub cluster console, run the command `oc get routes -n open-cluster-management` which would look like `multicloud-console.apps.ocp4ai.oot.lab.eng.bos.redhat.com` for example.

5. Add the link in the `/etc/hosts` file as a hostname/DNS entry with the same IP address mapped to the hosting OCP cluster. Now, the link should be directly accessible.


## Reference
- [RHACM installation guide](https://access.redhat.com/documentation/en-us/red_hat_advanced_cluster_management_for_kubernetes/2.0/html/install/installing#installing-red-hat-advanced-cluster-management-from-the-cli)
