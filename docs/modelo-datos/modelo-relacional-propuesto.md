# Modelo relacional propuesto — Sistema de gestión escolar

**Versión:** 1.0 — revisión integral de PK, FK y cardinalidades.  
**Estado:** propuesta para validar con el equipo; no es aún el DDL ejecutable.  
**Motor objetivo:** MariaDB con Galera Cluster.  
**Referencia de relaciones:** [Matriz completa de relaciones y cardinalidades](./relaciones-y-cardinalidades.md).

## Convenciones

- Tablas y columnas en minúsculas y `snake_case`.
- PK en todas las tablas; FK explícitas para relaciones.
- Tipos numéricos decimales para créditos, calificaciones y ponderaciones.
- Correo institucional común en `persona`, para garantizar unicidad entre alumnos y empleados.
- Los registros académicos se conservan con estados/movimientos; no se borran físicamente como operación normal.
- `NULL` se utiliza solo donde se explica como opcional.
- Las tablas asociativas resuelven las relaciones N:M; sus PK/FK compuestas se indican expresamente.

## 1. Roles, permisos, personas y cuentas

| Tabla | Columnas (PK marcada con **PK**) | FK | Restricciones principales |
|---|---|---|---|
| `rol` | **PK** `id_rol`, `nombre`, `descripcion` | — | `nombre` UNIQUE. Registros iniciales: administrador, Servicios Escolares, coordinador académico, profesor y alumno. |
| `permiso` | **PK** `id_permiso`, `codigo`, `descripcion` | — | `codigo` UNIQUE. |
| `rol_permiso` | **PK/FK** `id_rol`, **PK/FK** `id_permiso` | `id_rol → rol`; `id_permiso → permiso` | PK compuesta (`id_rol`, `id_permiso`). Resuelve la relación N:M entre roles y permisos. |
| `persona` | **PK** `id_persona`, `nombres`, `apellidos`, `correo_institucional`, `telefono` nullable | — | `correo_institucional` UNIQUE y NOT NULL. Centraliza los datos comunes y garantiza que el correo no se repita entre alumnos y empleados. |
| `alumno` | **PK** `id_alumno`, **FK/UNIQUE** `id_persona`, `numero_control`, `fecha_ingreso`, `estado_academico` | `id_persona → persona` | `id_persona` UNIQUE; `numero_control` UNIQUE y NOT NULL. Cada alumno tiene exactamente una persona asociada. |
| `empleado` | **PK** `id_empleado`, **FK/UNIQUE** `id_persona`, `numero_empleado`, `fecha_ingreso`, `estado` | `id_persona → persona` | `id_persona` UNIQUE; `numero_empleado` UNIQUE y NOT NULL. Unifica el número de empleado de profesores y personal administrativo. |
| `profesor` | **PK** `id_profesor`, **FK/UNIQUE** `id_empleado`, `grado_academico` | `id_empleado → empleado` | Cada profesor es un empleado; un empleado puede no ser profesor. |
| `personal` | **PK** `id_personal`, **FK/UNIQUE** `id_empleado`, `tipo_personal` | `id_empleado → empleado` | Para personal no docente, como Servicios Escolares y coordinación académica. En la primera versión un empleado no se registra simultáneamente como profesor y personal. |
| `cuenta_usuario` | **PK** `id_usuario`, **FK** `id_rol`, **FK/UNIQUE** `id_persona` nullable, `identificador_acceso`, `password_hash`, `estado`, `intentos_fallidos`, `bloqueado_hasta` nullable | `id_rol → rol`; `id_persona → persona` | `identificador_acceso` UNIQUE; `id_persona` UNIQUE nullable. Cada persona tiene como máximo una cuenta. La cuenta administrativa puede tener `id_persona` NULL y usar nombre de usuario; alumno usa número de control; empleado usa número de empleado. El rol debe corresponder al subtipo de persona. Contraseñas solo como hash. |
| `sesion_usuario` | **PK** `id_sesion`, **FK** `id_usuario`, `fecha_inicio`, `ultima_actividad`, `fecha_expiracion`, `fecha_cierre` nullable, `estado` | `id_usuario → cuenta_usuario` | Una cuenta puede tener varias sesiones históricas. `fecha_expiracion` aplica el cierre automático por inactividad. |

## 2. Departamentos y estructura académica

