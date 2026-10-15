# Nota Avanzada - Clase 5: NAT Estática, NAT Dinámica y PAT

## Introducción

**NAT (Network Address Translation)** modifica direcciones IP al atravesar el router de borde. En los ejercicios del módulo permite que redes internas accedan a la red del ISP y que un servidor interno tenga una dirección global estable.

Esta nota desarrolla la clase 5 y la topología izquierda de la **Tarea 5 NAT**. Mantiene sus direcciones para facilitar la comparación; son valores de la simulación. Los ejemplos de NAT dinámica y PAT son alternativas, no configuraciones para aplicar simultáneamente al mismo conjunto de clientes.

---

## 1. Direcciones Privadas y Direcciones Globales

| Bloque privado RFC 1918 | Máscara | Rango |
|---|---|---|
| `10.0.0.0/8` | `255.0.0.0` | `10.0.0.0` a `10.255.255.255` |
| `172.16.0.0/12` | `255.240.0.0` | `172.16.0.0` a `172.31.255.255` |
| `192.168.0.0/16` | `255.255.0.0` | `192.168.0.0` a `192.168.255.255` |

**No todo `172.x.x.x` es privado.** Por ejemplo, `172.168.0.0/24`, utilizado en la Tarea 7, queda fuera de RFC 1918. Las clases A/B/C son una referencia histórica; en estas configuraciones se trabaja con prefijos CIDR.

---

## 2. Los Cuatro Términos de NAT

| Término | Significado | Ejemplo de la tarea |
|---|---|---|
| **Inside local / Local interna** | Dirección del host interno vista desde la red interna | `172.16.0.254` |
| **Inside global / Global interna** | Dirección que representa al host interno hacia el exterior | `209.165.200.254` |
| **Outside global / Global externa** | Dirección del host externo en su propia red | `200.0.0.2` |
| **Outside local / Local externa** | Dirección con que se ve al host externo desde dentro | `200.0.0.2` si no se traduce el destino |

**Inside/outside** identifica el lado del host; **local/global** identifica la representación de la dirección. Local no significa obligatoriamente privada. En el ejemplo, outside local y outside global coinciden porque no se traduce al host externo. [Cisco: términos de NAT](https://www.cisco.com/c/en/us/support/docs/ip/network-address-translation-nat/4606-8.html).

```text
LANs 192.168.10.0/24 y 192.168.11.0/24
                   |
                  [R1] -- 10.0.0.0/30 -- [R2] -- 200.0.0.0/30 -- [R3/ISP]
                                         |
                                  172.16.0.0/24
                                 servidor .254

R2: Fa0/0 y S0/0/0 = INSIDE; S0/0/1 = OUTSIDE
```

---

## 3. Preparar Enrutamiento e Interfaces

En R2, con las IP y enlaces ya configurados como en la tarea:

```text
interface fastEthernet 0/0
 ip nat inside
 exit
interface serial 0/0/0
 ip nat inside
 exit
interface serial 0/0/1
 ip nat outside
 exit

ip route 0.0.0.0 0.0.0.0 200.0.0.2
router ospf 1
 network 10.0.0.0 0.0.0.3 area 0
 network 172.16.0.0 0.0.0.255 area 0
 passive-interface fastEthernet 0/0
 default-information originate
 exit
```

En R1 deben estar anunciadas ambas LANs y el enlace `10.0.0.0/30`. La ruta por defecto debe existir en R2 para que `default-information originate` la anuncie sin `always`.

**NAT no crea rutas.** R2 necesita rutas hacia las LANs, R1 necesita una salida y el ISP necesita el camino de regreso a las direcciones traducidas.

---

## 4. NAT Estática: Una Dirección por Host

En R2:

```text
ip nat inside source static 172.16.0.254 209.165.200.254
```

En R3/ISP:

```text
ip route 209.165.200.254 255.255.255.255 200.0.0.1
```

La asociación es permanente y puede utilizarse para acceder desde fuera al servidor, siempre que haya rutas, gateway correcto y ACLs que permitan el servicio. NAT estática por dirección no limita por sí sola el acceso a HTTP.

---

## 5. NAT Dinámica: Pool de Direcciones

### Paso 1: Crear el Pool

```text
ip nat pool CCNA 209.165.200.241 209.165.200.246 netmask 255.255.255.248
```

### Paso 2: Seleccionar las Redes Internas

```text
ip access-list standard NAT_PCS
 permit 192.168.10.0 0.0.0.255
 permit 192.168.11.0 0.0.0.255
 exit
```

### Paso 3: Relacionar ACL y Pool

```text
ip nat inside source list NAT_PCS pool CCNA
```

La ACL aquí **selecciona tráfico para traducción**. Una dirección no permitida no se traduce por esta regla; eso no equivale a descartarla mediante una ACL de interfaz. [Cisco: configuración NAT](https://www.cisco.com/c/en/us/support/docs/ip/network-address-translation-nat/13772-12.html).

### Paso 4: Ruta de Retorno en el ISP

```text
ip route 209.165.200.240 255.255.255.248 200.0.0.1
```

| Dato del bloque `/29` | Valor |
|---|---|
| Red | `209.165.200.240` |
| Primera utilizable | `209.165.200.241` |
| Última utilizable | `209.165.200.246` |
| Broadcast | `209.165.200.247` |
| Direcciones utilizables | 6 |

Sin sobrecarga, cada host traducido ocupa una dirección del pool mientras conserva su asociación. Si las seis están ocupadas, otro host debe esperar una liberación. El servidor estático `.254` queda fuera de este bloque y conserva su ruta `/32` independiente.

---

## 6. PAT: Sobrecarga de Direcciones

PAT permite compartir direcciones globales distinguiendo flujos por protocolo y puertos; en ICMP se utilizan identificadores.

### 6.1 PAT con Pool: Variante de la Tarea 5

```text
ip nat pool POOL_PCS 209.165.200.241 209.165.200.242 netmask 255.255.255.248
ip nat inside source list NAT_PCS pool POOL_PCS overload
```

El pool puede usar solo parte de las direcciones utilizables del `/29`. El ISP conserva la ruta al bloque `209.165.200.240/29`.

### 6.2 PAT con la Dirección de la Interfaz

```text
ip nat inside source list NAT_PCS interface serial 0/0/1 overload
```

Esta alternativa utiliza la IP de S0/0/1 de R2 (`200.0.0.1` en la tarea). No requiere pool; sí requiere `inside`, `outside`, ACL de selección y rutas.

| Modalidad | Asociación | Comando distintivo |
|---|---|---|
| Estática | Una local a una global fija | `static LOCAL GLOBAL` |
| Dinámica | Una global del pool por host traducido | `list ACL pool POOL` |
| PAT con pool | Muchos flujos comparten las globales del pool | `pool POOL overload` |
| PAT con interfaz | Muchos flujos comparten la IP de salida | `interface INTERFAZ overload` |

---

## 7. Cambiar o Eliminar una Regla

Al pasar del ejemplo dinámico CCNA a otra variante, retirar primero su asociación y después el pool:

```text
no ip nat inside source list NAT_PCS pool CCNA
no ip nat pool CCNA
```

Si IOS indica que hay traducciones activas, detener el tráfico del laboratorio y ejecutar desde modo privilegiado `clear ip nat translation *`, luego reintentar. Limpiar traducciones interrumpe las sesiones dinámicas; no sustituye la eliminación de una regla estática de configuración.

---

## 8. Verificación y Diagnóstico

| Comando | Utilidad |
|---|---|
| `show ip route` | Comprobar rutas internas y salida por defecto |
| `show ip nat translations` | Ver direcciones locales/globales y puertos o identificadores |
| `show ip nat statistics` | Revisar interfaces, pools y estadísticas |
| `show access-lists NAT_PCS` | Revisar la selección de las LANs |
| `debug ip nat` | Observar traducciones mientras se genera tráfico |
| `undebug all` | Terminar la depuración |

**Prueba:** generar tráfico desde una PC interna hacia el ISP y revisar la tabla en R2. Un ping generado por el propio router puede utilizar una dirección de origen distinta y no prueba necesariamente la regla de los clientes.

| Síntoma | Revisar primero |
|---|---|
| No aparecen traducciones dinámicas | Tráfico generado, ACL y marcas inside/outside |
| Hay traducción pero no respuesta | Ruta de retorno del ISP, ACLs y servicio destino |
| Unos hosts funcionan y otros no | Agotamiento del pool sin overload o LAN ausente en ACL |
| Se llega por IP pero no por nombre | Configuración y permiso de DNS |
| No se accede al servidor estático | Gateway del servidor, ruta `/32` y filtros de entrada |

---

## Resumen Ejecutivo

- NAT estática mantiene una asociación fija; NAT dinámica asigna direcciones de un pool.
- PAT agrega `overload` para compartir direcciones entre múltiples flujos.
- Marcar todas las interfaces internas por las que llegan clientes, incluidas subinterfaces cuando corresponda.
- Una ACL de NAT selecciona traducciones; una ACL aplicada a interfaz filtra paquetes.
- Verificar ida, traducción y retorno por separado.
- Mantener las direcciones estáticas fuera del pool dinámico usado por otros hosts.

## Referencias

- [Notas originales de clase 5](./NOTAS%20CLASE%205%20MOD%203.txt).
- [Notas de Tarea 5 NAT](../../Topolog%C3%ADas/Topolog%C3%ADas%20de%20Modulo%203/Notas%20de%20Tarea%205%20NAT.txt).
- Cisco: [términos NAT](https://www.cisco.com/c/en/us/support/docs/ip/network-address-translation-nat/4606-8.html) y [configuración NAT](https://www.cisco.com/c/en/us/support/docs/ip/network-address-translation-nat/13772-12.html).
