# Nota Avanzada - Clase 8: Práctica de Examen e Integración del Módulo 3

## Introducción

La práctica de examen reúne VLANs, DHCP Relay, PPP, Frame Relay, OSPF, EIGRP, redistribución, NAT y ACLs. Se organiza como **clase 8** para mantener la numeración continua de los resúmenes; su material de origen es la **Tarea 9** del repositorio.

Esta guía explica cómo se relacionan las configuraciones de `TAREA 9 Practica Examen.txt`. Los resultados indicados son **criterios para verificar en Packet Tracer**, no una afirmación de que se haya ejecutado aquí el archivo `.pkt`.

---

## 1. Mapa de la Topología

```text
VLAN 10,20,30 -- R4 -- PPP CHAP -- R1
                                  |\
                                  | \  Frame Relay + OSPF área 0
                                  |  \
VLAN 40,50,60 -- R5 -- PPP CHAP -- R2---R3 -- PPP PAP -- ISP
                                      |
                                  PPP CHAP
                                      |
                                      R6 -- VLAN 70,80,90
```

| Segmento | Red | Direcciones de los extremos |
|---|---|---|
| R1–R2, Frame Relay | `20.0.0.0/30` | R1 `.1`, R2 `.2` |
| R1–R3, Frame Relay | `20.0.0.4/30` | R1 `.5`, R3 `.6` |
| R2–R3, Frame Relay | `20.0.0.8/30` | R2 `.9`, R3 `.10` |
| R1–R4, CHAP | `40.0.0.0/30` | R1 `.1`, R4 `.2` |
| R2–R5, CHAP | `40.0.0.4/30` | R2 `.5`, R5 `.6` |
| R3–R6, CHAP | `40.0.0.8/30` | R3 `.9`, R6 `.10` |
| R3–ISP, PAP | `200.0.0.0/30` | R3 `.1`, ISP `.2` |

Las VLANs usan `172.16.ID.0/24`, con gateway `.1`. Se conservan las redes WAN del ejercicio como direcciones de simulación.

---

## 2. Orden de Configuración

1. Crear VLANs y asignar puertos de acceso y troncales.
2. Configurar subinterfaces 802.1Q y gateways.
3. Asignar IP a enlaces WAN y completar PPP y Frame Relay.
4. Configurar EIGRP en las sucursales y OSPF en el núcleo.
5. Redistribuir las rutas entre protocolos y comprobar ida y vuelta.
6. Habilitar DHCP Relay y verificar los pools del servidor.
7. Configurar ruta por defecto, NAT y retorno del ISP.
8. Aplicar ACLs y probar cada condición por separado.
9. Guardar con `copy running-config startup-config` desde modo privilegiado.

Aplicar los filtros después de tener conectividad facilita distinguir fallos de enrutamiento de bloqueos intencionados.

---

## 3. VLANs y DHCP Relay

El servidor DHCP está en **172.16.30.10**, dentro de VLAN 30. Las VLANs remotas necesitan retransmisión de las solicitudes; VLAN 30 comparte el segmento con el servidor.

Ejemplo de R5, desde configuración global:

```text
interface fastEthernet 0/0
 no ip address
 no shutdown
 exit
interface fastEthernet 0/0.50
 encapsulation dot1Q 50
 ip address 172.16.50.1 255.255.255.0
 ip helper-address 172.16.30.10
 exit
```

El servidor necesita un pool para cada VLAN con máscara `/24`, gateway `172.16.ID.1` y el DNS indicado por el ejercicio. El relay no crea esos pools. Su dirección de gateway identifica la subred desde la que se solicita la concesión.

**Comprobación:** cada PC recibe una dirección de su VLAN, gateway correcto y DNS; el servidor puede devolver tráfico hacia las redes de clientes.

---

## 4. EIGRP en Sucursales y OSPF en el Núcleo

| Dominio | Routers | Redes |
|---|---|---|
| EIGRP AS 10 | R1 y R4 | Enlace CHAP y VLANs 10–30 |
| EIGRP AS 20 | R2 y R5 | Enlace CHAP y VLANs 40–60 |
| EIGRP AS 30 | R3 y R6 | Enlace CHAP y VLANs 70–90 |
| OSPF proceso 1, área 0 | R1, R2, R3 | Enlaces Frame Relay |

