# Nota Avanzada - Clase 13: Diseño de Redes y Resolución de Problemas

## Introducción

Esta clase reúne diseño y diagnóstico de ENSA. El diseño establece cómo debe funcionar la red; la documentación y las mediciones permiten comparar ese estado esperado con lo que realmente ocurre.

Se utilizan las sucursales del caso de estudio como referencia, sin confundir su topología de laboratorio con una arquitectura obligatoria para todas las empresas.

---

## 1. Diseño Jerárquico

```text
                 NÚCLEO
              /          \
      DISTRIBUCIÓN     DISTRIBUCIÓN
        /     \          /     \
     ACCESO ACCESO    ACCESO ACCESO
        |     |          |     |
       Usuarios y dispositivos finales
```

| Capa | Responsabilidad |
|---|---|
| Acceso | Conectar usuarios, teléfonos, APs y aplicar controles del puerto |
| Distribución | Agregar acceso, aplicar políticas y conectar dominios de enrutamiento |
| Núcleo | Transportar tráfico entre bloques con alta disponibilidad |

En un diseño de **dos capas**, núcleo y distribución pueden combinarse como núcleo colapsado. Esto no elimina la necesidad de redundancia y dimensionamiento.

---

## 2. Escalabilidad y Resiliencia

| Técnica | Propósito | Dependencia |
|---|---|---|
| VLANs y subredes | Segmentar usuarios y dominios de difusión | Plan de direccionamiento |
| Sumarización | Reducir anuncios y simplificar rutas | Bloques contiguos y diseño jerárquico |
| EtherChannel | Agrupar enlaces físicos | Configuración compatible de extremos |
| STP / RSTP | Evitar bucles de capa 2 | Elección de raíz y puertos |
| FHRP | Redundancia del gateway | Routers y segmento compatibles |
| Rutas alternativas | Continuidad ante fallo de enlaces | Enrutamiento y retorno correctos |

Dos enlaces que pasan por el mismo equipo, energía o ducto pueden compartir un punto de fallo. La redundancia se evalúa por los fallos que permite soportar, no solo por contar cables.

### Capacidad y Hardware

Comparar densidad y velocidad de puertos, rendimiento de forwarding, capacidad de tablas, PoE, módulos, fuentes y compatibilidad de funciones. Un equipo modular facilita ampliaciones; uno fijo puede simplificar costo y despliegue. La tasa física de los puertos no representa por sí sola el rendimiento total bajo todas las funciones activas.

---

## 3. Documentación y Línea Base

| Documento | Información |
|---|---|
| Topología física | Cableado, puertos, ubicaciones y equipos |
| Topología lógica | VLANs, subredes, áreas y dominios de routing |
| Inventario | Modelo, versión, identificadores y función |
| Tabla de direcciones | IP, máscara, gateway y DNS |
| Configuración de referencia | Estado aprobado y respaldo |
| Línea base | Utilización, latencia, errores y disponibilidad habituales |

Una medición de CPU o latencia tiene más significado si se compara con su línea base y el horario de operación.

---

## 4. Proceso de Diagnóstico

1. **Definir el problema:** qué falla, desde dónde, hacia dónde y desde cuándo.
2. **Recopilar evidencia:** mensajes, alcance, cambios recientes y mediciones.
3. **Formular hipótesis:** causas compatibles con la evidencia.
4. **Probar una hipótesis:** cambiar una variable o hacer una prueba controlada.
5. **Aplicar una corrección:** con posibilidad de volver al estado previo.
6. **Verificar:** servicio afectado y servicios relacionados.
7. **Documentar:** causa, cambio y resultado.

| Enfoque | Aplicación |
|---|---|
| Bottom-up | Empezar por medio físico e interfaces |
| Top-down | Empezar por aplicación y bajar capas |
| Divide and conquer | Probar una capa intermedia, por ejemplo alcance IP |
| Seguir el camino | Revisar cada salto de ida y retorno |
| Comparación | Contrastar con un equipo o segmento que funciona |
| Sustitución | Probar un componente conocido como correcto |

