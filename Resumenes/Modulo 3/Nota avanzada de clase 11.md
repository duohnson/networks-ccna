# Nota Avanzada - Clase 11: Calidad de Servicio (QoS)

## Introducción

**QoS (Quality of Service)** permite dar tratamientos diferentes al tráfico según sus necesidades. Es especialmente útil cuando varias aplicaciones compiten por un enlace: una descarga puede esperar, pero una conversación pierde calidad si sus paquetes llegan demasiado tarde.

Esta clase completa el tema de QoS de ENSA con conceptos, comparaciones y un ejemplo didáctico. QoS administra capacidad existente; no crea ancho de banda.

---

## 1. Factores de Calidad

| Factor | Significado | Efecto |
|---|---|---|
| Ancho de banda | Capacidad del enlace | Limita la tasa disponible |
| Retardo | Tiempo de tránsito | Afecta interacción y respuesta |
| Jitter | Variación del retardo | Afecta reproducción de voz/video |
| Pérdida | Paquetes que no llegan | Interrupciones o retransmisiones |

El retardo incluye procesamiento, espera en colas, serialización y propagación. El tiempo de serialización de un paquete de L bits sobre un enlace de R bit/s es `L/R` segundos.

**Ejemplo:** 1500 bytes son 12000 bits; sobre 1 Mbit/s necesitan aproximadamente 12 ms de serialización, sin contar otros retardos.

---

## 2. Características del Tráfico

| Tráfico | Necesidad predominante |
|---|---|
| Voz interactiva | Poco retardo, jitter y pérdida |
| Video interactivo | Retardo controlado y capacidad suficiente |
| Aplicación transaccional | Respuestas consistentes |
| Transferencia de archivos | Rendimiento y entrega; tolera más espera |
| Gestión de red | Conservar visibilidad y control |

La clasificación debe reconocer el tráfico real. Marcar todo como prioritario elimina la diferenciación.

---

## 3. Modelos de QoS

| Modelo | Funcionamiento |
|---|---|
| Best Effort | Sin garantía diferenciada |
| IntServ | Reserva de recursos por flujo, asociada a señalización como RSVP |
| DiffServ | Agrupa tráfico en clases con comportamientos definidos por salto |

DiffServ permite que los dispositivos apliquen políticas de clase a partir de marcas. Una marca por sí sola no garantiza tratamiento: los equipos del recorrido deben reconocerla y aplicar políticas compatibles.

---

## 4. Clasificación, Marcado y Frontera de Confianza

```text
Paquete --> Clasificar --> Marcar --> Aplicar cola / límite --> Transmitir
```

| Campo | Dónde se utiliza | Tamaño |
|---|---|---|
| CoS / PCP | Etiqueta 802.1Q de capa 2 | 3 bits |
| DSCP | Campo DS de IPv4 o Traffic Class de IPv6 | 6 bits |
| ECN | Señalización explícita de congestión | 2 bits |

Ejemplos de marcas: **DSCP EF = 46**, usado habitualmente para voz; **AF41 = 34**; **CS0 = 0**, tratamiento por defecto. Los valores numéricos de DSCP no forman una escala universal donde todo número mayor implica mayor prioridad.

La **frontera de confianza** indica dónde se aceptan marcas de otros equipos. Un cliente no debe obtener prioridad simplemente porque marque todos sus paquetes como EF.

---

## 5. Algoritmos de Colas

| Método | Idea principal |
|---|---|
| FIFO | Atiende por orden de llegada |
| WFQ | Distribuye servicio entre flujos |
| CBWFQ | Define clases con garantías de ancho de banda durante congestión |
| LLQ | Agrega tratamiento prioritario para tráfico sensible al retardo |

Una clase sin tráfico puede dejar capacidad disponible para otras. La prioridad debe estar controlada para no dejar sin servicio a las demás clases. [Cisco: colas y QoS](https://www.cisco.com/c/en/us/support/docs/quality-of-service-qos/qos-policing/22833-qos-faq.html).

### Congestión y Descarte

**Tail drop** descarta cuando la cola se llena. Los mecanismos como **WRED** pueden comenzar a descartar antes de llenarse para señalar congestión a flujos que reaccionan a ella. No sustituyen la prioridad de voz ni garantizan resolver cualquier sobrecarga.

---

## 6. Policing y Shaping

| Técnica | Tratamiento del exceso | Consecuencia |
|---|---|---|
| Policing | Descarta o remarca según la política | Limita tráfico sin suavizarlo mediante una cola de shaping |
| Shaping | Retiene temporalmente y transmite a una tasa controlada | Suaviza ráfagas, pero agrega espera |

Un enlace físico de 1 Gbit/s puede tener un servicio contratado de 100 Mbit/s. El shaping de salida puede ajustar la tasa al servicio; no basta con que la interfaz física sea más rápida. [Cisco: policing frente a shaping](https://www.cisco.com/c/en/us/support/docs/quality-of-service-qos/qos-policing/19645-policevsshape.html).

---

## 7. Lectura de una Política IOS

Ejemplo para interpretar, en una plataforma que soporte MQC y con tráfico EF previamente clasificado de forma confiable:

```text
class-map match-any VOZ
 match dscp ef
 exit
policy-map SALIDA-WAN
 class VOZ
  priority percent 20
  exit
 class class-default
  bandwidth percent 50
  exit
 exit
interface gigabitEthernet 0/1
 service-policy output SALIDA-WAN
 exit
```

| Parte | Función |
|---|---|
| `class-map` | Define qué tráfico coincide |
| `policy-map` | Define el tratamiento de cada clase |
| `service-policy output` | Aplica la política en la salida indicada |

Los porcentajes dependen del ancho de banda de referencia de la interfaz/política. El soporte de reservas, límites y comandos varía según plataforma; este bloque no presupone compatibilidad con todos los modelos de Packet Tracer.

---

## 8. Verificación y Repaso

Usar `show policy-map interface`, donde esté disponible, para examinar coincidencias, tasas y descartes. Comparar una prueba sin congestión con otra donde voz y transferencia de archivos compiten por el enlace.

| Pregunta | Respuesta |
|---|---|
| ¿QoS aumenta la velocidad del enlace? | No |
| ¿Shaping puede aumentar el retardo? | Sí, porque almacena paquetes |
| ¿DSCP 46 es un puerto? | No, es una marca de calidad de servicio |
| ¿Marcar basta para garantizar extremo a extremo? | No, se necesitan políticas coherentes |
| ¿Priorizar todas las aplicaciones ayuda? | No permite distinguir sus necesidades |

---

## Resumen Ejecutivo

- Identificar ancho de banda, retardo, jitter y pérdida.
- Clasificar antes de aplicar una política y definir dónde se confían las marcas.
- CBWFQ organiza clases; LLQ atiende tráfico sensible al retardo con prioridad controlada.
- Policing limita mediante acciones sobre el exceso; shaping lo retiene temporalmente.
- Verificar la política bajo congestión y con contadores.

## Referencias

- Cisco: [fundamentos de colas QoS](https://www.cisco.com/c/en/us/support/docs/quality-of-service-qos/qos-policing/22833-qos-faq.html).
- Cisco: [policing y shaping](https://www.cisco.com/c/en/us/support/docs/quality-of-service-qos/qos-policing/19645-policevsshape.html).
