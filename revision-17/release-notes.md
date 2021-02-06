# Rev. 17 Release Notes

## Kubernetes k8s Cluster

- Rebuilt the cluster with 1 Master and 3 Nodes maximizing vCPU and Memory for all nodes 

## k3s and k8s k8s-bigip-ctrl configMap Cleaning

- Changed naming in all k3s services in udf-services partition
- Changed configMap Services organization to allocate all VS for a single applicaition in the same application declaration
- Changed naming in k8s services in performance-monitoring partition
- Changed configMap Services organization to allocate all VS for prometheus in the same application declaration
- Added Prometheus-k8s Access Method to UDF BIG-IP component

## Ansible playbooks

- Added 98.deploy-all-usecases.yaml to deploy all use cases in one shot
- Added 99.undeploy-all-usecases.yaml to undeploy all use cases and go back to the initial status of the deployment

### How to deploy all use cases

Connect with ssh to infra-server

```
cd ~/f5-udf-k8s-deployments/ansible
ansible-playbook playbooks/98.deploy-all-usecases.yaml
```

### How to deploy all use cases

Connect with ssh to infra-server

```
cd ~/f5-udf-k8s-deployments/ansible
ansible-playbook playbooks/99.undeploy-all-usecases.yaml
```

