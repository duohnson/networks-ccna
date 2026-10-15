# Notas de Estudio CCNA - Resumenes

Este repositorio contiene mis notas completas del curso CCNA (Cisco Certified Network Associate), incluyendo tanto los resúmenes de clase como notas avanzadas ampliadas con más profundidad.

## Estructura del Repositorio

```
Resumenes/
├── Modulo 1/
│   ├── General.md                           (Conceptos generales)
│   ├── Subneteo_Conceptos.md               (Subneting avanzado)
│   ├── Notas de clase 1-11.txt             (Notas originales por clase)
│   └── Nota avanzada de clase [X-Y].md    (Versiones expandidas)
├── Modulo 2/
│   ├── Notas de clase 1-12.txt              (Notas originales por clase)
│   └── Nota avanzada de clase [X].md       (Versiones expandidas)
└── Modulo 3/
    ├── NOTAS CLASE ... .txt               (Originales 1-5 y resúmenes 6-15)
    ├── Nota avanzada de clase [1-15].md  (Versiones expandidas)
    └── Nota avanzada de caso de estudio.md (Integración HQ y sucursales)
```

---

## Módulo 1: Fundamentos de Redes

### Conceptos Introductorios y Evaluación
- **[Definición de Redes](/Resumenes/Modulo%201/General.md)** - Dispositivos, IPs, QoS, tipos de red, convergencia
- **[Conceptos de Subneteo](/Resumenes/Modulo%201/Subneteo_Conceptos.md)** - VLSM, CIDR, cálculo de subredes
- **[Práctica de Examen Módulo 1](/Resumenes/Modulo%201/Practica_Examen_Modulo1.md)** - Ejercicios prácticos y preguntas de repaso

### Capítulos 1-3: Protocolos y Modelos de Red
**Notas Originales:**
- [Clase 1 - Protocolos y Capa OSI](/Resumenes/Modulo%201/Notas%20de%20clase%201.txt)
- [Clase 2 - Suite de Protocolos](/Resumenes/Modulo%201/Notas%20de%20clase%202.txt)
- [Clase 3 - Modelo OSI y Encapsulación](/Resumenes/Modulo%201/Notas%20de%20clase%203.txt)

**Nota Avanzada:**
- [**Nota Avanzada - Clase 1**](/Resumenes/Modulo%201/Nota%20avanzada%20de%20clase%201.md)
  - Elementos de comunicación profundizados
  - Requisitos de protocolos explicados
  - Tipos de entrega (unicast, multicast, broadcast)
  - Protocolo OSI vs TCP/IP
  - Convergencia de redes
- [**Nota Avanzada - Clases 2-3**](/Resumenes/Modulo%201/Nota%20avanzada%20de%20clase%202-3.md)
  - Capas del Modelo OSI y TCP/IP
  - Uso de puertos y PDUs
  - Subcapa MAC y FCS
  - Métodos de acceso al medio (CSMA/CD y CSMA/CA)

### Capítulos 4-5: Capa de Enlace de Datos
**Notas Originales:**
- [Clase 4 - Ethernet Switching](/Resumenes/Modulo%201/Notas%20de%20clase%204%20-%20ETHERNET%20SWITCHING.txt)
- [Clase 5 - Control de Acceso al Medio](/Resumenes/Modulo%201/Notas%20de%20clase%205%20-%20ACCESO%20AL%20MEDIO.txt)

**Nota Avanzada:**
- [**Nota Avanzada - Capítulos 4-5**](/Resumenes/Modulo%201/Nota%20avanzada%20de%20clase%204-5.md)
  - Direcciones MAC en profundidad
  - ARP (Address Resolution Protocol) con ejemplos
  - Tabla de conmutación en switches
  - Conmutación vs Enrutamiento
  - CSMA/CD y CSMA/CA
  - Estructura de tramas Ethernet
  - VLAN introducción

### Capítulos 6-7: Capa Física y Medios de Transmisión
**Notas Originales:**
- [Clase 6 - Medios Inalámbricos](/Resumenes/Modulo%201/Notas%20de%20clase%206.txt)
- [Clase 6 Extendida - Capa de Red](/Resumenes/Modulo%201/Notas%20de%20clase%206%20parte%202%20mas%20extendido.txt)

