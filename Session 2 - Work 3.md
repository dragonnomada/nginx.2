# Trabajo 3. Prendido y Apagado del Ambiente Nginx

## 1. Encender el Ambiente en forma manual

	$ /ABC/nginx-abc/sbin/nginx

## 2. Verificar que el ambiente esté prendido

	$ ps ax | grep nginx

## 3. Detener el proceso de Nginx por PID

	$ sudo kill <pid>

## 4. Verficar que el ambiente esté apagado

	$ ps ax | grep nginx

## 5. Detener todos los procesos de nginx

	$ sudo killall nginx

## 6. Verficar que el ambiente por usuario

	$ ps -ef | grep nginx

## 7. Consultar el PID del ambiente `<prefix>/logs/nginx.pid`

	$ cat nginx.pid

## 8. Detener el proceso de Nginx

	$ sudo kill $(cat /ABC/nginx-abs/logs/nginx.pid)

## 9. Crear un alias para nuestro ejecutable de nginx

	$ alias nginx-abc="/ABC/nginx-abc/sbin/nginx"

## 10. Detener el proceso de Nginx mediante señales

	$ nginx-abc -s stop

## 11. Comprobar que las configuraciones sea correctas

	$ nginx-abc -t

## 12. Recargar nuevas configuraciones

	$ nginx-abc -s reload



