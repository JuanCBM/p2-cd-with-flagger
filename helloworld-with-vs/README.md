# ejem-4-canary-zerodowntime

# Deploy V1

1. Start Minikube

```
minikube start --profile canary-istio --kubernetes-version v1.20.0 --memory=8192 --cpus=4  --driver=virtualbox
```


minikube addons enable ingress -p canary-istio
minikube addons enable istio-provisioner -p canary-istio
minikube addons enable istio -p canary-istio



kubectl apply -f deplyment.yaml
kubectl apply -f service.yaml
kubectl apply -f gateway.yaml
kubectl apply -f virtual-service-v1.yaml


$ export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')
$ export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].nodePort}')
$ export INGRESS_HOST=$(minikube ip --profile canary-istio ip)
$ export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT

curl http://$GATEWAY_URL/api/books