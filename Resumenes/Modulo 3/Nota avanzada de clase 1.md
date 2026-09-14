# Nota Avanzada - Clase 1: Fundamentos de OSPF de Área Única (Single-Area OSPFv2 y OSPFv3)

## Introducción

**OSPF** (*Open Shortest Path First*) es un protocolo de enrutamiento dinámico de tipo **Estado de Enlace** (*Link-State*) desarrollado como un estándar abierto (definido para IPv4 en la **RFC 2328** y para IPv6 en la **RFC 5340**). Diseñado para reemplazar a protocolos de vector distancia como RIP, OSPF ofrece una convergencia rápida, escalabilidad superior mediante segmentación en áreas, soporte nativo de VLSM/CIDR y un uso eficiente del ancho de banda.

Esta nota avanzada profundiza en los fundamentos operativos de OSPF en topologías de una sola área (Área 0), el funcionamiento del algoritmo de Dijkstra, los tipos de paquetes y adyacencias, el mecanismo de elección de DR/BDR en redes multiacceso y su configuración en Cisco IOS.

---

## 1. Características Principales de OSPF

| Característica | Detalle Técnico |
|---|---|
| **Tipo de Protocolo** | IGP (*Interior Gateway Protocol*) - Opera dentro de un mismo Sistema Autónomo (AS). |
| **Clase** | Estado de Enlace (*Link-State*). |
| **Algoritmo** | Dijkstra / Shortest Path First (SPF). |
| **Distancia Administrativa (AD)** | **110** por defecto en Cisco IOS. |
| **Métrica** | **Costo** (*Cost*), inversamente proporcional al ancho de banda de la interfaz. |
| **Direcciones Multicast** | `224.0.0.5` (Todos los routers OSPF) / `224.0.0.6` (DR y BDR).<br>En IPv6: `FF02::5` y `FF02::6`. |
| **Transporte** | Protocolo IP número **89** (no utiliza TCP ni UDP). |
| **Actualizaciones** | Desencadenadas por eventos (*Triggered updates*) y refrescos periódicos cada 30 minutos. |

---

## 2. Componentes Fundamentales de OSPF

El funcionamiento de OSPF se apoya en tres componentes estructurales:

```text
┌────────────────────────────────────────────────────────────────────────┐
│                        COMPONENTES DE OSPF                            │
├──────────────────────────┬───────────────────────┬─────────────────────┤
│   1. Estructuras de Datos│ 2. Mensajes OSPF      │ 3. Algoritmo SPF    │
│      (Tablas OSPF)       │    (5 Tipos)          │    (Dijkstra)       │
├──────────────────────────┼───────────────────────┼─────────────────────┤
│ • Tabla de Vecinos       │ • Hello               │ • Calcula la ruta de│
│ • Tabla de Topología     │ • DBD                 │   menor costo a     │
│   (LSDB)                 │ • LSR                 │   cada red destino  │
│ • Tabla de Enrutamiento  │ • LSU (contiene LSAs) │ • Construye el      │
│                          │ • LSAck               │   árbol de caminos  │
└──────────────────────────┴───────────────────────┴─────────────────────┘
```

### 2.1 Las Tres Tablas de OSPF

1. **Tabla de Vecinos (*Neighbor Table*)**:
   - Lista todos los routers vecinos con los que se ha establecido una comunicación bidireccional.
   - Es única para cada router y depende de los enlaces directos.
   - Comando de verificación: `show ip ospf neighbor`.

2. **Tabla de Topología / Base de Datos de Estado de Enlace (*LSDB - Link-State Database*)**:
   - Representa el mapa topológico completo del área.
   - **Regla fundamental**: Todos los routers dentro de la misma área OSPF poseen exactamente la misma LSDB.
   - Comando de verificación: `show ip ospf database`.

3. **Tabla de Enrutamiento (*Routing Table / RIB*)**:
   - Contiene únicamente las mejores rutas hacia cada destino calculadas por el algoritmo SPF a partir de la LSDB.
   - Las rutas aprendidas por OSPF se identifican con el código **`O`**.
   - Comando de verificación: `show ip route ospf` o `show ip route`.

---

## 3. Mensajes de Protocolo (Paquetes OSPF)

OSPF utiliza 5 tipos de paquetes para descubrir vecinos, mantener adyacencias y sincronizar la base de datos:

| Tipo | Paquete | Nombre Completo | Función Principal |
|:---:|---|---|---|
| **1** | **Hello** | *Hello Packet* | Descubre vecinos, negocia parámetros y mantiene adyacencias (Keepalive). En redes multiacceso elige al DR y BDR. |
| **2** | **DBD** | *Database Description* | Contiene una lista abreviada de los LSAs en la LSDB del emisor; el receptor la compara con su propia LSDB. |
| **3** | **LSR** | *Link-State Request* | Solicita información detallada de cualquier LSA identificado en el DBD que haga falta o esté desactualizado. |
| **4** | **LSU** | *Link-State Update* | Respuesta explícita al LSR. Transporta los registros de estado de enlace (**LSAs**) detallados. |
| **5** | **LSAck** | *Link-State Acknowledgment* | Confirma la recepción de un LSU, garantizando confiabilidad sobre IP (ya que OSPF no usa TCP). |

```text
[Router A]                                                   [Router B]
    │                                                            │
    │ ── 1. Hello (Descubrimiento y verificación de parámetros) ─>│
    │ <─ 1. Hello (Bidireccional establecido: Estado 2-WAY) ──── │
    │                                                            │
    │ ── 2. DBD (Resumen de registros en LSDB) ──────────────────>│
    │ <─ 2. DBD (Resumen de registros en LSDB) ────────────────── │
    │                                                            │
    │ ── 3. LSR (Solicito LSA faltante para la red X) ───────────>│
    │ <─ 4. LSU (Envío LSA detallado de la red X) ───────────────│
    │                                                            │
    │ ── 5. LSAck (Acuse de recibo explícito del LSU) ───────────>│
    │                                                            │
    ▼                                                            ▼
                        ESTADO FULL (Convergencia)
```

---

## 4. Estados Operativos de OSPF (Proceso de Adyacencia)

Para que dos routers alcancen la adyacencia completa, deben progresar por una serie de estados bien definidos:

1. **Down**: Ningún paquete Hello ha sido detectado aún en el enlace.
2. **Init**: El router recibe un Hello del vecino, pero su propio Router ID no figura en la lista de vecinos vistos de ese Hello.
3. **Two-Way (2-Way)**: Comunicación bidireccional confirmada (el router ve su propio Router ID en el paquete Hello del vecino).
   > [!NOTE]
   > En redes multiacceso (como Ethernet), es al final del estado **Two-Way** donde se eligen el **DR** y el **BDR**.
4. **ExStart**: Los routers determinan la relación Master/Slave y el número de secuencia inicial para el intercambio de DBDs. El router con el **Router ID más alto** actúa como Master.
5. **Exchange**: Los routers intercambian paquetes DBD. Si se detectan rutas más recientes o faltantes, se preparan las solicitudes.
6. **Loading**: Se envían paquetes **LSR** y se reciben paquetes **LSU** con la información completa de los LSAs. Se confirman con **LSAck**.
7. **Full**: La base de datos topológica (LSDB) está 100% sincronizada entre los routers. La adyacencia es total.

---

## 5. Redes Multiacceso: Elección de DR, BDR y DROthers

### 5.1 El Problema de la Saturación de Adyacencias
En un medio compartido de difusión (como un switch Ethernet donde conviven $N$ routers), si cada router estableciera adyacencia directa con todos los demás, el número de adyacencias totales crecería exponencialmente según la fórmula:

$$\text{Total de Adyacencias} = \frac{N(N - 1)}{2}$$

*Ejemplo*: Con **50 routers**, habría:

$$\frac{50 \times 49}{2} = 1225 \text{ adyacencias}$$

Esto generaría una tormenta inmanejable de actualizaciones (flooding de LSUs) que saturaría la CPU y el ancho de banda.

### 5.2 La Solución: Roles DR, BDR y DROther
Para evitar este desbordamiento, OSPF elige dos roles centrales:
- **DR (*Designated Router*)**: Punto central de recolección y distribución de actualizaciones topológicas.
- **BDR (*Backup Designated Router*)**: Router de respaldo que asume inmediatamente si el DR falla.
- **DROther**: Cualquier otro router en la red multiacceso.

```text
               ┌───────────────┐
               │   SWITCH LAN  │
               └───┬───┬───┬───┘
                   │   │   │
        ┌──────────┘   │   └──────────┐
        ▼              ▼              ▼
   [ Router A ]   [ Router B ]   [ Router C ]
     (  DR  )       ( BDR  )      ( DROther )
```

### 5.3 Uso de Direcciones Multicast
- **`224.0.0.6`**: Reservada exclusivamente para escuchar al DR y BDR. Cuando un DROther detecta un cambio de enlace, envía su LSU a la IP `224.0.0.6`.
- **`224.0.0.5`**: Todos los routers OSPF escuchan en esta dirección. El DR toma el LSU recibido y lo retransmite hacia todos los routers (incluidos los DROthers) a través de la IP `224.0.0.5`.

### 5.4 Relaciones de Adyacencia en Multiacceso
- **`FULL/DR`**: Adyacencia completa entre un BDR/DROther y el DR.
- **`FULL/BDR`**: Adyacencia completa entre un DR/DROther y el BDR.
- **`FULL/DROTHER`**: Vista desde el DR/BDR hacia un DROther.
- **`2-WAY/DROTHER`**: Relación entre dos routers DROther ordinarios. **No intercambian LSDB ni llegan a estado FULL**, optimizando la memoria y el procesamiento.

