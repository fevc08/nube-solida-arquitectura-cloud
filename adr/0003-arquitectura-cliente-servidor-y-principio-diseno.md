# ADR-0003: Adopción de arquitectura cliente-servidor con backend en microservicios y componentes serverless

**Estado:** Aceptado
**Fecha:** 2026-07-02
**Decisores:** Equipo de Arquitectura - Área de Infraestructura y Seguridad

## Contexto

Con los modelos de servicio (ADR-0001) y de implementación (ADR-0002) ya
definidos, es necesario formalizar el patrón arquitectónico estructural del
backend de "Nube Sólida", de forma que soporte los principios de modularidad,
desacoplamiento, resiliencia, elasticidad y seguridad exigidos por el proyecto.

## Decisión

Se adopta una **arquitectura cliente-servidor**, donde el servidor se implementa
como un conjunto de **microservicios desplegados en contenedores** (para la
lógica de negocio principal) combinados con **funciones serverless (FaaS)** para
el procesamiento asíncrono orientado a eventos. Se incorporan un **API Gateway**
como punto único de entrada y un **Load Balancer** para distribución de carga.

## Alternativas consideradas

| Opción | Ventajas | Desventajas |
|---|---|---|
| **A. Arquitectura monolítica** | Simplicidad de desarrollo y despliegue inicial; menor latencia interna. | Dificulta el escalado independiente de funcionalidades; un fallo puede derribar todo el sistema; contradice directamente el objetivo de escalabilidad y resiliencia del proyecto. |
| **B. Microservicios sin contenedores (VMs independientes por servicio)** | Aislamiento de fallos similar a la opción elegida. | Mayor costo y tiempo de aprovisionamiento; menor portabilidad; no aprovecha las ventajas de eficiencia de los contenedores. |
| **C. Microservicios en contenedores + FaaS para eventos (elegida)** | Modularidad, escalado independiente, aislamiento de fallos, y costo-eficiencia (FaaS solo cobra por ejecución en cargas esporádicas). | Mayor complejidad de orquestación y observabilidad que un monolito; requiere disciplina de diseño (contratos entre servicios). |

## Justificación

1. **Modularidad y escalabilidad independiente**: permite escalar únicamente el
   microservicio bajo carga (ej. el de reportes en cierre de mes) sin
   sobredimensionar el resto del sistema.
2. **Resiliencia**: el aislamiento entre microservicios evita que el fallo de un
   componente (ej. notificaciones) derribe la operación completa del sistema,
   requerimiento explícito de la situación inicial del proyecto.
3. **Coherencia con ADR-0001**: el uso de contenedores sobre el servicio PaaS ya
   definido, y de FaaS para el componente de Procesamiento Asíncrono, es una
   extensión natural de las decisiones de modelo de servicio ya tomadas, sin
   introducir nuevas tecnologías no contempladas.
4. **Desacoplamiento vía API Gateway**: el cliente web nunca se comunica
   directamente con los microservicios internos, lo que permite evolucionar el
   backend sin impactar al cliente.

## Consecuencias

**Positivas**
- Escalado y despliegue independiente por dominio funcional.
- Mayor tolerancia a fallos parciales del sistema.
- Costo optimizado en cargas esporádicas gracias a FaaS.

**Negativas / trade-offs asumidos**
- Mayor complejidad de observabilidad y trazabilidad distribuida (mitigado con
  el componente de Monitoreo definido en ADR-0001).
- Mayor esfuerzo de diseño inicial (definición de contratos/APIs entre
  microservicios) comparado con un monolito.

**Riesgos y mitigación**
- *Riesgo*: latencia adicional por comunicación entre microservicios.
  *Mitigación*: comunicación síncrona solo donde es estrictamente necesaria;
  uso de la Cola de Mensajes para flujos que toleran procesamiento asíncrono.
- *Riesgo*: complejidad operativa de gestionar múltiples contenedores.
  *Mitigación*: uso de orquestación gestionada (parte del servicio PaaS ya
  seleccionado), evitando administración manual de infraestructura.

## Referencias
- Manual L4 - Principios fundamentales de diseño de una arquitectura.
- [principios-diseno.md](../docs/04-diseno-arquitectonico/principios-diseno.md)
- [arquitectura-cliente-servidor.md](../docs/04-diseno-arquitectonico/arquitectura-cliente-servidor.md)
- [ADR-0001: Modelos de servicio por componente.](../adr/0001-modelos-de-servicio-por-componente.md)
- [ADR-0002: Modelo de implementación en la nube.](../adr/0002-modelo-de-implementacion-nube.md)
