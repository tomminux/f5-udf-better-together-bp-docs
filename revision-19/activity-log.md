# Rev. 19 Activity Log

## Adding Firefox in a browser POD in k3s

We are going to run a new POD in k3s cluster on infra-server in order to run Firefox in a browser. This will be published in UDF through a VS on the BIG-IP using a new declaration in CIS

    kubectl config use-context k3s
    cd k3s-manifests/udf-services/
    ka 7.firefox.yaml
    ka 9.bigip-ctlr-configMap.yaml
    
A new access method called "Firefox" was added to BIG-IP Access Methods

## Adding Aspen Mesh configuration in istio-system

- Added the istio-namespace
- Added the storageClass infra-server-nfs-server-istio with nfs-provisioner

We are going to use Bart Van Bos's repository for an Aspen Mesh deployment using less resource, this will help in order to save $$$ in UDF: [Bart Van Bos's repository for an Aspen Mesh](https://github.com/CloudDevOpsEMEA/udf-aspenmesh-k8s-slim). This was forked by Paolo Arcagni to add configuration needed to integrate Aspen Mesh Ingress with BIG-IP CIS

    git clone https://github.com/tomminux/udf-aspenmesh-k8s-slim.git
    kubectl config use-context k8s
    kubectl apply -f ~/k8s-manifests/istio-system/0.namespace-preparation/0.namespace.yaml
    kubectl apply -f ~/k8s-manifests/istio-system/0.namespace-preparation/1.storageClass.yaml
    kubectl apply -f ~/k8s-manifests/istio-system/0.namespace-preparation/2.pvc.yaml
    