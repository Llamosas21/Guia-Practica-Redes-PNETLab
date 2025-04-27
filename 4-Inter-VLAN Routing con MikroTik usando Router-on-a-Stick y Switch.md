
### Requisitos:

- Cloud0 (Management):1
- Windows Server:  1 
- Router Mikrotik:  1 
- Computadoras: 1
- Linux Server:  1
- Switch: 1
### Aclaraciones
- La imagen del servidor Linux será la linux-alpine-xfce.
- El Network (ISP), será el Managment(Cloud0).
- Se trabajará solamente con la consola, aunque se pueden usar softwares como Winbox.
- Las IPs pueden variar ya que se usará el protocolo DHCP.

### Conexión:

![[Esquema 4.png]]

### PASOS A SEGUIR:

 #### Switch:
- [[2-Comunicación entre dos VLANs gestionadas por un Switch#Crear VLANs en Switch (Capa 2)|Crear las VLANs en Switch (VLAN Primaria y VLAN Secundaria)]]
- [[3-Configuración de IPs, Hostnames, VLANs y Troncales en Switches#Modo acceso (access)|Configurar los puertos en modo acceso en el Switch]]
- [[3-Configuración de IPs, Hostnames, VLANs y Troncales en Switches#Configuración de puertos del Switch Trunk|Asignar la VLAN de tipo TRUNK en el Switch]] 

| Nombre          | VLAN ID | Subred          | Máscara |
| --------------- | ------- | --------------- | ------- |
| VLAN Primaria   | 2209    | 172.22.17.0/24  | /24     |
| VLAN Secundaria | 2309    | 172.23.17.0/24  | /24     |
| VLAN Nativa     | 2409    | (Para el Trunk) | /24     |

#### Router Mikrotik:

##### **IMPORTANTE**: 

 La primera vez que se conecte, va a pedir un usuario (admin) y una contraseña que por default está vacía, luego va a aparecer algo como esto:

	o you want to see the software license? [Y/n]: n
	En donde se recomineda no ver la licencia. 

Finalmente se les pedirá una nueva contraseña luego del primer Enter se les pedirá que la escriban una vez más, si amabas coinciden mostrará este mensaje:

	Change your password  
	    
	new password> *****  
	repeat new password> *****  
	  
	Password changed  
	[admin@MikroTik] >


#### Configurar el Router MikroTik (Router-on-a-Stick):
 
###### Crear las interfaces VLAN sobre la física definida en el Switch:

- [[#**IMPORTANTE**|Leer en caso de problemas]]
 
 Dentro de la consola de Mikrotik, procedemos a irnos a la interfaz y luego agregaremos cada una de las VLANs, tener presente que ether2 es el puerto del router que va al Switch.
 
	/interface vlan  
	/interface/vlan> add interface=ether2 name=vlan_primaria vlan-id=2209  
	/interface/vlan> add interface=ether2 name=vlan_secundaria vlan-id=2309  
	/interface/vlan>


###### Asignar IPs a las Interfaces VLAN del Mikrotik:

	/ip address 
	/ip/address> add address=172.22.17.1/24 interface=vlan_primaria  
	/ip/address> add address=172.23.17.1/24 interface=vlan_secundaria  

 Las IPs `X.X.X.1` son las puertas de enlace (gateway), para cada VLAN.

##### Crear servidores DHCP por VLAN:

###### DHCP Primaria

	/ip pool
	add name=dhcp_pool_2209 ranges=172.22.17.100-172.22.17.200
	
	/ip dhcp-server
	add name=dhcp_primaria interface=vlan_primaria address-pool=dhcp_pool_2209 lease-time=1h disabled=no
	
	/ip dhcp-server network
	add address=172.22.17.0/24 gateway=172.22.17.1 dns-server=8.8.8.8

###### DHCP Secundaria

	/ip pool
	add name=dhcp_pool_2309 ranges=172.23.17.100-172.23.17.200
	
	/ip dhcp-server
	add name=dhcp_secundaria interface=vlan_secundaria address-pool=dhcp_pool_2309 lease-time=1h disabled=no
	
	/ip dhcp-server network
	add address=172.23.17.0/24 gateway=172.23.17.1 dns-server=8.8.8.8

###### Configurar NAT para Internet (Firewall)

	/ip firewall nat
	add chain=srcnat out-interface=ether1 action=masquerade

El puerto a seleccionar debe ser el que se conecta con el en este caso Cloud0 (el ISP)


##### Asignar IP estática a una VPCs 

**Mikrotik**:

	/ip address
	add address=172.20.17.1/24 interface=ether2
	
	/ip firewall nat
	add chain=srcnat out-interface=ether1 src-address=172.20.17.0/24 action=masquerade
	
	/system backup save

SWITCH:

	conf t
	int eth0/3
	sw acc vlan 2409
	exit

Lo que hace esto es camiar la conexión a esto:

| VLAN | Name               | Status    | Ports                        |
| ---: | ------------------ | --------- | ---------------------------- |
|    1 | default            | active    |                              |
| 1002 | fddi-default       | act/unsup |                              |
| 1003 | token-ring-default | act/unsup |                              |
| 1004 | fddinet-default    | act/unsup |                              |
| 1005 | trnet-default      | act/unsup |                              |
| 2209 | VLAN Primaria      | active    | Et0/1                        |
| 2309 | VLAN Secundaria    | active    | Et0/2                        |
| 2409 | VLAN2409           | active    | Et0/3 *(Trunk)* *Puerto VPCs |



