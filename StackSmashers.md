## Enumeracion de Puertos, Servicios y Versiones

```ruby
nmap -Pn -n -sS -p- --open -sCV --min-rate 5000 172.17.0.2 -oN nmap
```

![image](https://github.com/user-attachments/assets/ea38b02e-fdc7-47a8-8268-0f16b127756b)

solo observo el puerto 80 (servicio web) y el puerto 22 (ssh) levantados, asi que comienzo chequeando la web

![image](https://github.com/user-attachments/assets/214c0875-108f-4749-8a48-fe5cf06a9fd3)

me descargo la `app` clicando en el enlace y lo pruebo

![image](https://github.com/user-attachments/assets/0556cd60-7ea9-43d8-8fc3-a76f10d09586)

chequeo si llega ser vulnerable a un `bof`

![image](https://github.com/user-attachments/assets/4b69f9c1-f00b-45c6-a1b9-c750ff884337)

ocurrio un error de segmentacion, por lo que podria llegar a ser vulnerable a un buffer overflow

## Explotando BOF

comienzo chequenado el binario a ver sus protecciones y la arquitectura 

![image](https://github.com/user-attachments/assets/24e41b51-3114-4d0d-950d-2eba7162b4c9)

aqui observamos que es un binario de 64 bit y solo tiene la proteccion `PIE` activa, continuo ahora a travez del depurador `gdb`

```ruby
gdb ./appregisters -q # corremos el binario con el depurador
```

ahora reviso las funciones del binario

![image](https://github.com/user-attachments/assets/5dd146d7-1e78-426c-8817-2afb8ef670f8)


ahora calculare el `offset` con 2 exploit de `metasploit`, el primero para crear una cadena especial y desbordar el buffer y el segundo para calcular el `offset`

![image](https://github.com/user-attachments/assets/75c78c02-6d01-4969-85a3-a8ecf5855874)

aqui primero hemos corrido el binario en el depurador, luego creamos la cadena con una logitud de 300 caracteres, despues dicha cadena se la paso como
dato al binario, se desborda el buffer y al buscar la direccion de retorno del `rip` obtengo `0x6141376141366141` que dicho valor ahora se lo pasamos al
segundo exploit de `metasploit` para calcular el offset

![image](https://github.com/user-attachments/assets/1e1cfeda-af1d-4ccd-adce-23be8dbea996)

veo que el `offset` es de 18 byte, por un lado no tiene sentido ejecutar un shellcode y por otro lado cuando se consultaron las funciones pude observar una llamada `boff`
asi que intentare redirigir el flujo del programa a dicha funcion a ver que resulta.... primero valido que pueda controlar el registro `rip` con el `offset` calculado
previamente

![image](https://github.com/user-attachments/assets/f6bcbeab-8d9b-42c2-81dd-66b8dae26058)

observamos que en efecto tenemos control del registro `rip` por lo que ahora consultamos nuevamente la direccion de memoria de la funcion `boff` ya que lo mas seguro
es que alla cambiado

![image](https://github.com/user-attachments/assets/82161e64-dc2f-427c-9c95-f30d22956b4b)

como era de esperar, la direccion de memoria del la funcion cambio, ahora con esta direccion intentare apuntar a ella para cambiar el flujo del programa y ver que corre 
dicha funcion

![image](https://github.com/user-attachments/assets/0f786325-4155-4e6d-a82e-273951df241f)

al apuntar a la funcion `boff` vemos que nos devuelve la cadena `6461726b733a63686f636f6c617469636f7269636f3236` que parece estar en hexadecimal, asi que chequeo

![image](https://github.com/user-attachments/assets/2781011d-d250-4ad6-b33b-d7208a9cd587)

si estaba en hexadecimal y he conseguido credenciales, ahora las probare en el servicio `ssh`

![image](https://github.com/user-attachments/assets/a0b0bf59-c846-4d15-8474-2c2bd194f508)

he conseguido el acceso al sistema!!!

## Escalada de Privilegios

### darks

chequeando, existen 2 usuarios mas en el sistema, `black` & `admin`, en el directorio de `darks` no existe nada (bueno tenemos la flag jejeje), asi que 
busco archivos que pertenezcan al user `admin` y me consigo con un `mail`

![image](https://github.com/user-attachments/assets/d228dc0c-aa67-479c-afe5-72eae3157b61)

vemos que le solicitan al user `darks` que analice el binario `LOCKFIT`, parece de interes asi que lo busco para analizarlo

```ruby
find / -name 'LOCKFIT' 2>/dev/null
```
lo consigo y lo copio al directorio de `darks`

![image](https://github.com/user-attachments/assets/5e60f9b0-303f-43df-b97e-93e4a37ff81b)

lo ejecuto pero parece que no pasa nada, asi que lo envio a mi maquina atacante para analizarlo

![image](https://github.com/user-attachments/assets/12d0d540-3d03-4922-82ac-a4d0150b4963)

ya teniendo el binario en mi maquina comienzo analizarlo

```ruby
strings LOCKFIT
```

![image](https://github.com/user-attachments/assets/fef85f26-d764-4489-84b7-8f41fffd1aaf)

parece que el binario intenta crear un socket y trasmitir informacion, asi que voy a ejecutarlo nuevamente y chequear la red con `wireshark`

![image](https://github.com/user-attachments/assets/5366b9fa-371a-4d14-a989-dcd6408fe102)

aqui puedo ver que tras ejecutar el binario y observar el trafico de red, se esta peguntando por la ip `172.17.0.11`, misma ip que observamos con `strings`
asi que cambiare mi ip por la del binario

```ruby
sudo ip addres del 172.17.0.1/16 dev docker0 # elimino mi actual direccion ip

sudo ip addres add 172.17.0.11/16 dev docker0 # me asigno la nueva ip
```
al cambiar la ip perdere la conexion via ssh, asi que vuelvo a acceder por ssh y ejecuto el binario `LOCKFIT` nuevamente y escanear la red con `wireshark` desde mi
maquina atacante

![image](https://github.com/user-attachments/assets/b8c40683-1c14-489d-b276-bc965e64a502)

ahora puedo observar mucha mas informacion:

```bash
ip origen: 172.17.0.2
ip destino: 172.17.0.11
protocolo: UDP
puerto origen: 45360
puerto destino: 23598
```

ya conociendo el protocolo y el puerto al que envia la informacion puedo colocarme en escucha con netcat a ver que esta transmitiendo el binario

![image](https://github.com/user-attachments/assets/1213fa2a-ad80-4cd6-b9af-36bf8afcb773)

al colocarme en escucha recibo la cadena

```ruby
LJDVM4LCPFBHOYRTJFTVSWCGGFQVGQTUMFME2Z2ZGNFGYWSHKZ2VSMTMNBREOVT2JFCG64CJI5FHGWKXJZZE63SCOJGXURLXJ5KVMRCVPJATKTLKMMYGCRZZOMFFUQLPHUFA====
```

parece ser `base64` asi que intento decodificar


![image](https://github.com/user-attachments/assets/808974b7-396b-4774-b308-fd0f32c82237)

esta codeada en `base32` y `base64`, obtengo credenciales para el user `black`

![image](https://github.com/user-attachments/assets/390f8ea1-d129-4673-b793-8704d190d7cc)

### black

si hacemos un `sudo -l` vemos que se puede ejeecutar el binario `python3` como `root` por lo que para escarlar ejecuto el siguiente comando:

```ruby
sudo /usr/bin/python3 -c 'import os; os.system("/bin/bash")'
```

### root

![image](https://github.com/user-attachments/assets/6d68d71a-4917-4063-a76c-9935c39493f0)







