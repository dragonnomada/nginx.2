# Sesión 2 - Instalar Nginx por Compilación

> 1. Versión de Nginx http://nginx.org/download/

> 2. Configuración personalizada

- Ruta de instalación
- Rutas adicionales (logs, temp, conf, sbin)
- Activar/Desactivar Módulos
- Configuraciones adicionales

> 3. Habilitar servicios pernalizados

## 1. Descargar el código fuente de Nginx

	$ wget http://nginx.org/download/nginx-1.21.5.tar.gz

## 2. Descomprimir el código fuente de Nginx

	$ tar zxf nginx-1.21.5.tar.gz

## Tabla de Archivos y Carpetas principales del Código Fuente de Nginx

Ubicación | Descripción
--- | ---
`configure` | Es un ejecutable encargado de crear el archivo `make` para poder compilar Nginx. El configure recibe todas las banderas y opciones de personalización.
**`conf/`** | Contiene las configuraciones iniciales al instalar. Aquí se encuentra el `nginx.conf` y podríamos modificarlo para indicar una arquitectura inicial (habilitar una carpeta sites-enabled).
`conf/nginx.conf` | Podemos personalizar la configuración por defecto de Nginx. Por ejemplo, deshabilitar el servidor del puerto 80.
**`html`** | Contiene los archivos html por defecto que son provistos.
**`src`** | Contine los códigos fuente de Nginx escritos en C. Están organizados por módulos.


## 3. Personalizar el `conf/nginx.conf`

	$ cd nginx-1.21.5/

	$ cat conf/nginx.conf

	$ nano conf/nginx.conf

```txt
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;

    keepalive_timeout  65;

    include sites-enabled/*;

}
```

* **NOTA:** El objetivo principal es eliminar las líneas
que crean un servidor virtual sobre el puerto 80. Esto
para que nuestra instalación no entre en conflicto
con ningua otra instalación o servicio ya habilitado.
Si no hacemos esto, podríamos entrar en conflicto por
querer utilizar el mismo puerto (el 80) con algún otro
proceso del servidor.

* **NOTA:** Habilitamos incluir todos los archivos
de configuración de la carpeta `site-enabled`. Esto
nos permitirá incluir automáticamente nuestras configuraciones
ya sea colocando archivos físicos ahí o símbolicos.

	$ mkdir conf/sites-enabled

## 4. Configurar la instalación (generar el Makefile)

	$ ./configure --help

### Configuraciones principales de Nginx

Opción | Descripción
--- | ---
**`--prefix=<path>`** | Establecemos cuál es la ruta (`<path>`) principal de la instalación. Esta ruta contendrá la carpeta `conf`, `html`, `log`, etc.
`--sbin-path=<path>` | Establecemos la carpeta del binario de Nginx. Si no lo hacemos la ruta será `<prefix>/sbin/nginx`. Se recomienda dejar el binario dentro de la carpeta de instación.
`--build=<name>` | Podemos establecer el nombre compilación, para no estar recompilando sobre los mismos archivos.
`--builddir=<dir>` | Podemos establecer el directorio dónde quedará el compilado.
`--user=<user>` | Usuario del sistema
`--group=<group>` | Grupo del sistema
`--with-http_ssl_module` | Activa un módulo para que esté disponible con la compilación.
`--without-http_ssl_module` | Desactiva un módulo para que no esté disponible con la compilación.

	$ sudo apt install -y gcc make libxml2-utils \
		xsltproc devscripts quilt \
		debhelper libssl-dev libpcre3-dev \
		zlib1g-dev libgeoip-dev libgd-dev

	$ ./configure --prefix=/ABC/nginx-abc \
		--user=ubuntu \
		--group=ubuntu \
		--build=2022.01.001 \
		--builddir=2022.01.001

### Resultado de la Compilación

```txt
Configuration summary
  + using system PCRE library
  + OpenSSL library is not used
  + using system zlib library

  nginx path prefix: "/ABC/nginx-abc"
  nginx binary file: "/ABC/nginx-abc/sbin/nginx"
  nginx modules path: "/ABC/nginx-abs/modules"
  nginx configuration prefix: "/ABC/nginx-abc/conf"
  nginx configuration file: "/ABC/nginx-abc/conf/nginx.conf"
  nginx pid file: "/ABC/nginx-abs/logs/nginx.pid"
  nginx error log file: "/ABC/nginx-abc/logs/error.log"
  nginx http access log file: "/ABC/nginx-abc/logs/access.log"
  nginx http client request body temporary files: "client_body_temp"
  nginx http proxy temporary files: "proxy_temp"
  nginx http fastcgi temporary files: "fastcgi_temp"
  nginx http uwsgi temporary files: "uwsgi_temp"
  nginx http scgi temporary files: "scgi_temp"
```

* **NOTA:** Es importante guardar el resumen de la configuración
de cada build, para poder recodar en el futuro
cuáles fueron las rutas utilizadas o generadas
en la instalación. Con el objetivo de poder desinstalar
(eliminar) todos los recursos de este ambiente compilado.

	$ ./configure --prefix=... > summary.2022.01.001

## 5. Compilar la instalación configurada (make build)

	$ cat Makefile

	$ make build

## 6. Aplicar la instalación compilada (make install)

	$ make install

* **NOTA:** En este momento `make install` ya ha generado
nuestro ambiente aislado de Nginx, según fue configurado
y compilado. Podemos revisar su estructura.

	$ tree /ABC/nginx-abc

	$ cat /ABC/nginx-abc/conf/nginx.conf

## 7. Verficar el ejecutable principal de nuestro ambiente

	$ /ABC/nginx-abc/sbin/nginx -V

### Resultado de la versión Nginx

```
nginx version: nginx/1.21.5 (2022.01.001)
built by gcc 9.3.0 (Ubuntu 9.3.0-17ubuntu1~20.04)
configure arguments: --prefix=/ABC/nginx-abc --build=2022.01.001 --builddir=2022.01.001 --user=ubuntu --group=ubuntu
```














