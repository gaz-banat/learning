

CONSIDERATIONS


Domain - domain for gitlab, e.g.


email address - Lets encrypt certificate







Helm chart installation

helm upgrade \--install 
gitlab gitlab/gitlab 
-n gitlab 
\--version 5.1.3 
\--set global.edition=ce 
\--set global.hosts.domain=gaz.net.nz 
\--set global.hosts.externalIP=34.129.165.213 
\--set certmanager-issuer.email=gbanatwala@gmail.com
