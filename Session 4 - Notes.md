## Sesión 4. Servidores como Recursos Dinámicos

Nginx provee un sistema de ubicaciones (locations) basados en
rutas fijas y rutas dinámicas capturadas a través de expresiones
regurales comunes al estándar Perl (PCRE).

Esto significa que podemos pensar en diseñar rutas que atrapen
recursos dinámicos. Con el fin de proveer recursos más sofisticados,
por ejemplo, crear listas de recursos, o cosa útiles.

## 1. Crear una lista de recusos

Nginx provee la directiva `autoindex` la cuál permite habilitar
o deshabilitar la capacidad de que una ruta autoindexe la raíz de
recursos. Por ejemplo, en un location que tenga especificado
un directorio con la directiva `root`, podrá autoindexarse.

> Sintaxis de la declarativa `autoindex`

```
# Crea una lista de los recursos disponibles en la ruta
autoindex <on|off>;

# Mostraría el tamaño exacto de los recursos
autoindex_exact_size <on|off>;

# Muestra sólo cierto tipo de archivos
autoindex_format <type1> | <type2> | ...;

# Cambia la hora para que no sea la del servidor, sino local al cliente
autoindex_locatime <on|off>
```

[http://nginx.org/en/docs/http/ngx_http_autoindex_module.html](http://nginx.org/en/docs/http/ngx_http_autoindex_module.html)

> Ejemplo

```
location /download {
	autoindex on;
	root /XYZ/download;
}
```

* **NOTA:** Para activar el servidor debemos crear el enlace dinámico
a la carpeta `sites-enabled`: 
`ln -s /ABS/sites/server_10003.conf /ABS/nginx-abs/conf/sites-enabled/`

* **NOTA:** Simpre que modifiquen un servidor se debe aplicar la señal
de reload `sudo systemctl reload nginx-xyz.service` o 
`/XYZ/nginx-xyz/sbin/nginx -s reload`

### Podemos deshabilitar el autoindexado en subcarpetas

Para no exponer archivos de un subdirectorio autoindexado
podemos desactivar el autoindexado en la subruta con `autoindex off`.

> Ejemplo

```
location /download {
	autoindex on;
	root /XYZ/download;
}

location /download/hidden {
	autoindex off;
}
```

### Podemos denegar el acceso a IPs o Dominios

Podríamos crear un whitelist de IPs o Dominios permitidos y a los demás
negarles el acceso a una subruta. También se podría diseñar lo contrario
un blacklist.

```
location /download {
	autoindex on;
	root /XYZ/download;
}

location /download/private {
	allow 189.123.456.789/24;
	deny all;
}
```

---

# Sesión 4. Server Side Includes (Módulo SSI)

Nginx provee un módulo llamado `SSI` (Server-Side Include / 
Inclusiones del lado del Servidor) que tiene el objetivo
de preprocesar archivos `.shtml` para que incluyan o ejecuten
comandos específicos del lado del servidor. Estos comandos
pueden ser la impresión de variables de Nginx, la inclusión
de archivos o la inclusión de respuestas de otros servidores.

Con esto podemos crear recursos dinámicos, como sitios web o
sitios de recursos dinámicos.

Los archivos `SHTML` son archivos HTML capaces de procesar
comentarios HTML estructurados de la forma:

	<!--#command <params> -->

La diferencia a un archivo tradicional `.html` es sólo la 
extensión (`.shtml`) para recordar que se ejecutarán
inclusiones o comandos del lado del servidor.

Existe una directiva llamada `ssi` que nos permite habilitar
o deshabilitar el procesamiento de los archivo `.shtml`
para que se construyan mediante el server-side include.

> Sintaxis de la declarativa `ssi`

```
ssi <on|off>;
```

[http://nginx.org/en/docs/http/ngx_http_ssi_module.html](http://nginx.org/en/docs/http/ngx_http_ssi_module.html)

> Ejemplo

```
location / {
	ssi on;
	root /ABS/www/ssi-example;
	index index.shtml;
}

# / -> /index.shtml; -> SSI (/ABS/www/ssi-example/index.shtml)
```

El módulo SSI nos permite acceder a las variables de Nginx,
tanto las del servidor, cómo las personalizadas mediante `map`.
También es posible acceder a las variables de petición del cliente
cómo los query params (`...?foo=bar&page=3`), también a los headers
y variables cómo la ruta solitada.

### Tabla de Variables de Nginx

Variable | Descripción
--- | ---
`$request_uri` | Variable original de la cadena de petición del cliente (`/products/123?filters=name,price&page=3`)
`$uri` | Equivalente a `$document_uri` contiene el último uri accedido.
`$document_root` | La raíz de acceso a los archivos asociados a la petición (declativa `root` o `alias`)
`$remote_addr` | Contiene la IP del cliente.
`$http_host` | El host del cliente
`$http_<header>` | Podemos acceder a los headers de petición (desde el cliente).
`$http_send_<header>` | Podemos acceder a los headers de respuesta (hacia el cliente).
`$args` | Contiene la cadena del query (`?foo=bar&page=5 -> $args="foo=bar&page=5"`)
`$arg_<name>` | Contiene el valor del parámetro query (`?foo=bar&page=5 -> $arg_foo="bar"`)

* **NOTA:** Los headers generalmente mezclan mayúsculas, minúsculas y guiones medios
pero hay una convención en Nginx para las variables. Todos los headers
serán en minúsculas y se sustituirá el guión medio (`-`) por guión bajo
(`_`) al construir la variable. Ejemplo `-H Content-Type` -> `$http_content_type`.
Por ejemplo, podemos recuperar headers del cliente personalizados `-H X-Token`
generaría la variable `$http_x_token`.

### Tabla de comandos de inclusión

Comandos | Descripción
--- | ---
`#echo var="<name>"` | Imprime el valor de una variable (sustituye la etiqueta por el continido de la varible).
`#include file="<path>"` | Incluye el contenido de un archivo directamente (sustituye la etiqueta por el contenido de otro archivo).
`#include virtual="<url>"` | Incluye el contenido de la respuesta de la url (sustituye la etiqueta por el contenido de la respuesta a la url).
`#set var="<name>" value="<value>"` | Crea una variable con un valor especificado.

### Generar un archivo SHTML (ssi file)

> Ejemplo de un `index.shtml`

```html
<h1>Bienvenido</h1>

<p>URI: <!--#echo var="request_uri" ---></p>

<p>QUERY: <!--#echo var="args" ---></p>

<p>QUERY FOO: <!--#echo var="foo" ---></p>

<p>QUERY PAGE: <!--#echo var="page" default="0" ---></p>
```

---

# Sesión 4. Reescribir rutas y Expresiones Regulares

Es común querer reescribir una ruta por otra en los siguientes casos
particulares.

1. **La ruta tiene un formato más sencillo que el objetivo**. Ejemplo 
`/products/123/info`, pero la real `/products.php?id=123&mode=info`.
2. **La ruta está internacionalizada**. Ejemplo 
`/en/about` -> `/about.php?lang=en`, `/es/acerca` -> `/about.php?lang=es`.
3. **La ruta consume recursos internos o privados**. Ejemplo
`/videos/project/xyz/123456` -> `/v/p/xyz--123456.mp4` (interna)

En Nginx contamos con la directiva `rewrite` que nos permitirá
reescribir rutas en otras. Con la ventaja de utilizar las
expresiones regurales para extraer datos de la ruta. Por ejemplo,
extraer el nombre del proyecto y el código de vídeo (en el último
caso particular).

Nginx utiliza un sistema de expresiones regulares compatibles con Perl
(PCRE - Perl Compatible Regular Expressions) [https://www.pcre.org/](https://www.pcre.org/)

### Tabla de Expresiones Regulares

Token | Descripción
--- | ---
`/about` | Coincide todas las rutas que contienen con `/about`. `/en</about>/...`
`hello` | Coincide todos los caracteres `h e l l o` en ese esa secuencia con la url. `/utils/<hello>/...`
`^` | **Comienzo**. `^/about`, sólo coinciden las rutas que comienzan con `/about`
`$` | **Finaliza**. `html$`, sólo coincide sí la ruta termina en html, `/en/about/privacy.html`
`()` | **Captura**. `/products/(.*)/info`, captura la parte encerrada que coincida.
`.*` | **Todos los símbolos**. `.*hello`, todos los caracteres antes de `hello`.

### Reescritura de Rutas

> Sintaxis de la declativa `rewrite`

```
# Sustituye la ruta por el código HTTP (200 OK | 401 UNATHORIZED | 403 FORBIDDEN | 500 INTERNAL SERVER ERROR | 501 ...)
return <code> [<message>];

# Sustituye la ruta original por otra ruta objetivo (internamente).
rewrite <source> <target> <mode>;
```

[http://nginx.org/en/docs/http/ngx_http_rewrite_module.html](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html)

### Modos de Reescritura de Rutas

Modo | Descripción
--- | ---
`last` | Reescribe la ruta y finaliza la ubicación (original)
`break` | Reescribe la ruta y rompe una posible condición, para continuar en la ubicación (original)
`redirect` | Redirige la ruta original a la objetivo (con efecto en el cliente).
`permanent` | Devuelve el código HTTP `301` indicando que la ruta ya no existe porque se movió (con efecto en el cliente).

* **NOTA:** El modo más utilizado es `last`. No especificar el modo o usar
otro, podría tener resultados no deseos.

### Caso Práctico: Productos Dinámicos

Supongamos que la ruta `/product-info.shtml?id=123` nos devuelve la información del
producto especificado mediante el parámetro query `id`.

Sin embargo, no es nuestro deseo exponer tal ruta. Sino, que el cliente
debería solicitar la ruta `/products/123/info`.

Entonces tenemos que reescribir la ruta `/products/123/info` hacia la ruta
`/product-info.shtml?id=123`.

> 1. Crear la ubicación final/objetivo (`target`)

```
root /XYZ/products;

# /XYZ/products/product-info.shtml
location /product-info.shtml {
	ssi on;
}
```

> /XYZ/products/product-info.shtml

```
<h1>Información del Producto</h1>

<!--#include file="info_$arg_id.txt"-->
```

> 2. Crear la ubicación inicial/fuente (`source`)

```
location /products {
	rewrite /products/(.*)/info /product-info.shtml?id=$1 last;
}
```

* **NOTA:** No se debe olvidar el modo `last`.


# Sesión 4. Autenticación Basica (usuario/contraseña)

Algunas veces vamos a requerir una autenticación básica para
proteger nuestras rutas y a través de `htpasswd` instalado mediante
`sudo apt install apache2-utils`. Podemos crear archivos de autenticación
básica mediante usuario y contraseña y activarlos sobre nuestras rutas
para protegerlas.

## 1. Generar un archivo de credenciales

	$ htpasswd -c /XYZ/security/credenciales1 user1

	--> New password: ***

## 2. Podríamos agregar más usuarios a las credenciales

	$ htpasswd /XYZ/security/credenciales1 user2

	--> New password: ***

## 3. Utilizar la directiva `auth_basic` para habilitar la autenticación

```
location / {
	auth_basic "Ruta protegida";
	auth_basic_user_file /XYZ/security/credentiales1;

	root /XYZ/www;
}
```

[https://docs.nginx.com/nginx/admin-guide/security-controls/configuring-http-basic-authentication/](https://docs.nginx.com/nginx/admin-guide/security-controls/configuring-http-basic-authentication/)



































