# Estudiar la implementación de la API REST con NestJS y PostgreSQL

## Objetivos

### Requisitos

- Establecer `openapi.yaml` como contrato formal de la API y analizar sus operaciones, esquemas y respuestas de error (`400`, `404`, `422`).
- Definir la estructura del proyecto NestJS (módulos, controladores, servicios, DTOs) que cubra `listTasks`, `getTaskById`, `createTask`, `updateTask` y `deleteTask`.
- Comparar y seleccionar el ORM para PostgreSQL (TypeORM vs Prisma), justificando la decisión.
- Seleccionar las bibliotecas de validación de DTOs y mapeo a las respuestas de la especificación, y de configuración del entorno.
- Determinar el modelado de datos en PostgreSQL coherente con los esquemas `Task`, `TaskCreate` y `TaskPatch`.
- Documentar las decisiones técnicas pendientes y los requisitos previos a la implementación.

### Criterios de aceptación

- El estudio cubre todas las operaciones y esquemas de `openapi.yaml`.
- Cada operación tiene asignado su flujo (controller → service → repositorio/ORM) y sus respuestas.
- La elección de ORM y bibliotecas queda documentada y justificada.
- El resultado define los pasos necesarios para iniciar la implementación y quedará reflejado en `context/current-feature.md`.

## Estudio de la implementación (issue #11)

### Contrato formal en `openapi.yaml`

Especificación OpenAPI 3.1.0, servidor de desarrollo `http://localhost`. Sin seguridad. Define 5 operaciones sobre el recurso `tasks`:

| Operación | Método y ruta | Éxito | Errores declarados |
| --- | --- | --- | --- |
| `listTasks` | `GET /tasks` | `200` (array de `Task`) | — |
| `createTask` | `POST /tasks` | `201` (`Task`) | `422` (`ValidationError`) |
| `getTaskById` | `GET /tasks/{id}` | `200` (`Task`) | `404` (`NotFound`) |
| `updateTask` | `PATCH /tasks/{id}` | `200` (`Task`) | `400` (`MalformedBody`), `422` (`ValidationError`), `404` (`NotFound`) |
| `deleteTask` | `DELETE /tasks/{id}` | `204` (sin cuerpo) | `404` (`NotFound`) |

Esquemas relevantes:

- `Task`: `id` (integer, `readOnly`, requerido, lo genera el servidor), `title` (requerido), `description` (opcional), `completed` (boolean, requerido en la respuesta).
- `TaskCreate`: `title` requerido; `description` y `completed` opcionales; `completed` con `default: false`; `additionalProperties: false` (se rechazan campos extra).
- `TaskPatch`: todos los campos opcionales; `additionalProperties: false`.
- `Error`: único campo `error` (string, requerido). Las tres respuestas de error (`400`, `404`, `422`) usan este esquema.
- `TaskId` (parámetro de ruta): `integer`, requerido.

### Estructura del proyecto NestJS

Estructura propuesta, siguiendo convenciones oficiales de Nest y del módulo de tareas:

```
src/
  main.ts                    # Bootstrap: ValidationPipe global, prefijo opcional, filtros
  app.module.ts              # ConfigModule + PrismaModule + TasksModule
  prisma/
    prisma.module.ts         # Módulo que provee PrismaService (global)
    prisma.service.ts        # extends PrismaClient, conecta/desconecta en el ciclo de vida
  tasks/
    tasks.module.ts
    tasks.controller.ts      # 5 endpoints, delega en TasksService
    tasks.service.ts         # Reglas de negocio + acceso a datos vía PrismaService
    dto/
      create-task.dto.ts     # Contrato TaskCreate
      update-task.dto.ts     # Contrato TaskPatch
  common/
    exceptions/
      http-exception.filter.ts  # Devuelve siempre { error: string } (esquema `Error`)
```

Mapeo de cada operación a su flujo (controller → service → DB):

| Operación | Controller | Service | Prisma Client |
| --- | --- | --- | --- |
| `listTasks` | `GET /tasks` → `list()` | `findAll()` | `task.findMany()` |
| `createTask` | `POST /tasks` → `create(dto)` | `create(dto)` | `task.create()` |
| `getTaskById` | `GET /tasks/:id` → `findOne(id)` | `findOne(id)` | `task.findUnique()` |
| `updateTask` | `PATCH /tasks/:id` → `update(id, dto)` | `update(id, dto)` | `task.update()` |
| `deleteTask` | `DELETE /tasks/:id` → `remove(id)` | `remove(id)` | `task.delete()` |

- `200`/`201`/`204` y `Task` se devuelven tal cual: los objetos planos de Prisma coinciden con el esquema `Task`.
- `findUnique`/`update`/`delete` que no encuentran registro deben traducirse a `NotFound` (404).

### Elección del ORM para PostgreSQL: Prisma

Comparación orientada a este proyecto (una única entidad sin relaciones):

| Criterio | TypeORM | Prisma |
| --- | --- | --- |
| Integración con Nest | Módulo oficial `@nestjs/typeorm` | Receta oficial de Nest + patrón `PrismaService` con DI |
| Type-safety | Parcial: se pierde en `select`/`joins`; tipos pueden no reflejar el runtime | Completa: el cliente generado tipa la forma exacta de cada query |
| Migraciones | `typeorm migration:*` (migraciones en clave de código) | `prisma migrate` (SQL generado y versionado) |
| Modelo | Entidades decoradas (clases) | `schema.prisma` declarativo + generación de cliente |
| Resultados | Instancias de entidad (requieren mapeo) | Objetos planos listos como respuestas JSON |
| Estado del ecosistema | Cambios de API recientes (1.x) que en algunos casos rompen versiones previas | Cliente estable Prisma 7.x, gran madurez y documentación |

