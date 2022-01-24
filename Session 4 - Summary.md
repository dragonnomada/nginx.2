# Sesión 4. Resumen de Recursos Estáticos y Dinámicos

1. Un servidor de recursos estáticos
mantiene sus recursos en directorios
y podemos generar rutas que apunten
hacía dichos recursos.

Ejemplo:

/XYZ/pictures/img/[gif|jpg|png|svg|...]

location /img {
  root /XYZ/pictures/;
}

# http://IP:PORT/img/logo.png
# -->  /XYZ/pictures/img/logo.png

2. Podemos crear o enlazar procesadores
de archivos/recursos como PHP/SHTML/NODEJS/...
Estos generalmente reciben sus parámetros
dinámicos para operar muchas tareas a la vez,
mediante los query params (parámetros sobre la url)

/process-image.php?name=logo.png&w=200&h=50

--> <Regresa la imagen procesada>

/api/product-list?minId=8&max=54&page=2

--> <Regresa el JSON de los productos>

location /view-image.shtml {
  ssi on;
  root /XYZ/picture;
}

# http://IP:PORT/view-image.shtml?name=logo.png
# --> /XYZ/picture/view-image.shtml

3. El problema es que las rutas de los procesadores
suelen ser complejas o requieren ser privadas.
Es decir, que un usuario/cliente no debería
consumir directamente dichos procesadores, para
evitar un mal uso, por ejemplo, que lo usen
de más, lo use cualquier persona, lo usen sin
seguridad, o coloquen mal los parámetro (mal formato).

location ~ /picture/(.*)x(.*)/(.*) {
  rewrite /picture/(.*)x(.*)/(.*) /view-image.shtml?n=$3&w=$1&h=2 last;
}

# http://IP:PORT/picture/64x96/logo.png
# --> http://IP:PORT/view-image.shtml?n=logo.png&w=64&h=96













