## Tarea DNS

1. Hacer el compose
~~~
services:
  asir_bind9:
    container_name: asir_bind9
    image: internetsystemsconsortium/bind9:9.16
    ports:
      - 53:53
    networks:
      bind9_subnet:
        ipv4_address: 172.28.5.1
    volumes:
      - ./config:/etc/bind
      - ./lib:/var/lib/bind
  asir_cliente:
    container_name: asir_cliente
    image: alpine
    networks:
      - bind9_subnet
    stdin_open: true # docker run -i
    tty: true # docker run -t
    dns:
      - 172.28.5.1 # el contenedor dns server
networks:
  bind9_subnet:
    external: true
~~~
2. Crear la red para el DNS
~~~
docker network create   
    --driver=bridge   
    --subnet=172.28.0.0/16   
    --ip-range=172.28.5.0/24   
    --gateway=172.28.5.254   
bind9_subnet
~~~
3. configurar los archivos conf
(named.conf)
~~~
include "/etc/bind/named.conf.options";
include "/etc/bind/named.conf.local";
~~~
(named.conf.local)
~~~
zone "asircastelao.int" {
	type master;
	file "/var/lib/bind/db.asircastelao.int";
	allow-query {
		any;
		};
	};
~~~
(named.conf.options)
~~~
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
~~~
4. Configuración en  lib
~~~
$TTL 38400	; 10 hours 40 minutes
@		IN SOA	ns.asircastelao.int. some.email.address. (
				10000002   ; serial
				10800      ; refresh (3 hours)
				3600       ; retry (1 hour)
				604800     ; expire (1 week)
				38400      ; minimum (10 hours 40 minutes)
				)
@		IN NS	ns.asircastelao.int.
ns		IN A		172.28.5.1
test	IN A		172.28.5.4
alias	IN CNAME	test
texto	IN TXT		mensaje
~~~
5. Docker compose up
(*"recordar estar en el directorio correcto"*)
~~~
docker compose up
~~~
## Comprobación
Instalamos el dig con estos comandos ya dentro del contenedor
~~~
apt update
apt install -y dnsutils
~~~
Una vez hecho esto comprobamos con el comando dig

