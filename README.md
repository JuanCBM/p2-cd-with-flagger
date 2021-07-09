

> minikube start --memory 11000 --cpus 4 --profile flagger --driver virtualbox --kubernetes-version v1.20.2
> 
> minikube --profile flagger addons enable istio-provisioner
> 
> minikube --profile flagger addons enable istio

#### Por si queremos métricas
> minikube --profile flagger addons enable metrics-server


Instalación
> helm repo add flagger https://flagger.app

> kubectl apply -f https://raw.githubusercontent.com/weaveworks/flagger/master/artifacts/flagger/crd.yaml

> helm upgrade -i flagger flagger/flagger \
 --namespace=istio-system \
 --set crd.create=false \
 --set meshProvider=istio \
 --set metricsServer=http://prometheus:9090


Flagger viene con un dashboard específico de Grafana para monitorizar las canaries, para activarlo:
> helm upgrade -i flagger-grafana flagger/grafana \
 --namespace=istio-system \
 --set url=http://prometheus.istio-system:9090 \
 --set user=admin \
 --set password=change-me
#### Verificar que funciona haciendo port forwarding del puerto de Grafana:
> kubectl -n istio-system port-forward svc/flagger-grafana 3000:80
#### Abrir la url http://localhost:3000






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
> kubectl apply -f deployment.yaml 


Vemos los logs del pod, al haber dos contenedores dentro del pod, debemos elegir el contendor concreto con -c XXXXX

> kubectl logs pod/XXXX -n flyway -c zerodowntime  | tail 








> kubectl apply -f canary.yaml 


Nos va a ir diciendo cada X tiempo si se están pasando de una a otra
> watch 'kubectl describe canary/zerodowntime -n flyway | tail'





> kubectl apply -f gateway.yaml 