**Nota Avanzada:**
- [**Nota Avanzada - Capítulos 6-7**](/Resumenes/Modulo%201/Nota%20avanzada%20de%20clase%206-7.md)
  - Capa Física detallada
  - Cobre: tipos de cables, PoE
  - Fibra Óptica: monomodo vs multimodo
  - Inalámbrico: bandas, estándares WiFi
  - Codificación de tramas
  - Conectores y interfaces
  - Velocidades de transmisión

### Capítulos 8-9: Capa de Red (IPv4 e IPv6)
**Notas Originales:**
- [Clase 7 - IPv6 y Direccionamiento IPv4](/Resumenes/Modulo%201/Notas%20de%20clase%207.txt)
- [Clase 8 - IPv6 Avanzado](/Resumenes/Modulo%201/Notas%20de%20clase%208.txt)

**Nota Avanzada:**
- [**Nota Avanzada - Capítulos 8-9**](/Resumenes/Modulo%201/Nota%20avanzada%20de%20clase%208-9.md)
  - IPv4: estructura y clases
  - CIDR y subneteo VLSM
  - Ejemplo práctico de subneteo
  - IPv6: estructura jerárquica
  - Tipos de direcciones IPv6 (GUA, ULA, LLA)
  - NDP (Neighbor Discovery Protocol)
  - Técnicas de migración IPv4→IPv6
  - Tabla de enrutamiento
  - ICMP, Ping, Traceroute

### Capítulos 14-15: Capa de Transporte y Aplicación
**Notas Originales:**
- [Clase 9 - TCP/UDP](/Resumenes/Modulo%201/Notas%20de%20clase%209.txt)
- [Clase 10 - Capa de Aplicación](/Resumenes/Modulo%201/Notas%20de%20clase%2010.txt)

**Nota Avanzada:**
- [**Nota Avanzada - Clase 10**](/Resumenes/Modulo%201/Nota%20avanzada%20de%20clase%2010.md)
  - Protocolos de Capa de Aplicación (HTTP, DNS, DHCP)
  - Correo electrónico (SMTP, POP, IMAP)
- [**Nota Avanzada - Capítulos 14-15**](/Resumenes/Modulo%201/Nota%20avanzada%20de%20clase%2014-15.md)
  - TCP vs UDP comparación completa
  - Three-Way Handshake
  - Ventanas deslizantes (TCP control de flujo)
  - Puertos: bien conocidos, registrados, dinámicos
  - HTTP/HTTPS: métodos, códigos de respuesta, TLS
  - DNS: proceso de resolución, registros
  - DHCP: proceso DORA
  - FTP/SFTP: diferencias
  - SMTP/POP3/IMAP: correo electrónico
  - SSH: autenticación segura
  - Flujo de datos completo (ejemplo: acceder a Google)

### Capítulo 11: Configuración de Dispositivos
**Notas Originales:**
- [Clase 11 - Interfaz, Telnet, SSH](/Resumenes/Modulo%201/Notas%20de%20clase%2011.txt)

**Nota Avanzada:**
- [**Nota Avanzada - Capítulo 11**](/Resumenes/Modulo%201/Nota%20avanzada%20de%20clase%2011.md)
  - Modos de Cisco IOS
  - Configuración de interfaces IPv4/IPv6
  - Securización: console, enable, VTY
  - Usuarios locales y autenticación
  - Telnet vs SSH
  - Configuración SSH paso a paso
  - Switch: configuración de acceso remoto
  - Comandos de verificación útiles
  - Guardado de configuración
  - Backup y restauración

---

## Módulo 2: Switching, Routing, and Wireless Essentials

### Clases 1 y 2: VLANs y Enrutamiento Inter-VLAN
**Notas Originales:**
- [Clase 1 - Basic Switching](/Resumenes/Modulo%202/Notas%20de%20clase%201%20CAP%201%20BASIC%20SWITCHING.txt)
- [Clase 2 - VLAN and Trunks](/Resumenes/Modulo%202/Notas%20de%20clase%202%20CAP%203%20VLAN%20AND%20TRUNKS.txt)

