# Sesión 3. Configuración de Nginx

Usos principales Nginx

* **Servidor Web** - Proveer recursos estáticos y dinámicos.
* **Proxy Inverso** - En un mismo puerto redireccionar las peticiones a otros servidores o puertos
* **Balanceador de Carga** - Forma grupos/clusters de servicios para equilibrar las peticiones hacia esos servicios

## 1. Definir un servidor virtual en un puerto

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





	
















