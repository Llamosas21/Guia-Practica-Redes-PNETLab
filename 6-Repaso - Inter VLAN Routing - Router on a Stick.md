# 📜 Inter-VLAN Routing - Router on a Stick (Mikrotik)

> Documentación técnica para configuración y estudio del lab de redes con **Router on a Stick**.

---

## 📊 Objetivo
Permitir la comunicación entre dispositivos ubicados en distintas VLANs mediante subinterfaces de un router Mikrotik y enlaces troncales con switches Cisco. Además se configurará salida a Internet.

---

## 🛋 Topología

![Inter VLAN](Imagenes/6-Inter-VLAN%20Routing%20-%20Router%20on%20a%20Stick%20(Mikrotik%20y%20Cisco)/Esquema%206A.png)

---

## 📃 VLANs 

| Nombre | VLAN ID | Subred         | Tipo     | Dirección Gateway |
| ------ | ------- | -------------- | -------- | ----------------- |
| PING   | 1XX     | 10.1XX.XX.0/24 | Dinámica | 10.1XX.XX.1       |
| PONG   | 2XX     | 10.2XX.XX.0/24 | Estática | 10.2XX.XX.1       |
| NATIVA | 3XX     | (sólo troncal) | Nativa   | -                 |

---

## 🔎 Comandos de Verificación

### En Mikrotik
```batch 
/interface vlan print
/ip address print
/ip dhcp-server print
/ip route print
/ping 8.8.8.8
```
### En VPC
```batch 
show ip
ping 10.10X.XX.1
ping 10.20X.XX.1
ping 8.8.8.8
```
---

## 💻 Mikrotik - Configuración completa

### Interfaces físicas
- `ether1`: hacia Internet
- `ether2`: trunk hacia Switch 1 (VLANs 1XX, 2XX, 3XX)

### Subinterfaces VLAN
```batch 
/interface vlan
add name=vlan1XX vlan-id=1XX interface=ether2 comment="VLAN PING"
add name=vlan2XX vlan-id=2XX interface=ether2 comment="VLAN PONG"
```
### IPs Gateway
```batch 
/ip address
add address=10.1XX.XX.1/24 interface=vlan1XX comment="GW VLAN 1XX"
add address=10.2XX.XX.1/24 interface=vlan2XX comment="GW VLAN 2XX"
```
### DHCP Server para VLAN 11X
```batch 
/ip pool
add name=dhcp_pool_1XX ranges=10.1XX.XX.10-10.1XX.XX.50

/ip dhcp-server
add name=dhcp_vlan1XX interface=vlan1XX address-pool=dhcp_pool_1XX disabled=no

/ip dhcp-server network
add address=10.1XX.XX.0/24 gateway=10.1XX.XX.1 dns-server=8.8.8.8
```
### NAT (salida a Internet)
```batch 
/ip firewall nat
add chain=srcnat out-interface=ether1 action=masquerade comment="NAT"
```
---

## VPCs - Configuración de PCs simuladas

### VPC 1 (VLAN 1XX)
- IP: automática (DHCP)

### VPC 2 (VLAN 2XX)
Config. manual:
```batch 
ip 10.2XX.XX.10 255.255.255.0 10.2XX.XX.1
```

---

## 🛠️ Switch Cisco - Config

### Switch 1 (Core - troncales)
```batch 
vlan 1XX
vlan 2XX
vlan 3XX

int e0/0
 switchport mode trunk
 switchport trunk encap dot1q
 switchport trunk native vlan 3XX

int e0/1
 switchport mode trunk
 switchport trunk encap dot1q
 switchport trunk native vlan 3XX

int e0/2
 switchport mode trunk
 switchport trunk encap dot1q
 switchport trunk native vlan 3XX
```
### Switch 2 (Acceso a VPC2 - VLAN 2XX)
```batch 
vlan 2XX
vlan 3XX

int e0/0
 switchport mode trunk
 switchport trunk encap dot1q
 switchport trunk native vlan 3XX

int e0/1
 switchport mode access
 switchport access vlan 2XX
```
### Switch 3 (Acceso a VPC 1 - VLAN 1XX)
```batch 
vlan 1XX
vlan 3XX

int e0/0
 switchport mode trunk
 switchport trunk encap dot1q
 switchport trunk native vlan 3XX

int e0/1
 switchport mode access
 switchport access vlan 1XX
```
---


# 📜 Inter-VLAN Routing - Router on a Stick (Cisco)

> Documentación técnica para configuración y estudio del lab de redes con **Router on a Stick**.

---

## 📊 Objetivo
Permitir la comunicación entre dispositivos ubicados en distintas VLANs mediante subinterfaces de un router Cisco y enlaces troncales con switches Cisco. Además se configurará salida a Internet.

---

## 🛋 Topología

![Inter VLAN](Imagenes/6-Inter-VLAN%20Routing%20-%20Router%20on%20a%20Stick%20(Mikrotik%20y%20Cisco)/Esquema%206B.png)

---

## Subinterfaces VLAN (Router on a Stick)
```batch 
int e0/0
 ip address dhcp
 no shut

int e0/1
 no shut
 no ip addr

int e0/1.1XX
 encap dot1Q 1XX
 ip addr 10.1XX.XX.1 255.255.255.0
```

## NAT para acceso a Internet
```batch 
int e0/1.2XX
 encap dot1Q 2XX
 ip addr 10.2XX.XX.1 255.255.255.0

ip nat inside source list 1 int e0/0 overload

access-list 1 permit 10.1XX.XX.0 0.0.0.255
access-list 1 permit 10.2XX.XX.0 0.0.0.255

int e0/0
 ip nat outside

int e0/1.1XX
 ip nat inside

int e0/1.2XX
 ip nat inside
```

## DHCP (para VLAN 1XX)
```batch 
ip dhcp pool VLAN1XX
 network 10.1XX.XX.0 255.255.255.0
 default-router 10.1XX.XX.1
 dns-server 8.8.8.8
```
---
## 🛠️ Switch Cisco - Config

### Switch 1 (Core - troncales)
```batch 
vlan 1XX
vlan 2XX
vlan 3XX

int e0/0
 switchport mode trunk
 switchport trunk encap dot1q
 switchport trunk native vlan 3XX

int e0/1
 switchport mode trunk
 switchport trunk encap dot1q
 switchport trunk native vlan 3XX

int e0/2
 switchport mode trunk
 switchport trunk encap dot1q
 switchport trunk native vlan 3XX
```
### Switch 2 (Acceso a VPC2 - VLAN 2XX)
```batch 
vlan 2XX
vlan 3XX

int e0/0
 switchport mode trunk
 switchport trunk encap dot1q
 switchport trunk native vlan 3XX

int e0/1
 switchport mode access
 switchport access vlan 2XX
```
### Switch 3 (Acceso a VPC 1 - VLAN 1XX)
```batch 
vlan 1XX
vlan 3XX

int e0/0
 switchport mode trunk
 switchport trunk encap dot1q
 switchport trunk native vlan 3XX

int e0/1
 switchport mode access
 switchport access vlan 1XX
```

---
*Luciano Toledo - Santiago Llamosas - Red de Laboratorio - Inter VLAN Routing Router on a Stick*
