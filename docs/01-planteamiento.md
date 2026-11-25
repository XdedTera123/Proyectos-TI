# Proyecto 01 — Implementación de una Red Empresarial Segura

## 1. Información general

**Empresa:** AndesTech S.A.C. (ficticia)

**Área:** Infraestructura de Tecnologías de la Información

**Tipo de proyecto:** Diseño, implementación y validación de una red empresarial.

**Entorno:** Laboratorio simulado mediante Cisco Packet Tracer.

**Estado:** Fase de análisis y planificación.

## 2. Contexto empresarial

AndesTech S.A.C. es una empresa ficticia que cuenta con tres departamentos:

- Administración
- Finanzas
- Tecnologías de la Información (TI)

Para este caso práctico se supone que los departamentos comparten una infraestructura de red sin segmentación lógica ni controles de acceso definidos.

La empresa requiere reorganizar su red interna para mejorar la administración, separar los departamentos y controlar el acceso a sus servicios.

## 3. Problemática identificada

La situación inicial presenta los siguientes problemas:

1. **Ausencia de segmentación:** los dispositivos comparten una misma red lógica.
2. **Control de acceso insuficiente:** no existen políticas definidas para restringir comunicaciones entre departamentos.
3. **Administración del direccionamiento:** no existe un esquema formal de direccionamiento IPv4.
4. **Servicios no centralizados:** se requiere establecer servicios internos de asignación de direcciones, resolución de nombres y acceso web.
5. **Falta de documentación:** la infraestructura no cuenta con un diseño ni procedimientos de verificación documentados.

Estos problemas constituyen las condiciones hipotéticas del caso y serán abordados durante el laboratorio.

## 4. Objetivo general

Diseñar, implementar y validar una infraestructura de red empresarial segmentada, utilizando tecnologías de conmutación, enrutamiento, servicios de red y controles de acceso.

## 5. Objetivos específicos

- Diseñar un esquema de direccionamiento IPv4.
- Implementar redes VLAN independientes para cada departamento.
- Configurar enlaces troncales mediante IEEE 802.1Q.
- Implementar enrutamiento entre VLAN.
- Automatizar la asignación de direcciones IP mediante DHCP.
- Configurar servicios internos DNS y HTTP.
- Implementar políticas de seguridad mediante ACL.
- Verificar la conectividad y las restricciones de acceso.
- Documentar las configuraciones y resultados obtenidos.

## 6. Requerimientos funcionales

| ID | Requerimiento | Prioridad |
|---|---|---|
| RF-01 | Separar lógicamente Administración, Finanzas y TI. | Alta |
| RF-02 | Incorporar una red independiente para los servidores. | Alta |
| RF-03 | Proporcionar direccionamiento IPv4 automático a las estaciones de trabajo. | Alta |
| RF-04 | Permitir comunicación entre VLAN cuando esté autorizada. | Alta |
| RF-05 | Restringir las comunicaciones entre departamentos según las políticas establecidas. | Alta |
| RF-06 | Proporcionar resolución de nombres mediante DNS interno. | Media |
| RF-07 | Permitir acceso autorizado a un servidor web interno. | Media |
| RF-08 | Verificar y documentar el funcionamiento de la infraestructura. | Alta |

## 7. Requerimientos no funcionales

| ID | Requerimiento |
|---|---|
| RNF-01 | La infraestructura debe contar con documentación reproducible. |
| RNF-02 | Las configuraciones deben permitir identificar fácilmente los dispositivos y sus funciones. |
| RNF-03 | Las restricciones de acceso deben poder verificarse mediante pruebas. |
| RNF-04 | La infraestructura debe utilizar un direccionamiento organizado y documentado. |
| RNF-05 | Los cambios de configuración deben registrarse mediante control de versiones. |

## 8. Alcance del proyecto

El laboratorio utilizará inicialmente:

- 1 router Cisco.
- 1 switch administrable Cisco.
- 3 computadoras, una por departamento.
- 1 servidor interno.
- 4 redes VLAN.
- Servicios DHCP, DNS y HTTP.
- Listas de control de acceso (ACL).

El proyecto se desarrollará en Cisco Packet Tracer.

No contempla, en esta primera versión, infraestructura física, conexión real a Internet, alta disponibilidad ni implementación en la nube.

## 9. Criterios de aceptación

La implementación se considerará satisfactoria cuando:

1. Cada equipo pertenezca a la VLAN definida para su departamento.
2. Las computadoras reciban correctamente su configuración IPv4.
3. Todos los equipos puedan comunicarse con sus puertas de enlace correspondientes.
4. Los usuarios autorizados puedan acceder a los servicios DNS y HTTP internos.
5. Las comunicaciones restringidas entre departamentos sean bloqueadas.
6. Las configuraciones y pruebas queden documentadas en el repositorio.

## 10. Entregables

- Documento de requerimientos.
- Diseño lógico y físico de la red.
- Tabla de direccionamiento IPv4.
- Archivo de Cisco Packet Tracer.
- Configuraciones de los dispositivos.
- Matriz de políticas de acceso.
- Evidencias de pruebas.
- Registro de problemas y soluciones.
- Documentación final en GitHub.

## 11. Estado actual

**Fase 01 — Análisis y planificación.**

Los requerimientos descritos corresponden al diseño propuesto. Su implementación y validación quedan pendientes para las siguientes etapas.