# Trabajo 9. Reescritura de Rutas

Crear una página llamada `about.shtml` que procese el parámetro quey lang
con el lenguaje específicado. Si es inglés deberá incluir al archivo
`about_en.html` y si el `lang="es"` entonces incluir el archivo
`about_es.html`. 

## 1. Crear el servidor que provea el archivo about.shtml

```
server {

	listen 12345;

	ssi on;

	root /XYZ/miempresa;

}
```

## 2. Defina la ruta en inglés para reescribirla al archivo `about.shtml?lang=en`

```
server {

	listen 12345;

	ssi on;

	root /XYZ/miempresa;

	location /en/about {
		rewrite /en/about /about.shtml?lang=en last;
	}

}
```

## 3. Agregar otra ruta en español

```
server {

	listen 12345;

	ssi on;

	root /XYZ/miempresa;

	location /en/about {
		rewrite /en/about /about.shtml?lang=en last;
	}

	location /es/about {
		rewrite /es/about /about.shtml?lang=es last;
	}

}
```

### Alternativamente de forma general (Opcional)

```
location ~ /(.*)/about {
	rewrite /(.*)/about /about.shtml?lang=$1 last;
}
```

## 4. Crear el archivo `about.shtml`

> `/XYZ/miempresa/about.shtml`

```
<!--#include file="about_$arg_lang.html"-->
```

## 5. Crear el archivo `about_en.html`

> `/XYZ/miempresa/about_en.html`

```
<h1>About my company is here</h1>
```

## 5. Crear el archivo `about_es.html`

> `/XYZ/miempresa/about_es.html`

```
<meta charset="utf-8">

<h1>Aquí acerca de mi empresa</h1>
```

## 6. Probar que funcione

	$ curl localhost:12345/en/about

	--- <h1>About my company is here</h1> ---

	$ curl localhost:12345/es/about

	--- <h1>Aquí acerca de mi empresa</h1> ---








