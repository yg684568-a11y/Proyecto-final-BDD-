# PROJECT_CONTEXT — Sistema de gestión escolar en clúster

> **Documento central de contexto.** Este archivo registra únicamente información confirmada, requisitos de la materia, propuestas pendientes y decisiones aprobadas. Debe mantenerse actualizado en el repositorio GitHub y compartirse con Gemini, Claude, ChatGPT y Antigravity para evitar repetir contexto.
>
> **Estado inicial:** borrador — levantamiento de requerimientos.  
> **Última actualización:** 2026-11-09.  
> **Regla:** ninguna propuesta tecnológica o modificación del modelo se considera aprobada hasta que el equipo la valide.

---

## 1. Resumen del proyecto

Desarrollar un sistema de gestión escolar para un **instituto**, que permita administrar información académica (por ejemplo, alumnos, profesores, cursos, horarios, inscripciones, calificaciones e historial académico) y que utilice una **base de datos distribuida en clúster de tres nodos**.

**Tipo de institución confirmado:** Instituto Tecnológico de Puebla (ITP). El sistema se basará en su funcionamiento académico. Las reglas concretas de inscripción/reinscripción, grupos, periodos, evaluación, horarios, kárdex y permisos deben documentarse con fuentes institucionales o confirmación del equipo; no se asumirán.

## 2. Equipo y nodos

| Integrante | Identificación | Responsabilidad conocida | Configuración / datos técnicos |
|---|---|---|---|
| Yosef | Nodo 1 | Coordinador del equipo; descrito inicialmente como nodo central | IP, hostname, SO y función técnica exacta: **PENDIENTE** |
| Andrés | Nodo 2 | Integrante responsable del nodo 2 | IP, hostname, SO y función técnica: **PENDIENTE** |
| Leonardo | Nodo 3 | Integrante responsable del nodo 3 | IP, hostname, SO y función técnica: **PENDIENTE** |

**Importante:** “Nodo central” describe por ahora la organización del equipo. No significa que el nodo 1 sea necesariamente un maestro exclusivo de la base de datos. La topología y el rol técnico de cada nodo deben definirse después de elegir el motor y el tipo de clúster.

## 3. Organización de herramientas

| Herramienta | Papel acordado para el proyecto |
|---|---|
| Gemini | IA principal: conducir las fases, analizar documentos y proponer diseños y procedimientos. |
| Claude | Revisor técnico: auditar modelo de datos, SQL, arquitectura, seguridad y propuestas de Gemini. |
| ChatGPT | Asesor y contraste técnico: resolver dudas, revisar decisiones y ayudar con redes, Linux, SQL y documentación. |
| Antigravity | Entorno/agente de implementación: trabajar con archivos, código, terminal y pruebas, bajo revisión del equipo. |

Las IA no deben aceptar automáticamente el trabajo de otra IA. Las decisiones deben contrastarse con documentación oficial, requisitos del profesor y pruebas reales.

## 4. Requisitos confirmados por la guía del proyecto

La guía del proyecto final de **Bases de Datos Distribuidas** solicita:

- Diseñar y configurar una solución de base de datos para control y consulta de historiales académicos o kárdex, distribución de horarios y gestión de listas.
- Documentar introducción, planteamiento del problema, objetivo general y objetivos específicos.
- Describir la arquitectura tecnológica seleccionada y las tecnologías utilizadas, incluidas máquinas virtuales y SGBD.
- Documentar el desarrollo, componentes arquitectónicos, configuraciones y modelo de datos (diagrama E-R o modelo lógico).
- Definir requerimientos funcionales y no funcionales.
- Integrar y probar operaciones `INSERT`, `DELETE` y `UPDATE`, así como transacciones `COMMIT` y `ROLLBACK`.
- Describir escenarios de prueba, comparar resultados esperados y obtenidos e incluir evidencias.
- Redactar conclusiones y bibliografía en formato APA 7.
- La guía destaca el uso de borrado lógico y la conservación de registros históricos frente al borrado físico.

**Nota de alcance:** la guía menciona un contexto del Instituto Tecnológico de Puebla (ITP) en el ejemplo del planteamiento del problema. Se debe confirmar si el sistema se construirá específicamente para el ITP o si ese texto es solo una referencia del profesor.

## 5. Tecnologías: estado de selección

Ninguna tecnología está aprobada definitivamente todavía.

| Componente | Opción por evaluar | Estado | Observaciones |
|---|---|---|---|
| Virtualización | VirtualBox / VMware / alternativa | **PENDIENTE** | Confirmar software disponible y dónde se alojarán las VMs. |
| Sistema operativo | Ubuntu Server | **CANDIDATO** | Confirmar versión y requisitos de recursos. |
| Motor de base de datos | MariaDB + Galera Cluster | **CANDIDATO** | Validar compatibilidad, requisitos y si cumple lo que pide el profesor. |
| Alternativa de base de datos | MySQL InnoDB Cluster u otra | **POR COMPARAR** | Elegir solo después de comparar requisitos y complejidad. |
| Backend | PHP + Laravel | **CANDIDATO** | Confirmar con el alcance y conocimientos del equipo. |
| Frontend | Blade + Bootstrap | **CANDIDATO** | Angular u otra opción solo si existe una necesidad concreta. |
| Control de versiones | Git + GitHub | **PROPUESTO** | Repositorio compartido del equipo. |
| Comunicación entre equipos/nodos | LAN / VPN (por ejemplo, Radmin VPN) | **PENDIENTE** | Confirmar topología y conectividad disponible. |

No instalar ni configurar estas tecnologías hasta registrar la elección y sus versiones en este documento.

## 6. Diagrama E-R recibido: inventario inicial

El diagrama entregado contiene las siguientes entidades y atributos visibles. Este inventario describe el diagrama actual; **no significa que el modelo ya esté validado**.

### 6.1 Entidades principales

- **Alumnos:** `id_alumno`, nombre, segundo nombre, primer apellido, segundo apellido y estado.
- **Datos_contacto_a:** `fk_id_alumno`, teléfono, segundo teléfono, correo y segundo correo.
- **Profesores:** `id_profesor`, nombre, segundo nombre, apellidos, grado académico, fecha de ingreso y `fk_id_departamento`.
- **Datos_contacto_p:** `fk_id_profesor`, teléfono, segundo teléfono, correo y segundo correo.
- **Departamentos:** `id_departamento`, nombre y descripción.
- **Cursos:** `id_curso`, nombre, `fk_id_profesor`, `fk_categoria`, descripción, nivel, duración en horas, estado, fecha de inicio y fecha de fin.
- **Categorias:** `id_categoria`, nombre y descripción.
- **Modulos:** `id_modulo`, título, descripción y `fk_id_curso`.
- **Horarios:** `id_horario`, `fk_id_curso`, día de la semana y `hora_fin` (según lo visible en la imagen; verificar si hay atributos recortados).
- **Historiales:** `id_inscripcion`, `fk_id_alumno`, `fk_id_curso`, calificación, fecha de inscripción, fecha fin, intento y estado.
- **Calf_modulos:** `id_calificación`, `fk_id_inscripcion`, `fk_id_modulo`, calificación del módulo y estado.

