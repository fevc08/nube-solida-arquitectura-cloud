# ADR-0002: Selección del modelo de implementación en la nube

**Estado:** Aceptado
**Fecha:** 2026-07-02
**Decisores:** Equipo de Arquitectura - Área de Infraestructura y Seguridad

## Contexto

Definidos los modelos de servicio por componente (ADR-0001), es necesario decidir
el modelo de implementación de la infraestructura cloud subyacente. La
organización es una empresa de tecnología que busca resolver tres problemas
concretos: escalabilidad limitada, costos elevados y baja resiliencia ante
fallos. Adicionalmente, se identifica que uno de los componentes, la base de
datos relacional que almacena información transaccional del negocio, constituye
**información crítica** que requiere un nivel de control y aislamiento superior
al del resto de la arquitectura.

## Decisión

Se adopta un **modelo de implementación híbrido**: nube privada dedicada para la
base de datos relacional (información crítica), y nube pública (AWS) para el
resto de los componentes de la arquitectura.

## Alternativas consideradas

| Opción | Ventajas | Desventajas |
|---|---|---|
| **A. Nube pública pura** | Menor costo; máxima escalabilidad; despliegue más simple (un solo entorno). | No satisface el requisito de aislamiento y control adicional que el área de Seguridad exige sobre la información transaccional crítica del negocio. |
| **B. Nube privada pura** | Máximo control y aislamiento en toda la arquitectura. | Altos costos de implementación y mantenimiento; escalabilidad limitada; contradice directamente el objetivo de reducción de costos para los componentes que no manejan información crítica. |
| **C. Nube híbrida (elegida)** | Aísla específicamente el dato crítico sin sacrificar costo/escalabilidad en el resto de la arquitectura; equilibrio directo entre seguridad y eficiencia. | Mayor complejidad de integración (conectividad segura entre entornos); costos adicionales de enlace dedicado/VPN y monitoreo cruzado; dependencia de más de un proveedor/entorno. |

## Justificación

1. **Protección proporcional al riesgo**: solo el componente que efectivamente
   maneja información crítica (base de datos relacional) se traslada a un
   entorno privado; el resto de los componentes, sin ese nivel de sensibilidad,
   permanecen en nube pública para no incurrir en sobrecostos innecesarios.
2. **Coherencia con ADR-0001**: la base de datos relacional se mantiene como
   PaaS (base de datos administrada); lo que cambia es únicamente el entorno de
   infraestructura subyacente sobre el que corre ese PaaS (privado en lugar de
   público), sin alterar la decisión de modelo de servicio ya tomada.
3. **Cumplimiento del mandato del área solicitante**: la Unidad solicitante es
   específicamente Infraestructura **y Seguridad**; un modelo híbrido demuestra
   de forma explícita cómo se protege la información crítica, más allá de
   controles a nivel de aplicación.
4. **Costo-eficiencia**: se minimiza la superficie de infraestructura privada
   (costosa de mantener) al mínimo estrictamente necesario, en lugar de aplicar
   ese sobrecosto a toda la arquitectura.

## Consecuencias

**Positivas**
- Mayor control y aislamiento específicamente sobre el activo más sensible del
  sistema (datos transaccionales del negocio).
- Se conservan los beneficios de costo y escalabilidad de la nube pública para
  el resto de los componentes.
- Facilita el cumplimiento de futuras exigencias regulatorias sobre el dato
  crítico, sin rediseñar toda la arquitectura.

**Negativas / trade-offs asumidos**
- Mayor complejidad operativa: se administran dos entornos en lugar de uno.
- Costos adicionales de conectividad (VPN/enlace dedicado) y de las
  herramientas de monitoreo e integración cruzada entre ambos entornos.
- Dependencia de múltiples proveedores/entornos, lo que puede generar desafíos
  de compatibilidad a futuro.

**Riesgos y mitigación**
- *Riesgo*: el enlace entre nube pública y privada se convierte en un punto
  único de fallo. *Mitigación*: enlace redundante y monitoreo activo de
  conectividad (se detalla en la Lección 5 - resiliencia).
- *Riesgo*: mayor superficie de ataque por gestionar dos entornos con
  configuraciones de seguridad distintas. *Mitigación*: políticas de seguridad
  unificadas (IAM, cifrado) aplicadas de forma consistente en ambos entornos.
- *Riesgo*: incremento de costos operativos por duplicidad de herramientas de
  gestión. *Mitigación*: uso de herramientas de observabilidad centralizada
  (SaaS ya definido en ADR-0001) con visibilidad sobre ambos entornos.

## Referencias
- Manual L3 - Modelos de implementación en la nube.
- [Análisis del Modelo de Implementación en la Nube](../docs/03-modelos-implementacion/analisis-modelo-implementacion.md)
- [ADR-0001: Asignación de modelos de servicio por componente](../adr/0001-modelos-de-servicio-por-componente.md)