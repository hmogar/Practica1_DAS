---
status: accepted
date: 2026-11-04
decision-makers: Daniel Marcos Serrano, Jaime Bergad Lario
consulted: Héctor Moreno García, Ivo Hagemann Martínez
informed: Raúl García Piedra
---

# ¿Cómo se estructura internamente el backend y como se aíslan las dependencias externas?

## Context and Problem Statement

El sistema debe comunicarse con los clientes exclusivamente mediante protocolos HTTP/REST (RD-3), utilizar Stripe como pasarela de pago obligatoria (RD-6) gestionando una política de hasta 3 reintentos de cobro automáticos en caso de fallo (RD-7), utilizar PostgreSQL 19 como SGBD (RD-8) y mantener bases de datos completamente separadas para administradores/redactores y suscriptores (RD-9). Se debe decidir cómo organizar internamente el backend para aislar tanto los mecanismos de transporte como las dependencias externas de la lógica de negocio, preservando la separación de datos requerida.

## Decision Drivers

* Desacoplar la lógica de negocio de los mecanismos de entrega (API HTTP/REST) y de persistencia/pasarela externa.
* Mantener la lógica de negocio independiente de tecnologías concretas como PostgreSQL y Stripe.
* Encapsular la lógica de comunicación y reintentos de cobro (RD-7) en el adaptador de pasarela de pago.
* Encapsular el acceso a las dos bases de datos para respetar su separación estricta por contexto (RD-9).
* Permitir probar la lógica de negocio sin depender directamente de los servicios externos (Stripe, PostgreSQL) ni del transporte HTTP.
* Mantener una estructura comprensible para el sistema.

## Considered Options

* Arquitectura Hexagonal
* Arquitectura en Capas

## Decision Outcome

Chosen option: "Arquitectura Hexagonal", because permite aislar la lógica de negocio tanto de los mecanismos de entrada (controladores HTTP/REST según RD-3) como de las dependencias externas de salida (PostgreSQL 19 y Stripe) mediante puertos (interfaces) y adaptadores.

### Consequences

* Good, because los endpoints HTTP/REST (RD-3) actúan como adaptadores primarios que delegan en los casos de uso del dominio a través de puertos de entrada.
* Good, because los cambios en PostgreSQL (RD-8, RD-9) o Stripe (RD-6, RD-7) quedan confinados a sus adaptadores secundarios sin afectar al núcleo de negocio.
* Good, because la lógica de negocio puede probarse de forma unitaria sin conectarse a la red, a Stripe ni a las bases de datos.
* Bad, because hay que diseñar y mantener los puertos e adaptadores, lo que añade mayor estructura y trabajo inicial.

### Confirmation

La decisión se confirma cuando la lógica de negocio depende exclusivamente de interfaces (puertos), los endpoints HTTP/REST (RD-3) se implementan como adaptadores primarios, la integración con Stripe (RD-6, RD-7) y PostgreSQL (RD-8, RD-9) se realiza a través de adaptadores secundarios, y el núcleo de dominio no importa directamente bibliotecas de transporte o de proveedores externos.

## Pros and Cons of the Options

### Arquitectura Hexagonal

* Good, because desacopla los controladores HTTP/REST (RD-3) y los adaptadores de persistencia y pagos (RD-6, RD-7, RD-8, RD-9) de las reglas de negocio.
* Good, because permite probar la lógica de negocio sustituyendo los adaptadores externos por versiones de prueba.
* Bad, because exige definir interfaces y adaptadores para cada interacción externa, lo que añade estructura y trabajo inicial.

### Arquitectura en Capas

* Good, because organiza el backend en niveles reconocibles, como presentación, lógica de negocio y acceso a datos.
* Good, because puede ser más directa de implementar cuando la aplicación tiene una estructura sencilla.
* Bad, because si las dependencias entre capas no se controlan, la lógica de negocio puede quedar ligada a controladores HTTP, a PostgreSQL o a Stripe.


## More Information