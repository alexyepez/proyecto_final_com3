# Documento Técnico: Diseño de Red Empresarial

## 1. Introducción
Este documento describe el diseño de una red empresarial que cumple con los requerimientos establecidos para el proyecto final de Comunicaciones III, Primer Semestre 2025. El objetivo es implementar una red segura, redundante y escalable que soporte conectividad entre tres edificios, con cuatro VLANs (Contabilidad, Desarrollo, Diseño, Administración de Red), asignación dinámica de direcciones, servicios de correo y archivos, y una VPN sitio a sitio con IPsec. El diseño se implementará en Packet Tracer y en hardware físico, utilizando las direcciones base 172.16.16.0/20 (IPv4) y 2001:dbad:acad::/48 (IPv6).

---

## 2. Diseño de Direccionamiento Jerárquico

### 2.1 Requerimientos
- **Crecimiento del 5%**: Cada VLAN debe incluir un margen de direcciones para un crecimiento del 5% en el número de usuarios.
- **Doble stack**: Soporte para IPv4 e IPv6 en todas las VLANs.
- **Escalabilidad**: Posibilidad de añadir más edificios o VLANs en el futuro.
- **Asignación dinámica**: Uso de DHCP para dispositivos finales y direcciones estáticas para equipos de red y administración.
- **Subnetting IPv6**: Subredes de /64 para cumplir con los estándares de asignación de prefijos.

### 2.2 Topología
- **Router EDGE**: Configurado como Router-on-a-Stick para enrutamiento inter-VLAN. Conectado por interfaz serial al Router ISP. Actúa como servidor DHCP para todas las VLANs.
- **Router ISP**: Conecta Router EDGE y Router RT_SERVICES mediante interfaces seriales.
- **Router RT_SERVICES**: Conectado al servidor de archivos y correo (Samba/Postfix) en la subred 172.16.20.160/28.
- **Servidor**: Máquina virtual Ubuntu con Samba y Postfix, accesible desde todas las VLANs a través de la VPN.
- **VPN**: IPsec sitio a sitio entre Router EDGE y Router RT_SERVICES para comunicación segura.

### 2.3 Direccionamiento IPv4
La dirección base es **172.16.16.0/20**, que proporciona 4096 direcciones (de 172.16.16.0 a 172.16.31.255). Se divide en subredes considerando el número de usuarios por VLAN en cada edificio, más un 5% de crecimiento. A continuación, se detalla el cálculo:

#### Usuarios por VLAN con Crecimiento
| VLAN | Edificio 1 | Edificio 2 | Edificio 3 | Total Usuarios (con 5%) |
|------|------------|------------|------------|-------------------------|
| Contabilidad | 100 (+5) = 105 | 220 (+11) = 231 | 4 (+0.2) = 5 | 341 |
| Desarrollo | 5 (+0.25) = 6 | 120 (+6) = 126 | 105 (+5.25) = 111 | 243 |
| Diseño | 5 (+0.25) = 6 | 20 (+1) = 21 | 120 (+6) = 126 | 153 |
| Adm. Red | 4 (+0.2) = 5 | 4 (+0.2) = 5 | 4 (+0.2) = 5 | 15 |

#### Subnetting IPv4
Se asignan subredes basadas en el número de hosts requeridos, utilizando potencias de 2 para optimizar el espacio de direcciones. Además, se reserva espacio para equipos de red (routers, switches, servidores) y posibles edificios futuros.

| VLAN | Usuarios (con 5%) | Subred | Rango de Direcciones | Gateway | Rango DHCP |
|------|-------------------|--------|----------------------|---------|------------|
| Contabilidad (VLAN 100) | 341 | 172.16.16.0/23 | 172.16.16.0 - 172.16.17.255 | 172.16.16.1 | 172.16.16.10 - 172.16.17.254 |
| Desarrollo (VLAN 200) | 242 | 172.16.18.0/24 | 172.16.18.0 - 172.16.18.255 | 172.16.18.1 | 172.16.18.10 - 172.16.18.254 |
| Diseño (VLAN 300) | 153 | 172.16.19.0/24 | 172.16.19.0 - 172.16.19.255 | 172.16.19.1 | 172.16.19.10 - 172.16.19.254 |
| Adm. Red (VLAN 400) | 13 + 6 equipos | 172.16.20.0/27 | 172.16.20.0 - 172.16.20.31 | 172.16.20.1 | 172.16.20.10 - 172.16.20.19 |
| Servidor | - | 172.16.20.32/29 | 172.16.20.32 - 172.16.20.39 | 172.16.20.33 | - |
| Serial EDGE-ISP | - | 172.16.20.40/30 | 172.16.20.40 - 172.16.20.43 | - | - |
| Serial ISP-RT_SERVICES | - | 172.16.20.44/30 | 172.16.20.44 - 172.16.20.47 | - | - |

