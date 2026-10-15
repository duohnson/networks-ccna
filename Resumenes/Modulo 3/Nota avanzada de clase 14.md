# Nota Avanzada - Clase 14: Virtualización, Nube y Redes Definidas por Software

## Introducción

La virtualización separa recursos lógicos de la plataforma física. SDN permite organizar el control de la red mediante interfaces programables y una visión coordinada. Esta clase prepara los conceptos necesarios para automatización sin asumir que virtualización, nube y SDN sean sinónimos.

---

## 1. Virtualización de Servidores

```text
Aplicación A     Aplicación B
Sistema A        Sistema B
   VM A             VM B
       HIPERVISOR
CPU / memoria / almacenamiento / interfaces físicas
```

| Concepto | Función |
|---|---|
| Host | Máquina física que aporta recursos |
| Hipervisor | Administra y aísla máquinas virtuales |
| Guest / VM | Sistema virtual con recursos asignados |
| vNIC | Interfaz de red virtual |
| vSwitch | Conmutador que conecta interfaces virtuales y enlaces físicos |

### Tipos de Hipervisor

| Tipo | Ubicación | Idea |
|---|---|---|
| Tipo 1 | Sobre el hardware | Plataforma de virtualización dedicada |
| Tipo 2 | Sobre un sistema operativo anfitrión | Virtualización como aplicación del host |

Consolidar servidores mejora utilización, pero concentra dependencia de hardware, energía y almacenamiento. La disponibilidad requiere un diseño adicional, no se obtiene solo por crear VMs.

### Contenedores

Los contenedores aíslan procesos y normalmente comparten el kernel del host; las VMs ejecutan su propio sistema operativo invitado. Un contenedor no es una VM de menor tamaño con exactamente el mismo modelo de aislamiento.

---

## 2. Computación en la Nube

| Modelo de servicio | Qué recibe el usuario | Ejemplo conceptual |
|---|---|---|
| IaaS | Cómputo, red y almacenamiento virtualizados | VM donde instala su sistema y aplicación |
| PaaS | Plataforma administrada para ejecutar aplicaciones | Entorno de despliegue de una aplicación |
| SaaS | Aplicación lista para utilizar | Servicio de correo o colaboración |

| Modelo de despliegue | Descripción |
|---|---|
| Pública | Servicios de un proveedor para distintos clientes |
| Privada | Entorno dedicado a una organización |
| Híbrida | Integración de entornos privados y públicos |
| Comunitaria | Entorno compartido por organizaciones con necesidades comunes |

La nube suele ofrecer autoservicio, acceso por red, recursos compartidos, elasticidad y medición de uso. Tener una VM local no implica disponer de todas esas características.

La responsabilidad se reparte entre proveedor y cliente: incluso con SaaS, el cliente debe administrar identidades, permisos y uso de la información según el servicio contratado.

---

## 3. Virtualización de Red

| Técnica | Qué separa o abstrae |
|---|---|
| VLAN | Dominios lógicos de capa 2 |
| VRF | Tablas de enrutamiento independientes en un mismo dispositivo |
| Red virtual | Conectividad lógica para recursos virtualizados |
| NFV | Funciones de red implementadas en software, como un router o firewall virtual |

**NFV** se centra en cómo se implementa una función; **SDN** se centra en cómo se organiza y programa el control. Pueden utilizarse juntos o por separado.

---

## 4. Planos de una Red

| Plano | Responsabilidad | Ejemplo |
|---|---|---|
| Datos | Reenviar o descartar paquetes | Consulta de tabla de forwarding |
| Control | Construir información para decidir el camino | OSPF calcula rutas |
| Gestión | Configurar y observar equipos | SSH, SNMP, API |

En redes tradicionales gran parte del control reside en cada equipo. Una arquitectura basada en controlador coordina decisiones y políticas mediante un sistema de control lógicamente centralizado. Este puede ejecutarse como un conjunto redundante de nodos.

---

## 5. Arquitectura SDN y APIs

```text
Aplicaciones / automatización / políticas
                    |
         API NORTHBOUND (hacia arriba)
                    |
              CONTROLADOR
                    |
         API SOUTHBOUND (hacia abajo)
                    |
          Routers / switches / APs
```

