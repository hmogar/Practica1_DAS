---
status: accepted
date: 2026-11-10
decision-makers: Daniel Marcos Serrano, Jaime Bergad Lario
consulted: Héctor Moreno García, Ivo Hagemann Martínez
informed: Raúl García Piedra
---

# ¿Qué estilo arquitectónico general adoptar para el sistema?

## Context and Problem Statement

El sistema corresponde a un periódico digital donde los lectores interactúan con la plataforma para gestionar suscripciones y los administradores/redactores acceden a estadísticas de uso. El requisito de dominio RD-1 establece de manera obligatoria que todas las peticiones deben realizarse mediante protocolos HTTP/REST. Se debe decidir qué estilo arquitectónico general debe adoptar el sistema para articular la comunicación y la separación entre la interfaz de usuario y el procesamiento de la información.

## Decision Drivers

* Cumplir con el requisito RD-1, que exige el uso de protocolos HTTP/REST para las comunicaciones.
* El front-end debe ser responsivo para PC y Tablet y el sistema debe discriminar el tipo de cliente, por lo que clientes distintos han de consumir un mismo servidor.
* Las interacciones principales (consultar noticias, suscribirse, consultar estadísticas) las inicia un usuario y esperan una respuesta inmediata.

## Considered Options

* Arquitectura Cliente-Servidor
* Arquitectura Dirigida por Eventos (EDA - Event-Driven Architecture)

## Decision Outcome

Chosen option: "Arquitectura Cliente-Servidor", because encaja de forma directa y natural con el modelo de peticiones y respuestas HTTP/REST impuesto por RD-1, ya que RD-1 impone que toda petición sea HTTP/REST y los clientes PC y Tablet se limitan a enviar peticiones a un único servidor que las atiende.

### Consequences

* Good, because da cumplimiento directo a RD-1 al estructurar la comunicación mediante peticiones HTTP/REST iniciadas por el cliente hacia el servidor.
* Good, because permite mantener una interfaz para PC y otra para Tablet sobre el mismo servidor REST, sin duplicar la lógica de negocio.
* Good, because no exige desplegar ningún broker de mensajería: basta un servidor que atienda peticiones HTTP/REST.
* Bad, because si el servidor cae, los lectores no pueden consultar noticias ni los redactores ver estadísticas, ya que todos los clientes dependen del mismo punto de entrada.
* Bad, because cada petición REST de un cliente PC o Tablet debe cruzar la red hasta el servidor, de modo que el tiempo de respuesta depende de la conexión de cada cliente.

### Confirmation

La decisión se confirma cuando el diagrama de arquitectura del sistema presente una separación nítida entre la capa de cliente (que formula las peticiones) y la capa de servidor (que las procesa y responde), conectadas a través de HTTP/REST (RD-1) en su comunicación.

## Pros and Cons of the Options

### Arquitectura Cliente-Servidor

* Good, because implementa de manera nativa la semántica de petición/respuesta de los protocolos HTTP/REST exigidos en RD-1.
* Good, because un único servidor expone los recursos del periódico (noticias, suscripciones, estadísticas) como endpoints REST, de modo que PC y Tablet usan la misma API.
* Good, because el despliegue se limita a un servidor, sin broker de mensajería ni varios servicios independientes que coordinar.
* Bad, because las suscripciones y bajas de los lectores y las estadísticas de administradores/redactores dependen del mismo servidor, por lo que una sobrecarga en una de estas funciones puede degradar también las demás.
* Bad, because requiere gestionar la concurrencia de peticiones y la latencia en las respuestas síncronas.

### Arquitectura Dirigida por Eventos (EDA - Event-Driven Architecture)

* Good, because el servidor que entrega una noticia no tendría que esperar a que otro componente registre la lectura, lo que evita ralentizar la respuesta al lector.
* Good, because la suscripción de los lectores a diferentes noticias (Anexo I) se corresponde con el modelo publicación-suscripción propio de EDA.
* Bad, because, aunque no es incompatible con HTTP/REST, RD-1 no contempla un broker y las interacciones principales siguen siendo petición/respuesta iniciadas por el usuario.
* Bad, because obliga a desplegar y operar un broker que ningún requisito contempla, en un sistema cuyas interacciones principales son peticiones iniciadas por el usuario.
* Bad, because tras suscribirse o darse de baja el lector espera ver el cambio en la siguiente petición, y con procesamiento asíncrono habría un intervalo en el que el estado aún no está actualizado.

## More Information

Esta decisión define el estilo arquitectónico global del sistema motivado por RD-1. Las decisiones sobre cómo estructurar internamente el backend o aislar servicios externos se abordan en sus decisiones correspondientes sin reutilizar este requisito.