**VLAN 400 Asignaciones**:
- 172.16.20.1: Gateway (Router EDGE).
- 172.16.20.2: Router EDGE (loopback).
- 172.16.20.3: Router RT_SERVICES (loopback).
- 172.16.20.4: Router ISP (loopback).
- 172.16.20.5-7: Switches (Edif. 1-3).
- 172.16.20.10-19: Usuarios administradores.
- 172.16.20.20-31: Reservado.

**Servidor Asignaciones**:
- 172.16.20.33: Gateway (Router RT_SERVICES).
- 172.16.20.34: Samba/Postfix.

**Nota**: La subred para equipos de red incluye direcciones estáticas para routers, switches y servidores. El espacio reservado permite añadir más edificios o VLANs.

### 2.3 Direccionamiento IPv6
La dirección base es **2001:dbad:acad::/48**. Se asignan subredes de /64, añadiendo un prefijo de 16 bits para identificar cada VLAN y edificio. Esto proporciona 65,536 subredes posibles.

| VLAN | Subred | Gateway | Asignaciones |
|------|--------|---------|--------------|
| Contabilidad (VLAN 100) | 2001:dbad:acad:1000::/64 | 2001:dbad:acad:1000::1 | SLAAC |
| Desarrollo (VLAN 200) | 2001:dbad:acad:2000::/64 | 2001:dbad:acad:2000::1 | SLAAC |
| Diseño (VLAN 300) | 2001:dbad:acad:3000::/64 | 2001:dbad:acad:3000::1 | SLAAC |
| Adm. Red (VLAN 400) | 2001:dbad:acad:4000::/64 | 2001:dbad:acad:4000::1 | ::2 (Router EDGE), ::3 (Router RT_SERVICES), ::4 (Router ISP), ::5-7 (Switches), ::10-19 (Usuarios) |
| Servidor | 2001:dbad:acad:5000::/64 | 2001:dbad:acad:5000::1 | ::2 (Samba/Postfix) |
| Serial EDGE-ISP | 2001:dbad:acad:6000::/126 | - | ::1 (EDGE), ::2 (ISP) |
| Serial ISP-RT_SERVICES | 2001:dbad:acad:6004::/126 | - | ::1 (ISP), ::2 (RT_SERVICES) |

**Nota**: Los prefijos se asignan jerárquicamente (1000 para Contabilidad, 2000 para Desarrollo, etc.) para facilitar la administración y escalabilidad.

### 2.4 Asignación Dinámica de Direcciones
- **IPv4**: El Router EDGE actúa como servidor DHCP, asignando direcciones dinámicas a dispositivos finales en cada VLAN. Se excluyen las primeras 10 direcciones de cada subred para gateways y equipos estáticos.
  ```plaintext
  ! Excluir direcciones reservadas
  ip dhcp excluded-address 172.16.16.1 172.16.16.9
  ip dhcp excluded-address 172.16.18.1 172.16.18.9
  ip dhcp excluded-address 172.16.19.1 172.16.19.9
  ip dhcp excluded-address 172.16.20.1 172.16.20.9

  ! Configurar pools DHCP para cada VLAN
  ip dhcp pool VLAN100
   network 172.16.16.0 255.255.254.0
   default-router 172.16.16.1
   dns-server 8.8.8.8
  ip dhcp pool VLAN200
   network 172.16.18.0 255.255.255.0
   default-router 172.16.18.1
   dns-server 8.8.8.8
  ip dhcp pool VLAN300
   network 172.16.19.0 255.255.255.0
   default-router 172.16.19.1
   dns-server 8.8.8.8
  ip dhcp pool VLAN400
   network 172.16.20.0 255.255.255.224
   default-router 172.16.20.1
   dns-server 8.8.8.8
  ```
