# 05. Implementación técnica de la red empresarial

**Proyecto:** Implementación de una Red Empresarial Segura

**Organización simulada:** AndesTech S.A.C.

**Herramienta:** Cisco Packet Tracer

**Tecnologías:** VLAN, IEEE 802.1Q, Router-on-a-Stick, DHCP, DNS, HTTP, ACL, Port Security, STP y SSHv2

**Estado:** Implementación funcional y validada en entorno de simulación

---

## 5.1. Objetivo de la implementación

Implementar una infraestructura de red empresarial segmentada y administrable, capaz de proporcionar conectividad controlada entre distintos departamentos, servicios internos y mecanismos de protección de acceso.

La solución se desarrolló mediante Cisco Packet Tracer, utilizando equipos Cisco simulados y una arquitectura Router-on-a-Stick.

Los principales objetivos técnicos fueron:

- Separar lógicamente los departamentos mediante VLAN.
- Implementar enrutamiento inter-VLAN controlado.
- Automatizar la asignación de direcciones IPv4 mediante DHCP.
- Proporcionar servicios internos DNS y HTTP.
- Restringir las comunicaciones mediante listas de control de acceso (ACL).
- Proteger los puertos físicos del switch.
- Implementar una red independiente de administración.
- Habilitar administración remota cifrada mediante SSHv2.
- Permitir la administración únicamente desde el equipo autorizado de TI.

## 5.2. Equipamiento y topología

La infraestructura implementada está compuesta por los siguientes dispositivos:

| Dispositivo | Modelo | Función |
|---|---|---|
| R1-CORE | Cisco 2911 | Enrutamiento inter-VLAN, DHCP, ACL y administración |
| SW1-CORE | Cisco 2960 | Conmutación, VLAN, trunk y seguridad de puertos |
| PC-ADMIN | PC-PT | Estación de Administración |
| PC-FIN | PC-PT | Estación de Finanzas |
| PC-IT | PC-PT | Estación de Tecnologías de Información |
| SRV-INTRANET | Server-PT | Servidor DNS y HTTP |

### 5.2.1. Conexiones físicas

| Origen | Interfaz | Destino | Interfaz |
|---|---|---|---|
| R1-CORE | GigabitEthernet0/0 | SW1-CORE | GigabitEthernet0/1 |
| SW1-CORE | FastEthernet0/1 | PC-ADMIN | FastEthernet0 |
| SW1-CORE | FastEthernet0/2 | PC-FIN | FastEthernet0 |
| SW1-CORE | FastEthernet0/3 | PC-IT | FastEthernet0 |
| SW1-CORE | FastEthernet0/4 | SRV-INTRANET | FastEthernet0 |

La topología utiliza una estructura en estrella, con el switch como punto central de conexión y el router como dispositivo de capa 3.

El enlace entre ambos equipos está configurado como trunk IEEE 802.1Q, permitiendo transportar múltiples VLAN mediante una sola conexión física.

**Archivo de simulación:** [AndesTech-Enterprise-Network-v01.pkt](../topology/AndesTech-Enterprise-Network-v01.pkt)

**Evidencia:** [Topología final](../evidence/04-topologia-final.png)

## 5.3. Segmentación lógica mediante VLAN

Se implementaron cinco VLAN operativas y una VLAN adicional destinada a puertos sin utilizar.

| VLAN | Nombre | Red IPv4 | Gateway | Propósito |
|---|---|---|---|---|
| 10 | ADMINISTRACION | 192.168.10.0/24 | 192.168.10.1 | Departamento administrativo |
| 20 | FINANZAS | 192.168.20.0/24 | 192.168.20.1 | Departamento financiero |
| 30 | TI | 192.168.30.0/24 | 192.168.30.1 | Departamento de TI |
| 40 | SERVIDORES | 192.168.40.0/24 | 192.168.40.1 | Servicios internos |
| 50 | GESTION | 192.168.50.0/24 | 192.168.50.1 | Administración de infraestructura |
| 999 | UNUSED_PORTS | Sin direccionamiento | No aplica | Puertos deshabilitados |

### 5.3.1. Configuración de VLAN en SW1-CORE

Las VLAN se crearon mediante el modo de configuración global del switch.

