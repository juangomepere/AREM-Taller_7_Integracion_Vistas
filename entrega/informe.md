# Informe Técnico del Taller 7 – Integración de Vistas de Arquitectura

## Nombre del Taller
Taller 7 – Integración de Vistas de Arquitectura Empresarial

## Integrantes del equipo
- Juan Gómez (juangomepere)
- Rodriguez
- Sosa

---

## Descripción general del trabajo

Este taller tuvo como objetivo integrar en una narrativa visual y analítica coherente todas las vistas arquitectónicas desarrolladas durante el semestre para el cliente **Suporta S.A.S**, empresa colombiana de capacitación virtual que actualmente gestiona evaluaciones post-capacitación mediante procesos 100% manuales basados en Google Forms, Excel y correo electrónico.

A lo largo de los talleres previos (1 al 6) se construyeron vistas independientes: el modelo de proceso BPMN (Taller 1), el modelo de información entidad-relación (Taller 2), la arquitectura de infraestructura actual (Taller 4), el análisis de seguridad con STRIDE (Taller 5) y la evaluación de normatividad (Taller 6). Este taller sintetiza esas cinco vistas en un único tablero integrado que muestra cómo se relacionan, se soportan mutuamente y conforman una arquitectura coherente orientada a los objetivos de Suporta.

---

## Proceso de desarrollo

El equipo adoptó el siguiente proceso para construir la integración:

1. **Revisión del caso base FarmApp (clase):** Se analizó el caso de referencia con cinco vistas arquitectónicas y se construyó un tablero en draw.io que sirvió como plantilla mental para el ejercicio con el cliente real. La lección clave fue que las vistas de negocio y aplicaciones son el "esqueleto" de la integración, mientras que infraestructura y seguridad son capas de soporte transversal.

2. **Mapeo de entregables previos a vistas:** Cada taller previo se mapeó a una vista arquitectónica:
   - Taller 1 (BPMN) → Vista de Negocio
   - Taller 2 (ER) → Vista de Información
   - Taller 4 (Estado actual) → Vista de Infraestructura
   - Taller 5 (STRIDE) → Vista de Seguridad
   - Taller 6 (Normatividad) → Capa transversal de restricciones sobre todas las vistas

3. **Construcción del tablero integrado:** Se organizaron cinco swimlanes horizontales (uno por vista) y se dibujaron las flechas de trazabilidad entre capas. La decisión más importante fue el **sentido de dependencia**: los procesos de negocio son los que determinan qué aplicaciones se necesitan; las aplicaciones determinan qué entidades de información gestionan; la infraestructura soporta las aplicaciones; y la seguridad es transversal (controla todos los niveles).

4. **Redacción de la narrativa:** Se analizó cómo las decisiones tomadas en cada taller se reflejan en la integración final y qué brechas quedan abiertas.

---

## Análisis del modelo propuesto

### Cómo se estructura el modelo

El modelo integrado tiene cinco capas:

| Capa | Derivada de | Componentes clave |
|------|-------------|-------------------|
| Negocio | Taller 1 – BPMN | 5 procesos principales del ciclo de evaluación |
| Información | Taller 2 – ER | 6 entidades: CLIENTE, EMPLEADO, CAPACITACION, EVALUACION, PREGUNTA, RESPUESTA_EMPLEADO, INFORME_DIARIO |
| Aplicaciones | Taller 4 | 4 sistemas: Plataforma Evaluaciones, Motor Calificación, Generador Informes, Sistema Notificaciones |
| Infraestructura | Taller 4 | 3 zonas: Suporta, Nube (Google Workspace), Cliente |
| Seguridad | Talleres 5 y 6 | Links únicos, aislamiento por client_id, MFA, cifrado, logging |

### Cómo el BPMN guía el diseño de aplicaciones

Cada proceso del BPMN (Taller 1) se traduce directamente en uno o más componentes de aplicación:

- **"Cargar lista de empleados"** → La *Plataforma de Evaluaciones* necesita un módulo de gestión de listas con validación de duplicados.
- **"Enviar invitación con link único"** → El *Sistema de Notificaciones* recibe el disparador del proceso y envía el correo con token único.
- **"Enviar recordatorios automáticos"** → El mismo *Sistema de Notificaciones*, activado por un scheduler.
- **"Calificación automática (100%)"** → El *Motor de Calificación* implementa la lógica de negocio con umbral del 100%, que es una regla de negocio crítica surgida del análisis del proceso.
- **"Generar informe diario por empresa"** → El *Generador de Informes*, activado por un job programado.

Esta trazabilidad directa valida que el diseño de la capa de aplicaciones no tiene funcionalidades superfluas ni gaps.

### Cómo las entidades ER soportan los procesos de negocio

El modelo de información (Taller 2) provee las estructuras de datos que permiten que los procesos funcionen:

- **EMPLEADO** (con cliente_id FK) permite la segmentación multi-tenant que el proceso "generar informe por empresa" requiere.
- **RESPUESTA_EMPLEADO** (con atributo `correcto`) permite que el *Motor de Calificación* calcule el porcentaje de aciertos y aplique el umbral del 100%.
- **INFORME_DIARIO** (con cliente_id FK y fecha) almacena el resultado del proceso de generación diaria y garantiza trazabilidad histórica.
- La relación N:M entre EMPLEADO y EVALUACION (vía RESPUESTA_EMPLEADO) soporta el caso de que un empleado pueda repetir una evaluación (reintentos).

### Cómo la infraestructura actual limita la arquitectura objetivo

La infraestructura identificada en el Taller 4 revela que Suporta opera hoy sobre Google Workspace (plan básico/gratuito, sin SLA garantizado) y una sola PC de la coordinadora como punto central de procesamiento. Esto genera tres restricciones críticas para la arquitectura objetivo:

1. **Sin SLA de infraestructura:** Google Workspace gratuito no garantiza disponibilidad. La arquitectura objetivo necesita migrar a Google Workspace Business o a una plataforma con SLA formal (AWS, Azure, GCP).
2. **Dependencia de una persona:** El proceso actual depende físicamente de la coordinadora. La arquitectura objetivo debe eliminar ese cuello de botella con jobs automatizados en la nube.
3. **Sin base de datos formal:** Toda la información reside en hojas de cálculo. La arquitectura objetivo requiere una base de datos relacional (PostgreSQL o similar) con backups automatizados.

### Cómo los controles STRIDE mapean a capas específicas

El análisis STRIDE (Taller 5) identificó amenazas que ahora se pueden ubicar exactamente en las capas del modelo integrado:

| Amenaza STRIDE | Capa afectada | Control en el tablero |
|----------------|---------------|----------------------|
| Spoofing (respuestas duplicadas) | Aplicaciones | Links únicos por empleado (Seguridad) |
| Tampering (Excel sin cifrar) | Infraestructura | MFA + Cifrado informes (Seguridad) |
| Repudiation (sin log de envíos) | Infraestructura | Logging de auditoría (Seguridad) |
| Information Disclosure (cross-tenant) | Información | Aislamiento por client_id (Seguridad) |
| Denial of Service (sin contingencia) | Infraestructura | Arquitectura en nube con redundancia |
| Elevation of Privilege (sin MFA) | Aplicaciones | MFA cuentas admin (Seguridad) |

### Cómo la normatividad impone restricciones transversales (Taller 6)

El análisis de cumplimiento normativo (Taller 6) identificó 9 brechas críticas que se reflejan en la arquitectura integrada como restricciones de diseño no negociables:

- **Ley 1581/2012 (Habeas Data):** La entidad EMPLEADO debe incluir un campo de consentimiento de tratamiento de datos. El proceso de "cargar lista de empleados" debe verificar dicho consentimiento antes de procesar.
- **ISO 27001 – Criptografía:** El *Sistema de Notificaciones* debe enviar informes cifrados (no como Excel plano por Gmail personal). Esto impacta el diseño de la Vista de Aplicaciones.
- **ISO 27001 – Gestión de incidentes:** El *Logging de Auditoría* debe ser suficiente para responder dentro del plazo legal de 15 días ante una brecha de datos. Esto impacta el diseño de la Vista de Infraestructura (retención de logs).
- **Contratos con clientes:** La Vista de Negocio debe incluir un proceso de firma de acuerdos de procesamiento de datos con cada empresa cliente, aún no modelado en el BPMN actual.

