# Modelo relacional propuesto — Sistema de gestión escolar

**Versión:** 0.1 — borrador para revisión.  
**Estado:** propuesta derivada del E-R propuesto; todavía no es el DDL definitivo.  
**Motor objetivo:** MariaDB con Galera Cluster.  
**Regla:** no crear tablas físicas hasta revisar y aprobar este modelo.

## Convenciones

- Nombres de tablas y columnas en minúsculas y `snake_case`.
- Cada tabla tendrá una PK estable.
- Las relaciones se implementan con FK.
- Los identificadores institucionales y claves de catálogo tendrán restricciones UNIQUE.
- Los registros académicos se conservarán mediante estados y movimientos; no se borrarán físicamente como operación normal.
- Se utilizarán tipos decimales para créditos, ponderaciones y calificaciones; no `FLOAT` para cálculos académicos.
- Las fechas y horas se almacenarán con tipos adecuados (`DATE`, `TIME`, `DATETIME`).

## 1. Usuarios, roles y seguridad

| Tabla | Columnas principales | PK | FK | Restricciones / notas |
|---|---|---|---|---|
| `rol` | `id_rol`, `nombre`, `descripcion` | `id_rol` | — | `nombre` UNIQUE. Roles iniciales: administrador, Servicios Escolares, coordinador académico, profesor y alumno. |
| `permiso` | `id_permiso`, `codigo`, `descripcion` | `id_permiso` | — | `codigo` UNIQUE. El catálogo de acciones específicas se terminará de definir en el diseño de la aplicación. |
| `rol_permiso` | `id_rol`, `id_permiso` | compuesta | `id_rol → rol`, `id_permiso → permiso` | UNIQUE/PK (`id_rol`, `id_permiso`). |
| `cuenta_usuario` | `id_usuario`, `id_rol`, `id_alumno` nullable, `id_profesor` nullable, `identificador_acceso`, `password_hash`, `estado`, `intentos_fallidos`, `bloqueado_hasta`, `debe_cambiar_password` | `id_usuario` | `id_rol → rol`, `id_alumno → alumno`, `id_profesor → profesor` | `identificador_acceso` UNIQUE; `id_alumno` UNIQUE nullable; `id_profesor` UNIQUE nullable. Una cuenta no se vincula simultáneamente a alumno y profesor. Personal institucional y administrador podrán tener ambos vínculos nulos. La contraseña se almacena únicamente como hash. `debe_cambiar_password` se usa para restablecimientos temporales, no para forzar cambio al crear cuentas nuevas. |

## 2. Expedientes y estructura académica

