# Guía Esencial de Django - SSR y Plantillas

## 1. Ambientes Virtuales

### ¿Por qué usar ambientes virtuales?

Un ambiente virtual es un espacio aislado para cada proyecto Python que permite:

- **Evitar conflictos entre proyectos**: Diferentes proyectos pueden usar diferentes versiones de Django u otros paquetes
- **Mantener dependencias específicas**: Cada proyecto tiene sus propias versiones de librerías
- **Reproducibilidad**: Generar `requirements.txt` para que otros instalen las mismas dependencias
- **Limpieza del sistema**: No contaminar la instalación global de Python
- **Evitar problemas de permisos**: Instalación sin necesidad de privilegios de administrador

### Comandos básicos

```bash
# Crear ambiente virtual
python -m venv env

# Activar ambiente virtual
env\Scripts\activate      # Windows
source env/bin/activate   # Linux/MacOS

# Desactivar ambiente virtual
deactivate
```

---

## 2. Estructura Básica de un Proyecto Django

### Flujo de creación

1. **Crear proyecto**
```bash
django-admin startproject nombreProyecto .
```
*El punto (.) crea el proyecto en el directorio actual sin carpeta adicional*

2. **Crear app**
```bash
python manage.py startapp nombreApp
```

3. **Registrar app en `settings.py`**
```python
INSTALLED_APPS = [
    # ...
    'nombreApp',
]
```

4. **Configurar URLs del proyecto** (`nombreProyecto/urls.py`)
```python
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('nombreApp.urls')),
]
```

5. **Configurar URLs de la app** (`nombreApp/urls.py`)
```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
```

### Conceptos clave

- **Proyecto**: Contenedor principal con configuración global
- **App**: Módulo funcional reutilizable (blog, usuarios, productos)
- Un proyecto puede tener múltiples apps

---

## 3. Server-Side Rendering (SSR)

### ¿Qué es SSR?

Django genera el HTML completo en el servidor y lo envía renderizado al navegador.

**Flujo SSR:**
1. Usuario solicita una página
2. Django ejecuta la vista en Python
3. Procesa datos (base de datos, APIs, cálculos)
4. Renderiza la plantilla HTML con esos datos
5. Envía HTML completo al navegador
6. Navegador solo muestra el resultado

**Contraste con SPA (React/Vue):**
- SPA: Servidor envía HTML vacío + JavaScript → Navegador hace peticiones y renderiza
- SSR: Servidor envía HTML completo → Navegador solo muestra

---

## 4. Sistema de Plantillas

### 4.1 Configuración inicial

En `settings.py`:

```python
from pathlib import Path
import os

BASE_DIR = Path(__file__).resolve().parent.parent

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'templates')],
        'APP_DIRS': True,
        # ...
    },
]
```

**`os.path.join(BASE_DIR, 'templates')`**: Construye la ruta absoluta a la carpeta templates de forma compatible con cualquier sistema operativo.

### 4.2 Renderizar plantilla básica

```python
# views.py
from django.shortcuts import render

def index(request):
    return render(request, 'homepage/index.html')
```

Estructura de carpetas:
```
templates/
└── homepage/
    └── index.html
```

---

## 5. Herencia de Plantillas

### Concepto

Evita repetir código HTML común (header, navbar, footer) en cada página.

### Plantilla base (`templates/dashboard/base.html`)

```html
<!DOCTYPE html>
<html>
<head>
    <title>Mi Aplicación</title>
</head>
<body>
    <nav><!-- Menú de navegación --></nav>
    
    {% block content %}
    <!-- Contenido por defecto -->
    {% endblock %}
    
    <footer><!-- Pie de página --></footer>
</body>
</html>
```

### Plantilla hija (`templates/dashboard/index.html`)

```html
{% extends "dashboard/base.html" %}

{% block content %}
    <h1>Bienvenido al Dashboard</h1>
    <p>Contenido específico de esta página</p>
{% endblock %}
```

### ¿Cómo funciona?

1. Django carga `base.html` completo
2. Busca `{% block content %}`
3. Lo **reemplaza** con el contenido definido en la plantilla hija
4. Envía HTML completo al navegador