| Tabla | Columnas (PK marcada con **PK**) | FK | Restricciones principales |
|---|---|---|---|
| `departamento` | **PK** `id_departamento`, `clave`, `nombre`, `descripcion` | — | `clave` y `nombre` UNIQUE; ambos obligatorios. |
| `adscripcion_profesor` | **PK** `id_adscripcion`, **FK** `id_profesor`, **FK** `id_departamento`, `fecha_inicio`, `fecha_fin` nullable, `vigente` | `id_profesor → profesor`; `id_departamento → departamento` | Historial. Solo una adscripción vigente por profesor; imponer con índice/columna generada o validación transaccional. |
| `carrera` | **PK** `id_carrera`, `clave`, `nombre`, `estado` | — | `clave` UNIQUE. Catálogo oficial y claves pendientes de verificar. |
| `plan_estudio` | **PK** `id_plan`, **FK** `id_carrera`, `clave_version`, `vigencia_inicio`, `vigencia_fin` nullable, `estado` | `id_carrera → carrera` | UNIQUE (`id_carrera`, `clave_version`). Versiones anteriores se conservan. |
| `alumno_plan` | **PK** `id_alumno_plan`, **FK** `id_alumno`, **FK** `id_plan`, `fecha_inicio`, `fecha_fin` nullable, `estado`, `vigente` | `id_alumno → alumno`; `id_plan → plan_estudio` | Historial de carrera/plan. Solo una asignación vigente por alumno; imponer con índice/columna generada o validación transaccional. |
| `materia` | **PK** `id_materia`, `clave`, `nombre`, `descripcion`, `estado` | — | `clave` UNIQUE. Catálogo reutilizable. |
| `plan_materia` | **PK** `id_plan_materia`, **FK** `id_plan`, **FK** `id_materia`, `creditos`, `semestre_sugerido`, `horas_teoria`, `horas_practica` | `id_plan → plan_estudio`; `id_materia → materia` | UNIQUE (`id_plan`, `id_materia`). Añadir UNIQUE (`id_plan_materia`, `id_materia`) si se requiere FK compuesta en validaciones de evaluación. |
| `prerrequisito` | **PK/FK** `id_plan_materia`, **PK/FK** `id_plan_materia_requisito`, `tipo` | ambas FK → `plan_materia` | PK compuesta. No permitir que una materia sea prerrequisito de sí misma. Resuelve una relación N:M recursiva. |
| `equivalencia_materia` | **PK** `id_equivalencia`, **FK** `id_plan_materia_a`, **FK** `id_plan_materia_b`, `estado_autorizacion`, **FK** `id_usuario_autoriza` nullable, `fecha_autorizacion` nullable | ambas materias → `plan_materia`; autorizador → `cuenta_usuario` | UNIQUE (`id_plan_materia_a`, `id_plan_materia_b`); A y B deben ser distintas. El autorizador puede ser NULL mientras esté pendiente. La orientación se normaliza para evitar pares duplicados invertidos. |

## 3. Periodos, grupos, espacios, inscripciones y movimientos

| Tabla | Columnas (PK marcada con **PK**) | FK | Restricciones principales |
|---|---|---|---|
| `periodo_escolar` | **PK** `id_periodo`, `nombre`, `tipo`, `fecha_inicio`, `fecha_fin`, `estado`, **FK** `id_usuario_crea` | `id_usuario_crea → cuenta_usuario` | Cada periodo registra al usuario coordinador que lo crea. `fecha_inicio < fecha_fin`. |
| `ventana_movimiento` | **PK** `id_ventana`, **FK/UNIQUE** `id_periodo`, `fecha_apertura`, `fecha_cierre`, **FK** `id_usuario_configura` | `id_periodo → periodo_escolar`; `id_usuario_configura → cuenta_usuario` | Un periodo puede no tener ventana aún o tener una; apertura anterior al cierre. |
| `aula` | **PK** `id_aula`, `edificio`, `clave_aula`, `capacidad_alumnos`, `tipo`, `estado` | — | `clave_aula` UNIQUE; capacidad positiva. Solo clases presenciales en primera versión. |
| `grupo` | **PK** `id_grupo`, **FK** `id_periodo`, **FK** `id_profesor`, `clave_grupo`, `cupo`, `fecha_limite_inscripcion`, `estado` | `id_periodo → periodo_escolar`; `id_profesor → profesor` | UNIQUE (`id_periodo`, `clave_grupo`). Un profesor responsable. Cupo positivo y no superior a la capacidad del aula; validar al programar/abrir grupo. |
| `grupo_plan_materia` | **PK/FK** `id_grupo`, **PK/FK** `id_plan_materia` | `id_grupo → grupo`; `id_plan_materia → plan_materia` | PK compuesta (`id_grupo`, `id_plan_materia`). Resuelve la N:M entre grupo y materia de plan; permite grupos compartidos autorizados. |
| `horario_grupo` | **PK** `id_horario`, **FK** `id_grupo`, **FK** `id_aula`, `dia_semana`, `hora_inicio`, `hora_fin` | `id_grupo → grupo`; `id_aula → aula` | `hora_inicio < hora_fin`. Evitar traslapes por aula y por alumno en validación transaccional. Cada grupo debe tener horario antes de abrirse a inscripciones. |
| `inscripcion` | **PK** `id_inscripcion`, **FK** `id_alumno`, `id_grupo`, `id_plan_materia`, `fecha_inscripcion`, `fecha_baja` nullable, `numero_intento`, `calificacion_final` nullable, `estado` | `id_alumno → alumno`; FK compuesta (`id_grupo`, `id_plan_materia`) → `grupo_plan_materia` | UNIQUE (`id_alumno`, `id_plan_materia`, `numero_intento`). Se conserva cada intento. La materia/equivalencia activa por periodo se valida transaccionalmente. |
| `movimiento_inscripcion` | **PK** `id_movimiento`, **FK** `id_inscripcion`, **FK** `id_grupo_origen`, **FK** `id_grupo_destino` nullable, **FK** `id_usuario_solicita`, **FK** `id_usuario_autoriza` nullable, `tipo`, `motivo`, `estado`, `fecha_solicitud`, `fecha_resolucion` nullable | inscripción → `inscripcion`; grupos → `grupo`; usuarios → `cuenta_usuario` | Cambio: destino obligatorio y compatible con la materia del plan; baja: destino NULL. Solicitante obligatorio; autorizador NULL cuando el alumno puede hacerlo directamente en la ventana. Conservar siempre grupo origen/destino. |