```cisco
configure terminal

vlan 10
 name ADMINISTRACION
exit

vlan 20
 name FINANZAS
exit

vlan 30
 name TI
exit

vlan 40
 name SERVIDORES
exit

vlan 50
 name GESTION
exit

vlan 999
 name UNUSED_PORTS
exit

end
```

### 5.3.2. Asignación de puertos

Se asignó un puerto de acceso independiente a cada dispositivo final.

```cisco
configure terminal

interface fastEthernet 0/1
 description PC-ADMIN_VLAN10
 switchport mode access
 switchport access vlan 10
exit

interface fastEthernet 0/2
 description PC-FIN_VLAN20
 switchport mode access
 switchport access vlan 20
exit

interface fastEthernet 0/3
 description PC-IT_VLAN30
 switchport mode access
 switchport access vlan 30
exit

interface fastEthernet 0/4
 description SRV-INTRANET_VLAN40
 switchport mode access
 switchport access vlan 40
exit

end
```

La separación mediante VLAN establece dominios de broadcast independientes. La comunicación entre departamentos requiere pasar por el router, donde se aplican las políticas de seguridad.

## 5.4. Configuración del enlace trunk

El enlace entre SW1-CORE y R1-CORE permite transportar el tráfico etiquetado de las VLAN 10, 20, 30, 40 y 50.

En SW1-CORE se configuró:

```cisco
configure terminal

interface gigabitEthernet 0/1
 description TRUNK_HACIA_R1-CORE
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40,50
 switchport nonegotiate
exit

end
```

El comando `switchport nonegotiate` deshabilita la negociación dinámica DTP en este enlace, que ya se encuentra configurado manualmente como trunk.

La lista de VLAN permitidas evita transportar la VLAN 999 por el enlace de interconexión.

Se verificó mediante:

```cisco
show interfaces trunk
```

La salida confirmó que las cinco VLAN operativas se encontraban permitidas, activas y en estado de reenvío de Spanning Tree.

## 5.5. Enrutamiento inter-VLAN mediante Router-on-a-Stick

El router R1-CORE realiza el enrutamiento entre las redes departamentales utilizando subinterfaces asociadas a identificadores 802.1Q.

Cada subinterfaz representa la puerta de enlace predeterminada de una VLAN.

### 5.5.1. Configuración de R1-CORE

```cisco
configure terminal

interface gigabitEthernet 0/0
 description TRUNK_HACIA_SW1-CORE
 no shutdown
exit

interface gigabitEthernet 0/0.10
 description GATEWAY_ADMINISTRACION
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
exit

interface gigabitEthernet 0/0.20
 description GATEWAY_FINANZAS
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
exit

interface gigabitEthernet 0/0.30
 description GATEWAY_TI
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
exit

interface gigabitEthernet 0/0.40
 description GATEWAY_SERVIDORES
 encapsulation dot1Q 40
 ip address 192.168.40.1 255.255.255.0
exit

interface gigabitEthernet 0/0.50
 description GATEWAY_VLAN50_GESTION
 encapsulation dot1Q 50
 ip address 192.168.50.1 255.255.255.0
exit

end
```

Se verificó el estado de las interfaces mediante:

```cisco
show ip interface brief
```

Las cinco subinterfaces presentaron estado `up/up`.

Antes de aplicar las ACL se comprobaron comunicaciones inter-VLAN entre las tres computadoras, lo que permitió validar el enrutamiento básico de la infraestructura.

## 5.6. Implementación del servicio DHCP

R1-CORE se configuró como servidor DHCP para los departamentos de Administración, Finanzas y TI.

### 5.6.1. Exclusión de direcciones

Se reservaron los rangos inferiores y superiores de cada subred para evitar su asignación automática.

```cisco
configure terminal

ip dhcp excluded-address 192.168.10.1 192.168.10.99
ip dhcp excluded-address 192.168.10.200 192.168.10.254

ip dhcp excluded-address 192.168.20.1 192.168.20.99
ip dhcp excluded-address 192.168.20.200 192.168.20.254

ip dhcp excluded-address 192.168.30.1 192.168.30.99
ip dhcp excluded-address 192.168.30.200 192.168.30.254

end
```

### 5.6.2. Creación de pools DHCP

