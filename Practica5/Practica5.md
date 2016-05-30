# Replicación de bases de datos MySQL

En esta practica vamos a manejarnos mediante las direcciones IP de dos máquinas, las cuales son:

- Máquina1: 172.16.186.130 Servidor web 1 (Esclavo)
- Máquina2: 172.16.186.131 Servidor web 2 (Maestro)

## Crear una BD e insertar datos

Deberemos crear una base de datos en MySQL e insertar algunos datos para llevar a cabo la práctica. Para crear la base de datos utilizaremos la interfaz de línea de comandos de MySQL.

Lo primero que tenemos que hacer es acceder a MySQL desde el terminal con el siguiente comando:

**mysql -u root -p**

		root@Ubuntu-Server-1:/# mysql -u root -p
		Enter password: 
		Welcome to the MySQL monitor.  Commands end with ; or \g.
		Your MySQL connection id is 38
		Server version: 5.5.47-0ubuntu0.14.04.1 (Ubuntu)

		Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

		Oracle is a registered trademark of Oracle Corporation and/or its
		affiliates. Other names may be trademarks of their respective
		owners.

		Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

A continuación crearemos la base de datos, llamada *Usuarios*:

**create database Usuarios;**

		mysql> create database Usuarios;
		Query OK, 1 row affected (0.03 sec)

Cambiamos a la base de datos creada para poder modificarla con **Use Usuarios** y le indicamos que nos muestre las tablas **show tables**, la cual no tiene ninguna creada:

		mysql> use Usuarios;
		Database changed

		mysql> show tables;
		Empty set (0.00 sec)

Crearemos una tabla en la base de datos, la cual se va a llamar *datos* y va a contener el nombre y teléfono de los usuarios, para ello usamos la siguiente orden:

**create table datos(nombre varchar(50), telefono int);**

		mysql> create table datos(nombre varchar(50), telefono int);
		Query OK, 0 rows affected (0.10 sec)

		mysql> show tables;
		+--------------------+
		| Tables_in_Usuarios |
		+--------------------+
		| datos              |
		+--------------------+
		1 row in set (0.01 sec)

Para poder hacer consultas y ver que se han clonado las bases de datos correctamente mas adelante, insertaremos datos a la tabla con la siguiente orden:

**insert into datos(nombre,telefono) values("andres",683847584);**

		mysql> insert into datos(nombre,telefono) values("andres",683847584);
		Query OK, 1 row affected (0.05 sec)

