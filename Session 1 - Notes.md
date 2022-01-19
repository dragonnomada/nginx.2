# Sesión 1. Instalación de Nginx Linux mediante el Gestor de Paquetes

## Introducción

Linux es un sistema operativo, cuya raíz principal es `/`, a diferencia de otros sistemas operativos como Windows,
este no posee C: o D: para acceder a los archivos disponibles. En su lugar, usa la ruta `/` para navegar desde
ahí hacía cualquier archivo disponible.

Podemos ver las carpetas principales de Linux mediante `ls /` o `ll /`.

### Carpetas Importantes de Linux

Carpeta | Descripción
--- | ---
**`/etc`** | Contiene los archivos de la mayoría de las aplicaciones mantenidas mediante el gestor de paquetes
**`/usr`** | Contiene archivos comunes dirigidos al usuario, instalados por las aplicaciones mantenidas mediante el gestor de paquetes
**`/usr/lib`** | Contiene archivos de librerías dirigidos al usuario, instalados por las aplicaciones mantenidas mediante el gestor de paquetes
**`/usr/sbin`** | Contiene achivos ejecutables (binarios) dirigidos al usuario, instalados por las aplicaciones mantenidas mediante el gestor de paquetes
**`/var`** | Contiene archivos variables, usados por las aplicaciones mantenidas mediante el gestor de paquetes
**`/var/log`** | Contiene carpetas que guardan archivos de Log, para aplicaciones mantenidas mediante el gestor de paquetes
**`/var/www`** | Ejemplo de una carpeta principal para almacenar archivos estáticos provistos por Apache o Nginx

## 1. Instalación Genérica de Nginx (gestor de paquetes)

	# Actualizar el Gestor de Paquetes (Ubuntu / Aptitude)
	$ sudo apt update

	# Instalar Nginx desde el Gestor de Paquetes
	$ sudo apt install nginx

### Carpeta Principal de Nginx (gestor de paquetes)

	# Ir a la carpeta principal de Nginx (instalada desde el gestor de paquetes)
	$ cd /etc/nginx/

### Carpetas y archivos importantes de Nginx

Ubicación | Descripción
--- | ---
**`/etc/nginx/`** | Ruta principal de Nginx (instalado desde el gestor de paquetes)
**`/etc/nginx/nginx.conf`** | Archivo principal de configuración
**`/etc/nginx/mime.types`** | Archvio de especificación de los tipos MIME y sus extensiones de archivos
**`/etc/nginx/conf.d/`** | Carpeta por defecto para las configuraciones principales enlazadas
**`/etc/nginx/modules-available/`** | Carpeta segura para colocar configuraciones de módulos disponibles
**`/etc/nginx/modules-enabled/`** | Carpeta por defecto para las configuraciones de módulos activadas (enlazadas o físcas)
**`/etc/nginx/sites-available/`** | Carpeta segura para colocar las configuraciones de servidores virtuales y configuraciones del módulo http
**`/etc/nginx/sites-enabled/`** | Carpeta por defecto para colocar las configuraciones de servidores virtuales y configuraciones del módulo http activadas (enlazadas preferente)
**`/usr/sbin/nginx`** | Ejecutable principal de Nginx (binario/demonio principal)
**`/var/log/nginx/access.log`** | Archivo de los logs de acceso (por gestor de paquetes)
**`/var/log/nginx/error.log`** | Archivo de los logs de errores (por gestor de paquetes)

### Carpeta de Ejecutables

	# Ir a la carpeta principal de ejecutables
	$ cd /usr/sbin
	
	# Filtrar los ejecutables que comienzan con `n`
	$ ls | grep '^n'

### Ejecutable principal de Nginx (gestor de paquetes)

	# Encender el servidor Nginx manualmente
	$ /usr/sbin/nginx

	# Detener el servidor de Nginx manualmente (señal de apagado forzado)
	$ /usr/sbin/nginx -s stop

	# Detener el servidor de Nginx manualmente (señal de apagado amistoso)
	$ /usr/sbin/nginx -s quit

	# Detener el servidor de Nginx manualmente (matar los procesos de nginx)
	$ killall nginx

### Carpeta principal de logs (gestor de paquetes)

	# Ir a la carpeta de los logs (instalado por el gestor de paquetes)
	$ cd /var/log/nginx

	# Mostrar todos los LOGS de acceso
	$ cat /var/log/nginx/access.log

	# Mostrar todos los LOGS de errores
	$ cat /var/log/nginx/error.log

	# Mostrar los últimos dos LOGS de acceso
	$ tail -n 2 /var/log/nginx/access.log

### Comprobar si Nginx funciona

	* En el navegador remoto
	http://3.131.99.91:80

	# En una terminal remota
	$ curl http://3.131.99.91:80

	# Dentro del servidor
	$ curl http://localhost:80

	* RESULTADO:
	<h1>Welcome to nginx!</h1>

## 2. Crear un Servidor Virtual de pruebas (independiente)

