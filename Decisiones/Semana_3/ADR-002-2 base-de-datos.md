---
status: proposed
date: 2026-11-17
decision-makers: Daniel Marcos Serrano, Jaime Bergad Lario
consulted: Héctor Moreno García, Ivo Hagemann Martínez
informed: Raúl García Piedra
---

# ¿Qué sistema gestor de bases de datos se utilizará?

## Context and Problem Statement

El requisito de dominio RD-3 exige que el sistema utilice PostgreSQL 19. Se debe registrar el sistema gestor de bases de datos que utilizará la aplicación, respetando esta restricción.

## Decision Drivers

* Cumplir el requisito de dominio RD-3.

## Considered Options

* PostgreSQL 19, exigido por RD-3.

## Decision Outcome

Chosen option: "PostgreSQL 19", because es el sistema gestor especificado por el requisito de dominio RD-3. Como el requisito lo exige, no se consideran otros sistemas gestores como alternativas válidas.

### Consequences

* Good, because se cumple RD-3.
* Good, because el sistema utiliza la tecnología de base de datos establecida por el requisito.
* Bad, because no se puede elegir otro sistema gestor mientras RD-3 siga vigente.

### Confirmation

La decisión se confirma cuando el diseño y la configuración de persistencia identifican PostgreSQL 19 como el sistema gestor de bases de datos.

## Pros and Cons of the Options

### PostgreSQL 19

* Good, because satisface directamente el requisito de dominio RD-3.
* Bad, because cambiar a otro sistema gestor requeriría modificar primero el requisito.

## More Information