El **AS de EIGRP debe coincidir entre vecinos**. El identificador de proceso OSPF tiene significado local; lo que debe coincidir en el enlace es el área y los demás parámetros de vecindad.

Ejemplo de R4:

```text
router eigrp 10
 no auto-summary
 network 40.0.0.0 0.0.0.3
 network 172.16.10.0 0.0.0.255
 network 172.16.20.0 0.0.0.255
 network 172.16.30.0 0.0.0.255
 passive-interface default
 no passive-interface serial 0/0/0
 exit
```

Las LANs se anuncian sin enviar Hellos a las PCs. La interfaz serial debe quedar habilitada para formar vecindad.

---

## 5. Redistribución OSPF ↔ EIGRP

Un router con ambos protocolos no comparte automáticamente sus rutas entre ellos. R1, R2 y R3 realizan esa unión.

Ejemplo de R1, adaptado para incluir rutas OSPF externas de las otras sucursales:

```text
router eigrp 10
 redistribute ospf 1 match internal external 1 external 2 metric 10000 100 255 1 1500
 exit
router ospf 1
 redistribute eigrp 10 subnets
 exit
```

En R2 usar AS 20 y en R3 AS 30. La sintaxis disponible puede variar en Packet Tracer: revisar `redistribute ospf 1 ?`. Las notas originales omiten `match`; verificar que las rutas externas `O E2` también se exporten a EIGRP, y no solo las internas.

| Parámetro de la métrica inicial EIGRP | Valor |
|---|---|
| Ancho de banda | `10000` Kbit/s |
| Retardo | `100` unidades de 10 microsegundos |
| Confiabilidad | `255` sobre 255 |
| Carga | `1` sobre 255 |
| MTU | `1500` bytes |

