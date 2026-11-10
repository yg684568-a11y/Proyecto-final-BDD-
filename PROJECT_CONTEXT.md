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

























### 6.28 Grupos compartidos entre carreras y planes

**Decisión confirmada:** se permitirá que alumnos de distintas carreras se inscriban en un mismo grupo cuando la materia sea equivalente en sus respectivos planes de estudio y el ITP permita compartir esa oferta académica.

**Reglas de diseño:**
- El grupo/oferta se registra una sola vez, con un profesor, horario, aula y cupo comunes.
- Los alumnos de las carreras/planes autorizados podrán inscribirse al mismo grupo.
- Se deberá verificar que la materia corresponda o sea equivalente en el plan del alumno.
- La inscripción debe conservar la carrera y el plan del alumno al momento de cursar, para mantener el historial académico correcto.
- La lista del grupo integrará alumnos de las distintas carreras sin duplicar la definición del grupo.

**Pendiente de validación institucional:** criterios oficiales de equivalencia, autorización para grupos multi-carrera, claves de materias compartidas y reglas de reconocimiento de la asignatura en cada plan. No asumir equivalencia solamente porque los nombres de las materias sean iguales.

### 6.27 Validación de carga académica y créditos

**Decisión confirmada:** el sistema calculará automáticamente la carga académica seleccionada por el alumno durante un periodo, sumando los créditos de las materias elegidas y comparándolos con el límite configurado para ese periodo.

**Flujo aprobado:**
1. Consultar los créditos de cada materia seleccionada.
2. Calcular la suma de créditos de la carga académica.
3. Compararla con el máximo permitido para el alumno/periodo, según las reglas institucionales.
4. Bloquear la selección cuando supere el límite.
5. Permitir excepciones únicamente cuando la normativa vigente lo permita, con autorización de coordinación académica y registro de auditoría.

**Pendiente de validación institucional:** máximo y mínimo de créditos, límites diferenciados por situación académica, reglas de carga reducida o especial y condiciones para autorizar excepciones. No fijar un número de créditos hasta verificar la normativa vigente del ITP/TecNM.

### 6.26 Validación de prerrequisitos académicos

**Decisión confirmada:** el sistema validará automáticamente los prerrequisitos antes de permitir que un alumno seleccione/inscriba una materia.

**Flujo aprobado:**
1. Consultar el plan de estudios asociado al alumno.
2. Identificar los prerrequisitos establecidos para la materia que desea cursar.
3. Verificar en el historial académico del alumno si cada prerrequisito está acreditado conforme a las reglas aplicables.
4. Si cumple, permitir que continúe con la selección del grupo, sujeto a cupo y horario.
5. Si no cumple, bloquear la inscripción y mostrar el motivo.
6. Permitir excepciones únicamente cuando la normativa y el procedimiento del ITP lo permitan, con autorización de coordinación académica y registro de auditoría.

**Pendiente de validación institucional:** reglas exactas de seriación, materias que pueden cursarse simultáneamente, tipos de prerrequisito (materia aprobada, créditos, semestre, promedio u otro), equivalencias y facultades de autorización de excepciones. No asumir que todos los prerrequisitos son simples relaciones entre dos materias.

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

**Revalidación aprobada:** justo antes de enviar una oferta de vacante, el sistema volverá a comprobar que el siguiente alumno de la lista todavía cumpla los requisitos académicos de inscripción: prerrequisitos, ausencia de choques de horario, límite de créditos e inscripción única por materia/equivalencia. La entrada previa a la lista no garantiza que siga siendo elegible si su situación académica cambió.

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

**Administración aprobada:**
- El coordinador académico será responsable de abrir/crear los grupos de cada materia en el periodo escolar y asignarles profesor, horario y cupo.
- Servicios Escolares podrá consultar los grupos disponibles para gestionar las inscripciones, pero no modificar su configuración.
- El coordinador académico podrá modificar el profesor, horario o cupo de un grupo después de su apertura.
- Si el grupo ya tiene alumnos inscritos, cada modificación deberá registrarse en la auditoría y notificarse a los alumnos afectados.
- Antes de guardar un cambio de horario, el sistema verificará que no genere conflictos con los horarios de los alumnos inscritos ni con la disponibilidad del aula. Si detecta un conflicto, impedirá guardar el cambio.
- El coordinador académico podrá cancelar un grupo cuando sea necesario.
- Si el grupo tiene alumnos inscritos, deberá registrarse el motivo de cancelación y el sistema notificará a los alumnos afectados para que puedan seleccionar otro grupo disponible.
- El coordinador académico podrá establecer una fecha límite de inscripción para cada grupo.
- Una vez alcanzada la fecha límite, el sistema impedirá nuevas inscripciones y nuevas entradas a la lista de espera de ese grupo, salvo autorización de Servicios Escolares o coordinación académica.

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

**Administración aprobada:**
- El coordinador académico podrá crear y configurar los periodos escolares, incluyendo sus fechas de inicio y fin.
- Servicios Escolares podrá consultar los periodos para gestionar los procesos de inscripción, pero no modificar su configuración.

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

### 6.29 Kárdex académico acumulativo

**Decisión confirmada:** el alumno podrá consultar un kárdex acumulativo que conserve su trayectoria académica completa y no sustituya intentos anteriores.

**Información mínima aprobada para cada registro cursado:**
- Carrera y plan de estudios que correspondían al alumno.
- Periodo escolar.
- Materia/asignatura.
- Créditos.
- Grupo.
- Calificación final.
- Número de intento.
- Estado de la materia (por ejemplo, acreditada/no acreditada, según los estados que se definan).

**Consultas adicionales aprobadas:**
- Calificaciones por unidad de cada intento, cuando estén publicadas.
- Intentos anteriores de una misma materia y el resultado de cada uno.
- Trayectoria histórica aunque el alumno haya cambiado de carrera o de plan, manteniendo una sola carrera activa a la vez.

**Pendiente de definición/verificación:** formato final del kárdex, promedio general y por periodo, avance curricular, créditos acumulados, equivalencias y reglas institucionales para calcularlos. No inferir cómo el ITP determina promedios o avance oficial.

### 6.30 Listas escolares por grupo

