---
status: proposed
date: 2026-11-11
decision-makers: Daniel Marcos Serrano, Jaime Bergad Lario
consulted: Héctor Moreno García, Ivo Hagemann Martínez
informed: Raúl García Piedra
---

# ¿Cómo manejar las bases de datos?

## Context and Problem Statement

El back-end del periódico digital debe almacenar y consultar su información utilizando el sistema gestor de bases de datos PostgreSQL 19 (RD-3) y mantener una separación estricta en dos bases de datos independientes según el perfil de usuario: una exclusiva para administradores y redactores, y otra separada para los suscriptores (RD-4). Se debe decidir qué patrón o estrategia de acceso a datos emplear en el servidor para organizar la persistencia, aislar la lógica de negocio de los detalles del motor SQL y dirigir cada operación a la base de datos que corresponde a su perfil.

## Decision Drivers

* El back-end debe incluir bases de datos PostgreSQL 19 (RD-3), por lo que el servidor necesita una estrategia concreta para acceder a ese SGBD.
* Garantizar y encapsular el acceso separado a las dos bases de datos por perfil: administradores/redactores frente a suscriptores (RD-4).
* El servidor debe atender tanto a los lectores, que se suscriben y se dan de baja de las noticias, como a los administradores/redactores, que consultan las estadísticas de uso.

## Considered Options

* Repository
* Active Record
* Acceso directo a PostgreSQL

## Decision Outcome

Chosen option: "Repository", because permite abstraer y aislar la lógica de negocio frente a PostgreSQL 19 (RD-3) mediante interfaces de repositorio dedicadas, y encapsular explícitamente en sus implementaciones la conexión independiente a cada una de las dos bases de datos por perfil exigidas en RD-4.

### Consequences

* Good, because separa las entidades y reglas de negocio del periódico de las sentencias SQL y detalles de conexión de PostgreSQL 19 (RD-3).
* Good, because permite crear repositorios diferenciados según el contexto (por ejemplo, repositorios dirigidos a la BD de suscriptores y otros dirigidos a la BD de administradores/redactores), asegurando por diseño la separación estricta de RD-4.
* Good, because la lógica de suscripciones de los lectores y la de estadísticas de uso de los administradores/redactores acceden a los datos solo a través de la interfaz de repositorio de su perfil, de modo que un cambio en la base de datos de un perfil no obliga a modificar el código del otro.
* Neutral, because exige definir explícitamente el mapeo entre los objetos de dominio y las tablas relacionales de PostgreSQL 19.
* Bad, because obliga a crear al menos un conjunto de repositorios para la base de datos de suscriptores y otro para la de administradores/redactores (RD-4), con más interfaces y clases que con Active Record o con llamadas SQL directas.

### Confirmation

La decisión se confirma cuando el diagrama de clases y paquetes del servidor muestre interfaces de repositorio diferenciadas para el contexto de suscriptores y el de administradores/redactores (RD-4), implementadas por clases concretas que encapsulen las consultas a PostgreSQL 19 (RD-3), manteniendo el dominio libre de sentencias SQL o dependencias del driver.

## Pros and Cons of the Options

### Repository

* Good, because concentra el acceso a PostgreSQL 19 (RD-3) en las implementaciones de los repositorios, mientras que con Active Record o con acceso directo ese acceso queda repartido por las entidades o por los servicios.
* Good, because encapsula claramente qué operaciones van dirigidas a la base de datos de suscriptores y cuáles a la de administradores/redactores (RD-4), evitando mezclar contextos.
* Good, because cada interfaz de repositorio expone solo las operaciones que necesita su perfil (por ejemplo, el alta y la baja de suscripciones en el de suscriptores), en lugar de dar acceso libre a toda la base de datos.
* Neutral, because requiere especificar de antemano el contrato de métodos de acceso a datos que necesita cada módulo.
* Bad, because una operación sencilla (por ejemplo, registrar una baja) requiere pasar por la interfaz y la implementación del repositorio en lugar de resolverse con una única consulta.

### Active Record

* Good, because operaciones sencillas como dar de alta o de baja una suscripción se codifican directamente sobre la entidad correspondiente.
* Good, because evita tener que crear interfaces de repositorio e implementaciones separadas.
* Neutral, because asocia directamente cada clase de negocio con una tabla del esquema relacional.
* Bad, because acopla fuertemente las reglas de negocio del periódico a la estructura de tablas de PostgreSQL 19 (RD-3), dificultando cambios independientes.
* Bad, because complica gestionar de forma transparente el enrutamiento hacia las dos bases de datos separadas por perfil (RD-4) dentro del propio código de las entidades de dominio.
* Bad, because las entidades del periódico (suscripciones, noticias) incluirían sentencias de acceso a PostgreSQL 19 (RD-3) en su propio código, mezclando en las mismas clases las reglas de negocio y la persistencia.

### Acceso directo a PostgreSQL

* Good, because otorga control directo y total sobre las consultas SQL nativas y su optimización en PostgreSQL 19 (RD-3), útil por ejemplo para las consultas de las métricas de uso (noticias más leídas, tiempo de permanencia, tasas de rebote y conversión de lectores a suscriptores).
* Good, because no hay que crear repositorios para ninguno de los dos perfiles (RD-4): los servicios consultan directamente la base de datos que necesitan.
* Neutral, because traslada la responsabilidad de construir sentencias y gestionar conexiones a los propios servicios que solicitan los datos.
* Bad, because mezcla la lógica de negocio con sentencias SQL y gestión de conexiones, rompiendo el principio de responsabilidad única en el modelo UML.
* Bad, because obliga a seleccionar manualmente en múltiples puntos del código cuál de las dos bases de datos se debe consultar, aumentando el riesgo de errores y código duplicado.
* Bad, because cualquier cambio en el esquema de una de las dos bases de datos obligaría a revisar todos los servicios que la consultan, repartidos por el código.


## More Information

Esta decisión se fundamenta exclusivamente en los requisitos **RD-3** (*SGBD PostgreSQL 19*) y **RD-4** (*Bases de datos separadas por perfil*).
