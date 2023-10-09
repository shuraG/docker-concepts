# Instalación
## Docker

```console
# Desintalar versiones antiguas de Docker

$ for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done

# Add official GPG key of Docker:

sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Add the repository to Apt sources:

echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

# Instalacion de paquetes Docker

$ sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
    
## Docker Compose (Linux - Ubuntu, previamente se entrega manual de instalación)

```console
$ sudo apt-get update
$ sudo apt-get install docker-compose-plugin
$ docker compose version
```
Salida esperada:
```console
Docker Compose version vN.N.N
```
# Verificando Instalación
 Desplegando un contenedor de prueba (Desde Docker Hub)

```console
$ sudo docker run hello-world
```



# Desplegando un contenedor desde Docker Hub

Gestión de Contenedores Básico:
+ Docker CLI
+ Docker Desktop

# Cómo desplegar un contenedor
Ciclo de vida de un contenedor
* Descarga de imagen desde un registry(Docker HUB) a nuestro host local. (Se omite si ya esta descargada)
* Crear un contenedor a partir de la imagen
* Correr el contenedor (Despliegue).
* Parar el contenedor.
* Eliminacion del contenedor.

![img](https://d33wubrfki0l68.cloudfront.net/25491ef22ebfd50575d20d0bb0365deed7cf5be3/c0453/img/blog/2022/07/docker_container_lifecycle_3x.webp)

# El comando Docker pull
Descarga una imagen en particular desde un registry `DockerHub`.
```code
docker pull [OPTIONS] NAME[:TAG|@DIGEST]
```
Desplegando contenedores (Mysql/Postgres)

```console
$ docker images
$ docker pull postgres
$ docker run --name some-postgres -e POSTGRES_PASSWORD=mysecretpassword -d postgres
$ docker ps -a

# Crear contenedor
$ docker create --name my-postgresql -e POSTGRES_USER=myusername -e POSTGRES_PASSWORD=mypassword postgres:11.21-bullseye
# Iniciar contenedor
$ docker start my-postgresql

#Entrar con bash en modo interactivo dentro del contenedor
$ docker exec -it <container_name> bash
#i: crea una sesion interactiva
#e: emula una termina

# Database
$ psql -U postgres

$ docker run --name postgresql -e POSTGRES_USER=myusername -e POSTGRES_PASSWORD=mypassword -p 5432:5432 -v /data:/var/lib/postgresql/data -d postgres

$ docker image pull myregistry.local:5000/testing/test-image
```

# Manejo de Imágenes
Una imagen puede ser visto como una receta o template para crear contenedores.
Las imagenes pueden ser:
+ Descargadas desde un registry.
+ Creadas desde un `Dockerfile` mediante el procesos de building.
+ Subidas a un registry (DockerHub, Harbor, etc).
+ Eliminadas.
+ Etiquetadas y nombradas segun sus versiones.
+ Poseen un ID unico e inmutable llamado `Digest`

# Dockerfile
Dockerfile es el concepto basico que nos sirve para la construccion de imagenes de Docker.

![img](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*CP98BIIBgMG2K3u5.png)

# ¿Qué es un archivo Dockerfile?
Un archivo `Dockerfile` es un arhivo de texto plano, sin extension. Este archivo contiene todas las instrucciones que nos dice como crear una imagen. Mayormente se encuentra ubicado en la raiz de la applicacion.

# Explorando un archivo Dockerfile
Las imagenes siempre son construidas a partir de otras imagenes base. Muchas de esas imagenes base son construidas a partir de otras. Teniendo varias capas hasta llegar a una imagen final. Esto mejora la reutilizacion y eficiencia.

```dockerfile
# Esto es un comentadio
COMMAND arguments
```
Por convencion los comandos se escriben en Mayusculas.

Ejemplo:

```dockerfile
# The latest version of Ubuntu is used as the base image for this Dockerfile.
FROM ubuntu:latest
# Set the author field for the generated image.
MAINTAINER rocky.chen@example.com
# Run some commands.
RUN apt-get update && apt-get install -yq curl
RUN apt-get clean
# Commands to run when the generated image gets launched.
CMD ["top"]
CMD ["ls", "-l"]
# Set working directory and environment variables.
WORKDIR /root
ENV VAR1 version1
# Add a shell script to the generated image.
ADD run.sh /root/run.sh
RUN chmod +x run.sh
# Set entry point and argments for the generated image.
ENTRYPOINT ["./run.sh"]
CMD ["arg1"]
```

Ejemplo para una aplicacion java:
    
```dockerfile
FROM adoptopenjdk:11-jre-hotspot

