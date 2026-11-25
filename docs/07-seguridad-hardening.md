# 07. Seguridad y hardening de infraestructura

**Proyecto:** Implementación de una Red Empresarial Segura

**Organización simulada:** AndesTech S.A.C.

**Plataforma:** Cisco Packet Tracer

**Dispositivos:** Cisco 2911 y Cisco 2960

---

## 7.1. Objetivo de seguridad

Aplicar controles básicos de seguridad sobre la infraestructura de AndesTech S.A.C. para reducir accesos no autorizados, limitar la comunicación entre departamentos y proteger la administración de los dispositivos de red.

La implementación sigue los principios de:

- Segmentación de red.
- Mínimo privilegio.
- Reducción de la superficie de ataque.
- Separación del tráfico de usuarios y administración.
- Autenticación de administradores.
- Protección de los puertos físicos.
- Defensa en profundidad mediante controles complementarios.

El alcance corresponde a un laboratorio de aprendizaje y demostración técnica, no a una infraestructura de producción auditada.

## 7.2. Arquitectura de seguridad

La red se divide en cinco VLAN operativas y una VLAN para puertos no utilizados.

| VLAN | Área | Protección principal |
|---|---|---|
| 10 | Administración | ACL_ADMIN_IN |
| 20 | Finanzas | ACL_FIN_IN |
| 30 | TI | ACL_TI_IN |
| 40 | Servidores | ACL_SERVIDORES_IN |
| 50 | Gestión | SSHv2 y restricciones VTY |
| 999 | Puertos no utilizados | Aislamiento lógico y shutdown |

Las VLAN 10, 20, 30 y 40 están asociadas a subinterfaces de R1-CORE con ACL de entrada.

La VLAN 50 proporciona direcciones de administración para el router y el switch. El acceso SSH se restringió al equipo PC-IT mediante reglas de red y controles en las líneas VTY.

## 7.3. Riesgos considerados

| Riesgo | Impacto potencial | Control implementado |
|---|---|---|
| Acceso entre departamentos sin autorización | Exposición de recursos internos | ACL departamentales |
| Uso indebido de servicios del servidor | Acceso no autorizado | ACL de servidores |
| Administración desde equipos no autorizados | Cambios indebidos en la infraestructura | SSH y ACL de origen |
| Intercepción de credenciales por Telnet | Exposición de autenticación | SSHv2 y bloqueo de Telnet |
| Conexión a puertos físicos libres | Acceso no autorizado a la red | Shutdown y VLAN 999 |
| Sustitución de dispositivos en puertos activos | Conexión de equipos no previstos | Port Security |
| Conexión accidental de switches a puertos de usuario | Alteración de STP | BPDU Guard |
| Sesiones administrativas abiertas | Uso indebido por terceros | Tiempo de inactividad |
| Acceso físico a la consola | Configuración no autorizada | Autenticación local |

Estos controles reducen riesgos concretos, pero no eliminan todas las amenazas posibles.

## 7.4. Seguridad mediante segmentación VLAN

Se utilizaron VLAN para separar lógicamente los departamentos de Administración, Finanzas, TI y Servidores.

La segmentación permite mantener dominios de broadcast independientes y obliga a utilizar R1-CORE para el enrutamiento entre las subredes.

Sin embargo, una VLAN por sí sola no constituye un firewall. Por ese motivo se aplicaron ACL extendidas en el router.

### 7.4.1. Política general

| Origen | Acceso permitido | Restricciones principales |
|---|---|---|
| Administración | DHCP, DNS, HTTP y ping al gateway | Tráfico general a otras redes |
| Finanzas | DHCP, DNS, HTTP y ping al gateway | Tráfico general a otras redes |
| TI | DHCP, DNS, HTTP, ICMP al servidor y SSH de gestión | Tráfico no autorizado |
| Servidores | Respuestas de servicios definidos y ping al gateway propio | Tráfico restante hacia otras redes |

La política busca mantener los servicios necesarios y restringir conexiones que no formen parte de los requerimientos de cada departamento.

## 7.5. Listas de control de acceso

### 7.5.1. ACL de Administración

La lista `ACL_ADMIN_IN` permite:

- Obtención de configuración por DHCP.
- Solicitudes ICMP al gateway 192.168.10.1.
- Consultas DNS al servidor 192.168.40.10.
- Acceso HTTP a la intranet.

El resto del tráfico originado desde la red 192.168.10.0/24 se bloquea mediante una regla de denegación.

### 7.5.2. ACL de Finanzas

`ACL_FIN_IN` aplica una política equivalente para la red 192.168.20.0/24, adaptando las direcciones al departamento financiero.

### 7.5.3. ACL de TI

`ACL_TI_IN` incorpora dos excepciones adicionales:

1. Solicitudes ICMP desde la VLAN 30 hacia SRV-INTRANET.
2. Conexiones SSH desde PC-IT hacia los equipos de gestión.

Reglas de administración:

```cisco
permit tcp host 192.168.30.100 host 192.168.50.2 eq 22
permit tcp host 192.168.30.100 host 192.168.50.1 eq 22
```

Estas reglas permiten el puerto TCP 22 únicamente desde la dirección IPv4 autorizada.

Las reglas aparecen antes de la denegación final, respetando el orden de evaluación de las ACL.

### 7.5.4. ACL de servidores

Se implementó `ACL_SERVIDORES_IN` en la subinterfaz GigabitEthernet0/0.40.

Esta lista contempla:

- Respuestas DNS por UDP y TCP.
- Respuestas HTTP.
- Respuestas ICMP hacia TI.
- Ping del servidor a su gateway.
- Denegación del tráfico restante procedente de la red de servidores.

Para algunas respuestas TCP se utilizó la palabra clave `established`.

**Limitación importante:** `established` no implementa inspección real de sesiones. Su funcionamiento se basa en indicadores TCP como ACK o RST, por lo que no debe presentarse como equivalente a un firewall con estado.

## 7.6. Alcance y limitaciones de las ACL

Las ACL aplicadas constituyen un mecanismo de filtrado de capa 3 y, cuando corresponde, capa 4.

Sus principales limitaciones son:

- No mantienen una tabla completa de conexiones legítimas.
- No analizan el contenido de las aplicaciones.
- No sustituyen mecanismos de autenticación de usuarios.
- No garantizan protección frente a direcciones IP falsificadas.
- No proporcionan inspección avanzada de amenazas.
- Requieren mantenimiento cuando cambian los servicios o las direcciones IP.

La política de TI permite que cualquier equipo de la VLAN 30 utilice algunos servicios generales definidos para esa red. La administración SSH, en cambio, está restringida específicamente a PC-IT.

La VLAN 50 no tiene actualmente una ACL de entrada propia en el router. La protección del acceso administrativo depende de los permisos en `ACL_TI_IN` y de las ACL de las líneas VTY, además de la ausencia de equipos de usuario conectados directamente a dicha VLAN.

En una implementación de producción sería recomendable incorporar una política de gestión más amplia, filtrado antispoofing y un firewall con inspección de estado.

## 7.7. Protección de puertos físicos

### 7.7.1. Deshabilitación de puertos no utilizados

En SW1-CORE se identificaron los puertos que no participan en la topología:

- FastEthernet0/5 a FastEthernet0/24.
- GigabitEthernet0/2.

Se configuraron como puertos de acceso, se asignaron a VLAN 999 y se deshabilitaron mediante `shutdown`.

Ejemplo:

```cisco
interface range fastEthernet 0/5 - 24
 switchport mode access
 switchport access vlan 999
 shutdown
```

Se deshabilitaron 21 puertos en total.

**Beneficio:** reduce la posibilidad de que un dispositivo conectado a un puerto libre obtenga conectividad inmediata con la red.

La medida no sustituye los controles físicos de acceso a las instalaciones.

## 7.8. Port Security

Port Security se configuró en Fa0/1, Fa0/2, Fa0/3 y Fa0/4.

La política aplicada permite una sola dirección MAC segura por puerto.

```cisco
interface range fastEthernet 0/1 - 4
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation restrict
```

### 7.8.1. Parámetros utilizados

| Parámetro | Valor | Propósito |
|---|---|---|
| Maximum | 1 | Limitar las MAC seguras |
| Aprendizaje | Sticky MAC | Registrar la MAC aprendida |
| Violación | Restrict | Descartar tráfico de MAC no autorizadas |
| Aplicación | Fa0/1–Fa0/4 | Proteger equipos finales |

### 7.8.2. Resultados observados

| Interfaz | Equipo | MAC segura | Violaciones |
|---|---|---|---|
| Fa0/1 | PC-ADMIN | 0090.2114.C24C | 0 |
| Fa0/2 | PC-FIN | 000A.F3B0.0DEC | 0 |
| Fa0/3 | PC-IT | 00E0.F7C9.65DE | 0 |
| Fa0/4 | SRV-INTRANET | 00E0.F7D9.B34B | 0 |

Las cuatro direcciones fueron registradas como `SecureSticky`.

**Consideración:** Port Security basado en MAC no proporciona identidad criptográfica del dispositivo. Un atacante con capacidad de suplantar una MAC podría intentar evadir este control. Para entornos empresariales se recomienda evaluar mecanismos adicionales como IEEE 802.1X.

**Evidencia:** [Port Security](../evidence/06-port-security.png).

## 7.9. PortFast y BPDU Guard

