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

	$ apachectl configtest	

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

# Sesión 6. Comunicación Proxy entre Nginx y Otros Servidores

Nginx tiene la capacidad de redigir las peticiones hacia otros 
servidores en otros puertos o en otros dominios.

Mediante la directiva `proxy_pass` podemos redigir una petición,
enviando incluso Headers personalizados.

> Sintaxis de la directiva `proxy_pass`

```
location ... {
	proxy_pass <url>;
}
```

Por ejemplo, nosotros tenemos un servidor de Nginx
usando el puerto `12345`. Y queremos que las peticiones
se las envie hacia el puerto `54321` (De Nginx a Apache).

> `/XYZ/sites/server_n.conf`

```
server {

	listen 12345;

	location / {
		proxy_pass http://localhost:54321;
	}

}
```

Cuándo tenemos múltiples servidores podemos enlazarlos en un mismo
servidor de Nginx en ubicaciones/rutas distintas, por ejemplo,
para poder administrar proyectos independientes sin mayor complejidad
o hacer versionamientos.

> `/XYZ/sites/server_n.conf`

```
server {

	listen 12345;

	location /project1 {
		proxy_pass http://localhost:54321/;
	}

	location /project2 {
		proxy_pass http://localhost:43210/;
	}

	location /project3 {
		proxy_pass http://localhost:32109/;
	}

	location /projectX {
		proxy_pass http://IP:PORT/;
	}

	location /projectY {
		proxy_pass http://DOMAIN/;
	}

	location /projectZ {
		proxy_pass http://<url>/v1.2/;
	}

}
```

El mayor que se le da la inversión del proxy, es cuándo
queremos colocar múltiples servidores atendiendo el puerto 80.
Este puerto es el puerto utilizado por defecto en los navegadores
y uno de los más solicitados.

Entonces todos los servidores idealmente quisieran estar bajo el puerto
80. Sin embargo, sólo puede ir uno.

Nginx provee en la capacidad del Reverse Proxy, autodirigir un mismo
puerto con diferentes nombres de servidor, usando la directiva
`server_name` y registrando los host en `/atc/hosts`.

Es característica se considera *Dominios Virtuales*.

Entonces podemos administrar un mismo puerto en múltiples
dominios y subdominios virtuales.

> Sintaxis de la declarativa `server_name`

```
server_name <domain1> <domain1> ...;
```

Al definir el Server Name, el servidor ahora podrá
procesar las peticiones a través del dominio virtual.

> Ejemplo

```
server {

	listen 80;
	server_name abs.com;

	... (abs)

}
```

	$ curl abs.com

	--- ... (abs) ---

> Ejemplo

```
server {

	listen 80;
	server_name jvs.com;

	... (jvs)

}
```

	$ curl jvs.com

	--- ... (jvs) ---


* **NOTA:** Esto sólo funciona dentro del mismo ambiente de Nginx.
Es decir, dos ambientes distintos no pueden usar el mismo puerto.

### Práctica

1. Crear un servidor sobre el puerto 12345 usando 3 dominios distintos
2. Apuntar cada dominio a otro puerto con `proxy_pass`

* **NOTA:** Registrar los dominios en `/etc/hosts`



















