# Tarea: Añadiendo seguridad a tu sitio web

Te habrás dado cuenta de que no has tenido que usar ninguna clave, excepto cuando, con anterioridad, usamos la interfaz de administración. Te habrás dado cuenta también de que esto significa que cualquiera puede añadir o editar entradas (posts) en tu blog. Yo no se tú, pero yo no querría que cualquiera publicase en mi blog. Así que hagamos algo al respecto.

## Autorización para añadir/editar entradas en el blog

Primero hagamos las cosas seguras. Protegeremos nuestras vistas `post_new`, `post_edit`, `post_draft_list`, `post_remove` y `post_publish` para que solamente usuarios registrados puedan acceder a ellas. Django viene con algunas herramientas para ello, son elementos bastante avanzados llamados _decoradores_ (decorators) . No te preocupes por tecnicismos ahora, puedes leer más sobre este tema después. El decorador que debemos usar viene en el módulo `django.contrib.auth.decorators` y se llama `login_required`.

Así que edita el archivo `blog/views.py` y añade estas líneas al principio junto al resto de los _imports_:

```python
from django.contrib.auth.decorators import login_required
```

Después añade una línea antes de cada vista `post_new`, `post_edit`, `post_draft_list`, `post_remove` y `post_publish` (decorándolas) como la siguiente:

```python
@login_required
def post_new(request):
    [...]
```

¡Y ya está! Ahora intenta acceder a `http://localhost:8000/post/new/`, ¿ves la diferencia?

> Si te ha salido un formulario vacío, seguramente estés autenticada desde el capítulo de la interfaz de administración. Ve a `http://localhost:8000/admin/logout/` para salir (log out), después ve a `http://localhost:8000/post/new` otra vez.

Deberías obtener uno de nuestros queridos errores. Este, de hecho, es bastante interesante: El decorador que añadimos antes te redirigirá a la pantalla de autenticación. Pero todavía no está disponible, así que recibirás un "Página no encontrada (404)".

No olvides añadir el decorador encima de `post_edit`, `post_remove`, `post_draft_list` y `post_publish` también.

Hurra, ¡hemos alcanzado parte de nuestro objetivo! Otras personas ya no podrán crear entradas en nuestro blog. Desafortunadamente nosotros tampoco podremos. Así que arreglemos esto ahora mismo.

## Autenticación de usuario

Ahora podríamos intentar hacer un montón de magia para implementar usuarios, claves y autenticación, pero hacer este tipo de cosas correctamente es bastante complicado. Como Django viene con las "pilas incluidas", alguien ha hecho este trabajo duro por nosotros, así que haremos uso de estas herramientas de autenticación proporcionadas.

En tu archivo `mysite/urls.py` añade la url `url(r'^accounts/login/$', 'django.contrib.auth.views.login')`. De manera que ahora el archivo se parezca a esto:

```python
from django.conf.urls import patterns, include, url

from django.contrib import admin
admin.autodiscover()

urlpatterns = patterns('',
    url(r'^admin/', include(admin.site.urls)),
    url(r'^accounts/login/$', 'django.contrib.auth.views.login'),
    url(r'', include('blog.urls')),
)
```

Ahora necesitamos una plantilla para la página de autenticación o inicio de sesión, así que creamos un directorio `blog/templates/registration` y un archivo dentro llamado `login.html`:

```django
{% extends "blog/base.html" %}

{% block content %}

{% if form.errors %}
<p>Usuario o clave correctos. Vuelve a intentarlo, por favor.</p>
{% endif %}

<form method="post" action="{% url 'django.contrib.auth.views.login' %}">
{% csrf_token %}
<table>
<tr>
    <td>{{ form.username.label_tag }}</td>
    <td>{{ form.username }}</td>
</tr>
<tr>
    <td>{{ form.password.label_tag }}</td>
    <td>{{ form.password }}</td>
</tr>
</table>

<input type="submit" value="login" />
<input type="hidden" name="next" value="{{ next }}" />
</form>

{% endblock %}
```

Verás que esto hace uso de nuestra plantilla base para que tenga el aspecto general de tu blog.

Lo maravilloso aquí es que esto simplemente funciona. No tenemos que ocuparnos del envío seguro de formularios y de claves. Sólo nos queda una cosa, deberíamos añadir una configuración a `mysite/settings.py`:

```python
LOGIN_REDIRECT_URL = '/'
```

Ahora cuando se accede directamente a la página de inicio de sesión, si el usuario se autentica con éxito, se le redirigirá a la raíz del sitio (/).

## Mejorando el diseño

Así que ahora nos hemos asegurado de que solo usuarios autorizados (ie. nosotros) pueden añadir, editar o publicar entradas en el blog. Pero los botones de añadir y editar entradas pueden ser vistos todavía por todo el mundo, escondámoslos para los usuarios no autenticados. Para esto necesitamos editar las plantillas, así que empecemos con la plantilla base `blog/templates/blog/base.html`:

```django
<body>
    <div class="page-header">
        {% if user.is_authenticated %}
        <a href="{% url 'post_new' %}" class="top-menu"><span class="glyphicon glyphicon-plus"></span></a>
        <a href="{% url 'post_draft_list' %}" class="top-menu"><span class="glyphicon glyphicon-edit"></span></a>
        {% else %}
        <a href="{% url 'django.contrib.auth.views.login' %}" class="top-menu"><span class="glyphicon glyphicon-lock"></span></a>
        {% endif %}
        <h1><a href="{% url 'blog.views.post_list' %}">Django Girls</a></h1>
    </div>
    <div class="content">
        <div class="row">
            <div class="col-md-8">
            {% block content %}
            {% endblock %}
            </div>
        </div>
    </div>
</body>
```

Tal vez reconozcas el patrón aquí. Hay una condición if dentro de la plantilla que comprueba que el usuario está autenticado para mostrar los botones de editar. De lo contrario muestra el botón de inicio de sesión (login).

*Deberes*: Edita la plantilla `blog/templates/blog/post_detail.html` para mostrar solamente los botones de editar a los usuarios autenticados.

## Más sobre usuarios autenticados

Añadamos un poco de sal y pimienta a nuestras plantillas mientras estemos en ellas. Primero añadiremos un indicador de que estamos autenticados. Edita `blog/templates/blog/base.html` de la siguiente manera:

```django
<div class="page-header">
    {% if user.is_authenticated %}
    <a href="{% url 'post_new' %}" class="top-menu"><span class="glyphicon glyphicon-plus"></span></a>
    <a href="{% url 'post_draft_list' %}" class="top-menu"><span class="glyphicon glyphicon-edit"></span></a>
    <p class="top-menu">Hola {{ user.username }}<small>(<a href="{% url 'django.contrib.auth.views.logout' %}">Log out</a>)</small></p>
    {% else %}
    <a href="{% url 'django.contrib.auth.views.login' %}" class="top-menu"><span class="glyphicon glyphicon-lock"></span></a>
    {% endif %}
    <h1><a href="{% url 'blog.views.post_list' %}">Django Girls</a></h1>
</div>
```

Esto añade un bonito "Hola &lt;nombre-de-usuario&gt;" para recordarnos quien somos y que estamos autenticados. También añade un link para cerrar la sesión. Pero como te habrás dado cuenta, esto todavía no funciona. ¡Ups! ¡Hemos roto internet! ¡Arreglémosla!

Hemos decidido dejar que Django gestione el inicio de sesión, veamos si Django puede gestionar también el cierre de sesión por nosotros. Echa un vistazo en https://docs.djangoproject.com/en/1.8/topics/auth/default/ a ver si puedes encontrar algo.

¿Terminaste de leer? Deberías estar pensando en añadir una url (en `mysite/urls.py`) apuntando a la vista `django.contrib.auth.views.logout`. Como esta:

```python
from django.conf.urls import patterns, include, url

from django.contrib import admin
admin.autodiscover()

urlpatterns = patterns('',
    url(r'^admin/', include(admin.site.urls)),
    url(r'^accounts/login/$', 'django.contrib.auth.views.login'),
    url(r'^accounts/logout/$', 'django.contrib.auth.views.logout', {'next_page': '/'}),
    url(r'', include('blog.urls')),
)
```

¡Ya está! Si has seguido todas las instrucciones hasta ahora (y hecho los deberes), tienes un blog donde:

 - necesitas un usuario y una clave para iniciar sesión y autenticarte,
 - necesitas estar autenticado para añadir/editar/publicar(/borrar) entradas
 - y puedes cerrar la sesión de nuevo
