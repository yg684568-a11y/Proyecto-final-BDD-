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

## 7. Preguntas pendientes de requerimientos

Resolver antes de aprobar el modelo definitivo:

1. **Respondida:** el sistema será para el Instituto Tecnológico de Puebla (ITP) y se basará en su funcionamiento académico. Las reglas institucionales específicas aún deben recopilarse y verificarse.
2. **Respondida:** se aprueban cinco perfiles: administrador, Servicios Escolares, coordinador académico, profesor y alumno. Sus responsabilidades generales son las descritas en este documento; los permisos detallados por módulo se definirán durante el diseño funcional.
3. **Respondida:** se aprueban 12 módulos iniciales: usuarios y roles; alumnos; profesores; departamentos y carreras; materias; periodos escolares; grupos; inscripciones; horarios; calificaciones; kárdex; y listas escolares. Los flujos y reglas de negocio detallados quedan pendientes.
4. **Respondida:** el diseño abarcará todas las carreras del ITP y permitirá administrar sus planes de estudio; para pruebas se utilizará un conjunto reducido de datos. El catálogo activo y las claves/planes vigentes deberán verificarse antes de cargar datos.
5. **Respondida:** se conservarán varias versiones de planes por carrera, incluidos planes anteriores. Cada alumno quedará asociado al plan que le corresponda. Las reglas institucionales para cambios de plan y equivalencias se investigarán antes de implementarlas.
6. **Respondida:** se utilizará una estructura semestral de referencia: enero–junio y agosto–diciembre, permitiendo registrar también periodos extraordinarios o de verano. Las fechas exactas se configurarán por periodo y deberán verificarse con los calendarios vigentes del ITP.
7. **Respondida:** se separarán el ingreso del alumno a una carrera/plan, su reinscripción por periodo y la inscripción a materias/grupos. Después se registrarán calificaciones y se conservará el historial académico. Las reglas concretas de carga, seriación, créditos, bajas y regularización deben verificarse con el ITP.
4. ¿El profesor exige un motor o tecnología específica para el clúster?
5. ¿Qué función técnica tendrá el nodo 1: punto de acceso de la aplicación, coordinador operativo o algún rol distinto?
6. ¿Las tres VMs estarán en una sola computadora o distribuidas entre las computadoras de los integrantes?
7. ¿Qué virtualizador utilizarán y cómo se comunicarán las VMs (LAN, VPN u otra red)?
8. ¿Qué lenguaje/framework prefiere o domina el equipo?
9. ¿Se evaluará también una aplicación web funcional o principalmente la base de datos distribuida?
10. ¿Cuál es la fecha de entrega y qué hitos intermedios existen?

Registrar aquí las respuestas y su fecha cuando el equipo las confirme.

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
- [ ] Confirmar el catálogo activo de carreras, modalidad, nivel y planes vigentes para el ciclo de referencia.
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
