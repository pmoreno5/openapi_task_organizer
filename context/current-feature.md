# Corregir los problemas detectados en `openapi.yaml`

## Objetivos

### Requisitos

- Aplicar las correcciones necesarias sobre `openapi.yaml` para resolver los problemas detectados en la revisión y validación (issue #4):
  - Añadir la respuesta `422` (ValidationError) a `PATCH /tasks/{id}` por consistencia con `POST /tasks`.
  - `TaskCreate`: hacer `completed` opcional con `default: false`, resolviendo la ambigüedad de valores por defecto.
  - Marcar la propiedad `id` de `Task` como `readOnly: true` (generado por el servidor).
  - Eliminar el bloque `security: [{}]` de la raíz (no hay `securitySchemes` definidos).
  - Añadir `additionalProperties: false` a `TaskCreate` y `TaskPatch`.
- Volver a validar la especificación tras las correcciones para confirmar que se han resuelto.

### Criterios de aceptación

- No quedan problemas pendientes de la validación.
- El resultado respeta las decisiones y requisitos de la especificación.

## Notas

- Fuente de referencia: issue #5 y problemas detectados en la revisión (issue #4).
- Antes de corregir se integró `origin/main` en la rama (`openapi.yaml` con el contenido generado en el issue #3).
- La validación se realizó con `redocly` y pasa sin errores. Quedan 2 warnings no bloqueantes: `no-server-example.com` (URL de desarrollo `http://localhost`) y `operation-4xx-response` (el listado `GET /tasks` no declara respuestas 4XX).

## Histórico