### 6.2 Observaciones para revisar en el rediseño

1. **Historiales / inscripciones:** por los campos `id_inscripcion`, alumno, curso, fecha de inscripción, intento y estado, la entidad parece representar una inscripción o intento de inscripción. Evaluar renombrarla a `inscripciones` y definir cómo se construirá el kárdex sin duplicar información.
2. **Grupos:** el modelo no muestra una entidad explícita de grupos/secciones. Si un curso puede impartirse a varios grupos, conviene modelar los grupos y asociar a ellos profesor y horarios.
3. **Periodos escolares:** no aparece una entidad para ciclos o periodos (por ejemplo, semestre o periodo). Confirmar si se necesitan y cómo se identifican.
4. **Horarios:** el diagrama muestra `hora_fin`, pero no se observa `hora_inicio`. Revisar también si el horario debe relacionarse con el grupo en vez de directamente con el curso.
5. **Aulas:** no aparece una entidad de aulas/salones. Agregarla solo si el sistema debe asignar espacios físicos.
6. **Profesor en Cursos:** `Cursos` contiene `fk_id_profesor`. Confirmar si un curso tiene un único profesor siempre o si cada grupo puede tener un profesor distinto. En el segundo caso, la relación probablemente debe trasladarse a grupos.
7. **Calificaciones de módulos:** revisar nombre de `Calf_modulos`, tipo de dato de la calificación, escala permitida y diferencia entre estado de entrega y resultado (pendiente/entregado frente a aprobado/reprobado).
8. **Datos de contacto:** confirmar si `Datos_contacto_a` y `Datos_contacto_p` son relaciones 1:1 obligatorias u opcionales y si sus claves foráneas también son claves primarias.
9. **Estados y borrado lógico:** definir estados válidos y una política de baja lógica/auditoría coherente con la guía. No hacer borrado físico de información académica sin una regla explícita.
10. **Restricciones:** revisar PK, FK, `UNIQUE`, `NOT NULL`, cardinalidades, índices y reglas para evitar inscripciones duplicadas o calificaciones repetidas indebidamente.
11. **Usuarios y permisos:** no se observan tablas de acceso, cuentas o roles. Confirmar si la aplicación requiere inicio de sesión para alumnos, profesores y administradores antes de incorporarlas.
12. **Convenciones:** definir nombres consistentes para tablas y columnas (por ejemplo, minúsculas y `snake_case`, sin espacios ni acentos) al aprobar el nuevo modelo.

### 6.3 Modelo propuesto, todavía no aprobado

Como hipótesis de trabajo, revisar esta estructura:

`periodos_escolares → grupos → inscripciones → calificaciones`

con cursos asociados a grupos, profesores asignados según las reglas académicas y horarios vinculados al grupo. Es solo una propuesta que depende de las respuestas sobre el funcionamiento real de la escuela.






















### 6.25 Estructura de materias dentro de los planes de estudio

**Decisión confirmada:** cada plan de estudios tendrá una estructura curricular propia de materias/asignaturas. Para cada materia se contempla registrar:
- Clave oficial.
- Nombre.
- Créditos.
- Semestre recomendado.
- Seriación/prerrequisitos, cuando correspondan.

**Reglas de diseño aprobadas:**
- Una misma materia podrá formar parte de diferentes planes de estudio.
- Se evitará duplicar la definición de una materia cuando represente la misma asignatura; cada plan la relacionará con su estructura curricular correspondiente.
- La relación de la materia con un plan deberá permitir conservar la ubicación curricular (por ejemplo, semestre recomendado) y los datos propios de esa inclusión.

**Pendiente de validación institucional:** catálogo de claves y nombres oficiales, sistema de créditos, seriación, equivalencias entre planes y tratamiento de materias optativas/residencias/servicio social/actividades complementarias, según aplique al plan correspondiente.

### 6.24 Auditoría del sistema

**Decisión confirmada:** el sistema contará con auditoría de acciones relevantes para mantener trazabilidad académica y administrativa.

**Eventos mínimos a registrar:**
- Inicios de sesión.
- Creación de registros.
- Modificaciones, incluyendo valores anteriores y nuevos cuando corresponda.
- Bajas lógicas/desactivaciones, con usuario, fecha y motivo.
- Captura, modificación y publicación de calificaciones.
- Autorizaciones y rechazos, incluyendo quién tomó la decisión.

**Acceso aprobado:**
- Administrador: podrá consultar toda la auditoría.
- Servicios Escolares y coordinación académica: podrán consultar únicamente movimientos relacionados con sus funciones.
- Los demás perfiles no tendrán acceso general a la auditoría; cualquier consulta adicional deberá definirse mediante permisos explícitos.

**Reglas de integridad:** los usuarios ordinarios no podrán modificar ni eliminar registros de auditoría mediante la aplicación. La estrategia técnica, retención, acceso de administradores de base de datos y protección contra alteración se definirá en el diseño de seguridad.

**Datos mínimos a evaluar para cada evento:** usuario/actor, acción, entidad afectada, identificador del registro, fecha/hora, resultado, motivo cuando aplique y valores anterior/nuevo cuando corresponda. Evitar guardar contraseñas, hashes de contraseñas, tokens o secretos en la auditoría.

### 6.23 Borrado lógico, bajas y conservación histórica

**Decisión confirmada:** se utilizará borrado lógico para evitar eliminar físicamente información que deba conservarse por razones académicas, administrativas o de auditoría.

**Aplicación general:**
- Alumnos, profesores y cuentas: podrán marcarse como inactivos o dados de baja, conservando sus datos e historial según los permisos y procedimientos aplicables.
- Carreras, planes, materias, grupos, aulas y catálogos: cuando ya no deban ofrecerse o utilizarse, deberán marcarse como inactivos/no vigentes, sin romper referencias históricas.
- Inscripciones y movimientos académicos: no se eliminarán físicamente como operación ordinaria; se conservarán y se registrará su estado (por ejemplo, activa, baja, cancelada o concluida, según el catálogo de estados que se apruebe).
- Calificaciones publicadas: no se borrarán físicamente; las correcciones seguirán el flujo controlado y auditado aprobado.
- Los cambios de estado deberán respetar permisos y, cuando corresponda, guardar quién realizó la acción, fecha/hora y motivo.

**Importante:** el borrado lógico no significa que todas las tablas deban tener exactamente una columna `deleted_at`. Cada entidad tendrá una estrategia apropiada (estado, vigencia, fecha de baja o registro de movimiento) según su naturaleza y las reglas institucionales. La eliminación física, si alguna vez fuera necesaria, requerirá una política explícita y autorización; no será el comportamiento normal de la aplicación.

