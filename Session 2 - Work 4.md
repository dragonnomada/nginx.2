# Trabajo 4. Registrar el Ambiente de Nginx como Servicio

## 1. Crear una unidad de Servicio

	$ sudo nano /usr/lib/systemd/system/nginx-abc.service

## 2. Habilitar el Servicio

	$ sudo systemctl enable nginx-abc.service

## 3. Deshabilitar el Servicio

	$ sudo systemctl disable nginx-abc.service

## 4. Verificar el estatus del Servicio (apagado)

	$ sudo systemctl status nginx-abc.service

## 5. Encender el Servicio

	$ sudo systemctl start nginx-abc.service

## 6. Verificar el estatus del Servicio (encendido)

	$ sudo systemctl status nginx-abc.service

## 7. Detener el Servicio

	$ sudo systemctl stop nginx-abc.service

## 8. Reiniciar el Servicio

	$ sudo systemctl restart nginx-abc.service

## 9. Recargar el Servicio (reload)

* Modifica el archivo `nginx.conf` para ver los resultados
actualizados.

	$ sudo systemctl reload nginx-abc.service