Se configuraron PortFast y BPDU Guard en los cuatro puertos de acceso utilizados por computadoras y servidor.

```cisco
interface range fastEthernet 0/1 - 4
 spanning-tree portfast
 spanning-tree bpduguard enable
```

### 7.9.1. PortFast

Permite que un puerto destinado a un dispositivo final pase rápidamente al estado de reenvío de Spanning Tree al establecerse el enlace.

Se utiliza en puertos donde no se espera conectar otro switch.

### 7.9.2. BPDU Guard

Protege el puerto frente a la recepción inesperada de BPDUs.

Cuando una interfaz protegida recibe una BPDU, puede pasar al estado `err-disabled`, según el funcionamiento del dispositivo.

Este control ayuda a evitar conexiones accidentales o indebidas de equipos que participan en Spanning Tree.

**Importante:** PortFast y BPDU Guard se aplicaron únicamente a Fa0/1–Fa0/4, no al enlace trunk Gi0/1.

## 7.10. Protección del enlace trunk

El enlace Gi0/1 conecta SW1-CORE con R1-CORE.

Se configuró de forma estática para transportar únicamente las VLAN necesarias.

```cisco
interface gigabitEthernet 0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40,50
 switchport nonegotiate
```

La configuración cumple tres objetivos:

- Evitar la negociación dinámica del modo trunk.
- Limitar las VLAN transportadas.
- Mantener la VLAN 999 fuera de la lista de VLAN permitidas.

### Consideración pendiente

La VLAN nativa del trunk permanece en VLAN 1, según el resultado de `show interfaces trunk`.

En una futura mejora se puede evaluar una VLAN nativa dedicada y no utilizada, configurada de manera consistente en ambos extremos.

Este cambio no se aplicó en la versión actual del laboratorio.

## 7.11. Red dedicada de administración

Se creó la VLAN 50 con el nombre `GESTION`.

| Dispositivo | Dirección de administración |
|---|---|
| R1-CORE | 192.168.50.1 |
| SW1-CORE | 192.168.50.2 |

Esta separación permite identificar y controlar el tráfico administrativo independientemente de las VLAN de usuarios y servidores.

La red de gestión utiliza el mismo enlace trunk físico, pero se encuentra identificada mediante una VLAN independiente.

**Limitación:** una VLAN de gestión no es equivalente a una red física de administración fuera de banda.

## 7.12. Administración segura mediante SSHv2

Se habilitó SSH versión 2 en ambos dispositivos.

La implementación incluye:

- Nombre de dominio `andestech.test`.
- Claves RSA de 2048 bits.
- Autenticación local.
- Restricción de acceso a las líneas VTY.
- Deshabilitación de Telnet mediante `transport input ssh`.
- Acceso desde la IP de PC-IT.

Se comprobó que `show ip ssh` mostrara:

```text
SSH Enabled - version 2.0
```

### 7.12.1. Restricción de origen

En R1-CORE se utilizó:

```cisco
ip access-list standard ACL_VTY_SOLO_TI
 permit host 192.168.30.100
 deny any
```

En SW1-CORE se implementó una lista equivalente denominada `ACL_SSH_SOLO_TI`.

Estas ACL se aplicaron a las líneas VTY mediante `access-class`.

### 7.12.2. Protección de las líneas VTY

Ejemplo de configuración utilizada:

```cisco
line vty 0 4
 login local
 transport input ssh
 access-class ACL_VTY_SOLO_TI in
 exec-timeout 10 0
```

En el router y switch se configuraron también las líneas VTY 5–15.

El tiempo de inactividad de diez minutos corresponde al valor seleccionado para el laboratorio. En algunas versiones de Cisco IOS puede omitirse de `running-config` al coincidir con el valor predeterminado.

### 7.12.3. Validación de acceso

| Origen | Router | Switch |
|---|---|---|
| PC-IT | SSH permitido | SSH permitido |
| PC-ADMIN | SSH bloqueado | SSH bloqueado |
| PC-FIN | SSH bloqueado | SSH bloqueado |

Las pruebas desde PC-IT permitieron acceder a ambos dispositivos.

Desde Administración y Finanzas no se establecieron conexiones administrativas.

**Evidencias:**

- [SSH hacia el router](../evidence/07-ssh-ti-router.png)
- [SSH hacia el switch](../evidence/08-ssh-ti-switch.png)
- [Acceso denegado desde Administración](../evidence/09-ssh-acceso-denegado-ADMIN.png)
- [Acceso denegado desde Finanzas](../evidence/10-ssh-acceso-denegado-FIN.png)

## 7.13. Autenticación y privilegios administrativos

Se configuró una cuenta local denominada `adminred`.

El router utiliza una cuenta con privilegio administrativo 15.

En el switch se comprobó que el usuario puede iniciar sesión y acceder al modo privilegiado mediante `enable`.

