---
status: accepted
date: 2026-11-04
decision-makers: Daniel Marcos Serrano, Jaime Bergad Lario
consulted: Héctor Moreno García, Ivo Hagemann Martínez
informed: Raúl García Piedra
---

# ¿Cómo desplegar el backend?

## Context and Problem Statement

El sistema incluye funciones diferenciadas, entre ellas la gestión de suscripciones y control de acceso (RF-1, RF-2, RF-3), la gestión de pagos y facturación (RF-5, RF-6) y el panel de métricas y estadísticas (RF-7). Asimismo, se requiere una separación estricta de bases de datos según el contexto de usuario (RD-9: una base de datos para administradores/redactores y otra para suscriptores). Se debe decidir cómo organizar y desplegar el backend para mantener separadas sus responsabilidades y dar soporte a esta arquitectura.

## Decision Drivers

* Mantener separadas las responsabilidades de las distintas funciones del backend.
* Mantener sencillo el desarrollo, despliegue y mantenimiento.
* No añadir distribución entre servicios si no hay un requisito de despliegue o escalado independiente.

## Considered Options

* Microservicios
* Monolito Modular

## Decision Outcome

Chosen option: "Monolito Modular", because permite separar responsabilidades y simplifica su despliegue y operación. 
### Consequences

* Good, because conserva la separación funcional y permite desplegar el backend como una aplicación.
* Good, because permite gestionar el acceso a las dos bases de datos separadas (RD-9) mediante módulos independientes sin la sobrecarga de orquestar servicios distribuidos.
* Bad, because no permite escalar o desplegar cada módulo de forma independiente.
* Bad, because un fallo que afecte al proceso de la aplicación puede interrumpir varias funciones a la vez.

### Confirmation

La decisión se confirma cuando suscripciones, pagos y estadísticas están organizados en módulos con responsabilidades definidas, gestionan sus respectivos accesos a datos (RD-9) y se despliegan como una única aplicación.

## Pros and Cons of the Options

### Microservicios

* Good, because cada servicio puede desplegarse y escalarse de forma independiente.
* Good, because un fallo puede quedar limitado a un servicio si se controlan las dependencias.
* Bad, because requiere gestionar varios despliegues y la comunicación entre servicios.
* Bad , because añade complejidad operativa y de desarrollo.

### Monolito Modular

* Good, because permite mantener separadas las responsabilidades dentro de una aplicación y aislar el acceso a las dos bases de datos (RD-9).
* Good, because simplifica el despliegue y la comunicación de módulos.
* Bad, because los módulos comparten el proceso de ejecución y un fallo grave puede afectar a toda la aplicación.
* Bad , because no permite desplegar ni escalar cada módulo de forma independiente.


## More Information
