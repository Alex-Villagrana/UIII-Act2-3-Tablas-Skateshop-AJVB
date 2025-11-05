## 📋 Proyecto Skateshop - Procedimientos Completos (Código Integrado)

### 0\. 📂 Estructura de Carpetas y Archivos

```
UIII_Skateshop_0402/
├── .venv/
├── backend_Skateshop/
│   ├── settings.py                <- MODIFICAR: Agregar 'app_Skateshop'.
│   ├── urls.py                    <- MODIFICAR: Enlazar con app_Skateshop.
│   └── ...
├── app_Skateshop/
│   ├── templates/
│   │   ├── base.html              <- CÓDIGO INCLUIDO.
│   │   ├── header.html            <- CÓDIGO INCLUIDO.
│   │   ├── navbar.html            <- CÓDIGO INCLUIDO.
│   │   ├── footer.html            <- CÓDIGO INCLUIDO.
│   │   ├── inicio.html            <- CÓDIGO INCLUIDO.
│   │   └── categoria/
│   │       ├── agregar_categoria.html   <- CÓDIGO INCLUIDO.
│   │       ├── ver_categorias.html      <- CÓDIGO INCLUIDO.
│   │       ├── actualizar_categoria.html <- CÓDIGO INCLUIDO.
│   │       └── borrar_categoria.html    <- CÓDIGO INCLUIDO.
│   ├── admin.py                   <- CÓDIGO INCLUIDO.
│   ├── models.py                  <- CÓDIGO INCLUIDO.
│   ├── urls.py                    <- CÓDIGO INCLUIDO.
│   └── views.py                   <- REQUIERE CÓDIGO (funciones declaradas, no implementadas).
└── manage.py
```

-----

### Primera parte: Configuración y Modelos

1.  **Procedimiento para crear carpeta del Proyecto:**
      * Crea la carpeta raíz del proyecto: `UIII_Skateshop_0402`
2.  **Procedimiento para abrir VS Code sobre la carpeta:**
      * Abre VS Code y luego la carpeta `UIII_Skateshop_0402`.
3.  **Procedimiento para abrir terminal en VS Code**
      * Abre una nueva terminal en VS Code.
4.  **Procedimiento para crear carpeta entorno virtual “.venv” desde terminal de VS Code**
      * Ejecuta: `python -m venv .venv`
5.  **Procedimiento para activar el entorno virtual**
      * **Windows (PowerShell):** `.\.venv\Scripts\Activate`
      * **Linux/macOS:** `source ./.venv/bin/activate`
6.  **Procedimiento para activar intérprete de Python**
      * Selecciona el intérprete Python del entorno `.venv` en VS Code.
7.  **Procedimiento para instalar Django**
      * Ejecuta: `pip install Django`
8.  **Procedimiento para crear proyecto `backend_Skateshop` (sin duplicar carpeta)**
      * Ejecuta: `django-admin startproject backend_Skateshop .`
9.  **Procedimiento para ejecutar servidor en el puerto 8036**
      * Ejecuta: `python manage.py runserver 8036`
10. **Procedimiento para copiar y pegar el link en el navegador**
      * Copia y abre `http://127.0.0.1:8036/` en el navegador.
11. **Procedimiento para crear aplicación `app_Skateshop`**
      * Ejecuta: `python manage.py startapp app_Skateshop`
12. **Aquí el modelo `models.py`** (Contenido para `app_Skateshop/models.py`):
    ```python
    from django.db import models
    # ==========================================
    # MODELO: CATEGORÍA
    # ==========================================
    class Categoria(models.Model):
        nombre = models.CharField(max_length=50, unique=True)
        descripcion = models.TextField(blank=True, null=True)
        fecha_creacion = models.DateField(auto_now_add=True)
        activo = models.BooleanField(default=True)
        popularidad = models.PositiveIntegerField(default=0)
        icono = models.CharField(max_length=100, blank=True, null=True)
        def __str__(self):
            return self.nombre
    # ==========================================
    # MODELO: PRODUCTO
    # ==========================================
    class Producto(models.Model):
        nombre = models.CharField(max_length=100)
        descripcion = models.TextField(blank=True, null=True)
        precio = models.DecimalField(max_digits=10, decimal_places=2)
        stock = models.PositiveIntegerField(default=0)
        categoria = models.ForeignKey(Categoria, on_delete=models.CASCADE, related_name="productos")
        marca = models.CharField(max_length=50)
        modelo = models.CharField(max_length=50, blank=True, null=True)
        fecha_agregado = models.DateField(auto_now_add=True)
        disponible = models.BooleanField(default=True)
        imagen = models.URLField(blank=True, null=True)
        def __str__(self):
            return f"{self.nombre} - {self.marca}"
    ```

