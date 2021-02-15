# Hackazon - Unsecure E-Commerce app

## Use Case Description
### What is Hackazon
In this use case you are going to deploy an e-commerce application called Hackazon ([Hackazon original installation guide ](https://docs.rapid7.com/appspider/hackazon-installation-guide/) - [Source code](https://github.com/rapid7/hackazon))

Hackazon is a free, vulnerable test site that is an online storefront built with the same technologies used in today’s rich client and mobile applications. Hackazon has an AJAX interface, strict workflows and RESTful API’s used by a companion mobile app providing uniquely-effective training and testing ground for IT security professionals. And, it’s full of your favorite vulnerabilities like SQL Injection, cross-site scripting and so on.

Today’s web and mobile applications as well as web services have a host of new technologies that are not being adequately tested for security vulnerabilities. It is critical for IT security professionals to have a vulnerable web application to use for testing the effectiveness of their tools and for honing their skills.

Hackazon enables users to configure each area of the application in order to change the vulnerability landscape to prevent “known vuln testing” or any other form of ‘cheating.’ Since the application includes RESTful interfaces that power AJAX functionality and mobile clients (JSON, XML, GwT, and AMF), users will need to the latest application security testing tools and techniques to discover all the vulnerabilities. Hackazon also requires detailed testing of strict workflows, like shopping carts,that are commonly used in business applications.

### What we are going to deploy in Kubernetes and BIG-IP
In kubernetes we are going to create a new **namespace** called "e-commerce" which is hosting all the ingrastructure needed to publish e-commerce related applications to the external world. The first application that we are deploying there is Hackazon, which is going to need a DB on MySql and a persistent directory to host web content. We are going to need a persistent storage solution for kubernetes and we are implementing this through Peristent Volumes (PV) and Persistent Volume Clains (PVC) pointing to the external NFS server running on infra-server.f5-udf.com

In order to manage Hackazon customers traffic (North-South traffic) we are using a  NGINX Plus Kubernetes Ingress Controller dedicated to the "e-commerce" namespace. This NGINX Plus KIC is going to expose ports 80, 443 and 8080 with ClusterIP services and we are publishing these services auto-configuring BIG-IP as a L4 load balancing thourgh the use of a dedicated CIS configMap 

Hackazon is going to be deployed in the e-commerce namespace with 1 POD, 1 service and 1 VS in NGINX Plus KIC. The FQDN of this application is "hackazon.f5-udf.com"

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
cd ~/k8s-manifests/e-commerce/0.namespace-preparation/
ka 0.namespace.yaml
ka 1.nfs-pv-pvc.yaml
ka 2.nginx-plus-common.yaml
ka 3.nplus.yaml
ka 4.bigip-ctlr-configMap.yaml
ka 5.mysql.yaml
```

### Dev1 deploying the Hackazon application
Connect to infra-server as uer "ubuntu":

```
cd ~/k8s-manifests/e-commerce/hackazon/
ka 1.hackazon.yaml
ka 2.hackazon-services.yaml
ka 3.hackazon-vs.yaml
```

## Deploying with automation with Ansible Playbooks

### DevOps1 Architect deploying the e-commerce namespace
Connect to infra-server as user "ubuntu":

```
cd ~/f5-udf-k8s-deployments/ansible/
ansible-playbook playbooks/1.deploy-e-commerce.yaml
```

### Dev1 deploying the Hackazon application
Connect to infra-server as user "ubuntu":

```
cd ~/f5-udf-k8s-deployments/ansible/
ansible-playbook playbooks/1.deploy-e-commerce-hackazon.yaml
```

## Deploying with Gitlab and CI/CD Pipelines
Connect with SSH to infra-server as user "ubuntu".  
Connect with xRDP to the linux-jumphost with the user "user".  

- Open Firefox
  - Open Gitlab and login as "devops1" user
  - Open Rancher GUI and select the "E Commerce" project on udf-k8s cluster

### DevOps1 Architect deploying the e-commerce namespace
On infra-server become user devops1

```
sudo su - devops1
cd e-commerce
touch $(date "+%s") ; git add . ; git commit -a -m "deploying infrastructure" ; git push -o ci.variable="udf_action=deploy"
```

Open project "e-commerce" and click on "CI / CD", where you should see a Pipeline running. At the same time, on the Rancher UI you should see PODs starting

### Dev1 deploying the Hackazon application
On gitlab, logout and login again, this time with the user "dev1"
Connect with SSH to infra-server as user "ubuntu" and become user dev1

```
sudo su - dev1
cd hackazon
touch $(date "+%s") ; git add . ; git commit -a -m "deploying app" ; git push -o ci.variable="udf_action=deploy"
```

On Gitlab, open project "hackazon" and click on "CI / CD", where you should see a Pipeline running. At the same time, on the Rancher UI you should see PODs starting
