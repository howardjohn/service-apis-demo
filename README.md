# Istio service-apis demo

This repo provides an example minimal deployment of Istio using the Kubernetes [service-apis](https://github.com/kubernetes-sigs/service-apis).

The demo installs a minimal deployment of Istio as a gateway controller only (no sidecars), an `httpbin` application as a test application, and example configuration to route to the httpbin backend. Additionally, the service-api CRDs are installed, but generally these would be done beforehand.

An Ingress is also setup with similar routing rules.

## Use

```
# Install Kubernetes API CRDs. These will not be installed by Istio
kubectl apply -k github.com/kubernetes-sigs/service-apis/config/crd?ref=b9010cfacdbeddd45d8aba22f7eeb9ffc341f930
kubectl apply -k github.com/howardjohn/service-apis-demo
kubectl wait --for=condition=Available deployment --all --timeout=120s -n istio-system
```

Send a request (LoadBalancer):
```shell
export INGRESS_HOST=$(kubectl -n istio-system get service istio -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
export INGRESS_PORT=$(kubectl -n istio-system get service istio -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
curl $INGRESS_HOST:$INGRESS_PORT/get -H "Host: gateway.local"
curl $INGRESS_HOST:$INGRESS_PORT/get -H "Host: ingress.local"
```

Send a request (NodePort):
```shell
export INGRESS_HOST=$(kubectl -n istio-system get pod -lapp=istio -o jsonpath='{.items[0].status.hostIP}')
export INGRESS_PORT=$(kubectl -n istio-system get svc istio -ojsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')
curl $INGRESS_HOST:$INGRESS_PORT/get -H "Host: gateway.local"
curl $INGRESS_HOST:$INGRESS_PORT/get -H "Host: ingress.local"
```