- **IPv6**: Se utiliza SLAAC (Stateless Address Autoconfiguration) para asignar direcciones automáticamente a dispositivos finales, con el router enviando anuncios de prefijo (/64). Para dispositivos de red, se asignan direcciones estáticas en la subred 2001:dbad:acad:5000::/64.
  ```plaintext
  interface GigabitEthernet0/1.100
   encapsulation dot1Q 100
   ip address 172.16.16.1 255.255.254.0
   ipv6 address 2001:dbad:acad:1000::1/64
   ipv6 nd prefix 2001:dbad:acad:1000::/64
   ipv6 nd managed-config-flag
   ipv6 nd other-config-flag
  interface GigabitEthernet0/1.200
   encapsulation dot1Q 200
   ip address 172.16.18.1 255.255.255.0
   ipv6 address 2001:dbad:acad:2000::1/64
   ipv6 nd prefix 2001:dbad:acad:2000::/64
   ipv6 nd managed-config-flag
   ipv6 nd other-config-flag
  interface GigabitEthernet0/1.300
   encapsulation dot1Q 300
   ip address 172.16.19.1 255.255.255.0
   ipv6 address 2001:dbad:acad:3000::1/64
   ipv6 nd prefix 2001:dbad:acad:3000::/64
   ipv6 nd managed-config-flag
   ipv6 nd other-config-flag
  interface GigabitEthernet0/1.400
   encapsulation dot1Q 400
   ip address 172.16.20.1 255.255.255.224
   ipv6 address 2001:dbad:acad:4000::1/64
   ipv6 nd prefix 2001:dbad:acad:4000::/64
   ipv6 nd managed-config-flag
   ipv6 nd other-config-flag
  ```
  
---

## 3. Diseño de Red Física y Redundancia

### 3.1 Topología
La red consta de:
- **Router EDGE**: Conecta la red interna con el exterior (Internet). Configurado con una interfaz Ethernet para comunicación inter-VLAN.
- **Switches de Distribución**: Uno por edificio, conectados al router EDGE mediante enlaces troncales (trunk).
- **Switches de Acceso**: Conectados a los switches de distribución, asignados a las VLANs específicas por edificio.
- **Servidores**: Un servidor virtualizado (físico) en la subred de equipos de red para servicios de correo y archivos.
- **STP**: Rapid Per-VLAN Spanning Tree (PVST) para evitar bucles.

### 3.2 Aumento de Ancho de Banda y Redundancia
- **EtherChannel**: Usaremos LACP (Link Aggregation Control Protocol) para formar un EtherChannel entre los switches, agrupando dos interfaces GigabitEthernet para duplicar el ancho de banda y proporcionar redundancia.
- **Protocolo STP (Spanning Tree Protocol)**: Se activa Rapid Per-VLAN Spanning Tree (PVST) para evitar bucles en la red. Cada VLAN tiene su propia instancia de STP, con el switch de distribución del Edificio 2 como raíz primaria para balanceo de carga.
- **Configuración en Switch Edificio 1 (Conecta a Router EDGE y Switches Edificio 2/3)**:
  ```plaintext
  ! Crear VLANs
  vlan 100
   name CONTABILIDAD
  vlan 200
   name DESARROLLO
  vlan 300
   name DISENO
  vlan 400
   name ADMON_RED

  ! Configurar interfaz de administración
  interface vlan 400
   ip address 172.16.20.5 255.255.255.224
   ipv6 address 2001:dbad:acad:4000::5/64
   no shutdown

  ! Configurar enlace troncal al Router EDGE
  interface GigabitEthernet0/1
   switchport mode trunk
   switchport trunk allowed vlan 100,200,300,400
   no shutdown

  ! Configurar EtherChannel a Switch Edificio 2
  interface range GigabitEthernet0/2 - 0/3
   switchport mode trunk
   switchport trunk allowed vlan 100,200,300,400
   channel-group 1 mode active
   no shutdown
  interface Port-channel1
   switchport mode trunk
   switchport trunk allowed vlan 100,200,300,400
   no shutdown

  ! Configurar EtherChannel a Switch Edificio 3
  interface range GigabitEthernet1/0 - 1/1
   switchport mode trunk
   switchport trunk allowed vlan 100,200,300,400
   channel-group 2 mode active
   no shutdown
  interface Port-channel2
   switchport mode trunk
   switchport trunk allowed vlan 100,200,300,400
   no shutdown

  ! Configurar PVST (Switch Edif. 1 como raíz primaria)
  spanning-tree mode rapid-pvst
  spanning-tree vlan 100,200,300,400 priority 4096
  ```