**Decisión confirmada:** los profesores podrán consultar las listas de los grupos que tengan asignados. Cada lista mostrará, como mínimo:
- Número de control del alumno.
- Nombre del alumno.
- Carrera.
- Grupo.
- Estado de inscripción.

**Exportación aprobada:** las listas podrán descargarse en formato PDF y Excel.

**Acceso aprobado:**
- Administrador: todas las listas.
- Servicios Escolares: todos los grupos.
- Coordinador académico: grupos de las carreras que administra.
- Profesor: únicamente sus propios grupos.
- Alumno: no puede consultar listas completas de otros estudiantes.

Las descargas PDF y Excel respetarán los mismos permisos.

**Pendiente de definición:** filtros disponibles, datos adicionales permitidos y tratamiento de listas cuando existan bajas, cambios de grupo o alumnos en lista de espera. La lista oficial del grupo debe reflejar únicamente las inscripciones confirmadas/activas, salvo que se solicite explícitamente un reporte histórico.

### 6.31 Expediente inicial del alumno

**Decisión confirmada:** el expediente inicial del alumno incluirá:
- Número de control (identificador institucional), obligatorio y único para cada alumno.
- Nombre(s).
- Apellidos.
- Correo institucional.
- Teléfono (opcional).
- Fecha de ingreso.
- Estado académico.

**Regla de diseño:** la carrera y el plan de estudios no se duplicarán como campos directos de la ficha del alumno; se manejarán mediante relaciones y registros históricos de asignación académica, permitiendo conservar cambios de carrera/plan.

**Restricción aprobada:** el número de control será obligatorio y único. El sistema impedirá registrar un nuevo alumno si el número de control ya está asociado a otro expediente. En el modelo relacional se implementará una restricción de unicidad además de la validación de campo obligatorio.

**Criterio de minimización:** almacenar únicamente los datos necesarios para los procesos académicos aprobados. No incorporar datos personales sensibles o adicionales sin un requerimiento claro y justificado.

**Responsabilidad operativa aprobada:** Servicios Escolares registrará a los alumnos nuevos, actualizará sus expedientes y asignará su carrera y plan de estudios correspondientes. El administrador se encargará de las cuentas de acceso, roles y permisos, no de la gestión académica de los expedientes.

### 6.32 Expediente inicial del profesor

**Decisión confirmada:** el expediente inicial del profesor incluirá:
- Número de empleado (identificador institucional), obligatorio y único para cada profesor.
- Nombre(s).
- Apellidos.
- Grado académico.
- Correo institucional.
- Teléfono (opcional).
- Fecha de ingreso.
- Departamento.
- Estado.

**Regla de diseño:** el departamento se almacenará en un catálogo independiente y se relacionará con el profesor mediante una llave foránea, evitando repetir el nombre del departamento en cada expediente.

**Restricción aprobada:** el número de empleado será obligatorio y único. El sistema impedirá registrar un profesor si ese número ya está asociado a otro expediente. En el modelo relacional se implementará una restricción de unicidad además de la validación de campo obligatorio.

**Pendiente de definición:** si un profesor puede pertenecer a más de un departamento, impartir materias en diferentes carreras o tener varios datos de contacto; revisar también qué campos exige el expediente institucional del ITP.

### 6.33 Relación entre profesores y departamentos

**Decisión confirmada:** cada profesor pertenecerá a un solo departamento. Un departamento podrá tener varios profesores.

**Implicación para el modelo relacional:** relación uno a muchos entre `departamentos` y `profesores`; la tabla de profesores tendrá una llave foránea hacia el departamento correspondiente.

**Pendiente de definición:** si el departamento de un profesor puede cambiar con el tiempo y, en ese caso, si se requiere conservar el historial de adscripciones.

### 6.34 Asignación de profesores a materias y grupos

**Decisión confirmada:**
- Un profesor podrá impartir varias materias.
- Un profesor podrá estar asignado a varios grupos durante un periodo escolar.
- Cada grupo tendrá un profesor responsable.

**Implicación para el modelo:** la asignación docente se realizará en la oferta/grupo del periodo, no directamente en la definición general de la materia. De este modo, la misma materia puede abrir varios grupos con profesores distintos y un profesor puede atender varios grupos.

**Pendiente de definición:** si se permitirán dos o más profesores responsables o docentes auxiliares en un mismo grupo; si existe límite de carga docente; y cómo se registra la asignación temporal o sustitución de un profesor.

### 6.35 Inscripción única por materia en el mismo periodo

**Decisión confirmada:**
- Un alumno no podrá mantener dos inscripciones activas a la misma materia dentro del mismo periodo escolar, aunque existan varios grupos disponibles.
- La restricción también deberá considerar las materias reconocidas como equivalentes para efectos de inscripción, de acuerdo con el catálogo de equivalencias que se valide.
- Para cambiar de grupo, el alumno deberá utilizar el procedimiento de cambio aprobado; el movimiento conservará la trazabilidad y el historial correspondiente.

**Implicación para el modelo:** se necesitará una validación que impida inscripciones activas duplicadas por alumno, periodo y materia (o equivalencia aplicable), sin impedir que la misma materia se curse en periodos posteriores como un nuevo intento.

### 6.36 Requisitos para ingresar a la lista de espera

**Decisión confirmada:** un alumno solo podrá ingresar a la lista de espera de un grupo lleno cuando cumpla las reglas académicas correspondientes:
- Tener acreditados los prerrequisitos de la materia.
- No presentar choques de horario con sus grupos inscritos.
- No exceder el límite de créditos permitido para el periodo.
- No tener una inscripción activa en la misma materia ni en una materia equivalente durante ese periodo.

**Implicación para el sistema:** antes de agregar al alumno a la lista de espera, se ejecutarán las mismas validaciones académicas aplicables a la inscripción. Esta regla complementa el orden FIFO y el plazo de aceptación de 24 horas ya aprobados.

### 6.37 Revalidación al liberar una vacante

**Decisión confirmada:** inmediatamente antes de ofrecer una vacante, el sistema volverá a validar que el siguiente alumno en el orden FIFO todavía cumpla los requisitos académicos de inscripción definidos para la lista de espera.

