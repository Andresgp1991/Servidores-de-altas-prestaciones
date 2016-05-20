# Comprobar el rendimiento de servidores web

En esta practica vamos a manejarnos mediante las direcciones IP de tres máquinas, las cuales son:

- Máquina1: 192.168.1.107 Servidor web 1
- Máquina2: 192.168.1.105 Servidor web 2
- Máquina2: 192.168.1.100 Balanceador

Hay varias herramientas para comprobar el rendimiento de servidores web. Entre las más utilizadas destacan:

- Apache Benchmark
- siege
- httperf
- OpenSTA
- JMeter
- openwebload
- the grinder

Con estas herramientas podemos analizar el rendimiento de servidores Apache, Nginx, Pound etc. Es conveniente ejecutar las herramientas en otra máquina diferente a las que forman parte de la granja web. Cada vez que ejecutemos las herramientas obtendremos resultados diferentes, debido a que el servidor puede encontrarse mas o menos sobrecargado en un momento, por eso haremos al menos 10 ejecuciones para obtener la media.

## Rendimiento de servidores web con Apache Benchmark

Apache Benchmark (ab) es un programa de línea de comandos de un solo subproceso que se instala junto con el servidor Apache para medir el rendimiento de los servidores web.

Para comprobar el rendimiento, creamos una página HTML sencilla y la incluimos en el espacio web de los servidores finales **/var/www/html/**, una vez hecho esto vamos a utilizar el siguiente comando:

		ab -n 1000 -c 5 http://192.168.1.107/index.html

Donde -n 1000 son las peticiones que se harán a esta página y -c 5 hace que se pidan concurrentemente de cinco en cinco.

Lo que haremos será medir el rendimiento de la siguiente manera:

- ab -n 1000 -c 5 http://192.168.1.107/index.html (de una sola máquina).
- ab -n 1000 -c 5 http://192.168.1.100/index.html (al balanceador, utilizando primero Nginx y luego Haproxy).

Esta seria la salida por pantalla del comando **ab**:

		This is ApacheBench, Version 2.3 <$Revision: 1638069 $>
		Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
		Licensed to The Apache Software Foundation, http://www.apache.org/

		Benchmarking 172.16.186.133 (be patient)
		Completed 100 requests
		Completed 200 requests
		Completed 300 requests
		Completed 400 requests
		Completed 500 requests
		Completed 600 requests
		Completed 700 requests
		Completed 800 requests
		Completed 900 requests
		Completed 1000 requests
		Finished 1000 requests


		Server Software:        nginx/1.2.6
		Server Hostname:        172.16.186.133
		Server Port:            80

		Document Path:          /index.html
		Document Length:        68 bytes

		Concurrency Level:      5
		Time taken for tests:   0.741 seconds
		Complete requests:      1000
		Failed requests:        0
		Total transferred:      303000 bytes
		HTML transferred:       68000 bytes
		Requests per second:    1348.86 [#/sec] (mean)
		Time per request:       3.707 [ms] (mean)
		Time per request:       0.741 [ms] (mean, across all concurrent requests)
		Transfer rate:          399.12 [Kbytes/sec] received

		Connection Times (ms)
		              min  mean[+/-sd] median   max
		Connect:        0    0   0.1      0       1
		Processing:     1    3  19.3      2     286
		Waiting:        1    3  19.3      2     286
		Total:          1    4  19.3      2     287

		Percentage of the requests served within a certain time (ms)
		  50%      2
		  66%      2
		  75%      2
		  80%      3
		  90%      3
		  95%      3
		  98%      4
		  99%      9
		 100%    287 (longest request)

Donde nos interesa **Time taken for tests** que es el tiempo que ha tardado en ejecutarse las 1000 peticiones, **Requests per second** que son las peticiones transferidas por segundo.

Una vez hechas las 10 ejecuciones del comando **ab**, obtenemos la siguiente **Tabla1**, en la cual vemos el número de ejecución, y el tiempo en segundos que han tardado en realizarse las 1000 peticiones junto con la velocidad de transferencia medida en peticiones/seg:

| Número | Una máquina | Nginx | Haproxy |
| :----: | :---------: | :---: | :-----: |
| 1 | 0,537 - 1860,87 | 1,893 - 528,26 | 1,722 - 580,69 |
| 2	| 1,16 - 861,7 | 1,777 - 562,73 | 2,059 - 485,61 |
| 3	| 0,542 - 1845,44 | 1,81 - 552,55 | 1,793 - 557,73 |
| 4 | 0,64 - 1562,08 | 1,659 - 602,6 | 1,818 - 550,2 |
| 5 | 0,538 - 1859,35 | 1,624 - 615,78 | 1,983 - 504,38 |
| 6 | 0,549 - 1821,95 | 2,029 - 492,78 | 1,954 - 511,8 |
| 7 | 0,592 - 1689,35 | 1,547 - 646,31 | 2,207 - 453,14 |
| 8 | 0,66 - 1514,55 | 1,564 - 639,4 | 1,796 - 556,67 |
| 9 | 0,631 -1585,5 | 1,871 - 534,46 | 2,48 - 403,29 |
| 10 | 0,764 - 1308,14 | 2,049 - 490,46 | 1,988 - 503 |
| Media |0,661 - 1590,89 | 1,782 - 566,53 | 1,98 - 510,65 |

		Tabla1: Tiempo de respuesta y velocidad Apache Benchmark

En la **Figura1** vemos la representación gráfica del tiempo de respuesta que han necesitado una sola máquina, Nginx y Haproxy, para servir las 1000 peticiones, representado en un diagrama de barras:

![imagen](https://github.com/Andresgp1991/Servidores-web-de-altas-prestaciones/blob/master/Practica4/Imagenes/Figura1.png)

		Figura1

En la **Figura2** vemos la representación gráfica de la velocidad de transferencia a la que han servido las peticiones una sola máquina, Nginx y Haproxy, representado en un diagrama de barras:

![imagen](https://github.com/Andresgp1991/Servidores-web-de-altas-prestaciones/blob/master/Practica4/Imagenes/Figura2.png)

		Figura2

## Rendimiento de servidores web con Siege

Siege es una herramienta de generación de carga HTTP/HTTPS para benchmarking. Se trata de una utilidad de línea de comandos, similar en interfaz al Apache Benchmark, aunque las opciones de ejecución son ligeramente diferentes. Para instalar siege utilizamos el siguiente comando:

		apt-get install siege

Si usamos la opción -b ejecutaremos los test sin pausas con lo que comprobaremos el rendimiento general. Sin la opción -b la herramienta inserta un segundo de pausa entre las diferentes peticiones para cada usuario simulado. Podemos indicar el tiempo que queremos ejecutar Siege, usando la opción -t de la siguiente manera:

- t60S (60 segundos)
- t1H (1 hora
- t120M (120 minutos)

Para comprobar el rendimiento, usaremos la página HTML creada anteriormente, y utilizaremos el siguiente comando:

		siege -b -t60S -v http://192.168.1.107/index.html

Donde la opción -b ejecuta los test sin pausa, -t60S con un tiempo de ejecución de 60 segundos y la opción -v imprime el estado de retorno HTTP y la solicitud GET a la pantalla.

Lo que haremos será medir el rendimiento de la siguiente manera:

- siege -b -t60S -v http://192.168.1.107/index.html (de una sola máquina).
- siege -b -t60S -v http://192.168.1.100/index.html (al balanceador, utilizando primero Nginx y luego Haproxy).

Esta seria la salida por pantalla del comando **siege**:

		Lifting the server siege...      done.

		Transactions:		      192905 hits
		Availability:		      100.00 %
		Elapsed time:		       60.16 secs
		Data transferred:	       12.51 MB
		Response time:		        0.00 secs
		Transaction rate:	     3206.53 trans/sec
		Throughput:		        0.21 MB/sec
		Concurrency:		       14.88
		Successful transactions:      192905
		Failed transactions:	           0
		Longest transaction:	        3.89
		Shortest transaction:	        0.00

Donde nos interesa **Transactions** que son las peticiones que se han realizado, **Availability** la disponibilidad del servidor, **Elapsed time** tiempo de ejecución y **Transaction rate** número de peticiones por segundo.

Una vez hechas las 10 ejecuciones del comando **siege**, obtenemos la siguiente **Tabla2**, en la cual vemos el número de ejecución, y el numero de peticiones que se han realizado junto con el tiempo de ejecucion de siege y la velocidad de transferencia medida en peticiones/seg:

| Número | Una máquina | Nginx | Haproxy |
| :----: | :---------: | :---: | :-----: |
| 1 | 74303 - 59,76 - 1243,36 | 26380 - 59,06 - 446,66 | 25958 - 59,86 - 433,65 |
| 2 | 71964 - 59,39 - 1211,72 | 34299 - 59,74 - 574,14 | 25000 - 59,71 - 418,69 |
| 3 | 86848 - 59,03 - 1471,25 | 30570 - 59,41 - 514,56 | 22719 - 59,07 - 384,61 |
| 4 | 93202 - 59,85 - 1557,26 | 27497 - 59,51 - 462,06 | 29870 - 59,9 - 498,66 |
| 5 | 72668 - 59,5 - 1221,31 | 25427 - 59,6 - 426,63 | 25825 - 59,25 - 435,86 |
| 6 | 83353 - 59,83 - 1393,16 | 28494 - 59,93 - 475,45 | 28066 - 59,6 - 470,91 |
| 7 | 64317 - 59,32 - 1084,24 | 31200 - 59,33 - 525,87 | 28713 - 59,31 - 484,12 |
| 8 | 65578 - 59,36 - 1104,75 | 31845 - 59,58 - 534,49 | 26651 - 59,09 - 451,02 |
| 9 | 84150 - 59,88 - 1405,31 | 29147 - 59,58 - 489,21 | 24253 - 59,01 - 411 |
| 10 | 90323 - 59,97 - 1506,14 | 31161 - 59,26 - 525,84 | 31424 - 59,72 - 526,19 |
| Media | 78671 - 59,58 - 1319,85 | 29602 - 59,5 - 497,49 | 26848 - 59,45 - 451,47 |

		Tabla2: Peticiones, tiempo de ejecución y velocidad de siege

En la **Figura3** vemos la representación gráfica del numero de peticiones realizadas en un minuto a una sola máquina, Nginx y Haproxy, representado en un diagrama de barras:

![imagen](https://github.com/Andresgp1991/Servidores-web-de-altas-prestaciones/blob/master/Practica4/Imagenes/Figura3.png)

		Figura3

En la **Figura4** vemos la representación gráfica de la velocidad de transferencia a la que han servido las peticiones una sola máquina, Nginx y Haproxy, representado en un diagrama de barras:

![imagen](https://github.com/Andresgp1991/Servidores-web-de-altas-prestaciones/blob/master/Practica4/Imagenes/Figura4.png)

		Figura4

## Rendimiento de servidores web con Httperf

httperf es una herramienta de línea de comandos, al contrario que Siege, httperf se ejecuta en un único thread. Por otra parte, httperf dispone de más opciones de bajo nivel para la configuración de la prueba. Para instalarlo utilizamos el siguiente comando:

		apt-get install httperf

Las opciones que permite httperf son:

- server: El hostname de la web ha realizar el test.
- uri: La url de la pagina que se abrirá.
- rate: Cuantas peticiones se quieren enviar por segundo.
- num-conn: El total de conexiones que se abrirán.
- num-call: Cuantas peticiones se enviaran por conexión.
- timeout: Cuantos segundos ha de esperar para que se considere que la petición se ha perdido.

Para comprobar el rendimiento, usaremos la página HTML creada anteriormente, y utilizaremos el siguiente comando:

		httperf --server 192.168.1.100 --port 80 --uri /index.html --rate 300 --num-conn 30000 --num-call 1 --timeout 5

En el ejemplo que he puesto, httperf hará 300 peticiones por segundo a http://192.168.1.100/index.html con un total de 30000 peticiones.

Pero vamos a poner otro ejemplo el cual actúa como Apache Benchmark, ejecutando el comando de la siguiente manera:

		httperf --server 192.168.1.100 --port 80 --uri /index.html --num-conn 1000 --num-call 1 --timeout 5

En este ejemplo httperf hará 1000 peticiones únicamente, no está obligado a realizar tantas peticiones por segundo como en el ejemplo anterior. 

Lo que haremos será medir el rendimiento de la siguiente manera:

- httperf --server 192.168.1.100 --port 80 --uri /index.html --num-conn 1000 --num-call 1 --timeout 5 (de una sola máquina).
- httperf --server 192.168.1.100 --port 80 --uri /index.html --num-conn 1000 --num-call 1 --timeout 5 (al balanceador, utilizando primero Nginx y luego Haproxy).

Esta seria la salida por pantalla del comando **httperf**:

		httperf: warning: open file limit > FD_SETSIZE; limiting max. # of open files to FD_SETSIZE
		Maximum connect burst length: 1

		Total: connections 1000 requests 1000 replies 1000 test-duration 1.028 s

		Connection rate: 973.1 conn/s (1.0 ms/conn, <=1 concurrent connections)
		Connection time [ms]: min 0.7 avg 1.0 max 81.3 median 0.5 stddev 2.7
		Connection time [ms]: connect 0.2
		Connection length [replies/conn]: 1.000

		Request rate: 973.1 req/s (1.0 ms/req)
		Request size [B]: 77.0

		Reply rate [replies/s]: min 0.0 avg 0.0 max 0.0 stddev 0.0 (0 samples)
		Reply time [ms]: response 0.8 transfer 0.0
		Reply size [B]: header 240.0 content 68.0 footer 0.0 (total 308.0)
		Reply status: 1xx=0 2xx=1000 3xx=0 4xx=0 5xx=0

		CPU time [s]: user 0.16 system 0.87 (user 15.2% system 84.9% total 100.0%)
		Net I/O: 365.9 KB/s (3.0*10^6 bps)

		Errors: total 0 client-timo 0 socket-timo 0 connrefused 0 connreset 0
		Errors: fd-unavail 0 addrunavail 0 ftab-full 0 other 0

Donde nos interesa **test-duration** que es el tiempo de ejecución de todas las peticiones, **Resquest rate** peticiones transferidas por segundo, y abajo podemos ver el uso de CPU y los errores durante la ejecución.

Una vez hechas las 10 ejecuciones del comando **httperf**, obtenemos la siguiente **Tabla3**, en la cual vemos el número de ejecución, y el tiempo en segundos que han tardado en realizarse las 1000 peticiones junto con la velocidad de transferencia medida en peticiones/seg:

| Número | Una máquina | Nginx | Haproxy |
| :----: | :---------: | :---: | :-----: |
| 1 | 0,315 - 3171,7 | 0,892 - 1121,6 | 0,881 - 1135,3 |
| 2 | 0,284 - 3523,9 | 0,897 - 1115,1 | 0,868 - 1152,6 |
| 3 | 0,312 - 3205,8 | 0,876 - 1142 | 0,87 - 1149,6 |
| 4 | 0,313 - 3198,4 | 0,875 - 1142,5 | 0,869 - 1150,6 |
| 5 | 0,3 - 3334 | 0,931 - 1074,1 | 0,876 - 1141,8 |
| 6 | 0,302 - 3316,7 | 0,919 - 1088,6 | 0,892 - 1121,1 |
| 7 | 0,326 - 3069,4 | 0,878 - 1139,3 | 0,878 - 1139,6 |
| 8 | 0,292 - 3427,9 | 0,896 - 1116,5 | 0,91 - 1098,3 |
| 9 | 0,312 - 3206,8 | 0,892 - 1120,9 | 0,886 - 1128,9 |
| 10 | 0,307 - 3259,3 | 1,101 - 908,6 | 0,879 - 1137,5 |
| Media | 0,306 - 3271,39 | 0,915 - 1096,92 | 0,881 - 1135,53 |

		Tabla3: Tiempo de respuesta y velocidad Httperf

En la **Figura5** vemos la representación gráfica del tiempo de respuesta que han necesitado una sola máquina, Nginx y Haproxy, para servir las 1000 peticiones, representado en un diagrama de barras:

![imagen](https://github.com/Andresgp1991/Servidores-web-de-altas-prestaciones/blob/master/Practica4/Imagenes/Figura5.png)

		Figura5

En la **Figura6** vemos la representación gráfica de la velocidad de transferencia a la que han servido las peticiones una sola máquina, Nginx y Haproxy, representado en un diagrama de barras:

![imagen](https://github.com/Andresgp1991/Servidores-web-de-altas-prestaciones/blob/master/Practica4/Imagenes/Figura6.png)

		Figura6

En la **Tabla4** vemos las media de todas las tablas que hemos hecho, sacando la conclusión de que la herramienta Httperf nos proporciona una carga mucho mayor de peticiones al servidor por segundo.

|        | Una máquina | Nginx | Haproxy |
| :----: | :---------: | :---: | :-----: |
| Media Apache Benchmark |0,661 - 1590,89 | 1,782 - 566,53 | 1,98 - 510,65 |
| Media Siege | 78671 - 1319,85 | 29602 - 497,49 | 26848 - 451,47 |
| Media Httperf | 0,306 - 3271,39 | 0,915 - 1096,92 | 0,881 - 1135,53 |

		Tabla4: Conclusion de todas las herramientas

En la **Tabla5** hemos hecho la media de las tres herramientas en cada uno de los escenarios que las hemos usado y la conclusión es que httperf ofrece una velocidad de transferencia mucho mayor para sobrecargar el servidor web, y una serie de opciones mas completa.

|        | Apache Benchmark | Siege | Httperf |
| :----: | :---------: | :---: | :-----: |
| Peticiones/seg | 889,35 | 756,27 | 1834,61 |

		Tabla5: Media de las tres herramientas

![imagen](https://github.com/Andresgp1991/Servidores-web-de-altas-prestaciones/blob/master/Practica4/Imagenes/Figura7.png)

		Figura7: Media de las tres herramientas
