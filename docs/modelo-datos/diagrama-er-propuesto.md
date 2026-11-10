# Diagrama E-R propuesto — Sistema de gestión escolar

**Versión:** 1.0 — propuesta para revisión del equipo.  
**Estado:** modelo conceptual/lógico preliminar; todavía no es el esquema SQL definitivo.  
**Base:** decisiones registradas en `PROJECT_CONTEXT.md` y diagrama E-R original compartido por el equipo.

## Criterio del rediseño

Se conserva lo aprovechable del diagrama original y se adapta a los procesos aprobados:

- `Cursos` se separa en `Materia` (catálogo académico) y `Grupo` (oferta real durante un periodo).
- `Historiales` pasa a ser `Inscripcion`, vinculada a un grupo y a la materia correspondiente al plan del alumno.
- `Horarios` pasa a depender de `Grupo`, con aula, día y hora de inicio/fin.
- `Modulos` se interpreta como `UnidadEvaluacion`; `Calf_modulos` como `CalificacionUnidad`. Esta es una decisión de diseño para continuar; se podrá ajustar si el equipo aclara otra intención.
- Se agregan carreras, planes, periodos, aulas, cuentas/roles, lista de espera, notificaciones y auditoría.
- Teléfono y correo se integran a los expedientes de alumno/profesor; se evita mantener tablas 1:1 de contacto sin necesidad.
- `Categorias` se deja fuera de la primera propuesta porque el diagrama original no explica qué representa ni qué regla controla.

## Diagrama 1. Estructura académica y usuarios

```mermaid
erDiagram
    ROL ||--o{ CUENTA_USUARIO : asigna
    ALUMNO ||--o| CUENTA_USUARIO : acceso
    PROFESOR ||--o| CUENTA_USUARIO : acceso
    CARRERA ||--|{ PLAN_ESTUDIO : contiene
    ALUMNO ||--o{ ALUMNO_PLAN : historial
    PLAN_ESTUDIO ||--o{ ALUMNO_PLAN : asignacion
    PLAN_ESTUDIO ||--|{ PLAN_MATERIA : integra
    MATERIA ||--o{ PLAN_MATERIA : aparece_en
    PLAN_MATERIA ||--o{ PRERREQUISITO : materia
    PLAN_MATERIA ||--o{ PRERREQUISITO : requisito
    DEPARTAMENTO ||--o{ ADSCRIPCION_PROFESOR : registra
    PROFESOR ||--o{ ADSCRIPCION_PROFESOR : historial
    PLAN_MATERIA ||--o{ EQUIVALENCIA_MATERIA : equivalencia_a
    PLAN_MATERIA ||--o{ EQUIVALENCIA_MATERIA : equivalencia_de

    ROL {
        int id_rol PK
        string nombre UK
        string descripcion
    }
    CUENTA_USUARIO {
        int id_usuario PK
        int id_rol FK
        int id_alumno FK "nullable unique"
        int id_profesor FK "nullable unique"
        string identificador_acceso UK
        string password_hash
        string estado
        int intentos_fallidos
    }
    ALUMNO {
        int id_alumno PK
        string numero_control UK
        string nombres
        string apellidos
        string correo_institucional UK
        string telefono "nullable"
        date fecha_ingreso
        string estado_academico
    }
    PROFESOR {
        int id_profesor PK
        string numero_empleado UK
        string nombres
        string apellidos
        string grado_academico
        string correo_institucional UK
        string telefono "nullable"
        date fecha_ingreso
        string estado
    }
    DEPARTAMENTO {
        int id_departamento PK
        string clave UK
        string nombre UK
        string descripcion
    }
    ADSCRIPCION_PROFESOR {
        int id_adscripcion PK
        int id_profesor FK
        int id_departamento FK
        date fecha_inicio
        date fecha_fin "nullable"
        boolean vigente
    }
    CARRERA {
        int id_carrera PK
        string clave UK
        string nombre
        string estado
    }
    PLAN_ESTUDIO {
        int id_plan PK
        int id_carrera FK
        string clave_version
        date vigencia_inicio
        date vigencia_fin "nullable"
        string estado
    }
    ALUMNO_PLAN {
        int id_alumno_plan PK
        int id_alumno FK
        int id_plan FK
        date fecha_inicio
        date fecha_fin "nullable"
        string estado
    }
    MATERIA {
        int id_materia PK
        string clave UK
        string nombre
        string descripcion
        string estado
    }
    PLAN_MATERIA {
        int id_plan_materia PK
        int id_plan FK
        int id_materia FK
        decimal creditos
        int semestre_sugerido
        decimal horas_teoria
        decimal horas_practica
    }
    PRERREQUISITO {
        int id_plan_materia PK, FK
        int id_plan_materia_requisito PK, FK
        string tipo
    }
    EQUIVALENCIA_MATERIA {
        int id_equivalencia PK
        int id_plan_materia_a FK
        int id_plan_materia_b FK
        string estado_autorizacion
        int id_usuario_autoriza FK
        date fecha_autorizacion
    }
```

