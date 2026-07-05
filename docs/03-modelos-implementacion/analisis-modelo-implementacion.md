# Análisis del Modelo de Implementación en la Nube

**Proyecto:** Nube Sólida
**Módulo:** M3 - Fundamentos de la Arquitectura Cloud
**Lección:** 3 - Modelos de implementación en la nube

## 1. Objetivo

Determinar y justificar el modelo de implementación (nube pública, privada o
híbrida) más adecuado para la arquitectura de "Nube Sólida", en base a los
componentes y modelos de servicio definidos en la Lección 2.

## 2. Criterios de decisión aplicados al caso

| Criterio | Situación de "Nube Sólida" |
|---|---|
| **Existencia de información crítica** | La base de datos relacional almacena datos transaccionales sensibles del negocio (información de clientes/operaciones), que requiere un nivel de control y aislamiento superior al del resto de los componentes. |
| **Seguridad y cumplimiento normativo** | No hay una normativa sectorial específica, pero sí un requisito interno explícito del área de Seguridad de proteger la información crítica del negocio con controles adicionales. |
| **Presupuesto disponible** | Uno de los problemas declarados es "costos elevados"; se busca minimizar el componente privado (más costoso) al mínimo indispensable. |
| **Escalabilidad y flexibilidad** | Otro problema declarado explícitamente es "problemas de escalabilidad"; se resuelve manteniendo en nube pública los componentes con mayor variabilidad de demanda. |
| **Nivel de control requerido** | Alto solo para el dato crítico (base de datos relacional); medio-bajo para el resto de los componentes, ya delegados a PaaS/SaaS (ADR-0001). |
| **Tiempo de implementación** | Se prioriza agilidad; solo el componente crítico justifica una implementación más compleja. |
| **Recursos de TI disponibles** | Limitados; se busca minimizar la superficie administrada internamente, restringiéndola solo al segmento privado indispensable. |
| **Resiliencia y continuidad del negocio** | Requerimiento explícito; se resuelve combinando alta disponibilidad nativa de la nube pública con redundancia dedicada para el segmento privado. |

## 3. Comparación de modelos para este caso

| Modelo | ¿Aplica a Nube Sólida? | Motivo |
|---|---|---|
| **Nube pública (pura)** | No, insuficiente | Resuelve costos, escalabilidad y resiliencia, pero no ofrece el nivel de aislamiento y control que el área de Seguridad exige para la información transaccional crítica del negocio. |
| **Nube privada (pura)** | No | Los altos costos de implementación y mantenimiento, y la escalabilidad limitada, contradicen directamente los objetivos de reducción de costos y elasticidad ante demanda variable, que sí aplican a la mayoría de los componentes no críticos. |
| **Nube híbrida (elegida)** | **Sí** | Permite mantener la información crítica en un entorno privado y controlado, mientras se aprovechan la elasticidad y el bajo costo de la nube pública para el resto de los componentes (frontend, backend, procesamiento asíncrono, storage, auth, monitoreo). |

## 4. Decisión

Se adopta un **modelo de implementación híbrido**:

- **Nube privada**: Base de datos relacional (datos transaccionales críticos del negocio).
- **Nube pública** (AWS como proveedor de referencia): todos los demás componentes
  definidos en el ADR-0001 (Cliente Web, API/Backend, Base de datos NoSQL,
  Almacenamiento de archivos, Procesamiento asíncrono, Cola de mensajes,
  Autenticación, Monitoreo).

La conectividad entre ambos entornos se establece mediante una red privada
virtual (VPN) o enlace dedicado, garantizando que el tráfico entre el backend
(nube pública) y la base de datos crítica (nube privada) no transite por
internet público.

La justificación extendida, alternativas descartadas y riesgos asumidos se
documentan formalmente en
**[ADR-0002](../../adr/0002-modelo-de-implementacion-nube.md)**.

## 5. Consideraciones de seguridad del modelo híbrido

- **Segmentación de red**: el segmento privado que aloja la base de datos crítica
  solo es accesible desde la capa de backend, mediante reglas de firewall/security
  groups restrictivas.
- **Cifrado en tránsito**: toda comunicación entre el entorno público y el privado
  viaja cifrada (VPN site-to-site o enlace dedicado con TLS).
- **Principio de mínimo privilegio**: solo los servicios de backend autorizados
  pueden establecer conexión con la base de datos crítica.
- **Redundancia del enlace híbrido**: se contempla un enlace de respaldo para
  evitar que la conectividad entre nubes se convierta en un punto único de fallo
  (detalle en la Lección 5 – atributos de calidad).

## 6. Conclusión

El modelo híbrido permite proteger específicamente la información crítica del
negocio sin renunciar a los beneficios de costo, escalabilidad y resiliencia que
ofrece la nube pública para el resto de la arquitectura. Esta decisión, junto con
el ADR-0001, define el marco sobre el cual se construye el diseño arquitectónico
detallado en la Lección 4.

## Referencias
- Mell, P., & Grance, T. (2011). *The NIST definition of cloud computing* (SP 800-145). NIST.
- Google Cloud Platform. (2024). *Public, private, and hybrid cloud: Choosing the right model*.
- Oracle Cloud Infrastructure. (2024). *Hybrid cloud solutions for modern businesses*.
- Cisco. (2024). *Security considerations for public and hybrid cloud deployments*.