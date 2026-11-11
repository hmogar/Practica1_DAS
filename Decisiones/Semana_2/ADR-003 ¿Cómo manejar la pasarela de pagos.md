---
status: proposed
date: 2026-11-11
decision-makers: Daniel Marcos Serrano, Jaime Bergad Lario
consulted: Héctor Moreno García, Ivo Hagemann Martínez
informed: Raúl García Piedra
---

# ¿Cómo manejar la pasarela de pagos?

## Context and Problem Statement

El sistema requiere integrar obligatoriamente Stripe como pasarela de pago online para procesar las transacciones económicas del periódico digital (RD-2). La API y el SDK oficial de Stripe presentan interfaces, tipos de datos y modelos de respuesta propios de este proveedor externo. Se debe decidir cómo integrar y encapsular la comunicación con Stripe dentro del servidor para evitar que la lógica de negocio quede acoplada a las particularidades técnicas de la pasarela externa.

## Decision Drivers

* El módulo de pago online del periódico, que gestiona los ciclos de facturación, debe cobrar mediante Stripe (RD-2), un proveedor externo con API y SDK propios.
* Cada cobro del módulo de pago online depende de un servicio de un tercero (Stripe) que el sistema no controla.
* Stripe es la única pasarela de pago prevista (RD-2), por lo que hay que valorar si compensa añadir una capa intermedia para un único proveedor con el fin, de reducir la complejidad estructural innecesaria.

## Considered Options

* Adapter
* Uso directo del SDK de Stripe

## Decision Outcome

Chosen option: "Adapter", because permite envolver el SDK de Stripe (RD-2) tras una interfaz propia definida por el sistema, de modo que las clases del módulo de pago online no dependen directamente del SDK de Stripe.

### Consequences

* Good, because la lógica de negocio interactúa únicamente con una interfaz genérica de pasarela de pago, quedando aislada de la implementación concreta de Stripe (RD-2).
* Good, because cualquier cambio en la API o el SDK de Stripe (RD-2) queda confinado en la clase adaptadora, sin modificar las clases del módulo de pago online que la utilizan.
* Good, because el adaptador concentra en una sola clase la comunicación con Stripe, un servicio externo, en lugar de repartirla por las clases del módulo de pago online.
* Good, because la integración con Stripe queda dentro del servidor definido en la decisión de estilo Cliente-Servidor (ADR-001), como una interfaz y una clase más de ese servidor.
* Bad, because obliga a definir y mantener una interfaz adicional y una clase adaptadora encargada de traducir peticiones, respuestas y errores entre nuestro modelo y el SDK de Stripe.

### Confirmation

La decisión se confirma cuando el diagrama de clases UML del backend incluya una interfaz propia para la pasarela de pago utilizada por los servicios de negocio y una clase concreta StripeAdapter que implemente dicha interfaz y encapsule todas las llamadas al SDK de Stripe (RD-2).

## Pros and Cons of the Options

### Adapter

* Good, because desacopla la lógica de negocio del SDK de Stripe (RD-2) mediante una interfaz estable bajo control del equipo.
* Good, because es un patrón estructural que solo afecta a la clase que se comunica con Stripe (RD-2) y no obliga a cambiar el estilo Cliente-Servidor (ADR-001) ni el patrón Repository elegido para las bases de datos (ADR-002).
* Good, because la interfaz queda definida por el propio sistema, que decide qué operaciones de pago expone (por ejemplo, cobrar una suscripción) en lugar de heredar las del SDK de Stripe.
* Neutral, because requiere mapear los datos y excepciones del dominio hacia las estructuras requeridas por Stripe.
* Bad, because introduce código traductor adicional y puede ocultar funcionalidades muy específicas del SDK de Stripe si la interfaz es demasiado genérica.

### Uso directo del SDK de Stripe

* Good, because permite utilizar de forma inmediata la librería oficial de Stripe (RD-2) sin escribir interfaces ni clases envoltorio adicionales.
* Good, because, al ser Stripe la única pasarela prevista (RD-2), el módulo de pago online puede usar todas las funciones y tipos nativos de Stripe sin limitaciones.
* Neutral, because la estructura de clases del servicio de pagos queda condicionada por los objetos de respuesta de Stripe.
* Bad, because acopla directamente las clases de negocio del periódico a la librería de un tercero; cualquier cambio en la API de Stripe obliga a modificar la lógica interna.
* Bad, because las llamadas a Stripe, un servicio externo, quedarían repartidas por las distintas clases que cobran suscripciones, en lugar de concentradas en un único punto.


## More Information

Esta decisión está motivada exclusivamente por el requisito de dominio **RD-2** *Pasarela de pago STRIPE*.
