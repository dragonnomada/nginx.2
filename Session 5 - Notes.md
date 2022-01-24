# Sesión 5. Integración de PHP con Nginx

PHP es uno de los motores de plantillas, archivos y recursos en genal
del sistema operativo más utilizado en la última decada. La facilidad
de integración con bases de datos y manejo archivos lo ha popularizado.

Existen dos formas muy comunes de usar PHP, una es directamente
con un procesador llamado FPM (Fastcgi Process Manager) el cuál
nos permite trabajar PHP directamente desde un socket de Unix.
Esto es muy común, cuándo PHP requiere ser administrado de forma
independiente de APACHE, Nginx o algún otro procesador que lo consuma.

Otra forma común de utilizar PHP es através de Apache y que lo
integre y configure directamente.

## 1. Instalar PHP FPM

	$ sudo apt install php-fpm

	$ sudo apt install php<7.2>-fpm

	$ sudo systemctl status php<7.4>-fpm.service

	$ ps ax | grep php

### Carpetas y archivos principales de PHP

Ubicación | Descripción
--- | ---
`/run/php/php7.4-fpm.sock` | Instancia física del PHP 7.4 para comunicación de socket de unix
`/run/php/php7.4-fpm.pid` | Archivo PID que contiene el id del proceso principal maestro
`/etc/php/7.4/fpm/php-fpm.conf` | Archivo de configuración principal, delegada a `/etc/php/7.4/fpm/pool.d/www.conf`
`/etc/php/7.4/fpm/php.ini` | Archivo de configuración general (MySQL, ODBC, EMAIL, ...).
`/etc/php/7.4/fpm/pool.d/www.conf` | Archivo de configuración específica orientado a los permisos.

## 2. Configurar los permisos de acceso a PHP FPM

> Habilitar los permisos para el usuario que podrá hacer peticiones vía POSIX/UNIX SOCKET hacía PHP FPM `/etc/php/7.4/fpm/pool.d/www.conf`

```
; Unix user/group of processes
; Note: The user is mandatory. If the group is not set, the default user's group
;       will be used.
user = ubuntu
group = ubuntu

; Set permissions for unix socket, if one is used. In Linux, read/write
; permissions must be set in order to allow connections from a web server. Many
; BSD-derived systems allow connections regardless of permissions.
; Default Values: user and group are set as the running user
;                 mode is set to 0660
listen.owner = ubuntu
listen.group = ubuntu
listen.mode = 0660
```

* **NOTA:** Cada que se actualiza la configuración de PHP, debemos
reiniciar el sistema con `sudo systemctl restart php7.4-fpm`

## 3. Integrar PHP con Nginx

FastCGI (Fast Common Gateway Interface) es una interfaz común de
transferencia de peticiones (del Gateway) y en la versión Fast es capaz
de recibir parámetros externos para complementar la petición.

Este protocolo es utilizado por Nginx, para especificar que
nuestras peticiones a recursos de tipo PHP sean mandas al FPM (FastCGI Process Manager).
Las peticiones irán desde Nginx hacía PHP para procesar los recursos
(los scripts PHP) y devolverle al usuario el resultado.

Por ejemplo, podemos pensar en un archivo de PHP que nos dé
como resultado un HTML con formato o cosas generadas del lado del procesador PHP.
Podríamos crear archivos que consulten bases de datos, procesen archivos
o hagan peticiones a otros servidores y el resultado mandarlo al cliente.

Para poder procesar archivos PHP, usaremos las peticiones de tipo
FastCGI y configuraremos las rutas mediante las directivas `fastcgi_...`.
Principalmente la directa `fastcig_pass` y `fastcgi_param`.

### Tabla de Directivas FastCGI

Directiva | Descripción
--- | ---
`fastcgi_split_path_info <patrón | formato>` | Especifica la ruta hacía el recurso `.php` solicitado. Defecto `^(.+?\.php)(/.*)$`.
`fastcgi_pass <url | socket>` | Apunta el paso de la petición hacía la url del servidor o el socket de unix (en este caso). Similar a la directiva `proxy_pass`.
`fastcgi_param <PARAM> <VALUE>` | Especifica el valor del parámetro que recibirá FastCGI. El más importante es `fastcgi_param SCRIPT_FILENAME   $request_filename;` que especifica la ruta hacía el archivo de PHP que será procesado.

### Configurar una ruta para procesar los archivos `.php`

Al definir un servidor en Nginx, debemos decirle a las rutas
que integren PHP, todos los parámetros de FastCGI para que
funcione correctamente.

Se puede especificar una ruta genérica `location ~ \.php$` 
(Todas las rutas que terminan con `.php`). Para decirle a Nginx
que esas rutas se las envíe a PHP FPM mediante el UNIX SOCK por
FastCGI (`fastcgi_pass unix:/run/php/php7.4-fpm.sock;`).

Y vamos especificar algunas generalidades como los parámetros de FastCGI
para que PHP FPM procese los scripts correctamente.

