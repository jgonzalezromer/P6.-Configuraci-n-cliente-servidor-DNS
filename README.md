<h1>
<p align=center>
P6. Configuración cliente + servidor DNS 
</p>
</h1>
<h3>
<p align=center>
Juan Gabriel González Romero
</p>
</h3>

---
### Proposta: Engadir un servizo, que se conectará ao DNS, ao docker compose do [proxecto anterior](https://github.com/jgonzalezromer/P5.-DNS---Docker-Compose).

---
## Recursos
Utilizaremos o [repositorio de Damián](https://github.com/damiancastelao/practica01-DNS).
---
## docker-compose.yml
Modificaremos o `docker-compose.yml` que fixemos con anterioridade:
```
services:
  #Servizos que se executarán no contenedor
  bind9: #Como queremos chamar nos o servizo
    image: internetsystemsconsortium/bind9:9.18 #Imaxe que utilizaremos no contenedor
    container_name: Practica5_bind9 #Nome específico do contenedor
    ports:
      #Mapeo dos portos host:contenedor
      - 54:53/udp
      - 54:53/tcp
      - 127.0.0.1:953:953/tcp #Neste só pertmite conexións desde localhost
    volumes:
      #Volumes onde se montará o contenedor
      - ./etc/bind:/etc/bind
      - ./var/cache/bind:/var/cache/bind
    restart: always #Esta opción indica que ó contenedor debe reiniciarse en ciertas condiciones
```
### networks
O primeiro será añadir o apartado de networks para así logo especificalas a cada contenedor.
Aquí vou por duas maneras de facelo:
1. Nesta manera a rede `P6_network` desplegarase xunto a creación dos contenedores.
```
networks:
  P6_network:
    driver: bridge
    ipam:
      config:
        - subnet: 172.18.0.0/16
          ip_range: 172.18.0.0/24
          gateway: 172.18.8.254

```
2. Nesta manera a rede xa debe estar creada na maquina onde se vaia a executar o docker-compose
```
networks:
  P6_network:
    external: true
```
### bind9
Na sección de bind9 deixaremos casi todo igual. Añadiremos un apartado de `networks`:
```
networks:
      P6_network:
        ipv4_address: 172.18.0.2
```
Esto nos servirá para indicar cal será a sua IP, que logo utilizaremos como DNS.

### cliente
O cliente que utilizaremos terá a imaxe `alpine` e o código do seu contenedor será:
```
cliente:
    image: alpine
    container_name: Practica6_alpine
    tty: true
    stdin_open: true
    networks:
      P6_network:
        ipv4_address: 172.18.0.3
    dns:
      - 172.18.0.2
```
### Completo
Este sería o arquivo `docker-compose.yml` completo:
```
services:
  #Servizos que se executarán no contenedor
  bind9: #Como queremos chamar nos o servizo
    image: internetsystemsconsortium/bind9:9.18 #Imaxe que utilizaremos no contenedor
    container_name: Practica5_bind9 #Nome específico do contenedor
    ports:
      #Mapeo dos portos host:contenedor
      - 54:53/udp
      - 54:53/tcp
      - 127.0.0.1:953:953/tcp #Neste só pertmite conexións desde localhost
    volumes:
      #Volumes onde se montará o contenedor
      - ./etc/bind:/etc/bind
      - ./var/cache/bind:/var/cache/bind
    restart: always #Esta opción indica que ó contenedor debe reiniciarse en ciertas condiciones
    networks:
      P6_network:
        ipv4_address: 172.18.0.2
  cliente:
    image: alpine
    container_name: Practica6_alpine
    tty: true
    stdin_open: true
    networks:
      P6_network:
        ipv4_address: 172.18.0.3
    dns:
      - 172.18.0.2
networks:
  P6_network:
    driver: bridge
    ipam:
      config:
        - subnet: 172.18.0.0/16
          ip_range: 172.18.0.0/24
          gateway: 172.18.8.254
```

## named.conf
O named.conf o utilizaremos para chamar a outros archivos(como se fora unha función main), para que así quede todo maís ordenado.
```
include "/etc/bind/named.conf.options";
include "/etc/bind/named.conf.local";
```

### named.conf.options
Este arquivo sirve para indicar as opcións do servidor DNS nel introduciremos o codigo:
```
options {
        directory "/var/cache/bind";

        forwarders {
                8.8.8.8;
                1.1.1.1;
         };
         forward only;

        listen-on { any; };
        listen-on-v6 { any; };

        allow-query {
                any;
        };
};
```
### named.conf.local
Este arquivo define a zona que terá o servidor DNS:
```
zone "asircastelao.int" {
        type master;
        file "/var/lib/bind/db.asircastelao.int";
        allow-query {
                any;
                };
        };
```

## Conectar cliente
Entraremos al cliente con el comando `docker exec -it Practica6_alpine /bin/sh` y utilizaremos los comandos:
```
apk update
apk add bind-tools
```
Ahora ya podremos utilizar el comando `dig` con el que comprobaremos el servidor.
