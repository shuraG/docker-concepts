
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