- **Configuración Switch 2 (Conecta a switch Edificio 1)**:
  ```plaintext
  ! Configurar VLANs
  vlan 100
   name CONTABILIDAD
  vlan 200
   name DESARROLLO
  vlan 300
   name DISENO
  vlan 400
   name ADMON_RED

  ! Configurar interfaz de administración
  interface vlan 400
   ip address 172.16.20.6 255.255.255.224
   ipv6 address 2001:dbad:acad:4000::6/64
   no shutdown

  ! Configurar EtherChannel a Switch Edificio 1
  interface range GigabitEthernet0/1 - 0/2
   switchport mode trunk
   switchport trunk allowed vlan 100,200,300,400
   channel-group 1 mode active
   no shutdown
  interface Port-channel1
   switchport mode trunk
   switchport trunk allowed vlan 100,200,300,400
   no shutdown

  ! Configurar PVST
  spanning-tree mode rapid-pvst
  spanning-tree vlan 100,200,300,400 priority 8192
  ```

- **Configuración en Switch 3 (Conecta Edificio 2)**:
  ```plaintext
  ! Crear VLANs
  vlan 100
   name CONTABILIDAD
  vlan 200
   name DESARROLLO
  vlan 300
   name DISENO
  vlan 400
   name ADMON_RED

  ! Configurar EtherChannel hacia Switch_D1
  interface Port-channel1
   switchport mode trunk
   switchport trunk allowed vlan 100,200,300,400
  interface range GigabitEthernet0/1 - 2
   switchport mode trunk
   switchport trunk allowed vlan 100,200,300,400
   channel-group 1 mode active
   no shutdown

  ! Configurar puertos de acceso (ejemplo para VLAN 100)
  interface GigabitEthernet0/3
   switchport mode access
   switchport access vlan 100
   no shutdown

  ! Configurar Rapid PVST
  spanning-tree mode rapid-pvst
  ```
---

## 4. Seguridad

### 4.1 Control de Acceso Físico
- **Port Security**: Para garantizar que solo los usuarios autorizados accedan físicamente a la red a través de los switches, configuramos Port Security en los puertos de acceso de los switches de cada edificio (Switch_A1, Switch_A2, Switch_A3). Esto limita el acceso a una sola dirección MAC por puerto y desactiva el puerto si se detecta un dispositivo no autorizado.

- **Configuración de Port Security (Switch Edificio 1)**:
  ```plaintext
  ! Configurar Port Security en puertos de acceso (ejemplo para VLAN 100)
  interface GigabitEthernet0/3
   switchport mode access
   switchport access vlan 100
   switchport port-security
   switchport port-security maximum 1
   switchport port-security violation shutdown
   switchport port-security mac-address sticky
   no shutdown

  ! Repetir para otros puertos de acceso (por ejemplo, GigabitEthernet0/4 para VLAN 200, etc.)
  interface GigabitEthernet0/4
   switchport mode access
   switchport access vlan 200
   switchport port-security
   switchport port-security maximum 1
   switchport port-security violation shutdown
   switchport port-security mac-address sticky
   no shutdown
  ```
