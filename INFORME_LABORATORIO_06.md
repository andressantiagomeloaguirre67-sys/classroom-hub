# Informe de Desarrollo y Resolución de Casos — Laboratorio 6: Fundamentos de Django y primera aplicación web (ClassroomHub)

- **Autor:** Andrés Santiago Melo Aguirre (`andressantiagomeloaguirre67-sys`)
- **Repositorio en GitHub:** [https://github.com/andressantiagomeloaguirre67-sys/classroom-hub](https://github.com/andressantiagomeloaguirre67-sys/classroom-hub)
- **Tecnologías utilizadas:** Python 3.14.8, Django 6.1.3, HTML5/CSS3, Git y GitHub.

---

## Parte I — Informe de lo Realizado en el Laboratorio

### 1. Introducción y Objetivo

En este laboratorio se desarrolló **ClassroomHub**, un portal web de recursos de aprendizaje construido con **Django**, con el propósito de comprender y aplicar de forma práctica la arquitectura **Modelo–Plantilla–Vista (MTV)** y el flujo completo de una solicitud web:

`Solicitud HTTP (URL) → Enrutador (urls.py) → Vista (views.py) → Plantilla (HTML) → Respuesta HTTP`

En esta primera etapa no se implementaron bases de datos propias, formularios, autenticación ni operaciones CRUD, enfocando el trabajo en la separación de responsabilidades entre rutas, vistas y plantillas reutilizables.

### 2. Actividades Realizadas Paso a Paso

1. **Entorno virtual y dependencias (Práctica 6.2 — Commit `ba2a51f`):**
   - Se creó el entorno virtual aislado con `python -m venv .venv` y se activó mediante `source .venv/bin/activate`.
   - Se instaló `Django==6.1.3` sobre Python 3.14.8 y se congelaron las dependencias en `requirements.txt` (`asgiref==3.13.0`, `Django==6.1.3`, `sqlparse==0.6.1`).
   - Se creó `.gitignore` excluyendo `.venv/`, `__pycache__/`, `*.py[cod]`, `*.log`, `.env` y `db.sqlite3`.
2. **Proyecto y aplicación `pages` (Práctica 6.3 y 6.4 — Commit `5319b73`):**
   - Se generó la configuración principal con `python -m django startproject config .` y la aplicación con `python manage.py startapp pages`.
   - Se registró `'pages'` en `INSTALLED_APPS` dentro de `config/settings.py`.
   - Se configuró el enrutador principal `config/urls.py` con `path("", include("pages.urls"))` y el enrutador de la aplicación `pages/urls.py` con `app_name = "pages"` y las rutas `home` (`""`), `resources` (`"resources/"`) y `about` (`"about/"`).
3. **Plantillas HTML, herencia y contexto (Práctica 6.5 — Commit `d3fbc16`):**
   - Se creó el directorio `pages/templates/pages/` con la plantilla maestra `base.html` y las plantillas hijas `home.html`, `resources.html` y `about.html`.
   - Se actualizaron las vistas en `pages/views.py` para usar `render(request, template_name, context)` enviando diccionarios de contexto a cada plantilla.
4. **Pruebas automatizadas y documentación (Práctica 6.6 — Commits `b7ee323` y `55df9a1`):**
   - Se implementaron 5 pruebas automatizadas en `pages/tests.py` con `SimpleTestCase` y `reverse()`.
   - Se documentó la instalación, ejecución y alcance en `README.md` y se publicó el repositorio en [https://github.com/andressantiagomeloaguirre67-sys/classroom-hub](https://github.com/andressantiagomeloaguirre67-sys/classroom-hub).

---

## Parte II — Preguntas de Análisis y Depuración

### 1. ¿Qué harías si el servidor de Django inicia correctamente, pero al acceder a `/resources/` aparece un error HTTP 404?

- **Qué haría:**
  1. Primero verificaría en la terminal qué proyecto y puerto está ejecutando `runserver` (por ejemplo, que no haya otro proyecto de Django ocupando el puerto `8000`) y que la URL escrita en el navegador sea exactamente `/resources/` con barra final.
  2. Luego inspeccionaría la cadena de enrutamiento en orden: primero `config/urls.py`, después `pages/urls.py` y finalmente la función asociada en `pages/views.py`.
- **Por qué:**
  Un código HTTP `404 Not Found` indica que el servidor está activo pero el *URL dispatcher* de Django recorrió `urlpatterns` desde `ROOT_URLCONF` (`config.urls`) sin encontrar ningún patrón que coincida con `/resources/`. Esto ocurre si falta `include("pages.urls")` en el proyecto principal, si el prefijo es distinto, o si el patrón `"resources/"` está mal escrito o ausente en `pages/urls.py`.
- **Cómo lo implementaría:**
  1. En `config/urls.py`, confirmaría que la raíz delegue las rutas a la aplicación `pages` mediante `include()`:
     ```python
     urlpatterns = [
         path("admin/", admin.site.urls),
         path("", include("pages.urls")),
     ]
     ```
  2. En `pages/urls.py` (introducido en el commit `5319b73`), verificaría que el patrón no lleve barra inicial (`"resources/"` y no `"/resources/"`) y que apunte a la función `views.resources`:
     ```python
     app_name = "pages"

     urlpatterns = [
         path("", views.home, name="home"),
         path("resources/", views.resources, name="resources"),
         path("about/", views.about, name="about"),
     ]
     ```
- **Cómo comprobaría el resultado:**
  - Manualmente: abriendo `http://127.0.0.1:8000/resources/` en el navegador y verificando que cargue con estado `200 OK`.
  - Automatizadamente: ejecutando `python manage.py test pages` para validar que `test_resources_page_shows_example_data` en `pages/tests.py` resuelva `reverse("pages:resources")` y devuelva `status_code == 200`.

---

### 2. ¿Qué harías si una vista está correctamente definida, pero Django presenta un error `TemplateDoesNotExist` al intentar mostrar una página?

- **Qué haría:**
  Revisaría tres puntos clave en orden:
  1. La ruta y nombre exacto del archivo pasado al segundo argumento de `render()` en `pages/views.py`.
  2. La estructura de carpetas dentro de la aplicación (`pages/templates/pages/`).
  3. La configuración de `INSTALLED_APPS` y `TEMPLATES` (`"APP_DIRS": True`) en `config/settings.py`.
- **Por qué:**
  La excepción `TemplateDoesNotExist` ocurre cuando la URL y la vista sí se ejecutaron correctamente, pero el motor de plantillas de Django buscó el archivo HTML y no lo encontró en ninguno de los directorios configurados. Con `"APP_DIRS": True`, Django solo busca dentro de la carpeta `templates/` de las aplicaciones que estén registradas en `INSTALLED_APPS`. Además, si se omite el subdirectorio con espacio de nombres (`pages/templates/pages/`), la ruta `"pages/resources.html"` no coincidirá.
- **Cómo lo implementaría:**
  1. En `config/settings.py`, me aseguraría de que `'pages'` esté en `INSTALLED_APPS` y que `TEMPLATES` tenga `'APP_DIRS': True`.
  2. Confirmaría que los archivos existan físicamente en la ruta con doble carpeta `pages/templates/pages/` (creados en el commit `d3fbc16`):
     - `pages/templates/pages/base.html`
     - `pages/templates/pages/home.html`
     - `pages/templates/pages/resources.html`
     - `pages/templates/pages/about.html`
  3. En `pages/views.py`, verificaría que cada vista llame a `render()` con el nombre exacto, por ejemplo:
     ```python
     return render(request, "pages/resources.html", context)
     ```
- **Cómo comprobaría el resultado:**
  Recargaría la página en el navegador para confirmar que desaparece la página de excepción de Django y ejecutaría `python manage.py test pages`, donde `self.assertTemplateUsed(response, "pages/resources.html")` valida que la plantilla se localizó y renderizó sin errores.

---

### 3. ¿Qué harías si la página de recursos se abre correctamente, pero no muestra los elementos enviados desde la vista o presenta estados de disponibilidad incorrectos?

- **Qué haría:**
  Compararía las claves del diccionario `context` en la función `resources(request)` de `pages/views.py` con los nombres de variables e instrucciones del lenguaje de plantillas (`{% for %}`, `{% if %}`, `{% empty %}`) en `pages/templates/pages/resources.html`.
- **Por qué:**
  El sistema de plantillas de Django está diseñado para no lanzar excepciones fatales en el navegador cuando una variable de contexto no existe; en su lugar, la evalúa como vacía. Por tanto:
  - Si en `pages/views.py` la clave se llama `"items"` pero en `resources.html` el bucle dice `{% for resource in resources %}`, la plantilla asumirá que `resources` no existe y mostrará el bloque `{% empty %}` (`No resources are available.`).
  - Si la clave booleana `"available"` está mal escrita en algún diccionario o en la condición `{% if resource.available %}`, se evaluará como `False` y mostrará `"Coming soon"` para todos los elementos.
- **Cómo lo implementaría:**
  1. En `pages/views.py`, verificaría que la clave principal sea `"resources"` y que cada diccionario tenga exactamente las claves `"title"`, `"category"` y `"available"` (con valores booleanos `True` o `False`, no cadenas de texto):
     ```python
     context = {
         "resources": [
             {"title": "Python basics", "category": "Programming", "available": True},
             {"title": "Git and GitHub", "category": "Collaboration", "available": True},
             {"title": "Django models", "category": "Web development", "available": False},
         ],
     }
     ```
  2. En `pages/templates/pages/resources.html`, verificaría la correspondencia exacta de nombres y bloques:
     ```html
     {% for resource in resources %}
         <article class="resource">
             <h3>{{ resource.title }}</h3>
             <p>Category: {{ resource.category }}</p>
             {% if resource.available %}
                 <p>Available</p>
             {% else %}
                 <p>Coming soon</p>
             {% endif %}
         </article>
     {% empty %}
         <p>No resources are available.</p>
     {% endfor %}
     ```
- **Cómo comprobaría el resultado:**
  - **Con recursos presentes (Prueba P02 y P06):** abriría `/resources/` y comprobaría que `"Python basics"` y `"Git and GitHub"` muestran `Available`, mientras que `"Django models"` muestra `Coming soon`. Además, ejecutaría `python manage.py test pages` para verificar `test_resources_page_shows_example_data`.
  - **Con lista vacía (Prueba P07):** cambiaría temporalmente `"resources": []` en `pages/views.py`, recargaría `/resources/` para comprobar que se renderiza el bloque `{% empty %}` con el texto `No resources are available.`, y finalmente restauraría los tres recursos originales.

---

### 4. ¿Qué harías si modificas el nombre de una ruta y los enlaces de navegación dejan de funcionar en varias páginas?

- **Qué haría:**
  Revisaría el identificador `app_name` y los parámetros `name="..."` dentro de `pages/urls.py`, y los contrastaría con las etiquetas `{% url 'pages:...' %}` definidas en `pages/templates/pages/base.html` y en las plantillas hijas (`home.html`).
- **Por qué:**
  Cuando se usa resolución inversa de URLs (`{% url 'pages:resources' %}`), Django busca dentro del espacio de nombres `app_name = "pages"` una ruta cuyo atributo `name` coincida. Si se cambia `name="resources"` en `pages/urls.py` sin actualizar `{% url %}` (o viceversa), Django lanza un error `NoReverseMatch`. Como todas las páginas heredan la barra de navegación desde `base.html`, un error en el menú de `base.html` afecta inmediatamente a todas las páginas que extienden de ella.
- **Cómo lo implementaría:**
  1. Verificaría en `pages/urls.py` que `app_name` y cada `name` sean consistentes:
     ```python
     app_name = "pages"

     urlpatterns = [
         path("", views.home, name="home"),
         path("resources/", views.resources, name="resources"),
         path("about/", views.about, name="about"),
     ]
     ```
  2. Revisaría `pages/templates/pages/base.html` (y el enlace de `home.html`) para asegurar que usen el formato `'app_name:name'`:
     ```html
     <nav aria-label="Main navigation">
         <a href="{% url 'pages:home' %}">Home</a>
         <a href="{% url 'pages:resources' %}">Resources</a>
         <a href="{% url 'pages:about' %}">About</a>
     </nav>
     ```
  - **Justificación de mantenimiento:** Gracias a la herencia de plantillas (`{% extends "pages/base.html" %}`), la barra de navegación está escrita en un único archivo (`base.html`) y no copiada en cada página; si el nombre de una ruta cambia, basta actualizarlo en `base.html`. Asimismo, usar nombres de ruta (`{% url 'pages:resources' %}`) permite cambiar la URL visible (por ejemplo, de `resources/` a `catalogo-recursos/`) en `pages/urls.py` sin tener que modificar ningún archivo HTML ni prueba.
- **Cómo comprobaría el resultado:**
  Navegaría manualmente haciendo clic en `Home`, `Resources` y `About` desde las tres páginas (Prueba P04) y ejecutaría `python manage.py test pages`, verificando especialmente la prueba `test_home_navigation_links_to_resources` que comprueba que el HTML contenga `href="/resources/"` generado mediante `reverse("pages:resources")`.

---

### 5. ¿Qué harías si `python manage.py check` termina sin errores, pero una prueba automatizada falla después de modificar una vista o plantilla que anteriormente funcionaba?

- **Qué haría:**
  1. Analizaría el mensaje de fallo arrojado por `python manage.py test pages` para identificar qué prueba de `pages/tests.py` falló y en qué aserción (`assertEqual`, `assertTemplateUsed` o `assertContains`).
  2. Revisaría los cambios recientes frente al último estado funcional usando el historial de Git (`git status`, `git diff` y `git log`).
  3. Reproduciría el fallo, corregidría la vista o plantilla afectada y volvería a ejecutar toda la suite de pruebas.
- **Por qué (`check` vs `test`):**
  - `python manage.py check` realiza únicamente una **validación estática del sistema y de la configuración** (sintaxis de `settings.py`, registro de `INSTALLED_APPS`, validez básica de patrones de URL). No ejecuta las vistas, no renderiza las plantillas ni comprueba el contenido HTML.
  - `python manage.py test pages` ejecuta **pruebas de comportamiento** simulando peticiones HTTP reales con `self.client.get()`. Por eso, si alguien borra accidentalmente la palabra `"Welcome"` de `home.html` o cambia una condición en `resources.html`, `manage.py check` dirá `0 issues`, pero `manage.py test pages` detectará la regresión inmediatamente.
- **Cómo lo implementaría:**
  1. Ejecutaría `python manage.py test pages` y leería el *traceback* (por ejemplo, si falla `self.assertContains(response, "Coming soon")` en `test_resources_page_shows_example_data`).
  2. Ejecutaría `git diff` (comparando contra commits estables como `d3fbc16` — *feat: render pages with shared templates* — o `b7ee323` — *test: verify page responses and navigation*) para localizar exactamente qué línea se modificó en `pages/views.py` o `pages/templates/pages/resources.html`.
  3. Corregiría el código en el archivo correspondiente para restaurar el comportamiento esperado.
- **Cómo comprobaría el resultado:**
  Ejecutaría nuevamente ambos comandos en la terminal:
  ```bash
  python manage.py check
  python manage.py test pages
  ```
  Confirmaría que `check` reporte `0 issues` y que `test pages` ejecute las **5 pruebas en estado `OK`** (`Ran 5 tests ... OK`), garantizando que el problema quedó resuelto sin introducir regresiones en las demás páginas ni en la navegación.
