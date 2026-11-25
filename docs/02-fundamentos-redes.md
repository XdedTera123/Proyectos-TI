# Fase 02 — Fundamentos de Redes

## 1. Direccionamiento IPv4

Una dirección IPv4 está compuesta por 32 bits, divididos en cuatro octetos. La máscara de subred determina qué parte de la dirección identifica la red y qué parte identifica al host.

### Notación CIDR

La notación CIDR representa la longitud del prefijo de red.

Ejemplo:

- Dirección: `192.168.10.100/24`
- Máscara: `255.255.255.0`
- Dirección de red: `192.168.10.0`
- Broadcast: `192.168.10.255`
- Hosts utilizables: 254

## 2. Plan de direccionamiento propuesto

| VLAN | Departamento | Red | Gateway |
|---|---|---|---|
| 10 | Administración | 192.168.10.0/24 | 192.168.10.1 |
| 20 | Finanzas | 192.168.20.0/24 | 192.168.20.1 |
| 30 | TI | 192.168.30.0/24 | 192.168.30.1 |
| 40 | Servidores | 192.168.40.0/24 | 192.168.40.1 |

## 3. Criterios de diseño

- Cada departamento utilizará una subred IPv4 independiente.
- Las VLAN de usuarios tendrán asignación dinámica mediante DHCP.
- Los gateways utilizarán la primera dirección utilizable de cada subred.
- El servidor interno utilizará la dirección estática 192.168.40.10.
- Las redes se comunicarán mediante enrutamiento inter-VLAN, sujeto a las políticas de seguridad que se definirán posteriormente.

## 4. Estado

Diseño preliminar de direccionamiento IPv4. Pendiente de implementación y validación en Cisco Packet Tracer.

## 5. Segmentación mediante VLAN

Una VLAN (Virtual Local Area Network) permite dividir lógicamente una infraestructura de conmutación en dominios de broadcast independientes.

En AndesTech S.A.C. se implementarán cuatro VLAN para separar los departamentos y los servicios internos.

### 5.1 Distribución de VLAN

| VLAN ID | Nombre | Propósito |
|---|---|---|
| 10 | ADMINISTRACION | Equipos del área administrativa |
| 20 | FINANZAS | Equipos del área financiera |
| 30 | TI | Equipos del personal de tecnología |
| 40 | SERVIDORES | Servicios internos empresariales |

### 5.2 Puertos de acceso

Los puertos Access conectarán los dispositivos finales a sus VLAN correspondientes.

| Switch | Interfaz | Dispositivo | VLAN |
|---|---|---|---|
| SW1-CORE | Fa0/1 | PC-ADMIN | 10 |
| SW1-CORE | Fa0/2 | PC-FIN | 20 |
| SW1-CORE | Fa0/3 | PC-IT | 30 |
| SW1-CORE | Fa0/4 | SRV-INTRANET | 40 |

### 5.3 Enlace troncal

Se utilizará un enlace Trunk IEEE 802.1Q entre el switch SW1-CORE y el router R1-CORE.

- Puerto del switch: G0/1.
- Puerto del router: G0/0.
- VLAN permitidas previstas: 10, 20, 30 y 40.
- Función: transportar el tráfico de las VLAN hacia el dispositivo encargado del enrutamiento.

### 5.4 Enrutamiento inter-VLAN

Se utilizará la arquitectura Router-on-a-Stick para permitir la comunicación controlada entre subredes.

El router tendrá una subinterfaz lógica por VLAN, con la dirección de gateway correspondiente.

### 5.5 Consideraciones de seguridad

La segmentación VLAN separará los dominios de broadcast, pero no sustituirá los controles de acceso entre redes.

Se implementarán listas ACL en el router para restringir las comunicaciones entre departamentos y permitir únicamente los servicios autorizados.

## 6. Estado del diseño

La distribución de VLAN y los puertos de acceso se encuentran definidos a nivel lógico.

La configuración en Cisco Packet Tracer y su validación permanecen pendientes.

## 7. Topología lógica propuesta

La infraestructura de AndesTech S.A.C. estará compuesta por un router principal, un switch administrable, tres estaciones de trabajo y un servidor interno.

La segmentación lógica se realizará mediante cuatro VLAN:

- VLAN 10: Administración
- VLAN 20: Finanzas
- VLAN 30: TI
- VLAN 40: Servidores

El enrutamiento entre VLAN se implementará mediante Router-on-a-Stick, utilizando un enlace troncal IEEE 802.1Q entre el switch SW1-CORE y el router R1-CORE.

### Imagen de topología lógica

> Pendiente de elaboración e incorporación en la siguiente etapa del proyecto.