# Diagrama E-R propuesto — Sistema de gestión escolar

**Versión:** 1.2 — corrección de inconsistencias entre E-R, modelo relacional y matriz.  
**Estado:** propuesta conceptual/lógica para validación del equipo; todavía no es DDL definitivo.  
**Motor objetivo:** MariaDB con Galera Cluster.

## Criterios del rediseño

- `Cursos` se separa en `Materia` (catálogo) y `Grupo` (oferta concreta de la materia en un periodo).
- `Historiales` se convierte en `Inscripcion`, conservando cada intento de cursado.
- `Horarios` depende de `Grupo` y de `Aula`.
- `Modulos` se interpreta provisionalmente como `UnidadEvaluacion`; `Calf_modulos` como `CalificacionUnidad`.
- Se incorporan carrera, planes, periodos, roles, permisos, cuentas, personas, empleados, personal no docente, lista de espera, ofertas, notificaciones, sesiones y auditoría.
- Los datos de contacto se normalizan en `Persona`, lo que permite que el correo sea único entre alumnos y empleados.
- `Categorias` queda fuera hasta confirmar su significado y utilidad.

## Diagrama 1. Identidad, seguridad y estructura curricular

```mermaid
erDiagram
    ROL ||--o{ CUENTA_USUARIO : asigna
    ROL ||--o{ ROL_PERMISO : concede
    PERMISO ||--o{ ROL_PERMISO : incluye

    PERSONA o|--o| CUENTA_USUARIO : tiene_acceso
    PERSONA ||--o| ALUMNO : puede_ser
    PERSONA ||--o| EMPLEADO : puede_ser
    EMPLEADO ||--o| PROFESOR : especializa
    EMPLEADO ||--o| PERSONAL : especializa

    CUENTA_USUARIO ||--o{ SESION_USUARIO : inicia
    CUENTA_USUARIO ||--o{ NOTIFICACION : recibe
    CUENTA_USUARIO o|--o{ AUDITORIA : ejecuta

    CARRERA ||--o{ PLAN_ESTUDIO : tiene
    ALUMNO ||--o{ ALUMNO_PLAN : historial
    PLAN_ESTUDIO ||--o{ ALUMNO_PLAN : asignado

    PLAN_ESTUDIO ||--o{ PLAN_MATERIA : incluye
    MATERIA ||--o{ PLAN_MATERIA : aparece_en
    PLAN_MATERIA ||--o{ PRERREQUISITO : materia_objetivo
    PLAN_MATERIA ||--o{ PRERREQUISITO : materia_requerida

    DEPARTAMENTO ||--o{ ADSCRIPCION_PROFESOR : agrupa
    PROFESOR ||--o{ ADSCRIPCION_PROFESOR : historial

    PLAN_MATERIA ||--o{ EQUIVALENCIA_MATERIA : lado_a
    PLAN_MATERIA ||--o{ EQUIVALENCIA_MATERIA : lado_b
    CUENTA_USUARIO o|--o{ EQUIVALENCIA_MATERIA : autoriza
```

**Reglas del diagrama 1**

- Cada cuenta tiene un rol. Una persona puede tener como máximo una cuenta; una cuenta administrativa puede existir sin persona asociada, porque su identificador es un nombre de usuario. El esquema actual asigna un solo rol por cuenta; la posibilidad de combinar perfiles en una misma cuenta queda pendiente de confirmación.
- `Persona` concentra nombres, apellidos, correo institucional y teléfono. El correo se conserva en un único campo para evitar duplicados entre alumnos y profesores; su obligatoriedad se valida según el tipo de perfil.
- `Alumno` y `Empleado` son perfiles que extienden a Persona. `Empleado` se especializa como `Profesor` o `Personal` no docente (por ejemplo, Servicios Escolares o coordinación académica). En la primera versión, un empleado no se registra en ambos subtipos; la exclusión entre subtipos requiere validación de aplicación/transacción.
- Una carrera tiene varios planes; un alumno conserva varias asignaciones históricas de plan, pero solo una vigente.
- Un plan tiene materias a través de `PlanMateria`; una materia puede reutilizarse en diferentes planes.
- `Prerrequisito` y `EquivalenciaMateria` son relaciones recursivas N:M entre filas de `PlanMateria`, resueltas por tablas asociativas.
- Las cuentas de los profesores y del personal se vinculan con la misma Persona/Empleado; el rol define si la persona actúa como profesor, Servicios Escolares o coordinación.
- Las relaciones de sesión, notificación, auditoría y autorización se detallan en la matriz completa de relaciones.

## Diagrama 2. Periodos, oferta, grupos, horarios e inscripciones