### 6.22 Captura, validación y publicación de calificaciones

**Decisión confirmada:** el flujo de calificaciones será:
1. El profesor captura las calificaciones por unidad.
2. Puede guardar los datos como borrador sin que sean visibles para el alumno.
3. El profesor revisa/valida la captura y publica las calificaciones.
4. El alumno solo podrá consultar calificaciones publicadas.
5. Si se requiere una corrección después de publicar, deberá seguir un procedimiento controlado y quedar registrada para auditoría.

**Correcciones posteriores a la publicación — flujo aprobado:**
1. El profesor solicita la corrección e indica el motivo.
2. El coordinador académico revisa la solicitud.
3. Si la aprueba, se aplica la corrección.
4. El sistema conserva la calificación anterior, la nueva, el motivo, quién solicitó, quién autorizó y las fechas/horas de cada acción.

El profesor no podrá cambiar directamente una calificación publicada sin este flujo de autorización.

**Pendiente de detalle:** definir si existen casos excepcionales en los que también deba intervenir Servicios Escolares, los plazos de atención y el formato de la solicitud.

### 6.21 Materias no acreditadas y múltiples intentos

**Decisión confirmada:** cuando un alumno no acredite una materia, se conservará la calificación obtenida y podrá volver a inscribirse en esa materia en otro periodo, conforme a las reglas académicas aplicables. Cada vez que curse nuevamente la materia se registrará un intento independiente.

**Reglas aprobadas para el historial:**
- No sobrescribir calificaciones de intentos anteriores.
- Conservar cada intento con su periodo, grupo, calificaciones por unidad y resultado final.
- El kárdex mostrará los intentos anteriores y el resultado de cada uno.
- El número/orden del intento deberá poder identificarse de forma consistente.

**Pendiente de validación institucional:** cuántas oportunidades se permiten, diferencias entre curso ordinario/recursamiento/regularización, criterios de acreditación, cómo se calcula el promedio y qué calificación/resultado se considera para efectos oficiales. No asumir que el último o el mejor intento reemplaza automáticamente a los anteriores.

### 6.20 Carrera activa del alumno e historial de cambios

**Decisión confirmada:** cada alumno podrá tener una sola carrera activa a la vez. El sistema no permitirá que un mismo alumno mantenga dos carreras activas simultáneamente.

Si el alumno cambia de carrera o de plan de estudios:
- Se conservará el registro histórico de la carrera/plan anterior.
- Se registrará la nueva carrera/plan como su asignación activa.
- No se sobrescribirá ni eliminará la trayectoria académica previa.

**Pendiente de definición/verificación:** quién autoriza cambios de carrera o de plan, fechas efectivas, requisitos, equivalencias/revalidaciones y cómo se reflejan las materias cursadas en el nuevo plan, conforme a los procedimientos del ITP/TecNM.

### 6.19 Cambio y recuperación sencilla de contraseñas

**Decisión confirmada para el alcance inicial:**
- El usuario podrá cambiar su contraseña desde su cuenta.
- Si la olvida, solicitará apoyo al administrador.
- El administrador podrá asignar/generar una contraseña temporal.
- En el siguiente inicio de sesión, el usuario deberá cambiar esa contraseña temporal antes de continuar.
- Se registrará el restablecimiento para auditoría.
- No se integrará por ahora un servicio externo de correo para recuperación.

**Pendiente de definición:** procedimiento de verificación de identidad antes de restablecer una contraseña, política mínima de contraseñas, vigencia de la contraseña temporal y eventos exactos que se guardarán en auditoría. La contraseña temporal debe tratarse como credencial de un solo uso y no guardarse en registros de auditoría.

### 6.18 Autenticación y cuentas de usuario

**Decisión confirmada:** todos los perfiles tendrán cuenta de acceso con contraseña y permisos asociados a su rol.

| Perfil | Identificador de inicio de sesión |
|---|---|
| Administrador | Nombre de usuario |
| Servicios Escolares | Número de empleado |
| Coordinador académico | Número de empleado |
| Profesor | Número de empleado |
| Alumno | Número de control (matrícula) |

**Reglas aprobadas:**
- El administrador podrá crear y desactivar cuentas.
- Los usuarios podrán cambiar su contraseña.
- La cuenta deberá vincularse con el registro de alumno o profesor correspondiente cuando aplique.
- Las contraseñas nunca se almacenarán en texto plano; se almacenarán usando un algoritmo de hash de contraseñas adecuado.
- Los permisos se asignarán según el rol del usuario.

**Pendiente de definición:** recuperación de contraseña, política de contraseñas, bloqueo por intentos fallidos, sesiones, segundo factor (si se requiere), alta de cuentas del personal y auditoría de cambios de permisos.

### 6.17 Lista de espera automática para grupos llenos

**Decisión confirmada:** cuando un grupo alcance su cupo, el alumno podrá solicitar ingresar a una lista de espera. Cuando se libere un lugar, el sistema ofrecerá la vacante al siguiente alumno de la lista y este deberá confirmar su inscripción dentro de un plazo establecido.

**Flujo aprobado:**
1. El alumno intenta inscribirse en un grupo sin cupo disponible.
2. El sistema ofrece incorporarlo a la lista de espera.
3. Si se libera un lugar, el sistema contacta/ofrece la vacante al siguiente alumno elegible.
4. El alumno confirma dentro del plazo establecido.
5. Si no confirma dentro del plazo, la vacante podrá ofrecerse al siguiente alumno conforme a las reglas que se definan.

**Orden aprobado:** la lista de espera utilizará FIFO (First In, First Out): el alumno que se registró primero tendrá prioridad para recibir la vacante. Se debe conservar la fecha y hora de entrada para demostrar el orden.

**Plazo aprobado:** el alumno tendrá 24 horas para aceptar la vacante. Si no responde dentro del plazo, perderá esa oportunidad y el sistema ofrecerá el lugar al siguiente alumno según el orden FIFO. El plazo será configurable por periodo escolar.

**Canal aprobado:** la notificación de una vacante se mostrará dentro del sistema (notificación interna al iniciar sesión). El correo electrónico no forma parte del alcance inicial para este aviso.

**Inicio del plazo aprobado:** las 24 horas comienzan desde la fecha y hora en que el sistema genera la notificación, independientemente de que el alumno inicie sesión o la lea.

**Contenido mínimo del aviso:** materia, grupo, horario y fecha/hora límite para aceptar la vacante. La notificación no equivale a una inscripción confirmada; el alumno debe aceptar expresamente dentro del plazo.

**Pendiente de definición:** presentación de notificaciones dentro del sistema (bandeja, contador, estado leído/no leído), qué sucede si el alumno deja de cumplir requisitos académicos y cómo se sincroniza la lista con bajas/cambios de grupo.

### 6.16 Cambios de grupo y bajas de materias

