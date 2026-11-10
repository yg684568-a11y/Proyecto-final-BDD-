# Modelo de datos

Guardar aquí el diagrama E-R original, sus versiones revisadas, el modelo relacional y las decisiones de normalización.

## Estado de la fase

**Fase 2 — Diseño del modelo de datos: EN CURSO.**

Este documento contiene un inventario conceptual preliminar derivado de las decisiones funcionales registradas en `PROJECT_CONTEXT.md`. Todavía no es el modelo E-R definitivo ni un esquema SQL. Las entidades y relaciones se revisarán antes de aprobar las llaves, cardinalidades, restricciones y normalización.

## Inventario conceptual preliminar de entidades

### 1. Usuarios y seguridad

| Entidad candidata | Propósito |
|---|---|
| CuentaUsuario | Identidad de acceso, identificador de inicio de sesión, hash de contraseña y estado de la cuenta. |
| Rol | Catálogo de perfiles: administrador, Servicios Escolares, coordinador académico, profesor y alumno. |
| Auditoria | Registro de acciones relevantes, cambios, autorizaciones e intentos de acceso sin almacenar contraseñas. |
| Notificacion | Avisos internos dirigidos a los usuarios, con fecha, tipo y estado de lectura. |

### 2. Estructura académica

| Entidad candidata | Propósito |
|---|---|
| Alumno | Expediente del alumno y sus datos institucionales. |
| Profesor | Expediente del profesor. |
| Departamento | Catálogo de departamentos académicos. |
| ProfesorDepartamento | Historial de adscripción del profesor a departamentos. |
| Carrera | Catálogo de carreras. |
| PlanEstudio | Versiones de los planes de estudio asociados a una carrera. |
| AlumnoPlan | Historial de carrera y plan asignados a cada alumno, conservando la asignación vigente y las anteriores. |
| Materia | Catálogo reutilizable de asignaturas. |
| PlanMateria | Materias que integran una versión específica de un plan, con créditos, semestre sugerido y demás atributos curriculares. |
| Prerrequisito | Relaciones de seriación entre materias dentro del contexto curricular correspondiente. |

### 3. Periodos, grupos e inscripciones

| Entidad candidata | Propósito |
|---|---|
| PeriodoEscolar | Periodos ordinarios y extraordinarios, con fechas de inicio y fin. |
| Aula | Espacios físicos, edificio y capacidad para alumnos. |
| Grupo | Oferta de una materia en un periodo, con un profesor responsable y cupo. |
| HorarioGrupo | Sesiones del grupo con día, hora de inicio y fin y aula asignada. |
| Inscripcion | Relación histórica entre alumno y grupo; conserva intentos, estado, fechas y calificación final. |
| ListaEspera | Lista FIFO asociada a un grupo que ha alcanzado su cupo. |
| EntradaListaEspera | Posición e historial de participación de un alumno en una lista de espera. |
| OfertaVacante | Oferta de una vacante, fecha límite, aceptación, rechazo o vencimiento. |

### 4. Evaluación e historial académico

| Entidad candidata | Propósito |
|---|---|
| UnidadEvaluacion | Unidades de evaluación configuradas para una materia y sus ponderaciones. |
| CalificacionUnidad | Calificación de un alumno en una unidad de un intento de inscripción. |
| SolicitudCorreccionCalificacion | Solicitud del profesor y autorización del coordinador para corregir una calificación publicada. |
| ReconocimientoMateria | Registro de materias reconocidas, equivalentes o revalidadas al cambiar de carrera o plan. |

## Relaciones preliminares

Estas relaciones se desprenden del alcance aprobado; sus cardinalidades y restricciones se verificarán en el diagrama E-R:

- Una carrera puede tener varios planes de estudio; cada plan corresponde a una carrera.
- Un plan de estudio se relaciona con varias materias mediante `PlanMateria`; una materia puede aparecer en varios planes.
- Un alumno puede tener varias asignaciones históricas de carrera/plan, pero solo una vigente a la vez.
- Un departamento puede tener varios profesores adscritos; el historial se conserva mediante `ProfesorDepartamento`.
- Un periodo escolar puede ofrecer varios grupos.
- Cada grupo corresponde a una materia, tiene un único profesor responsable y puede contar con uno o varios registros de horario por validar.
- Un alumno puede tener varias inscripciones históricas; cada inscripción corresponde a un alumno y un grupo.
- Un grupo puede tener varias inscripciones, limitadas por el cupo establecido.
- Una lista de espera corresponde a un grupo y contiene entradas ordenadas FIFO.
- Una entrada de lista de espera puede generar una o más ofertas a lo largo de la historia, según las reglas de revalidación y vacantes.
- Una inscripción puede tener calificaciones de varias unidades de evaluación.
- Las notificaciones pertenecen a un usuario y las acciones relevantes generan registros de auditoría.

## Decisiones y puntos por validar antes de cerrar el E-R

1. Contrastar este inventario con el diagrama E-R original entregado al equipo. El PDF de guía localizado en el repositorio contiene las instrucciones del reporte, pero no incluye ese diagrama.
2. Definir los atributos completos y tipos de dato de cada entidad.
3. Establecer PK, FK, restricciones UNIQUE, NOT NULL y reglas de borrado lógico.
4. Confirmar las cardinalidades y opcionalidades, en especial horarios por grupo, aulas y relaciones históricas.
5. Definir el modelo de unidades de evaluación, ponderaciones y calificación final de acuerdo con la normativa que se logre verificar.
6. Incorporar los detalles de cambios de carrera/plan, equivalencias y revalidaciones sin perder la historia.
7. Revisar normalización y evitar duplicidad de datos antes de crear el esquema físico.

**Importante:** no crear todavía scripts definitivos en `database/schema`. Primero aprobar el diagrama conceptual, el modelo lógico y las restricciones.
