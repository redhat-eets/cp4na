## General Notes
These YAML files represent the installation files to setup/install OpenShift Data Foundation 4.9+ operator and its storage cluster instance on an existing OCP 4.x cluster. Certain storage parameters have been fine-tuned to suit the requirements of the IBM CP4NA operator. 

These files include the following:
- Namespace, operator group and subscription creation for the local storage operator.
- Worker node labelling for local volume discovery.
- Local volume discovery creation.
- Creation of local volume set.
- Namespace, operator group and subscription creation for the ODF operator.
- Creation of the storage system resource.
- Creation of the storage cluster resource.
- Default storage class setting.


## Ansible Implementation
These YAML files have been fine tuned to be referred and run within Ansible if the user decides to install the ODF operator using Ansible. Within these YAML files, there might be key values with the pattern `{{ variable_name }}` which are essentially values that are plugged in through Ansible. If the user wants to run and apply these files manually, replace the key value patters with appropriate values before running them. Users can also follow the installation guide [here](https://access.redhat.com/articles/5692201).


## Testing and Verification

Use the `cephfs-nginx-svc-test.yaml` , `rbd-nginx-svc-test.yaml` and `cephfs-nginx-svc-test-rwx.yaml` to create cluster IP services that use `ocs-storagecluster-cephfs` and `ocs-storagecluster-ceph-rbd` storage class to create pv and pvc to test out that ODF is working fine on the cluster.

Follow the below steps

 
1. `oc create -f cephfs-nginx-svc-test.yaml` 

    The above creates pvc, pod, deployment and svc resources; a pv resource is created as well to realize the pvc resource from `ocs-storagecluster-cephfs` storage class the resources will be created in the default namespace
    
2. `oc create -f rbd-nginx-svc-test.yaml`

    The above creates pvc, pod, deployment and svc resources; a pv resource is created as well to realize the pvc resource from `ocs-storagecluster-ceph-rbd` storage class the resources will be created in the default namespace
    
3. `oc create -f cephfs-nginx-svc-test-rwx.yaml`

    The above tests the RWX access mode

4. Execute `oc get svc` to ensure the services mentioned above are created

```
[openshift@host ~]$ oc get svc
NAME           TYPE           CLUSTER-IP       EXTERNAL-IP                            PORT(S)                       AGE
kubernetes     ClusterIP      172.30.0.1       <none>                                 443/TCP                       8d
nginx-cephfs   ClusterIP      172.30.232.167   <none>                                 80/TCP                        33m
nginx-rbd      ClusterIP      172.30.129.177   <none>                                 80/TCP                        35m
```

5. Execute `oc get ep` to ensure there are endpoints associated with the service resources created

```
[openshift@host ~]$ oc get ep
NAME           ENDPOINTS                                              AGE
kubernetes     10.19.111.28:6443,10.19.111.30:6443,10.19.111.8:6443   8d
nginx-cephfs   10.129.1.30:80                                         37m
nginx-rbd      10.129.1.28:80                                         39m
```

6. Execute `oc run curl-pod --image=radial/busyboxplus:curl -i --tty --rm` followed by `curl nginx-rbd` and `curl nginx-cephfs` to test out that the nginx services are reachable, this also tests out the PVs, PVCs are created and mounted successfully

```
[openshift@host ~]$ oc run curl-pod --image=radial/busyboxplus:curl -i --tty --rm
If you don't see a command prompt, try pressing enter.
[ root@curl-pod:/ ]$ curl nginx-rbd
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p><p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p><p><em>Thank you for using nginx.</em></p>
</body>
</html>

[ root@curl-pod:/ ]$ curl nginx-cephfs
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p><p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p><p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

7. Execute `oc delete -f cephfs-nginx-svc-test.yaml`, `oc delete -f rbd-nginx-svc-test.yaml` and `oc delete -f cephfs-nginx-svc-test-rwx.yaml` to delete the test services, PVs and PVCs.
