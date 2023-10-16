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
