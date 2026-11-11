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

---

## 2. Orden del Día

1. **Definición del patrón de acceso a datos del servidor:** Evaluación de la propuesta para ADR-002.
2. **Definición de la integración con la pasarela de pago:** Evaluación de la propuesta para ADR-003.
3. **Validación de la trazabilidad con los requisitos:** Justificación de ADR-002 a partir de **RD-3** (*SGBD PostgreSQL 19*) y **RD-4** (*Bases de datos separadas por perfil*), y de ADR-003 a partir de **RD-2** (*Pasarela de pago STRIPE*).
4. **Instrucciones para el modelado UML.**

---

## 3. Desarrollo de la Discusión

### 3.1. Debate sobre ADR-002: Manejo de las Bases de Datos

* **Propuesta inicial de los ASS:**
  * Adoptar el patrón **Repository** como estrategia de acceso a datos del servidor, con interfaces de repositorio diferenciadas para el contexto de suscriptores y el de administradores/redactores.
  * La decisión se fundamenta en los requisitos de dominio **RD-3** (PostgreSQL 19) y **RD-4** (una base de datos para administradores/redactores y otra para suscriptores).
  * Se busca que la lógica de negocio del periódico no contenga sentencias SQL y que cada implementación de repositorio encapsule la conexión a la base de datos de su perfil.

* **Cuestionamientos y alternativas planteados por los ASC:**
  * **Descarte de Active Record:** Los dos ASC llegan a la conclusión de que Active Record era una mala idea para este sistema. Acopla las entidades del periódico a la estructura de tablas de PostgreSQL 19 (RD-3), obliga a que cada entidad sepa a cuál de las dos bases de datos pertenece (RD-4) y mezcla en las mismas clases las reglas de negocio y la persistencia.
  * **Repository frente a acceso directo (Ivo):** Ivo pregunta por qué usar Repository y no conectarse directamente a PostgreSQL desde los servicios, si ambas opciones se parecen mucho.
  * **Exigencia de justificación (Ivo):** Tras la respuesta de los ASS, Ivo acepta el planteamiento, pero exige que cada ventaja e inconveniente de las opciones quede justificado con su porqué y ligado al caso del periódico.

* **Defensa y argumentación de los ASS:**
  * **Respuesta de Dani sobre el acceso directo:** Conectarse directamente es una buena idea al principio, porque es rápido de escribir. El problema aparece después: cambiar el código se complica porque se producen duplicados.
  * **Duplicación y cambios de esquema:** Con acceso directo, cada servicio debe elegir manualmente cuál de las dos bases de datos consulta (RD-4) y repetir sus propias consultas y gestión de conexiones. Un cambio en el esquema de una base de datos obligaría a revisar todos los servicios que la consultan.
  * **Concentración del acceso a PostgreSQL 19:** Con Repository, el acceso a PostgreSQL 19 (RD-3) queda concentrado en las implementaciones de los repositorios y el dominio permanece libre de SQL.
  * **Separación por perfil por diseño:** Cada interfaz de repositorio expone solo las operaciones que necesita su perfil (por ejemplo, el alta y la baja de suscripciones en el de suscriptores), lo que hace cumplir la separación de RD-4 en el propio diseño.
  * **Coste asumido:** Repository exige más interfaces y clases, y definir el mapeo entre dominio y tablas. Los ASS lo aceptan porque evita los duplicados y el acoplamiento de las otras dos opciones.
  * **Ventaja reconocida del acceso directo:** Da control total sobre las consultas SQL, útil para las métricas de uso (noticias más leídas, tiempo de permanencia, tasas de rebote y conversión). Se considera que no compensa el coste de mezclar SQL con la lógica de los servicios.

* **Conclusión y Acuerdo:**
  * Atendiendo a la petición de Ivo, los ASS revisan el ADR para que los pros y contras de cada opción estén contextualizados con RD-3 y RD-4 y con el caso del periódico.
  * Los ASC aceptan las razones expuestas por los ASS. Se descartan Active Record y el Acceso directo a PostgreSQL.
  * Se aprueba unánimemente y pasa a estado **aceptado** (*accepted*) el patrón **Repository** para ADR-002.

### 3.2. Debate sobre ADR-003: Manejo de la Pasarela de Pagos

* **Propuesta inicial de los ASS:**
  * Adoptar el patrón **Adapter** para integrar Stripe: una interfaz de pasarela de pago definida por el propio sistema y una clase **StripeAdapter** que la implementa y encapsula las llamadas al SDK de Stripe.
  * La decisión se fundamenta en el requisito de dominio **RD-2**, que establece que la pasarela de pago será Stripe.
  * Se busca que las clases del módulo de pago online no dependan directamente del SDK de Stripe.