### Ventajas

- ✅ **DRY**: Código común en un solo lugar
- ✅ **Mantenibilidad**: Cambias el navbar una vez, se actualiza en todas las páginas
- ✅ **Consistencia**: Todas las páginas tienen la misma estructura

---

## 6. Fragmentos de Plantilla (Partials)

### Concepto

Piezas reutilizables de HTML que se pueden insertar donde sea necesario usando `{% include %}`.

### Ejemplo

**Fragmento** (`templates/dashboard/partials/header.html`):
```html
<h1 class="text-6xl">Bienvenido al Dashboard</h1>
```

**Uso en plantilla** (`templates/dashboard/index.html`):
```html
{% extends "dashboard/base.html" %}

{% block content %}
    {% include "./partials/header.html" %}
    {% include "./content/data.html" %}
{% endblock %}
```

### Herencia vs Fragmentos

| Concepto | Herencia (`extends`) | Fragmentos (`include`) |
|----------|---------------------|------------------------|
| **Propósito** | Estructura completa de página | Componentes reutilizables |
| **Cantidad** | Solo 1 `extends` por plantilla | Múltiples `include` |
| **Alcance** | Toda la página | Pedazos específicos |
| **Ejemplo** | Layout general (navbar, footer) | Botón, card, formulario |

### Analogía

- **Herencia** = Estructura de una casa (paredes, techo, piso)
- **Fragmentos** = Muebles que puedes mover (mesa, silla, lámpara)

---

## 7. Pasar Datos del Servidor a Plantillas

### En la vista (`views.py`)

```python
from django.shortcuts import render

def index(request):
    # Crear diccionario con datos
    data = {
        'title': "Landing Page' Dashboard",
        'usuarios': 150,
        'ventas': 42,
    }
    
    # Pasar como contexto a la plantilla
    return render(request, 'dashboard/index.html', data)
```

### En la plantilla (`index.html`)

```html
<h2>{{ title }}</h2>
<p>Usuarios registrados: {{ usuarios }}</p>
<p>Ventas hoy: {{ ventas }}</p>
```

### Flujo completo

1. Usuario visita la URL
2. Django ejecuta la vista
3. Python crea el diccionario `data` con información dinámica
4. Django renderiza la plantilla
5. Reemplaza `{{ variable }}` por su valor del contexto
6. Servidor envía HTML completo al navegador

### Datos dinámicos

```python
def index(request):
    # Datos de base de datos
    total_usuarios = Usuario.objects.count()
    
    # Datos de API externa
    response = requests.get('https://api.clima.com')
    temperatura = response.json()['temp']
    
    data = {
        'usuarios': total_usuarios,
        'clima': temperatura,
    }
    
    return render(request, 'dashboard/index.html', data)
```

---

## 8. Consumir APIs Externas

### Instalación de `requests`

```bash
pip install requests
```

### Propósito

Permite que Django se comunique con servicios externos (APIs de clima, pagos, redes sociales, etc.)

### Configuración en `settings.py`

```python
# Centralizar URLs de APIs
API_URL = 'https://jsonplaceholder.typicode.com/posts'
```

**Ventajas:**
- Un solo lugar para configuración
- Fácil cambiar de API de prueba a producción
- Reutilización en múltiples vistas

### Uso en vista

```python
from django.shortcuts import render
from django.conf import settings
import requests

def index(request):
    # Hacer petición a API externa
    response = requests.get(settings.API_URL)
    posts = response.json()
    
    # Procesar datos
    total_responses = len(posts)
    
    # Pasar a plantilla
    data = {
        'title': "Dashboard",
        'total_responses': total_responses,
    }
    
    return render(request, 'dashboard/index.html', data)
```

### En la plantilla

```html
<p>Número total de respuestas: {{ total_responses }}</p>
```

### Flujo completo con API

1. Usuario → Solicita `/dashboard/`
2. Django → Ejecuta vista `index()`
3. Vista → Hace petición HTTP a API externa con `requests`
4. API externa → Devuelve JSON
5. Vista → Procesa datos (cuenta, filtra, etc.)
6. Vista → Pasa datos al contexto
7. Django → Renderiza HTML insertando variables
8. Servidor → Envía HTML completo
9. Navegador → Muestra resultado