---

## Diagrama final entregado

> Ver: `tablero-integrado-cliente.drawio` (abrir en https://app.diagrams.net)

El diagrama muestra los cinco swimlanes (Negocio, Información, Aplicaciones, Infraestructura, Seguridad) con flechas de trazabilidad:
- Flechas sólidas naranjas: Negocio → Aplicaciones (soporta / implementa)
- Flechas sólidas verdes: Aplicaciones → Información (accede / escribe) y Aplicaciones → Infraestructura (desplegado en)
- Flechas punteadas rojas: Seguridad → todas las capas (protege / controla / registra)

---

## Tabla de componentes y actores

| Elemento | Vista | Tipo | Descripción |
|----------|-------|------|-------------|
| Coordinadora de Capacitaciones | Negocio | Actor | Gestiona el ciclo completo de evaluaciones |
| Asesora de Seguimiento | Negocio | Actor | Monitorea cumplimiento por empresa cliente |
| Contacto RRHH | Negocio | Actor externo | Recibe informes y gestiona empleados de su empresa |
| Empleado | Negocio | Actor externo | Completa evaluaciones post-capacitación |
| Cargar Lista de Empleados | Negocio | Proceso | Carga y valida la lista de empleados por empresa |
| Enviar Invitación con Link Único | Negocio | Proceso | Genera token único por empleado y envía invitación |
| Calificación Automática | Negocio | Proceso | Aplica umbral 100% y registra resultado |
| Generar Informe Diario | Negocio | Proceso | Job programado que genera reporte por empresa |
| CLIENTE | Información | Entidad | Empresa cliente con su contacto RRHH |
| EMPLEADO | Información | Entidad | Persona que toma la evaluación, ligada a un CLIENTE |
| EVALUACION | Información | Entidad | Instrumento de evaluación con umbral de aprobación |
| RESPUESTA_EMPLEADO | Información | Entidad | Registro de respuesta individual con flag `correcto` |
| INFORME_DIARIO | Información | Entidad | Resumen de estado de cumplimiento por empresa y fecha |
| Plataforma de Evaluaciones | Aplicaciones | Sistema | Portal web con acceso por link único tokenizado |
| Motor de Calificación | Aplicaciones | Sistema | Lógica de calificación automática (100% threshold) |
| Generador de Informes Diarios | Aplicaciones | Sistema | Scheduled job que produce informes por empresa |
| Sistema de Notificaciones | Aplicaciones | Sistema | Envío automatizado de invitaciones y recordatorios |
| Zona Suporta | Infraestructura | Nodo | PC de la coordinadora y herramientas actuales |
| Zona Nube (Google Workspace) | Infraestructura | Nodo | Forms, Drive, Gmail; sin SLA garantizado actualmente |
| Zona Cliente | Infraestructura | Nodo | Dispositivos de HR y empleados |
| Links Únicos por Empleado | Seguridad | Control | Token de acceso único, previene respuestas duplicadas |
| Aislamiento por client_id | Seguridad | Control | Todos los queries filtran por client_id (multi-tenant) |
| MFA + Cifrado de Informes | Seguridad | Control | MFA en cuentas admin; informes enviados cifrados |
| Logging de Auditoría | Seguridad | Control | Registro de envíos y accesos para cumplimiento legal |

---

## Investigación complementaria

### Tema investigado: Documentación de vistas arquitectónicas en empresas EdTech y SaaS de evaluación

En empresas de tecnología educativa (EdTech) y SaaS de gestión de evaluaciones, la integración de vistas arquitectónicas es una práctica establecida para garantizar coherencia entre los objetivos de negocio y la implementación técnica. Se investigaron los siguientes enfoques:

**1. Modelo C4 aplicado a SaaS educativo**

Brown (2018) propone el modelo C4 (Context, Container, Component, Code) como un estándar ligero para documentar arquitecturas de software. En el contexto de Suporta, el tablero de integración desarrollado en este taller corresponde al nivel "Container" del C4, donde se muestran los sistemas de software y sus relaciones con los usuarios y sistemas externos. Coursera, por ejemplo, documenta su plataforma en cuatro niveles C4, lo que permite que equipos de negocio entiendan el diagrama de contexto sin necesidad de conocimientos técnicos [1].

**2. TOGAF y el Repositorio de Arquitectura**

TOGAF (The Open Group Architecture Framework) propone mantener un repositorio de arquitectura donde todas las vistas (negocio, datos, aplicación, tecnología) estén interconectadas y referenciadas. La integración de vistas del Taller 7 sigue el mismo principio: cada vista referencia artefactos de las demás (por ejemplo, la vista de seguridad referencia componentes de la vista de aplicaciones). Empresas como Platzi han adoptado principios similares para documentar la evolución de su arquitectura de microservicios [2].

**3. Ejemplos reales de documentación integrada**

Khan Academy publica parcialmente su arquitectura técnica y muestra cómo sus vistas de proceso (flujos de aprendizaje), datos (entidades de progreso del estudiante), aplicaciones (servicios internos) e infraestructura (Google Cloud) se articulan en una arquitectura coherente. El principio clave es la **trazabilidad**: cada componente técnico puede rastrearse hasta un objetivo de negocio específico [3].

Para Suporta, la lección aplicable es que la trazabilidad entre el proceso BPMN y los componentes de aplicación (e.g., "calificación automática" → "Motor de Calificación") no solo es un artefacto de documentación, sino la base para la planificación del desarrollo y la priorización de funcionalidades.

---

## Reflexión crítica sobre la coherencia de la arquitectura

### Fortalezas de la arquitectura integrada

1. **Alta trazabilidad proceso-aplicación:** Cada proceso del BPMN tiene un componente de aplicación correspondiente. No hay funcionalidades "huérfanas" en la capa de aplicaciones.
2. **Modelo de información completo:** Las entidades del Taller 2 soportan todos los flujos de datos requeridos por los procesos de negocio.
3. **Seguridad por diseño:** Los controles STRIDE están integrados desde la capa de negocio (links únicos en el proceso de invitación) hasta la infraestructura (logging de auditoría).
4. **Coherencia normativa:** Las restricciones de Habeas Data e ISO 27001 impactan el diseño de la plataforma, no solo los procedimientos operativos.

### Brechas identificadas

1. **La Vista de Infraestructura es el eslabón más débil:** La infraestructura actual (Google Workspace sin SLA) no puede soportar la arquitectura objetivo. La integración expone que mientras las vistas de negocio y aplicaciones son robustas, la infraestructura actual crea un riesgo operativo significativo.
2. **Proceso de consentimiento no modelado:** El proceso BPMN no incluye la obtención del consentimiento de tratamiento de datos de los empleados (requerido por Ley 1581/2012). Esto es una brecha entre la Vista de Negocio y los requisitos normativos.
3. **Integración con sistema de capacitación no definida:** El proceso parte del supuesto de que las capacitaciones ya se realizaron, pero no hay un proceso modelado para recibir la notificación de que una capacitación finalizó. Esto es un gap en la Vista de Negocio.

### Hoja de ruta sugerida

| Prioridad | Acción | Vista afectada |
|-----------|--------|----------------|
| Crítica | Migrar infraestructura a Google Workspace Business o equivalente con SLA | Infraestructura |
| Crítica | Agregar proceso de consentimiento al BPMN y campo en entidad EMPLEADO | Negocio + Información |
| Alta | Implementar MFA en cuentas admin de Google Workspace | Seguridad |
| Alta | Cifrar informes enviados por correo (no Excel plano) | Seguridad + Aplicaciones |
| Media | Modelar proceso de integración con plataforma de capacitación | Negocio |
| Media | Definir retención de logs (mínimo 2 años para cumplimiento ISO 27001) | Infraestructura + Seguridad |

---

## Referencias

Ver archivo `referencias.md`.

---

_Este documento hace parte de la entrega del Taller 7 del curso AREM (Arquitectura Empresarial) - Universidad de La Sabana._
