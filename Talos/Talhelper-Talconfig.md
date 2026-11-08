
# TALHELPER and TALCONFIG


To construct a talhelper 

1. You need to know the talhelper configuration reference
https://budimanjojo.github.io/talhelper/latest/reference/configuration/


2. Provide talhelper with these areas of information in talconfig.yaml

```yaml
clusterName
talosVersion
kubernetesVersion
endpoint
domain
additionalMachineCertSans
additionalApiServerCertSans
cniConfig
patches: a strategicmerge patch to v1alpha1 document OR a full document of any kind conforming to apiVersion: v1alpha1

+

# Common controlPlane configuration
controlPlane:
    ...
    patches: a strategicmerge patch to v1alpha1 document OR a full document of any kind conforming to apiVersion: v1alpha1
    # DEPRECATED extraManifests: Any manifests in v1alpha1/<specification> style

+

# Common worker configuration
worker:
    ...
    patches: a strategicmerge patch to v1alpha1 document OR a full document of any kind conforming to apiVersion: v1alpha1
    # DEPRECATED extraManifests: Any manifests in v1alpha1/<specification> style

+

# Individual Node Configuration
nodes:
    - hostname
      ...
      patches: a strategicmerge patch to v1alpha1 document OR a full document of any kind conforming to apiVersion: v1alpha1
      # DEPRECATED extraManifests: Any manifests in v1alpha1/<specification> style


    - hostname
```