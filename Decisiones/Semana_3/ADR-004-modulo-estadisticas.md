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

Se debe decidir qué clase asume la responsabilidad de ofrecer estas consultas.

## Decision Drivers

* Cumplir el requisito funcional RF-5 y ofrecer sus métricas a los usuarios autorizados.
* Mantener en un lugar claro la responsabilidad de proporcionar las consultas estadísticas.
* Permitir que redactores y administradores soliciten las métricas sin duplicar esa responsabilidad.
* Representar de forma comprensible cómo se accede a las consultas estadísticas desde el controlador.

## Considered Options

* **Módulo de estadísticas común**: `ModuloEstadisticas` ofrece todas las consultas; los usuarios con privilegios (administradores y redactores) las solicitan a través de ControladorHTTP.
* **Módulos separados por ámbito:** un módulo ofrece estadísticas de lectura y otro ofrece estadísticas de suscripción; `Admin` y `Redactor` coordinan las consultas a ambos.
* **Responsabilidad distribuida:** las clases `Admin`, `Redactor` y `GestorSuscripciones` implementan cada una las consultas relacionadas con su ámbito.

## Decision Outcome

Chosen option: "Módulo de estadísticas común", because concentra la responsabilidad de ofrecer las métricas de RF-5 y permite que todos los usuarios con privilegios consulten la misma clase sin duplicar consultas.

### Consequences

* Good, because los usuarios con privilegios consultan un módulo común y comparten la misma responsabilidad estadística.
* Good, because la lógica de consulta de las métricas no se reparte entre otras clases.
* Bad, because `ModuloEstadisticas` concentra varias responsabilidades de consulta y podría crecer si se incorporan muchas métricas nuevas.

### Confirmation

La decisión se confirma cuando el diagrama de clases muestra:

* ModuloEstadisticas como responsable de ofrecer las consultas de RF-5;
* ControladorHTTP encaminando las peticiones de métricas hacia ModuloEstadisticas;
* la responsabilidad estadística concentrada en el módulo común, sin duplicarla en otras clases

## Pros and Cons of the Options

### Módulo de estadísticas común

* Good, because reúne en una clase la responsabilidad de ofrecer las métricas solicitadas por distintos perfiles.
* Good, because evita que cada tipo de usuario implemente por separado las mismas consultas.
* Bad, because el módulo puede acumular demasiadas responsabilidades si el conjunto de métricas crece.

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