**Nota Avanzada:**
- [**Nota Avanzada - Clases 1 y 2**](/Resumenes/Modulo%202/Nota%20avanzada%20de%20clase%201-2.md)
  - Conceptos básicos de switching
  - Beneficios y tipos de VLANs
  - Puertos de acceso y troncales (802.1Q)
  - Enrutamiento Inter-VLAN (Router-on-a-Stick y SVI)
  - Resolución de problemas básica

### Clase 3: Enrutamiento Inter-VLAN
**Nota Original:**
- [Clase 3 - Inter VLAN Routing](/Resumenes/Modulo%202/Notas%20de%20clase%203%20CAP%204%20%20INTER%20VLAN%20ROUTING.txt)

**Nota Avanzada:**
- [**Nota Avanzada - Clase 3**](/Resumenes/Modulo%202/Nota%20avanzada%20de%20clase%203.md)
  - Métodos de enrutamiento (Legacy, Router-on-a-Stick, SVI)
  - Configuración detallada y Troubleshooting

### Clase 4: Spanning Tree Protocol (STP)
**Nota Original:**
- [Clase 4 - STP](/Resumenes/Modulo%202/Notas%20de%20clase%204%20%20STP%20%20%20SPANNING%20TREE%20PROTOCOL.txt)

**Nota Avanzada:**
- [**Nota Avanzada - Clase 4**](/Resumenes/Modulo%202/Nota%20avanzada%20de%20clase%204.md)
  - Problemas de bucles de Capa 2
  - Funcionamiento, Puente Raíz y roles de puertos
  - Variantes (PVST+, RSTP)
  - PortFast y BPDU Guard

### Clase 5: EtherChannel
**Nota Original:**
- [Clase 5 - EtherChannel](/Resumenes/Modulo%202/Notas%20de%20clase%205%20CAP%206%20ETHERCHANNEL.txt)

**Nota Avanzada:**
- [**Nota Avanzada - Clase 5**](/Resumenes/Modulo%202/Nota%20avanzada%20de%20clase%205.md)
  - Ventajas de agregación de enlaces
  - Protocolos de negociación (PAgP, LACP)
  - Configuración y requisitos

### Clase 6: DHCP, SLAAC y DHCPv6
**Nota Original:**
- [Clase 6 - DHCP y SLAAC](/Resumenes/Modulo%202/Notas%20de%20clase%206%20DHCP%20Y%20SLAAC%20Y%20DHCPV6.txt)

**Nota Avanzada:**
- [**Nota Avanzada - Clase 6**](/Resumenes/Modulo%202/Nota%20avanzada%20de%20clase%206.md)
  - DHCPv4 y Agente de Retransmisión (Relay)
  - Asignación dinámica IPv6 (SLAAC, EUI-64)
  - DHCPv6 Sin Estado y Con Estado

### Clase 7: FHRP (First Hop Redundancy Protocols)
**Nota Original:**
- [Clase 7 - FHRP](/Resumenes/Modulo%202/Notas%20de%20clase%207%20CAP%209%20FHRP.txt)

**Nota Avanzada:**
- [**Nota Avanzada - Clase 7**](/Resumenes/Modulo%202/Nota%20avanzada%20de%20clase%207.md)
  - Problema del gateway único
  - HSRP (Hot Standby Router Protocol)
  - VRRP (Virtual Router Redundancy Protocol)
  - GLBP (Gateway Load Balancing Protocol)
  - Comparación de protocolos FHRP

### Clase 8: Conceptos de Seguridad en LAN
**Nota Original:**
- [Clase 8 - LAN Security](/Resumenes/Modulo%202/Notas%20de%20clase%208%20CAP%2010%20LAN%20SECURITY.txt)

**Nota Avanzada:**
- [**Nota Avanzada - Clase 8**](/Resumenes/Modulo%202/Nota%20avanzada%20de%20clase%208.md)
  - Ataques de Capa 2: MAC Flooding, VLAN Hopping, DHCP/ARP Spoofing
  - Mejores prácticas de seguridad en la LAN
  - Defensa en profundidad

