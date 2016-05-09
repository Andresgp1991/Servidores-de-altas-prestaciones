# Balanceo de carga

En esta practica vamos a manejarnos mediante las direcciones IP de tres máquinas, las cuales son:

- Máquina1: 192.168.1.108 Servidor web 1
- Máquina2: 192.168.1.109 Servidor web 2
- Máquina3: 192.168.1.111 Balanceador

## El servidor web nginx

Procedemos a instalar el servidor nginx, lo primero que tenemos que hacer es importar la clave del repositorio del software, para ello utilizamos los siguientes comandos:

**cd /tmp/**

**wget http://nginx.org/keys/nginx_signing.key**

**apt-key add /tmp/nginx_signing.key**

**rm -f /tmp/nginx_signing.key**

A continuación, debemos añadir el repositorio al fichero /etc/apt/sources.list:

**echo "deb http://nginx.org/packages/ubuntu/ lucid nginx" >> /etc/apt/sources.list**

**echo "deb-src http://nginx.org/packages/ubuntu/ lucid nginx" >> /etc/apt/sources.list**

Una vez añadidos los repositorios, procedemos a instalar nginx:

**apt-get update**

**apt-get install nginx**

### Balanceo de carga usando nginx

Para que nginx funcione como balanceador tenemos que modificar el fichero **/etc/nginx/conf.d/default.conf**, primero tenemos que definir que máquinas formaran nuestra granja web, en la sección *upstream* añadiendo las direcciones IP de nuestros servidores (en este caso la máquina1 y máquina 2) quedando de esta manera:

	upstream apaches {
		server 172.16.168.108;
		server 172.16.168.109;
	}

En la siguiente sección *sever*, es importante para que el proxy_pass funcione correctamente que indiquemos que la conexión entre nginx y los servidores finales sea HTTP 1.1 así como especificarle que debe eliminar la cabecera Connection para evitar que se pase al servidor final la cabecera que indica el usuario, quedando de esta manera:

	server{
		listen 80;
		server_name balanceador;

		access_log /var/log/nginx/balanceador.access.log;
		error_log /var/log/nginx/balanceador.error.log;
		root /var/www/;**

		location / {
			proxy_pass http://apaches;
			proxy_set_header Host $host;
			proxy_set_header X-Real-IP $remote_addr;
			proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
			proxy_http_version 1.1;
			proxy_set_header Connection "";
		}
	}

En la **figura1** podemos ver la configuración del archivo **/etc/nginx/conf.d/default.conf**:

![imagen](https://github.com/Andresgp1991/Servidores-web-de-altas-prestaciones/blob/master/Practica3/Figura1.png)
				
		Figura1: configuración de nginx "round-robin"

Con esta configuración hemos usado balanceo mediante el algoritmo **"round-robin"** con la misma prioridad para todos los servidores. Para que tengan efecto los cambios reiniciamos el servicio nginx:

**service nginx restart**

Podemos probar la configuración haciendo peticiones a la IP de esta máquina, usamos el comando curl y podemos ver que reparte la misma carga a las máquinas como vemos en la **figura2**:

![imagen](https://github.com/Andresgp1991/Servidores-web-de-altas-prestaciones/blob/master/Practica3/Figura2.png)
				
		Figura2: comando curl "round-robin"

Si sabemos que alguna de las máquinas finales es más potente, podemos modificar la sección *upstream* para pasarle más tráfico que al resto. Para ello, tenemos un modificador llamado “weight”, al que le damos un valor numérico que indica la carga que le asignamos, por defecto tiene el valor 1, que repartirá la misma carga a todas las máquinas:

	upstream apaches {
		server 172.16.168.108 weight=2;
		server 172.16.168.109 weight=1;
	}

En este ejemplo la máquina2 recibirá menos carga que la primera, de cada tres peticiones la máquina1 atenderá dos de ellas, y la máquina2 una. El archivo de configuración quedaría de la siguiente forma como vemos en la **Figura3**:

![imagen](https://github.com/Andresgp1991/Servidores-web-de-altas-prestaciones/blob/master/Practica3/Figura3.png)
		
		Figura3: configuración de nginx "round-robin" ponderado

También podemos hacer un balanceo por IP, todo el tráfico que venga de una misma IP se servirá durante toda la sesión por la misma máquina. Para ello, usaremos la directiva ip_hash en la sección *upstream*:

	upstream apaches {
		ip_hash;
		server 172.16.168.108;
		server 172.16.168.109;
	}

La desventaja del balanceo ip_hash es que todos los usuarios detrás de un proxy serán dirigidos a la misma máquina, por lo que puede a ver una sobrecarga de peticiones en un servidor. Para evitar esto, los balanceadores modernos permiten balancear usando una cookie, que identifica a los usuarios.

### Opciones de configuración del nginx para establecer cómo le pasará trabajo a las máquinas servidoras finales

Existen otras opciones para gestionar posibles errores o caídas de los servidores:

- **weight = NUMBER**: permite especificar un peso para el servidor (por defecto es 1).
- **max_fails = NUMBER**: especifica un número de intentos de comunicación erróneos en "fail_timeout" segundos para considerar al servidor no operativo (por defecto es 1, un valor de 0 lo desactivaría).
- **fail_timeout = TIME**: indica el tiempo en el que deben ocurrir "max_fails" intentos fallidos de conexión para considerar al servidor no operativo. Por defecto es 10 segundos.
- **down**: marca el servidor como permanentemente offline (para ser usado con ip_hash).
- **backup**: reserva este servidor y sólo le pasa tráfico si alguno de los otros servidores no-backup está caído u ocupado. No es compatible con la directiva ip_hash

## Balanceo de carga con haproxy

Lo primero que tenemos que hacer es instalar **haproxy**, para ello utilizaremos el comando:

**apt-get install haproxy**

Una vez instalado, debemos modificar el archivo **/etc/haproxy/haproxy.cfg**, editamos el fichero de configuración de haproxy quedando de la siguiente manera:

	global
		daemon
		maxconn 256
	defaults
		mode http
		contimeout 4000
		clitimeout 42000
		srvtimeout 43000
	frontend http-in
		bind *:80
		default_backend servers
	backend servers
		server m1 172.16.168.108:80 maxconn 32
		server m2 172.16.168.109:80 maxconn 32

En la **figura4** podemos ver la configuración del archivo **/etc/haproxy/haproxy.cfg**:

![imagen](https://github.com/Andresgp1991/Servidores-web-de-altas-prestaciones/blob/master/Practica3/Figura4.png)
				
		Figura4: configuración de haproxy

Vemos que en la sección *backend servers* configuramos nuestros servidores finales (máquina1 y máquina2), y en la sección *frontend http-in* el puerto en el que está escuchando nuestro servidor *haproxy*, el puerto 80. Una vez configurado el servidor, lanzamos el servicio haproxy mediante el comando:

**/usr/sbin/haproxy -f /etc/haproxy/haproxy.cfg**

Podemos probar la configuración haciendo peticiones a la IP de esta máquina, usamos el comando curl y podemos ver que reparte la misma carga a las máquinas.

## Balanceo de carga con Pound

Pound es un reverse-proxy capaz de realizar balance de carga. Pound acepta peticiones de clientes HTTP/HTTPS y las distribuye hacia uno o varios servidores web. Pound es capaz de detectar cuando un servidor web no responde, evitando enviar peticiones a éstos hasta que restablezcan su función correcta.

- **Listeners**: Los Listeners definen la forma en que Pound recibe las peticiones de los clientes. Los dos tipos posibles de Listeners son HTTP y HTTPS.
- **Servicios**: Los servicios definen el modo en que se responderá a las peticiones. Los servicios pueden ser definidos dentro de un listener (actuando únicamente para las peticiones que cumplan las condiciones definidas en ese listener), o globalmente, actuando para todas las peticiones.
- **Back-Ends**: Los Back-Ends identifican a los servidores web reales que contienen la información pedida por el cliente. En cada servicio pueden definirse varios Back-Ends. En ese caso Pound los elegirá aleatoriamente basándose en las prioridades asignadas a cada uno.

Para configurar Pound tenemos que modificar el archivo de configuración **/etc/pound/pound-cfg** quedando de la siguiente manera:

	User	“www-data”
	Group	“www-data”

	LogLevel	1

	Alive	30

	control "/var/run/pound/poundctl.socket"

	ListenHTTP
		Address 192.168.1.111
		Port	80
        
		Service
			BackEnd
				Address 192.168.1.108
				Port	80
			End

			BackEnd
				Address 192.168.1.109
				Port	80
			End
		End
	End

![imagen](https://github.com/Andresgp1991/Servidores-web-de-altas-prestaciones/blob/master/Practica3/Figura5.png)
			
		Figura5: configuración de Pound

Podemos probar la configuración haciendo peticiones a la IP de esta máquina, usamos el comando curl y podemos ver que da el doble de prioridad a la máquina1 como vemos en la **figura6**:

![imagen](https://github.com/Andresgp1991/Servidores-web-de-altas-prestaciones/blob/master/Practica3/Figura6.png)
						
		Figura6: comando curl Pound con prioridad
