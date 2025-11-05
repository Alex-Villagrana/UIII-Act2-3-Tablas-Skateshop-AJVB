### 0\. 📂 Estructura de Carpetas y Archivos

```
UIII_Skateshop_0402/
├── .venv/
├── backend_Skateshop/
│   ├── settings.py
│   ├── urls.py
│   └── ...
├── app_Skateshop/
│   ├── templates/
│   │   ├── base.html
│   │   ├── header.html
│   │   ├── navbar.html    <- ACTUALIZADO con Proveedores y Clientes.
│   │   ├── footer.html
│   │   ├── inicio.html
│   │   └── cliente/             <- ¡CAMBIADO de 'categoria' a 'cliente'!
│   │       ├── agregar_cliente.html   <- CÓDIGO INCLUIDO.
│   │       ├── ver_clientes.html      <- CÓDIGO INCLUIDO.
│   │       ├── actualizar_cliente.html <- CÓDIGO INCLUIDO.
│   │       └── borrar_cliente.html    <- CÓDIGO INCLUIDO.
│   ├── admin.py                   <- ACTUALIZADO para los 3 modelos.
│   ├── models.py                  <- ACTUALIZADO para los 3 modelos.
│   ├── urls.py                    <- ACTUALIZADO para el CRUD de Cliente.
│   └── views.py                   <- ACTUALIZADO para el CRUD de Cliente.
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
    # MODELO: CLIENTE
    # ==========================================
    class Cliente(models.Model):
        nombre = models.CharField(max_length=100)
        apellido = models.CharField(max_length=100)
        email = models.EmailField(unique=True)
        telefono = models.CharField(max_length=15, blank=True, null=True)
        fecha_registro = models.DateField(auto_now_add=True)
        activo = models.BooleanField(default=True)
        
        def __str__(self):
            return f"{self.nombre} {self.apellido}"

    # ==========================================
    # MODELO: PROVEEDOR
    # ==========================================
    class Proveedor(models.Model):
        nombre_empresa = models.CharField(max_length=100, unique=True)
        contacto = models.CharField(max_length=100)
        telefono = models.CharField(max_length=15)
        email = models.EmailField(unique=True)
        fecha_registro = models.DateField(auto_now_add=True)
        activo = models.BooleanField(default=True)

        def __str__(self):
            return self.nombre_empresa
            
    # ==========================================
    # MODELO: PRODUCTO (Actualizado para usar Proveedor)
    # ==========================================
    class Producto(models.Model):
        nombre = models.CharField(max_length=100)
        descripcion = models.TextField(blank=True, null=True)
        precio = models.DecimalField(max_digits=10, decimal_places=2)
        stock = models.PositiveIntegerField(default=0)
        # Relacionado con Proveedor (antes era Categoria)
        proveedor = models.ForeignKey(Proveedor, on_delete=models.CASCADE, related_name="productos") 
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

### Segunda parte: Vistas, Templates y Enrutamiento (CRUD Cliente)

13. **Primero trabajamos con el MODELO: CLIENTE**

14. **En `views.py` de `app_Skateshop` crear las funciones con sus códigos correspondientes:**

      * **`inicio_skateshop`**, **`agregar_cliente`**, **`actualizar_cliente`**, **`realizar_actualizacion_cliente`**, **`borrar_cliente`**.

    <!-- end list -->

    ```python
    # Contenido de app_Skateshop/views.py
    from django.shortcuts import render, redirect, get_object_or_404
    from .models import Cliente, Producto, Proveedor # Importamos los modelos

    # 1. Función de Inicio
    def inicio_skateshop(request):
        return render(request, 'inicio.html')

    # 2. CRUD: Agregar Cliente
    def agregar_cliente(request):
        if request.method == 'POST':
            # No validar entrada de datos:
            Cliente.objects.create(
                nombre=request.POST.get('nombre'),
                apellido=request.POST.get('apellido'),
                email=request.POST.get('email'),
                telefono=request.POST.get('telefono'),
                activo=request.POST.get('activo', False) == 'on' 
            )
            return redirect('ver_clientes')
        return render(request, 'cliente/agregar_cliente.html')

    # 3. CRUD: Ver Clientes (Listado)
    def ver_clientes(request):
        clientes = Cliente.objects.all().order_by('apellido')
        context = {'clientes': clientes}
        return render(request, 'cliente/ver_clientes.html', context)

    # 4. CRUD: Actualizar Cliente (Mostrar Formulario)
    def actualizar_cliente(request, pk):
        cliente_a_actualizar = get_object_or_404(Cliente, pk=pk)
        context = {'cliente': cliente_a_actualizar}
        return render(request, 'cliente/actualizar_cliente.html', context)

    # 5. CRUD: Actualizar Cliente (Procesar POST)
    def realizar_actualizacion_cliente(request, pk):
        cliente = get_object_or_404(Cliente, pk=pk)
        if request.method == 'POST':
            cliente.nombre = request.POST.get('nombre')
            cliente.apellido = request.POST.get('apellido')
            cliente.email = request.POST.get('email')
            cliente.telefono = request.POST.get('telefono')
            cliente.activo = request.POST.get('activo', False) == 'on'
            cliente.save()
            return redirect('ver_clientes')
        return redirect('actualizar_cliente', pk=pk)

    # 6. CRUD: Borrar Cliente
    def borrar_cliente(request, pk):
        cliente = get_object_or_404(Cliente, pk=pk)
        if request.method == 'POST':
            cliente.delete()
            return redirect('ver_clientes')
        context = {'cliente': cliente}
        return render(request, 'cliente/borrar_cliente.html', context)
    ```

15. **Crear la carpeta `templates` dentro de `app_Skateshop`.**

16. **En la carpeta `templates` crear los archivos HTML:**

      * `base.html`, `header.html`, `navbar.html`, `footer.html`, `inicio.html`.

17. **En el archivo `base.html` agregar Bootstrap para CSS y JS.** (Mismo código anterior)

18. **En el archivo `navbar.html` incluir las opciones:** (ACTUALIZADO para Clientes y Proveedores)

    ```html
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
        <div class="container-fluid">
            <a class="navbar-brand" href="#"><i class="bi bi-gear-fill me-2"></i>Sistema de Administración Skateshop</a>
            <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNavDropdown" aria-controls="navbarNavDropdown" aria-expanded="false" aria-label="Toggle navigation"><span class="navbar-toggler-icon"></span></button>
            <div class="collapse navbar-collapse" id="navbarNavDropdown">
                <ul class="navbar-nav">
                    <li class="nav-item"><a class="nav-link" href="{% url 'inicio_skateshop' %}"><i class="bi bi-house-door-fill me-1"></i>Inicio</a></li>
                    
                    <li class="nav-item dropdown">
                        <a class="nav-link dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown"><i class="bi bi-people-fill me-1"></i>Clientes</a>
                        <ul class="dropdown-menu">
                            <li><a class="dropdown-item" href="{% url 'agregar_cliente' %}">Agregar Cliente</a></li>
                            <li><a class="dropdown-item" href="{% url 'ver_clientes' %}">Ver Clientes</a></li>
                            <li><a class="dropdown-item disabled" href="#">Actualizar Cliente</a></li>
                            <li><a class="dropdown-item disabled" href="#">Borrar Cliente</a></li>
                        </ul>
                    </li>

                    <li class="nav-item dropdown">
                        <a class="nav-link dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown"><i class="bi bi-truck-flatbed me-1"></i>Proveedores</a>
                        <ul class="dropdown-menu">
                            <li><a class="dropdown-item" href="#">Agregar Proveedor</a></li>
                            <li><a class="dropdown-item" href="#">Ver Proveedores</a></li>
                            <li><a class="dropdown-item" href="#">Actualizar Proveedor</a></li>
                            <li><a class="dropdown-item" href="#">Borrar Proveedor</a></li>
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