```cisco
configure terminal

ip dhcp pool VLAN10_ADMIN
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 192.168.40.10
exit

ip dhcp pool VLAN20_FIN
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
 dns-server 192.168.40.10
exit

ip dhcp pool VLAN30_TI
 network 192.168.30.0 255.255.255.0
 default-router 192.168.30.1
 dns-server 192.168.40.10
exit

end
```

### 5.6.3. Asignaciones verificadas

| Equipo | IP obtenida | Gateway | DNS |
|---|---|---|---|
| PC-ADMIN | 192.168.10.100 | 192.168.10.1 | 192.168.40.10 |
| PC-FIN | 192.168.20.100 | 192.168.20.1 | 192.168.40.10 |
| PC-IT | 192.168.30.100 | 192.168.30.1 | 192.168.40.10 |

Los clientes obtuvieron sus direcciones mediante DHCP y respondieron correctamente al ping hacia sus respectivos gateways.

No se necesitó configurar `ip helper-address` porque el propio router proporciona DHCP para las redes conectadas directamente.

## 5.7. Implementación de DNS y HTTP

Se configuró un servidor interno en la VLAN 40.

### 5.7.1. Direccionamiento del servidor

| Parámetro | Valor |
|---|---|
| Dispositivo | SRV-INTRANET |
| Dirección IPv4 | 192.168.40.10 |
| Máscara | 255.255.255.0 |
| Gateway | 192.168.40.1 |
| DNS | 192.168.40.10 |
| Asignación | Estática |

### 5.7.2. Servicio DNS

En la sección **Services → DNS** de Server-PT se activó el servicio y se creó el siguiente registro:

| Nombre de dominio | Tipo | Dirección IPv4 |
|---|---|---|
| intranet.andestech.test | A | 192.168.40.10 |

Este registro permite acceder a la intranet utilizando un nombre en lugar de escribir directamente su dirección IP.

### 5.7.3. Servicio HTTP

En **Services → HTTP** se habilitó el servidor web y se configuró una página de intranet identificada con AndesTech S.A.C.

La dirección utilizada fue:

```text
http://intranet.andestech.test
```

El funcionamiento conjunto de DNS y HTTP se verificó desde las tres computadoras.

**Evidencias:**

- [HTTP y DNS — Administración](../evidence/01-http-dns-pc-admin.png)
- [HTTP y DNS — Finanzas](../evidence/02-http-dns-pc-fin.png)
- [HTTP y DNS — TI](../evidence/03-http-dns-pc-it.png)

## 5.8. Implementación de políticas de acceso mediante ACL

Después de validar la conectividad básica se implementaron ACL extendidas para controlar las comunicaciones entre VLAN.

Las listas fueron aplicadas en dirección de entrada (`in`) sobre las subinterfaces correspondientes de R1-CORE.

### 5.8.1. Política de Administración

La VLAN 10 dispone de permisos para DHCP, ping a su propio gateway, consultas DNS y acceso HTTP a la intranet.

Las demás comunicaciones originadas desde dicha red se deniegan.

```cisco
ip access-list extended ACL_ADMIN_IN
 permit udp any eq bootpc any eq bootps
 permit icmp 192.168.10.0 0.0.0.255 host 192.168.10.1 echo
 permit udp 192.168.10.0 0.0.0.255 host 192.168.40.10 eq domain
 permit tcp 192.168.10.0 0.0.0.255 host 192.168.40.10 eq domain
 permit tcp 192.168.10.0 0.0.0.255 host 192.168.40.10 eq www
 deny ip 192.168.10.0 0.0.0.255 any
```

Aplicación:

```cisco
interface gigabitEthernet 0/0.10
 ip access-group ACL_ADMIN_IN in
```

### 5.8.2. Política de Finanzas

La VLAN 20 mantiene los servicios internos necesarios, pero no puede iniciar comunicaciones generales hacia las otras redes.

```cisco
ip access-list extended ACL_FIN_IN
 permit udp any eq bootpc any eq bootps
 permit icmp 192.168.20.0 0.0.0.255 host 192.168.20.1 echo
 permit udp 192.168.20.0 0.0.0.255 host 192.168.40.10 eq domain
 permit tcp 192.168.20.0 0.0.0.255 host 192.168.40.10 eq domain
 permit tcp 192.168.20.0 0.0.0.255 host 192.168.40.10 eq www
 deny ip 192.168.20.0 0.0.0.255 any
```

