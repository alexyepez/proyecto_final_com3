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

### 2.2 Direccionamiento IPv4
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

| VLAN | Edificio | Usuarios (con 5%) | Hosts Requeridos | Subred Asignada | Rango de Direcciones | Máscara |
|------|----------|-------------------|------------------|-----------------|----------------------|---------|
| Contabilidad | Edif. 1 | 105 | 128 | 172.16.16.0/25 | 172.16.16.0 - 172.16.16.127 | /25 |
| Contabilidad | Edif. 2 | 231 | 256 | 172.16.17.0/24 | 172.16.17.0 - 172.16.17.255 | /24 |
| Contabilidad | Edif. 3 | 5 | 8 | 172.16.18.0/29 | 172.16.18.0 - 172.16.18.7 | /29 |
| Desarrollo | Edif. 1 | 6 | 8 | 172.16.18.8/29 | 172.16.18.8 - 172.16.18.15 | /29 |
| Desarrollo | Edif. 2 | 126 | 128 | 172.16.18.128/25 | 172.16.18.128 - 172.16.18.255 | /25 |
| Desarrollo | Edif. 3 | 111 | 128 | 172.16.19.0/25 | 172.16.19.0 - 172.16.19.127 | /25 |
| Diseño | Edif. 1 | 6 | 8 | 172.16.19.128/29 | 172.16.19.128 - 172.16.19.135 | /29 |
| Diseño | Edif. 2 | 21 | 32 | 172.16.19.160/27 | 172.16.19.160 - 172.16.19.191 | /27 |
| Diseño | Edif. 3 | 126 | 128 | 172.16.20.0/25 | 172.16.20.0 - 172.16.20.127 | /25 |
| Adm. Red | Edif. 1 | 5 | 8 | 172.16.20.128/29 | 172.16.20.128 - 172.16.20.135 | /29 |
| Adm. Red | Edif. 2 | 5 | 8 | 172.16.20.136/29 | 172.16.20.136 - 172.16.20.143 | /29 |
| Adm. Red | Edif. 3 | 5 | 8 | 172.16.20.144/29 | 172.16.20.144 - 172.16.20.151 | /29 |
| Equipos de Red | Todos | - | 16 | 172.16.20.160/28 | 172.16.20.160 - 172.16.20.175 | /28 |
| Reservado | Futuro | - | - | 172.16.21.0 - 172.16.31.255 | - | - |

**Nota**: La subred para equipos de red incluye direcciones estáticas para routers, switches y servidores. El espacio reservado permite añadir más edificios o VLANs.

### 2.3 Direccionamiento IPv6
La dirección base es **2001:dbad:acad::/48**. Se asignan subredes de /64, añadiendo un prefijo de 16 bits para identificar cada VLAN y edificio. Esto proporciona 65,536 subredes posibles.

| VLAN | Edificio | Subred Asignada | Prefijo |
|------|----------|-----------------|---------|
| Contabilidad | Edif. 1 | 2001:dbad:acad:1000::/64 | 1000 |
| Contabilidad | Edif. 2 | 2001:dbad:acad:1001::/64 | 1001 |
| Contabilidad | Edif. 3 | 2001:dbad:acad:1002::/64 | 1002 |
| Desarrollo | Edif. 1 | 2001:dbad:acad:2000::/64 | 2000 |
| Desarrollo | Edif. 2 | 2001:dbad:acad:2001::/64 | 2001 |
| Desarrollo | Edif. 3 | 2001:dbad:acad:2002::/64 | 2002 |
| Diseño | Edif. 1 | 2001:dbad:acad:3000::/64 | 3000 |
| Diseño | Edif. 2 | 2001:dbad:acad:3001::/64 | 3001 |
| Diseño | Edif. 3 | 2001:dbad:acad:3002::/64 | 3002 |
| Adm. Red | Edif. 1 | 2001:dbad:acad:4000::/64 | 4000 |
| Adm. Red | Edif. 2 | 2001:dbad:acad:4001::/64 | 4001 |
| Adm. Red | Edif. 3 | 2001:dbad:acad:4002::/64 | 4002 |
| Equipos de Red | Todos | 2001:dbad:acad:5000::/64 | 5000 |

**Nota**: Los prefijos se asignan jerárquicamente (1000 para Contabilidad, 2000 para Desarrollo, etc.) para facilitar la administración y escalabilidad.

### 2.4 Asignación Dinámica de Direcciones
- **IPv4**: Se configura un servidor DHCP en cada edificio para cada VLAN, asignando direcciones dinámicas a dispositivos finales. Los servidores DHCP se configuran en la subred de equipos de red (172.16.20.160/28). Se excluyen las primeras 10 direcciones de cada subred para asignaciones estáticas (gateways, servidores, etc.).
- **IPv6**: Se utiliza SLAAC (Stateless Address Autoconfiguration) para asignar direcciones automáticamente a dispositivos finales, con el router enviando anuncios de prefijo (/64). Para dispositivos de red, se asignan direcciones estáticas en la subred 2001:dbad:acad:5000::/64.