19. **En el archivo `footer.html` incluir:** (Mismo código anterior)

20. **En el archivo `inicio.html` colocar información del sistema...** (Mismo código anterior)

21. **Crear la subcarpeta `cliente` dentro de `app_Skateshop/templates/`**

22. **Crear los archivos HTML con su código correspondiente dentro de `app_Skateshop/templates/cliente`:** (Adaptados a `Cliente`)

      * **`agregar_cliente.html`**
        ```html
        {% extends 'base.html' %}
        {% block title %}Agregar Cliente{% endblock %}
        {% block content %}
        <div class="card shadow-sm">
            <div class="card-header bg-primary text-white"><h5 class="mb-0"><i class="bi bi-people-fill me-2"></i>Agregar Nuevo Cliente</h5></div>
            <div class="card-body">
                <form method="post" action="{% url 'agregar_cliente' %}"> {% csrf_token %}
                    <div class="row">
                        <div class="col-md-6 mb-3"><label for="nombre" class="form-label">Nombre:</label><input type="text" class="form-control" id="nombre" name="nombre" required></div>
                        <div class="col-md-6 mb-3"><label for="apellido" class="form-label">Apellido:</label><input type="text" class="form-control" id="apellido" name="apellido" required></div>
                    </div>
                    <div class="mb-3"><label for="email" class="form-label">Email:</label><input type="email" class="form-control" id="email" name="email" required></div>
                    <div class="mb-3"><label for="telefono" class="form-label">Teléfono:</label><input type="text" class="form-control" id="telefono" name="telefono"></div>
                    <div class="form-check mb-4"><input class="form-check-input" type="checkbox" name="activo" id="activo" checked><label class="form-check-label" for="activo">Activo</label></div>
                    <button type="submit" class="btn btn-success"><i class="bi bi-person-plus-fill me-2"></i>Guardar Cliente</button>
                    <a href="{% url 'ver_clientes' %}" class="btn btn-secondary ms-2">Cancelar</a>
                </form>
            </div>
        </div>
        {% endblock %}
        ```
      * **`ver_clientes.html`**
        ```html
        {% extends 'base.html' %}
        {% block title %}Ver Clientes{% endblock %}
        {% block content %}
        <div class="card shadow-sm">
            <div class="card-header bg-info text-dark"><h5 class="mb-0"><i class="bi bi-list-task me-2"></i>Lista de Clientes</h5></div>
            <div class="card-body">
                <a href="{% url 'agregar_cliente' %}" class="btn btn-primary mb-3"><i class="bi bi-plus me-1"></i>Agregar Cliente</a>
                <div class="table-responsive">
                    <table class="table table-hover table-striped">
                        <thead class="table-light"><tr><th>ID</th><th>Nombre Completo</th><th>Email</th><th>Activo</th><th>Acciones</th></tr></thead>
                        <tbody>
                            {% for cliente in clientes %}
                            <tr>
                                <td>{{ cliente.pk }}</td>
                                <td>{{ cliente.nombre }} {{ cliente.apellido }}</td>
                                <td>{{ cliente.email }}</td>
                                <td><i class="bi {% if cliente.activo %}bi-check-circle-fill text-success{% else %}bi-x-circle-fill text-danger{% endif %}"></i></td>
                                <td>
                                    <a href="#" class="btn btn-sm btn-outline-primary" title="Ver"><i class="bi bi-eye"></i></a>
                                    <a href="{% url 'actualizar_cliente' pk=cliente.pk %}" class="btn btn-sm btn-outline-warning" title="Editar"><i class="bi bi-pencil"></i></a>
                                    <a href="{% url 'borrar_cliente' pk=cliente.pk %}" class="btn btn-sm btn-outline-danger" title="Borrar"><i class="bi bi-trash"></i></a>
                                </td>
                            </tr>
                            {% empty %}
                            <tr><td colspan="5" class="text-center">No hay clientes registrados.</td></tr>
                            {% endfor %}
                        </tbody>
                    </table>
                </div>
            </div>
        </div>
        {% endblock %}
        ```
      * **`actualizar_cliente.html`**
        ```html
        {% extends 'base.html' %}
        {% block title %}Actualizar Cliente{% endblock %}
        {% block content %}
        <div class="card shadow-sm">
            <div class="card-header bg-warning text-dark"><h5 class="mb-0"><i class="bi bi-pencil-square me-2"></i>Actualizar Cliente</h5></div>
            <div class="card-body">
                <form method="post" action="{% url 'realizar_actualizacion_cliente' pk=cliente.pk %}"> 
                    {% csrf_token %}
                    <h6 class="card-subtitle mb-3 text-muted">Editando Cliente: {{ cliente.nombre }} {{ cliente.apellido }}</h6>
                    <div class="row">
                        <div class="col-md-6 mb-3"><label for="nombre" class="form-label">Nombre:</label><input type="text" class="form-control" id="nombre" name="nombre" value="{{ cliente.nombre }}" required></div>
                        <div class="col-md-6 mb-3"><label for="apellido" class="form-label">Apellido:</label><input type="text" class="form-control" id="apellido" name="apellido" value="{{ cliente.apellido }}" required></div>
                    </div>
                    <div class="mb-3"><label for="email" class="form-label">Email:</label><input type="email" class="form-control" id="email" name="email" value="{{ cliente.email }}" required></div>
                    <div class="mb-3"><label for="telefono" class="form-label">Teléfono:</label><input type="text" class="form-control" id="telefono" name="telefono" value="{{ cliente.telefono|default_if_none:'' }}"></div>
                    <div class="form-check mb-4"><input class="form-check-input" type="checkbox" name="activo" id="activo" {% if cliente.activo %}checked{% endif %}><label class="form-check-label" for="activo">Activo</label></div>
                    <button type="submit" class="btn btn-success"><i class="bi bi-check-circle-fill me-2"></i>Guardar Cambios</button>
                    <a href="{% url 'ver_clientes' %}" class="btn btn-secondary ms-2">Cancelar</a>
                </form>
            </div>
        </div>
        {% endblock %}
        ```
      * **`borrar_cliente.html`**
        ```html
        {% extends 'base.html' %}
        {% block title %}Borrar Cliente{% endblock %}
        {% block content %}
        <div class="alert alert-danger shadow-sm" role="alert">
            <h4 class="alert-heading"><i class="bi bi-exclamation-triangle-fill me-2"></i>¡Advertencia!</h4>
            <p>Estás a punto de **eliminar permanentemente** al cliente:</p>
            <ul class="list-group mb-4">
                <li class="list-group-item"><strong>ID:</strong> {{ cliente.pk }}</li>
                <li class="list-group-item"><strong>Nombre:</strong> {{ cliente.nombre }} {{ cliente.apellido }}</li>
                <li class="list-group-item"><strong>Email:</strong> {{ cliente.email }}</li>
            </ul>
            <hr>
            <p class="mb-0">¿Estás seguro de que deseas continuar con la eliminación?</p>
            <form method="post" action="{% url 'borrar_cliente' pk=cliente.pk %}" class="mt-3">
                {% csrf_token %}
                <button type="submit" class="btn btn-danger me-2"><i class="bi bi-trash-fill me-2"></i>Sí, Eliminar Cliente</button>
                <a href="{% url 'ver_clientes' %}" class="btn btn-secondary">Cancelar</a>
            </form>
        </div>
        {% endblock %}
        ```