### Notas del diagrama 1

- Una cuenta puede vincularse con un alumno o profesor. Las cuentas de administración y personal institucional pueden no tener esos dos vínculos; usan nombre de usuario o número de empleado como identificador de acceso.
- Cada alumno puede conservar varias asignaciones históricas de plan, pero solo una vigente.
- Cada profesor puede conservar varias adscripciones, pero solo una vigente.
- `PLAN_MATERIA` registra la materia dentro de una versión concreta del plan, con sus créditos y semestre sugerido.
- Las equivalencias entre planes requieren autorización conforme a las reglas institucionales.

## Diagrama 2. Periodos, grupos, horarios, inscripciones y lista de espera

```mermaid
erDiagram
    PERIODO_ESCOLAR ||--o{ GRUPO : programa
    PERIODO_ESCOLAR ||--o| VENTANA_MOVIMIENTO : configura
    PROFESOR ||--o{ GRUPO : responsable
    GRUPO ||--|{ GRUPO_PLAN_MATERIA : ofrece
    PLAN_MATERIA ||--o{ GRUPO_PLAN_MATERIA : correspondencia
    GRUPO ||--o{ HORARIO_GRUPO : agenda
    AULA ||--o{ HORARIO_GRUPO : ocupa
    GRUPO_PLAN_MATERIA ||--o{ INSCRIPCION : permite
    ALUMNO ||--o{ INSCRIPCION : realiza
    GRUPO ||--o| LISTA_ESPERA : puede_tener
    LISTA_ESPERA ||--o{ ENTRADA_LISTA_ESPERA : contiene
    ALUMNO ||--o{ ENTRADA_LISTA_ESPERA : solicita
    PLAN_MATERIA ||--o{ ENTRADA_LISTA_ESPERA : cursaria
    ENTRADA_LISTA_ESPERA ||--o{ OFERTA_VACANTE : recibe

    PERIODO_ESCOLAR {
        int id_periodo PK
        string nombre
        string tipo
        date fecha_inicio
        date fecha_fin
        int horas_aceptacion_vacante
        string estado
    }
    VENTANA_MOVIMIENTO {
        int id_ventana PK
        int id_periodo FK, UK
        datetime fecha_apertura
        datetime fecha_cierre
    }
    GRUPO {
        int id_grupo PK
        int id_periodo FK
        int id_profesor FK
        string clave_grupo
        int cupo
        datetime fecha_limite_inscripcion
        string estado
    }
    GRUPO_PLAN_MATERIA {
        int id_grupo PK, FK
        int id_plan_materia PK, FK
    }
    AULA {
        int id_aula PK
        string edificio
        string clave_aula UK
        int capacidad_alumnos
        string tipo
        string estado
    }
    HORARIO_GRUPO {
        int id_horario PK
        int id_grupo FK
        int id_aula FK
        string dia_semana
        time hora_inicio
        time hora_fin
    }
    INSCRIPCION {
        int id_inscripcion PK
        int id_alumno FK
        int id_grupo FK
        int id_plan_materia FK
        datetime fecha_inscripcion
        date fecha_baja "nullable"
        int numero_intento
        decimal calificacion_final "nullable"
        string estado
    }
    LISTA_ESPERA {
        int id_lista PK
        int id_grupo FK, UK
        datetime fecha_apertura
        datetime fecha_cierre "nullable"
        string estado
    }
    ENTRADA_LISTA_ESPERA {
        int id_entrada PK
        int id_lista FK
        int id_alumno FK
        int id_plan_materia FK
        datetime fecha_hora_registro
        string estado
    }
    OFERTA_VACANTE {
        int id_oferta PK
        int id_entrada FK
        datetime fecha_hora_oferta
        datetime fecha_limite
        datetime fecha_respuesta "nullable"
        string respuesta_estado
    }
```

