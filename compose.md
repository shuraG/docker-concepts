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

