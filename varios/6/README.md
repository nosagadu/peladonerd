## Tomado de: https://github.com/kylemanna/docker-openvpn/blob/master/docs/docker-compose.md

## **OpenVPN: Instalarlo con docker-compose**

*   Crea un archivo docker-compose.yaml

```yaml
version: '2'
services:
  openvpn:
    cap_add:
     - NET_ADMIN
    image: kylemanna/openvpn
    container_name: openvpn
    ports:
     - "1194:1194/udp"
    restart: always
    volumes:
     - ./openvpn-data/conf:/etc/openvpn
```

*   Inicializa los archivos de configuracion y certificados

```bash
docker-compose run --rm openvpn ovpn_genconfig -u udp://<IP-DE-TU-SERVIDOR>
docker-compose run --rm openvpn ovpn_initpki
```

*   Arregla tus permisos (puede no ser necesario si ya estás haciendo todo con root)

```bash
sudo chown -R $(whoami): ./openvpn-data
```

*   Inicia el contenedor de OpenVPN

```bash
docker-compose up -d
```

*   Puedes ver los logs de contenedor con:

```bash
docker-compose logs -f
```

*   Generar un certificado de cliente

```bash
export CLIENTNAME="el_nombre_del_cliente"
# con contraseña
docker-compose run --rm openvpn easyrsa build-client-full $CLIENTNAME
# sin contraseña
docker-compose run --rm openvpn easyrsa build-client-full $CLIENTNAME nopass
```

###### _\* Debes setear el CLIENTNAME cada vez que deseas crear en certificado_

*   Crea el archivo de configuración del cliente

```bash
docker-compose run --rm openvpn ovpn_getclient $CLIENTNAME > $CLIENTNAME.ovpn 
```

*   Revoca el certificado de un cliente

```bash
# Dejando los archivos crt, key y req.
docker-compose run --rm openvpn ovpn_revokeclient $CLIENTNAME
# Borrando los correspondientes archivos crt, key y req.
docker-compose run --rm openvpn ovpn_revokeclient $CLIENTNAME remove
```

## OpenVPN y Pi-Hole

Configurar OpenVPN para que trabaje con Pi-hole como servidor DNS así los clientes conectados a la VPN puede seguir protegiendo su tráfico con Pi-hole.

*   Debes crear una red donde le puedas asignar manualmente la IP a los contenedores

```shell
networks:
    vpn-network:
        external: true
```

*   Asignas la red a los contenedores de openvpn y pi-hole

```
openvpn:
    networks:
        vpn-network:
            ipv4_address: 172.16.68.61

pihole:
    networks:
        vpn-network:
            ipv4_address: 172.16.68.62
```

*   El comando para crear la red manualmente

```shell
docker network create --driver=bridge --subnet=172.16.68.0/24 --gateway=172.16.68.1 vpn-network
```

*   Necesitas decirle a openvpn que utilice tu servidor DNS para eso debes modificar estas lineas en el siguiente archivo: _./openvpn-data/conf/server.conf_

```shell
push "dhcp-option DNS IP_CONTAINER_PI-HOLE" 
#push "dhcp-option DNS 8.8.8.8"
```
