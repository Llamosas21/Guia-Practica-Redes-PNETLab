#### Requisitos:

- Computadoras: 4
- Switch: 1

#### Conexión:

Se procederá a hacer las conexiones físicas entre VPCs y Switch, para luego prender todos los dispositivos.

Para esta conexión se utilizarán los siguientes rangos:

	Número de VLAN: 100
	Nombre: ESTUDIANTES
	IP red: 172.20.100.0/16

	Número de VLAN: 200
	Nombre: PROFESORES
	IP red: 172.20.200.0/16


![[Esquema 1.png]]



En primer lugar procederemos a [[1-Conexión entre dos VPCs por medio de un Switch#Asignar IP a las VPCs|asignarles la IP]] de cada una de las VPCs.

### Crear VLANs en Switch (Capa 2)

	Switch>enable  
	
	Switch#conf t  
	Enter configuration commands, one per line.  End with CNTL/Z.  
	
	Switch(config)#ho nombreSwitch #Asigna un nombre al Switch (Buena práctica)
	
	Switch(config)#vlan 100  
	
	Switch(config-vlan)#name ESTUDIANTES 
	 
	Switch(config-vlan)#exit  
	
	
	Switch(config)#vlan 200  
	
	Switch(config-vlan)#name PROFESORES  
	
	Switch(config-vlan)#exit  

### VER VLANs

En el modo **enable** del switch es posible visualizar las VLANs creadas, así como los puertos activos en cada una de ellas. En este caso particular, aún no se han configurado los puertos, por lo que todas las conexiones al switch aparecen asignadas a la VLAN por defecto.

	Switch#sh vl br  
	  
	VLAN Name                             Status    Ports  
	---- -------------------------------- --------- -----------------------------
	1    default                          active    Et0/0, Et0/1, Et0/2, Et0/3  
	100  ESTUDIANTES                      active       
	200  PROFESORES                       active       
	1002 fddi-default                     act/unsup    
	1003 token-ring-default               act/unsup    
	1004 fddinet-default                  act/unsup    
	1005 trnet-default                    act/unsup    


