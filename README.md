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
   ip default-gateway 172.16.20.1

  ! Configurar EtherChannel a Switch 2 (Port-channel 1)
  interface range GigabitEthernet0/1 - 2
   switchport mode trunk
   switchport trunk allowed vlan 100,200,300,400
   channel-group 1 mode active
  interface Port-channel1
   switchport mode trunk
   switchport trunk allowed vlan 100,200,300,400

  ! Configurar EtherChannel a Switch 3 (Port-channel 2)
  interface range GigabitEthernet1/0 - 1
   switchport mode trunk
   switchport trunk allowed vlan 100,200,300,400
   channel-group 2 mode active
  interface Port-channel2
   switchport mode trunk
   switchport trunk allowed vlan 100,200,300,400

  ! Configurar enlace troncal a Router EDGE
  interface GigabitEthernet2/0
   switchport mode trunk
   switchport trunk allowed vlan 100,200,300,400

  ! Configurar Rapid PVST (Switch 1 como raíz primaria)
  spanning-tree mode rapid-pvst
  spanning-tree vlan 100,200,300,400 priority 4096
  ```

- **Configuración Switch 2 (Conecta a Switches Edificio 1/3)**:
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
   ip default-gateway 172.16.20.1

  ! Configurar EtherChannel a Switch 1 (Port-channel 1)
  interface range GigabitEthernet0/1 - 2
   switchport mode trunk
   switchport trunk allowed vlan 100,200,300,400
   channel-group 1 mode active
  interface Port-channel1
   switchport mode trunk
   switchport trunk allowed vlan 100,200,300,400

  ! Configurar EtherChannel a Switch 3 (Port-channel 3)
  interface range GigabitEthernet1/0 - 1
   switchport mode trunk
   switchport trunk allowed vlan 100,200,300,400
   channel-group 3 mode active
  interface Port-channel3
   switchport mode trunk
   switchport trunk allowed vlan 100,200,300,400

  ! Configurar Rapid PVST
  spanning-tree mode rapid-pvst
  spanning-tree vlan 100,200,300,400 priority 8192
  ```

