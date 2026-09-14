# Nota Avanzada - Clase 3: Seguridad de Red y Listas de Control de Acceso (ACLs IPv4)

## Introducción

La seguridad perimetral y el filtrado granular de paquetes son pilares esenciales en el diseño de redes empresariales. Las **Listas de Control de Acceso (ACLs - *Access Control Lists*)** constituyen la herramienta primaria en routers y switches multicapa de Cisco para implementar políticas de seguridad, controlar el flujo de tráfico, mitigar vectores de ataque y priorizar servicios.

Esta nota avanzada profundiza en los fundamentos criptográficos de la tríada de seguridad (**CIA**), los mecanismos internos de evaluación secuencial de las ACLs, la lógica binaria de las máscaras wildcard, las diferencias arquitectónicas y de ubicación entre ACLs Estándar y Extendidas, y la resolución paso a paso de escenarios típicos de examen de certificación CCNA.

---

## 1. Fundamentos de Seguridad de la Información (Tríada CIA)

Cualquier política de control de acceso corporativa se fundamenta en los tres pilares de la seguridad informática (**Tríada CIA**):

```text
                     ┌─────────────────────────────┐
                     │    CONFIDENCIALIDAD (C)     │
                     │  Privacidad de los datos    │
                     └──────────────┬──────────────┘
                                    │
                                    │
           ┌────────────────────────┴────────────────────────┐
           │                                                 │
┌──────────┴──────────┐                           ┌──────────┴──────────┐
│   INTEGRIDAD (I)    │                           │ DISPONIBILIDAD (A)  │
│ Datos inalterados   │                           │ Sistemas y datos    │
│    en tránsito      │                           │ accesibles 24/7     │
└─────────────────────┘                           └─────────────────────┘
```

### 1.1 Confidencialidad (*Confidentiality*)
Garantiza que la información solo sea accesible por usuarios o procesos expresamente autorizados mediante técnicas de cifrado:
- **Cifrado Simétrico** (Misma clave secreta compartida para cifrar y descifrar):
  - *DES* (56 bits - obsoleto y vulnerable).
  - *3DES* (168 bits - en desuso).
  - *AES* (*Advanced Encryption Standard* - 128, 192 o 256 bits): Estándar moderno de la industria, altamente seguro y eficiente.
- **Cifrado Asimétrico** (Par de claves: clave pública para cifrar y clave privada para descifrar):
  - *RSA* (comúnmente de 2048 o 4096 bits), *ECC* (Criptografía de Curva Elíptica), *Diffie-Hellman* (intercambio seguro de claves).

### 1.2 Integridad (*Integrity*)
Garantiza que el mensaje o paquete no haya sido alterado, corrompido o manipulado durante el tránsito:
- **Funciones Hash Criptográficas** (Unidireccionales y de longitud fija):
  - *MD5* (128 bits - susceptible a colisiones, no recomendado para alta seguridad).
  - *SHA-1* (160 bits - deprecado).
  - *SHA-2* (*SHA-256*, *SHA-512*): Estándar actual altamente confiable.
- **HMAC (*Hashed Message Authentication Code*)**: Combina una función hash criptográfica con una clave secreta compartida, garantizando simultáneamente **integridad y autenticación de origen**.

### 1.3 Disponibilidad (*Availability*)
Garantiza que los servicios, redes y datos permanezcan operativos y accesibles para los usuarios autorizados cuando los requieran. Las ACLs protegen la disponibilidad al bloquear ataques de denegación de servicio (DoS), inundaciones de tráfico anómalo o escaneos de puertos no autorizados.

---

## 2. Principios y Arquitectura de las ACLs

Una ACL es un conjunto ordenado de sentencias o reglas (`permit` o `deny`) que el router aplica secuencialmente a los paquetes que atraviesan una interfaz o intentan acceder al dispositivo.

### 2.1 Reglas Fundamentales de Procesamiento
1. **Evaluación Secuencial Top-Down (De arriba hacia abajo)**:
   El router compara el paquete contra cada línea de la ACL en orden estricto de secuencia.
2. **Coincidencia Inmediata (*First Match Execution*)**:
   En el momento exacto en que un paquete cumple con los criterios de una sentencia, se ejecuta la acción (`permit` o `deny`) y **se detiene inmediatamente la evaluación de las siguientes líneas**.
