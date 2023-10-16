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
