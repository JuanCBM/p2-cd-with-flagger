<h1 align="center">Práctica 2. Despliegue continuo 👨🏻‍💻 </h1>

<p align="center">
  <a href="/docs" target="_blank">
    <img alt="Documentation" src="https://img.shields.io/badge/documentation-yes-brightgreen.svg" />
  </a>
  <a href="#" target="_blank">
    <img alt="License: MIT" src="https://img.shields.io/badge/License-MIT-yellow.svg" />
  </a>
</p>

Despliegue continuo Istio con VirtualService y DestinationRule.

## Authors

👤 **JuanCBM**: Juan Carlos Blázquez Muñoz

* Github: [@JuanCBM](https://github.com/JuanCBM)

👤 **mahuerta**: Miguel Ángel Huerta Rodríguez

* Github: [@mahuerta](https://github.com/mahuerta)


# Despliegue continuo con Istio
Hemos elegido la versión con Istio porque con Flagger no hemos conseguido que funcionara correctamente.

## Imágenes a dockerhub:
- [DockerHub repo](https://hub.docker.com/r/juablazmahuerta/book-app)

Generar las imágenes de docker:

```
mvn spring-boot:build-image -DskipTests -Dspring-boot.build-image.imageName=juablazmahuerta/book-app:v1 
mvn spring-boot:build-image -DskipTests -Dspring-boot.build-image.imageName=juablazmahuerta/book-app:v2 
mvn spring-boot:build-image -DskipTests -Dspring-boot.build-image.imageName=juablazmahuerta/book-app:v3 
mvn spring-boot:build-image -DskipTests -Dspring-boot.build-image.imageName=juablazmahuerta/book-app:v4 

```

Push image to repo:

```
docker push docker.io/juablazmahuerta/book-app:v1
docker push docker.io/juablazmahuerta/book-app:v2
docker push docker.io/juablazmahuerta/book-app:v3
docker push docker.io/juablazmahuerta/book-app:v4

```

## Sobre la aplicación

### Deploy V1
1. Arrancamos minikube
```
minikube start --memory 8192 --cpus 4 --driver virtualbox --kubernetes-version v1.21.2 --addons istio-provisioner --addons istio --addons ingress
```

2. Base de datos
```
kubectl apply -f db-deployment.yaml
```

3. Deployment de v1

```
kubectl apply -f service.yaml
```

```
kubectl apply -f deployment-v1.yaml
```

4. Gateway
```
kubectl apply -f gateway.yaml
```

5. Destination rule
```
kubectl apply -f 01-destination-rule.yaml
```

6. Virtual service
```
kubectl apply -f 02-virtual-service-v1.yaml
```

7. Comprobamos el Acceso a la aplicación

```
export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')

export INGRESS_HOST=$(minikube ip)

export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT
```

Petición de obtención de libros:

> curl http://$GATEWAY_URL/api/books/

> curl http://$GATEWAY_URL/api/books/1


**La versión 1 está desplegada y lista para actualizaciones.**

### Deploy V2

Deploy v2:
> kubectl apply -f deployment-v2.yaml

Destination rule:
> kubectl apply -f 03-destination-rule.yaml

Realización de la canary:
- Se despliega comenzando por un 10% de tráfico
- Se sube en intervalos de 10% en 30s
- Se acepta la canary si llegamos al 50% sin problemas

Despliegue de la aplicación 90% de v1 y 10% de v2:

> kubectl apply -f 04-virtual-service-v1-90-v2-10.yaml

Pasados 30 segundos, despliegue de la aplicación 80% de v1 y 20% de v2:

> kubectl apply -f 05-virtual-service-v1-80-v2-20.yaml

Pasados 30 segundos, despliegue de la aplicación 70% de v1 y 30% de v2:

> kubectl apply -f 06-virtual-service-v1-70-v2-30.yaml

Pasados 30 segundos, despliegue de la aplicación 60% de v1 y 40% de v2:

> kubectl apply -f 07-virtual-service-v1-60-v2-40.yaml

Pasados 30 segundos, despliegue de la aplicación 50% de v1 y 50% de v2:

> kubectl apply -f 08-virtual-service-v1-50-v2-50.yaml

Todo el tráfico a v2:

> kubectl apply -f 09-virtual-service-v2.yaml

Destination rule v2
> kubectl apply -f 10-destination-rule-v2.yaml

Borramos el despliegue de v1:
> kubectl delete -f deployment-v1.yaml

Petición de obtención de libros:

> curl http://$GATEWAY_URL/api/books/

> curl http://$GATEWAY_URL/api/books/1


**Se podría aplicar en sucesivas versiones**

### Desplegamos v3
> kubectl apply -f deployment-v3.yaml

Virtual service v2 a v3 (Con las configuraciones de pesos)
> kubectl apply -f virtual-service-v2-to-v3.yaml

Destination rule v2 a v3 
> kubectl apply -f destination-rule-v2-to-v3.yaml

Virtual service de v3
> kubectl apply -f virtual-service-v3.yaml

Destination rule v3 
> kubectl apply -f destination-rule-v3.yaml

Delete deployment v2:
> kubectl delete -f deployment-v2.yaml


### Desplegamos v4
> kubectl apply -f deployment-v4.yaml

Virtual service v3 a v4 (Con las configuraciones de pesos)
> kubectl apply -f virtual-service-v3-to-v4.yaml

Destination rule v3 a v4 
> kubectl apply -f destination-rule-v3-to-v4.yaml

Virtual service de v4
> kubectl apply -f virtual-service-v4.yaml

Destination rule v4 
> kubectl apply -f destination-rule-v4.yaml

Delete deployment v3:
> kubectl delete -f deployment-v3.yaml

Tendríamos todo el tráfico en la versión nueva v4.

#### Reference
[github.com/MasterCloudApps/4.4.Despliegue-continuo/tree/master/ejem-4-canary-zerodowntime](https://github.com/MasterCloudApps/4.4.Despliegue-continuo/tree/master/ejem-4-canary-zerodowntime)

#### Istio issue
[https://istio.io/latest/about/faq/security/#mysql-with-mtls](https://istio.io/latest/about/faq/security/#mysql-with-mtls)