**Tratamiento aprobado para alumnos no elegibles:** si el alumno deja de cumplir algún requisito al momento de la revalidación, el sistema no le enviará la oferta, registrará el motivo de la omisión y continuará con el siguiente alumno según el orden FIFO. El alumno omitido conservará su posición en la lista; por tanto, podrá ser revalidado cuando se libere otra vacante. El alumno omitido recibirá una notificación interna con el motivo por el que no pudo recibir la vacante y se le informará que conserva su posición en la lista. El mecanismo para corregir su situación queda pendiente.

### 6.38 Cierre automático de listas de espera

**Decisión confirmada:** la lista de espera de un grupo se cerrará automáticamente cuando comience el periodo de clases de la materia o cuando termine el periodo escolar, lo que ocurra primero.

**Notificación aprobada:** al cerrarse la lista, el sistema enviará una notificación interna a los alumnos que permanezcan en ella para informarles del cierre.

**Implicación para el sistema:** después del cierre no se aceptarán nuevas entradas ni se ofrecerán vacantes mediante esa lista. Se conservará el registro de sus integrantes y movimientos para consulta y auditoría.

### 6.39 Historial auditable de listas de espera

**Decisión confirmada:** el sistema conservará el historial de cada alumno que haya formado parte de una lista de espera, incluyendo:
- Fecha y hora de ingreso.
- Posición FIFO asignada.
- Ofertas de vacantes y respuestas del alumno.
- Motivos de omisión por falta de elegibilidad.
- Fecha y motivo de salida de la lista o de su cierre.

**Implicación para el sistema:** los eventos y cambios de estado de la lista deberán conservarse para permitir la revisión y auditoría del orden de atención y de las decisiones tomadas. El historial no se eliminará al cerrar la lista.

### 6.40 Centro de notificaciones internas

**Decisión confirmada:** el sistema contará con una bandeja central de notificaciones internas que incluirá:
- Contador de notificaciones sin leer.
- Estado de cada notificación: leída o no leída.
- Fecha y hora de generación.
- Historial de notificaciones anteriores.

**Eventos contemplados:** avisos de vacantes disponibles, omisiones de la lista de espera, cierres de listas y otros eventos académicos.

**Implicación para el sistema:** las notificaciones se conservarán para consulta desde la bandeja. Los avisos de vacantes y demás notificaciones internas no dependerán del correo electrónico en el alcance inicial.

**Pendiente de definición:** reglas de retención, filtros, categorías, prioridad, acciones disponibles desde cada aviso y permisos administrativos sobre las notificaciones.

### 6.41 Marcado de notificaciones como leídas

**Decisión confirmada:**
- Al abrir una notificación, esta cambiará automáticamente a estado **leída**.
- El usuario podrá marcar manualmente una notificación como leída.
- El usuario podrá marcar todas sus notificaciones como leídas.

**Implicación para el sistema:** las acciones actualizarán el estado de lectura y el contador de notificaciones sin leer de la bandeja.

### 6.42 Conservación y eliminación de notificaciones

**Decisión confirmada:** los usuarios no podrán eliminar definitivamente sus notificaciones. Estas permanecerán disponibles en su historial para consulta, incluso después de haber sido marcadas como leídas.

**Implicación para el sistema:** la interfaz de usuario no ofrecerá una acción para borrar definitivamente una notificación. Se conservarán los registros para mantener la trazabilidad de los avisos académicos.

**Restricción adicional aprobada:** ningún perfil, incluido el administrador, podrá eliminar definitivamente las notificaciones. La conservación de los registros aplica a todos los roles.

**Conservación e historial aprobados:** las notificaciones permanecerán disponibles sin fecha de vencimiento. El historial se presentará de forma sencilla y paginada; se podrán consultar notificaciones anteriores mediante filtros básicos por fecha o tipo.

### 6.43 Gestión de catálogos académicos

**Decisión confirmada:**
- El coordinador académico será responsable de registrar y actualizar las carreras, los planes de estudio y el catálogo de materias.
- Servicios Escolares utilizará esos catálogos para gestionar expedientes e inscripciones, pero no modificará su estructura académica.

**Implicación para permisos:** la administración de estos catálogos corresponde a coordinación académica. El detalle de permisos por acción y los alcances específicos por carrera se completarán en la matriz de acceso.

## 7. Estado del levantamiento de requerimientos

### 7.1 Decisiones funcionales confirmadas

Las decisiones aprobadas están desarrolladas en las secciones 6.4 a 6.43 y resumidas en el registro de decisiones:

- Instituto Tecnológico de Puebla (ITP) como institución de referencia.
- Cinco perfiles: administrador, Servicios Escolares, coordinador académico, profesor y alumno.
- Doce módulos funcionales iniciales.
- Diseño que soporte todas las carreras y planes de estudio del ITP, con datos de prueba limitados.
- Versiones históricas de planes; cada alumno conserva su carrera/plan correspondiente y solo una carrera activa a la vez.
- Periodos semestrales ordinarios (enero–junio y agosto–diciembre), más periodos extraordinarios/verano.
- Separación entre ingreso, reinscripción por periodo e inscripción a materias/grupos.
- Materias reutilizables en diferentes planes; cada plan tiene su estructura curricular.
- Una materia puede ofertarse en varios grupos; cada grupo tiene profesor, horario y lista propios.
- El alumno selecciona grupos; se valida cupo, choque de horario, prerrequisitos, carga académica y que no mantenga otra inscripción activa a la misma materia o a una materia equivalente en el periodo.
- Aulas con catálogo, horario y capacidad; el cupo del grupo no puede superar la capacidad del aula.
- Cambios y bajas durante una ventana; fuera de ella requieren autorización y trazabilidad.
- Lista de espera FIFO, con oferta interna y plazo de aceptación de 24 horas desde la generación de la notificación.
- Antes de ofrecer una vacante, el sistema revalidará que el siguiente alumno FIFO todavía cumpla los requisitos académicos de inscripción.
- Si un alumno no es elegible al revalidar, no recibirá la oferta; se registrará el motivo, conservará su posición en la lista y el sistema continuará con el siguiente alumno FIFO.
- El alumno omitido recibirá una notificación interna con el motivo y la confirmación de que conserva su posición en la lista de espera.
- Las listas de espera se cerrarán automáticamente al comenzar las clases de la materia o al terminar el periodo escolar, lo que ocurra primero; los alumnos restantes recibirán una notificación interna.
- El sistema conservará un historial auditable por alumno en las listas de espera: ingreso y posición FIFO, ofertas y respuestas, motivos de omisión, fecha y motivo de salida o cierre.
- El sistema tendrá una bandeja central de notificaciones internas con contador de pendientes, estado leído/no leído, fecha y hora, e historial; incluirá avisos de vacantes, omisiones, cierres de listas y otros eventos académicos.
- Al abrir una notificación se marcará automáticamente como leída; el usuario también podrá marcar una notificación individual o todas como leídas.
- Ningún perfil, incluido el administrador, podrá eliminar definitivamente las notificaciones; permanecerán en el historial para consulta y trazabilidad.
- Las notificaciones se conservarán sin fecha de vencimiento y el historial se mostrará de forma paginada, con filtros básicos por fecha o tipo.
- Para entrar a la lista de espera, el alumno debe cumplir prerrequisitos, no tener choques de horario, respetar el límite de créditos y no estar inscrito en la misma materia o una equivalente en ese periodo.
- Calificaciones por unidades configurables por materia, con borrador, validación, publicación y correcciones autorizadas.
- Kárdex acumulativo con carrera/plan, periodo, materia, créditos, grupo, calificación final, intento, estado, calificaciones por unidad publicadas e intentos anteriores.
- El coordinador académico administra carreras, planes de estudio y catálogo de materias; Servicios Escolares los utiliza sin modificar su estructura.
- El coordinador académico crea y configura los periodos escolares, incluidas sus fechas de inicio y fin; Servicios Escolares puede consultarlos para gestionar inscripciones.
- Servicios Escolares registra alumnos nuevos, actualiza expedientes y asigna carrera/plan; el administrador se limita a cuentas de acceso, roles y permisos.
- El número de control del alumno será obligatorio y único; no se permitirán expedientes duplicados con el mismo número.
- El número de empleado del profesor será obligatorio y único; no se permitirán expedientes duplicados con el mismo número.
- El coordinador académico abre los grupos de cada materia y asigna profesor, horario y cupo; Servicios Escolares consulta la oferta disponible para gestionar inscripciones.
- El coordinador académico puede modificar profesor, horario o cupo de grupos abiertos; si hay alumnos inscritos, el cambio se audita y se notifica a los afectados.
- Antes de guardar cambios de horario, el sistema validará conflictos con los alumnos inscritos y la disponibilidad del aula; bloqueará los cambios que generen conflictos.
- El coordinador académico podrá cancelar grupos; si hay alumnos inscritos, registrará el motivo y se notificará a los afectados para que seleccionen otro grupo disponible.
- El coordinador académico podrá fijar una fecha límite de inscripción por grupo; después de ella se bloquearán nuevas inscripciones y entradas a lista de espera, salvo autorización de Servicios Escolares o coordinación académica.
- Autenticación por perfil, recuperación sencilla, permisos por rol, auditoría y borrado lógico.

### 7.2 Requerimientos funcionales pendientes de detalle

1. **Parcialmente respondida:** el expediente del alumno incluirá número de control obligatorio y único, nombre(s), apellidos, correo institucional, teléfono opcional, fecha de ingreso y estado académico; la carrera/plan se manejarán mediante relaciones e historial. Servicios Escolares registra alumnos nuevos, actualiza sus expedientes y asigna carrera/plan. El administrador gestiona cuentas de acceso, roles y permisos. El expediente del profesor incluirá número de empleado obligatorio y único, nombre(s), apellidos, grado académico, correo institucional, teléfono opcional, fecha de ingreso, departamento y estado. Cada profesor pertenecerá a un solo departamento, que podrá tener varios profesores. Faltan campos de otros catálogos.
2. **Parcialmente respondida:** coordinación académica administra carreras, planes de estudio y catálogo de materias; Servicios Escolares utiliza estos catálogos sin modificar su estructura. Falta detallar permisos por acción (consultar, crear, modificar, autorizar y desactivar) para cada rol y módulo.
3. **Definida para el alcance inicial:** bandeja sencilla de notificaciones internas, contador de no leídas, estado leído/no leído, fecha y hora, historial paginado sin vencimiento y filtros básicos por fecha o tipo. Al abrir un aviso se marca como leído; el usuario podrá marcar una o todas como leídas. Ningún perfil, incluido el administrador, podrá eliminarlas definitivamente. No se prevén más funciones de notificaciones en el alcance inicial.
4. **Parcialmente respondida:** se aprobaron los requisitos para ingresar a la lista de espera y su revalidación antes de ofrecer una vacante, así como el tratamiento y aviso a alumnos no elegibles. Falta definir cómo se gestionan cambios/bajas mientras existe una oferta pendiente.
5. Definir duración y fechas de la ventana de cambios/bajas, así como qué tipos de movimiento requieren autorización.
6. Definir los atributos finales de horarios, modalidades y aulas; aclarar si la capacidad de aula incluye al profesor y cómo se representan sesiones remotas.
7. **Parcialmente respondida:** el kárdex mostrará carrera, plan, periodo, materia, créditos, grupo, calificación final, intento y estado; también calificaciones por unidad publicadas e intentos anteriores. Falta definir formato final, promedios, avance curricular y créditos acumulados según reglas del ITP. Las listas escolares mostrarán número de control, nombre, carrera, grupo y estado de inscripción; podrán exportarse a PDF y Excel. Falta definir permisos de consulta/descarga y filtros.
8. Definir flujo de corrección de calificaciones publicadas en casos excepcionales y plazos de autorización.
9. Verificar las reglas oficiales para recursamiento, número de oportunidades y efecto de cada intento en el historial/promedio.
10. Verificar la seriación, equivalencias, materias simultáneas, carga máxima/mínima y excepciones académicas.
11. **Respondida:** se permitirán grupos compartidos entre carreras/planes cuando la materia sea equivalente y el ITP permita esa oferta común. El grupo se registrará una sola vez y las inscripciones conservarán la carrera/plan del alumno.
12. **Respondida:** un profesor podrá impartir varias materias y estar asignado a varios grupos por periodo; cada grupo tendrá un profesor responsable. Falta definir si se permiten docentes auxiliares/sustitutos o más de un responsable.
13. Definir cómo se documenta y valida oficialmente la equivalencia de materias para grupos compartidos.