### Clase 9: Configuración de Seguridad en Switches
**Nota Original:**
- [Clase 9 - Switch Security](/Resumenes/Modulo%202/Notas%20de%20clase%209%20CAP%2011%20SWITCH%20SECURITY.txt)

**Nota Avanzada:**
- [**Nota Avanzada - Clase 9**](/Resumenes/Modulo%202/Nota%20avanzada%20de%20clase%209.md)
  - Port Security (Sticky, violaciones, err-disabled)
  - DHCP Snooping (trusted/untrusted)
  - Dynamic ARP Inspection (DAI)
  - IP Source Guard

### Clase 10: WLAN (Wireless LAN)
**Nota Original:**
- [Clase 10 - WLAN](/Resumenes/Modulo%202/Notas%20de%20clase%2010%20CAP%2012-13%20WLAN.txt)

**Nota Avanzada:**
- [**Nota Avanzada - Clase 10**](/Resumenes/Modulo%202/Nota%20avanzada%20de%20clase%2010.md)
  - Estándares 802.11 (WiFi 4/5/6)
  - Bandas 2.4 GHz vs 5 GHz
  - Seguridad: WEP → WPA → WPA2 → WPA3
  - Modos de AP: Autónomo vs Lightweight (CAPWAP)
  - Amenazas WLAN (Rogue AP, Evil Twin)

### Clase 11: Conceptos de Enrutamiento
**Nota Original:**
- [Clase 11 - Enrutamiento](/Resumenes/Modulo%202/Notas%20de%20clase%2011%20CAP%2014%20ENRUTAMIENTO.txt)

**Nota Avanzada:**
- [**Nota Avanzada - Clase 11**](/Resumenes/Modulo%202/Nota%20avanzada%20de%20clase%2011.md)
  - Rutas estáticas vs Protocolos de enrutamiento dinámico
  - Tabla de enrutamiento: 7 elementos clave
  - Distancias Administrativas y Métricas
  - Clasificación: IGP (Vector Distancia, Estado de Enlace) y EGP (BGP)
  - Configuración de RIP, EIGRP y OSPF
  - Clases de direccionamiento IP
  - Niveles de rutas en la tabla de enrutamiento

### Clase 12: Rutas Estáticas (IPv4 e IPv6)
**Nota Original:**
- [Clase 12 - Rutas Estáticas](/Resumenes/Modulo%202/Notas%20de%20clase%2012%20CAP%2015%20RUTAS%20ESTATICAS.txt)

**Nota Avanzada:**
- [**Nota Avanzada - Clase 12**](/Resumenes/Modulo%202/Nota%20avanzada%20de%20clase%2012.md)
  - Tipos de rutas estáticas (estándar, predeterminada, flotante, hacia host)
  - Sintaxis IPv4 e IPv6 con ejemplos
  - Sumarización de rutas IPv4 e IPv6
  - Ruta completamente especificada
  - Verificación de rutas estáticas

---

## Módulo 3: Enterprise Networking, Security, and Automation

### Clase 1 (Capítulo 1): Fundamentos de OSPF de Área Única
**Nota Original:**
- [Clase 1 - Fundamentos de OSPF](/Resumenes/Modulo%203/NOTAS%20CLASE%201%20MOD%203.txt)

**Nota Avanzada:**
- [**Nota Avanzada - Clase 1**](/Resumenes/Modulo%203/Nota%20avanzada%20de%20clase%201.md)
  - Protocolo OSPFv2 y OSPFv3: algoritmo Dijkstra (SPF) y costo métrico
  - Las 3 estructuras de datos (Vecinos, LSDB, Enrutamiento)
  - Los 5 tipos de paquetes OSPF (Hello, DBD, LSR, LSU, LSAck)
  - Estados de adyacencia (Down → Init → Two-Way → ExStart → Exchange → Loading → Full)
  - Redes multiacceso: Roles DR, BDR y DROther (fórmula N(N-1)/2)
  - Proceso de elección de DR/BDR (Prioridad y Router ID)
  - Configuración paso a paso de OSPFv2 y OSPFv3 con interfaces pasivas

