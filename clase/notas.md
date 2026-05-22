# Registro de Trabajo en Clase - Taller 7

## Fecha de la sesión
22 de mayo de 2026

## Integrantes presentes
- Juan Gómez (juangomepere)
- Rodriguez
- Sosa

---

## Actividades realizadas en clase

Durante la sesión se trabajó con el caso base de referencia **FarmApp**, una cadena nacional de farmacias con plataforma e-commerce integrada. Las actividades realizadas fueron:

- **Análisis del caso:** El equipo revisó la descripción de FarmApp y sus cinco vistas arquitectónicas (negocio, información, aplicaciones, infraestructura y seguridad).
- **Organización del tablero:** Se construyó en draw.io un tablero visual con cinco swimlanes horizontales, uno por cada vista, colocando los componentes correspondientes en cada capa.
- **Identificación de relaciones entre capas:** Se trazaron flechas de trazabilidad para mostrar cómo cada proceso de negocio es soportado por una o más aplicaciones, cómo las aplicaciones acceden a las entidades de información, cómo se despliegan en la infraestructura y cómo los controles de seguridad son transversales.
- **Discusión de decisiones de integración:** El equipo analizó qué relaciones son las más críticas para la coherencia de la arquitectura:
  - Los procesos de Compra Online y Prescripción Médica son los de mayor impacto y requieren integración con la App Móvil, el E-Commerce y el POS.
  - La entidad Pedido es la más central desde el punto de vista de información, conectando múltiples procesos y aplicaciones.
  - La Nube Híbrida es el componente de infraestructura más crítico porque soporta las aplicaciones de mayor tráfico.
  - Los controles de seguridad son transversales y aplican a todas las capas (no solo a una).

### Herramientas usadas
- draw.io (app.diagrams.net) para el tablero digital
- GitHub para control de versiones

---

## Boceto inicial del modelo

El tablero se estructuró como una matriz de vistas × componentes:

```
+---------------------+---------------------------+---------------------------+---------------------------+
| VISTA DE NEGOCIO    | Compra Online             | Prescripción Médica       | Despacho / Entrega        |
+---------------------+---------------------------+---------------------------+---------------------------+
| VISTA DE INFORMACIÓN| Entidad: Producto         | Entidad: Cliente          | Entidad: Pedido           | Entidad: Descuento
+---------------------+---------------------------+---------------------------+---------------------------+
| VISTA DE APLICACIONES| App Móvil                | Plataforma E-Commerce     | Sistema POS               | CRM
+---------------------+---------------------------+---------------------------+---------------------------+
| VISTA INFRAESTRUCTURA| Servidores Regionales    | Nube Híbrida (AWS/Azure)  | Base de Datos Replicada   |
+---------------------+---------------------------+---------------------------+---------------------------+
| VISTA DE SEGURIDAD  | Control Acceso por Rol    | Cifrado de Datos          | Monitoreo de Fraude       |
+---------------------+---------------------------+---------------------------+---------------------------+
```

Las relaciones clave identificadas:
- Compra Online → App Móvil / E-Commerce (soporta)
- Prescripción → POS (soporta)
- Despacho → CRM (soporta)
- App Móvil → Entidad Pedido, Entidad Cliente (accede)
- E-Commerce → Entidad Producto, Entidad Descuento (accede)
- App Móvil / E-Commerce → Nube Híbrida (desplegado en)
- POS → Servidores Regionales (desplegado en)
- Control Acceso por Rol → App Móvil (controla)
- Cifrado de Datos → Base de Datos Replicada (protege)
- Monitoreo de Fraude → E-Commerce (monitorea)

> Ver diagrama completo en: `tablero-farmapp.drawio`

---

## Tareas definidas para complementar el taller

| Tarea asignada | Responsable | Fecha estimada |
|----------------|-------------|----------------|
| Refinar tablero-farmapp.drawio con feedback del docente | Juan Gómez | 23/05/2026 |
| Realizar tablero integrado de Suporta S.A.S (entrega/) | Rodriguez | 24/05/2026 |
| Redacción del informe.md (Parte 2) | Sosa | 25/05/2026 |
| Compilar referencias.md | Juan Gómez | 25/05/2026 |

---

_Este documento resume el trabajo colaborativo realizado durante la sesión del Taller 7 en el curso AREM - Universidad de La Sabana._
