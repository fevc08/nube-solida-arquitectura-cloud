# Atributos de Calidad de la Arquitectura Nube Sólida

**Proyecto:** Nube Sólida
**Módulo:** M3 - Fundamentos de la Arquitectura Cloud
**Lección:** 5 - Atributos de calidad en la arquitectura en la nube

## 1. Objetivo

Definir las estrategias y mecanismos concretos que garantizan resiliencia,
seguridad y escalabilidad sobre los componentes ya definidos en las Lecciones
2, 3 y 4, y reflejados en el diagrama de arquitectura.

## 2. Resiliencia y disponibilidad

### 2.1 Estrategias por componente

| Componente | Mecanismo de resiliencia |
|---|---|
| Amazon ECS (Backend/microservicios) | Múltiples réplicas desplegadas en al menos 2 zonas de disponibilidad (AZ); reinicio automático de tareas fallidas. |
| Elastic Load Balancing | Redirige tráfico automáticamente a instancias sanas mediante health checks; elimina el punto único de fallo en la capa de balanceo. |
| Amazon RDS (nube privada) | Configuración Multi-AZ con instancia réplica en standby y failover automático. |
| Amazon DynamoDB | Replicación automática multi-AZ nativa del servicio gestionado. |
| Amazon S3 | Redundancia geográfica nativa (múltiples zonas dentro de la región). |
| Amazon SQS | Actúa como buffer de resiliencia: si el backend o Lambda fallan temporalmente, los mensajes persisten en la cola sin pérdida de eventos. |
| AWS Lambda | Reintentos automáticos configurables ante fallos de ejecución; escalado independiente por invocación evita que un pico sature el resto del sistema. |
| Enlace VPN (público↔privado) | Se contempla un túnel VPN redundante (ruta de respaldo) para evitar que la conectividad híbrida sea un punto único de fallo. |
| Amazon CloudWatch | Monitoreo proactivo y alarmas que detectan degradación antes de una caída total del servicio. |

### 2.2 Patrones de tolerancia a fallos

- **Aislamiento de fallos**: la arquitectura de microservicios (ADR-0003) evita que la caída de un servicio (ej. notificaciones vía Lambda) derribe el sistema completo.
- **Circuit breaker**: el backend implementa este patrón al invocar servicios externos (Cognito, RDS), evitando reintentos en cascada que agraven una falla existente.
- **Desacoplamiento vía cola**: SQS entre el backend y Lambda (ADR-0003) permite que el procesamiento asíncrono se recupere sin pérdida de eventos ante una interrupción temporal.

### 2.3 Recuperación ante desastres (DR)

| Elemento | Estrategia |
|---|---|
| Backups de RDS | Snapshots automáticos diarios + point-in-time recovery. |
| Backups de DynamoDB | Point-in-time recovery habilitado. |
| Objetivo de recuperación (RTO) | < 1 hora para componentes en nube pública (autoescalado/reinicio automático); < 4 horas para el segmento de nube privada (RDS), dado su mayor complejidad operativa. |
| Objetivo de punto de recuperación (RPO) | Cercano a cero para RDS (Multi-AZ síncrono) y DynamoDB (replicación nativa). |

## 3. Escalabilidad

### 3.1 Escalabilidad horizontal vs. vertical por componente

| Componente | Tipo de escalado | Detalle |
|---|---|---|
| Amazon ECS | **Horizontal** | Se agregan/quitan tareas (contenedores) según CPU/memoria/tráfico. |
| AWS Lambda | **Horizontal (nativo)** | Escala automáticamente de cero a miles de ejecuciones concurrentes según el volumen de eventos en SQS. |
| Amazon DynamoDB | **Horizontal** | Particionamiento automático según volumen de datos y throughput. |
| Amazon RDS (nube privada) | **Vertical** (con límite) | Al ser el componente crítico en nube privada, prioriza estabilidad y control sobre elasticidad total; se dimensiona la instancia según capacidad proyectada, con posibilidad de añadir réplicas de lectura si el volumen de consultas lo justifica. |
| Amazon S3 | **Horizontal (nativo)** | Escalabilidad prácticamente ilimitada, gestionada por el proveedor. |

### 3.2 Autoescalado y elasticidad

