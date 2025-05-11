## REQUISITOS DEL LABORATORIO
### 🧩 OBJETIVO GENERAL

Implementar un esquema de enrutamiento entre VLANs (Router-on-a-Stick) usando un switch Cisco L2, un router Cisco y 3 tipos de dispositivos finales, con:
- **Inter-VLAN Routing**
- **Servidor DHCP por VLAN**
- **NAT para salida a Internet**
- **VLAN nativa para trunk**
---
### 💪 TOPOLOGÍA Y ELEMENTOS
**Dispositivos usados:**

| Dispositivo         | Función                         |
| ------------------- | ------------------------------- |
| Switch Cisco IOL L2 | Segmentación VLAN               |
| Router Cisco IOSv   | Enrutamiento, DHCP, NAT         |
| Alpine Linux XFCE   | Cliente DHCP (VLAN 25XX)        |
| Windows Server      | Cliente DHCP (VLAN 26XX)        |
| VPC                 | Cliente IP estática (VLAN 27XX) |

**Conexiones:**
|        |          |         |          |
| ------ | -------- | ------- | -------- |
| Origen | Interfaz | Destino | Interfaz |
| Switch | e0/0     | Router  | e0/1     |
| Switch | e0/1     | Alpine  | eth0     |
| Switch | e0/2     | WinSrv  | eth0     |
| Switch | e0/3     | VPC     | eth0     |

**VLANs utilizadas:**
|      |         |                |                     |
| ---- | ------- | -------------- | ------------------- |
| VLAN | Nombre  | Subred         | Tipo                |
| 25XX | ALFA    | 172.25.XX.0/24 | Cliente DHCP        |
| 26XX | OMEGA   | 172.26.XX.0/24 | Cliente DHCP        |
| 27XX | EPSILON | 172.20.XX.0/24 | Cliente IP estática |
| 28XX | NATIVA  | - (sin IP)     | VLAN nativa trunk   |

![Inter VLAN](Imagenes/6-Inter%20VLAN%20Routing%20-%20Router%20(Cisco)%20on%20a%20Stick/6-Inter%20VLAN%20Routing%20-%20Router%20(Cisco)%20on%20a%20Stick.png)


---
## 🔧 CONFIGURACIÓN DEL SWITCH L2

### 1. Crear VLANs
```batch
vlan 25XX
 name ALFA
vlan 26XX
 name OMEGA
vlan 27XX
 name EPSILON
vlan 28XX
 name NATIVA
```

### 2. Asignar puertos de acceso a VLANs

```batch
interface e0/1
 switchport mode access
 switchport access vlan 25XX
 spanning-tree portfast

interface e0/2
 switchport mode access
 switchport access vlan 26XX
 spanning-tree portfast

interface e0/3
 switchport mode access
 switchport access vlan 27XX
 spanning-tree portfast
```
### 3. Configurar el puerto trunk

```batch
interface e0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 28XX
 switchport nonegotiate
```

---
## 🚀 CONFIGURACIÓN DEL ROUTER CISCO

### 1. Crear subinterfaces con gateway para cada VLAN

```batch
interface e0/1.25XX
 encapsulation dot1Q 25XX
 ip address 172.25.XX.1 255.255.255.0
 no shutdown

interface e0/1.26XX
 encapsulation dot1Q 26XX
 ip address 172.26.XX.1 255.255.255.0
 no shutdown

interface e0/1.27XX
 encapsulation dot1Q 27XX
 ip address 172.20.XX.1 255.255.255.0
 no shutdown

interface e0/1
 no shutdown
```
### 2. Configurar DHCP por VLAN

```batch
ip dhcp excluded-address 172.25.XX.1 172.25.XX.10
ip dhcp excluded-address 172.26.XX.1 172.26.XX.10

ip dhcp pool VLAN25XX
 network 172.25.XX.0 255.255.255.0
 default-router 172.25.XX.1
 dns-server 8.8.8.8

ip dhcp pool VLAN26XX
 network 172.26.XX.0 255.255.255.0
 default-router 172.26.XX.1
 dns-server 8.8.8.8
```

### 3. Configurar NAT para salida a internet

```batch
interface e0/0
 ip address dhcp
 ip nat outside
 no shutdown

interface e0/1
 ip nat inside
interface e0/1.25XX
 ip nat inside
interface e0/1.26XX
 ip nat inside
interface e0/1.27XX
 ip nat inside

access-list 1 permit 172.25.XX.0 0.0.0.255
access-list 1 permit 172.26.XX.0 0.0.0.255
access-list 1 permit 172.20.XX.0 0.0.0.255

ip nat inside source list 1 interface e0/0 overload
```

## 💼 CONFIGURACIÓN VPC (VLAN 2718 - IP ESTÁTICA)

```batch
ip 172.20.XX.10 255.255.255.0 172.20.XX.1
dns 8.8.8.8
```

## 🔌 PRUEBAS DE CONECTIVIDAD

### Desde VPC:

```batch
ping www.google.com   # Google
ping 8.8.8.8         # Salida a Internet
```

### Desde Alpine o Windows:

```batch
ping www.google.com   # Google
ping 8.8.8.8         # Salida a Internet
```


## 📄 RESULTADO ESPERADO

- Todos los dispositivos tienen conectividad entre sí (inter-VLAN routing)
    
- Alpine y Windows reciben IP por DHCP
    
- VPC usa IP estática y puede hacer ping
    
- Todos tienen salida a Internet mediante NAT en el router
    

---
**Luciano Toledo - Red de Laboratorio - Router on a Stick Router cisco**