| Interfaz | Relación |
|---|---|
| Northbound | Aplicaciones con el controlador |
| Southbound | Controlador con dispositivos o infraestructura |

REST es frecuente en northbound. NETCONF, RESTCONF y otros mecanismos pueden usarse hacia dispositivos según el producto. OpenFlow es un ejemplo de protocolo southbound, no una obligación de toda red SDN.

Una API de controlador puede expresar una política para varios equipos; una API de dispositivo puede modificar un recurso local. Una respuesta exitosa del controlador debe complementarse con verificación del despliegue.

---

## 6. Underlay, Overlay y Fabric

| Término | Función |
|---|---|
| Underlay | Red de transporte que permite alcanzar los extremos |
| Overlay | Red lógica construida mediante encapsulación sobre el transporte |
| Fabric | Conjunto coordinado de infraestructura y mecanismos de control |

```text
Overlay:      Red lógica A ================= Red lógica A
                         encapsulación
Underlay:     Equipo ----- Routers IP ------ Equipo
```

Un overlay no elimina la necesidad de routing, capacidad y MTU en el underlay. Si los extremos no se alcanzan, la red lógica también falla.

### Spine-Leaf

En un diseño spine-leaf típico, cada leaf se conecta a cada spine y los servidores se conectan a los leaf. Favorece caminos predecibles para tráfico entre servidores (**east-west**). El tráfico hacia usuarios o redes externas suele describirse como **north-south**. No confundir estas direcciones con northbound/southbound de APIs.

---

## 7. Gestión Basada en Controlador e Intención

Un sistema como Cisco Catalyst Center, conocido en materiales anteriores como DNA Center, puede integrar inventario, aprovisionamiento, políticas y observación. Sus APIs permiten que aplicaciones consuman esas funciones.

El enfoque basado en intención puede representarse como:

1. **Expresar el resultado:** por ejemplo, separar usuarios invitados de servidores internos.
2. **Traducirlo a políticas y configuración.**
3. **Activar la política en la infraestructura.**
4. **Observar y comprobar que el comportamiento coincide con lo esperado.**

No significa que cualquier controlador interprete cualquier frase ni que toda corrección deba aplicarse automáticamente. [Cisco: APIs northbound de DNA Center](https://www.cisco.com/c/en/us/td/docs/cloud-systems-management/network-automation-and-management/dna-center-platform/2-3-5/user-guide/b-dnac-platform-ug-2-3-5.pdf).

### Gestión desde la Nube

Un plano de gestión alojado en la nube permite administrar sitios sin ubicar todo el software de gestión en cada sede. Eso no significa que todos los paquetes de usuarios deban atravesar la nube de gestión; depende de la arquitectura del servicio.

---

## 8. Ejercicio de Clasificación

| Situación | Concepto |
|---|---|
| Ejecutar dos sistemas operativos en un servidor | Virtualización mediante VMs |
| Mantener dos tablas de rutas separadas | VRF |
| Aplicación consulta inventario al controlador | API northbound |
| Controlador configura interfaces del router | Comunicación southbound |
| Firewall ejecutado como software virtual | NFV |
| Túnel que depende de conectividad IP previa | Overlay sobre underlay |

---

## Resumen Ejecutivo

- VM, contenedor y servicio cloud tienen modelos distintos.
- IaaS, PaaS y SaaS cambian el nivel de servicio y responsabilidades.
- Separar planos de datos, control y gestión permite entender SDN.
- Northbound conecta aplicaciones; southbound conecta infraestructura.
- Underlay transporta el overlay; ambos necesitan verificación.

## Referencias

- [Cisco NetAcad: ENSA, virtualización de red](https://www.netacad.com/courses/ccna-enterprise-networking-security-automation?courseLang=en-US).
- [Cisco: plataforma y APIs de DNA Center](https://www.cisco.com/c/en/us/td/docs/cloud-systems-management/network-automation-and-management/dna-center-platform/2-3-5/user-guide/b-dnac-platform-ug-2-3-5.pdf).
- [Cisco DevNet: NETCONF y RESTCONF](https://developer.cisco.com/docs/cisco-netconf-and-rest-device-management-api-documentation/).
