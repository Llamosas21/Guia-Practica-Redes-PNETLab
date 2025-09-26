
## REQUISITOS DEL LABORATORIO

### 🧩 OBJETIVO GENERAL

Implementar un esquema de enrutamiento entre VLANs (Router-on-a-Stick) utilizando un switch, un router Mikrotik y varios dispositivos finales, con:
- **Inter-VLAN Routing**
- **Servidor DHCP por VLAN**
- **NAT para salida a Internet**
- **VLAN nativa para trunk**

---

### 💪 TOPOLOGÍA Y ELEMENTOS

**Dispositivos usados:**

| Dispositivo        | Cantidad | Función                         | Imagen de Referencia |
|--------------------|----------|---------------------------------|-----------------------|
| Cloud0 (Management)| 1        | Conexión a Internet (ISP)       |                       |
| Windows Server     | 1        | Cliente DHCP (VLAN Secundaria)  |                       |
| Router Mikrotik    | 1        | Enrutamiento, DHCP, NAT         |                       |
| Computadora        | 1        | Cliente DHCP (VLAN Primaria)    |                       |
| Linux Server       | 1        | Cliente DHCP (VLAN Primaria)    | linux-alpine-xfce     |
| Switch             | 1        | Segmentación VLAN               |                       |

**Conexiones:**

![Inter VLAN](Imagenes/4-Inter-VLAN%20Routing%20con%20MikroTik%20usando%20Router-on-a-Stick%20y%20Switch/Esquema%204.png)

**VLANs utilizadas:**

| VLAN | Nombre          | Subred            | Máscara | Tipo              |
|------|-----------------|-------------------|---------|-------------------|
| 22XX | VLAN Primaria   | 172.22.xx.0/24    | /24     | Cliente DHCP      |
| 23XX | VLAN Secundaria | 172.23.xx.0/24    | /24     | Cliente DHCP      |
| 20XX | VLAN Admin      | 172.20.xx.0/24    | /24     | Cliente IP Estática |
| 24XX | VLAN Nativa     | (Para el Trunk)   | /24     | VLAN nativa trunk |

---

## 🔧 CONFIGURACIÓN DEL SWITCH

### 1. Crear VLANs

```batch
enable
conf t
vlan 22XX
 name VLAN_Primaria
exit
vlan 23XX
 name VLAN_Secundaria
exit
vlan 20XX
 name VLAN_Admin
exit
vlan 24XX
 name VLAN_Nativa
exit
```

### 2. Configurar los puertos en modo access

```batch
int eth0/1
 switchport mode access
 switchport access vlan 22XX
exit

int eth0/2
 switchport mode access
 switchport access vlan 23XX
exit

int eth0/3
 switchport mode access
 switchport access vlan 20XX
exit
```

### 3. Configurar el puerto eth0/0 en TRUNK

```batch
int e0/0
 sw tr enc dot1q
 sw mo tr
 sw tr nat vlan 24XX
 sw tr all vlan 20XX,22XX,23XX,24XX
exit
exit
wr
```

### 4. Comprobar configuración de VLANs

```batch
exit
sh vl br
```

**Resultado esperado:**

```
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active
1002 fddi-default                     act/unsup
1003 token-ring-default               act/unsup
1004 fddinet-default                  act/unsup
1005 trnet-default                    act/unsup
2009 VLAN_Admin                       active    Et0/3
2209 VLAN_Primaria                    active    Et0/1
2309 VLAN_Secundaria                  active    Et0/2
2409 VLAN_Nativa                      active
```

### 5. Comprobar configuración del puerto Trunk

```batch
Switch#sh int eth0/0 sw
```

**Resultado esperado:**

```
Switchport: Enabled
Administrative Mode: trunk
Operational Mode: trunk
Administrative Trunking Encapsulation: dot1q
Operational Trunking Encapsulation: dot1q
```

---

## 🚀 CONFIGURACIÓN DEL ROUTER MIKROTIK

**IMPORTANTE:**

La primera vez que se conecte, va a pedir un usuario (admin) y una contraseña que por defecto está vacía, luego va a aparecer algo como esto:

```
Do you want to see the software license? [Y/n]: n
```

En donde se recomienda no ver la licencia.

Finalmente se les pedirá una nueva contraseña luego del primer Enter se les pedirá que la escriban una vez más, si ambas coinciden mostrará este mensaje:

```
Change your password

new password> *****
repeat new password> *****

Password changed
[admin@MikroTik] >
```

### 1. Crear las interfaces VLAN sobre la física definida en el Switch

