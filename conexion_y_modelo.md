# Actividad N° 2 – Conexión a Base de Datos y Uso del ORM en Django

## 1. Conexión a PostgreSQL

### Instalación del paquete necesario

Para conectar Django con PostgreSQL se instala el paquete `psycopg2-binary`, que actúa como traductor entre Django y el motor de base de datos:

```bash
pip install psycopg2-binary
```

### Creación de la base de datos

Se creó una base de datos vacía llamada `libreria` directamente desde el shell de PostgreSQL:

```sql
CREATE DATABASE libreria;
```

### Configuración en settings.py

En el archivo `libreria/settings.py` se reemplazó la configuración por defecto de SQLite por la conexión a PostgreSQL:

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'libreria',
        'USER': 'tu_usuario',
        'PASSWORD': 'tu_contraseña',
        'HOST': 'localhost',
        'PORT': '5432',
    }
}
```

> **Importante:** Las credenciales reales no se incluyen en este archivo. Se recomienda usar variables de entorno con `python-decouple` o `django-environ` para protegerlas.

**¿Qué significa cada campo?**

| Campo | Descripción |
|-------|-------------|
| `ENGINE` | Le indica a Django que use el backend de PostgreSQL |
| `NAME` | Nombre de la base de datos creada |
| `USER` | Usuario de PostgreSQL (por defecto `postgres`) |
| `PASSWORD` | Contraseña del usuario |
| `HOST` | Dirección del servidor (`localhost` porque está en la misma máquina) |
| `PORT` | Puerto estándar de PostgreSQL (`5432`) |

---

## 2. Definición del modelo

Se creó la app `catalogo` con el comando:

```bash
python manage.py startapp catalogo
```

Y se registró en `INSTALLED_APPS` dentro de `settings.py`:

```python
INSTALLED_APPS = [
    ...
    'catalogo',
]
```

### Modelo Libro

En el archivo `catalogo/models.py` se definió el siguiente modelo:

```python
from django.db import models

class Libro(models.Model):
    titulo = models.CharField(max_length=100)
    autor = models.CharField(max_length=50)
    anio_publicacion = models.IntegerField()
    disponible = models.BooleanField(default=True)

    def __str__(self):
        return self.titulo
```

**Descripción de cada campo:**

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `titulo` | `CharField` | Texto con máximo 100 caracteres |
| `autor` | `CharField` | Texto con máximo 50 caracteres |
| `anio_publicacion` | `IntegerField` | Número entero para el año |
| `disponible` | `BooleanField` | Verdadero o falso, por defecto `True` |

### Clave primaria por defecto

Django crea automáticamente un campo `id` de tipo entero autoincremental en cada modelo cuando no se define una clave primaria explícita. Este campo actúa como clave primaria y se incrementa automáticamente con cada nuevo registro (1, 2, 3...).

### Clave primaria compuesta manual

Si se quisiera definir una clave primaria compuesta manualmente, se usaría `UniqueConstraint` dentro de la clase `Meta`:

```python
class Libro(models.Model):
    titulo = models.CharField(max_length=100)
    autor = models.CharField(max_length=50)
    anio_publicacion = models.IntegerField()
    disponible = models.BooleanField(default=True)

    class Meta:
        constraints = [
            models.UniqueConstraint(fields=['titulo', 'autor'], name='pk_compuesta_libro')
        ]
```

Esto garantiza que no puedan existir dos libros con el mismo título **y** el mismo autor simultáneamente.

---

## 3. Aplicar migraciones

Se ejecutaron los siguientes comandos desde la terminal:

```bash
python manage.py makemigrations
```

**¿Qué hace?** Analiza los modelos definidos en `models.py` y genera archivos de migración en la carpeta `migrations/`. Estos archivos describen los cambios que se deben aplicar a la base de datos (crear tablas, agregar columnas, etc.). No modifica la base de datos todavía.

```bash
python manage.py migrate
```

**¿Qué hace?** Lee los archivos de migración generados y los ejecuta contra la base de datos, creando o modificando las tablas reales. También aplica las migraciones internas de Django (admin, auth, sessions, etc.).

---

## 4. Operaciones CRUD con el ORM

Las operaciones se realizaron desde el shell interactivo de Django:

```bash
python manage.py shell
```

Primero se importó el modelo:

```python
from catalogo.models import Libro
```

### Crear un nuevo libro

```python
libro = Libro.objects.create(
    titulo='Cien años de soledad',
    autor='Gabriel García Márquez',
    anio_publicacion=1967,
    disponible=True
)
```

`objects.create()` crea el registro y lo guarda en la base de datos en una sola operación.

### Listar todos los libros

```python
Libro.objects.all()
# Resultado: <QuerySet [<Libro: Cien años de soledad>]>
```

`objects.all()` retorna un QuerySet con todos los registros de la tabla.

### Buscar un libro por su título

```python
Libro.objects.get(titulo='Cien años de soledad')
# Resultado: <Libro: Cien años de soledad>
```

`objects.get()` retorna un único objeto que coincida con el filtro. Si no existe lanza una excepción `DoesNotExist`.

### Actualizar el campo disponible

```python
libro = Libro.objects.get(titulo='Cien años de soledad')
libro.disponible = False
libro.save()
```

Se obtiene el objeto, se modifica el atributo y se llama a `save()` para persistir el cambio en la base de datos.

### Eliminar un libro

```python
libro = Libro.objects.get(titulo='Cien años de soledad')
libro.delete()
# Resultado: (1, {'catalogo.Libro': 1})
```

`delete()` elimina el registro de la base de datos. El resultado indica cuántos objetos fueron eliminados.
