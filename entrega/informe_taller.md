# 📄 Informe Técnico del Taller

## 🔖 Nombre del Taller
_Taller 2: Modelo de Información y Diagrama de Contexto_
link del diagrama de dominio:
https://dbdiagram.io/d/6999240bbd82f5fce25abad3
## 👥 Integrantes del equipo
- Juan Abril (Juan-Abril21)
- Bryam Diaz (bryamDigar)
- Jose Guzman (joseph8884)

## 🧠 Descripción general del trabajo
Durante la segunda parte del taller adaptamos el modelo de información y el diagrama de contexto al caso práctico asignado (procesos de pagaduría y consolidación de novedades). El objetivo fue identificar las entidades y flujos de datos necesarios para soportar la gestión de novedades, consolidación y notificación de alertas, priorizando claridad para posteriores implementaciones en pipelines de datos y herramientas de integración.

**Caso del cliente — Sistema de Alertas y Consolidación CPP (Pagos)**

- Problem Statement: el reporte de novedades de libranza (descuentos de nómina) es un proceso manual propenso a error humano; la falta de alertas preventivas genera riesgos de no recaudo cuando un analista omite cargar información a tiempo.
- Business Justification: mitigar riesgo operativo y financiero, asegurando que las novedades lleguen a las pagadurías según calendario.
- Scope (in-scope): sistema que consolide repositorios, valide fechas de corte parametrizables y envíe alertas automáticas (correo/dashboard) si falta información antes del vencimiento.
- Objectives: garantizar que el 100% de las novedades se reporten dentro de los tiempos establecidos.

El análisis AS-IS mostró que los analistas consultan manualmente múltiples repositorios y montan archivos según su memoria, lo que implica riesgo alto de omisión. En el benchmark consideramos soluciones RPA y ETL con monitoreo, así como prácticas de job scheduling y dashboards tipo "semáforo".

## 🔧 Proceso de desarrollo
Decisiones tomadas (tono de estudiante):

- Empezamos modelando las entidades transaccionales críticas: `PROCESO_CONSOLIDACION`, `DETALLE_NOVEDAD` y `REGISTRO_ALERTA`. Como el objetivo del cliente es evitar omisiones, priorizamos las tablas que soportan trazabilidad y generación de alertas.
- Separamos `REPOSITORIO_ORIGEN` y `PAGADURIA` para que las conexiones y parámetros por fuente queden centralizados; esto facilita la automatización de ingestas desde distintas fuentes (financiera, cooperativa) sin replicar metadatos.
- incluimos campos como `Fecha_Generacion`, `Fecha_Vencimiento` y `Fecha_Envio_Real` en los procesos para soportar reglas de corte y cuadros de control tipo "semáforo" que muestren pendientes y vencimientos.
- diseñamos `REGLA_CALENDARIO` para que el equipo de negocio pueda definir días de corte y anticipación de alertas sin tocar el modelo físico esto reduce la dependencia de cambios en código.
- usamos `integer`, `varchar`, `date`/`timestamp` para mantener compatibilidad con motores relacionales y herramientas ETL/RPA, pensando en integraciones con Airflow/ETL o herramientas RPA cuando sea necesario.
- el ERD y el diagrama de contexto se diseñaron en dbdiagram.io / draw.io para facilitar revisiones colaborativas y para que el equipo de data engineering pueda extraer el esquema inicial para pipelines.

## 🧩 Análisis del modelo propuesto
Cómo se estructura el modelo:

- El modelo central está orientado al flujo operativo: `REPOSITORIO_ORIGEN` → `DETALLE_NOVEDAD` → `PROCESO_CONSOLIDACION` → `REGISTRO_ALERTA`. Las entidades de soporte (`PAGADURIA`, `REGLA_CALENDARIO`) contienen la parametrización requerida por cada pagaduría.

Representación de necesidades del cliente:

- Dado el riesgo operacional identificado en el AS-IS, diseñamos la solución para maximizar trazabilidad y permitir la automatización de comprobaciones previas al envío (validaciones de corte, alertas anticipadas). El modelo permite construir dashboards del estado de cumplimiento (enviado / pendiente / vence hoy).

Supuestos principales:

- El proceso opera por lotes con ventanas de corte configurables, pero debe soportar ejecuciones incrementales si se requiere.
- Los orígenes proporcionan, al menos, identificadores por registro que permitan consolidación sin ambigüedad.
- La seguridad y el enmascaramiento se implementarán en la capa de ingesta o almacenamiento según políticas del cliente.

## 📈 Diagrama final entregado
El diagrama final está incluido en los archivos de la entrega y refleja las decisiones descritas en las secciones anteriores. Se modeló también un diagrama de contexto donde aparecen: fuentes externas (Financiera, Cooperativa), el Consolidador, el Motor de Alertas y el Dashboard/Notificador.

## 📋 Tabla de actores, entidades o componentes (si aplica)

| Nombre del elemento | Tipo | Descripción | Responsable |
|---------------------|------|-------------|-------------|
| Usuario / Analista  | Actor | Persona que revisa y valida novedades; recibe alertas | Equipo de negocio |
| Pagaduría           | Componente | Sistema administrativo que parametriza reglas y recibe consolidado | Cliente / IT |
| Repositorio_Origen  | Componente | Fuente de datos (archivos, APIs) donde se detectan novedades | Integraciones |
| Motor_Alertas       | Componente lógico | Servicio que evalúa reglas y genera `REGISTRO_ALERTA` | Plataforma de notificaciones |
| Consolidador        | Proceso | Módulo que agrupa `DETALLE_NOVEDAD` en `PROCESO_CONSOLIDACION` | Data Engineering |


_Este documento hace parte de la entrega del taller 2 del curso AREM (Arquitectura Empresarial) - Universidad de La Sabana._
