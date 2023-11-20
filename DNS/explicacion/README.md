## Tarea DNS+Cliente

## Explicación
Como ya teniamos creados los archivos de configuración y de zonas anteriormente no hace falta configurarlos otra vez a no ser que quieras cambiar algo pero en este caso lo dejamos igual solo que añadiendo la carpeta para logs.

1. Hacemos el compose
~~
services:
  asir_bind9:
    container_name: asir_bind9
    image: ubuntu/bind9
    # usamos esta imagen para ver los logs
    # la imagen internetconsortium no muestra logs
    platform: linux/amd64
    ports:
      - 53:53
      #Mapeo de puertos
    networks:
    #Creamos la red y le damos ip al DNS
      bind9_subnet:
        ipv4_address: 172.28.5.1
    volumes:
    #Volumenes
      - ./config:/etc/bind
      - ./lib:/var/lib/bind
      - ./logs:/var/log/bind
  cliente:
    container_name: asir_cliente
    image: ubuntu
    platform: linux/amd64
    tty: true
    stdin_open: true
    networks:
      - bind9_subnet
networks:
  bind9_subnet:
    external: true
~~

2. Ejecutamos el docker compose up
~~
docker compose up (mirar que estamos en el directorio correcto)
~~
3. Iniciamos los contenedores
Hacemos click derecho para hacer un attach shell

4. Una vez dentro hacemos un apt update y apt install iputils-ping 
~~
apt update
apt install iputils-ping
~~
5. Si vemos que funciona pasamos ya a utilizar el dig primero lo instalamos y luego comprobamos si hay resolución de los pedido.
~~
apt install dnsutils
dig @172.28.5.1 test.asircastelao.int
~~
6. Comprobación




![imagen](../Comprobaci%C3%B3n.png)