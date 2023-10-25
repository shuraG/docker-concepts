
# Taller
## Desarrollo de un sistema web (Se entrega código fuente)

El codigo fuente esta ubicado en la siguiete direccion:

`https://drive.google.com/file/d/1ybuFgVMNrrb3QPHV5zc3kcKGVltDshSU/view?usp=sharing`

Caracteristicas del proyecto

`Lenguaje:` Java 17

`Building Tool:` Graddle

`Base de datos:` PostgreSQL

Detalles

El proyecto consiste en registarar saludos en una base de datos, cada vez que se realize una llamada al siguiente endpoint:

`GET http://localhost:8080/hello?name=uio` 

en donde GET es el tipo de http request y `name` el nombre al que vamos a realizar un saludo. Cuando llamemos a ese endpoint si todo es funcionando de manera correcta debemos recibir un response:

`HELLO uio!`

y eso sera guardado en la tabla `greetings` en nuestra base de datos. El sistema crea automaticamente la tabla en la DB una vez que estos estan conectados.

La cadena a la base de datos es la siguiente: 

`spring.datasource.url=jdbc:postgresql://database:5432/user`

en donde `user` es la base de datos que cotiene la tabla `greetings`

## Construcción de un Dockerfile para el despliegue del sistema web

Se debe escribir un archivo `Dockerfile` para que la aplicacion pueda ser construida como una imagen, para esto tenemos lo siguiente.

+ La imagen base para el `Dockerfile` es: `openjdk:17-jdk-slim-buster`
+ Comando para que nuestra aplicacion sea ejecutada es: `ENTRYPOINT ["java", "-jar", "app.jar"]` 
+ el .jar o ejecutable a copiar se encuentra ubicado en build/libs/demo-0.0.1-SNAPSHOT.jar

## Construcción de docker-compose.yml para el despliegue organizado de contenedores:

Debemos escribir un archivo `docker-compose.yaml` que cumpla lo siguiente:
+ Construir la imagen docker.
+ Desplegar la aplicacion web.
+ Desplegar la base de datos.
+ Puedan conectarse entre ellos.
  
## Despliegue de Imagen construida del sistema web
Para esto utilizaremos los comandos vistos anteriormente como: `docker run imageName|imageID`
## Despliegue de Base de datos ( Postgresql desde Docker Hub)
En esta seccion debemos buscar la imagen de Postgresql en docker hub (podemos usar latest), y ver cuales son los variable de ambiente que especifica la documentacion, para poder realizar el correcto despliegue.

## Construcción y despliegue de Proxy (Nginx desde Docker Hub)
Debemos realizar un despliegue de Nginx, para esto primero debemos entrar en `Docker hub` y buscar la imagen oficial de Nginx. Para el despliegue podemos usar un `docker pull` y `docker start` o simplemente usar directamente el `docker run` (No olvidar exponer del puerto).
## Gestión de puertos

Para revisar los puertos expuestos y las ips de cada contenedor dentro de la red a la que agregamos hacemos un `docker inspect containerName|contaienrId`

## Gestión de red

Para ver nuestra red y como fue creada ejecutamos un

 `docker inspect networkname|networkID`

## Interconexión entre contenedores por nombre de contenedor

Para realizar la conexion entre nombres de contenedor, debemos usar la propiedad LINK en nuestro archivo `docker-compose` esto puede ser cubierto, cuando conectemos nuestra aplicacion web y nuestra base de datos postgresql.



## Solucion

Para construir nuestra imagen necesitamos de el archivo `Dockerfile` por lo tanto necesitamos crear este archivo y ubicarlo en la raiz del proyecto. El archivo que resuelve el problema planteado es el siguiente, en donde se detalla las instrucciones:

```Dockerfile

#Partimos de una imagen que contiene los componentes necesarios para ejecutar una aplicacion en JAVA 17
FROM openjdk:17-jdk-slim-buster

#Nos ubicamos en al path /app dentro del contenedor, sino existe se crea el path, a esto le llamamos nuestro directorio de trabajo o WORKDIR
WORKDIR /app

# El empaquetado de nuestra aplicacion JAVA es un .jar nombrado demo-0.0.1-SNAPSHOT.jar, este contiene todo para ser ejecutado dentro de un ambiente JDK. En definitiva este .jar es nuestra aplicacion. Por lo tanto debemos copiar demo-0.0.1-SNAPSHOT.jar dentro del contenedor, para esto usamos el comando COPY. COPY tiene 2 argumento, el primero es la fuente del archivo y el segundo es el destino(el path dentro del contenedor). Entonces el archivo demo-0.0.1-SNAPSHOT.jar va a ser copiado dentro /app (definnido en WORKDIR) con el nombre app.jar
COPY build/libs/demo-0.0.1-SNAPSHOT.jar app.jar

#El comando RUN ejecuta un comando dentro del contenedor y produce una nuevo layer o capa, generando asi una nueva imagen. No se recomienda realizarlo, debido a que como se menciona, esta produciendo una capa adicional y nuestra contenedor va a ser mas pesado. Por lo tanto se deberia intalar curl, cuando necesitamos y cuando el contenedor esta corriendo. En esta ocasion se ha puesto aqui de manera de ejemplo. Asi podremos realizar una peticion http dentro de nuestro contenedor, hacia nuestra applicacion web.
RUN apt-get update && apt-get install -yq curl

# Definimos nuestro comando para correr nuestra aplicacion. Tanto ENTTRYPOINT como CMD son ejecutados cuando nuestro contenedor entra en estado RUNNING, por lo tanto no se genera una capa o layer adicional en nuestra imagen. En este caso ENTRYPOINT recibe como argumento el nombre de nuestra aplicacion app.jar que fue previamente copiada con COPY en el directorio definido en WORKDIR. Adicionalmente el arumento app.jar puede ser sustituido al momento de arrancar nuestro contenedor, ya sea con docker run o docker start
ENTRYPOINT ["java", "-jar", "app.jar"]
```

Para construir nuestra imagen usamos el comando

`docker build -t myimagen .`

en donde le damos una etiqueta con `-t` y le pasamos nuestro contexto que en este caso es `.` ubicados en donde esta nuestro `dockerfile`.

Debido a que tenemos que ejecutar nuestra aplicacion en conjunto con una base de datos, hacemos uso de `docker compose` para facilitar la gestion de la creacion y configuracion de los contenedores que necesita nuestra aplicacion.

Para usar `docker compose` necesitamos definir nuestro archivo yaml de la siguiente forma:

```yaml
#Dentro de services definismo nuestros dos servicios demo que es nuestra app, y db nuestra base de datos
services:
    #Esta es la app de saludos
  demo:
    #Aqui definismo en donde esta ubicado nuestro dockerfile para poder hacer la construccion de la imagen
    build: ./demo
    #Estamos exponiendo los puertos 8080 en el contenedor hacis 8080 en el host.
    ports: 
      - "8080:8080"
    # Todos nuestros contenedores definidos son puestos dentro de una misma red de tipo bridge, asi pueden contectarse entre ellos unicamente por el nombre en este caso demo puede comunicarse con db solo usando db, Docker se encarga de gestionar el DNS. En este caso Links, le da un alias a db de database.
    links:
      - "db:database"
  # Definimos nuestra base de datos
  db:
    #En este caso usamos directo una imagen postgres
    image: postgres
    #Exponemos puertos hacia el host, pero no es neceesario debido a que demo puede conectarse internamente directo con db.
    ports: 
      - "5433:5432"
    #la configuracion de postgres puede ser hecha mediante variables de ambiente por lo que en esta seccion definsimo nuestro usuario y password de la base de datos.
    environment:
      - POSTGRES_PASSWORD=1234
      - POSTGRES_USER=user
    # Debido a que necesitamos persistir nuestros datos mas alla del contenedor, usamos un volumen para que los datos de la db no se pierdan cuando el contenedor sea destruido. Para eso usamos el volumen data_volume que apunta hacia el directorio /var/lib/postgresql/data dentro del contenedor, que es donde se almacenan los datos.
    volumes:
      - data_volume:/var/lib/postgresql/data
volumes:
  #Creamos uns volumen de nombre data_volume que esta alojado de manera local
  data_volume:
    driver: local
```

Finalmente para arrancar nuestra aplicacion hacemos uso de `docker-compose up` en donde sigue todas las instrucciones definidas en nuestro `docker-compose.yaml`

Finalmente para desplegar un nginx debemos seguir los siguiente pasos:
+ Dirigirnos a docker hub y buscar la imagen oficial de nginx
+ Revisar los caracteristicas y configuraciones de la imagen.
+ Ejecutar docker run para descargarnos la imagen, correrla y configurar el puerto `80` que es el que tiene por defecto nginx, darle un nombre y ejecutarlo en modo detach(background) comando: `docker run --name mynginx1 -p 80:80 -d nginx`
