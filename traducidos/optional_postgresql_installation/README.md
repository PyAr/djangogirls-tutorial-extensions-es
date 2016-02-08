# Instalación de PostgreSQL

> Parte de este capítulo esta basado en los tutoriales de Geek Girls Carrots (http://django.carrots.pl/).

> Parte de este capítulo esta basado en [django-marcador
tutorial](http://django-marcador.keimlink.de/) licenciado bajo Creative Commons
Attribution-ShareAlike 4.0 International License. El tutorial django-marcador tiene derechos de autor de Markus Zapke-Gründemann et al.


## Windows

La forma mas fácil de instalar Postgres en Windows es usando un programa que puedes encontrar aquí http://www.enterprisedb.com/products-services-training/pgdownload#windows

Elige la versión mas nueva disponible para tu sistema operativo. Descarga el instalador, ejecuta lo y sigue las instrucciones disponibles aquí: http://www.postgresqltutorial.com/install-postgresql/. Toma nota del directorio de instalación ya que lo necesitarás en el siguiente paso (usualmente es `C:\Program Files\PostgreSQL\9.3`).

## Mac OS X

La forma mas fácil es descargar [Postgres.app](http://postgresapp.com/) e instalarla como cualquier otra aplicación en tu sistema operativo.

Descárgala, arrástrala al directorio Aplicaciones y ejecútala con doble click. ¡Eso es todo!

También vas a tener que agregar las herramientas de linea de comandos de Postgres a tu variable `PATH`, lo que se explica [aquí](http://postgresapp.com/documentation/cli-tools.html).

## Linux

Los pasos de instalación varían de acuerdo a tu distribución. Abajo están los comandos para Ubuntu y Fedora, pero si estas usando una distribución diferente [Revisa la documentación de PostgreSQL](https://wiki.postgresql.org/wiki/Detailed_installation_guides#General_Linux).

### Ubuntu

Ejecuta el siguiente comando:

    sudo apt-get install postgresql postgresql-contrib

### Fedora

Ejecuta el siguiente comando:

    sudo yum install postgresql93-server

# Crear la base de datos

A continuación, necesitamos crear nuestra primera base de datos y un usuario que pueda acceder a esa base de datos. PostgreSQL te permite crear tantas bases de datos y usuarios como gustes, así que si estas corriendo mas de un sitio deberías crear una base de datos para cada uno.

## Windows

If you're using Windows, there's a couple more steps we need to complete. For now it's not important for you to understand the configuration we're doing here, but feel free to ask your coach if you're curious as to what's going on.
Si estas usando Windows, hay un par de pasos más que necesitas completar. Por ahora esto no es importante para que entiendas la configuración que estamos haciendo aquí. pero sientete libre de preguntar a tu guia si tienes curiosidad por saber que esta pasando.

1. Abre el Símbolo del Sistema (Menú inicio → Todos los programas → Accesorios → Simbolo del Sistema)
2. Ejecuta los siguiente escribiendo y presionando la tecla `Enter`: `setx PATH "%PATH%;C:\Program Files\PostgreSQL\9.3\bin"`. Puedes pegar esto en el Simbolo del Sistema haciendo click derecho y seleccionando `Pegar`. Asegúrate de que la dirección del directorio es igual a la que tomaste nota durante la instalación más el añadido `\bin` al final. Deberías ver el mensaje `EXITO: El valor específico fue guardado`.
3. Cierra y vuelve a abrir el Simbolo de Sistema.

## Crear la base de datos

Primero, vamos a lanzar la consola de Postgres ejecutando `psql`. ¿Recuerdas como lanzar la consola?
> En Mac OS X puedes hacer esto lanzando la aplicación `Terminal` (está en Aplicaciones → Utilidades). En Linux, está probablemente en Aplicaciones → Accesorios → Terminal. En Windows necesitas ir a Menú inicio → Todos los programas → Accesorios → Simbolo del Sistema. Además, en Windows, `psql` podría requerir iniciar sesión usando el usuario y contraseña que elegiste durante la instalación. Si `psql` pregunta por tu contraseña y parece no funcionar, prueba `psql -U <usuario> -W` presiona Enter e ingresa tu contraseña.

    $ psql
    psql (9.3.4)
    Type "help" for help.
    #

Nuestro `$` ahora cambió a `#` lo cual significa que ahora estamos enviando comandos a PostgreSQL. Vamos a crear un usuario:

    # CREATE USER name;
    CREATE ROLE

Reemplaza `name` con tu propio nombre. No deberías usar letras con acentos o espacios en blanco (ejemplo: `bożena maria` es inválido - necesitas convertirlo a `bozena_maria`).

Ahora es tiempo de crear una base de datos para tu proyecto Django:

    # CREATE DATABASE djangogirls OWNER name;
    CREATE DATABASE

Recuerda reemplazar `name` con el nombre que elegiste (ejemplo: `bozena_maria`).

Genial - ¡El tema bases de datos ya está listo!

# Actualizando la configuración de django

Encuentra esta parte en tu archivo `mysite/settings.py`:

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}
```

Y reemplázala con esto:

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'djangogirls',
        'USER': 'name',
        'PASSWORD': '',
        'HOST': 'localhost',
        'PORT': '',
    }
}
```

Recuerda cambiar `name` por el nombre de usuario que creaste anteriormente en este capítulo.

# Instalando el paquete de PostgreSQL para Python

Primero, instala Heroku Toolbelt desde https://toolbelt.heroku.com/. Vamos a necesitarlo para desplegar tu sitio mas adelante, esto incluye Git, lo que podría ser útil ahora.

Luego, necesitamos instalar un paquete el cual permite a Python hablar con PostgreSQL - este se llama `psycopg2`. Las instrucciones de instalación difieren levemente entre Windows y Linux/OS X.

## Windows

Para Windows, descarga el archivo pre-construido desde http://www.stickpeople.com/projects/python/win-psycopg/

Asegúrate de obtener el archivo correspondiente a tu versión de Python (3.4 debería ser la última linea) y la correcta arquitectura (32 bits en la columna izquierda o 64 bits en la columna derecha).

Renombra el archivo descargado y muévelo de modo que este disponible en `C:\psycopg2.exe`.

Una vez que este hecho, ingresa el siguiente comando en la terminal (asegúrate que tu `entorno virtual` este activado):

    easy_install C:\psycopg2.exe

## Linux y OS X

Ejecuta el siguiente comando en tu consola:

    (myvenv) ~/djangogirls$ pip install psycopg2

Si todo va bien, verás algo como esto

    Downloading/unpacking psycopg2
    Installing collected packages: psycopg2
    Successfully installed psycopg2
    Cleaning up...

---

Una vez que este completo, ejecuta `python -c "import psycopg2"`. Si no obtienes ningún error, todo esta instalado satisfactoriamente.
