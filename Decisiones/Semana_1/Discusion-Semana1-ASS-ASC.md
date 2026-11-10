# Acta de Discusión de Decisiones de Diseño – Semana 1

**Asignatura:** Diseño y Arquitectura Software (DAS) – Universidad Rey Juan Carlos  
**Práctica:** Práctica 1: Captura y Representación de Decisiones de Diseño  
**Iteración:** Tarea 1 (Semana 1) – Análisis de requisitos y definición del estilo arquitectónico  
**Fecha de reunión:** 10 de noviembre de 2026  

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

---

## 2. Orden del Día

1. **Definición del estilo arquitectónico general del sistema:** Evaluación de la propuesta para ADR-001.
2. **Validación de la trazabilidad con los requisitos:** Justificación de la decisión a partir del requisito **RD-1** (*Peticiones HTTP/REST*).
3. **Instrucciones para el modelado UML inicial.**

---

## 3. Desarrollo de la Discusión

### 3.1. Debate sobre ADR-001: Estilo Arquitectónico General del Sistema

* **Propuesta inicial de los ASS:**
  * Adoptar la **Arquitectura Cliente-Servidor** como único estilo arquitectónico global del periódico digital.
  * La decisión se fundamenta en el requisito de dominio imperativo **RD-1**, que exige que todas las peticiones se realicen mediante protocolos HTTP/REST.
  * Se busca separar claramente la capa de presentación que se ejecuta en el dispositivo del usuario del procesamiento y almacenamiento centralizado en el servidor.

* **Cuestionamientos y alternativas planteados por los ASC:**
  * **Alternativa Dirigida por Eventos (EDA - Event-Driven Architecture):** Los ASC plantean si un estilo arquitectónico basado en eventos asíncronos y publicación/suscripción (*pub/sub*) no aportaría mayor desacoplamiento y reactividad al sistema, especialmente para gestionar las notificaciones o la actualización de métricas de uso en segundo plano.
  * **Riesgo de punto único de fallo y cuello de botella:** Se argumenta que en una arquitectura Cliente-Servidor clásica, el servidor centralizado puede sufrir caídas o degradación de rendimiento ante picos de concurrencia, afectando a la totalidad de los usuarios.

* **Defensa y argumentación de los ASS:**
  * **Alineación directa con RD-1 (Peticiones HTTP/REST):** El protocolo HTTP/REST prescrito por el requisito RD-1 funciona de manera nativa mediante un ciclo síncrono de **petición-respuesta** (*Request-Response*) iniciado por el cliente. Una arquitectura orientada a eventos desvirtúa este flujo natural de navegación y consulta web.
  * **Complejidad operativa y sobrecoste de infraestructura:** Una arquitectura EDA requiere incorporar y gestionar infraestructura adicional de mensajería, lo que introduce una complejidad técnica innecesaria que no está justificada por ningún requisito del enunciado.
  * **Consistencia inmediata y simplicidad:** La operativa del periódico se beneficia de la confirmación inmediata propia del modelo Cliente-Servidor, evitando la complejidad de la consistencia eventual que suele acompañar a las arquitecturas basadas en eventos.
  * **Separación de responsabilidades:** Cliente-Servidor permite desacoplar limpiamente la interfaz de usuario para que pueda evolucionar de forma independiente del backend.

* **Conclusión y Acuerdo:**
  * Los ASC aceptan las razones expuestas por los ASS. Se descarta la opción Dirigida por Eventos (EDA) por su incompatibilidad conceptual con el modelo de interacción REST y su sobrecarga de infraestructura.
  * Se aprueba unánimemente y pasa a estado **aceptado** (*accepted*) la **Arquitectura Cliente-Servidor** para ADR-001.

---

## 4. Trazabilidad de Requisitos

Para dar cumplimiento estricto a las normas de evaluación y garantizar que ningún requisito se reutilice en más de una decisión:

* **ADR-001:** Se motiva **única y exclusivamente** por el requisito **RD-1** (*Peticiones HTTP/REST*).
* Los restantes requisitos de dominio (RD-2, RD-3, RD-4) y funcionales (RF-1 a RF-5) quedan reservados para las decisiones de refinamiento y diseño detallado de las siguientes semanas.

---

## 5. Instrucciones para el Modelado UML

1. **Diagrama de Paquetes / Componentes (Estilo Arquitectónico):**
   * Representar la **Arquitectura Cliente-Servidor** reflejada en ADR-001.
   * Modelar el paquete **Cliente (Front-end Web)** y el paquete **Servidor (Backend)** como bloques claramente diferenciados.
   * Mostrar la relación de comunicación cliente-servidor mediante una dependencia etiquetada explícitamente con `<<HTTP/REST>>` (RD-1).
2. **Entregables de la semana:**
   * Exportar el diagrama en formatos **PNG** y **SVG** y ubicarlo en la carpeta `Arquitecturas/Semana_1/` antes del cierre de la semana de trabajo.

---

**Firmas de conformidad:**

* *Daniel Marcos Serrano* (ASS)
* *Jaime Bergad Lario* (ASS)
* *Héctor Moreno García* (ASC)
* *Ivo Hagemann Martínez* (ASC)
* *Raúl García Piedra* (ASJ)
