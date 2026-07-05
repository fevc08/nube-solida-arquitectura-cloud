# Principios Fundamentales de Diseño Arquitectónico

**Proyecto:** Nube Sólida
**Módulo:** M3 - Fundamentos de la Arquitectura Cloud
**Lección:** 4 - Principios fundamentales de diseño de una arquitectura

## 1. Objetivo

Aplicar los principios fundamentales de diseño para la nube (modularidad,
desacoplamiento, resiliencia, elasticidad y seguridad) a la arquitectura de
"Nube Sólida", consolidando las decisiones tomadas en las Lecciones 2 y 3
(ADR-0001 y ADR-0002) en un diseño estructurado.

## 2. Patrón arquitectónico base

Antes de aplicar los principios, se define el patrón arquitectónico del backend,
ya que condiciona cómo se implementa la modularidad:

| Patrón evaluado | Decisión |
|---|---|
| Monolítico | Descartado: dificulta el escalado independiente de funcionalidades y contradice el objetivo de escalabilidad. |
| Microservicios | **Adoptado** para el backend, con despliegue en contenedores sobre el servicio PaaS definido en ADR-0001. |
| Serverless (FaaS) | **Adoptado** específicamente para el componente de Procesamiento Asíncrono (ADR-0001), no para el backend completo. |
| Basado en contenedores | **Adoptado** como mecanismo de empaquetado y portabilidad de los microservicios del backend. |

**Resultado**: arquitectura **cliente-servidor** con un backend compuesto por
**microservicios en contenedores** (para la lógica de negocio principal) y
**funciones serverless** (para el procesamiento orientado a eventos), consistente
con los modelos de servicio ya definidos.

## 3. Aplicación de los 5 principios de diseño para la nube

### 3.1 Modularidad

| Mecanismo | Aplicación en Nube Sólida |
|---|---|
| División en componentes pequeños e independientes | El backend se divide en microservicios por dominio funcional (ej: servicio de negocio, servicio de reportes, servicio de notificaciones). |
| Microservicios y contenedores | Cada microservicio se empaqueta en un contenedor independiente, desplegado sobre el servicio PaaS. |
| Flexibilidad en la implementación | Un cambio en el servicio de reportes no requiere redesplegar el resto del backend. |

### 3.2 Desacoplamiento

| Mecanismo | Aplicación en Nube Sólida |
|---|---|
| Separación de responsabilidades | El cliente web solo conoce la API pública (vía API Gateway); no conoce la implementación interna del backend. |
| Comunicación asíncrona vía colas de mensajes | La Cola de Mensajes (ADR-0001) desacopla el backend del componente de Procesamiento Asíncrono (FaaS): el backend publica un evento y continúa, sin esperar el procesamiento. |
| Mayor resiliencia ante fallos | Si el componente de Procesamiento Asíncrono falla temporalmente, los eventos quedan en la cola y se procesan al restablecerse, sin afectar al backend. |

### 3.3 Resiliencia

| Mecanismo | Aplicación en Nube Sólida |
|---|---|
| Redundancia | Múltiples instancias del backend en al menos 2 zonas de disponibilidad. |
| Balanceo de carga | Un Load Balancer distribuye tráfico entre las instancias del backend. |
| Aislamiento de fallos | Arquitectura de microservicios evita que la caída de un servicio (ej: reportes) derribe el sistema completo. |
| Enlace híbrido redundante | El enlace entre nube pública y privada (ADR-0002) cuenta con ruta de respaldo para evitar punto único de fallo. |

*(El detalle completo de resiliencia, con circuit breakers y recuperación
automática, se desarrolla en la Lección 5.)*

### 3.4 Elasticidad

| Mecanismo | Aplicación en Nube Sólida |
|---|---|
| Autoescalado del backend | Las instancias/contenedores del backend escalan automáticamente según CPU/tráfico. |
| Escalado nativo de FaaS | El componente de Procesamiento Asíncrono escala a cero cuando no hay eventos y crece automáticamente ante picos (ej: campaña con muchas notificaciones). |
| Escalado de base de datos NoSQL | Escalado horizontal automático provisto por el servicio PaaS gestionado. |

### 3.5 Seguridad

| Mecanismo | Aplicación en Nube Sólida |
|---|---|
| Autenticación y autorización | Gestionadas por el componente SaaS de Identidad (ADR-0001), vía OAuth/OpenID Connect. |
| Segmentación de red | El segmento de nube privada (base de datos relacional) solo es alcanzable desde el backend, vía VPN (ADR-0002). |
| Cifrado | En tránsito (TLS) y en reposo en todos los componentes con datos. |
| Principio de mínimo privilegio | Aplicado vía IAM en todos los servicios gestionados. |

*(El detalle completo de seguridad se desarrolla en la Lección 5.)*

## 4. Conclusión

La combinación de arquitectura cliente-servidor, microservicios en contenedores
y funciones serverless permite aplicar de forma consistente los cinco principios
de diseño para la nube sobre los componentes y modelos ya definidos en las
Lecciones 2 y 3. Esta base estructural se detalla componente por componente en
[arquitectura-cliente-servidor.md](arquitectura-cliente-servidor.md) y se
formaliza en **[ADR-0003](../../adr/0003-arquitectura-cliente-servidor-y-principios-diseno.md)**.

## Referencias
- Manual L4 - Principios fundamentales de diseño de una arquitectura.
- Fowler, M. (2019). *Microservices: A definition of this new architectural term*.
- Bass, L., Clements, P., & Kazman, R. (2012). *Software architecture in practice* (3rd ed.).
