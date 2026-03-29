

# CRDS

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


# COMPONENTS

trivy-operator

trivy server




# Commands

```shell
# Get the latest vulnerability scans
kubectl -n administration get vuln -o wide

# look at the critical vulnerability in detail for one of the latest scans
kubectl -n administration get vuln cronjob-talos-etcd-defrag-talosctl -o=json | jq '.report.vulnerabilities[] | select(.severity == "CRITICAL")'

# to do a new scan just delete the existing one, trivy will see that there is no scan and kick one off
kubectl delete -n administration vuln cronjob-talos-etcd-defrag-talosctl
```