# Nota Avanzada - Clase 4: Edición de ACLs, Acceso Remoto y ACLs IPv6

## Introducción

Esta clase continúa las ACLs IPv4 de la clase 3. El objetivo es convertir una política en reglas ordenadas, corregirlas mediante secuencias y distinguir el filtrado de tráfico del control de acceso administrativo al router.

Se basa en las **notas de clases 3 y 4**, la **Tarea 4 ACL** y los ejemplos IPv6 de la **Tarea 2**. Los bloques de configuración se introducen desde `configure terminal`, salvo los comandos de verificación, que se ejecutan desde modo privilegiado.

---

## 1. Método para Resolver una ACL

Antes de escribir comandos, hacer el **desglose** utilizado en clase:

| Pregunta | Ejemplo de la Tarea 4 |
|---|---|
| **Origen** | VLAN 30: `192.168.3.0/24` |
| **Destino** | VLAN 40: `192.168.4.0/24` |
| **Protocolo** | ICMP |
| **Tipo o puerto** | `echo`: solicitud de ping |
| **Acción** | Denegar solicitudes y permitir el resto |
| **Aplicación** | R1, `FastEthernet0/0.30`, entrada |

```text
VLAN 30 --> Fa0/0.30 [R1] Fa0/0.40 --> VLAN 40
                   IN       OUT
```

**IN y OUT se interpretan desde el router**, no desde la PC. Una ACL de entrada en la VLAN 30 examina lo que sus equipos envían al router.

### Las Tres P

En el filtrado de interfaces estudiado se aplica una ACL **por protocolo de red (IPv4/IPv6), por dirección y por interfaz**. Si ya existe una ACL IPv4 de entrada, las nuevas condiciones deben integrarse en ella; aplicar otro nombre reemplaza la asociación anterior.

---

## 2. Bloqueo de Ping entre VLANs

Para impedir que ambas VLANs inicien ping hacia la otra se filtra cada origen en su subinterfaz:

```text
access-list 100 deny icmp 192.168.3.0 0.0.0.255 192.168.4.0 0.0.0.255 echo
access-list 100 permit ip any any
access-list 101 deny icmp 192.168.4.0 0.0.0.255 192.168.3.0 0.0.0.255 echo
access-list 101 permit ip any any

interface fastEthernet 0/0.30
 ip access-group 100 in
 exit
interface fastEthernet 0/0.40
 ip access-group 101 in
 exit
```

| Sentencia | Qué bloquea |
|---|---|
| `deny icmp ... echo` | Solicitudes de ping |
| `deny icmp ...` | Todos los tipos ICMP coincidentes |
| `deny ip ...` | Todo el tráfico IPv4 entre los orígenes y destinos indicados |

La tercera alternativa también bloquearía TCP y UDP. El nombre de una ACL, por ejemplo `NO_PING`, no determina lo que filtra: lo determinan sus sentencias.

---

## 3. Restringir la Administración por VTY

**Requerimiento de la Tarea 4:** permitir administración de R1, R2 y R3 desde `192.168.5.0/24`.

```text
ip access-list standard ADMIN_VTY
 permit 192.168.5.0 0.0.0.255
 deny any
 exit

line vty 0 4
 access-class ADMIN_VTY in
 login local
 transport input ssh
 exit
```

Repetir en **cada router que se administra**. Si el equipo tiene líneas 5–15, aplicar también allí la política; comprobarlo con `show running-config`.

### Requisitos de SSH

La ACL no crea usuarios ni habilita SSH. Ejemplo de preparación en R1, con credencial exclusiva del laboratorio:

```text
hostname R1
ip domain-name ccna.lab
username CCNA secret ClaveLab123
crypto key generate rsa
```

Cuando IOS solicite el tamaño de la clave, introducir `2048` si el dispositivo lo admite. Luego:

```text
ip ssh version 2
```

Desde una PC permitida: `ssh -l CCNA 192.168.3.1`. El nombre de usuario debe respetar las mayúsculas de la configuración.

| Comando | Uso |
|---|---|
| `ip access-group ADMIN_VTY in` | Filtra tráfico IPv4 que entra por una interfaz; no sustituye la política de VTY |
| `access-class ADMIN_VTY in` | Restringe los orígenes que pueden iniciar acceso a las líneas VTY |
| `transport input ssh` | Selecciona SSH como transporte administrativo |
| `login local` | Autentica contra usuarios locales |

---

## 4. Edición mediante Números de Secuencia

Primero revisar la lista real con `show ip access-lists PERRITO2`. Para corregir el origen de la línea 40:

```text
ip access-list extended PERRITO2
 no 40
 40 permit tcp 192.168.0.0 0.0.0.255 any eq 443
 exit
```

Para insertar una excepción antes de una denegación en la secuencia 30:

```text
ip access-list extended PERRITO2
 25 permit tcp host 192.168.0.10 any eq 80
 exit
```

**La primera coincidencia decide.** Una excepción posterior a `deny ip any any` nunca se alcanza.

> **Aclaración de las notas originales:** no todas las ACLs numeradas requieren borrarse completas. En IOS que admite secuencias se puede entrar, por ejemplo, con `ip access-list standard 10` y editar sus entradas. La disponibilidad depende del IOS o dispositivo simulado.

---

## 5. ACLs IPv6

En la sintaxis de estas prácticas, las ACLs IPv6 son **nombradas**, permiten criterios extendidos y utilizan **prefijos**, no máscaras wildcard.

```text
ipv6 access-list NO_PING
 deny icmp host 2001:db8:cafe:a001::10 host 2001:db8:cafe:a002::10 echo-request
 permit ipv6 any any
 exit

interface gigabitEthernet 0/0
 ipv6 traffic-filter NO_PING in
 exit
```

Este ejemplo adapta la Tarea 2 para bloquear **solo solicitudes de ping**. Su sentencia original `deny ipv6 host ... host ...` bloquea todos los protocolos IPv6 entre esos hosts.

### 5.1 Neighbor Discovery y Denegación Implícita

En el IOS clásico estudiado, al final de una ACL IPv6 existen permisos implícitos para Neighbor Solicitation y Neighbor Advertisement, seguidos por la denegación general. Un `deny ipv6 any any` **explícito** colocado antes puede impedir que se alcancen esos permisos.

Ejemplo didáctico que permite ICMPv6 y HTTP desde una LAN:

```text
ipv6 access-list WEB_V6
 permit icmp any any
 permit tcp 2001:db8:acad:1::/64 any eq 80
 deny ipv6 any any
 exit
```

Permitir ICMPv6 completo facilita ND, descubrimiento de routers y mensajes de error en el laboratorio, pero también permite ping. Si la política pide solo tipos concretos, desglosarlos antes del `deny`; conservar únicamente ND-NS/ND-NA no cubre todas las funciones de ICMPv6.

### 5.2 Bloqueo Web de la Tarea 2

Para HTTP y HTTPS sobre TCP, las líneas son:

```text
ipv6 access-list NO_WEB
 deny tcp host 2001:db8:cafe:a003::10 any eq 80
 deny tcp host 2001:db8:cafe:a003::10 any eq 443
 permit ipv6 any any
 exit
```

TCP 53 corresponde a **DNS**, no a HTTPS. UDP 80 tampoco sustituye TCP 80. La práctica de examen añade UDP 443 para contemplar QUIC; el filtrado por puertos no identifica todos los posibles métodos de navegación.

---

## 6. Verificación y Resultados Esperados

| Comando o prueba | Qué comprobar |
|---|---|
| `show access-lists` | Orden y contadores de coincidencias |
| `show ip interface fastEthernet 0/0.30` | ACL 100 en entrada |
| `show ipv6 access-list` | Reglas y coincidencias IPv6 |
| `show ipv6 interface gigabitEthernet 0/0` | Asociación del filtro IPv6 |
| `show ip ssh` | SSH habilitado y versión |
| SSH desde VLAN 50 y otra VLAN | Permiso desde la primera y rechazo desde la segunda |
| Ping VLAN 30 hacia VLAN 40 y viceversa | Solicitudes bloqueadas |
| Otro servicio permitido entre VLANs | Conectividad conservada por `permit ip any any` |

Probar tanto tráfico permitido como denegado. Un ping fallido también puede deberse a una ruta o gateway incorrectos: correlacionarlo con los contadores de la ACL.

---

## Resumen Ejecutivo

- Desglosar origen, destino, protocolo, puerto o tipo ICMP y sentido antes de configurar.
- `access-class` protege VTY; `ip access-group` filtra interfaces IPv4.
- Una secuencia permite corregir o insertar reglas conservando su orden lógico.
- En IPv6 se usan prefijos e `ipv6 traffic-filter`.
- Un `deny` explícito puede bloquear los permisos implícitos de Neighbor Discovery.
- Verificar permisos y bloqueos con el servicio real y los contadores.

## Referencias

- [Notas originales de clases 3 y 4](./NOTAS%20CLASE%203%20Y%204%20ACLS.txt).
- [Tarea 4 ACL](../../Topolog%C3%ADas/Topolog%C3%ADas%20de%20Modulo%203/Notas%20de%20Tarea%204%20ACL.txt).
- [Tarea 2 OSPF IPv6](../../Topolog%C3%ADas/Topolog%C3%ADas%20de%20Modulo%203/Notas%20de%20Tarea%202%20OSPF%20IPV6.txt).
- [Cisco: referencia de comandos, ACLs IPv6](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst3750/software/release/12-2_52_se/command/reference/3750cr.pdf).