12.5 **Procedimiento para realizar las migraciones (`makemigrations` y `migrate`)**
\* Ejecuta: `python manage.py makemigrations app_Skateshop`
\* Ejecuta: `python manage.py migrate`

-----

### Segunda parte: Vistas, Templates y Enrutamiento (CRUD Categoría)

13. **Primero trabajamos con el MODELO: CATEGORÍA**
14. **En `views.py` de `app_Skateshop` crear las funciones con sus códigos correspondientes:**
      * `inicio_skateshop`, `agregar_categoria`, `actualizar_categoria`, `realizar_actualizacion_categoria`, `borrar_categoria`.
💻 Procedimiento: Código para `app_Skateshop/views.py`

python
from django.shortcuts import render, redirect, get_object_or_404
from .models import Categoria # Importamos el modelo Categoria

# =======================================================
# 1. Función de Inicio
# =======================================================

def inicio_skateshop(request):
    """
    Muestra la página de inicio del sistema.
    """
    return render(request, 'inicio.html')


# =======================================================
# 2. CRUD: Agregar Categoría
# =======================================================

def agregar_categoria(request):
    """
    Maneja la visualización y el procesamiento del formulario para agregar una categoría.
    """
    if request.method == 'POST':
        # 29. No validar entrada de datos: se asume que los datos son correctos.
        nombre_recibido = request.POST.get('nombre')
        descripcion_recibida = request.POST.get('descripcion')
        activo_recibido = request.POST.get('activo', False) == 'True' # Convierte el checkbox

        Categoria.objects.create(
            nombre=nombre_recibido,
            descripcion=descripcion_recibida,
            activo=activo_recibido
        )
        # Redirige a la lista de categorías después de guardar
        return redirect('ver_categorias')
    
    # Si es GET, muestra el formulario
    return render(request, 'categoria/agregar_categoria.html')


# =======================================================
# 3. CRUD: Ver Categorías (Listado)
# =======================================================

def ver_categorias(request):
    """
    Muestra una lista de todas las categorías activas.
    """
    # Recupera todas las categorías de la base de datos
    categorias = Categoria.objects.all().order_by('nombre')
    
    context = {
        'categorias': categorias
    }
    # Renderiza la plantilla con la tabla de categorías
    return render(request, 'categoria/ver_categorias.html', context)


# =======================================================
# 4. CRUD: Actualizar Categoría (Mostrar Formulario)
# =======================================================

def actualizar_categoria(request, pk):
    """
    Muestra el formulario pre-llenado para editar una categoría específica.
    """
    # Obtiene el objeto Categoria o devuelve un 404 si no existe
    categoria_a_actualizar = get_object_or_404(Categoria, pk=pk)
    
    context = {
        'categoria': categoria_a_actualizar
    }
    # Renderiza el formulario de actualización
    return render(request, 'categoria/actualizar_categoria.html', context)


# =======================================================
# 5. CRUD: Actualizar Categoría (Procesar POST)
# (renombrada de realizar_actualizacion_categoria para claridad)
# =======================================================

def realizar_actualizacion_categoria(request, pk):
    """
    Procesa los datos del formulario de actualización (POST).
    """
    categoria = get_object_or_404(Categoria, pk=pk)

    if request.method == 'POST':
        # 29. No validar entrada de datos: se actualizan los campos directamente.
        categoria.nombre = request.POST.get('nombre')
        categoria.descripcion = request.POST.get('descripcion')
        categoria.activo = request.POST.get('activo', False) == 'True'
        # Guarda los cambios en la base de datos
        categoria.save()
        
        # Redirige a la lista de categorías después de actualizar
        return redirect('ver_categorias')
    
    # Si por alguna razón se accede por GET, redirige al formulario
    return redirect('actualizar_categoria', pk=pk)


# =======================================================
# 6. CRUD: Borrar Categoría
# =======================================================

def borrar_categoria(request, pk):
    """
    Muestra la página de confirmación de borrado o procesa la eliminación.
    """
    categoria = get_object_or_404(Categoria, pk=pk)

    if request.method == 'POST':
        # Procesa la eliminación
        categoria.delete()
        # Redirige a la lista de categorías
        return redirect('ver_categorias')

    # Si es GET, muestra la página de confirmación
    context = {
        'categoria': categoria
    }
    return render(request, 'categoria/borrar_categoria.html', context)
    
