# Nota Avanzada - Clase 6: Enlaces WAN, HDLC y PPP con PAP y CHAP

## Introducción

Los enlaces seriales de la **Tarea 6 PPP** permiten comparar HDLC y PPP, separar la conectividad física de la autenticación y comprobar después el enrutamiento OSPF. Esta nota reconstruye el tema a partir de la tarea; el repositorio no contenía un apunte original de clase 6.

**PPP (Point-to-Point Protocol)** establece un enlace de capa 2 entre dos extremos. Puede autenticar al vecino mediante PAP o CHAP, pero esa autenticación no cifra el tráfico de los usuarios.

---

## 1. HDLC, PPP y sus Componentes

| Elemento | Función |
|---|---|
| **Cisco HDLC** | Encapsulación serial utilizada en el enlace R1–R3 de la tarea |
| **PPP** | Encapsulación punto a punto con negociación de parámetros |
| **LCP** | Establece, configura y termina el enlace PPP; negocia opciones como autenticación |
| **PAP / CHAP** | Autentican al vecino si se configuró autenticación |
| **NCP** | Negocia parámetros del protocolo de red; IPCP corresponde a IPv4 |

```text
Enlace físico --> LCP --> Autenticación, si se exige --> NCP --> Tráfico IP
```

Ambos extremos de un mismo enlace deben usar encapsulaciones compatibles. Una dirección IP correcta no resuelve una diferencia entre PPP y HDLC.

### 1.1 DCE y DTE

En un cable serial de laboratorio, el extremo **DCE** suministra la señal de reloj. Verificarlo con `show controllers serial 0/0/0` antes de configurar `clock rate 64000`.

`clock rate` configura el reloj; `bandwidth` informa un ancho de banda lógico usado por protocolos y cálculos de métricas. Cambiar `bandwidth` no cambia la velocidad física del enlace.

---

## 2. Direccionamiento de la Tarea 6

```text
             PAP
       R1 -------- R2
        \          /
    HDLC \        / CHAP
          \      /
             R3
```

| Enlace | Red / máscara | Extremo 1 | Extremo 2 |
|---|---|---|---|
| R1–R2 | `172.16.0.0/30` | R1 S0/0/0: `.1` | R2 S0/0/0: `.2` |
| R2–R3 | `172.16.0.4/30` | R2 S0/0/1: `.5` | R3 S0/0/1: `.6` |
| R1–R3 | `172.16.0.8/30` | R1 S0/0/1: `.9` | R3 S0/0/0: `.10` |
| LAN R1 | `10.0.0.0/25` | Gateway `10.0.0.1` | Broadcast `10.0.0.127` |
| LAN R3 | `10.0.0.128/25` | Gateway `10.0.0.129` | Broadcast `10.0.0.255` |

Las redes `/30` usan máscara `255.255.255.252` y wildcard `0.0.0.3`. Las LANs `/25` usan máscara `255.255.255.128` y wildcard `0.0.0.127`.

---

## 3. PAP: Credenciales Enviadas por el Enlace

