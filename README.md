# Network Scheme - Topologías y redes.

Documentación y estudio de redes.

## Mis Topologías de Red:

![Simulación de red actual](/Imagenes/Captura%202.png)
![Simulación de red básica](/Imagenes/test_lab.png)

## Descripción

Este es un repositorio de documentación, estudio, resúmenes y topologías de red.

## Estructura del Proyecto

```text
network-scheme/
├── Documentos/                 Guías de referencia y documentación técnica
├── Escaneo tShark/             Prácticas y capturas de tráfico con tShark
├── Imagenes/                   Referencias visuales y capturas de pantalla
├── Red WAN - Mega Topología/   Topología de red de área amplia (WAN)
├── Resumenes/                  Notas completas de estudio para CCNA (Módulos 1, 2 y 3)
├── Topologías/                 Otras topologías de red en Packet Tracer
├── RED_SIMPLE.MD               Documentación de topología de red simple
├── network-isp.pkt             Simulación de red ISP en Cisco Packet Tracer
└── README.md                   Este archivo
```

## Archivos de Referencia

Los documentos PDF en `Documentos/` contienen comandos y configuraciones CCNA para enrutamiento, VLANs y seguridad básica.

## Requisitos

- Cisco Packet Tracer (versión 7.0 o superior)
- Conocimientos básicos de enrutamiento y NAT

---

# Redes CCNA con CISCO - Notas y Resúmenes

## Notas de Estudio CCNA 

En la carpeta de [Resumenes](./Resumenes/) encontrarás las notas completas de estudio para los tres módulos de certificación CCNA (Módulo 1: Fundamentos de Redes, Módulo 2: Switching, Routing & Wireless, y Módulo 3: Enterprise Networking, Security & Automation), incluyendo tanto las notas de clase originales como las notas avanzadas detalladas.

### Módulo 1 - Fundamentos de Redes CCNA

#### Inicio Rápido y Evaluación
- **[Definición de Redes](/Resumenes/Modulo%201/General.md)** - Conceptos iniciales: dispositivos, IPs, QoS, tipos de red
- **[Conceptos de Subneteo](/Resumenes/Modulo%201/Subneteo_Conceptos.md)** - VLSM, CIDR y cálculo práctico de subredes
- **[Práctica de Examen Módulo 1](/Resumenes/Modulo%201/Practica_Examen_Modulo1.md)** - Preguntas de opción múltiple y ejercicios prácticos de repaso

#### Capítulos 1-3: Protocolos y Modelos de Red
| Recurso | Descripción |
|---------|-------------|
| [Clase 1](./Resumenes/Modulo%201/Notas%20de%20clase%201.txt) | Protocolos, Capa OSI, TCP/IP |
| [Clase 2](./Resumenes/Modulo%201/Notas%20de%20clase%202.txt) | Suite de Protocolos |
| [Clase 3](./Resumenes/Modulo%201/Notas%20de%20clase%203.txt) | Modelo OSI, PDU, Encapsulación |
| **[→ Nota Avanzada 1](./Resumenes/Modulo%201/Nota%20avanzada%20de%20clase%201.md)** | **Elementos de comunicación, requisitos de protocolos, tipos de entrega, convergencia** |
| **[→ Nota Avanzada 2-3](./Resumenes/Modulo%201/Nota%20avanzada%20de%20clase%202-3.md)** | **Suite de protocolos OSI vs TCP/IP, Puertos, MAC, CSMA/CD y CA** |

#### Capítulos 4-5: Capa de Enlace de Datos
| Recurso | Descripción |
|---------|-------------|
| [Clase 4](./Resumenes/Modulo%201/Notas%20de%20clase%204%20-%20ETHERNET%20SWITCHING.txt) | Ethernet, ARP, Direcciones MAC |
| [Clase 5](./Resumenes/Modulo%201/Notas%20de%20clase%205%20-%20ACCESO%20AL%20MEDIO.txt) | Control de Acceso, CSMA |
| **[→ Nota Avanzada](./Resumenes/Modulo%201/Nota%20avanzada%20de%20clase%204-5.md)** | **Direcciones MAC en profundidad, ARP con ejemplos, tabla de conmutación, CSMA/CD vs CSMA/CA, VLAN** |

#### Capítulos 6-7: Capa Física y Medios
| Recurso | Descripción |
|---------|-------------|
| [Clase 6](./Resumenes/Modulo%201/Notas%20de%20clase%206.txt) | Medios inalámbricos, cables |
| [Clase 6 Ext.](./Resumenes/Modulo%201/Notas%20de%20clase%206%20parte%202%20mas%20extendido.txt) | Modelo OSI, Capa de Red |
| **[→ Nota Avanzada](./Resumenes/Modulo%201/Nota%20avanzada%20de%20clase%206-7.md)** | **Cobre (tipos, PoE), Fibra (monomodo/multimodo), WiFi (bandas, estándares), codificación, conectores** |

