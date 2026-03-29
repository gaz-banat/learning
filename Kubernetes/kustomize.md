
# HEADING FOR KUSTOMIZE FILE

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
```


# ORDER OF OPERATIONS

NOTE: https://github.com/kubernetes-sigs/kustomize/blob/master/kustomize/commands/internal/kustfile/kustomizationfile.go#L41-L73

ordered := []string{
		"MetaData",
		"SortOptions",
		"Resources",
		"Bases",
		"NamePrefix",
		"NameSuffix",
		"Namespace",
		"Crds",
		"CommonLabels",
		"Labels",
		"CommonAnnotations",
		"PatchesStrategicMerge",
		"PatchesJson6902",
		"Patches",
		"ConfigMapGenerator",
		"SecretGenerator",
		"HelmCharts",
		"HelmChartInflationGenerator",
		"HelmGlobals",
		"GeneratorOptions",
		"Vars",
		"Images",
		"Replacements",
		"Replicas",
		"Configurations",
		"Generators",
		"Transformers",
		"Validators",
		"Components",
		"OpenAPI",
		"BuildMetadata",
	}



# RESOURCES

resources:
  - /path/to/dir                # a kustomization file should exist in the directory
  - /path/to/file




# COMPONENTS

## First produce a component file with name kustomization.yaml in a directory, say /path/component1/kustomization.yaml

```yaml
---
# This is what a component file looks like

## first declare the component header
apiVersion: kustomize.config.k8s.io/v1alpha1
kind: Component

## then any valid kustomize directive
resources:
- resources/file1
- resources/file2

patchesJson6902:  
- target:
    group: <api group>
    version: <version of api group>
    kind: <kind of resource>
    name: <name of resource>
  path: <path/to/patch/file>
```


## Ensure that there is a resources/file1 and resources/file2 in /path/components1 as required by the kustomization.yaml

/path/component1/resource/file1
/path/component1/resource/file2


## Then include the component in your main kustomization file

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

components:
  - path/component1       
```




# HELMCHARTS






# GENERATORS

## Generators field

generators:
  - path/to/generator/file
  - path/to/another/generator/file

(and then use one of these generators in its own file - SecretGenerator, ConfigMapGenerator, HelmChartInflationGenerator)


```yaml
# this is what a generator for helmchartinflationgenerator looks like
---
apiVersion: builtin
kind: HelmChartInflationGenerator
metadata:
  name: kps
repo: oci://aplregistry.aarnet.edu.au/helm-charts/prometheus-community.github.io/helm-charts/kube-prometheus-stack
name: kube-prometheus-stack
includeCRDs: false
releaseName: kps
version: 67.4.0 # chart version, not app version
valuesFile: values.opsk8stest.yam

# this is what a generator for confimapgenerator looks like
---
apiVersion: builtin
kind: ConfigMapGenerator
metadata:
  name: kps-grafana
  namespace: monitoring
behavior: replace
files:
  - resources/configmaps/grafana/grafana.ini
options:
  disableNameSuffixHash: true
```

generatorOptions:

  
## Convenience fields

secretGenerator:

configMapGenerator:
  - name: grafana-dashboards-infra
    files:
      - resources/grafana-dashboards/infrastructure/ceph-cluster-dashboard.json
      - resources/grafana-dashboards/infrastructure/ceph-osd-single-dashboard.json
      - resources/grafana-dashboards/infrastructure/ceph-pools-dashboard.json
    options:
      disableNameSuffixHash: true
      labels:
        aarnet.net.au/grafana-dashboard: "true"
      annotations:
        aarnet.net.au/grafana-dashboard-folder: "Infrastructure"

        
helmCharts:

helmGlobals:




# TRANSFORMERS

## Transformers field

transformers:
  - path/to/transformer/file
  - path/to/another/transformer/file

(and then use a tranfsformer in its own file - NamespaceTransformer, PrefixSuffixTransformer, LabelTransformer)


## Convenience fields

commonAnnotations:

replicas:         # convenience field for number of replicas


images generally used in an overlay to overwrite a base value 
images:
  - name: lissy93/dashy
    newName: aplregistry.aarnet.edu.au/hub.docker.com/lissy93/dashy
    newTag: 3.1.0


