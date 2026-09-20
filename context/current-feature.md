# Construir `openapi.yaml`

## Objetivos

### Requisitos

- Crear el fichero `openapi.yaml` completando el esqueleto actual (actualmente vacío).
- Definir el modelo `Task` y los esquemas de error.
- Definir los cinco endpoints: listar, obtener por ID, crear, modificar y eliminar.

### Decisiones a aplicar (fase de identificación)

- Versión de OpenAPI: `3.1.0`.
- Path del recurso: `/tasks` (inglés).
- Tipos y formatos de los campos de `Task`:
  - `id`: integer.
  - `title`: string.
  - `description`: string.
  - `completed`: boolean.
- Campos obligatorios de `Task`: `id`, `title` y `completed`. `description` es el único opcional.
- Método de modificación: `PATCH` (modificación parcial).
- Códigos de estado por operación:
  - `GET /tasks` → `200`.
  - `GET /tasks/{id}` → `200`, `404`.
  - `POST /tasks` → `201`, `422`.
  - `PATCH /tasks/{id}` → `200`, `400`, `404`.
  - `DELETE /tasks/{id}` → `204`, `404`.
  - Significado de los códigos de error: `400` cuerpo de la petición malformado; `422` errores de validación de campos; `404` recurso no encontrado.
- Formato de las respuestas de error: objeto JSON con un mensaje en inglés que ayude al cliente, p. ej. `{"error": "Task not found"}`.

### Criterios de aceptación

- `openapi.yaml` es una especificación OpenAPI válida.
- Cubre todas las operaciones y campos descritos en `spec.txt`.
- Respeta las decisiones registradas en la fase de identificación.

## Notas

- Fuente de referencia: descripción informal de `spec.txt` y decisiones acordadas en la fase de identificación (issue #2).
- Fuera del alcance de esta feature:
  - Valores por defecto (¿`completed=false`? ¿`id` autogenerado por el servidor?).
  - Paginación, filtrado y ordenación.
- La validez del fichero generado se verificará en la revisión (issue #4).

## Histórico