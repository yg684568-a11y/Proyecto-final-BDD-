# Matriz completa de relaciones y cardinalidades

**Versión:** 1.0 — revisión integral del E-R.  
**Estado:** referencia de diseño para verificar que todas las entidades tengan relaciones explícitas.  
**Motor previsto:** MariaDB con Galera Cluster.

## Convención

- **1:1**: una fila se relaciona con una sola fila del otro lado.
- **1:N**: una fila padre puede relacionarse con cero o muchas filas hijas; cada hija corresponde a una sola fila padre.
- **N:M**: relación muchos a muchos, resuelta mediante una tabla asociativa.
- **Opcional**: se indica cuando una FK puede ser NULL o cuando un padre puede no tener hijos.
- En las relaciones que usan dos roles hacia la misma tabla, se distingue el nombre funcional de cada FK (por ejemplo, solicitante y autorizador).

## A. Seguridad, cuentas y personas

| ID | Relación | Cardinalidad | FK / implementación | Regla |
|---|---|---|---|---|
| R01 | Rol — CuentaUsuario | 1:N | `cuenta_usuario.id_rol → rol.id_rol` | Cada cuenta tiene exactamente un rol; un rol puede no tener cuentas o tener muchas. |
| R02 | Rol — RolPermiso | 1:N | `rol_permiso.id_rol → rol.id_rol` | La tabla asociativa registra permisos concedidos a cada rol. |
| R03 | Permiso — RolPermiso | 1:N | `rol_permiso.id_permiso → permiso.id_permiso` | Rol y permiso forman una relación N:M resuelta por `rol_permiso`; PK compuesta (`id_rol`, `id_permiso`). |
| R04 | Persona — Alumno | 1:0..1 | `alumno.id_persona → persona.id_persona` UNIQUE | Cada alumno tiene exactamente una persona; una persona puede no ser alumno o serlo una sola vez. |
| R05 | Persona — Empleado | 1:0..1 | `empleado.id_persona → persona.id_persona` UNIQUE | Cada empleado tiene exactamente una persona; una persona puede no ser empleado. |
| R06 | Empleado — Profesor | 1:0..1 | `profesor.id_empleado → empleado.id_empleado` UNIQUE | Un profesor es un tipo de empleado; un empleado puede no ser profesor. |
| R07 | Empleado — Personal | 1:0..1 | `personal.id_empleado → empleado.id_empleado` UNIQUE | Personal representa empleados no docentes, como Servicios Escolares y coordinación. Un empleado no debe duplicarse como profesor y personal en la primera versión. |
| R08 | Persona — CuentaUsuario | 1:0..1 | `cuenta_usuario.id_persona → persona.id_persona` UNIQUE, nullable | Cada usuario institucional tiene como máximo una cuenta. La cuenta de administrador puede no tener persona vinculada porque su acceso usa nombre de usuario. |
| R09 | CuentaUsuario — SesionUsuario | 1:N | `sesion_usuario.id_usuario → cuenta_usuario.id_usuario` | Una cuenta puede generar muchas sesiones históricas; cada sesión pertenece a una cuenta. |
| R10 | CuentaUsuario — Notificacion | 1:N | `notificacion.id_usuario → cuenta_usuario.id_usuario` | Cada notificación tiene un destinatario; una cuenta recibe cero o muchas notificaciones. |
| R11 | CuentaUsuario — Auditoria | 1:N opcional | `auditoria.id_usuario → cuenta_usuario.id_usuario` nullable | Una acción puede ser automática y no tener usuario humano; si la ejecuta una persona, se asocia su cuenta. |

## B. Estructura académica, planes y personal

