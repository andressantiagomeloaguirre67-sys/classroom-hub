# ClassroomHub

**ClassroomHub** es un pequeño portal web de recursos de aprendizaje construido con Django como parte del **Laboratorio 6 — Fundamentos de Django y primera aplicación web**. Su propósito es demostrar el recorrido completo de una solicitud web (`URL → vista → plantilla → respuesta HTTP`) utilizando una arquitectura modular y plantillas con diseño compartido.

## Versiones utilizadas

- **Python:** 3.14 (compatible con Python 3.12, 3.13, 3.14 o 3.15)
- **Django:** 6.1.3

## Estructura del proyecto

```text
classroom-hub/
├── .gitignore
├── README.md
├── requirements.txt
├── manage.py
├── config/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   ├── asgi.py
│   └── wsgi.py
└── pages/
    ├── __init__.py
    ├── admin.py
    ├── apps.py
    ├── migrations/
    │   └── __init__.py
    ├── models.py
    ├── tests.py
    ├── urls.py
    ├── views.py
    └── templates/
        └── pages/
            ├── base.html
            ├── home.html
            ├── resources.html
            └── about.html
```

## Instalación y configuración local

1. **Clonar el repositorio y entrar al directorio:**

   ```bash
   git clone <URL_DEL_REPOSITORIO>
   cd classroom-hub
   ```

2. **Crear y activar el entorno virtual:**

   En Linux / macOS:
   ```bash
   python -m venv .venv
   source .venv/bin/activate
   ```

   En Windows (PowerShell):
   ```powershell
   python -m venv .venv
   .\.venv\Scripts\Activate.ps1
   ```

3. **Instalar las dependencias:**

   ```bash
   python -m pip install -r requirements.txt
   ```

4. **Verificar la configuración del proyecto:**

   ```bash
   python manage.py check
   ```

## Ejecución del servidor de desarrollo

Para iniciar el servidor local:

```bash
python manage.py runserver
```

Luego abre `http://127.0.0.1:8000/` en tu navegador.

## Rutas públicas disponibles

| Ruta | Nombre de ruta | Vista | Descripción |
|---|---|---|---|
| `/` | `pages:home` | `pages.views.home` | Página principal con mensaje de bienvenida y acceso a recursos |
| `/resources/` | `pages:resources` | `pages.views.resources` | Lista de recursos de aprendizaje y su estado de disponibilidad |
| `/about/` | `pages:about` | `pages.views.about` | Información sobre el propósito del proyecto |

## Ejecución de pruebas automatizadas

Para ejecutar las pruebas de la aplicación `pages`:

```bash
python manage.py test pages
```

Las pruebas verifican:
- Respuesta `200 OK` y plantilla correcta en `/`, `/resources/` y `/about/`.
- Renderizado de los datos de ejemplo y estados (`Available` / `Coming soon`).
- Enlaces de navegación generados con nombres de ruta (`reverse("pages:resources")`).
- Respuesta `404 Not Found` para rutas inexistentes (`/not-found/`).

## Alcance actual (qué funcionalidades todavía no incluye)

Esta primera versión se enfoca exclusivamente en rutas, vistas y plantillas:
- **No utiliza base de datos propia ni modelos persistentes:** los recursos son datos de ejemplo definidos en memoria dentro de `pages/views.py`.
- **No incluye formularios ni operaciones CRUD:** todavía no es posible crear, editar o eliminar recursos desde la interfaz.
- **No incluye autenticación de usuarios ni panel administrativo personalizado** (estos temas se abordan en el Laboratorio 7).
