# Clonar la información de un sitio web

## Funcionamiento de la copia de archivos por ssh

En esta practica vamos a manejarnos mediante las direcciones IP de las dos máquinas, las cuales son:

- Máquina1: 192.168.1.107
- Máquina2: 192.168.1.109

Podemos copiar archivos de forma muy sencilla mediante el comando scp de ssh, mediante el comando:

**scp ~/Andres.html 192.168.1.109:/var/www/**

De esta forma copiariamos el archivo de la máquina1 **Andres.html** en el directorio /var/www/ de nuestra máquina2 con dirección IP **192.168.1.109**

## Clonado de una carpeta entre las dos máquinas

Mediante la herramienta rsync procedemos a clonar la carpeta /var/www/ de la máquina2 a la máquina1 mediante el comando:

**rsync -avz -e ssh 192.168.1.109:/var/www/ /var/www/**

En la **Figura1** podemos ver los archivos que se están recibiendo de la máquina2, en este caso el archivo Andres.html que habíamos copiado en el apartado anterior:

![imagen](https://github.com/Andresgp1991/Servidores-web-de-altas-prestaciones/blob/master/Practica2/Imagenes/Figura1.png)
		Figura 1: clonado de la carpeta /var/www/

## Configuración de ssh para acceder sin que solicite contraseña

Mediante ssh-keygen generamos las claves pública y privada mediante el comando:

**ssh-keygen -t dsa**

dsa: genera el fichero id_dsa para la clave privada, y el fichero id_dsa.pub para la clave pública.

Para hacer la copia de la clave usaremos el comando ssh-copy-id. Lo que haremos es copiarla a la máquina1, a la que querremos acceder sin tener que introducir la contraseña, le enviaremos la clave pública desde la máquina2 a la máquina1 mediante el comando:

**ssh-copy-id -i .ssh/id_dsa.pub 192.168.1.107**

En la **Figura2** podemos ver que haciendo ssh a la máquina2, no nos pide la contraseña para conectarnos:

![imagen](https://github.com/Andresgp1991/Servidores-web-de-altas-prestaciones/blob/master/Practica2/Imagenes/Figura2.png)

**Figura 2: conexión ssh sin contraseña**

## Programar tareas con crontab

Cron es un administrador procesos en segundo plano que ejecuta procesos en el instante indicado en el fichero crontab, este fichero tiene el siguiente aspecto:

![imagen](https://github.com/Andresgp1991/Servidores-web-de-altas-prestaciones/blob/master/Practica2/Imagenes/Figura3.png)

**Figura 3: fichero crontab**

Los 7 campos están organizados de la siguiente manera:

- Minuto: indica el minuto de la hora en que el comando será ejecutado.
- Hora: indica la hora en que el comando será ejecutado.
- DíaDelMes: indica el día del mes en que se quiere ejecutar el comando.
- Mes: indica el mes en que el comando se ejecutará (1-12).
- DíaDeLaSemana: indica el día de la semana en que se ejecutará el comando
(1=lunes y hasta 7=domingo).
- Usuario: indica el usuario que ejecuta el comando.
- Comando: indica el comando que se desea ejecutar.

Lo que vamos a hacer es introducir una tarea a la lista de crontab para que la carpeta **/var/www/** de la máquina1 sea clonada en la máquina2 cada hora, exactamente en el minuto 59 de cada hora, la linea de código que tendríamos que añadir al fichero crontab seria:

59 * * * * root rsync -avz -e ssh 192.168.1.109:/var/www/ /var/www/

El fichero crontab quedaría de la siguiente forma como vemos en la **Figura4**:

![imagen](https://github.com/Andresgp1991/Servidores-web-de-altas-prestaciones/blob/master/Practica2/Imagenes/Figura4.png)

**Figura 4: modificacion fichero crontab**










