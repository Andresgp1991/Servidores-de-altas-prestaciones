# Clonar la información de un sitio web

## Funcionamiento de la copia de archivos por ssh

En esta practica vamos a manejarnos mediante las direcciones IP de las dos máquinas, las cuales son:

- Maquina1: 192.168.1.107
- Maquina2: 192.168.1.109

Podemos copiar archivos de forma muy sencilla mediante el comando scp de ssh, mediante el comando:

**scp ~/Andres.html 192.168.1.109:/var/www/**

De esta forma copiariamos el archivo de la máquina1 **Andres.html** en el directorio /var/www/ de nuestra máquina2 con dirección IP **192.168.1.109**

## Clonado de una carpeta entre las dos máquinas

Mediante la herramienta rsync procedemos a clonar la carpeta /var/www/ de la máquina2 a la máquina1 mediante el comando:

**rsync -avz -e ssh 192.168.1.109:/var/www/ /var/www/**

En la **Figura1** podemos ver los archivos que se estan reciviendo de la maquina2, en este caso el archivo Andres.html que habiamos copiado en el apartado anterior:

![imagen](https://github.com/Andresgp1991/Servidores-web-de-altas-prestaciones/blob/master/Practica2/Figura1.png)

**Figura 1: clonado de la carpeta /var/www/**

## Configuración de ssh para acceder sin que solicite contraseña

Mediante ssh-keygen generamos las claves pública y privada mediante el comando:

**ssh-keygen -t dsa**

dsa: genera el fichero id_dsa para la clave privada, y el fichero id_dsa.pub para la clave pública.

Para hacer la copia de la clave usaremos el comando ssh-copy-id. Lo que haremos es copiarla a la máquina1, a la que querremos acceder sin tener que introducir la contraseña, le enviaremos la clave pública desde la máquina2 a la máquina1 mediante el comando:

**ssh-copy-id -i .ssh/id_dsa.pub 192.168.1.107**

En la **Figura2** podemos ver que haciendo ssh a la máquina2, no nos pide la contraseña para conectarnos:

![imagen](https://github.com/Andresgp1991/Servidores-web-de-altas-prestaciones/blob/master/Practica2/Figura2.png)

**Figura 2: conexión ssh sin contraseña**





