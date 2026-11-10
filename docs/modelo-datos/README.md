# Modelo de datos

Guardar aquí el diagrama E-R original, sus versiones revisadas, el modelo relacional y las decisiones de normalización.

## Estado de la fase

**Fase 2 — Diseño del modelo de datos: EN CURSO.**

Este documento contiene un inventario conceptual preliminar derivado de las decisiones funcionales registradas en `PROJECT_CONTEXT.md`. Todavía no es el modelo E-R definitivo ni un esquema SQL. Las entidades y relaciones se revisarán antes de aprobar las llaves, cardinalidades, restricciones y normalización.

## Revisión del diagrama E-R original del equipo

**Estado:** revisado como modelo inicial; todavía no aprobado como modelo definitivo.

El diagrama actual incluye las entidades `Cursos`, `Horarios`, `Categorias`, `Profesores`, `Datos_contacto_p`, `Departamentos`, `Modulos`, `Calf_modulos`, `Historiales`, `Alumnos` y `Datos_contacto_a`.

### Elementos que pueden aprovecharse

| Elemento actual | Evaluación |
|---|---|
| `Departamentos` → `Profesores` | La relación 1:N coincide con la decisión de que cada profesor tenga un departamento vigente y que un departamento pueda tener varios profesores. |
| `Alumnos` → `Historiales` | Es una buena base para conservar varias inscripciones/intentos de un alumno. |
| `Historiales` → `Calf_modulos` | La relación permite asociar calificaciones parciales a una inscripción concreta. |
| `Cursos` → `Modulos` | Puede conservarse si los módulos representan unidades o componentes de evaluación de una materia. |
| `Datos_contacto_a` y `Datos_contacto_p` | Pueden mantenerse como datos de contacto separados si se definen correctamente sus llaves y restricciones 1:1. |
| `Categorias` → `Cursos` | Puede conservarse si la categoría tiene un significado real para el catálogo académico; ese significado debe aclararse. |

### Cambios estructurales necesarios

1. **Separar `Cursos` en `Materia` y `Grupo`.** La materia será el catálogo académico reutilizable. El grupo representará la oferta concreta de esa materia durante un periodo, con profesor responsable, cupo y programación. El profesor y las fechas de oferta no deben permanecer como atributos de la materia general.
2. **Incorporar `PeriodoEscolar`.** Cada grupo debe pertenecer a un periodo. Esto permite repetir la misma materia en distintos periodos sin duplicar su definición.
3. **Reubicar `Horarios` como `HorarioGrupo`.** Los horarios deben depender del grupo, no de la materia general. En la imagen solo se alcanza a identificar `hora_fin`; el diseño debe contemplar hora de inicio y fin, día, edificio y aula.
4. **Incorporar `Aula`.** Se necesita un catálogo de espacios físicos con capacidad para alumnos. La capacidad del grupo no puede superar la del aula asignada.
5. **Reinterpretar `Historiales` como `Inscripcion`.** Cada registro debe asociar alumno y grupo, conservar el periodo, estado, fechas e intento. Cada recursamiento será un registro independiente; no se debe sobrescribir el intento anterior. La calificación final debe distinguirse de las calificaciones parciales.
6. **Revisar `Modulos` y `Calf_modulos`.** Por su relación actual, parecen representar unidades o módulos evaluables y sus calificaciones. Si se confirma esa interpretación, conviene usar nombres claros como `UnidadEvaluacion` y `CalificacionUnidad`. El atributo `Estado` mezcla estados de entrega (`pendiente`, `entregado`) con resultados académicos (`aprobado`, `reprobado`); deben separarse o definirse con precisión.
7. **Incorporar carrera y plan de estudios.** Hacen falta `Carrera`, `PlanEstudio`, la relación de materias que integran cada plan y el historial de carrera/plan del alumno.
8. **Incorporar entidades de gestión.** El alcance aprobado también requiere cuentas/roles, lista de espera y sus ofertas, notificaciones, auditoría, prerrequisitos, reconocimiento de materias y solicitudes de corrección de calificaciones.
9. **Completar identificadores y restricciones.** El expediente del alumno necesita número de control obligatorio y único; el del profesor, número de empleado obligatorio y único. Los correos institucionales deben ser obligatorios y únicos en el sistema. Si los contactos permanecen separados, la relación 1:1 requiere una restricción `UNIQUE` en la FK del contacto y debe definirse si el registro de contacto es obligatorio u opcional.
10. **Revisar el catálogo `Categorias`.** El diagrama no permite determinar qué representan las categorías ni qué regla académica dependen de ellas. No se eliminarán ni se conservarán automáticamente hasta aclarar su función.

### Cardinalidades observadas y destino propuesto

- Departamento 1:N Profesor: conservar como asignación vigente, agregando historial de cambios de departamento.
- Alumno 1:N Inscripcion: conservar.
- Grupo 1:N Inscripcion: establecer para las inscripciones al grupo concreto.
- Materia 1:N Grupo: establecer; la misma materia puede ofertarse en distintos grupos y periodos.
- Grupo 1:N HorarioGrupo: establecer.
- Materia 1:N UnidadEvaluacion: conservar si los módulos actuales equivalen a unidades de evaluación.
- Inscripcion 1:N CalificacionUnidad: conservar la idea, para que las calificaciones pertenezcan a un intento específico.
- Profesor 1:N Grupo: sustituye la asignación actual Profesor 1:N Curso.
- Carrera 1:N PlanEstudio y PlanEstudio N:M Materia mediante una entidad intermedia: incorporar.

### Conclusión de la revisión

El diagrama actual sirve como punto de partida. No conviene desecharlo ni implementarlo tal como está. La modificación central será separar el catálogo de materias de la oferta de grupos por periodo y conectar las inscripciones y horarios con esos grupos. Después se agregará la estructura de carreras/planes y las entidades necesarias para los procesos aprobados.

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

1. Confirmar la interpretación de `Modulos` como unidades de evaluación y aclarar la función del catálogo `Categorias`.
2. Definir los atributos completos y tipos de dato de cada entidad.
3. Establecer PK, FK, restricciones UNIQUE, NOT NULL y reglas de borrado lógico.
4. Confirmar las cardinalidades y opcionalidades, en especial horarios por grupo, aulas y relaciones históricas.
5. Definir el modelo de unidades de evaluación, ponderaciones y calificación final de acuerdo con la normativa que se logre verificar.
6. Incorporar los detalles de cambios de carrera/plan, equivalencias y revalidaciones sin perder la historia.
7. Revisar normalización y evitar duplicidad de datos antes de crear el esquema físico.

**Importante:** no crear todavía scripts definitivos en `database/schema`. Primero aprobar el diagrama conceptual, el modelo lógico y las restricciones.


## Propuesta actualizada

La propuesta de rediseño del E-R está en [diagrama-er-propuesto.md](./diagrama-er-propuesto.md). Incluye tres diagramas Mermaid: estructura académica/usuarios; periodos, grupos, inscripciones/lista de espera; y evaluación, reconocimientos, notificaciones/auditoría.

El documento mantiene el estado de propuesta, no de modelo final aprobado. Debe revisarse con el equipo antes de generar SQL.

## Documentos de diseño


- [Modelo relacional propuesto](./modelo-relacional-propuesto.md): tablas candidatas, PK, FK, restricciones y vistas recomendadas.
