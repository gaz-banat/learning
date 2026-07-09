
UNDERSTANDING KUBERNETES CONFIG/CONTEXT/AUTHENTICATION


Kubeconfig	- 	a listing of clusters, users (with authentication information) and contexts

Kubeconfig
	clusters:
		- name: <cluster-name>					a label to identify a cluster
		  cluster:
			server: <url>						the url of the cluster

	users:
		- name: <user-name>					usually USERNAME/CLUSTERNAME
		  user:
			token: <token>
			OR
			client-certificate-data: <cert>
			client-key-data: <pvt key for the cert>
			OR
			exec:

	contexts
		- context:
			cluster: <cluster-name>
			namespace: <namespace>
			user: <user-name>


Context		-	from contexts section of a kubeconfig file, identifies the cluster to target and the user to use, optionally specifies the namespace as well


Authentication	-	from user section of a kubeconfig file, identifies the authentication method for a user when talking to a kubernetes cluster	



PARAMETERS


API Authentication:
kubeconfig + context					-	the kubeconfig file location and the context to use when authenticating with the kubernetes API
username + password
certfile + keyfile
api_key							-	the token to use while authenticating


host
ca_cert/ssl_ca_cert

verify_ssl



KUBERNETES DYNAMIC INVENTORY PLUGIN

enable plugin in ansible.cfg and create an inventory.k8s.yml with the text ‘plugin: kubernetes.core.k8s’



MODULES

k8s								-	manage kubernetes objects
k8s_facts							-	get information on a kind/type of object
k8s_info (previously k8s_facts)			-	describe kubernetes objects
k8s_auth							-	can be used for OAuth2 in OpenShift
k8s_exec
k8s_log
k8s_scale
k8s_service




k8s Module

kubeconfig
context
ca_cert							-	path to ca cert file that can validate the kubeapiserver cert



k8s_auth Module




k8s_info Module



k8s_cluster_info Module
