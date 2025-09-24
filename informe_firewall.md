# **Informe de Implementación de Firewall en MikroTik**

## **Introducción**

Este informe documenta la configuración completa de una política de seguridad corporativa en un entorno de laboratorio virtual.  
El objetivo principal fue transformar una red con conectividad básica en un entorno seguro y segmentado, donde el acceso a los recursos se controla estrictamente según el rol de cada usuario.

La topología implementada es **Router on a Stick**, con las siguientes subredes y VLANs:

- **VLAN IT**: `192.168.68.0/24`
- **VLAN Empleados**: `192.168.68.128/26`
- **VLAN Invitados**: `192.168.68.192/27`
- **IP PC de IT**: `192.168.68.254`

---

## **Configuración Inicial de la Red**

### 🚀 **Configuración del Router MikroTik**

Se configuraron las interfaces VLAN, las direcciones IP, el NAT y el servidor DHCP en el router MikroTik para establecer la base de la red.

1. **Crear las Interfaces VLAN**
   ```shell
   /interface vlan
   add name=VLAN_IT vlan-id=168 interface=ether2
   add name=VLAN_EMPLEADOS vlan-id=268 interface=ether2
   add name=VLAN_INVITADOS vlan-id=368 interface=ether2
   ```

2. **Asignar Direcciones IP a las Interfaces VLAN**
   ```shell
   /ip address
   add address=192.168.68.1/24 interface=VLAN_IT network=192.168.68.0
   add address=192.168.68.129/26 interface=VLAN_EMPLEADOS network=192.168.68.128
   add address=192.168.68.193/27 interface=VLAN_INVITADOS network=192.168.68.192
   ```

3. **Configurar NAT para Salida a Internet**
   ```shell
   /ip firewall nat
   add chain=srcnat out-interface=ether1 action=masquerade
   ```

4. **Configurar DHCP Server**
   ```shell
   /ip pool
   add name=pool_IT ranges=192.168.68.2-192.168.68.254
   add name=pool_EMPLEADOS ranges=192.168.68.130-192.168.68.190
   add name=pool_INVITADOS ranges=192.168.68.194-192.168.68.222

   /ip dhcp-server
   add name=DHCP_IT interface=VLAN_IT address-pool=pool_IT disabled=no
   add name=DHCP_EMPLEADOS interface=VLAN_EMPLEADOS address-pool=pool_EMPLEADOS disabled=no
   add name=DHCP_INVITADOS interface=VLAN_INVITADOS address-pool=pool_INVITADOS disabled=no

   /ip dhcp-server network
   add address=192.168.68.0/24 gateway=192.168.68.1 dns-server=192.168.68.1
   add address=192.168.68.128/26 gateway=192.168.68.129 dns-server=192.168.68.1
   add address=192.168.68.192/27 gateway=192.168.68.193 dns-server=192.168.68.1
   ```

---

### 🎛️ **Configuración del Switch**

Se configuró el switch para manejar las VLANs y establecer el puerto troncal hacia el router, además de los puertos de acceso para cada PC.

```shell
! Entrar al modo de configuración global
configure terminal

! Configurar la interfaz troncal
interface GigabitEthernet0/0
 switchport mode trunk
 switchport trunk native vlan 468
 no shutdown

! Configurar interfaces de acceso
interface GigabitEthernet0/1
 switchport mode access
 switchport access vlan 168
 no shutdown
exit

interface GigabitEthernet0/2
 switchport mode access
 switchport access vlan 268
 no shutdown
exit

interface GigabitEthernet0/3
 switchport mode access
 switchport access vlan 368
 no shutdown
exit

! Habilitar VLANs
vlan 168
vlan 268
vlan 368
vlan 468
end
write memory
```

---

## **Implementación de las Tareas de Firewall**

### 🛡️ **Tarea 1: Base de Seguridad (Firewall Stateful)**

```shell
/ip firewall filter
add action=accept chain=input connection-state=established,related comment="Aceptar trafico establecido y relacionado"
add action=accept chain=forward connection-state=established,related comment="Aceptar trafico establecido y relacionado"
add action=drop chain=input connection-state=invalid comment="Descartar trafico invalido"
add action=drop chain=forward connection-state=invalid comment="Descartar trafico invalido"
```