`subnets` incluye las redes con sus máscaras en la redistribución hacia OSPF. Las métricas se inicializan según el protocolo receptor, no se copian numéricamente de OSPF a EIGRP. [Cisco: redistribución de protocolos](https://www.cisco.com/c/en/us/support/docs/ip/enhanced-interior-gateway-routing-protocol-eigrp/8606-redist.html).

La redistribución mutua debe limitarse al diseño de la práctica. Si se agregan fronteras redundantes hacia el mismo dominio, hacen falta políticas de filtrado y etiquetado para evitar reintroducir rutas.

### Códigos que Deben Reconocerse

| Código | Origen |
|---|---|
| `D` | EIGRP interno |
| `D EX` | Ruta externa redistribuida a EIGRP |
| `O` | OSPF intra-área |
| `O E2` | Ruta externa OSPF de tipo 2 |
| `S*` | Ruta estática candidata por defecto |

---

## 6. Salida al ISP y NAT

En R3:

```text
ip route 0.0.0.0 0.0.0.0 200.0.0.2
router ospf 1
 default-information originate
 exit
router eigrp 30
 redistribute static metric 10000 100 255 1 1500
 exit
```

La segunda redistribución exporta las rutas estáticas elegibles de R3, no exclusivamente la predeterminada. En esta práctica se usa para que R6 tenga salida; comprobar cuáles existen antes de reutilizarla.

**PAT de la práctica:** ACL `REDES-NAT` permite las nueve LANs; pool `PUBLICAS` usa `209.165.200.1` a `.6`, máscara `/29`.

```text
ip nat pool PUBLICAS 209.165.200.1 209.165.200.6 netmask 255.255.255.248
ip nat inside source list REDES-NAT pool PUBLICAS overload

interface serial 0/0/0.31
 ip nat inside
 exit
interface serial 0/0/0.32
 ip nat inside
 exit
interface serial 0/0/1
 ip nat inside
 exit
interface serial 0/1/0
 ip nat outside
 exit
```

Crear previamente `REDES-NAT` con las nueve sentencias `/24` del archivo original. En el ISP:

```text
ip route 209.165.200.0 255.255.255.248 200.0.0.1
```

**Recorrido:** PC → gateway de VLAN → EIGRP → núcleo OSPF → R3/PAT → ISP. La respuesta regresa a una dirección del pool y R3 deshace la traducción.

---

## 7. ACLs del Examen

### 7.1 VLAN 10 sin Web

En R4, entrada de Fa0/0.10:

```text
ip access-list extended VLAN10-SIN-WEB
 deny tcp 172.16.10.0 0.0.0.255 any eq 80
 deny tcp 172.16.10.0 0.0.0.255 any eq 443
 deny udp 172.16.10.0 0.0.0.255 any eq 443
 permit ip any any
 exit
interface fastEthernet 0/0.10
 ip access-group VLAN10-SIN-WEB in
 exit
```

La línea UDP 443 contempla QUIC. La política bloquea los puertos indicados; no constituye inspección de aplicaciones.

### 7.2 VLAN 50 Limitada

En R5, entrada de Fa0/0.50:

```text
ip access-list extended VLAN50-LIMITADA
 permit udp any eq 68 any eq 67
 permit udp 172.16.50.0 0.0.0.255 host 209.165.201.20 eq 53
 permit tcp 172.16.50.0 0.0.0.255 host 209.165.201.20 eq 53
 permit tcp 172.16.50.0 0.0.0.255 any eq 80
 permit tcp 172.16.50.0 0.0.0.255 any eq 443
 permit udp 172.16.50.0 0.0.0.255 any eq 443
 permit icmp 172.16.50.0 0.0.0.255 any echo-reply
 deny ip any any
 exit
interface fastEthernet 0/0.50
 ip access-group VLAN50-LIMITADA in
 exit
```

La primera regla contempla clientes DHCP aún sin IP. `echo-reply` permite que las PCs respondan a pings recibidos; no les permite iniciar ping. La ACL de entrada no filtra las respuestas que salen del router hacia esa VLAN.

### 7.3 Acceso Administrativo desde VLAN 80

```text
access-list 80 permit 172.16.80.0 0.0.0.255
access-list 80 deny any
line vty 0 4
 access-class 80 in
 exit
```

Aplicar en cada equipo que deba restringirse y cubrir todas sus líneas VTY. El ejercicio usa Telnet; para el ejemplo de SSH y sus requisitos consultar la avanzada de clase 4.

---

## 8. Matriz de Pruebas

| Prueba | Resultado que se debe comprobar |
|---|---|
| PC de cada VLAN solicita DHCP | Dirección, gateway y DNS de su segmento |
| Vecinos PPP y PVC Frame Relay | Enlaces activos antes de revisar routing |
| R4 consulta una LAN de R5 | Ruta externa EIGRP y camino de retorno |
| PC interna consulta servicio externo permitido | Traducción en R3 y respuesta del ISP |
| VLAN 10 abre HTTP/HTTPS | Bloqueado; contador de ACL aumenta |
| VLAN 50 consulta DNS autorizado y HTTP | Permitido |
| VLAN 50 inicia ping | Bloqueado |
| Otra VLAN hace ping a VLAN 50 | Respuesta permitida si no hay otro filtro |
| Gestión desde VLAN 80 / otra VLAN | Permitida / denegada por VTY |

Comandos clave: `show ip route`, `show ip protocols`, `show ip eigrp neighbors`, `show ip ospf neighbor`, `show frame-relay pvc`, `show ip nat translations` y `show access-lists`.

---

## Resumen Ejecutivo

- Verificar cada capa antes de incorporar el siguiente servicio.
- EIGRP conecta cada sucursal; OSPF conecta el núcleo Frame Relay.
- La redistribución permite cruzar los dominios, incluidas las rutas externas de otras sucursales.
- DHCP Relay requiere rutas de retorno y pools correctos en el servidor.
- NAT requiere todas las entradas internas y retorno del ISP al pool.
- Probar el sentido exacto de cada ACL con tráfico permitido y denegado.

## Referencias

- [Configuración original de Tarea 9](../../Topolog%C3%ADas/Topolog%C3%ADas%20de%20Modulo%203/TAREA%209%20Practica%20Examen.txt).
- [Topología de Tarea 9](../../Topolog%C3%ADas/Topolog%C3%ADas%20de%20Modulo%203/TAREA%209%20Practica%20Examen.pkt).
- [Cisco: redistribución de protocolos](https://www.cisco.com/c/en/us/support/docs/ip/enhanced-interior-gateway-routing-protocol-eigrp/8606-redist.html).
