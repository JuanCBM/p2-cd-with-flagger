# Continuous deployment with Istio

- [DockerHub repo](https://hub.docker.com/r/juablazmahuerta/book-app)

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


```
export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')

31191

export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].nodePort}')

31274

export INGRESS_HOST=$(minikube ip)

192.168.99.189

export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT

192.168.99.189:31191

```

Check:

`curl http://$GATEWAY_URL/api/books/`

**V1 is deployed and ready for version updates**

Deploy v2:

`kubectl apply -f deployment-v2.yaml`

Distribute application 90% to v1 and 10% to v2:

`kubectl apply -f virtual-service-v1-90-v2-10.yaml`

After 30 seconds, distribute application 80% to v1 and 20% to v2:

`kubectl apply -f virtual-service-v1-80-v2-20.yaml`

After 30 seconds, distribute application 70% to v1 and 30% to v2:

`kubectl apply -f virtual-service-v1-70-v2-30.yaml`

After 30 seconds, distribute application 60% to v1 and 40% to v2:

`kubectl apply -f virtual-service-v1-60-v2-40.yaml`

After 30 seconds, distribute application 50% to v1 and 50% to v2:

`kubectl apply -f virtual-service-v1-50-v2-50.yaml`

Execute the virtual service to only serve to v2:

`kubectl apply -f virtual-service-v2.yaml`

Delete deployment v1:

`kubectl delete -f deployment-v1.yaml`


```
watch kubectl get pods,services,deployments

kubectl logs pod/db-7df89cc477-ws7cs -c practice2 
```

## Reference

[github.com/MasterCloudApps/4.4.Despliegue-continuo/tree/master/ejem-4-canary-zerodowntime](https://github.com/MasterCloudApps/4.4.Despliegue-continuo/tree/master/ejem-4-canary-zerodowntime)

## Istio issue

[https://istio.io/latest/about/faq/security/#mysql-with-mtls](https://istio.io/latest/about/faq/security/#mysql-with-mtls)
