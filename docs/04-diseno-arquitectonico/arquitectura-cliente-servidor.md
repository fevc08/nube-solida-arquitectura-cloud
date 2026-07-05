# Arquitectura Cliente-Servidor de Nube Sólida

**Proyecto:** Nube Sólida
**Módulo:** M3 – Fundamentos de la Arquitectura Cloud
**Lección:** 4 – Principios fundamentales de diseño de una arquitectura

## 1. Objetivo

Detallar la arquitectura cliente-servidor de "Nube Sólida": tipos de cliente y
servidor, capas, conectividad y protocolos, consolidando en una única vista
todos los componentes y decisiones previas (ADR-0001 y ADR-0002). Esta vista es
la base directa para el diagrama de arquitectura de la Lección 4.

## 2. Tipo de cliente

| Tipo | Aplica | Justificación |
|---|---|---|
| Cliente ligero (Thin Client) | **Sí** | La SPA se ejecuta en el navegador y delega toda la lógica de negocio y persistencia al backend. |
| Cliente pesado / móvil / híbrido | No, en esta fase | Fuera de alcance del diseño conceptual actual; arquitectura preparada para incorporarlos a futuro sin cambios estructurales (mismo API Gateway). |

## 3. Tipos de servidor presentes

| Tipo de servidor | Componente correspondiente |
|---|---|
| Servidor de aplicaciones | Backend (microservicios) |
| Servidor de bases de datos | Base de datos relacional + Base de datos NoSQL |
| Servidor de autenticación | Componente SaaS de Identidad |
| Servidor de archivos | Almacenamiento de objetos (IaaS) |

## 4. Componentes adicionales de infraestructura (capa de integración y balanceo)

Para completar el modelo cliente-servidor según el Manual L4, se incorporan dos
componentes no descritos aún en lecciones anteriores:

| Componente | Función | Capa |
|---|---|---|
| **API Gateway** | Punto único de entrada del cliente hacia el backend; enruta solicitudes y valida tokens de autenticación. | Integración |
| **Load Balancer** | Distribuye el tráfico entre las instancias/contenedores del backend. | Balanceo |

## 5. Conectividad y protocolos

| Comunicación | Protocolo |
|---|---|
| Cliente Web ↔ API Gateway | HTTPS (REST) |
| API Gateway ↔ Backend (microservicios) | HTTPS (REST) interno |
| Backend ↔ Componente de Identidad | OAuth 2.0 / OpenID Connect |
| Backend ↔ Cola de Mensajes | Protocolo de mensajería gestionado (ej. AMQP) |
| Cola de Mensajes ↔ Procesamiento Asíncrono (FaaS) | Invocación basada en eventos |
| Backend ↔ Base de datos NoSQL | Driver nativo del servicio gestionado (TLS) |
| Backend ↔ Almacenamiento de archivos | API del servicio de objetos (HTTPS) |
| Backend ↔ Base de datos relacional (nube privada) | **VPN / enlace dedicado** (TLS) — cruce entre nube pública y privada, ver ADR-0002 |
| Todos los componentes → Monitoreo | Agentes/instrumentación (HTTPS) |

## 6. Tabla consolidada de arquitectura (base para el diagrama)

Esta es la tabla maestra: reúne capa, modelo de servicio (ADR-0001) y modelo de
implementación (ADR-0002) de **todos** los componentes.

| # | Componente | Capa | Modelo de servicio | Modelo de implementación |
|---|---|---|---|---|
| 1 | Cliente Web (SPA) | Cliente | PaaS | Nube Pública |
| 2 | API Gateway | Integración | PaaS | Nube Pública |
| 3 | Load Balancer | Balanceo | PaaS | Nube Pública |
| 4 | Backend (microservicios en contenedores) | Aplicación / Lógica | PaaS | Nube Pública |
| 5 | Autenticación e Identidad | Aplicación (transversal) | SaaS | Nube Pública |
| 6 | Cola de Mensajes | Integración | PaaS | Nube Pública |
| 7 | Procesamiento Asíncrono | Aplicación | FaaS | Nube Pública |
| 8 | Base de datos NoSQL | Persistencia | PaaS | Nube Pública |
| 9 | Almacenamiento de archivos/objetos | Persistencia | IaaS | Nube Pública |
| 10 | **Base de datos relacional** | Persistencia | PaaS | **Nube Privada** |
| 11 | Monitoreo y Observabilidad | Transversal (todas las capas) | SaaS | Nube Pública (visibilidad sobre ambos entornos) |
| 12 | Enlace VPN / dedicado | Conectividad | — (infraestructura de red) | Frontera entre Nube Pública y Nube Privada |

## 7. Conclusión

Esta tabla consolidada es el insumo directo para el diagrama de arquitectura:
cada fila es un componente/caja del diagrama, la columna "Capa" define su
agrupación visual (cliente / integración / aplicación / persistencia), y la
columna "Modelo de implementación" define en qué "contenedor visual" (nube
pública vs. nube privada) debe dibujarse cada componente.

## Referencias
- Manual L4 - Principios fundamentales de diseño de una arquitectura.
- [ADR-0001: Modelos de servicio por componente.](../adr/0001-modelos-de-servicio-por-componente.md)
- [ADR-0002: Modelo de implementación en la nube.](../adr/0002-modelo-de-implementacion-nube.md)
