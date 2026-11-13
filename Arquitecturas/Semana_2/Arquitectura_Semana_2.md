
# Documento de Arquitectura Software – Semana 2
---

## 1. Base Arquitectónica Global 

El sistema mantiene como marco global el estilo acordado en la primera iteración:

* **Estilo Arquitectónico Global:** **Cliente-Servidor** (ver ADR-001).
* **Requisito Motivador:** **`RD-1`** (*Peticiones HTTP/REST*).
* **Fundamento:** 
  * Desacopla la interfaz de presentación (Front-end Web en el cliente) del procesamiento y persistencia centralizados en el servidor.
  * La comunicación cliente-servidor se realiza mediante protocolos estándar **HTTP/REST** sin estado (*stateless*), utilizando ciclos síncronos de petición (*request*) y respuesta (*response*).
  * Se descartó la alternativa Dirigida por Eventos (EDA) para evitar introducir la sobrecarga de intermediarios de mensajería (*brokers*) que no encajan con la semántica REST directa exigida.

---

## 2. Nuevos Patrones de Diseño del Backend

Durante la Semana 2 se refina la estructura interna del servidor mediante patrones orientados a objetos que aíslan las dependencias externas críticas sin alterar el estilo Cliente-Servidor:
### 2.1. Patrón Repository: Gestión y Aislamiento de Bases de Datos (ADR-002)

* **Requisitos Motivadores:**
  * **`RD-3` (SGBD PostgreSQL 19):** El backend debe emplear PostgreSQL 19 como motor de persistencia relacional.
  * **`RD-4` (Bases de datos separadas por perfil):** Exigencia estricta de mantener dos bases de datos físicas separadas: una exclusiva para administradores/redactores y otra para suscriptores.
* **Propósito y Funcionamiento:**
  * Media entre la capa de dominio/negocio y la capa de mapeo de datos, emulando una colección de objetos en memoria.
  * Se definen interfaces de repositorio en el dominio.
  * Las clases de implementación concretas encapsulan las consultas SQL y el acceso al driver de PostgreSQL 19 (`RD-3`), asociando cada repositorio al *DataSource* de la base de datos correspondiente (`RD-4`).
* **Ventajas frente a alternativas evaluadas:**
  * **Frente a Active Record:** Evita acoplar las entidades de negocio al esquema de tablas relacionales y permite gestionar limpiamente la conexión a dos bases de datos distintas sin contaminar el dominio.
  * **Frente a Acceso Directo SQL:** Evita dispersar sentencias SQL y manejo manual de conexiones por los servicios de negocio, asegurando el principio de responsabilidad única.
---
### 2.2. Patrón Adapter: Aislamiento de la Pasarela de Pago (ADR-003)

* **Requisito Motivador:**
  * **`RD-2` (Pasarela de pago STRIPE):** La pasarela de pago para gestionar cobros debe ser imperativamente STRIPE.
* **Propósito y Funcionamiento:**
  * Convierte la interfaz del SDK oficial de Stripe en una interfaz propia definida por nuestro sistema.
  * Una clase concreta `StripeAdapter` implementa dicha interfaz, encapsulando todas las dependencias, llamadas a la API externa de Stripe, conversión de parámetros y captura de excepciones.
* **Ventajas frente a alternativas evaluadas:**
  * **Frente al Uso Directo del SDK de Stripe:** Evita que el código de facturación dependa de librerías de terceros; si la API de Stripe cambia o se requiere simular cobros en tests unitarios, solo se modifica el adaptador.
  * **Frente a Puerto y Adaptador Hexagonal:** Proporciona el nivel exacto de desacoplamiento necesario sin introducir la sobrecarga de un estilo arquitectónico completo, manteniendo la coherencia con el estilo Cliente-Servidor.