3. **El Deny Implícito al Final (*Implicit Deny All*)**:
   Toda ACL en Cisco IOS tiene una última línea invisible e ineludible:
   - En estándar: `deny any`
   - En extendida: `deny ip any any`
   > [!CRITICAL]
   > Si un paquete no coincide con ninguna de las sentencias explícitas de la lista, **será descartado silenciosamente**. Por este motivo, cualquier ACL que contenga sentencias `deny` requiere al menos una sentencia `permit` (como `permit ip any any`) para no aislar la red por completo.
4. **Criterio de Diseño**:
   Las sentencias más específicas (por ejemplo, permitir o bloquear un host en particular) deben posicionarse siempre **antes** de las sentencias generales (por ejemplo, subredes completas o `any`).

---

## 3. Lógica de la Máscara Wildcard (*Wildcard Mask*)

A diferencia de las máscaras de subred convencionales (donde los 1s indican red y los 0s host), en una máscara wildcard:
- **Bit `0`**: El bit correspondiente en la dirección IP **DEBE coincidir exactamente**.
- **Bit `1`**: El bit correspondiente en la dirección IP es ignorado (**"no me importa" / *wildcard***).

```text
Cálculo rápido de la Wildcard:
  255.255.255.255 (Máscara base completa)
- 255.255.255.0   (Máscara de subred /24)
─────────────────
    0.  0.  0.255 (Wildcard resultante)
```

### Palabras Clave de Reemplazo:
- **`host <IP>`**: Equivale a `<IP> 0.0.0.0` (coincidencia exacta de los 32 bits de un único dispositivo).
- **`any`**: Equivale a `0.0.0.0 255.255.255.255` (ignora los 32 bits, coincide con cualquier dirección IP del mundo).

---

## 4. Comparativa: ACLs Estándar vs ACLs Extendidas

| Parámetro | ACLs Estándar | ACLs Extendidas |
|---|---|---|
| **Criterio de Filtrado** | **Únicamente la Dirección IP de Origen**. | **IP de Origen, IP de Destino, Protocolo de Capa 3 (IP, ICMP) y Puertos L4 (TCP, UDP)**. |
| **Rango Numerado Clásico** | **1 a 99** | **100 a 199** |
| **Rango Numerado Expandido** | **1300 a 1999** | **2000 a 2699** |
| **Soporte Nombrado** | `ip access-list standard <NOMBRE>` | `ip access-list extended <NOMBRE>` |
| **Ubicación Recomendada** | **Lo más cerca posible del DESTINO**. | **Lo más cerca posible del ORIGEN**. |

```text
┌────────────────────────────────────────────────────────────────────────┐
│                        REGLA DE ORO DE UBICACIÓN                       │
├───────────────────────────────────┬────────────────────────────────────┤
│ ACL ESTÁNDAR ──> Cerca del DESTINO│ ACL EXTENDIDA ──> Cerca del ORIGEN │
├───────────────────────────────────┼────────────────────────────────────┤
│ ¿Por qué? Como solo filtra por    │ ¿Por qué? Al filtrar origen,       │
│ origen, si se coloca cerca del    │ destino y puerto específico, evita │
│ origen bloquearía el acceso del   │ que tráfico no deseado consuma     │
│ host hacia TODOS los destinos.    │ ancho de banda en la red troncal.  │
└───────────────────────────────────┴────────────────────────────────────┘
```

---

## 5. ACLs Estándar (Configuración y Ejemplos)

### 5.1 Sintaxis Numerada y Nombrada
```text
! Estándar Numerada (1-99):
Router(config)# access-list 10 permit 192.168.1.0 0.0.0.255
Router(config)# access-list 10 deny any ! (No es obligatorio escribirlo, ya existe implícito)

! Estándar Nombrada:
Router(config)# ip access-list standard BLOQUEO_VENTAS
Router(config-std-nacl)# deny 192.168.10.0 0.0.0.255
Router(config-std-nacl)# permit any
Router(config-std-nacl)# exit
```

### 5.2 Análisis de Buenas y Malas Prácticas en el Orden
```text
! DISEÑO CORRECTO: El host específico va primero
access-list 1 permit host 192.168.0.10
access-list 1 deny 192.168.0.0 0.0.0.255
access-list 1 permit any

! DISEÑO INCORRECTO: El host 192.168.0.10 NUNCA será evaluado porque
! coincide primero con la línea 'deny' de la subred completa /24.
access-list 1 deny 192.168.0.0 0.0.0.255
access-list 1 permit host 192.168.0.10  ! <-- Línea muerta (Inalcanzable)
access-list 1 permit any
```

---

## 6. ACLs Extendidas (Configuración y Operadores L4)