- **Configuración en Switch 3 (Conecta a Switches Edificio 1/2)**:
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
   ip address 172.16.20.7 255.255.255.224
   ipv6 address 2001:dbad:acad:4000::7/64
   ip default-gateway 172.16.20.1

  ! Configurar EtherChannel a Switch 1 (Port-channel 2)
  interface range GigabitEthernet0/1 - 2
   switchport mode trunk
   switchport trunk allowed vlan 100,200,300,400
   channel-group 2 mode active
  interface Port-channel2
   switchport mode trunk
   switchport trunk allowed vlan 100,200,300,400

  ! Configurar EtherChannel a Switch 2 (Port-channel 3)
  interface range GigabitEthernet1/0 - 1
   switchport mode trunk
   switchport trunk allowed vlan 100,200,300,400
   channel-group 3 mode active
  interface Port-channel3
   switchport mode trunk
   switchport trunk allowed vlan 100,200,300,400

  ! Configurar Rapid PVST
  spanning-tree mode rapid-pvst
  spanning-tree vlan 100,200,300,400 priority 12288
  ```
- **NOTAS**:
  - EtherChannel: Usa LACP (mode active) para negociar los enlaces. Cada Port-channel es un tronco que permite todas las VLANs.
  - Switch 1 es la raíz primaria (prioridad 4096), Switch 2 secundaria (8192), y Switch 3 terciaria (12288) para balanceo y redundancia.
  - Interfaz de Administración: Cada switch tiene una dirección en VLAN 400 (172.16.20.5-7) para gestión remota.
---

## 4. Seguridad

### 4.1 Control de Acceso Físico
- **Port Security**: El control de acceso físico se implementa con Port Security en los puertos de acceso de los switches (donde se conectan los dispositivos finales, como PCs). Esto garantiza que solo dispositivos autorizados (por dirección MAC) puedan conectarse, y los puertos se deshabilitarán ante intentos no autorizados.
- **Configuración Genérica para Puertos de Acceso (en cada switch)**:
  - Se aplica a los puertos de acceso (por ejemplo, FastEthernet0/23-24, asumiendo que los puertos 0/23-24/ están usados para EtherChannel).

- **Configuración de Port Security (Switch Edificio 1)**:
  ```plaintext
  ! Configurar Port Security en puertos de acceso
  interface range FastEthernet1/1 - 24
   switchport mode access
   switchport access vlan 100  ! Cambiar a 200, 300, o 400 según el puerto
   switchport port-security
   switchport port-security maximum 1
   switchport port-security violation shutdown
   switchport port-security mac-address sticky

  ! Repetir para otros puertos de acceso (por ejemplo, FastEthernet0/2 para VLAN 200, etc.)
  interface FastEthernet0/2
   switchport mode access
   switchport access vlan 200
   switchport port-security
   switchport port-security maximum 1
   switchport port-security violation shutdown
   switchport port-security mac-address sticky
   no shutdown
  ```
- **Configuración en Switch 2 (Edificio 2) y Switch 3 (Edificio 3)**: Similar a Switch 1, aplicando Port Security a todos los puertos de acceso conectados a dispositivos finales (PCs) en las VLANs 100, 200, 300, y 400.
- **NOTAS**
  - maximum 1: Permite solo una dirección MAC por puerto.
  - violation shutdown: Desactiva el puerto si se conecta un dispositivo no autorizado.
  - mac-address sticky: Aprende automáticamente la primera dirección MAC que se conecta y la guarda como autorizada.
  - Los puertos troncales (hacia Router EDGE) no requieren Port Security, ya que conectan dispositivos de red confiables.

### 4.2 Comunicación Segura
- **VPN Sitio a Sitio con IPsec**: Se configura una VPN entre el router EDGE y un sitio remoto para garantizar confidencialidad. Se utiliza IPsec con cifrado AES-256 y autenticación SHA-256.

- **Objetivo de la VPN**:
  - Garantizar confidencialidad para el tráfico entre las VLANs (en la red del Router EDGE) y el servidor Samba/Postfix (en la red del Router RT_SERVICES).
  - La VPN usa IPsec con cifrado AES-256 y autenticación SHA-256, aplicada a las interfaces seriales de los routers EDGE (172.16.20.41) y RT_SERVICES (172.16.20.46).

- **Configuración de la VPN**:
    La VPN IPsec requiere configuraciones espejo en el Router EDGE y el Router RT_SERVICES, aplicadas a sus interfaces seriales (Serial0/0/0 en ambos routers). La ACL define el tráfico a cifrar: desde las VLANs (172.16.16.0/20) hacia la subred del servidor (172.16.20.32/29) y viceversa.
    - Política ISAKMP (fase 1): Establece los parámetros de autenticación y cifrado.
    - Clave precompartida: Para autenticar los routers.
    - Transform-set: Define el cifrado y autenticación para la fase 2.
    - ACL: Especifica el tráfico a cifrar (VLANs ↔ servidor).
    - Crypto map: Combina la configuración y se aplica a la interfaz serial.

- **Configuración en router EDGE**:
  ```plaintext
  ! Definir política ISAKMP (fase 1)
  crypto isakmp policy 10
   encryption aes 256
   authentication pre-share
   group 2
  crypto isakmp key SecretKey address 172.16.20.46

  ! Definir transform-set (fase 2)
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
   ipv6 address 2001:dbad:acad:6000::1/126
   crypto map CMAP
  ```
- **RT_SERVICES**:
  ```plaintext
  ! Definir política ISAKMP (fase 1)
  crypto isakmp policy 10
   encryption aes 256
   authentication pre-share
   group 2
  crypto isakmp key SecretKey address 172.16.20.41

  ! Definir transform-set (fase 2)
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
   ipv6 address 2001:dbad:acad:6004::2/126
   crypto map CMAP
  ```
- **Pruebas de la VPN**:
  - Verificar el Túnel:
    ```plaintext
    show crypto isakmp sa
    show crypto ipsec sa
    ```
  - Probar conectividad
    - Desde un PC en cualquier VLAN (por ejemplo, 172.16.16.10 en VLAN 100), accede al servidor Samba (\\172.16.20.34\Contabilidad) o Postfix (IMAP en 172.16.20.34:143).
    - El tráfico debe pasar por el túnel IPsec.
  ---
  
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
 router-id 1.1.1.1
 network 172.16.16.0 0.0.1.255 area 0  ! VLAN 100
 network 172.16.18.0 0.0.0.255 area 0  ! VLAN 200
 network 172.16.19.0 0.0.0.255 area 0  ! VLAN 300
 network 172.16.20.0 0.0.0.31 area 0   ! VLAN 400
 network 172.16.20.40 0.0.0.3 area 0   ! Serial EDGE-ISP
 network 172.16.20.2 0.0.0.0 area 0    ! Loopback

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
interface Serial0/0/0
 ipv6 ospf 1 area 0
interface Loopback0
 ipv6 ospf 1 area 0
```

**ISP**:
```plaintext
! OSPF para IPv4
router ospf 1
 router-id 2.2.2.2
 network 172.16.20.40 0.0.0.3 area 0  ! Serial EDGE-ISP
 network 172.16.20.44 0.0.0.3 area 0  ! Serial ISP-RT_SERVICES
 network 172.16.20.4 0.0.0.0 area 0   ! Loopback

! OSPFv3 para IPv6
ipv6 router ospf 1
 router-id 2.2.2.2
interface Serial0/0/0
 ipv6 ospf 1 area 0
interface Serial0/0/1
 ipv6 ospf 1 area 0
interface Loopback0
 ipv6 ospf 1 area 0
```

**RT_SERVICES**:
```plaintext
! OSPF para IPv4
router ospf 1
 router-id 3.3.3.3
 network 172.16.20.32 0.0.0.7 area 0  ! Subred Servidor
 network 172.16.20.44 0.0.0.3 area 0  ! Serial ISP-RT_SERVICES
 network 172.16.20.3 0.0.0.0 area 0   ! Loopback

! OSPFv3 para IPv6
ipv6 router ospf 1
 router-id 3.3.3.3
interface GigabitEthernet0/0
 ipv6 ospf 1 area 0
interface Serial0/0/0
 ipv6 ospf 1 area 0
interface Loopback0
 ipv6 ospf 1 area 0
```
- **NOTAS**:
  - Cada router tiene un router-id único.
  - Todas las subredes (VLANs, servidor, seriales, loopbacks) se anuncian en el área 0.
  - OSPFv3 requiere habilitar IPv6 en las interfaces relevantes.
---

## 6. Servicios

### 6.1 Servicio de Correo
- **Implementación**: Postfix y Dovecot en máquina virtual Ubuntu (IPv4: 172.16.20.34, IPv6: 2001:dbad:acad:5000::2). 10 usuarios configurados (correo1@empresa.com a correo10@empresa.com).
- **Configuración Postfix**:
  ```ini
  myhostname = mail.empresa.com
  mydomain = empresa.com
  inet_interfaces = 172.16.20.34, 2001:dbad:acad:5000::2
  mynetworks = 172.16.16.0/20, 2001:dbad:acad::/48
  mydestination = $myhostname, localhost.$mydomain, $mydomain
  ```
- **Configuración Dovecot**:
  - Protocolo: IMAP (puerto 143).
  - Ubicación de buzones: `maildir:~/Maildir`.
  - Autenticación: Usuarios del sistema.
  ```ini
  service imap-login {
    inet_listener imap {
      address = 172.16.20.34, 2001:dbad:acad:5000::2
      port = 143
    }
  }
  ```
- **Detalles Técnicos**:
  - Sistema Operativo: Ubuntu Server en VMware.
  - Usuarios: 10 usuarios del sistema con buzones Maildir.
  - Red: Accesible desde todas las VLANs vía VPN.
  - Firewall: Puertos 25 (SMTP) y 143 (IMAP) abiertos en `ufw`.
- **Acceso**: Clientes usan IMAP (e.g., `imap://172.16.20.34:143`). Soporte para IPv6 habilitado.
- **Validación**: Envío y recepción de correos entre usuarios.
- **Simulación en Packet Tracer**: Servidor genérico con correo simulado.

### 6.2 Servicio de Archivos
- **Implementación**: Samba en la misma máquina virtual Ubuntu (IPv4: 172.16.20.34, IPv6: 2001:dbad:acad:5000::2). Tres perfiles:
  - Contabilidad: Solo lectura, grupo `contabilidad`.
  - Desarrollo: Lectura/escritura, grupo `desarrollo`.
  - Administración: Acceso completo, grupo `admon`.
- **Configuración Samba**:
  ```ini
  [global]
     server string = Samba Server
     bind interfaces only = yes
     interfaces = 172.16.20.34 2001:dbad:acad:5000::2
     security = user

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
  - Sistema Operativo: Ubuntu Server en VMware.
  - Usuarios y Grupos: Grupos `contabilidad`, `desarrollo`, `admon` con usuarios asociados.
  - Permisos: Directorios `/srv/samba/*` con permisos 750 (Contabilidad) y 770 (Desarrollo, Admon).
  - Red: Accesible desde todas las VLANs vía VPN.
  - Firewall: Puertos SMB/CIFS (137, 138, 139, 445) abiertos en `ufw`.
- **Acceso**: Clientes acceden mediante SMB (e.g., `\\172.16.20.34\Contabilidad`). Soporte para IPv6 habilitado.
- **Validación**: Permisos verificados por perfil.
- **Simulación en Packet Tracer**: Servidor genérico con FTP.

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
- **Router ISP en VLAN 400**: Para gestión remota.
- **EtherChannel**: Elegido para aumentar el ancho de banda y proporcionar redundancia sin depender de hardware adicional.
- **Servidor en Subred Separada**: Samba/Postfix en 172.16.20.32/29 para simplificar enrutamiento.
- **PVST**: Seleccionado para optimizar la redundancia por VLAN y evitar bucles.
- **OSPF/OSPFv3**: Usado para enrutamiento dinámico por su escalabilidad y soporte en doble stack.
- **Samba para Archivos**: Escogido por su compatibilidad con múltiples plataformas y facilidad de configuración de perfiles.
- **Postfix para Correo**: Seleccionado por su robustez y soporte para múltiples usuarios.
- **IPsec con AES-256**: Implementado para garantizar confidencialidad en la VPN, cumpliendo con estándares de seguridad modernos.
- **Interfaces Seriales**: Subredes /30 para enlaces punto a punto.

---

## 9. Conclusiones
El diseño propuesto cumple con todos los requerimientos del proyecto, proporcionando una red escalable, segura y redundante. La implementación en Packet Tracer y en hardware físico permitirá validar el diseño en entornos simulados y reales. El uso de protocolos estándares como OSPF, STP y IPsec asegura compatibilidad y estabilidad.

---

## 10. Anexos
- **Topología Packet Tracer**: Archivo .pkt disponible para revisión.
- **Configuraciones Completas**: Scripts de configuración para routers y switches.
- **Guías de Implementación**: Documentación adicional para servidores Postfix y Samba.

## 11. Configuración de Thunderbird en Windows
### Instalación
1. Descarga e instala Thunderbird desde [https://www.thunderbird.net](https://www.thunderbird.net).

### Creación de cuenta
2. Crea una nueva cuenta (puedes omitir este paso si no tienes un correo real):
   - **Nombre:** `correo1`
   - **Correo:** `correo1@empresa.com`
   - **Contraseña:** `correo123`

### Configuración manual
3. Configura los servidores manualmente:

   #### Servidor entrante (IMAP)
   - **Servidor:** `172.16.20.34`
   - **Puerto:** `143`
   - **Seguridad:** `Sin cifrado` (STARTTLS si está habilitado)
   - **Autenticación:** `Contraseña normal`

   #### Servidor saliente (SMTP)
   - **Servidor:** `172.16.20.34`
   - **Puerto:** `25`
   - **Seguridad:** `Sin cifrado`
   - **Autenticación:** `Contraseña normal`
   - **Usuario:** `correo1`

4. Guarda los cambios y prueba enviándote un correo de prueba.

## 12. Script de configuración Samba/PostFix-Dovecot para red en casa desde ISP
```ini
#!/bin/bash
# setup_casa_completo.sh
# Configuración inicial completa (Samba, correo, red local)

echo "Configurando entorno completo en casa..."

# Interfaz de red (ajustar según tu VM)
INTERFAZ="ens33"
IPV4="192.168.1.100/24"
GW4="192.168.1.1"

# 1. Configurar red local
echo "1. Configurando red..."
sudo tee /etc/netplan/01-netcfg.yaml > /dev/null <<EOF
network:
  version: 2
  ethernets:
    $INTERFAZ:
      addresses:
        - $IPV4
      gateway4: $GW4
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
EOF

sudo netplan apply

# 2. Instalar paquetes necesarios
echo "2. Instalando paquetes..."
sudo apt update && sudo apt install -y samba postfix dovecot-core dovecot-imapd ufw

# 3. Crear carpetas y grupos para Samba
echo "3. Configurando Samba..."
sudo mkdir -p /srv/samba/{contabilidad,desarrollo,admon}
sudo groupadd contabilidad
sudo groupadd desarrollo
sudo groupadd admon

# 4. Crear usuarios
create_user() {
  if id "$1" &>/dev/null; then
    echo "Usuario $1 ya existe"
  else
    sudo useradd -m -g "$2" -s /bin/false "$1"
    echo "$1:$3" | sudo chpasswd
    echo "$1" | sudo smbpasswd -a "$1"
  fi
}
create_user "usuario_cont1" "contabilidad" "1234"
create_user "usuario_dev1" "desarrollo" "1234"
create_user "usuario_adm1" "admon" "1234"
create_user "correo1" "users" "correo123"

# 5. Permisos
sudo chown :contabilidad /srv/samba/contabilidad && sudo chmod 750 /srv/samba/contabilidad
sudo chown :desarrollo /srv/samba/desarrollo && sudo chmod 770 /srv/samba/desarrollo
sudo chown :admon /srv/samba/admon && sudo chmod 770 /srv/samba/admon

# 6. Configuración Samba
sudo tee /etc/samba/smb.conf > /dev/null <<EOF
[global]
   server string = Samba Server
   workgroup = WORKGROUP
   bind interfaces only = yes
   interfaces = $IPV4
   security = user

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
EOF

# 7. Configurar correo (Postfix/Dovecot)
sudo tee -a /etc/postfix/main.cf > /dev/null <<EOF
myhostname = mail.empresa.com
mydomain = empresa.com
myorigin = /etc/mailname
inet_interfaces = $IPV4
mydestination = \$myhostname, localhost.\$mydomain, \$mydomain
mynetworks = 192.168.1.0/24
mailbox_size_limit = 0
recipient_delimiter = +
EOF

sudo sed -i "s|^#mail_location =.*|mail_location = maildir:~/Maildir|" /etc/dovecot/conf.d/10-mail.conf
sudo sed -i "s/^#disable_plaintext_auth = yes/disable_plaintext_auth = no/" /etc/dovecot/conf.d/10-auth.conf
sudo sed -i "s/^auth_mechanisms = .*/auth_mechanisms = plain login/" /etc/dovecot/conf.d/10-auth.conf

