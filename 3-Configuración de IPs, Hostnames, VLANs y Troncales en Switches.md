#### Requisitos:

- Computadoras: 4
- Switch: 2

#### Conexión:

Se procederá a hacer las conexiones físicas entre los Switches y luego entre las VPCs, para luego prender todos los dispositivos.

Para esta conexión se utilizarán los siguientes rangos:

	Número de VLAN: 100
	Nombre: ESTUDIANTES
	IP red: 172.20.100.0/16

	Número de VLAN: 200
	Nombre: PROFESORES
	IP red: 172.20.200.0/16

![Inter VLAN](Imagenes/3-Configuración%20de%20IPs%2C%20Hostnames%2C%20VLANs%20y%20Troncales%20en%20Switches/Esquema%203.png)


#### PASOS A SEGUIR:
- Procederemos a [[1-Conexión entre dos VPCs por medio de un Switch#Asignar IP a las VPCs|asignarles la IP]] de cada una de las VPCs. 
- Luego procederemos  [[2-Comunicación entre dos VLANs gestionadas por un Switch#Crear VLANs en Switch (Capa 2)|crear VLANs en cada Switch.]]
- [[#Configuración de puertos de acceso|Configurar los puertos de Acceso en según corresponda.]]
- [[#Configuración de puertos del Switch Trunk|Configurar los puertos Trunk de cada Switch]]


### PUERTOS [[#Configuración de puertos de acceso|ACCESSO]]-[[#Configuración de puertos del Switch Trunk|TRUNK]]

Existen dos formas principales de configurar los puertos en un switch: en modo **trunk** y en modo **acceso**.

#### Modo acceso (access)

El modo **acceso** se utiliza cuando el puerto del switch se conecta directamente a un dispositivo final, como:

- Computadoras
- Impresoras
- Cámaras
- Servidores

Este tipo de puerto solo puede pertenecer a una **única VLAN**. Es ideal cuando el dispositivo conectado no necesita participar en múltiples VLANs.

**Características principales:**
- Solo maneja tráfico de una VLAN específica.
- Simplifica la configuración para dispositivos finales.
- El tráfico que pasa por el **no esta etiquetado**  

---

#### Modo trunk

El modo **trunk** se usa cuando el puerto del switch se conecta a otro **switch** o a un **router** que necesita transportar tráfico de varias VLANs simultáneamente.

Este tipo de conexión permite el paso de múltiples VLANs a través del mismo enlace físico, etiquetando los paquetes para indicar a qué VLAN pertenecen (usualmente con el protocolo 802.1Q).

**Características principales:**
- Transporta tráfico de **múltiples VLANs**.
- Utiliza tráfico **etiquetado VLAN** (VLAN tagging).
- Esencial para la comunicación entre switches y redes más complejas.

---

#### Diferencias clave

| Característica   | Modo Acceso           | Modo Trunk             |
| ---------------- | --------------------- | ---------------------- |
| Tipo de conexión | Dispositivos finales  | Switches, routers      |
| VLANs soportadas | Una sola VLAN         | Múltiples VLANs        |
| Etiquetado VLAN  | No                    | Sí (802.1Q)            |
| Uso común        | PCs, impresoras, etc. | Enlaces entre switches |

### VLAN Trunking Protocols

Cuando configuramos un trunk, necesitamos definir el **protocolo de encapsulación** que transportará varias VLANs por un solo enlace.

#### Tipos de Encapsulaciones

|Protocolo|Creador|Estado|Compatibilidad|
|---|---|---|---|
|**ISL** (Inter-Switch Link)|Cisco|Obsoleto|Solo Cisco|
|**802.1Q** (dot1q)|IEEE|Vigente|Multivendor|

---

#### ISL (Inter-Switch Link)

- Encapsula **completamente** la trama Ethernet original.
- Agrega **26 bytes** de encabezado y **4 bytes** de trailer (30 bytes en total).
- Solo funciona en **equipos Cisco**.
- Actualmente **en desuso**.

**Configuración:**

	Switch(config-if)# switchport trunk encapsulation isl
	Switch(config-if)# switchport mode trunk`

**Abreviado:**

	Switch(config-if)# sw tr en isl
	Switch(config-if)# sw mo tr


---

#### 802.1Q (dot1q)

- Inserta un **campo de 4 bytes** en el encabezado Ethernet (no encapsula todo).
- Permite el tráfico entre **equipos de diferentes fabricantes**.
- Es el **estándar actual** para trunking.

**Configuración:**
	
	Switch(config-if)# switchport trunk encapsulation dot1q
	Switch(config-if)# switchport mode trunk


**Abreviado:**

	Switch(config-if)# sw tr en do
	Switch(config-if)# sw mo tr


---

#### Diferencias entre ISL y 802.1Q

|Característica|ISL|802.1Q|
|---|---|---|
|¿Encapsula la trama completa?|✅|❌|
|Overhead agregado|30 bytes|4 bytes|
|¿Estandarizado?|❌|✅|
|Estado actual|Obsoleto|Vigente|
|Compatibilidad|Solo Cisco|Multivendor|

---

#### Native VLAN en 802.1Q

- **Tráfico sin etiquetar**.
    
- Se define con:
    

	Switch(config-if)# switchport trunk native vlan <VLAN_ID>	
	Switch(config-if)# switchport trunk native vlan 99`

---

#### Notas Finales

- Hoy en día **se usa exclusivamente 802.1Q**.
- Algunos switches modernos **no requieren configurar encapsulation**, usan **dot1q** por defecto.
- Siempre verificar compatibilidad si trabajás con equipos de diferentes marcas.

### Configuración de puertos de acceso

Se accede al modo privilegiado del switch, luego se ingresa a la configuración global y se configura la interfaz Ethernet 0/0 en modo acceso, asignándola a la VLAN 100
	
	Switch> enable
	
	Switch# conf t
	
	Switch(config)# int et0/0
	
	Switch(config-if)# sw mo acc
	
	Switch(config-if)# sw acc vlan 100
	
	Switch(config-if)# exit
	
	Switch(config)# end
	
	Switch#wr
	
- **enable**: Accede al modo privilegiado (similar a "root").
- **conf t**: Entra al modo de configuración global.
- **int et0/1**: Selecciona la interfaz Ethernet 0/1.
- **sw mo acc**: Configura la interfaz en modo acceso.
- **sw acc vlan 100**: Asigna la interfaz a la VLAN 100.
- **exit**: Sale de la configuración de la interfaz.
- **end**: Sale de la configuración global
- **wr**: Guarda la configuración. Este comando puede ejecutarse cada vez que se cree una interfaz o al finalizar toda la configuración, siempre que el switch este con el pront: **Switch#wr**.}

#### Extra

para cuando se deben configurar más de un puerto ya sea en modo access o trunk lo que se puede hacer es 

	Switch#conf t
	
	Switch(config)#int range e0/1 - 2

- **int**: versión abreviada de Interfaz.
- **range**: define un rango:
- **e0/1 - 2**: El rango va de eN/ - n

##### ¿Existen limitaciones?

Sí, **algunas importantes**:

- **Las interfaces deben ser del mismo tipo** (todas Ethernet, todas FastEthernet, todas GigabitEthernet, etc.). No podés mezclar, por ejemplo, `e0/1` con `g0/1` en el mismo rango.

- **El rango debe ser válido**: si escribís un puerto que no existe en el switch, te va a tirar error.

- **Cuidado con las configuraciones**: como afecta a todas las interfaces seleccionadas, si metés un comando mal, podrías romper varias conexiones al mismo tiempo.

- **No se pueden hacer ranges discontinuos directamente**: por ejemplo, no podés hacer `int range e0/1, e0/3`. Para eso deberías usar comas, pero depende del IOS del switch (algunos sí soportan `int range e0/1, e0/3`, otros no).

### Configuración de puertos del Switch Trunk

	Switch# conf t
	
	Switch(config)# int e0/0
	
	Switch(config-if)# sw tr en do
	
	Switch(config-if)# sw tru nat vlan 300
	
	Switch(config-if)# sw tru all vlan 100,200,300  
	
	Switch(config-if)# exit


- **int e0/0**: Es la interfaz Ethernet donde se va a pasar el tráfico **etiquetado** (tagged).
- **sw tr en do**: Se configura el **tipo de encapsulamiento** del enlace como **802.1Q** (dot1q), el estándar recomendado para trunking. [[#VLAN Trunking Protocols|Ampliar]]
- **swi tru nat vlan 300**:  Se establece la **VLAN nativa** del trunk en la **VLAN 300**. (La VLAN nativa es aquella cuyo tráfico circula **sin etiquetas** en el enlace trunk.) 
- **swi tru all vlan 100,200,300**:  Se permite que el tráfico de **solo las VLANs 100, 200 y 300** pase a través del trunk. (Controla qué VLANs están **autorizadas** para circular por ese enlace.) 

#### Importante
Ambos Switches deberán estar conectados en modo **tronkal** y bajo el mismo protocolo de encapsulación, si no configuras el **Allowed**, por defecto pasan todas las VLANs por tales puertos.

#### Error: Native VLAN mismatch

El mensaje de error **"Native VLAN mismatch"** se produce cuando dos dispositivos de red configurados para formar un enlace troncal tienen diferentes configuraciones de VLAN nativa.

**Por ejemplo**: En un enlace troncal, intervienen dos switches:

- Switch 1: VLAN nativa 1
- Switch 2: VLAN nativa 300

Para resolver este error, es necesario definir el ID de la VLAN nativa que se empleará y configurarlo de manera idéntica en ambos extremos del enlace troncal.

#### Comprobar que esté bien configurada:

	Switch#sh int et0/0 sw  
	
**Aparecerá algo como esto:**

	Name: Et0/0  
	Switchport: Enabled  
	Administrative Mode: trunk  
	Operational Mode: trunk  
	Administrative Trunking Encapsulation: dot1q  
	Operational Trunking Encapsulation: dot1q  
	Negotiation of Trunking: On  
	Access Mode VLAN: 1 (default)  
	Trunking Native Mode VLAN: 1 (default)  
	Administrative Native VLAN tagging: enabled  
	Voice VLAN: none  
	Administrative private-vlan host-association: none    
	Administrative private-vlan mapping: none    
	Administrative private-vlan trunk native VLAN: none  
	Administrative private-vlan trunk Native VLAN tagging: enabled  
	Administrative private-vlan trunk encapsulation: dot1q  
	Administrative private-vlan trunk normal VLANs: none  
	Administrative private-vlan trunk associations: none  
	Administrative private-vlan trunk mappings: none  
	Operational private-vlan: none  
	Trunking VLANs Enabled: 100,200  
	Pruning VLANs Enabled: 2-1001  
	Capture Mode Disabled  
	Capture VLANs Allowed: ALL  
	            
	Protected: false  
	Appliance trust: none  
	Switch#

