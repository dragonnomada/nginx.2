# Trabajo 10. Reescribir rutas dinámicas hacía procesadores

## 1. Creamos un procesador de nuestras peticiones.

Tenemos 3 alternativas:

1. Usar SSI dentro de Nginx
2. Usar PHP integrado de Nginx
3. Consumir un backend externo a Nginx

> `/XYZ/pictures/view-image.shtml`

	Muestra una imagen mediante una etiqueta img con ancho y alto
	específicos y con la ruta específica hacía la url de la imagen original.

```html
<img
	src="/img/<!--#echo var='arg_n' default='logo.png'-->"
	width="<!--#echo var='arg_w' default='100'-->"
	height="<!--#echo var='arg_h' default='100'-->"
>
```

> `/XYZ/sites/server_n.conf`

```
server {
	listen 12345;

	...

	location /view-image.shtml {
		root /XYZ/pictures;
	}
}
```

> Test 1. `curl http://localhost:12345/view-image.shtml?n=foo.jpg&w=45&h=65`

```
<img src="/img/foo.jpg" width="45" height="65" >
```

## 2. Proveer los recursos estáticos

Cómo el procesador depende de las rutas `/img/<name.jpg>`. Exponer dichos
recursos estáticos.


> `/XYZ/sites/server_n.conf`

```
server {
	listen 12345;

	location /img {
		alias /XYZ/pictures/img;
	}

	location /view-image.shtml {
		root /XYZ/pictures;
	}
}
```

> Test 2. `curl http://localhost:12345/img/name.jpg --output test.jpg`

* **NOTA:** Puedes descargar la imagen con `wget <url>.jpg -o <name>.jpg`

## 3. Reescribir las rutas canónicas (dinámicas) por rutas originales.

La idea es reescribir todas las rutas con la estructura `/picture/(.*)x(.*)/(.*)`
por la ruta `/view-image.shtml?n=$3&w=$1&h=$2`.


> `/XYZ/sites/server_n.conf`

```
server {
	listen 12345;

	# Recursos estáticos
	location /img {
		alias /XYZ/pictures/img;
	}

	# El procesador de las imágenes
	location /view-image.shtml {
		root /XYZ/pictures;
	}

	# La reescritura
	location ~ /picture/(.*)x(.*)/(.*) {
		rewrite /picture/(.*)x(.*)/(.*) /view-image.shtml?n=$3&w=$1&h=$2 last;
	}

}
```

> Test 3. `curl http://localhost:12345/picture/45x65/foo.jpg`

```
<img src="/img/foo.jpg" width="45" height="65" >
```

