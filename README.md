# Infraestructura 1 – Seguridad de Redes

**Estudiante:** Merolyn Mejía  
**Matrícula:** 2025-0827  
**Asignatura:** Seguridad de Redes  
**Práctica:** Infraestructura 1  

---

## 🎥 Video demostrativo

▶️ **Video de demostración:**  
[https://youtu.be/D94mXhJJL-g]

> En el video demuestro el funcionamiento de los principales controles de seguridad implementados en la infraestructura, incluyendo segmentación mediante VLAN, DMZ, control de acceso SSH, restricciones entre redes y registros de violaciones de políticas.

---

# 1. Descripción del laboratorio

En esta práctica se diseñó y configuró una infraestructura de red utilizando **GNS3**, un firewall **FortiGate**, un switch Cisco, dos equipos de usuarios y tres servidores.

El objetivo principal fue crear una red segmentada y aplicar controles de seguridad que permitan controlar qué dispositivos pueden comunicarse entre sí.

La infraestructura fue dividida en:

- VLAN 10 – Usuarios
- VLAN 20 – Usuarios autorizados para SSH
- DMZ – Servidores
- WAN – Salida hacia Internet

El FortiGate funciona como el dispositivo principal de seguridad, controlando la comunicación entre las diferentes redes mediante políticas de firewall.

---

# 2. Propósito del laboratorio

El propósito de este laboratorio es aplicar de manera práctica conceptos de seguridad de redes como:

- Segmentación mediante VLAN.
- Separación de servidores mediante una DMZ.
- Control del tráfico mediante políticas de firewall.
- Restricción del acceso SSH.
- Restricción del acceso de usuarios a determinados servidores.
- Bloqueo del tráfico iniciado desde la DMZ hacia las redes internas.
- Restricción del acceso a Internet de los servidores.
- Registro de violaciones de políticas.
- Configuración básica de seguridad en un switch.
- Uso de Port Security.
- Uso de BPDU Guard.
- Administración segura mediante SSH.

---

# 3. Topología

La infraestructura utilizada fue la siguiente:

                    INTERNET / NAT
                          |
                      FortiGate
                 port1 | port2 | port3
                   WAN | TRUNK | DMZ
                       |       |
                 Cisco IOSvL2  Switch
                   /      \      |
              VLAN 10   VLAN 20  |
              Alpine    Windows  |
                                / | \
                            Caja Inventario DB

### Equipos utilizados

- 1 FortiGate
- 1 Cisco IOSvL2
- 1 switch Ethernet para la DMZ
- 1 cliente Alpine Linux
- 1 cliente Windows
- 2 servidores web Ubuntu
- 1 servidor de base de datos Ubuntu
- NAT de GNS3 para conexión a Internet

---

# 4. Plan de direccionamiento IP

| Red / Equipo | Dirección |
|---|---|
| VLAN 10 | 10.8.27.0/25 |
| Gateway VLAN 10 | 10.8.27.1 |
| Rango DHCP VLAN 10 | 10.8.27.2 - 10.8.27.126 |
| Cliente Alpine | 10.8.27.3 |
| VLAN 20 | 10.8.27.128/25 |
| Gateway VLAN 20 | 10.8.27.129 |
| Rango DHCP VLAN 20 | 10.8.27.130 - 10.8.27.254 |
| Cliente Windows | 10.8.27.130 |
| DMZ | 172.8.27.0/28 |
| Gateway DMZ | 172.8.27.1 |
| Web Server Caja | 172.8.27.2 |
| Web Server Inventario | 172.8.27.3 |
| DB Server | 172.8.27.4 |

El direccionamiento fue realizado tomando como referencia la matrícula **2025-0827**.

---

# 5. Configuración del FortiGate

Toda la configuración correspondiente al FortiGate fue realizada mediante su **interfaz gráfica (GUI)**.

Se utilizaron tres interfaces principales:

### port1 – WAN

Esta interfaz proporciona la salida hacia Internet utilizando el NAT de GNS3.

La dirección fue obtenida mediante DHCP.

### port2 – Trunk

Esta interfaz conecta el FortiGate con el switch Cisco.

Sobre esta interfaz se configuraron las VLAN:

**VLAN 10**

- Nombre: `VLAN10-USUARIOS`
- VLAN ID: 10
- Dirección: `10.8.27.1/25`

**VLAN 20**

- Nombre: `VLAN20-USUARIOS`
- VLAN ID: 20
- Dirección: `10.8.27.129/25`

### port3 – DMZ

La interfaz port3 fue utilizada para la red de servidores.

- Alias: `DMZ-SERVIDORES`
- Dirección: `172.8.27.1/28`

Los servidores utilizan direcciones IP estáticas.

---

# 6. DHCP

Se configuró DHCP para las dos VLAN de usuarios.

### VLAN 10

Gateway:

    10.8.27.1

Rango:

    10.8.27.2 - 10.8.27.126

### VLAN 20

Gateway:

    10.8.27.129

Rango:

    10.8.27.130 - 10.8.27.254

Como servidores DNS se utilizaron:

    8.8.8.8
    1.1.1.1

Las pruebas confirmaron que los clientes recibieron correctamente sus configuraciones mediante DHCP.

---

# 7. Configuración de la DMZ

Los tres servidores fueron colocados dentro de una red DMZ independiente:

    172.8.27.0/28

La puerta de enlace de los servidores es:

    172.8.27.1

Los servidores configurados fueron:

### Web Server – Sistema de Caja

IP:

    172.8.27.2

Servicio principal:

    Apache

### Web Server – Sistema de Inventario

IP:

    172.8.27.3

Servicio principal:

    Apache

### Database Server

IP:

    172.8.27.4

Servicio principal:

    MySQL

Esta separación permite mantener los servidores fuera de las redes internas de usuarios y controlar su comunicación mediante el FortiGate.

---

# 8. Servidores web

Se configuró Apache en los dos servidores web.

En el servidor:

    172.8.27.2

se creó una página correspondiente al:

**Sistema de Caja**

En:

    172.8.27.3

se creó una página correspondiente al:

**Sistema de Inventario**

Durante las pruebas se comprobó que Apache se encontraba activo y funcionando correctamente.

---

# 9. Servidor de base de datos

En el servidor:

    172.8.27.4

se configuró **MySQL Server**.

Se creó la base de datos:

    sistema_empresa

También se configuró un usuario para permitir la comunicación de los servidores web con la base de datos.

Las pruebas realizadas desde ambos servidores web confirmaron que podían conectarse correctamente al servidor MySQL.

---

# 10. Objetos de direcciones del FortiGate

Para facilitar la administración de las políticas se crearon objetos para representar las diferentes redes y servidores.

Entre los objetos utilizados se encuentran:

    VLAN10-USUARIOS
    VLAN20-USUARIOS
    DMZ-SERVIDORES
    WEB-CAJA
    WEB-INVENTARIO
    DB-SERVER
    DNS-GOOGLE
    DNS-CLOUDFLARE
    UBUNTU-ARCHIVE
    UBUNTU-SECURITY

Esto permite utilizar nombres descriptivos dentro de las políticas en lugar de trabajar solamente con direcciones IP.

---

# 11. Políticas de seguridad

Se crearon diferentes políticas en FortiGate para controlar el tráfico entre las redes.

## VLAN 20 → SSH → DMZ

Política:

    VLAN20-SSH-DMZ

Esta política permite que únicamente los usuarios pertenecientes a la VLAN 20 puedan utilizar SSH hacia los servidores de la DMZ.

Origen:

    VLAN20-USUARIOS

Destino:

    WEB-CAJA
    WEB-INVENTARIO
    DB-SERVER

Servicio:

    SSH

Acción:

    ACCEPT

La prueba realizada desde Windows confirmó que la conexión SSH hacia el servidor de base de datos era permitida.

---

# 12. Bloqueo de SSH desde VLAN 10

Política:

    BLOQUEO-VLAN10-SSH

La VLAN 10 no está autorizada para administrar los servidores mediante SSH.

Origen:

    VLAN10-USUARIOS

Destino:

    Servidores DMZ

Servicio:

    SSH

Acción:

    DENY

Desde el cliente Alpine se realizó una prueba intentando establecer una conexión SSH hacia:

    172.8.27.4

La conexión fue bloqueada.

En los registros del FortiGate se observó:

    Deny: policy violation

Esto confirmó que la política estaba funcionando correctamente.

---

# 13. Bloqueo del Sistema de Inventario para VLAN 10

Se creó la política:

    BLOQUEO-VLAN10-INVENTARIO

Esta política evita que los usuarios pertenecientes a VLAN 10 puedan acceder al servidor web del Sistema de Inventario.

Origen:

    VLAN10-USUARIOS

Destino:

    WEB-INVENTARIO

Servicio:

    HTTP

Acción:

    DENY

Desde Alpine se realizó la prueba:

    wget -S -O- http://172.8.27.3

El acceso fue bloqueado.

El FortiGate registró nuevamente el evento como:

    Deny: policy violation

De esta forma se pudo comprobar visualmente la violación de la política solicitada en la práctica.

---

# 14. Protección de las redes internas

Los servidores ubicados en la DMZ no deben poder iniciar conexiones libremente hacia las redes internas.

Para esto se implementaron dos políticas.

### DMZ → VLAN 10

Política:

    BLOQUEO-DMZ-VLAN10

Origen:

    DMZ-SERVIDORES

Destino:

    VLAN10-USUARIOS

Servicio:

    ALL

Acción:

    DENY

La prueba se realizó desde el servidor de Caja:

    ping -c 4 10.8.27.3

Resultado:

    100% packet loss

El FortiGate registró el intento como tráfico denegado.

### DMZ → VLAN 20

Política:

    BLOQUEO-DMZ-VLAN20

Origen:

    DMZ-SERVIDORES

Destino:

    VLAN20-USUARIOS

Servicio:

    ALL

Acción:

    DENY

La prueba utilizada fue:

    ping -c 4 10.8.27.130

Resultado:

    100% packet loss

El intento también quedó registrado como una violación de política.

---

# 15. Restricción de Internet para la DMZ

Los servidores de la DMZ no poseen acceso abierto hacia Internet.

Solamente se permitió el tráfico necesario para servicios específicos.

Se configuró acceso DNS hacia:

    8.8.8.8
    1.1.1.1

También se permitió comunicación con los repositorios utilizados para las actualizaciones de Ubuntu:

    archive.ubuntu.com
    security.ubuntu.com

La política utilizada fue:

    DMZ-UBUNTU-UPDATES

Servicios permitidos:

    HTTP
    HTTPS

Se realizó:

    sudo apt update

La actualización de los repositorios funcionó correctamente.

Esto demuestra que los servidores pueden realizar las comunicaciones necesarias para sus actualizaciones sin tener una política general de acceso libre hacia Internet.

---

# 16. Configuración de VLAN en el switch

En el Cisco IOSvL2 se configuraron:

    VLAN 10
    VLAN 20

El puerto:

    GigabitEthernet0/0

fue configurado como trunk y solamente permite las VLAN:

    10,20

Configuración principal:

    interface GigabitEthernet0/0
     switchport trunk allowed vlan 10,20
     switchport trunk encapsulation dot1q
     switchport mode trunk

---

# 17. Puerto de VLAN 10

El puerto:

    GigabitEthernet0/1

fue configurado como puerto de acceso para VLAN 10.

Configuración:

    interface GigabitEthernet0/1
     description USUARIO-VLAN10
     switchport access vlan 10
     switchport mode access
     switchport port-security violation restrict
     switchport port-security mac-address sticky
     switchport port-security
     spanning-tree portfast edge
     spanning-tree bpduguard enable

---

# 18. Puerto de VLAN 20

El puerto:

    GigabitEthernet0/2

fue configurado como puerto de acceso para VLAN 20.

Configuración:

    interface GigabitEthernet0/2
     description USUARIO-VLAN20
     switchport access vlan 20
     switchport mode access
     switchport port-security violation restrict
     switchport port-security mac-address sticky
     switchport port-security
     spanning-tree portfast edge
     spanning-tree bpduguard enable

---

# 19. Port Security

Se implementó Port Security en los puertos de usuarios.

Se utilizó:

    switchport port-security
    switchport port-security mac-address sticky
    switchport port-security violation restrict

De esta forma, el switch aprende la dirección MAC conectada al puerto y limita la conexión de dispositivos no autorizados.

En las verificaciones realizadas, Port Security se mostró:

    Enabled
    Secure-up
    Restrict

También se confirmó que cada puerto tenía una dirección MAC sticky aprendida.

---

# 20. PortFast y BPDU Guard

Los puertos de acceso también fueron protegidos mediante:

    spanning-tree portfast edge
    spanning-tree bpduguard enable

PortFast permite que los puertos destinados a equipos finales entren rápidamente en estado de reenvío.

BPDU Guard agrega protección ante la conexión no autorizada de dispositivos que envíen BPDUs en estos puertos.

---

# 21. Puertos no utilizados

Como medida adicional de seguridad, los puertos del switch que no forman parte de la infraestructura fueron deshabilitados.

Se utilizó:

    shutdown

y se agregó la descripción:

    PUERTO-NO-UTILIZADO

Esto reduce la posibilidad de que un dispositivo sea conectado a un puerto que no debería estar disponible.

---

# 22. Administración segura del switch

También se configuraron medidas básicas para proteger la administración del switch.

Se agregó un banner de acceso restringido:

    ACCESO RESTRINGIDO - SOLO PERSONAL AUTORIZADO

Se configuró autenticación local para las líneas VTY y solamente se permitió SSH:

    line vty 0 4
     login local
     transport input ssh

También se configuró:

    ip domain-name infra1.local
    ip ssh version 2

y se generaron claves RSA de 2048 bits.

De esta manera, la administración remota del switch utiliza SSH en lugar de protocolos de administración sin cifrado como Telnet.

> Por seguridad, las contraseñas y credenciales utilizadas en el laboratorio no se publican en este repositorio.

---

# 23. Verificaciones realizadas

Durante el laboratorio se realizaron diferentes pruebas para comprobar el funcionamiento de la infraestructura.

| Prueba | Resultado |
|---|---|
| DHCP VLAN 10 | Correcto |
| DHCP VLAN 20 | Correcto |
| Apache Sistema de Caja | Activo |
| Apache Sistema de Inventario | Activo |
| MySQL DB Server | Activo |
| Web Server → DB | Permitido |
| VLAN 20 → SSH → DMZ | Permitido |
| VLAN 10 → SSH → DMZ | Bloqueado |
| VLAN 10 → Inventario | Bloqueado |
| DMZ → VLAN 10 | Bloqueado |
| DMZ → VLAN 20 | Bloqueado |
| DMZ → repositorios Ubuntu | Permitido |
| Port Security | Activo |
| BPDU Guard | Activo |
| Puertos no utilizados | Deshabilitados |

---

# 24. Evidencias

Las evidencias del laboratorio se encuentran en la carpeta:

    /Evidencias

Se incluyen capturas de:

1. Topología completa.
2. Interfaces del FortiGate.
3. DHCP VLAN 10.
4. DHCP VLAN 20.
5. Web Server Caja activo.
6. Web Server Inventario activo.
7. DB Server activo.
8. Políticas del FortiGate.
9. SSH permitido desde VLAN 20.
10. Logs de las políticas de seguridad.
11. Actualizaciones permitidas desde la DMZ.
12. VLAN y trunk del switch.
13. Port Security.
14. Estado de las VLAN del switch.

---

# 25. Estructura del repositorio

    Infraestructura-1-Seguridad-de-Redes/
    │
    ├── README.md
    │
    ├── Evidencias/
    │   ├── 01_Topologia_Completa.png
    │   ├── 02_Interfaces_FortiGate.png
    │   ├── 03_DHCP_VLAN10.png
    │   ├── 04_DHCP_VLAN20.png
    │   ├── 05_Server_Caja_Activo.png
    │   ├── 06_Server_Inventario.png
    │   ├── 07_DB_Server_Activo.png
    │   ├── 08_Politicas_FortiGate.png
    │   ├── 09_VLAN20_SSH_Permitido.png
    │   ├── 10_Logs_Politicas_Seguridad.png
    │   ├── 11_DMZ_Actualizaciones.png
    │   ├── 12_Switch_VLANs_Trunk.png
    │   ├── 13_Switch_PortSecurity.png
    │   └── 14_Switch_VLANs.png
    │
    ├── Diagramas/
    │   └── Diagrama_Infraestructura1.png
    │
    ├── Running-Configs/
    │   └── Switch_Running-Config.txt
    │
    ├── Scripts/
    │
    └── Documentacion/
        └── Documentacion_Infraestructura1.pdf

---

# 26. Conclusión

Con esta práctica se logró crear una infraestructura segmentada y aplicar diferentes controles de seguridad para proteger tanto a los usuarios como a los servidores.

La división entre VLAN 10, VLAN 20 y la DMZ permitió controlar mejor qué tipo de comunicación puede realizar cada equipo. Las políticas del FortiGate permitieron bloquear accesos no autorizados, limitar el acceso de los servidores hacia Internet y evitar que los equipos de la DMZ puedan iniciar conexiones hacia las redes internas.

También se aplicaron medidas de seguridad en el switch, como Port Security, BPDU Guard, desactivación de puertos no utilizados y administración mediante SSH.

Finalmente, las pruebas realizadas permitieron comprobar que las políticas funcionan de acuerdo con los objetivos establecidos para el laboratorio.
