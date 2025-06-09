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


![Hostpot](Imagenes/6-Inter-VLAN%20Routing%20-%20Router%20on%20a%20Stick%20(Mikrotik%20y%20Cisco)/Esquema%206A.png)

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
/ip hotspot profile set [find default=yes] html-directory=toledo
```


```HTML LOGIN.HTML

<!doctype html>

<html lang="en">

  

<head>

    <meta charset="utf-8">

    <meta http-equiv="pragma" content="no-cache" />

    <meta http-equiv="expires" content="-1" />

    <meta name="viewport" content="width=device-width, initial-scale=1.0" />

    <title>Internet hotspot - Log in</title>

    <style>

        body {

            min-height: 100vh;

            margin: 0;

            font-family: 'Segoe UI', 'Roboto', Arial, sans-serif;

            background: linear-gradient(135deg, #1e3c72 0%, #2a5298 100%);

            display: flex;

            align-items: center;

            justify-content: center;

        }

  

        .main {

            width: 100vw;

            min-height: 100vh;

            display: flex;

            align-items: center;

            justify-content: center;

        }

  

        .wrap {

            background: rgba(255, 255, 255, 0.95);

            border-radius: 18px;

            box-shadow: 0 8px 32px 0 rgba(31, 38, 135, 0.25);

            padding: 2.7rem 2rem 2.2rem 2rem;

            max-width: 370px;

            width: 100%;

            display: flex;

            flex-direction: column;

            align-items: center;

            animation: fadeIn 0.8s;

        }

  

        @keyframes fadeIn {

            from {

                opacity: 0;

                transform: translateY(30px);

            }

  

            to {

                opacity: 1;

                transform: translateY(0);

            }

        }

  

        .logos-container {

            display: flex;

            flex-direction: column;

            align-items: center;

            gap: 0.7rem;

            margin-bottom: 1.7rem;

            width: 100%;

        }

  

        .logo-ext {

            width: 90px;

            max-width: 100%;

            margin: 0;

            border-radius: 12px;

            box-shadow: 0 2px 8px rgba(42, 82, 152, 0.1);

            background: #fff;

            padding: 8px;

        }

  

        .logo-mikrotik {

            width: 110px;

            max-width: 100%;

            margin: 0;

            display: block;

        }

  

        form[name="login"] {

            width: 100%;

            display: flex;

            flex-direction: column;

            align-items: center;

            margin-top: 0.5rem;

        }

  

        .info {

            color: #2a5298;

            font-size: 1rem;

            margin-bottom: 1.5rem;

            text-align: center;

        }

  

        .info.alert {

            color: #d32f2f;

            background: #ffeaea;

            border-radius: 8px;

            padding: 0.5em 0.7em;

        }

  

        label {

            width: 100%;

            display: flex;

            align-items: center;

            background: #f3f6fa;

            border-radius: 8px;

            margin-bottom: 1.1rem;

            padding: 0.4rem 0.8rem;

            border: 1px solid #e0e7ef;

            transition: border 0.2s;

        }

  

        label:focus-within {

            border: 1.5px solid #2a5298;

        }

  

        .ico {

            width: 22px;

            margin-right: 0.7rem;

            opacity: 0.7;

        }

  

        input[type="text"],

        input[type="password"] {

            border: none;

            outline: none;

            background: transparent;

            font-size: 1.08rem;

            width: 100%;

            padding: 0.7rem 0;

            color: #222;

        }

  

        input[type="submit"] {

            width: 100%;

            background: linear-gradient(90deg, #1e3c72 0%, #2a5298 100%);

            color: #fff;

            font-weight: 600;

            font-size: 1.1rem;

            border: none;

            border-radius: 8px;

            padding: 0.8rem 0;

            margin-top: 0.5rem;

            cursor: pointer;

            box-shadow: 0 2px 8px rgba(42, 82, 152, 0.08);

            transition: background 0.2s, transform 0.1s;

        }

  

        input[type="submit"]:hover {

            background: linear-gradient(90deg, #2a5298 0%, #1e3c72 100%);

            transform: translateY(-2px) scale(1.03);

        }

  

        .bt {

            margin-top: 1.7rem;

            font-size: 0.93rem;

            color: #888;

            letter-spacing: 0.03em;

        }

  

        @media (max-width: 480px) {

            .wrap {

                padding: 1.2rem 0.5rem 1.5rem 0.5rem;

                max-width: 98vw;

            }

        }

    </style>

</head>

  

<body>

    <!-- two other colors

  

<body class="lite">

<body class="dark">

  

-->

  

    $(if chap-id)

    <form name="sendin" action="$(link-login-only)" method="post" style="display:none">

        <input type="hidden" name="username" />

        <input type="hidden" name="password" />

        <input type="hidden" name="dst" value="$(link-orig)" />

        <input type="hidden" name="popup" value="true" />

    </form>

  

    <script src="/md5.js"></script>

    <script>

        function doLogin() {

            document.sendin.username.value = document.login.username.value;

            document.sendin.password.value = hexMD5('$(chap-id)' + document.login.password.value + '$(chap-challenge)');

            document.sendin.submit();

            return false;

        }

    </script>

    $(endif)

    <div class="ie-fixMinHeight">

        <div class="main">

            <div class="wrap animated fadeIn">

                <div class="logos-container">

                    <!-- Logo externo proporcionado -->

                    <img class="logo-ext" src="./img/ITEL3.png" alt="Logo" />

                    <!-- Logo MikroTik SVG -->

                    <svg class="logo-mikrotik" data-name="Layer 1" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 174 42">

                        <path fill="#fff" d="M7.32 13.66L0 41.74 3.12 41.74 9.49 15.94 9.58 15.94 15.01 41.74 18.22 41.74 36.86 16.34 36.95 16.34 30.02 41.74 33.18 41.74 40.4 13.66 35.73 13.66 17.23 38.87 11.99 13.66 7.32 13.66zM43.43 21.45L38.19 41.74 41.16 41.74 46.4 21.45 43.43 21.45zM50.68 21.45L45.5 41.74 48.47 41.74 50.36 34.39 55.55 30.77 62.02 41.74 65.27 41.74 57.91 29.28 69.43 21.45 65.46 21.45 51.21 31.36 51.12 31.28 53.66 21.45 50.68 21.45z" />

                        <path d="M71.18 21.45L65.94 41.74h3l2.74-10.62c1-3.81 3.82-7.39 9.16-7.47.56 0 1.13 0 1.7 0l.66-2.48c-.52 0-1.09 0-1.61 0-4.34 0-6.94 2-8.82 5h-.1l1.23-4.68zM103.8 28.8c0-5-4-7.94-9.63-7.94-8.69 0-13.59 6.37-13.59 13 0 5.07 3.44 8.45 9.72 8.45 9 0 13.5-6.68 13.5-13.53m-3 .52c0 4.72-3.44 10.93-9.95 10.93-5 0-7.32-2.68-7.32-6.61 0-4.76 3.59-10.7 10.1-10.7 4.77 0 7.17 2.6 7.17 6.38M132.33 21.43L134.26 13.66 105.19 13.66 103.27 21.45 112.59 21.45 112.59 21.45 122.99 21.45 122.99 21.45 132.33 21.43zM111.67 25.17L107.55 41.74 117.93 41.74 122.06 25.17 122.06 25.16 111.67 25.16 111.67 25.17zM134 25.17l-4.11 16.57h9.35l4.1-16.57zm10.28-3.73l1.94-7.78h-9.34l-2 7.79zM150.09 13.66L143.11 41.74 152.45 41.74 153.91 35.92 156.04 34.34 159.34 41.74 169.49 41.74 163.26 29.55 174.26 21.33 163.07 21.33 156.18 27.3 156.09 27.3 159.44 13.66 150.09 13.66zM47.45 0c1.14 7.93 5.39 12.74 14.07 13.14A10.69 10.69 0 0 1 47.45 0" fill="#fff" fill-rule="evenodd" />

                        <path d="M42.91,1.4c.1,0,.11,0,.12.11A16.55,16.55,0,0,0,48.26,13a16.6,16.6,0,0,0,12,4.66c-10,4-20.55-5.6-17.33-16.28" fill="#fff" fill-rule="evenodd" />

                    </svg>

                </div>

                <form name="login" action="$(link-login-only)" method="post" $(if chap-id) onSubmit="return doLogin()" $(endif)>

                    <input type="hidden" name="dst" value="$(link-orig)" />

                    <input type="hidden" name="popup" value="true" />

                    <p class="info $(if error)alert$(endif)">

                        $(if error == "")Por favor, inicia sesión para usar el servicio de hotspot de internet $(if trial == 'yes')<br />Prueba gratuita disponible, <a href="$(link-login-only)?dst=$(link-orig-esc)&amp;username=T-$(mac-esc)">haz clic aquí</a>.$(endif)

                        $(endif)

  

                        $(if error)$(error)$(endif)

                    </p>

                    <label>

                        <img class="ico" src="img/user.svg" alt="#" />

                        <input name="username" type="text" value="$(username)" placeholder="Usuario" autocomplete="username" />

                    </label>

  

                    <label>

                        <img class="ico" src="img/password.svg" alt="#" />

                        <input name="password" type="password" placeholder="Contraseña" autocomplete="current-password" />

                    </label>

  

                    <input type="submit" value="Conectar" />

  

                </form>

                <p class="info bt">Powered by MikroTik RouterOS</p>

  

            </div>

        </div>

    </div>

</body>

  

</html>

```

```HTML LOGOUT.HTML
<!doctype html>

<html lang="es">

<head>

<meta charset="utf-8">

<meta name="viewport" content="width=device-width, initial-scale=1.0" />  

<meta http-equiv="pragma" content="no-cache">

<meta http-equiv="expires" content="-1">

<title>Internet hotspot - Cierre de sesión</title>

<style>

    body {

        min-height: 100vh;

        margin: 0;

        font-family: 'Segoe UI', 'Roboto', Arial, sans-serif;

        background: linear-gradient(135deg, #1e3c72 0%, #2a5298 100%);

        display: flex;

        align-items: center;

        justify-content: center;

    }

    .main {

        width: 100vw;

        min-height: 100vh;

        display: flex;

        align-items: center;

        justify-content: center;

    }

    .wrap {

        background: rgba(255,255,255,0.95);

        border-radius: 18px;

        box-shadow: 0 8px 32px 0 rgba(31,38,135,0.25);

        padding: 2.7rem 2rem 2.2rem 2rem;

        max-width: 370px;

        width: 100%;

        display: flex;

        flex-direction: column;

        align-items: center;

        animation: fadeIn 0.8s;

    }

    @keyframes fadeIn {

        from { opacity: 0; transform: translateY(30px);}

        to { opacity: 1; transform: translateY(0);}

    }

    .logos-container {

        display: flex;

        flex-direction: column;

        align-items: center;

        gap: 0.7rem;

        margin-bottom: 1.7rem;

        width: 100%;

    }

    .logo-ext {

        width: 90px;

        max-width: 100%;

        margin: 0;

        border-radius: 12px;

        box-shadow: 0 2px 8px rgba(42,82,152,0.10);

        background: #fff;

        padding: 8px;

    }

    .logo-mikrotik {

        width: 110px;

        max-width: 100%;

        margin: 0;

        display: block;

    }

    h1 {

        text-align: center;

        color: #2a5298;

        font-size: 1.35rem;

        margin-bottom: 1.2rem;

        font-weight: 600;

    }

    table {

        border-collapse: collapse;

        width: 100%;

        margin-bottom: 1.2rem;

    }

    table td {

        color: #2a5298;

        border-bottom: 1px solid #e0e7ef;

        padding: 10px 4px 10px 0;

        font-size: 1rem;

    }

    table td:first-child {

        font-weight: 700;

        width: 50%;

    }

    input[type="submit"], button {

        width: 100%;

        background: linear-gradient(90deg, #1e3c72 0%, #2a5298 100%);

        color: #fff;

        font-weight: 600;

        font-size: 1.1rem;

        border: none;

        border-radius: 8px;

        padding: 0.8rem 0;

        margin-top: 0.5rem;

        cursor: pointer;

        box-shadow: 0 2px 8px rgba(42,82,152,0.08);

        transition: background 0.2s, transform 0.1s;

    }

    input[type="submit"]:hover, button:hover {

        background: linear-gradient(90deg, #2a5298 0%, #1e3c72 100%);

        transform: translateY(-2px) scale(1.03);

    }

    .bt {

        margin-top: 1.7rem;

        font-size: 0.93rem;

        color: #888;

        letter-spacing: 0.03em;

        text-align: center;

    }

    @media (max-width: 480px) {

        .wrap {

            padding: 1.2rem 0.5rem 1.5rem 0.5rem;

            max-width: 98vw;

        }

    }

</style>

<script>

    function openLogin() {

        if (window.name != 'hotspot_logout') return true;

        open('$(link-login)', '_blank', '');

        window.close();

        return false;

    }

</script>

</head>

<body>

    <div class="ie-fixMinHeight">

        <div class="main">

            <div class="wrap">

                <div class="logos-container">

                    <img class="logo-ext" src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSDwxNZ3QSX7A6j2fgssJCr13qOfoDpsBIPtA&s" alt="Logo" />

                    <svg class="logo-mikrotik" data-name="Layer 1" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 174 42">

                        <path fill="#fff" d="M7.32 13.66L0 41.74 3.12 41.74 9.49 15.94 9.58 15.94 15.01 41.74 18.22 41.74 36.86 16.34 36.95 16.34 30.02 41.74 33.18 41.74 40.4 13.66 35.73 13.66 17.23 38.87 11.99 13.66 7.32 13.66zM43.43 21.45L38.19 41.74 41.16 41.74 46.4 21.45 43.43 21.45zM50.68 21.45L45.5 41.74 48.47 41.74 50.36 34.39 55.55 30.77 62.02 41.74 65.27 41.74 57.91 29.28 69.43 21.45 65.46 21.45 51.21 31.36 51.12 31.28 53.66 21.45 50.68 21.45z" />

                        <path d="M71.18 21.45L65.94 41.74h3l2.74-10.62c1-3.81 3.82-7.39 9.16-7.47.56 0 1.13 0 1.7 0l.66-2.48c-.52 0-1.09 0-1.61 0-4.34 0-6.94 2-8.82 5h-.1l1.23-4.68zM103.8 28.8c0-5-4-7.94-9.63-7.94-8.69 0-13.59 6.37-13.59 13 0 5.07 3.44 8.45 9.72 8.45 9 0 13.5-6.68 13.5-13.53m-3 .52c0 4.72-3.44 10.93-9.95 10.93-5 0-7.32-2.68-7.32-6.61 0-4.76 3.59-10.7 10.1-10.7 4.77 0 7.17 2.6 7.17 6.38M132.33 21.43L134.26 13.66 105.19 13.66 103.27 21.45 112.59 21.45 112.59 21.45 122.99 21.45 122.99 21.45 132.33 21.43zM111.67 25.17L107.55 41.74 117.93 41.74 122.06 25.17 122.06 25.16 111.67 25.16 111.67 25.17zM134 25.17l-4.11 16.57h9.35l4.1-16.57zm10.28-3.73l1.94-7.78h-9.34l-2 7.79zM150.09 13.66L143.11 41.74 152.45 41.74 153.91 35.92 156.04 34.34 159.34 41.74 169.49 41.74 163.26 29.55 174.26 21.33 163.07 21.33 156.18 27.3 156.09 27.3 159.44 13.66 150.09 13.66zM47.45 0c1.14 7.93 5.39 12.74 14.07 13.14A10.69 10.69 0 0 1 47.45 0" fill="#fff" fill-rule="evenodd" />

                        <path d="M42.91,1.4c.1,0,.11,0,.12.11A16.55,16.55,0,0,0,48.26,13a16.6,16.6,0,0,0,12,4.66c-10,4-20.55-5.6-17.33-16.28" fill="#fff" fill-rule="evenodd" />

                    </svg>

                </div>

                <h1>¡Has cerrado sesión!</h1>

                <table>  

                    <tr><td>Usuario</td><td>$(username)</td></tr>

                    <tr><td>Dirección IP</td><td>$(ip)</td></tr>

                    <tr><td>Dirección MAC</td><td>$(mac)</td></tr>

                    <tr><td>Tiempo de sesión</td><td>$(uptime)</td></tr>

                    $(if session-time-left)

                    <tr><td>Tiempo restante</td><td>$(session-time-left)</td></tr>

                    $(endif)

                    <tr><td>Bytes subidos / bajados</td><td>$(bytes-in-nice) / $(bytes-out-nice)</td></tr>

                </table>

                <form action="$(link-login)" name="login" onSubmit="return openLogin()">

                    <input type="submit" value="Iniciar sesión">

                </form>

                <p class="bt">Desarrollado por MikroTik RouterOS</p>

            </div>

        </div>

    </div>

</body>

</html>

```

```HTML STATUS.HTML
<!doctype html>

<html lang="en">

<head>

    $(if refresh-timeout)

    <meta http-equiv="refresh" content="$(refresh-timeout-secs)">

    $(endif)

    <meta charset="utf-8">

    <meta name="viewport" content="width=device-width, initial-scale=1.0" />  

    <meta http-equiv="pragma" content="no-cache">

    <meta http-equiv="expires" content="-1">

    <title>Internet hotspot - Status</title>

    <style>

        body {

            min-height: 100vh;

            margin: 0;

            font-family: 'Segoe UI', 'Roboto', Arial, sans-serif;

            background: linear-gradient(135deg, #1e3c72 0%, #2a5298 100%);

            display: flex;

            align-items: center;

            justify-content: center;

        }

        .main {

            width: 100vw;

            min-height: 100vh;

            display: flex;

            align-items: center;

            justify-content: center;

        }

        .wrap {

            background: rgba(255,255,255,0.95);

            border-radius: 18px;

            box-shadow: 0 8px 32px 0 rgba(31,38,135,0.25);

            padding: 2.7rem 2rem 2.2rem 2rem;

            max-width: 370px;

            width: 100%;

            display: flex;

            flex-direction: column;

            align-items: center;

            animation: fadeIn 0.8s;

        }

        @keyframes fadeIn {

            from { opacity: 0; transform: translateY(30px);}

            to { opacity: 1; transform: translateY(0);}

        }

        .logos-container {

            display: flex;

            flex-direction: column;

            align-items: center;

            gap: 0.7rem;

            margin-bottom: 1.7rem;

            width: 100%;

        }

        .logo-ext {

            width: 90px;

            max-width: 100%;

            margin: 0;

            border-radius: 12px;

            box-shadow: 0 2px 8px rgba(42,82,152,0.10);

            background: #fff;

            padding: 8px;

        }

        .logo-mikrotik {

            width: 110px;

            max-width: 100%;

            margin: 0;

            display: block;

        }

        h1 {

            text-align: center;

            color: #2a5298;

            font-size: 1.35rem;

            margin-bottom: 1.2rem;

            font-weight: 600;

        }

        table {

            border-collapse: collapse;

            width: 100%;

            margin-bottom: 1.2rem;

        }

        table td {

            color: #2a5298;

            border-bottom: 1px solid #e0e7ef;

            padding: 10px 4px 10px 0;

            font-size: 1rem;

        }

        table td:first-child {

            font-weight: 700;

            width: 50%;

        }

        a {

            color: #2a5298;

            text-decoration: underline;

            transition: color 0.2s;

        }

        a:hover {

            color: #1e3c72;

        }

        input[type="submit"], button {

            width: 100%;

            background: linear-gradient(90deg, #1e3c72 0%, #2a5298 100%);

            color: #fff;

            font-weight: 600;

            font-size: 1.1rem;

            border: none;

            border-radius: 8px;

            padding: 0.8rem 0;

            margin-top: 0.5rem;

            cursor: pointer;

            box-shadow: 0 2px 8px rgba(42,82,152,0.08);

            transition: background 0.2s, transform 0.1s;

        }

        input[type="submit"]:hover, button:hover {

            background: linear-gradient(90deg, #2a5298 0%, #1e3c72 100%);

            transform: translateY(-2px) scale(1.03);

        }

        .bt {

            margin-top: 1.7rem;

            font-size: 0.93rem;

            color: #888;

            letter-spacing: 0.03em;

            text-align: center;

        }

        @media (max-width: 480px) {

            .wrap {

                padding: 1.2rem 0.5rem 1.5rem 0.5rem;

                max-width: 98vw;

            }

        }

    </style>

    <script>

  

$(if advert-pending == 'yes')

    var popup = '';

    function focusAdvert() {

    if (window.focus) popup.focus();

    }

    function openAdvert() {

    popup = open('$(link-advert)', 'hotspot_advert', '');

    setTimeout("focusAdvert()", 1000);

    }

$(endif)

    function openLogout() {

    if (window.name != 'hotspot_status') return true;

        open('$(link-logout)', 'hotspot_logout', 'toolbar=0,location=0,directories=0,status=0,menubars=0,resizable=1,width=280,height=250');

    window.close();

    return false;

    }

</script>

</head>

<body $(if advert-pending == 'yes') onLoad="openAdvert()" $(endif)>

    <div class="ie-fixMinHeight">

        <div class="main">

            <div class="wrap">

                <div class="logos-container">

                    <img class="logo-ext" src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSDwxNZ3QSX7A6j2fgssJCr13qOfoDpsBIPtA&s" alt="Logo" />

                    <svg class="logo-mikrotik" data-name="Layer 1" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 174 42">

                        <path fill="#fff" d="M7.32 13.66L0 41.74 3.12 41.74 9.49 15.94 9.58 15.94 15.01 41.74 18.22 41.74 36.86 16.34 36.95 16.34 30.02 41.74 33.18 41.74 40.4 13.66 35.73 13.66 17.23 38.87 11.99 13.66 7.32 13.66zM43.43 21.45L38.19 41.74 41.16 41.74 46.4 21.45 43.43 21.45zM50.68 21.45L45.5 41.74 48.47 41.74 50.36 34.39 55.55 30.77 62.02 41.74 65.27 41.74 57.91 29.28 69.43 21.45 65.46 21.45 51.21 31.36 51.12 31.28 53.66 21.45 50.68 21.45z" />

                        <path d="M71.18 21.45L65.94 41.74h3l2.74-10.62c1-3.81 3.82-7.39 9.16-7.47.56 0 1.13 0 1.7 0l.66-2.48c-.52 0-1.09 0-1.61 0-4.34 0-6.94 2-8.82 5h-.1l1.23-4.68zM103.8 28.8c0-5-4-7.94-9.63-7.94-8.69 0-13.59 6.37-13.59 13 0 5.07 3.44 8.45 9.72 8.45 9 0 13.5-6.68 13.5-13.53m-3 .52c0 4.72-3.44 10.93-9.95 10.93-5 0-7.32-2.68-7.32-6.61 0-4.76 3.59-10.7 10.1-10.7 4.77 0 7.17 2.6 7.17 6.38M132.33 21.43L134.26 13.66 105.19 13.66 103.27 21.45 112.59 21.45 112.59 21.45 122.99 21.45 122.99 21.45 132.33 21.43zM111.67 25.17L107.55 41.74 117.93 41.74 122.06 25.17 122.06 25.16 111.67 25.16 111.67 25.17zM134 25.17l-4.11 16.57h9.35l4.1-16.57zm10.28-3.73l1.94-7.78h-9.34l-2 7.79zM150.09 13.66L143.11 41.74 152.45 41.74 153.91 35.92 156.04 34.34 159.34 41.74 169.49 41.74 163.26 29.55 174.26 21.33 163.07 21.33 156.18 27.3 156.09 27.3 159.44 13.66 150.09 13.66zM47.45 0c1.14 7.93 5.39 12.74 14.07 13.14A10.69 10.69 0 0 1 47.45 0" fill="#fff" fill-rule="evenodd" />

                        <path d="M42.91,1.4c.1,0,.11,0,.12.11A16.55,16.55,0,0,0,48.26,13a16.6,16.6,0,0,0,12,4.66c-10,4-20.55-5.6-17.33-16.28" fill="#fff" fill-rule="evenodd" />

                    </svg>

                </div>

                $(if login-by == 'trial')

                    <h1>¡Hola, usuario de prueba!</h1>

                $(elif login-by != 'mac')

                    <h1>¡Hola, $(username)!</h1>

                $(endif)

                <form action="$(link-logout)" name="logout" onSubmit="return openLogout()">

                    <table>

                        <tr><td>Dirección IP</td><td>$(ip)</td></tr>

                        <tr><td>Bytes subidos / bajados</td><td>$(bytes-in-nice) / $(bytes-out-nice)</td></tr>

                    $(if session-time-left)

                        <tr><td>Conectado / restante</td><td>$(uptime) / $(session-time-left)</td></tr>

                    $(else)

                        <tr><td>Conectado</td><td>$(uptime)</td></tr>

                    $(endif)

                    $(if blocked == 'yes')

                        <tr><td>Estado</td><td>

                    <a href="$(link-advert)" target="hotspot_advert">Publicidad requerida</a></td>

                        </tr>

                    $(elif refresh-timeout)

                        <tr><td>Actualización de estado</td><td>$(refresh-timeout)</td></tr>

                    $(endif)

                    </table>

                    $(if login-by-mac != 'yes')

                    <input type="submit" value="Cerrar sesión">

                    $(endif)

                </form>

                <p class="bt">Desarrollado por MikroTik RouterOS</p>

            </div>

        </div>

    </div>

</body>

</html>
```

### 10. PORTAL CAUTIVO (IMAGENES)

![[Logout.png]]
![[Login.png]]![[Status.png]]
