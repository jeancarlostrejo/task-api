# Task API (Laravel)

API REST sencilla para gestionar tareas (Tasks). Usa `JsonResource` para formatear uniformemente las respuestas y personaliza el mensaje JSON de los errores de Route Model Binding (404) para mayor seguridad.

## Instalación rapida

```bash

# Descarga o clona el repositorio
git clone <repo_url>
cd task-api

# Instalar dependencias
composer install

# Crea el archivo de variable de entorno y edita las variables
Copia en archivo .env.example y renombralo a .env

# Genera la clave de la aplicación
php artisan key:generate

# Migraciones
php artisan migrate

# Levantar servidor de desarrollo
php artisan serve
# Base URL por defecto
# http://127.0.0.1:8000
```

Base de las rutas: `/api`

## Modelo de Datos

Tabla `tasks`:
- `id`: integer, autoincremental
- `title`: string (requerido al crear)
- `description`: text, opcional
- `is_completed`: boolean, por defecto `false`
- `completed_at`: datetime, opcional; requerido si `is_completed=true`
- `created_at`, `updated_at`: timestamps

## Formato JSON (Resource)

Las respuestas usan `TaskResource`:
- `is_completed`: siempre boolean
- `completed_at`, `created_at`, `updated_at`: se formatean como `d/m/Y H:i:s`

Ejemplo de objeto `Task` en respuesta:
```json
{
  "id": 1,
  "title": "Comprar pan",
  "description": "En la panadería de la esquina",
  "is_completed": false,
  "completed_at": null,
  "created_at": "01/01/2026 10:00:00",
  "updated_at": "01/01/2026 10:00:00"
}
```

## Endpoints

### Listar tareas
- Método: `GET`
- URL: `/api/tasks`

Ejemplo:
```bash
curl -s http://127.0.0.1:8000/api/tasks
```

Respuesta (200):
```json
{
  "data": [
    {
      "id": 1,
      "title": "Comprar pan",
      "description": "En la panadería de la esquina",
      "is_completed": false,
      "completed_at": null,
      "created_at": "01/01/2026 10:00:00",
      "updated_at": "01/01/2026 10:00:00"
    }
  ]
}
```

### Crear tarea
- Método: `POST`
- URL: `/api/tasks`
- Body (JSON):
  - `title` (string, requerido, max 255)
  - `description` (string, opcional)
  - `is_completed` (boolean, opcional)
  - `completed_at` (date, opcional; requerido si `is_completed=true`)

Ejemplo:
```bash
curl -s -X POST http://127.0.0.1:8000/api/tasks \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Comprar pan",
    "description": "En la panadería de la esquina",
    "is_completed": false
  }'
```

Respuesta (201):
```json
{
  "data": {
    "id": 2,
    "title": "Comprar pan",
    "description": "En la panadería de la esquina",
    "is_completed": false,
    "completed_at": null,
    "created_at": "01/01/2026 10:05:00",
    "updated_at": "01/01/2026 10:05:00"
  }
}
```

Errores de validación (422), por ejemplo sin `title`:
```json
{
  "message": "The title field is required.",
  "errors": {
    "title": ["The title field is required."]
  }
}
```

### Ver tarea
- Método: `GET`
- URL: `/api/tasks/{task}`

Ejemplo:
```bash
curl -s http://127.0.0.1:8000/api/tasks/1
```

Respuesta (200):
```json
{
  "data": {
    "id": 1,
    "title": "Comprar pan",
    "description": "En la panadería de la esquina",
    "is_completed": false,
    "completed_at": null,
    "created_at": "01/01/2026 10:00:00",
    "updated_at": "01/01/2026 10:00:00"
  }
}
```

### Actualizar tarea (parcial)
- Método: `PATCH`
- URL: `/api/tasks/{task}`
- Body (JSON):
  - `title` (string, opcional)
  - `description` (string, opcional)
  - `is_completed` (boolean, opcional)
  - `completed_at` (date, opcional; requerido si `is_completed=true`)

Ejemplos:
```bash
# Marcar como completada (nota: si envías is_completed=true, debes enviar completed_at válido)
curl -s -X PATCH http://127.0.0.1:8000/api/tasks/1 \
  -H "Content-Type: application/json" \
  -d '{
    "is_completed": true,
    "completed_at": "2026-01-01 12:34:56"
  }'

# Actualización parcial de título y descripción
curl -s -X PATCH http://127.0.0.1:8000/api/tasks/1 \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Comprar pan integral",
    "description": "Con semillas"
  }'
```

Respuesta (200):
```json
{
  "data": {
    "id": 1,
    "title": "Comprar pan integral",
    "description": "Con semillas",
    "is_completed": true,
    "completed_at": "01/01/2026 12:34:56",
    "created_at": "01/01/2026 10:00:00",
    "updated_at": "01/01/2026 12:35:00"
  }
}
```

### Eliminar tarea
- Método: `DELETE`
- URL: `/api/tasks/{task}`

Ejemplo:
```bash
curl -i -X DELETE http://127.0.0.1:8000/api/tasks/1
```

Respuesta (204 No Content):
```
(no body)
```

## Manejo de errores 404 (Route Model Binding)

Para peticiones API que esperan JSON en producción, cuando falla el Route Model Binding se devuelve:
```json
{
  "message": "Resource Not Found"
}
```
Código HTTP: `404`.

Nota: este comportamiento está configurado en `bootstrap/app.php` y aplica en entorno de producción (`APP_ENV=production`) para rutas que cumplan `api/*` y esperen una respuesta json con `expectsJson()`.

## Notas
- `completed_at` acepta valores reconocibles como fecha por Laravel; el recurso siempre serializa en formato `d/m/Y H:i:s`.
- Las rutas de tareas actualmente no requieren autenticación.
- Los endpoints usan `TaskResource` para garantizar consistencia en los campos y formatos.