| ID | Relación | Cardinalidad | FK / implementación | Regla |
|---|---|---|---|---|
| R12 | Carrera — PlanEstudio | 1:N | `plan_estudio.id_carrera → carrera.id_carrera` | Cada plan pertenece a una carrera; una carrera conserva varias versiones de plan. |
| R13 | Alumno — AlumnoPlan | 1:N | `alumno_plan.id_alumno → alumno.id_alumno` | Cada fila guarda una asignación histórica del alumno. Solo una puede estar vigente. |
| R14 | PlanEstudio — AlumnoPlan | 1:N | `alumno_plan.id_plan → plan_estudio.id_plan` | Un plan puede estar asignado a muchos alumnos a lo largo del tiempo. |
| R15 | PlanEstudio — PlanMateria | 1:N | `plan_materia.id_plan → plan_estudio.id_plan` | Cada fila indica que una materia forma parte de una versión del plan. |
| R16 | Materia — PlanMateria | 1:N | `plan_materia.id_materia → materia.id_materia` | Una materia puede reutilizarse en varios planes. PlanEstudio y Materia forman N:M a través de PlanMateria. |
| R17 | PlanMateria — Prerrequisito (materia destino) | 1:N | `prerrequisito.id_plan_materia → plan_materia.id_plan_materia` | Una materia de plan puede tener varios prerrequisitos. |
| R18 | PlanMateria — Prerrequisito (materia requisito) | 1:N | `prerrequisito.id_plan_materia_requisito → plan_materia.id_plan_materia` | Una materia puede ser prerrequisito de varias materias. La relación PlanMateria–PlanMateria es N:M recursiva, resuelta por Prerrequisito. |
| R19 | Departamento — AdscripcionProfesor | 1:N | `adscripcion_profesor.id_departamento → departamento.id_departamento` | Un departamento puede tener historial de muchas adscripciones. |
| R20 | Profesor — AdscripcionProfesor | 1:N | `adscripcion_profesor.id_profesor → profesor.id_profesor` | Cada adscripción pertenece a un profesor; un profesor puede tener varias adscripciones históricas y solo una vigente. |
| R21 | PlanMateria — EquivalenciaMateria (lado A) | 1:N | `equivalencia_materia.id_plan_materia_a → plan_materia.id_plan_materia` | Una materia puede participar en varias equivalencias como origen/lado A. |
| R22 | PlanMateria — EquivalenciaMateria (lado B) | 1:N | `equivalencia_materia.id_plan_materia_b → plan_materia.id_plan_materia` | Una materia puede participar en varias equivalencias como destino/lado B. La relación es N:M recursiva y se resuelve en EquivalenciaMateria. |
| R23 | CuentaUsuario — EquivalenciaMateria (autorizador) | 1:N | `equivalencia_materia.id_usuario_autoriza → cuenta_usuario.id_usuario` nullable hasta autorizar | Cada autorización la realiza un usuario; un coordinador puede autorizar varias equivalencias. |

## C. Periodos, oferta académica, horarios e inscripciones

| ID | Relación | Cardinalidad | FK / implementación | Regla |
|---|---|---|---|---|
| R24 | PeriodoEscolar — Grupo | 1:N | `grupo.id_periodo → periodo_escolar.id_periodo` | Cada grupo se ofrece en un periodo; cada periodo puede contener muchos grupos. |
| R25 | CuentaUsuario — PeriodoEscolar (creador) | 1:N | `periodo_escolar.id_usuario_crea → cuenta_usuario.id_usuario` | El usuario coordinador que crea el periodo queda registrado. |
| R26 | PeriodoEscolar — VentanaMovimiento | 1:0..1 | `ventana_movimiento.id_periodo → periodo_escolar.id_periodo` UNIQUE | Un periodo puede tener una ventana configurada o ninguna; cada ventana pertenece a un periodo. |
| R27 | CuentaUsuario — VentanaMovimiento (configurador) | 1:N | `ventana_movimiento.id_usuario_configura → cuenta_usuario.id_usuario` | Se registra quién configuró las fechas. |
| R28 | Profesor — Grupo | 1:N | `grupo.id_profesor → profesor.id_profesor` | Cada grupo tiene exactamente un profesor responsable en la primera versión; un profesor puede atender muchos grupos. |
| R29 | Grupo — GrupoPlanMateria | 1:N | `grupo_plan_materia.id_grupo → grupo.id_grupo` | Cada fila asociativa pertenece a un grupo; un grupo debe ofrecer al menos una materia de plan. |
| R30 | PlanMateria — GrupoPlanMateria | 1:N | `grupo_plan_materia.id_plan_materia → plan_materia.id_plan_materia` | Una materia de plan puede ofertarse en varios grupos. Grupo y PlanMateria forman N:M resuelta por GrupoPlanMateria. |
| R31 | Grupo — HorarioGrupo | 1:N | `horario_grupo.id_grupo → grupo.id_grupo` | Cada horario pertenece a un único grupo; un grupo presencial tendrá uno o varios bloques de horario. |
| R32 | Aula — HorarioGrupo | 1:N | `horario_grupo.id_aula → aula.id_aula` | Cada bloque se realiza en un aula; un aula puede tener muchos bloques en horarios distintos. |
| R33 | Alumno — Inscripcion | 1:N | `inscripcion.id_alumno → alumno.id_alumno` | Cada inscripción corresponde a un alumno; el alumno conserva varias inscripciones históricas. |
| R34 | GrupoPlanMateria — Inscripcion | 1:N | FK compuesta `inscripcion(id_grupo,id_plan_materia) → grupo_plan_materia(id_grupo,id_plan_materia)` | Garantiza que el grupo ofrece la materia del plan que el alumno cursa. |
| R35 | Inscripcion — MovimientoInscripcion | 1:N | `movimiento_inscripcion.id_inscripcion → inscripcion.id_inscripcion` | Una inscripción puede tener cero o muchos movimientos históricos (cambio de grupo o baja). |
| R36 | Grupo — MovimientoInscripcion (origen) | 1:N | `movimiento_inscripcion.id_grupo_origen → grupo.id_grupo` | Cada movimiento registra el grupo del que parte. |
| R37 | Grupo — MovimientoInscripcion (destino) | 1:N opcional | `movimiento_inscripcion.id_grupo_destino → grupo.id_grupo` nullable | Para cambio se indica el nuevo grupo; para una baja queda NULL. |
| R38 | CuentaUsuario — MovimientoInscripcion (solicitante) | 1:N | `movimiento_inscripcion.id_usuario_solicita → cuenta_usuario.id_usuario` | Siempre se registra quién ejecuta/solicita el movimiento, incluido el alumno si lo hace directamente. |
| R39 | CuentaUsuario — MovimientoInscripcion (autorizador) | 1:N opcional | `movimiento_inscripcion.id_usuario_autoriza → cuenta_usuario.id_usuario` nullable | Solo se llena cuando el movimiento necesita autorización fuera de la ventana. |

