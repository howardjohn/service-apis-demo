# Istio service-apis demo

This repo provides an example minimal deployment of Istio using the Kubernetes [service-apis](https://github.com/kubernetes-sigs/service-apis).

The demo installs a minimal deployment of Istio as a gateway controller only (no sidecars), an `httpbin` application as a test application, and example configuration to route to the httpbin backend. Additionally, the service-api CRDs are installed, but generally these would be done beforehand.

An Ingress is also setup with similar routing rules.

## Use

```shell
# Install Kubernetes API CRDs. These will not be installed by Istio
kubectl apply -k 'github.com/kubernetes-sigs/service-apis/config/crd?ref=56154e7bfde5ebf1a04f45be7abdf983a11c6a32'
# Deploy Istio
kubectl apply -k 'github.com/howardjohn/service-apis-demo'
# Wait for things to get started
kubectl wait --for=condition=Available deployment --all --timeout=120s -n istio-system
```

### Setup variables

LoadBalancer
```shell
export INGRESS_HOST=$(kubectl -n istio-system get service istio -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
export INGRESS_PORT=$(kubectl -n istio-system get service istio -o jsonpath='{.spec.ports[?(@.name=="http")].port}')
export INGRESS_PORT_HTTPS=$(kubectl -n istio-system get service istio -o jsonpath='{.spec.ports[?(@.name=="https")].port}')
```

NodePort
```shell
export INGRESS_HOST=$(kubectl -n istio-system get pod -lapp=istio -o jsonpath='{.items[0].status.hostIP}')
export INGRESS_PORT=$(kubectl -n istio-system get svc istio -ojsonpath='{.spec.ports[?(@.name=="http")].nodePort}')
export INGRESS_PORT_HTTPS=$(kubectl -n istio-system get svc istio -ojsonpath='{.spec.ports[?(@.name=="https")].nodePort}')
```

## Simple Example

This deploys a single deployment and Service, with a simple route exposing it over HTTP and HTTPS. A demo self signed certificate is provided.

```shell
kubectl apply -k 'github.com/howardjohn/service-apis-demo/httpbin'
kubectl wait --for=condition=Available deployment --all --timeout=120s -n service-apis-demo
```

Send a request:
```shell
# HTTP, to Gateway
curl http://$INGRESS_HOST:$INGRESS_PORT/get -H "Host: gateway.local"
# HTTPS, to Gateway
curl --resolve gateway.local:$INGRESS_PORT_HTTPS:$INGRESS_HOST https://gateway.local:$INGRESS_PORT_HTTPS/get -k
# HTTP, to Ingress
curl http://$INGRESS_HOST:$INGRESS_PORT/get -H "Host: ingress.local"
```

## Traffic Splitting


This deploys a single deployment and Service, with a simple route exposing it over HTTP and HTTPS. A demo self signed certificate is provided.

```shell
kubectl apply -k 'github.com/howardjohn/service-apis-demo/split'
kubectl wait --for=condition=Available deployment --all --timeout=120s -n team1
kubectl wait --for=condition=Available deployment --all --timeout=120s -n team2
```

Send a request:
```shell
# Team 1 goes to team1 v1
curl -H "host: team1.example.com" http://$INGRESS_HOST:$INGRESS_PORT/ping
# Team 1 with staging header goes to team1 v2
curl -H "host: team1.example.com" -H "env: stage" http://$INGRESS_HOST:$INGRESS_PORT/ping
# Team 1 with canary header goes to team1 v1 (80%) and team1 v2 (20%)
for i in {0..10}; do curl -H "host: team1.example.com" -H "env: canary" http://$INGRESS_HOST:$INGRESS_PORT/ping; echo; done
curl -H "host: team2.example.com" http://$INGRESS_HOST:$INGRESS_PORT/ping
```