labels:           # will update only labels, optionally update selectors (in deployments and services) and templates (pods in deployment, sts, ds, etc)

commonLabels:     # will update labels and selectors                      DEPRECATED

namePrefix:       # convenience field for PrefixSuffixTransformer

nameSuffix:       # convenience field for PrefixSuffixTransformer

namespace:




# VALIDATORS




# PATCHING

## Patches field

patches:                  
- path: <path/to/file>                  # strategic merge ($patch = replace, delete, etc.) 

- target:                               # JSON6902 (op = add, replace, remove, move, copy, test)  
    group: <api group>
    version: <version of api group>
    kind: <kind of resource>
    name: <name of resource>
  path: <path/to/patch/file>

- target:                               # this seems to work for strategic merge but for some reason this did not 'quite' work for JSON6902, i.e. to supply the patch content inline
  patch: |
    <patch content>


## Convenience fields

patchesStrategicMerge:                  # convenience field for strategic merge patch   DEPRECATED
- <path/to/patch/file>                  # no need to specify target field, target is identified from the patch file itself


patchesJson6902:                        # convenience field for Json6902 patch          DEPRECATED
- target:
    group: <api group>
    version: <version of api group>
    kind: <kind of resource>
    name: <name of resource>
  path: <path/to/patch/file>
  OR
  patch: |-
    <patch content>


```yaml
---
## This is what a Strategic Merge patch file looks like
$patch: delete
apiVersion: v1
kind: Secret
metadata:
  name: dex-config-test-no-create


---
## This is what a JSON6902 patch file looks like
- op: replace
  path: /spec/dataFrom/0/extract/key
  value: k8s/lab/traefik/dashboard
```



# CONFIGURATIONS







# KUSTOMIZE CLI

build
completion    - generate shell completion
cfg           - read and write configuration
create        - create a new kustomization
edit          - edit a kustomize file
fn            - for running functions against configurations 