15. **Crear la carpeta `templates` dentro de `app_Skateshop`.**
16. **En la carpeta `templates` crear los archivos HTML:**
      * `base.html`, `header.html`, `navbar.html`, `footer.html`, `inicio.html`.
17. **En el archivo `base.html` agregar Bootstrap para CSS y JS.**
    ```html
    {% load static %}
    <!DOCTYPE html>
    <html lang="es">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>{% block title %}Skateshop Admin{% endblock %}</title>
        <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
        <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.3/font/bootstrap-icons.min.css">
        <style>
            body { background-color: #f8f9fa; }
            .page-container { display: flex; flex-direction: column; min-height: 100vh; }
            .content-wrap { flex: 1; padding-bottom: 70px; }
            .footer { position: fixed; bottom: 0; width: 100%; height: 60px; background-color: #343a40; color: white; padding: 10px 0; z-index: 1030; }
        </style>
    </head>
    <body class="page-container">
        <header>{% include 'header.html' %}{% include 'navbar.html' %}</header>
        <div class="content-wrap container mt-4">{% block content %}{% endblock %}</div>
        {% include 'footer.html' %}
        <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
    </body>
    </html>
    ```
18. **En el archivo `navbar.html` incluir las opciones:**
    ```html
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
        <div class="container-fluid">
            <a class="navbar-brand" href="#"><i class="bi bi-gear-fill me-2"></i>Sistema de Administración Skateshop</a>
            <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNavDropdown" aria-controls="navbarNavDropdown" aria-expanded="false" aria-label="Toggle navigation"><span class="navbar-toggler-icon"></span></button>
            <div class="collapse navbar-collapse" id="navbarNavDropdown">
                <ul class="navbar-nav">
                    <li class="nav-item"><a class="nav-link" href="{% url 'inicio_skateshop' %}"><i class="bi bi-house-door-fill me-1"></i>Inicio</a></li>
                    <li class="nav-item dropdown">
                        <a class="nav-link dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown"><i class="bi bi-tags-fill me-1"></i>Categorías</a>
                        <ul class="dropdown-menu">
                            <li><a class="dropdown-item" href="{% url 'agregar_categoria' %}">Agregar Categoría</a></li>
                            <li><a class="dropdown-item" href="{% url 'ver_categorias' %}">Ver Categorías</a></li>
                            <li><a class="dropdown-item disabled" href="#">Actualizar Categoría</a></li>
                            <li><a class="dropdown-item disabled" href="#">Borrar Categoría</a></li>
                        </ul>
                    </li>
                    <li class="nav-item dropdown">
                        <a class="nav-link dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown"><i class="bi bi-box-seam-fill me-1"></i>Productos</a>
                        <ul class="dropdown-menu">
                            <li><a class="dropdown-item" href="#">Agregar Producto</a></li>
                            <li><a class="dropdown-item" href="#">Ver Productos</a></li>
                            <li><a class="dropdown-item" href="#">Actualizar Producto</a></li>
                            <li><a class="dropdown-item" href="#">Borrar Producto</a></li>
                        </ul>
                    </li>
                </ul>
            </div>
        </div>
    </nav>
    ```
19. **En el archivo `footer.html` incluir:**
    ```html
    <footer class="footer mt-auto py-3 bg-dark">
        <div class="container text-center">
            <span class="text-white-50 small">
                &copy; Derechos de autor - {{ "now"|date:"Y" }} | Fecha del sistema: {{ "now"|date:"d/m/Y H:i:s" }}
                <br>
                Creado por Alejandro Jesus Villagrana Bojorquez, CBTis 128
            </span>
        </div>
    </footer>
    ```
20. **En el archivo `inicio.html` colocar información del sistema más una imagen tomada desde la red sobre Skateshop o Skateboarding.**
    ```html
    {% extends 'base.html' %}
    {% load static %}
    {% block title %}Inicio - Skateshop Admin{% endblock %}
    {% block content %}
        <div class="row">
            <div class="col-md-7">
                <h1 class="display-4 text-primary">Bienvenido al Sistema de Administración Skateshop</h1>
                <p class="lead">Este sistema utiliza **Python** con el *framework* **Django** y **Bootstrap 5** para la interfaz.</p>
                <hr class="my-4">
                <p><strong>Proyecto:</strong> Skateshop<br><strong>Lenguaje:</strong> Python<br><strong>Framework:</strong> Django</p>
            </div>
            <div class="col-md-5">
                <img src="https://images.unsplash.com/photo-1596162397940-d9d30009c9af?q=80&w=1974&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D" class="img-fluid rounded shadow-sm" alt="Imagen de Skateboarding" style="max-height: 400px; object-fit: cover;">
            </div>
        </div>
    {% endblock %}
    ```