---

### 🛡️ **Tarea 2: Asegurar el Acceso Administrativo**

```shell
/ip firewall address-list
add list=IT_ADMIN_IP address=192.168.68.254 comment="IP estatica de la PC de IT"
add list=LAN_IT address=192.168.68.0/24 comment="Subred de la VLAN de IT"
add list=LAN_EMPLEADOS address=192.168.68.128/26 comment="Subred de la VLAN de Empleados"
add list=LAN_INVITADOS address=192.168.68.192/27 comment="Subred de la VLAN de Invitados"

/ip firewall filter
add action=drop chain=input protocol=icmp src-address-list=LAN_EMPLEADOS comment="Bloquear ping desde Empleados"
add action=drop chain=input protocol=icmp src-address-list=LAN_INVITADOS comment="Bloquear ping desde Invitados"
add action=accept chain=input protocol=icmp src-address-list=LAN_IT comment="Permitir ping solo desde IT"
add action=accept chain=input src-address-list=IT_ADMIN_IP protocol=tcp dst-port=22 comment="Permitir acceso SSH desde IT"
add action=accept chain=input src-address-list=IT_ADMIN_IP protocol=tcp dst-port=8291 comment="Permitir acceso WinBox desde IT"
add action=accept chain=input protocol=udp dst-port=53 comment="Permitir DNS UDP desde VLANs internas"
add action=accept chain=input protocol=tcp dst-port=53 comment="Permitir DNS TCP desde VLANs internas"

/ip service enable mac-winbox
/ip service disable telnet,www,ftp,api,mac-ssh
```

---

### 🛡️ **Tarea 3: Control de Navegación con Listas Dinámicas (FQDN)**

```shell
/ip firewall address-list
add list=Sitios_Bloqueados address=facebook.com
add list=Sitios_Bloqueados address=instagram.com
add list=Sitios_Bloqueados address=x.com
add list=Servicios_Permitidos address=gmail.com
add list=Servicios_Permitidos address=google.com
add list=Servicios_Permitidos address=mercadopago.com.ar
add list=Servicios_Permitidos address=mercadolibre.com
add list=Servicios_Permitidos address=mercadolibre.com.ar

/ip firewall filter
add action=drop chain=forward src-address-list=LAN_EMPLEADOS dst-address-list=Sitios_Bloqueados comment="Bloquear redes sociales para Empleados"
add action=accept chain=forward src-address-list=LAN_INVITADOS dst-address-list=Servicios_Permitidos protocol=tcp dst-port=80,443 comment="Permitir trafico web a sitios de la lista blanca"
add action=drop chain=forward src-address-list=LAN_INVITADOS protocol=tcp dst-port=80,443 comment="Bloquear todo el trafico web no permitido a Invitados"
```

---

### 🛡️ **Tarea 4: Aislamiento de Redes y Acceso Controlado**

```shell
/ip firewall filter
add action=drop chain=forward protocol=icmp src-address-list=LAN_EMPLEADOS dst-address-list=LAN_IT comment="Bloquear ping de Empleados a IT"
add action=accept chain=forward protocol=tcp dst-port=80 src-address-list=LAN_EMPLEADOS dst-address=192.168.68.254 comment="Permitir acceso web de Empleados a servidor IT"
add action=drop chain=forward src-address-list=LAN_EMPLEADOS dst-address-list=LAN_IT comment="Bloquear todo el resto del trafico de Empleados a IT"
```

---

### 🛡️ **Tarea 5: Permitir Salida a Internet, Bloqueo Final y Logs**

```shell
/ip firewall filter
add action=accept chain=forward comment="Permitir salida a internet"
add action=drop chain=input comment="Bloqueo por defecto de la cadena input"
add action=drop chain=forward comment="Bloqueo por defecto de la cadena forward"

/ip firewall filter set [find comment="Bloqueo por defecto de la cadena input"] log=yes log-prefix="ACCESO-NO-AUTORIZADO"
/ip firewall filter set [find comment="Bloqueo por defecto de la cadena forward"] log=yes log-prefix="PAQUETE-DESCARTADO"
```
