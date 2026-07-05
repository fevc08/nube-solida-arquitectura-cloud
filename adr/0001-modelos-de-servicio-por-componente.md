# ADR-0001: Asignación de modelos de servicio por componente

**Estado:** Aceptado
**Fecha:** 2026-07-02
**Decisores:** Equipo de Arquitectura - Área de Infraestructura y Seguridad

## Contexto

La organización requiere migrar "Nube Sólida", una aplicación web empresarial
(frontend + backend + persistencia mixta + procesamiento asíncrono), resolviendo
los problemas de escalabilidad, costos elevados y baja resiliencia identificados en
la situación inicial del proyecto. Es necesario decidir, para cada componente de la
solución, qué modelo de servicio en la nube (IaaS, PaaS, SaaS o FaaS) utilizar.

## Decisión

Se adopta un **modelo de servicio híbrido por componente**: PaaS como modelo por
defecto para reducir carga operativa, IaaS únicamente para el almacenamiento de
archivos (donde se requiere control granular), FaaS para el procesamiento
asíncrono/basado en eventos, y SaaS para funcionalidades no diferenciadoras del
negocio (identidad y monitoreo).

Ver el detalle completo por componente en
[analisis-modelos-servicio.md](../docs/02-modelos-servicio/analisis-modelos-servicio.md).

## Alternativas consideradas

| Opción | Ventajas | Desventajas |
|---|---|---|
| **A. Todo en IaaS** (VMs propias para cada componente) | Control total del entorno; sin dependencia de servicios gestionados propietarios. | Alta carga operativa (parchado, escalado manual); contradice el objetivo de reducir costos y mejorar resiliencia; requiere equipo de TI más numeroso. |
| **B. Todo en SaaS/PaaS** (delegar el máximo posible a terceros) | Mínima carga operativa; despliegue muy rápido. | Pérdida de control sobre almacenamiento de archivos con requisitos de retención propios; riesgo de vendor lock-in generalizado; menor flexibilidad para lógica muy específica del negocio. |
| **C. Híbrido por componente (elegida)** | Balancea control operativo y velocidad de entrega; cada componente usa el modelo que mejor se adapta a su necesidad real. | Requiere gestionar más de un tipo de servicio (mayor curva de aprendizaje inicial para el equipo). |

## Justificación

- **PaaS por defecto** (API, bases de datos, mensajería, cliente web): la organización
  busca reducir costos operativos y mejorar la velocidad de entrega; PaaS permite
  esto sin renunciar a control sobre la lógica de negocio.
- **FaaS para procesamiento asíncrono**: el patrón de carga (notificaciones, generación
  de reportes, procesamiento de archivos) es intermitente y basado en eventos —
  el modelo de pago por ejecución de FaaS es el más costo-eficiente para este patrón.
- **IaaS solo para almacenamiento de archivos**: se requiere control granular de
  políticas de ciclo de vida y permisos que un servicio SaaS no permitiría
  personalizar al nivel necesario.
- **SaaS para identidad y monitoreo**: son funcionalidades estándar, de alto riesgo si
  se implementan mal (seguridad) y de bajo valor diferencial si se desarrollan
  internamente; existen soluciones de mercado maduras y certificadas.

## Consecuencias

**Positivas**
- Reducción de la carga operativa del equipo de Infraestructura.
- Menor tiempo de time-to-market para nuevas funcionalidades.
- Costos alineados al consumo real (pago por uso en PaaS/FaaS).

**Negativas / trade-offs asumidos**
- Cierto grado de dependencia (lock-in) del proveedor de nube elegido en los
  componentes PaaS/FaaS.
- Menor personalización de infraestructura subyacente en los componentes PaaS.

**Riesgos y mitigación**
- *Riesgo*: vendor lock-in. *Mitigación*: uso de estándares abiertos (REST, SQL
  estándar) en la capa de aplicación para facilitar una eventual migración.
- *Riesgo*: dependencia de disponibilidad de terceros SaaS (identidad, monitoreo).
  *Mitigación*: exigir SLA contractual y plan de contingencia documentado en la
  Lección 5 (atributos de calidad).

## Referencias
- Manual L2 - Modelos de servicio en la nube.
- [Análisis de Modelos de Servicio en la Nube](../docs/02-modelos-servicio/analisis-modelos-servicio.md)
