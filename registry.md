# Docker Registry y Docker Hub
## ¿Qué es el Docker Registry?
+ Docker Registry es un almacen y un gestor de imagenes de docker de alta escalabilidad. Permitiendonos guardar nuestras imagenes y distribuirlas.
+ Se puede incluir en un ciclo CI/CD, teniendo un control total de las distribucion de las imagenes.
+ Es una implementacion de OCI(Open Container initiative)
+ Registry es un proyecto de codigo abierto.
+ Esta bajo la licencia Apache.
## Iniciando docker registry local
Docker registry puede ser desplegado en nuestra propia infraestructura, utilizando el mismo ecosistema de Docker, y pudiendo ser configurado como un sistema de alta escalabilidad y tolerante a fallos. Por otro lado tambien puede ser desplegado en unico nodo o de manera local, que en este caso como un objetivo academivo.

Para esto necesitamos descargar la imagen `docker registry`

```console
$ $ docker run -d -p 5000:5000 --name registry registry:2
```

`Docker registry` esta corriendo y esta expuesto en el puerto 5000.

## Gestionando contenedores en Registry local

El siguiente ejercicio realiza la descarga de una imagen, etiquetamos con una nueva etiqueta, subimos a nuestro registry local y finalmente lo descargamos.
 
```bash
# Descarga de una imagen desde Docker Hub
docker pull ubuntu
# Le damos una nuego tag
docker image tag ubuntu localhost:5000/myfirstimage
# Subimos nuestra imagen a nuestro registry local
docker push localhost:5000/myfirstimage
# Nos volvemos a descargar nuestra imagen 
docker pull localhost:5000/myfirstimage
# Borrar el contenido de registry local
docker container stop registry && docker container rm -v registry
```

![image](https://blog.octo.com/wp-content/uploads/2014/01/Diapositive1.png)

## Trabajando en DockerHub

DockerHub es un registry corriendo en la nube que es distribuido, tolerante a fallos y escalable.

Para poder acceder y publicar imagener en `Docker Hub` necesitamos crearnos una cuenta. `Docker Hub` tiene diferentes planes, pero la version basica no tiene costo.

Para ingresar necesitamos hacer un login, para esto usamos el comeando
```console
  docker login
```
en donde digitamos nuestro usuario y contrasena. 

Para poder realizar un push o subir nuestra imagen a `Docker Hub` debemos etiquetarla con nuestro de nombre de usuario.

`nombre_usuario/mi_imagen`

Para subir la imagen:

```console
$ docker push nombre_usuario/mi_imagen
```
Ingresamos a nuestra cuenta y podemos verificar que nuestra imagen a sido subida.


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
