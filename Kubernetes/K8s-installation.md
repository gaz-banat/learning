INSTALLATION


**Before you start think about

spec of master node and whether one or HA
spec of worker nodes
k8s network subnet for networking within the cluster
the choice of network provider, e.g. calico, flannel, weave
DNS or how the nodes will find each other on the network

and then
  would you like the k8s web ui
  the choice of ingress controller
  

==Master Node==

** Prerequisites

disable SELinux
	setenforce 0
	sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux

the following ports need to be open
	6443/tcp, 2379-2380/tcp, 10250/tcp, 10251/tcp, 10252/tcp, 10255/tcp

The br_netfilter module must be loaded into the kernel
	modprobe br_netfilter

Iptables should be able to see bridged traffic
	echo ‘1’ > /proc/sys/net/bridge/bridge-nf-call-iptables OR sysctl -w net.bridge.bridge-nf-call-iptables=1	  [not persistent] 
        OR
	sysctl -w net.bridge.bridge-nf-call-iptables=1 >> /etc/sysctl.conf and sysctl  - - system
	IF ipv6 is required then sysctl -w net.bridge.bridge-nf-call-ip6tables=1 >> /etc/sysctl.conf and sysctl  - - system

Swap should be turned off
	swapoff -a  and comment swap partitions in /etc/fstab

Docker should be installed and started up


**Repositories

the following repo file should exist /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
              https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg


**Installation

Install the kubeadm package
	yum -y install kubeadm

Enable the kubelet
	systemctl restart kubelet && systemctl enable kubelet			

Initialize the kubernetes master
	kubeadm config images pull
	kubeadm init --apiserver-advertise-address=<ip_addr> --pod-network-cidr=<pod_subnet> >> /root/kubeinit.log 

Copy the kubeadm join  - - token <token> <ip_addr>:<port> and save it somewhere
	OR you can get it later with kubeadm token create --print-join-command 

Allow your logged in user to use the cluster as root user
	mkdir -p $HOME/.kube
	cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
	chown $(id -u):$(id -g) $HOME/.kube/config

A quick check
	kubectl get nodes				[master will not be ready]
	kubectl get pods  - -all-namespaces

Deploy the pod network
    Weave
	export kubever=$(kubectl version | base64 | tr -d ‘\n’)
	kubectl apply -f “https://cloud.weave.works/k8s/net?k8s-version=${kubever}”
    Flannel
	kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/k8s-manifests/kube-flannel-rbac.yml	kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml 

Quick check again
	kubectl get nodes				[master should be ready]


==Worker Node==

**Prerequisites

Disable SELinux

Add these firewall ports 
	10250/tcp, 10255/tcp, 3000-32767/tcp, 6783/tcp

Iptables should be able to see bridged traffic

Swap should be turned off

**Repositories

Repository for Kubernetes

Install kubeadm

Join worker node to master
	kubeadm join  - - token <token> <ip_addr>:<port> 


==Master Node==

Quick check
	kubectl get nodes				[worker node should be visible]


Ingress with Nginx	
	kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.44.0/deploy/static/provider/baremetal/deploy.yaml
	kubectl get pods --all-namespaces -l app.kubernetes.io/name=ingress-nginx
	kubectl get services ingress-nginx-controller --namespace=ingress-nginx
	kubectl delete -A validatingwebhookconfigurations ingress-nginx-admission







