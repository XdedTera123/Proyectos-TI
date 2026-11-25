# Fase 02.3 — Enrutamiento Inter-VLAN y Políticas de Seguridad

## 1. Objetivo

Definir el mecanismo de enrutamiento y las políticas de comunicación entre las VLAN de AndesTech S.A.C., siguiendo el principio de mínimo privilegio.

## 2. Arquitectura de enrutamiento

Se utilizará una arquitectura Router-on-a-Stick mediante un router Cisco 2911 y un switch Cisco 2960.

La comunicación entre ambos dispositivos utilizará un enlace troncal IEEE 802.1Q.

### 2.1 Subinterfaces del router

| Subinterfaz | VLAN | Dirección IPv4 | Función |
|---|---|---|---|
| G0/0.10 | 10 | 192.168.10.1/24 | Gateway Administración |
| G0/0.20 | 20 | 192.168.20.1/24 | Gateway Finanzas |
| G0/0.30 | 30 | 192.168.30.1/24 | Gateway TI |
| G0/0.40 | 40 | 192.168.40.1/24 | Gateway Servidores |

Las subinterfaces permitirán que el router procese tráfico de diferentes VLAN utilizando una sola interfaz física.

## 3. Principio de mínimo privilegio

Se establecerán políticas que permitan únicamente las comunicaciones necesarias para el funcionamiento de los servicios empresariales.

Las comunicaciones no autorizadas entre departamentos serán restringidas mediante listas de control de acceso (ACL).

## 4. Matriz de control de acceso

| Origen | Destino | Servicio | Acción |
|---|---|---|---|
| Administración | Finanzas | Todos | DENY |
| Administración | TI | Todos | DENY |
| Finanzas | Administración | Todos | DENY |
| Finanzas | TI | Todos | DENY |
| TI | Administración | Todos | DENY |
| TI | Finanzas | Todos | DENY |
| Administración | Servidor interno | DNS y HTTP | PERMIT |
| Finanzas | Servidor interno | DNS y HTTP | PERMIT |
| TI | Servidor interno | DNS, HTTP e ICMP | PERMIT |
| Usuarios | Gateway de su VLAN | ICMP | PERMIT |
| Usuarios | Servicio DHCP del router | DHCP | PERMIT |

Las comunicaciones de usuarios no contempladas en esta matriz se denegarán en esta primera versión.

## 5. Protocolos y servicios

| Servicio | Protocolo | Puerto |
|---|---|---|
| DNS | TCP/UDP | 53 |
| HTTP | TCP | 80 |
| DHCP | UDP | 67/68 |
| ICMP | ICMP | No aplica |

## 6. Estrategia de implementación

Se utilizarán ACL extendidas para filtrar el tráfico según la dirección de origen, dirección de destino, protocolo y servicio.

Inicialmente, las ACL se aplicarán en dirección de entrada en las subinterfaces correspondientes a las VLAN de usuarios.

La implementación deberá contemplar:

1. Permitir el servicio DHCP.
2. Permitir las pruebas ICMP hacia el gateway correspondiente.
3. Permitir los servicios DNS y HTTP autorizados.
4. Permitir ICMP desde TI hacia el servidor.
5. Denegar las comunicaciones no autorizadas.
6. Verificar que el tráfico de respuesta de los servicios permitidos funcione correctamente.

Las ACL convencionales no realizan seguimiento completo del estado de las conexiones. Por ello, las restricciones iniciales sobre VLAN de usuarios no equivalen a un aislamiento bidireccional completo de toda la infraestructura.

## 7. Plan de validación

| ID | Prueba | Resultado esperado |
|---|---|---|
| SEC-01 | Administración → Finanzas mediante ping | Bloqueado |
| SEC-02 | Finanzas → Administración mediante ping | Bloqueado |
| SEC-03 | Finanzas → TI mediante ping | Bloqueado |
| SEC-04 | Administración → Servidor mediante HTTP | Permitido |
| SEC-05 | Finanzas → Servidor mediante DNS | Permitido |
| SEC-06 | TI → Servidor mediante ping | Permitido |
| SEC-07 | Finanzas → Servidor mediante ping | Bloqueado |
| SEC-08 | Cada PC → su gateway mediante ping | Permitido |
| SEC-09 | Cada PC → servicio DHCP del router | Permitido |

## 8. Limitaciones de la primera versión

- Las políticas corresponden a un laboratorio simulado.
- No se implementará un firewall de inspección con estado.
- No se contempla conectividad hacia Internet.
- No se implementará alta disponibilidad.
- No se garantiza todavía el filtrado bidireccional completo de comunicaciones iniciadas desde la VLAN de servidores.
- La efectividad de las ACL deberá demostrarse mediante pruebas reales en Packet Tracer.

## 9. Estado

Diseño de políticas de acceso definido.

**Pendiente:** configuración y validación de ACL en Cisco Packet Tracer.