- **Configuración en Switch_A2 (Edificio 2) y Switch_A3 (Edificio 3)**: Similar a Switch_A1, aplicando Port Security a todos los puertos de acceso conectados a dispositivos finales (PCs) en las VLANs 100, 200, 300, y 400.
- **NOTAS**
  - maximum 1: Permite solo una dirección MAC por puerto.
  - violation shutdown: Desactiva el puerto si se conecta un dispositivo no autorizado.
  - mac-address sticky: Aprende automáticamente la primera dirección MAC que se conecta y la guarda como autorizada.
  - Los puertos troncales (hacia Switch_Dx o Router EDGE) no requieren Port Security, ya que conectan dispositivos de red confiables.

### 4.2 Comunicación Segura
- **VPN Sitio a Sitio con IPsec**: Se configura una VPN entre el router EDGE y un sitio remoto (simulado) para garantizar confidencialidad. Se utiliza IPsec con cifrado AES-256 y autenticación SHA-256.

- **Objetivo de la VPN**:
  - Garantizar confidencialidad para el tráfico entre las VLANs (en la red del Router EDGE) y el servidor Samba/Postfix (en la red del Router RT_SERVICES).
  - Usar IPsec con cifrado AES-256 y autenticación SHA-256.

- **Configuración de la VPN**:
    La VPN IPsec requiere configuraciones espejo en el Router EDGE y el Router RT_SERVICES, aplicadas a sus interfaces seriales (Serial0/0/0 en ambos routers). La ACL define el tráfico a cifrar: desde las VLANs (172.16.16.0/20) hacia la subred del servidor (172.16.20.32/29) y viceversa.

- **Configuración en router EDGE**:
  ```plaintext
  ! Definir política ISAKMP (Fase 1)
  crypto isakmp policy 10
   encryption aes 256
   authentication pre-share
   group 2
  crypto isakmp key SecretKey address 172.16.20.46

  ! Definir transform-set (Fase 2)
  crypto ipsec transform-set TS esp-aes 256 esp-sha-hmac

  ! Definir ACL para tráfico VPN
  ip access-list extended ACL_VPN
   permit ip 172.16.16.0 0.0.15.255 172.16.20.32 0.0.0.7

  ! Configurar crypto map
  crypto map CMAP 10 ipsec-isakmp
   set peer 172.16.20.46
   set transform-set TS
   match address ACL_VPN

  ! Aplicar crypto map a la interfaz serial
  interface Serial0/0/0
   ip address 172.16.20.41 255.255.255.252
   crypto map CMAP
   no shutdown
  ```
- **RT_SERVICES**:
  ```plaintext
  ! Definir política ISAKMP (Fase 1)
  crypto isakmp policy 10
   encryption aes 256
   authentication pre-share
   group 2
  crypto isakmp key SecretKey address 172.16.20.41

  ! Definir transform-set (Fase 2)
  crypto ipsec transform-set TS esp-aes 256 esp-sha-hmac

  ! Definir ACL para tráfico VPN
  ip access-list extended ACL_VPN
   permit ip 172.16.20.32 0.0.0.7 172.16.16.0 0.0.15.255

  ! Configurar crypto map
  crypto map CMAP 10 ipsec-isakmp
   set peer 172.16.20.41
   set transform-set TS
   match address ACL_VPN

  ! Aplicar crypto map a la interfaz serial
  interface Serial0/0/0
   ip address 172.16.20.46 255.255.255.252
   crypto map CMAP
   no shutdown
  ```

### 4.3 Acceso Remoto Seguro
- **SSH**: Los dispositivos de red (routers y switches) se configuran con SSH para administración remota, deshabilitando Telnet.
  ```plaintext
  hostname Router_EDGE
  ip domain-name empresa.com
  crypto key generate rsa
   2048
  line vty 0 4
   transport input ssh
   login local
  username admin privilege 15 secret AdminPass123
  ```

---

## 5. Comunicación Inter-VLAN

### 5.1 Configuración en Router EDGE
Se utiliza **Router-on-a-Stick** con una sola interfaz Ethernet (GigabitEthernet0/1) configurada con subinterfaces para cada VLAN.
```plaintext
interface GigabitEthernet0/1
 no ip address
interface GigabitEthernet0/1.100
 encapsulation dot1Q 100
 ip address 172.16.16.1 255.255.255.128
 ipv6 address 2001:dbad:acad:1000::1/64
interface GigabitEthernet0/1.200
 encapsulation dot1Q 200
 ip address 172.16.18.129 255.255.255.128
 ipv6 address 2001:dbad:acad:2000::1/64
interface GigabitEthernet0/1.300
 encapsulation dot1Q 300
 ip address 172.16.20.1 255.255.255.128
 ipv6 address 2001:dbad:acad:3000::1/64
interface GigabitEthernet0/1.400
 encapsulation dot1Q 400
 ip address 172.16.20.129 255.255.255.248
 ipv6 address 2001:dbad:acad:4000::1/64
```

