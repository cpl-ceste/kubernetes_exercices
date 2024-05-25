# EJERCICIO 1 - KUBERNETES: INTRO

Documentacion de referencia [Docker Docs](https://docs.docker.com/)

## Set-up del entorno de trabajo

Instalar Docker Desktop [Get Docker](https://docs.docker.com/get-docker/) y el Plugin Docker en VStudio Code

Crear una cuenta en [Docker Hub ](https://hub.docker.com/)

## Probando el primer contenedor

Creamos un contenedor de Linux, con Ubuntu.

1) Conectarse a DockerHub desde Docker Desktop y VStudio Code

`$ docker login`

2) Buscar imagen de UBUNTU en DockerHub y correr el contenedor

`$ docker run -it ubuntu bash`

3) Comprobar que el contenedor corre en VStudio Code y Docker Desktop y que tenemos una imagen que ha bajado del repositorio oficial DockerHub

4) Vemos como funciona un contenedor:
-Entramos en el contenedor, creamos un fichero en /home (copiar uno de /var/log/ por ejemplo).
-Comprobamos que el fichero esta en el contenedor.
-Paramos el contenedor
-Lo rearrancamos y comprobamos si el fichero esta o no en /home
-Paramos y borramos el contenedor.
-Volvemos a arrancarlo desde la imagen oficial y comprobamos si el fichero esta o no en /home

5) Borramos contenedor e imagen


## Probamos un segundo contenedor con una aplicación web 

Crearemos un contenedor con la imagen de la aplicacion welcome-to-docker

1) Buscamos la aplicacion en DockerHub o Docker Desktop

2) Arrancamos el contenedor

`$ docker run --name welcome-to-docker -p 8080:80 docker/welcome-to-docker`

3) Probamos diferentes opciones de arranque del contenedor:
- con nombre, sin nombre
- detached o no
- levantamos varios contenedores de la misma imagen en puertos diferentes

4) Probamos los comandos de docker para trabajar con contenedores:

`$ docker ps (-a)`

`$ docker inspect <name> (or <container id>)`

`$ docker cp FILE_HOST <container_name_or_ID>:FILE_CONTAINER`

`$ docker stop <name> (or <container id>)`

`$ docker start <name> (or <container id>)`

`$ docker logs <name> (or <container id>)`

`$ docker exec -it <name> (or <container id>) <command>`

`$ docker rm name (or <container id>)`


## Probamos un tercer contenedor con un servidor de base de datos MySQL

Crearemos un contenedor con DBMS de MySQL

1) Buscamos la imagen de MySQL en DockerHub o Docker Desktop

2) Observar en la documentacion las variables de entorno que pueden pasarse a Docker para arrancar el contenedor

3) Arrancamos el contenedor con una password especifica para el usuario root

`$ docker run --name some-mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=mypass -d mysql`

3) Esperamos que se haya iniciado (ver los logs) y nos conectamos a la base de datos y creamos una base de datos y una tabla con el fichero `file_mysql.sql`

4) Si el contenedor se borra y vuelve a arrancarse, tenemos la tabla? hay persistencia de datos? Pero podemos hacer un dump de todas las bases de datos:

`$ docker exec some-mysql sh -c 'exec mysqldump --all-databases -uroot -p"$MYSQL_ROOT_PASSWORD"' > ./all-databases.sql`

Ver la documentacion sobre los dumps y restores