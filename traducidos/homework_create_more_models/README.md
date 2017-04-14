# Deberes: crear el modelo Comment

Hasta aquí solo tenemos un modelo Post, pero ¿qué hay de recibir algunos comentarios de tus lectores?

## Creando el modelo de comentarios del blog

Vamos a abrir `blog/models.py` y agregar este fragmento de código al final del archivo:

```python
class Comment(models.Model):
    post = models.ForeignKey('blog.Post', related_name='comments')
    author = models.CharField(max_length=200)
    text = models.TextField()
    created_date = models.DateTimeField(default=timezone.now)
    approved_comment = models.BooleanField(default=False)

    def approve(self):
        self.approved_comment = True
        self.save()

    def __str__(self):
        return self.text
```

Puedes volver al capítulo **Modelos en Django** si necesitas recordar lo que significa cada tipo de campo.

En este capítulo tenemos un nuevo tipo de campo:
- `models.BooleanField` - este es un campo verdadero/falso.

Además la opción `related_name` en `models.ForeignKey` nos permite tener acceso a los comentarios desde el modelo post.

## Crear tablas para los modelos en tu base de datos

Ahora es momento de agregar el modelo de comentarios a la base de datos. Para esto tenemos que hacerle conocer a Django que hicimos cambios en nuestro modelo. Escribe `python manage.py makemigrations blog`. De esta forma:

    (myvenv) ~/djangogirls$ python manage.py makemigrations blog
    Migrations for 'blog':
      0002_comment.py:
        - Create model Comment

Puedes ver que este comando crea por nosotros otro archivo de migración en el directorio `blog/migrations/`. Ahora necesitamos aplicar estos cambios con `python manage.py migrate blog`. Debería verse así:

    (myvenv) ~/djangogirls$ python manage.py migrate blog
    Operations to perform:
      Apply all migrations: blog
    Running migrations:
      Rendering model states... DONE
      Applying blog.0002_comment... OK

Nuestro modelo Comment ahora existe en la base de datos. Sería lindo si tenemos acceso a él en nuestro panel de administración.

## Registrar el modelo comment en el panel de administración

Para registrar el modelo en el panel de administración ve a `blog/admin.py` y agrega la línea:

```python
admin.site.register(Comment)
```

No olvides de importar el modelo Comment, el archivo debería verse como esto:

```python
from django.contrib import admin
from .models import Post, Comment

admin.site.register(Post)
admin.site.register(Comment)
```