### 5.5 Criterio de Selección de DR y BDR
La elección no es apropiativa (*non-preemptive*). Si entra un nuevo router con mejores credenciales cuando ya hay un DR y BDR activos, no los desbancará hasta que el proceso OSPF se reinicie o fallen los actuales.

El orden de evaluación es el siguiente:
1. **Mayor Prioridad de Interfaz OSPF** (`ip ospf priority <0-255>`):
   - El valor por defecto es **1**.
   - Una prioridad de **0** deshabilita al router de convertirse en DR o BDR (permanece siempre como DROther).
2. **Mayor Router ID (RID)**:
   Si las prioridades son idénticas, se desempata por el Router ID más alto, determinado según esta jerarquía:
   1. Configuración manual directa: `router-id <A.B.C.D>`.
   2. La dirección IPv4 activa más alta en cualquier **interfaz Loopback**.
   3. La dirección IPv4 activa más alta en cualquier **interfaz física**.

---

## 6. Métrica de Costo de OSPF

OSPF utiliza el **Costo** como métrica para seleccionar la mejor ruta (menor costo acumulado desde el origen hasta la red de destino):

$$\text{Costo} = \frac{\text{Ancho de Banda de Referencia}}{\text{Ancho de Banda de la Interfaz}}$$

Por defecto, en Cisco IOS el ancho de banda de referencia es $10^8\text{ bps}$ ($100\text{ Mbps}$):

| Interfaz | Ancho de Banda Típico | Cálculo por Defecto | Costo OSPF |
|---|---|---|:---:|
| **Serial T1** | $1.544\text{ Mbps}$ | $\frac{100\,000\,000}{1\,544\,000}$ | **64** |
| **Ethernet** | $10\text{ Mbps}$ | $\frac{100\,000\,000}{10\,000\,000}$ | **10** |
| **FastEthernet** | $100\text{ Mbps}$ | $\frac{100\,000\,000}{100\,000\,000}$ | **1** |
| **GigabitEthernet** | $1\,000\text{ Mbps}$ | $\frac{100\,000\,000}{1\,000\,000\,000} = 0.1 \to$ *(mínimo entero)* | **1** *(inadecuado)* |
| **10 GigabitEthernet** | $10\,000\text{ Mbps}$ | $\frac{100\,000\,000}{10\,000\,000\,000} = 0.01 \to$ | **1** *(inadecuado)* |

> [!WARNING]
> Como el costo debe ser un número entero mayor o igual a 1, un enlace de $100\text{ Mbps}$, uno de $1\text{ Gbps}$ y uno de $10\text{ Gbps}$ reciben todos un costo de **1** con el valor por defecto. Es obligatorio calibrar el ancho de banda de referencia en redes modernas.

---

## 7. Configuración Práctica de OSPFv2 (Paso a Paso)

### 7.1 Topología de Laboratorio
Tres routers (RA, RB, RC) interconectados en un segmento multiacceso `10.0.0.0/29` (máscara `255.255.255.248`, wildcard `0.0.0.7`) y cada uno con su respectiva red LAN local:
- **RA LAN**: `192.168.1.0/24` en `FastEthernet 0/1`
- **RB LAN**: `192.168.2.0/24` en `FastEthernet 0/1`
- **RC LAN**: `192.168.3.0/24` en `FastEthernet 0/1`

### 7.2 Configuración Básica y Publicación de Redes

```text
! ==========================================
! Router RA
! ==========================================
Router-A(config)# router ospf 1
Router-A(config-router)# router-id 1.1.1.1
Router-A(config-router)# network 10.0.0.0 0.0.0.7 area 0
Router-A(config-router)# network 192.168.1.0 0.0.0.255 area 0
Router-A(config-router)# passive-interface fastEthernet 0/1
Router-A(config-router)# exit

! ==========================================
! Router RB
! ==========================================
Router-B(config)# router ospf 1
Router-B(config-router)# router-id 2.2.2.2
Router-B(config-router)# network 10.0.0.0 0.0.0.7 area 0
Router-B(config-router)# network 192.168.2.0 0.0.0.255 area 0
Router-B(config-router)# passive-interface fastEthernet 0/1
Router-B(config-router)# exit

! ==========================================
! Router RC
! ==========================================
Router-C(config)# router ospf 1
Router-C(config-router)# router-id 3.3.3.3
Router-C(config-router)# network 10.0.0.0 0.0.0.7 area 0
Router-C(config-router)# network 192.168.3.0 0.0.0.255 area 0
Router-C(config-router)# passive-interface fastEthernet 0/1
Router-C(config-router)# exit
```

