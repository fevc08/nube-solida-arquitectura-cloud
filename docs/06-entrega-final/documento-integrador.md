# Nube Sólida: Documento Integrador de Arquitectura Cloud

**Proyecto:** Nube Sólida
**Módulo:** M3 - Fundamentos de la Arquitectura Cloud
**Tipo de entrega:** Documento integrador final

---

## 1. Resumen ejecutivo

Este documento consolida el diseño conceptual de arquitectura en la nube desarrollado para "Nube Sólida", en respuesta a la solicitud del Área de Infraestructura y Seguridad de una empresa de tecnología que atraviesa un proceso de migración a la nube.

La organización presentaba tres problemas centrales: problemas de escalabilidad, costos operativos elevados y baja resiliencia ante fallos. El diseño propuesto resuelve estos tres problemas mediante:

- Un modelo de servicio diferenciado por componente (PaaS, SaaS, FaaS, IaaS).
- Un modelo de implementación híbrido, que aísla la información crítica del negocio en un segmento de nube privada, manteniendo el resto de la arquitectura en nube pública.
- Una arquitectura cliente-servidor basada en microservicios y funciones serverless.
- Mecanismos concretos de resiliencia, seguridad y escalabilidad aplicados sobre cada componente real de la solución.

Todas las decisiones de arquitectura relevantes quedaron formalizadas como registros de decisión (ADR), disponibles en el repositorio del proyecto.

---

## 2. Situación inicial y objetivo del proyecto

**Unidad solicitante:** Área de Infraestructura y Seguridad de una empresa de tecnología.

La organización atraviesa un proceso de migración hacia la nube para modernizar sus servicios y mejorar la disponibilidad de sus aplicaciones. Las soluciones existentes presentan problemas de escalabilidad, costos elevados y baja resiliencia ante fallos.

**Objetivo:** Desarrollar un diseño conceptual de arquitectura en la nube que integre los fundamentos de la computación cloud, aplicando principios de diseño arquitectónico y garantizando escalabilidad, resiliencia y seguridad, basado en el modelo cliente-servidor.

---

## 3. Fundamentos de computación en la nube

La computación en la nube se adopta como modelo porque resuelve de forma directa los tres problemas declarados:

| Problema de la organización | Cómo lo resuelve la nube |
|---|---|
| Escalabilidad limitada | Elasticidad y autoescalado bajo demanda |
| Costos elevados | Modelo de pago por uso, sin inversión en hardware propio |
| Baja resiliencia | Infraestructura distribuida en múltiples zonas de disponibilidad |

Como proveedor de referencia se utiliza **AWS**, dada su amplitud de servicios de cómputo, redes y bases de datos.

*Detalle completo en: [Fundamentos de Cloud Computing](../docs/01-fundamentos/fundamentos-cloud-computing.md)*

---

## 4. Modelos de servicio adoptados

El sistema "Nube Sólida" es una aplicación web empresarial con frontend, backend, persistencia mixta (SQL + NoSQL) y procesamiento asíncrono. Se asignó un modelo de servicio a cada componente según el nivel de control y carga operativa requeridos:

| Componente | Modelo de servicio | Justificación resumida |
|---|---|---|
| Cliente Web (SPA) | PaaS | Despliegue gestionado, sin administración de servidores |
| API / Backend de aplicación | PaaS | Foco en lógica de negocio, no en infraestructura |
| Base de datos relacional | PaaS | Reduce carga operativa manteniendo control del modelo de datos |
| Base de datos NoSQL | PaaS | Escalabilidad horizontal automática |
| Almacenamiento de archivos | IaaS | Requiere control granular de políticas de retención y permisos |
| Procesamiento asíncrono/eventos | FaaS | Carga esporádica orientada a eventos; pago solo por ejecución |
| Cola de mensajes | PaaS | Desacopla componentes sin operar un broker propio |
| Autenticación e identidad | SaaS | Funcionalidad no diferenciadora, alto estándar de seguridad de mercado |
| Monitoreo y observabilidad | SaaS | Herramienta de terceros consumida "as-is" |

