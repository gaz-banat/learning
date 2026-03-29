
# TALHELPER and TALCONFIG


To construct a talhelper 

1. You need to know the talhelper configuration reference
https://budimanjojo.github.io/talhelper/latest/reference/configuration/


2. Provide talhelper with these areas of information

clusterName
talosVersion
endpoint
domain
additionalMachineCertSans
additionalApiServerSans
cniConfig
patches: Any patches directly in v1alpha1 style applying to all nodes

+

Common controlPlane configuration
    ...
    patches: directly in v1alpha1 style
    extraManifests: Any manifests in v1alpha1/<specification> style

+

Common worker configuration
    ...
    patches: directly in v1alpha1 style
    extraManifests: Any manifests in v1alpha1/<specification> style

+

Nodes
    - node
      ...
      patches: directly in v1alpha1 style
      extraManifests: Any manifests in v1alpha1/<specification> style






# TALOSCTL and NODE CONFIGURATION FILES

a node configuration file is always made up of Talos Configuration manifests

v1alpha1
v1alpha1/ExtensionServiceConfig
v1alpha1/TrustedRootsConfig
v1alpha1/NetworkDefaultActionConfig
v1alpha1/NetworkRuleConfig
and so on