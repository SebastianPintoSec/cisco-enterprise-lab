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

## Cisco ISR 4451-X — Inventario observado (21 octubre 2026)

> Salidas recibidas del CLI. Números de serie y direcciones MAC omitidos deliberadamente.

| Parámetro | Observación |
|---|---|
| Chasis | ISR4451-X/K9, revisión V08 |
| IOS XE | 16.06.04 / IOS 16.6.4 Everest (compilación julio 2018) |
| Imagen | UNIVERSALK9 |
| Módulo de puertos | ISR4451-X-4x1GE |
| Fuente | PWR-4450-AC, 450 W |
| Interfaces de datos | GigabitEthernet0/0/0 a 0/0/3 |
| Gestión | GigabitEthernet0 |
| Interfaz candidata para WAN | GigabitEthernet0/0/1 — up/up, sin IP |
| Resto de interfaces | down/down, sin IP |

La salida `show ip interface brief` indica que Gi0/0/1 está activa a nivel físico/enlace, pero **aún no confirma** el extremo RB4011 ni el funcionamiento IPv4.

### Próxima comprobación, antes de cambios

- [ ] `enable` y consultar `show interfaces GigabitEthernet0/0/1`.
- [ ] Consultar `show running-config interface GigabitEthernet0/0/1` y `show ip route`.
- [ ] Confirmar cable Gi0/0/1 ↔ ether6.
- [ ] Revisar licencias con `show license summary` o comandos compatibles antes de habilitar funciones avanzadas.

### Seguridad y ciclo de vida

IOS XE 16.6.4 es una versión antigua. Mantener la administración aislada y revisar avisos de seguridad y compatibilidad antes de permitir exposición externa; no asumir que una fecha de fin de soporte del hardware garantiza parches para esta versión.

## Línea base Cisco — verificación Gi0/0/1 (21 octubre 2026)

Salida CLI recibida y sintetizada sin datos de identificación física:

| Indicador | Resultado |
|---|---|
| Gi0/0/1 estado | up/up |
| Negociación | automática, 1000 Mbps full-duplex, RJ45 |
| MTU | 1500 bytes |
| Errores entrada | 0 (incluye CRC, frame, overruns) |
| Errores salida | 0; output drops 0 |
| Interface resets | 1 (causa y fecha no determinadas) |
| Configuración actual | `no ip address`; `negotiation auto` |
| Tabla de rutas | sin gateway de último recurso y sin rutas IPv4 mostradas |

**Interpretación:** enlace Ethernet negociado correctamente; sin errores de entrada/salida observados. Un reinicio de interfaz acumulado no demuestra por sí mismo fallo físico. Los paquetes multicast recibidos no prueban reachability IPv4. La interfaz no tiene IP, de modo que aún no podemos probar gateway o NAT.

### Próxima configuración controlada (no ejecutada)

Configurar `172.16.255.2/30` en Gi0/0/1 tras confirmar el cable hacia RB4011 ether6, verificar la ruta conectada `C 172.16.255.0/30` y ruta local `L 172.16.255.2/32`, probar ping a `172.16.255.1`. Después, añadir ruta por defecto estática al gateway y probar salida. No guardar permanentemente hasta validar.

## Primer intento de conectividad L3 — 21 octubre 2026

**Configurado en ISR Gi0/0/1:** `description WAN_TO_ZN-LAB-UPSTREAM-RTR01`, `ip address 172.16.255.2 255.255.255.252`, `no shutdown`.

**Resultados observados:**

- `show ip interface brief`: `GigabitEthernet0/0/1 172.16.255.2 manual up/up`.
- `show ip route connected`: `C 172.16.255.0/30` y `L 172.16.255.2/32`, ambas vía Gi0/0/1.
- `show arp`: solo dirección propia `172.16.255.2`; no hay vecino `172.16.255.1` en la salida capturada **antes del ping**.
- `ping 172.16.255.1`: **0/5 respuestas**.
- No hay gateway de último recurso (esperado en esta etapa).