### 7.3 Validación institucional pendiente

- Confirmar el catálogo oficial vigente de carreras, niveles, modalidades, claves y planes del ITP.
- Verificar calendarios y fechas de periodos, inscripción/reinscripción y ventanas de movimientos.
- Verificar reglas del TecNM/ITP para seriación, créditos, bajas, repetición, regularización y equivalencias.
- Verificar lineamiento vigente de evaluación: escala, ponderaciones, fórmula, redondeo, mínimo aprobatorio y oportunidades.
- Confirmar las funciones que la aplicación del proyecto debe implementar frente a las que ya atienden plataformas institucionales existentes.

### 7.4 Infraestructura y desarrollo pendientes

1. ¿El profesor exige un motor o tecnología específica para el clúster?
2. ¿Qué función técnica tendrá el nodo 1: punto de acceso de la aplicación, coordinador operativo o algún rol distinto?
3. ¿Las tres máquinas virtuales estarán en una sola computadora o distribuidas entre las computadoras de los integrantes?
4. ¿Qué virtualizador utilizarán y cómo se comunicarán las VMs (LAN, VPN u otra red)?
5. ¿Qué lenguaje/framework prefiere o domina el equipo?
6. ¿Se evaluará también una aplicación web funcional o principalmente la base de datos distribuida?
7. ¿Cuál es la fecha de entrega y qué hitos intermedios existen?

No asumir reglas institucionales ni decisiones técnicas todavía pendientes. Registrar las respuestas y su fecha cuando el equipo las confirme.

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
| Administrador | Gestionar cuentas de acceso, roles y permisos; no gestionar expedientes académicos de alumnos. |
| Servicios Escolares | Registrar alumnos nuevos, actualizar expedientes y asignar carrera/plan; gestionar inscripciones y consultar catálogos, periodos y grupos disponibles sin modificar su estructura. |
| Coordinador académico | Gestionar carreras, planes de estudio, catálogo de materias, periodos escolares y apertura/configuración de grupos, asignando profesor, horario y cupo. |
| Profesor | Consultar listas y registrar calificaciones. |
| Alumno | Consultar materias, horarios, calificaciones y kárdex. |

Los permisos específicos (crear, consultar, modificar, autorizar y dar de baja) se detallarán por módulo en la matriz de acceso. No asumir permisos adicionales hasta aprobar esa matriz.

