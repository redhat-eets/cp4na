This directory contains the yaml files to deploy:

	a) the IBM CP4NA operator 
	
	b) create an instance of the operator 
	
	c) edit the orchestration subscription from Automatic to Manual

Steps to create orchestration instance

1. First edit each of the three yaml files by replacing the pattern `{{ some_var_name }}` with actual valid values. (e.g; replace instances of `{{ cp4na_ns }}` occurences with `ibmcp4na` )

2. `oc create -f https://raw.githubusercontent.com/redhat-eets/cp4na/main/ibm_orchestrator/01-cp4na-operator-deploy.yaml`.

    The above successfuly installs the CP4NA operator. The above creates a namespace, ldap secret, ibm-entitlment-key secret, service account, catalog sources, operator group and subscription    resource objects. Wait for all resources to be up and running before proceeding to the next step. Below are the variables that you will need to substitute in the yaml file as an example

	
```
	
	{{ channel_version }}: v2.2

        {{ cp4na_ns }}: ibmcp4na

        # Run the command `oc create secret docker-registry ibm-entitlement-key --docker-username=cp --docker-password=<entitlement-key> --docker-server=cp.icr.io --namespace=ibmcp4na --dry-run=client -o yaml' and input the data[.dockerconfigjson] field value from resulting output below. Note: below cred is for illustration only and is invalid

        {{ ibm_key_credjson }}: eyJhdXRocyuaW8iOnsidXNlcm5hbWUiOiJjcCIsInBhc3N3b3JkIjoiZXlKaGJHY2lPaUpJVXpJMU5pSjkuZXlKcGMzTWlPaUpKUWswZ1RXRnlhMlYwY0d4aFkyVWlMQ0pwWVhRaU9qRTJNakV5TlRjNE5UTXNJbXAwYVNJNklqQTFNall3T0dJeVpqa3hNelExTVdKaFpUTXpaREUwWlRRMk16UTNNek16SW4wLk9ieU9Ga3VRMXRUMVNoLV84emxCNkFXbjlYTmJqdE5fUzMyZXdhTk8zOW8iLCJhdXRoIjoiWTNBNlpYbEthR0pIWTJsUGFVcEpWWHBKTVU1cFNqa3VaWGxLY0dNelRXbFBhVXBLVVdzd1oxUlhSbmxoTWxZd1kwZDRhRmt5VldsTVEwcHdXVmhSYVU5cVJUSk5ha1Y1VGxSak5FNVVUWE5KYlhBd1lWTkpOa2xxUVRGTmFsbDNUMGRKZVZwcWEzaE5lbEV4VFZkS2FGcFVUWHBhUkVVd1dsUlJNazE2VVROTmVrMTZTVzR3TGs5aWVVOUdhM1ZSTVhSVU1WTm9MVjg0ZW14Q05rRlaFRrOHpPVzg9In19fQ==

        {{ ldap_admin_password }}: adminpassword

        {{ ldap_base_dn }}: dc=alm,dc=com

        {{ ldap_bind_dn }}: cn=admin,dc=alm,dc=com

        {{ ldap_password_attr }}: userPassword

        {{ ldap_url }}: ldap://openldap.default.svc.cluster.local:389

        {{ ldap_user_search_base }}: ou=people

        {{ ldap_user_search_filter }}: uid={0}
	
```

3. `oc create -f https://raw.githubusercontent.com/redhat-eets/cp4na/main/ibm_orchestrator/02-cp4na-instance.yaml`.

   The above creates an instance of the IBM CP4NA operator and takes about 45-75 mins to complete. Below are variables that you will need to substitute in the yaml files as an example

```
  	{{ cp4na_ns }}: ibmcp4na

        {{ ver }}: 2.2.2
```

4. `oc apply -f https://github.com/redhat-eets/cp4na/blob/main/ibm_orchestrator/03-edit-subscription-manual.yaml`.Below are variables that you will need to substitute in the yaml files as an example

```
	{{ cp4na_ns }}: ibmcp4na

	{{ ip_approval_method }}: Manual
```

References:

- https://www.ibm.com/docs/en/cloud-paks/cp-network-auto/2.2.x?topic=installing-cloud-pak

(Note: The yaml files are structured to be consumed by ansible playbooks as well)