También se configuró una contraseña específica para el modo privilegiado mediante `enable secret`.

### 7.13.1. Protección de consola

Se habilitó `login local` en las líneas de consola del router y del switch.

Esto exige autenticación local al iniciar una nueva sesión por consola.

### 7.13.2. Protección de credenciales

Para evitar divulgar secretos del laboratorio:

- No se publican contraseñas en texto claro.
- No se incluyen hashes de contraseñas en la documentación.
- No se almacenan credenciales reales en archivos Markdown.
- Las contraseñas utilizadas en la simulación deben ser exclusivas del laboratorio.
- Cualquier contraseña compartida previamente debe reemplazarse antes de publicar el proyecto.

**Limitación criptográfica:** las versiones antiguas de IOS representadas en Packet Tracer pueden utilizar hashes de tipo 5, basados en MD5. Estos mecanismos no corresponden a las mejores opciones criptográficas para entornos productivos actuales.

## 7.14. Evaluación de controles implementados

| Control | Objetivo | Resultado de implementación |
|---|---|---|
| VLAN departamentales | Separación lógica | Implementado |
| ACL extendidas | Filtrado entre redes | Implementado y probado en escenarios definidos |
| VLAN de gestión | Separación administrativa | Implementada |
| SSHv2 | Administración cifrada | Implementado y probado |
| ACL en VTY | Restricción de administradores | Implementada y probada |
| Port Security | Control básico de MAC | Implementado |
| BPDU Guard | Protección de puertos de acceso | Implementado |
| PortFast | Operación eficiente en puertos finales | Implementado |
| VLAN 999 | Aislamiento de puertos libres | Implementada |
| Shutdown de puertos | Reducción de superficie de ataque | Implementado |
| Autenticación local | Control de acceso a dispositivos | Implementado |

## 7.15. Riesgos residuales

Aunque los controles implementados mejoran la seguridad de la infraestructura, permanecen algunos riesgos:

**1. Falta de redundancia.** La falla del router o switch principal puede interrumpir el funcionamiento de la red.

**2. Credenciales locales.** La administración no se encuentra centralizada mediante un servidor AAA.

**3. Uso de HTTP.** El servicio de intranet utiliza HTTP sin cifrado HTTPS.

**4. Limitaciones de ACL.** No existe inspección completa del estado de las sesiones.

**5. Ausencia de autenticación de red por dispositivo.** No se implementó IEEE 802.1X.

**6. Protección de capa 2 parcial.** No se implementaron ni probaron todos los mecanismos disponibles contra DHCP spoofing, ARP spoofing o manipulación de STP.

**7. Administración basada en dirección IP.** La autorización por IP no sustituye la autenticación fuerte de identidad.

**8. Falta de supervisión centralizada.** No existe integración con syslog remoto, SIEM o monitoreo de eventos de seguridad.

**9. Características del simulador.** Algunas opciones de IOS, criptografía y comportamiento de red pueden diferir de los equipos reales.

## 7.16. Recomendaciones de mejora

Para evolucionar el proyecto hacia una arquitectura más completa se propone:

1. Implementar un firewall con estado entre zonas de seguridad.
2. Incorporar HTTPS con certificados de prueba.
3. Configurar autenticación centralizada mediante AAA, RADIUS o TACACS+.
4. Evaluar IEEE 802.1X para los puertos de acceso.
5. Aplicar DHCP Snooping y Dynamic ARP Inspection en equipos compatibles.
6. Fortalecer la red de gestión con controles adicionales y una política específica de origen.
7. Incorporar monitoreo y registros centralizados.
8. Diseñar redundancia para los componentes críticos.
9. Incorporar un proceso formal de gestión de cambios y respaldos.
10. Ejecutar pruebas de seguridad adicionales y documentar sus resultados.

Estas mejoras se plantean como actividades futuras; no forman parte de la configuración implementada en esta versión.

## 7.17. Conclusión

La implementación de AndesTech S.A.C. demuestra la aplicación integrada de controles básicos de seguridad de redes empresariales.

La combinación de segmentación VLAN, ACL extendidas, Port Security, BPDU Guard, deshabilitación de puertos no utilizados, SSHv2 y autenticación administrativa permite reducir distintos riesgos asociados a la infraestructura.

La validación evidenció el funcionamiento de los servicios autorizados y la denegación de accesos SSH desde departamentos no autorizados.

La solución representa una base funcional de aprendizaje y demostración de habilidades en infraestructura TI y ciberseguridad.

**Documentos relacionados:**

- [05. Implementación técnica](05-implementacion.md)
- [06. Pruebas y validación](06-pruebas-validacion.md)

---

**Proyecto 01 — Implementación de una Red Empresarial Segura | AndesTech S.A.C.**
