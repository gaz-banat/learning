
# SOC Configuration

https://git.aarnet.edu.au/socop/infrastructure/soc-kubernetes/soc-k8s-app-monitoring/-/blob/main/base/values.yaml



# LINKS


## Prometheus Operator
Prometheus Operator                   -       https://prometheus-operator.dev/docs/getting-started/introduction/


## Kube Prometheus

Kube Prometheus source                -       https://github.com/prometheus-operator/kube-prometheus    (JSONNET)

Chart Repo                            -       https://prometheus-community.github.io/helm-charts

Chart Source                          -       https://github.com/prometheus-community/helm-charts/
                                              https://github.com/prometheus-community/helm-charts/tree/main/charts

Kube Prometheus Stack Chart Source    -       https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack




# UNDERSTANDING

There are 2 different things here

- The promethues operator
- Kube Prometheus (aka Kube Prometheus Stack or KPS)        -   prometheus operator + other things, meant for kubernetes cluster monitoring


# KUBE PROMETHEUS COMPONENTS

Prometheus Operator
Prometheus
AlertManager
Prometheus Blackbox Exporter
Prometheus Adapater for Kubernetes Metrics API      -   this adapter is an extension API server
Grafana
Kube State Metrics                                  -   a simple service that listens to the Kubernetes API server and generates metrics about the state of the objects
Prometheus Node Exporter                            -   provides the host level metrics (cpu, memory, network, etc.)
Prometheus Windows Exporter


# INSTALLATION

Looks like there are 3 ways to install
- download the repo of kube prometheus and do a kubectl apply of manifests/setup and then manifests/
- download the repo of kube prometheus and configure with jsonnet
- install using helm chart


## HELM BASED INSTALLATION

The helm based installation is split up into many charts

The main chart to install is the kube-prometheus-stack chart
 - the main chart installs some core components of kube prometheus but not all components, for e.g. it leaves out blackbox exporter and prometheus adapter
      prometheus-operator
      prometheus
      alertmanager
      various supporting resources (e.g. rbac, prometheusrules, pvc, etc.)
 - the main chart then installs subcharts based on selections
      crds
      grafana
      kube-state-metrics
      prometheus-node-exporter 
      prometheus-windows-exporter
 - updating a value in the main chart, e.g. grafana.image will update the 'image' field in the grafana subchart

Uninstalling the chart does not remove the CRDs



# CRDs

alertmanagerconfigs.monitoring.coreos.com
alertmanagers.monitoring.coreos.com
podmonitors.monitoring.coreos.com
probes.monitoring.coreos.com
prometheusagents.monitoring.coreos.com
prometheuses.monitoring.coreos.com
prometheusrules.monitoring.coreos.com
scrapeconfigs.monitoring.coreos.com
servicemonitors.monitoring.coreos.com
thanosrulers.monitoring.coreos.com


# RESOURCES OF KUBE PROMETHEUS STACK

The resources carry a label for the application that they belong to


## Operator

```shell
kbeh kube-prometheus-stack/overlays/ossk8sdev | yq 'select(.metadata.labels["app"] == "kube-prometheus-stack-operator") | (.kind,.metadata.name)'

```

ServiceAccount/kps-operator
ClusterRole/kps-operator
ClusterRoleBinding/kps-operator
Service/kps-operator
Deployment/kps-operator
ServiceMonitor/kps-operator


## Prometheus

```shell
kbeh kube-prometheus-stack/overlays/ossk8sdev | yq 'select(.metadata.labels["app"] == "kube-prometheus-stack-prometheus") | (.kind,.metadata.name)'

```

ServiceAccount/kps-prometheus
ClusterRole/kps-prometheus
ClusterRoleBinding/kps-prometheus
Service/kps-prometheus
Prometheus/kps-prometheus
ServiceMonitor/kps-prometheus
Ingress/kps-prometheus



## AlertManager

```shell
kbeh kube-prometheus-stack/overlays/ossk8sdev | yq 'select(.metadata.labels["app"] == "kube-prometheus-stack-alertmanager") | (.kind,.metadata.name)'
```

ServiceAccount/kps-alertmanager
Secret/alertmanager-kps-alertmanager
Service/kps-alertmanager
Alertmanager/kps-alertmanager
ServiceMonitor/kps-alertmanager
Ingress/kps-alertmanager



## Grafana

