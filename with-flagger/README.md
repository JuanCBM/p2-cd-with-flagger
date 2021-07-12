

> minikube start --memory 100000 --cpus 4 --profile flagger --driver virtualbox --kubernetes-version v1.20.2

#### instalación



#### Activamos
> minikube --profile flagger addons enable istio-provisioner

> minikube --profile flagger addons enable istio

> minikube --profile flagger addons enable metrics-server


#### Instalamos
> helm repo add flagger https://flagger.app

> kubectl apply -f https://raw.githubusercontent.com/weaveworks/flagger/master/artif
acts/flagger/crd.yaml

> helm upgrade -i flagger flagger/flagger --namespace=istio-system --set crd.create=false --set meshProvider=istio --set metricsServer=http://prometheus:9090

> helm upgrade -i flagger-grafana flagger/grafana --namespace=istio-system --set url=http://prometheus.istio-system:9090 --set user=admin --set password=change-me






¿??¿?¿
# Desplegamos el horizontal pod autoscaler
> kubectl apply -k github.com/weaveworks/flagger//kustomize/podinfo


¿?¿¿?¿?
Desplegamos el servicio para generación de carga
$ kubectl apply -k github.com/weaveworks/flagger//kustomize/tester








### EJECUCIÓN DE FLYWAY


> kubectl create ns flyway


#### Despliegue de la BBDD
> kubectl apply -n flyway -f db-deployment.yaml 


#### Activamos la inyección del sidecar de istio en el namespace test
> kubectl label namespace flyway istio-injection=enabled


Nos va a ir diciendo cada los pods que se crean

> watch kubectl get pod,deployment,service,horizontalpodautoscaler,destinationrule,virtualservice,canary -n flyway



#### Aplicamos el deployment
> kubectl apply -f db-deployment.yaml 

> kubectl apply -f deployment.yaml 


Vemos los logs del pod, al haber dos contenedores dentro del pod, debemos elegir el contendor concreto con -c XXXXX

> kubectl logs pod/db-797b84c6bc-2tzq5 -n flyway -c zerodowntime  | tail 


Nos va a ir diciendo cada X tiempo si se están pasando de una a otra
> watch 'kubectl describe canary/zerodowntime -n flyway | tail'





> kubectl apply -f gateway.yaml 


> kubectl apply -f gateway.yaml

> kubectl apply -f latency-metric-template.yaml