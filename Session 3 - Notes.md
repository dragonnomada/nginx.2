# Sesión 3. Configuración de Nginx

Usos principales Nginx

* **Servidor Web** - Proveer recursos estáticos y dinámicos.
* **Proxy Inverso** - En un mismo puerto redireccionar las peticiones a otros servidores o puertos
* **Balanceador de Carga** - Forma grupos/clusters de servicios para equilibrar las peticiones hacia esos servicios

## 1. Definir un Servidor Virtual en un Puerto

Generalmente las peticiones web se realizan mediante
el protocolo HTTP y la comunicación se da a tavés del
protocolo TCP. Esto quiere decir, que cuándo un cliente
remoto quiere consultar un recurso del servidor, 
establecerá una *Petición Web* que consiste en
una URL (cadena de texto), cabeceras asociadas a la petición
(headers) y un cuerpo de petición (body o contenido).

La petición es enviada al servidor mediante el protocolo
TCP y se da en un puerto específico. Para lograr esto
se debe especificar un SERVIDOR/DOMINIO (nombre del servidor)
o directamente la IP pública. De esta manera, el cliente
remoto podrá enviar la petición hacía el puerto específico
del servidor.

Por ejemplo. Un cliente remoto necesita la página
about.html del servidor. Por lo que consulta lo siguiente

	HOST 3.131.99.91
	PORT 10000
	HTTP GET /about.html
	-H Content-Type text/html
	-H Authorization Bearer 123456789

	URL http://3.131.99.91:10000/about.html

El servidor recibe en el puerto específico (10000)
la petición mediante el TCP y la procesa mediante HTTP.

Para esto, entonces, el servidor necesita acaparar o
reservar el puerto 10000 para procesar todas
las peticiones envíadas hacia ese puerto.

En Nginx podemos crear *Servidores Virtuales* (Virtual Hosts)
en los que definamos las reglas (directivas) que guiarán
el procesamiento de las peticiones en tres posibles caminos

* **Procesar la petición como un recurso**. Ejemplo, `/about.html`
entregar el contenido del archivo `www/about.html`, `/img/logo.png`
entregar el contenido del archivo `www/images/logo.png`.
* **Reescribir la petición hacía otra ruta**. Ejemplo, `/about`
entregar la ruta `/about.html`.
* **Pasar la petición vía proxy o fastcgi hacía otro servidor
o puerto**. Ejemplo, `/products/123/info` pasarle la petición
al servidor `http://localhost:4141/`.
* **Devolver algún código de HTTP**. Ejemplo, `404` o `Not Found`,
`500` o `Internal Server Error`.

La forma más directa de definir un Servidor Virtual de Nginx
es manipular el archivo `conf/nginx.conf` para establecer
las directivas del servidor.

Las directivas son reglas o grupos de reglas que guian
al servidor Nginx para que funcione de forma personalizada
según esas reglas. Las podemos pensar como imposiciones
o directivas que guían la forma de operación de Nginx.

La directiva principal para establecer los servidores virtuales
es la directiva de grupo `http`, la cuál pertenece al módulo
`http`. La directiva `http` nos permite definir la forma
de procesar las peticiones HTTP. Por ejemplo, habilitar
el envío de archivos, decidir cuánto tiempo se mantendrá
viva la conexión, determinar los tipos de contenido MIME
para las diferentes extensiones de los archivos y la
definición de servidores virtuales.

> Sintaxis del bloque `http`

```
http {
	<REGLAS | DIRECTIVAS>
}
```

Existe una directiva de bloque llamada `server` que es
utilizada dentro del bloque `http` para establecer
servidores virtuales. Por cada `server` se definirá
un nuevo servidor virtual.

> Sintaxis del bloque `server`

```
http {
	
	server {
		<REGLAS | DIRECTIVAS>
	}

	server {
		<REGLAS | DIRECTIVAS>
	}

	...

}
```

La regla o directiva principal para configurar un servidor
virtual es `listen`. Esta directiva `listen` reserverá
(acapará) el puerto especificado para uso exclusivo
del servidor virtual. Podemos reservar múltiples puertos
y dominios/alcances.

> Sintaxis de la directiva `listen`

```
server {
	listen 10000;
	...
}
```

Cada que el archivo `nginx.conf` o alguna configuración
derivada sea modificada, tendremos que integrar los nuevos
cambios al servidor de ser posible. Usando la señal `reload`.

	$ /ABC/nginx-abc/sbin/nginx -s reload

	$ sudo systemctl reload nginx-abc

> Ejemplo de una configuración mínima para un servidor virtual.

```
worker_processes  1;

events {
    worker_connections  1024;
}

http {

        server {
                listen 10000;
        }

}
```

## 2. Separar las definiciones de los Servidores Virtuales

No es buena practica estar administrando los servidores
virtuales directamente sobre el `nginx.conf`.

1. No hay separabildad. Si múltiples administradores
modifican el archivo `nginx.conf` podrían tener colisiones
de código. Es decir, se podrían sobreescribir o interferir.
2. Ilegible. Las configuraciones específicas quedan
mezcladas con las configuraciones generales.
3. Seguridad. No hay garantías de que al eliminar
accidentalmente el ambiente, este guarde las configuraciones
en un lugar seguro. Las configuraciones específicas
no deberían estar dentro del ambiente de Nginx.

En Nignx existe una directiva llamada `include` que
nos permite separar las directivas y bloques en 
otros archivos. Podemos incluir archivos específicos
o usar los comodines (wildcards) para incluir
automáticamente todos archivos de una carpeta
incluso indicando la extensión.

> Sintaxis de `include`

```
worker_processes  1;

# Incluye todos los archivos de la carpeta /ABC/conf
include /ABC/conf/*;

events {
    worker_connections  1024;
}

http {

	# Incluye los archivos .conf de la carpeta /ABC/sites
	# Las configuraciones quedarían bajo http { ... }
	include /ABC/sites/*.conf;

        server {
                listen 10000;

		# Incluye las directivas/reglas del archivo
		# Las configuraciones estarían bajo server { ... }
		include /ABC/sites/server-1.location.conf;
        }

}
```

Observa que dónde se mande a incluir un archivo,
a ese nivel quedaría incluído. Es como si hubieramos
escrito directamente las reglas en lugar del include.

### Organización clásica de las cofiguraciones de Nginx

En nuestro ambiente de Nginx podemos determinar una carpeta
`sites-enabled` que contenga todos los archivos que deberían
contener directivas de tipo `server` indicando la configuración
de cada uno de nuestros servidores disponibles.

Esta carpeta la debería incluir en el ambiente dentro de
`nginx.conf` mediante la directiva `include sites-enabled/*`.

Cada que queramos enlazar una configuración (activarla)
deberíamos colocar un enlace simbólico o un archivo físico
(no es buena practica), de nuestra nuestra configuración
a la carpeta `conf/sites-enabled`.

	$ mkdir -p /ABC/nginx-abc/conf/sites-enabled

	$ ln -s /ABC/sites/server_n.conf /ABC/nginx-abc/conf/sites-enabled/


