Decisión: **Prisma**.

Justificación: para esta API el factor decisivo no es rendimiento ni transacciones avanzadas (una sola tabla), sino la coherencia de tipos entre el esquema y las respuestas de la especificación, migraciones simples y corte de dependencias. Prisma devuelve objetos planos que mapean directamente al esquema `Task` (sin capa de transformación), ofrece type-safety completa y encaja con la DI de Nest mediante un `PrismaService`. TypeORM también es viable, pero su integración requiere entidades con decoradores que luego hay que serializar a los DTOs de respuesta.

### Bibliotecas de validación y configuración

- **`class-validator` + `class-transformer`**: `ValidationPipe` global con `transform: true`, `whitelist: true` y `forbidNonWhitelisted: true`. El `whitelist`/`forbidNonWhitelisted` implementa `additionalProperties: false` rechazando campos extra de `TaskCreate`/`TaskPatch`.
  - El `ValidationPipe` por defecto responde `400`; para cumplir la especificación se configura `exceptionFactory` para devolver `422` con `{ error: "Validation failed" }`.
- **`@nestjs/config`**: lectura de variables de entorno (p. ej. `DATABASE_URL`) vía `.env`.
- **Filtro global de excepciones (`HttpExceptionFilter`)**: garantiza que todo error cumpla el esquema `Error` (`{ error: string }`), preservando el código HTTP (`400`, `404`, `422`).
- **DTOs**:
  - `CreateTaskDto`: `title` `@IsString()` + `@IsNotEmpty()`; `description` `@IsString()` + `@IsOptional()`; `completed` `@IsBoolean()` + `@IsOptional()`. El `default: false` se aplica en el modelo de datos (ver modelado).
  - `UpdateTaskDto`: mismos decoradores pero todos `@IsOptional()`.

### Modelado de datos en PostgreSQL

Tabla `tasks`, coherente con los esquemas de la especificación:

| Columna | Tipo | Restricciones | Notas |
| --- | --- | --- | --- |
| `id` | `serial` / `IDENTITY` | PK | Generado por el servidor ⇒ `readOnly` en `Task` |
| `title` | `varchar(255)` | `NOT NULL` | Requerido en `TaskCreate` |
| `description` | `text` | `NULL` | Ausente en la respuesta si es nula (opcional) |
| `completed` | `boolean` | `NOT NULL DEFAULT false` | `default: false` en `TaskCreate`/`TaskPatch` |

SQL equivalente:

```sql
CREATE TABLE tasks (
  id          serial PRIMARY KEY,
  title       varchar(255) NOT NULL,
  description text,
  completed   boolean NOT NULL DEFAULT false
);
```

Esquema Prisma equivalente:

```prisma
model Task {
  id          Int     @id @default(autoincrement())
  title       String
  description String?
  completed   Boolean @default(false)
}
```

Coherencia con la especificación:

- `id` autogenerado cubre `readOnly: true` de `Task` ("generado por el servidor").
- `completed DEFAULT false` implementa el `default: false` de `TaskCreate` sin lógica extra en el servicio.
- `description` nula se omite (o se serializa) para no aparecer como campo requerido.

### Manejo de errores según la especificación

| Caso | Código | Mecanismo | Cuerpo |
| --- | --- | --- | --- |
| Body JSON malformado | `400` | Error de `body-parser` capturado por el filtro global | `{ "error": "Malformed request body" }` |
| Fallo de validación (DTO y campos extra) | `422` | `ValidationPipe.exceptionFactory` personalizado | `{ "error": "Validation failed" }` |
| Recurso inexistente | `404` | `NotFoundException` (o `notFound` de Prisma) | `{ "error": "Task not found" }` |

Decisión de diseño: `GET/PATCH/DELETE /tasks/{id}` con `id` no entero responderán `400` con el esquema `Error` (vía `ParseIntPipe`), aunque la especificación no lo declare explícitamente; se documenta como comportamiento asumido.

### Requisitos previos para la implementación

- **Node.js v24.16.0** (instalado vía Windows) y **npm 11.13.0** disponibles en el entorno.
- **NestJS v12** (stable, agosto 2026) con `@nestjs/cli` para el scaffolding.
- **Prisma 7.x** y **@nestjs/config 12.x**.
- **PostgreSQL no está instalado localmente y no hay Docker** en el entorno: requisito previo instalar PostgreSQL (local o WSL) o provisionar una instancia (p. ej. contenedor/hosted) con la conexión `DATABASE_URL`. Es el único bloqueante de infraestructura detectado.

## Notas

- Fuente de referencia: issue #11 y definición formal de la API en `openapi.yaml`.
- La rama de trabajo es `feature/6-study-nestjs-postgresql-implementation`.
- El estudio concluye con: estructura NestJS definida, ORM **Prisma** seleccionado y justificado, bibliotecas de validación/configuración elegidas, modelado PostgreSQL coherente con `Task`/`TaskCreate`/`TaskPatch`, y requisitos previos documentados (falta instalar/provisionar PostgreSQL).
- Decisiones asumidas y documentadas: `id` copia no entero → `400`; longitud límite de `title` (255); `description` nula omitida en la respuesta.

## Histórico