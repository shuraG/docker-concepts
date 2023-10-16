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
+ copy-on-write (CoW)


![](https://docs.docker.com/storage/storagedriver/images/container-layers.jpg)

**Nota:** Estas caracteristicas se deben al uso de [Internals-Union Filesystem](https://martinheinz.dev/blog/44) por el sistema de Docker, el que se usa por defecto es `OverlayFS overlay 2`, pero Docker soporta mas implementaciones. 

```console
.
├── upper
│   ├── code.py  # Content: `print("Hello Overlay!")`
│   └── script.py
└── lower
    ├── code.py  # Content: `print("This is some code...")`
    └── config.yaml
```

Ejecutamos el comando para hacer `overlay`

```console
$ mount -t overlay \
    -o lowerdir=./lower,\
       upperdir=./upper,\
       workdir=./workdir \
    overlay /mnt/merged

$ ls /mnt/merged
code.py  config.yaml  script.py

$ cat /mnt/merged/code.py
print("Hello Overlay!")

$ umount overlay
```

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

# Redes en Docker
Las redes dentro de `Docker` hace referencia a la capacidad que tiene un contenedor de comunicarse con otros contenedores.
## Docker Inspect

El siguiente es un extracto de un docker inspect de un contenedor que no esta asignado a una red ni tiene publicado un puerto.

Para extraer unicamente la parte de networkd se puede usar el siguiente comando:

```console
 $ docker inspect --format '{{json .NetworkSettings}}' containername|IDContainer
```

```json
{
  "Bridge": "",
  "SandboxID": "37fcd048dbcbd59066cca3de7094d4578b6e486b6bc37b3cb98f02551019b856",
  "HairpinMode": false,
  "LinkLocalIPv6Address": "",
  "LinkLocalIPv6PrefixLen": 0,
  "Ports": {
    "80/tcp": null
  },
  "SandboxKey": "/var/run/docker/netns/37fcd048dbcb",
  "SecondaryIPAddresses": null,
  "SecondaryIPv6Addresses": null,
  "EndpointID": "11438979b153a5c243d09ba6e2a5d3566d6531361638268140ef2a9166c5ab47",
  "Gateway": "172.17.0.1",
  "GlobalIPv6Address": "",
  "GlobalIPv6PrefixLen": 0,
  "IPAddress": "172.17.0.3",
  "IPPrefixLen": 16,
  "IPv6Gateway": "",
  "MacAddress": "02:42:ac:11:00:03",
  "Networks": {
    "bridge": {
      "IPAMConfig": null,
      "Links": null,
      "Aliases": null,
      "NetworkID": "0ba33cfd48343f96a0cb800e26c25646b3654f85df42fddb7473efdccdfcc43d",
      "EndpointID": "11438979b153a5c243d09ba6e2a5d3566d6531361638268140ef2a9166c5ab47",
      "Gateway": "172.17.0.1",
      "IPAddress": "172.17.0.3",
      "IPPrefixLen": 16,
      "IPv6Gateway": "",
      "GlobalIPv6Address": "",
      "GlobalIPv6PrefixLen": 0,
      "MacAddress": "02:42:ac:11:00:03",
      "DriverOpts": null
    }
  }
}
```

Cuando un contenedor es creado usando `create` o `run` por defecto ningun puerto es expuesto hacia afuera. Para exponer un contenedor hacia afuera usamos la flag `--publish hostport:containerport` o `-p hostport:containerport`. Esto crea una regla en el firewall del host, haciendo un mapeo del puerto del contenedor a un puesto en el host.

```json
{
  "Bridge": "",
  "SandboxID": "02b29359201e67de21ec974c2d38f5be5ad6c2af31d7d8dbf4093f0cd9213406",
  "HairpinMode": false,
  "LinkLocalIPv6Address": "",
  "LinkLocalIPv6PrefixLen": 0,
  "Ports": {
    "80/tcp": [
      {
        "HostIp": "0.0.0.0",
        "HostPort": "8081"
      }
    ]
  },
  "SandboxKey": "/var/run/docker/netns/02b29359201e",
  "SecondaryIPAddresses": null,
  "SecondaryIPv6Addresses": null,
  "EndpointID": "1f6c14e331a3a131f9cc481dbc379a9c5be0afca354173c9e09fbf7b643ce19b",
  "Gateway": "172.17.0.1",
  "GlobalIPv6Address": "",
  "GlobalIPv6PrefixLen": 0,
  "IPAddress": "172.17.0.5",
  "IPPrefixLen": 16,
  "IPv6Gateway": "",
  "MacAddress": "02:42:ac:11:00:05",
  "Networks": {
    "bridge": {
      "IPAMConfig": null,
      "Links": null,
      "Aliases": null,
      "NetworkID": "0ba33cfd48343f96a0cb800e26c25646b3654f85df42fddb7473efdccdfcc43d",
      "EndpointID": "1f6c14e331a3a131f9cc481dbc379a9c5be0afca354173c9e09fbf7b643ce19b",
      "Gateway": "172.17.0.1",
      "IPAddress": "172.17.0.5",
      "IPPrefixLen": 16,
      "IPv6Gateway": "",
      "GlobalIPv6Address": "",
      "GlobalIPv6PrefixLen": 0,
      "MacAddress": "02:42:ac:11:00:05",
      "DriverOpts": null
    }
  }
}
```
![image](https://i.stack.imgur.com/Nb6Om.png)

![image](https://i.stack.imgur.com/s4k6E.png)

## Inspeccionando redes por defecto (Bridge)

Docker crea una red por defecto, llamada `bridge` en caso de que esta no exista. Esta red por defecto brinda las capacidades basicas de red a los contenedores en el mismo host.

Todo contenedor que no ha sido expecificado una red, usara la red por defecto `bridge`.

Todos los contenedores reciben su IP a travez de DHCP. Asi se asigna una IP a cada contenedor cuando esto son arrancados.

El subneting se realiza de forma automatica. Por defecto Docker usa el rango `172.17.0.0/16`.

Por defecto los contenedores toman la configuracion del host `/etc/resolv.conf`


## Gestión de redes
`Docker` gestiona las redes categorizando como drivers, teniendo los siguientes:
  + Bridge: Permite conectarse entre contenedores que pertenecen al mismo `bridge`. Aislando de los que no pertenecen
  + Overlay: Usado cuando se quiere trabajar en un cluster. Un cluster en donde tenemos varios `Docker hosts` alojados en diferentes nodos.
  + Host: Usa la red del host directamente.
  + MacVlan: Se asigna un mac adress, por lo que puede ser tratado como un nuevo dispositivo fisico dentro de la red.
  + None: Desactiva la red de un contenedor.
  
## Interconectando contenedores
En este caso para interconectar contenedores entre si, vamos hacer el uso del driver `bridge`.

Crear una red bridge

```console
$ docker network create my_net
$ docker container run -dit --name container-a --network returngis-net alpine ash
$ docker container run -dit --name container-b --network returngis-net alpine ash
```

Conectar a una red adicional

```console
$ docker network connect returngis-net container-1
```

Creacion de una mcvlan network

```console
 docker network create -d macvlan \                                                 
  --subnet=172.16.86.0/24 \
  --gateway=172.16.86.1 \
  -o parent=eth0 \
  my-macvlan-net

```

`ctrl + p` `ctrl + q`

# Docker Compose
## ¿Qué es docker compose?
Docker compose es una herramienta desarrollada por docker que sirve para definir y correr varios contenedores.

Un archivo `YAML` es usado para definir todas las configuraciones de los servicios.

Pagina oficial de YAML: `https://yaml.org/`

Nos ayuda a levantar aplicaciones que contengan varios contenedores, asi como: ver los estado de los servicios, pararlos, arrancarlos y apagarlos.

Los casos de uso mas comunes para `docker compose` son los siguientes:
+ Tener multiples ambientes aislados dentro del mismo host.
+ Manejar volumenes cuando los contenedores son creados.
+ Unicamente recrear contenedores que han sido modificados.
+ Mover las aplicaciones entre ambientes diferentes.


```console
$ sudo apt-get update
$ sudo apt-get install ./docker-desktop-<version>-<arch>.deb
$ systemctl --user start docker-desktop
```

## Explorando un archivo docker-compose.yml

Como se menciono toda la definicion de los servicios y sus configuraciones, son realizadas en un archivo `YAML`. El cual es nombrado `docker-compose.yaml`

```yaml
services:
  demo:
    build: ./demo
    ports: 
      - "8080:8080"
    links:
      - "db:database"
  db:
    environment:
      - POSTGRES_PASSWORD=1234
      - POSTGRES_USER=user
    image: postgres
    volumes:
      - data_volume:/var/lib/postgresql/data
volumes:
  data_volume:
    driver: local
```

## Despliegue de contenedores con Docker compose

Para hacer el despliegue de nuestra aplicacion debemos ubicarnos en donde esta nuestro archivo `docker-compose.yaml` y usar el comando 

```console
  docker-compose up
```

Lo que vamos a tener es una creacion automatico de nuestro contenedores y seran agregados a una red que tambien sera creada con el nombre del directorio en el que estamos.
Por ejemplo si estamos en el directotop `app`  la red por defecto de tipo bridge sera llamada `app_default`

Dando de baja nuestro servicio

```console
  docker-compose down
```

Vamos a parar y eliminar nuestros contenedores relacionados a este servicio

Adicionalmente muchas veces debido al caching que realiza docker de nuestras imagenes debemos hacer un re build de estas, para esto tenemos dos opciones

```console
docker-compose build
docker-compose up --build
```
  ## Comandos Adicionales
```bash
# comandos utiles
# Crea un archivo con una cadena de texto
$ sudo echo "Hello World" >> /hostvolume/host-hello.txt
# usa volmenes desde otro contenedor
docker run --volumes-from <source_container> <target_container>
# actualizar reps
apt update && sudo apt upgrade
#install ping utility
apt install iputils-ping
# adjunta un container al driver host, comprobar en http://localhost:80
docker run --rm -d --network host --name my_container nginx
# dehsabilitar la red de un contenedor
docker run --rm -dit --network none --name my_container myimgexp
#Unirnos a un contendor con terminal previamente ejecutada
docker attach container-1
#entrar a la base de datos
psql -U testuser -d testdb
# rebuild las imagenes
docker-compose up --build
# volume docker 
 -v /custom/mount:/var/lib/postgresql/data
server.port=8081
environment.name=${OS}
```

# Docker Registry y Docker Hub)
## ¿Qué es el Docker Registry?
+ Docker Registry es un almacen y un gestor de imagenes de docker de alta escalabilidad. Permitiendonos guardar nuestras imagenes y distribuirlas.
+ Se puede incluir en un ciclo CI/CD, teniendo un control total de las distribucion de las imagenes.
+ Es una implementacion de OCI(Open Container initiative)
+ Registry es un proyecto de codigo abierto.
+ Esta bajo la licencia Apache.
## Iniciando docker registry local
Docker registry puede ser desplegado en nuestra propia infraestructura, utilizando el mismo ecosistema de Docker, y pudiendo ser configurado como un sistema de alta escalabilidad y tolerante a fallos. Por otro lado tambien puede ser desplegado en unico nodo o de manera local, que en este caso como un objetivo academivo.

Para esto necesitamos descargar la imagen de `docker registry`
![image](https://blog.octo.com/wp-content/uploads/2014/01/Diapositive1.png)
## Gestionando contenedores en Registry local
## Trabajando en DockerHub