---

## 3. Diseño de Red Física y Redundancia

### 3.1 Topología
La red consta de:
- **Router EDGE**: Conecta la red interna con el exterior (Internet). Configurado con una interfaz Ethernet para comunicación inter-VLAN.
- **Switches de Distribución**: Uno por edificio, conectados al router EDGE mediante enlaces troncales (trunk).
- **Switches de Acceso**: Conectados a los switches de distribución, asignados a las VLANs específicas por edificio.
- **Servidores**: Un servidor virtualizado (físico) en la subred de equipos de red para servicios de correo y archivos.

### 3.2 Aumento de Ancho de Banda y Redundancia
- **EtherChannel**: Se configuran enlaces EtherChannel entre switches de distribución y acceso para aumentar el ancho de banda. Cada EtherChannel agrupa dos o más interfaces GigabitEthernet, proporcionando redundancia y mayor capacidad.
- **Protocolo STP (Spanning Tree Protocol)**: Se activa Rapid Per-VLAN Spanning Tree (PVST) para evitar bucles en la red. Cada VLAN tiene su propia instancia de STP, con el switch de distribución del Edificio 2 como raíz primaria para balanceo de carga.
- **Configuración Ejemplo (Switch Distribución Edif. 2)**:
  ```plaintext
  interface Port-channel1
   switchport mode trunk
   switchport trunk allowed vlan 100,200,300,400
  interface range GigabitEthernet0/1 - 2
   channel-group 1 mode active
   switchport mode trunk
  spanning-tree vlan 100,200,300,400 priority 4096
  ```

---

## 4. Seguridad

### 4.1 Control de Acceso Físico
- **Port Security**: Se configura en los puertos de los switches de acceso para permitir solo direcciones MAC autorizadas. Si se detecta un dispositivo no autorizado, el puerto se desactiva.
  ```plaintext
  interface GigabitEthernet0/1
   switchport mode access
   switchport access vlan 100
   switchport port-security
   switchport port-security maximum 1
   switchport port-security violation shutdown
   switchport port-security mac-address sticky
  ```

### 4.2 Comunicación Segura
- **VPN Sitio a Sitio con IPsec**: Se configura una VPN entre el router EDGE y un sitio remoto (simulado) para garantizar confidencialidad. Se utiliza IPsec con cifrado AES-256 y autenticación SHA-256.
  ```plaintext
  crypto isakmp policy 10
   encryption aes 256
   authentication pre-share
   group 2
  crypto isakmp key SecretKey address 203.0.113.1
  crypto ipsec transform-set TS esp-aes 256 esp-sha-hmac
  crypto map CMAP 10 ipsec-isakmp
   set peer 203.0.113.1
   set transform-set TS
   match address ACL_VPN
  ip access-list extended ACL_VPN
   permit ip 172.16.16.0 0.0.15.255 192.168.1.0 0.0.0.255
  interface GigabitEthernet0/0
   crypto map CMAP
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
```plaintext
router ospf 1
 network 172.16.16.0 0.0.15.255 area 0
ipv6 router ospf 1
 router-id 1.1.1.1
interface GigabitEthernet0/1.100
 ipv6 ospf 1 area 0
```

---

## 6. Servicios

### 6.1 Servicio de Correo
- **Implementación**: Se configura un servidor de correo (Postfix en Linux virtualizado) en la subred de equipos de red (172.16.20.161, 2001:dbad:acad:5000::2). Se crean 10 cuentas de usuario accesibles desde todas las VLANs.
- **Acceso**: Los clientes usan IMAP (puerto 143) para acceder al correo. Se configura un firewall para permitir solo tráfico autorizado.

### 6.2 Servicio de Archivos
- **Implementación**: Se configura un servidor Samba en el mismo equipo virtualizado con tres perfiles de acceso:
  - **Perfil 1 (Lectura)**: Acceso de solo lectura para VLAN Contabilidad.
  - **Perfil 2 (Escritura)**: Acceso de lectura/escritura para VLAN Desarrollo.
  - **Perfil 3 (Administrador)**: Acceso completo para VLAN Adm. Red.
- **Configuración Samba**:
  ```plaintext
  [Contabilidad]
   path = /srv/samba/contabilidad
   read only = yes
   valid users = @contabilidad
  [Desarrollo]
   path = /srv/samba/desarrollo
   read only = no
   valid users = @desarrollo
  [Admon]
   path = /srv/samba/admon
   read only = no
   create mask = 0777
   valid users = @admon
  ```

---

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
