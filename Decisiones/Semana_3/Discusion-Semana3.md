# Acta de Discusión de Decisiones de Diseño – Semana 3

**Asignatura:** Diseño y Arquitectura Software (DAS) – Universidad Rey Juan Carlos  
**Práctica:** Práctica 1: Captura y Representación de Decisiones de Diseño  
**Iteración:** Tarea 3 (Semana 3) – Completar el diseño hasta terminar los requisitos  
**Fecha de reunión:** 17 de noviembre de 2026  

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

1. Revisión de ADR-003 (integración con Stripe y cobros recurrentes) y posterior rechazo; registro de la pasarela de pagos: ADR-003-1 y ADR-003-2.
2. Responsabilidad de consultar estadísticas: ADR-004.
3. Ubicación de las reglas de alta y baja de las suscripciones anónimas: ADR-005.
4. Modelado de las modalidades de facturación: ADR-006.
5. Cancelación de ADR-002 (patrón Repository) y registro del sistema gestor de bases de datos: ADR-002-1 y ADR-002-2.
6. Validación de la trazabilidad con los requisitos.
7. Instrucciones para el modelado UML.

## 3. Desarrollo de la Discusión

La discusión se desarrolló en dos momentos. En el primero, los ASS y los ASC revisaron la actualización de ADR-003 y las propuestas de ADR-004, ADR-005 y ADR-006. En el segundo, los ASS repasaron el conjunto de ADR resultante, incorporaron los cambios de nivel de detalle (cancelación de ADR-002 y rechazo de ADR-003) y reformularon ADR-004. Las secciones siguientes recogen ambos momentos por decisión.

### 3.1. Revisión de ADR-003: integración con Stripe y cobros recurrentes. Rechazo y selección de la pasarela (ADR-003-1 y ADR-003-2)

* **Propuesta de los ASS (revisión de ADR-003):**
  * Actualizar ADR-003 para que cumpla el requisito de dominio RD-2 (Stripe como pasarela de pagos), los cobros recurrentes de RF-4 y el límite de tres reintentos indicado en el Anexo I, requisito que no se había recogido en la versión anterior.
  * Mantener **Adapter** como frontera con el SDK de Stripe y delegar en Stripe la gestión de la recurrencia.
  * ADR-006 define las modalidades (trimestral, semestral y anual) y se relaciona con ADR-003, pero el diagrama de la Semana 2 no necesita cambios por ello.

* **Defensa y argumentación de los ASS:**
  * Los ASS indican que los cambios se han hecho en local sobre el material de la Semana 2 (ADR y acta de discusión), sin haberse subido aún al repositorio, y que se tratan como una actualización de ese material.
  * La decisión sobre quién ejecuta los cobros no cambia cómo se modelan las modalidades, que se resuelven en ADR-006.

* **Posterior reconsideración y rechazo (ADR-003-1):**
  * Al repasar el nivel de detalle del conjunto de ADR, los ASS concluyen que ADR-003 se ha ido a un nivel demasiado bajo. El requisito RD-2 establece que el sistema usa Stripe, no cómo se conecta ni cómo funciona la integración.
  * Los ASS deciden **rechazar ADR-003 en su conjunto** y registrar por separado la selección de la pasarela, tal como exige RD-2 (ADR-003-2). El rechazo no elimina los requisitos de cobro recurrente ni de reintentos, sino que deja fuera de esta decisión cómo se implementarán.
  * Los ASS reconocen que ADR-003-2 queda con una única alternativa. Lo justifican en que RD-2 impone Stripe como pasarela, por lo que no existen alternativas válidas que comparar, y señalan que el enunciado también les obliga a registrar esta decisión.

* **Conclusión y acuerdo:**
  * ADR-003 queda **rechazado** y se conserva como registro histórico. Esto sustituye el estado *accepted* acordado en la Semana 2.
  * ADR-003-1 (rechazo) se registra con estado **accepted**.
  * ADR-003-2 registra Stripe como pasarela de pagos por RD-2. 