| ID | Decisión | Motivo / evidencia | Fecha | Estado |
|---|---|---|---|---|
| D-001 | Gemini será la IA principal; Claude revisará; ChatGPT asesorará; Antigravity apoyará la implementación. | Acuerdo inicial del equipo. | 2026-11-09 | Aprobada |
| D-002 | El equipo se compone de Yosef (nodo 1), Andrés (nodo 2) y Leonardo (nodo 3). | Información proporcionada por el equipo. | 2026-11-09 | Confirmada |
| D-003 | Motor de base de datos y topología del clúster. | Pendiente de evaluación y validación del profesor/equipo. | — | Pendiente |
| D-004 | El sistema estará basado en el funcionamiento académico del Instituto Tecnológico de Puebla (ITP). | Confirmación del usuario; reglas institucionales concretas por verificar. | 2026-11-09 | Aprobada |
| D-005 | Se utilizarán cinco perfiles: administrador, Servicios Escolares, coordinador académico, profesor y alumno. | Aprobación expresa del usuario. | 2026-11-09 | Aprobada |
| D-006 | El alcance funcional inicial incluirá 12 módulos: usuarios y roles, alumnos, profesores, departamentos y carreras, materias, periodos escolares, grupos, inscripciones, horarios, calificaciones, kárdex y listas escolares. | Aprobación expresa del usuario; flujos detallados pendientes. | 2026-11-09 | Aprobada |
| D-007 | El diseño abarcará todas las carreras del ITP y sus planes de estudio; se utilizará un conjunto limitado de datos de prueba. | Confirmación expresa del usuario; catálogo vigente por verificar. | 2026-11-09 | Aprobada |
| D-008 | Cada carrera podrá conservar varias versiones de planes de estudio y los alumnos permanecerán asociados al plan que les corresponda, incluidos planes anteriores no vigentes. | Aprobación expresa; cambios de plan/equivalencias por verificar. | 2026-11-09 | Aprobada |
| D-009 | El sistema manejará periodos semestrales ordinarios (enero–junio y agosto–diciembre) y permitirá periodos extraordinarios o de verano. | Aprobación expresa; fechas exactas por calendario del ITP. | 2026-11-09 | Aprobada |
| D-010 | Se distinguirá el ingreso del alumno a una carrera/plan, la reinscripción por periodo y la inscripción a materias/grupos. | Aprobación expresa; reglas académicas específicas por verificar. | 2026-11-09 | Aprobada |
| D-011 | Una materia podrá ofertarse en varios grupos por periodo; cada grupo tendrá profesor, horario y lista de alumnos propios. | Aprobación expresa del usuario. | 2026-11-09 | Aprobada |
| D-012 | Durante la reinscripción, el alumno seleccionará los grupos, materias y horarios disponibles; se bloquearán inscripciones cuando el grupo alcance su cupo. | Aprobación expresa; reglas académicas relacionadas pendientes. | 2026-11-09 | Aprobada |
| D-013 | El profesor registrará varias calificaciones por unidad y el sistema calculará la calificación final de la materia. | Aprobación expresa; fórmula sujeta a normativa e instrumentación didáctica. | 2026-11-09 | Aprobada |
| D-014 | El número de unidades y sus ponderaciones serán configurables por materia. | Aprobación expresa; fórmula y restricciones concretas por verificar. | 2026-11-09 | Aprobada |
| D-015 | El sistema validará automáticamente que los grupos seleccionados por un alumno no tengan horarios traslapados e impedirá confirmar selecciones conflictivas. | Aprobación expresa del usuario. | 2026-11-09 | Aprobada |
| D-016 | Los horarios registrarán día, inicio, fin, aula, edificio y modalidad; habrá catálogo de aulas y validación de conflictos de ocupación. | Aprobación expresa del usuario. | 2026-11-09 | Aprobada |
| D-017 | El cupo del grupo no podrá superar la capacidad del aula asignada; ambos valores se administrarán por separado. | Aprobación expresa; tratamiento de excepciones pendiente. | 2026-11-09 | Aprobada |
| D-018 | Los cambios de grupo y bajas podrán realizarse durante una ventana establecida; fuera de ella requerirán autorización de Servicios Escolares o coordinación académica. Se conservará trazabilidad, sin borrar físicamente el registro. | Aprobación expresa; fechas y reglas específicas por verificar. | 2026-11-09 | Aprobada |
| D-019 | Los grupos llenos admitirán lista de espera automática; al liberarse una vacante, se ofrecerá al siguiente alumno, quien deberá confirmar en un plazo establecido. | Aprobación expresa del usuario. | 2026-11-09 | Aprobada |
| D-020 | La lista de espera utilizará prioridad FIFO (First In, First Out), con base en fecha y hora de registro. | Aprobación expresa del usuario. | 2026-11-10 | Aprobada |
| D-021 | El alumno tendrá 24 horas para aceptar una vacante; si no responde, se ofrecerá al siguiente alumno FIFO. El plazo será configurable por periodo escolar. | Aprobación expresa del usuario. | 2026-11-10 | Aprobada |
| D-022 | Las ofertas de vacante se notificarán dentro del sistema; el correo electrónico queda fuera del alcance inicial para este aviso. | Selección expresa del usuario. | 2026-11-10 | Aprobada |
| D-023 | El plazo de 24 horas comienza cuando el sistema genera la notificación, sin depender de que el alumno la lea o inicie sesión. El aviso muestra materia, grupo, horario y fecha/hora límite. | Aprobación expresa del usuario. | 2026-11-10 | Aprobada |
| D-024 | Se implementará autenticación con nombre de usuario para administrador, número de empleado para personal/profesores y número de control para alumnos; las cuentas tendrán contraseña, roles/permisos y vinculación al registro correspondiente cuando aplique. Contraseñas almacenadas con hash; el administrador podrá crear/desactivar cuentas y los usuarios cambiar su contraseña. | Aprobación expresa del usuario. | 2026-11-10 | Aprobada |
| D-025 | La recuperación sencilla de contraseña será gestionada por el administrador mediante una contraseña temporal que el usuario deberá cambiar en el siguiente acceso; se auditará y no se usará correo externo en el alcance inicial. | Aprobación expresa del usuario. | 2026-11-10 | Aprobada |
| D-026 | Cada alumno tendrá una sola carrera activa a la vez. Los cambios de carrera o plan conservarán la asignación académica anterior como historial. | Aprobación expresa; autorización y equivalencias por verificar con el ITP. | 2026-11-10 | Aprobada |
| D-027 | Las materias no acreditadas podrán cursarse nuevamente conforme a reglas académicas; cada intento tendrá registro independiente y se conservarán calificaciones anteriores para mostrarlas en el kárdex. | Aprobación expresa; oportunidades y criterios oficiales por verificar. | 2026-11-10 | Aprobada |
| D-028 | Las calificaciones podrán guardarse como borrador; el profesor las validará y publicará; el alumno solo consultará las publicadas. Las correcciones posteriores deberán quedar auditadas. | Aprobación expresa del usuario. | 2026-11-10 | Aprobada |
| D-029 | Se utilizará borrado lógico/desactivación o cierre de movimientos para preservar la información histórica y de auditoría. Los registros académicos no se eliminarán físicamente como operación normal. | Aprobación expresa y alineación con la guía del proyecto. | 2026-11-10 | Aprobada |
| D-030 | El sistema registrará eventos de auditoría (inicio de sesión, altas, modificaciones con valores anterior/nuevo, bajas lógicas, calificaciones y autorizaciones). El administrador verá toda la auditoría; Servicios Escolares y coordinación solo la relacionada con sus funciones. | Aprobación expresa del usuario. | 2026-11-10 | Aprobada |
| D-031 | Las correcciones de calificaciones publicadas deberán ser solicitadas por el profesor con motivo y autorizadas por el coordinador académico. Se conservarán valores anterior y nuevo, motivo, solicitante, autorizador y fecha/hora. | Aprobación expresa del usuario. | 2026-11-10 | Aprobada |
| D-032 | Cada plan de estudios tendrá su estructura curricular; cada materia registrará clave, nombre, créditos, semestre recomendado y prerrequisitos cuando correspondan. Una misma materia podrá reutilizarse en varios planes sin duplicar su definición. | Aprobación expresa; catálogo y reglas curriculares por verificar con el ITP. | 2026-11-10 | Aprobada |
| D-033 | El sistema validará automáticamente los prerrequisitos; bloqueará la inscripción si no se cumplen y permitirá excepciones solo cuando la normativa lo permita, con autorización de coordinación y auditoría. | Aprobación expresa; tipos y reglas exactas por verificar con el ITP. | 2026-11-10 | Aprobada |
| D-034 | El sistema calculará los créditos seleccionados por el alumno y validará su carga contra el límite configurado para el periodo; bloqueará excesos y admitirá excepciones autorizadas cuando la normativa lo permita. | Aprobación expresa; máximos, mínimos y variantes por verificar con ITP/TecNM. | 2026-11-10 | Aprobada |\n| D-035 | Se permitirán grupos compartidos entre carreras/planes cuando la materia sea equivalente y el ITP autorice la oferta común. El grupo se registra una sola vez y las inscripciones conservan la carrera/plan del alumno. | Aprobación expresa del usuario. Criterios oficiales de equivalencia y autorización pendientes de verificación con el ITP. | 2026-11-10 | Aprobada |

| D-036 | El kárdex será acumulativo y mostrará carrera, plan, periodo, materia, créditos, grupo, calificación final, intento y estado; permitirá consultar calificaciones por unidad publicadas e intentos anteriores. | Aprobación expresa del usuario. Promedios, avance curricular y formato oficial quedan pendientes de verificación con el ITP. | 2026-11-10 | Aprobada |

