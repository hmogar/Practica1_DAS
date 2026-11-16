---
status: accepted
date: 2026-11-11
decision-makers: Daniel Marcos Serrano, Jaime Bergad Lario
consulted: Héctor Moreno García, Ivo Hagemann Martínez
informed: Raúl García Piedra
---

# ¿Cómo integrar Stripe y ejecutar los cobros recurrentes?

## Context and Problem Statement

El sistema debe utilizar Stripe como pasarela de pago (RD-2) y realizar cobros recurrentes cada 3, 6 o 12 meses (RF-4). El Anexo I, en el apartado «Módulo y pasarela de Pago Online», establece que, si falla una petición de pago, debe permitirse reintentarla hasta tres veces. Se debe decidir cómo integrar Stripe y quién programa los cobros recurrentes y sus reintentos.

ADR-006 define los periodos e importes de las modalidades mediante `CicloFacturacion`. Esta decisión utiliza esos datos para configurar o ejecutar los cobros; no cambia cómo se modelan las modalidades.

## Decision Drivers

* Cumplir RF-4 y el máximo de tres reintentos indicado en el Anexo I.
* Integrar Stripe conforme a RD-2.
* Mantener coherencia con las modalidades e importes definidos en ADR-006.
* Aislar las llamadas al SDK de Stripe de la lógica de la aplicación.
* Mantener actualizado el estado local de las suscripciones a partir del resultado de los pagos.

## Considered Options

* **Uso directo del SDK y planificador propio:** los servicios de facturación invocan el SDK de Stripe; el backend programa los cobros y controla los reintentos.
* **Adapter y planificador propio:** `StripeAdapter` encapsula el SDK; el backend programa los cobros y controla los reintentos a través de la interfaz de pagos.
* **Adapter y recurrencia gestionada por Stripe:** el backend configura una suscripción recurrente mediante `StripeAdapter`; Stripe ejecuta los cobros y reintentos, y notifica sus resultados al backend mediante webhooks.

## Decision Outcome

Chosen option: "Adapter y recurrencia gestionada por Stripe", because cumple RD-2, mantiene el SDK encapsulado y delega en Stripe la programación de cobros recurrentes y los reintentos. El backend configura el máximo de tres reintentos establecido en el Anexo I y procesa las notificaciones para actualizar el estado local.

Las operaciones salientes de Stripe pasan por la interfaz propia y `StripeAdapter`. El endpoint de webhooks recibe las notificaciones HTTP y las traduce a operaciones de la aplicación; no expone el SDK directamente al dominio.

### Consequences

* Good, because la lógica de facturación no depende directamente de los tipos ni de las llamadas del SDK de Stripe.
* Good, because Stripe gestiona los cobros recurrentes y la programación de reintentos configurada.
* Good, because el periodo y el importe se obtienen de `CicloFacturacion`, de acuerdo con ADR-006.
* Bad, because el backend debe recibir webhooks y mantener sincronizado el estado local de las suscripciones con el resultado comunicado por Stripe.
* Bad, because hay que configurar y comprobar que los reintentos de Stripe respetan el máximo de tres establecido en el Anexo I.
* Bad, because el sistema debe conservar y relacionar los identificadores de Stripe con las suscripciones locales.

### Confirmation

La decisión se confirma cuando el diseño muestra que:

* el servicio de facturación obtiene periodo e importe de `CicloFacturacion`;
* el backend crea la suscripción recurrente mediante la interfaz de pagos implementada por `StripeAdapter`;
* Stripe envía el resultado de los cobros a un endpoint webhook del backend;
* el manejador de notificaciones actualiza la suscripción mediante la lógica de aplicación y el repositorio correspondiente;
* la configuración de reintentos se limita a un máximo de tres reintentos por pago fallido, según el Anexo I.

## Pros and Cons of the Options

### Uso directo del SDK y planificador propio

* Good, because el backend controla cuándo iniciar cada cobro y cómo contar los reintentos del Anexo I.
* Good, because la implementación puede utilizar directamente todas las operaciones del SDK necesarias para solicitar un pago.
* Bad, because la lógica de facturación queda acoplada a Stripe y a los tipos de su SDK.
* Bad, because el backend debe implementar y mantener el planificador, el registro de intentos y la coordinación de cada cobro.

### Adapter y planificador propio

* Good, because las llamadas al SDK quedan encapsuladas y el backend mantiene el control de los vencimientos y reintentos.
* Good, because la implementación de Stripe puede sustituirse o simularse tras la interfaz de pagos.
* Bad, because el backend debe implementar y mantener el planificador y el control de hasta tres reintentos.
* Bad, because un resultado de pago que no llegue o no se registre correctamente requiere que el backend concilie el intento antes de repetirlo.

### Adapter y recurrencia gestionada por Stripe

* Good, because conserva el aislamiento del SDK mediante `StripeAdapter` y delega en Stripe la ejecución recurrente.
* Good, because Stripe permite configurar una programación personalizada de hasta tres reintentos y notifica los fallos de pago mediante webhook.
* Good, because el backend recibe los cambios de pago de forma notificada y puede reflejarlos en la suscripción local.
* Bad, because hay que implementar y mantener el endpoint webhook y la actualización del estado local.
* Bad, because la configuración de Stripe debe verificarse para que el número de reintentos cumpla el Anexo I.

## More Information

Las notificaciones webhook son peticiones HTTP de Stripe al backend y no sustituyen el estilo Cliente-Servidor de ADR-001 ni requieren introducir un broker de eventos.
