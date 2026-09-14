# Nota Avanzada - Clase 2: OSPF Multiárea, Optimización de Métricas, Temporizadores y Tipos de LSA

## Introducción

A medida que una red corporativa escala, mantener un único dominio de enrutamiento OSPF (Single-Area) genera serios problemas de rendimiento: bases de datos topológicas (LSDB) masivas, alto consumo de memoria RAM y CPU, y recálculos continuos del algoritmo SPF ante cualquier oscilación de enlace (*link flap*).

Para resolver estos desafíos, OSPF implementa un diseño **jerárquico multiárea**. Esta nota avanzada aborda la arquitectura multiárea, los roles especializados de los routers (ABR, ASBR), la sumarización de rutas en las fronteras, la taxonomía de los **Link-State Advertisements (LSAs)**, la calibración de métricas en infraestructuras de alta velocidad y el ajuste de temporizadores.

---

## 1. Arquitectura Jerárquica de OSPF Multiárea

OSPF divide un Sistema Autónomo en múltiples áreas lógicas conectadas entre sí a través de un núcleo central.

```text
               ┌─────────────────────────────────────────┐
               │         ÁREA 0 (BACKBONE / TRONCAL)     │
               │            (Núcleo de Tránsito)         │
               └───────────┬─────────────────┬───────────┘
                           │                 │
              ┌────────────┴────┐       ┌────┴────────────┐
              │    ROUTER ABR   │       │   ROUTER ABR    │
              └────────────┬────┘       └────┬────────────┘
                           │                 │
        ┌──────────────────┴──────┐   ┌──────┴──────────────────┐
        │   ÁREA 1 (Estándar)     │   │   ÁREA 2 (Estándar)     │
        │   Edificio Ventas       │   │   Centro de Datos       │
        └─────────────────────────┘   └─────────────────────────┘
```

### 1.1 Tipos de Áreas
1. **Área 0 (Área Backbone / Red Troncal)**:
   - Es el núcleo obligatorio de toda red OSPF multiárea.
   - Su función primordial es interconectar de forma rápida y sin bucles todas las demás áreas entre sí.
   - **Regla inquebrantable**: Todas las áreas que no sean el Área 0 deben tener conexión directa (o mediante un túnel lógico *virtual-link*) hacia el Área 0.
2. **Áreas Estándar / Regulares (No-Backbone)**:
   - Conectan usuarios finales y recursos locales.
   - Aíslan el tráfico de LSAs internos: los aleteos de enlaces dentro del Área 1 no provocan recálculos SPF en el Área 2.

### 1.2 Beneficios del Diseño Multiárea
- **LSDB más pequeña por router**: Cada router interno solo almacena el mapa topológico detallado de su propia área.
- **Menor sobrecarga de CPU/RAM**: El algoritmo SPF se ejecuta de manera localizada.
- **Reducción del tamaño de la tabla de enrutamiento**: Permite condensar cientos de subredes en un único prefijo sumarizado en la frontera.

---

## 2. Roles de Routers en OSPF Multiárea

Un router en OSPF puede desempeñar uno o varios de los siguientes roles según su ubicación física y lógica:

| Rol | Sigla | Definición y Función |
|---|:---:|---|
| **Router Interno** | *Internal* | Tiene **todas** sus interfaces activas dentro de una misma área OSPF. Posee la LSDB completa de esa área únicamente. |
| **Router de Red Troncal** | *Backbone* | Cualquier router que posea al menos una interfaz perteneciente al **Área 0** (incluye ABRs). |
| **Router de Borde de Área** | **ABR** | *Area Border Router*. Dispositivo situado en la frontera entre dos o más áreas (siempre con una interfaz en el Área 0 y otra en un área estándar). Mantiene una LSDB separada por cada área y **convierte LSAs Tipo 1/2 en LSAs Tipo 3**. |
| **Router de Límite de Sistema Autónomo** | **ASBR** | *Autonomous System Boundary Router*. Router que conecta el dominio OSPF con una red externa (otro protocolo como BGP/EIGRP o una ruta estática a Internet). Es el encargado de redistribuir rutas externas generando **LSAs Tipo 5**. |