Aplicación:

```cisco
interface gigabitEthernet 0/0.20
 ip access-group ACL_FIN_IN in
```

### 5.8.3. Política de TI

La VLAN 30 dispone de permisos adicionales para realizar pruebas ICMP al servidor y administrar los dispositivos de red mediante SSH.

```cisco
ip access-list extended ACL_TI_IN
 permit udp any eq bootpc any eq bootps
 permit icmp 192.168.30.0 0.0.0.255 host 192.168.30.1 echo
 permit udp 192.168.30.0 0.0.0.255 host 192.168.40.10 eq domain
 permit tcp 192.168.30.0 0.0.0.255 host 192.168.40.10 eq domain
 permit tcp 192.168.30.0 0.0.0.255 host 192.168.40.10 eq www
 permit icmp 192.168.30.0 0.0.0.255 host 192.168.40.10 echo
 permit tcp host 192.168.30.100 host 192.168.50.2 eq 22
 permit tcp host 192.168.30.100 host 192.168.50.1 eq 22
 deny ip 192.168.30.0 0.0.0.255 any
```

Aplicación:

```cisco
interface gigabitEthernet 0/0.30
 ip access-group ACL_TI_IN in
```

### 5.8.4. Política de servidores

Se creó `ACL_SERVIDORES_IN` para restringir el tráfico que ingresa al router desde VLAN 40.

La política permite respuestas DNS, respuestas HTTP, respuestas ICMP autorizadas hacia TI y ping del servidor a su propio gateway.

```cisco
ip access-list extended ACL_SERVIDORES_IN
 permit udp host 192.168.40.10 eq 53 192.168.10.0 0.0.0.255
 permit udp host 192.168.40.10 eq 53 192.168.20.0 0.0.0.255
 permit udp host 192.168.40.10 eq 53 192.168.30.0 0.0.0.255

 permit tcp host 192.168.40.10 eq 53 192.168.10.0 0.0.0.255 established
 permit tcp host 192.168.40.10 eq 53 192.168.20.0 0.0.0.255 established
 permit tcp host 192.168.40.10 eq 53 192.168.30.0 0.0.0.255 established

 permit tcp host 192.168.40.10 eq 80 192.168.10.0 0.0.0.255 established
 permit tcp host 192.168.40.10 eq 80 192.168.20.0 0.0.0.255 established
 permit tcp host 192.168.40.10 eq 80 192.168.30.0 0.0.0.255 established

 permit icmp host 192.168.40.10 192.168.30.0 0.0.0.255 echo-reply
 permit icmp host 192.168.40.10 host 192.168.40.1 echo

 deny ip 192.168.40.0 0.0.0.255 any
```

Aplicación:

```cisco
interface gigabitEthernet 0/0.40
 ip access-group ACL_SERVIDORES_IN in
```

**Consideración técnica:** las ACL tradicionales son filtros de paquetes sin inspección completa del estado de las conexiones. La palabra clave `established` comprueba indicadores TCP ACK/RST, pero no verifica una tabla de sesiones legítimas. Por tanto, estas políticas ofrecen control de tráfico básico y no sustituyen a un firewall con estado.

### 5.8.5. Verificación de ACL

Se utilizaron los comandos:

```cisco
show ip access-lists
show ip interface gigabitEthernet 0/0.10
show ip interface gigabitEthernet 0/0.20
show ip interface gigabitEthernet 0/0.30
show ip interface gigabitEthernet 0/0.40
```

Las cuatro ACL quedaron asociadas a sus subinterfaces respectivas.

Se realizaron pruebas de comunicación permitida y denegada para comprobar el comportamiento de las políticas.

**Evidencia:** [Segmentación mediante ACL](../evidence/05-segmentacion-acl.png)

## 5.9. Implementación de la VLAN de gestión

La VLAN 50 se creó para separar la administración de los dispositivos de red de las VLAN de usuarios y servidores.

### 5.9.1. Direccionamiento de gestión