> `/XYZ/sites/server_n.conf`

```
server {

	listen 12345;

	location ~ \.php$ {
		root /XYZ/php;

		fastcgi_split_path_info ^(.+?\.php)(/.*)$;
		fastcgi_pass unix:/run/php/php7.4-fpm.sock;
		fastcgi_index index.php;
  
		fastcgi_param SCRIPT_FILENAME   $request_filename;
		fastcgi_param PATH_INFO         $fastcgi_path_info;
		fastcgi_param PATH_TRANSLATED   $document_root$fastcgi_path_info;
		fastcgi_param QUERY_STRING      $query_string;
		fastcgi_param REQUEST_METHOD    $request_method;
		fastcgi_param CONTENT_TYPE      $content_type;
		fastcgi_param CONTENT_LENGTH    $content_length;
		fastcgi_param SCRIPT_NAME       $fastcgi_script_name;
		fastcgi_param REQUEST_URI       $request_uri;
		fastcgi_param DOCUMENT_URI      $document_uri;
		fastcgi_param DOCUMENT_ROOT     $document_root;
		fastcgi_param SERVER_PROTOCOL   $server_protocol;
		fastcgi_param REQUEST_SCHEME    $scheme;
		fastcgi_param HTTPS             $https if_not_empty;
		fastcgi_param HTTP_PROXY        "";
		fastcgi_param GATEWAY_INTERFACE CGI/1.1;
		fastcgi_param SERVER_SOFTWARE   nginx/$nginx_version;
		fastcgi_param REMOTE_ADDR       $remote_addr;
		fastcgi_param REMOTE_PORT       $remote_port;
		fastcgi_param SERVER_ADDR       $server_addr;
		fastcgi_param SERVER_PORT       $server_port;
		fastcgi_param SERVER_NAME       $server_name;
		fastcgi_param REDIRECT_STATUS   200;

		fastcgi_connect_timeout 60;
		fastcgi_send_timeout 180;
		fastcgi_read_timeout 180;
		fastcgi_buffer_size 512k;
		fastcgi_buffers 512 16k;
		fastcgi_busy_buffers_size 1m;
		fastcgi_temp_file_write_size 4m;
		fastcgi_max_temp_file_size 4m;
		fastcgi_intercept_errors off;

	}

}
```

Cada que queramos integrar al servidor de Nginx PHP, este deberá
pasarle toda la ejecución de PHP al FPM mediante FastCGI, sin embargo,
esa configuración es constante y siempre la misma salvo la directiva
`root` que especifica la carpeta de dónde se sacarán los archivos
`.php`. Es decir, cada directiva `location` que quiera PHP deberá
configurar los parámetros de FastCGI.

Esto podría ser molesto, por lo que podemos concentrar todos esos
parámetros constantes, dentro de un archivo `.conf` para incluirlo
en las rutas o la ruta genérica que procese PHP.

Entonces, podemos crear un archivo llamado `php-fpm.conf` para
incluirlo fácilemente en cualquier servidor medianter la directiva
`include <path>/php-fpm.conf`. Automáticamente Nginx cargará
todas la reglas/directivas del archivo, sobre la inclusión.

> `/XYZ/conf/php7.4-fpm.conf`

```
fastcgi_split_path_info ^(.+?\.php)(/.*)$;
fastcgi_pass unix:/run/php/php7.4-fpm.sock;
fastcgi_index index.php;
  
fastcgi_param SCRIPT_FILENAME   $request_filename;
...
```

> `/XYZ/sites/server_n.conf`

```
server {

	listen 12345;

	# Procesa todas las rutas que finalicen en `.php`
	# Ejemplo: /wp-content/index.php -> /XYZ/php/wp-content/index.php
	location ~ \.php$ {
		root /XYZ/php; # $document_root

		# Procesador PHP (mediante FastCGI)
		include /XYZ/conf/php7.4-fpm.conf;
	}

	# Procesa la ruta /products.php
	# Ejemplo: /products.php -> /XYZ/store/products.php
	location /products.php {
		root /XYZ/store; # $document_root	

		# Procesador PHP (mediante FastCGI)
		include /XYZ/conf/php7.4-fpm.conf;
	}

}

```

## 4. Crear algunos script PHP de prueba

> `/XYZ/php/index.php`

```php
<?php

	phpinfo();

?>
```

> `/XYZ/php/hello.php`

```php
<?php

	echo "Hola PHP :D";

?>
<h1>Este es el título</h1>
```

> `/XYZ/php/demo.php`

```php
<h1>Demo</h1>

<table>
	<tbody>
		<?php for ($i = 0; $i < 10; $i++) { ?>
			<tr>
				<td>Hola</td>
				<td><?= $i + 1 ?></td>
			</tr>
		<?php } ?>
	</tbody>
</table>
```

> `/XYZ/store/products.php`

```php
<h1>Productos</h1>

<?php

	// TODO: Obtener los productos de MYSQL y pintarlos

	echo "En mantenimiento";

?>
```



















