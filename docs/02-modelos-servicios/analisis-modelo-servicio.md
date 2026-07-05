# Análisis de Modelos de Servicio en la Nube

**Proyecto:** Nube Sólida
**Módulo:** M3 - Fundamentos de la Arquitectura Cloud
**Lección:** 2 - Modelos de servicio en la nube

## 1. Objetivo

Analizar los modelos de servicio en la nube (IaaS, PaaS, SaaS, FaaS) y asignar el
modelo más adecuado a cada componente de la solución "Nube Sólida", en base al
nivel de control, flexibilidad y carga operativa requeridos.

## 2. Componentes funcionales del sistema

"Nube Sólida" es una aplicación web empresarial con arquitectura cliente-servidor,
compuesta por los siguientes componentes:

| # | Componente | Descripción |
|---|---|---|
| 1 | Cliente Web (SPA) | Interfaz de usuario que consume la API del backend. |
| 2 | API / Backend de aplicación | Contiene la lógica de negocio y expone servicios al cliente. |
| 3 | Base de datos relacional | Almacena datos transaccionales estructurados (usuarios, operaciones del negocio). |
| 4 | Base de datos NoSQL | Almacena datos semi-estructurados (sesiones, logs de actividad, catálogos flexibles). |
| 5 | Almacenamiento de archivos/objetos | Guarda documentos adjuntos y reportes generados. |
| 6 | Procesamiento asíncrono / eventos | Ejecuta tareas de fondo: envío de notificaciones, generación de reportes, procesamiento de archivos subidos. |
| 7 | Cola de mensajes | Desacopla el backend del procesamiento asíncrono. |
| 8 | Autenticación e identidad | Gestiona el login y la seguridad de acceso de usuarios. |
| 9 | Monitoreo y observabilidad | Supervisa el estado y rendimiento de la plataforma. |

## 3. Análisis de cada modelo de servicio

### 3.1 IaaS (Infraestructura como Servicio)
Entrega acceso a recursos de cómputo, almacenamiento y red virtualizados. El equipo
de arquitectura mantiene el control sobre el sistema operativo y el entorno, pero no
sobre el hardware físico. Es el modelo con mayor control y mayor responsabilidad
operativa.

### 3.2 PaaS (Plataforma como Servicio)
Entrega un entorno de ejecución/desarrollo administrado, incluyendo bases de datos
gestionadas. Elimina la necesidad de administrar sistema operativo o parches,
permitiendo enfocarse en la lógica de negocio.

### 3.3 SaaS (Software como Servicio)
Entrega software completamente gestionado y listo para usar, consumido por
suscripción o API. Se utiliza cuando la funcionalidad no es diferenciadora para el
negocio y existe una solución de mercado madura y confiable.

### 3.4 FaaS (Función como Servicio / Serverless)
Permite ejecutar código en respuesta a eventos, sin administrar infraestructura ni
procesos persistentes, con cobro únicamente por tiempo de ejecución. Ideal para
cargas de trabajo esporádicas o dirigidas por eventos.

## 4. Asignación de modelo de servicio por componente

| Componente | Modelo asignado | Justificación breve |
|---|---|---|
| Cliente Web (SPA) | **PaaS** (hosting estático gestionado) | No requiere gestión de servidores; se prioriza despliegue rápido y CDN integrada. |
| API / Backend de aplicación | **PaaS** (App Service / Elastic Beanstalk) | Se necesita foco en lógica de negocio, no en administración de OS; incluye autoescalado. |
| Base de datos relacional | **PaaS** (Base de datos administrada) | Reduce carga operativa de parchado/backups manteniendo control sobre el modelo de datos. |
| Base de datos NoSQL | **PaaS** (Base de datos administrada NoSQL) | Escalabilidad horizontal automática para datos semi-estructurados de alto volumen. |
| Almacenamiento de archivos/objetos | **IaaS** (Object Storage) | Se requiere control granular sobre políticas de retención, ciclo de vida y permisos de los archivos. |
| Procesamiento asíncrono / eventos | **FaaS** | Cargas de trabajo esporádicas y dirigidas por eventos; pago solo por ejecución; sin servidores que administrar. |
| Cola de mensajes | **PaaS** (Mensajería gestionada) | Desacopla componentes sin requerir operar un broker propio. |
| Autenticación e identidad | **SaaS** (Identity as a Service) | Funcionalidad no diferenciadora del negocio; se prioriza una solución madura y certificada en seguridad. |
| Monitoreo y observabilidad | **SaaS** | Herramienta de terceros consumida "as-is", sin desarrollo propio. |

> La justificación extendida de esta asignación, junto con las alternativas
> evaluadas, se documenta formalmente en **[ADR-0001](../../adr/0001-modelos-de-servicio-por-componente.md)**.

## 5. Vista de capas (cliente-servidor) según modelo de servicio

| Capa | Componente | Modelo de servicio |
|---|---|---|
| **Cliente** | Cliente Web (SPA) | PaaS |
| **Servidor** | API / Backend de aplicación | PaaS |
| **Servidor** | Autenticación e Identidad | SaaS |
| **Servidor** | Cola de Mensajes | PaaS |
| **Servidor** | Procesamiento Asíncrono | FaaS |
| **Servidor** | Base de datos relacional | PaaS |
| **Servidor** | Base de datos NoSQL | PaaS |
| **Servidor** | Almacenamiento de archivos | IaaS |
| **Servidor** | Monitoreo y Observabilidad | SaaS |

*(El diagrama gráfico formal de esta arquitectura se desarrolla en la Lección 4,
una vez definidos los principios de diseño arquitectónico.)*

## 6. Proveedor de referencia

Para mantener consistencia y evitar la complejidad operativa de una estrategia
multi-nube (evaluada y descartada como riesgo innecesario, ver Lección 5), se usa
**AWS** como proveedor de referencia para los ejemplos de servicios concretos a lo
largo de este proyecto, sin que ello impida una futura migración o estrategia
multi-cloud si el negocio lo requiriera.

## 7. Conclusión

La combinación de modelos de servicio seleccionada prioriza reducir la carga
operativa del equipo de Infraestructura (uso extendido de PaaS y SaaS), reservando
IaaS solo donde se requiere control granular (almacenamiento de archivos) y FaaS
donde el patrón de carga es naturalmente orientado a eventos. Esta asignación es la
base para definir, en la Lección 3, el modelo de implementación (pública, privada o
híbrida) más adecuado.

## Referencias
- Mell, P., & Grance, T. (2011). *The NIST definition of cloud computing* (SP 800-145). NIST.
- Amazon Web Services. (2024). *Cloud computing models*.
- Google Cloud Platform. (2024). *Understanding cloud computing*.
- Microsoft. (2024). *Cloud services explained*.