---

## 3. Taxonomía de los Paquetes LSA (Link-State Advertisements)

Los LSAs son los bloques de construcción que conforman la LSDB. Cada tipo de LSA describe una parte específica de la topología y se propaga con reglas de inundación (*flooding*) distintas:

| Tipo LSA | Nombre Técnico | Generado Por | Ámbito de Difusión (*Flooding*) | Código en Tabla (`show ip route`) |
|:---:|---|---|---|:---:|
| **Tipo 1** | **Router LSA** | Todo router del área | Se difunde únicamente dentro de la **misma área**. Describe los enlaces e IPs directas del router. | **`O`** |
| **Tipo 2** | **Network LSA** | El **DR** del segmento multiacceso | Se difunde únicamente dentro de la **misma área**. Describe la red multiacceso y lista todos los routers conectados al switch. | **`O`** |
| **Tipo 3** | **Summary LSA** (Resumen de Red) | **ABR** | Se difunde entre áreas a través del Área 0. Informa a otras áreas sobre las redes existentes sin incluir la topología interna. | **`O IA`**<br>(*Inter-Area*) |
| **Tipo 4** | **ASBR Summary LSA** | **ABR** | Se difunde a través de las áreas. Informa a los demás routers la ubicación exacta y el costo para alcanzar al ASBR. | **`O IA`** |
| **Tipo 5** | **AS External LSA** | **ASBR** | Se difunde por **todo el dominio OSPF** (excepto áreas Stub). Describe redes externas redistribuidas o la ruta por defecto hacia Internet. | **`O E1`** / **`O E2`** |

> [!NOTE]
> **Diferencia entre O E1 y O E2:**
> - **E2 (Por defecto en Cisco)**: El costo de la ruta es fijo y refleja únicamente el costo reportado por el ASBR hacia la red externa (ignora el costo interno de los saltos dentro de OSPF).
> - **E1**: El costo es acumulativo (costo externo reportado por el ASBR + costo de todos los saltos internos OSPF para llegar al ASBR).

---

## 4. Métodos de Publicación de Redes en OSPF

En Cisco IOS existen dos métodos para declarar interfaces dentro de OSPF:

### Método 1: Comando Global `network`
Se especifica la subred y la máscara wildcard en el submodo del proceso OSPF:

```text
! Publicar por subred completa:
Router(config)# router ospf 1
Router(config-router)# network 192.168.10.0 0.0.0.255 area 0
Router(config-router)# exit

! Publicar la IP exacta de una interfaz con wildcard 0.0.0.0 (Host Mask):
Router(config)# router ospf 1
Router(config-router)# network 10.0.0.1 0.0.0.0 area 0
Router(config-router)# network 10.0.0.5 0.0.0.0 area 1
Router(config-router)# exit
```

### Método 2: Habilitación Directa bajo la Interfaz (Método Moderno Recomendado)
Se entra a la interfaz física o subinterfaz y se asocia directamente al proceso y área:

```text
Router(config)# router ospf 1
Router(config-router)# router-id 1.1.1.1
Router(config-router)# exit

Router(config)# interface gigabitEthernet 0/0/0
Router(config-if)# ip ospf 1 area 0
Router(config-if)# exit

Router(config)# interface gigabitEthernet 0/0/1
Router(config-if)# ip ospf 1 area 1
Router(config-if)# exit
```
*Ventajas del Método 2*: No requiere calcular máscaras wildcard y evita publicar accidentalmente interfaces adicionales que compartan el mismo rango.

---

## 5. Sumarización de Rutas Inter-Área en el ABR

> [!IMPORTANT]
> En OSPF, la sumarización de rutas **NO** se puede realizar en cualquier router arbitrario ni dentro de la misma área. La sumarización inter-área se efectúa **exclusivamente en los routers ABR**.

### 5.1 Caso Práctico de Cálculo de Sumarización
Supongamos que en el **Área 1** se encuentran las siguientes 4 subredes que el ABR debe publicar hacia el Área 0:
- `192.168.10.0/24`
- `192.168.20.0/24`
- `192.168.30.0/24`
- `192.168.40.0/24`

