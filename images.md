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
