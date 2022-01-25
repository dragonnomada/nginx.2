# Sesión 6. Nginx y Apache

* VirtualHosts - Habilitan un puerto para recibir peticiones y guiarlas.
* Reverse Proxy - Redirigir las peticiones y soportar nombre de servidores.
* Load Balancing - Partir el tráfico de peticiones a servidores diferentes.

## 1. Instalación de Apache 2

	$ sudo apt install apache2

	$ sudo systemctl status apache2

## 2. Configuración de Apache 2

### Tabla de Directorios y Archivos Principales

Ubicación | Descripción
--- | ---
**`/etc/apache2/`** | Directorio principal del ambiente apache (gestor de paquetes)
`/etc/apache2/apache2.conf` | Punto de partida para la configuración globlal. (Activa diversas declarativas de tipo `IncludeOptional` similar a la directiva `include` en Nginx, para extender la configuración en otros lugares).
`/etc/apache2/envvars` | Se encuentrar las variables globales usadas en `/etc/apache2/apache2.conf`.
`/etc/apache2/ports.conf` | Activa los puertos globales donde podemos crear servidores virtuales.
**`/etc/apache2/sites-enabled/`** | Aquí se toman los `.conf` disponibles para ser servidores virtuales. (Generalmente son enlaces simbólicos).
**`/var/www/html/`** | Es la carpeta principal propuesta para poner recursos estáticos.

> Cambiar el usuario que ejecuta Apache `/etc/apache2/envvars`

```
export APACHE_RUN_USER=ubuntu
export APACHE_RUN_GROUP=ubuntu
```

> Cambiar los puertos globales `/etc/apache2/ports.conf`

```
# Listen 80
Listen 12000
Listen 13000
```

### Directivas de Apache 2 vs Nginx

Apache | Nginx | Comentarios
--- | --- | ---
`User <user>` | `user <user>;` | Establece el usuario principal del sistema operativo
`Group <group>` | `user <user> <group>;` | Establece el grupo para el usuario principal del sistema operativo.
`LogFormat "<format>" <name>` | `log_format <name> "<format>";` | Establece un formato para los logs. (En apache es un poco más difícil estructurar por los `%*` `%{}`, en Nginx sólo se usan variables `$xxxx`)
`ErrorLog <path>.log` | `error_log <path>.log [<format>] <mode>;` | Establece el archivo de logs de errores al nivel en que se encuentre (a nivel global o a nivel local).
`CustomLog <path>.log` | `access_log <path>.log [<format>] <mode>;` | Establece el archivo de logs de acceso al nivel en que se encuentre (a nivel global o a nivel local).
`<Directory <path> >` | `** Nota 1` | Establecer los permisos del directorio del sistema operativo. (Define si se puede listar, leer, escribir, o se bloquea).
`<FilesMatch "<patten>" >` | `location ~ pattern {` | Crear un bloque para agrupar reglas/directivas especifícas que cumplen un patrón de peticiones.
`IncludeOption <path>` | `include <path>;` | Incluye archivos de configuración a este nivel.
`Listen <port>` | `server { listen <port>;` | Activa un puerto para ser controlado por algún virtual host.
`<VirtualHost <domain>:<port> >` | `server { listen <port>;` | Activa un servidor virtual sobre el puerto especificado (acapara o reserva/integra).
`ServerName <hostname>` | `server_name <hostname>;` | Establece el nombre del servidor virtual para procesar todas las peticiones con ese nombre en ese puerto.
`DocumentRoot <path>` | `root <path>;` | Establece a ese nivel que se usará la carpeta establecida como la principal para sacar los recursos.

## 3. Casos Prácticos en Apache y Nginx

### 1. Montar un Servidor virtual en `Apache`/`Nginx` sobre el puerto 
`43210`/`12345` que tome sus recursos estáticos de la carpeta 
`/XYZ/www/apache`/`XYZ/www/nginx`.

	Nginx

> Crear un archivo de configuración de Nginx para un servidor virtual

	$ mkdir /XYZ/sites-nginx/

> `/XYZ/sites-nginx/server_n.conf`

```
server {
        listen 12345;

        root /ABS/www/nginx;
}
```

> Enlazamos el archivo de configuración a los `sites-enabled`

	$ ln -s /XYZ/sites-nginx/server_n.conf /XYZ/nginx-xyz/conf/sites-enabled/

> Recargar el Nginx

	$ sudo systemctl reload nginx-xyz

> Creamos recursos de muestra

	$ mkdir /XYZ/www/nginx

> `/XYZ/www/nginx/index.html`

```html
<h1>Hola desde Nginx</h1>
```

> `/XYZ/www/nginx/about.html`

```html
<h1>Acerca de Nginx</h1>
``` 

> Probar los recursos de muestra

	$ curl localhost:12345
	--- Hola desde Nginx ---

	$ curl localhost:12345/about.html
	--- Acerca de Nginx ---

---

	Apache

> Crear un archivo de configuración de Apache para un servidor virtual

	$ mkdir /XYZ/sites-apache

> `/XYZ/sites-apache/server_m.conf`

```
Listen 43210

<VirtualHost *:43210>

	DocumentRoot /XYZ/www/apache

	<Directory />
		Options FollowSymLinks
		AllowOverride None
		Require all granted
	</Directory>

</VirtualHost>
```

> Enlazamos el archivo de configuración a los `sites-enabled`

	$ sudo ln -s /XYZ/sites-apache/server_m.conf /etc/apache2/sites-enabled/

> Reiniciar el Apache

	$ sudo systemctl restart apache2

> Creamos recursos de muestra

	$ mkdir /XYZ/www/apache

> `/XYZ/www/apache/index.html`

```html
<h1>Hola desde Apache</h1>
```

> `/XYZ/www/apache/about.html`

```html
<h1>Acerca de Apache</h1>
``` 

> Probar los recursos de muestra

	$ curl localhost:43210
	--- Hola desde Apache ---

	$ curl localhost:43210/about.html
	--- Acerca de Apache ---

---

























