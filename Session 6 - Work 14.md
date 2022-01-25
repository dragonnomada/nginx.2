# Trabajo 14. Usar un mismo Puerto con distos Nombres de Servidor

## 1. Crear 3 servidores sobre el mismo puerto

> `/XYZ/sites/server_n_one.conf`

```
server {

	listen 12345;
	server_name one.com;
	
	location / {
		root /XYZ/www/one;
	}

}
```

> `/XYZ/sites/server_n_two.conf`

```
server {

	listen 12345;
	server_name two.com;
	
	location / {
		root /XYZ/www/two;
	}

}
```

> `/XYZ/sites/server_n_three.conf`

```
server {

	listen 12345;
	server_name three.com;
	
	location / {
		root /XYZ/www/three;
	}

}
```

## 2. Enlazar los servidores

	$ ln -s /XYZ/sites/server_n_one.conf /XYZ/nginx-xyz/conf/sites-enabled

	$ ln -s /XYZ/sites/server_n_two.conf /XYZ/nginx-xyz/conf/sites-enabled

	$ ln -s /XYZ/sites/server_n_three.conf /XYZ/nginx-xyz/conf/sites-enabled

## 3. Registrar los dominios virtuales

> `/etc/hosts`

```
127.0.0.1 localhost one.com two.com three.com

...
```

## 4. Recargar el ambiente

	$ sudo systemctl reload nginx-xyz

## 5. Recursos de prueba

	$ mkdir -p /XYZ/www/one

	$ mkdir -p /XYZ/www/two

	$ mkdir -p /XYZ/www/three

	$ echo "One" > /XYZ/www/one/index.html

	$ echo "Two" > /XYZ/www/two/index.html

	$ echo "Three" > /XYZ/www/three/index.html

## 6. Probar los dominios

	$ curl one.com:12345

	--- One

	$ curl two.com:12345

	--- Two

	$ curl three.com:12345

	--- Three



