WORKDIR /app

COPY target/your-application.jar app.jar

CMD ["java", "-jar", "app.jar"]

```
`FROM`: Imagen en la que nos estamos basando

`WORKDIR`: Directorio en donde nos ubicamos dentro del contenedor

`COPY`: Copia los archivos indicados al directorio configurado en `WORKDIR`

`CMD`: Comando a ser ejecutado cuando el contenedor corra.

`ENTRYPOINT`: Comando orientado a recibir argumentos

Se puede sobreescribir entrypoint

    docker run —entrypoint <command and arguments>

# Construyendo Imágenes


La construccion de imagenes se hace mediante el comando `build`

```console
$ docker build -t myimage[:tag] .

$ docker run -p 8080:8080 myimage
```
`Docker daemon` es el que construye la imagen, por lo tanto todo el contexto es enviado hacia alla.


    Sending build context to Docker daemon  3.584kB
`.dockerignore` es usado para evitar transferir archivos innecesario a `Docker daemon`
## Layers

En cada paso se genera un contenedor y la instrucción se ejecuta dentro del contenedor generado. Una vez que la instrucción tiene éxito, el contenedor se almacena como una nueva imagen, ya que se añade una nueva capa.

Primera vez que una imagen se construye. 

Output:

```console

Sending build context to Docker daemon  3.584kB
Step 1/12 : FROM ubuntu:latest
 ---> d70eaf7277ea
Step 2/12 : MAINTAINER rocky.chen@example.com
 ---> Running in 5d1f44d64602
Removing intermediate container 5d1f44d64602
 ---> 82eb396b28ec
Step 3/12 : RUN apt-get update && apt-get install -yq curl
 ---> Running in cc6adb835c4d
...
Removing intermediate container cc6adb835c4d
 ---> bbd0dc87c576
Step 4/12 : RUN apt-get clean
 ---> Running in 7664a4778017
Removing intermediate container 7664a4778017
 ---> a9ed19e6119d
Step 5/12 : CMD ["top"]
 ---> Running in 7e37bb170506
Removing intermediate container 7e37bb170506
 ---> d6c8736f0e9f
Step 6/12 : CMD ["ls", "-l"]
 ---> Running in cbd1a7512232
Removing intermediate container cbd1a7512232
 ---> 0c087c832e7f
Step 7/12 : WORKDIR /root
 ---> Running in 3068d7075458
Removing intermediate container 3068d7075458
 ---> d80343d04808
Step 8/12 : ENV VAR1 version1
 ---> Running in ae6480ddac50
Removing intermediate container ae6480ddac50
 ---> 3c649e8e249c
Step 9/12 : ADD run.sh /root/run.sh
 ---> b0419c4d6864
Step 10/12 : RUN chmod +x run.sh
 ---> Running in 12520302b333
Removing intermediate container 12520302b333
 ---> f52955a93631
Step 11/12 : ENTRYPOINT ["./run.sh"]
 ---> Running in 22bc5cc335b1
Removing intermediate container 22bc5cc335b1
 ---> 409dee822026
Step 12/12 : CMD ["arg1"]
 ---> Running in 44cf42bcd050
Removing intermediate container 44cf42bcd050
 ---> 8c2b84201d65
Successfully built 8c2b84201d65
Successfully tagged myimage:latest