21. **Crear la subcarpeta `categoria` dentro de `app_Skateshop/templates/`**
22. **Crear los archivos HTML con su código correspondiente dentro de `app_Skateshop/templates/categoria`:**
      * **`agregar_categoria.html`**
        ```html
        {% extends 'base.html' %}
        {% block title %}Agregar Categoría{% endblock %}
        {% block content %}
        <div class="card shadow-sm">
            <div class="card-header bg-primary text-white"><h5 class="mb-0"><i class="bi bi-tag-fill me-2"></i>Agregar Nueva Categoría</h5></div>
            <div class="card-body">
                <form method="post" action="#"> {% csrf_token %}
                    <div class="mb-3"><label for="nombre" class="form-label">Nombre:</label><input type="text" class="form-control" id="nombre" name="nombre" required></div>
                    <div class="mb-3"><label for="descripcion" class="form-label">Descripción:</label><textarea class="form-control" id="descripcion" name="descripcion" rows="3"></textarea></div>
                    <div class="form-check mb-4"><input class="form-check-input" type="checkbox" value="True" id="activo" name="activo" checked><label class="form-check-label" for="activo">Activo</label></div>
                    <button type="submit" class="btn btn-success"><i class="bi bi-plus-circle-fill me-2"></i>Guardar Categoría</button>
                    <a href="{% url 'ver_categorias' %}" class="btn btn-secondary ms-2">Cancelar</a>
                </form>
            </div>
        </div>
        {% endblock %}
        ```
      * **`ver_categorias.html`**
        ```html
        {% extends 'base.html' %}
        {% block title %}Ver Categorías{% endblock %}
        {% block content %}
        <div class="card shadow-sm">
            <div class="card-header bg-info text-dark"><h5 class="mb-0"><i class="bi bi-list-task me-2"></i>Lista de Categorías</h5></div>
            <div class="card-body">
                <a href="{% url 'agregar_categoria' %}" class="btn btn-primary mb-3"><i class="bi bi-plus me-1"></i>Agregar</a>
                <div class="table-responsive">
                    <table class="table table-hover table-striped">
                        <thead class="table-light"><tr><th>ID</th><th>Nombre</th><th>Activo</th><th>Acciones</th></tr></thead>
                        <tbody>
                            <tr>
                                <td>1</td>
                                <td>Tablas</td>
                                <td><i class="bi bi-check-circle-fill text-success"></i></td>
                                <td>
                                    <a href="#" class="btn btn-sm btn-outline-primary" title="Ver"><i class="bi bi-eye"></i></a>
                                    <a href="{% url 'actualizar_categoria' pk=1 %}" class="btn btn-sm btn-outline-warning" title="Editar"><i class="bi bi-pencil"></i></a>
                                    <a href="{% url 'borrar_categoria' pk=1 %}" class="btn btn-sm btn-outline-danger" title="Borrar"><i class="bi bi-trash"></i></a>
                                </td>
                            </tr>
                        </tbody>
                    </table>
                </div>
            </div>
        </div>
        {% endblock %}
        ```
      * **`actualizar_categoria.html`**
        ```html
        {% extends 'base.html' %}
        {% block title %}Actualizar Categoría{% endblock %}
        {% block content %}
        <div class="card shadow-sm">
            <div class="card-header bg-warning text-dark"><h5 class="mb-0"><i class="bi bi-pencil-square me-2"></i>Actualizar Categoría</h5></div>
            <div class="card-body">
                <form method="post" action="{% url 'realizar_actualizacion_categoria' pk=1 %}"> 
                    {% csrf_token %}
                    <h6 class="card-subtitle mb-3 text-muted">Editando Categoría: (Nombre de la Categoría)</h6>
                    <div class="mb-3"><label for="nombre" class="form-label">Nombre:</label><input type="text" class="form-control" id="nombre" name="nombre" value="Tablas" required></div>
                    <div class="mb-3"><label for="descripcion" class="form-label">Descripción:</label><textarea class="form-control" id="descripcion" name="descripcion" rows="3">Decks de madera.</textarea></div>
                    <div class="form-check mb-4"><input class="form-check-input" type="checkbox" value="True" id="activo" name="activo" checked><label class="form-check-label" for="activo">Activo</label></div>
                    <button type="submit" class="btn btn-success"><i class="bi bi-check-circle-fill me-2"></i>Guardar Cambios</button>
                    <a href="{% url 'ver_categorias' %}" class="btn btn-secondary ms-2">Cancelar</a>
                </form>
            </div>
        </div>
        {% endblock %}
        ```
      * **`borrar_categoria.html`**
        ```html
        {% extends 'base.html' %}
        {% block title %}Borrar Categoría{% endblock %}
        {% block content %}
        <div class="alert alert-danger shadow-sm" role="alert">
            <h4 class="alert-heading"><i class="bi bi-exclamation-triangle-fill me-2"></i>¡Advertencia!</h4>
            <p>Estás a punto de **eliminar permanentemente** la categoría: **Tablas**.</p>
            <hr>
            <p class="mb-0">¿Estás seguro de que deseas continuar con la eliminación?</p>
            <form method="post" action="{% url 'borrar_categoria' pk=1 %}" class="mt-3">
                {% csrf_token %}
                <button type="submit" class="btn btn-danger me-2"><i class="bi bi-trash-fill me-2"></i>Sí, Eliminar Categoría</button>
                <a href="{% url 'ver_categorias' %}" class="btn btn-secondary">Cancelar</a>
            </form>
        </div>
        {% endblock %}
        ```