Las ACLs extendidas inspeccionan el encabezado de Capa 3 y Capa 4:

### 6.1 Operadores de Comparación de Puertos
- **`eq`** (*Equal*): Coincidencia exacta (ej. `eq 80`).
- **`neq`** (*Not Equal*): Cualquier puerto excepto el especificado.
- **`gt`** (*Greater Than*): Puertos mayores al indicado.
- **`lt`** (*Less Than*): Puertos menores al indicado.
- **`range`** (*Range*): Rango inclusivo de puertos (ej. `range 1024 65535`).

### 6.2 Puertos y Protocolos Comunes en Redes Cisco

| Protocolo de Transporte | Puerto | Servicio Asociado |
|:---:|:---:|---|
| **TCP** | **20 / 21** | FTP (Datos / Control) |
| **TCP** | **22** | SSH (Secure Shell) |
| **TCP** | **23** | Telnet (Texto plano inseguro) |
| **TCP** | **25** | SMTP (Envío de correo) |
| **UDP / TCP** | **53** | DNS (Domain Name System) |
| **UDP** | **67 / 68** | DHCP (Servidor / Cliente) |
| **TCP** | **80** | HTTP (Navegación web no cifrada) |
| **TCP** | **110** | POP3 (Recepción de correo) |
| **TCP** | **143** | IMAP (Recepción sincronizada de correo) |
| **UDP** | **161 / 162** | SNMP (Gestión y traps de red) |
| **TCP** | **443** | HTTPS (Navegación web segura / SSL-TLS) |

---

## 7. Aplicación de ACLs en Interfaces y Líneas VTY

### 7.1 En Interfaces Físicas o Subinterfaces
Se asocian mediante el comando `ip access-group <ID|Nombre> <in|out>`:
- **`in` (Entrada)**: Los paquetes se filtran **apenas entran al router**, antes de que la CPU procese la tabla de enrutamiento. Ahorra recursos si el paquete debe ser descartado.
- **`out` (Salida)**: Los paquetes se filtran **después de haber sido enrutados**, inmediatamente antes de ser transmitidos por el cable/fibra hacia afuera.

```text
Router(config)# interface gigabitEthernet 0/0/1
Router(config-if)# ip access-group FILTRO_WEB in
Router(config-if)# exit
```

### 7.2 En Líneas Virtuales de Administración (VTY - SSH/Telnet)
Para restringir quién puede conectarse a gestionar el router, se utiliza una **ACL estándar** aplicada con el comando `access-class`:

```text
! Permitir únicamente a la IP del administrador gestionar el router:
Router(config)# access-list 50 permit host 192.168.1.100

Router(config)# line vty 0 4
Router(config-line)# access-class 50 in
Router(config-line)# transport input ssh
Router(config-line)# exit
```

---

## 8. Escenarios Prácticos de Laboratorio y Certificación

### Escenario A: Denegar acceso Web a una subred, permitir todo lo demás

**Requerimiento**: La red `192.168.0.0/24` conectada en la interfaz `FastEthernet 0/0` del Router R1 no debe tener acceso a ningún servidor web (HTTP ni HTTPS). El resto del tráfico de esa red debe funcionar normalmente.

**Paso 1: Desglose Técnico:**
- Origen: `192.168.0.0 0.0.0.255`
- Destino: Cualquier destino (`any`)
- Protocolo: TCP
- Puertos a bloquear: `80` (HTTP) y `443` (HTTPS)
- Tráfico restante: Permitir explícitamente (`permit ip any any`)
- Ubicación: Router R1, interfaz de entrada `FastEthernet 0/0` (lo más cerca del origen).

**Paso 2: Configuración en Cisco IOS:**
```text
R1(config)# ip access-list extended BLOQUEO_WEB
R1(config-ext-nacl)# deny tcp 192.168.0.0 0.0.0.255 any eq 80
R1(config-ext-nacl)# deny tcp 192.168.0.0 0.0.0.255 any eq 443
R1(config-ext-nacl)# permit ip any any
R1(config-ext-nacl)# exit

R1(config)# interface fastEthernet 0/0
R1(config-if)# ip access-group BLOQUEO_WEB in
R1(config-if)# exit
```

---

### Escenario B: Permitir ÚNICAMENTE acceso Web, denegar todo lo demás

> [!WARNING]
> **Trampa clásica de examen**: Si solo configuras `permit tcp ... eq 80` y `permit tcp ... eq 443`, los usuarios **no podrán navegar**, porque antes de abrir una página web el navegador debe solicitar una IP por **DHCP** y resolver nombres de dominio mediante **DNS**. Si bloqueas DHCP y DNS, la web no funcionará.