```

# Listando Imágenes

```console
$ docker images
```
Podemos usar filtros para mostrar las imagenes requeridas

# Eliminando Imágenes

```console
$ docker rmi [imageID]
```

Podemos combinar con docker images para borrar imagenes en masa.


# Manejo de Contenedores
# Desplegando contenedores
`docker create [OPTIONS] IMAGE [COMMAND] [ARG...]`

`it:` abrimos una terminal interactiva

`d:` detach mode, podemos correr en background

## Nombre contenedor
+ Unico dentro de nuestro ambiente de contenedores.
+ Mas facil de identificar en lugar del ID.

`docker run --name [name] image`

`docker create --name [name] image`


Mediante la option --name podemos definir un nombre en el momento de crear un contenedor. 

### Renombrar a un contenedor
`docker rename container_name new_container_name`

Podemos renombrar un contenedor en cualquier momento con el comando mostrado anteriormente.
## Puertos de despliegue
  `docker run --name [name] -p [host port]:[container port] image`

  `--publish -p :` Se publica un puerto en el host

## Especificando la Imagen

# Listado de Contenedores (Activos/Inactivos)
Para listar los contenedores usamos los siguientes comandos

`docker ps [OPTIONS]`

`docker container ls [OPTIONS]`

`-a:` nos permite visualizar todos los contenedores incluyendo los que no estan corriendo.

`--filter:` Una de las opciones mas importantes para filtar es el uso de fitlros en los `OPTIONS` dentro de los comando de listado.

En un ambiente de produccion podemos tener cientos de contenedores, poder filtrarlos nos da la opcion de encontrar problemas de una formas mas eficiente.
 
 `docker ps -a --filter status=exited`

 `status:` created, restarting, running, removing, paused, exited y dead

`name:` filtra haciendo un match en el nombre.
# Deteniendo/Reiniciando un contenedor
  docker stop and docker start son los comandos que nos permiten parar y arrancar un contenedor.

`docker stop containername|containerID`

`docker start containername|containerID`
# Eliminando un contenedor
Previamente el contenedor debe estar en estado `STOP`

  `docker rm namecontainer|idcontainer`

# Interactuando con un contenedor
## Explorando un contenedor
Al ser cada contenedor una instancia unica de una imagen, estos poseen propiedades y configuraciones unicas. Existen varios comandos para poder visualizar esas propiedades y configuraciones.
## Revisión de Logs y Docker Inspect
Muchas veces necesitamos hacer debugging en contenedores especificos para poder detectar problemas dentro de un contenedor. Una de las maneras es revisar los logs, para esto el siguiente comando es util:

`docker logs [OPTIONS] CONTAINER`

Como experimento el siguiente comando crear un contenedor que imprime la fecha cada segundo.

```console
docker run --name containerlogs -d ubuntu sh -c "while true; do $(echo date); sleep 1; done"
```

Haciendo el uso del comando `logs` podemos visualizar los registros y salidas de fechas que ejecutamos anteriormente.
```console
docker logs -f --since=2s containerlogs
```
`-f:` nos permite ver en forma de un stream.

`-since:` Vemos los logs desde los ultimos 2 segundos.

## Docker stats
Nos ayuda a monitorear los recursos que estan siendo usados por un dterminado contenedor.

`docker stats containername|containerID`
## Transferencia de archivos
Para transferir archivos desde el host hacia un contenedor usamos el comando `CP` que ees la abreviatura de `copy`.

`docker cp mypathhost:CONTAINERNAME:mypathinsidecontainer`

# Practica Docker
+ Despliegue completo de un base de datos postgres
+ Correr un contenedor con una html basico. 
  + Imagen base de un servidor http: `httpd:2.4`
  + Path del server http para html files: `/usr/local/apache2/htdocs`
+ Transferir un segundo html file hacia el contenedor
  + Visualizar el nuevo html desde el browser
+ Mostrar los logs de los contendedores
+ Eliminar contenedores e imagenes y levantarlos nuevamente.


# Volúmenes en docker
Los contenedores tiene el principio de ser creados y eliminados tantas veces como sea necesario. Asi un contenedor tiene las siguiente caracteristicas:

+ Una imagen es creada sobre capas de lectura.

+ Cuando un contenedor es creado se anade una capa de escritura. Llamada `container layer.`

+ Una vez que un contenedor muere, la capa es eliminada, y todos los archivos que estan en esta. 


![](https://docs.docker.com/storage/storagedriver/images/container-layers.jpg)

**Nota:** Estas caracteristicas se deben a Docker [Internals-Union Filesystem](https://martinheinz.dev/blog/44)

## Persistencia de información de un contenedor

![image](https://docs.docker.com/storage/images/types-of-mounts.png)

Los volumenes solucionan este problema. Un volumen persiste los datos, incluso despues de que un contenedor es eliminado.

Un volumen no depende del contenedor, y reside en la maquina host.

Desde el punto de vista del contenedor un volumen actua como un directorio.





## Creando un volumen / docker volume create

Crear un volumen

    docker volume create --name [volume name]

Ejemplo:

```console
$ docker volume create --name mivolumen
```
Para listar nuestro volumenes hacemos uso del siguiente comando

`docker volume ls`

```console
DRIVER    VOLUME NAME
local     0fb73aae6bab8fd2127da9165f95261e25beb333e382c6c84f3061cc3b70d137
local     anothervolumen
local     hello
```

En donde se muestra los volumenes que tenemos creados. Tanto el driver que en este caso todos los volumenes son locales, y el nombre de este. Cabe mencionar que el nombre de un volumen siempre es unico.

Se menciona que cuando creamos volumnes con nombre estos crean un path automaticamente a:

`/var/lib/docker/volumes`

## Desplegando contenedores con volúmenes

Para agreagar nuestro volumen al contenedor usamos la bande `-v` con el siguiente formato:

    -v [volume name]:[container directory] 

```console
$ docker run -v mivolumen:/app myimage
```

Adicionalmente se puede crear un volumen junto con el contenedor. Especificando directamente nuestra ruta que va a ser montada en nuestro contenedor.

```console
$ docker run -it -v /app:/volume/app ubuntu /bin/bash
```
en donde `\app` es el directorio del host y `/volume/app` el directorio en el contenedor.

La siguiente manera de crear un volumen es usando el `Dockerfile`, la ventaja es que de esta manera la creacion de volumenes esta automatizada


```Dockerfile
# The source image to start with
FROM centos

