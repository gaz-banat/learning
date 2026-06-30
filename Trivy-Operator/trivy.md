
# FEATURES

Trivy can scan for 
- CVEs
- Exposed secrets and sensitive information
- Misconfigurations (for example Terraform configuration, Kubernetes manifests)
- OS packages and software dependencies

Trivy can also
- generate an sbom
- attestation 



# COMPONENTS

trivy cli

trivy-operator              -           is a Kubernetes-native controller that automatically and continuously scans cluster workloads 
                                        for vulnerabilities and configuration issues, producing CRD-based reports. 

trivy server                -           is a component that provides a centralized, updated vulnerability database to the operator, 
                                        often used in client-server mode to optimize scan performance



# CONCEPTS

Vulnerability Report

SBOM








# KUBERNETES

## CRDS

clustercompliancereports.aquasecurity.github.io     
clusterconfigauditreports.aquasecurity.github.io    
clusterinfraassessmentreports.aquasecurity.github.io
clusterrbacassessmentreports.aquasecurity.github.io 
clustersbomreports.aquasecurity.github.io           
clustervulnerabilityreports.aquasecurity.github.io  
configauditreports.aquasecurity.github.io           
exposedsecretreports.aquasecurity.github.io         
infraassessmentreports.aquasecurity.github.io       
rbacassessmentreports.aquasecurity.github.io        
sbomreports.aquasecurity.github.io                              -       the sbom report of an aritifact
vulnerabilityreports.aquasecurity.github.io



## Trivy-Operator Configuration

```kubectl get configmap -n trivy trivy-operator-config -o yaml | ditch_meta.sh```

OPERATOR_SCANNER_REPORT_TTL: 24h            - how long should a report live before being expired and rescanned


## Commands

```shell
# Get the latest vulnerability scans
kubectl -n administration get vuln -o wide

# look at the critical vulnerability in detail for one of the latest scans
kubectl -n administration get vuln cronjob-talos-etcd-defrag-talosctl -o=json | jq '.report.vulnerabilities[] | select(.severity == "CRITICAL")'

# to do a new scan just delete the existing one, trivy will see that there is no scan and kick one off
kubectl delete -n administration vuln cronjob-talos-etcd-defrag-talosctl
```