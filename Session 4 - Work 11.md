# Trabajo 11. Autenticación Básica

## 1. Crear un archivo de credenciales

Para registrar usuarios y contraseñas de forma segura.

	$ htpasswd -c /XYZ/security/credenciales1 user1

	>>> New password: ****

> Inspeccionar las credenciales

	$ cat /XYZ/security/credenciales1

	--- user1:x2312sdcxcz22134ZXxzcz

## 2. Registrar más usuarios en nuestro archivo de credenciales

	$ htpasswd /XYZ/security/credenciales1 user2

	>>> New password: ****

> Inspeccionar las credenciales

	$ cat /XYZ/security/credenciales1

	--- user1:********
	--- user2:********

## 3. Activar una ruta para solicitar las credenciales

> `/XYZ/sites/server_n.conf`

```
server {

	listen 12345;

	location / {
		auth_basic: "Leyenda | Ruta Protegida";
		auth_basic_user_file /XYZ/security/credenciales1;

		root /XYZ/www;
	}

}
```

## 4. Validar las peticiones seguras

	http://IP:12345/
	--> user: user1
	--> password: *****

	$ curl -u "user1:****" http://localhost:12345/

	$ curl -u http://user1:****@localhost:12345/













