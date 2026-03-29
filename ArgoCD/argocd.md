

# ARGOCD COMPONENTS

ArgoCD Server				            -	    gRPC/REST server that does app mgmt & status reports
                                      app operations (sync, rollback, etc.), creds mgmt, auth & auth delegation, RBAC, listener/forwarder for Git Webhook events

Application Controller          -     responsible for reconciling the Application resource in Kubernetes synchronizing the desired application state (in Git) with the live state (in Kubernetes)
                                      also responsible for reconciling the Project resource.
ApplicationSet Controller       -     responsible for reconciling the ApplicationSet resource.
Repo Server					            -	    responsible for building kubernetes manifests from helm, oci or git repos.

Notification Controller
Dex Server


Redis HA Server                 -     used by Argo CD to provide a cache layer reducing requests sent to the Kube API as well as to the Git provider
Redis HA Proxy


# CONCEPTS


## Repository Support

ArgoCD supports 
- git
- Helm 
- OCI


## Clusters

a cluster is specified in a secret with a label of 'argocd.argoproj.io/secret-type: cluster' on the secret
the local argocd k8s cluster is not present as a secret



## ApplicationSets

ApplicationSet Controller/ApplicationSet -----> Application Controller/Application -------> Deployment/Service/ConfigMap/etc



## Generators

Generators are used to generate parameters. Thos parameters can then be used in an application template within the ApplicationSet resource

Types of Generators:
list								-	any type of key/value element, e.g. cluster: <name> and url: <url>
cluster								-	name, nameNormalized, server, metadata.labels.<key>, metadata.annotations.<key>
git
  directory generator
  file generator
matrix								-	this generator is used to combine the generated parameters of two separate generators
merge								-	this generator is used to merge the generated parameters of two or more generators
scm provider						-	this generator queries the API of an SCM provider to discover repositories within an organization
pull request						-	this generator uses the API of an SCMaaS provider to discover open pull requests within a repo
cluster decision resource			-	this generator is used to interface with K8s custom resources that use custom resource specific logic to decide which set of ArgoCD clusters to deploy to



# CONFIGURATION

## argoc-cm is the config map that helps with configuring the behaviour of the argocd application

kustomize.buildOptions: "--enable-helm"
resource.exclusions
application.instanceLabelKey

## Config Management Plugin (CMP)

This is the sidecar method

First you will need the specification of the plugin in a
argoproj.io/v1alpha1 ConfigManagementPlugin file


Argo CD expects the plugin configuration file to be located at /home/argocd/cmp-server/config/plugin.yaml in the sidecar container.
One way to do this is to put the ConfigManagementPlugin into the data of a ConfigMap and mount the configmap at the above location


Register the sidecar
To install a plugin, patch argocd-repo-server to run the plugin container as a sidecar, with argocd-cmp-server as its entrypoint
e.g. argocd-repo-server pod
containers:
- name: my-plugin
  command: [/var/run/argocd/argocd-cmp-server] # Entrypoint should be Argo CD lightweight CMP server i.e. argocd-cmp-server


Use the plugin in an application
Specify the spec.source.plugin.name field with the value being the name of the plugin

NOTE: there are other ways in which the plugin can also be used, e.g. discover.find or discover.fileName in the plugin spec


# ARGOCD SYNC PROCESS

## How a sync is done

- first the phase
- then the wave in the phase
- then by kind, e.g. namespace, network policy, resource quota, limit range, etc.
- then by name


## Sync Options in Applications

  syncPolicy:
    automated:
      prune: true                     # if a managed resource was removed from the source Git repo then remove that resource from the cluster
      selfHeal: true 				          # roll back any manual changes that were done on the application, e.g. using kubectl 
    syncOptions:
      - CreateNamespace=true
      - ApplyOutOfSyncOnly=true		    # apply only those resources that are out of sync between source and cluster (Selective Sync)
      - PruneLast=true 				        # prune this application last if this application was deployed as part of multiple applications
      - Replace=true 				          # the resources will be replaced on each sync (so last-applied-configuration is not avaialble on the resource), but note that applyoutofsynconly is true right now
      - FailOnSharedResource=true 	  # fail the sync if any resource is found on other applications 
	  - ServerSideApply=true 		        # uses kubectly apply --server-side=true to apply the resource, last-applied-configuration is not used, managedFields is used


## Sync Options in Resources via annotations 

argocd.argoproj.io/sync-options:Prune=false     # Argocd will not prune the resource even if deleted from Git
argocd.argoproj.io/sync-options:Validate=false  # Argocd will not validate this resource via kubectl before applying 
argocd.argoproj.io/sync-options:PruneLast=true  # Prune this application last if this application was deployed as part of multiple applications
argocd.argoproj.io/sync-options:Replace=true    # The resource will be replaced on each sync, it means that last-applied-configuration will not be available on the resource 






## Tracking strategies for GIT (targetRevision: )

Commit SHA			-	29c856a3
Tags				-	v1.2
Branch 				-	hotfix/IASP-9534
Symbolic Reference 	-	HEAD


## Tracking strategies for Helm (targetRevision: )

