# Identificar las decisiones necesarias para la especificación OpenAPI

## Objetivos

### Decisiones acordadas

- Versión de OpenAPI: `3.1.0`.
- Tipos y formatos de los campos de `Task`:
  - `id`: integer.
  - `title`: string.
  - `description`: string.
  - `completed`: boolean.
- Campos obligatorios: `id`, `title` y `completed`. `description` es el único opcional.
- Path del recurso: `/tasks` (inglés).
- Método de modificación: `PATCH` (modificación parcial).
- Códigos de estado por operación:
  - `GET /tasks` → `200`.
  - `GET /tasks/{id}` → `200`, `404`.
  - `POST /tasks` → `201`, `422`.
  - `PATCH /tasks/{id}` → `200`, `400`, `404`.
  - `DELETE /tasks/{id}` → `204`, `404`.
  - `400`: cuerpo de la petición malformado.
  - `422`: errores de validación de campos.
  - `404`: recurso no encontrado.
- Formato de las respuestas de error: objeto JSON con un mensaje en inglés que ayude al cliente, p. ej. `{"error": "Task not found"}`.

### Criterios de aceptación

- Cada decisión queda registrada de forma explícita.
- Las decisiones son coherentes con la descripción informal de `spec.txt`.

## Notas

- Fuente de referencia: descripción informal de `spec.txt`.
- Las decisiones acordadas servirán de base para construir `openapi.yaml` (issue #3).
- Ambigüedades de `spec.txt` que quedan fuera del alcance de esta issue:
  - Valores por defecto (¿`completed=false`? ¿`id` autogenerado por el servidor?).
  - Paginación, filtrado y ordenación no especificados.

## Histórico