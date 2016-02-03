# Despliegue en Heroku (así como en PythonAnyhere)

Siempre es bueno para un desarrollador tener un par de opciones diferentes de despliegue en su haber. ¿Por qué no intentar desplegar tu sitio web en Heroku, así como en PythonAnywhere?

[Heroku](http://heroku.com/) es también gratuito para pequeñas aplicaciones que no tienen muchas visitas, pero un poco más complicado para desplegar.

Seguiremos este tutorial: https://devcenter.heroku.com/articles/getting-started-with-django, pero lo pegamos aquí por si es más fácil para ti.


## El archivo `requirements.txt`

Si no lo creaste antes, necesitamos crear ahora un archivo `requirements.txt` para decirle a Heroku qué paquetes de Python necesitan estar instalados en nuestro servidor.

Pero primero, Heroku necesita que nosotros instalemos algunos paquetes nuevos. Ve a la consola con tu `virtualenv` (entorno virtual) activado y escribe lo siguiente:

    (myvenv) $ pip install dj-database-url gunicorn whitenoise

Cuando finalice la instalación, ve al directorio `djangogirls` y ejecuta este comando:

    (myvenv) $ pip freeze > requirements.txt

Esto creará un archivo llamado `requirements.txt` con una lista de tus paquetes instalados (i.e. bibliotecas de Python que estás usando, por ejemplo Django :)).

> __Nota__: `pip freeze` lista todas las bibliotecas Python instaladas en tu virtualenv, y el `>` toma la salida de `pip freeze` y la pone en un archivo. ¡Intenta ejecutar `pip freeze` sin el `> requirements.txt` para ver qué pasa!

Abre este archivo y añádele la siguiente línea al final:

    psycopg2==2.5.4

Esta línea se necesita para que tu aplicación funcione en Heroku.


## Procfile

Otro archivo que Heroku necesita es el Procfile. Éste le dice a Heroku qué comandos ejecutar para iniciar nuestro sitio web. Abre el editor de código, crea un archivo llamado `Procfile` en el directorio `djangogirls` y añade esta línea:

    web: gunicorn mysite.wsgi

Esta línea significa que vamos a desplegar una aplicación `web`, y que lo haremos ejecutando el comando `gunicorn mysite.wsgi` (`gunicorn` es como una versión más potente del comando `runserver` de Django).

Ahora guárdalo. ¡Hecho!

## El archivo `runtime.txt`

También necesitamos decirle a Heroku qué versión de Python queremos usar. Esto se hace creando un archivo `runtime.txt` en el directorio `djangogirls` usando el comando "nuevo archivo" de tu editor, y poniendo el siguiente texto (¡y nada más!) dentro:

    python-3.4.2

## `mysite/local_settings.py`

Como es más restrictivo que PythonAnywhere, Heroku necesita usar una configuración diferente de la que usamos localmente (en nuestro ordenador). Por ejemplo, Heroku quiere usar Postgres mientras que nosotros usamos SQLite. Por eso necesitamos crear un archivo de configuración separado que estará sólo disponible para nuestro entorno local.

Continúa y crea el archivo `mysite/local_settings.py`. Debería contener la configuración para `DATABASE` de tu archivo `mysite/settings.py`. Así:

```python
import os
BASE_DIR = os.path.dirname(os.path.dirname(__file__))

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}

DEBUG = True
```

Ahora ¡sólo guárdalo! :) 

## mysite/settings.py

Otra cosa que necesitamos hacer es modificar el archivo `settings.py` de nuestro sitio web. Abre `mysite/settings.py` en tu editor y añade las siguientes líneas al final del archivo:

```python
import dj_database_url
DATABASES['default'] = dj_database_url.config()

SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')

ALLOWED_HOSTS = ['*']

DEBUG = False

try:
    from .local_settings import *
except ImportError:
    pass
```

Esto hará la configuración necesaria para Heroku y también importará tu configuración local si existe el archivo `mysite/local_settings.py`.

Guarda el archivo.

## mysite/wsgi.py

Abre el archivo `mysite/wsgi.py` y añade estas líneas al final:

```python
from whitenoise.django import DjangoWhiteNoise
application = DjangoWhiteNoise(application)
```

¡Perfecto!

## Cuenta en Heroku

Necesitas instalar la herramienta Heroku *toolbelt* que puedes encontrar aquí (puedes saltarte la instalación si ya lo has hecho durante la configuración inicial): https://toolbelt.heroku.com/

> Si ejecutas el programa de instalación de Heroku *toolbelt* en Windows asegúrate de elegir "Custom Installation" cuando te pregunte qué componentes instalar. En la lista de componentes que aparece después añade a la selección "Git and SSH".

> En Windows también debes ejecutar la siguiente instrucción para añadir Git y SSH al `PATH` de tu ventana de comandos: `setx PATH "%PATH%;C:\Program Files\Git\bin"`. Reinicia tu ventana de comandos para habilitar el cambio.

