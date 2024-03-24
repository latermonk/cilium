#  Kind cluster with Cilium and no kube-proxy

## Create a kind k8s cluster

```
kind delete cluster && kind create cluster --config - <<EOF
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
networking:
  disableDefaultCNI: true
  kubeProxyMode: none
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    listenAddress: 127.0.0.1
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    listenAddress: 127.0.0.1
    protocol: TCP
- role: worker
- role: worker
- role: worker
EOF
```


```
kubectl cluster-info --context kind-kind
```

## Install Cilium with Hubble UI enabled and accessible under hubble-ui.127.0.0.1.nip.io ingress

```
helm upgrade --install --namespace kube-system --repo https://helm.cilium.io cilium cilium --values - <<EOF
kubeProxyReplacement: strict
k8sServiceHost: kind-control-plane
k8sServicePort: 6443
hostServices:
  enabled: false
externalIPs:
  enabled: true
nodePort:
  enabled: true
hostPort:
  enabled: true
image:
  pullPolicy: IfNotPresent
ipam:
  mode: kubernetes
hubble:
  enabled: true
  relay:
    enabled: true
  ui:
    enabled: true
    ingress:
      enabled: true
      annotations:
        kubernetes.io/ingress.class: nginx
      hosts:
        - hubble-ui.127.0.0.1.nip.io
EOF
```


# Install ingress-nginx ingress controller
```
kubectl apply -f  deploy.yaml
```

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```



##  test with hubble-ui
http://hubble-ui.127.0.0.1.nip.io   




# kind install metallb

```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.7/config/manifests/metallb-native.yaml
```


```
kubectl wait --namespace metallb-system \
                --for=condition=ready pod \
                --selector=app=metallb \
                --timeout=90s
```

```
docker network inspect -f '{{.IPAM.Config}}' kind
```

metallb-config.yaml
```
kubectl apply -f - <<EOF
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: example
  namespace: metallb-system
spec:
  addresses:
  - 192.168.228.200-192.168.228.250
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: empty
  namespace: metallb-system
EOF
```

```
kubectl apply -f  above-metallb-config.yaml

```


Test:
```
kubectl apply -f https://kind.sigs.k8s.io/examples/loadbalancer/usage.yaml
```


```
LB_IP=$(kubectl get svc/foo-service -o=jsonpath='{.status.loadBalancer.ingress[0].ip}')
```

```
# should output foo and bar on separate lines 
for _ in {1..10}; do
  curl ${LB_IP}:5678
done
```





---
# Reference:
https://medium.com/@charled.breteche/kind-cluster-with-cilium-and-no-kube-proxy-c6f4d84b5a9d     

https://kind.sigs.k8s.io/docs/user/ingress/  

https://github.com/GoogleCloudPlatform/microservices-demo  

https://docs.cilium.io/en/latest/network/kubernetes/kubeproxy-free/  

https://kind.sigs.k8s.io/docs/user/loadbalancer/  