### Clase 2 (Capítulo 2): OSPF Multiárea, Métricas, Temporizadores y LSAs
**Nota Original:**
- [Clase 2 - OSPF Multiárea](/Resumenes/Modulo%203/NOTAS%20CLASE%202%20MOD%203.txt)

**Nota Avanzada:**
- [**Nota Avanzada - Clase 2**](/Resumenes/Modulo%203/Nota%20avanzada%20de%20clase%202.md)
  - Jerarquía multiárea: Área 0 (Backbone) y áreas estándar
  - Roles de routers: Interno, Troncal, ABR y ASBR
  - Taxonomía completa de LSAs (Tipo 1 al 5) y códigos en tabla de enrutamiento (`O`, `O IA`, `O E1/E2`)
  - Sumarización inter-área en ABR (`area range`) con desglose binario
  - Optimización de costo: calibración de `auto-cost reference-bandwidth` para enlaces Gigabit y 10G
  - Ajuste de temporizadores Hello y Dead
  - Propagación de ruta por defecto (`default-information originate`)

### Clase 3 (Capítulo 3): Seguridad de Red y Listas de Control de Acceso (ACLs)
**Nota Original:**
- [Clase 3 - ACLs y Seguridad](/Resumenes/Modulo%203/NOTAS%20CLASE%203%20ACLS.txt)

**Nota Avanzada:**
- [**Nota Avanzada - Clase 3**](/Resumenes/Modulo%203/Nota%20avanzada%20de%20clase%203.md)
  - Fundamentos de ciberseguridad: Tríada CIA, cifrado simétrico/asimétrico y hashing
  - Principios de ACLs: evaluación Top-Down, primer match y Deny Implícito
  - Lógica y cálculo de máscaras Wildcard (`host` y `any`)
  - ACLs Estándar vs Extendidas: diferencias, rangos (1-99 / 100-199) y sintaxis nombrada
  - Regla de oro de ubicación (Estándar cerca del destino / Extendida cerca del origen)
  - Aplicación en interfaces (`ip access-group in/out`) y líneas VTY (`access-class in`)
  - Casos prácticos de examen: bloqueo web vs autorización exclusiva de servicios (DNS/DHCP/Web)
  - Edición moderna con números de secuencia

---

### Clase 4: Continuación de ACLs, VTY y Filtrado IPv6
**Nota Original:**
- [Clases 3 y 4 - ACLs](/Resumenes/Modulo%203/NOTAS%20CLASE%203%20Y%204%20ACLS.txt)

**Nota Avanzada:**
- [**Nota Avanzada - Clase 4**](/Resumenes/Modulo%203/Nota%20avanzada%20de%20clase%204.md)
  - Desglose de políticas y sentido IN/OUT
  - Bloqueo de solicitudes ICMP entre VLANs
  - Restricción de VTY, requisitos SSH y edición por secuencias
  - ACLs IPv6, prefijos y permisos de Neighbor Discovery

### Clase 5: NAT Estática, Dinámica y PAT
**Nota Original:**
- [Clase 5 - NAT](/Resumenes/Modulo%203/NOTAS%20CLASE%205%20MOD%203.txt)

**Nota Avanzada:**
- [**Nota Avanzada - Clase 5**](/Resumenes/Modulo%203/Nota%20avanzada%20de%20clase%205.md)
  - RFC 1918 y los cuatro términos de NAT
  - Interfaces inside/outside y rutas de retorno
  - NAT estática, pools dinámicos y PAT con pool o interfaz
  - Verificación de traducciones y diagnóstico

### Tarea 6: PPP, PAP, CHAP y HDLC
**Resumen de la Tarea:**
- [Notas rápidas - PPP](/Resumenes/Modulo%203/NOTAS%20CLASE%206%20MOD%203.txt)

**Nota Avanzada:**
- [**Nota Avanzada - Clase 6**](/Resumenes/Modulo%203/Nota%20avanzada%20de%20clase%206.md)
  - Enlaces seriales, DCE/DTE, LCP y NCP
  - Autenticación PAP y CHAP en ambos extremos
  - Direccionamiento y OSPF de la Tarea 6
  - Diagnóstico por capas y pruebas de cada enlace