```mermaid
erDiagram
    CUENTA_USUARIO ||--o{ PERIODO_ESCOLAR : crea
    PERIODO_ESCOLAR ||--o{ GRUPO : ofrece
    PERIODO_ESCOLAR ||--o| VENTANA_MOVIMIENTO : configura
    CUENTA_USUARIO ||--o{ VENTANA_MOVIMIENTO : configura

    PROFESOR ||--o{ GRUPO : responsable
    GRUPO ||--|{ GRUPO_PLAN_MATERIA : ofrece
    PLAN_MATERIA ||--o{ GRUPO_PLAN_MATERIA : pertenece

    GRUPO ||--|{ HORARIO_GRUPO : programa
    AULA ||--o{ HORARIO_GRUPO : ocupa

    ALUMNO ||--o{ INSCRIPCION : realiza
    GRUPO_PLAN_MATERIA ||--o{ INSCRIPCION : recibe

    INSCRIPCION ||--o{ MOVIMIENTO_INSCRIPCION : historial
    GRUPO_PLAN_MATERIA ||--o{ MOVIMIENTO_INSCRIPCION : grupo_materia_origen
    GRUPO_PLAN_MATERIA o|--o{ MOVIMIENTO_INSCRIPCION : grupo_materia_destino
    CUENTA_USUARIO ||--o{ MOVIMIENTO_INSCRIPCION : solicita
    CUENTA_USUARIO o|--o{ MOVIMIENTO_INSCRIPCION : autoriza

    GRUPO ||--o| LISTA_ESPERA : puede_tener
    LISTA_ESPERA ||--o{ ENTRADA_LISTA_ESPERA : contiene
    ALUMNO ||--o{ ENTRADA_LISTA_ESPERA : solicita
    GRUPO_PLAN_MATERIA ||--o{ ENTRADA_LISTA_ESPERA : materia_solicitada
    ENTRADA_LISTA_ESPERA ||--o| OFERTA_VACANTE : recibe
    OFERTA_VACANTE o|--|| NOTIFICACION : genera
    CUENTA_USUARIO ||--o{ NOTIFICACION : destinatario
```

**Reglas del diagrama 2**

- Cada grupo pertenece a un periodo y tiene un solo profesor responsable.
- `GrupoPlanMateria` resuelve la relación N:M entre grupo y materia del plan. Permite compartir un grupo entre planes únicamente cuando la equivalencia haya sido autorizada.
- Cada grupo presencial tendrá uno o varios bloques de horario; cada bloque se realiza en un aula. El cupo del grupo no puede superar la capacidad para alumnos de ninguna de las aulas asignadas.
- Cada inscripción pertenece a un alumno y a una combinación válida de grupo y materia del plan, garantizada mediante FK compuesta.
- Los cambios y bajas se conservan en `MovimientoInscripcion`; cada movimiento identifica la inscripción/intento y la combinación de grupo y materia de plan de origen. En un cambio, el destino debe ofrecer esa misma materia de plan; en una baja el destino queda vacío. El historial de movimientos y la continuidad entre grupos se validan transaccionalmente.
- Un grupo puede no tener lista de espera o tener una; las entradas se ordenan FIFO. Cada entrada se refiere a una materia de plan que el grupo realmente ofrece.
- Una entrada recibe como máximo una oferta. Si el alumno rechaza o deja vencer su oferta, sale de la lista; si fue omitido por no cumplir requisitos, conserva su posición y se revalida antes de crear una oferta posterior.
- Cada oferta debe generar exactamente una notificación interna en la misma transacción. Una notificación general puede no estar vinculada a ninguna oferta. Por eso la FK de `notificacion.id_oferta` es opcional, pero una oferta no debe quedar sin su notificación.

## Diagrama 3. Evaluación, reconocimientos, correcciones y trazabilidad

```mermaid
erDiagram
    PLAN_MATERIA ||--o{ UNIDAD_EVALUACION : define
    INSCRIPCION ||--o{ CALIFICACION_UNIDAD : obtiene
    UNIDAD_EVALUACION ||--o{ CALIFICACION_UNIDAD : evalua

    CALIFICACION_UNIDAD ||--o{ CORRECCION_CALIFICACION : historial
    CUENTA_USUARIO ||--o{ CORRECCION_CALIFICACION : solicita
    CUENTA_USUARIO o|--o{ CORRECCION_CALIFICACION : autoriza

    ALUMNO ||--o{ RECONOCIMIENTO_MATERIA : recibe
    PLAN_MATERIA ||--o{ RECONOCIMIENTO_MATERIA : destino
    INSCRIPCION o|--o{ RECONOCIMIENTO_MATERIA : origen
    CUENTA_USUARIO ||--o{ RECONOCIMIENTO_MATERIA : autoriza
```

**Reglas del diagrama 3**