### Notas del diagrama 2

- `GRUPO` es la oferta de una materia en un periodo. `GRUPO_PLAN_MATERIA` permite ofrecer el mismo grupo a materias equivalentes de distintos planes, cuando se autorice.
- `INSCRIPCION` conserva el grupo y la materia del plan del alumno. La pareja `id_grupo` + `id_plan_materia` debe existir en `GRUPO_PLAN_MATERIA` (FK compuesta).
- El cupo del grupo no puede superar la capacidad del aula; para esta regla solo se cuentan alumnos.
- Cada horario pertenece a un grupo y aula; se validan traslapes de aula y choques de horario del alumno.
- La lista de espera usa FIFO por fecha/hora de registro. Si un alumno deja de ser elegible, conserva su posición y se registra el motivo de omisión.
- Una entrada puede recibir varias ofertas a lo largo del tiempo. Cada oferta conserva fecha, vencimiento y respuesta. El plazo inicial es de 24 horas desde la generación del aviso.
- Mientras tenga una oferta pendiente, el alumno no puede mover esa misma materia; sus demás materias no se bloquean.
- La regla de una sola inscripción activa por materia/equivalente en el periodo se valida como regla de negocio y con las restricciones de datos que correspondan.
- `VENTANA_MOVIMIENTO` contiene la apertura y cierre configurados por coordinación académica.

## Diagrama 3. Evaluación, reconocimientos, notificaciones y auditoría

```mermaid
erDiagram
    MATERIA ||--|{ UNIDAD_EVALUACION : define
    INSCRIPCION ||--o{ CALIFICACION_UNIDAD : obtiene
    UNIDAD_EVALUACION ||--o{ CALIFICACION_UNIDAD : evalua
    CALIFICACION_UNIDAD ||--o{ CORRECCION_CALIFICACION : historial
    CUENTA_USUARIO ||--o{ CORRECCION_CALIFICACION : solicita
    CUENTA_USUARIO ||--o{ CORRECCION_CALIFICACION : autoriza
    ALUMNO ||--o{ RECONOCIMIENTO_MATERIA : solicita
    PLAN_MATERIA ||--o{ RECONOCIMIENTO_MATERIA : reconoce
    INSCRIPCION ||--o{ RECONOCIMIENTO_MATERIA : origen
    CUENTA_USUARIO ||--o{ RECONOCIMIENTO_MATERIA : autoriza
    CUENTA_USUARIO ||--o{ NOTIFICACION : recibe
    CUENTA_USUARIO ||--o{ AUDITORIA : ejecuta

    UNIDAD_EVALUACION {
        int id_unidad PK
        int id_materia FK
        string nombre
        decimal ponderacion
        int orden
        string estado
    }
    CALIFICACION_UNIDAD {
        int id_calificacion PK
        int id_inscripcion FK
        int id_unidad FK
        decimal calificacion
        string estado_captura
        datetime fecha_captura
        datetime fecha_publicacion "nullable"
    }
    CORRECCION_CALIFICACION {
        int id_correccion PK
        int id_calificacion FK
        int id_usuario_solicita FK
        int id_usuario_autoriza FK
        decimal valor_anterior
        decimal valor_nuevo
        string motivo
        string estado
        datetime fecha_solicitud
        datetime fecha_resolucion "nullable"
    }
    RECONOCIMIENTO_MATERIA {
        int id_reconocimiento PK
        int id_alumno FK
        int id_plan_materia_destino FK
        int id_inscripcion_origen FK "nullable"
        int id_usuario_autoriza FK
        string tipo
        string estado
        date fecha_resolucion "nullable"
    }
    NOTIFICACION {
        int id_notificacion PK
        int id_usuario FK
        string tipo
        string titulo
        string contenido
        datetime fecha_creacion
        datetime fecha_lectura "nullable"
    }
    AUDITORIA {
        int id_auditoria PK
        int id_usuario FK
        string accion
        string entidad
        int id_registro
        datetime fecha_hora
        string motivo
        text valor_anterior
        text valor_nuevo
    }
```

