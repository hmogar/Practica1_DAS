# Acta de Discusión de Decisiones de Diseño – Semana 1

**Asignatura:** Diseño y Arquitectura Software (DAS) – Universidad Rey Juan Carlos  
**Práctica:** Práctica 1: Captura y Representación de Decisiones de Diseño  
**Iteración:** Tarea 1 (Semana 1) – Análisis de requisitos y definición del estilo arquitectónico  
**Fecha de reunión:** 5 de noviembre de 2026  

---

## 1. Participantes y Roles

* **Arquitectos Software Senior (ASS):**
  * Daniel Marcos Serrano
  * Jaime Bergad Lario

* **Arquitectos Software Cognitivos (ASC):**
  * Héctor Moreno García
  * Ivo Hagemann Martínez

---

## 2. Orden del Día

1. **Revisión del estilo global y despliegue del backend:** Evaluación de la propuesta para ADR-001.
2. **Revisión de la estructura interna y aislamiento de dependencias:** Evaluación de la propuesta para ADR-002.
3. **Validación con los requisitos de la práctica.**

---

## 3. Desarrollo de la Discusión

### 3.1. Debate sobre ADR-001: Despliegue y Organización del Backend

* **Propuesta inicial de los ASS:**
  * Adoptar un **Monolito Modular** como estilo de organización y despliegue del backend.
  * Se agrupan las funciones por responsabilidades funcionales (suscripciones, control de acceso, pagos, métricas) dentro de un único artefacto desplegable.

* **Cuestionamientos y alternativas planteados por los ASC (Héctor e Ivo):**
  * **Riesgo de punto único de fallo:** Si todo el backend corre en un único proceso, un fallo o fuga de memoria en la generación de estadísticas pesadas (RF-7) podría degradar o tumbar el servicio crítico de pagos (RF-5) o el acceso a noticias (RF-3).
  * **Alternativa de Microservicios:** Se plantea si no sería más seguro y moderno desplegar microservicios independientes (Servicio de Suscripciones/Lectores, Servicio de Pagos, Servicio de Analítica/Administración), permitiendo además aislar físicamente el acceso a las dos bases de datos requeridas por **RD-9**.

* **Defensa y argumentación de los ASS:**
  * **Sobrecarga y complejidad innecesaria:** Los microservicios introducen una latencia de red adicional, necesidad de orquestación, gestión de transacciones distribuidas y mantenimiento de múltiples pipelines de despliegue, lo que viola el principio de mantener sencillo el desarrollo y mantenimiento.
  * **Cumplimiento de RD-9 en monolito:** La separación estricta de bases de datos exigida por RD-9 (una para redactores/administradores y otra para suscriptores) se puede satisfacer plenamente dentro de un monolito modular configurando fuentes de datos independientes por módulo, sin necesidad de separar el despliegue en procesos distribuidos.

* **Conclusión y Acuerdo:**
  * Los ASC aceptan la justificación. Se descarta la opción de Microservicios por su sobrecoste operativo y se **acepta** el **Monolito Modular**.

---

### 3.2. Debate sobre ADR-002: Estructura Interna y Aislamiento de Dependencias Externas

* **Propuesta inicial de los ASS:**
  * Aplicar el patrón arquitectónico de **Arquitectura Hexagonal** para organizar el código interno del backend.

* **Cuestionamientos y alternativas planteados por los ASC:**
  * **Complejidad y sobrecarga de código:** La arquitectura hexagonal exige la definición explícita de puertos (interfaces) y adaptadores para cada interacción, lo que incrementa el número inicial de clases, paquetes y complejidad estructural frente a una **Arquitectura en Capas tradicional**.

* **Defensa y argumentación de los ASS (Daniel y Jaime):**
  * **Acoplamiento tecnológico en Capas:** En una arquitectura en capas tradicional, la capa de negocio suele acoplarse directamente a librerías de persistencia (PostgreSQL) o clientes SDK externos (Stripe), haciendo costoso sustituir o actualizar estas tecnologías.
  * **Aislamiento de requisitos críticos:**
    * **RD-3 (HTTP/REST):** Los controladores REST actúan limpiamente como adaptadores primarios, desacoplando el protocolo web de las reglas de negocio.
    * **RD-6 (Stripe) y RD-7 (Reintentos):** La integración con la pasarela Stripe y la política obligatoria de hasta 3 reintentos de cobro quedan totalmente encapsuladas en un adaptador secundario, sin contaminar los casos de uso de facturación.
    * **RD-8 y RD-9 (PostgreSQL 19 y dos bases de datos):** La arquitectura hexagonal permite disponer de adaptadores de persistencia diferenciados para cada una de las dos bases de datos, garantizando que el dominio no conozca los detalles de conexión física.
  * **Testabilidad:** Permite verificar la lógica de negocio y los casos de uso mediante tests unitarios rápidos sustituyendo los adaptadores externos por dobles de prueba, sin depender de servicios externos ni levantar bases de datos.

* **Conclusión y Acuerdo:**
  * Los ASC consideran fundamentada la necesidad de aislamiento ante las exigencias de Stripe (RD-6, RD-7) y las dos bases de datos (RD-8, RD-9). Se **acepta** la **Arquitectura Hexagonal**.

---

**Firmas:**
* *Daniel Marcos Serrano* (ASS)
* *Jaime Bergad Lario* (ASS)
* *Héctor Moreno García* (ASC)
* *Ivo Hagemann Martínez* (ASC)