| Tabla | Columnas principales | PK | FK | Restricciones / notas |
|---|---|---|---|---|
| `alumno` | `id_alumno`, `numero_control`, `nombres`, `apellidos`, `correo_institucional`, `telefono` nullable, `fecha_ingreso`, `estado_academico` | `id_alumno` | — | `numero_control` UNIQUE y NOT NULL. `correo_institucional` UNIQUE y NOT NULL. |
| `profesor` | `id_profesor`, `numero_empleado`, `nombres`, `apellidos`, `grado_academico`, `correo_institucional`, `telefono` nullable, `fecha_ingreso`, `estado` | `id_profesor` | — | `numero_empleado` UNIQUE y NOT NULL. `correo_institucional` UNIQUE y NOT NULL. |
| `departamento` | `id_departamento`, `clave`, `nombre`, `descripcion` | `id_departamento` | — | `clave` y `nombre` UNIQUE; nombre y clave obligatorios. |
| `adscripcion_profesor` | `id_adscripcion`, `id_profesor`, `id_departamento`, `fecha_inicio`, `fecha_fin` nullable, `vigente` | `id_adscripcion` | `id_profesor → profesor`, `id_departamento → departamento` | Historial de adscripciones; una sola vigente por profesor. |
| `carrera` | `id_carrera`, `clave`, `nombre`, `estado` | `id_carrera` | — | `clave` UNIQUE. El catálogo oficial y sus claves se verificarán con el ITP. |
| `plan_estudio` | `id_plan`, `id_carrera`, `clave_version`, `vigencia_inicio`, `vigencia_fin` nullable, `estado` | `id_plan` | `id_carrera → carrera` | UNIQUE (`id_carrera`, `clave_version`). Versiones anteriores se conservan. |
| `alumno_plan` | `id_alumno_plan`, `id_alumno`, `id_plan`, `fecha_inicio`, `fecha_fin` nullable, `estado` | `id_alumno_plan` | `id_alumno → alumno`, `id_plan → plan_estudio` | Historial de carrera/plan del alumno; una sola asignación vigente. |
| `materia` | `id_materia`, `clave`, `nombre`, `descripcion`, `estado` | `id_materia` | — | `clave` UNIQUE. Catálogo reutilizable de materias. |
| `plan_materia` | `id_plan_materia`, `id_plan`, `id_materia`, `creditos`, `semestre_sugerido`, `horas_teoria`, `horas_practica` | `id_plan_materia` | `id_plan → plan_estudio`, `id_materia → materia` | UNIQUE (`id_plan`, `id_materia`). Los créditos y semestre se definen para cada plan. |
| `prerrequisito` | `id_plan_materia`, `id_plan_materia_requisito`, `tipo` | compuesta | ambas columnas → `plan_materia` | PK compuesta; impedir que una materia sea prerrequisito de sí misma. Validar seriación institucional. |
| `equivalencia_materia` | `id_equivalencia`, `id_plan_materia_a`, `id_plan_materia_b`, `estado_autorizacion`, `id_usuario_autoriza` nullable, `fecha_autorizacion` nullable | `id_equivalencia` | ambas materias → `plan_materia`; autorizador → `cuenta_usuario` | Guardar la autorización y evitar equivalencias duplicadas o reflexivas. Criterios sujetos al ITP/TecNM. |

## 3. Periodos, grupos, aulas e inscripciones

| Tabla | Columnas principales | PK | FK | Restricciones / notas |
|---|---|---|---|---|
| `periodo_escolar` | `id_periodo`, `nombre`, `tipo`, `fecha_inicio`, `fecha_fin`, `estado` | `id_periodo` | — | Tipos ordinario y extraordinario/verano. Fechas y calendario configurados por periodo. |
| `ventana_movimiento` | `id_ventana`, `id_periodo`, `fecha_apertura`, `fecha_cierre` | `id_ventana` | `id_periodo → periodo_escolar` | UNIQUE (`id_periodo`); apertura anterior al cierre. |
| `aula` | `id_aula`, `edificio`, `clave_aula`, `capacidad_alumnos`, `tipo`, `estado` | `id_aula` | — | `clave_aula` UNIQUE. La capacidad cuenta únicamente alumnos. |
| `grupo` | `id_grupo`, `id_periodo`, `id_profesor`, `clave_grupo`, `cupo`, `fecha_limite_inscripcion`, `estado` | `id_grupo` | `id_periodo → periodo_escolar`, `id_profesor → profesor` | Un solo profesor responsable en la primera versión. Cupo positivo y no superior a capacidad del aula asignada. |
| `grupo_plan_materia` | `id_grupo`, `id_plan_materia` | compuesta (`id_grupo`, `id_plan_materia`) | `id_grupo → grupo`, `id_plan_materia → plan_materia` | PK compuesta. Permite asociar un grupo a materias de varios planes cuando exista equivalencia autorizada. |
| `horario_grupo` | `id_horario`, `id_grupo`, `id_aula`, `dia_semana`, `hora_inicio`, `hora_fin` | `id_horario` | `id_grupo → grupo`, `id_aula → aula` | `hora_inicio < hora_fin`. Evitar traslapes por aula y por alumno mediante validación transaccional. Primera versión presencial. |
| `inscripcion` | `id_inscripcion`, `id_alumno`, `id_grupo`, `id_plan_materia`, `fecha_inscripcion`, `fecha_baja` nullable, `numero_intento`, `calificacion_final` nullable, `estado` | `id_inscripcion` | `id_alumno → alumno`, (`id_grupo`, `id_plan_materia`) → `grupo_plan_materia` | UNIQUE (`id_alumno`, `id_plan_materia`, `numero_intento`). La FK compuesta garantiza que el grupo ofrezca esa materia del plan. Los intentos anteriores no se sobrescriben. La unicidad de inscripción activa por materia/equivalente y periodo se valida en la lógica transaccional. |
| `movimiento_inscripcion` | `id_movimiento`, `id_inscripcion`, `id_grupo_origen`, `id_grupo_destino` nullable, `tipo`, `solicitado_por`, `autorizado_por` nullable, `motivo`, `estado`, `fecha_solicitud`, `fecha_resolucion` nullable | `id_movimiento` | `id_inscripcion → inscripcion`; grupos origen/destino → `grupo`; solicitante/autorizador → `cuenta_usuario` | Registrar cambios de grupo y bajas sobre una inscripción. La inscripción mantiene su identidad/intento; en un cambio se actualiza el grupo vigente y el movimiento conserva el grupo anterior y el nuevo. En una baja, el destino queda NULL y se conserva la inscripción con su estado de baja. |