**Decisión confirmada:** se combinará una ventana de movimientos con un flujo de autorización para excepciones:
- Durante una ventana de fechas establecida para cada periodo, el alumno podrá solicitar o realizar cambios de grupo y bajas de materias conforme a las reglas que se definan.
- Fuera de esa ventana, los cambios o bajas requerirán autorización de Servicios Escolares o de coordinación académica, según el tipo de movimiento.
- Los movimientos deberán conservar trazabilidad: alumno, inscripción afectada, tipo de movimiento, fecha, motivo cuando corresponda, estado de la solicitud y usuario que autorizó o rechazó.

**Regla de conservación:** no borrar físicamente la inscripción original para representar un cambio o una baja. Mantener el registro histórico y registrar el estado/movimiento correspondiente, de acuerdo con el diseño final.

**Pendiente de definición/verificación:** fechas y duración de ventanas, qué movimientos puede hacer directamente el alumno, qué autoridad aprueba cada excepción, causales, plazos y efectos académicos/administrativos de las bajas.

### 6.15 Cupo de grupos y capacidad de aulas

**Decisión confirmada:** el sistema manejará dos valores independientes:
- **Cupo del grupo:** máximo de alumnos que pueden inscribirse en ese grupo.
- **Capacidad del aula:** máximo de personas que puede admitir el espacio físico.

**Regla aprobada:** el cupo configurado para un grupo no podrá superar la capacidad del aula asignada. Esta validación debe ejecutarse al crear o modificar el grupo, al asignar/cambiar aula y al modificar el cupo.

**Ejemplo ilustrativo:** un grupo con cupo de 30 alumnos puede asignarse a un aula con capacidad de 35, pero no a un aula con capacidad de 25.

**Pendiente de definición:** si la capacidad considera solamente alumnos o también al profesor; tratamiento de grupos sin aula física y de sesiones en línea; excepciones autorizadas.

### 6.14 Horarios, aulas y espacios educativos

**Decisión confirmada:** los horarios de los grupos deberán poder registrar:
- Día de la semana.
- Hora de inicio.
- Hora de fin.
- Aula.
- Edificio.
- Modalidad (por ejemplo, presencial; las modalidades válidas se definirán después).

Se mantendrá un catálogo de aulas/espacios para evitar registrar repetidamente la información de un mismo salón.

**Reglas de validación aprobadas:**
- Un alumno no podrá confirmar grupos cuyos horarios se traslapen.
- Un aula no podrá estar asignada a dos grupos distintos en el mismo día y horario.
- La validación de disponibilidad del aula se realizará al crear o modificar la programación del grupo.

**Pendiente de definición:** capacidad de cada aula, tipos de espacio (salón, laboratorio, taller), recursos disponibles, modalidades admitidas y tratamiento de clases en línea o sesiones extraordinarias.

### 6.13 Validación automática de choques de horario

**Decisión confirmada:** antes de confirmar una selección de grupos, el sistema verificará que los horarios del alumno no se traslapen. Si dos grupos seleccionados tienen sesiones que coinciden en día y horario, la selección conflictiva no podrá confirmarse.

**Implicaciones para el futuro modelo y reglas:**
- Los horarios deberán registrar al menos día de la semana, hora de inicio y hora de fin.
- La validación debe considerar todas las sesiones de los grupos seleccionados dentro del periodo.
- Definir posteriormente manejo de sesiones extraordinarias, cambios de horario, zonas horarias (si aplicara) y excepciones autorizadas.
- El cupo y los choques de horario deben validarse dentro del flujo transaccional de inscripción para que no queden selecciones inválidas.

### 6.12 Configuración de unidades y ponderaciones por materia

**Decisión confirmada:** el número de unidades y sus ponderaciones serán configurables por materia/asignatura, de acuerdo con la instrumentación didáctica que corresponda. No se impondrá una cantidad fija de unidades ni ponderaciones iguales para todas las materias.

**Ejemplo ilustrativo (no son datos oficiales):** Bases de Datos puede tener 4 unidades, Redes 5 y Programación 3.

**Implicaciones para el futuro modelo:**
- Definir cómo se configura la estructura de evaluación de cada materia y cómo se conserva por periodo/grupo si cambia entre ciclos.
- Asociar las calificaciones de cada alumno a la unidad correspondiente de la materia/grupo que cursa.
- Calcular la calificación final utilizando las ponderaciones configuradas y validadas.
- Definir restricciones para impedir ponderaciones inválidas (por ejemplo, valores negativos o suma distinta a 100%, según la regla institucional que se confirme).

**Pendiente de verificación:** fórmula de cálculo, regla de suma de ponderaciones, escala, decimales, redondeo, calificación mínima aprobatoria y políticas de modificación/publicación. No codificar supuestos hasta revisar los criterios vigentes del ITP/TecNM.

### 6.11 Evaluación y calificaciones por unidades

**Decisión confirmada:** el profesor registrará varias calificaciones por unidad durante el semestre y el sistema calculará la calificación final de la materia.

**Alcance aprobado:**
- Registrar calificaciones de cada alumno por unidad y por grupo/materia cursada.
- Permitir consultar el avance por unidad.
- Calcular y conservar el resultado final de la materia.

**Reglas pendientes antes de implementar el cálculo:**
- Número de unidades: confirmar si cambia según la materia/curso.
- Ponderación: definir si las unidades tienen el mismo peso o si cada materia puede tener ponderaciones distintas.
- Escala, precisión decimal, redondeo y calificación mínima aprobatoria: verificar con la normatividad y políticas docentes vigentes.
- Evaluaciones de recuperación, segunda oportunidad, regularización y otros casos: verificar reglas del TecNM/ITP aplicables.
- Definir si una calificación puede modificarse después de publicada y qué rol puede autorizar el cambio; conservar trazabilidad.

**Criterio de diseño provisional:** no codificar un número fijo de unidades ni asumir ponderaciones iguales. El modelo deberá soportar la configuración de unidades y criterios de evaluación por materia/grupo o la estructura que resulte de la instrumentación didáctica autorizada.

**Fuentes normativas de referencia por validar para el ITP:**
- Lineamiento para el Proceso de Evaluación y Acreditación de Asignaturas del TecNM (documento publicado en un campus TecNM): https://www.culiacan.tecnm.mx/wp-content/uploads/2016/01/LINEAMIENTO-PARA-LA-EVALUACION-Y-ACREDITACION-DE-ASIGNATURAS-AGOSTO-2015.pdf
- La documentación institucional de gestión de curso del TecNM incluye listas de calificaciones por unidad y remite al lineamiento vigente aplicable: https://www.tecomatlan.tecnm.mx/wp-content/uploads/2023/02/TecNM-CA-MC-001-A05-PLAN-DE-CALIDAD-DEL-SERVICIO-EDUCATIVO.pdf

Estas fuentes orientan la revisión, pero no confirman por sí solas todas las reglas actualmente aplicables en el ITP. Verificar la versión vigente con el plantel o el profesor antes de implementar porcentajes, escalas o oportunidades.