### Notas del diagrama 3

- Las unidades de evaluación son configurables por materia. Las calificaciones pertenecen a una inscripción/intentο concreto.
- `estado_captura` distingue borrador y publicado; no mezcla estos estados con aprobado/reprobado.
- La calificación final se guarda en `INSCRIPCION` como resultado calculado con la fórmula que se valide institucionalmente.
- Una corrección publicada requiere solicitud del profesor y autorización del coordinador; se guarda valor anterior, nuevo, motivo y responsables.
- El kárdex se consulta a partir de inscripciones y reconocimientos históricos; no se crea una tabla redundante para el mismo contenido.
- Las notificaciones no se eliminan definitivamente ni vencen en el alcance inicial; `fecha_lectura` nula indica que no se ha leído.
- La auditoría guarda acciones relevantes, nunca contraseñas ni secretos.

## Correspondencia entre el diagrama original y el propuesto

| Diagrama original | Tratamiento propuesto |
|---|---|
| `Cursos` | Dividir en `Materia` y `Grupo`. Profesor, periodo, cupo y programación pasan al grupo. |
| `Horarios` | Convertir en `HorarioGrupo`, relacionado con grupo y aula; agregar hora de inicio. |
| `Categorias` | Dejar fuera hasta aclarar su función. |
| `Profesores` | Conservar; agregar número de empleado y correo institucional únicos. |
| `Datos_contacto_p` | Integrar correo y teléfono al expediente del profesor. |
| `Departamentos` | Conservar y agregar clave única; registrar cambios mediante `AdscripcionProfesor`. |
| `Modulos` | Convertir en `UnidadEvaluacion`, con la interpretación de unidades de calificación. |
| `Calf_modulos` | Convertir en `CalificacionUnidad`, separando el estado de captura del valor numérico. |
| `Historiales` | Convertir en `Inscripcion`, vinculada a alumno, grupo y materia del plan. |
| `Alumnos` | Conservar; agregar número de control y correo únicos. |
| `Datos_contacto_a` | Integrar correo y teléfono al expediente del alumno. |

## Reglas de integridad que implementaremos

1. PK en cada entidad y FK para todas las relaciones.
2. UNIQUE para número de control, número de empleado, correos institucionales, claves de catálogos y login.
3. Un solo plan vigente por alumno y una sola adscripción vigente por profesor.
4. Un único profesor responsable por grupo en la primera versión.
5. Cupo del grupo menor o igual a capacidad del aula.
6. Evitar traslapes de horario por alumno y por aula.
7. Evitar inscripciones activas duplicadas en la misma materia o equivalente y periodo.
8. Revalidar prerrequisitos, carga de créditos y elegibilidad antes de confirmar una inscripción u ofrecer una vacante.
9. Conservar intentos, calificaciones, reconocimientos, movimientos y auditoría; evitar borrado físico de registros académicos.
10. Probar operaciones concurrentes y consistencia entre los nodos del clúster.

## Pendientes antes del esquema SQL

- Confirmar con las reglas vigentes del ITP/TecNM créditos, seriación, recursamiento, ponderaciones, redondeo y calificación mínima.
- Revisar si la interpretación de `Modulos` como unidades de evaluación corresponde a la intención original del equipo; esta propuesta la adopta para dar continuidad.
- Terminar de definir atributos, tipos de datos, índices, PK/FK compuestas y restricciones de MariaDB.
- Validar la cardinalidad y restricciones de grupos compartidos y materias equivalentes.
- Revisar y aprobar el modelo lógico con el equipo antes de generar scripts SQL.

**Regla:** esta propuesta no debe implementarse directamente. Primero se revisará el modelo, se corregirán las cardinalidades y se aprobará el esquema relacional.
