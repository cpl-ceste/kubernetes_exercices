# EJERCICIO 2 - KUBERNETES: DESPLIEGUE DE APPLICACION

Documentacion de referencia [Kubernetes Docs](https://kubernetes.io/docs/home/)

Desplegaremos Wordpress con MySQL

## Crear un SECRETO

Crear Secretos para el paso seguro de info.


1) Codificar la info en base64 (opcion -n para no incluir newline en la codificacion)

`$ echo -n "app-db" | base64`

`$ echo -n "app-user" | base64`

`$ echo -n "app-pass" | base64`

`$ echo -n "app-rootpass" | base64`

2) Crear el secreto

`$ kubectl create -f app-secret.yaml`

`$ kubectl get secrets`

## Crear un VOLUMEN PERSISTENTE

Desplegamos un volumen para almacenar los datos de MySQL de forma persistente

1) Creamos un volumen en el host

`$ sudo mkdir -p /data/app`

2) Creamos el volumen persistente

`$ kubectl create -f app-pv.yaml`

`$ kubectl describe pv/app-pv`

## Hacer un CLAIM de un VOLUMEN PERSISTENTE

1) Crear el claim. En el yaml de creacion se especifica la etiqueta del volumen creado anteriormente

`$ kubectl create -f app-pvc.yaml`


## Desplegar MySQL

1) Creamos el despliegue

`$ kubectl create -f mysql-deployment.yaml`

2) Inspeccionamos que todo esta correcto

`$ kubectl get pv`

`$ kubectl get pvc`

`$ kubectl get deployments`

## Crear un servicio para MySQL

1) Los pods son efímeros, para conectarnos a la Base de Datos necesitamos una IP que este desacoplada de los pods cuya IP puede cambiar cada vez que se crean. Para ello crearemos un servicio `mysql-service` que permitirá que la IP del servicio pueda resolverse desde dentro del DNS del ClusterIP

`$ kubectl create -f mysql-service.yaml`

2) Verificamos que se creo correctamente y que se mapea correctamente con el Pod, porque  las IPs de los Endpoints IP del servicio es la direccion IP del Pod de MySQL

`$ kubectl describe svc/mysql-service`

`$ kubectl get pods -o wide`

## Desplegamos Wordpress

1) Creamos un despliegue de wordpress

`$ kubectl create -f wordpress-deployment.yaml`

2) Verificamos el despliegue

`$ kubectl get deployments`

`$ kubectl get pods`

3) Entramos en uno de los pods con un terminal interactivo. 
Nota: Hemos de poner el nombre del pod con el hash correcto

`$ kubectl exec -it wordpress-deployment-<hash del pod> -- bash`

4) Check the MySQL service can be accessed from the Pod

`$ getent hosts mysql-service`

5) Chequeamos la configuracion de WordPress

`$ grep -i db /var/www/html/wp-config.php`

Vemos que el pod de WordPress se ha configurado a sí mismo utilizando las variables de entorno "inyectadas" en el, utilizando los valores del secreto de Kubernetes que definimos anteriormente.

6) Salimos del Pod

`$ exit`

## Crear un servicio para WordPress

1) Creamos el servicio

`$ kubectl create -f wordpress-service.yaml`

El servicio es de tipo NodePort y será accesible desde la maquina EC2 donde tenemos instalado el cluster de minicube.

Utilizaremos:

- puerto: 80: el puerto interno en el que escucha el servicio dentro del clúster

- nodePort: 30080: el puerto externo de cada nodo que se asigna al puerto interno del servicio, lo que permite el acceso desde fuera del clúster


2) Verificamos el servicio

`$ kubectl describe svc/wordpress-service`

`$ kubectl get pods -o wide`

3) Comporbamos los puertos de comunicacion y accedemos al servicio, conectando con el nodo del cluster (minicube solo tiene un nodo)

`$ export NODE_PORT=$(kubectl get services/wordpress-service -o go-template='{{(index .spec.ports 0).nodePort}}')`

`$ echo NODE_PORT=$NODE_PORT`

Accedemos a la pagina principal de wordpress

`$ curl $(minikube ip):$NODE_PORT`

Si se quiere solo ver el `status_code` de la respuesta

`$ curl -s -o /dev/null -w "%{http_code}" $(minikube ip):$NODE_PORT`

3) Exponemos el NodePort del nodo del cluster dentro del host. El comando `kubectl port-forward` reenvía un puerto local (30080) de la maquina EC2 al puerto interno del servicio (80) para acceder al servicio directamente desde fuera de la maquina EC2.

`$ kubectl port-forward --address 0.0.0.0 svc/wordpress-service --namespace default 30080:80`

3) Con un navegador accedemos al Wordpress site que esta publicado en el cluster de la maquina EC2 desde Internet

`http://<ip EC2>:30080/`

Nota: hay que abrir el puerto 30080 en la maquina EC3 para que pueda ser accesible el servicio desde Internet

5) Borramos los despliegues, el volumen, el reclamo de volumen y los secretos. Para eliminar los recursos en Kubernetes de manera ordenada y correcta, hay que seguir una secuencia que respete las dependencias entre los recursos.

En nuestro caso el orden recomendado para la eliminación sería eliminar los servicios y deployments antes de eliminar los volúmenes persistentes y los secretos:

- Eliminar los servicios primero: Esto evita que haya tráfico hacia los pods mientras se están eliminando.

`$ kubectl delete -f wordpress-service.yaml`

`$ kubectl delete -f mysql-service.yaml`

- Eliminar los deployments: Esto eliminará los pods gestionados por los deployments.

`$ kubectl delete -f wordpress-deployment.yaml`

`$ kubectl delete -f mysql-deployment.yaml`

- Eliminar los persistent volume claims (PVCs): Estos son utilizados por los pods para almacenar datos.

`$ kubectl delete -f app-pvc.yaml`

- Eliminar los persistent volumes (PVs): Esto garantiza que los volúmenes persistentes sean liberados correctamente.

`$ kubectl delete -f app-pv.yaml`

- Eliminar los secretos (Secrets): Estos pueden ser eliminados al final, ya que no afectan el funcionamiento de los demás recursos.

`$ kubectl delete -f app-secret.yaml`