Si escribes `python manage.py runserver` en el símbolo de sistema y vas con el navegador a [http://127.0.0.1:8000/admin/](http://127.0.0.1:8000/admin/) deberías tener acceso a listar, agregar y eliminar comentarios. No dudes en jugar con esto.

## Hacer visible nuestros comentarios

Ve al archivo `blog/templates/blog/post_detail.html` y agrega las líneas previas al tag `{% endblock %}`:

```django
<hr>
{% for comment in post.comments.all %}
    <div class="comment">
        <div class="date">{{ comment.created_date }}</div>
        <strong>{{ comment.author }}</strong>
        <p>{{ comment.text|linebreaks }}</p>
    </div>
{% empty %}
    <p>No comments here yet :(</p>
{% endfor %}
```

Ahora podremos ver la sección de comentarios en las páginas con detalles del post.

Pero esto puede verse algo mejor, agrega algo de css a `static/css/blog.css`:

```css
.comment {
    margin: 20px 0px 20px 20px;
}
```

Podemos también permitir a los visitantes conocer sobre los comentarios en la página de listado de publicaciones. Vea al archivo `blog/templates/blog/post_list.html` y agrega la línea:

```django
<a href="{% url 'blog.views.post_detail' pk=post.pk %}">Comments: {{ post.comments.count }}</a>
```

Luego de esto nuestra plantilla debería verse así:

```django
{% extends 'blog/base.html' %}

{% block content %}
    {% for post in posts %}
        <div class="post">
            <div class="date">
                {{ post.published_date }}
            </div>
            <h1><a href="{% url 'blog.views.post_detail' pk=post.pk %}">{{ post.title }}</a></h1>
            <p>{{ post.text|linebreaks }}</p>
            <a href="{% url 'blog.views.post_detail' pk=post.pk %}">Comments: {{ post.comments.count }}</a>
        </div>
    {% endfor %}
{% endblock content %}
```

## Deja que tus lectores escriban comentarios

Ahora mismo podemos ver comentarios en nuestro blog pero no podemos agregarlos, ¡vamos a cambiar esto!

Ve a `blog/forms.py` y agrega estas líneas al final del archivo:

```python
class CommentForm(forms.ModelForm):

    class Meta:
        model = Comment
        fields = ('author', 'text',)
```

No olvides importar el modelo Comment, cambia la línea:

```python
from .models import Post
```

por:

```python
from .models import Post, Comment
```

Ahora ve a `blog/templates/blog/post_detail.html` y antes de la línea `{% for comment in post.comments.all %}` agrega:

```django
<a class="btn btn-default" href="{% url 'add_comment_to_post' pk=post.pk %}">Add comment</a>
```

Ve a la página de detalles del post y deberías ver el error:

![NoReverseMatch](images/url_error.png)

Vamos a solucionarlo. Ve a `blog/urls.py` y agrega este patrón a `urlpatterns`:

```python
url(r'^post/(?P<pk>[0-9]+)/comment/$', views.add_comment_to_post, name='add_comment_to_post'),
```

Ahora deberías ver este error:

![AttributeError](images/views_error.png)

Para arreglarlo, agrega este fragmento de código a `blog/views.py`:

```python
def add_comment_to_post(request, pk):
    post = get_object_or_404(Post, pk=pk)
    if request.method == "POST":
        form = CommentForm(request.POST)
        if form.is_valid():
            comment = form.save(commit=False)
            comment.post = post
            comment.save()
            return redirect('blog.views.post_detail', pk=post.pk)
    else:
        form = CommentForm()
    return render(request, 'blog/add_comment_to_post.html', {'form': form})
```

No olvides los imports al comienzo del archivo:

```python
from .forms import PostForm, CommentForm
```


Ahora, en la página de detalles de la publicación, deberías ver el botón "Agregar Comentario"

![AddComment](images/add_comment_button.png)

Sin embargo, cuando hagas click en el botón deberías ver:

![TemplateDoesNotExist](images/template_error.png)


Tal como menciona el error, la plantilla no existe, crea una como `blog/templates/blog/add_comment_to_post.html` y agrega estas líneas:

```django
{% extends 'blog/base.html' %}

{% block content %}
    <h1>New comment</h1>
    <form method="POST" class="post-form">{% csrf_token %}
        {{ form.as_p }}
        <button type="submit" class="save btn btn-default">Send</button>
    </form>
{% endblock %}
```

¡Sí! ¡Ahora tus lectores pueden hacerte saber qué es lo que piensan de tus publicaciones!

## Moderar tus comentarios

No todos los comentarios deberían ser mostrados. El dueño del blog debería tener la opción de aprobar o eliminar comentarios. Vamos a hacer algo sobre esto.

Ve a `blog/templates/blog/post_detail.html` y cambia las líneas:

```django
{% for comment in post.comments.all %}
    <div class="comment">
        <div class="date">{{ comment.created_date }}</div>
        <strong>{{ comment.author }}</strong>
        <p>{{ comment.text|linebreaks }}</p>
    </div>
{% empty %}
    <p>No comments here yet :(</p>
{% endfor %}
```

por:

```django
{% for comment in post.comments.all %}
    {% if user.is_authenticated or comment.approved_comment %}
    <div class="comment">
        <div class="date">
            {{ comment.created_date }}
            {% if not comment.approved_comment %}
                <a class="btn btn-default" href="{% url 'comment_remove' pk=comment.pk %}"><span class="glyphicon glyphicon-remove"></span></a>
                <a class="btn btn-default" href="{% url 'comment_approve' pk=comment.pk %}"><span class="glyphicon glyphicon-ok"></span></a>
            {% endif %}
        </div>
        <strong>{{ comment.author }}</strong>
        <p>{{ comment.text|linebreaks }}</p>
    </div>
    {% endif %}
{% empty %}
    <p>No comments here yet :(</p>
{% endfor %}
```

Deberías ver `NoReverseMatch`, porque no existe una url que coincida con el patrón `comment_remove`y `comment_approve`

Agrega el patrón de url a `blog/urls.py`:

```python
url(r'^comment/(?P<pk>[0-9]+)/approve/$', views.comment_approve, name='comment_approve'),
url(r'^comment/(?P<pk>[0-9]+)/remove/$', views.comment_remove, name='comment_remove'),
```

Ahora deberías ver `AttributeError`. Para deshacerte de esto, crea mas vistas en `blog/views.py`:

```python
@login_required
def comment_approve(request, pk):
    comment = get_object_or_404(Comment, pk=pk)
    comment.approve()
    return redirect('blog.views.post_detail', pk=comment.post.pk)

@login_required
def comment_remove(request, pk):
    comment = get_object_or_404(Comment, pk=pk)
    post_pk = comment.post.pk
    comment.delete()
    return redirect('blog.views.post_detail', pk=post_pk)
```

Y por supuesto arregla los imports.

Todo funciona, pero hay un error. En nuestra página de listado de publicaciones bajo la publicación podemos ver el número de todos los comentarios agregados, pero ahí queremos tener el número de comentarios aprobados.

Ve a `blog/templates/blog/post_list.html` y modifica la línea:

```django
<a href="{% url 'blog.views.post_detail' pk=post.pk %}">Comments: {{ post.comments.count }}</a>
```

por:

```django
<a href="{% url 'blog.views.post_detail' pk=post.pk %}">Comments: {{ post.approved_comments.count }}</a>
```

Y también agrega este método al modelo Post en `blog/models.py`:

```python
def approved_comments(self):
    return self.comments.filter(approved_comment=True)
```

Ahora la funcionalidad sobre comentarios está finalizada! Felicitaciones :-)