## 4. Listas de espera y ofertas

| Tabla | Columnas principales | PK | FK | Restricciones / notas |
|---|---|---|---|---|
| `lista_espera` | `id_lista`, `id_grupo`, `fecha_apertura`, `fecha_cierre` nullable, `estado` | `id_lista` | `id_grupo → grupo` | UNIQUE (`id_grupo`); se cierra al inicio de clases o al finalizar el periodo, lo que ocurra primero. |
| `entrada_lista_espera` | `id_entrada`, `id_lista`, `id_grupo`, `id_plan_materia`, `id_alumno`, `fecha_hora_registro`, `estado`, `motivo_omision` nullable | `id_entrada` | `id_lista → lista_espera`, `id_alumno → alumno`, (`id_grupo`, `id_plan_materia`) → `grupo_plan_materia` | UNIQUE (`id_lista`, `id_alumno`). La combinación lista/grupo debe corresponder al grupo de la materia en espera. Orden FIFO por fecha/hora e identificador; conservar posición e historial. |
| `oferta_vacante` | `id_oferta`, `id_entrada`, `fecha_hora_oferta`, `fecha_limite`, `fecha_respuesta` nullable, `respuesta_estado` | `id_oferta` | `id_entrada → entrada_lista_espera` | Una oferta por turno; permitir historial de varias ofertas sobre la misma entrada cuando proceda. Plazo inicial: 24 horas desde generar la notificación. |

## 5. Evaluación, reconocimiento y auditoría

| Tabla | Columnas principales | PK | FK | Restricciones / notas |
|---|---|---|---|---|
| `unidad_evaluacion` | `id_unidad`, `id_materia`, `nombre`, `ponderacion`, `orden`, `estado` | `id_unidad` | `id_materia → materia` | Número de unidades y ponderaciones configurables por materia. Fórmula y escala sujetas a verificación oficial. |
| `calificacion_unidad` | `id_calificacion`, `id_inscripcion`, `id_unidad`, `calificacion`, `estado_captura`, `fecha_captura`, `fecha_publicacion` nullable | `id_calificacion` | `id_inscripcion → inscripcion`, `id_unidad → unidad_evaluacion` | UNIQUE (`id_inscripcion`, `id_unidad`). Estado de captura: borrador/publicada. No mezclarlo con acreditada/no acreditada. |
| `correccion_calificacion` | `id_correccion`, `id_calificacion`, `id_usuario_solicita`, `id_usuario_autoriza`, `valor_anterior`, `valor_nuevo`, `motivo`, `estado`, `fecha_solicitud`, `fecha_resolucion` nullable | `id_correccion` | calificación → `calificacion_unidad`; usuarios → `cuenta_usuario` | Solicitud del profesor y autorización del coordinador. Mantener valores anterior y nuevo. |
| `reconocimiento_materia` | `id_reconocimiento`, `id_alumno`, `id_plan_materia_destino`, `id_inscripcion_origen` nullable, `id_usuario_autoriza`, `tipo`, `estado`, `fecha_resolucion` nullable | `id_reconocimiento` | alumno → `alumno`; materia destino → `plan_materia`; inscripción origen → `inscripcion`; autorizador → `cuenta_usuario` | Registrar equivalencia/revalidación sin alterar calificaciones ni historia original. |
| `notificacion` | `id_notificacion`, `id_usuario`, `tipo`, `titulo`, `contenido`, `fecha_creacion`, `fecha_lectura` nullable | `id_notificacion` | `id_usuario → cuenta_usuario` | No hay borrado definitivo ni vencimiento en el alcance inicial. `fecha_lectura` NULL = no leída. |
| `auditoria` | `id_auditoria`, `id_usuario` nullable, `accion`, `entidad`, `id_registro`, `fecha_hora`, `motivo`, `valor_anterior`, `valor_nuevo` | `id_auditoria` | `id_usuario → cuenta_usuario` | Registrar altas, cambios, bajas lógicas, autorizaciones y eventos de seguridad. Nunca guardar contraseñas ni secretos. |

