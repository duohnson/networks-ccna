# Nota Avanzada - Clase 9: Seguridad de Redes, Amenazas y Defensa en Profundidad

## Introducción

Esta clase amplía la seguridad y las ACLs de las clases 3 y 4 con amenazas, protección de dispositivos, AAA y controles de infraestructura. Es material complementario de estudio CCNA/ENSA; no se presenta como una transcripción de una clase recibida.

Una ACL resuelve una parte del problema: decidir qué paquetes pueden pasar. La seguridad también requiere proteger identidades, equipos finales, servicios y disponibilidad.

---

## 1. Amenaza, Vulnerabilidad y Riesgo

| Concepto | Significado | Ejemplo |
|---|---|---|
| Activo | Recurso que necesita protección | Configuración de HQ o servidor DNS |
| Amenaza | Causa potencial de daño | Robo de credenciales |
| Vulnerabilidad | Debilidad aprovechable | Contraseña compartida o servicio innecesario |
| Exploit | Medio para aprovechar una vulnerabilidad | Código que provoca ejecución no autorizada |
| Riesgo | Probabilidad e impacto de un incidente | Pérdida de acceso a todas las sucursales |
| Mitigación | Control que reduce probabilidad o impacto | MFA, segmentación, parches y respaldos |

La tríada **CIA** representa confidencialidad, integridad y disponibilidad. La autenticación verifica identidad; no sustituye a disponibilidad como tercer componente de CIA.

---

## 2. Categorías de Ataques

| Categoría | Ejemplo | Control relacionado |
|---|---|---|
| Reconocimiento | Enumeración de servicios y equipos | Limitar servicios expuestos y monitorear |
| Acceso | Uso de credenciales robadas | AAA, MFA y mínimo privilegio |
| Denegación de servicio | Saturación de enlace o recursos | Filtrado, límites y coordinación con proveedor |
| Ingeniería social | Mensaje que induce a revelar claves | Verificación de identidad y capacitación |
| Malware | Troyano, ransomware o gusano | Protección de endpoint, parches y copias recuperables |
| Suplantación | Dirección IP, MAC o servidor falso | Controles de origen y autenticación |

**DoS** puede originarse en una fuente; **DDoS** utiliza múltiples fuentes. Bloquear paquetes en HQ no recupera por sí solo un enlace del ISP que ya está saturado.

### 2.1 Ataques a Servicios

- **TCP SYN flood:** consume recursos con intentos de conexión; debe distinguirse de tráfico legítimo intenso.
- **DNS spoofing:** induce resoluciones falsas; comprobar fuentes de DNS y mecanismos de validación.
- **DHCP falso:** entrega gateway o DNS controlados por un tercero; DHCP Snooping ayuda en el acceso LAN.
- **ARP spoofing:** altera asociaciones IP–MAC; Dynamic ARP Inspection puede validar mensajes con información confiable.
- **VLAN hopping:** intenta cruzar segmentación de capa 2; configurar acceso y troncales de forma explícita.

Los controles de capa 2 se desarrollan en las avanzadas 8 y 9 del módulo 2.

---

## 3. Defensa en Profundidad

```text
Internet --> Borde / firewall --> Segmentos internos --> Servidor / endpoint
                   |                     |                    |
                Filtrado             Segmentación        Identidad y parches
                   +---------------- Registros ----------------+
```

| Control | Función | Límite |
|---|---|---|
| ACL | Filtra campos de paquetes | No equivale a inspección completa de sesiones |
| Firewall con estado | Sigue el estado de las conexiones | Necesita políticas y capacidad adecuadas |
| IDS | Detecta y alerta | Normalmente no está en la ruta de bloqueo |
| IPS | Detecta y puede bloquear en línea | Un falso positivo puede afectar tráfico válido |
| VPN | Protege comunicaciones sobre redes de tránsito | No vuelve confiable a un equipo comprometido |
| Protección de endpoint | Controla amenazas en el host | Complementa al perímetro |

La segmentación limita el movimiento entre zonas. Un servidor público puede ubicarse en una **DMZ** con reglas específicas hacia Internet y hacia las redes internas.

