# Guía Esencial: Autenticación en Django

## 1. Migraciones de Base de Datos

### ¿Para qué sirven?
Sincronizan tu código Python con la base de datos.

```bash
# Crear archivos de migración
python manage.py makemigrations

# Aplicar cambios a la BD
python manage.py migrate

# Crear superusuario
python manage.py createsuperuser
```

### ¿Por qué son necesarias siempre?
Incluso sin modelos propios, Django necesita tablas para:
- Sistema de autenticación (usuarios, grupos, permisos)
- Sesiones
- Panel de administración
- Registro de migraciones

---

## 2. Configuración de Seguridad

### ALLOWED_HOSTS
Protege contra ataques de Host Header Poisoning.

```python
# settings.py

# Desarrollo
ALLOWED_HOSTS = ["*"]  # ⚠️ Solo para desarrollo

# Producción
ALLOWED_HOSTS = ["miapp.com", "www.miapp.com"]
```

### CSRF_TRUSTED_ORIGINS
Define orígenes confiables para protección CSRF.

```python
# settings.py
CSRF_TRUSTED_ORIGINS = [
    "https://*.app.github.dev",  # GitHub Codespaces
    "https://localhost:8000",
    "http://127.0.0.1:8000"
]
```

---

## 3. Autenticación vs Autorización

| Aspecto | Autenticación | Autorización |
|---------|---------------|--------------|
| **Pregunta** | ¿Quién eres? | ¿Qué puedes hacer? |
| **Verifica** | Identidad | Permisos |
| **Django** | `@login_required` | `@permission_required` |
| **HTTP** | 401 Unauthorized | 403 Forbidden |

---

## 4. Sistema de Login/Logout

### Configuración en settings.py

```python
# settings.py

# Dónde redirigir si no está autenticado
LOGIN_URL = '/login/'

# Dónde ir después de login exitoso
LOGIN_REDIRECT_URL = '/'
```

### Rutas en urls.py

```python
# urls.py
from django.contrib.auth import views as auth_views

urlpatterns = [
    # Login
    path('login/', 
         auth_views.LoginView.as_view(
             template_name='security/login.html'
         ), 
         name='login'),
    
    # Logout
    path('logout/', 
         auth_views.LogoutView.as_view(
             next_page='/login/'
         ), 
         name='logout'),
]
```

### Proteger vistas

```python
# views.py
from django.contrib.auth.decorators import login_required

@login_required
def dashboard(request):
    return render(request, 'dashboard.html')
```

---

## 5. Token CSRF

### ¿Qué es?
Protección contra ataques Cross-Site Request Forgery.

### Uso obligatorio en formularios POST

```html
<form method="post" action="{% url 'login' %}">
    {% csrf_token %}  <!-- ⚠️ OBLIGATORIO -->
    
    <input name="username" type="text">
    <input name="password" type="password">
    <button type="submit">Entrar</button>
</form>
```

### ¿Por qué es necesario?
Sin CSRF token, sitios maliciosos podrían enviar peticiones en tu nombre sin tu conocimiento.

---

## 6. Cookie de Sesión

### ¿Qué hace?
Mantiene el estado de autenticación entre peticiones HTTP.

### Flujo
1. **Login exitoso** → Django crea sesión y envía cookie
2. **Peticiones siguientes** → Navegador envía cookie automáticamente
3. **Django identifica** → Lee cookie y carga usuario
4. **Logout** → Django elimina sesión e invalida cookie

### Ver en el navegador
F12 → Application → Cookies → `sessionid`

---

## 7. Templates de Django

### Condicionales

```django
{% if user.is_authenticated %}
    <p>Bienvenido, {{ user.username }}</p>
{% else %}
    <a href="{% url 'login' %}">Iniciar sesión</a>
{% endif %}
```

### Objeto `user` (siempre disponible)

```django
{{ user.username }}         <!-- Nombre de usuario -->
{{ user.is_authenticated }} <!-- ¿Está logueado? -->
{{ user.is_staff }}         <!-- ¿Puede acceder al admin? -->
{{ user.is_superuser }}     <!-- ¿Tiene todos los permisos? -->
```

### Objeto `form` (en vistas de formulario)

```django
{# Mostrar errores generales #}
{% if form.non_field_errors %}
    <div class="error">
        Credenciales inválidas
    </div>
{% endif %}

{# Mostrar errores por campo #}
{% if form.username.errors %}
    <span>{{ form.username.errors }}</span>
{% endif %}
```

---

## 8. Plantilla de Login Completa

```html
<!DOCTYPE html>
<html>
<head>
    <title>Login</title>
</head>
<body>
    <h2>Iniciar Sesión</h2>
    
    <!-- Mensaje de error -->
    {% if form.non_field_errors %}
        <div class="error">
            Usuario o contraseña incorrectos
        </div>
    {% endif %}
    
    <!-- Formulario -->
    <form method="post" action="{% url 'login' %}">
        {% csrf_token %}
        
        <input name="username" type="text" placeholder="Usuario" required>
        <input name="password" type="password" placeholder="Contraseña" required>
        
        <button type="submit">Entrar</button>
    </form>
</body>
</html>
```

---

## 9. Botón de Logout

```html
<form method="post" action="{% url 'logout' %}">
    {% csrf_token %}
    <button type="submit">Cerrar sesión</button>
</form>
```

---

## 10. Checklist de Seguridad

### Desarrollo
- ✅ `ALLOWED_HOSTS = ["*"]`
- ✅ `DEBUG = True`
- ✅ `SESSION_COOKIE_SECURE = False`

### Producción
- ✅ `ALLOWED_HOSTS = ["midominio.com"]`
- ✅ `DEBUG = False`
- ✅ `SESSION_COOKIE_SECURE = True`
- ✅ `CSRF_COOKIE_SECURE = True`
- ✅ `SESSION_COOKIE_HTTPONLY = True`
- ✅ `CSRF_COOKIE_HTTPONLY = True`

---

## 11. Comandos Útiles

```bash
# Crear migraciones
python manage.py makemigrations

# Aplicar migraciones
python manage.py migrate

# Crear superusuario
python manage.py createsuperuser

# Levantar servidor
python manage.py runserver

# Ver usuarios en shell
python manage.py shell
>>> from django.contrib.auth.models import User
>>> User.objects.all()
```

---

## 12. Flujo Completo de Autenticación

```
1. Usuario accede a /dashboard/
   ↓
2. @login_required verifica autenticación
   ↓
3. No autenticado → Redirige a /login/?next=/dashboard/
   ↓
4. Usuario ingresa credenciales
   ↓
5. LoginView valida con CSRF token
   ↓
6. Credenciales correctas → Crea sesión + cookie
   ↓
7. Redirige a /dashboard/ (parámetro ?next=)
   ↓
8. Cookie enviada en cada petición
   ↓
9. Django identifica usuario automáticamente
   ↓
10. Logout → Elimina sesión + cookie
```

---

## Errores Comunes

### 404 en /accounts/login/
**Solución:** Configurar `LOGIN_URL = '/login/'` en settings.py

### CSRF verification failed
**Solución:** Agregar `{% csrf_token %}` en el formulario

### Cookie no persiste
**Solución:** Verificar que el navegador acepte cookies

### Usuario no se muestra en template
**Solución:** Verificar `{% if user.is_authenticated %}`

---

## Recursos

- Documentación oficial: https://docs.djangoproject.com/en/stable/topics/auth/
- Panel admin: http://127.0.0.1:8000/admin/
- Login personalizado: http://127.0.0.1:8000/login/