**Paso 1: Convertir a binario los octetos donde difieren:**
```text
Subred 1 (10):  192.168. 0 0 0 0 1 0 1 0 .0
Subred 2 (20):  192.168. 0 0 0 1 0 1 0 0 .0
Subred 3 (30):  192.168. 0 0 0 1 1 1 1 0 .0
Subred 4 (40):  192.168. 0 0 1 0 1 0 0 0 .0
                         | |
Coincidencia:            0 0 (Solo los 2 primeros bits del 3er octeto son idénticos)
```

**Paso 2: Determinar la longitud del prefijo común:**
- 1er octeto: 8 bits coincidentes
- 2do octeto: 8 bits coincidentes
- 3er octeto: 2 bits coincidentes (`00xxxxxx`)
- **Total de bits de máscara de red**: $8 + 8 + 2 = \mathbf{18\text{ bits}}$ (`/18`).

**Paso 3: Calcular la dirección de red base y máscara decimal:**
- Dirección base: `192.168.0.0`
- Máscara de subred `/18`: `11111111.11111111.11000000.00000000` = `255.255.192.0`.

### 5.2 Configuración en el Router ABR
Se utiliza el comando `area <área-origen> range <dirección-sumarizada> <máscara>`:

```text
ABR-Router(config)# router ospf 1
ABR-Router(config-router)# area 1 range 192.168.0.0 255.255.192.0
ABR-Router(config-router)# exit
```

Al aplicar este comando, el ABR suprime la emisión de múltiples LSAs Tipo 3 individuales y emite **un único LSA Tipo 3** hacia el Área 0 con el prefijo `/18`, reduciendo drásticamente las tablas de los demás routers.

---

## 6. Optimización y Calibración del Costo OSPF

### 6.1 El Problema del Ancho de Banda de Referencia por Defecto
La fórmula del costo es:
$$\text{Costo} = \frac{\text{Reference Bandwidth}}{\text{Interface Bandwidth}}$$

El valor predeterminado de Cisco IOS es **$100\text{ Mbps}$** ($10^8\text{ bps}$). Dado que el costo mínimo es 1:
- Enlace FastEthernet ($100\text{ Mbps}$): $100 / 100 = 1$
- Enlace GigabitEthernet ($1000\text{ Mbps}$): $100 / 1000 = 0.1 \to \mathbf{1}$
- Enlace 10-GigabitEthernet ($10000\text{ Mbps}$): $100 / 10000 = 0.01 \to \mathbf{1}$

**Consecuencia**: OSPF considerará que una interfaz de $100\text{ Mbps}$ y una de $10\text{ Gbps}$ tienen exactamente el mismo costo, tomando decisiones de enrutamiento subóptimas.

### 6.2 Solución 1: Modificar el Ancho de Banda de Referencia (Recomendado)
Ajustar el valor globalmente a $10\text{ Gbps}$ ($10000\text{ Mbps}$) o $100\text{ Gbps}$ ($100000\text{ Mbps}$):

```text
Router(config)# router ospf 1
Router(config-router)# auto-cost reference-bandwidth 10000
% OSPF: Reference bandwidth is changed.
        Please ensure reference bandwidth is consistent across all routers.
Router(config-router)# exit
```

> [!CAUTION]
> El comando `auto-cost reference-bandwidth` **debe configurarse de manera idéntica en TODOS los routers** del dominio OSPF. De lo contrario, los routers tendrán métricas incompatibles y podrían generarse bucles de enrutamiento.

Con referencia de $10000\text{ Mbps}$ ($10\text{ Gbps}$):
- FastEthernet ($100\text{ M}$): Costo = $10000 / 100 = \mathbf{100}$
- GigabitEthernet ($1\text{ G}$): Costo = $10000 / 1000 = \mathbf{10}$
- 10-GigabitEthernet ($10\text{ G}$): Costo = $10000 / 10000 = \mathbf{1}$

### 6.3 Solución 2: Forzar el Costo Manual en la Interfaz
Para desviar o priorizar tráfico por un enlace específico sin alterar la fórmula global:

```text
Router(config)# interface gigabitEthernet 0/0/0
Router(config-if)# ip ospf cost 5
Router(config-if)# exit
```

