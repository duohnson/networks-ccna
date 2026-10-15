# Nota Avanzada - Clase 15: Automatización de Red, APIs, JSON e IA

## Introducción

La automatización aplica tareas repetibles mediante software: recopilar inventario, respaldar configuraciones, comprobar políticas o desplegar cambios. Esta clase desarrolla la automatización de ENSA y añade los conceptos de Ansible, Terraform e IA/ML del temario CCNA v1.1 consultado el 15 de octubre de 2026.

El objetivo es comprender el flujo de trabajo, interpretar datos y distinguir herramientas. Los ejemplos locales no modifican dispositivos y no requieren abrir Packet Tracer.

---

## 1. Gestión Manual y Automatización

| Aspecto | Gestión manual | Automatización |
|---|---|---|
| Ejecución | Operador repite acciones | Programa aplica un procedimiento |
| Consistencia | Depende de cada ejecución | Puede usar plantillas y validaciones |
| Escala | Más equipos requieren más trabajo repetitivo | Una ejecución puede abarcar muchos equipos |
| Error | Puede afectar una operación puntual | Un error puede propagarse a muchos equipos |
| Evidencia | Debe registrarse | Puede guardar resultados estructurados |

Automatizar una configuración incorrecta no la corrige. Primero definir resultado esperado, alcance, comprobaciones y recuperación.

```text
Inventario --> Estado actual --> Estado deseado --> Diferencias
                    --> Revisión --> Aplicación --> Verificación
```

### Conceptos Clave

- **Fuente de verdad:** registro autorizado del estado deseado, por ejemplo redes y VLANs aprobadas.
- **Plantilla:** estructura reutilizable con valores variables por equipo.
- **Idempotencia:** repetir una operación sobre el estado ya deseado no produce cambios adicionales innecesarios.
- **Drift:** diferencia entre el estado real y el esperado.
- **Imperativo:** describe pasos para lograr el resultado.
- **Declarativo:** describe el resultado que debe existir.

---

## 2. Formatos de Datos: JSON, YAML y XML

### 2.1 JSON

**JSON** representa objetos, arreglos, cadenas, números, booleanos y `null`. Las claves y cadenas usan comillas dobles; no admite comentarios ni comas finales en su forma estándar.

```json
{
  "hostname": "R1",
  "management_ip": "192.168.100.1",
  "enabled": true,
  "interfaces": [
    {"name": "GigabitEthernet0/0", "vlan": 10, "up": true},
    {"name": "GigabitEthernet0/1", "vlan": 20, "up": false}
  ],
  "description": null
}
```

| Expresión conceptual | Resultado |
|---|---|
| `hostname` | Cadena `R1` |
| `interfaces` | Arreglo con dos objetos |
| `interfaces[0].vlan` | Número 10 |
| `interfaces[1].up` | Booleano false |

`"10"` es una cadena y `10` un número. `false` no es igual a `"false"`.

### 2.2 YAML

```yaml
hostname: R1
management_ip: 192.168.100.1
interfaces:
  - name: GigabitEthernet0/0
    vlan: 10
    up: true
```

YAML utiliza indentación para expresar estructura y es frecuente en inventarios y playbooks. Usar espacios de forma consistente; no confundir facilidad de lectura con ausencia de reglas de tipos.

### 2.3 XML

```xml
<device>
  <hostname>R1</hostname>
  <management-ip>192.168.100.1</management-ip>
</device>
```

XML utiliza etiquetas y puede incluir namespaces. El formato de datos no define por sí solo el protocolo utilizado para transportarlo.

---

## 3. APIs y Arquitectura REST

Una **API** establece cómo un programa solicita funciones o datos a otro. En una API REST sobre HTTP se identifican recursos mediante URI y se opera sobre ellos con métodos y representaciones.

```text
Cliente --> Solicitud HTTP: método + URI + headers + cuerpo opcional
Servidor --> Respuesta HTTP: estado + headers + cuerpo opcional
```

Una arquitectura REST es **sin estado de sesión entre solicitudes** en el sentido de que cada petición aporta el contexto necesario para procesarla. Esto no significa que el servidor no almacene recursos o bases de datos.

| Método | Uso habitual | Relación CRUD |
|---|---|---|
| GET | Leer un recurso | Read |
| POST | Crear o ejecutar una operación definida por la API | Create, habitualmente |
| PUT | Crear o reemplazar el recurso identificado | Update / reemplazo |
| PATCH | Modificar parcialmente | Update parcial |
| DELETE | Eliminar el recurso | Delete |

