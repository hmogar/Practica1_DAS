---
status: proposed
date: 2026-11-16
decision-makers: Daniel Marcos Serrano, Jaime Bergad Lario
consulted: Héctor Moreno García, Ivo Hagemann Martínez
informed: Raúl García Piedra
---

# ¿Cómo se modelan las modalidades de facturación de las suscripciones?

## Context and Problem Statement

El sistema debe disponer de un módulo de pago online que gestione los ciclos de facturación de las suscripciones (RF-3). Según el Anexo I existen tres modalidades: trimestral (70 €), semestral (130 €) y anual (190 €), cada una con su periodo y su importe. Se debe decidir cómo se representan estas modalidades en el diseño y qué clase conoce el periodo y el importe de cada una.

## Decision Drivers

* Representar las tres modalidades de facturación definidas en RF-3, cada una con su periodo y su importe.
* Poder añadir o modificar una modalidad sin cambiar la clase `Suscripcion`.
* Mantener clara la responsabilidad de cada clase: la suscripción representa al suscriptor y su estado, y el ciclo de facturación representa las condiciones económicas.
* Evitar una estructura de clases más compleja de lo que necesitan las diferencias reales entre modalidades.

## Considered Options

* Herencia: `Suscripcion` como clase abstracta con una subclase por modalidad (`SuscripcionTrimestral`, `SuscripcionSemestral`, `SuscripcionAnual`).
* Composición con Strategy: `Suscripcion` tiene un `CicloFacturacion` (interfaz) implementado por una clase por modalidad.
* Enumerado `ModalidadFacturacion` con periodo e importe como atributos de cada valor.

## Decision Outcome

Propuesta: elegir "Composición con Strategy", porque separa las condiciones de cada modalidad de la clase `Suscripcion`, de modo que la modalidad se puede asignar o cambiar sin crear otro tipo de suscripción y una modalidad nueva solo requiere una nueva implementación de la interfaz.

`CicloFacturacion` define las operaciones para obtener el importe y calcular la fecha del siguiente periodo de facturación. `CicloTrimestral`, `CicloSemestral` y `CicloAnual` la implementan con sus valores (3, 6 y 12 meses; 70, 130 y 190 €). `Suscripcion` mantiene una referencia a su `CicloFacturacion` y le delega esos cálculos.

### Consequences

* Good, because cada modalidad queda encapsulada en una clase con su periodo e importe, y el módulo de facturación consulta siempre la misma interfaz.
* Good, because una nueva modalidad se añade creando una clase, sin modificar `Suscripcion` ni el código que usa `CicloFacturacion`.
* Good, because `Suscripcion` se mantiene como una única clase, sin multiplicarse por modalidad.
* Bad, because las tres implementaciones se diferencian solo en periodo e importe, por lo que hay clases con muy poco comportamiento propio.
* Bad, because hay que definir qué operaciones forman la interfaz `CicloFacturacion` y qué parte de la lógica de facturación pertenece a ella y cuál al módulo de pago.

### Confirmation

La decisión se confirma cuando el diagrama muestra la interfaz `CicloFacturacion`, tres clases que la implementan (una por modalidad) y la clase `Suscripcion` asociada a esa interfaz, sin subclases de `Suscripcion` por modalidad.

## Pros and Cons of the Options

### Herencia: una subclase de Suscripcion por modalidad

* Good, because cada modalidad es una clase con sus propios valores de periodo e importe, y el diagrama muestra de forma directa las tres modalidades.
* Good, because no necesita una clase adicional para el ciclo: la modalidad es el propio tipo de suscripción.
* Bad, because la modalidad queda fijada al tipo del objeto: si un suscriptor pasa de trimestral a anual habría que crear un objeto de otra clase en lugar de cambiar un dato.
* Bad, because las tres subclases se diferencian solo en periodo e importe, y la jerarquía multiplica las clases de `Suscripcion` por una característica que no cambia su identidad.
* Bad, because obliga a que `Suscripcion` mezcle en su jerarquía el estado del suscriptor y las condiciones de facturación.

### Composición con Strategy

* Good, because separa las condiciones de cada modalidad de la clase `Suscripcion` y permite cambiar la modalidad sustituyendo el ciclo asociado.
* Good, because añadir una modalidad no modifica `Suscripcion`.
* Bad, because introduce una interfaz y tres clases que solo se diferencian en sus valores.
* Bad, because hay que definir el contrato de `CicloFacturacion` antes de poder implementar las modalidades.

### Enumerado con periodo e importe

* Good, because es la opción con menos clases: una sola enumeración con tres valores y sus atributos.
* Good, because refleja con claridad que las modalidades se diferencian solo en datos.
* Bad, because cualquier comportamiento que dependa de la modalidad acabaría en condicionales sobre el enumerado dentro de otras clases, y añadir una modalidad obligaría a modificarlas.
* Bad, because ofrece poco detalle de diseño para el diagrama de clases, y no permite que una modalidad tenga reglas propias si el negocio las requiriera.

## More Information

Esta decisión se centra en cómo se representan las modalidades de facturación y no en cómo ni cuándo se ejecuta el cobro. `Suscripcion` es la clase de ADR-005, que no cambia por esta decisión: aquí solo se le asocia un `CicloFacturacion`.


