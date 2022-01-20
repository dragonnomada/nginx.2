# Trabajo 5. Configurar el Ambiente de Nginx con Sites Enabled

## 1. Editar el archivo `nginx.conf` para habilitar `sites-enabled`

	$ nano /ABC/nginx-abc/conf/nginx.conf

```
worker_processes  1;

events {
    worker_connections  1024;
}

http {

    include sites-enabled/*;

}
```

## 2. Crear un servidor de pruebas en su carpeta sites local

	$ mkdir /ABC/sites
	
	$ nano /ABC/sites/server_n.conf

```
server {
	listen 12345;
}
```

## 3. Activar el servidor local en nuestro ambiente (enlace simbólico)

	$ mkdir -p /ABC/nginx-abc/conf/sites-enabled

	$ ln -s /ABC/sites/server_n.conf /ABC/nginx-abc/conf/sites-enabled

## 4. Recargamos/Reiniciar el Ambiente

	$ /ABC/nginx-abc/sbin/nginx -s reload

	$ sudo systemctl reload nginx-abc