**Decisión formalizada en:** [ADR-0001: Modelos de servicio por componente.](../adr/0001-modelos-de-servicio-por-componente.md)

*Detalle completo en: [Modelos de Servicio](../docs/02-modelos-servicio/analisis-modelos-servicio.md)*

---

## 5. Modelo de implementación adoptado

Se identificó que la base de datos relacional almacena información transaccional crítica del negocio, lo que exige un nivel de aislamiento superior al del resto de los componentes. Por ello, se adoptó un **modelo de implementación híbrido**:

| Segmento | Modelo de implementación | Componentes |
|---|---|---|
| Nube pública (AWS) | La mayoría de los componentes | Cliente Web, API Gateway, Load Balancer, Backend, Cola de Mensajes, Procesamiento Asíncrono, Base de datos NoSQL, Almacenamiento de archivos, Autenticación, Monitoreo |
| Nube privada | Información crítica | Base de datos relacional |

La conectividad entre ambos entornos se establece mediante un enlace **VPN site-to-site**, cifrado en tránsito, evitando que el tráfico entre el backend y la base de datos crítica transite por internet público.

**Decisión formalizada en:** [ADR-0002: Modelo de implementación en la nube.](../adr/0002-modelo-de-implementacion-nube.md)

*Detalle completo en: [Modelo de Implementación](../docs/03-modelos-implementacion/analisis-modelo-implementacion.md)*

---

## 6. Diseño arquitectónico: principios y arquitectura cliente-servidor

### 6.1 Patrón arquitectónico

Se adoptó una arquitectura **cliente-servidor**, con el servidor implementado como:

- **Microservicios en contenedores** (Amazon ECS) para la lógica de negocio principal.
- **Funciones serverless** (AWS Lambda) para el procesamiento asíncrono orientado a eventos.
- **API Gateway** como punto único de entrada del cliente.
- **Load Balancer** para distribución de carga entre instancias del backend.

### 6.2 Aplicación de los cinco principios de diseño para la nube

| Principio | Aplicación en Nube Sólida |
|---|---|
| Modularidad | Backend dividido en microservicios por dominio funcional, empaquetados en contenedores independientes |
| Desacoplamiento | Cliente solo conoce el API Gateway; cola de mensajes desacopla backend de procesamiento asíncrono |
| Resiliencia | Redundancia multi-zona, balanceo de carga, aislamiento de fallos entre microservicios |
| Elasticidad | Autoescalado de contenedores y escalado nativo a cero de las funciones serverless |
| Seguridad | Autenticación centralizada, cifrado en tránsito y en reposo, segmentación de red |

**Decisión formalizada en:** [ADR-0003: Arquitectura cliente-servidor y principios de diseño.](../adr/0003-arquitectura-cliente-servidor-y-principio-diseno.md)

*Detalle completo en: [Principios de Diseño](..docs/04-diseno-arquitectonico/principios-diseno.md) y [Arquitectura Cliente-Servidor](../docs/04-diseno-arquitectonico/arquitectura-cliente-servidor.md)*

---

## 7. Diagrama de arquitectura

[Nube Solida Arquitectura](../diagrams/export/arquitectura-nube-solida.png)

El diagrama representa los dos entornos del modelo híbrido (nube pública y nube privada), organizados en capas (cliente, integración/balanceo, aplicación, persistencia), con las conexiones y protocolos entre cada componente, y el monitoreo transversal (Amazon CloudWatch) supervisando ambos entornos.

---

## 8. Atributos de calidad

### 8.1 Resiliencia

- Amazon RDS (dato crítico) en configuración **Multi-AZ** con failover automático.
- Amazon DynamoDB y S3 con redundancia nativa multi-zona del proveedor.
- Amazon SQS actúa como buffer de resiliencia entre el backend y el procesamiento asíncrono.
- Patrón circuit breaker en el backend para evitar fallos en cascada.
- RTO objetivo: menor a 1 hora en nube pública, menor a 4 horas en el segmento privado. RPO cercano a cero en los componentes de base de datos.

