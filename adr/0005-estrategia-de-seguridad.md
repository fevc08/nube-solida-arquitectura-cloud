# ADR-0005: Estrategia de seguridad y protección de datos

**Estado:** Aceptado
**Fecha:** 2026-07-03
**Decisores:** Equipo de Arquitectura - Área de Infraestructura y Seguridad

## Contexto

El proyecto es solicitado explícitamente por el área de Infraestructura **y
Seguridad**, y ya se decidió (ADR-0002) aislar la información crítica en un
segmento de nube privada. Es necesario definir los mecanismos concretos de
autenticación, cifrado y control de acceso que se aplicarán de forma
consistente sobre toda la arquitectura, tanto en el segmento público como en
el privado.

## Decisión

Se adopta un modelo de seguridad basado en:
1. Autenticación centralizada vía Cognito (OAuth 2.0/OIDC) validada en el API Gateway.
2. Cifrado en tránsito (TLS 1.3) en todas las comunicaciones, incluyendo el enlace VPN.
3. Cifrado en reposo (AES-256) gestionado por AWS KMS en RDS, DynamoDB y S3, con evaluación de CMEK para RDS.
4. Segmentación de red estricta (Security Groups) para el segmento privado.
5. Enfoque Zero Trust: ningún servicio confía implícitamente en otro por ubicación de red.

## Alternativas consideradas

| Opción | Ventajas | Desventajas |
|---|---|---|
| **A. Cifrado gestionado por el proveedor únicamente (server-side, sin CMEK)** | Menor complejidad operativa; el proveedor administra el ciclo de vida de claves. | Menor control sobre las claves del dato más crítico del sistema (RDS); no diferencia el nivel de protección según criticidad del componente. |
| **B. Cifrado con claves propias (CMEK) en todos los componentes** | Máximo control sobre todas las claves criptográficas. | Sobrecarga operativa innecesaria para componentes no críticos (DynamoDB, S3); contradice el objetivo de reducir carga operativa (ADR-0001). |
| **C. Cifrado diferenciado por criticidad: server-side por defecto, CMEK evaluado solo para RDS (elegida)** | Proporciona mayor control exactamente donde se requiere (dato crítico), sin sobrecargar el resto de la arquitectura. | Requiere gestionar dos enfoques de cifrado distintos dentro del mismo proyecto (complejidad menor, acotada). |

## Justificación

- Es coherente con el criterio ya usado en ADR-0002 (protección proporcional al
  riesgo): el mayor nivel de control criptográfico se reserva para el
  componente que efectivamente aloja información crítica.
- La validación de identidad en el API Gateway antes de llegar a los
  microservicios evita exponer innecesariamente la lógica de negocio a
  solicitudes no autenticadas, reduciendo la superficie de ataque.
- El enfoque Zero Trust es consistente con operar en un modelo híbrido: al no
  existir un perímetro de red único y confiable, cada solicitud debe
  autenticarse independientemente de su origen.

## Consecuencias

**Positivas**
- Protección proporcional y justificada de cada componente según su criticidad real.
- Reducción de superficie de ataque mediante validación temprana en el API Gateway.

**Negativas / trade-offs asumidos**
- Gestión de dos enfoques de cifrado (server-side y CMEK) incrementa levemente la complejidad operativa respecto de un esquema uniforme.

**Riesgos y mitigación**
- *Riesgo*: pérdida o rotación indebida de claves CMEK en RDS. *Mitigación*: uso de AWS KMS con políticas de rotación automática y auditoría (CloudTrail).
- *Riesgo*: mal manejo de tokens OAuth. *Mitigación*: tiempos de expiración cortos y renovación vía refresh tokens gestionados por Cognito.

## Referencias
- Manual L5 – Principales atributos de calidad en una arquitectura en la nube.
- [atributos-calidad.md](../docs/05-atributos-calidad/atributos-calidad.md)
- [ADR-0002: Modelo de implementación en la nube.](../adr/0002-modelo-de-implementacion-nube.md)
