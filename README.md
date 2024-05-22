# Load-Balancer-and-Ingress-Controller----k8s
Metallb and Nginx

DEPLOYMENT STEPS:

---
**I. METALLB**
1. Install Metallb as Load Balancer: https://www.youtube.com/watch?v=k8bxtsWe9qw
   
   1.1 Use Netplan to add additional IP addresses to the Network Adapter of the Master kubernetes node
   
 - Change directory to netplan location
 - create a config file 01_config.yaml
 - fill in the configuration to add additional IP addressess on the specified network adapder
 - apply the netplan
 - check if the IP addresses have been added

       cd /etc/netplan
       vi 01_config.yaml

       ---
       network:
         version: 2
         renderer: networkd
         ethernets:
           ens160:
             addresses:
             - 10.40.0.132/24
             - 10.40.0.135/24
             - 10.40.0.136/24
             gateway4: 10.40.0.254
             nameservers:
               addresses: [10.40.0.8, 1.1.1.1]
       ---

       sudo netplan apply
       ip addr show ens160

   
   1.2 Edit the kube-proxy Config Map in the kube-system namespace in order to enable strict ARP.

 - Change data.config.conf.ipvs.strictARP from „false” to „true” 

       kubectl edit configmap -n kube-system kube-proxy


   1.3 Install Metallb

 - check if the deployment was successfull

       kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.5/config/manifests/metallb-native.yaml
       k get all -n metallb-system


   1.4 Create an IPAddressPool object:

 - Create a directory to store the metallb manifest files
 - Create the IPAddressPool yaml file
 - specify the range of external IPs which was previously defined on the Network interface
 - apply the manifest file
 - check if the IPAddressPool was created

       mkdir metallb-files
       cd metallb-files
       vi pool-1.yaml

       ---
       apiVersion: metallb.io/v1beta1
	   kind: IPAddressPool
	   metadata:
	     name: first-pool
	     namespace: metallb-system
       spec:
	     addresses:
	     - 10.40.0.135-10.40.0.136
       ---

       kubectl apply -f pool-1.yaml -n metallb-system
       kubectl get IPAddressPool -n metallb-system


   1.5 Create an L2Advertisement object:
   
 - Create the L2Advertisement yaml file
 - Specify the IP Address Pool advertised
 - apply the manifest file
 - check if the L2Advertisement was created

       vi l2-advertisement.yaml

       ---
       apiVersion: metallb.io/v1beta1
	   kind: L2Advertisement
	   metadata:
	     name: demo
	     namespace: metallb-system
	   spec:
	     ipAddressPools:
	     - first-pool
       ---

       kubectl apply -f l2-advertisement.yaml
       kubectl get l2advertisement -n metallb-system

---

![image](https://github.com/denis-314/Load-Balancer-and-Ingress-Controller----k8s/assets/112620749/6f6fd8d7-790c-465c-87cd-d79a9840c6e9)


**II. INGRESS SERVICE**

2. Install NGINX as Ingress Controller:

   2.1 Install Nginx
 - apply the manifest file
 - check if the deployment was successfull and if the Ingress Controller Service has picked up an external IP

       kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/cloud/deploy.yaml
       kubectl get all -n ingress-nginx
