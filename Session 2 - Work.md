# Trabajo 2 - Configurar un ambiente aislado de Nginx por Compilación

## 1. Descarga el código fuente de Nginx v1.21.5

	$ wget http://nginx.org/download/nginx-1.21.5.tar.gz

## 2. Descomprime el código fuente

	$ tar zxf nginx-1.21.5.tar.gz

## 3. Personaliza el archvio principal de configuración `nginx.conf`

	$ nano conf/nginx.conf

## 4. Configura la instalación

	$ ./configure --prefix=/ABC/nginx-abc \
		--user=ubuntu \
		--group=ubuntu \
		--build=2022.01.001 \
		--builddir=2022.01.001 > summary.2022.01.001

## 5. Compila el código fuente

	$ make build

## 6. Instala el código compilado

	$ make install

## 7. Verifica la información de la compilación

	$ /ABC/nginx-abc/sbin/nginx -V