### Definir la configuración de un servidor virtual

	# Crear una carpeta de administración de servidores virtuales
	$ sudo mkdir /ABC

	# Otorgar los permisos
	$ sudo chmod 777 /ABC

	# Crear la carpeta de `sites` para guardar las configuraciones
	# de servidores virtuales
	$ mkdir /ABC/sites

	# Crear un archivo
	$ nano /ABC/sites/abc_server_n.conf

> ~ `/ABC/sites/abc_server_n.conf`

```
# Declaramos el servidor
server {
	# Que escuche sobre el puerto 12345
	listen 12345;
  
	# Y para cada petición
	location / {
		# Devuelva el estatus 500 (Internal Server Error)
		return 500;
	}
}
```

* **NOTA:** En `nano` pulsa `ctrl+o` + `enter` para guardar y `ctrl+x` para salir.

* **NOTA:** De ser necesario, se deben otorgar los permisos recursivamente: `sudo chmod -R /ABC`

### Inspeccionar la carpeta de configuraciones `ABC`

	$ tree /ABC
	.
	├── ports.txt
	└── sites
	    └── abc_server_n.conf

	1 directory, 2 files

## 3. Activar la configuración local a la global

Vamos a crear un enlace simbólico, entre el archivo físico (/ABC/sites/abc_server_n.conf) para enlazarlo a la carpeta de los sitios habilitados en nginx (/etc/nginx/sites-enabled/).

	# Crear un enlace simbólico de una ubicación a otra
	$ sudo ln -s /ABC/sites/abc_server_n.conf /etc/nginx/sites-enabled/

* **NOTA:** Usa la ruta absoluta (`/ABC/sites/abc_server_n.conf`), sino el enlace será inválido.

### Verificar previamente que los enlaces simbólicos sean correctos

	# Listamos todas las configuraciones activas
	$ ll /etc/nginx/sites-enabled/

	# El resultado debería indicar algo similar (en color positivo)
	XXX XXX XXX abc_server_n.conf -> /ABC/sites/abc_server_n.conf

	! Si el color es negativo (rojo), el enlace simbólico es incorrecto
	! Se debe usar siempre la ruta absoluta al crear enlaces simbólicos

### Explicación

El archivo de configuración principal `/etc/nginx/nginx.conf` incluye dentro del bloque `http` 
todas las configuraciones de `/etc/nginx/sites-enabled/*`.

Creamos un enlace simbólico del archivo local a la administración `/ABC/sites/abc_server_n.conf`
hacía la carpeta `/etc/nginx/sites-enabled/`, dando como resultado
simular haber creado el archivo `/etc/nginx/sites-enabled/abc_server_n.conf`.

El archivo `/etc/nginx/sites-enabled/abc_server_n.conf` será incluído como parte de las configuraciones
del bloque `http`. Dando como resultado la inclusión de nuestro archivo del servidor como parte de
las configuraciones de Nginx.

Si editamos el archivo `/ABC/sites/abc_server_n.conf`, significaría modificar la configuración de Nginx,
ya que equivaldría a modificar el archivo `/etc/nginx/sites-enabled/abc_server_n.conf`.

## 4. Verificar la sintaxis de las configuraciones

	# Prueba si la sintaxis de la configuración es correcta
	$ sudo /usr/sbin/nginx -t

* **NOTA:** Si todo sale bien debería decir `syntax ok ... successful`.

* **NOTA:** Si la configuración no es correcta, no se recargará el servidor cuándo se le pida.

### Para quitarlo de Nginx hacemos:

	# Eliminar un enlace dinámico malformado o que se quiera desactivar dicha configuración
	$ sudo rm /etc/nginx/site-enabled/abc_server_n.conf

* **NOTA:** Se borra el archivo del enlace simbólico, más no el original, es decir, el enlazado. Entonces, 
al borrar el enlace simbólico ya no hay referencia a la configuración local/externa.

### Ventajas de separar la configuración

1. Mantenibilidad/Orginación: Separación de configuraciones nos permite aislar las configuraciones secundares de la principal para poder activarlas y desactivarlas a voluntad, también permite que múltiples administradores tengan sus configuraciones separadas sin el riesgo que por error se las borren o modifiquen.
2. Precaución y Seguridad: En caso de restauración de Nginx (reinstalación/actualización), se podría perder toda la carpeta /etc/nginx y como consecuencia todas las configuraciones contenidas.

## 5. Recargar Nginx

	-- Al recargar Nginx toma las nuevas configuraciones para los servidores virtuales 
	-- y demás configuraciones actualizadas sin tener que reiniciar el servidor (recarga en tiempo real). 
	-- Los usuarios no se ven afectados, más que quizás por un pequeño retraso en la petición, 
	-- pero no hay bloqueos ni baja del servicio.

	# Enviar la señal para actualizar la configuración de Nginx
	# Esta señal indica que se recargue la configuración sin detener el servidor de Nginx
	$ sudo /usr/sbin/nginx -s reload

## 6. Comprobar que el nuevo servidor virtual funciona

	* En el navegador remoto
	http://3.131.99.91:12345

	# En una terminal remota
	$ curl http://3.131.99.91:12345

	# Dentro del servidor
	$ curl http://localhost:12345

	* RESULTADO:
	<h1>500 - Internal Server Error</h1>
