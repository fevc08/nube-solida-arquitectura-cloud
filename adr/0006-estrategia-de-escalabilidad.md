# ADR-0006: Estrategia de escalabilidad y autoescalado

**Estado:** Aceptado
**Fecha:** 2026-07-03
**Decisores:** Equipo de Arquitectura - Área de Infraestructura y Seguridad

## Contexto

"Problemas de escalabilidad" es uno de los tres problemas explícitos de la
situación inicial. Con los componentes ya definidos (ADR-0001, ADR-0003), es
necesario decidir el enfoque de escalado (horizontal, vertical o mixto) para
cada tipo de componente, en particular para el componente crítico (RDS), cuya
naturaleza en nube privada limita su elasticidad frente al resto de la
arquitectura.

## Decisión

Se adopta **escalabilidad horizontal automática** como estrategia por defecto
para todos los componentes en nube pública (ECS, Lambda, DynamoDB, S3), y
**escalabilidad vertical planificada, con réplicas de lectura opcionales**, para
Amazon RDS en nube privada.

## Alternativas consideradas

| Opción | Ventajas | Desventajas |
|---|---|---|
| **A. Escalabilidad vertical uniforme en toda la arquitectura** | Simplicidad conceptual (un solo enfoque). | No aprovecha la elasticidad nativa de los servicios gestionados en nube pública; tiene límite físico de expansión; contradice el objetivo de escalabilidad del proyecto. |
| **B. Escalabilidad horizontal también para RDS (múltiples instancias activas)** | Máxima elasticidad en todos los componentes. | Complejiza significativamente la gestión de consistencia transaccional del dato crítico; mayor costo y riesgo operativo, no justificado por el volumen actual del proyecto. |
| **C. Horizontal por defecto + vertical planificado solo para RDS (elegida)** | Aprovecha la elasticidad nativa donde es segura y sencilla (nube pública); mantiene RDS predecible y controlado, coherente con su rol de dato crítico. | RDS requiere dimensionamiento manual proactivo (menos automático que el resto de los componentes). |

## Justificación

- Los servicios gestionados en nube pública (ECS, Lambda, DynamoDB, S3) están
  diseñados nativamente para escalado horizontal automático; no aprovecharlo
  sería contradecir la razón misma de haberlos elegido como PaaS/FaaS
  (ADR-0001).
- RDS, al alojar el dato crítico del negocio en nube privada (ADR-0002),
  prioriza consistencia y control sobre elasticidad total: un escalado vertical
  planificado, complementado con réplicas de lectura si el volumen de
  consultas lo justifica, es suficiente para el perfil de carga esperado sin
  introducir la complejidad de un clúster activo-activo.
- Este enfoque diferenciado es consistente con el criterio de "protección y
  tratamiento proporcional al riesgo/criticidad" ya aplicado en ADR-0002 y
  ADR-0005.

## Consecuencias

**Positivas**
- Optimización de costos: se paga solo por los recursos usados en cada momento en los componentes de nube pública.
- Capacidad de absorber picos de demanda (ej. campañas, cierres de mes) sin intervención manual en la mayoría de los componentes.

**Negativas / trade-offs asumidos**
- El componente RDS requiere monitoreo y dimensionamiento proactivo por parte del equipo de Infraestructura, en lugar de escalar automáticamente.

**Riesgos y mitigación**
- *Riesgo*: RDS se convierte en cuello de botella si el volumen de escritura crece más rápido de lo previsto. *Mitigación*: monitoreo continuo vía CloudWatch con alarmas de utilización, y plan de revisión trimestral de capacidad.

## Referencias
- Manual L5 - Principales atributos de calidad en una arquitectura en la nube.
- [atributos-calidad.md](../docs/05-atributos-calidad/atributos-calidad.md)
- [ADR-0001: Modelos de servicio por componente.](../adr/0001-modelos-de-servicio-por-componente.md)
