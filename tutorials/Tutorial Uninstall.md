# Tutorial - Desinstalación de Nginx (Por Gestor de Paquetes)

## 1. Remover el servicio

	$ sudo systemctl stop nginx.service

	$ sudo systemctl disable nginx.service

	$ sudo rm /lib/systemd/system/nginx.service

## 2. Remover las carpetas de Nginx

	$ sudo rm -rf /etc/nginx
	
	$ sudo rm -rf /var/log/nginx

## 3. Remover el binario de nginx

	$ sudo rm /usr/sbin/nginx

