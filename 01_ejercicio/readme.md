# EJERCICIO 1 - KUBERNETES: INTRO

Documentacion de referencia [Kubernetes Docs](https://kubernetes.io/docs/home/)

Desplegaremos una aplicacion sencilla para interactuar con todos los componenetes basicos de Kubernetes

## Crear entorno de trabajo

### Levantaremos una maquina EC2 en AWS: 

1) Crear connection keys y security group para acceso SSH desde nuestro PC

2) Servidor Amazon Linux 2 (t2.medium: 2vCPU, 2GB free RAM, 20GB free disk)

3) Conectarse con SSH

Primero dar permisos de solo lectura a la llave, porque AWS exige esta medida de seguridad para permitir la conexion

`$ chmod 400 key_file.pem`

Conectarnos con SSH

`$ ssh -i "key_file.pem" ec2-user@ip-publica-ec2`

### Instalamos el sorftware del entorno de trabajo en la maquina

1) Instalamos docker y docker-compose

`$ sudo yum update -y`

`$ sudo yum install docker -y`

`$ sudo systemctl start docker`

`$ sudo systemctl enable docker`

`$ sudo usermod -aG docker $USER && newgrp docker`

`$ sudo mkdir -p /usr/local/lib/docker/cli-plugins`

`$ sudo curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose`

`$ sudo chmod +x /usr/local/bin/docker-compose`

Chequeamos la instalación

`$ docker version`

`$ docker-compose version`

2) Instalamos `kubectl` la aplicacion CLI de acceso a Kubernetes

`$ curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"`

`$ sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl`

Comprobamos instalación

`$ kubectl version --client`

3) Instalaremos **Minicube** (Kubernetes con un cluster de un nodo)
#INSTALAR MINIKUBE

`$ curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64`

`$ sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64`

Comprobamos instalacion

`$ minikube version`

Y si `kubectl`esta conectado al cluster (si el cluster no esta arrancado, veremos que no podemos conectar)

`$ kubectl version`

## Crear el cluster

Al instalarse Minikube se instala Kubernetes con un cluster con solo nodo. Arrancamos el cluster

`$ minikube start`

Hemos instalado tambien `kubectl` herramienta CLI para comunicarnos con Kubernetes. Chequeamos la info del cluster y vemos los nodos (`--help` para ver todos las opciones del comando `get nodes`)

`$ kubectl cluster-info`

`$ kubectl get nodes`

`$ kubectl get nodes --help`

## Desplegar la aplicacion

En el proceso de despliegue Kubernetes realiza las siguientes operaciones:

- buscar un nodo adecuado donde se pueda ejecutar una instancia de la aplicación (solo tenemos 1 nodo disponible)

- programar la aplicación para ejecutarse en ese nodo

- configurar el clúster para reprogramar la instancia en un nuevo nodo cuando sea necesario

1) Desplegamos una aplicacion a partir de una imagen 

`$ kubectl create deployment kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1`

Comprobamos el despliegue, listando los que hay ahora mismo en el cluster

`$ kubectl get deployments`


2) La app esta en el cluster, en una red privada y aislada (despliegue por defecto). 

En otra terminal crear un proxy que expone el endpoint por pod. Usamos el comando `create proxy` (Starting to serve on 127.0.0.1:8001) (Ctrl-C to stop it)

`$ echo -e "Starting Proxy. After starting it will not output a response. Please return to your original terminal window\n“`

`$ kubectl proxy`

3) Accedemos al Pod (API endpoint version) que esta desplegado desde el terminal de la maquina EC2 (server host del cluster)

`$ curl http://localhost:8001/version`

Creamos una variable de entorno para recoger el nombre del Pod

`$ export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')`

`$ echo Name of the Pod: $POD_NAME `

Accedemos al pod (url the route al API del Pod)

`$ curl http://localhost:8001/api/v1/namespaces/default/pods/$POD_NAME`

4) Exploramos la aplicacion

Chequeamos que la app esta corriendo

`$ kubectl get pods`

Vemos los contenedores deentro del Pod y las imagenes que usan

`$ kubectl describe pods`

Accedemos al pod (url the route al API del Pod)

`$ curl http://localhost:8001/api/v1/namespaces/default/pods/$POD_NAME`

Accedemos a los logs

`$ kubectl logs $POD_NAME`

5) Accedemos directamente al Pod y ejecutamos comandos dentro de su contenedor una vez que esta levantado

Listamos las variables de entorno del contenedor que hay en el Pod (como solo hay uno no hay que especificarlo)

`$ kubectl exec $POD_NAME -- env`

Abrimos una terminal bash interactiva dentro del contenedor del Pod y ejecutamos algunos comandos

`$ kubectl exec -ti $POD_NAME -- bash`

`$ cat server.js`

`$ curl localhost:8080`

`$ exit`

## Visualizar el deployment en el Dashboard de Kubernetes

Creamos un tunnel para acceder al Dashboard de Kubernetes de Minikube

1) En una nueva terminal de conexion SSH a la instancia EC2 consultamos la lista de addons de minikube y comprobamos que tenemos el dashboard

`$ minikube addons list`

2) Consultamos la url en la que se expone el dashboard

`$ minikube dashboard --url`

