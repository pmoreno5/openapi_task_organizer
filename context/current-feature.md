# API REST para gestionar tareas

## Objetivos

### Requisitos funcionales

- Modelo de datos de una tarea con los campos:
  - `id`
  - `title`
  - `description`
  - `completed`
- Operaciones de la API:
  - Consultar todas las tareas.
  - Consultar una tarea por su ID.
  - Crear una tarea.
  - Modificar una tarea.
  - Eliminar una tarea.

### Requisitos técnicos

- Definir una especificación OpenAPI para la API en `openapi.yaml`.
- La API es de tipo REST.

### Criterios de aceptación

- Se han extraído los requisitos de la descripción informal de `spec.txt`.
- Las ambigüedades quedan recogidas para su resolución en la fase de decisiones.

## Notas

- Ambigüedades detectadas pendientes de resolución:
  - Tipos de datos de cada campo (`id`: integer vs UUID/string; `title` y `description`: string; `completed`: boolean).
  - Sintaxis del path del recurso: `/tasks` (inglés) vs `/tareas`.
  - Modificación: `PUT` (sustitución total) vs `PATCH` (parcial).
  - Campos obligatorios (¿`title` siempre requerido? ¿`description` opcional?).
  - Valores por defecto (¿`completed=false`? ¿`id` autogenerado por el servidor?).
  - Códigos de estado y formato de errores (400, 404, 201, 204, etc.).
  - Paginación, filtrado y ordenación no especificados.

## Histórico