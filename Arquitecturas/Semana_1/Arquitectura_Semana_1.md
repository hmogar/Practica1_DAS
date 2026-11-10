
#  Documento de Arquitectura Software – Semana 1

---

## 1.  Estilo Arquitectónico Seleccionado: Arquitectura Cliente-Servidor

### 1.1. Requisito Motivador
* **RD-1 (Peticiones HTTP/REST):** El sistema establece expresamente que *«Las peticiones se realizan mediante protocolos HTTP/REST»*.

### 1.2. Justificación Arquitectónica
* **Semántica petición-respuesta:** El protocolo HTTP/REST funciona de manera síncrona mediante ciclos de petición (*request*) y respuesta (*response*), donde los clientes inician la comunicación y el servidor procesa y devuelve los resultados. El estilo Cliente-Servidor es el que da soporte nativo y directo a este requisito.
* **Desacoplamiento de la interfaz:** Permite que la presentación (clientes web) evolucione, se rediseñe o se adapte de forma totalmente independiente de la lógica de procesamiento y almacenamiento del backend.
* **Centralización de datos y seguridad:** Permite centralizar en el servidor el control de acceso, la gestión de suscripciones anónimas, la facturación periódica y la persistencia de datos.

### 1.3. Justificación frente a Alternativas Evaluadas
* En **ADR-001** se evaluó como alternativa la **Arquitectura Dirigida por Eventos (EDA - Event-Driven Architecture)**.
* **Motivo del descarte de EDA:** El modelo asíncrono de publicación/suscripción (*pub/sub*) no se ajusta al ciclo directo de peticiones web HTTP/REST exigido por RD-1 y requiere incorporar y gestionar infraestructura intermedia de mensajería (*event brokers* o colas), lo cual introduce una sobrecarga y complejidad operativa innecesaria para el sistema.

---

## 2. Descomposición de Componentes de la Arquitectura

```
+-----------------------------------------------------------+
|                      CLIENTE (Front-end Web)              |
|  - Interfaz de usuario para lectores, suscriptores        |
|    y administradores/redactores                           |
|  - Emisión de peticiones y renderizado de respuestas      |
+-----------------------------------------------------------+
                              |
                              | Peticiones HTTP/REST (RD-1)
                              v
+-----------------------------------------------------------+
|                     SERVIDOR (Back-end)                   |
|  - Recepción y enrutamiento de peticiones HTTP/REST       |
|  - Lógica de suscripción y baja anónima (RF-1, RF-2)      |
|  - Gestión de cobros y facturación recurrente (RF-3, RF-4)|
|  - Módulo de estadísticas de uso (RF-5)                   |
|  - Integración con pasarela STRIPE (RD-2)                 |
|  - Persistencia en PostgreSQL 19 con BD separadas         |
|    por perfil (RD-3, RD-4)                                |
+-----------------------------------------------------------+
```

### 2.1. Capa Cliente (Front-end Web)
* **Responsabilidad:** Proporcionar la interfaz gráfica e interacción para los diferentes perfiles:
  * Lectores y suscriptores: navegar por noticias, solicitar suscripciones anónimas (`RF-1`) y tramitar bajas (`RF-2`).
  * Administradores y redactores: consultar las métricas y estadísticas de uso de la plataforma (`RF-5`).
* **Comportamiento:** Actúa como agente activo que origina las peticiones síncronas HTTP/REST hacia el servidor. No almacena datos persistentes de negocio ni contiene lógica de facturación.

### 2.2. Canal de Comunicación
* **Protocolo:** Estándar **HTTP/REST** en cumplimiento de **RD-1**.
* **Naturaleza:** Comunicación cliente-servidor sin estado (*stateless*), desacoplada y orientada a recursos mediante métodos estándar (GET, POST, PUT, DELETE).

### 2.3. Capa Servidor (Back-end)
* **Responsabilidad:** Atender, validar y procesar las peticiones procedentes del cliente. Centraliza:
  * El control de acceso y gestión del ciclo de vida de las suscripciones anónimas (`RF-1`, `RF-2`).
  * La coordinación de los cobros periódicos trimestrales, semestrales o anuales (`RF-3`, `RF-4`).
  * El cálculo y agregación de las métricas de noticias más leídas, tiempo de permanencia, tasas de rebote y conversión (`RF-5`).
  * La comunicación segura con la pasarela de pagos Stripe (`RD-2`) y el almacenamiento en bases de datos PostgreSQL 19 independientes para suscriptores y administradores (`RD-3`, `RD-4`).
 