GET es de lectura; PUT y DELETE tienen semántica idempotente. Eso no exige que todos los intentos devuelvan exactamente el mismo código de respuesta. POST no es necesariamente idempotente; PATCH depende de la operación aplicada.

### 3.1 Ejemplo HTTP Didáctico

```http
GET /api/v1/devices/R1 HTTP/1.1
Host: controlador.example
Accept: application/json
Authorization: Bearer TOKEN_DE_EJEMPLO
```

La ruta es ficticia y solo ilustra la estructura. La documentación del producto define su endpoint real, autenticación y campos.

```http
HTTP/1.1 200 OK
Content-Type: application/json

{"hostname":"R1","reachable":true}
```

`Accept` indica qué formato acepta el cliente. `Content-Type` identifica el formato del cuerpo enviado. HTTPS agrega protección del transporte; un token no reemplaza esa protección.

### 3.2 Códigos de Respuesta

| Código | Interpretación habitual |
|---|---|
| 200 | Solicitud completada correctamente |
| 201 | Recurso creado |
| 202 | Aceptada para procesamiento; puede seguir pendiente |
| 204 | Éxito sin cuerpo de respuesta |
| 400 | Solicitud inválida |
| 401 | Faltan credenciales válidas |
| 403 | Operación no permitida |
| 404 | Recurso no encontrado |
| 429 | Límite de solicitudes excedido |
| 500 | Error del servidor |

Si una API devuelve 202 y un identificador de tarea, consultar el estado de esa tarea antes de afirmar que la configuración quedó aplicada. Considerar paginación cuando una consulta de inventario devuelve solo parte de los equipos.

---

## 4. NETCONF, RESTCONF y YANG

| Elemento | Propósito |
|---|---|
| YANG | Define el modelo de datos, estructura y restricciones |
| NETCONF | Protocolo de gestión mediante operaciones RPC y datos XML |
| RESTCONF | Interfaz basada en HTTP para datos modelados con YANG |