## 4. Lista de espera, ofertas y notificaciones

| Tabla | Columnas (PK marcada con **PK**) | FK | Restricciones principales |
|---|---|---|---|
| `lista_espera` | **PK** `id_lista`, **FK/UNIQUE** `id_grupo`, `fecha_apertura`, `fecha_cierre` nullable, `estado` | `id_grupo → grupo` | Un grupo puede tener cero o una lista. Añadir UNIQUE (`id_lista`, `id_grupo`) para FK compuesta desde entradas. |
| `entrada_lista_espera` | **PK** `id_entrada`, `id_lista`, `id_grupo`, `id_plan_materia`, **FK** `id_alumno`, `fecha_hora_registro`, `estado`, `motivo_omision` nullable | `id_alumno → alumno`; (`id_lista`, `id_grupo`) → `lista_espera`; (`id_grupo`, `id_plan_materia`) → `grupo_plan_materia` | UNIQUE (`id_lista`, `id_alumno`). Las FK compuestas garantizan que la entrada corresponda al mismo grupo que la lista y a una materia que ese grupo ofrece. FIFO por fecha/hora e id de entrada. |
| `oferta_vacante` | **PK** `id_oferta`, **FK/UNIQUE** `id_entrada`, `fecha_hora_oferta`, `fecha_limite`, `fecha_respuesta` nullable, `respuesta_estado` | `id_entrada → entrada_lista_espera` | Una entrada recibe como máximo una oferta. Plazo inicial: 24 horas desde generar la notificación. |
| `notificacion` | **PK** `id_notificacion`, **FK** `id_usuario`, **FK/UNIQUE** `id_oferta` nullable, `tipo`, `titulo`, `contenido`, `fecha_creacion`, `fecha_lectura` nullable | `id_usuario → cuenta_usuario`; `id_oferta → oferta_vacante` | Cada aviso tiene destinatario. `id_oferta` solo se llena en avisos de vacante; una oferta genera su notificación durante la misma transacción. Notificaciones generales tienen NULL. No se eliminan ni vencen. |

## 5. Evaluación, correcciones, reconocimientos y auditoría

| Tabla | Columnas (PK marcada con **PK**) | FK | Restricciones principales |
|---|---|---|---|
| `unidad_evaluacion` | **PK** `id_unidad`, **FK** `id_materia`, `nombre`, `ponderacion`, `orden`, `estado` | `id_materia → materia` | Las unidades y ponderaciones son configurables por materia. Fórmula y escala quedan sujetas a verificación institucional. |
| `calificacion_unidad` | **PK** `id_calificacion`, **FK** `id_inscripcion`, **FK** `id_unidad`, `calificacion`, `estado_captura`, `fecha_captura`, `fecha_publicacion` nullable | `id_inscripcion → inscripcion`; `id_unidad → unidad_evaluacion` | UNIQUE (`id_inscripcion`, `id_unidad`). Estado de captura: borrador/publicada. Validar que la unidad pertenezca a la materia del intento de inscripción. |
| `correccion_calificacion` | **PK** `id_correccion`, **FK** `id_calificacion`, **FK** `id_usuario_solicita`, **FK** `id_usuario_autoriza` nullable, `valor_anterior`, `valor_nuevo`, `motivo`, `estado`, `fecha_solicitud`, `fecha_resolucion` nullable | `id_calificacion → calificacion_unidad`; usuarios → `cuenta_usuario` | Una solicitud identifica al profesor; autorizador se llena al resolver. Conservar valores anterior y nuevo. |
| `reconocimiento_materia` | **PK** `id_reconocimiento`, **FK** `id_alumno`, **FK** `id_plan_materia_destino`, **FK** `id_inscripcion_origen` nullable, **FK** `id_usuario_autoriza`, `tipo`, `estado`, `fecha_resolucion` nullable | alumno → `alumno`; destino → `plan_materia`; origen → `inscripcion`; autorizador → `cuenta_usuario` | La inscripción de origen es NULL para antecedentes externos. Cuando se informa, debe corresponder al mismo alumno. |
| `auditoria` | **PK** `id_auditoria`, **FK** `id_usuario` nullable, `accion`, `entidad`, `id_registro`, `fecha_hora`, `motivo`, `valor_anterior`, `valor_nuevo` | `id_usuario → cuenta_usuario` | `id_usuario` NULL para procesos automáticos. `entidad` + `id_registro` es referencia polimórfica al registro auditado, no FK a una tabla única. No guardar contraseñas ni secretos. |