La url será una cadena con el siguiente formato, por ejemplo, donde vemos que el puerto en el que se consume es el 36991 (REMOTE PORT)

`http://127.0.0.1:36991/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/`


3) En una terminal ssh nueva configuramos el tunel ssh para mapear el puerto del servicio de dashboard en nuestra maquina local

`$ ssh -i <EC2_KEY.pem> -N -L <LOCALPORT>:localhost:<DASHBOARD REMOTE PORT> ec2-user@<IP EC2>`

4) Localmente en nuestra maquina podemos acceder ahora a la aplicacion del Dashboard de Kubernetes en el puerto `<LOCALPORT>`

`http://localhost:<LOCALPORT>/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/`


## Exponer o publicar una aplicación

Crear un servicio y exponerlo para que sea acesible desde el exterior del cluster

Chequear que la app esta corriendo (pods running)

`$ kubectl get pods`

Y los servicios actuales

`$ kubectl get services`

Creamos un nuevo servicio con un puerto 8080 y lo exponemos con un NodePort que sera accesible fuera del cluster y mapeara a un puerto dentro del cluster

`$ kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 8080`

Vemos la descripcion del servicio
`$ kubectl describe services/kubernetes-bootcamp`

Vemos la IP y el NodePort de acceso externo del cluster al servicio

`$ export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}')`

`$ echo NODE_PORT=$NODE_PORT`

Accedemos a la app desde la consola de EC2

`$ curl $(minikube ip):$NODE_PORT`

Hacemos un forward del puerto de NodePort al puerto del servicio para poder accederlo directamente desde fuera de la maquina EC2

`$ kubectl port-forward –address 0.0.0.0 svc/kubernetes-bootcamp -- namespace default 8080:8080`

## Trabajar con etiquetas

Veamos los despliegues 

`$ kubectl describe deployment`

Listamos todos los pods que tienen una determinada etiqueta

`$ kubectl get pods -l app=kubernetes-bootcamp`

`$ kubectl get services -l app=kubernetes-bootcamp`

Podemos aplicar nuevas etiquetas
`$ kubectl label pods $POD_NAME version=v1`$ 

Comprobamos que las etiquetas estan asignadas a los pods

`$ kubectl describe pods $POD_NAME`
`$ kubectl get pods -l version=v1`

## Borrar un servicio

Podemos borrar el servicio

`$ kubectl get services`
`$ kubectl delete service -l app=kubernetes-bootcamp`
`$ kubectl get services`

Vemos que ya no esta accesible en el cluster

`$ curl $(minikube ip):$NODE_PORT`

Pero el Pod esta todavia levantado y la app corriendo

`$ kubectl exec -ti $POD_NAME -- curl localhost:8080`

## Escalar un despliegue

Podemos escalar un despliegue con mas de un Pod (replicas)

Recrearemos el servicio
`$ kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 8080`

`$ kubectl get deployments`

Veamos el `ReplicaSet`creado por el despliegue

`$ kubectl get rs`

Escalamos el despliegue a 4 replicas
`$ kubectl scale deployments/kubernetes-bootcamp --replicas=4`
`$ kubectl get deployments`

Veamos el numero de Pods del despliegue

`$ kubectl get pods -o wide`

Veamos los logs del despliegue

`$ kubectl describe deployments/kubernetes-bootcamp`

Veamos el servicio
`$ kubectl describe services/kubernetes-bootcamp`

Pasamos el numero de replicas de 4 a 2

`$ kubectl scale deployments/kubernetes-bootcamp --replicas=2`

`$ kubectl get deployments`

`$ kubectl get pods -o wide`

## Actualizar un despliegue

Listamos los despleigues y pods

`$ kubectl get deployments`

`$ kubectl get pods`

Chequeamos la imagen del despleigue

`$ kubectl describe pods`

Actualizamos la imagen de la aplicacion a la version 2

`$ kubectl set image deployments/kubernetes-bootcamp`

`$ kubernetes-bootcamp=jocatalin/kubernetes-bootcamp:v2`

`$ kubectl get pods`

Chequeamos que la app esta corriendfo

`$ kubectl describe services/kubernetes-bootcamp`

`$ export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}')`

`$ echo NODE_PORT=$NODE_PORT`

`$ curl $(minikube ip):$NODE_PORT`

Confirmamos la actualizacion ejecutando el `rollout`

`$ kubectl rollout status deployments/kubernetes-bootcamp`

Chequeamos la imagen en el despliegue

`$ kubectl describe pods`

Hacemos un rollback de la actualizacion. Para ello desplegamos una imagen que tiene una version erronea

`$ kubectl set image deployments/kubernetes-bootcamp`

`$ kubernetes-bootcamp=gcr.io/google-samples/kubernetes-bootcamp:v10`

Vemos el estado del despliegue

`$ kubectl get deployments`

`$ kubectl get pods`

`$ kubectl describe pods`

Hacmeos un `RollBack` del despleigue a la ultima version que funciona

`$ kubectl rollout undo deployments/kubernetes-bootcamp`
`$ kubectl get pods`
`$ kubectl describe pods`

## Borramos el despleigue

Para borrar todos los Pods del cluster, borramos el despleigue

`$ kubectl create deployment kubernetes-bootcamp`