---

## 5. Herramientas por Capa

| Capa | Comandos y pruebas | Qué buscar |
|---|---|---|
| Física | `show interfaces`, inspección de cable | Estado, errores, velocidad y dúplex |
| Enlace | `show vlan brief`, `show interfaces trunk`, `show spanning-tree` | VLAN y camino de capa 2 |
| Red | `show ip route`, `ping`, `traceroute` | Ruta elegida, gateway y saltos |
| Vecindad | `show ip ospf neighbor`, `show ip eigrp neighbors` | Estado de routing |
| Filtrado/NAT | `show access-lists`, `show ip nat translations` | Coincidencias y traducciones |
| Aplicación | DNS, HTTP, SSH desde el cliente | Servicio real y dependencias |

En PCs se utiliza `ipconfig` y, según el entorno, `tracert` o `nslookup`. Packet Tracer no reproduce todos los comandos de un sistema operativo real.

---

## 6. Diagnóstico IPv4 e IPv6

### IPv4

Revisar IP, máscara y gateway; luego ARP del siguiente salto y ruta al destino. Para destinos externos a la subred, la PC resuelve la MAC del gateway, no la MAC del servidor remoto.

### IPv6

Revisar prefijo, direcciones link-local, router predeterminado, Neighbor Discovery y rutas. IPv6 utiliza ICMPv6 para funciones esenciales; un filtro que bloquee todo ICMPv6 puede romper más que el ping.

```text
show ipv6 interface brief
show ipv6 neighbors
show ipv6 route
show ipv6 ospf neighbor
```

Un siguiente salto link-local debe interpretarse junto con su interfaz. El mismo texto `FE80::1` puede existir en enlaces diferentes.

---

## 7. Caso Práctico: “No Abre la Página”

```text
¿La PC tiene IP correcta?
  --> ¿Alcanza su gateway?
      --> ¿Existe ruta de ida y retorno?
          --> ¿Resuelve DNS?
              --> ¿Responde HTTP/HTTPS?
                  --> ¿Qué muestran ACL, NAT y registros?
```

| Evidencia | Hipótesis que ayuda a separar |
|---|---|
| Dirección incorrecta | DHCP, VLAN o configuración local |
| Gateway inaccesible | Enlace local, VLAN, SVI/subinterfaz o filtro |
| Funciona por IP y falla por nombre | DNS o dependencia de nombres |
| NAT tiene entrada sin respuesta | Retorno, filtro exterior o servicio |
| Aumenta contador deny | La política coincide con ese tráfico |
| Ping falla pero HTTP funciona | ICMP puede estar filtrado; no demuestra caída del servidor |

`traceroute` con asteriscos tampoco prueba por sí solo que el camino esté interrumpido: los mensajes de diagnóstico pueden ser filtrados mientras el tráfico de aplicación pasa.

---

## 8. Práctica de Fallo Controlado

En una copia de la topología, cambiar la VLAN de un solo puerto de PC y observar IP, alcance al gateway y tabla de VLANs. Restaurar el valor original y comprobar el servicio. Repetir con una ruta incorrecta, sin introducir ambos fallos a la vez.

Registrar: síntoma, hipótesis, comando usado, evidencia, corrección y prueba final. Este registro sirve como historial y como material de repaso.

---

## Resumen Ejecutivo

- Diseñar por funciones y requisitos de capacidad y disponibilidad.
- Documentar estado esperado y línea base antes de diagnosticar.
- Formular hipótesis y probar cambios controlados.
- Revisar ida y retorno y utilizar el servicio real, no solo ping.
- ICMPv6 y Neighbor Discovery requieren atención especial en IPv6.

## Referencias

- [Cisco NetAcad: ENSA, diseño y troubleshooting](https://www.netacad.com/courses/ccna-enterprise-networking-security-automation?courseLang=en-US).
- [Caso de estudio del módulo 3](./Nota%20avanzada%20de%20caso%20de%20estudio.md).
- [Guía de comandos del repositorio](../../Documentos/CCNA-Comandos.pdf).
