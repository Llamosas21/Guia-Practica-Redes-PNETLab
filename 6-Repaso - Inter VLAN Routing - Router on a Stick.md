

# 📜 Inter-VLAN Routing - Router on a Stick (Mikrotik)

> Documentación técnica para configuración y estudio del lab de redes con **Router on a Stick**.

---

## 📊 Objetivo
Permitir la comunicación entre dispositivos ubicados en distintas VLANs mediante subinterfaces de un router Mikrotik y enlaces troncales con switches Cisco. Además se configurará salida a Internet.

---

## 🛋 Topología

![Inter VLAN](Imagenes/6-Repaso%20-%20Inter%20VLAN%20Routing%20-%20Router%20on%20a%20Stick/Repaso%20-%20Inter%20VLAN%20Routing%20-%20Router%20on%20a%20Stick.png)
- `ether1`: WAN (Internet)
- `ether2`: Trunk hacia Switch26
- VLANs usadas: 118 (PING), 218 (PONG), 318 (NATIVA)

---

## 📃 VLANs 

| Nombre | VLAN ID | Subred         | Tipo     | Dirección Gateway |
| ------ | ------- | -------------- | -------- | ----------------- |
| PING   | 1XX     | 10.1xx.xx.0/24 | Dinámica | 10.1xx.xx.1       |
| PONG   | 2XX     | 10.2xx.xx.0/24 | Estática | 10.2xx.xx.1       |
| NATIVA | 3XX     | (sólo troncal) | Nativa   | -                 |

---

## 🔎 Comandos de Verificación

### En Mikrotik
/interface vlan print
/ip address print
/ip dhcp-server print
/ip route print
/ping 8.8.8.8

### En VPC
show ip
ping 10.109.9.1
ping 10.209.9.1
ping 8.8.8.8

---
---

## 💻 Mikrotik - Configuración completa

### Interfaces físicas
- `ether1`: hacia Internet
- `ether2`: trunk hacia Switch26 (VLANs 1XX, 2XX, 3XX)

### Subinterfaces VLAN
/interface vlan
add name=vlan1xx vlan-id=1xx interface=ether2 comment="VLAN PING"
add name=vlan2xx vlan-id=2xx interface=ether2 comment="VLAN PONG"

### IPs Gateway
/ip address
add address=10.1xx.xx.1/24 interface=vlan1xx comment="GW VLAN 1xx"
add address=10.2xx.xx.1/24 interface=vlan2xx comment="GW VLAN 2xx"

### DHCP Server para VLAN 118
/ip pool
add name=dhcp_pool_1xx ranges=10.1xx.xx.10-10.1xx.xx.50

/ip dhcp-server
add name=dhcp_vlan1xx interface=vlan1xx address-pool=dhcp_pool_1xx disabled=no

/ip dhcp-server network
add address=10.1xx.xx.0/24 gateway=10.1xx.xx.1 dns-server=8.8.8.8

### NAT (salida a Internet)
/ip firewall nat
add chain=srcnat out-interface=ether1 action=masquerade comment="NAT"

---

## VPCs - Configuración de PCs simuladas

### VPC29 (VLAN 1XX)
- IP: automática (DHCP)
- Ej. IP esperada: 10.1xx.xx.10-50
- GW: 10.1xx.xx.1
- DNS: 8.8.8.8

### VPC30 (VLAN 2XX)
Config. manual:
IP: 10.2xx.xx.2
Máscara: 255.255.255.0
GW: 10.2xx.xx.1
DNS: 8.8.8.8

---

## 🛠️ Switch Cisco - Config

### Switch26 (Core - troncales)
vlan 1xx
vlan 2xx
vlan 3xx

int e0/0
 switchport mode trunk
 switchport trunk encap dot1q
 switchport trunk native vlan 3xx

int e0/1
 switchport mode trunk
 switchport trunk encap dot1q
 switchport trunk native vlan 3xx

int e0/2
 switchport mode trunk
 switchport trunk encap dot1q
 switchport trunk native vlan 3xx

### Switch27 (Acceso a VPC30 - VLAN 218)
vlan 2xx
vlan 3xx

int e0/0
 switchport mode trunk
 switchport trunk encap dot1q
 switchport trunk native vlan 3xx

int e0/1
 switchport mode access
 switchport access vlan 2xx

### Switch28 (Acceso a VPC29 - VLAN 118)
vlan 1xx
vlan 3xx

int e0/0
 switchport mode trunk
 switchport trunk encap dot1q
 switchport trunk native vlan 3xx

int e0/1
 switchport mode access
 switchport access vlan 1xx

---


# 📜 Inter-VLAN Routing - Router on a Stick (Cisco)

> Documentación técnica para configuración y estudio del lab de redes con **Router on a Stick**.

---

## 📊 Objetivo
Permitir la comunicación entre dispositivos ubicados en distintas VLANs mediante subinterfaces de un router Cisco y enlaces troncales con switches Cisco. Además se configurará salida a Internet.

---

## 🛋 Topología

![Inter VLAN](Imagenes/6-Repaso%20-%20Inter%20VLAN%20Routing%20-%20Router%20on%20a%20Stick/Repaso%20-%20Inter%20VLAN%20Routing%20-%20Router%20on%20a%20Stick.png)
- `ether1`: WAN (Internet)
- `ether2`: Trunk hacia Switch26
- VLANs usadas: 118 (PING), 218 (PONG), 318 (NATIVA)

---

int e0/0
 ip address dhcp
 no shut

int e0/1
 no shut
 no ip addr

int e0/1.109
 encap dot1Q 109
 ip addr 10.109.9.1 255.255.255.0

int e0/1.209
 encap dot1Q 209
 ip addr 10.209.9.1 255.255.255.0

ip nat inside source list 1 int e0/0 overload

access-list 1 permit 10.109.9.0 0.0.0.255
access-list 1 permit 10.209.9.0 0.0.0.255

int e0/0
 ip nat outside

int e0/1.109
 ip nat inside

int e0/1.209
 ip nat inside

ip dhcp pool VLAN109
 network 10.109.9.0 255.255.255.0
 default-router 10.109.9.1
 dns-server 8.8.8.8

> 📖 *Doc técnica para repaso - Inter-VLAN Routing - Router (Mikrotik y Cisco) on a Stick*
"""