### Tarea 7: Frame Relay
**Resumen de la Tarea:**
- [Notas rápidas - Frame Relay](/Resumenes/Modulo%203/NOTAS%20CLASE%207%20MOD%203.txt)

**Nota Avanzada:**
- [**Nota Avanzada - Clase 7**](/Resumenes/Modulo%203/Nota%20avanzada%20de%20clase%207.md)
  - PVC, DLCI, LMI y mapas estáticos
  - Subinterfaces multipunto y punto a punto
  - OSPF non-broadcast, vecinos y temporizadores
  - Estados ACTIVE, INACTIVE y DELETED

### Clase 8: Práctica de Examen (Tarea 9)
**Resumen de la Tarea:**
- [Notas rápidas - Examen](/Resumenes/Modulo%203/NOTAS%20CLASE%208%20MOD%203.txt)

**Nota Avanzada:**
- [**Nota Avanzada - Clase 8**](/Resumenes/Modulo%203/Nota%20avanzada%20de%20clase%208.md)
  - VLANs, Router-on-a-Stick y DHCP Relay
  - PPP, Frame Relay y redistribución OSPF/EIGRP
  - Salida al ISP, NAT y ACLs del examen
  - Matriz de pruebas permitidas y denegadas

### Clase 9: Seguridad de Red

**Resumen de Estudio:**
- [Notas rapidas - Clase 9](/Resumenes/Modulo%203/NOTAS%20CLASE%209%20MOD%203.txt)

**Nota Avanzada:**
- [**Nota Avanzada - Clase 9**](/Resumenes/Modulo%203/Nota%20avanzada%20de%20clase%209.md)
  - Amenazas, defensa en profundidad, AAA, SSH y criptografia

### Clase 10: WAN, VPN e IPsec

**Resumen de Estudio:**
- [Notas rapidas - Clase 10](/Resumenes/Modulo%203/NOTAS%20CLASE%2010%20MOD%203.txt)

**Nota Avanzada:**
- [**Nota Avanzada - Clase 10**](/Resumenes/Modulo%203/Nota%20avanzada%20de%20clase%2010.md)
  - Acceso WAN, topologias, VPN site-to-site y remota, IKE, ESP y GRE

### Clase 11: Calidad de Servicio (QoS)

**Resumen de Estudio:**
- [Notas rapidas - Clase 11](/Resumenes/Modulo%203/NOTAS%20CLASE%2011%20MOD%203.txt)

**Nota Avanzada:**
- [**Nota Avanzada - Clase 11**](/Resumenes/Modulo%203/Nota%20avanzada%20de%20clase%2011.md)
  - Retardo, jitter, marcado DSCP, colas, policing y shaping

### Clase 12: Gestion de Red

**Resumen de Estudio:**
- [Notas rapidas - Clase 12](/Resumenes/Modulo%203/NOTAS%20CLASE%2012%20MOD%203.txt)

**Nota Avanzada:**
- [**Nota Avanzada - Clase 12**](/Resumenes/Modulo%203/Nota%20avanzada%20de%20clase%2012.md)
  - CDP, LLDP, NTP, SNMP, Syslog, respaldos y mantenimiento IOS

### Clase 13: Diseno y Resolucion de Problemas

**Resumen de Estudio:**
- [Notas rapidas - Clase 13](/Resumenes/Modulo%203/NOTAS%20CLASE%2013%20MOD%203.txt)

**Nota Avanzada:**
- [**Nota Avanzada - Clase 13**](/Resumenes/Modulo%203/Nota%20avanzada%20de%20clase%2013.md)
  - Jerarquia, escalabilidad, documentacion, linea base y diagnostico IPv4/IPv6

### Clase 14: Virtualizacion, Nube y SDN

**Resumen de Estudio:**
- [Notas rapidas - Clase 14](/Resumenes/Modulo%203/NOTAS%20CLASE%2014%20MOD%203.txt)

**Nota Avanzada:**
- [**Nota Avanzada - Clase 14**](/Resumenes/Modulo%203/Nota%20avanzada%20de%20clase%2014.md)
  - Hipervisores, cloud, NFV, planos, controladores, underlay y overlay

