
# Documento de Arquitectura Software – Semana 1

## 1. Niveles Arquitectónicos y Estilos Seleccionados

### 1.1. Nivel 1: Macro-arquitectura (Estilo Global del Sistema)
* **Estilo:** **Cliente-Servidor**
* **Requisitos que lo motivan:**
  * **RD-1 (Interfaz web responsiva):** El sistema dispone de un Front-end web construido para clientes PC y Tablet.
  * **RD-3 (Comunicación HTTP/REST):** Todas las interacciones cliente-servidor se realizan obligatoriamente mediante protocolos HTTP/REST estándar.
* **Descripción:** 
  * El **Cliente** (Front-end Web) asume la presentación, renderizado de vistas responsivas, manejo de tokens anónimos de sesión (**RD-2**) y visualización parcial o total de noticias (**RF-3**).
  * El **Servidor** (Back-end) centraliza la lógica de suscripciones, control de cobros recurrentes y cálculo de analítica.

---

### 1.2. Nivel 2: Meso-arquitectura (Organización y Despliegue del Backend)
* **Estilo / Decisión:** **Monolito Modular** (recogida en ADR-001)
* **Requisitos que lo motivan:**
  * **RF-1 a RF-7:** Funciones claramente diferenciadas con perfiles de usuario distintos (lectores anónimos vs administradores/redactores).
  * **RD-9 (Separación de bases de datos por contexto):** Exigencia de contar con una base de datos para administradores/redactores y otra separada para suscriptores.
* **Justificación frente a alternativas:** Se descarta microservicios por evitar complejidad.
* **Módulos definidos:**
  1. **Módulo de Suscripciones y Lectura:** Alta y baja anónima (`RF-1`, `RF-2`), sesiones anónimas (`RD-2`) y control de acceso a noticias (`RF-3`).
  2. **Módulo de Facturación y Pagos:** Selección de planes (`RF-5`, `RD-5`), gestión de cobros y renovación periódica (`RF-6`).
  3. **Módulo de Analítica y Estadísticas:** Panel de métricas de permanencia, rebote, noticias leídas y conversión (`RF-7`).

---

### 1.3. Nivel 3: Micro-arquitectura (Estructura Interna de Componentes)
* **Patrón de Diseño:** **Arquitectura Hexagonal (recogida en ADR-002)
* **Requisitos que lo motivan:**
  * **RD-3:** Exposición de endpoints HTTP/REST limpios.
  * **RD-6 y RD-7:** Integración imperativa con la pasarela Stripe y gestión de política de hasta 3 reintentos de cobro.
  * **RD-8 y RD-9:** Uso del SGBD PostgreSQL 19 y separación de bases de datos por contexto.
* **Estructura por Módulo:**
  * **Adaptadores Primarios (Driving):** Controladores REST que reciben las peticiones HTTP del Front-end y delegan en los casos de uso.
  * **Capa de Dominio / Aplicación:** Entidades de negocio, reglas de dominio y definición de **Puertos** (interfaces Java/TypeScript/etc.) de entrada y salida.
  * **Adaptadores Secundarios (Driven):**
    * *Adaptador de Stripe:* Implementa la interfaz de pasarela de pago y maneja los 3 reintentos (`RD-7`).
    * *Adaptadores de Persistencia:* Implementan las interfaces de repositorio y se conectan mediante DataSource independiente a cada base de datos PostgreSQL 19 (`RD-8`, `RD-9`).