23. **Procedimiento para crear el archivo `urls.py` en `app_Skateshop`** (Actualizado a Cliente)

    ```python
    # Contenido de app_Skateshop/urls.py (ACTUALIZADO)
    from django.urls import path
    from . import views
    urlpatterns = [
        path('', views.inicio_skateshop, name='inicio_skateshop'),
        
        # Rutas CRUD de Clientes
        path('cliente/agregar/', views.agregar_cliente, name='agregar_cliente'),
        path('cliente/ver/', views.ver_clientes, name='ver_clientes'),
        path('cliente/actualizar/<int:pk>/', views.actualizar_cliente, name='actualizar_cliente'),
        path('cliente/actualizar_post/<int:pk>/', views.realizar_actualizacion_cliente, name='realizar_actualizacion_cliente'),
        path('cliente/borrar/<int:pk>/', views.borrar_cliente, name='borrar_cliente'),
        
        # Rutas futuras para Producto y Proveedor (solo placeholders)
        # path('producto/...', views.alguna_funcion_producto, name='...'),
        # path('proveedor/...', views.alguna_funcion_proveedor, name='...'),
    ]
    ```

24. **Procedimiento para agregar `app_Skateshop` en `settings.py` de `backend_Skateshop`.** (Mismo código anterior)

25. **Realizar las configuraciones correspondientes en `urls.py` de `backend_Skateshop`** (Mismo código anterior)

