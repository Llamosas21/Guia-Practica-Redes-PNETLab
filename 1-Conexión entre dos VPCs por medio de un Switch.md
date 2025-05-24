#### Requisitos:

- Computadoras: 2
- Switch: 1

#### Conexión:

Se procederá a hacer las conexiones físicas entre VPCs y Switch, para luego prender todos los dispositivos.

Para esta conexión se utilizará el Rango:

	- Ip red: 192.168.111.0/24
	
	- Ip Broadcast: 192.168.111.255
	
	- Ip pool: 192.168.111.1 - 192.168.111.254

![Inter VLAN](Imagenes/1-Conexión%20entre%20dos%20PCs%20por%20medio%20de%20un%20Switch/Esquema%202.png)

### Asignar IP a las VPCs

Abriremos las terminales de cada una para poder asignarles la IP dentro del rango válido, esto se hace mediante el comando IP/MASC, procurando no repetir ninguna.


VPC1:

	VPC1> ip 192.168.111.2/24  
	
	Checking for duplicate address...  
	
	VPC1 : 192.168.111.2 255.255.255.0
	
	VPC1 : 

VPC2:

	VPC2> ip 192.168.111.3/24  
	
	Checking for duplicate address...  
	
	VPC2 : 192.168.111.3 255.255.255.0  
	  
	VPC2>

Finalmente comprobaremos la correcta conexión mediante el comando PING, el cual le envía 5 paquetes a la ip, su estructura es *PING IP:

VPC1:

	VPC1> ping 192.168.111.3  
	
	84 bytes from 192.168.111.3 icmp_seq=1 ttl=64 time=0.798 ms  
	
	84 bytes from 192.168.111.3 icmp_seq=2 ttl=64 time=0.942 ms  
	
	84 bytes from 192.168.111.3 icmp_seq=3 ttl=64 time=1.256 ms  
	
	84 bytes from 192.168.111.3 icmp_seq=4 ttl=64 time=1.009 ms  
	
	84 bytes from 192.168.111.3 icmp_seq=5 ttl=64 time=0.798 ms

VPC2:

	VPC2> ping 192.168.111.2  
	  
	84 bytes from 192.168.111.2 icmp_seq=1 ttl=64 time=0.980 ms  
	
	84 bytes from 192.168.111.2 icmp_seq=2 ttl=64 time=1.155 ms  
	
	84 bytes from 192.168.111.2 icmp_seq=3 ttl=64 time=0.792 ms  
	
	84 bytes from 192.168.111.2 icmp_seq=4 ttl=64 time=1.109 ms  
	
	84 bytes from 192.168.111.2 icmp_seq=5 ttl=64 time=0.702 ms


En caso de una mala configuración las salidas del comando *PING* será la siguiente:

VPCx:

	VPCx> ping 192.168.111.x  
	  
	host (192.168.111.x) not reachable

**IMPORTANTE**

Para guardar se utiliza el comando save, sino cuando se apague el dispositivo se perderá la configuración:

