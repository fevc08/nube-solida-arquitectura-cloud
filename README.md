# Nube Sólida: Diseño Conceptual de Arquitectura Cloud

## Descripción del proyecto

Diseño conceptual de arquitectura en la nube para "Nube Sólida", desarrollado en respuesta a la solicitud del Área de Infraestructura y Seguridad de una empresa de tecnología en proceso de migración a la nube.

El proyecto resuelve tres problemas centrales identificados en la situación inicial: escalabilidad limitada, costos operativos elevados y baja resiliencia ante fallos, mediante un modelo de servicio diferenciado por componente, un modelo de implementación híbrido y una arquitectura cliente-servidor basada en microservicios y funciones serverless.

**Proveedor de referencia:** AWS
**Patrón arquitectónico:** Cliente-servidor, microservicios en contenedores + funciones serverless
**Modelo de implementación:** Híbrido (nube pública + nube privada para información crítica)

## Índice de documentación

| Lección | Documento |
|---|---|
| 1 – Fundamentos de cloud computing | [docs/01-fundamentos/fundamentos-cloud-computing.md](docs/01-fundamentos/fundamentos-cloud-computing.md) |
| 2 – Modelos de servicio | [docs/02-modelos-servicio/analisis-modelos-servicio.md](docs/02-modelos-servicio/analisis-modelos-servicio.md) |
| 3 – Modelo de implementación | [docs/03-modelos-implementacion/analisis-modelo-implementacion.md](docs/03-modelos-implementacion/analisis-modelo-implementacion.md) |
| 4 – Principios de diseño | [docs/04-diseno-arquitectonico/principios-diseno.md](docs/04-diseno-arquitectonico/principios-diseno.md) |
| 4 – Arquitectura cliente-servidor | [docs/04-diseno-arquitectonico/arquitectura-cliente-servidor.md](docs/04-diseno-arquitectonico/arquitectura-cliente-servidor.md) |
| 5 – Atributos de calidad | [docs/05-atributos-calidad/atributos-calidad.md](docs/05-atributos-calidad/atributos-calidad.md) |
| Entrega final – Documento integrador | [docs/06-entrega-final/documento-integrador.md](docs/06-entrega-final/documento-integrador.md) |

## Decisiones de arquitectura (ADR)

| ADR | Título |
|---|---|
| [0001](adr/0001-modelos-de-servicio-por-componente.md) | Modelos de servicio por componente |
| [0002](adr/0002-modelo-de-implementacion-nube.md) | Modelo de implementación en la nube (híbrido) |
| [0003](adr/0003-arquitectura-cliente-servidor-y-principios-diseno.md) | Arquitectura cliente-servidor y principios de diseño |
| [0004](adr/0004-estrategia-de-resiliencia.md) | Estrategia de resiliencia |
| [0005](adr/0005-estrategia-de-seguridad.md) | Estrategia de seguridad |
| [0006](adr/0006-estrategia-de-escalabilidad.md) | Estrategia de escalabilidad |

## Diagrama de arquitectura

- Archivo editable: [diagrams/src/arquitectura-nube-solida.drawio](diagrams/src/arquitectura-nube-solida.drawio)
- Imagen exportada: [diagrams/export/arquitectura-nube-solida.png](diagrams/export/arquitectura-nube-solida.png)

## Estructura del repositorio

```
nube-solida-arquitectura-cloud/
├── README.md
├── docs/
│   ├── 01-fundamentos/
│   ├── 02-modelos-servicio/
│   ├── 03-modelos-implementacion/
│   ├── 04-diseno-arquitectonico/
│   ├── 05-atributos-calidad/
│   └── 06-entrega-final/
├── adr/
│   ├── 0000-plantilla.md
│   ├── 0001-modelos-de-servicio-por-componente.md
│   ├── 0002-modelo-de-implementacion-nube.md
│   ├── 0003-arquitectura-cliente-servidor-y-principios-diseno.md
│   ├── 0004-estrategia-de-resiliencia.md
│   ├── 0005-estrategia-de-seguridad.md
│   └── 0006-estrategia-de-escalabilidad.md
└── diagrams/
    ├── src/
    └── export/
```

## Autor

Fidel Vera Chourio - Proyecto desarrollado como evaluación del Módulo 3: Fundamentos de la Arquitectura Cloud (Talento Digital).