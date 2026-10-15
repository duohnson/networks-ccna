# Nota Avanzada - Caso de Estudio: HQ, B1, B2, B3 y NewB

## Introducción

El caso de estudio integra los módulos anteriores con las tecnologías WAN del módulo 3. **HQ** concentra servidores, salida al ISP, NAT y filtrado; **B1, B2 y B3** incorporan VLANs, DHCP, switching y Frame Relay; **NewB** utiliza PPP.

Esta nota explica la configuración de `Notas caso de estudio.txt` y la información del informe. Los resultados del informe pertenecen a la implementación documentada por su autor; no son pruebas nuevas ejecutadas durante la preparación de este resumen.

---

## 1. Topología y Direccionamiento WAN

```text
B1 --\                              LAN servidores: 10.0.1.0/24
B2 --- Nube Frame Relay --- HQ ------+
B3 --/                      | \
                            |  +--- PPP CHAP --- ISP
                         PPP PAP
                            |
                           NewB -- 10.4.5.0/24
```

| Enlace | HQ | Remoto | Detalle |
|---|---|---|---|
| HQ–B1 | `10.255.255.1/30` | `.2/30` | S0/0/0.41, DLCI HQ 41 |
| HQ–B2 | `10.255.255.5/30` | `.6/30` | S0/0/0.42, DLCI HQ 42 |
| HQ–B3 | `10.255.255.9/30` | `.10/30` | S0/0/0.43, DLCI HQ 43 |
| HQ–NewB | `10.255.255.253/30` | `.254/30` | S0/0/1, PAP |
| HQ–ISP | `209.165.201.1/30` | `.2/30` | S0/1/0, CHAP |

En HQ la interfaz física Frame Relay no lleva IP; las direcciones se asignan a las subinterfaces punto a punto. El DLCI 43 debe quedar asociado a `.43`, como señala la corrección de las notas.

LMI es local al enlace con el proveedor: HQ usa `q933a`; B3 usa `ansi` y encapsulación IETF según la actividad. La compatibilidad de encapsulación de datos debe verificarse aparte de LMI; si el PVC está activo pero no pasan paquetes, revisar ambos extremos y la configuración de la nube.

---

## 2. VLANs de las Sucursales

En la tabla, **X = 1, 2 o 3**, según la sucursal. Cada gateway termina en `.1`.

| VLAN | Nombre | Red | Uso |
|---|---|---|---|
| 10 | Admin | `10.X.10.0/24` | Administración de usuarios |
| 20 | Sales | `10.X.20.0/24` | Ventas |
| 30 | Production | `10.X.30.0/24` | Producción |
| 88 | Wireless | `10.X.88.0/24` | Conexión WAN del WRT300N |
| 99 | Mgmt&Native | `10.X.99.0/24` | Gestión de switches y VLAN nativa |

Ejemplo de Router-on-a-Stick en B1:

```text
interface fastEthernet 0/0
 no ip address
 no shutdown
 exit
interface fastEthernet 0/0.10
 encapsulation dot1Q 10
 ip address 10.1.10.1 255.255.255.0
 exit
interface fastEthernet 0/0.99
 encapsulation dot1Q 99 native
 ip address 10.1.99.1 255.255.255.0
 exit
```

Completar `.20`, `.30` y `.88` como en las notas originales. La VLAN nativa del troncal del switch debe coincidir con VLAN 99.

---

## 3. VTP, STP y Seguridad de Puertos

| Función | Configuración del caso |
|---|---|
| VTP | S1 servidor; S2/S3 clientes en cada sucursal |
| Dominio | `xyzcorp` |
| STP raíz principal | S1, prioridad 4096 |
| STP respaldo | S3, prioridad 8192 |
| Puertos de usuarios | S2 Fa0/6, Fa0/11 y Fa0/16 |
| Port Security | Una MAC, sticky, violación shutdown |
| Gestión de switches | VLAN 99, direcciones `.21`, `.22`, `.23` |

Compartir nombre de dominio VTP no extiende automáticamente las VLANs a través de los routers WAN. Los anuncios VTP circulan por los troncales de cada dominio de capa 2.

Ejemplo de puerto de usuario en S2:

```text
interface fastEthernet 0/6
 switchport mode access
 switchport access vlan 10
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown
 exit
```

Un puerto alternativo bloqueado por STP puede ser normal. Un puerto en `err-disabled` por Port Security requiere revisar la causa de la violación.

---

## 4. DHCP y Red Inalámbrica

Para VLANs 10/20/30 se reservan `.1` a `.10`; en VLAN 88 se reservan `.1` a `.24`. El DNS corporativo es `10.0.1.4`.

```text
! Ejemplo en B1
ip dhcp excluded-address 10.1.10.1 10.1.10.10
ip dhcp pool B1_VLAN10
 network 10.1.10.0 255.255.255.0
 default-router 10.1.10.1
 dns-server 10.0.1.4
 exit
```

El informe distingue la **WAN del WRT300N**, que obtiene dirección en VLAN 88, de su **LAN inalámbrica `10.X.40.0/24`**, que tiene DHCP propio. No deben confundirse con una única subred.

El informe usa WEP y administración remota del WRT300N por requisitos del ejercicio. Se documentan como parámetros de esa actividad, no como una configuración de seguridad para desplegar fuera del laboratorio.

---

## 5. EIGRP y Sumarización

HQ y B1–B3 usan **EIGRP AS 100**. Las sucursales resumen sus redes como `10.1.0.0/16`, `10.2.0.0/16` y `10.3.0.0/16`.

Ejemplo de B1:

```text
router eigrp 100
 no auto-summary
 network 10.0.0.0 0.255.255.255
 passive-interface default
 no passive-interface serial 0/0/0
 exit
interface serial 0/0/0
 ip summary-address eigrp 100 10.1.0.0 255.255.0.0
 exit
ip route 0.0.0.0 0.0.0.0 10.255.255.1
```

El resumen incluye más direcciones que las VLANs existentes. No crea conectividad hacia subredes que no estén realmente configuradas.

En HQ, habilitar las adyacencias en `.41`, `.42` y `.43`, y conservar pasivas las LANs. Las notas también habilitan Fa0/1; comprobar su conexión real antes de atribuirle un vecino.

| Router | Ruta adicional |
|---|---|
| HQ | Predeterminada hacia ISP por S0/1/0 |
| HQ | `10.4.5.0/24` por S0/0/1 hacia NewB |
| B1 | Predeterminada hacia `10.255.255.1` |
| B2 | Predeterminada hacia `10.255.255.5` |
| B3 | Predeterminada hacia `10.255.255.9` |
| NewB | Predeterminada hacia `10.255.255.253`, indicada en el informe |

---

## 6. NAT Centralizada en HQ

```text
ip access-list standard NAT_LIST
 permit 10.0.0.0 0.255.255.255
 exit
ip nat pool XYZCORP 209.165.200.241 209.165.200.245 netmask 255.255.255.248
ip nat inside source list NAT_LIST pool XYZCORP
ip nat inside source static 10.0.1.2 209.165.200.246
```

| Tipo | Direcciones |
|---|---|
| NAT dinámica sin overload | Cinco direcciones: `.241` a `.245` |
| NAT estática del servidor web | `10.0.1.2` ↔ `209.165.200.246` |
| Red del bloque | `209.165.200.240/29` |
| Broadcast | `209.165.200.247` |

Marcar `ip nat inside` en Fa0/0, S0/0/0.41, .42, .43 y S0/0/1. Marcar `ip nat outside` en S0/1/0.

**Comprobación de retorno:** el ISP debe poder alcanzar `209.165.200.240/29` a través de HQ. Si esa ruta no viene preconfigurada en la actividad, la ruta correspondiente sería:

```text
! En ISP, completar solo si no existe ya una ruta válida
ip route 209.165.200.240 255.255.255.248 209.165.201.1
```

Esta línea se presenta como requisito de conectividad a comprobar; no aparece en las notas originales. No añadir `overload` por costumbre: el caso distingue NAT dinámica de PAT y reserva `.246` para el servidor.

---

## 7. ACL FIREWALL y su Alcance Real

En HQ:

```text
ip access-list extended FIREWALL
 permit tcp any host 209.165.200.246 eq 80
 permit tcp any any established
 permit icmp any any echo-reply
 deny ip any any
 exit
interface serial 0/1/0
 ip access-group FIREWALL in
 exit
```

La primera regla usa la **dirección global interna** del servidor porque el paquete llega desde el ISP antes de la traducción de destino del NAT clásico de esta práctica.

| Regla | Efecto |
|---|---|
| TCP destino `.246:80` | Permite HTTP hacia el servidor publicado |
| TCP `established` | Coincide con paquetes que tienen ACK o RST |
| ICMP `echo-reply` | Permite respuestas de ping desde fuera |
| `deny ip any any` | Rechaza el resto del tráfico IPv4 entrante |

**`established` no mantiene una tabla de sesiones** ni verifica que una conexión se haya iniciado desde dentro. La ACL tampoco permite de forma general las respuestas UDP, como DNS externo. El DNS interno y las pruebas permitidas deben analizarse por su recorrido, no suponiendo que esta lista funciona como un firewall con estado.

---

## 8. Comprobación del Caso

| Bloque | Evidencia que revisar |
|---|---|
| Frame Relay | PVC activos, DLCI en subinterfaz correcta y ping a B1/B2/B3 |
| PPP | Estado de enlace y autenticación con NewB e ISP |
| VLANs | `show vlan brief`, `show interfaces trunk` |
| STP | `show spanning-tree vlan 10` y raíz esperada |
| DHCP | IP, máscara, gateway y DNS obtenidos por cada PC |
| EIGRP | `show ip eigrp neighbors` y tres resúmenes `/16` en HQ |
| NAT | Traducciones dinámicas y entrada estática del servidor |
| ACL | HTTP externo permitido y solicitud de ping externa bloqueada |

El informe documenta conectividad entre sedes, DHCP, traducciones NAT, acceso HTTP externo al servidor y bloqueo de ping externo. Para reproducir esos resultados se debe abrir la actividad `.pka` y repetir las pruebas; un resumen no sustituye esa validación.

---

## Resumen Ejecutivo

- HQ concentra la WAN, las rutas de salida, NAT y la ACL de entrada.
- B1–B3 repiten el esquema de VLANs con direccionamiento propio por sede.
- EIGRP anuncia resúmenes `/16`; NewB se alcanza mediante ruta estática.
- El pool dinámico contiene cinco direcciones y la sexta utilizable se reserva al servidor.
- `established` filtra bits TCP, no valida sesiones.
- Separar resultados documentados de pruebas nuevas al repasar el caso.

## Referencias

- [Notas del caso de estudio](../../Topolog%C3%ADas/Topolog%C3%ADas%20de%20Modulo%203/Notas%20caso%20de%20estudio.txt).
- [Informe del caso de estudio](../../Topolog%C3%ADas/Topolog%C3%ADas%20de%20Modulo%203/Caso%20de%20estudio%20informe.pdf).
- [Actividad Packet Tracer](../../Topolog%C3%ADas/Topolog%C3%ADas%20de%20Modulo%203/CASO%20ESTUDIO%20MOD%203.pka).
