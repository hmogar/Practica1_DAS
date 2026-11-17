---
status: accepted
date: 2026-11-17
decision-makers: Daniel Marcos Serrano, Jaime Bergad Lario
consulted: Héctor Moreno García, Ivo Hagemann Martínez
informed: Raúl García Piedra
---

# Selección de la pasarela de pagos

## Context and Problem Statement

El requisito de dominio RD-2 exige que el sistema utilice Stripe como pasarela de pagos. Se registra la pasarela que utilizará el sistema de acuerdo con esta restricción.

## Decision Drivers

* Cumplir el requisito de dominio RD-2.
* Utilizar Stripe como pasarela de pagos, tal como exige el requisito.
## Considered Options

* Stripe, exigida por RD-2.

## Decision Outcome

Chosen option: "Stripe", because el requisito de dominio RD-2 establece que esta será la pasarela de pagos del sistema. Al ser una restricción explícita, no se consideran otras pasarelas como alternativas válidas.

### Consequences

* Good, because se cumple RD-2.
* Good, because se utiliza la pasarela de pagos especificada por el requisito.
* Bad, because el sistema depende de Stripe para procesar los pagos mientras RD-2 siga vigente.

### Confirmation

La decisión se confirma cuando el diseño del sistema identifica Stripe como su pasarela de pagos.

## Pros and Cons of the Options

### Stripe

* Good, because satisface directamente el requisito de dominio RD-2.
* Bad, because cambiar a otra pasarela requeriría modificar primero el requisito.

## More Information

Esta decisión está motivada únicamente por RD-2, que exige Stripe como pasarela de pagos.