### Clase 15: Automatizacion de Red

**Resumen de Estudio:**
- [Notas rapidas - Clase 15](/Resumenes/Modulo%203/NOTAS%20CLASE%2015%20MOD%203.txt)

**Nota Avanzada:**
- [**Nota Avanzada - Clase 15**](/Resumenes/Modulo%203/Nota%20avanzada%20de%20clase%2015.md)
  - JSON, YAML, XML, REST, NETCONF/RESTCONF, Ansible, Terraform e IA/ML

### Caso de Estudio: HQ, B1, B2, B3 y NewB
- [**Nota Avanzada - Caso de Estudio**](/Resumenes/Modulo%203/Nota%20avanzada%20de%20caso%20de%20estudio.md)
  - VLANs, VTP, STP, Port Security, DHCP y WLAN
  - WAN, sumarización EIGRP y rutas estáticas
  - NAT centralizada y alcance de la ACL FIREWALL
  - Referencias al informe, notas de configuración y actividad `.pka`

**Numeración y fuentes:** se conserva una secuencia continua de clases 1–15. La clase 8 resume la Tarea 9 original. Las clases 9–15 completan los temas de ENSA con material de estudio elaborado a partir de documentación oficial; cada nota identifica sus fuentes. Los números de clase son el orden de este repositorio, no los números de módulo de NetAcad.

---

### Correspondencia con el temario ENSA

La secuencia de clases conserva el orden personal de estas notas. Esta tabla permite localizar todos los bloques del curso sin confundirlos con los numeros de tarea:

| Bloque de ENSA | Clases de este repositorio |
|---|---|
| Conceptos y configuracion OSPFv2 | 1 y 2 |
| Seguridad de red | 3 y 9 |
| Conceptos y configuracion de ACLs IPv4 | 3 y 4 |
| NAT para IPv4 | 5 |
| Conceptos WAN | 6, 7 y 10 |
| VPN e IPsec | 10 |
| QoS | 11 |
| Gestion de red | 12 |
| Diseno de red | 13 |
| Resolucion de problemas | 13 |
| Virtualizacion de red | 14 |
| Automatizacion de red | 15 |
| Repaso integrado y caso de estudio | 8 y nota del caso |

La clase 15 incorpora ademas Ansible, Terraform e IA/ML para el repaso CCNA v1.1. Referencias: [curso oficial ENSA](https://www.netacad.com/courses/ccna-enterprise-networking-security-automation?courseLang=en-US) y [temario CCNA](https://learningnetwork.cisco.com/s/ccna-exam-topics), consultados el 15 de octubre de 2026.

## Cómo Usar Estas Notas

### Para Principiantes
1. Lee primero **General.md** para entender conceptos básicos
2. Lee las **notas de clase originales** en orden (Clase 1 → 11)
3. Consulta **notas avanzadas** cuando necesites más profundidad

### Para Revisión Rápida
- Usa **notas de clase originales** para recordar conceptos
- Usa **notas avanzadas** para profundizar en un tema

### Para Preparación CCNA
1. Lee todas las **notas avanzadas** en orden
2. Realiza ejercicios prácticos con Cisco Packet Tracer
3. Verifica con comandos reales en laboratorio
4. Consulta **notas originales** si necesitas aclaración

---

## Complementos

### Archivos en el Repositorio
- **Topologías/** - Archivos .pkt para Cisco Packet Tracer
- **Documentos/** - Guías de referencia CCNA
- **Imagenes/** - Diagramas y referencias visuales

### Recursos Externos Recomendados
- Cisco Learning Network
- Cisco Documentation Library
- Packet Tracer (simulador oficial Cisco)
- Professor Messer (YouTube)

---

## Estructura de las Notas Avanzadas

Cada nota avanzada sigue este formato:

1. **Introducción** - Contexto del tema
2. **Conceptos fundamentales** - Base teórica
3. **Explicaciones profundas** - Detalles técnicos
4. **Diagramas y ejemplos** - Visualización
5. **Tablas de referencia** - Comparativas
6. **Resumen** - Puntos clave
7. **Referencias** - RFCs y estándares

---

xd
