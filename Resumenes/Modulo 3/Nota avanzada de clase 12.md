# Nota Avanzada - Clase 12: Gestión de Red, NTP, SNMP, Syslog y Archivos IOS

## Introducción

Administrar una red implica conocer sus equipos, observar su estado, correlacionar eventos y recuperar configuraciones. Esta clase reúne el material de las guías PDF del repositorio y completa SNMP, Syslog y mantenimiento del sistema para el repaso ENSA.

---

## 1. CDP y LLDP: Vecinos Directos

| Protocolo | Característica | Verificación |
|---|---|---|
| CDP | Descubrimiento asociado a Cisco | `show cdp neighbors detail` |
| LLDP | Estándar IEEE 802.1AB | `show lldp neighbors detail` |

Ambos intercambian información en capa 2. Permiten conocer nombre, puerto local, puerto remoto y capacidades anunciadas, pero no demuestran conectividad IP de extremo a extremo.

```text
! Configuración global
cdp run
lldp run
interface fastEthernet 0/0
 no cdp enable
 lldp receive
 no lldp transmit
 exit
```

El ejemplo deshabilita CDP en un puerto y deja LLDP solo en recepción. `no cdp run` deshabilitaría CDP globalmente; no es requisito para activar LLDP.

---

## 2. NTP: Tiempo Consistente

**NTP** sincroniza relojes a través de la red y utiliza UDP 123. La jerarquía de stratum expresa distancia respecto de una fuente de referencia, no zona horaria. Un equipo de stratum 1 se sincroniza directamente con una referencia de stratum 0; stratum 16 representa falta de sincronización.

Ejemplo con servidor de laboratorio `192.168.100.10`:

```text
ntp server 192.168.100.10
service timestamps log datetime msec
```

Desde modo privilegiado:

```text
show clock
show ntp associations
show ntp status
```

No asumir sincronización inmediata. Verificar servidor alcanzable, servicio activo, filtros y asociación. La zona horaria de presentación no cambia el objetivo de sincronizar una referencia de tiempo común.

---

## 3. SNMP: Monitoreo mediante Objetos

```text
Gestor NMS ---- GET / consulta ----> Agente en router o switch
Gestor NMS <--- Respuesta ---------- Agente
Gestor NMS <--- Trap / Inform ------ Agente
```

| Elemento | Función |
|---|---|
| Manager / NMS | Sistema que consulta y presenta información |
| Agent | Servicio del dispositivo administrado |
| MIB | Definiciones de objetos administrables |
| OID | Identificador de un objeto |
| GET | Consulta un valor |
| SET | Solicita cambiar un valor cuando está permitido |
| Trap | Notificación sin confirmación propia de recepción |
| Inform | Notificación con confirmación |

SNMP utiliza habitualmente UDP 161 para consultas al agente y UDP 162 para notificaciones al gestor. [Cisco: soporte SNMP](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/snmp/configuration/15-mt/snmp-15-mt-book/nm-snmp-cfg-snmp-support.html).

### Versiones y Seguridad

| Versión | Característica |
|---|---|
| SNMPv1 | Modelo original basado en comunidades |
| SNMPv2c | Comunidades y mejoras como operaciones masivas; no aporta cifrado |
| SNMPv3 | Seguridad basada en usuarios y niveles de autenticación/privacidad |

Niveles SNMPv3: **noAuthNoPriv**, **authNoPriv** y **authPriv**. Que un dispositivo use v3 no demuestra que esté cifrando: depende del nivel configurado.

Ejemplo conceptual de monitoreo: consultar contadores de octetos de una interfaz en dos momentos y calcular su diferencia por unidad de tiempo. Detectar reinicios y reinicio de contadores antes de interpretar la tasa.

---

## 4. Syslog: Registro de Eventos

| Nivel | Nombre | Significado |
|---|---|---|
| 0 | Emergencies | Sistema inutilizable |
| 1 | Alerts | Acción inmediata |
| 2 | Critical | Condición crítica |
| 3 | Errors | Error |
| 4 | Warnings | Advertencia |
| 5 | Notifications | Evento normal significativo |
| 6 | Informational | Información |
| 7 | Debugging | Depuración |