### 6.10 Selección de grupos durante la reinscripción

**Decisión confirmada:** se utilizará el modelo de selección por el alumno. El alumno podrá consultar los grupos disponibles y elegir las materias/grupos y horarios que desea cursar, respetando el cupo disponible.

**Reglas funcionales confirmadas para el proyecto:**
- El alumno solo podrá seleccionar grupos que estén publicados/disponibles para el periodo correspondiente.
- El sistema deberá consultar y validar el cupo del grupo antes de confirmar la inscripción.
- Un grupo que haya alcanzado su cupo no aceptará nuevas inscripciones.
- La disponibilidad debe actualizarse correctamente después de cada inscripción o baja autorizada.

**Reglas pendientes de definición/verificación institucional:**
- Cómo se determinan las materias que el alumno puede cursar de acuerdo con su plan.
- Seriación y requisitos académicos (materias previas, créditos, situación académica).
- Prevención de choques de horario entre grupos seleccionados.
- Fechas de apertura/cierre, cambios de grupo, bajas y lista de espera.
- Capacidad y excepciones autorizadas por coordinación.

**Consideración técnica para la futura base de datos distribuida:** la validación de cupo y la creación de la inscripción deberán formar parte de una operación transaccional consistente, para evitar que dos solicitudes simultáneas ocupen el último lugar disponible. La estrategia concreta se definirá al diseñar e implementar la base de datos y se demostrará con pruebas concurrentes.

### 6.9 Oferta de materias por grupos

**Decisión confirmada:** una misma materia podrá ofertarse en varios grupos durante un periodo escolar. Cada grupo tendrá su propio profesor asignado, horario y lista de alumnos inscritos.

**Implicaciones para el futuro modelo (pendientes de diseño):**
- Diferenciar la materia del plan de estudios de su oferta concreta en un periodo.
- El grupo debe pertenecer a una oferta/curso impartido en un periodo y plan/carrera cuando corresponda.
- La asignación del profesor y el horario deben asociarse al grupo/oferta, no directamente al registro general de materia.
- La inscripción del alumno debe apuntar al grupo específico que cursará.
- Definir posteriormente reglas para cambios de grupo, cupo, choques de horario, grupos compartidos entre carreras y grupos sin profesor asignado.

### 6.8 Flujo académico de ingreso, reinscripción e inscripción a materias

**Decisión confirmada:** se diferenciarán estos procesos:

1. **Ingreso/alta del alumno:** registrar al alumno en una carrera y asociarlo con el plan de estudios que le corresponde.
2. **Reinscripción por periodo:** registrar su continuidad académica en cada periodo escolar.
3. **Inscripción a materias/grupos:** durante el periodo, asociar al alumno con las materias ofertadas en grupos concretos.
4. **Evaluación:** registrar las calificaciones correspondientes al periodo y a las materias cursadas.
5. **Historial académico/kárdex:** conservar la trayectoria del alumno a lo largo de los periodos, de acuerdo con su plan de estudios.

**Regla de diseño:** no confundir el alta o ingreso institucional del alumno con sus reinscripciones por periodo ni con la inscripción a cada materia/grupo. La estructura exacta de entidades, restricciones y cardinalidades se definirá en la fase de modelo de datos.

**Validación institucional pendiente:** revisar las reglas vigentes del ITP para carga académica, materias permitidas, seriación, créditos, bajas, repetición, regularización y límites de inscripción. No implementar estas reglas por suposición.

### 6.7 Periodos escolares

**Decisión confirmada:** el sistema manejará periodos escolares semestrales. Como estructura de referencia se contemplan dos periodos ordinarios: enero–junio y agosto–diciembre. También se permitirá registrar periodos extraordinarios o de verano cuando correspondan.

**Implicaciones para el futuro modelo (pendientes de diseño):**
- Los periodos deben ser registros configurables, no fechas rígidas incrustadas en el código.
- Cada periodo tendrá identificador, nombre/ciclo, fechas de inicio y fin, tipo (ordinario, extraordinario o verano) y estado, sujeto al diseño final.
- Grupos, oferta de materias e inscripciones deberán asociarse al periodo correspondiente.
- Las fechas y reglas oficiales de cada ciclo deberán verificarse con los calendarios y avisos vigentes del ITP.

### 6.6 Versionamiento de planes de estudio

**Decisión confirmada:** cada carrera podrá conservar varias versiones de su plan de estudios (vigentes y anteriores). Cada alumno deberá quedar asociado al plan de estudios que le corresponda, y los planes anteriores se conservarán aunque dejen de estar vigentes, para poder consultar correctamente su trayectoria e historial académico.

**Implicaciones para el futuro modelo (pendientes de diseño):**
- Una carrera podrá tener uno o varios planes de estudio.
- Un plan de estudios tendrá su propia versión/identificador y datos de vigencia por definir.
- Las materias/asignaturas deberán asociarse al plan correspondiente, evitando que cambios de un plan alteren el historial de alumnos de otro.
- La regla para asignar o cambiar de plan a un alumno (por ejemplo, equivalencias o cambio de plan) debe definirse con el procedimiento institucional del ITP; no asumir migraciones automáticas.

### 6.5 Alcance del catálogo de carreras

**Decisión confirmada por el equipo:** el diseño deberá permitir registrar todas las carreras del ITP y administrar sus respectivos planes de estudio. Para el desarrollo y las demostraciones se utilizará un conjunto limitado de datos de prueba, sin limitar el diseño a una sola carrera.

**Validación pendiente antes de cargar el catálogo:**
- La página institucional de la División de Estudios Profesionales describe la gestión de carreras, planes/programas de estudio, cargas académicas y horarios, y publica coordinaciones y retículas por programa: https://www.puebla.tecnm.mx/division-de-estudios-profesionales/
- El sitio institucional del ITP presenta su oferta académica: https://www.puebla.tecnm.mx/
- El catálogo central de TecNM consultado muestra al Instituto Tecnológico de Puebla con un conteo de 9 carreras: https://www.tecnm.mx/index.php?vista=Licenciaturas
- Las páginas pueden presentar diferencias de actualización o de agrupación de programas. Antes de insertar datos reales, confirmar el listado activo, nivel (licenciatura/posgrado), modalidad y planes vigentes correspondientes al ciclo que abarcará el sistema. No completar nombres o claves faltantes por inferencia.

**Alcance de datos de prueba:** crear inicialmente registros ficticios/representativos para pocas carreras y planes seleccionados; la estructura debe soportar todas las carreras. No utilizar datos personales reales de estudiantes o docentes.

### 6.4 Referencias institucionales verificadas (ITP)

Consulta inicial de páginas oficiales del ITP, realizada el 2026-11-09. Estos datos sirven para orientar el levantamiento; no sustituyen reglamentos, procedimientos completos ni validación del equipo:

- **Servicios Escolares:** su página institucional atribuye al departamento la coordinación de inscripción, reinscripción, cambios y traslados; procesos de regularización, acreditación, certificación y titulación; y trámites de convalidación, revisión, revalidación, equivalencia y constancias. Fuente: https://www.puebla.tecnm.mx/servicios-escolares/
- **División de Estudios Profesionales:** su página institucional describe funciones relacionadas con carreras, planes y programas de estudio, cargas académicas, seguimiento académico-administrativo, captura de horarios y optimización de espacios educativos; también menciona asesoría para asignación de carga y reinscripción mediante coordinadores de carrera. Fuente: https://www.puebla.tecnm.mx/division-de-estudios-profesionales/
- **Portal estudiantil AMBAR:** el sitio institucional incluye el acceso AMBAR Estudiantes. Un documento oficial de preguntas frecuentes de 2025 indica que el estudiante consulta calificaciones/boleta, kárdex, pagos y horario mediante su usuario. Debe verificarse el alcance vigente antes de replicar funciones. Fuente: https://www.puebla.tecnm.mx/ y https://www.puebla.tecnm.mx/wp-content/archivos/DesAcademico/archivos/prefuntas_frecuentes_2025C.pdf
- **Reinscripciones:** el sitio publica avisos y procesos por periodo, por lo que la aplicación de clase puede modelar periodos escolares y procesos de reinscripción, pero sus reglas concretas deben obtenerse de los avisos/reglamentos aplicables. Fuente: https://www.puebla.tecnm.mx/reinscripciones/

**Implicación para el proyecto:** los perfiles potenciales a evaluar son personal de Servicios Escolares, personal de División de Estudios Profesionales/coordinaciones de carrera, docentes y estudiantes. No asumir que todos requieren acceso ni que el sistema escolar del proyecto reemplazará los sistemas oficiales del ITP.

## 7. Estado del levantamiento de requerimientos

### 7.1 Decisiones funcionales confirmadas

Las decisiones aprobadas hasta ahora están documentadas en las secciones 6.4 a 6.12 y en el registro de decisiones. En resumen:

- Institución objetivo: Instituto Tecnológico de Puebla (ITP).
- Cinco perfiles: administrador, Servicios Escolares, coordinador académico, profesor y alumno.
- Doce módulos iniciales aprobados.
- Catálogo de todas las carreras del ITP; datos de prueba limitados.
- Versiones históricas de planes de estudio y asociación del alumno con su plan.
- Periodos semestrales ordinarios y periodos extraordinarios/verano.
- Separación entre ingreso, reinscripción por periodo e inscripción a materias/grupos.
- Una materia puede abrirse en varios grupos; cada grupo tiene profesor, horario y lista propios.
- El alumno selecciona grupos y horarios; el sistema valida cupo.
- Calificaciones por unidades con número de unidades y ponderaciones configurables por materia.

### 7.2 Requerimientos funcionales todavía por precisar

1. **Respondida:** el sistema debe impedir confirmar una selección de grupos si existen horarios traslapados para el mismo alumno.
2. **Respondida:** autenticación por nombre de usuario para administrador, número de empleado para personal/profesores y número de control para alumnos; contraseñas con hash, cuentas vinculadas a los registros correspondientes y administración básica de cuentas.
3. **Respondida:** el usuario podrá cambiar su contraseña; para recuperación, el administrador podrá asignar una contraseña temporal que deberá cambiarse en el siguiente acceso. Se mantendrá un procedimiento sencillo sin correo externo y se registrará el restablecimiento.
2. **Respondida:** el horario tendrá día, hora de inicio y fin, aula, edificio y modalidad; se administrará un catálogo de aulas y se impedirá asignar la misma aula a dos grupos en el mismo horario.
3. **Respondida:** el cupo del grupo y la capacidad del aula serán valores independientes; el cupo del grupo no podrá superar la capacidad del aula asignada.
4. **Respondida:** se permitirán cambios de grupo y bajas durante una ventana establecida; fuera de ella se requerirá autorización de Servicios Escolares o coordinación académica. Los movimientos conservarán trazabilidad y no se borrarán físicamente.
5. **Respondida:** cuando un grupo esté lleno, el alumno podrá apuntarse a una lista de espera automática. Al liberarse una vacante, se ofrecerá al siguiente alumno, quien deberá confirmarla dentro de un plazo. El orden, plazo, canal de notificación y reglas de elegibilidad quedan pendientes.
2. ¿Qué datos debe mostrar cada horario (día, hora de inicio, hora de fin, aula, modalidad u otros)?
3. ¿El cupo será obligatorio para todos los grupos? ¿Quién podrá autorizar excepciones?
4. ¿Se permitirán cambios de grupo, bajas parciales y listas de espera? ¿En qué ventanas de tiempo?
5. ¿Qué reglas de seriación, créditos y materias permitidas se deben aplicar al seleccionar materias?
6. ¿Qué datos y procesos exactos se incluirán en el expediente, kárdex y lista escolar?
7. ¿Qué reglas del ITP aplican a la captura, modificación, publicación y cálculo de calificaciones?

### 7.3 Requerimientos de infraestructura y desarrollo pendientes

8. ¿El profesor exige un motor o tecnología específica para el clúster?
9. ¿Qué función técnica tendrá el nodo 1: punto de acceso de la aplicación, coordinador operativo o algún rol distinto?
10. ¿Las tres máquinas virtuales estarán en una sola computadora o distribuidas entre las computadoras de los integrantes?
11. ¿Qué virtualizador utilizarán y cómo se comunicarán las VMs (LAN, VPN u otra red)?
12. ¿Qué lenguaje/framework prefiere o domina el equipo?
13. ¿Se evaluará también una aplicación web funcional o principalmente la base de datos distribuida?
14. ¿Cuál es la fecha de entrega y qué hitos intermedios existen?

### 7.4 Validación institucional pendiente

- Obtener/confirmar el catálogo oficial vigente de carreras, niveles, modalidades y planes del ITP.
- Verificar calendarios, periodos y reglas institucionales de inscripción/reinscripción.
- Verificar seriación, créditos, bajas, repetición, regularización y equivalencias.
- Verificar el lineamiento vigente de evaluación, escala, ponderaciones, redondeo y oportunidades.
- Distinguir las funciones que el sistema académico del proyecto debe simular de aquellas que corresponden a plataformas institucionales existentes.

Registrar las respuestas y su fecha cuando el equipo las confirme. No asumir reglas institucionales no documentadas.

## 8. Direccionamiento y configuración de nodos

Completar cuando se defina la infraestructura. No inventar IPs ni contraseñas.