| Dispositivo | Dirección | Interfaz |
|---|---|---|
| R1-CORE | 192.168.50.1/24 | GigabitEthernet0/0.50 |
| SW1-CORE | 192.168.50.2/24 | Vlan50 |

### 5.9.2. Configuración de la interfaz de gestión del switch

```cisco
configure terminal

interface vlan 50
 description ADMINISTRACION_DEL_SWITCH
 ip address 192.168.50.2 255.255.255.0
 no shutdown
exit

ip default-gateway 192.168.50.1

end
```

La comunicación entre R1-CORE y SW1-CORE se verificó mediante ping a `192.168.50.2`, obteniendo cinco respuestas correctas en una segunda ejecución de la prueba.

## 5.10. Implementación de SSHv2

Se habilitó administración remota cifrada en ambos dispositivos.

### 5.10.1. Preparación del servicio SSH

La implementación incluyó los siguientes parámetros:

- Dominio: `andestech.test`.
- Generación de claves RSA de 2048 bits.
- Utilización exclusiva de SSH versión 2.
- Autenticación mediante usuarios locales.
- Restricción de acceso a través de ACL aplicadas a las líneas VTY.
- Deshabilitación de Telnet mediante `transport input ssh`.

Los comandos principales utilizados fueron:

```cisco
configure terminal

ip domain-name andestech.test
crypto key generate rsa
ip ssh version 2

end
```

Durante la generación de claves RSA se seleccionó un módulo de 2048 bits.

Las credenciales reales se configuran directamente en cada dispositivo y no se incluyen en este documento.

### 5.10.2. Política de acceso administrativo

Se crearon ACL estándar para permitir únicamente conexiones desde PC-IT (`192.168.30.100`).

En R1-CORE se utiliza `ACL_VTY_SOLO_TI`, mientras que en SW1-CORE se utiliza `ACL_SSH_SOLO_TI`.

Ejemplo de política de origen:

```cisco
ip access-list standard ACL_VTY_SOLO_TI
 permit host 192.168.30.100
 deny any
```

Configuración aplicada a las líneas VTY del router:

```cisco
line vty 0 4
 login local
 transport input ssh
 access-class ACL_VTY_SOLO_TI in
exit

line vty 5 15
 login local
 transport input ssh
 access-class ACL_VTY_SOLO_TI in
exit
```

En el switch se aplicó una configuración equivalente con la ACL `ACL_SSH_SOLO_TI`.

### 5.10.3. Pruebas SSH

Desde PC-IT se verificaron las conexiones:

```text
ssh -l adminred 192.168.50.1
ssh -l adminred 192.168.50.2
```

Ambas conexiones resultaron exitosas y se comprobó la disponibilidad de SSH versión 2.

También se realizaron intentos desde PC-ADMIN y PC-FIN, los cuales no consiguieron establecer conexiones SSH con los dispositivos de gestión.

**Evidencias:**

- [SSH autorizado hacia el router](../evidence/07-ssh-ti-router.png)
- [SSH autorizado hacia el switch](../evidence/08-ssh-ti-switch.png)
- [SSH denegado desde Administración](../evidence/09-ssh-acceso-denegado-ADMIN.png)
- [SSH denegado desde Finanzas](../evidence/10-ssh-acceso-denegado-FIN.png)

## 5.11. Endurecimiento básico del switch

Se implementaron medidas adicionales de seguridad sobre las interfaces físicas de SW1-CORE.

### 5.11.1. Deshabilitación de puertos no utilizados

Los puertos Fa0/5 a Fa0/24 y Gi0/2 fueron asignados a VLAN 999 y deshabilitados administrativamente.

```cisco
configure terminal

interface range fastEthernet 0/5 - 24
 switchport mode access
 switchport access vlan 999
 shutdown
exit

interface gigabitEthernet 0/2
 switchport mode access
 switchport access vlan 999
 shutdown
exit

end
```

En total se deshabilitaron 21 puertos que no eran necesarios para la operación del laboratorio.

### 5.11.2. PortFast y BPDU Guard

Se aplicaron ambos controles a los puertos de acceso de los dispositivos finales.

```cisco
configure terminal

interface range fastEthernet 0/1 - 4
 spanning-tree portfast
 spanning-tree bpduguard enable
exit

end
```