**Paso 1: Desglose Técnico de Dependencias:**
1. **DHCP**: Necesario si los hosts obtienen IP dinámica (puertos UDP 67 y 68).
2. **DNS**: Necesario para resolver `www.google.com` a una dirección IP (puerto 53 UDP y TCP).
3. **Web**: Tráfico HTTP (puerto 80 TCP) y HTTPS (puerto 443 TCP).
4. **Bloqueo de todo lo demás**: Concedido automáticamente por el `deny ip any any` implícito al final.

**Paso 2: Configuración Completa:**
```text
R1(config)# ip access-list extended SOLO_NAVEGACION
! Permitir tráfico DHCP si el router reenvía solicitudes
R1(config-ext-nacl)# permit udp any any eq 67
R1(config-ext-nacl)# permit udp any any eq 68

! Permitir consultas DNS hacia cualquier servidor
R1(config-ext-nacl)# permit udp 192.168.0.0 0.0.0.255 any eq 53
R1(config-ext-nacl)# permit tcp 192.168.0.0 0.0.0.255 any eq 53

! Permitir navegación Web HTTP y HTTPS
R1(config-ext-nacl)# permit tcp 192.168.0.0 0.0.0.255 any eq 80
R1(config-ext-nacl)# permit tcp 192.168.0.0 0.0.0.255 any eq 443

! Denegar explícitamente el resto con fines de auditoría/conteo
R1(config-ext-nacl)# deny ip any any
R1(config-ext-nacl)# exit

R1(config)# interface fastEthernet 0/0
R1(config-if)# ip access-group SOLO_NAVEGACION in
R1(config-if)# exit
```

---

## 9. Edición Moderna mediante Números de Secuencia

En versiones antiguas de Cisco IOS, si se cometía un error en una línea, era necesario eliminar la lista completa con `no access-list X` y reescribirla. Con las ACLs nombradas se pueden insertar y borrar sentencias usando **números de secuencia**:

```text
Router# show access-lists BLOQUEO_WEB
Extended IP access list BLOQUEO_WEB
    10 deny tcp 192.168.0.0 0.0.0.255 any eq www
    20 deny tcp 192.168.0.0 0.0.0.255 any eq 443
    30 permit ip any any

! Insertar una excepción para un host privilegiado antes del bloqueo general:
Router(config)# ip access-list extended BLOQUEO_WEB
Router(config-ext-nacl)# 5 permit tcp host 192.168.0.10 any eq www
Router(config-ext-nacl)# 6 permit tcp host 192.168.0.10 any eq 443

! Eliminar una línea específica sin afectar al resto:
Router(config-ext-nacl)# no 10
Router(config-ext-nacl)# exit
```

---

## 10. Comandos de Verificación y Diagnóstico

| Comando | Función de Diagnóstico |
|---|---|
| `show access-lists` | Muestra todas las ACLs configuradas en el equipo junto con el **contador de coincidencias (*matches*)** de cada línea. |
| `show ip access-list <nombre>` | Detalla las sentencias y números de secuencia de una ACL específica. |
| `show ip interface <id>` | Revela si una interfaz física o lógica tiene una ACL aplicada en sentido de entrada (*Inbound access list*) o salida (*Outgoing access list*). |
| `clear access-list counters` | Reinicia a cero los contadores de coincidencias para pruebas de tráfico controladas. |

---

## Resumen Ejecutivo

- La tríada **CIA** (Confidencialidad, Integridad y Disponibilidad) rige las políticas de control de acceso.
- Las ACLs se leen de forma **Top-Down** y aplican la regla del **primer match**.
- Toda ACL finaliza con un **Deny Implícito** que descarta cualquier paquete no autorizado expresamente.
- **Máscara Wildcard**: `0` significa coincidencia exacta obligatoria; `1` significa ignorar el bit.
- **ACL Estándar**: Filtra únicamente por IP de origen. Se ubica **cerca del destino**.
- **ACL Extendida**: Filtra por origen, destino, protocolo y puertos. Se ubica **cerca del origen**.
- En interfaces se aplica con `ip access-group <in|out>`; en líneas VTY con `access-class in`.
- Al permitir tráfico web en una red estricta, siempre deben habilitarse previamente los servicios de soporte indispensables (**DNS** puerto 53 y **DHCP** puertos 67/68).
