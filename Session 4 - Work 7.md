# Trabajo 7. Crear un autoindexado

## 1. Crear un servidor con la ruta principal autoindexada

```
server {
	
	listen 12345;

	location / {
		autoindex on;
		alias /XYZ/music
	}

}
```

## 2. Simular algunos archivos de música

	$ mkdir -p /XYZ/music

	$ mkdir -p /XYZ/music/artist/chente

	$ touch /XYZ/music/gaga.mp3
	$ touch /XYZ/music/travis.mp3
	$ touch /XYZ/music/artist/chente/---.mp4

## 3. Verificar que el índice sea correcto

	$ curl localhost:12345/

	--- <a href="artist/...">....mp3</a> ---