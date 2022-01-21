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



