Para hacer una consulta a la base de datos, por ejemplo, mostrar todas las tuplas de la tabla *datos*, lo haremos de la siguiente forma (tenemos que estar dentro de la base de datos a la que queremos hacer la consulta:

**select * from datos;**

		mysql> select * from datos;
		+--------+-----------+
		| nombre | telefono  |
		+--------+-----------+
		| andres | 683847584 |
		+--------+-----------+
		1 row in set (0.02 sec)

Para ver de que datos se compone una tabla, de que tipo son... Utilizaremos la orden:

**describe datos**

En este caso *datos* seria el nombre de la tabla que queremos ver su información.

		mysql> describe datos;
		+----------+-------------+------+-----+---------+-------+
		| Field    | Type        | Null | Key | Default | Extra |
		+----------+-------------+------+-----+---------+-------+
		| nombre   | varchar(50) | YES  |     | NULL    |       |
		| telefono | int(11)     | YES  |     | NULL    |       |
		+----------+-------------+------+-----+---------+-------+
		2 rows in set (0.08 sec)

## Replicar una BD MySQL con mysqldump

La herramienta mysqldum nos la ofrece MySQL para clonar las BD que tenemos en nuestra máquina.

Esta herramienta soporta una cantidad considerable de opciones, en la siguiente URL se explican con detalle todas las opciones posibles:

[http://dev.mysql.com/doc/refman/5.0/es/mysqldump.html](http://dev.mysql.com/doc/refman/5.0/es/mysqldump.html)

Primero tenemos que tener en cuenta que los datos pueden estar actualizándose constantemente en el servidor de base de datos principal, en este caso, antes de hacer la copia de seguridad debemos evitar que se acceda a la base de datos para cambiar nada, para ello entramos en MySQL y utilizamos la siguiente orden:

**mysql -u root -p**

**flush tables with read lock;**

		mysql> flush tables with read lock;
		Query OK, 1 row affected (0.02 sec)

Ahora podemos hacer mysqldump para guardar la copia de la base de datos. En el servidor web 1:

**mysqldump -u root -p Usuarios > ~andres/Desktop/Usuarios.sql**

Como habíamos bloqueado las tablas, debemos desbloquearlas:

**mysql -u root -p**

**unlock tables;**

		mysql> unlock tables;;
		Query OK, 1 row affected (0.00 sec)

Ahora nos vamos al servidow web 2 y copiaremos el archivo creado en el servidoe web 1 para posteriormente importar la base de datos:

**scp 172.16.186.130:/root/Usuarios.sql /root/**

La orden mysqldump no incluye en ese archivo la sentencia para crear la base de datos, es necesario que nosotros la creemos en un primer paso, antes de restaurar las tablas de esa base de datos:

**mysql -u root -p**

**create database Usuarios;**

		mysql> create database Usuarios;
		Query OK, 1 row affected (0.03 sec)

Ya podemos importar la base de datos completa en el MySQL. Para ello utilizamos la orden:

**mysql -u root -p Usuarios < /root/Usuarios.sql**

		-- MySQL dump 10.13  Distrib 5.5.47, for debian-linux-gnu (x86_64)
		--
		-- Host: localhost    Database: Usuarios
		-- ------------------------------------------------------
		-- Server version	5.5.47-0ubuntu0.14.04.1

		/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
		/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
		/*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
		/*!40101 SET NAMES utf8 */;
		/*!40103 SET @OLD_TIME_ZONE=@@TIME_ZONE */;
		/*!40103 SET TIME_ZONE='+00:00' */;
		/*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
		/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
		/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
		/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;
		/*!40103 SET TIME_ZONE=@OLD_TIME_ZONE */;

		/*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
		/*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
		/*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
		/*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
		/*!40101 SET CHARACTER_SET_RESULTS=@OLD_CHARACTER_SET_RESULTS */;
		/*!40101 SET COLLATION_CONNECTION=@OLD_COLLATION_CONNECTION */;
		/*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;

		-- Dump completed on 2016-05-20 21:47:09

## Replicación de BD mediante una configuración maestro-esclavo

MySQL tiene la opción de configurar el demonio para hacer replicación de las bases de datos sobre un esclavo a partir de los datos que almacena el maestro.

Se trata de un proceso automático que resulta muy adecuado en un entorno de
producción real. Implica realizar algunas configuraciones, tanto en el servidor principal como en el secundario.

Lo primero que debemos hacer es la configuración de MySQL del maestro. Para ello editamos **/etc/mysql/my.cnf**:

Comentamos el parámetro bind-address que sirve para que escuche a un servidor:

**#bind-address 127.0.0.1**

Le indicamos el archivo donde almacenar el log de errores. De esta forma, si por ejemplo al reiniciar el servicio cometemos algún error en el archivo de configuración, en el archivo de log nos mostrará con detalle lo sucedido:

**log_error = /var/log/mysql/error.log**

Establecemos el identificador del servidor:

**server-id = 1**

El registro binario contiene toda la información que está disponible en el registro de actualizaciones, en un formato más eficiente y de una manera que es segura para las transacciones:

**log_bin = /var/log/mysql/mysql-bin.log**

Guardamos el documento y reiniciamos el servicio:

**service mysql restart**

Si no nos ha dado ningún error la configuración del maestro, pasamos a hacer la configuración del esclavo, es igual a la del maestro, con la diferencia de que el *server-id* será 2. Una vez hechas las configuraciones Podemos volver al maestro para crear un usuario y darle permisos de acceso para la replicación. Entramos en MySQL y ejecutamos las siguientes sentencias:

**create user esclavo identified by 'esclavo';**

**grant replication slave on \*.\* to 'esclavo'@'%' identified by 'esclavo';**

**flush privileges;**

**flush tables;**

**flush tables with read lock**

		mysql> create user esclavo identified by 'esclavo';
		Query OK, 0 row affected (0.00 sec)
		mysql> grant replication slave on *.* to 'esclavo'@'%' identified by 'esclavo';
		Query OK, 0 row affected (0.00 sec)
		mysql> flush privileges;
		Query OK, 0 row affected (0.00 sec)
		mysql> flush tables;
		Query OK, 0 row affected (0.01 sec)
		mysql> flush tables with read lock
		Query OK, 0 row affected (0.00 sec)


Para finalizar, obtenemos los datos de la base de datos que vamos a replicar para posteriormente usarlos en la configuración del esclavo:

**show master status;**

		mysql> show master status;
		+------------------+----------+--------------+------------------+
		| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
		+------------------+----------+--------------+------------------+
		| mysql-bin.000001 |      501 |              |                  |
		+------------------+----------+--------------+------------------+
		1 row in set (0.00 sec)

![Imagen](https://github.com/Andresgp1991/Servidores-web-de-altas-prestaciones/blob/master/Practica5/Imagenes/Figura1.png)

Volvemos a la máquina esclava, entramos en MySQL y le damos los datos del maestro. En MySQL ejecutamos la siguiente sentencia:

**change master to master_host='172.16.186.130',master_user='esclavo',master_password='esclavo',master_log_file='mysql-bin.000001',master_log_pos=501,master_port=3306;**

		mysql> change master to master_host='172.16.186.130', master_user='esclavo', master_password='esclavo', master_log_file='mysql-bin.000001', master_log_pos=501, master_port=3306;
		Query OK, 0 row affected (0.07 sec)


Arrancamos el esclavo:

**start slave;**

		mysql> start slave;
		Query OK, 0 row affected (0.00 sec)

Volvemos al maestro y activamos las tablas para que puedan meterse nuevos datos en el maestro:

**unlock tables;**

		mysql> unlock tables;
		Query OK, 0 row affected (0.01 sec)

Si queremos asegurarnos de que todo funciona perfectamente y que el esclavo no tiene ningún problema para replicar la información, nos vamos al esclavo y con la siguiente orden:

**show slave status\G**

		mysql> show slave status\G
	*************************** 1. row ***************************
	               Slave_IO_State: Waiting for master to send event
	                  Master_Host: 172.16.186.131
	                  Master_User: esclavo
	                  Master_Port: 3306
	                Connect_Retry: 60
	              Master_Log_File: mysql-bin.000002
	          Read_Master_Log_Pos: 1088
	               Relay_Log_File: mysqld-relay-bin.000003
	                Relay_Log_Pos: 1234
	        Relay_Master_Log_File: mysql-bin.000002
	             Slave_IO_Running: Yes
	            Slave_SQL_Running: Yes
	              Replicate_Do_DB: 
	          Replicate_Ignore_DB: 
	           Replicate_Do_Table: 
	       Replicate_Ignore_Table: 
	      Replicate_Wild_Do_Table: 
	  Replicate_Wild_Ignore_Table: 
	                   Last_Errno: 0
	                   Last_Error: 
	                 Skip_Counter: 0
	          Exec_Master_Log_Pos: 1088
	              Relay_Log_Space: 1537
	              Until_Condition: None
	               Until_Log_File: 
	                Until_Log_Pos: 0
	           Master_SSL_Allowed: No
	           Master_SSL_CA_File: 
	           Master_SSL_CA_Path: 
	              Master_SSL_Cert: 
	            Master_SSL_Cipher: 
	               Master_SSL_Key: 
	        Seconds_Behind_Master: 0
	Master_SSL_Verify_Server_Cert: No
	                Last_IO_Errno: 0
	                Last_IO_Error: 
	               Last_SQL_Errno: 0
	               Last_SQL_Error: 
	  Replicate_Ignore_Server_Ids: 
	             Master_Server_Id: 1
	1 row in set (0.00 sec)

Revisamos si el valor de la variable “Seconds_Behind_Master” es distinto de “null”. En ese caso es 0, por lo que no hay ningún error y todo funciona perfectamente. Y ya está todo listo para que los demonios de MySQL de las dos máquinas repliquen automáticamente los datos que se introduzcan/modifiquen/borren en el servidor maestro.

Para comprobar que todo funciona, nos vamos al maestro e introducimos nuevos datos:

**insert into datos(nombre,telefono) values("Alvaro",627373643);**

		mysql> insert into datos(nombre,telefono) values("Alvaro",627373643);
		Query OK, 1 row affected (0.05 sec)

![Imagen](https://github.com/Andresgp1991/Servidores-web-de-altas-prestaciones/blob/master/Practica5/Imagenes/Figura2.png)