**Decisión formalizada en:** [ADR-0004: Estrategia de resiliencia.](../adr/0004-estrategia-de-resiliencia.md)

### 8.2 Seguridad

- Autenticación centralizada vía Amazon Cognito (OAuth 2.0 / OpenID Connect), validada en el API Gateway.
- Cifrado en tránsito (TLS 1.3) en todas las comunicaciones, incluido el enlace VPN.
- Cifrado en reposo (AES-256) mediante AWS KMS; evaluación de claves gestionadas por el cliente (CMEK) para el componente crítico (RDS).
- Segmentación de red estricta entre el segmento público y privado, mediante Security Groups.
- Enfoque Zero Trust: ningún servicio confía implícitamente en otro por su ubicación de red.

**Decisión formalizada en:** [ADR-0005: Estrategia de seguridad.](../adr/0005-estrategia-de-seguridad.md)

### 8.3 Escalabilidad

- Escalabilidad horizontal automática por defecto en los componentes de nube pública (ECS, Lambda, DynamoDB, S3).
- Escalabilidad vertical planificada para Amazon RDS, con posibilidad de réplicas de lectura, priorizando control y consistencia sobre elasticidad total dado su rol como dato crítico.
- Autoescalado basado en métricas de CPU/memoria/tráfico para ECS; escalado nativo a cero para Lambda.

**Decisión formalizada en:** [ADR-0006: Estrategia de escalabilidad.](../adr/0006-estrategia-de-escalabilidad.md)

*Detalle completo en: [Atributos de Calidad](../docs/05-atributos-calidad/atributos-calidad.md)*

---

## 9. Registro de decisiones arquitectónicas (ADR)

| ADR | Título | Estado |
|---|---|---|
| ADR-0001 | Modelos de servicio por componente | Aceptado |
| ADR-0002 | Modelo de implementación en la nube (híbrido) | Aceptado |
| ADR-0003 | Arquitectura cliente-servidor y principios de diseño | Aceptado |
| ADR-0004 | Estrategia de resiliencia | Aceptado |
| ADR-0005 | Estrategia de seguridad | Aceptado |
| ADR-0006 | Estrategia de escalabilidad | Aceptado |

---

## 10. Conclusiones generales

El diseño conceptual de "Nube Sólida" demuestra cómo una decisión de arquitectura tomada en una etapa temprana (modelo de servicio por componente) se mantiene coherente y se referencia consistentemente a lo largo de todas las decisiones posteriores: modelo de implementación, patrón arquitectónico y estrategias de calidad.

El punto de mayor valor arquitectónico del diseño es el tratamiento diferenciado del componente crítico (base de datos relacional): en lugar de aplicar un mismo nivel de control, seguridad y escalabilidad a toda la arquitectura, se aplicó protección y tratamiento proporcional al riesgo real de cada componente, evitando tanto la sobre-ingeniería como la exposición innecesaria de información sensible.

Esta arquitectura queda documentada como base de portafolio profesional, evidenciando la capacidad de tomar y justificar decisiones de diseño cloud sólidas, trazables y fundamentadas.

---

## 11. Referencias generales

- Mell, P., & Grance, T. (2011). *The NIST definition of cloud computing* (SP 800-145). National Institute of Standards and Technology.
- Amazon Web Services. (2024). *Cloud computing overview / models / best practices for architecting resilient cloud systems*.
- Google Cloud Platform. (2024). *Introduction to cloud computing / Public, private, and hybrid cloud: Choosing the right model / Security in cloud computing*.
- Microsoft Azure. (2024). *Understanding cloud services / Cloud deployment models / Scalability and performance optimization in cloud applications*.
- Fowler, M. (2019). *Microservices: A definition of this new architectural term*.
- Bass, L., Clements, P., & Kazman, R. (2012). *Software architecture in practice* (3rd ed.). Addison-Wesley.
- Netflix Tech Blog. (2024). *Chaos engineering and resilience testing*.
- Cisco. (2024). *Security considerations for public and hybrid cloud deployments*.
