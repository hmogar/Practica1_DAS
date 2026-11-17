---
status: accepted
date: 2026-11-17
decision-makers: Daniel Marcos Serrano, Jaime Bergad Lario
consulted: Héctor Moreno García, Ivo Hagemann Martínez
informed: Raúl García Piedra
---

# ¿Cómo se organiza la responsabilidad de consultar estadísticas?

## Context and Problem Statement

El sistema debe permitir a redactores y administradores consultar las métricas de uso definidas en RF-5: noticias más leídas, tiempo de permanencia, tasas de rebote y conversión de lectores a suscriptores.

Se debe decidir qué clase asume la responsabilidad de ofrecer estas consultas y cómo colaboran con ella las clases que las solicitan o participan en los procesos relacionados con la lectura y las suscripciones. Esta decisión se limita a las responsabilidades y colaboraciones entre clases; no define cómo se transporta, almacena o recupera la información.

## Decision Drivers

* Cumplir el requisito funcional RF-5 y ofrecer sus métricas a los usuarios autorizados.
* Mantener en un lugar claro la responsabilidad de proporcionar las consultas estadísticas.
* Permitir que redactores y administradores soliciten las métricas sin duplicar esa responsabilidad.
* Representar colaboraciones comprensibles entre el módulo de estadísticas y las clases relacionadas con su consulta.

## Considered Options

* **Módulo de estadísticas común:** `ModuloEstadisticas` ofrece las consultas; `Admin` y `Redactor` solicitan las métricas que necesitan. Los procesos de lectura y de suscripción colaboran con el módulo cuando aportan información relevante para las métricas.
* **Módulos separados por ámbito:** un módulo ofrece estadísticas de lectura y otro ofrece estadísticas de suscripción; `Admin` y `Redactor` coordinan las consultas a ambos.
* **Responsabilidad distribuida:** las clases `Admin`, `Redactor` y `GestorSuscripciones` implementan cada una las consultas relacionadas con su ámbito.

## Decision Outcome

Propuesta: elegir **Módulo de estadísticas común**, porque concentra la responsabilidad de ofrecer las métricas de RF-5 y permite que `Admin` y `Redactor` colaboren con la misma clase sin duplicar consultas. Los procesos de lectura y de suscripción aportan al módulo la información pertinente para las métricas de uso y conversión, sin que esta decisión determine el mecanismo técnico de esa colaboración.

### Consequences

* Good, because `Admin` y `Redactor` consultan un módulo común y comparten la misma responsabilidad estadística.
* Good, because la lógica de consulta de las métricas no se reparte entre las clases de usuario ni entre las clases de suscripción.
* Good, because las colaboraciones necesarias para las métricas de lectura y conversión quedan representadas en el diseño.
* Bad, because `ModuloEstadisticas` concentra varias responsabilidades de consulta y podría crecer si se incorporan muchas métricas nuevas.
* Bad, because deben definirse los contratos de colaboración entre el módulo y las clases que aportan información relevante.

### Confirmation

La decisión se confirma cuando el diagrama de clases muestra:

* `ModuloEstadisticas` como responsable de ofrecer las consultas de RF-5;
* `Admin` y `Redactor` colaborando con `ModuloEstadisticas` para solicitar métricas;
* las clases de los procesos de lectura y suscripción colaborando con el módulo respecto a la información relevante para las métricas;
* la responsabilidad estadística concentrada en el módulo común, sin duplicarla en las clases solicitantes.

## Pros and Cons of the Options

### Módulo de estadísticas común

* Good, because reúne en una clase la responsabilidad de ofrecer las métricas solicitadas por distintos perfiles.
* Good, because evita que `Admin` y `Redactor` implementen por separado las mismas consultas.
* Good, because permite representar una colaboración común con los procesos relacionados con lectura y suscripción.
* Bad, because el módulo puede acumular demasiadas responsabilidades si el conjunto de métricas crece.
* Bad, because hay que acordar qué información aporta cada clase colaboradora.

### Módulos separados por ámbito

* Good, because cada módulo mantiene una responsabilidad más acotada, relacionada con lectura o suscripción.
* Good, because las consultas de cada ámbito pueden evolucionar con menor impacto sobre el otro.
* Bad, because `Admin` y `Redactor` deben coordinar consultas a más de un módulo para obtener el conjunto de métricas.
* Bad, because puede duplicarse la coordinación común de las consultas y la presentación de resultados.

### Responsabilidad distribuida

* Good, because cada clase implementa consultas cercanas a su propio ámbito funcional.
* Good, because no se añade un módulo común dedicado a estadísticas.
* Bad, because la responsabilidad de ofrecer las métricas queda repartida entre varias clases.
* Bad, because redactores y administradores podrían recibir resultados o comportamientos distintos para consultas equivalentes.
* Bad, because las métricas que combinan lectura y suscripción requieren coordinación entre clases con responsabilidades distintas.

## More Information

Este ADR decide la responsabilidad del módulo de estadísticas y sus colaboraciones a nivel de clases. Los detalles de comunicación técnica y de persistencia quedan fuera de su alcance.
