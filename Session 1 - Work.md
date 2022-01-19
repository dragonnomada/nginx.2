# Trabajo 1. Configuraciones separadas

## Introducción

En este trabajo realizaremos una implementación práctica de diversas configuraciones separadas, para generar servidores virtuales.

Recuerda no salirte de los puertos que te fueron asignados.

## Paso 1. Crea al menos una configuración para un servidor virtual

> 1. Definimos los archivos de configuración de cada servidor virtual. [Definición]

	$ nano XXX/sites/xxx_server_1.conf

	$ nano YYY/sites/yyy_server_1.conf

	$ nano ZZZ/sites/zzz_server_1.conf

## Paso 2. Enlaza la configuración del servidor virtual a `sites-enabled`

> 2. Enlazamos los archivos de configuración a los sitios habilitados por la configuración por defecto de nginx (gestor de paquetes) ubicados en `/etc/nginx/sites-enabled/*`. [Activación]

	$ sudo ln -s /XXX/sites/xxx_server_1.conf /etc/nginx/sites-enabled/

	$ sudo ln -s /YYY/sites/yyy_server_1.conf /etc/nginx/sites-enabled/

	$ sudo ln -s /ZZZ/sites/zzz_server_1.conf /etc/nginx/sites-enabled/

### Paso 3. Comprueba que la configuración sea correcta

> 3. Comprobar que la configuración sea correcta. [Verificación]

	$ sudo /usr/sbin/nginx -t
	
	... ok ... successful

### Paso 4. Recarga Nginx mediante la *señal reload*

> 4. Integrar los cambios mediante la recarga de Nginx. [Integración]

	$ sudo /usr/sbin/nginx -s reload

### Paso 5. Consulta tu servidor virtual

> 5. Comprueba que el servidor virtual esté funcionando

	$ curl localhost:12345

	<h1>500 Internal Server Error</h1>


