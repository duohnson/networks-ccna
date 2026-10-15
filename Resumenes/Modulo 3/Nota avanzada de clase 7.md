# Nota Avanzada - Clase 7: Frame Relay, DLCI, LMI y OSPF

## Introducción

**Frame Relay** es una tecnología WAN de capa 2 que conecta sedes mediante circuitos virtuales. Se estudia aquí porque aparece en la **Tarea 7**, la **práctica de examen 9** y el **caso de estudio**. Esta nota reconstruye la clase a partir de esos materiales y conserva su numeración.

La tarea utiliza una subinterfaz **multipunto** compartida por cuatro routers; el examen utiliza subinterfaces **punto a punto**. Comprender esa diferencia explica los cambios de direccionamiento, DLCI y vecindades OSPF.

---

## 1. Conceptos Fundamentales

| Concepto | Significado |
|---|---|
| **PVC** | Circuito virtual permanente entre extremos |
| **DLCI** | Identificador local de un circuito virtual |
| **LMI** | Intercambio de estado entre router y switch Frame Relay del proveedor |
| **NBMA** | Red multiacceso sin difusión nativa |
| **Inverse ARP** | Permite aprender una dirección de red asociada a un DLCI |
| **Mapa estático** | Asociación manual entre IP remota y DLCI local |

Un circuito puede usar DLCI 101 en un extremo y 201 en el otro. **Los números no tienen que coincidir entre routers**: deben corresponder al circuito configurado en la nube.

LMI puede ser `cisco`, `ansi` o `q933a`. Debe coincidir con el puerto del proveedor al que se conecta cada router. La encapsulación de datos (`cisco` o `ietf`) es un ajuste distinto de LMI.

---

## 2. Subinterfaces Multipunto y Punto a Punto

| Característica | Multipunto | Punto a punto |
|---|---|---|
| Vecinos en una subinterfaz | Varios | Uno |
| Direccionamiento | Subred compartida | Subred por enlace lógico |
| Asociación habitual | `frame-relay map ip ...` | `frame-relay interface-dlci ...` |
| OSPF de los ejercicios | Non-broadcast, vecinos manuales | Point-to-point |
| DR/BDR | Sí en tipo OSPF non-broadcast | No |

```text
MULTIPUNTO: RA, RB, RC y RD comparten 10.0.0.0/29

PUNTO A PUNTO DEL EXAMEN:
 R1 -- 20.0.0.0/30 -- R2
  |                   |
20.0.0.4/30        20.0.0.8/30
  |                   |
  +------- R3 --------+
```

---

## 3. Direccionamiento y Mapas de la Tarea 7

WAN: `10.0.0.0/29`, máscara `255.255.255.248`, wildcard `0.0.0.7`, hosts `.1` a `.6`, broadcast `.7`.

| Router | IP WAN | LAN de las notas | DLCI local hacia los demás |
|---|---|---|---|
| RA | `10.0.0.1` | `172.168.0.0/24` | RB: 101; RC: 104; RD: 103 |
| RB | `10.0.0.2` | `172.168.1.0/24` | RA: 201; RC: 204; RD: 203 |
| RC | `10.0.0.3` | `172.168.2.0/24` | RA: 401; RB: 402; RD: 403 |
| RD | `10.0.0.4` | `172.168.3.0/24` | RA: 301; RB: 302; RC: 304 |

Estas LANs se conservan para reconocer la tarea, pero **172.168 no es un rango privado RFC 1918**.

### 3.1 Configuración de RA

Desde configuración global, con el módulo serial instalado:

```text
hostname RA
interface fastEthernet 0/0
 ip address 172.168.0.1 255.255.255.0
 no shutdown
 exit

interface serial 0/0/0
 no ip address
 encapsulation frame-relay
 frame-relay lmi-type cisco
 no shutdown
 exit

interface serial 0/0/0.1 multipoint
 ip address 10.0.0.1 255.255.255.248
 frame-relay map ip 10.0.0.2 101 broadcast
 frame-relay map ip 10.0.0.3 104 broadcast
 frame-relay map ip 10.0.0.4 103 broadcast
 ip ospf network non-broadcast
 exit

router ospf 1
 network 10.0.0.0 0.0.0.7 area 0
 network 172.168.0.0 0.0.0.255 area 0
 passive-interface fastEthernet 0/0
 neighbor 10.0.0.2
 neighbor 10.0.0.3
 neighbor 10.0.0.4
 exit
```

