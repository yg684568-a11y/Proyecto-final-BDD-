# Revisión técnica y correcciones del modelo E-R

**Versión:** 1.0  
**Fecha:** 2026-11-09  
**Estado:** correcciones documentales aplicadas; modelo lógico pendiente de aprobación del equipo.  
**Alcance:** diagrama E-R, modelo relacional y matriz R01–R55.  
**Motor objetivo:** MariaDB con Galera Cluster.

## 1. Resultado general

Se revisaron las cuatro vistas Mermaid del E-R y se contrastaron sus relaciones con el modelo relacional y la matriz R01–R55. El modelo mantiene 36 entidades base y 55 relaciones documentadas. No se detectaron entidades aisladas ni relaciones de la matriz cuyos extremos estén ausentes en los diagramas.

Las correcciones se aplicaron en conjunto para conservar la correspondencia entre cardinalidades, atributos y llaves foráneas. Los diagramas mantienen el mismo formato Markdown/Mermaid y la organización de cuatro diagramas del repositorio.

## 2. Correcciones aplicadas

| N.º | Inconsistencia | Corrección |
|---|---|---|
| 1 | `alumno.semestre` no aparece en el expediente aprobado por D-039 y podía duplicar/representar de forma ambigua el avance del alumno. | Se eliminó de `alumno`. Se conserva `plan_materia.semestre_sugerido` como dato curricular, no como semestre actual del estudiante. |
| 2 | `departamento.edificio` no está respaldado por D-070 y mezcla la adscripción académica con la ubicación física. | Se eliminó de `departamento`. El edificio se conserva asociado a `aula`, conforme a la programación presencial de D-090. |
| 3 | `persona.correo_institucional` estaba definido NOT NULL para todas las personas, aunque D-065 solo confirma obligatoriedad para alumnos y profesores. | Se volvió nullable y UNIQUE en `persona`. La aplicación exige el dato para alumnos y profesores. El campo centralizado conserva la unicidad entre ambos perfiles. |
| 4 | La cardinalidad de oferta–notificación permitía que una oferta no tuviera notificación, aunque el proceso aprobado requiere avisar al alumno. | Mermaid ahora muestra que cada oferta tiene exactamente una notificación; una notificación puede ser general y no corresponder a ninguna oferta. El vínculo se aplica en la misma transacción. |
| 5 | Los movimientos de inscripción registraban grupos de origen/destino sin forzar que ambos ofrecieran la misma materia de plan que el intento inscrito. | Se agregó `id_plan_materia` a `movimiento_inscripcion` y FK compuestas hacia `inscripcion` y `grupo_plan_materia` para origen y destino. El destino es NULL únicamente para bajas. |
| 6 | El reconocimiento de una materia podía apuntar a una inscripción de otro alumno, aunque el texto decía que debía ser del mismo alumno. | Se agregó UNIQUE `(id_alumno,id_inscripcion)` en `inscripcion` y FK compuesta desde `reconocimiento_materia`. |
| 7 | El E-R exigía al menos una materia de plan y un horario para cada grupo, pero la matriz solo decía 1:N. | R29 y R31 ahora expresan cardinalidad mínima 1:N, igual que Mermaid. |
| 8 | Dos relaciones UNIQUE independientes no impedían que un empleado apareciera simultáneamente como profesor y personal. | Se mantuvo la regla de exclusividad de la primera versión y se documentó que debe validarse en la transacción de alta/cambio de subtipo. |

## 3. Estado de cardinalidades y relaciones

- Cuatro diagramas Mermaid conservados.
- 36 entidades base representadas.
- 55 relaciones R01–R55 en la matriz.
- Todas las relaciones de la matriz tienen sus entidades representadas en los diagramas.
- Ninguna entidad base queda aislada.
- Las relaciones opcionales conservan su semántica y nulabilidad documentadas.
- Las relaciones repetidas entre diagramas corresponden a actores transversales (cuenta, notificación y auditoría), no a entidades duplicadas.

## 4. Reglas que dependen de transacciones o de la aplicación

Estas reglas no quedan garantizadas únicamente por las cardinalidades del E-R:

1. Que cada oferta se cree junto con su notificación y no se confirme sin ella.
2. Que el correo institucional sea obligatorio para alumno/profesor, pero pueda quedar NULL para otros perfiles.
3. Que un empleado no se registre simultáneamente como profesor y personal.
4. Que el origen de un movimiento corresponda al grupo vigente al solicitarlo y que la secuencia de movimientos sea continua.
5. Que un movimiento de cambio conserve la misma materia de plan, cupo y elegibilidad.
6. Que los grupos no se traslapen en el aula y que el cupo no supere la capacidad de cada aula asignada.
7. Que una persona tenga solo una asignación vigente de plan y un profesor una adscripción vigente.
8. Que los permisos de cada usuario se restrinjan por su rol y ámbito de datos.

## 5. Decisiones que aún deben confirmarse

- **Múltiples roles por persona:** el modelo actual permite un rol por cuenta mediante `cuenta_usuario.id_rol`. D-005 enumera cinco perfiles, pero no determina si una persona puede acumular más de uno. Confirmar este comportamiento antes de DDL; si se requieren múltiples roles se deberá sustituir la FK simple por una tabla asociativa `cuenta_rol`.
- **Interpretación de módulos:** confirmar que los `Modulos` del diagrama original correspondan a unidades de evaluación.
- **Reglas académicas institucionales:** créditos, intentos, seriación, ponderaciones, promedio y equivalencias deben verificarse con el ITP/TecNM.

## 6. Validación para la siguiente fase

**Resultado:** el modelo está preparado para iniciar el diseño del DDL de MariaDB como siguiente etapa de trabajo, siempre que el equipo acepte las decisiones pendientes indicadas y mantenga las reglas transaccionales como parte del diseño físico y de la aplicación.

Esta revisión no constituye una prueba ejecutada contra MariaDB ni una aprobación institucional del modelo. Todavía no se ha generado ni ejecutado SQL.