> [!TIP]
> **¿Por qué usar `passive-interface` en las LANs?**
> Evita que se envíen paquetes Hello innecesarios hacia equipos terminales (PCs, impresoras). Esto previene el desperdicio de ancho de banda y mitiga riesgos de seguridad (evitando que un atacante inyecte rutas falsas conectando un router no autorizado).

---

### 7.3 Manipulación de la Elección de DR y BDR

#### Opción A: Por Prioridad de Interfaz (Método más confiable)
Forzar a que **RC** sea el **DR** del segmento multiacceso:
```text
Router-C(config)# interface fastEthernet 0/0
Router-C(config-if)# ip ospf priority 255
Router-C(config-if)# exit
```

Evitar que un router participe de la elección (DROther permanente):
```text
Router-A(config)# interface fastEthernet 0/0
Router-A(config-if)# ip ospf priority 0
Router-A(config-if)# exit
```

#### Opción B: Por Router ID mediante Loopback
```text
Router-A(config)# interface loopback 0
Router-A(config-if)# ip address 200.0.0.1 255.255.255.255
Router-A(config-if)# exit
```

#### Opción C: Por Comando Manual `router-id`
```text
Router-B(config)# router ospf 1
Router-B(config-router)# router-id 201.0.0.1
Router-B(config-router)# exit
```

> [!IMPORTANT]
> Para aplicar cambios en el Router ID o forzar una nueva elección de DR/BDR sin reiniciar físicamente los equipos, ejecute en modo privilegiado:
> ```text
> Router# clear ip ospf process
> Reset ALL OSPF processes? [no]: yes
> ```

---

## 8. Fundamentos de OSPFv3 (Soporte IPv6)

OSPFv3 es la evolución de OSPF diseñada específicamente para IPv6 (RFC 5340).

### Diferencias Clave entre OSPFv2 y OSPFv3:
1. **Habilitación en la interfaz**: OSPFv3 no utiliza el comando global `network <red> <wildcard>`; se habilita directamente dentro del modo de configuración de cada interfaz.
2. **Router ID**: Aunque enruta prefijos IPv6 de 128 bits, OSPFv3 sigue requiriendo obligatoriamente un **Router ID de 32 bits** (con formato decimal con punto idéntico a una dirección IPv4).
3. **Siguiente Salto Link-Local**: Las adyacencias y rutas en OSPFv3 utilizan como siguiente salto la dirección **Link-Local (`FE80::/10`)** de la interfaz del vecino.

### Configuración Paso a Paso de OSPFv3:
```text
Router(config)# ipv6 unicast-routing
Router(config)# ipv6 router ospf 1
Router(config-rtr)# router-id 1.1.1.1
Router(config-rtr)# exit

Router(config)# interface fastEthernet 0/0
Router(config-if)# ipv6 address 2001:db8:acad:1::1/64
Router(config-if)# ipv6 ospf 1 area 0
Router(config-if)# exit
```

---

## 9. Comandos de Verificación y Diagnóstico

| Comando | Propósito y Salida Clave |
|---|---|
| `show ip ospf neighbor` | Muestra los vecinos OSPF, su estado de adyacencia (`FULL/DR`, `FULL/BDR`, `2-WAY/DROTHER`), la IP y la interfaz local. |
| `show ip ospf interface <id>` | Revela el costo asignado, el rol en el segmento (DR/BDR/DROther), temporizadores Hello/Dead y prioridad. |
| `show ip ospf database` | Lista todos los LSAs contenidos en la LSDB local del router. |
| `show ip route ospf` | Muestra únicamente las rutas aprendidas dinámicamente mediante OSPF (marcadas con `O`). |
| `show ip protocols` | Detalla los IDs de proceso OSPF activos, el Router ID, las redes declaradas y las interfaces pasivas. |

---

## Resumen Ejecutivo

- OSPF es un protocolo IGP de estado de enlace con distancia administrativa **110**, basado en el algoritmo de Dijkstra.
- Administra tres estructuras de datos: **Tabla de Vecinos**, **LSDB** (idéntica en toda el área) y **Tabla de Enrutamiento**.
- Se comunica mediante 5 paquetes: Hello, DBD, LSR, LSU y LSAck.
- En segmentos multiacceso, previene la saturación $\frac{N(N-1)}{2}$ eligiendo un **DR** y un **BDR** a través de las IPs multicast `224.0.0.5` y `224.0.0.6`.
- La elección de DR se basa en **Prioridad más alta** (0 descalifica) y, en caso de empate, en el **Router ID más alto**.
- Las interfaces hacia clientes finales deben configurarse como `passive-interface`.
- OSPFv3 transporta IPv6 y se activa directamente en la interfaz del router manteniendo un Router ID de 32 bits.