### 5.2 Enrutamiento
Se configura OSPF para IPv4 y OSPFv3 para IPv6 para enrutamiento dinámico entre VLANs y con el exterior.
**EDGE**:
```plaintext
! OSPF para IPv4
router ospf 1
 network 172.16.16.0 0.0.1.255 area 0  ! VLAN 100
 network 172.16.18.0 0.0.0.255 area 0  ! VLAN 200
 network 172.16.19.0 0.0.0.255 area 0  ! VLAN 300
 network 172.16.20.0 0.0.0.31 area 0   ! VLAN 400
 network 172.16.20.40 0.0.0.3 area 0   ! Serial EDGE-ISP

! OSPFv3 para IPv6
ipv6 router ospf 1
 router-id 1.1.1.1
interface GigabitEthernet0/1.100
 ipv6 ospf 1 area 0
interface GigabitEthernet0/1.200
 ipv6 ospf 1 area 0
interface GigabitEthernet0/1.300
 ipv6 ospf 1 area 0
interface GigabitEthernet0/1.400
 ipv6 ospf 1 area 0
interface Loopback0
 ipv6 ospf 1 area 0
interface Serial0/0/0
 ipv6 ospf 1 area 0
```

**ISP**:
```plaintext
! OSPF para IPv4
router ospf 1
 network 172.16.20.4 0.0.0.0 area 0   ! Loopback
 network 172.16.20.40 0.0.0.3 area 0  ! Serial EDGE-ISP
 network 172.16.20.44 0.0.0.3 area 0  ! Serial ISP-RT_SERVICES

! OSPFv3 para IPv6
ipv6 router ospf 1
 router-id 2.2.2.2
interface Loopback0
 ipv6 ospf 1 area 0
interface Serial0/0/0
 ipv6 ospf .ser 1 area 0
interface Serial0/0/1
 ipv6 ospf 1 area 0
```

**RT_SERVICES**:
```plaintext
! OSPF para IPv4
router ospf 1
 network 172.16.20.3 0.0.0.0 area 0   ! Loopback
 network 172.16.20.32 0.0.0.7 area 0  ! Servidor
 network 172.16.20.44 0.0.0.3 area 0  ! Serial ISP-RT_SERVICES

! OSPFv3 para IPv6
ipv6 router ospf 1
 router-id 3.3.3.3
interface Loopback0
 ipv6 ospf 1 area 0
interface GigabitEthernet0/0
 ipv6 ospf 1 area 0
interface Serial0/0/0
 ipv6 ospf 1 area 0
```
---

## 6. Servicios

### 6.1 Servicio de Correo
- **Implementación**: Se configura un servidor de correo (Postfix en Linux virtualizado) en la subred de equipos de red (172.16.20.161, 2001:dbad:acad:5000::2). Se crean 10 cuentas de usuario accesibles desde todas las VLANs.
- **Acceso**: Los clientes usan IMAP (puerto 143) para acceder al correo. Se configura un firewall para permitir solo tráfico autorizado.

### 6.2 Servicio de Archivos
- **Implementación**: Se configuró un servidor Samba en una máquina virtual existente con **Ubuntu Server** (VMware) en la subred de equipos de red (IPv4: 172.16.20.161, IPv6: 2001:dbad:acad:5000::2). El servidor proporciona tres recursos compartidos con políticas de acceso diferenciadas:
  - **Perfil 1 (Contabilidad)**: Acceso de solo lectura, restringido al grupo `contabilidad`.
  - **Perfil 2 (Desarrollo)**: Acceso de lectura/escritura, restringido al grupo `desarrollo`.
  - **Perfil 3 (Administración)**: Acceso completo (lectura/escritura con permisos totales), restringido al grupo `admon`.