### Métodos comunes de `requests`

```python
# GET - Obtener datos
response = requests.get(url)

# POST - Enviar datos
response = requests.post(url, data={'key': 'value'})

# PUT - Actualizar recursos
response = requests.put(url, data={'key': 'value'})

# DELETE - Eliminar recursos
response = requests.delete(url)

# Convertir respuesta JSON a diccionario
data = response.json()
```

---

## 9. Gestión de Dependencias

### Generar `requirements.txt`

```bash
pip freeze > requirements.txt
```

### ¿Por qué es importante?

Es la **receta exacta** de todos los paquetes que tu proyecto necesita, incluyendo versiones específicas.

### Casos de uso

**1. Trabajo en equipo:**
```bash
# Compañero clona el repo e instala dependencias
pip install -r requirements.txt
```

**2. Despliegue a producción:**
El servidor lee `requirements.txt` e instala automáticamente todo.

**3. Nueva computadora:**
Clonas el repo, creas ambiente virtual nuevo y ejecutas:
```bash
pip install -r requirements.txt
```

**4. Control de versiones:**
Asegura que todos usen las mismas versiones:
```
Django==5.0.1
requests==2.31.0
```

### Contenido de ejemplo

```
asgiref==3.7.2
certifi==2023.11.17
charset-normalizer==3.3.2
Django==5.0.1
idna==3.6
requests==2.31.0
sqlparse==0.4.4
tzdata==2023.4
urllib3==2.1.0
```

### Buenas prácticas

**Actualizar cuando instalas paquetes nuevos:**
```bash
pip install pillow
pip freeze > requirements.txt
```

**Commitearlo a Git:**
```bash
git add requirements.txt
git commit -m "Add dependencies"
```

### Sin `requirements.txt` = Problemas

- ❌ Nadie sabe qué paquetes instalar
- ❌ Versiones diferentes en cada máquina
- ❌ "En mi máquina funciona" pero en producción falla
- ❌ Imposible automatizar despliegues

---

## 10. Resumen de Comandos Esenciales

```bash
# Ambiente virtual
python -m venv env
env\Scripts\activate  # Windows
source env/bin/activate  # Linux/Mac
deactivate

# Django
django-admin startproject nombreProyecto .
python manage.py startapp nombreApp
python manage.py runserver

# Dependencias
pip install paquete
pip freeze > requirements.txt
pip install -r requirements.txt
```

---

## 11. Estructura de Carpetas Típica

```
mi_proyecto/
├── env/                          # Ambiente virtual (no commitear)
├── mi_proyecto/                  # Configuración del proyecto
│   ├── __init__.py
│   ├── settings.py              # Configuración global
│   ├── urls.py                  # URLs principales
│   └── wsgi.py
├── mi_app/                       # Aplicación
│   ├── __init__.py
│   ├── views.py                 # Lógica de vistas
│   ├── urls.py                  # URLs de la app
│   ├── models.py                # Modelos de datos
│   └── admin.py
├── templates/                    # Plantillas HTML
│   ├── mi_app/
│   │   ├── base.html           # Plantilla base
│   │   ├── index.html          # Plantilla hija
│   │   ├── partials/           # Fragmentos reutilizables
│   │   │   └── header.html
│   │   └── content/
│   │       └── data.html
├── static/                       # CSS, JS, imágenes
├── manage.py                     # Utilidad de Django
└── requirements.txt              # Dependencias del proyecto
```

---

## 12. Tips Finales

1. **Siempre usa ambientes virtuales** - Evita conflictos entre proyectos
2. **Organiza tus plantillas** - Usa herencia y fragmentos para código limpio
3. **Centraliza configuración** - URLs de APIs y constantes en `settings.py`
4. **Mantén `requirements.txt` actualizado** - Esencial para colaboración y despliegue
5. **Usa nombres descriptivos** - Para apps, vistas, plantillas y variables
6. **SSR es poderoso** - Procesa datos en el servidor, envía HTML listo al navegador