* **Cuestionamientos y alternativas planteados por los ASC:**
  * **Opción "SDK" y Adapter (Héctor):** Héctor pregunta por qué una de las opciones es el SDK si el patrón Adapter ya utiliza ese SDK. Le parece que es como poner una interfaz a la interfaz.
  * **Aclaración de la opción (Dani e Ivo):** Dani aclara que se refería al uso directo del SDK, sin ninguna capa intermedia. Ivo propone que la opción se llame explícitamente **Uso directo del SDK de Stripe**, para dejarlo más claro y evitar confusiones. El grupo lo acepta.
  * **Necesidad de la capa intermedia (Ivo):** Ivo pregunta por qué no se usa directamente el SDK de Stripe, si no se va a cambiar de pasarela.

* **Defensa y argumentación de los ASS:**
  * **Respuesta de Dani:** Con Adapter se separa la lógica de negocio de la pasarela, y el SDK se puede actualizar con más facilidad.
  * **Los motivos no son cambiar de proveedor:** Los ASS reconocen que RD-2 fija Stripe y que no se contempla otra pasarela. La ventaja está en confinar en una sola clase cualquier cambio en la API o el SDK de Stripe, sin tocar las clases del módulo de pago online que la utilizan.
  * **Comunicación con el servicio externo en un único punto:** Con uso directo, las llamadas a Stripe quedarían repartidas por las distintas clases que cobran suscripciones. Con Adapter se concentran en una sola clase.
  * **Adapter y SDK no son la misma capa:** Adapter es el único componente que conoce el SDK, y la interfaz es del propio sistema, que decide qué operaciones de pago expone (por ejemplo, cobrar una suscripción) en lugar de heredar las del SDK.
  * **Coste asumido:** Se añade una interfaz y una clase traductora para un único proveedor, y hay riesgo de ocultar funciones muy específicas de Stripe si la interfaz es demasiado genérica. Los ASS lo aceptan y concretan que la interfaz solo debe exponer las operaciones de pago que necesita el periódico.
  * **Ventaja reconocida del uso directo:** Da acceso inmediato a todas las funciones y tipos nativos de Stripe sin escribir clases adicionales. Se descarta porque acopla las clases de negocio a la librería de un tercero.
  * **Coherencia con las decisiones previas:** El adaptador queda dentro del servidor definido en ADR-001 y no afecta al patrón Repository de ADR-002.

* **Conclusión y Acuerdo:**
  * Héctor e Ivo aceptan las razones expuestas por los ASS. Se descarta el Uso directo del SDK de Stripe por el acoplamiento de las clases de negocio con la librería externa.
  * Se aprueba unánimemente y pasa a estado **aceptado** (*accepted*) el patrón **Adapter** para ADR-003.

---

## 4. Trazabilidad de Requisitos

Para dar cumplimiento estricto a las normas de evaluación y garantizar que ningún requisito se reutilice en más de una decisión:

* **ADR-001:** Se motiva **única y exclusivamente** por el requisito **RD-1** (*Peticiones HTTP/REST*).
* **ADR-002:** Se motiva **únicamente** por los requisitos **RD-3** (*SGBD PostgreSQL 19*) y **RD-4** (*Bases de datos separadas por perfil*).
* **ADR-003:** Se motiva **única y exclusivamente** por el requisito **RD-2** (*Pasarela de pago STRIPE*).
* Los requisitos funcionales (RF-1 a RF-5) quedan reservados para las decisiones de refinamiento y diseño detallado de las siguientes semanas.

---

## 5. Instrucciones para el Modelado UML

1. **Diagrama de Clases y Paquetes (Acceso a datos, ADR-002):**
   * Representar dentro del paquete **Servidor (Backend)** de ADR-001 un paquete de persistencia con las **interfaces de repositorio** diferenciadas para el contexto de suscriptores y el de administradores/redactores.
   * Modelar las **clases concretas** que implementan cada interfaz y encapsulan las consultas a su base de datos PostgreSQL 19 (RD-3), una para suscriptores y otra para administradores/redactores (RD-4).
   * Las clases de negocio deben depender únicamente de las interfaces de repositorio, y el dominio no debe mostrar dependencias con sentencias SQL ni con el driver de la base de datos.
2. **Diagrama de Clases y Paquetes (Pasarela de pago, ADR-003):**
   * Representar una **interfaz de pasarela de pago** propia del sistema, utilizada por los servicios del módulo de pago online.
   * Modelar la clase **StripeAdapter**, que implementa dicha interfaz y es la única que depende del SDK de Stripe (RD-2).
   * Ubicar ambos elementos dentro del paquete **Servidor (Backend)**.
3. **Entregables de la semana:**
   * Exportar los diagramas en formatos **PNG** y **SVG** y ubicarlos en la carpeta `Arquitecturas/Semana_2/`.
   * Subir los ADR-002 y ADR-003 a la carpeta `Decisiones/` y esta acta de discusión antes del cierre de la semana de trabajo.

---

**Firmas de conformidad:**

* *Daniel Marcos Serrano* (ASS)
* *Jaime Bergad Lario* (ASS)
* *Héctor Moreno García* (ASC)
* *Ivo Hagemann Martínez* (ASC)
* *Raúl García Piedra* (ASJ)
