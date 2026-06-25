# Task Manager API — Documentación del Proyecto

> Proyecto portfolio Backend Junior | Spring Boot 3 · MongoDB · JWT · Arquitectura Hexagonal

---

## ÍNDICE
1. [Qué estamos construyendo](#qué-estamos-construyendo)
2. [Historias de Usuario](#historias-de-usuario)
3. [Definition of Done (DoD)](#definition-of-done)
4. [Tickets por Sprint](#tickets-por-sprint)
5. [Flujo de Git — Cómo trabajar profesionalmente](#flujo-de-git)
6. [Relación Frontend · Backend · Base de Datos](#relación-entre-capas)

---

## Qué estamos construyendo

Una **API REST** para gestionar tareas personales. Piénsalo como el backend de Todoist o Any.do.

Un usuario puede:
- Registrarse y hacer login
- Crear, ver, editar y borrar sus tareas
- Cambiar el estado de una tarea (Pendiente → En progreso → Completada)
- Ver solo SUS tareas, no las de otros usuarios

**Stack tecnológico:**
- **Java 17 + Spring Boot 3** → el motor de la API
- **MongoDB Atlas** → base de datos en la nube
- **Spring Security + JWT** → autenticación y seguridad
- **Lombok** → menos código aburrido
- **Arquitectura Hexagonal** → código organizado y mantenible

---

## Historias de Usuario

Las historias de usuario siguen el formato:
> **"Como [tipo de usuario], quiero [acción], para [beneficio]"**

---

### ÉPICA 1 — Gestión de Tareas (CRUD)

#### US-01 · Crear una tarea
```
Como usuario autenticado,
quiero poder crear una tarea con título, descripción y estado,
para tener registradas mis pendientes en un solo lugar.
```

**Criterios de aceptación (Gherkin):**
```gherkin
Dado que soy un usuario autenticado
Cuando envío POST /api/tasks con título y descripción válidos
Entonces la API devuelve la tarea creada con id, status PENDING y fecha de creación
Y la tarea queda guardada en MongoDB

Dado que soy un usuario autenticado
Cuando envío POST /api/tasks sin título
Entonces la API devuelve 400 Bad Request con mensaje de error descriptivo
```

---

#### US-02 · Ver mis tareas
```
Como usuario autenticado,
quiero ver un listado de todas mis tareas,
para saber qué tengo pendiente en cada momento.
```

**Criterios de aceptación:**
```gherkin
Dado que soy un usuario autenticado con 3 tareas creadas
Cuando envío GET /api/tasks
Entonces recibo una lista con mis 3 tareas en formato JSON
Y NO veo las tareas de otros usuarios

Dado que soy un usuario autenticado sin tareas
Cuando envío GET /api/tasks
Entonces recibo una lista vacía []
```

---

#### US-03 · Ver el detalle de una tarea
```
Como usuario autenticado,
quiero ver el detalle completo de una tarea específica,
para consultar su descripción y estado actual.
```

**Criterios de aceptación:**
```gherkin
Dado que soy un usuario autenticado y la tarea con id "123" es mía
Cuando envío GET /api/tasks/123
Entonces recibo el detalle completo de esa tarea

Dado que soy un usuario autenticado
Cuando envío GET /api/tasks/999 y esa tarea no existe
Entonces recibo 404 Not Found con mensaje "Task not found"

Dado que soy un usuario autenticado
Cuando envío GET /api/tasks/456 y esa tarea es de otro usuario
Entonces recibo 403 Forbidden
```

---

#### US-04 · Editar una tarea
```
Como usuario autenticado,
quiero poder editar el título, descripción y estado de una tarea,
para mantener mis tareas actualizadas.
```

**Criterios de aceptación:**
```gherkin
Dado que soy un usuario autenticado y la tarea "123" es mía
Cuando envío PUT /api/tasks/123 con nuevos datos válidos
Entonces la tarea se actualiza y recibo la tarea modificada
Y el campo updatedAt se actualiza con la fecha actual

Dado que soy un usuario autenticado
Cuando envío PUT /api/tasks/123 con un status que no existe (ej: "INVENTADO")
Entonces recibo 400 Bad Request
```

---

#### US-05 · Eliminar una tarea
```
Como usuario autenticado,
quiero poder eliminar una tarea,
para limpiar mi listado de tareas que ya no necesito.
```

**Criterios de aceptación:**
```gherkin
Dado que soy un usuario autenticado y la tarea "123" es mía
Cuando envío DELETE /api/tasks/123
Entonces recibo 204 No Content
Y la tarea ya no existe en MongoDB

Dado que soy un usuario autenticado
Cuando envío DELETE /api/tasks/999 y no existe
Entonces recibo 404 Not Found
```

---

### ÉPICA 2 — Autenticación y Seguridad

#### US-06 · Registro de usuario
```
Como visitante de la aplicación,
quiero poder registrarme con email y contraseña,
para tener acceso a mis tareas desde cualquier lugar.
```

**Criterios de aceptación:**
```gherkin
Dado que soy un visitante
Cuando envío POST /auth/register con email y password válidos
Entonces se crea mi cuenta y recibo confirmación
Y mi password queda guardada hasheada (nunca en texto plano)

Dado que soy un visitante
Cuando envío POST /auth/register con un email ya registrado
Entonces recibo 409 Conflict con mensaje "Email already exists"

Dado que soy un visitante
Cuando envío POST /auth/register con password menor a 8 caracteres
Entonces recibo 400 Bad Request con mensaje descriptivo
```

---

#### US-07 · Login de usuario
```
Como usuario registrado,
quiero poder hacer login con mi email y contraseña,
para obtener un token y acceder a mis tareas.
```

**Criterios de aceptación:**
```gherkin
Dado que soy un usuario registrado
Cuando envío POST /auth/login con credenciales correctas
Entonces recibo un JWT token válido
Y puedo usar ese token en el header "Authorization: Bearer {token}"

Dado que soy un usuario registrado
Cuando envío POST /auth/login con password incorrecta
Entonces recibo 401 Unauthorized
Y NO recibo ningún token
```

---

#### US-08 · Protección de endpoints
```
Como administrador del sistema,
quiero que todos los endpoints de tareas estén protegidos,
para que solo usuarios autenticados puedan acceder a sus datos.
```

**Criterios de aceptación:**
```gherkin
Dado que soy un visitante sin token
Cuando envío GET /api/tasks sin header Authorization
Entonces recibo 401 Unauthorized

Dado que soy un usuario con token expirado
Cuando envío cualquier petición a /api/tasks
Entonces recibo 401 Unauthorized con mensaje "Token expired"
```

---

### ÉPICA 3 — Calidad y Documentación

#### US-09 · Documentación de la API
```
Como desarrollador que consume la API,
quiero tener documentación interactiva de todos los endpoints,
para poder probarlos sin necesidad de Postman.
```

**Criterios de aceptación:**
```gherkin
Dado que soy un desarrollador
Cuando accedo a GET /swagger-ui.html
Entonces veo todos los endpoints documentados con sus parámetros y respuestas
Y puedo probar los endpoints directamente desde el navegador
```

---

#### US-10 · Manejo global de errores
```
Como desarrollador que consume la API,
quiero recibir respuestas de error consistentes y descriptivas,
para poder gestionar los errores correctamente en el frontend.
```

**Criterios de aceptación:**
```gherkin
Dado que ocurre cualquier error en la API
Cuando la API devuelve un error
Entonces el formato es siempre: { timestamp, status, error, message, path }
Y nunca se expone información sensible del servidor en el error
```

---

## Definition of Done

Una historia de usuario se considera **DONE** cuando:

### Código
- [ ] El código compila sin errores
- [ ] Sigue la estructura de arquitectura hexagonal (domain / application / infrastructure / api)
- [ ] No hay código comentado ni TODOs sin resolver
- [ ] Los nombres de variables y métodos son descriptivos en inglés
- [ ] Se usan DTOs — la entidad nunca se expone directamente en el controller

### Tests
- [ ] Existe al menos un test unitario del servicio con JUnit + Mockito
- [ ] El test cubre el caso feliz (happy path) y al menos un caso de error
- [ ] Todos los tests pasan (`mvn test`)

### Git
- [ ] El código está en una rama `feature/nombre-feature`
- [ ] El commit tiene mensaje descriptivo: `feat: add task creation endpoint`
- [ ] Se ha hecho Pull Request a `develop`
- [ ] El PR tiene descripción de qué hace y por qué

### API
- [ ] El endpoint responde con el código HTTP correcto (200, 201, 204, 400, 401, 404...)
- [ ] Las validaciones funcionan y devuelven 400 con mensaje descriptivo
- [ ] Probado manualmente con Postman

---

## Tickets por Sprint

### SPRINT 0 — Setup (ya completado ✅)
| ID | Tarea | Estado |
|---|---|---|
| T-001 | Crear proyecto Spring Boot con Spring Initializr | ✅ Done |
| T-002 | Configurar MongoDB Atlas y conectar a la API | ✅ Done |
| T-003 | Crear estructura de paquetes hexagonal | ✅ Done |
| T-004 | Subir proyecto a GitHub con rama main y develop | ✅ Done |
| T-005 | Proteger application.properties con .gitignore | ✅ Done |
| T-006 | Crear application.properties.example | ✅ Done |

---

### SPRINT 1 — CRUD de Tareas
| ID | Tarea | US | Capa |
|---|---|---|---|
| T-007 | Crear entidad Task con Lombok y anotaciones MongoDB | US-01 | domain |
| T-008 | Crear enum TaskStatus (PENDING, IN_PROGRESS, COMPLETED) | US-01 | domain |
| T-009 | Crear TaskRepository extendiendo MongoRepository | US-01 | infrastructure |
| T-010 | Crear TaskService con método create() | US-01 | application |
| T-011 | Crear TaskService con método findAll() | US-02 | application |
| T-012 | Crear TaskService con método findById() | US-03 | application |
| T-013 | Crear TaskService con métodos update() y delete() | US-04, US-05 | application |
| T-014 | Crear TaskController con POST /api/tasks | US-01 | api |
| T-015 | Crear TaskController con GET /api/tasks | US-02 | api |
| T-016 | Crear TaskController con GET /api/tasks/{id} | US-03 | api |
| T-017 | Crear TaskController con PUT /api/tasks/{id} | US-04 | api |
| T-018 | Crear TaskController con DELETE /api/tasks/{id} | US-05 | api |
| T-019 | Añadir validaciones con @Valid y Bean Validation | US-01 | api |
| T-020 | Probar todos los endpoints con Postman | Todas | - |

---

### SPRINT 2 — Autenticación JWT
| ID | Tarea | US | Capa |
|---|---|---|---|
| T-021 | Crear entidad User | US-06 | domain |
| T-022 | Crear UserRepository | US-06 | infrastructure |
| T-023 | Implementar POST /auth/register con hash BCrypt | US-06 | api |
| T-024 | Crear JwtTokenProvider (generar y validar tokens) | US-07 | security |
| T-025 | Crear JwtAuthenticationFilter | US-08 | security |
| T-026 | Implementar POST /auth/login devolviendo JWT | US-07 | api |
| T-027 | Configurar SecurityFilterChain | US-08 | security |
| T-028 | Implementar UserDetailsService | US-08 | security |

---

### SPRINT 3 — Multiusuario
| ID | Tarea | US | Capa |
|---|---|---|---|
| T-029 | Añadir campo userId a Task | US-02 | domain |
| T-030 | Filtrar tareas por usuario autenticado en TaskService | US-02 | application |
| T-031 | Proteger que un usuario no acceda a tareas de otro | US-03 | application |
| T-032 | Añadir roles USER y ADMIN | US-08 | security |
| T-033 | Endpoint GET /api/admin/tasks solo para ADMIN | US-08 | api |

---

### SPRINT 4 — Calidad
| ID | Tarea | US | Capa |
|---|---|---|---|
| T-034 | Crear @ControllerAdvice para errores globales | US-10 | api |
| T-035 | Añadir Swagger/OpenAPI con springdoc | US-09 | api |
| T-036 | Escribir tests unitarios de TaskService con Mockito | Todas | test |
| T-037 | Refactorizar a puertos e interfaces (hexagonal completo) | Todas | application |

---

## Flujo de Git

### Las ramas que vas a usar

```
main          → código en producción. NUNCA se toca directamente.
develop       → integración. Aquí llegan todas las features terminadas.
feature/xxx   → una rama por cada ticket. Se hace PR a develop al terminar.
```

### Cómo trabajar en cada ticket — paso a paso

**1. Antes de empezar un ticket, crea su rama:**
```bash
git checkout develop          # asegúrate de estar en develop
git pull                      # trae los últimos cambios
git checkout -b feature/T-007-task-entity   # crea la rama del ticket
```

**2. Trabaja en el código normalmente en IntelliJ**

**3. Cuando termines, guarda los cambios:**
```bash
git add .
git commit -m "feat: create Task entity with Lombok and MongoDB annotations"
git push origin feature/T-007-task-entity
```

**4. Ve a GitHub y crea un Pull Request:**
- Base: `develop`
- Compare: `feature/T-007-task-entity`
- Título: `T-007 · Create Task entity`
- Descripción: qué hiciste y por qué

**5. Revisa el PR tú mismo (o pide review) y haz merge**

---

### Convención de mensajes de commit

```
feat:     nueva funcionalidad
fix:      corrección de bug
refactor: reorganización de código sin cambiar funcionalidad
test:     añadir o modificar tests
docs:     documentación
chore:    tareas de mantenimiento (dependencias, config...)
```

**Ejemplos reales:**
```
feat: add TaskRepository extending MongoRepository
feat: implement POST /api/tasks endpoint
fix: return 404 when task not found instead of 500
test: add unit tests for TaskService create method
docs: update README with API endpoints
```

---

### Comandos Git que usarás cada día

```bash
git status                    # ver qué archivos han cambiado
git diff                      # ver exactamente qué cambió línea a línea
git log --oneline             # ver historial de commits resumido
git checkout develop          # cambiar a la rama develop
git pull                      # traer cambios de GitHub
git checkout -b feature/xxx   # crear nueva rama
git add .                     # preparar todos los cambios
git commit -m "mensaje"       # guardar snapshot
git push origin feature/xxx   # subir rama a GitHub
```

---

## Relación entre capas

Esta tabla muestra cómo se relacionan Frontend, Backend y Base de Datos en cada historia:

| Acción del usuario | Frontend | Backend (nuestra API) | Base de Datos |
|---|---|---|---|
| Crear una tarea | Formulario → POST /api/tasks | TaskController → TaskService → TaskRepository | Inserta documento en colección "tasks" |
| Ver mis tareas | GET /api/tasks → renderiza lista | TaskController → TaskService → TaskRepository | SELECT * WHERE userId = "yo" |
| Hacer login | Formulario → POST /auth/login | AuthController → genera JWT | Busca usuario por email |
| Cambiar estado | Click en tarea → PUT /api/tasks/id | TaskController → TaskService → guarda | Actualiza campo "status" |

### ¿Por qué necesitamos cada capa?

```
📱 FRONTEND (React / HTML)
   "Muestra los datos y recoge lo que escribe el usuario"
   No sabe nada de MongoDB. Solo habla con la API.
        ↕ JSON por HTTP
🚪 API (TaskController)
   "Recibe las peticiones HTTP y las transforma en objetos Java"
   No sabe nada de MongoDB. Solo habla con el Service.
        ↕ Objetos Java
⚙️  APPLICATION (TaskService)
   "Aquí vive la lógica de negocio"
   "¿Puede este usuario borrar esta tarea? ¿El título es válido?"
   No sabe nada de MongoDB. Solo usa interfaces (puertos).
        ↕ Objetos Java
🗄️  INFRASTRUCTURE (TaskRepository)
   "Sabe cómo hablar con MongoDB"
   Si mañana cambiamos a PostgreSQL, solo cambia esta capa.
        ↕ Queries
💾 MONGODB
   "Guarda los datos para siempre"
```

---

*Task Manager API · Portfolio Backend Junior · Max Pachón López*
*Metodología Scrum · Sprints semanales · Arquitectura Hexagonal*