| Parámetro | Nodo 1 — Yosef | Nodo 2 — Andrés | Nodo 3 — Leonardo |
|---|---|---|---|
| Hostname | **PENDIENTE** | **PENDIENTE** | **PENDIENTE** |
| Dirección IP | **PENDIENTE** | **PENDIENTE** | **PENDIENTE** |
| Máscara / prefijo | **PENDIENTE** | **PENDIENTE** | **PENDIENTE** |
| Gateway | **PENDIENTE** | **PENDIENTE** | **PENDIENTE** |
| Sistema operativo y versión | **PENDIENTE** | **PENDIENTE** | **PENDIENTE** |
| CPU / RAM / disco de VM | **PENDIENTE** | **PENDIENTE** | **PENDIENTE** |
| Motor y versión | **PENDIENTE** | **PENDIENTE** | **PENDIENTE** |
| Rol dentro del clúster | **PENDIENTE** | **PENDIENTE** | **PENDIENTE** |
| Estado de configuración | No iniciado | No iniciado | No iniciado |

### Registro de configuración

| Fecha | Nodo(s) | Configuración / comando aplicado | Resultado observado | Evidencia / archivo |
|---|---|---|---|---|
| — | — | Aún no se han registrado configuraciones | — | — |

## 9. Registro de decisiones aprobadas

Solo registrar aquí decisiones confirmadas por el equipo. Incluir motivo y fecha.


### Módulos aprobados (alcance funcional inicial)

El equipo aprobó los siguientes 12 módulos como alcance inicial del sistema. Su desarrollo todavía no ha comenzado; los flujos, reglas de negocio y permisos detallados se especificarán en fases posteriores.

| # | Módulo | Funcionalidad prevista |
|---:|---|---|
| 1 | Usuarios y roles | Inicio de sesión y permisos. |
| 2 | Alumnos | Expedientes y datos personales. |
| 3 | Profesores | Información y asignaciones docentes. |
| 4 | Departamentos y carreras | Organización académica. |
| 5 | Materias | Catálogo de asignaturas y planes de estudio. |
| 6 | Periodos escolares | Semestres y ciclos escolares. |
| 7 | Grupos | Asignación de alumnos y profesores. |
| 8 | Inscripciones | Inscripción y reinscripción. |
| 9 | Horarios | Distribución de clases. |
| 10 | Calificaciones | Captura y consulta de evaluaciones. |
| 11 | Kárdex | Historial académico del alumno. |
| 12 | Listas escolares | Listas de alumnos por grupo. |

### Roles confirmados

| Rol | Responsabilidad general aprobada |
|---|---|
| Administrador | Gestionar cuentas, roles y permisos. |
| Servicios Escolares | Gestionar alumnos, inscripciones y expedientes académicos. |
| Coordinador académico | Gestionar carreras, grupos, cargas académicas y horarios. |
| Profesor | Consultar listas y registrar calificaciones. |
| Alumno | Consultar materias, horarios, calificaciones y kárdex. |

Los permisos específicos (crear, consultar, modificar, autorizar y dar de baja) se detallarán por módulo en la matriz de acceso. No asumir permisos adicionales hasta aprobar esa matriz.

| ID | Decisión | Motivo / evidencia | Fecha | Estado |
|---|---|---|---|---|
| D-001 | Gemini será la IA principal; Claude revisará; ChatGPT asesorará; Antigravity apoyará la implementación. | Acuerdo inicial del equipo. | 2026-11-09 | Aprobada |
| D-002 | El equipo se compone de Yosef (nodo 1), Andrés (nodo 2) y Leonardo (nodo 3). | Información proporcionada por el equipo. | 2026-11-09 | Confirmada |
| D-003 | Motor de base de datos y topología del clúster. | Pendiente de evaluación y validación del profesor/equipo. | — | Pendiente |
| D-004 | El sistema estará basado en el funcionamiento académico del Instituto Tecnológico de Puebla (ITP). | Confirmación del usuario. Las reglas académicas concretas deben verificarse con fuentes institucionales o con el equipo. | 2026-11-09 | Aprobada |
| D-005 | Se utilizarán cinco perfiles: administrador, Servicios Escolares, coordinador académico, profesor y alumno. | Aprobación expresa del usuario sobre los roles y responsabilidades generales propuestos. | 2026-11-09 | Aprobada |
| D-006 | El alcance funcional inicial incluirá 12 módulos: usuarios y roles, alumnos, profesores, departamentos y carreras, materias, periodos escolares, grupos, inscripciones, horarios, calificaciones, kárdex y listas escolares. | Aprobación expresa del usuario a la propuesta de módulos. Los flujos y reglas detallados quedan pendientes. | 2026-11-09 | Aprobada |
| D-007 | El diseño del catálogo abarcará todas las carreras del ITP y sus planes de estudio; se utilizará un conjunto limitado de datos de prueba. | Confirmación del usuario mediante selección de la opción recomendada. El catálogo vigente se verificará con fuentes institucionales antes de cargar datos. | 2026-11-09 | Aprobada |
| D-008 | Cada carrera podrá conservar varias versiones de planes de estudio, y los alumnos permanecerán asociados al plan que les corresponda, incluyendo planes anteriores no vigentes. | Aprobación expresa del usuario. Las reglas de cambios de plan/equivalencias quedan pendientes de verificación institucional. | 2026-11-09 | Aprobada |
| D-009 | El sistema manejará periodos semestrales ordinarios (enero–junio y agosto–diciembre) y permitirá registrar periodos extraordinarios o de verano. | Aprobación expresa del usuario. Las fechas exactas se configurarán por ciclo y se validarán con el calendario vigente del ITP. | 2026-11-09 | Aprobada |
| D-010 | El sistema distinguirá el ingreso del alumno a una carrera/plan, la reinscripción por periodo y la inscripción a materias/grupos. | Aprobación expresa del usuario. Reglas académicas específicas pendientes de validación institucional. | 2026-11-09 | Aprobada |
| D-011 | Una materia podrá ofertarse en varios grupos por periodo; cada grupo tendrá su propio profesor, horario y lista de alumnos. | Aprobación expresa del usuario. Reglas de cupo, cambios de grupo y choques de horario quedan pendientes. | 2026-11-09 | Aprobada |
| D-012 | Durante la reinscripción, el alumno seleccionará los grupos, materias y horarios disponibles; el sistema bloqueará inscripciones cuando el grupo alcance su cupo. | Aprobación expresa del usuario. Las reglas de seriación, choques de horario, cambios y bajas quedan pendientes. | 2026-11-09 | Aprobada |
| D-013 | El profesor registrará varias calificaciones por unidad y el sistema calculará la calificación final de la materia. | Aprobación expresa del usuario. Número de unidades y fórmula/ponderaciones sujetos a validación con la instrumentación didáctica y normativa vigente del TecNM/ITP. | 2026-11-09 | Aprobada |
| D-014 | El número de unidades y sus ponderaciones serán configurables por materia, según la instrumentación didáctica aplicable. | Aprobación expresa del usuario. Fórmula y restricciones concretas pendientes de validación con las reglas vigentes. | 2026-11-09 | Aprobada |
| D-015 | El sistema validará automáticamente que los grupos seleccionados por un alumno no tengan horarios traslapados e impedirá confirmar selecciones conflictivas. | Aprobación expresa del usuario. La estructura de sesiones/horarios y posibles excepciones se detallará posteriormente. | 2026-11-09 | Aprobada |\n| D-016 | Los horarios registrarán día, inicio, fin, aula, edificio y modalidad; habrá catálogo de aulas y validación para impedir que un aula se asigne a dos grupos en el mismo horario. | Aprobación expresa del usuario. Capacidad, tipos de espacios, recursos y modalidades se detallarán posteriormente. | 2026-11-09 | Aprobada |\n| D-017 | El cupo del grupo no podrá superar la capacidad del aula asignada. El cupo del grupo y la capacidad del aula se administrarán por separado. | Aprobación expresa del usuario. Excepciones y tratamiento de modalidades sin aula física quedan pendientes. | 2026-11-09 | Aprobada |\n| D-018 | Los cambios de grupo y bajas podrán realizarse durante una ventana establecida; fuera de ella requerirán autorización de Servicios Escolares o coordinación académica. Los movimientos conservarán trazabilidad, sin borrar físicamente el registro original. | Aprobación expresa del usuario. Ventanas, autorizaciones específicas y efectos académicos quedan pendientes de definición/verificación. | 2026-11-09 | Aprobada |\n| D-019 | Los grupos llenos admitirán lista de espera automática; al liberarse una vacante, se ofrecerá al siguiente alumno, quien deberá confirmar en un plazo establecido. | Aprobación expresa del usuario. Orden FIFO aprobado; plazo, canal de notificación y elegibilidad quedan pendientes. | 2026-11-09 | Aprobada |