NETCONF se utiliza habitualmente sobre SSH en TCP 830. RESTCONF suele utilizar HTTPS en TCP 443 y representaciones XML o JSON. Los modelos y operaciones soportados deben consultarse en el equipo. [Cisco DevNet: interfaces modeladas](https://developer.cisco.com/docs/cisco-netconf-and-rest-device-management-api-documentation/).

No confundir **YANG** con **YAML**: YANG define modelos; YAML es un formato de serialización utilizado por herramientas como Ansible.

---

## 5. Herramientas de Gestión de Configuración

| Herramienta | Característica para el repaso |
|---|---|
| Ansible | Automatización sin agente instalado en cada equipo gestionado; usa módulos, inventarios y playbooks |
| Terraform | Infraestructura como código mediante recursos, providers, planificación y estado |
| Puppet | Modelo declarativo de configuración, habitualmente con agentes y un servicio central |
| Chef | Configuración mediante recetas/cookbooks, habitualmente con ejecución de clientes |

Puppet y Chef se incluyen como contexto de materiales ENSA anteriores; Ansible y Terraform son relevantes para el temario CCNA v1.1.

### 5.1 Ansible

Un **inventario** identifica equipos y grupos; un **playbook** define tareas. Las conexiones pueden utilizar SSH, NETCONF o APIs según la colección y el dispositivo.

Ejemplo de lectura para equipos IOS con la colección `cisco.ios` y conexión previamente configuradas:

```yaml
- name: Consultar interfaces de routers IOS
  hosts: routers
  gather_facts: false
  tasks:
    - name: Leer resumen de interfaces
      cisco.ios.ios_command:
        commands:
          - show ip interface brief
      register: interfaces
    - name: Mostrar resultado
      ansible.builtin.debug:
        var: interfaces.stdout_lines
```

La arquitectura es agentless, pero la máquina de control sí necesita Ansible, las colecciones y acceso autorizado. La idempotencia depende del módulo y de cómo se construye la tarea; enviar comandos arbitrarios no la garantiza. [Documentación de Ansible](https://docs.ansible.com/projects/ansible/latest/getting_started/introduction.html).

### 5.2 Terraform

```text
Configuración --> terraform plan --> Cambios propuestos --> terraform apply
                         ^                                  |
                         +------------- Estado -------------+
```

Un provider conecta Terraform con la plataforma. El estado relaciona recursos declarados con objetos gestionados; puede contener información sensible y debe protegerse. `plan` ayuda a revisar diferencias, pero no sustituye las pruebas posteriores. [HashiCorp: flujo de trabajo](https://developer.hashicorp.com/terraform/intro/core-workflow) y [estado](https://developer.hashicorp.com/terraform/language/state).

---

## 6. IA y Machine Learning en Operaciones de Red

| Concepto | Ejemplo |
|---|---|
| IA | Sistemas que ejecutan tareas de análisis, inferencia o generación |
| Machine Learning | Modelos que aprenden patrones de datos |
| IA predictiva | Estimar saturación o detectar comportamiento anómalo |
| IA generativa | Producir explicaciones, consultas o borradores de configuración |

**Ejemplo predictivo:** comparar métricas históricas con tráfico actual para señalar una desviación. **Ejemplo generativo:** resumir eventos Syslog o proponer una hipótesis de fallo.

Una anomalía no demuestra un ataque; puede representar una copia de seguridad programada. Una configuración generada puede incluir sintaxis inexistente o supuestos incorrectos. Validar contra inventario, documentación, políticas y pruebas antes de aplicarla. Cisco incluye IA generativa, predictiva y ML en el enfoque del CCNA v1.1. [Cisco: actualización CCNA](https://blogs.cisco.com/learning/understanding-the-updated-ccna-v1-1-with-ai-machine-learning-and-more).

---

## 7. Ejercicio Local: Interpretar una Respuesta JSON

Este código Python usa datos incluidos en el ejemplo y solo imprime interfaces caídas:

```python
import json

respuesta = '''{
  "hostname": "R1",
  "interfaces": [
    {"name": "GigabitEthernet0/0", "up": true},
    {"name": "GigabitEthernet0/1", "up": false}
  ]
}'''

equipo = json.loads(respuesta)
for interfaz in equipo["interfaces"]:
    if not interfaz["up"]:
        print(f'{equipo["hostname"]}: {interfaz["name"]} está caída')
```

Resultado esperado:

```text
R1: GigabitEthernet0/1 está caída
```

En una respuesta real, validar esquema, claves, tipos y errores antes de usar los datos para tomar decisiones. Un estado caído puede ser intencional; compararlo con el estado deseado.

---

## 8. Flujo de Automatización Verificable

1. Seleccionar los equipos y el cambio o consulta exactos.
2. Obtener inventario y estado actual, con credenciales protegidas.
3. Validar datos de entrada y construir la diferencia propuesta.
4. Probar con un equipo o entorno de laboratorio.
5. Aplicar dentro del alcance definido y registrar cada resultado.
6. Comprobar servicios y estado operativo, no solo éxito de la API.
7. Recuperar el estado previo si la verificación falla y documentar.

No guardar tokens o contraseñas dentro del repositorio. Los ejemplos usan datos ficticios y no configuran acceso a equipos reales.

---

## Resumen Ejecutivo

- Automatizar permite repetir procedimientos y verificar consistencia a escala.
- JSON, YAML y XML representan datos; una API define operaciones y contrato.
- Interpretar métodos HTTP, códigos, autenticación y tareas asíncronas.
- YANG modela; NETCONF y RESTCONF transportan operaciones de gestión.
- Ansible ejecuta tareas mediante módulos; Terraform administra recursos declarados y estado.
- IA/ML aporta análisis y generación, con validación de resultados.

## Referencias

- [Cisco: temario CCNA](https://learningnetwork.cisco.com/s/ccna-exam-topics).
- [RFC 8259: formato JSON](https://www.rfc-editor.org/info/rfc8259/).
- [Cisco DevNet: NETCONF y RESTCONF](https://developer.cisco.com/docs/cisco-netconf-and-rest-device-management-api-documentation/).
- [Ansible: introducción](https://docs.ansible.com/projects/ansible/latest/getting_started/introduction.html) y [automatización de red](https://docs.ansible.com/projects/ansible/latest/network/getting_started/).
- HashiCorp: [flujo Terraform](https://developer.hashicorp.com/terraform/intro/core-workflow) y [estado](https://developer.hashicorp.com/terraform/language/state).
- [Cisco: IA y ML en CCNA v1.1](https://blogs.cisco.com/learning/understanding-the-updated-ccna-v1-1-with-ai-machine-learning-and-more).
