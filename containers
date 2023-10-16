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
