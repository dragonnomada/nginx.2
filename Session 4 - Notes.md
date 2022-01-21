## Sesión 4. Servidores como Recursos Dinámicos

Nginx provee un sistema de ubicaciones (locations) basados en
rutas fijas y rutas dinámicas capturadas a través de expresiones
regurales comunes al estándar Perl (PCRE).

Esto significa que podemos pensar en diseñar rutas que atrapen
recursos dinámicos. Con el fin de proveer recursos más sofisticados,
por ejemplo, crear listas de recursos, o cosa útiles.

## 1. Crear una lista de recusos

Nginx provee la directiva `autoindex` la cuál permite habilitar
o deshabilitar la capacidad de que una ruta autoindexe la raíz de
recursos. Por ejemplo, en un location que tenga especificado
un directorio con la directiva `root`, podrá autoindexarse.

> Sintaxis de la declarativa `autoindex`

```
# Crea una lista de los recursos disponibles en la ruta
autoindex <on|off>;

# Mostraría el tamaño exacto de los recursos
autoindex_exact_size <on|off>;

# Muestra sólo cierto tipo de archivos
autoindex_format <type1> | <type2> | ...;

# Cambia la hora para que no sea la del servidor, sino local al cliente
autoindex_locatime <on|off>
```

[http://nginx.org/en/docs/http/ngx_http_autoindex_module.html](http://nginx.org/en/docs/http/ngx_http_autoindex_module.html)

> Ejemplo

```
location /download {
	autoindex on;
	root /XYZ/download;
}
```

* **NOTA:** Para activar el servidor debemos crear el enlace dinámico
a la carpeta `sites-enabled`: 
`ln -s /ABS/sites/server_10003.conf /ABS/nginx-abs/conf/sites-enabled/`

* **NOTA:** Simpre que modifiquen un servidor se debe aplicar la señal
de reload `sudo systemctl reload nginx-xyz.service` o 
`/XYZ/nginx-xyz/sbin/nginx -s reload`























