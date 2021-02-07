# Hackazon - Unsecure E-Commerce app Protected with NGINX App Protect Sidecar Proxy 

## Use Case Description

### What is Hackazon

PLease see the description on the original use case [here](https://github.com/tomminux/f5-udf-better-together-bp-docs/blob/main/use-cases/e-commerce-hackazon.md#what-is-hackazon)

### What we are going to deploy in Kubernetes and BIG-IP

We are using the same namespace calle **e-commerce** we already deployed and used in the original Hackazon use case; please, follow the steps already described [here](https://github.com/tomminux/f5-udf-better-together-bp-docs/blob/main/use-cases/e-commerce-hackazon.md#deploying-manually-with-kubectl-and-manifest-files) to deploy the "e-commerce" namespace and, if needed, the unprotected Hackazon POD.

## The Role play

In this use case, as explained, the namespace is already there and we are playing the role of a Developer who is going to release his / her application (Hackazon) in production, and decided to protect him with a NGINX App Protect (NAP) sidecar proxy. We are deploying 1 POD with 2 contaioners, the fisrt one is NAP which will reverse proxy traffic to a local to the POD web server delivering Hackazon.

## Deploying manually with kubectl and manifest files

**Please be sure to have the "e-commerce" namespace already deployed. If not, please follow the steps already described [here](https://github.com/tomminux/f5-udf-better-together-bp-docs/blob/main/use-cases/e-commerce-hackazon.md#deploying-manually-with-kubectl-and-manifest-files)**

### Dev1 deploying the Hackazon application

As you will see, we are deployng a configMap first: this is where we declare the sidecar reverse proxy configuration and the NAP Policy to be used. As a second step we deploy the POD with the two containers, the services and the VS for the NGINX Plus Kubernetes Ingress Controller. 

Connect to infra-server as uer "ubuntu":

```
cd ~/k8s-manifests/e-commerce/hackazon-nap/
ka 1.configmap-nginxconf.yaml
ka 2.hackazon-nap.yaml
```

## Deploying with automation with Ansible Playbooks

### Dev1 deploying the Hackazon application
Connect to infra-server as user "ubuntu":

```
cd ~/f5-udf-k8s-deployments/ansible/
ansible-playbook playbooks/1.deploy-e-commerce-hackazon-nap.yaml
```

## Deploying with Gitlab and CI/CD Pipelines
Connect with SSH to infra-server as user "ubuntu".  
Connect with xRDP to the linux-jumphost with the user "user".  

- Open Firefox
  - Open Gitlab and login as "devops1" user
  - Open Rancher GUI and select the "E Commerce" project on udf-k8s cluster

### Dev1 deploying the Hackazon application
On gitlab, logout and login again, this time with the user "dev1"
Connect with SSH to infra-server as user "ubuntu" and become user dev1

```
sudo su - dev1
cd hackazon-nap
touch $(date "+%s") ; git add . ; git commit -a -m "deploying app" ; git push -o ci.variable="udf_action=deploy"
```

On Gitlab, open project "hackazon-nap" and click on "CI / CD", where you should see a Pipeline running. At the same time, on the Rancher UI you should see PODs starting