```shell
kbeh kube-prometheus-stack/overlays/ossk8sdev | yq 'select(.metadata.labels["app"] == "kube-prometheus-stack-grafana") | (.kind,.metadata.name)'
kbeh kube-prometheus-stack/overlays/ossk8sdev | yq 'select(.metadata.labels["app.kubernetes.io/name"] == "grafana") | (.kind,.metadata.name)'
```

ServiceAccount/kps-grafana

Role/kps-grafana

ClusterRole/kps-grafana-clusterrole

ClusterRoleBinding/kps-grafana-clusterrolebinding

RoleBinding/kps-grafana

Service/kps-grafana

Deployment/kps-grafana

ConfigMap/kps-grafana
ConfigMap/kps-grafana-config-dashboards

ConfigMap/kps-alertmanager-overview
ConfigMap/kps-apiserver
ConfigMap/kps-cluster-total
ConfigMap/kps-controller-manager
ConfigMap/kps-etcd
ConfigMap/kps-grafana-datasource
ConfigMap/kps-grafana-overview
ConfigMap/kps-k8s-coredns
ConfigMap/kps-k8s-resources-cluster
ConfigMap/kps-k8s-resources-multicluster
ConfigMap/kps-k8s-resources-namespace
ConfigMap/kps-k8s-resources-node
ConfigMap/kps-k8s-resources-pod
ConfigMap/kps-k8s-resources-workload
ConfigMap/kps-k8s-resources-workloads-namespace
ConfigMap/kps-kubelet
ConfigMap/kps-namespace-by-pod
ConfigMap/kps-node-cluster-rsrc-use
ConfigMap/kps-node-rsrc-use
ConfigMap/kps-nodes
ConfigMap/kps-persistentvolumesusage
ConfigMap/kps-pod-total
ConfigMap/kps-prometheus                                                   
ConfigMap/kps-scheduler
ConfigMap/kps-workload-total

PersistentVolumeClaim/kps-grafana

ServiceMonitor/kps-grafana

Ingress/kps-grafana


## Kube State Metrics

```shell
kbeh kube-prometheus-stack/overlays/ossk8sdev | yq 'select(.metadata.labels["app.kubernetes.io/name"] == "kube-state-metrics") | (.kind,.metadata.name)'
```

ServiceAccount/kps-kube-state-metrics
ClusterRole/kps-kube-state-metrics
ClusterRoleBinding/kps-kube-state-metrics
Service/kps-kube-state-metrics
Deployment/kps-kube-state-metrics
ServiceMonitor/kps-kube-state-metrics


## Node Exporter

```shell
kbeh kube-prometheus-stack/overlays/ossk8sdev | yq 'select(.metadata.labels["app.kubernetes.io/name"] == "prometheus-node-exporter") | (.kind,.metadata.name)'
```

ServiceAccount/kps-prometheus-node-exporter
Service/kps-prometheus-node-exporter
DaemonSet/kps-prometheus-node-exporter
ServiceMonitor/kps-prometheus-node-exporter


## Kube Prometheus Stack (think these resources are not part of any of the components)

```shell
kbeh kube-prometheus-stack/overlays/ossk8sdev | yq 'select(.metadata.labels["app"] == "kube-prometheus-stack") | (.kind,.metadata.name)'
```

PrometheusRule/kps-alertmanager.rules
PrometheusRule/kps-config-reloaders
PrometheusRule/kps-etcd
PrometheusRule/kps-general.rules
PrometheusRule/kps-k8s.rules.container-cpu-usage-seconds-total
PrometheusRule/kps-k8s.rules.container-memory-cache
PrometheusRule/kps-k8s.rules.container-memory-rss
PrometheusRule/kps-k8s.rules.container-memory-swap
PrometheusRule/kps-k8s.rules.container-memory-working-set-bytes
PrometheusRule/kps-k8s.rules.container-resource
PrometheusRule/kps-k8s.rules.pod-owner
PrometheusRule/kps-kube-apiserver-availability.rules
PrometheusRule/kps-kube-apiserver-burnrate.rules
PrometheusRule/kps-kube-apiserver-histogram.rules
PrometheusRule/kps-kube-apiserver-slos
PrometheusRule/kps-kube-prometheus-general.rules
PrometheusRule/kps-kube-prometheus-node-recording.rules
PrometheusRule/kps-kube-scheduler.rules
PrometheusRule/kps-kube-state-metrics
PrometheusRule/kps-kubelet.rules
PrometheusRule/kps-kubernetes-apps
PrometheusRule/kps-kubernetes-resources
PrometheusRule/kps-kubernetes-storage
PrometheusRule/kps-kubernetes-system
PrometheusRule/kps-kubernetes-system-apiserver
PrometheusRule/kps-kubernetes-system-controller-manager
PrometheusRule/kps-kubernetes-system-kubelet
PrometheusRule/kps-node-exporter
PrometheusRule/kps-node-exporter.rules
PrometheusRule/kps-node-network
PrometheusRule/kps-node.rules
PrometheusRule/kps-prometheus
PrometheusRule/kps-prometheus-operator



