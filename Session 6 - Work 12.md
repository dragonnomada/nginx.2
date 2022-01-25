# Trabajo 13. Configurar un servidor de Apache

## 1. Crear una carpeta dónde colocar los archivos de configuración de Apache

	$ mkir /XYZ/apache-sites

## 2. Crear un servidor virtual sobre una carpeta de recursos

> `/XYZ/apache-sites/server_m.conf`

```
Listen 54321

<VirtualHost *:54321>

	DocumentRoot /XYZ/www/apache

	<Directory /XYZ/www/apache>
		# Seguir enlaces simbólicos
		Options FollowSymLinks
		# Escritura
		AllowOverride None
		# Lectura
		Require all granted
	</Directory>

</VirtualHost>
```

## 3. Enlazar la configuración hacía el `sites-enabled`

	$ sudo ln -s /XYZ/apache-sites/server_m.conf /etc/apache2/sites-enabled

## 4. Reiniciar el servidor

	$ apachectl configtest

	$ sudo systemctl restart apache2

## 5. Colocar algunos archivos de muestra

	$ mkdir -p /XYZ/www/apache

	$ echo "Hola Apache XYZ" > /XYZ/www/apache/index.html

	$ echo "Acerca de Apache XYZ" > /XYZ/www/apache/about.html

## 6. Probar los recursos de muestra

	$ curl localhost:43210
	--- Hola desde Apache ---

	$ curl localhost:43210/about.html
	--- Acerca de Apache ---



