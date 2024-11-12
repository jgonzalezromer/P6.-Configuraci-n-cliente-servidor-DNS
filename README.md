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
  P6_network: #Nome da rede
    driver: bridge #Tipo de rede
    ipam:
      config:
        - subnet: 172.18.0.0/16 #Rango de enderezos IP
          ip_range: 172.18.0.0/24 #Subrango de enderezos
          gateway: 172.18.8.254 #Dirección gateway

```
2. Nesta manera a rede xa debe estar creada na maquina onde se vaia a executar o docker-compose
```
networks: 
  P6_network: #Nome da rede
    external: true #A rede buscarala
```
### bind9
Na sección de bind9 deixaremos casi todo igual. Añadiremos un apartado de `networks`:
```
networks:
  P6_network: #Nome da rede
    ipv4_address: 172.18.0.2 #IP que lle otorgamos ao DNS
```
> [!NOTE]
> Esto nos servirá para indicar cal será a sua IP, que logo utilizaremos como DNS.

### cliente
O cliente que utilizaremos terá a imaxe `alpine` e o código do seu contenedor será:
```
cliente:
    image: alpine #Imaxe do cliente
    container_name: Practica6_alpine #Nome do container
    tty: true
    stdin_open: true
    networks:
      P6_network: #Nome da rede
        ipv4_address: 172.18.0.3 #IP que lle otorgamos ao cliente
    dns:
      - 172.18.0.2 #IP do servidor DNS
```
> [!NOTE]
> Os comandos `stdin_open: true` e `tty: true` os utilizamos para que o container se quede activo.
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
    image: alpine #Imaxe do cliente
    container_name: Practica6_alpine #Nome do container
    tty: true
    stdin_open: true
    networks:
      P6_network: #Nome da rede
        ipv4_address: 172.18.0.3 #IP que lle otorgamos ao cliente
    dns:
      - 172.18.0.2 #IP do servidor DNS
networks:
  P6_network: #Nome da rede
    driver: bridge #Tipo de rede
    ipam:
      config:
        - subnet: 172.18.0.0/16 #Rango de enderezos IP
          ip_range: 172.18.0.0/24 #Subrango de enderezos
          gateway: 172.18.8.254 #Dirección gateway
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
        directory "/var/cache/bind"; #Directorio onde se almacenarán os datos de caché

        forwarders {
                8.8.8.8; #DNS de Google
                1.1.1.1; #DNS de Cloudflare
         };
         forward only; #Indicamos que as consultas que non podan resolverse de una forma local serán reenviadas aos servidores indicados arriba

        listen-on { any; }; #O servidor escoitará todas as interface IPv4 dispoñibles no servidor
        listen-on-v6 { any; }; #O servidor escoitará todas as interface IPv6 dispoñibles no servidor

        allow-query {
                any;
        }; #Calquera dispositivo pode realizar consultas DNS
};
```
> [!WARNING]
> No comando `allow-query` non é recomendable por `any` nun entorno real
### named.conf.local
Este arquivo define a zona que terá o servidor DNS:
```
zone "asircastelao.int" {
        type master; #Indicamos que o servidor será o principal na zona
        file "/var/lib/bind/db.asircastelao.int"; #Ruta ao ficheiro que contén os rexistros DNS na zona
        allow-query {
                any;
                };
        }; #Calquera dispositivo pode realizar consultas DNS
```
> [!WARNING]
> No comando `allow-query` non é recomendable por `any` nun entorno real

## Conectarse o cliente
Entraremos no cliente co comando `docker exec -it Practica6_alpine /bin/sh` e utilizaremos os comandos:
```
apk update #Comando para actualizar os paquetes
apk add bind-tools #Comando para descargar o paquete bind-tools o cal inclue dig e nslookup.
```
> [!IMPORTANT]
> Estos comandos os utilizamos xa que estamos nunha terminal `sh`, cambiarián se utilizaramos noutra como pode ser `bash`

## Comprobar o servidor
Comprobaremos que o servidor funciona facendo o comando:
```
dig @172.18.0.2 asircastelao.int
```
Esto serviranos para que `dig` faga a consulta ao dominio `asircastelao.int` usando o servidor que lle indicamos, `172.18.0.2`. Unha resposta típica sería: 
```
; <<>> DiG 9.18.27 <<>> @172.18.0.10 asircastelao.int
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: SERVFAIL, id: 10236
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: db2188d2c9e8bdf401000000672e693f293ee1fab23d7c48 (good)
;; QUESTION SECTION:
;asircastelao.int.		IN	A

;; Query time: 0 msec
;; SERVER: 172.18.0.10#53(172.18.0.10) (UDP)
;; WHEN: Fri Nov 08 19:40:47 UTC 2024
;; MSG SIZE  rcvd: 73
```