### 3.2. Debate sobre ADR-004: responsabilidad de consultar estadísticas

* **Propuesta inicial de los ASS (versión sobre registro de eventos):**
  * Los ASS habían reformulado las opciones de ADR-004 tras las objeciones de la reunión anterior. La primera opción era un simple contador que no almacenaba información y no cubría todas las métricas de RF-5, y se les pidió pasar del *qué* al *cómo*.
  * La nueva propuesta consistía en tres opciones que sí cubren RF-5: registrar una visita por lector con sus datos resumidos (inicio, fin, noticias leídas y si terminó en suscripción), registrar eventos de uso y calcular bajo demanda mediante `StatsGeneration`, o precalcular periódicamente las métricas. La opción elegida era la segunda.

* **Cuestionamientos y alternativas planteados por los ASC:**
  * Los ASC señalan que el enunciado no menciona lectores sin suscripción, pero que RF-5.4 habla de conversión de *lectores a suscriptores*, lo que implicaría que existen lectores no suscritos. Los ASS recuerdan que, al preguntar al profesor la primera semana, se les indicó que no debían añadir requisitos que el enunciado no recoge explícitamente. Los requisitos ya no pueden modificarse (límite del 10/11), por lo que la duda queda abierta.
  * Sobre la opción de la visita con datos resumidos, los ASC preguntan cómo se conoce la hora de fin, ya que hasta que el lector abre otra noticia no se sabe si salió, y por qué se sobrescribe el registro en lugar de acumular información.
  * Los ASC subrayan que las métricas de RF-5 son globales y no por usuario, y preguntan en qué momento los datos de cada visita pasan a las métricas globales y dónde se almacenan estas.
  * Los ASC consideran que la discusión debe centrarse en cómo se obtienen y se muestran las estadísticas, y no tanto en cómo se guardan los datos de cada persona.
  * Los ASC plantean dónde se guardan los datos para que no se pierdan al apagar el servidor (base de datos, clase en memoria con volcado periódico, caché), y sostienen que el ADR debe mencionar de dónde salen los datos que se muestran.

* **Defensa y argumentación de los ASS:**
  * Los ASS explican que `StatsGeneration` concentra las reglas de cálculo y que los eventos registran la interacción de cada lector con cada noticia y si se suscribe. Las noticias más leídas se obtienen sumando los registros de cada noticia.
  * Los ASS indican que el enunciado solo pide mostrar estadísticas de uso a administradores y redactores, y que no especifica ni el almacenamiento ni que haya estadísticas por usuario concreto. Registrar sesiones individuales sería asumir más de lo que dicen los requisitos.
  * Como acuerdo provisional, la visita de cada lector se registra durante su sesión y, al terminar, se actualizan las estadísticas globales y se descarta la información específica del usuario.
  * Los ASS y los ASC acuerdan trasladar al profesor las dudas sobre ADR-004 y sobre la base de datos en la siguiente clase.

* **Reformulación de ADR-004:**
  * Tras esta discusión y la revisión del nivel de detalle del conjunto de ADR, los ASS reformulan ADR-004 para decidir quién asume la responsabilidad de ofrecer las consultas de RF-5, dejando fuera los detalles de comunicación técnica y persistencia.
  * Opciones: (1) **módulo de estadísticas común**, al que administradores y redactores solicitan las métricas y que las calcula internamente; (2) módulos separados por ámbito, cada uno con su propio módulo de estadísticas.
  * Se cambia el nombre del componente a `ModuloEstadisticas`.
  * Los ASS eligen la primera opción porque habrá consultas comunes (por ejemplo, las visitas) y el enunciado no exige módulos separados. Reconocen como coste que el módulo puede crecer si se añaden muchas métricas.

