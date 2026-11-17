---
status: accepted
date: 2026-11-17
decision-makers: Daniel Marcos Serrano, Jaime Bergad Lario
consulted: Héctor Moreno García, Ivo Hagemann Martínez
informed: Raúl García Piedra
---

# Rechazo de ADR-003

## Context and Problem Statement

ADR-003 reúne en una sola decisión la elección de Stripe como pasarela, la forma de integrar su SDK, la ejecución de cobros recurrentes, los reintentos y la recepción de webhooks. Se ha decidido rechazar ADR-003 en su conjunto para separar la selección exigida por el requisito de dominio RD-2 de las decisiones de diseño e implementación de la integración.

La selección de Stripe queda registrada de forma independiente en ADR-003-2.

## Decision Drivers

* Separar la elección de la pasarela, establecida por RD-2, de las decisiones sobre cómo integrarla.
* Evitar mantener como una única decisión aspectos distintos de la integración y de los cobros recurrentes.
* Conservar la trazabilidad de la selección de Stripe en ADR-003-2.

## Considered Options

* Mantener ADR-003 como decisión aceptada sobre la integración con Stripe y los cobros recurrentes.
* Rechazar ADR-003 y documentar por separado la selección de la pasarela en ADR-003-2.

## Decision Outcome

Chosen option: "Rechazar ADR-003 y documentar por separado la selección de la pasarela en ADR-003-2", because ADR-003 combina la elección obligatoria de Stripe con decisiones de diseño sobre la integración, la recurrencia y los reintentos que deben mantenerse separadas.

ADR-003-2 recoge la decisión de utilizar Stripe como pasarela de pagos, tal como exige RD-2. El rechazo de ADR-003 no elimina los requisitos de cobro recurrente ni los reintentos; deja fuera de esta decisión cómo se implementarán.

### Consequences

* Good, because la selección de Stripe queda documentada de forma independiente y trazable en ADR-003-2.
* Good, because se evita presentar como una sola decisión la elección de proveedor y los detalles de integración y facturación.
* Bad, because la estrategia de integración, la gestión de los cobros recurrentes y los reintentos dejan de estar resueltos por ADR-003.

### Confirmation

La decisión se confirma cuando ADR-003 queda identificado como rechazado y ADR-003-2 mantiene la selección de Stripe por RD-2, sin atribuirle decisiones sobre la implementación de la integración o de los cobros recurrentes.

## More Information

ADR-003 se conserva como registro histórico de la decisión rechazada. Los requisitos funcionales de cobro recurrente y los reintentos siguen vigentes y deberán cumplirse mediante el diseño correspondiente.