- Cada unidad de evaluación pertenece a una materia dentro de un plan; las calificaciones pertenecen a una inscripción/intento y a una unidad de esa misma materia de plan. Las FK compuestas descritas en el modelo relacional refuerzan esta coincidencia.
- El estado de captura distingue borrador y publicada. No se mezcla con aprobado/reprobado.
- La calificación final se obtiene de las unidades publicadas conforme a la fórmula oficial que se verifique; puede conservarse como resultado calculado en la inscripción.
- Cada corrección se asocia con una calificación, una cuenta solicitante y una cuenta que resuelve/autorizador. Mientras esté pendiente, el autorizador puede ser NULL.
- Un reconocimiento pertenece a un alumno y a una materia destino del plan; la inscripción de origen es opcional para antecedentes externos. Cuando se informa, una FK compuesta garantiza que la inscripción de origen pertenece al mismo alumno. El autorizador queda registrado.
- Las relaciones de notificaciones y auditoría con cuenta/objeto están explicitadas en los diagramas 1 y 2 y en la matriz completa.

## Diagrama 4. Relaciones operativas entre cuentas y procesos

```mermaid
erDiagram
    CUENTA_USUARIO ||--o{ SESION_USUARIO : administra
    CUENTA_USUARIO ||--o{ NOTIFICACION : recibe
    OFERTA_VACANTE o|--|| NOTIFICACION : origina
    CUENTA_USUARIO o|--o{ AUDITORIA : actor

    CUENTA_USUARIO ||--o{ PERIODO_ESCOLAR : crea
    CUENTA_USUARIO ||--o{ VENTANA_MOVIMIENTO : configura
    CUENTA_USUARIO ||--o{ MOVIMIENTO_INSCRIPCION : solicita
    CUENTA_USUARIO o|--o{ MOVIMIENTO_INSCRIPCION : autoriza
    CUENTA_USUARIO o|--o{ EQUIVALENCIA_MATERIA : autoriza
    CUENTA_USUARIO ||--o{ CORRECCION_CALIFICACION : solicita
    CUENTA_USUARIO o|--o{ CORRECCION_CALIFICACION : autoriza
    CUENTA_USUARIO ||--o{ RECONOCIMIENTO_MATERIA : autoriza
```

**Nota:** las cuentas son actores transversales de los procesos. Se muestran juntas aquí para facilitar la revisión, aunque algunas de esas relaciones también aparecen en los diagramas de su módulo funcional.

## Correspondencia con el diagrama original

| Elemento original | Tratamiento |
|---|---|
| `Cursos` | Separar en `Materia` y `Grupo`. El profesor, periodo, cupo y programación pertenecen al grupo. |
| `Horarios` | Convertir en `HorarioGrupo`, con aula, día, hora de inicio y fin. |
| `Categorias` | No incluir hasta confirmar su significado académico. |
| `Profesores` | Conservar como perfil de `Empleado`; agregar vínculo común a `Persona`. |
| `Datos_contacto_p` | Integrar correo y teléfono a `Persona`. |
| `Departamentos` | Conservar; el historial de adscripción se guarda en `AdscripcionProfesor`. |
| `Modulos` | Interpretar provisionalmente como `UnidadEvaluacion`. |
| `Calf_modulos` | Convertir en `CalificacionUnidad`, separando valor y estado de captura. |
| `Historiales` | Convertir en `Inscripcion`, vinculada a alumno, grupo y materia del plan. |
| `Alumnos` | Conservar como perfil de `Persona`, con número de control único. |
| `Datos_contacto_a` | Integrar correo y teléfono a `Persona`. |

## Auditoría de conectividad

La lista completa de relaciones R01–R55, con cardinalidad, optionalidad y FK concreta, está en [relaciones-y-cardinalidades.md](./relaciones-y-cardinalidades.md). Esa matriz es la referencia para contrastar el diagrama y el modelo relacional.

**Resultado de revisión:** las cuatro vistas Mermaid representan las 36 entidades base, sin entidades aisladas, y las relaciones están documentadas en R01–R55. Se corrigieron las cardinalidades mínimas de grupo–materia de plan y grupo–horario; el movimiento de inscripción ahora se relaciona con la combinación grupo–materia de origen/destino; y una oferta debe generar una notificación, aunque las notificaciones generales no dependan de ofertas. La auditoría referencia al objeto de negocio mediante tipo de entidad + id de registro, una relación polimórfica explicada en la matriz.

## Pendiente antes de SQL

- Confirmar reglas oficiales de créditos, seriación, recursamiento, ponderaciones, redondeo, calificación mínima y equivalencias.
- Confirmar si una persona puede acumular más de un perfil/rol; la estructura actual permite un solo rol por cuenta.
- Validar atributos y restricciones finales, incluidas las FK compuestas y las reglas de exclusividad de subtipos.
- Revisar con el equipo si `Modulos` representa unidades de evaluación.
- Aprobar el modelo lógico y después traducirlo a DDL MariaDB.