PortFast agiliza la transición del puerto al estado de reenvío. BPDU Guard protege frente a la recepción inesperada de BPDUs en puertos destinados a equipos finales.

### 5.11.3. Port Security

Se configuró una dirección MAC segura como máximo por puerto, utilizando aprendizaje Sticky MAC y acción de infracción `restrict`.

```cisco
configure terminal

interface range fastEthernet 0/1 - 4
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation restrict
exit

end
```

La verificación confirmó una dirección MAC segura en cada uno de los cuatro puertos activos y cero infracciones registradas.

**Evidencia:** [Port Security](../evidence/06-port-security.png)

## 5.12. Verificación y persistencia de la configuración

Se utilizaron comandos de diagnóstico para revisar la configuración y el estado operativo de los dispositivos.

| Comando | Propósito |
|---|---|
| `show ip interface brief` | Verificar IP y estado de interfaces |
| `show vlan brief` | Verificar VLAN y puertos asociados |
| `show interfaces trunk` | Verificar transporte de VLAN por el trunk |
| `show ip access-lists` | Consultar políticas ACL y coincidencias |
| `show port-security` | Consultar seguridad de los puertos |
| `show port-security address` | Consultar direcciones MAC seguras |
| `show ip ssh` | Comprobar el estado y versión de SSH |
| `show privilege` | Consultar el nivel de privilegio actual |
| `show running-config` | Consultar la configuración activa |

Finalmente se guardó la configuración de los dispositivos mediante:

```cisco
copy running-config startup-config
```

También se guardó la simulación de Packet Tracer en el archivo principal del proyecto.

## 5.13. Limitaciones y consideraciones técnicas

Esta implementación corresponde a un laboratorio empresarial simulado. Aunque permite demostrar conceptos de segmentación, conectividad y endurecimiento de infraestructura, presenta algunas limitaciones:

1. **ACL sin estado:** las ACL convencionales no ofrecen inspección completa de sesiones como un firewall empresarial.
2. **Autenticación local:** el laboratorio emplea usuarios configurados individualmente; no integra un sistema centralizado AAA mediante RADIUS o TACACS+.
3. **Protección de capa 2:** se implementaron Port Security y BPDU Guard, pero no una solución integral de protección contra todos los ataques de capa 2.
4. **Redundancia:** se utiliza un único router y un único switch, por lo que no existe tolerancia a fallos de estos dispositivos.
5. **Servicios internos:** DNS y HTTP operan en un servidor simulado. La intranet utiliza HTTP, sin cifrado HTTPS.
6. **Credenciales del simulador:** Cisco Packet Tracer puede almacenar hashes y material de configuración en el archivo `.pkt`. Las credenciales utilizadas deben ser exclusivas del laboratorio y no reutilizarse en sistemas reales.
7. **Administración por SSH:** se implementaron restricciones por IP de origen y SSHv2, pero no mecanismos adicionales como MFA, acceso mediante VPN o supervisión centralizada.

Estas limitaciones representan posibles líneas de mejora para futuras versiones de la infraestructura.

## 5.14. Resultado de la implementación

La infraestructura de AndesTech S.A.C. quedó implementada con segmentación departamental, direccionamiento dinámico, servicios internos, control de acceso y administración remota restringida.

Se logró:

- Establecer cinco VLAN operativas.
- Implementar Router-on-a-Stick mediante IEEE 802.1Q.
- Configurar DHCP para los tres departamentos.
- Habilitar DNS y HTTP mediante un servidor interno.
- Aplicar cuatro ACL extendidas para el filtrado de tráfico.
- Implementar una red de gestión independiente.
- Restringir SSH al equipo autorizado de TI.
- Proteger los puertos de acceso mediante Port Security y BPDU Guard.
- Deshabilitar administrativamente los puertos no utilizados.
- Guardar la configuración de los equipos y el archivo de simulación.

La implementación demuestra una arquitectura de red empresarial pequeña, funcional y con controles básicos de seguridad verificables.

La evaluación detallada de los resultados se presenta en el siguiente documento: **[06. Pruebas y validación](06-pruebas-validacion.md)**.

---

**Proyecto 01 — Implementación de una Red Empresarial Segura | AndesTech S.A.C.**
