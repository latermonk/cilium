

#  Create a  cilium cni k8s cluster [with kube-proxyß]


kind-c1-config.yaml  
```
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
- role: worker
networking:
  disableDefaultCNI: true
  podSubnet: "10.0.0.0/16"
  serviceSubnet: "10.1.0.0/16"
```


```
kind create cluster --name c1 --config kind-c1-config.yaml
```




```
kubectl config use-context kind-c1
```



```
helm repo add cilium https://helm.cilium.io/
```



```
helm install cilium cilium/cilium --version 1.10.5 \
   --namespace kube-system \
   --set nodeinit.enabled=true \
   --set kubeProxyReplacement=partial \
   --set hostServices.enabled=false \
   --set externalIPs.enabled=true \
   --set nodePort.enabled=true \
   --set hostPort.enabled=true \
   --set cluster.name=c1 \
   --set cluster.id=1
```




---
# Reference:
https://piotrminkowski.com/2021/10/25/kubernetes-multicluster-with-kind-and-cilium/  