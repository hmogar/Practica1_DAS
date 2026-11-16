# Acta de Discusión de Decisiones de Diseño – Semana 2

**Asignatura:** Diseño y Arquitectura Software (DAS) – Universidad Rey Juan Carlos  
**Práctica:** Práctica 1: Captura y Representación de Decisiones de Diseño  
**Iteración:** Tarea 2 (Semana 2) – Resolución de los restantes problemas de diseño  
**Fecha de reunión:** 11 de noviembre de 2026  

---

## 1. Participantes y Roles

* **Arquitectos Software Senior (ASS):**
  * Daniel Marcos Serrano
  * Jaime Bergad Lario
* **Arquitectos Software Cognitivos (ASC):**
  * Héctor Moreno García
  * Ivo Hagemann Martínez
* **Arquitecto Software Junior (ASJ):**
  * Raúl García Piedra

## 2. Orden del Día

1. Definición del patrón de acceso a datos del servidor: ADR-002.
2. Definición de la integración con Stripe y de la gestión de los cobros recurrentes y sus reintentos: ADR-003.
3. Validación de la trazabilidad con los requisitos.
4. Instrucciones para el modelado UML.

## 3. Desarrollo de la Discusión

### 3.1. Debate sobre ADR-002: Manejo de las Bases de Datos

* **Propuesta inicial de los ASS:**
  * Adoptar el patrón **Repository** como estrategia de acceso a datos del servidor, con interfaces diferenciadas para las bases de datos de suscriptores y administradores/redactores.
  * Fundamentar la decisión en RD-3 (PostgreSQL 19) y RD-4 (bases de datos separadas por perfil).
  * Mantener las sentencias SQL y las conexiones dentro de las implementaciones concretas de los repositorios.

* **Cuestionamientos y alternativas planteados por los ASC:**
  * Los ASC descartan **Active Record** para este diseño porque mezclaría las reglas de negocio y la persistencia, además de complicar el acceso separado a las dos bases de datos.
  * Ivo pregunta por qué elegir Repository en lugar de acceder directamente a PostgreSQL desde los servicios, ya que ambas opciones podrían parecer similares.
  * Ivo pide que cada ventaja e inconveniente se justifique en el contexto del periódico y se relacione con RD-3 o RD-4.

* **Defensa y argumentación de los ASS:**
  * El acceso directo puede ser rápido de escribir, pero reparte consultas y selección de base de datos entre los servicios.
  * Repository concentra el acceso a PostgreSQL en clases concretas y permite dirigir las operaciones a la base de datos del perfil correspondiente.
  * Los ASS reconocen que el patrón añade interfaces y clases, y aceptan ese coste para mantener los accesos separados y evitar duplicar consultas y conexiones.
  * El acceso directo ofrece control sobre SQL, pero ese beneficio no compensa mezclar consultas y lógica de negocio en los servicios.

* **Conclusión y acuerdo:**
  * Los ASS contextualizan los pros y contras con RD-3, RD-4 y las funciones del periódico.
  * Los ASC aceptan la justificación y se descartan Active Record y el acceso directo a PostgreSQL.
  * Se acepta el patrón **Repository** para ADR-002.

### 3.2. Debate sobre ADR-003: Integración con Stripe y cobros recurrentes

* **Propuesta de los ASS:**
  * Mantener una interfaz propia de pago y una clase **StripeAdapter** que encapsule las llamadas al SDK de Stripe, según RD-2.
  * Ampliar el alcance de ADR-003 para decidir también quién programa los cobros recurrentes de RF-4 y quién controla los reintentos de pago indicados en el Anexo I.
  * Considerar los periodos e importes definidos por `CicloFacturacion` en ADR-006 como datos de configuración de los cobros.

* **Cuestionamientos y alternativas planteados por los ASC:**
  * Héctor pregunta si la alternativa «SDK de Stripe» significa usar directamente el SDK o si también lo usa Adapter. Se aclara que la alternativa es el **uso directo del SDK**, sin una interfaz propia intermedia.
  * Ivo pregunta si merece la pena añadir Adapter si Stripe es la única pasarela prevista. La discusión mantiene el foco en aislar las clases de negocio de las particularidades del SDK, no en cambiar de proveedor.
  * Al ampliarse la decisión, los ASC piden comparar quién gestiona los vencimientos y reintentos: un planificador del backend o la recurrencia configurada en Stripe.
  * También se pide explicar cómo las notificaciones de Stripe actualizan la suscripción local y cómo se respeta el máximo de tres reintentos del Anexo I.

* **Defensa y argumentación de los ASS:**
  * Adapter encapsula el SDK y concentra en `StripeAdapter` la traducción entre las operaciones internas y las llamadas a Stripe. Que Stripe sea el único proveedor no elimina la dependencia de su API y SDK.
  * Se comparan tres alternativas completas: uso directo del SDK con planificador propio; Adapter con planificador propio; y Adapter con recurrencia y reintentos gestionados por Stripe.
  * Los periodos e importes siguen procediendo de `CicloFacturacion`, definido en ADR-006. La decisión sobre quién ejecuta los cobros no cambia cómo se modelan las modalidades.
  * La opción elegida delega la recurrencia y la programación de hasta tres reintentos en Stripe. El backend recibe las notificaciones webhook y actualiza el estado local mediante la lógica de aplicación y el repositorio correspondientes.
  * El webhook es una petición HTTP de Stripe al backend. No sustituye el estilo Cliente-Servidor de ADR-001 ni requiere introducir un broker de mensajería.
  * Los ASS reconocen como coste la recepción y validación de notificaciones y la sincronización del estado local con Stripe.

* **Conclusión y acuerdo actualizado (16 de noviembre de 2026):**
  * Se mantiene **Adapter** como frontera entre el sistema y el SDK de Stripe.
  * Se acepta que Stripe gestione la recurrencia y los reintentos, configurando el límite de hasta tres reintentos indicado en el Anexo I.
  * El backend procesará las notificaciones de Stripe y actualizará la suscripción local.
  * La decisión consolidada queda registrada en ADR-003 con estado **accepted**. ADR-007 se elimina porque su contenido queda incorporado en ADR-003.

## 4. Trazabilidad de Requisitos

* **ADR-001:** RD-1 (peticiones HTTP/REST).
* **ADR-002:** RD-3 (PostgreSQL 19) y RD-4 (bases de datos separadas por perfil).
* **ADR-003:** RD-2 (Stripe), RF-4 (cobros recurrentes) y el límite de reintentos del Anexo I, en el apartado «Módulo y pasarela de Pago Online».
* Los requisitos funcionales restantes se reservan para las decisiones de refinamiento y diseño detallado de las siguientes semanas.

---

**Firmas de conformidad:**

* *Daniel Marcos Serrano* (ASS)
* *Jaime Bergad Lario* (ASS)
* *Héctor Moreno García* (ASC)
* *Ivo Hagemann Martínez* (ASC)
* *Raúl García Piedra* (ASJ)
