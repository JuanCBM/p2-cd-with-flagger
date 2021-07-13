# Continuous deployment with Istio

- [DockerHub repo](https://hub.docker.com/)
- Meter aqui el repo de dockerHub

## Istio steps

```
minikube start --memory 10500 --cpus 4 --driver virtualbox --kubernetes-version v1.21.2 --addons istio-provisioner --addons istio --addons ingress



kubectl apply -f db-deployment.yaml

kubectl apply -f sidecar.yaml

kubectl apply -f deployment-v1.yaml

kubectl apply -f gateway.yaml

kubectl apply -f destinationrule.yaml

kubectl apply -f virtual-service-v1.yaml

```
**V1 is deployed and ready for version updates**

## V1 TO V2

1 - Deploy new version v2
```
kubectl apply -f deployment-v2.yaml

```
2 - Distribute application 90% to v1 and 10% to v2
```
kubectl apply -f virtual-service-v1-90-v2-10.yaml

```
## V2
1 - Execute the virtual service to only serve to v2:
```
kubectl apply -f deployment-v2.yaml

```
2 - Delete deployment V1
```
kubectl delete deployment practice2-v1

```

```
export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')

31191

export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].nodePort}')

31274

export INGRESS_HOST=$(minikube --profile canary-istio ip)

192.168.99.189

export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT

192.168.99.189:31191

```


```
watch kubectl get pods,services,deployments -n practice

kubectl logs pod/db-7df89cc477-ws7cs -c practice2 -n practice
```

## Reference

[github.com/MasterCloudApps/4.4.Despliegue-continuo/tree/master/ejem-4-canary-zerodowntime](https://github.com/MasterCloudApps/4.4.Despliegue-continuo/tree/master/ejem-4-canary-zerodowntime)

## Istio issue

[https://istio.io/latest/about/faq/security/#mysql-with-mtls](https://istio.io/latest/about/faq/security/#mysql-with-mtls)
