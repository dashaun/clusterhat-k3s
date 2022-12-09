# clusterhat-k3s

## Prerequisite

Clusterhat v2.5 deployment
 - See this article https://dashaun.com/posts/k3s-on-raspberry-pi-clusterhat/
 - Use CBRIDGE instead of CNAT

## Setup the k3s cluster

```bash
export CONTROL_NODE_IP=10.99.1.112
k3sup install --ip $CONTROL_NODE_IP --user pi --k3s-extra-args '--disable traefik' --merge --local-path ~/.kube/config --context clusterhat01
k3sup join --ip 10.99.1.70 --server-ip $CONTROL_NODE_IP --user pi
k3sup join --ip 10.99.1.71 --server-ip $CONTROL_NODE_IP --user pi
k3sup join --ip 10.99.1.72 --server-ip $CONTROL_NODE_IP --user pi
k3sup join --ip 10.99.1.73 --server-ip $CONTROL_NODE_IP --user pi
```

## Install Knative

```bash
#kubectl apply -f https://github.com/knative/serving/releases/download/knative-v1.8.0/serving-crds.yaml
kubectl apply -f 1-serving-crds.yaml
#kubectl apply -f https://github.com/knative/serving/releases/download/knative-v1.8.0/serving-core.yaml
kubectl apply -f 2-serving-core.yaml
#kubectl apply -f https://raw.githubusercontent.com/dashaun/clusterhat-k3s/main/contour.yaml
kubectl apply -f 3-contour.yaml
#kubectl apply -f https://github.com/knative/net-contour/releases/download/knative-v1.8.0/net-contour.yaml
kubectl apply -f 4-net-contour.yaml
#kubectl patch configmap/config-network \
#  --namespace knative-serving \
#  --type merge \
#  --patch '{"data":{"ingress-class":"contour.ingress.networking.knative.dev"}}'
```

## Magic DNS with SSLIP

```bash
kubectl apply -f https://github.com/knative/serving/releases/download/knative-v1.8.1/serving-default-domain.yaml
```

## Install Cert Manager and Knative integration

```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.10.1/cert-manager.yaml
kubectl apply -f https://github.com/knative/net-certmanager/releases/download/knative-v1.8.0/release.yaml
# Replace knative.example.com with your domain suffix
#kubectl patch configmap/config-domain \
#  --namespace knative-serving \
#  --type merge \
#  --patch '{"data":{"kn.cluster00.dashaun.dev":""}}'

```

## Auto TLS

```bash
kubectl apply -f 7-cluster-issuer.yaml
```

## Deploy our service

```bash
kubectl apply -f 8-spring-boot-native-pi-service.yaml
```

## Setup Cloudflared with Envoy

The secret:

```bash
kubectl create secret generic tunnel-credentials --from-file=credentials.json=/Users/dashaunc/.cloudflared/68d2bff7-197c-4f8f-a4c6-cd484cf08272.json
```

```bash
kubectl apply -f https://raw.githubusercontent.com/dashaun/clusterhat-k3s/main/cloudflared.yaml
```