## TBC

ServiceAccount/kps-admission

Role/kps-admission

ClusterRole/kps-admission

RoleBinding/kps-admission

ClusterRoleBinding/kps-admission



ConfigMap/grafana-dashboards-infra
ConfigMap/kps-namespace-by-workload



Endpoints/kps-kube-etcd

Service/kps-coredns
Service/kps-kube-controller-manager
Service/kps-kube-etcd
Service/kps-kube-scheduler


Job/kps-admission-create
Job/kps-admission-patch


ServiceMonitor/kps-apiserver
ServiceMonitor/kps-coredns
ServiceMonitor/kps-kube-controller-manager
ServiceMonitor/kps-kube-etcd
ServiceMonitor/kps-kube-scheduler
ServiceMonitor/kps-kubelet


MutatingWebhookConfiguration/kps-admission

ValidatingWebhookConfiguration/kps-admission



# PROMETHEUS

## Configuration

The configuration needs to be updated by the operator as monitors, alerts, etc. are added and removed

```shell
# the running configuration for prometheus

## we start with a secret called prometheus-kps-prometheus
$ k get secret -n monitoring prometheus-kps-prometheus -o jsonpath='{.data.prometheus\.yaml\.gz}' | base64 -d | gunzip | less

## becomes a volume in the pod
$ k get pod prometheus-kps-prometheus-0 -n monitoring -o yaml | yq '.spec.volumes[] | select(.name == "config")'
name: config
secret:
  defaultMode: 420
  secretName: prometheus-kps-prometheus

## gets mounted to the config-reloader container
$ k get pod prometheus-kps-prometheus-0 -n monitoring -o yaml | yq '.spec.containers[] | select(.name == "config-reloader") | .volumeMounts[] | select(.name == "config")'
mountPath: /etc/prometheus/config
name: config

## and can be found at the path in the config-reloader container
$ k exec -n monitoring prometheus-kps-prometheus-0 -c config-reloader -- ls -l /etc/prometheus/config/ 
lrwxrwxrwx    1 root     2000            25 Oct  7 05:36 prometheus.yaml.gz -> ..data/prometheus.yaml.gz # still in the compressed form

# the prometheus container in the pod gets the same file at
k exec -n monitoring prometheus-kps-prometheus-0 -c prometheus -- cat /etc/prometheus/config_out/prometheus.env.yaml | less
(time=2025-10-13T06:29:14.850Z level=INFO source=main.go:1497 msg="Loading configuration file" filename=/etc/prometheus/config_out/prometheus.env.yaml)
```



# ALERTMANAGER


## Configuration

```shell
kubectl get secret alertmanager-kps-alertmanager -n monitoring -o jsonpath='{.data.alertmanager\.yaml}' | base64 -d


kubectl get secret alertmanager-kps-alertmanager-generated -n monitoring -o jsonpath="{.data.alertmanager\.yaml\.gz}" | base64 -d | gunzip


kubectl exec -it alertmanager-kps-alertmanager-0 -- cat /etc/alertmanager/config_out/alertmanager.env.yaml
```


## Alerts

```shell
## Get firing alerts
kubectl -n monitoring exec sts/alertmanager-kps-alertmanager -- wget -q -O- 'http://127.0.0.1:9093/api/v2/alerts/groups?silenced=false&inhibited=false&muted=false&active=true' | jq '.[].alerts[].labels'
```



# GRAFANA

## Configuration

the configuration for data sources

```shell
k get cm -n monitoring kps-grafana-datasource -o yaml
```

The prometheus/overview dashboard

```shell
k get cm kps-prometheus -n monitoring -o jsonpath='{.data.prometheus\.json}' | jq .
```