23. **Procedimiento para crear el archivo `urls.py` en `app_Skateshop`**
    ```python
    # Contenido de app_Skateshop/urls.py
    from django.urls import path
    from . import views
    urlpatterns = [
        path('', views.inicio_skateshop, name='inicio_skateshop'),
        path('categoria/agregar/', views.agregar_categoria, name='agregar_categoria'),
        path('categoria/ver/', views.ver_categorias, name='ver_categorias'),
        path('categoria/actualizar/<int:pk>/', views.actualizar_categoria, name='actualizar_categoria'),
        path('categoria/actualizar_post/<int:pk>/', views.realizar_actualizacion_categoria, name='realizar_actualizacion_categoria'),
        path('categoria/borrar/<int:pk>/', views.borrar_categoria, name='borrar_categoria'),
    ]
    ```
24. **Procedimiento para agregar `app_Skateshop` en `settings.py` de `backend_Skateshop`.**
      * **MODIFICAR** `backend_Skateshop/settings.py` agregando `'app_Skateshop'` a `INSTALLED_APPS`.
    <!-- end list -->
    ```python
    # Dentro de backend_Skateshop/settings.py
    INSTALLED_APPS = [
        # ... apps por defecto
        'django.contrib.staticfiles',
        'app_Skateshop',  # <--- AGREGAR
    ]
    ```
25. **Realizar las configuraciones correspondientes en `urls.py` de `backend_Skateshop`**
      * **MODIFICAR** `backend_Skateshop/urls.py` para enlazar con `app_Skateshop`.
    <!-- end list -->
    ```python
    # Dentro de backend_Skateshop/urls.py
    from django.contrib import admin
    from django.urls import path, include # Importar include
    urlpatterns = [
        path('admin/', admin.site.urls),
        path('', include('app_Skateshop.urls')), # <--- AGREGAR
    ]
    ```
26. **Procedimiento para registrar los modelos en `admin.py` y volver a realizar las migraciones.**
      * **MODIFICAR** `app_Skateshop/admin.py` para registrar los modelos.
    <!-- end list -->
    ```python
    # Contenido de app_Skateshop/admin.py
    from django.contrib import admin
    from .models import Categoria, Producto 

    @admin.register(Categoria)
    class CategoriaAdmin(admin.ModelAdmin):
        list_display = ('nombre', 'activo', 'fecha_creacion')
        search_fields = ('nombre',)

    @admin.register(Producto)
    class ProductoAdmin(admin.ModelAdmin):
        list_display = ('nombre', 'precio', 'stock', 'categoria', 'disponible')
        list_filter = ('disponible', 'categoria')
        search_fields = ('nombre', 'marca')
    ```
      * Volver a realizar migraciones (Opcional, si `admin.py` no afecta la base de datos): `python manage.py makemigrations` y `python manage.py migrate`.

-----

### Tercera parte: Estilo y Finalización

27. **Por lo pronto solo trabajar con “Categoría”; dejar pendiente Producto si aún no se ha terminado.**
28. **Utilizar colores suaves, atractivos y modernos.** (Usado en el código HTML con Bootstrap primary, info, y colores de fondo suaves).
29. **Código de las páginas web sencillo.**
30. **No validar entrada de datos.**
31. **Al inicio crear la estructura completa de carpetas y archivos.** (Hecho)
32. **Proyecto totalmente funcional.**
33. **Finalmente ejecutar servidor en el puerto 8043.**
      * Ejecuta: `python manage.py runserver 8043`