## 10. Registro de errores y soluciones

| ID | Fecha | Nodo / componente | Error y mensaje exacto | Causa confirmada | Solución aplicada | Verificación |
|---|---|---|---|---|---|---|
| — | — | — | Aún no hay errores registrados. | — | — | — |

No registrar una causa como confirmada si solo es una hipótesis. Añadir comandos y salidas relevantes, ocultando contraseñas, tokens y datos privados.

## 11. Fases y seguimiento

| Fase | Actividad | Estado | Criterio para darla por terminada |
|---|---|---|---|
| 1 | Levantamiento de requerimientos y alcance | **EN CURSO** | Preguntas respondidas y alcance validado por el equipo. |
| 2 | Rediseño E-R, normalización y modelo relacional | Pendiente | Diagrama, cardinalidades, PK/FK y restricciones aprobados. |
| 3 | Diseño de arquitectura del clúster | Pendiente | Motor, topología, roles, red y requisitos definidos. |
| 4 | Preparación de máquinas virtuales | Pendiente | Tres VMs conectadas y documentadas. |
| 5 | Instalación y configuración del clúster | Pendiente | Los tres nodos forman el clúster según pruebas verificables. |
| 6 | Pruebas de replicación, disponibilidad y recuperación | Pendiente | Escenarios y resultados documentados. |
| 7 | Desarrollo de la aplicación escolar | Pendiente | Funciones acordadas implementadas. |
| 8 | Integración aplicación–clúster | Pendiente | Operaciones de la aplicación verificadas contra el clúster. |
| 9 | Pruebas de transacciones y reglas de negocio | Pendiente | INSERT/UPDATE/DELETE, COMMIT/ROLLBACK y escenarios definidos probados. |
| 10 | Reporte y evidencias finales | Pendiente | Documento final, capturas, resultados y bibliografía completos. |

### Próximas tareas inmediatas

- [x] Definir el tipo de institución: instituto.
- [x] Definir la institución de referencia: Instituto Tecnológico de Puebla (ITP).
- [ ] Recopilar y verificar las reglas académicas del ITP que afecten el sistema.
- [x] Definir los cinco perfiles generales del sistema.
- [ ] Elaborar y aprobar la matriz detallada de permisos por módulo.
- [ ] Responder las demás preguntas de requerimientos de la sección 7.
- [x] Definir que el modelo debe soportar todas las carreras del ITP y sus planes de estudio.
- [ ] Confirmar el catálogo activo de carreras, modalidad, nivel y planes vigentes para el ciclo de referencia.\n- [x] Definir estructura curricular por plan y reutilización de materias compartidas.
- [x] Definir que se conservarán planes de estudio anteriores y la asociación del alumno con su plan correspondiente.
- [ ] Verificar reglas del ITP para cambios de plan, equivalencias y transición entre versiones.
- [ ] Confirmar el alcance real de las reglas de inscripción/reinscripción y evaluación.
- [ ] Revisar y aprobar el modelo E-R corregido.
- [ ] Elegir el motor y arquitectura de clúster con base en los requisitos del profesor.
- [ ] Completar direcciones IP y características de las VMs cuando se conozcan.

## 12. Reglas para las IA y colaboradores

Al recibir este archivo:

1. Léelo antes de proponer cambios o generar código.
2. Respeta los estados **APROBADO**, **PENDIENTE**, **CANDIDATO** y **POR VALIDAR**.
3. No conviertas una propuesta en decisión aprobada por tu cuenta.
4. No inventes IPs, versiones instaladas, resultados, errores ni evidencias.
5. Trabaja únicamente en la fase actual salvo que el equipo solicite explícitamente otra cosa.
6. Responde de forma concisa y evita repetir el contexto que ya aparece aquí.
7. Si detectas una contradicción, señala el apartado y propone una corrección; espera validación antes de cambiar una decisión aprobada.
8. Al finalizar una sesión, entrega un bloque breve de cambios sugeridos para actualizar este archivo: decisiones, configuraciones, errores, resultados y siguiente tarea.
9. No incluyas contraseñas, claves privadas, tokens, archivos `.env` ni datos personales reales en este documento o en GitHub.

## 13. Archivos del proyecto

Mantener una estructura sencilla en el repositorio; adaptar cuando el equipo lo necesite:

```text
sistema-gestion-escolar/
├── PROJECT_CONTEXT.md
├── README.md
├── docs/
│   ├── requisitos/
│   ├── modelo-datos/
│   ├── arquitectura/
│   ├── configuracion-nodos/
│   ├── pruebas/
│   └── evidencias/
├── database/
│   ├── schema/
│   ├── seeds/
│   └── procedures/
├── app/
└── .gitignore
```

No subir respaldos de máquinas virtuales, archivos pesados innecesarios, credenciales ni datos reales de alumnos.

---

## Fuentes base

- `Diagrama E-R sistema de escuela`: imagen del modelo proporcionada por el equipo. Se debe conservar el original en `docs/modelo-datos/`.
- `REPORTE FINAL BDD.pdf`: guía del proyecto final de Bases de Datos Distribuidas proporcionada por el equipo. Se debe conservar una copia en `docs/requisitos/`.

Este archivo es un resumen operativo, no sustituye los documentos originales ni la validación del profesor.