sudo tee -a /etc/dovecot/conf.d/10-master.conf > /dev/null <<EOF

service imap-login {
  inet_listener imap {
    address = 192.168.1.100
    port = 143
  }
}
EOF

# 8. Activar firewall
sudo ufw --force reset
sudo ufw allow 25/tcp
sudo ufw allow 143/tcp
sudo ufw allow 137/udp
sudo ufw allow 138/udp
sudo ufw allow 139/tcp
sudo ufw allow 445/tcp
sudo ufw --force enable

# 9. Reiniciar servicios
sudo systemctl restart smbd nmbd postfix dovecot

echo "✅ Configuración de casa completada."
```
---
## 13. Script de configuración de red para laboratorio de cisco
```ini
#!/bin/bash
# setup_red_laboratorio.sh
# Cambia la configuración de red para el entorno del laboratorio

echo "Configurando red del LABORATORIO..."

INTERFAZ="ens33"
IPV4="172.16.20.34/29"
GW4="172.16.20.33"
GW4="172.16.20.33"
GW6="2001:dbad:acad:5000::1"

sudo tee /etc/netplan/01-netcfg.yaml > /dev/null <<EOF
network:
  version: 2
  ethernets:
    $INTERFAZ:
      addresses:
        - $IPV4
        - $IPV6
      gateway4: $GW4
      gateway6: $GW6
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
EOF

sudo netplan apply
sudo systemctl restart smbd nmbd postfix dovecot

echo "✅ Red del laboratorio configurada."
```
---
## 14. Script de configuración de red al volver a la red local de casa
```ini
#!/bin/bash
# setup_red_casa.sh
# Cambia la configuración de red para volver al entorno de casa

echo "Volviendo a red de CASA..."

INTERFAZ="ens33"
IPV4="192.168.1.100/24"
GW4="192.168.1.1"

sudo tee /etc/netplan/01-netcfg.yaml > /dev/null <<EOF
network:
  version: 2
  ethernets:
    $INTERFAZ:
      addresses:
        - $IPV4
      gateway4: $GW4
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
EOF

sudo netplan apply
sudo systemctl restart smbd nmbd postfix dovecot

echo "✅ Red de casa configurada nuevamente."
```
