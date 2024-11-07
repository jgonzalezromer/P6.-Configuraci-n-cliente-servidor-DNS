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
### Recursos
Utilizaremos o [repositorio de Damián](https://github.com/damiancastelao/practica01-DNS)
---
### docker-compose.yml
O primeiro que faremos é modificar o docker-compose.yml que fixemos con anterioridade:
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


