
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