> Después de reiniciar tu ventana de comandos, ¡no olvides ir a tu carpeta `djangogirls` otra vez y activar tu entorno virtual! (Pista: [Comprueba el capítulo de instalación de Django](../django_installation/README.md#working-with-virtualenv))

Crea también, por favor, una cuenta gratuita de Heroku aquí: https://id.heroku.com/signup/www-home-top

Luego autentica tu cuenta de Heroku en tu ordenador ejecutando el siguiente comando:

    $ heroku login

En caso de que no tengas una clave SSH, este comando generará una automáticamente. Las claves SSH se necesitan para hacer *push* de código a Heroku, es decir, enviar tu código al servidor.

## Git commit

Heroku usa git para sus despliegues. A diferencia de PythonAnywhere, puedes hacer un *push* a Heroku directamente, sin utilizar Github como intermediario. Pero necesitamos hacer unas pequeñas modificaciones antes.

Abre el archivo llamado `.gitignore` en tu directorio `djangogirls` y añade `local_settings.py` al mismo. Queremos que git ignore `local_settings`, para que se quede en nuestro ordenador local y no sea subido a Heroku.

    *.pyc
    db.sqlite3
    myvenv
    __pycache__
    local_settings.py


Y hacemos un commit de nuestros cambios

    $ git status
    $ git add -A .
    $ git commit -m "archivos adicionales y cambios para Heroku"


## Selecciona un nombre para la aplicación

Estamos haciendo nuestro blog disponible en la Web en `[nombre de tu blog].herokuapp.com`, por lo que necesitamos seleccionar un nombre que nadie más esté usando. Este nombre no tiene que estar relacionado con nuestra aplicación `blog` de Django o con `mysite` ni con nada de lo que hemos creado hasta ahora. El nombre puede ser cualquier cosa que tú quieras, pero Heroku es bastante estricto con respecto a los caracteres que puedes usar: sólo se permiten letras minúsculas simples (no mayúsculas ni acentos), números y guiones (`-`).

Una vez que hayas pensado en un nombre (quizás algo con tu nombre o tu apodo), ejecuta este comando, reemplazando `djangogirlsblog` con el nombre seleccionado:

    $ heroku create djangogirlsblog

> __Nota__: Recuerda reemplazar `djangogirlsblog` con el nombre de tu aplicación en Heroku.

Si no se te ocurre ningún nombre, también puedes ejecutar simplemente:

    $ heroku create

y Heroku elegirá un nombre no usado para ti (probablemente algo como `enigmatic-cove-2527`).

Si en algún momento te apetece cambiar el nombre de tu aplicación en Heroku, puedes hacerlo con este comando (reemplaza `nombre-nuevo` con el nuevo nombre que quieras usar):

    $ heroku apps:rename nombre-nuevo

> __Nota__: Recuerda que después de cambiar el nombre de la aplicación, tienes que visitar `[nombre-nuevo].herokuapp.com` para ver tu sitio web.

## ¡Desplegar en Heroku!

Eso fue un montón de configuración e instalación, ¿verdad? ¡Pero sólo necesitas hacerlo una vez! ¡Ahora ya puedes desplegar!

Cuando ejecutaste `heroku create`, automáticamente se añadió el Heroku *remote* para la aplicación en nuestro repositorio. Ahora podemos hacer un simple git push para desplegar nuestra aplicación:

    $ git push heroku master

> __Nota__: Esto probablemente producirá un *montón* de salida por consola la primera vez que lo ejecutas, ya que Heroku compila e instala psycopg. Sabrás que has tenido éxito si ves algo como `https://tunombredeaplicacion.herokuapp.com/  deployed to Heroku` hacia el final del texto.

## Visita tu aplicación

Has desplegado tu código en Heroku, y especificado el tipo de proceso en un `Procfile` (nosotros elegimos tipo de proceso `web`). Ahora podemos decirle a Heroku que inicie este `proceso web`.

Para hacerlo, ejecuta el siguiente comando:

    $ heroku ps:scale web=1

Este comando le dice a Heroku que ejecute solamente una instancia de nuestro proceso `web`. Ya que nuestra aplicación de blog es bastante simple, no necesitamos demasiada potencia y por tanto está bien ejecutar solamente un proceso. Es posible pedirle a Heroku que ejecute más procesos (por cierto, Heroku le llama a estos procesos "Dynos" así que no te sorprendas si ves ese nombre) pero entonces dejaría de ser gratuito.

Ahora podemos visitar la aplicación en nuestro navegador con `heroku open`.

    $ heroku open

> __Nota__: ¡Verás una pagina de error! Hablaremos de eso en un minuto.

Esto abrirá una url como [https://djangogirlsblog.herokuapp.com/]() en tu navegador, y al momento verás probablemente una página de error.

El error que ves es porque cuando desplegamos en Heroku creamos una nueva base de datos que está vacía. Necesitamos ejecutar los comandos `migrate` y `createsuperuser`, tal como hicimos en PythonAnywhere. Esta vez, se ejecutan vía una línea de comandos especial en nuestro ordenador, `heroku run`:

    $ heroku run python manage.py migrate

    $ heroku run python manage.py createsuperuser

Este comando te pedirá que elijas un nombre de usuario y password otra vez. Éstas serán tus credenciales en la página de administración de tu sitio web.


Refresca tu navegador, y ¡ahí está! Ahora ya sabes como desplegar en dos plataformas de hosting diferentes. Elige tu favorita :)