* **Conclusión y acuerdo:**
  * Se acepta el **módulo de estadísticas común** (`ModuloEstadisticas`). ADR-004 queda con estado **accepted**.
  * La propuesta previa basada en eventos de uso y `StatsGeneration` queda sustituida por esta versión.
  * Queda pendiente consultar al profesor el tratamiento de los datos de las métricas y los lectores no suscritos de RF-5.4.

### 3.3. Debate sobre ADR-005: reglas de alta y baja de las suscripciones anónimas

* **Propuesta de los ASS:**
  * Encapsular en `Suscripcion` las reglas de sus propias operaciones de alta y baja anónimas (RF-1 y RF-2), y que el servicio de aplicación solo localice la suscripción y la llame para darla de alta o de baja.
  * Alternativas: servicio centrado en la lógica o modelo de dominio con reglas. Se elige el modelo de dominio con reglas.

* **Cuestionamientos y alternativas planteados por los ASC y por los ASS entre sí:**
  * Los ASC recuerdan que ADR-005 no ha cambiado desde la discusión anterior. Los ASS aclaran que «las reglas» se refiere a cómo se hace el alta y la baja, no a la facturación.
  * Se plantea si debe incorporarse un patrón relativo a suscriptores y se identifica con Observer (publish-subscribe), mecanismo para notificar a varios objetos cuando cambia el objeto observado.
  * Algunos ASS preguntan si incluir un patrón haría descender demasiado el nivel de detalle de la decisión.

* **Defensa y argumentación de los ASS:**
  * Las opciones de ADR-005 responden a *dónde viven las reglas* de alta y baja (servicio o `Suscripcion`). Observer responde a *cómo se notifica* a otros objetos cuando algo cambia, por ejemplo al publicarse contenido nuevo, y no resuelve dónde van las validaciones de RF-1 y RF-2, por lo que no compite con ellas.
  * Observer solo tendría sentido combinado con la opción elegida si algún requisito pidiera notificar a los suscriptores. Como no existe ese requisito, se descarta incorporarlo.
  * Los ASS recuerdan que los patrones sí pueden usarse en las decisiones, y que el problema detectado en otros ADR fue el nivel de detalle y no el uso de patrones.
  * Se reconoce como coste que `Suscripcion` incluye comportamiento además de datos, y que hay que decidir qué validaciones corresponden a la clase y cuáles al servicio.

* **Conclusión y acuerdo:**
  * Se mantiene el **modelo de dominio con reglas**, sin Observer.

### 3.4. Debate sobre ADR-006: modalidades de facturación de las suscripciones

* **Propuesta de los ASS:**
  * Representar las modalidades del Anexo I (trimestral 70 €, semestral 130 € y anual 190 €) mediante **composición con Strategy**. `Suscripcion` mantiene una referencia a la interfaz `CicloFacturacion`, implementada por `CicloTrimestral`, `CicloSemestral` y `CicloAnual`, que aportan el importe y el cálculo del siguiente periodo (3, 6 y 12 meses, RF-4).
  * Alternativas: herencia con una subclase de `Suscripcion` por modalidad, composición con Strategy y enumerado con periodo e importe.
  * El requisito que motiva la decisión es RF-3 (gestión de ciclos de facturación mediante un módulo de pago online).

* **Cuestionamientos y alternativas planteados por los ASC:**
  * Los ASC preguntan si la herencia no resulta una opción de demasiado bajo nivel.
  * Los ASC cuestionan si ADR-006 es necesario, por parecer poco específico.