| D-037 | Las listas escolares por grupo mostrarán número de control, nombre, carrera, grupo y estado de inscripción; los profesores podrán consultarlas y exportarlas a PDF y Excel. | Aprobación expresa del usuario. | 2026-11-10 | Aprobada |
| D-038 | El acceso a listas será por alcance: administrador y Servicios Escolares podrán consultar todas; coordinación solo las carreras que administra; profesores solo sus grupos; alumnos no verán listas completas. Las exportaciones respetarán los mismos permisos. | Aprobación expresa del usuario. Filtros y detalles del reporte quedan pendientes. | 2026-11-10 | Aprobada |

| D-039 | El expediente inicial del alumno incluirá número de control, nombre(s), apellidos, correo institucional, teléfono opcional, fecha de ingreso y estado académico. La carrera y el plan se manejarán por relaciones e historial, no como datos duplicados en la ficha. | Aprobación expresa del usuario; minimización de datos personales. | 2026-11-10 | Aprobada |

| D-040 | El expediente inicial del profesor incluirá número de empleado, nombre(s), apellidos, grado académico, correo institucional, teléfono opcional, fecha de ingreso, departamento y estado. El departamento se relacionará mediante llave foránea. | Aprobación expresa del usuario; se evita duplicar el nombre del departamento. | 2026-11-10 | Aprobada |

| D-041 | Cada profesor pertenecerá a un solo departamento; un departamento podrá tener varios profesores. La relación será uno a muchos mediante una FK en profesores. | Aprobación expresa del usuario; coincide con la estructura inicial del diagrama E-R. | 2026-11-10 | Aprobada |

| D-042 | Un profesor podrá impartir varias materias y estar asignado a varios grupos durante un periodo; cada grupo tendrá un profesor responsable. La asignación docente se realizará en el grupo/oferta del periodo. | Aprobación expresa del usuario. Docentes auxiliares, sustituciones y límites de carga quedan pendientes. | 2026-11-10 | Aprobada |
| D-043 | Un alumno no podrá tener dos inscripciones activas a la misma materia —ni a materias equivalentes según el catálogo aplicable— dentro del mismo periodo. Los cambios de grupo se harán mediante el procedimiento autorizado y conservarán la trazabilidad; la materia podrá cursarse de nuevo en periodos posteriores como otro intento. | Aprobación expresa del usuario. La validación concreta de equivalencias depende del catálogo académico que se confirme. | 2026-11-10 | Aprobada |
| D-044 | Para ingresar a la lista de espera de un grupo lleno, el alumno deberá cumplir los prerrequisitos, no tener choques de horario, no exceder el límite de créditos y no estar inscrito en la misma materia o una equivalente durante el periodo. | Aprobación expresa del usuario. La regla complementa la lista de espera FIFO y el plazo de 24 horas ya aprobados. | 2026-11-10 | Aprobada |
| D-045 | Antes de ofrecer una vacante, el sistema volverá a validar que el siguiente alumno de la lista FIFO todavía cumpla los requisitos académicos de inscripción. | Aprobación expresa del usuario. El tratamiento de los no elegibles se concreta en D-046. | 2026-11-10 | Aprobada |
| D-046 | Si un alumno deja de cumplir los requisitos al revalidar una vacante, no recibirá la oferta; el sistema registrará el motivo, conservará su posición en la lista de espera y continuará con el siguiente alumno FIFO. El alumno podrá ser revalidado al liberarse otra vacante. | Aprobación expresa del usuario. La notificación interna al alumno se concreta en D-047; el mecanismo para corregir su situación queda pendiente. | 2026-11-10 | Aprobada |
| D-047 | Cuando un alumno sea omitido por no cumplir los requisitos al revalidar una vacante, recibirá una notificación interna que indique el motivo y confirme que conserva su posición en la lista de espera. | Aprobación expresa del usuario. | 2026-11-10 | Aprobada |
| D-048 | La lista de espera de un grupo se cerrará automáticamente al comenzar el periodo de clases de la materia o al terminar el periodo escolar, lo que ocurra primero. Los alumnos que permanezcan en ella recibirán una notificación interna de cierre. | Aprobación expresa del usuario. | 2026-11-10 | Aprobada |
| D-049 | Se conservará un historial auditable de cada alumno que haya formado parte de una lista de espera: fecha y hora de ingreso, posición FIFO, ofertas y respuestas, motivos de omisión, fecha y motivo de salida o cierre. | Aprobación expresa del usuario. | 2026-11-10 | Aprobada |
| D-050 | El sistema contará con una bandeja central de notificaciones internas con contador de no leídas, estado leído/no leído, fecha y hora, e historial. Incluirá avisos de vacantes, omisiones, cierres de listas y otros eventos académicos. | Aprobación expresa del usuario. Retención, filtros, categorías, prioridades, acciones y permisos administrativos quedan pendientes. | 2026-11-10 | Aprobada |
| D-051 | Al abrir una notificación se marcará automáticamente como leída; el usuario podrá marcar manualmente una notificación individual o todas sus notificaciones como leídas. | Aprobación expresa del usuario. | 2026-11-10 | Aprobada |
| D-052 | Ningún usuario podrá eliminar definitivamente sus notificaciones; estas permanecerán en el historial para consulta y trazabilidad, incluso después de ser marcadas como leídas. | Aprobación expresa del usuario. La restricción incluye explícitamente al administrador según D-053. El plazo de conservación y archivo quedan pendientes. | 2026-11-10 | Aprobada |
| D-053 | Ningún perfil, incluido el administrador, podrá eliminar definitivamente las notificaciones. La restricción aplica a todos los roles para preservar la trazabilidad de los avisos académicos. | Aprobación expresa del usuario. | 2026-11-10 | Aprobada |
| D-054 | Las notificaciones se conservarán sin fecha de vencimiento. El historial será sencillo y paginado, con filtros básicos por fecha o tipo. | Aprobación expresa del usuario. Se mantiene un alcance simple, sin añadir funciones innecesarias de notificaciones. | 2026-11-10 | Aprobada |
| D-055 | El coordinador académico será responsable de registrar y actualizar carreras, planes de estudio y catálogo de materias. Servicios Escolares utilizará esos catálogos para gestionar expedientes e inscripciones, sin modificar su estructura académica. | Aprobación expresa del usuario. Los permisos detallados por acción y alcance se especificarán en la matriz de acceso. | 2026-11-10 | Aprobada |
| D-056 | El coordinador académico podrá crear y configurar los periodos escolares, incluyendo fechas de inicio y fin. Servicios Escolares podrá consultar los periodos para gestionar inscripciones, pero no modificar su configuración. | Aprobación expresa del usuario. Las fechas concretas de cada ciclo se verificarán con el calendario vigente del ITP. | 2026-11-10 | Aprobada |
| D-057 | El coordinador académico será responsable de abrir/crear los grupos de cada materia en un periodo escolar y asignarles profesor, horario y cupo. Servicios Escolares podrá consultar los grupos disponibles para gestionar las inscripciones, sin modificar su configuración. | Aprobación expresa del usuario. | 2026-11-10 | Aprobada |
| D-058 | El coordinador académico podrá modificar el profesor, horario o cupo de un grupo ya abierto. Si existen alumnos inscritos, cada modificación deberá quedar registrada en auditoría y notificarse a los alumnos afectados. | Aprobación expresa del usuario. | 2026-11-10 | Aprobada |
| D-059 | Antes de guardar un cambio de horario de un grupo, el sistema comprobará que no genere conflictos con los horarios de los alumnos inscritos ni con la disponibilidad del aula. Si detecta un conflicto, impedirá guardar el cambio. | Aprobación expresa del usuario. | 2026-11-10 | Aprobada |
| D-060 | El coordinador académico podrá cancelar un grupo. Si existen alumnos inscritos, deberá registrar el motivo de cancelación y el sistema notificará a los alumnos afectados para que puedan seleccionar otro grupo disponible. | Aprobación expresa del usuario. | 2026-11-10 | Aprobada |
| D-061 | El coordinador académico podrá establecer una fecha límite de inscripción para cada grupo. Después de esa fecha, el sistema impedirá nuevas inscripciones y nuevas entradas a la lista de espera, salvo autorización de Servicios Escolares o coordinación académica. | Aprobación expresa del usuario. Los detalles del procedimiento de excepción se definirán en la matriz de permisos y reglas de inscripción. | 2026-11-10 | Aprobada |
| D-062 | Servicios Escolares será responsable de registrar alumnos nuevos, actualizar sus expedientes y asignarles la carrera y el plan de estudios correspondientes. El administrador se encargará de las cuentas de acceso, roles y permisos, no de la gestión académica de los expedientes. | Aprobación expresa del usuario. | 2026-11-10 | Aprobada |
| D-063 | El número de control será obligatorio y único para cada alumno. El sistema impedirá registrar dos expedientes con el mismo número de control. | Aprobación expresa del usuario. | 2026-11-10 | Aprobada |
| D-064 | El número de empleado será obligatorio y único para cada profesor. El sistema impedirá registrar dos expedientes de profesor con el mismo número de empleado. | Aprobación expresa del usuario. | 2026-11-10 | Aprobada |

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