La palabra `broadcast` permite reenviar tráfico de difusión/multidifusión por ese mapa; **no cambia por sí sola el tipo de red OSPF**. En non-broadcast se configuran vecinos manualmente. Aplicar el mismo tipo OSPF en los extremos. [Cisco: configuración y diagnóstico Frame Relay](https://team-development.cisco.com/c/en/us/support/docs/wan/frame-relay/16563-12.html).

### 3.2 Repetir en RB, RC y RD

Usar las IP, LANs y DLCI locales de la tabla. Configurar como vecinos las otras tres IP WAN. La nube necesita los PVC bidireccionales correspondientes:

| Circuito | Extremo A | Extremo B |
|---|---|---|
| RA–RB | RA 101 | RB 201 |
| RA–RC | RA 104 | RC 401 |
| RA–RD | RA 103 | RD 301 |
| RB–RC | RB 204 | RC 402 |
| RB–RD | RB 203 | RD 302 |
| RC–RD | RC 403 | RD 304 |

En RD, el texto original `EXITDOWN` es un error de escritura: usar `no shutdown` y luego `exit` como líneas separadas.

---

## 4. OSPF sobre Frame Relay

| Tipo OSPF | Descubrimiento del ejercicio | Hello / Dead habituales en IOS | DR/BDR |
|---|---|---|---|
| Non-broadcast | `neighbor IP` | 30 / 120 s | Sí |
| Point-to-point | Automático por el enlace lógico | 10 / 40 s | No |

Verificar los valores reales con `show ip ospf interface`. Área y temporizadores deben coincidir entre vecinos. En una topología hub-and-spoke NBMA, el DR debe tener conectividad con todos los participantes; las subinterfaces punto a punto evitan la elección de DR/BDR por circuito.

---

## 5. Punto a Punto: Ejemplo R1–R2 del Examen

La nube de la práctica conecta **R1 DLCI 203** con **R2 DLCI 101**.

### R1

```text
interface serial 0/0/0
 no ip address
 encapsulation frame-relay
 frame-relay lmi-type cisco
 no shutdown
 exit
interface serial 0/0/0.12 point-to-point
 ip address 20.0.0.1 255.255.255.252
 frame-relay interface-dlci 203
 ip ospf network point-to-point
 exit
router ospf 1
 network 20.0.0.0 0.0.0.3 area 0
 exit
```

### R2

```text
interface serial 0/0/0
 no ip address
 encapsulation frame-relay
 frame-relay lmi-type cisco
 no shutdown
 exit
interface serial 0/0/0.21 point-to-point
 ip address 20.0.0.2 255.255.255.252
 frame-relay interface-dlci 101
 ip ospf network point-to-point
 exit
router ospf 1
 network 20.0.0.0 0.0.0.3 area 0
 exit
```

Los números `.12` y `.21` son identificadores de subinterfaz, no DLCI. Los otros dos circuitos utilizan las subredes `20.0.0.4/30` y `20.0.0.8/30` según las notas del examen.

---

## 6. Verificación y Solución de Problemas

| Comando | Qué comprobar |
|---|---|
| `show interfaces serial 0/0/0` | Estado físico y encapsulación |
| `show frame-relay lmi` | Intercambio y errores LMI |
| `show frame-relay pvc` | DLCI y estado de los circuitos |
| `show frame-relay map` | Correspondencia entre IP remota y DLCI local |
| `show ip ospf interface` | Tipo de red y temporizadores |
| `show ip ospf neighbor` | Vecindades; 2-WAY entre DROthers puede ser normal |
| `show ip route ospf` | Redes remotas aprendidas |

| Estado del PVC | Interpretación habitual |
|---|---|
| **ACTIVE** | El proveedor informa que el circuito está disponible |
| **INACTIVE** | Circuito reconocido, pero no operativo de extremo a extremo |
| **DELETED** | El switch no anuncia el DLCI esperado; revisar circuito y LMI |

Un PVC ACTIVE no demuestra por sí solo que las IP o el enrutamiento sean correctos. Probar primero la IP WAN remota y después la LAN remota. En el caso de estudio, comprobar además que DLCI 43 esté asociado a **S0/0/0.43** de HQ.

---

## Resumen Ejecutivo

- DLCI identifica un circuito local; la nube relaciona ambos extremos.
- LMI informa estado y debe coincidir con el puerto del proveedor.
- Multipunto comparte subred; punto a punto asigna una subred por circuito.
- `broadcast` en un mapa no reemplaza `ip ospf network`.
- Revisar físico, LMI, PVC, mapa, IP y OSPF en ese orden.

## Referencias

- [Notas originales de Tarea 7 Frame Relay](../../Topolog%C3%ADas/Topolog%C3%ADas%20de%20Modulo%203/Notas%20de%20Tarea%207%20FRAME%20RELAY.txt).
- [Práctica de examen 9](../../Topolog%C3%ADas/Topolog%C3%ADas%20de%20Modulo%203/TAREA%209%20Practica%20Examen.txt).
- [Cisco: configuración y diagnóstico de Frame Relay](https://team-development.cisco.com/c/en/us/support/docs/wan/frame-relay/16563-12.html).