- **Configuración Samba**:
  ```ini
  [Contabilidad]
     path = /srv/samba/contabilidad
     read only = yes
     valid users = @contabilidad
     browsable = yes

  [Desarrollo]
     path = /srv/samba/desarrollo
     read only = no
     valid users = @desarrollo
     browsable = yes

  [Admon]
     path = /srv/samba/admon
     read only = no
     create mask = 0777
     directory mask = 0777
     valid users = @admon
     browsable = yes
  ```
- **Detalles Técnicos**:
  - **Sistema Operativo**: Ubuntu Server, configurado en una máquina virtual VMware.
  - **Usuarios y Grupos**: Se crearon grupos (`contabilidad`, `desarrollo`, `admon`) y usuarios asociados, con contraseñas Samba para autenticación.
  - **Permisos**: Los directorios `/srv/samba/*` tienen permisos ajustados (750 para Contabilidad, 770 para Desarrollo y Admon) para reflejar las políticas de acceso.
  - **Red**: La máquina virtual está configurada con direcciones estáticas (172.16.20.161/28 y 2001:dbad:acad:5000::2/64) y es accesible desde todas las VLANs gracias al enrutamiento inter-VLAN.
  - **Firewall**: Se configuró `ufw` para permitir tráfico SMB/CIFS (puertos 137, 138, 139, 445).
- **Acceso**: Los clientes en las VLANs acceden a los recursos compartidos mediante SMB (e.g., `\\172.16.20.161\Contabilidad` en Windows o `smbclient` en Linux). Soporte para IPv6 está habilitado.
- **Validación**: Se verificó que los usuarios de cada grupo tienen los permisos correctos (solo lectura para Contabilidad, lectura/escritura para Desarrollo, control total para Admon).

## 7. Implementación y Validación

### 7.1 Implementación en Packet Tracer
- Se crea la topología con routers, switches y PCs según la descripción.
- Se configuran VLANs, EtherChannel, STP, DHCP, SSH, VPN, y servicios.
- Se simulan pruebas de conectividad entre VLANs, acceso a servicios, y comunicación segura.

### 7.2 Implementación Física
- Se utilizarán routers y switches Cisco en el laboratorio.
- Los servicios de correo y archivos se desplegarán en una máquina virtual (VMware o VirtualBox) conectada a la red física.
- Se probará la conectividad y seguridad durante la clase del 19 de mayo de 2025.

### 7.3 Validación
- **Direccionamiento**: Verificación de asignación de direcciones dinámicas y estáticas.
- **Conectividad**: Pruebas de ping entre VLANs y edificios.
- **Servicios**: Acceso a correo y archivos desde diferentes VLANs.
- **Seguridad**: Pruebas de port security y VPN.
- **Administración**: Acceso remoto seguro via SSH.

---

## 8. Decisiones de Diseño
- **EtherChannel**: Elegido para aumentar el ancho de banda y proporcionar redundancia sin depender de hardware adicional.
- **PVST**: Seleccionado para optimizar la redundancia por VLAN y evitar bucles.
- **OSPF/OSPFv3**: Usado para enrutamiento dinámico por su escalabilidad y soporte en doble stack.
- **Samba para Archivos**: Escogido por su compatibilidad con múltiples plataformas y facilidad de configuración de perfiles.
- **Postfix para Correo**: Seleccionado por su robustez y soporte para múltiples usuarios.
- **IPsec con AES-256**: Implementado para garantizar confidencialidad en la VPN, cumpliendo con estándares de seguridad modernos.

---

## 9. Conclusiones
El diseño propuesto cumple con todos los requerimientos del proyecto, proporcionando una red escalable, segura y redundante. La implementación en Packet Tracer y en hardware físico permitirá validar el diseño en entornos simulados y reales. El uso de protocolos estándares como OSPF, STP y IPsec asegura compatibilidad y estabilidad.

---

## 10. Anexos
- **Topología Packet Tracer**: Archivo .pkt disponible para revisión.
- **Configuraciones Completas**: Scripts de configuración para routers y switches.
- **Guías de Implementación**: Documentación adicional para servidores Postfix y Samba.