26. **Procedimiento para registrar los modelos en `admin.py` y volver a realizar las migraciones.** (Actualizado para los 3 modelos)

    ```python
    # Contenido de app_Skateshop/admin.py (ACTUALIZADO)
    from django.contrib import admin
    from .models import Cliente, Producto, Proveedor # Importar los 3 modelos

    @admin.register(Cliente)
    class ClienteAdmin(admin.ModelAdmin):
        list_display = ('nombre', 'apellido', 'email', 'activo')
        search_fields = ('nombre', 'apellido', 'email')

    @admin.register(Proveedor)
    class ProveedorAdmin(admin.ModelAdmin):
        list_display = ('nombre_empresa', 'contacto', 'telefono', 'activo')
        search_fields = ('nombre_empresa',)

    @admin.register(Producto)
    class ProductoAdmin(admin.ModelAdmin):
        list_display = ('nombre', 'precio', 'stock', 'proveedor', 'disponible')
        list_filter = ('disponible', 'proveedor')
        search_fields = ('nombre', 'marca')
    ```

-----

### Tercera parte: Estilo y Finalización

27. **Por lo pronto solo trabajar con “Cliente”; dejar pendiente Producto y Proveedor si aún no se ha terminado.**
28. **Utilizar colores suaves, atractivos y modernos.**
29. **Código de las páginas web sencillo.**
30. **No validar entrada de datos.**
31. **Al inicio crear la estructura completa de carpetas y archivos.**
32. **Proyecto totalmente funcional.**
33. **Finalmente ejecutar servidor en el puerto 8043.**
      * Ejecuta: `python manage.py runserver 8043`
