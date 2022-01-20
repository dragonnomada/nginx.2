# Trabajo 6. Servidor de Recursos Estáticos

## 1. Crear un servidor de recursos estáticos

	$ nano /XYZ/sites/server_n.conf

```
server {

	listen 12345;
	
	location / {
		root /XYZ/www;
	}

	location /about.html {
		root /XYZ/docs;
	}

	location /img {
		root /XYZ/pictures;
	}
	

}
```

## 2. Enlazar el servidor

	$ ln -s /XYZ/sites/server_n.conf /XYZ/nginx-xyz/conf/sites-enabled

## 3. Recargar la configuración del servidor

	$ sudo systemctl reload nginx-xyz

## 4. Genera algunos archivos de prueba

	$ echo "<h1>Index</h1>" > /XYZ/www/index.html

	$ echo "<h1>About</h1>" > /XYZ/docs/about.html

	$ touch > /XYZ/pictures/img/logo.png