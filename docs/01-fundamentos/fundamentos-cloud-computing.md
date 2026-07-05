# Fundamentos de la Computación en la Nube

**Proyecto:** Nube Sólida
**Módulo:** M3 - Fundamentos de la Arquitectura Cloud
**Lección:** 1 - Introducción a la computación en la nube

## 1. Contexto del proyecto

La organización (empresa de tecnología, área de Infraestructura y Seguridad), está
migrando sus servicios a la nube para resolver tres problemas actuales:
- Escalabilidad limitada de la infraestructura on-premise.
- Costos operativos elevados.
- Baja resiliencia ante fallos.

Este documento resume los fundamentos de la computación en la nube que sustentan
las decisiones de arquitectura que se tomarán en las siguientes etapas del proyecto.

## 2. Definición

La computación en la nube es un modelo de prestación de servicios de TI que entrega
recursos informáticos (cómputo, almacenamiento, redes, bases de datos) a través de
internet, bajo demanda y sin necesidad de que la organización posea o administre
infraestructura física propia.

Se apoya en dos pilares técnicos:
- **Virtualización**: permite compartir recursos físicos entre múltiples usuarios/cargas de trabajo de forma aislada.
- **Automatización y orquestación**: permite aprovisionar, escalar y liberar recursos con mínima intervención manual.

## 3. Características principales

| Característica | Relevancia para Nube Sólida |
|---|---|
| **Escalabilidad y elasticidad** | Responde directamente al problema de "problemas de escalabilidad" mencionado en la situación inicial. |
| **Pago por uso** | Ataca el problema de "costos elevados". |
| **Alta disponibilidad y recuperación ante desastres** | Ataca el problema de "baja resiliencia ante fallos". |
| **Accesibilidad y conectividad global** | Habilita el modelo cliente-servidor distribuido que pide la consigna. |
| **Seguridad y cumplimiento** | Base para el atributo de calidad "seguridad" exigido en el proyecto. |
| **Automatización y autogestión** | Reduce carga operativa del equipo de Infraestructura. |

## 4. Beneficios esperados para la organización

1. Reducción de costos de capital (CAPEX) al eliminar inversión en hardware propio, pasando a un modelo de gasto operativo (OPEX).
2. Capacidad de escalar recursos dinámicamente ante picos de demanda, sin sobreaprovisionar.
3. Mejora de la disponibilidad mediante infraestructura distribuida en múltiples zonas/regiones.
4. Aceleración en la adopción de tecnologías avanzadas (IA, analítica de datos) sin inversión inicial en esa infraestructura.
5. Reducción de la carga de mantenimiento del equipo interno de TI, permitiendo foco en valor de negocio.

## 5. Principales proveedores considerados

Como referencia para las siguientes etapas (selección de modelos de servicio e
implementación), se consideran los tres proveedores líderes del mercado:

| Proveedor | Fortaleza principal |
|---|---|
| **Amazon Web Services (AWS)** | Mayor amplitud de servicios de cómputo, redes y bases de datos; líder de mercado. |
| **Microsoft Azure** | Integración natural con ecosistema Microsoft (Office 365, Active Directory). |
| **Google Cloud Platform (GCP)** | Fortaleza en inteligencia artificial y big data. |

> La selección final de proveedor(es) se justificará formalmente en el ADR
> correspondiente a la Lección 2, en función de los modelos de servicio elegidos.

## 6. Modelos de costos en la nube

| Modelo | Descripción | Aplicabilidad en Nube Sólida |
|---|---|---|
| Pago por uso (on-demand) | Se factura según consumo real | Cargas variables / impredecibles |
| Instancias reservadas | Descuento por compromiso a largo plazo | Cargas base predecibles y constantes |
| Planes de suscripción | Tarifa fija periódica | Servicios SaaS de soporte |
| Computación sin servidor | Se paga solo por ejecución de código | Procesos eventuales/orientados a eventos |

## 7. Conclusión de la lección

La adopción de un modelo cloud le permite a la organización resolver de forma
directa los tres problemas planteados en la situación inicial (escalabilidad, costos,
resiliencia), sentando las bases conceptuales para las decisiones de modelo de
servicio (Lección 2), modelo de implementación (Lección 3), diseño arquitectónico
(Lección 4) y atributos de calidad (Lección 5) que se desarrollan a continuación.

## Referencias
- Mell, P., & Grance, T. (2011). *The NIST definition of cloud computing* (SP 800-145). NIST.
- Amazon Web Services. (2024). *Cloud computing overview*.
- Google Cloud Platform. (2024). *Introduction to cloud computing*.
- Microsoft. (2024). *Understanding cloud services*.