## 6. Relaciones completas

La matriz [relaciones-y-cardinalidades.md](./relaciones-y-cardinalidades.md) contiene las relaciones R01–R55, las cardinalidades, opcionalidades y las columnas FK. Todas las tablas de este documento participan en al menos una FK entrante o saliente.

## 7. Vistas recomendadas (no duplicar datos)

| Vista | Consulta que resuelve |
|---|---|
| `vw_kardex_alumno` | Inscripciones, materias, planes, periodos, calificaciones y reconocimientos históricos. |
| `vw_horario_alumno` | Horarios de grupos con inscripción activa de un alumno. |
| `vw_lista_grupo` | Alumnos con inscripción activa, aplicando permisos de consulta/descarga. |
| `vw_grupos_profesor` | Grupos, horarios y listas asociados al profesor autenticado. |

El kárdex se deriva de las inscripciones; no se almacena una copia redundante.

## 8. Reglas y validaciones transaccionales

1. Cada cuenta tiene un rol; el rol debe coincidir con el tipo de perfil asociado a `persona`.
2. Las cuentas de alumno, profesor y personal deben estar vinculadas a Persona; la cuenta de administrador puede ser independiente.
3. Un empleado se registra como profesor o como personal no docente, nunca en ambos subtipos en la primera versión.
4. Un alumno tiene solo un plan vigente; un profesor solo una adscripción vigente.
5. Un grupo publicado tiene un profesor, al menos una materia de plan asociada y por lo menos un bloque de horario presencial.
6. La materia asociada a un grupo compartido debe tener equivalencia autorizada para cada plan participante.
7. Una inscripción debe apuntar a la combinación real de grupo y materia existente en `grupo_plan_materia`.
8. El grupo no supera capacidad del aula; para esa capacidad se cuentan solo alumnos.
9. Evitar traslapes de aula y horarios del alumno.
10. Evitar inscripciones activas simultáneas a la misma materia o una equivalente durante el mismo periodo.
11. Validar prerrequisitos y carga de créditos antes de inscribir y antes de enviar una oferta de vacante.
12. La aceptación de vacante, verificación del cupo, alta de inscripción y cambio de estado de la oferta se realizan en una transacción para evitar sobrecupo.
13. Un cambio de grupo valida que el grupo destino ofrezca la materia del plan del alumno; se guarda origen y destino en `movimiento_inscripcion`.
14. Una corrección de calificación se procesa en transacción, guarda la versión anterior y la nueva y registra la auditoría.
15. Una notificación de vacante y su oferta se crean en la misma transacción.
16. Los cambios de información y la fila de auditoría correspondiente se confirman o revierten juntos.
17. Los permisos efectivos se aplican por rol y por ámbito de datos; las FK no sustituyen la autorización de la aplicación.
18. Los registros académicos, ofertas, notificaciones e historial se conservan; no ejecutar borrado físico como operación normal.

## 9. Pendientes que no bloquean la conectividad del modelo

- Verificar con ITP/TecNM créditos, seriación, recursamiento, escala, ponderaciones, redondeo y calificación mínima.
- Confirmar catálogo oficial de carreras, planes, claves y materias.
- Definir tipos de datos/tamaños finales, índices y restricciones según versión concreta de MariaDB.
- Decidir cómo se implementarán con seguridad las reglas de "solo una vigente" y las validaciones de traslapes (índices, columnas generadas, transacciones o procedimientos).
- Confirmar la interpretación de `Modulos` como unidades de evaluación.
- Validar con el equipo el modelo lógico antes de escribir el DDL.

**Criterio de salida de esta etapa:** E-R, matriz R01–R55 y modelo relacional deben coincidir en nombres, PK, FK, cardinalidades y opcionalidad. Después se podrá generar el esquema SQL.