PAP envía usuario y contraseña sin cifrar. En la autenticación mutua, cada router solicita verificar al otro y también presenta sus propias credenciales. [Cisco: configuración PAP](https://www.cisco.com/c/en/us/support/docs/wan/point-to-point-protocol-ppp/10313-config-pap.html).

### R1: Configuración desde Modo Global

```text
hostname R1
username R2 password CISCO
interface serial 0/0/0
 ip address 172.16.0.1 255.255.255.252
 encapsulation ppp
 ppp authentication pap
 ppp pap sent-username R1 password CISCO
 clock rate 64000
 no shutdown
 exit
```

### R2: Configuración del Otro Extremo

```text
hostname R2
username R1 password CISCO
interface serial 0/0/0
 ip address 172.16.0.2 255.255.255.252
 encapsulation ppp
 ppp authentication pap
 ppp pap sent-username R2 password CISCO
 no shutdown
 exit
```

Las contraseñas reproducen el laboratorio. `username R2` en R1 valida las credenciales recibidas de R2; `ppp pap sent-username R1` define lo que R1 envía. No son comandos intercambiables.

---

## 4. CHAP: Desafío y Respuesta

CHAP utiliza un desafío y una respuesta calculada con un secreto compartido. Por defecto, Cisco identifica al router mediante su **hostname**; la entrada `username` del vecino debe coincidir, incluidas mayúsculas. [Cisco: funcionamiento de CHAP](https://www.cisco.com/c/en/us/support/docs/wan/point-to-point-protocol-ppp/25647-understanding-ppp-chap.html).

```text
R2                           R3
 | ------ Desafío ----------> |
 | <----- Respuesta -------- |
 | ------ Éxito / Fallo ----> |
```

### R2

```text
username R3 password CISCO
interface serial 0/0/1
 ip address 172.16.0.5 255.255.255.252
 encapsulation ppp
 ppp authentication chap
 clock rate 64000
 no shutdown
 exit
```

### R3

```text
hostname R3
username R2 password CISCO
interface serial 0/0/1
 ip address 172.16.0.6 255.255.255.252
 encapsulation ppp
 ppp authentication chap
 no shutdown
 exit
```

| Comparación | PAP | CHAP |
|---|---|---|
| Intercambio | Credenciales y aceptación/rechazo | Desafío, respuesta y resultado |
| Contraseña por el enlace | Se transmite en claro | No se transmite directamente |
| Identidad en este laboratorio | `ppp pap sent-username` | Hostname del router |
| Comando de autenticación | `ppp authentication pap` | `ppp authentication chap` |
| Cifrado del tráfico IP | No | No |

---

## 5. HDLC entre R1 y R3

En R1:

```text
interface serial 0/0/1
 ip address 172.16.0.9 255.255.255.252
 encapsulation hdlc
 no shutdown
 exit
```

En R3:

```text
interface serial 0/0/0
 ip address 172.16.0.10 255.255.255.252
 encapsulation hdlc
 clock rate 64000
 no shutdown
 exit
```

Los relojes de los ejemplos corresponden a los extremos indicados por las notas; comprobar el DCE si se reconstruye el cableado.

---

## 6. Integración con OSPF

Después de comprobar los vecinos directamente conectados, anunciar las redes:

```text
! R1
router ospf 1
 network 172.16.0.0 0.0.0.3 area 0
 network 172.16.0.8 0.0.0.3 area 0
 network 10.0.0.0 0.0.0.127 area 0
 passive-interface fastEthernet 0/1
 exit

! R2: introducir en R2
router ospf 1
 network 172.16.0.0 0.0.0.3 area 0
 network 172.16.0.4 0.0.0.3 area 0
 exit

! R3: introducir en R3
router ospf 1
 network 172.16.0.4 0.0.0.3 area 0
 network 172.16.0.8 0.0.0.3 area 0
 network 10.0.0.128 0.0.0.127 area 0
 passive-interface fastEthernet 0/1
 exit
```

Las LANs se configuran con los gateways de la tabla y `no shutdown`. Los equipos finales deben usar su gateway correspondiente.

---

## 7. Diagnóstico por Capas

| Síntoma | Comprobación |
|---|---|
| `administratively down` | Falta `no shutdown` |
| Interfaz física caída | Cable, módulo, extremo remoto y reloj DCE |
| Física activa, protocolo caído | Encapsulación, LCP y autenticación |
| PPP activo pero sin vecino OSPF | Área, anuncio de interfaz, timers e interfaz pasiva |
| Vecindad correcta pero sin acceso entre PCs | Rutas, direcciones, gateway y filtros |

```text
show ip interface brief
show interfaces serial 0/0/0
show interfaces serial 0/0/1
show ip ospf neighbor
show ip route
debug ppp authentication
undebug all
```

Verificar **cada enlace directamente**: la topología triangular puede conservar conectividad por otro camino y ocultar que PAP o CHAP está fallando.

---

## Resumen Ejecutivo

- PPP negocia el enlace con LCP y el protocolo de red con NCP.
- PAP y CHAP autentican al vecino; no cifran los datos.
- En PAP se distingue la cuenta local del vecino de las credenciales enviadas.
- En CHAP se comprueban hostname, username y secreto compartido.
- El reloj se configura en el DCE y la encapsulación debe ser compatible en ambos extremos.
- Primero verificar el enlace; después OSPF y las LANs.

## Referencias

- [Notas originales de Tarea 6 PPP](../../Topolog%C3%ADas/Topolog%C3%ADas%20de%20Modulo%203/Notas%20de%20Tarea%206%20PPP.txt).
- Cisco: [PAP](https://www.cisco.com/c/en/us/support/docs/wan/point-to-point-protocol-ppp/10313-config-pap.html) y [CHAP](https://www.cisco.com/c/en/us/support/docs/wan/point-to-point-protocol-ppp/25647-understanding-ppp-chap.html).
