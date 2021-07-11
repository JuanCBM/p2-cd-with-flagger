# ejem-4-canary-zerodowntime

Generar las imágenes de docker:

```
mvn spring-boot:build-image -DskipTests -Dspring-boot.build-image.imageName=juablazmahuerta/book-app:v1
```

# Deploy V1

1. Start Minikube

```
minikube start --profile canary-istio-books --kubernetes-version v1.20.0 --memory=8192 --cpus=4  --driver=virtualbox
```

2. Enable istio-provisioner and istio

```
minikube --profile canary-istio-books addons enable istio-provisioner
minikube --profile canary-istio-books addons enable istio
```

3. Create namespace
```
kubectl create ns practice
```

4. Deploy database deployment:

```
kubectl apply -f db-deployment.yaml
```

5. Deploy application deployment:

```
kubectl apply -f deployment.yaml
```

6. Create istio gateway

```
kubectl apply -f book-app-gateway.yaml
```

7. Create destination rules we will use in this example:

```
kubectl apply -f book-app-destinationrule.yaml
```

8. Create the virtual service:

```
kubectl apply -f virtual-service-v1.yaml
```


$ export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')
$ export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].nodePort}')
$ export INGRESS_HOST=$(minikube ip -p canary-istio-books)
$ export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT


curl http://$GATEWAY_URL/api/book

**V1 is deployed and ready for zerodowntime version updates**

# V1 to V2

1. Deploy new version v2 deployment

```
kubectl apply -f deployment-v2.yaml
```

2. Distribute application 90% to v1 and 10% to v2

```
kubectl apply -f zerodowntime-virtual-service-v1-to-v2.yaml
```

# V2

1. Execute the virtual service to only serve to v2:

```
kubectl apply -f zerodowntime-virtual-service-v2.yaml
```

2. Delete deployment V1

```
kubectl delete deployment zerodowntime-v1
```

# V2 to V3

1. Deploy new version v3 deployment

```
kubectl apply -f deployment-v3.yaml
```

2. Distribute application 90% to v2 and 10% to v3

```
kubectl apply -f zerodowntime-virtual-service-v2-to-v3.yaml
```

# V3

1. Execute the virtual service to only serve to v3:

```
kubectl apply -f zerodowntime-virtual-service-v3.yaml
```

2. Delete deployment v2

```
kubectl delete deployment zerodowntime-v2
```

# V3 to V4

1. Deploy new version v4 deployment

```
kubectl apply -f deployment-v4.yaml
```

2. Distribute application 90% to v3 and 10% to v4

```
kubectl apply -f zerodowntime-virtual-service-v3-to-v4.yaml
```

# V4

1. Execute the virtual service to only serve to v4:

```
kubectl apply -f zerodowntime-virtual-service-v4.yaml
```

2. Delete deployment v3

```
kubectl delete deployment zerodowntime-v3
```