## 6. Vistas recomendadas (no tablas duplicadas)

| Vista candidata | Información |
|---|---|
| `vw_kardex_alumno` | Historial derivado de inscripciones, materias, planes, periodos, calificaciones y reconocimientos. |
| `vw_horario_alumno` | Horarios de los grupos con inscripción activa de un alumno. |
| `vw_lista_grupo` | Alumnos con inscripción activa para el grupo, respetando los permisos de cada rol. |
| `vw_grupos_profesor` | Grupos y horarios asignados al profesor autenticado. |

Estas vistas reducen duplicidad: el kárdex no se almacena como una copia adicional de las inscripciones.

## 7. Reglas de integridad y transacciones

1. Verificar que cada cuenta tenga un rol válido y que el vínculo a expediente corresponda con el perfil.
2. Garantizar que cada alumno tenga un solo plan vigente y cada profesor una sola adscripción vigente.
3. Garantizar que cada grupo pertenezca a un periodo, tenga un único profesor y tenga oferta de al menos una materia de plan.
4. Verificar que todas las materias de plan asociadas a un grupo compartido sean equivalentes y estén autorizadas.
5. Verificar que el cupo del grupo no exceda la capacidad del aula asignada.
6. Evitar solapamientos de horarios para la misma aula y para cada alumno.
7. Evitar inscripciones activas duplicadas en una misma materia o en una materia equivalente durante un periodo.
8. Validar prerrequisitos y límites de créditos antes de confirmar inscripción y antes de ofrecer una vacante.
9. Gestionar aceptación de oferta, actualización del cupo e inscripción mediante una transacción para evitar que dos alumnos ocupen la misma vacante.
10. Conservar los intentos anteriores, las inscripciones cambiadas o dadas de baja y la historia de calificaciones.
11. Ejecutar cambios autorizados de calificación en transacción, conservando los valores anterior y nuevo.
12. Registrar la auditoría junto con la operación relevante, evitando inconsistencias entre el cambio y su evidencia.

## 8. Pendientes de validación antes de SQL

- Política ITP/TecNM de créditos, seriación, recursamiento, calificación mínima, ponderaciones, redondeo y promedios.
- Catálogo oficial de carreras, planes, claves y materias.
- Reglas de las materias equivalentes y grupos compartidos.
- Atributos y tipos de datos definitivos, tamaños, nulabilidad e índices.
- Cómo implementar restricciones de una sola asignación vigente y validaciones de traslapes de forma segura en MariaDB/Galera.
- Configuración final de usuarios institucionales y permisos detallados por acción.

**Siguiente entregable:** revisar el modelo con el equipo, ajustar las cardinalidades y restricciones, y preparar el diagrama relacional final. No crear aún DDL ni tablas físicas.
