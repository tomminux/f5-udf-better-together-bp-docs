# Rev. 17 Activity Log

## k8s rebuilding

We need to delete the kubernetes cluster and re-build it with more vCPU and memory on each node and addinig a new node

Deleting the old cluster:

- Deleted k8s-master
- deleted k8s-node1
- deleted k8s-node2

Adding boxes to create a new cluster:

- Added k8s-master, type c4.2xlarge, 50GB vDisk, bound network interface on 10.1.20.60
- Added k8s-node1, type c4.2xlarge, 50GB vDisk, bound network interface on 10.1.20.61
- Added k8s-node2, type c4.2xlarge, 50GB vDisk, bound network interface on 10.1.20.62
- Added k8s-node3, type c4.2xlarge, 50GB vDisk, bound network interface on 10.1.20.63

Execute Ansible Playbook to build the cluster:

    git clone https://github.com/tomminux/f5-udf-k8s-cluster.git
    cd ~/f5-udf-k8s-cluster/ansible
    ansible-playbook playbooks/install-k8s-cluster.yaml
    
Disconnect and reconnect to the infra-server

    cd ~/f5-udf-infra-server
    bash init-scripts/3.create-kubeconfig.sh

Be sure everything is running fine with the command:

    k9s
    
now connect to the BIG-IP and be sure the BGP configuration is like this one  

```
!
no service password-encryption
!
router bgp 64512
 bgp graceful-restart restart-time 120
 neighbor calico-k3s peer-group
 neighbor calico-k3s remote-as 64512
 neighbor calico-k8s peer-group
 neighbor calico-k8s remote-as 64512
 neighbor 10.1.20.20 peer-group calico-k3s
 neighbor 10.1.20.60 peer-group calico-k8s
 neighbor 10.1.20.61 peer-group calico-k8s
 neighbor 10.1.20.62 peer-group calico-k8s
 neighbor 10.1.20.63 peer-group calico-k8s
!
line con 0
 login
line vty 0 39
 login
!
end
```

and verify BGP is distributing 10.244.0.0/16 routes towards k8s nodes

```
imish
ena
show ip route
```

### Deploy k8s-bigip-ctlr and Prometheus-k8s

#### BIG-IP CIS Controller in k8s

First of all, ensure you are working in the k8s context

    kubectl config use-context k8s
    
and deploy the namespace, CDRs and the k8s-bigip-ctlr

```
cd ~/k8s-manifests/ingress-services/
ka 0.namespace.yaml
ka 1.nginx-plus-common.yaml
ka 2.bigip-ctlr-common.yaml
ka 3.bigip-ctlr-deployment.yaml
```

Deploy performance-monitoring namespace and prometheus

```
cd ~/k8s-manifests/performance-monitoring/0.namespace-preparation/
ka 0.namespace.yaml
ka 1.nfs-pv-pvc.yaml
cd ~/k8s-manifests/performance-monitoring/
ka 1.prometheus.yaml
ka 9.bigip-ctlr-configMap.yaml
```
