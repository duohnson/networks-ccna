# Nota Avanzada - Clase 10: WAN, VPN e IPsec

## Introducción

Las clases 6 y 7 desarrollan PPP y Frame Relay de las tareas. Esta clase amplía la visión WAN con conexiones empresariales, VPN e IPsec, para cubrir los conceptos de ENSA que no estaban en esos apuntes.

Una WAN conecta ubicaciones distantes; una VPN crea conectividad lógica privada sobre una infraestructura compartida. La conectividad y la protección criptográfica deben analizarse por separado.

---

## 1. Componentes de una WAN

| Término | Significado |
|---|---|
| CPE | Equipo situado en las instalaciones del cliente |
| Demarcación | Límite de responsabilidad entre cliente y proveedor |
| Última milla | Tramo de acceso que conecta la sede con el proveedor |
| Red del proveedor | Infraestructura de transporte entre puntos de acceso |
| SLA | Acuerdo de servicio: disponibilidad, capacidad y otras condiciones |

```text
LAN sede A -- CPE -- Acceso -- Red del proveedor -- Acceso -- CPE -- LAN sede B
```

### Opciones de Acceso

| Opción | Característica | Aspecto a evaluar |
|---|---|---|
| Enlace dedicado | Capacidad contratada entre puntos | Costo, disponibilidad y redundancia |
| Metro Ethernet | Servicio Ethernet del operador | Alcance, VLANs y SLA |
| MPLS del proveedor | Transporte basado en etiquetas y servicios VPN | Separación lógica no implica cifrado |
| Internet empresarial | Conectividad IP compartida | VPN, capacidad y calidad variable |
| DSL / cable / fibra de acceso | Alternativas de última milla | Simetría y contención |
| Celular / satélite | Acceso o respaldo donde aplica | Cobertura, latencia y consumo |

La elección se basa en requisitos de las aplicaciones, sedes, continuidad y presupuesto. Frame Relay permanece en este repositorio por sus prácticas de laboratorio.

---

## 2. Topologías WAN

| Topología | Ventaja | Compromiso |
|---|---|---|
| Punto a punto | Relación simple entre dos extremos | Escala mediante más enlaces |
| Hub-and-spoke | Centraliza servicios y políticas | El hub concentra tráfico y dependencia |
| Malla completa | Caminos directos entre sedes | Más conexiones y complejidad |
| Malla parcial | Redundancia donde hace falta | Requiere diseñar los caminos |

Para N sedes, una malla completa de enlaces entre pares requiere `N(N-1)/2` relaciones. No confundir esta fórmula con las adyacencias FULL reducidas por DR/BDR en OSPF multiacceso.

---

## 3. VPN Site-to-Site y Acceso Remoto

```text
SITE-TO-SITE:
LAN A -- Gateway A == Túnel sobre Internet == Gateway B -- LAN B

ACCESO REMOTO:
Usuario remoto == Túnel sobre Internet == Gateway empresarial -- Servicios
```

| Tipo | Extremos | Uso |
|---|---|---|
| Site-to-site | Gateways de dos redes | Conectar sucursales |
| Acceso remoto | Cliente y gateway | Conectar un usuario fuera de la oficina |

En **full tunnel**, todo el tráfico definido por la política sale por la VPN. En **split tunnel**, solo los destinos seleccionados utilizan el túnel. Esta decisión cambia el camino de Internet y qué inspección se aplica.

---

## 4. Servicios de IPsec

IPsec es un conjunto de protocolos para proteger comunicaciones IP. Según la política negociada puede ofrecer confidencialidad, integridad, autenticación del origen y protección contra repetición.

| Elemento | Función |
|---|---|
| ESP | Protección del tráfico; puede aportar cifrado e integridad según los algoritmos |
| AH | Integridad y autenticación; no cifra el contenido y presenta dificultades con NAT |
| IKE | Negocia parámetros, autentica pares y establece material de claves |
| SA | Asociación de seguridad que define parámetros para un sentido de protección |
| Anti-replay | Detecta reutilización de paquetes mediante secuencias y ventana |

ESP utiliza protocolo IP 50 y AH protocolo IP 51: **no son puertos TCP/UDP**. IKE suele usar UDP 500; NAT Traversal utiliza UDP 4500 para atravesar NAT cuando se negocia.

### Modo Transporte y Modo Túnel

| Modo | Idea |
|---|---|
| Transporte | Protege la carga del paquete manteniendo su encabezado IP original como encabezado externo |
| Túnel | Encapsula el paquete IP original dentro de uno nuevo entre extremos del túnel |

```text
Sin túnel:       [IP original][Datos]
Túnel IPsec:    [IP externa][Protección IPsec: IP original + datos]
```

---

## 5. GRE, IPsec y VPN del Proveedor

**GRE** encapsula tráfico y puede transportar protocolos que una VPN IPsec basada únicamente en selectores no maneja de la misma forma. GRE por sí solo no aporta cifrado; puede combinarse con IPsec.

Una **VPN MPLS** separa redes del cliente dentro del proveedor, pero no supone automáticamente cifrado extremo a extremo. “Privado” en el servicio y “cifrado” en el transporte son propiedades distintas.

---

## 6. Diseño de un Ejemplo HQ–Sucursal

| Decisión | Ejemplo didáctico |
|---|---|
| LAN HQ | `10.0.1.0/24` |
| LAN sucursal | `10.1.10.0/24` |
| Transporte | Conectividad IP del proveedor |
| Extremos de VPN | Router de borde HQ y router de sucursal |
| Tráfico protegido | Comunicación entre ambas LANs |
| Enrutamiento | Rutas hacia la red remota por la VPN según el diseño |
| Validación | Contadores de cifrado/descifrado y pruebas desde hosts |

No copiar una configuración IPsec completa sin definir plataforma, IKE, autenticación y algoritmos compatibles. El objetivo CCNA de este apartado es explicar componentes y tipos de VPN; los comandos exactos varían entre plataformas.

### Problemas Frecuentes

| Síntoma | Revisar |
|---|---|
| No se negocia el túnel | Alcance entre pares, autenticación y propuestas compatibles |
| Túnel activo sin tráfico | Rutas, selectores, ACLs y tratamiento NAT |
| Funciona un sentido | Ruta de retorno y contadores por extremo |
| Paquetes pequeños pasan y grandes fallan | MTU, sobrecarga del túnel y Path MTU Discovery |

En IOS que los soporte, `show crypto ikev2 sa` y `show crypto ipsec sa` permiten revisar asociaciones y contadores. Su disponibilidad no se presupone en todos los dispositivos Packet Tracer.

---

## Resumen Ejecutivo

- WAN describe conectividad entre sedes; VPN describe una conexión lógica privada.
- Site-to-site conecta redes; acceso remoto conecta usuarios.
- IPsec puede proteger confidencialidad, integridad, origen y repetición.
- IKE negocia; ESP protege tráfico; GRE solo encapsula si no se combina con protección.
- Una VPN necesita rutas y retorno además de asociaciones de seguridad.

## Referencias

- [Cisco NetAcad: ENSA, WAN y VPN/IPsec](https://www.netacad.com/courses/ccna-enterprise-networking-security-automation?courseLang=en-US).
- [Clase 6: PPP](./Nota%20avanzada%20de%20clase%206.md).
- [Clase 7: Frame Relay](./Nota%20avanzada%20de%20clase%207.md).