- **ECS**: políticas de autoescalado basadas en % de utilización de CPU/memoria (ej. escalar +1 tarea si CPU > 70% durante 5 min).
- **Lambda**: elasticidad nativa; sin configuración manual de capacidad, responde a los eventos publicados en SQS.
- **Optimización de costos**: al usar autoescalado y FaaS, el sistema paga solo por los recursos efectivamente utilizados en cada momento, atacando directamente el problema de "costos elevados" de la situación inicial.

### 3.3 Escalabilidad geográfica (consideración a futuro)

En esta primera fase, "Nube Sólida" opera en una única región de AWS. Se deja
documentado como evolución futura (no parte del alcance actual) el uso de CDN
(Amazon CloudFront) para el Cliente Web y despliegues multi-región para el
backend, si el crecimiento del negocio lo requiere.

## 4. Seguridad

### 4.1 Autenticación y autorización

- **Amazon Cognito** gestiona el ciclo de vida de identidades y emite tokens vía **OAuth 2.0 / OpenID Connect** (ya definido en ADR-0001 y ADR-0003).
- El **API Gateway** valida el token en cada solicitud antes de enrutarla al backend, evitando que peticiones no autenticadas lleguen a los microservicios.
- Se aplica el **principio de mínimo privilegio** mediante roles IAM diferenciados por servicio (ECS, Lambda, RDS, DynamoDB, S3 tienen permisos acotados únicamente a lo que cada uno requiere).

### 4.2 Cifrado de datos

| Tipo | Mecanismo | Dónde se aplica |
|---|---|---|
| En tránsito | TLS 1.3 | Todas las comunicaciones del diagrama (Cliente↔API Gateway, ECS↔servicios, y especialmente el enlace VPN hacia RDS). |
| En reposo | AES-256 | RDS, DynamoDB y S3, mediante cifrado gestionado por el proveedor (AWS KMS). |
| Claves gestionadas por el cliente (CMEK) | Se evalúa como opción para RDS (dato crítico), dado el mayor control que exige su ubicación en nube privada. |

### 4.3 Segmentación de red

- El segmento de **nube privada** (RDS) solo es alcanzable desde ECS, a través del túnel **VPN site-to-site**, nunca expuesto directamente a internet (ya definido en ADR-0002).
- Se aplican **Security Groups** restrictivos: ECS solo puede conectarse a RDS por el puerto de base de datos específico; ningún otro componente tiene esa ruta habilitada.

### 4.4 Zero Trust y monitoreo de amenazas

- Ningún componente confía implícitamente en otro por estar "dentro" de la red: cada solicitud entre servicios se autentica (Cognito/IAM), consistente con un enfoque **Zero Trust Architecture**.
- **Amazon CloudWatch** centraliza logs y métricas de seguridad de ambos entornos (público y privado), permitiendo detectar actividad anómala.

## 5. Cómo se complementan los tres atributos

La arquitectura híbrida (ADR-0002) es precisamente el punto donde los tres
atributos convergen: el componente con mayor exigencia de **seguridad** (RDS)
sacrifica algo de **escalabilidad** horizontal a cambio de mayor control, mientras
que el resto de los componentes prioriza **escalabilidad** y **resiliencia** al
operar en nube pública con autoescalado nativo. El resultado es un balance
consciente y justificado, no una limitación accidental del diseño.

## 6. Conclusión

Los mecanismos descritos en este documento completan el diseño conceptual de
"Nube Sólida" iniciado en la Lección 1, aplicando de forma concreta —sobre cada
componente real del diagrama— los atributos de calidad exigidos por la
consigna del proyecto. El detalle de las decisiones tomadas y sus alternativas
se documenta en ADR-0004 (resiliencia), ADR-0005 (seguridad) y ADR-0006
(escalabilidad).

## Referencias
- Manual L5 - Principales atributos de calidad en una arquitectura en la nube.
- Amazon Web Services. (2024). *Best practices for architecting resilient cloud systems*.
- Google Cloud. (2024). *Security in cloud computing: Advanced threat protection*.
- Microsoft Azure. (2024). *Scalability and performance optimization in cloud applications*.
- Netflix Tech Blog. (2024). *Chaos engineering and resilience testing*.
