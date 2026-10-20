# LAB-001 — Arquitectura y conectividad WAN

**Estado: EN PROGRESO — enlace físico activo, conectividad IP pendiente.**

## Objetivos

Comprender interfaces L3, direccionamiento /30, gateway, rutas estáticas, NAT, flujo de tráfico, firewall `input` versus `forward`, aislamiento y diagnóstico estructurado.

## Topología

```text
ONT del ISP
   |
MikroTik RB4011 — router upstream existente
   | ether6 (CISCO-LAB-WAN): 172.16.255.1/30
   |
   | Cisco WAN propuesta: 172.16.255.2/30
Cisco ISR 4451 — interfaz WAN por identificar
   |
   | Enlace L3 propuesto: 10.255.0.0/30
Catalyst 3850 — core L3 (IP Services)
   + VLAN 10 / usuarios (propuesta)
   + VLAN 20 / servidores (propuesta)
   + VLAN 30 / seguridad (propuesta)
   + VLAN 99 / gestión (propuesta)
```

Verificar ausencia de solapamientos antes de habilitar nuevas subredes.

## Estado observado en el RB4011

- Interfaz `CISCO-LAB-WAN` con `172.16.255.1/30`.
- Enlace físico `link-ok`, autonegociación a **1 Gbps full-duplex**.
- Ruta por defecto upstream hacia `192.168.1.1`; masquerade por lista WAN.
- Reglas de bloqueo desde Cisco hacia redes internas y segmentos ISP simulados.
- Regla de permiso de salida actual limitada a origen `172.16.255.2`, no a futuras VLAN Cisco.
- Regla final DROP en `forward`. Es necesario revisar el efecto de reglas anteriores (establecidas y administración remota).

**No confundir la negociación Ethernet con conectividad IP comprobada.**

## Lista de verificación

- [ ] Confirmar limpieza del puerto antiguo en la tabla estática VLAN 80.
- [ ] Confirmar listas de interfaces y excepciones relevantes del firewall MikroTik.
- [ ] Inventariar ISR: `show version`, `show inventory`, `show ip interface brief`.
- [ ] Identificar y configurar la interfaz física WAN Cisco.
- [ ] Probar reachability Cisco → 172.16.255.1.
- [ ] Validar ruta por defecto y salida real a Internet.
- [ ] Comprobar pruebas negativas hacia redes protegidas.
- [ ] Antes de habilitar VLAN Cisco, diseñar rutas de retorno y política de salida.
- [ ] Documentar rollback y resultados sanitizados.

## Criterio de finalización

Explicación teórica + configuración comentada + resultados de prueba positivos y negativos + diagnóstico y evidencias.
