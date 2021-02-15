# Rev. 19 Activity Log

## Adding Firefox in a browser POD in k3s

We are going to run a new POD in k3s cluster on infra-server in order to run Firefox in a browser. This will be published in UDF through a VS on the BIG-IP using a new declaration in CIS

    kubectl config use-context k3s
    cd k3s-manifests/udf-services/
    ka 7.firefox.yaml
    ka 9.bigip-ctlr-configMap.yaml
    
A new access method called "Firefox" was added to BIG-IP Access Methods

## Adding Aspen Mesh configuration in istio-system

### Changes to kube-apiserver.yaml
In order to run Aspen Mesh on our kubernetes cluster in UDF, we need to change the configuration of the Kubernetes' API Server, editing the following file **on k8s-master node**:

    vim /etc/kubernetes/manifests/kube-apiserver.yaml
    
and adding the following commmand line arguments in the POD's configuration:

    - --service-account-issuer=kubernetes.default.svc
    - --api-audiences=api,istio-ca
    - --service-account-key-file=/etc/kubernetes/pki/etcd/server.key
    - --service-account-signing-key-file=/etc/kubernetes/pki/etcd/server.key
    
### Aspen Mesh installation

- Added the istio-namespace
- Added the storageClass infra-server-nfs-server-istio with nfs-provisioner

We are going to use Bart Van Bos's repository for an Aspen Mesh deployment using less resource, this will help in order to save $$$ in UDF: [Bart Van Bos's repository for an Aspen Mesh](https://github.com/CloudDevOpsEMEA/udf-aspenmesh-k8s-slim). This was forked by Paolo Arcagni to add configuration needed to integrate Aspen Mesh Ingress with BIG-IP CIS

    git clone https://github.com/tomminux/udf-aspenmesh-k8s-slim.git
    kubectl config use-context k8s
    kubectl apply -f ~/k8s-manifests/istio-system/0.namespace-preparation/0.namespace.yaml
    kubectl apply -f ~/k8s-manifests/istio-system/0.namespace-preparation/1.storageClass.yaml
    kubectl apply -f ~/k8s-manifests/istio-system/0.namespace-preparation/2.pvc.yaml
    
- Added service-80.yaml and service-443.yaml to istio-ingress and modified service.yaml

```   
ls -al ~/udf-aspenmesh-k8s-slim/aspenmesh-1.6.14-am2/manifests/charts/gateways/istio-ingress/templates/
-rw-rw-r-- 1 ubuntu ubuntu  1471 Feb 14 18:11 service-443.yaml
-rw-rw-r-- 1 ubuntu ubuntu  1468 Feb 14 18:11 service-80.yaml
-rw-rw-r-- 1 ubuntu ubuntu  1464 Feb 14 18:11 service.yaml
```
    
- Re-configured ~/udf-aspenmesh-k8s-slim/aspenmesh-1.6.14-am2/manifests/charts/gateways/istio-ingress/values.yam



