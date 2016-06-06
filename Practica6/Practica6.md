# Discos en RAID

RAID es que la combinación de discos duros, a efectos prácticos del usuario, se traduce en un único almacén mucho más robusto que un disco duro por sí solo.

En esta practica vamos a manejarnos mediante las direcciones IP de dos máquinas, las cuales son:

- Máquina1: 172.16.186.130 Servidor web 1 (Servidor NFS)
- Máquina2: 172.16.186.131 Servidor web 2

## Configuración del RAID por software

Lo primero que tenemos que hacer es añadir dos discos duros a la máquina virtual estando apagada, del mismo tipo y capacidad. Arrancamos la máquina e instalamos el software necesario para configurar el RAID:

        apt-get install mdadm

![Imagen](https://github.com/Andresgp1991/Servidores-web-de-altas-prestaciones/blob/master/Practica6/Imagenes/Figura1.png "Instalación mdadm")


Buscar la información (identificación asignada por Linux) de ambos discos con el siguiente comando:

        fdisk -l

![Imagen](https://github.com/Andresgp1991/Servidores-web-de-altas-prestaciones/blob/master/Practica6/Imagenes/Figura2.png "Comando fdisk")

Ahora ya podemos crear el RAID 1, usando el dispositivo **/dev/md0**, indicando el número de dispositivos a utilizar (2)y su ubicación:

        mdadm -C /dev/md0 --level=raid1 --raid-devices=2 /dev/sdb /dev/sdc

![Imagen](https://github.com/Andresgp1991/Servidores-web-de-altas-prestaciones/blob/master/Practica6/Imagenes/Figura3.png "Creación del disco RAID")

El dispositivo se habrá creado con el nombre **/dev/md0** pero en cuanto reiniciemos la máquina, Linux lo renombrará y pasará a llamarlo **/dev/md127** como vemos en la Figura4.

![Imagen](https://github.com/Andresgp1991/Servidores-web-de-altas-prestaciones/blob/master/Practica6/Imagenes/Figura4.png "/dev/md0 a /dev/md127")

Una vez creado el dispositivo RAID, sin reiniciar la máquina, le daremos formato a /dev/md0 con el comando:

        mkfs /dev/md0

![Imagen](https://github.com/Andresgp1991/Servidores-web-de-altas-prestaciones/blob/master/Practica6/Imagenes/Figura5.png "Formato con mkfs")

Por defecto, mkfs inicializa un dispositivo de almacenamiento con formato ext2. Ahora ya podemos crear el directorio en el que se montará la unidad del RAID y comprobar si se ha montado bien la unidad:

        mkdir /dat
        mount /dev/md0 /dat
        mount

![Imagen](https://github.com/Andresgp1991/Servidores-web-de-altas-prestaciones/blob/master/Practica6/Imagenes/Figura6.png "Dispositivos y particiones del sistema")

Para comprobar el estado del RAID, ejecutaremos la siguiente orden:

        mdadm --detail /dev/md0

![Imagen](https://github.com/Andresgp1991/Servidores-web-de-altas-prestaciones/blob/master/Practica6/Imagenes/Figura7.png "Detalles del disco RAID")

Conviene utilizar el identificador único de cada dispositivo de almacenamiento en lugar de simplemente el nombre del dispositivo en este caso **/dev/md0**, para obtener los UUID de todos los dispositivos de almacenamiento que tenemos, debemos ejecutar la orden:

        ls -l /dev/disk/by-uuid/

![Imagen](https://github.com/Andresgp1991/Servidores-web-de-altas-prestaciones/blob/master/Practica6/Imagenes/Figura8.png "UUID de los discos")

A continuación configuramos el sistema para que monte el dispositivo RAID al arrancar el sistema, para ello editamos el archivo **/etc/fstab** y añadimos la siguiente línea como vemos en la Figura9:

        UUID=ccbbbbcc-dddd-eeee-ffff-aaabbbcccddd /dat ext2 defaults 0 0

![Imagen](https://github.com/Andresgp1991/Servidores-web-de-altas-prestaciones/blob/master/Practica6/Imagenes/Figura9.png "Añadiendo disco a fstab")

Una vez que esté funcionando el dispositivo RAID, podemos simular un fallo en uno de los discos y ver los detalles del dispositivo RAID para ver el fallo:

        sudo mdadm --manage --set-faulty /dev/md0 /dev/sdb

![Imagen](https://github.com/Andresgp1991/Servidores-web-de-altas-prestaciones/blob/master/Practica6/Imagenes/Figura10.png "Provocando fallo en disco /dev/sdb")

Para cambiar el disco, primero tenemos que eliminar el disco que ha fallado y luego añadir un nuevo disco sin necesidad de apagar la máquina:

        mdadm --manage --remove /dev/md0 /dev/sdb
        mdadm --manage --add /dev/md0 /dev/sdb

En todo momento podemos obtener información detallada del estado del RAID y de
los discos que lo componen como vemos en la Figura11.

![Imagen](https://github.com/Andresgp1991/Servidores-web-de-altas-prestaciones/blob/master/Practica6/Imagenes/Figura11.png "Eliminamos disco, añadimos y vemos detalles")

## Configuración de servidor NFS

NFS (Network File System, Sistema de archivos de red) es un sistema de archivos, que permite que en una red Unix/Linux podamos compartir archivos entre todos los equipos que la forman. Los usuarios de la red tendrán la sensación de que los datos a los que acceden están en su propia máquina.

Instalamos los siguiente paquetes:

        apt-get install nfs-kernel-server nfs-common

- nfs-common contiene los programas necesarios para utilizar el servicio NFS tanto en el cliente como en el servidor (lockd, statd, showmount, y nfsstat).

- nfs-kernel-server contiene el soporte necesario en el kernel linux para poder usar el servidor NFS.

Para poder configurar los recursos compartidos (discos duros o carpetas) en el servidor NFS hay que tener permisos de administrador (root) y editar el archivo **/etc/exports**

Cada linea del fichero /etc/exports hace referencia a un recurso compartido y la sintaxis es la siguiente:

        ruta de recurso compartido     hosts clientes     permisos  

- Ruta de recurso compartido  es la ruta local absoluta del recurso que se comparte.

- Hosts clientes  IP del equipo al que le permitimos acceder al recurso compartido. Si tenemos un servidor DNS que nos resuelva los nombres del las máquinas locales podemos usar dichos nombres en vez de la dirección IP.

- Permisos  Controlan el acceso al recurso compartido.             

Algunas opciones de permisos:

- rw/ro exporta el directorio en modo lectura/escritura o sólo lectura.

- root_squash mapea los requerimientos del UID/GID 0 al usuario anónimo (por defecto usuario nobody, con UID/GID 65534); es la opción por defecto.

- no_root_squash no mapea root al usuario anónimo.

- all_squash mapea todos los usuarios al usuario anónimo.

- squash_uids/squash_gids especifica una lista de UIDs o GIDs que se deberían trasladar al usuario anónimo: squash_uids=0-15,20,25-50.

- anonuid/anongid fija el UID/GID del usuario anónimo (por defecto 65534).

- subtree_check/no_subtree_check si se exporta un subdirectorio (no un filesystem completo) el servidor comprueba que el fichero solicitado esté en el árbol de directorios exportado.

- sync modo síncrono: requiere que todas las escrituras se completen antes de continuar; es opción por defecto.

- async modo asíncrono: no requiere que todas las escrituras se completen; más rápido, pero puede provocar pérdida de datos en una caída.

- secure los requerimientos deben provenir de un puerto por debajo de 1024.

- insecure los requerimientos pueden provenir de cualquier puerto.

En nuestro caso añadiremos la siguiente linea como vemos en la Figura12:

        /dev/md127 172.16.186.131(rw,sync,no_subtree_check)

![Imagen](https://github.com/Andresgp1991/Servidores-web-de-altas-prestaciones/blob/master/Practica6/Imagenes/Figura12.png "Archivo /etc/exports")

Para que podamos montar la unidad y utilizar el servidor NFS debemos instalar el cliente NFS con el siguiente comando en el Servidor web 2 en este caso:

        apt-get install nfs-common

Lo siguiente que haremos será crear la carpeta **/dat** en el Servidor web 2 y montar la carpeta **/dat** del Servidor web 1 con los siguientes comandos:

        mkdir /dat
        mount 172.16.186.130:/dat /dat

Una vez realizado, si hacemos un *df -h* veremos que efectivamente la unidad ha sido montada satisfactoriamente, y tenemos acceso a ella a través de **/dat** como vemos en la Figura13:

![Imagen](https://github.com/Andresgp1991/Servidores-web-de-altas-prestaciones/blob/master/Practica6/Imagenes/Figura13.png "Dispositivos montados en el sistema")

Con el comando *mount* también podemos ver que se ha montado correctamente el dispositivo como vemos en la Figura14:

![Imagen](https://github.com/Andresgp1991/Servidores-web-de-altas-prestaciones/blob/master/Practica6/Imagenes/Figura14.png "Dispositivos montados en el sistema con mount")

Ahora tendremos permiso de lectura/escritura desde el Servidor web 2 a la carpeta **/dat** del servidor web 1 (servidor NFS).

Para que esto se mantenga tras el reinicio de la máquina, hemos de incluir la línea correspondiente en el fichero **/etc/fstab**:

        172.16.186.131:/dat /dat nfs rw 0 0

![Imagen](https://github.com/Andresgp1991/Servidores-web-de-altas-prestaciones/blob/master/Practica6/Imagenes/Figura15.png "Archivo fstab")

Para comprobar que todo funciona correctamente, podemos crear un archivo en la carpeta **/dat** del Servidor web 1, y ver como se ha copiado en la carpeta **/dat** del Servidor web 2 y comprobar que podemos leer y escribir ese archivo.