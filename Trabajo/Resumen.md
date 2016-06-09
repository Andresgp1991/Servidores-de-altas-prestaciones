# Configuraciones básicas de seguridad para Nginx y Apache

En este documento vamos a configurar distintos tipos de seguridad que podemos aplicar a los servidores web nginx y apache. Iremos paso a paso en cada configuración, comenzando primero con la configuración del servidor web nginx y posteriormente con la configuración del servidor web apache.

Para Nginx hemos decidido enfocarnos en el control de acceso de HTTP, ocultar la firma y versión del servidor, establecer límites para el Buffer y el tiempo de espera, limitar el número de conexiones por Ip, configurar la seguridad para PHP y, crear y configurar el certifcado SSL.

Para Apache también hemos ocultado la firma y la versión del servidor, y a parte, hemos deshabilitado el HTTP Trace, configurado el ModSecurity de Apache y hecho una prueba para defendernos de ataques de SQL Injection.

Realizado por:

Andrés Guerrero Pinteño
José Carlos Baena Ariza