#### Capítulos 8-9: Capa de Red (IPv4/IPv6)
| Recurso | Descripción |
|---------|-------------|
| [Clase 7](./Resumenes/Modulo%201/Notas%20de%20clase%207.txt) | IPv6, NDP, Direccionamiento IPv4 |
| [Clase 8](./Resumenes/Modulo%201/Notas%20de%20clase%208.txt) | IPv6 avanzado |
| **[→ Nota Avanzada](./Resumenes/Modulo%201/Nota%20avanzada%20de%20clase%208-9.md)** | **IPv4 completo, CIDR, subneteo VLSM con ejemplos, IPv6 estructura jerárquica, NDP, migración, routing, ICMP** |

#### Capítulos 14-15: Capa de Transporte y Aplicación
| Recurso | Descripción |
|---------|-------------|
| [Clase 9](./Resumenes/Modulo%201/Notas%20de%20clase%209.txt) | TCP/UDP, puertos |
| [Clase 10](./Resumenes/Modulo%201/Notas%20de%20clase%2010.txt) | Protocolos de aplicación |
| **[→ Nota Avanzada 10](./Resumenes/Modulo%201/Nota%20avanzada%20de%20clase%2010.md)** | **Protocolos HTTP, DNS, DHCP, Correo Electrónico (SMTP, POP, IMAP)** |
| **[→ Nota Avanzada 14-15](./Resumenes/Modulo%201/Nota%20avanzada%20de%20clase%2014-15.md)** | **TCP vs UDP, handshake, control de flujo, HTTP/HTTPS, DNS, DHCP, SMTP/POP3/IMAP, FTP/SFTP, SSH, flujo completo** |

#### Capítulo 11: Configuración de Dispositivos
| Recurso | Descripción |
|---------|-------------|
| [Clase 11](./Resumenes/Modulo%201/Notas%20de%20clase%2011.txt) | Interfaz, Telnet, SSH |
| **[→ Nota Avanzada](./Resumenes/Modulo%201/Nota%20avanzada%20de%20clase%2011.md)** | **Modos IOS, IPv4/IPv6, seguridad (console/enable/VTY), usuarios, Telnet vs SSH, backup, comandos útiles** |

### Módulo 2 - Switching, Routing, and Wireless Essentials

#### Clases 1 y 2: VLANs y Enrutamiento
| Recurso | Descripción |
|---------|-------------|
| [Clase 1](./Resumenes/Modulo%202/Notas%20de%20clase%201%20CAP%201%20BASIC%20SWITCHING.txt) | Basic Switching, configuración inicial |
| [Clase 2](./Resumenes/Modulo%202/Notas%20de%20clase%202%20CAP%203%20VLAN%20AND%20TRUNKS.txt) | Creación de VLANs, Puertos Access y Trunks |
| **[→ Nota Avanzada 1-2](./Resumenes/Modulo%202/Nota%20avanzada%20de%20clase%201-2.md)** | **Conceptos de switching, VLANs, Troncales (802.1Q), Inter-VLAN Routing** |

#### Clases 3-6: Enrutamiento, STP, EtherChannel y DHCP
| Recurso | Descripción |
|---------|-------------|
| [Clase 3](./Resumenes/Modulo%202/Notas%20de%20clase%203%20CAP%204%20%20INTER%20VLAN%20ROUTING.txt) | Inter-VLAN Routing, SVI |
| [Clase 4](./Resumenes/Modulo%202/Notas%20de%20clase%204%20%20STP%20%20%20SPANNING%20TREE%20PROTOCOL.txt) | Spanning Tree Protocol (STP) |
| [Clase 5](./Resumenes/Modulo%202/Notas%20de%20clase%205%20CAP%206%20ETHERCHANNEL.txt) | EtherChannel, LACP y PAgP |
| [Clase 6](./Resumenes/Modulo%202/Notas%20de%20clase%206%20DHCP%20Y%20SLAAC%20Y%20DHCPV6.txt) | DHCPv4, SLAAC, DHCPv6 |
| **[→ Nota Avanzada 3](./Resumenes/Modulo%202/Nota%20avanzada%20de%20clase%203.md)** | **Enrutamiento Inter-VLAN, subinterfaces, SVI detallado** |
| **[→ Nota Avanzada 4](./Resumenes/Modulo%202/Nota%20avanzada%20de%20clase%204.md)** | **STP, RSTP, PVST+, BPDU Guard, PortFast** |
| **[→ Nota Avanzada 5](./Resumenes/Modulo%202/Nota%20avanzada%20de%20clase%205.md)** | **Agregación de enlaces, modos y requisitos** |
| **[→ Nota Avanzada 6](./Resumenes/Modulo%202/Nota%20avanzada%20de%20clase%206.md)** | **DHCP Relay, asignación dinámica IPv6, EUI-64** |