# Create a volume
VOLUME /dockerfilevolume
```

## Docker Inspect y eliminando contenedores con volúmenes

Para poder visualizar las propiedas de un volumen especifico utilizamos

`docker inspect volumenname|volumeID`

En donde podemos visualizar informacion a detalle del volumen.

```console
[
    {
        "CreatedAt": "2023-10-06T21:46:50Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/hello/_data",
        "Name": "hello",
        "Options": null,
        "Scope": "local"
    }
]
```

Para la eliminacion de un contenedor y los volumenes asociados se usa el flag `--volumes`

```console
$ docker rm --volumes mycontainer
```
Note: Si un volumen es nombrado, este no sera eliminado.

```console
$ docker create -v awesome:/foo -v /bar --name hello redis
```
En este caso solo `/bar` es eliminado.

## Permisos de escritura/lectura en los volúmenes
Cuando docker crea un volumen este por defecto es creadp con permisos de lectura y escritura. Muchas veces necesitamos por seguridad que algunos contenedores no puedan modificar datos o eliminarlos, ya que son solo consumidores. Para cambiar los permisos o privilegios de un contenedor con respecto a un volumen tenemos que agregar `:ro` al final del montaje del volumen.

`docker run -v /directory:/path:ro`


## Eliminación de volúmenes / docker volume rm / prune

Para la eliminacion del volumen se hace uso de:

`docker volumen rm volumenname|volumeID`


Cabe mencionar que solo se pueden eliminar volumnes que no estan siendo referenciados.

Para eliminar todos los volumnes que no estan siendo utilizados o referenciados por algun contenedor usamos:

`docker volume prune`

## Cambiando ubicación del volumen

Para cambiar la ubicacion del volumen es necesario:
+ Inspecsionar el volumen, en especial `Mountpoint`.
+ Parar los contenedores.
+ Mover los datos hacia donde va a estar el nuevo volumen
+ Crear un nuevo volumen 
+ Arrancar los contenedores con el nuevo volumen.

```console
$ docker volume create --driver local \
  --opt type=none \
  --opt device=/new/path/on/host \
  --opt o=bind my_new_volume
```

`type=none`: Docker no manejara el contenido del volumen y el path es especificado en el device option.

`device`: El path en el host donde los datos del volumen seran alojados.

`o=bind`: Crea un symbolic link entre el volumen y el directorio especificado en `device`.


## Comandos Adicionales
```bash
# comandos utiles
# Crea un archivo con una cadena de texto
$ sudo echo "Hello World" >> /hostvolume/host-hello.txt
# usa volmenes desde otro contenedor
docker run --volumes-from <source_container> <target_container>
```

# Redes en Docker / Pŕactica (2h)
## Docker Inspect
## Inspeccionando redes por defecto (Bridge)
## Gestión de redes
## Interconectando contenedores

`ctrl + p` `ctrl + q`

![image](https://i.stack.imgur.com/Nb6Om.png)

![image](https://i.stack.imgur.com/s4k6E.png)