**Diagnóstico abierto:** la dirección IP y la ruta conectada del Cisco se instalaron bien. Confirmar la entrada ARP **después del ping**, el estado del puerto RB4011 y si el cable corresponde a ether6; revisar firewall `input` si ARP resuelve pero ICMP no responde. No agregar NAT, rutas estáticas ni excepciones de firewall hasta aislar el problema.

## Incidente INC-001 — Cable conectado al puerto equivocado

**Causa raíz identificada por el operador:** el cable Ethernet se encontraba conectado al puerto físico incorrecto. Se corrigió la conexión física.

**Síntoma previo:** Gi0/0/1 figuraba `up/up`, la dirección `172.16.255.2/30` y las rutas conectadas eran correctas, pero `ping 172.16.255.1` fallaba (0/5).

**Aprendizaje:** `up/up` confirma enlace Ethernet con *algún* dispositivo, no identidad ni conectividad con el vecino pretendido. Confirmar correspondencia de puertos de extremo a extremo mediante etiquetado, revisión física y LLDP/CDP si está disponible.

**Estado del incidente:** causa física identificada y cableado corregido según reporte; **pendiente de adjuntar salida de nuevo ping y tabla ARP** para registrar validación L3. No se aplicaron cambios de NAT/routing para resolver la causa física.

### Prueba de cierre pendiente

En Cisco: `ping 172.16.255.1`, `show arp`; después, una vez validado el vecino, configurar ruta por defecto de forma controlada y evaluar conectividad externa.

## Verificación posterior a corrección física — 21 octubre 2026

**Conectividad de tránsito IPv4 confirmada** entre Cisco ISR 4451 Gi0/0/1 (`172.16.255.2/30`) y RB4011 (`172.16.255.1/30`).

- `ping 172.16.255.1 repeat 5`: `.!!!!` — 4/5 respuestas (80 %), RTT mínimo/promedio/máximo: 1/1/1 ms.
- `show arp`: vecino `172.16.255.1` resuelto en Gi0/0/1, entrada ARPA; se omitió la MAC física en documentación pública.
- Interpretación: la primera pérdida podría corresponder a resolución inicial de ARP, pero no está demostrada causalmente. Repetir prueba con 20 ecos.
- Estado: **enlace de tránsito L3 validado; conectividad a Internet y aislamiento extremo a extremo pendientes**.

## Verificación WAN exitosa — 21 octubre 2026

**Resultados observados en Cisco ISR 4451-X:**

- Ping del gateway `172.16.255.1` con 20 ecos: **20/20 (100%)**, RTT 1/1/1 ms.
- Se añadió en configuración activa `ip route 0.0.0.0 0.0.0.0 172.16.255.1`.
- Ping externo `8.8.8.8`: **5/5 (100%)**, RTT min/prom/max 2/3/5 ms.
- WAN Cisco: `172.16.255.2/30` en Gi0/0/1; gateway RB4011 `172.16.255.1`.

**Conclusión acotada:** conectividad ICMP IPv4 del propio ISR hacia el gateway e Internet validada. No implica aún DNS funcional, VLANs operativas, seguridad integral ni conectividad a Internet de los futuros hosts internos.

**Pendientes:** comprobar `show ip route` para constatar entrada S*, verificar `traceroute 8.8.8.8`, DNS, pruebas negativas hacia redes restringidas, política firewall y persistencia (`startup-config`). Confirmar que la configuración es segura antes de guardar.

### Aprendizaje

La ruta por defecto envía destinos sin ruta más específica a un siguiente salto directamente alcanzable. En esta etapa el origen del ping es la IP WAN del Cisco, incluida en la regla existente de salida del RB4011. Las futuras VLAN internas necesitarán rutas de retorno y políticas explícitas; no extrapolar el éxito actual a esos segmentos.