* **Defensa y argumentación de los ASS:**
  * Los ASS recuerdan que se pueden plantear clases, paquetes, patrones y estilos, y que modelar la jerarquía de clases es una decisión de diseño legítima.
  * Con herencia, `Suscripcion` sería abstracta con tres subclases, cuando `Suscripcion` ya realiza por sí misma el alta y la baja (ADR-005). Además, la modalidad quedaría fijada al tipo del objeto, y cambiar de plan obligaría a crear un objeto de otra clase y a añadir condicionales en el código.
  * Con Strategy, lo relacionado con la suscripción (alta y baja) permanece en `Suscripcion`, y lo relacionado con el pago (importe y fecha del siguiente periodo) queda en un objeto `CicloFacturacion` asociado a ella pero que no gestiona la propia suscripción. Un cambio de modalidad consiste en sustituir ese objeto, y una modalidad nueva requiere solo una nueva implementación.
  * Los ASS responden que ADR-006 es necesario porque RF-3 lo exige, y que los requisitos no pueden modificarse.
  * ADR-006 no decide cómo ni cuándo se ejecuta el cobro, solo cómo se representan las modalidades. Se reconoce como coste que hay clases con muy poco comportamiento propio y que hay que definir el contrato de `CicloFacturacion`.

* **Conclusión y acuerdo:**
  * Los ASC aceptan la propuesta. Se elige la **composición con Strategy**. 
### 3.5. Cancelación de ADR-002 y selección del gestor de bases de datos (ADR-002-1 y ADR-002-2)

* **Propuesta de los ASS:**
  * Cancelar ADR-002 (patrón Repository), porque la decisión se había centrado en un nivel demasiado bajo (ADR-002-1).
  * Registrar en ADR-002-2 la selección de **PostgreSQL 19** como sistema gestor, que exige RD-3. El requisito establece que debe haber una base de datos, no cómo se conecta.
  * Alternativas de ADR-002-1: mantener ADR-002 o cancelarlo y registrar el gestor por separado.

* **Cuestionamientos y alternativas planteados:**
  * Los ASC no formularon objeciones a la cancelación en esta reunión.
  * Los ASS indican que ADR-002-2 contiene una sola alternativa, dado que RD-3 impone PostgreSQL 19. Con ADR-003-2 son dos de los seis ADR con una única decisión posible, y asumen que el enunciado y los requisitos no les dejan otra vía.
  * Los participantes acuerdan preguntar al profesor en la siguiente clase por la base de datos.

* **Conclusión y acuerdo:**
  * ADR-002 queda **cancelado**. ADR-002-1 se registra con estado **accepted**.
  * ADR-002-2 registra PostgreSQL 19 por RD-3.
  * El patrón de acceso a datos queda pendiente para una etapa de diseño posterior.

## 4. Trazabilidad de Requisitos

| ADR       | Estado    | Requisitos que lo motivan                                                   |
| --------- | --------- | --------------------------------------------------------------------------- |
| ADR-001   | accepted  | RD-1                                                                        |
| ADR-002   | cancelado | RD-3, RD-4 (según el acta de la Semana 2)                                   |
| ADR-002-1 | accepted  | Cancelación de ADR-002 (sin requisito propio)                               |
| ADR-002-2 | accepted  | RD-3                                                                        |
| ADR-003   | rechazado | RD-2, RF-4, límite de reintentos del Anexo I (según el acta de la Semana 2) |
| ADR-003-1 | accepted  | Rechazo de ADR-003 (sin requisito propio)                                   |
| ADR-003-2 | accepted  | RD-2                                                                        |
| ADR-004   | accepted  | RF-5 (RF-5.1 a RF-5.4)                                                      |
| ADR-005   | accepted  | RF-1, RF-2                                                                  |
| ADR-006   | accepted  | RF-3, RF-4                                                                  |

## 5. Cuestiones abiertas para consultar al profesor

1. Si la selección del gestor de bases de datos (ADR-002-2) debe mantenerse como decisión independiente.
2. Cómo debe tratarse en ADR-004 el origen y el almacenamiento de los datos de las métricas de RF-5.
3. Si RF-5.4 (conversión de lectores a suscriptores) implica la existencia de lectores no suscritos.


---

**Firmas de conformidad:**

* *Daniel Marcos Serrano* (ASS)
* *Jaime Bergad Lario* (ASS)
* *Héctor Moreno García* (ASC)
* *Ivo Hagemann Martínez* (ASC)
* *Raúl García Piedra* (ASJ)