## D. Lista de espera y ofertas de vacantes

| ID | Relación | Cardinalidad | FK / implementación | Regla |
|---|---|---|---|---|
| R40 | Grupo — ListaEspera | 1:0..1 | `lista_espera.id_grupo → grupo.id_grupo` UNIQUE | Un grupo puede no tener lista de espera o tener una; la lista pertenece a un solo grupo. |
| R41 | ListaEspera — EntradaListaEspera | 1:N | `entrada_lista_espera.id_lista → lista_espera.id_lista` | Cada entrada pertenece a una lista; una lista contiene muchas entradas ordenadas FIFO. |
| R42 | Alumno — EntradaListaEspera | 1:N | `entrada_lista_espera.id_alumno → alumno.id_alumno` | Un alumno puede entrar en varias listas de grupos distintos a lo largo del tiempo. |
| R43 | GrupoPlanMateria — EntradaListaEspera | 1:N | FK compuesta `entrada_lista_espera(id_grupo,id_plan_materia) → grupo_plan_materia(id_grupo,id_plan_materia)` | Garantiza que la lista del grupo ofrece la materia del plan solicitada. Además se verifica que `id_lista` corresponde al mismo `id_grupo`. |
| R44 | EntradaListaEspera — OfertaVacante | 1:0..1 | `oferta_vacante.id_entrada → entrada_lista_espera.id_entrada` UNIQUE | Una entrada recibe como máximo una oferta: si rechaza o vence, sale; si fue omitida por no elegible, conserva su lugar hasta que pueda revalidarse. |
| R45 | OfertaVacante — Notificacion | 1:1 de negocio | `notificacion.id_oferta → oferta_vacante.id_oferta` UNIQUE, nullable | Cada oferta genera una notificación interna. Otras notificaciones académicas tienen `id_oferta` NULL. Se crean en una misma transacción. |

## E. Evaluación, correcciones y reconocimientos