Specific version 	-  v1.2
Range 			 	- 	1.2.* 
Latest 			 	- 	*



## Diffing Customization

## Allows ArgoCD to ignore the difference between the source repo manifest and the applied manifest for specified fields.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  ...
spec:
  ...
  ignoreDifferences:
    - group: apps  																				# Note that api group is specified without versioning, e.g. apps rather than apps/v1 
      kind: Deployment
      # Use either
      jsonPointers:
      - /spec/replicas
      # or
      jqPathExpressions:
      - .spec.template.spec.initContainers[] | select(.name == "injected-init-container")
      # or 
      managedFieldsManagers:
      - kube-controller-manager
  ...
```


## Sync Phases and Hook Deletion Policies

## think of a hook as a kubernetes resource that has some kind of a hook annotation
## resources with hooks (aka resource hooks, or for that matter just hooks) will not run during a selectivesync

annotations:
  argocd.argoproj.io/hook: PreSync 		#	e.g database schema migration, normaly used with jobs
  argocd.argoproj.io/hook: Sync			#	e.g. Actual syncing of workloads
  argocd.argoproj.io/hook: PostSync		#	e.g. Sending notifications


annotations:
  argocd.argoproj.io/hook-delete-policy: HookSucceeded 			# Resource will be deleted after successful completion
  argocd.argoproj.io/hook-delete-policy: HookFailed				# Resource will be deleted if a sync hook failed
  argocd.argoproj.io/hook-delete-policy: BeforeHookCreation



## Sync Waves

for each phase, a wave is way to order how argocd applies the manifests
all manifests for a phase are in wave 0 by default

annotations:
  argocd.argoproj.io/sync-wave: "<num>"		# a number can be arbitary, argocd will read all annotations and start with the lowest number and then going to highest




# RBAC

Policies are defined using the following syntax:
p, <subject>, <resource>, <action>, <object>, <effect> (for permissions)
g, <subject>, <inherited-subject> (for role assignments/inheritance) 


        # https://github.com/argoproj/argo-cd/blob/master/docs/operator-manual/rbac.md
        roles:
          - name: <role_name>
            description: <description>
            policies:
              # Allow viewing resources
              - p, proj:dev-netflow-loaders:maintainer, applications, get, dev-netflow-loaders/*, allow
              # Allow syncing
              - p, proj:dev-netflow-loaders:maintainer, applications, sync, dev-netflow-loaders/*, allow
              # Allow restarts
              - p, proj:dev-netflow-loaders:maintainer, applications, delete/*/Pod/*, dev-netflow-loaders/*, allow
              - p, proj:dev-netflow-loaders:maintainer, applications, delete/batch/Job/*, dev-netflow-loaders/*, allow
              - p, proj:dev-netflow-loaders:maintainer, applications, action/apps/Deployment/restart, dev-netflow-loaders/*, allow 
              - p, proj:dev-netflow-loaders:maintainer, applications, action/apps/StatefulSet/restart, dev-netflow-loaders/*, allow 
              - p, proj:dev-netflow-loaders:maintainer, applications, action/apps/DaemonSet/restart, dev-netflow-loaders/*, allow
              # Allow viewing secrets from within ArgoCD
              - p, proj:dev-netflow-loaders:maintainer, secrets, get, dev-netflow-loaders/*, allow
              # Allow viewing logs
              - p, proj:dev-netflow-loaders:maintainer, logs, get, dev-netflow-loaders/*, allow
              # Allow executing into pods
              - p, proj:dev-netflow-loaders:maintainer, exec, create, dev-netflow-loaders/*, allow
              # Deny deleting of ArgoCD applications and namespaces (mostly for safety)
              - p, proj:dev-netflow-loaders:maintainer, applications, delete, dev-netflow-loaders/Namespace/*, deny
              - p, proj:dev-netflow-loaders:maintainer, applications, delete, dev-netflow-loaders/*, deny


Built-in Roles
Argo CD provides two default roles: 
role:readonly: Provides view-only access to all resources. This is often the default role assigned to all authenticated users.
role:admin: A superuser role with unrestricted access to the entire system. 



There is more to understand
https://git.aarnet.edu.au/SysAdmin/k8s/argocd/infra/argocd/-/blob/main/docs/argocd-api-access.md?ref_type=heads




# HOW TO

## Get ArgoCD to delete resources for an application when the application resource itself is deleted

add a finalizer on the application resource

```yaml
---
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: authelia-apps
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
...
```

## annotate an application to prevent argocd from syncing it
```shell
$ application=<application>
$ k annotate applications/$application argocd.argoproj.io/skip-reconcile="true" -n argocd
```


# COMMANDS

```shell
## Either Login to argocd
argocd login argocd.opsk8s.aarnet.net.au --grpc-web --insecure --username admin
## OR
# set your context to argocd namespace <---- IMPORTANT
# and just use argocd --core


## Show the argocd version
argocd --core version

## Show the argocd cluster
argocd --core cluster list

## List applications
kubectl get applications -n argocd
argocd --core app list

## Sync an application
argocd --core app sync <app>

## Generate a token for a project role
argocd --core proj role create-token <project> <role>
```