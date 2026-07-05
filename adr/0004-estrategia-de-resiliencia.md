# ADR-0004: Estrategia de resiliencia y recuperación ante fallos

**Estado:** Aceptado
**Fecha:** 2026-07-03
**Decisores:** Equipo de Arquitectura - Área de Infraestructura y Seguridad

## Contexto

La situación inicial del proyecto identifica "baja resiliencia ante fallos" como
uno de los tres problemas centrales a resolver. Con la arquitectura ya definida
(ADR-0001, ADR-0002, ADR-0003), es necesario decidir el nivel y mecanismo de
redundancia a aplicar, en particular para el componente crítico en nube privada
(RDS), dado que un exceso de redundancia implica sobrecosto y un defecto
compromete la continuidad del negocio.

## Decisión

Se adopta redundancia **Multi-AZ con failover automático** para Amazon RDS
(componente crítico), redundancia nativa del proveedor para los servicios
gestionados en nube pública (DynamoDB, S3), y un patrón de **cola de mensajes
como buffer de resiliencia** (SQS) entre el backend y el procesamiento
asíncrono (Lambda).

## Alternativas consideradas

| Opción | Ventajas | Desventajas |
|---|---|---|
| **A. Single-AZ para RDS (sin redundancia adicional)** | Menor costo. | Punto único de fallo sobre el dato crítico del negocio; inaceptable dado el requisito explícito de resiliencia del proyecto. |
| **B. Multi-región para RDS** | Máxima resiliencia posible, incluso ante caída de una región completa. | Costo y complejidad significativamente mayores; latencia de replicación entre regiones; no justificado para el volumen/criticidad actual del proyecto. |
| **C. Multi-AZ con failover automático (elegida)** | Balance adecuado entre costo y resiliencia; failover automático sin intervención manual; cumple el requisito de continuidad de negocio sin sobre-ingeniería. | No protege ante la caída de una región completa (riesgo aceptado dado el alcance actual del proyecto). |

## Justificación

- Multi-AZ resuelve el escenario de fallo más probable (caída de una zona de
  disponibilidad) sin incurrir en la complejidad y costo de una estrategia
  multi-región, que el manual L1 identifica como generadora de "complejidad
  operativa y costos incrementales" cuando no está claramente justificada.
- El uso de SQS como buffer entre ECS y Lambda garantiza que una interrupción
  temporal del procesamiento asíncrono no implique pérdida de eventos,
  reforzando la resiliencia del sistema completo sin componentes adicionales.

## Consecuencias

**Positivas**
- Continuidad operativa ante el escenario de fallo más común (caída de AZ).
- Recuperación automática sin intervención manual del equipo de Infraestructura.

**Negativas / trade-offs asumidos**
- No hay protección ante la caída de una región completa de AWS (riesgo residual aceptado).

**Riesgos y mitigación**
- *Riesgo*: caída de región completa. *Mitigación*: backups exportables a otra
  región como plan de contingencia manual, sin necesidad de mantener
  infraestructura activa en una segunda región de forma permanente.

## Referencias
- Manual L5 - Principales atributos de calidad en una arquitectura en la nube.
- [Atributos de Calidad](../docs/05-atributos-calidad/atributos-calidad.md)