| ID | Relación | Cardinalidad | FK / implementación | Regla |
|---|---|---|---|---|
| R46 | Materia — UnidadEvaluacion | 1:N | `unidad_evaluacion.id_materia → materia.id_materia` | Cada unidad pertenece a una materia; las unidades y ponderaciones son configurables por materia. |
| R47 | Inscripcion — CalificacionUnidad | 1:N | `calificacion_unidad.id_inscripcion → inscripcion.id_inscripcion` | Las calificaciones se registran por intento de inscripción. |
| R48 | UnidadEvaluacion — CalificacionUnidad | 1:N | `calificacion_unidad.id_unidad → unidad_evaluacion.id_unidad` | Una unidad genera calificaciones para muchos alumnos/inscripciones. Se valida que la unidad pertenezca a la materia de la inscripción. |
| R49 | CalificacionUnidad — CorreccionCalificacion | 1:N | `correccion_calificacion.id_calificacion → calificacion_unidad.id_calificacion` | Se conserva el historial de solicitudes de corrección de cada calificación publicada. |
| R50 | CuentaUsuario — CorreccionCalificacion (solicitante) | 1:N | `correccion_calificacion.id_usuario_solicita → cuenta_usuario.id_usuario` | El profesor responsable solicita la corrección. |
| R51 | CuentaUsuario — CorreccionCalificacion (autorizador) | 1:N | `correccion_calificacion.id_usuario_autoriza → cuenta_usuario.id_usuario` nullable mientras pendiente | El coordinador académico resuelve la solicitud; se registra la cuenta que autorizó o rechazó. |
| R52 | Alumno — ReconocimientoMateria | 1:N | `reconocimiento_materia.id_alumno → alumno.id_alumno` | Un alumno puede tener varias materias reconocidas. |
| R53 | PlanMateria — ReconocimientoMateria (destino) | 1:N | `reconocimiento_materia.id_plan_materia_destino → plan_materia.id_plan_materia` | Cada reconocimiento se aplica a una materia de un plan vigente/histórico. |
| R54 | Inscripcion — ReconocimientoMateria (origen) | 1:N opcional | `reconocimiento_materia.id_inscripcion_origen → inscripcion.id_inscripcion` nullable | Se indica la inscripción de origen cuando el reconocimiento proviene de una materia cursada dentro del sistema; puede ser NULL para antecedentes externos. |
| R55 | CuentaUsuario — ReconocimientoMateria (autorizador) | 1:N | `reconocimiento_materia.id_usuario_autoriza → cuenta_usuario.id_usuario` | El coordinador autoriza; Servicios Escolares registra el resultado en el expediente. |

## F. Recuento de entidades y verificación de conexión

Todas las entidades propuestas aparecen en al menos una relación de las secciones A–E:

`rol`, `permiso`, `rol_permiso`, `persona`, `alumno`, `empleado`, `profesor`, `personal`, `cuenta_usuario`, `sesion_usuario`, `departamento`, `adscripcion_profesor`, `carrera`, `plan_estudio`, `alumno_plan`, `materia`, `plan_materia`, `prerrequisito`, `equivalencia_materia`, `periodo_escolar`, `ventana_movimiento`, `aula`, `grupo`, `grupo_plan_materia`, `horario_grupo`, `inscripcion`, `movimiento_inscripcion`, `lista_espera`, `entrada_lista_espera`, `oferta_vacante`, `unidad_evaluacion`, `calificacion_unidad`, `correccion_calificacion`, `reconocimiento_materia`, `notificacion` y `auditoria`.

**No hay entidades sin relación.** Las únicas relaciones con comportamiento opcional son intencionales y están descritas: cuentas administrativas sin persona asociada, alumnos/empleados sin cuenta, personal/empleado no docente, ventanas de movimiento aún no configuradas, destinos de movimientos de baja, reconocimientos sin inscripción de origen, auditorías automáticas sin usuario humano y notificaciones genéricas sin oferta asociada.

## Reglas transversales que no se expresan con cardinalidad solamente

1. Cada persona tendrá como máximo una cuenta. Una cuenta de administrador puede existir sin persona; el resto de perfiles debe vincularse con la persona correspondiente y el subtipo correcto (alumno, profesor o personal).
2. Un empleado se clasificará como profesor o personal no docente en la primera versión, no en ambos subtipos.
3. Solo puede existir un plan vigente por alumno y una adscripción vigente por profesor.
4. Un grupo tendrá un único profesor y al menos una materia de plan asociada.
5. Una inscripción debe apuntar a la pareja de grupo y materia autorizada en `grupo_plan_materia`.
6. La materia de una unidad de evaluación debe coincidir con la materia de la inscripción; se valida al guardar la calificación.
7. La lista y la entrada de espera deben corresponder al mismo grupo; se valida con FK compuesta.
8. Antes de insertar una inscripción o aceptar una oferta se bloquea/valida el cupo de forma transaccional para evitar sobrecupo.
9. Las restricciones académicas de inscripción única, créditos, prerrequisitos y equivalencias se validan dentro de la transacción de inscripción.
10. Los cruces de rol (profesor solo opera sus grupos, alumno solo sus datos) se aplican en autorización de aplicación y vistas, no solamente con FK.
11. La auditoría usa `entidad` + `id_registro` como referencia polimórfica al objeto auditado; ese par no es FK a una tabla de negocio porque puede apuntar a distintas entidades. La FK normal de auditoría es el usuario actor.
12. El sistema conserva las notificaciones y los registros académicos; no se usa borrado físico para los movimientos académicos normales.
