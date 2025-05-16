
---

# 📜 Inter-VLAN Routing - Router on a Stick (Mikrotik)

> Documentación técnica para configuración y estudio del laboratorio de redes con **Router on a Stick**.

---

## 📊 Objetivo
Permitir la comunicación entre dispositivos ubicados en distintas VLANs mediante subinterfaces de un router Mikrotik y enlaces troncales con switches Cisco. Además se configurará acceso a Internet.

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

## 💻 Mikrotik - Configuración completa

### Interfaces físicas
- `ether1`: hacia Internet
- `ether2`: trunk hacia Switch26 (VLANs 1XX, 2XX, 3XX)

### Subinterfaces VLAN
```bash
/interface vlan
add name=vlan1xx vlan-id=1xx interface=ether2 comment="VLAN PING"
add name=vlan2xx vlan-id=2xx interface=ether2 comment="VLAN PONG"
```

### IPs Gateway
```bash
/ip address
add address=10.1xx.xx.1/24 interface=vlan1xx comment="Gateway VLAN 1xx"
add address=10.2xx.xx.1/24 interface=vlan2xx comment="Gateway VLAN 2xx"
```

### DHCP Server para VLAN 118
```bash
/ip pool
add name=dhcp_pool_1xx ranges=10.1xx.xx.10-10.1xx.xx.50

/ip dhcp-server
add name=dhcp_vlan1xx interface=vlan1xx address-pool=dhcp_pool_1xx disabled=no

/ip dhcp-server network
add address=10.1xx.xx.0/24 gateway=10.1xx.xx.1 dns-server=8.8.8.8
```

### NAT (Salida a Internet)
```bash
/ip firewall nat
add chain=srcnat out-interface=ether1 action=masquerade comment="NAT"
```

---

## 🚗 VPCs - Configuración de PCs simuladas

### VPC29 (VLAN 1XX)
- IP: Automática por DHCP
- Ejemplo de IP esperada: 10.1xx.xx.10-50
- Gateway: 10.1xx.xx.1
- DNS: 8.8.8.8

### VPC30 (VLAN 2XX)
Configuración manual:
```text
IP: 10.2xx.xx.2
Máscara: 255.255.255.0
Gateway: 10.2xx.xx.1
DNS: 8.8.8.8
```

---

## 🛠️ Switch Cisco - Configuración

### Switch26 (Core - troncales)
```bash
vlan 1xx
vlan 2xx
vlan 3xx

interface e0/0
 switchport mode trunk
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 3xx

interface e0/1
 switchport mode trunk
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 3xx

interface e0/2
 switchport mode trunk
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 3xx
```

### Switch27 (Acceso a VPC30 - VLAN 218)
```bash
vlan 2xx
vlan 3xx

interface e0/0
 switchport mode trunk
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 3xx

interface e0/1
 switchport mode access
 switchport access vlan 2xx
```

### Switch28 (Acceso a VPC29 - VLAN 118)
```bash
vlan 1xx
vlan 3xx

interface e0/0
 switchport mode trunk
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 3xx

interface e0/1
 switchport mode access
 switchport access vlan 1xx
```

---

## 🔎 Comandos de Verificación

### En Mikrotik
```bash
/interface vlan print
/ip address print
/ip dhcp-server print
/ip route print
/ping 8.8.8.8
```

### En VPC
```bash
show ip
ping 10.1xx.xx.1
ping 10.2xx.xx.1
ping 8.8.8.8
```


---

> 📖 *Documentación hecha para repaso - inter VLAN Routing - Router (Mikrotik) on a Stick*
