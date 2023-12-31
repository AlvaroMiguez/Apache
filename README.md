# Tarea Apache + Virtual Host

1. Creamos el docker compose donde añadimos el DNS el Apache y un cliente
```
services:
  asir_apache-2:
    container_name: asir_apache-2
    image: httpd:latest
    ports:
      - "80:80"
    # Mapeamos el puerto 80 de la máquina con el 80 del contenedor
    networks:
      mi-red:
        ipv4_address: 10.1.0.50
        # ip fija del Apache
    volumes:
      - ./paginas:/usr/local/apache2/htdocs
      - ./conf:/usr/local/apache2/conf
      # Mapeamos los volumenes para así poder meter nuestra configuración de una manera más fácil
  asir_bind9-2:
    container_name: asir_bind9-2
    image: ubuntu/bind9
    # usamos esta imagen para ver los logs
    platform: linux/amd64
    ports:
      - 53:53
      #Mapeo de puertos
    networks:
    #Creamos la red y le damos ip al DNS
      mi-red:
        ipv4_address: 10.1.0.2
    volumes:
    #Mapeo de volumenes
      - ./config:/etc/bind
      - ./zonas:/var/lib/bind
  cliente-2:
    container_name: asir_cliente-2
    image: ubuntu
    platform: linux/amd64
    networks:
      mi-red:
       ipv4_address: 10.1.0.5
       # Damos una ip fija al cliente 
    tty: true
    stdin_open: true
    dns:
      - 10.1.0.2
      # Le indicamos que el DNS que utilizará será el creado anteriormente
networks:
  mi-red:
    ipam:
      driver: default
      config:
        - subnet: 10.1.0.0/16 
          gateway: 10.1.0.1
  #Configuramos un poco la red que vamos a utilizar en la práctica
```
2. Configuración de los volumenes
    - Configuración DNS
        Para el DNS tenemos los volúmenes config y zonas.
        
        En config tenemos los siguientes archivos:

        (named.conf)
        ```
        include "/etc/bind/named.conf.options";
        include "/etc/bind/named.conf.local";
        ```
        (named.conf.local) 

        Este archivo es el único que hay que cambiar a diferencia de otras prácticas ya que hay que añadirle una zona más en el archivo.
        ```
        
        zone "fabulasmaravillosas.int" {
            type master;
            file "/var/lib/bind/db.fabulasmaravillosas.int";
            allow-query {
                any;
                };
            };

        zone "fabulasoscuras.int" {
            type master;
            file "/var/lib/bind/db.fabulasoscuras.int";
            allow-query {
                any;
                };
            };
        ```
        (named.conf.options)
        ```
        options {
            directory "/var/cache/bind";

            forwarders {
                8.8.8.8;
                1.1.1.1;
            } ;
            forward only;

            listen-on { any; };
            listen-on-v6 { any; };

            allow-query {
                any;
            };
        };
        ```

        En zonas tenemos los siguientes archivos:

        (db.fabulasoscuras.int)
        ```
        $TTL 38400	; 10 hours 40 minutes
        @		IN SOA	ns.fabulasoscuras.int. some.email.address. (
                        10000003   ; serial
                        10800      ; refresh (3 hours)
                        3600       ; retry (1 hour)
                        604800     ; expire (1 week)
                        38400      ; minimum (10 hours 40 minutes)
                        )
        @		IN NS		ns
        ns		IN A		10.1.0.2
         www		IN A		10.1.0.50
        ```

        (db.fabulasmaravillosas.int)

        ```
         $TTL 38400	; 10 hours 40 minutes
        @		IN SOA	ns.fabulasmaravillosas.int. some.email.address. (
                        10000004  ; serial
                        10800      ; refresh (3 hours)
                        3600       ; retry (1 hour)
                        604800     ; expire (1 week)
                        38400      ; minimum (10 hours 40 minutes)
                        )
        @		IN NS		ns
        ns		IN A		10.1.0.1
        www		IN A		10.1.0.50
        ```
    - Configuración Apache
        Para el Apache tenemos los volúmenes conf y paginas.
        
        En conf tenemos los siguientes archivos:

        (httpd.conf)

        En este archivo es donde hacemos los cambios en la configuración  y para copiar la configuración estándar hacemos `docker run --rm httpd:2.4 cat /usr/local/apache2/conf/httpd.conf > httpd.conf`
         
        (mime.types)

        Este archivo solo es para copiar la configuración no haremos ningún cambio y para copiarla es igual que el anterior cambiando los https.conf por mime.types

3. Creación de los Virtual Host

  Los Virtual Host se añaden al archivo de configuración al final del todo

  (Virtual Host 1)

  ```
  <VirtualHost *:80>
  ServerName www.fabulasoscuras.int
  ServerAlias fabulasoscuras.int *.fabulasoscuras.int
  DocumentRoot /paginas/oscuras
  </VirtualHost>
  ```

  (Virtual Host 2)
  ```
  <VirtualHost *:80>
  ServerName www.fabulasmaravillosas.int
  ServerAlias fabulasmaravillosas.int *.fabulasmaravillosas.int
  DocumentRoot /paginas/maravillosas
  </VirtualHost>
  ```
4. Comprobación

  - Primero de todo entramos en el cliente y hacemos un `apt update` y  luego instalamos la herramienta ping con `apt install iputils-ping` y hacemos ping a fabulasoscuras y fabulasmaravillosas si funciona bien debería de funcionar.

  - Luego hacemos un dig ha ambos por si acaso aunque debería de funcionar perfectamente
  
  - Por último hay dos opciones o instalas una opción de navegador en el cliente y así poder ver los dominios y sus respectivos index, por otro lado, si no quieres hacer eso cambias en los archivos de el equipo para que en el navegador puedan acceder ambos dominios y aparecerían si está bien hecho