**Menor número = mayor gravedad.** Un umbral 4 incluye niveles 0 a 4. [Cisco: niveles Syslog](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/esm/command/esm-cr-book/esm-cr-a1.html).

```text
logging host 192.168.100.10
logging trap warnings
logging buffered 16384 warnings
```

Revisar con `show logging`. En una sesión remota, `terminal monitor` permite mostrar mensajes de monitoreo cuando corresponda. `logging trap` selecciona severidad para el envío Syslog; no significa que se esté configurando una notificación SNMP trap.

Formato habitual IOS:

```text
%LINK-3-UPDOWN: Interface Serial0/0/0, changed state to down
```

`LINK` identifica la facilidad, `3` la severidad y `UPDOWN` el identificador del mensaje. Correlacionar la hora con otros equipos antes de concluir que varios eventos pertenecen a una misma causa.

---

## 5. Configuración y Almacenamiento

| Ubicación | Uso habitual en los equipos de las prácticas |
|---|---|
| RAM / running-config | Configuración activa y estructuras operativas |
| NVRAM / startup-config | Configuración de arranque |
| Flash | Imagen o paquetes del sistema y archivos |
| Servidor remoto | Copias externas |

```text
show running-config
show startup-config
show file systems
dir flash:
show version
copy running-config startup-config
```

Una configuración que funciona pero no se guarda puede perderse después del reinicio. Guardar no crea automáticamente una copia externa independiente.

---

## 6. TFTP y Copias de Seguridad

Con servidor TFTP habilitado en `192.168.100.10`, desde modo privilegiado:

```text
Router# copy running-config tftp:
Address or name of remote host []? 192.168.100.10
Destination filename [router-confg]? R1-respaldo.cfg
```

Para cargar una configuración:

```text
Router# copy tftp: running-config
Address or name of remote host []? 192.168.100.10
Source filename []? R1-respaldo.cfg
Destination filename [running-config]?
```

Copiar a `running-config` combina comandos con la configuración existente; no garantiza borrar las líneas que no aparezcan en el archivo. Comprobar el resultado antes de guardar.

TFTP no autentica ni cifra. La solicitud inicial utiliza UDP 69; el resto de la transferencia utiliza puertos negociados. Las reglas de filtrado deben considerar el flujo completo.

---

## 7. Mantenimiento de la Imagen del Sistema

Antes de sustituir una imagen IOS: identificar modelo y versión, confirmar compatibilidad, revisar almacenamiento, respaldar configuración, verificar integridad de la descarga y planificar recuperación. IOS clásico e IOS XE pueden usar mecanismos de arranque diferentes; no existe una secuencia universal de actualización para todos los equipos.

**Práctica de lectura:** consultar `show version` y `dir flash:` y registrar la versión, nombre del archivo, espacio disponible y configuración de arranque visible. No es necesario modificar la imagen para estudiar estos conceptos.

---

## 8. Comparativa de Herramientas

| Necesidad | Herramienta |
|---|---|
| Saber qué equipo está conectado | CDP / LLDP |
| Comparar eventos por hora | NTP |
| Graficar utilización y consultar contadores | SNMP |
| Recibir mensajes de cambios y errores | Syslog |
| Guardar y recuperar archivos | Copias locales y transferencia de archivos |

---

## Resumen Ejecutivo

- Descubrir vecinos no equivale a probar servicios de extremo a extremo.
- NTP permite correlacionar los registros de la red.
- SNMP consulta objetos; Syslog comunica eventos.
- SNMPv3 tiene distintos niveles de protección y debe verificarse el elegido.
- Respaldar configuración activa, guardada y archivos del sistema según su función.

## Referencias

- [Guía de configuraciones del repositorio](../../Documentos/CCNA-Configuraciones.pdf).
- [Guía de comandos show](../../Documentos/CCNA-Comandos.pdf).
- Cisco: [SNMP](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/snmp/configuration/15-mt/snmp-15-mt-book/nm-snmp-cfg-snmp-support.html) y [Syslog](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/esm/command/esm-cr-book/esm-cr-a1.html).
