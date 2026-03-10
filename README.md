# Custom Swagger UI + drf-spectacular (Django)

Bu hujjat aynan ushbu loyihadagi custom Swagger UI (`templates/swagger_ui.html`) ni
`drf-spectacular` bilan ulash va ishlatish uchun yozilgan.

## 1. Kerakli paketlar

```bash
.venv/bin/pip install drf-spectacular djangorestframework
```

Agar JWT ishlatsangiz:

```bash
.venv/bin/pip install djangorestframework-simplejwt
```

## 2. Django sozlamalari

### `config/settings.py`

`INSTALLED_APPS` ichida quyidagilar bo'lishi kerak:

- `rest_framework`
- `drf_spectacular`

Template papkasi ham ulangan bo'lishi kerak:

```python
"DIRS": [os.path.join(BASE_DIR, "templates")]
```

### `config/restframework_settings.py`

Schema class ni spectacularga ulang:

```python
REST_FRAMEWORK = {
    "DEFAULT_SCHEMA_CLASS": "drf_spectacular.openapi.AutoSchema",
}
```

### `config/spectacular_settings.py`

Minimal ishlaydigan variant:

```python
SPECTACULAR_SETTINGS = {
    "TITLE": "My app",
    "VERSION": "1.0.0",
    "SERVE_INCLUDE_SCHEMA": False,
}
```

`TITLE` va `VERSION` ni custom UI tepadagi headerda avtomatik o'qiydi.

## 3. Custom Swagger view

### `myapp/views.py`

`SpectacularSwaggerView` dan voris qilib, custom template bering:

```python
from drf_spectacular.views import SpectacularSwaggerView


class CustomSwaggerView(SpectacularSwaggerView):
    template_name = "swagger_ui.html"
```

## 4. URL ulash

### `config/swagger_urls.py`

`schema`, `swagger-ui`, `redoc` yo'llarini ulang:

```python
from django.urls import path
from drf_spectacular.views import SpectacularAPIView, SpectacularRedocView
from myapp.views import CustomSwaggerView

swagger_urls = [
    path("schema/", SpectacularAPIView.as_view(), name="schema"),
    path("", CustomSwaggerView.as_view(), name="swagger-ui"),
    path("redoc/", SpectacularRedocView.as_view(), name="redoc"),
]
```

### `config/urls.py`

Asosiy urlconf ga qo'shing:

```python
from .swagger_urls import swagger_urls

urlpatterns += swagger_urls
```

## 5. Custom template ichida schema URL

### `templates/swagger_ui.html`

JS ichida schema endpoint quyidagicha olinishi kerak:

```javascript
const SCHEMA_URL = "{% url 'schema' %}";
```

Shunda UI har doim to'g'ri schema manzilidan o'qiydi.

## 6. Ishga tushirish

```bash
.venv/bin/python manage.py migrate
.venv/bin/python manage.py runserver
```

Ochish:

- Swagger UI: `http://127.0.0.1:8000/`
- OpenAPI schema: `http://127.0.0.1:8000/schema/`
- ReDoc: `http://127.0.0.1:8000/redoc/`
