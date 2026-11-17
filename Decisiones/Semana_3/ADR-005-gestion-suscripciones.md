---
status: accepted
date: 2026-11-14
decision-makers: Daniel Marcos Serrano, Jaime Bergad Lario
consulted: Héctor Moreno García, Ivo Hagemann Martínez
informed: Raúl García Piedra
---

# ¿Dónde se ubican las reglas de alta y baja de las suscripciones anónimas?

## Context and Problem Statement

El sistema debe permitir crear suscripciones anónimas (RF-1) y permitir que los suscriptores se den de baja de forma anónima (RF-2). Se debe decidir si las reglas de estas operaciones se concentran en un servicio o se encapsulan en el modelo de dominio de la suscripción.

## Decision Drivers

* Representar las operaciones de alta y baja anónimas definidas en RF-1 y RF-2.
* Definir con claridad qué clase aplica las reglas de esas operaciones.
* Mantener explícita la coordinación entre las peticiones de la aplicación y la suscripción.

## Considered Options

* Servicio centrado en la lógica: un servicio gestiona las reglas de alta y baja; la suscripción actúa principalmente como estructura de datos.
* Modelo de dominio con reglas: `Suscripcion` encapsula las reglas de sus propias operaciones y un servicio de aplicación coordina el flujo.

## Decision Outcome

Chosen option: "Modelo de dominio con reglas", because permite que las operaciones de alta y baja definidas en RF-1 y RF-2 se expresen como comportamiento de la clase `Suscripcion`, mientras el servicio de aplicación coordina su ejecución.

### Consequences

* Good, because las reglas de alta y baja quedan asociadas a la clase que representa la suscripción.
* Good, because el servicio de aplicación puede coordinar las operaciones sin concentrar en sí mismo todas las reglas del ciclo de vida.
* Bad, because la clase `Suscripcion` debe incluir comportamiento además de sus datos.
* Bad, because hay que definir qué validaciones pertenecen a `Suscripcion` y cuáles corresponden al servicio de aplicación.

### Confirmation

La decisión se confirma cuando el diagrama muestra una clase `Suscripcion` con operaciones relacionadas con su alta y baja, y un servicio de aplicación que las coordina. Las operaciones deben permitir los flujos anónimos de RF-1 y RF-2.

## Pros and Cons of the Options

### Servicio centrado en la lógica

* Good, because los flujos de alta y baja se encuentran en un punto de coordinación explícito.
* Good, because la clase de suscripción puede mantenerse como una representación sencilla de los datos.
* Bad, because el servicio debe concentrar y mantener las reglas asociadas a las operaciones de suscripción anónima.

### Modelo de dominio con reglas

* Good, because `Suscripcion` puede expresar el comportamiento asociado a las altas y bajas anónimas requeridas.
* Good, because el servicio puede coordinar la operación sin ser el único lugar donde se definen las reglas de la suscripción.
* Bad, because hay que decidir qué comportamiento se asigna a la entidad y cuál al servicio de aplicación.

## More Information