```shell
kustomize build --enable-helm --load-restrictor=LoadRestrictionsNone


kustomize build infra/overlays/opsk8stest --enable-helm \
--helm-kube-version 1.34 \
--helm-api-versions access.strimzi.io/v1alpha1 --helm-api-versions access.strimzi.io/v1alpha1/KafkaAccess --helm-api-versions acme.cert-manager.io/v1 --helm-api-versions acme.cert-manager.io/v1/Challenge --helm-api-versions acme.cert-manager.io/v1/Order --helm-api-versions admissionregistration.k8s.io/v1 --helm-api-versions admissionregistration.k8s.io/v1/MutatingWebhookConfiguration --helm-api-versions admissionregistration.k8s.io/v1/ValidatingAdmissionPolicy --helm-api-versions admissionregistration.k8s.io/v1/ValidatingAdmissionPolicyBinding --helm-api-versions admissionregistration.k8s.io/v1/ValidatingWebhookConfiguration --helm-api-versions apiextensions.k8s.io/v1 --helm-api-versions apiextensions.k8s.io/v1/CustomResourceDefinition --helm-api-versions apiregistration.k8s.io/v1 --helm-api-versions apiregistration.k8s.io/v1/APIService --helm-api-versions apps/v1 --helm-api-versions apps/v1/ControllerRevision --helm-api-versions apps/v1/DaemonSet --helm-api-versions apps/v1/Deployment --helm-api-versions apps/v1/ReplicaSet --helm-api-versions apps/v1/StatefulSet --helm-api-versions aquasecurity.github.io/v1alpha1 --helm-api-versions aquasecurity.github.io/v1alpha1/ClusterComplianceReport --helm-api-versions aquasecurity.github.io/v1alpha1/ClusterConfigAuditReport --helm-api-versions aquasecurity.github.io/v1alpha1/ClusterInfraAssessmentReport --helm-api-versions aquasecurity.github.io/v1alpha1/ClusterRbacAssessmentReport --helm-api-versions aquasecurity.github.io/v1alpha1/ClusterSbomReport --helm-api-versions aquasecurity.github.io/v1alpha1/ClusterVulnerabilityReport --helm-api-versions aquasecurity.github.io/v1alpha1/ConfigAuditReport --helm-api-versions aquasecurity.github.io/v1alpha1/ExposedSecretReport --helm-api-versions aquasecurity.github.io/v1alpha1/InfraAssessmentReport --helm-api-versions aquasecurity.github.io/v1alpha1/RbacAssessmentReport --helm-api-versions aquasecurity.github.io/v1alpha1/SbomReport --helm-api-versions aquasecurity.github.io/v1alpha1/VulnerabilityReport --helm-api-versions argoproj.io/v1alpha1 --helm-api-versions argoproj.io/v1alpha1/AppProject --helm-api-versions argoproj.io/v1alpha1/Application --helm-api-versions argoproj.io/v1alpha1/ApplicationSet --helm-api-versions autoscaling/v1 --helm-api-versions autoscaling/v1/HorizontalPodAutoscaler --helm-api-versions autoscaling/v2 --helm-api-versions autoscaling/v2/HorizontalPodAutoscaler --helm-api-versions barmancloud.cnpg.io/v1 --helm-api-versions barmancloud.cnpg.io/v1/ObjectStore --helm-api-versions batch/v1 --helm-api-versions batch/v1/CronJob --helm-api-versions batch/v1/Job --helm-api-versions ceph.rook.io/v1 --helm-api-versions ceph.rook.io/v1/CephBlockPool --helm-api-versions ceph.rook.io/v1/CephBlockPoolRadosNamespace --helm-api-versions ceph.rook.io/v1/CephBucketNotification --helm-api-versions ceph.rook.io/v1/CephBucketTopic --helm-api-versions ceph.rook.io/v1/CephCOSIDriver --helm-api-versions ceph.rook.io/v1/CephClient --helm-api-versions ceph.rook.io/v1/CephCluster --helm-api-versions ceph.rook.io/v1/CephFilesystem --helm-api-versions ceph.rook.io/v1/CephFilesystemMirror --helm-api-versions ceph.rook.io/v1/CephFilesystemSubVolumeGroup --helm-api-versions ceph.rook.io/v1/CephNFS --helm-api-versions ceph.rook.io/v1/CephObjectRealm --helm-api-versions ceph.rook.io/v1/CephObjectStore --helm-api-versions ceph.rook.io/v1/CephObjectStoreUser --helm-api-versions ceph.rook.io/v1/CephObjectZone --helm-api-versions ceph.rook.io/v1/CephObjectZoneGroup --helm-api-versions ceph.rook.io/v1/CephRBDMirror --helm-api-versions cert-manager.io/v1 --helm-api-versions cert-manager.io/v1/Certificate --helm-api-versions cert-manager.io/v1/CertificateRequest --helm-api-versions cert-manager.io/v1/ClusterIssuer --helm-api-versions cert-manager.io/v1/Issuer --helm-api-versions certificates.k8s.io/v1 --helm-api-versions certificates.k8s.io/v1/CertificateSigningRequest --helm-api-versions cilium.io/v2 --helm-api-versions cilium.io/v2/CiliumBGPAdvertisement --helm-api-versions cilium.io/v2/CiliumBGPClusterConfig --helm-api-versions cilium.io/v2/CiliumBGPNodeConfig --helm-api-versions cilium.io/v2/CiliumBGPNodeConfigOverride --helm-api-versions cilium.io/v2/CiliumBGPPeerConfig --helm-api-versions cilium.io/v2/CiliumCIDRGroup --helm-api-versions cilium.io/v2/CiliumClusterwideNetworkPolicy --helm-api-versions cilium.io/v2/CiliumEndpoint --helm-api-versions cilium.io/v2/CiliumExternalWorkload --helm-api-versions cilium.io/v2/CiliumIdentity --helm-api-versions cilium.io/v2/CiliumLoadBalancerIPPool --helm-api-versions cilium.io/v2/CiliumNetworkPolicy --helm-api-versions cilium.io/v2/CiliumNode --helm-api-versions cilium.io/v2/CiliumNodeConfig --helm-api-versions cilium.io/v2alpha1 --helm-api-versions cilium.io/v2alpha1/CiliumBGPAdvertisement --helm-api-versions cilium.io/v2alpha1/CiliumBGPClusterConfig --helm-api-versions cilium.io/v2alpha1/CiliumBGPNodeConfig --helm-api-versions cilium.io/v2alpha1/CiliumBGPNodeConfigOverride --helm-api-versions cilium.io/v2alpha1/CiliumBGPPeerConfig --helm-api-versions cilium.io/v2alpha1/CiliumBGPPeeringPolicy --helm-api-versions cilium.io/v2alpha1/CiliumCIDRGroup --helm-api-versions cilium.io/v2alpha1/CiliumL2AnnouncementPolicy --helm-api-versions cilium.io/v2alpha1/CiliumLoadBalancerIPPool --helm-api-versions cilium.io/v2alpha1/CiliumNodeConfig --helm-api-versions cilium.io/v2alpha1/CiliumPodIPPool --helm-api-versions coordination.k8s.io/v1 --helm-api-versions coordination.k8s.io/v1/Lease --helm-api-versions core.strimzi.io/v1 --helm-api-versions core.strimzi.io/v1/StrimziPodSet --helm-api-versions core.strimzi.io/v1beta2 --helm-api-versions core.strimzi.io/v1beta2/StrimziPodSet --helm-api-versions csi.ceph.io/v1 --helm-api-versions csi.ceph.io/v1/CephConnection --helm-api-versions csi.ceph.io/v1/ClientProfile --helm-api-versions csi.ceph.io/v1/ClientProfileMapping --helm-api-versions csi.ceph.io/v1/Driver --helm-api-versions csi.ceph.io/v1/OperatorConfig --helm-api-versions csi.ceph.io/v1alpha1 --helm-api-versions csi.ceph.io/v1alpha1/CephConnection --helm-api-versions csi.ceph.io/v1alpha1/ClientProfile --helm-api-versions csi.ceph.io/v1alpha1/ClientProfileMapping --helm-api-versions csi.ceph.io/v1alpha1/Driver --helm-api-versions csi.ceph.io/v1alpha1/OperatorConfig --helm-api-versions dex.coreos.com/v1 --helm-api-versions dex.coreos.com/v1/AuthCode --helm-api-versions dex.coreos.com/v1/AuthRequest --helm-api-versions dex.coreos.com/v1/Connector --helm-api-versions dex.coreos.com/v1/DeviceRequest --helm-api-versions dex.coreos.com/v1/DeviceToken --helm-api-versions dex.coreos.com/v1/OAuth2Client --helm-api-versions dex.coreos.com/v1/OfflineSessions --helm-api-versions dex.coreos.com/v1/Password --helm-api-versions dex.coreos.com/v1/RefreshToken --helm-api-versions dex.coreos.com/v1/SigningKey --helm-api-versions discovery.k8s.io/v1 --helm-api-versions discovery.k8s.io/v1/EndpointSlice --helm-api-versions events.k8s.io/v1 --helm-api-versions events.k8s.io/v1/Event --helm-api-versions external-secrets.io/v1 --helm-api-versions external-secrets.io/v1/ClusterExternalSecret --helm-api-versions external-secrets.io/v1/ClusterSecretStore --helm-api-versions external-secrets.io/v1/ExternalSecret --helm-api-versions external-secrets.io/v1/SecretStore --helm-api-versions external-secrets.io/v1alpha1 --helm-api-versions external-secrets.io/v1alpha1/ClusterPushSecret --helm-api-versions external-secrets.io/v1alpha1/PushSecret --helm-api-versions flowcontrol.apiserver.k8s.io/v1 --helm-api-versions flowcontrol.apiserver.k8s.io/v1/FlowSchema --helm-api-versions flowcontrol.apiserver.k8s.io/v1/PriorityLevelConfiguration --helm-api-versions gateway.networking.k8s.io/v1 --helm-api-versions gateway.networking.k8s.io/v1/GRPCRoute --helm-api-versions gateway.networking.k8s.io/v1/Gateway --helm-api-versions gateway.networking.k8s.io/v1/GatewayClass --helm-api-versions gateway.networking.k8s.io/v1/HTTPRoute --helm-api-versions gateway.networking.k8s.io/v1beta1 --helm-api-versions gateway.networking.k8s.io/v1beta1/Gateway --helm-api-versions gateway.networking.k8s.io/v1beta1/GatewayClass --helm-api-versions gateway.networking.k8s.io/v1beta1/HTTPRoute --helm-api-versions gateway.networking.k8s.io/v1beta1/ReferenceGrant --helm-api-versions generators.external-secrets.io/v1alpha1 --helm-api-versions generators.external-secrets.io/v1alpha1/ACRAccessToken --helm-api-versions generators.external-secrets.io/v1alpha1/CloudsmithAccessToken --helm-api-versions generators.external-secrets.io/v1alpha1/ClusterGenerator --helm-api-versions generators.external-secrets.io/v1alpha1/ECRAuthorizationToken --helm-api-versions generators.external-secrets.io/v1alpha1/Fake --helm-api-versions generators.external-secrets.io/v1alpha1/GCRAccessToken --helm-api-versions generators.external-secrets.io/v1alpha1/GeneratorState --helm-api-versions generators.external-secrets.io/v1alpha1/GithubAccessToken --helm-api-versions generators.external-secrets.io/v1alpha1/Grafana --helm-api-versions generators.external-secrets.io/v1alpha1/MFA --helm-api-versions generators.external-secrets.io/v1alpha1/Password --helm-api-versions generators.external-secrets.io/v1alpha1/QuayAccessToken --helm-api-versions generators.external-secrets.io/v1alpha1/SSHKey --helm-api-versions generators.external-secrets.io/v1alpha1/STSSessionToken --helm-api-versions generators.external-secrets.io/v1alpha1/UUID --helm-api-versions generators.external-secrets.io/v1alpha1/VaultDynamicSecret --helm-api-versions generators.external-secrets.io/v1alpha1/Webhook --helm-api-versions groupsnapshot.storage.k8s.io/v1beta1 --helm-api-versions groupsnapshot.storage.k8s.io/v1beta1/VolumeGroupSnapshot --helm-api-versions groupsnapshot.storage.k8s.io/v1beta1/VolumeGroupSnapshotClass --helm-api-versions groupsnapshot.storage.k8s.io/v1beta1/VolumeGroupSnapshotContent --helm-api-versions kafka.strimzi.io/v1 --helm-api-versions kafka.strimzi.io/v1/Kafka --helm-api-versions kafka.strimzi.io/v1/KafkaBridge --helm-api-versions kafka.strimzi.io/v1/KafkaConnect --helm-api-versions kafka.strimzi.io/v1/KafkaConnector --helm-api-versions kafka.strimzi.io/v1/KafkaMirrorMaker2 --helm-api-versions kafka.strimzi.io/v1/KafkaNodePool --helm-api-versions kafka.strimzi.io/v1/KafkaRebalance --helm-api-versions kafka.strimzi.io/v1/KafkaTopic --helm-api-versions kafka.strimzi.io/v1/KafkaUser --helm-api-versions kafka.strimzi.io/v1alpha1 --helm-api-versions kafka.strimzi.io/v1alpha1/KafkaTopic --helm-api-versions kafka.strimzi.io/v1alpha1/KafkaUser --helm-api-versions kafka.strimzi.io/v1beta1 --helm-api-versions kafka.strimzi.io/v1beta1/KafkaTopic --helm-api-versions kafka.strimzi.io/v1beta1/KafkaUser --helm-api-versions kafka.strimzi.io/v1beta2 --helm-api-versions kafka.strimzi.io/v1beta2/Kafka --helm-api-versions kafka.strimzi.io/v1beta2/KafkaBridge --helm-api-versions kafka.strimzi.io/v1beta2/KafkaConnect --helm-api-versions kafka.strimzi.io/v1beta2/KafkaConnector --helm-api-versions kafka.strimzi.io/v1beta2/KafkaMirrorMaker2 --helm-api-versions kafka.strimzi.io/v1beta2/KafkaNodePool --helm-api-versions kafka.strimzi.io/v1beta2/KafkaRebalance --helm-api-versions kafka.strimzi.io/v1beta2/KafkaTopic --helm-api-versions kafka.strimzi.io/v1beta2/KafkaUser --helm-api-versions kyverno.io/v1 --helm-api-versions kyverno.io/v1/ClusterPolicy --helm-api-versions kyverno.io/v1/Policy --helm-api-versions kyverno.io/v1beta1 --helm-api-versions kyverno.io/v1beta1/UpdateRequest --helm-api-versions kyverno.io/v2 --helm-api-versions kyverno.io/v2/CleanupPolicy --helm-api-versions kyverno.io/v2/ClusterCleanupPolicy --helm-api-versions kyverno.io/v2/PolicyException --helm-api-versions kyverno.io/v2/UpdateRequest --helm-api-versions kyverno.io/v2alpha1 --helm-api-versions kyverno.io/v2alpha1/GlobalContextEntry --helm-api-versions kyverno.io/v2beta1 --helm-api-versions kyverno.io/v2beta1/CleanupPolicy --helm-api-versions kyverno.io/v2beta1/ClusterCleanupPolicy --helm-api-versions kyverno.io/v2beta1/ClusterPolicy --helm-api-versions kyverno.io/v2beta1/Policy --helm-api-versions kyverno.io/v2beta1/PolicyException --helm-api-versions monitoring.coreos.com/v1 --helm-api-versions monitoring.coreos.com/v1/Alertmanager --helm-api-versions monitoring.coreos.com/v1/PodMonitor --helm-api-versions monitoring.coreos.com/v1/Probe --helm-api-versions monitoring.coreos.com/v1/Prometheus --helm-api-versions monitoring.coreos.com/v1/PrometheusRule --helm-api-versions monitoring.coreos.com/v1/ServiceMonitor --helm-api-versions monitoring.coreos.com/v1/ThanosRuler --helm-api-versions monitoring.coreos.com/v1alpha1 --helm-api-versions monitoring.coreos.com/v1alpha1/AlertmanagerConfig --helm-api-versions monitoring.coreos.com/v1alpha1/PrometheusAgent --helm-api-versions monitoring.coreos.com/v1alpha1/ScrapeConfig --helm-api-versions networking.k8s.io/v1 --helm-api-versions networking.k8s.io/v1/IPAddress --helm-api-versions networking.k8s.io/v1/Ingress --helm-api-versions networking.k8s.io/v1/IngressClass --helm-api-versions networking.k8s.io/v1/NetworkPolicy --helm-api-versions networking.k8s.io/v1/ServiceCIDR --helm-api-versions node.k8s.io/v1 --helm-api-versions node.k8s.io/v1/RuntimeClass --helm-api-versions objectbucket.io/v1alpha1 --helm-api-versions objectbucket.io/v1alpha1/ObjectBucket --helm-api-versions objectbucket.io/v1alpha1/ObjectBucketClaim --helm-api-versions policies.kyverno.io/v1alpha1 --helm-api-versions policies.kyverno.io/v1alpha1/DeletingPolicy --helm-api-versions policies.kyverno.io/v1alpha1/GeneratingPolicy --helm-api-versions policies.kyverno.io/v1alpha1/ImageValidatingPolicy --helm-api-versions policies.kyverno.io/v1alpha1/MutatingPolicy --helm-api-versions policies.kyverno.io/v1alpha1/PolicyException --helm-api-versions policies.kyverno.io/v1alpha1/ValidatingPolicy --helm-api-versions policy/v1 --helm-api-versions policy/v1/PodDisruptionBudget --helm-api-versions postgresql.cnpg.io/v1 --helm-api-versions postgresql.cnpg.io/v1/Backup --helm-api-versions postgresql.cnpg.io/v1/Cluster --helm-api-versions postgresql.cnpg.io/v1/ClusterImageCatalog --helm-api-versions postgresql.cnpg.io/v1/Database --helm-api-versions postgresql.cnpg.io/v1/FailoverQuorum --helm-api-versions postgresql.cnpg.io/v1/ImageCatalog --helm-api-versions postgresql.cnpg.io/v1/Pooler --helm-api-versions postgresql.cnpg.io/v1/Publication --helm-api-versions postgresql.cnpg.io/v1/ScheduledBackup --helm-api-versions postgresql.cnpg.io/v1/Subscription --helm-api-versions rbac.authorization.k8s.io/v1 --helm-api-versions rbac.authorization.k8s.io/v1/ClusterRole --helm-api-versions rbac.authorization.k8s.io/v1/ClusterRoleBinding --helm-api-versions rbac.authorization.k8s.io/v1/Role --helm-api-versions rbac.authorization.k8s.io/v1/RoleBinding --helm-api-versions reports.kyverno.io/v1 --helm-api-versions reports.kyverno.io/v1/ClusterEphemeralReport --helm-api-versions reports.kyverno.io/v1/EphemeralReport --helm-api-versions resource.k8s.io/v1 --helm-api-versions resource.k8s.io/v1/DeviceClass --helm-api-versions resource.k8s.io/v1/ResourceClaim --helm-api-versions resource.k8s.io/v1/ResourceClaimTemplate --helm-api-versions resource.k8s.io/v1/ResourceSlice --helm-api-versions scheduling.k8s.io/v1 --helm-api-versions scheduling.k8s.io/v1/PriorityClass --helm-api-versions snapshot.storage.k8s.io/v1 --helm-api-versions snapshot.storage.k8s.io/v1/VolumeSnapshot --helm-api-versions snapshot.storage.k8s.io/v1/VolumeSnapshotClass --helm-api-versions snapshot.storage.k8s.io/v1/VolumeSnapshotContent --helm-api-versions storage.k8s.io/v1 --helm-api-versions storage.k8s.io/v1/CSIDriver --helm-api-versions storage.k8s.io/v1/CSINode --helm-api-versions storage.k8s.io/v1/CSIStorageCapacity --helm-api-versions storage.k8s.io/v1/StorageClass --helm-api-versions storage.k8s.io/v1/VolumeAttachment --helm-api-versions storage.k8s.io/v1/VolumeAttributesClass --helm-api-versions talos.dev/v1alpha1 --helm-api-versions talos.dev/v1alpha1/ServiceAccount --helm-api-versions traefik.io/v1alpha1 --helm-api-versions traefik.io/v1alpha1/IngressRoute --helm-api-versions traefik.io/v1alpha1/IngressRouteTCP --helm-api-versions traefik.io/v1alpha1/IngressRouteUDP --helm-api-versions traefik.io/v1alpha1/Middleware --helm-api-versions traefik.io/v1alpha1/MiddlewareTCP --helm-api-versions traefik.io/v1alpha1/ServersTransport --helm-api-versions traefik.io/v1alpha1/ServersTransportTCP --helm-api-versions traefik.io/v1alpha1/TLSOption --helm-api-versions traefik.io/v1alpha1/TLSStore --helm-api-versions traefik.io/v1alpha1/TraefikService --helm-api-versions v1 --helm-api-versions v1/ConfigMap --helm-api-versions v1/Endpoints --helm-api-versions v1/Event --helm-api-versions v1/LimitRange --helm-api-versions v1/Namespace --helm-api-versions v1/Node --helm-api-versions v1/PersistentVolume --helm-api-versions v1/PersistentVolumeClaim --helm-api-versions v1/Pod --helm-api-versions v1/PodTemplate --helm-api-versions v1/ReplicationController --helm-api-versions v1/ResourceQuota --helm-api-versions v1/Secret --helm-api-versions v1/Service --helm-api-versions v1/ServiceAccount --helm-api-versions velero.io/v1 --helm-api-versions velero.io/v1/Backup --helm-api-versions velero.io/v1/BackupRepository --helm-api-versions velero.io/v1/BackupStorageLocation --helm-api-versions velero.io/v1/DeleteBackupRequest --helm-api-versions velero.io/v1/DownloadRequest --helm-api-versions velero.io/v1/PodVolumeBackup --helm-api-versions velero.io/v1/PodVolumeRestore --helm-api-versions velero.io/v1/Restore --helm-api-versions velero.io/v1/Schedule --helm-api-versions velero.io/v1/ServerStatusRequest --helm-api-versions velero.io/v1/VolumeSnapshotLocation --helm-api-versions velero.io/v2alpha1 --helm-api-versions velero.io/v2alpha1/DataDownload --helm-api-versions velero.io/v2alpha1/DataUpload --helm-api-versions wgpolicyk8s.io/v1alpha2 --helm-api-versions wgpolicyk8s.io/v1alpha2/ClusterPolicyReport --helm-api-versions wgpolicyk8s.io/v1alpha2/PolicyReport
```