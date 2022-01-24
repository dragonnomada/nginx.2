# Trabajo 12. Integración de PHP con Nginx

## 1. Instalar PHP FPM

	$ sudo apt install php-fpm

## 2. Configurar los permisos de acceso a PHP FPM

> `/etc/php/7.4/fpm/pool.d/www.conf`

```
user = ubuntu
group = ubuntu

...

listen.owner = ubuntu
listen.group = ubuntu
listen.mode = 0660
```

	$ sudo systemctl restart php7.4-fpm

## 3. Crear un archivo de directivas FastCGI

> `/XYZ/conf/php-7.4.conf`

```
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
```

## 4. Integrar el archivo de directivas FastCGI en todas las rutas `.php`

> `/XYZ/sites/server_n.conf`

```
server {

	listen 12345;

	location ~ \.php$ {
		root /XYZ/php;

		include /XYZ/conf/php7.4-fpm.conf;
	}

}

```

## 5. Crear algunos script PHP de prueba

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

## 6. Verficar los archivos `.php`

	$ curl localhost:12345/index.php

	--- <table> ... PHP version 7.4.3 ...

	$ curl localhost:12345/hello.php

	--- Hola PHP :D<h1>Este es el título</h1> ---

	$ curl localhost:12345/hello.php

	--- <h1>Demo</h1> ---
	--- ... ---
	--- <td>Hola</td> ---
	--- <td>8</td> ---
