#### Clases 7-10: FHRP, Seguridad LAN, Switch Security y WLAN
| Recurso | Descripción |
|---------|-------------|
| [Clase 7](./Resumenes/Modulo%202/Notas%20de%20clase%207%20CAP%209%20FHRP.txt) | FHRP: HSRP, VRRP, GLBP |
| [Clase 8](./Resumenes/Modulo%202/Notas%20de%20clase%208%20CAP%2010%20LAN%20SECURITY.txt) | Ataques de Capa 2, mejores prácticas |
| [Clase 9](./Resumenes/Modulo%202/Notas%20de%20clase%209%20CAP%2011%20SWITCH%20SECURITY.txt) | Port Security, DHCP Snooping, DAI |
| [Clase 10](./Resumenes/Modulo%202/Notas%20de%20clase%2010%20CAP%2012-13%20WLAN.txt) | WLAN, WiFi, estándares 802.11, WPA2/WPA3 |
| **[→ Nota Avanzada 7](./Resumenes/Modulo%202/Nota%20avanzada%20de%20clase%207.md)** | **HSRP, VRRP, GLBP, gateway virtual, preemption** |
| **[→ Nota Avanzada 8](./Resumenes/Modulo%202/Nota%20avanzada%20de%20clase%208.md)** | **MAC Flooding, VLAN Hopping, DHCP/ARP Spoofing, defensa en profundidad** |
| **[→ Nota Avanzada 9](./Resumenes/Modulo%202/Nota%20avanzada%20de%20clase%209.md)** | **Port Security, DHCP Snooping, DAI, IP Source Guard** |
| **[→ Nota Avanzada 10](./Resumenes/Modulo%202/Nota%20avanzada%20de%20clase%2010.md)** | **802.11 (WiFi 4/5/6), seguridad WEP→WPA3, CAPWAP, amenazas WLAN** |

#### Clases 11-12: Enrutamiento y Rutas Estáticas
| Recurso | Descripción |
|---------|-------------|
| [Clase 11](./Resumenes/Modulo%202/Notas%20de%20clase%2011%20CAP%2014%20ENRUTAMIENTO.txt) | Conceptos de enrutamiento, protocolos dinámicos, tabla de enrutamiento |
| [Clase 12](./Resumenes/Modulo%202/Notas%20de%20clase%2012%20CAP%2015%20RUTAS%20ESTATICAS.txt) | Rutas estáticas IPv4/IPv6, sumarización |
| **[→ Nota Avanzada 11](./Resumenes/Modulo%202/Nota%20avanzada%20de%20clase%2011.md)** | **Rutas estáticas vs dinámicas, AD, métricas, RIP/EIGRP/OSPF, BGP, clases IP** |
| **[→ Nota Avanzada 12](./Resumenes/Modulo%202/Nota%20avanzada%20de%20clase%2012.md)** | **Tipos de rutas estáticas, sintaxis IPv4/IPv6, sumarización, rutas flotantes** |

### Módulo 3 - Enterprise Networking, Security, and Automation

#### Capítulos 1-2: Protocolo OSPF (Área Única y Multiárea)
| Recurso | Descripción |
|---------|-------------|
| [Clase 1](./Resumenes/Modulo%203/NOTAS%20CLASE%201%20MOD%203.txt) | Fundamentos de OSPF, Dijkstra, métricas, DR/BDR |
| [Clase 2](./Resumenes/Modulo%203/NOTAS%20CLASE%202%20MOD%203.txt) | OSPF Multiárea, roles de routers, sumarización ABR, LSAs |
| **[→ Nota Avanzada 1](./Resumenes/Modulo%203/Nota%20avanzada%20de%20clase%201.md)** | **Fundamentos de OSPFv2/OSPFv3, paquetes, adyacencias, elección DR/BDR, costo y verificación** |
| **[→ Nota Avanzada 2](./Resumenes/Modulo%203/Nota%20avanzada%20de%20clase%202.md)** | **OSPF Multiárea, ABR/ASBR, LSA tipos 1-5, sumarización inter-área, reference-bandwidth, timers** |

#### Capítulo 3: Seguridad de Red y Listas de Control de Acceso (ACLs)
| Recurso | Descripción |
|---------|-------------|
| [Clase 3](./Resumenes/Modulo%203/NOTAS%20CLASE%203%20ACLS.txt) | Tríada CIA, listas de control de acceso estándar y extendidas |
| **[→ Nota Avanzada 3](./Resumenes/Modulo%203/Nota%20avanzada%20de%20clase%203.md)** | **Seguridad CIA, lógica Wildcard, ACLs estándar vs extendidas, reglas de ubicación y escenarios prácticos** |

#### Ver todas las notas completas
→ **[Acceder a Resumenes/README.md](./Resumenes/README.md)** para guía completa y estructura detallada

========================================================================================================================================================

## Filtros utiles en Wireshark:

![Wireshark](/Imagenes/tshark.jpg)