---

## 7. Temporizadores OSPF (Hello y Dead Intervals)

Los temporizadores controlan la velocidad de detección de caídas de vecinos:

| Temporizador | Valor por Defecto (Broadcast / Point-to-Point) | Regla de Oro |
|---|:---:|---|
| **Hello Interval** | **10 segundos** | Intervalo periódico con que se envían paquetes Hello. |
| **Dead Interval** | **40 segundos** | Tiempo que el router espera antes de declarar a un vecino como caído. Por defecto es **4 veces el Hello**. |

> [!WARNING]
> **Condición estricta de adyacencia**: Para que dos routers OSPF formen vecindad, **los valores de Hello y Dead deben coincidir exactamente** en ambos extremos del enlace. Si difieren, los paquetes Hello serán descartados y no habrá adyacencia.

### Configuración de Temporizadores en la Interfaz:
```text
Router(config)# interface gigabitEthernet 0/0/0
Router(config-if)# ip ospf hello-interval 5
Router(config-if)# ip ospf dead-interval 20
Router(config-if)# exit
```

---

## 8. Inyección y Propagación de Rutas por Defecto

En la topología de borde (router conectado hacia el ISP o Internet), se debe configurar una ruta estática por defecto y redistribuirla automáticamente hacia todos los routers internos de OSPF.

```text
! 1. Configurar la ruta estática por defecto hacia el proveedor (IPv4 e IPv6)
Borde-Router(config)# ip route 0.0.0.0 0.0.0.0 serial 0/0/0
Borde-Router(config)# ipv6 route ::/0 serial 0/0/0

! 2. Inyectar la ruta en OSPF
Borde-Router(config)# router ospf 1
Borde-Router(config-router)# default-information originate
Borde-Router(config-router)# exit
```

> [!TIP]
> El comando `default-information originate` solo inyecta la ruta si la ruta estática `0.0.0.0/0` existe en la tabla de enrutamiento. Si se desea forzar la inyección incluso si no existe una ruta por defecto previa, se utiliza la palabra clave `always`:
> ```text
> Borde-Router(config-router)# default-information originate always
> ```

En los routers internos, esta ruta se aprenderá automáticamente como:
```text
O*E2 0.0.0.0/0 [110/1] via 10.0.0.1, GigabitEthernet0/0/0
```
(Donde `*` significa ruta candidata por defecto, `O` aprendida por OSPF, y `E2` LSA Tipo 5).

---

## 9. Comandos de Verificación Multiárea

| Comando | Utilidad Diagnóstica |
|---|---|
| `show ip ospf border-routers` | Lista todos los ABRs y ASBRs descubiertos, su costo y su Router ID. |
| `show ip ospf database summary` | Detalla los LSAs Tipo 3 (redes entre áreas) recibidos en el router. |
| `show ip route ospf` | Muestra rutas intra-área (`O`), inter-área (`O IA`) y externas (`O E2` o `O E1`). |
| `show ip ospf interface <id>` | Muestra temporizadores Hello/Dead, costo aplicado y rol de la interfaz. |
| `show ip ospf database external` | Lista los LSAs Tipo 5 generados por los ASBRs. |

---

## Resumen Ejecutivo

- OSPF Multiárea divide la red en un **Área 0 central** (Backbone) y **áreas estándar**, reduciendo el tamaño de la LSDB y el cómputo de SPF.
- **ABR** conecta el Área 0 con áreas estándar y genera **LSAs Tipo 3 (O IA)**.
- **ASBR** conecta con sistemas externos y genera **LSAs Tipo 5 (O E1/E2)**.
- La sumarización inter-área se configura exclusivamente en el ABR con el comando `area <id> range <red> <máscara>`.
- En infraestructuras modernas de $1\text{ Gbps}$ y superiores, se debe ajustar el `auto-cost reference-bandwidth 10000` de forma homogénea en toda la red.
- Los temporizadores Hello y Dead **deben ser idénticos** entre routers vecinos para formar adyacencia.
- La ruta por defecto se inyecta desde el router de borde mediante `default-information originate`.
