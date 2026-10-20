# Cisco Enterprise Lab

**Laboratorio independiente de ingeniería de redes, seguridad y troubleshooting sobre hardware Cisco real.**

> Estado: en desarrollo. La conectividad WAN de extremo a extremo todavía no está validada.

## Infraestructura

| Equipo | Función | Información |
|---|---|---|
| Cisco ISR 4451 | Router de borde | Inventario IOS XE y licencias pendientes de revisión |
| Cisco Catalyst 3850, 24 puertos Gigabit | Core L2/L3 | IOS XE 16.6.5 Everest, IP Services permanente (reportado) |
| MikroTik RB4011 | Upstream de Internet | Enlace físico 1 Gbps full-duplex confirmado |

El RB4011 forma parte de **otro laboratorio** y se utiliza como proveedor upstream. Este repositorio es totalmente independiente de `network-engineering-labs`.

## Objetivo

Desarrollar conocimientos demostrables en Cisco IOS XE: switching, routing, ACL, seguridad, observabilidad, automatización y análisis de fallas. El propósito es aprender **por qué** funcionan los protocolos y comandos, no ejecutar configuraciones sin comprenderlas.

## Metodología

Cada laboratorio documentará: fundamentos, objetivo, diseño, prerequisitos, comandos explicados, efectos y riesgos, pruebas positivas/negativas, troubleshooting, rollback y evidencias sanitizadas.

Un laboratorio **no** se declara completado solo porque un comando fue aceptado. Debe validarse su funcionamiento y explicar la causa técnica.

- [Estándar de aprendizaje y evidencia](docs/LEARNING-STANDARD.md)
- [LAB-001: Arquitectura y conectividad WAN](01-foundations/LAB-001-wan-connectivity/README.md)

## Ruta de aprendizaje

1. Arquitectura, IPv4, interfaces y conectividad WAN.
2. IOS XE, administración segura, SSH y AAA.
3. VLAN, trunks, STP, EtherChannel y seguridad de puertos.
4. SVIs, rutas estáticas, OSPF y análisis de enrutamiento.
5. ACL, segmentación y servicios de red.
6. Automatización con Python, observabilidad y resolución de incidentes.

## Seguridad documental

Antes de publicar evidencia, se revisarán credenciales, claves, tokens, seriales, MAC, dominios internos y direcciones operativas. Todo resultado se clasificará como planificado, configurado o validado para evitar afirmaciones no comprobadas.

**Autor:** SebastianPintoSec
