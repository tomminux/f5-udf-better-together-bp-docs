# Online Boutique - Microservices app from Google

## Use Case Description
### What is Online Boutique
In this use case you are going to deploy an e-commerce application called Online Boutique ([Google original repo](https://github.com/GoogleCloudPlatform/microservices-demo))

Online Boutique is a cloud-native microservices demo application. Online Boutique consists of a 10-tier microservices application. The application is a web-based e-commerce app where users can browse items, add them to the cart, and purchase them.

Google uses this application to demonstrate use of technologies like Kubernetes/GKE, Istio, Stackdriver, gRPC and OpenCensus. This application works on any Kubernetes cluster, as well as Google Kubernetes Engine. It’s easy to deploy with little to no configuration.

### What we are going to deploy in Kubernetes and BIG-IP
In kubernetes we are going to create a new **namespace** called "online-boutique" which is hosting all the ingrastructure needed to publish this application to the external world. 

In order to manage Online Boutique customers traffic (North-South traffic) we are using a  NGINX Plus Kubernetes Ingress Controller dedicated to the "online-boutique" namespace. This NGINX Plus KIC is going to expose ports 80, 443 and 8080 with ClusterIP services and we are publishing these services auto-configuring BIG-IP as a L4 load balancing thourgh the use of a dedicated CIS configMap 

Online Boutique is going to be deployed in the online-boutique namespace with 10 PODs, services and 1 VS in NGINX Plus KIC addressing the "frontend" POD. The FQDN of this application is "online-boutique.f5-udf.com"

## The Role play
Generally speaking, all use cases are aarchitected so that each one of them will live is its own namespace. This way it is possible to have a situation where all the 5 use cases are running together, showing the multitenancy of Kubernetes and the "per-namespace" deployment of NGINX Plus Kubernetes Ingress Controller. 

Each use case, in fact, requires its own namespace, with some pre-defined resources avilable. This is where a devops architect or an infrastruture opertor should operate to deploy:

- the namespace itself
- the PV / PVC needed to use persistent storage on NFS Server (if needed)
- the NGINX Kubernetes Ingress Controller
- The BIG-IP Cis AS3 Config Map to link BIG-IP with NGINX KIC
- The mysql POD (if needed)

When the namespace is deployed and configured, the fool is for the Developer which is going to deploy the application itself in the namespace, in this case the e-commerce Hackazon

## Deploying manually with kubectl and manifest files

### DevOps1 Architect deploying the e-commerce namespace
Connect to infra-server as uer "ubuntu":

```
cd ~/k8s-manifests/online-boutique/0.namespace-preparation/
ka 0.namespace.yaml
ka 2.nginx-plus-common.yaml
ka 3.nplus-nap.yaml
ka 4.bigip-ctlr-configMap.yaml
```

### Dev1 deploying the Hackazon application
Connect to infra-server as uer "ubuntu":

```
cd ~/k8s-manifests/online-boutique/online-boutique
ka kubernetes-manifests.yaml
```

## Deploying with automation with Ansible Playbooks

### DevOps1 Architect deploying the e-commerce namespace
Connect to infra-server as user "ubuntu":

```
cd ~/f5-udf-k8s-deployments/ansible/
ansible-playbook playbooks/4.deploy-online-boutique.yaml
```

### Dev1 deploying the Hackazon application
Connect to infra-server as user "ubuntu":

```
cd ~/f5-udf-k8s-deployments/ansible/
ansible-playbook playbooks/4.deploy-online-boutique-app.yaml
```

## Deploying with Gitlab and CI/CD Pipelines
Connect with SSH to infra-server as user "ubuntu".  
Connect with xRDP to the linux-jumphost with the user "user".  

- Open Firefox from the UDF interface, access menu on BIG-IP
  - Open Gitlab and login as "devops1" user
  - Open Rancher GUI and select the "E Commerce" project on udf-k8s cluster

### DevOps1 Architect deploying the e-commerce namespace
On infra-server become user devops1

```
sudo su - devops1
cd online-boutique
touch $(date "+%s") ; git add . ; git commit -a -m "deploying infrastructure" ; git push -o ci.variable="udf_action=deploy"
```

Open project "e-commerce" and click on "CI / CD", where you should see a Pipeline running. At the same time, on the Rancher UI you should see PODs starting

### Dev1 deploying the Hackazon application
On gitlab, logout and login again, this time with the user "dev1"
Connect with SSH to infra-server as user "ubuntu" and become user dev1

```
sudo su - dev1
cd online-boutique
touch $(date "+%s") ; git add . ; git commit -a -m "deploying app" ; git push -o ci.variable="udf_action=deploy"
```

On Gitlab, open project "hackazon" and click on "CI / CD", where you should see a Pipeline running. At the same time, on the Rancher UI you should see PODs starting