```batch
/interface vlan
add interface=ether2 name=vlan_primaria vlan-id=22XX
add interface=ether2 name=vlan_secundaria vlan-id=23XX
add interface=ether2 name=vlan_admin vlan-id=20XX
add interface=ether2 name=vlan_nativa vlan-id=24XX
```

### 2. Asignar IPs a las Interfaces VLAN del Mikrotik

```batch
/ip address
add address=172.22.xx.1/24 interface=vlan_primaria
add address=172.23.xx.1/24 interface=vlan_secundaria
add address=172.20.xx.1/24 interface=vlan_admin
```

*Las IPs X.X.X.1 son las puertas de enlace (gateway), para cada VLAN.*

### 3. Crear servidores DHCP por VLAN

**DHCP Primaria**

```batch
/ip pool
add name=dhcp_pool_2209 ranges=172.22.xx.100-172.22.xx.200

/ip dhcp-server
add name=dhcp_primaria interface=vlan_primaria address-pool=dhcp_pool_2209 lease-time=1h disabled=no

/ip dhcp-server network
add address=172.22.xx.0/24 gateway=172.22.xx.1 dns-server=8.8.8.8
```

**DHCP Secundaria**

```batch
/ip pool
add name=dhcp_pool_2309 ranges=172.23.xx.100-172.23.xx.200

/ip dhcp-server
add name=dhcp_secundaria interface=vlan_secundaria address-pool=dhcp_pool_2309 lease-time=1h disabled=no

/ip dhcp-server network
add address=172.23.xx.0/24 gateway=172.23.xx.1 dns-server=8.8.8.8
```

### 4. Configurar NAT para Internet (Firewall)

```batch
/ip firewall nat
add chain=srcnat out-interface=ether1 action=masquerade
/system backup save
```

*El puerto a seleccionar (ether1) debe ser el que se conecta con el ISP (Cloud0 en este caso).*

---

## 💼 CONFIGURACIÓN VPC (VLAN Admin - IP ESTÁTICA)

```batch
ip 172.20.xx.xx 255.255.255.0 172.20.xx.1
```

---

## 🔌 PRUEBAS DE CONECTIVIDAD

### Desde la Computadora (VLAN Primaria - DHCP):

- Asegurarse de que la computadora reciba una dirección IP mediante DHCP en el rango 172.22.17.100 - 172.22.17.200.
- Abrir la consola o terminal y ejecutar:
  ```batch
  ping 172.22.xx.xx   # Ping al gateway de la VLAN Primaria
  ping 172.23.xx.xx   # Ping al gateway de la VLAN Secundaria (Inter-VLAN)
  ping 172.20.xx.xx   # Ping al gateway de la VLAN Admin (Inter-VLAN)
  ping 8.8.8.8      # Ping a un servidor DNS público (Prueba de salida a Internet)
  ping [www.google.com](https://www.google.com) # Ping a un sitio web (Prueba de salida a Internet)
  ```

### Desde Windows Server (VLAN Secundaria - DHCP):

- Asegurarse de que el servidor reciba una dirección IP mediante DHCP en el rango 172.23.xx.100 - 172.23.xx.200.
- Abrir el símbolo del sistema y ejecutar:
  ```batch
  ping 172.23.xx.1   # Ping al gateway de la VLAN Secundaria
  ping 172.22.xx.1   # Ping al gateway de la VLAN Primaria (Inter-VLAN)
  ping 172.20.xx.1   # Ping al gateway de la VLAN Admin (Inter-VLAN)
  ping 8.8.8.8      # Ping a un servidor DNS público (Prueba de salida a Internet)
  ping [www.google.com](https://www.google.com) # Ping a un sitio web (Prueba de salida a Internet)
  ```

### Desde el dispositivo con IP estática (VLAN Admin):

- Asegurarse de que el dispositivo tenga la IP configurada: 172.20.17.10 con máscara 255.255.255.0 y gateway 172.20.17.1.
- Abrir la consola o terminal y ejecutar:
  ```batch
  ping 172.20.xx.1   # Ping al gateway de la VLAN Admin
  ping 172.22.xx.1   # Ping al gateway de la VLAN Primaria (Inter-VLAN)
  ping 172.23.xx.1   # Ping al gateway de la VLAN Secundaria (Inter-VLAN)
  ping 8.8.8.8      # Ping a un servidor DNS público (Prueba de salida a Internet)
  ping [www.google.com](https://www.google.com) # Ping a un sitio web (Prueba de salida a Internet)
  ```

---
*Luciano Toledo - Santiago Llamosas - Red de Laboratorio - Inter-VLAN Routing con MikroTik usando Router on a Stick y Switch*