- [x] Confirmar el tipo y la institución de referencia: instituto / ITP.
- [x] Aprobar los cinco perfiles y los doce módulos funcionales iniciales.
- [x] Definir el alcance de carreras, planes versionados y materias compartidas.
- [x] Definir periodos semestrales y flujo de ingreso, reinscripción e inscripción a grupos.
- [x] Aprobar selección de grupos por el alumno, validación de cupo y horarios.
- [x] Aprobar aulas, capacidad, cambios/bajas y lista de espera FIFO con plazo de 24 horas y notificación interna.
- [x] Aprobar calificaciones por unidades configurables, publicación, corrección autorizada y conservación de intentos.
- [x] Aprobar autenticación, recuperación sencilla, borrado lógico y auditoría.
- [x] Aprobar validación de prerrequisitos y carga académica.
- [x] Aprobar kárdex acumulativo con información académica e intentos históricos.
- [x] Aprobar listas escolares con campos básicos y exportación a PDF/Excel.
- [x] Aprobar los campos iniciales del expediente del alumno y la regla de no duplicar carrera/plan.
- [x] Aprobar el número de control obligatorio y único para cada alumno.
- [x] Aprobar los campos iniciales del expediente del profesor y la relación del departamento mediante llave foránea.
- [x] Aprobar el número de empleado obligatorio y único para cada profesor.
- [x] Definir que cada profesor pertenecerá a un único departamento.
- [x] Aprobar que un profesor pueda impartir varias materias y grupos, con un profesor responsable por grupo.
- [x] Aprobar inscripción única por materia (considerando equivalencias) para cada alumno en un mismo periodo.
- [x] Aprobar los requisitos académicos para ingresar a la lista de espera de un grupo lleno.
- [x] Aprobar la revalidación de requisitos académicos antes de ofrecer una vacante de la lista de espera.
- [x] Definir el tratamiento de alumnos no elegibles al liberar una vacante: registrar motivo, conservar posición y continuar FIFO.
- [x] Aprobar la notificación interna al alumno omitido, indicando el motivo y que conserva su posición en la lista.
- [x] Aprobar el cierre automático de las listas de espera y la notificación interna a los alumnos restantes.
- [x] Aprobar el historial auditable de los alumnos en listas de espera y sus movimientos.
- [x] Aprobar el centro de notificaciones internas con contador, estado de lectura, fecha/hora e historial.
- [x] Aprobar el marcado automático al abrir y las opciones de marcar una o todas como leídas.
- [x] Aprobar que ningún perfil, incluido el administrador, pueda eliminar definitivamente las notificaciones.
- [x] Definir conservación sin vencimiento e historial sencillo y paginado con filtros básicos.
- [x] Aprobar permisos diferenciados para consulta y descarga de listas.
- [ ] Precisar los requerimientos funcionales que permanecen abiertos en la sección 7.2, incluida la matriz de permisos detallados y el procedimiento de excepciones a las fechas límite de inscripción.
- [x] Aprobar grupos compartidos entre carreras/planes cuando exista equivalencia y autorización del ITP.
- [ ] Verificar reglas y catálogos académicos vigentes del ITP/TecNM, incluidos los calendarios y fechas oficiales de los periodos.
- [ ] Responder preguntas de infraestructura y desarrollo de la sección 7.4.
- [ ] Revisar y aprobar el modelo E-R corregido.
- [ ] Elegir el motor y arquitectura del clúster con base en los requisitos del profesor.
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
