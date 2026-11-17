---
status: proposed
date: 2026-11-16
decision-makers: Daniel Marcos Serrano, Jaime Bergad Lario
consulted: Héctor Moreno García, Ivo Hagemann Martínez
informed: Raúl García Piedra
---

# ¿Cómo se registran y calculan las estadísticas de uso?

## Context and Problem Statement

El sistema debe ofrecer estadísticas para administradores y redactores (RF-5), incluyendo noticias más leídas (RF-5.1), tiempo de permanencia (RF-5.2), tasas de rebote (RF-5.3) y conversión de lectores a suscriptores (RF-5.4). Las visitas las realizan lectores que no están identificados, por lo que hay que decidir cómo registrar sus interacciones sin identificar a la persona, cómo agruparlas por visita y quién calcula las métricas a partir de ellas.

## Decision Drivers

* Obtener los datos necesarios para calcular todas las métricas definidas en RF-5.1 a RF-5.4.
* Poder agrupar las interacciones de una misma visita mediante un identificador anónimo, ya que permanencia, rebote y conversión se miden por visita y no por noticia.
* Mantener separada la responsabilidad de registrar interacciones de la de calcular y presentar métricas.
* Reflejar estas responsabilidades en clases comprensibles dentro del diseño.

## Considered Options

* Registrar una visita por lector con sus datos resumidos (inicio, fin, noticias leídas y si terminó en suscripción) y obtener las métricas de esos registros.
* Registrar eventos de uso y calcular las métricas bajo demanda mediante un servicio de estadísticas (`StatsGeneration`).
* Registrar eventos de uso y precalcular periódicamente las métricas en una tabla de resultados que consulta `StatsGeneration`.

## Decision Outcome

Propuesta: elegir "Registrar eventos de uso y calcular las métricas bajo demanda mediante un servicio de estadísticas", porque las métricas solicitadas se derivan de distintos tipos de interacción (lectura, salida y suscripción) y este enfoque conserva el detalle de cada una sin mantener registros que haya que actualizar ni tareas periódicas.

Cada interacción se guarda como un `EventoUso` con su tipo, la noticia afectada (si procede), una marca de tiempo y un identificador de visita anónimo. `EventoUsoRepository` guarda los eventos y `StatsGeneration` los consulta y calcula las métricas:

* RF-5.1: cuenta los eventos de lectura por noticia.
* RF-5.2: calcula el tiempo entre el primer y el último evento de cada visita.
* RF-5.3: cuenta las visitas con un único evento de lectura frente al total de visitas.
* RF-5.4: compara las visitas que incluyen un evento de suscripción con el total de visitas.

El repositorio de eventos se incluye en el contexto de administradores/redactores definido en ADR-002, ya que son los destinatarios de las estadísticas. De este modo, todos los datos necesarios para las métricas, incluido el evento de suscripción, quedan en un mismo almacén y no hay que cruzar datos entre contextos.

### Consequences

* Good, because los eventos registrados proporcionan los datos para calcular las noticias más leídas, la permanencia, el rebote y la conversión.
* Good, because el identificador de visita anónimo permite agrupar los eventos de una visita sin identificar al lector.
* Good, because los eventos solo se insertan y nunca se modifican, por lo que no hay que gestionar visitas abiertas ni actualizar registros.
* Good, because la responsabilidad de calcular las métricas queda en `StatsGeneration`, separada de los componentes que registran las interacciones.
* Bad, because hay que definir qué tipos de evento se registran y cómo se interpretan para cada métrica (por ejemplo, qué evento marca el final de una visita para medir la permanencia).
* Bad, because cada lectura de una noticia genera un registro y cada consulta de estadísticas recorre los eventos, por lo que el volumen de datos y el tiempo de respuesta crecen con el tráfico acumulado.
* Bad, because el diseño requiere clases adicionales para representar y almacenar eventos de uso.

### Confirmation

La decisión se confirma cuando el diagrama muestra la clase `EventoUso` (con tipo, noticia, marca de tiempo e identificador de visita), el repositorio `EventoUsoRepository` ubicado en el contexto de administradores/redactores, y la clase `StatsGeneration` con operaciones que obtienen esos registros y calculan cada métrica de RF-5.1 a RF-5.4.

## Pros and Cons of the Options

### Registro de visitas con datos resumidos

* Good, because cada visita ya contiene su inicio, su fin, las noticias leídas y si hubo suscripción, de modo que permanencia (RF-5.2), rebote (RF-5.3) y conversión (RF-5.4) se obtienen sin recorrer interacciones sueltas, y las noticias más leídas (RF-5.1) se cuentan sobre las noticias de cada visita.
* Good, because genera un registro por visita y no uno por interacción, lo que reduce el volumen de datos.
* Bad, because el registro de la visita debe actualizarse en cada interacción (hora de fin, noticias leídas, suscripción), y hay que decidir cuándo se da por cerrada una visita.
* Bad, because solo se conserva el resumen: si se redefine el rebote o la permanencia con otro criterio, los registros ya guardados pueden no tener el dato necesario.

### Registro de eventos y cálculo bajo demanda mediante un servicio de estadísticas

* Good, because los eventos de lectura, salida y suscripción permiten derivar las métricas requeridas en RF-5.1 a RF-5.4.
* Good, because los eventos solo se insertan, nunca se modifican, y no hay que gestionar visitas abiertas.
* Good, because `StatsGeneration` concentra las reglas de cálculo sin repartirlas entre las clases que procesan las interacciones.
* Good, because una métrica nueva o redefinida puede calcularse sobre los eventos ya guardados.
* Bad, because hay que acordar el modelo de evento y guardar los registros necesarios para cada métrica.
* Bad, because cada consulta de estadísticas recorre los eventos, por lo que el tiempo de respuesta crece.

### Registro de eventos y precálculo periódico de métricas

* Good, because las consultas de los administradores/redactores leen métricas ya calculadas.
* Good, because conserva los eventos, por lo que las métricas pueden recalcularse si cambia su definición.
* Bad, because requiere una tarea periódica y una clase para almacenar las métricas calculadas, además de las clases de evento y repositorio.
* Bad, because las estadísticas mostradas pueden estar desactualizadas hasta el siguiente cálculo.
* Bad, because se almacenan dos veces los datos: eventos y métricas derivadas.

## More Information

El ADR decide cómo registrar las interacciones de los lectores y cuándo se calculan las métricas de RF-5. Los eventos son registros persistidos que consulta `StatsGeneration`, no mensajes asíncronos entre componentes, por lo que no introducen un broker ni contradicen el estilo Cliente-Servidor de ADR-001.

Si más adelante se necesitaran consultas más rápidas, el precálculo periódico puede añadirse sin cambiar el modelo de eventos.
