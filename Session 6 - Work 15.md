# Trabajo 15. Balanceo de carga en un grupo de servidores

## 1. Definir un grupo de servidores

> `/XYZ/groups/group_apache1.conf`

```
upstream apache1 {

	server localhost:16500 weight=2 down;
	server localhost:14500 weight=3;
	server localhost:15001 weight=2;
	server localhost:13500 weight=1;
	server localhost:20000 weight=1;
	server localhost:12101 backup;
	
	# least_conn;
}

```

## 2. Consumir el grupo de servidores a tráves de `proxy_pass`

> `/XYZ/sites/server_n.conf`

```
server {

	listen 12345;

	location / {
		proxy_pass http://apache1/;
	}

}
```

## 3. Probar que los recursos se distribuyan correctamente

	$ curl localhost:1234


















