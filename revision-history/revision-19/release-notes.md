# Rev. 19 Release Notes

## Kubernetes k8s Cluster

- Aspen Mesh added and ready to be deployed
- API Server was reconfigured to be read to accept Aspen Mesh Installation, see [kubeadm detailed instructions](https://my.aspenmesh.io/client/docs/1.6/operations/configuring-third-party-token-projection/#kubeadm)
- /home/ubuntu/udf-aspenmesh-k8s-slim directory added to be able to install Aspen Mesh with scripts

## infra-server 

- Firefox in a browser added, please see Access Methods from BIG-IP Components: you can use this Firefox in a browser on your Mac or Win box to browse things as you were on the jumphost
- Aspen Mesh Dashboard access method added
- BIG-IP CIS ready to forward traffic to Aspen Mesh Ingress Controller

## Aspen Mesh Deployment Procedure

```
cd ~/udf-aspenmesh-k8s-slim
make namespace-preparation
make install-am
make post-install
```

## Online Boutique - Aspem Mesh enabled

Online boutique reconfigured to run in a istio-enabled namespace:

    cd /home/ubuntu/k8s-manifests/online-boutique-am 
    ka 0.namespace.yaml
    ka 1.kubernetes-manifests.yaml
    ka 2.frontend-gateway.yaml