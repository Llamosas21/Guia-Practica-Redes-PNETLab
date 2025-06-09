## 🧠 PARTE 1: INFORME INFORMATIVO – HOTSPOT EN MIKROTIK SIN VLANs

### 🔹 Objetivo del Proyecto

El objetivo fue implementar un **portal cautivo (Hotspot)** utilizando un **router Mikrotik** dentro de una topología virtual en **EVE-NG**, sin usar VLANs, controlando el acceso a Internet por usuarios, perfiles y excepciones (bypass y bloqueos), con portal HTML personalizado.

---

### 🔹 Dispositivos utilizados en EVE-NG

|Dispositivo|Tipo / Imagen|Función principal|
|---|---|---|
|Mikrotik v7.x CHR|`chr-7.x.qcow2`|Servidor principal de Hotspot y DHCP|
|Cisco Switch|`IOSv-L2`|Conmutador para simular red física|
|Alpine Linux (x3)|`alpine-3.x-xfce.qcow2`|Dispositivos cliente (usuarios finales)|
|Nube de conexión|`Cloud0 (Mgmt)`|Simulación de acceso a Internet (WAN)|


![Hostpot](Imagenes/7-Portal%20Cautivo%20en%20Mikrotik/HOSTPOT%20(PORTAL%20CAUTIVO)%20MIKROTIK.png)

---

### 🔹 Esquema IP aplicado

- Red LAN: `10.xxx.xx.0/24`
    
- Gateway: `10.xxx.xx.1`
    
- DHCP Pool: `10.xxx.xx.100 - 10.xxx.xx.200`
    

---

### 🔹 ¿Por qué se usó Hotspot?

El Hotspot de Mikrotik permite:

- Redireccionar automáticamente a los usuarios hacia una **pantalla de login web**.
    
- Controlar el **ancho de banda**, tiempo de sesión, y permisos.
    
- Implementar **usuarios y perfiles diferenciados**.
    
- Permitir excepciones sin login (bypass por IP o MAC).
    
- Bloquear dispositivos no deseados.
    

---

### 🔹 ¿Por qué NO se usaron VLANs?

Aunque originalmente se solicitó trabajar con una VLAN 1xx, no se implementó por los siguientes motivos técnicos:

1. **Las interfaces VLAN virtuales (`etherX.1xx`) no reciben correctamente tráfico 802.1Q en EVE-NG.**
    
2. El portal Hotspot **queda cargando indefinidamente** porque no se intercepta el tráfico HTTP correctamente.
    
3. EVE-NG no interpreta VLANs tagged en enlaces directos entre dispositivos sin configuración adicional avanzada.
    
4. El Hotspot depende de capas L2/L3 (ARP, DNS, HTTP) que **no funcionan bien en subinterfaces VLAN simuladas**.
    

**Solución aplicada:** se utilizó directamente la **interfaz física `ether2` del Mikrotik como LAN**, obteniendo un funcionamiento 100% estable del portal cautivo.

---

### 🔹 Comportamiento esperado y validado

- Los clientes reciben IP automáticamente desde Mikrotik (DHCP).
    
- El navegador redirige automáticamente al portal (`wifi.xxxxx.lan`).
    
- Los usuarios acceden con login y contraseña, según su perfil.
    
- Dispositivos autorizados por MAC o IP navegan sin login.
    
- Dispositivos bloqueados no pueden navegar.
    
- Portal HTML está traducido y personalizado.
    

---

### 🔹 Perfiles y usuarios

|Perfil|Límite de velocidad|
|---|---|
|trial1|512 Kbps / 256 Kbps|
|ventas|2 Mbps / 512 Kbps|
|marketing|3 Mbps / 1 Mbps|
|invitados|1 Mbps / 256 Kbps|

|Usuario|Perfil|Contraseña|
|---|---|---|
|ventas1|ventas|itel|
|ventas2|ventas|itel|
|marketing1|marketing|itel|
|marketing2|marketing|itel|
|invitado1|invitados|itel|

---

### 🔹 Excepciones configuradas

| Tipo      | Identificador           | Acción                       |
| --------- | ----------------------- | ---------------------------- |
| Bypass    | MAC `XX:EE:BB:XX:AA:XX` | Navega sin login (Marketing) |
| Bypass    | IP `10.xxx.xx.195`      | Navega sin login (Ventas)    |
| Bloqueado | IP `10.xx.xx.196`       | No puede navegar (Invitado)  |

---

## 🔧 PARTE 2: COMANDOS USADOS EN MIKROTIK

### 1. Renombrar interfaces


```powershell
/interface ethernet set [find default-name=ether1] name=WAN set [find default-name=ether2] name=LAN
```

---

### 2. Asignar IP al LAN



```powershell
/ip address add address=10.128.18.1/24 interface=LAN
```

---

### 3. Configurar DHCP

```powershell 
/ip pool add name=dhcp_pool ranges=10.128.18.100-10.128.18.200
/ip dhcp-server add name=dhcp1 interface=LAN address-pool=dhcp_pool disabled=no 
/ip dhcp-server network add address=10.128.18.0/24 gateway=10.128.18.1 dns-server=1.1.1.1  /ip dhcp-server enable dhcp1
```

---

### 4. Salida a Internet (NAT)

```powershell
/ip firewall nat add chain=srcnat action=masquerade out-interface=WAN`
```
---

### 5. Asistente de Hotspot


```powershell 
/ip hotspot setup
```
- Interface: `LAN`
    
- Address: `10.xxx.xx.1/24`
    
- Pool: `dhcp_pool`
    
- DNS name: `wifi.xxxx.lan`
    
- SMTP: dejar vacío
    
- Create user: sí (admin sin clave)
    

---

### 6. Crear perfiles


```powershell
/ip hotspot user profile
add name=trial1 rate-limit=512k/256k idle-timeout=15m keepalive-timeout=4h 
add name=ventas rate-limit=2M/512k 
add name=marketing rate-limit=3M/1M 
add name=invitados rate-limit=1M/256k
```

---

### 7. Crear usuarios

```powershell
/ip hotspot user 
add name=ventas1 password=itel profile=ventas 
add name=ventas2 password=itel profile=ventas 
add name=marketing1 password=itel profile=marketing 
add name=marketing2 password=itel profile=marketing 
add name=invitado1 password=itel profile=invitados
```

---

### 8. Excepciones (bypass y bloqueos)


```powershell
/ip hotspot ip-binding 
add mac-address=XX:EE:BB:XX:AA:XX type=bypassed comment="MARKETING - BY MAC" 
add address=10.xxx.xx.195 type=bypassed comment="VENTAS - BY IP" 
add address=10.xxx.xxx.196 type=blocked comment="INVITADO BLOQUEADO"`
```
---

### 9. Portal personalizado

```powershell
/ip hotspot profile set [find default=yes] html-directory=xxxx
```

[PORTAL PERSONALIZADO HTML](7-%20HOTSPOT%20HTML)



### 10. PORTAL CAUTIVO (IMAGENES)

![Hostpot](Imagenes/7-Portal%20Cautivo%20en%20Mikrotik/Login.png)
![Hostpot](Imagenes/7-Portal%20Cautivo%20en%20Mikrotik/Status.png)
![Hostpot](Imagenes/7-Portal%20Cautivo%20en%20Mikrotik/Logout.png)