---

## 4. AAA: Identidad, Permisos y Registro

| Componente | Pregunta |
|---|---|
| Authentication | ¿Quién eres? |
| Authorization | ¿Qué puedes hacer? |
| Accounting | ¿Qué hiciste y cuándo? |

| Tecnología | Uso y característica |
|---|---|
| Base local | Usuarios almacenados en cada equipo |
| RADIUS | Acceso de usuarios a la red; común en 802.1X y WLAN empresarial |
| TACACS+ | Administración de dispositivos y autorización de comandos |
| 802.1X | Control de acceso basado en puerto: supplicant, authenticator y servidor de autenticación |

En RADIUS clásico se utilizan UDP 1812 para autenticación y 1813 para accounting. TACACS+ utiliza TCP 49. No confundir el servidor AAA con el switch que actúa como authenticator.

---

## 5. Proteger el Acceso Administrativo

Ejemplo de laboratorio con usuario local y SSH, desde configuración global:

```text
hostname R1
ip domain-name ccna.lab
username admin secret ClaveLab123
enable secret PrivilegioLab123
ip ssh version 2
ip access-list standard GESTION
 permit 192.168.100.0 0.0.0.255
 deny any
 exit
line vty 0 4
 login local
 transport input ssh
 access-class GESTION in
 exec-timeout 5 0
 exit
```

Generar previamente claves RSA con `crypto key generate rsa` y elegir un tamaño admitido por el equipo. Aplicar la política a todas las líneas VTY existentes. Las claves del ejemplo son exclusivamente didácticas.

`service password-encryption` en los ejercicios IOS clásicos no convierte todas las contraseñas en secretos criptográficamente fuertes. Tampoco cifra Telnet; el transporte seguro del ejemplo es SSH.

---

## 6. Cifrado, Hash e Identidad

| Mecanismo | Propósito |
|---|---|
| Cifrado simétrico | Proteger contenido con una clave compartida, por ejemplo AES |
| Criptografía asimétrica | Utilizar un par de claves para funciones como firmas o establecimiento de confianza |
| Hash | Obtener una huella del contenido; un hash sin clave no autentica al remitente |
| HMAC | Combinar clave y hash para integridad y autenticación de origen |
| Firma digital | Verificar integridad y origen con claves asimétricas |
| Certificado | Vincular una identidad con una clave pública mediante una firma |

Diffie-Hellman es un mecanismo de acuerdo de claves, no un cifrado de los datos de usuario. El cifrado de un túnel VPN se estudia en la clase 10.

---

## 7. Práctica y Verificación

1. Identificar la VLAN de administración y los equipos que necesitan acceso.
2. Verificar SSH desde un origen permitido y desde uno excluido.
3. Consultar `show access-lists`, `show ip ssh` y `show logging`.
4. Revisar puertos y servicios que no sean necesarios.
5. Probar un respaldo y documentar cómo se recuperaría la administración.

**Preguntas de repaso:** ¿NAT autentica usuarios? No. ¿IDS y IPS siempre hacen lo mismo? No: la posición en línea permite al IPS bloquear. ¿Un hash sin clave acredita identidad? No.

---

## Resumen Ejecutivo

- Distinguir amenaza, vulnerabilidad, riesgo y mitigación.
- Combinar segmentación, protección de identidad, controles de red y endpoint.
- AAA separa autenticación, autorización y registro.
- SSH protege el transporte administrativo; las ACLs restringen quién llega.
- Cifrado, hash y firma digital resuelven problemas diferentes.

## Referencias

- [Cisco NetAcad: ENSA, seguridad de red](https://www.netacad.com/courses/ccna-enterprise-networking-security-automation?courseLang=en-US).
- [Clase 3: fundamentos y ACLs](./Nota%20avanzada%20de%20clase%203.md).
- [Módulo 2: seguridad LAN](../Modulo%202/Nota%20avanzada%20de%20clase%208.md).
- [Módulo 2: seguridad de switches](../Modulo%202/Nota%20avanzada%20de%20clase%209.md).
