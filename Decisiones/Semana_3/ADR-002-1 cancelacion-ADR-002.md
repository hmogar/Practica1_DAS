---
status: accepted
date: 2026-11-17
decision-makers: Daniel Marcos Serrano, Jaime Bergad Lario
consulted: Héctor Moreno García, Ivo Hagemann Martínez
informed: Raúl García Piedra
---

# Cancelación de ADR-002

## Context and Problem Statement

ADR-002 decidía el patrón de acceso a datos y detallaba su organización mediante repositorios. Tras revisar el nivel de detalle con el profesor, se ha decidido cancelar ADR-002 porque esa decisión corresponde a un nivel de diseño demasiado bajo para esta entrega.

La selección del sistema gestor de bases de datos se trata por separado en ADR-002-2.

## Decision Drivers

* Mantener las decisiones de esta entrega en el nivel de diseño acordado con el profesor.
* Separar la selección del sistema gestor de bases de datos de los detalles de implementación del acceso a datos.

## Considered Options

* Mantener ADR-002 y la decisión sobre el patrón Repository.
* Cancelar ADR-002 y registrar la selección del sistema gestor en ADR-002-2.

## Decision Outcome

Chosen option: "Cancelar ADR-002 y registrar la selección del sistema gestor en ADR-002-2", because así se retira la decisión de bajo nivel sobre el patrón de acceso a datos y se mantiene la selección del gestor de bases de datos en su propio ADR.

ADR-002-2 recoge la decisión sobre el sistema gestor de bases de datos: PostgreSQL 19.

### Consequences

* Good, because se evita fijar en esta entrega un patrón de implementación para el acceso a datos.
* Good, because la decisión sobre el sistema gestor queda documentada por separado en ADR-002-2.
* Bad, because el patrón de acceso a datos queda pendiente para una etapa de diseño posterior.

### Confirmation

La decisión se confirma cuando ADR-002 queda identificado como cancelado y ADR-002-2 conserva la decisión sobre el sistema gestor de bases de datos.

## More Information

Esta cancelación se acordó con el profesor. No modifica la decisión sobre el sistema gestor documentada en ADR-002-2.
